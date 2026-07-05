## Update

**Modifying existing records in Drift**

---

# What is it?

**Update** operations modify existing records in your database tables. In Drift, you use the `update()` function combined with a companion to change specific fields of one or multiple records. Updates can target individual rows, groups of rows, or all rows in a table.

> **Think of Update like "editing a file"** – you find the right document, make changes to specific fields, and save the updated version.

```dart
// 👇 Basic update
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'),
    age: Value(26),
  ));

// Generated SQL:
// UPDATE users SET name = 'John Updated', age = 26 WHERE id = 1
```

> **What's happening here?**
> - **`update(users)`** – Target the users table
> - **`where()`** – Specify which records to update
> - **`write()`** – Apply the changes
> - **Companion** – Only provided fields are updated

---

# Why does it exist?

- **Modify Data** – Update existing records
- **Partial Updates** – Update specific fields only
- **Batch Updates** – Update multiple records at once
- **Conditional Updates** – Update based on conditions
- **Data Correction** – Fix incorrect data
- **Business Logic** – Apply business rule changes

---

# Basic Updates

> **Simple update operations**

## Update Single Record

```dart
// 👇 Update a user by ID
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Doe'),
    email: Value('john.doe@example.com'),
  ));

// Generated SQL:
// UPDATE users SET name = 'John Doe', email = 'john.doe@example.com' WHERE id = 1
```

## Update Multiple Fields

```dart
// 👇 Update multiple fields
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('Jane Smith'),
    email: Value('jane@example.com'),
    age: Value(28),
    isActive: Value(true),
  ));
```

---

## Partial Update

```dart
// 👇 Update only name (keep everything else)
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('Updated Name'),
    // Other fields use Value.absent() (not changed)
  ));

// Generated SQL:
// UPDATE users SET name = 'Updated Name' WHERE id = 1
// (Other fields unchanged)
```

---

## Update with Condition

```dart
// 👇 Update all inactive users
await (update(users)..where((u) => u.isActive.equals(false)))
  .write(UsersCompanion(
    isActive: Value(true),
    updatedAt: Value(DateTime.now()),
  ));

// 👇 Update users older than 60
await (update(users)..where((u) => u.age > const Variable(60)))
  .write(UsersCompanion(
    status: Value('senior'),
  ));
```

---

## Update All Records

```dart
// 👇 Update all users (use with caution!)
await update(users).write(UsersCompanion(
  status: Value('active'),
  updatedAt: Value(DateTime.now()),
));

// Generated SQL:
// UPDATE users SET status = 'active', updated_at = ...
// (No WHERE clause)
```

---

# Advanced Updates

> **Complex update patterns**

## Pattern 1: Update with Returning

```dart
// 👇 Update and return the updated record
final updatedUser = await (update(users)..where((u) => u.id.equals(1)))
  .writeReturning(UsersCompanion(
    name: Value('Updated Name'),
  ));

print('Updated user: ${updatedUser.name}, ID: ${updatedUser.id}');

// 👇 Update and return specific fields
final result = await (update(users)..where((u) => u.id.equals(1)))
  .writeReturning(
    UsersCompanion(name: Value('Updated Name')),
    returning: (u) => [u.id, u.name],
  );
```

---

## Pattern 2: Batch Update

```dart
// 👇 Batch update multiple records
await into(users).batch((batch) {
  batch.update(
    users,
    UsersCompanion(
      status: Value('active'),
      updatedAt: Value(DateTime.now()),
    ),
    (u) => u.isActive.equals(true),
  );
  
  batch.update(
    users,
    UsersCompanion(
      status: Value('inactive'),
      updatedAt: Value(DateTime.now()),
    ),
    (u) => u.isActive.equals(false),
  );
});

// All updates in one transaction
```

---

## Pattern 3: Conditional Update with Variable

```dart
// 👇 Update with dynamic value
Future<void> updateUserAge(int userId, int newAge) async {
  await (update(users)..where((u) => u.id.equals(userId)))
    .write(UsersCompanion(
      age: Value(newAge),
      updatedAt: Value(DateTime.now()),
    ));
}

// 👇 Update with null
Future<void> clearUserAge(int userId) async {
  await (update(users)..where((u) => u.id.equals(userId)))
    .write(UsersCompanion(
      age: const Value(null),
      updatedAt: Value(DateTime.now()),
    ));
}
```

