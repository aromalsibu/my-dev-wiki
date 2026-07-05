## Insert

**Adding new records to your Drift database**

---

# What is it?

**Insert** operations add new records to your database tables. In Drift, you use the `into()` function combined with a companion or insertable object to create new rows. Drift provides multiple ways to insert data, from single records to bulk operations, with full type safety.

> **Think of Insert like "adding a new file to a filing cabinet"** – you create a new record, fill in the required information, and place it in the correct drawer (table).

```dart
// 👇 Basic insert with companion
await into(users).insert(
  UsersCompanion.insert(
    name: 'John Doe',
    email: 'john@example.com',
    age: Value(25),
  ),
);

// Generated SQL:
// INSERT INTO users (name, email, age) VALUES ('John Doe', 'john@example.com', 25)
```

> **What's happening here?**
> - **`into(users)`** – Target the users table
> - **`insert()`** – Perform the insert operation
> - **Companion** – Provides the data to insert
> - **Return Value** – The ID of the new record

---

# Why does it exist?

- **Add New Data** – Create new records in the database
- **Type Safety** – Compile-time validation of data
- **Default Values** – Automatic handling of defaults
- **Return Values** – Get inserted IDs and data
- **Bulk Operations** – Insert multiple records efficiently
- **Validation** – Enforce constraints at database level

---

# Single Record Insert

> **Inserting one record at a time**

## Basic Insert

```dart
// lib/database/database.dart
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  // 👇 Insert a new user
  Future<int> createUser(String name, String email) async {
    return await into(users).insert(
      UsersCompanion.insert(
        name: name,
        email: email,
      ),
    );
  }
}

// Usage
final userId = await db.createUser('John Doe', 'john@example.com');
print('Created user with ID: $userId');
```

## Insert with Optional Fields

```dart
// 👇 Insert with some optional fields
await into(users).insert(
  UsersCompanion.insert(
    name: 'Jane Smith',
    email: 'jane@example.com',
    age: Value(28),        // Optional field
    isActive: Value(true), // Override default
  ),
);

// 👇 Insert with explicit NULL
await into(users).insert(
  UsersCompanion.insert(
    name: 'Bob Wilson',
    email: 'bob@example.com',
    age: const Value(null), // Explicitly NULL
  ),
);
```

---

## Insert with Returning

> **Get back the inserted data**

```dart
// 👇 Insert and return the full record
final insertedUser = await into(users).insertReturning(
  UsersCompanion.insert(
    name: 'John Doe',
    email: 'john@example.com',
    age: Value(25),
  ),
);

print('Inserted user: ${insertedUser.name}, ID: ${insertedUser.id}');

// 👇 Insert and return specific fields
final result = await into(users).insertReturning(
  UsersCompanion.insert(
    name: 'Jane Doe',
    email: 'jane@example.com',
  ),
  returning: (u) => [u.id, u.name], // Only return id and name
);

print('Inserted: ${result.first}');
```

---

# Bulk Insert

> **Inserting multiple records at once**

## Insert All (List)

```dart
// 👇 Insert multiple users
await into(users).insertAll([
  UsersCompanion.insert(
    name: 'User 1',
    email: 'user1@example.com',
  ),
  UsersCompanion.insert(
    name: 'User 2',
    email: 'user2@example.com',
    age: Value(30),
  ),
  UsersCompanion.insert(
    name: 'User 3',
    email: 'user3@example.com',
    isActive: Value(false),
  ),
]);
```

## Insert All with Return

```dart
// 👇 Insert multiple and get all IDs
final ids = await into(users).insertAllReturning(
  [
    UsersCompanion.insert(
      name: 'User 1',
      email: 'user1@example.com',
    ),
    UsersCompanion.insert(
      name: 'User 2',
      email: 'user2@example.com',
    ),
  ],
  returning: (u) => u.id,
);

print('Inserted IDs: $ids');
```

## Batch Insert for Performance

```dart
// 👇 Using batch for large inserts
Future<void> bulkInsertUsers(List<UserData> users) async {
  final batch = await into(users).batch((batch) {
    for (final user in users) {
      batch.insert(
        UsersCompanion.insert(
          name: user.name,
          email: user.email,
          age: Value(user.age),
        ),
      );
    }
  });
  
  print('Inserted ${batch.length} users');
}

class UserData {
  final String name;
  final String email;
  final int? age;
  
  UserData({required this.name, required this.email, this.age});
}
```