---

## Pattern 4: Update with Complex Conditions

```dart
// 👇 Complex WHERE conditions
await (update(users)
  ..where((u) => 
    u.age > const Variable(18) &
    u.age < const Variable(30) &
    u.isActive.equals(true) &
    u.isVerified.equals(true)
  ))
  .write(UsersCompanion(
    status: Value('young_active'),
    updatedAt: Value(DateTime.now()),
  ));

// 👇 OR conditions
await (update(users)
  ..where((u) => 
    u.age < const Variable(18) |
    u.age > const Variable(65)
  ))
  .write(UsersCompanion(
    status: Value('special_age'),
  ));
```

---

# Update with Relations

> **Updating related data**

## Update Related Tables

```dart
// 👇 Update user and their posts
Future<void> updateUserWithPosts(int userId, {
  required String userName,
  required String postTitle,
}) async {
  await transaction(() async {
    // 1️⃣ Update user
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(name: Value(userName)));
    
    // 2️⃣ Update user's posts
    await (update(posts)..where((p) => p.userId.equals(userId)))
      .write(PostsCompanion(title: Value(postTitle)));
  });
}
```

---

## Update with Join

```dart
// 👇 Update posts by author condition
// Drift doesn't support JOIN in UPDATE directly
// Use custom SQL for complex updates
await customUpdate(
  '''
  UPDATE posts 
  SET status = 'inactive' 
  WHERE user_id IN (
    SELECT id FROM users WHERE is_active = 0
  )
  '''
).go();
```

---

# Update Validations

> **Validating before update**

```dart
// 👇 Update with validation
Future<void> updateUserWithValidation(int userId, {
  String? name,
  String? email,
  int? age,
}) async {
  // 👇 Validate before update
  if (name != null && name.length < 2) {
    throw ArgumentError('Name must be at least 2 characters');
  }
  
  if (email != null && !email.contains('@')) {
    throw ArgumentError('Invalid email format');
  }
  
  if (age != null && (age < 0 || age > 150)) {
    throw ArgumentError('Age must be between 0 and 150');
  }
  
  // 👇 Check if user exists
  final existing = await (select(users)..where((u) => u.id.equals(userId)))
      .getSingleOrNull();
  if (existing == null) {
    throw Exception('User not found');
  }
  
  // 👇 Perform update
  await (update(users)..where((u) => u.id.equals(userId)))
    .write(UsersCompanion(
      name: name != null ? Value(name) : const Value.absent(),
      email: email != null ? Value(email) : const Value.absent(),
      age: age != null ? Value(age) : const Value.absent(),
      updatedAt: Value(DateTime.now()),
    ));
}
```

---

# Real-World Example

> **Complete e-commerce update system**

```dart
// lib/database/database.dart
@DriftDatabase(tables: [Users, Products, Orders, OrderItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // ==================== USER UPDATES ====================
  
  // 👇 Update user profile
  Future<void> updateUserProfile(int userId, {
    String? username,
    String? email,
    String? fullName,
    String? phoneNumber,
    int? age,
  }) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        username: username != null ? Value(username) : const Value.absent(),
        email: email != null ? Value(email) : const Value.absent(),
        fullName: fullName != null ? Value(fullName) : const Value.absent(),
        phoneNumber: phoneNumber != null ? Value(phoneNumber) : const Value.absent(),
        age: age != null ? Value(age) : const Value.absent(),
        updatedAt: Value(DateTime.now()),
      ));
  }
  
  // 👇 Update user status
  Future<void> updateUserStatus(int userId, bool isActive) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        isActive: Value(isActive),
        updatedAt: Value(DateTime.now()),
      ));
  }
  
  // 👇 Verify user
  Future<void> verifyUser(int userId) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        isVerified: const Value(true),
        updatedAt: Value(DateTime.now()),
      ));
  }
  
  // 👇 Update last login
  Future<void> updateLastLogin(int userId) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        lastLogin: Value(DateTime.now()),
      ));
  }

  // ==================== PRODUCT UPDATES ====================
  
  // 👇 Update product
  Future<void> updateProduct(int productId, {
    String? name,
    String? description,
    double? price,
    int? stock,
    bool? isActive,
  }) async {
    await (update(products)..where((p) => p.id.equals(productId)))
      .write(ProductsCompanion(
        name: name != null ? Value(name) : const Value.absent(),
        description: description != null ? Value(description) : const Value.absent(),
        price: price != null ? Value(price) : const Value.absent(),
        stock: stock != null ? Value(stock) : const Value.absent(),
        isActive: isActive != null ? Value(isActive) : const Value.absent(),
        updatedAt: Value(DateTime.now()),
      ));
  }
  
  // 👇 Update product stock
  Future<void> updateProductStock(int productId, int newStock) async {
    await (update(products)..where((p) => p.id.equals(productId)))
      .write(ProductsCompanion(
        stock: Value(newStock),
        updatedAt: Value(DateTime.now()),
      ));
  }
  
  // 👇 Adjust product stock (increment/decrement)
  Future<void> adjustProductStock(int productId, int adjustment) async {
    final product = await (select(products)
      ..where((p) => p.id.equals(productId)))
      .getSingle();
    
    final newStock = product.stock + adjustment;
    if (newStock < 0) {
      throw Exception('Insufficient stock');
    }
    
    await (update(products)..where((p) => p.id.equals(productId)))
      .write(ProductsCompanion(
        stock: Value(newStock),
        updatedAt: Value(DateTime.now()),
      ));
  }
  
  // 👇 Bulk price update
  Future<void> bulkUpdatePrices(double percentage) async {
    // This requires custom SQL for dynamic calculations
    await customUpdate(
      'UPDATE products SET price = price * ? WHERE is_active = 1',
      variables: [Variable.withDouble(1 + percentage / 100)],
    ).go();
  }

  // ==================== ORDER UPDATES ====================
  
  // 👇 Update order status
  Future<void> updateOrderStatus(int orderId, String newStatus) async {
    final validStatuses = ['pending', 'processing', 'paid', 'shipped', 'delivered', 'cancelled'];
    if (!validStatuses.contains(newStatus)) {
      throw Exception('Invalid status: $newStatus');
    }
    
    final companion = OrdersCompanion(
      status: Value(newStatus),
      shippedDate: newStatus == 'shipped' ? Value(DateTime.now()) : const Value.absent(),
      deliveredDate: newStatus == 'delivered' ? Value(DateTime.now()) : const Value.absent(),
      isPaid: newStatus == 'paid' || newStatus == 'shipped' || newStatus == 'delivered'
          ? const Value(true)
          : const Value.absent(),
      isShipped: newStatus == 'shipped' || newStatus == 'delivered'
          ? const Value(true)
          : const Value.absent(),
      isDelivered: newStatus == 'delivered'
          ? const Value(true)
          : const Value.absent(),
    );
    
    await (update(orders)..where((o) => o.id.equals(orderId)))
      .write(companion);
  }
  
  // 👇 Mark order as paid
  Future<void> markOrderPaid(int orderId, String paymentMethod) async {
    await (update(orders)..where((o) => o.id.equals(orderId)))
      .write(OrdersCompanion(
        isPaid: const Value(true),
        paymentStatus: Value('paid'),
        paymentMethod: Value(paymentMethod),
        status: Value('paid'),
      ));
  }
  
  // 👇 Cancel order with stock restoration
  Future<void> cancelOrder(int orderId) async {
    await transaction(() async {
      // 1️⃣ Get order items
      final items = await (select(orderItems)
        ..where((i) => i.orderId.equals(orderId)))
        .get();
      
      // 2️⃣ Restore stock
      for (final item in items) {
        final product = await (select(products)
          ..where((p) => p.id.equals(item.productId)))
          .getSingle();
        
        await (update(products)..where((p) => p.id.equals(product.id)))
          .write(ProductsCompanion(
            stock: Value(product.stock + item.quantity),
            updatedAt: Value(DateTime.now()),
          ));
      }
      
      // 3️⃣ Update order status
      await (update(orders)..where((o) => o.id.equals(orderId)))
        .write(OrdersCompanion(
          status: Value('cancelled'),
          isPaid: const Value(false),
          paymentStatus: Value('refunded'),
        ));
    });
  }
  
  // 👇 Update shipping info
  Future<void> updateOrderShipping(int orderId, {
    required String trackingNumber,
    required DateTime shippedDate,
  }) async {
    await (update(orders)..where((o) => o.id.equals(orderId)))
      .write(OrdersCompanion(
        trackingNumber: Value(trackingNumber),
        shippedDate: Value(shippedDate),
        isShipped: const Value(true),
        status: Value('shipped'),
      ));
  }

  // ==================== ORDER ITEM UPDATES ====================
  
  // 👇 Update order item quantity
  Future<void> updateOrderItemQuantity(
    int orderId,
    int productId,
    int newQuantity,
  ) async {
    if (newQuantity <= 0) {
      throw Exception('Quantity must be positive');
    }
    
    await transaction(() async {
      // Get current item
      final item = await (select(orderItems)
        ..where((i) => i.orderId.equals(orderId) & i.productId.equals(productId)))
        .getSingle();
      
      // Adjust stock
      final product = await (select(products)
        ..where((p) => p.id.equals(productId)))
        .getSingle();
      
      final stockDiff = item.quantity - newQuantity;
      await (update(products)..where((p) => p.id.equals(productId)))
        .write(ProductsCompanion(
          stock: Value(product.stock + stockDiff),
          updatedAt: Value(DateTime.now()),
        ));
      
      // Update item
      await (update(orderItems)
        ..where((i) => i.orderId.equals(orderId) & i.productId.equals(productId)))
        .write(OrderItemsCompanion(
          quantity: Value(newQuantity),
          total: Value(product.price * newQuantity),
        ));
    });
  }
  
  // ==================== BULK UPDATES ====================
  
  // 👇 Bulk update user status
  Future<void> bulkUpdateUserStatus(List<int> userIds, bool isActive) async {
    await into(users).batch((batch) {
      for (final userId in userIds) {
        batch.update(
          users,
          UsersCompanion(
            isActive: Value(isActive),
            updatedAt: Value(DateTime.now()),
          ),
          (u) => u.id.equals(userId),
        );
      }
    });
  }
  
  // 👇 Bulk product activation
  Future<void> bulkActivateProducts(List<int> productIds) async {
    await into(products).batch((batch) {
      for (final id in productIds) {
        batch.update(
          products,
          ProductsCompanion(
            isActive: const Value(true),
            updatedAt: Value(DateTime.now()),
          ),
          (p) => p.id.equals(id),
        );
      }
    });
  }
}
```