---

# Insert with Custom Insertable

> **Using custom insertable objects**

```dart
// 👇 Custom insertable for user registration
class UserRegistration implements Insertable<User> {
  final String username;
  final String email;
  final String password;
  final bool isAdmin;
  
  UserRegistration({
    required this.username,
    required this.email,
    required this.password,
    this.isAdmin = false,
  });
  
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    return {
      'username': Variable(username),
      'email': Variable(email),
      'password_hash': Variable(_hashPassword(password)),
      'is_admin': Variable(isAdmin ? 1 : 0),
      'created_at': Variable(DateTime.now()),
      'is_active': Variable(1),
      'is_verified': Variable(0),
    };
  }
  
  String _hashPassword(String password) {
    // Hash password
    return 'hashed_$password';
  }
}

// Usage
await into(users).insert(
  UserRegistration(
    username: 'john_doe',
    email: 'john@example.com',
    password: 'secure_password',
    isAdmin: true,
  ),
);
```

---

# Insert with Validation

> **Validating data before insert**

```dart
// 👇 Insert with validation
Future<int> createValidatedUser({
  required String name,
  required String email,
  int? age,
}) async {
  // 👇 Validate before insert
  if (name.length < 2) {
    throw ArgumentError('Name must be at least 2 characters');
  }
  
  if (!email.contains('@')) {
    throw ArgumentError('Invalid email format');
  }
  
  if (age != null && (age < 0 || age > 150)) {
    throw ArgumentError('Age must be between 0 and 150');
  }
  
  return await into(users).insert(
    UsersCompanion.insert(
      name: name,
      email: email,
      age: Value(age),
    ),
  );
}
```

---

# Insert with Transaction

> **Atomic multiple inserts**

```dart
// 👇 Insert order with items in transaction
Future<int> createOrderWithItems({
  required int userId,
  required List<OrderItem> items,
}) async {
  return await transaction(() async {
    // 1️⃣ Insert order
    final orderId = await into(orders).insert(
      OrdersCompanion.insert(
        userId: userId,
        total: _calculateTotal(items),
        status: 'pending',
      ),
    );
    
    // 2️⃣ Insert order items
    for (final item in items) {
      await into(orderItems).insert(
        OrderItemsCompanion.insert(
          orderId: orderId,
          productId: item.productId,
          quantity: item.quantity,
          price: item.price,
        ),
      );
      
      // 3️⃣ Update product stock
      await (update(products)..where((p) => p.id.equals(item.productId)))
        .write(ProductsCompanion(
          stock: Value(item.product.stock - item.quantity),
        ));
    }
    
    return orderId;
  });
}

double _calculateTotal(List<OrderItem> items) {
  return items.fold(0.0, (sum, item) => sum + item.price * item.quantity);
}
```

---

# Insert with Conflict Resolution

> **Handling duplicate records**

```dart
// 👇 Insert or ignore (skip duplicates)
await into(users).insert(
  UsersCompanion.insert(
    email: 'john@example.com',
    name: 'John',
  ),
  onConflict: DoUpdate(
    target: users.email,
    update: UsersCompanion(
      name: Value('John Updated'),
      updatedAt: Value(DateTime.now()),
    ),
  ),
);

// 👇 Insert or replace
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1), // Attempt to insert with existing ID
    name: 'John',
    email: 'john@example.com',
  ),
  onConflict: DoNothing(), // Skip if conflict
);

// 👇 Insert or update (upsert)
await into(users).insert(
  UsersCompanion.insert(
    email: 'john@example.com',
    name: 'John',
    age: Value(30),
  ),
  onConflict: DoUpdate(
    target: users.email,
    update: UsersCompanion(
      name: Value('John Updated'),
      age: Value(31),
      updatedAt: Value(DateTime.now()),
    ),
  ),
);
```

---

# Insert with Custom Data Class

> **Using custom data classes for insert**