```dart
// lib/ui/pages/profile_page.dart
class ProfilePage extends StatefulWidget {
  final AppDatabase db;
  final int userId;
  
  const ProfilePage({required this.db, required this.userId});
  
  @override
  _ProfilePageState createState() => _ProfilePageState();
}

class _ProfilePageState extends State<ProfilePage> {
  final _formKey = GlobalKey<FormState>();
  late TextEditingController _nameController;
  late TextEditingController _emailController;
  late TextEditingController _ageController;
  bool _isLoading = true;
  bool _isSaving = false;
  
  @override
  void initState() {
    super.initState();
    _nameController = TextEditingController();
    _emailController = TextEditingController();
    _ageController = TextEditingController();
    _loadUser();
  }
  
  Future<void> _loadUser() async {
    setState(() => _isLoading = true);
    
    try {
      final user = await (widget.db.select(widget.db.users)
        ..where((u) => u.id.equals(widget.userId)))
        .getSingle();
      
      setState(() {
        _nameController.text = user.username;
        _emailController.text = user.email;
        _ageController.text = user.age?.toString() ?? '';
        _isLoading = false;
      });
    } catch (e) {
      setState(() => _isLoading = false);
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Error loading profile: $e'),
          backgroundColor: Colors.red,
        ),
      );
    }
  }
  
  Future<void> _saveProfile() async {
    if (!_formKey.currentState!.validate()) return;
    
    setState(() => _isSaving = true);
    
    try {
      final age = int.tryParse(_ageController.text);
      
      await widget.db.updateUserProfile(
        widget.userId,
        username: _nameController.text,
        email: _emailController.text,
        age: age,
      );
      
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Profile updated successfully!'),
          backgroundColor: Colors.green,
        ),
      );
      
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Error: $e'),
          backgroundColor: Colors.red,
        ),
      );
    } finally {
      setState(() => _isSaving = false);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return Scaffold(
        body: Center(child: CircularProgressIndicator()),
      );
    }
    
    return Scaffold(
      appBar: AppBar(title: Text('Edit Profile')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _nameController,
                decoration: InputDecoration(labelText: 'Username'),
                validator: (v) {
                  if (v?.isEmpty ?? true) return 'Username required';
                  if (v!.length < 3) return 'Minimum 3 characters';
                  return null;
                },
              ),
              TextFormField(
                controller: _emailController,
                decoration: InputDecoration(labelText: 'Email'),
                validator: (v) {
                  if (v?.isEmpty ?? true) return 'Email required';
                  if (!v!.contains('@')) return 'Invalid email';
                  return null;
                },
              ),
              TextFormField(
                controller: _ageController,
                decoration: InputDecoration(labelText: 'Age'),
                keyboardType: TextInputType.number,
                validator: (v) {
                  if (v?.isEmpty ?? true) return null;
                  final age = int.tryParse(v!);
                  if (age == null) return 'Must be a number';
                  if (age < 0 || age > 150) return 'Invalid age';
                  return null;
                },
              ),
              SizedBox(height: 16),
              ElevatedButton(
                onPressed: _isSaving ? null : _saveProfile,
                child: _isSaving
                    ? CircularProgressIndicator()
                    : Text('Save Changes'),
              ),
            ],
          ),
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _ageController.dispose();
    super.dispose();
  }
}
```

---

# Best Practices

- **Use partial updates** – Update only changed fields
- **Validate before update** – Ensure data is valid
- **Use `Value.absent()`** – For fields you don't want to change
- **Use transactions** – For related updates
- **Use `writeReturning`** – When you need updated data
- **Add updatedAt timestamps** – Track when records change
- **Use batch updates** – For multiple records
- **Always check existence** – Before updating

---

# Common Mistakes

## Mistake 1: Forgetting WHERE clause

Wrong:
```dart
// 🚫 Updates ALL records (dangerous!)
await update(users).write(UsersCompanion(
  isActive: Value(true),
));
```

Correct:
```dart
// ✅ Always specify condition
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    isActive: Value(true),
  ));
```

## Mistake 2: Using `Value.absent()` incorrectly

Wrong:
```dart
// 🚫 This won't update anything meaningful
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: const Value.absent(), // No effect
  ));
```

Correct:
```dart
// ✅ Only set fields you want to change
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('New Name'), // Actually updates
  ));
```

## Mistake 3: Not handling update errors

Wrong:
```dart
// 🚫 Update fails silently
await update(users).write(companion);
```

Correct:
```dart
// ✅ Handle errors
try {
  await update(users).write(companion);
} catch (e) {
  print('Update failed: $e');
  // Handle error
}
```

---

# Summary

| Method | Use Case | Returns |
|--------|----------|---------|
| `write` | Basic update | Number of rows |
| `writeReturning` | Update with data | Updated records |
| `batch.update` | Multiple updates | Batch results |
| `customUpdate` | Complex SQL | Number of rows |

---

# Next Steps

Now you understand update operations, let's dive deeper:

- [Delete](link) – Deleting records
- [Upsert](link) – Insert or update
- [Batch Operations](link) – Advanced batch patterns

---

# Did You Know?

- **Partial updates only change specified fields** – Others remain unchanged

- **`writeReturning` requires SQLite 3.35+** – Returns updated rows

- **Updates can be chained** – With complex WHERE conditions

- **Batch updates are atomic** – All succeed or fail together

- **Custom SQL can do complex updates** – With JOINs, calculations

- **`updatedAt` timestamps are common** – Track record changes

- **Updates can be reversed** – With proper transaction rollback

- **Updates are type-safe** – Compile-time checking

---