```dart
// 👇 Custom data class with insert method
class AppUser {
  final String name;
  final String email;
  final int? age;
  
  AppUser({
    required this.name,
    required this.email,
    this.age,
  });
  
  // 👇 Insert method
  Future<int> insert(AppDatabase db) async {
    return await db.into(db.users).insert(
      UsersCompanion.insert(
        name: name,
        email: email,
        age: Value(age),
      ),
    );
  }
  
  // 👇 Insert with validation
  Future<int> insertValidated(AppDatabase db) async {
    if (name.length < 2) {
      throw ArgumentError('Name too short');
    }
    if (!email.contains('@')) {
      throw ArgumentError('Invalid email');
    }
    return await insert(db);
  }
}

// Usage
final user = AppUser(
  name: 'John Doe',
  email: 'john@example.com',
  age: 25,
);

final userId = await user.insert(db);
```

---

# Real-World Example

> **Complete e-commerce insert system**

```dart
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get fullName => text().nullable()();
  TextColumn get passwordHash => text()();
  IntColumn get age => integer().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  BoolColumn get isVerified => boolean().withDefault(const Constant(false))();
  BoolColumn get isAdmin => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
  DateTimeColumn get lastLogin => dateTime().nullable()();
}

// lib/database/database.dart
@DriftDatabase(tables: [Users, Products, Orders, OrderItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // ==================== USER INSERTS ====================
  
  // 👇 Simple user insert
  Future<int> createUser({
    required String username,
    required String email,
    required String password,
    String? fullName,
    int? age,
    bool isAdmin = false,
  }) async {
    return await into(users).insert(
      UsersCompanion.insert(
        username: username,
        email: email,
        passwordHash: _hashPassword(password),
        fullName: Value(fullName),
        age: Value(age),
        isAdmin: Value(isAdmin),
      ),
    );
  }
  
  // 👇 User insert with validation
  Future<int> createValidatedUser({
    required String username,
    required String email,
    required String password,
    String? fullName,
    int? age,
  }) async {
    // Validate
    if (username.length < 3) {
      throw Exception('Username must be at least 3 characters');
    }
    
    if (!email.contains('@')) {
      throw Exception('Invalid email format');
    }
    
    if (password.length < 8) {
      throw Exception('Password must be at least 8 characters');
    }
    
    if (age != null && (age < 0 || age > 150)) {
      throw Exception('Age must be between 0 and 150');
    }
    
    // Check if email exists
    final existing = await (select(users)
      ..where((u) => u.email.equals(email)))
      .getSingleOrNull();
    if (existing != null) {
      throw Exception('Email already registered');
    }
    
    return await createUser(
      username: username,
      email: email,
      password: password,
      fullName: fullName,
      age: age,
    );
  }
  
  // 👇 Bulk user insert
  Future<List<int>> createUsers(List<UserInput> usersData) async {
    final companions = usersData.map((user) => 
      UsersCompanion.insert(
        username: user.username,
        email: user.email,
        passwordHash: _hashPassword(user.password),
        fullName: Value(user.fullName),
        age: Value(user.age),
      )
    ).toList();
    
    return await into(users).insertAllReturning(
      companions,
      returning: (u) => u.id,
    );
  }

  // ==================== PRODUCT INSERTS ====================
  
  // 👇 Product insert
  Future<int> createProduct({
    required String sku,
    required String name,
    required double price,
    String? description,
    int stock = 0,
    List<String> tags = const [],
  }) async {
    return await into(products).insert(
      ProductsCompanion.insert(
        sku: sku,
        name: name,
        price: price,
        description: Value(description),
        stock: Value(stock),
        tags: Value(tags.join(',')),
      ),
    );
  }
  
  // 👇 Bulk product insert
  Future<List<int>> createProducts(List<ProductInput> productsData) async {
    final companions = productsData.map((product) =>
      ProductsCompanion.insert(
        sku: product.sku,
        name: product.name,
        price: product.price,
        description: Value(product.description),
        stock: Value(product.stock),
      )
    ).toList();
    
    return await into(products).insertAllReturning(
      companions,
      returning: (p) => p.id,
    );
  }

  // ==================== ORDER INSERTS ====================
  
  // 👇 Order with items (transaction)
  Future<int> createOrder({
    required int userId,
    required List<CartItem> cartItems,
    required String shippingAddress,
    required String billingAddress,
    required String paymentMethod,
  }) async {
    return await transaction(() async {
      // 1️⃣ Calculate totals
      double subtotal = 0;
      final items = <OrderItemDetail>[];
      
      for (final cartItem in cartItems) {
        final product = await (select(products)
          ..where((p) => p.id.equals(cartItem.productId)))
          .getSingle();
        
        if (product.stock < cartItem.quantity) {
          throw Exception('Insufficient stock for ${product.name}');
        }
        
        final itemTotal = product.price * cartItem.quantity;
        subtotal += itemTotal;
        
        items.add(OrderItemDetail(
          productId: product.id,
          quantity: cartItem.quantity,
          price: product.price,
          total: itemTotal,
        ));
      }
      
      final tax = subtotal * 0.1; // 10% tax
      final total = subtotal + tax;
      
      // 2️⃣ Create order
      final orderId = await into(orders).insert(
        OrdersCompanion.insert(
          orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
          userId: userId,
          subtotal: subtotal,
          tax: tax,
          shippingCost: 0.0,
          discount: 0.0,
          total: total,
          status: 'pending',
          shippingAddress: shippingAddress,
          billingAddress: billingAddress,
          paymentMethod: paymentMethod,
          paymentStatus: 'unpaid',
        ),
      );
      
      // 3️⃣ Create order items
      for (final item in items) {
        await into(orderItems).insert(
          OrderItemsCompanion.insert(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: item.price,
            subtotal: item.total,
            total: item.total,
          ),
        );
        
        // 4️⃣ Update stock
        final product = await (select(products)
          ..where((p) => p.id.equals(item.productId)))
          .getSingle();
        
        await (update(products)..where((p) => p.id.equals(product.id)))
          .write(ProductsCompanion(
            stock: Value(product.stock - item.quantity),
          ));
      }
      
      return orderId;
    });
  }
  
  // 👇 Insert order with returning
  Future<Order> createOrderReturning({
    required int userId,
    required List<CartItem> cartItems,
    required String shippingAddress,
  }) async {
    // ... calculate totals similar to above
    
    return await into(orders).insertReturning(
      OrdersCompanion.insert(
        orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
        userId: userId,
        subtotal: subtotal,
        tax: tax,
        total: total,
        status: 'pending',
        shippingAddress: shippingAddress,
        billingAddress: shippingAddress,
        paymentMethod: 'cash',
        paymentStatus: 'unpaid',
      ),
    );
  }

  // ==================== HELPER METHODS ====================
  
  String _hashPassword(String password) {
    // Simulate password hashing
    return 'hashed_$password';
  }
}

// ==================== DATA CLASSES ====================

class UserInput {
  final String username;
  final String email;
  final String password;
  final String? fullName;
  final int? age;
  
  UserInput({
    required this.username,
    required this.email,
    required this.password,
    this.fullName,
    this.age,
  });
}

class ProductInput {
  final String sku;
  final String name;
  final double price;
  final String? description;
  final int stock;
  
  ProductInput({
    required this.sku,
    required this.name,
    required this.price,
    this.description,
    this.stock = 0,
  });
}

class CartItem {
  final int productId;
  final int quantity;
  
  CartItem({
    required this.productId,
    required this.quantity,
  });
}

class OrderItemDetail {
  final int productId;
  final int quantity;
  final double price;
  final double total;
  
  OrderItemDetail({
    required this.productId,
    required this.quantity,
    required this.price,
    required this.total,
  });
}
```

```dart
// lib/ui/pages/checkout_page.dart
class CheckoutPage extends StatefulWidget {
  final AppDatabase db;
  final int userId;
  final List<CartItem> items;
  
  const CheckoutPage({
    required this.db,
    required this.userId,
    required this.items,
  });
  
  @override
  _CheckoutPageState createState() => _CheckoutPageState();
}

class _CheckoutPageState extends State<CheckoutPage> {
  final _formKey = GlobalKey<FormState>();
  final _addressController = TextEditingController();
  final _paymentMethodController = TextEditingController();
  bool _isSubmitting = false;
  
  Future<void> _submitOrder() async {
    if (!_formKey.currentState!.validate()) return;
    
    setState(() => _isSubmitting = true);
    
    try {
      final orderId = await widget.db.createOrder(
        userId: widget.userId,
        cartItems: widget.items,
        shippingAddress: _addressController.text,
        billingAddress: _addressController.text,
        paymentMethod: _paymentMethodController.text,
      );
      
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Order #$orderId created successfully!'),
          backgroundColor: Colors.green,
        ),
      );
      
      Navigator.pop(context, true);
      
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Error: $e'),
          backgroundColor: Colors.red,
        ),
      );
    } finally {
      setState(() => _isSubmitting = false);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Checkout')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _addressController,
                decoration: InputDecoration(labelText: 'Shipping Address'),
                validator: (v) => v?.isEmpty ?? true ? 'Required' : null,
              ),
              TextFormField(
                controller: _paymentMethodController,
                decoration: InputDecoration(labelText: 'Payment Method'),
                validator: (v) => v?.isEmpty ?? true ? 'Required' : null,
              ),
              SizedBox(height: 16),
              // Items summary
              Expanded(
                child: ListView.builder(
                  itemCount: widget.items.length,
                  itemBuilder: (context, index) {
                    final item = widget.items[index];
                    return ListTile(
                      title: Text('Product #${item.productId}'),
                      subtitle: Text('Qty: ${item.quantity}'),
                      trailing: Text('\$${(item.quantity * 10.0).toStringAsFixed(2)}'),
                    );
                  },
                ),
              ),
              SizedBox(height: 16),
              ElevatedButton(
                onPressed: _isSubmitting ? null : _submitOrder,
                child: _isSubmitting
                    ? CircularProgressIndicator()
                    : Text('Place Order'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

# Best Practices

- **Use transactions** – For related inserts
- **Validate before insert** – Catch errors early
- **Use returning** – When you need inserted data
- **Use bulk insert** – For multiple records
- **Handle constraints** – Use onConflict when needed
- **Use custom insertable** – For complex logic
- **Check for duplicates** – Prevent duplicates
- **Log insert operations** – For debugging

---

# Common Mistakes

## Mistake 1: Not handling return value

Wrong:
```dart
// 🚫 Ignoring inserted ID
await into(users).insert(companion);
```

Correct:
```dart
// ✅ Use inserted ID
final id = await into(users).insert(companion);
print('Inserted ID: $id');
```

## Mistake 2: Not using transactions

Wrong:
```dart
// 🚫 Partial insert if one fails
await into(orders).insert(order);
await into(orderItems).insertAll(items); // If fails, order orphaned
```

Correct:
```dart
// ✅ Use transaction
await transaction(() async {
  await into(orders).insert(order);
  await into(orderItems).insertAll(items);
});
```

## Mistake 3: Not handling validation

Wrong:
```dart
// 🚫 Invalid data inserted
await into(users).insert(
  UsersCompanion.insert(
    name: '', // Empty name allowed
    email: 'invalid', // Invalid email
  ),
);
```

Correct:
```dart
// ✅ Validate before insert
if (name.isEmpty) throw Exception('Name required');
if (!email.contains('@')) throw Exception('Invalid email');
await into(users).insert(companion);
```

---

# Summary

| Method | Use Case | Returns |
|--------|----------|---------|
| `insert` | Single record | Inserted ID |
| `insertReturning` | Single with data | Full record |
| `insertAll` | Multiple records | List of IDs |
| `insertAllReturning` | Multiple with data | Full records |
| `batch` | Large inserts | Insert count |

---

# Next Steps

Now you understand insert operations, let's dive deeper:

- [Batch Insert](link) – Advanced bulk operations
- [Update](link) – Updating existing records
- [Delete](link) – Deleting records

---

# Did You Know?

- **Insert returns the last inserted row ID** – For auto-increment columns

- **`insertReturning` works with SQLite 3.35+** – Returns full inserted rows

- **Batch inserts are much faster** – For 100+ records

- **Transactions ensure atomicity** – All or nothing operations

- **Conflict resolution handles duplicates** – `DoUpdate`, `DoNothing`

- **Custom insertables can validate** – Before insertion

- **Insert operations are type-safe** – Compile-time checking

- **You can insert from any data source** – API, forms, files

---
