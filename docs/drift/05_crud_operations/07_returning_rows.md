## Returning Rows

**Getting data back from insert, update, and delete operations**

---

# What is it?

**Returning Rows** is a feature that allows you to retrieve the data from records that were just inserted, updated, or deleted. Instead of just getting a row count or ID, you can get the full records back, including generated values like auto-increment IDs and default timestamps.

> **Think of Returning Rows like "getting a receipt"** – after you make a change, you get a copy of what was changed, including any auto-generated information like the receipt number.

```dart
// 👇 Insert and return the full record
final user = await into(users).insertReturning(
  UsersCompanion.insert(
    name: 'John Doe',
    email: 'john@example.com',
  ),
);

print('Inserted user: ${user.id}, ${user.name}, ${user.createdAt}');

// 👇 Update and return the updated record
final updatedUser = await (update(users)..where((u) => u.id.equals(1)))
  .writeReturning(
    UsersCompanion(name: Value('John Updated')),
  );

print('Updated: ${updatedUser.name}');
```

> **What's happening here?**
> - **`insertReturning()`** – Insert and return the record
> - **`writeReturning()`** – Update and return the record
> - **`returning()`** – Delete and return the record
> - **Full data** – All columns of the affected rows

---

# Why does it exist?

- **Get Generated Values** – Auto-increment IDs, timestamps
- **Avoid Additional Queries** – No need to select after changes
- **Audit Trails** – Log exactly what was changed
- **UI Updates** – Refresh UI with new data
- **Data Validation** – Verify what was stored
- **Chain Operations** – Use returned data for next steps

---

# Insert with Returning

> **Getting data back from inserts**

## Insert Returning Full Record

```dart
// 👇 Insert and get the full user record
final user = await into(users).insertReturning(
  UsersCompanion.insert(
    name: 'John Doe',
    email: 'john@example.com',
    age: Value(25),
  ),
);

print('ID: ${user.id}');
print('Name: ${user.name}');
print('Created at: ${user.createdAt}'); // Auto-generated timestamp

// Generated SQL:
// INSERT INTO users (name, email, age, created_at)
// VALUES ('John Doe', 'john@example.com', 25, CURRENT_TIMESTAMP)
// RETURNING id, name, email, age, created_at
```

## Insert Returning Specific Fields

```dart
// 👇 Only return specific fields
final result = await into(users).insertReturning(
  UsersCompanion.insert(
    name: 'Jane Smith',
    email: 'jane@example.com',
  ),
  returning: (u) => [u.id, u.name, u.createdAt],
);

final id = result[0] as int;
final name = result[1] as String;
final createdAt = result[2] as DateTime;

print('Inserted: $name (ID: $id) at $createdAt');
```

---

## Bulk Insert with Returning

```dart
// 👇 Insert multiple and return all
final users = await into(users).insertAllReturning([
  UsersCompanion.insert(name: 'User 1', email: 'user1@example.com'),
  UsersCompanion.insert(name: 'User 2', email: 'user2@example.com'),
  UsersCompanion.insert(name: 'User 3', email: 'user3@example.com'),
]);

for (final user in users) {
  print('Created user: ${user.name} (ID: ${user.id})');
}
```

## Insert with Returning Using insertReturningOrNull

```dart
// 👇 Insert and return null if no data (for insertAll)
final user = await into(users).insertReturningOrNull(
  UsersCompanion.insert(
    name: 'John Doe',
    email: 'john@example.com',
  ),
);

if (user != null) {
  print('Inserted: ${user.name}');
}
```

---

# Update with Returning

> **Getting data back from updates**

## Update Returning Full Record

```dart
// 👇 Update and get the updated record
final updatedUser = await (update(users)..where((u) => u.id.equals(1)))
  .writeReturning(
    UsersCompanion(
      name: Value('John Updated'),
      age: Value(26),
      updatedAt: Value(DateTime.now()),
    ),
  );

print('Updated: ${updatedUser.name}, Age: ${updatedUser.age}');

// Generated SQL:
// UPDATE users SET name = 'John Updated', age = 26, updated_at = ...
// WHERE id = 1
// RETURNING id, name, email, age, updated_at
```

## Update Returning Specific Fields

```dart
// 👇 Only return specific fields
final result = await (update(users)..where((u) => u.id.equals(1)))
  .writeReturning(
    UsersCompanion(name: Value('John Updated')),
    returning: (u) => [u.id, u.name, u.updatedAt],
  );

final id = result[0] as int;
final name = result[1] as String;
final updatedAt = result[2] as DateTime;
```

---

## Batch Update Returning

```dart
// 👇 Update multiple and return results
final updatedUsers = await into(users).batchWriteReturning(
  (batch) {
    batch.update(
      users,
      UsersCompanion(
        isActive: Value(false),
        updatedAt: Value(DateTime.now()),
      ),
      (u) => u.isActive.equals(true),
    );
  },
  returning: (u) => [u.id, u.name],
);

for (final result in updatedUsers) {
  final id = result[0] as int;
  final name = result[1] as String;
  print('Deactivated: $name (ID: $id)');
}
```

---

# Delete with Returning

> **Getting data back from deletes**

## Delete Returning Full Record

```dart
// 👇 Delete and get the deleted record
final deletedUsers = await (delete(users)..where((u) => u.id.equals(1)))
  .returning();

if (deletedUsers.isNotEmpty) {
  final deletedUser = deletedUsers.first;
  print('Deleted: ${deletedUser.name} (ID: ${deletedUser.id})');
}

// Generated SQL:
// DELETE FROM users WHERE id = 1
// RETURNING id, name, email, age, created_at
```

## Delete Returning Specific Fields

```dart
// 👇 Only return specific fields
final results = await (delete(users)..where((u) => u.id.equals(1)))
  .returning(returning: (u) => [u.id, u.name]);

final id = results[0][0] as int;
final name = results[0][1] as String;

print('Deleted: $name (ID: $id)');
```

---

## Bulk Delete Returning

```dart
// 👇 Delete multiple and return results
final deletedUsers = await (delete(users)
  ..where((u) => u.isActive.equals(false)))
  .returning();

for (final user in deletedUsers) {
  print('Removed inactive user: ${user.name}');
}

// 👇 Delete with returning specific fields
final results = await (delete(users)
  ..where((u) => u.createdAt < const Variable(
    DateTime.now().subtract(Duration(days: 365))
  )))
  .returning(returning: (u) => [u.id, u.name, u.createdAt]);
```

---

# Real-World Example

> **Complete e-commerce returning system**

```dart
// lib/database/database.dart
@DriftDatabase(tables: [Users, Products, Orders, OrderItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // ==================== INSERT WITH RETURNING ====================
  
  // 👇 Create user and get full record
  Future<User> createUserReturning({
    required String name,
    required String email,
    int? age,
  }) async {
    return await into(users).insertReturning(
      UsersCompanion.insert(
        name: name,
        email: email,
        age: Value(age),
        isActive: Value(true),
      ),
    );
  }
  
  // 👇 Create user and get specific fields
  Future<Map<String, dynamic>> createUserReturningId({
    required String name,
    required String email,
  }) async {
    final result = await into(users).insertReturning(
      UsersCompanion.insert(
        name: name,
        email: email,
      ),
      returning: (u) => [u.id, u.name, u.createdAt],
    );
    
    return {
      'id': result[0],
      'name': result[1],
      'createdAt': result[2],
    };
  }
  
  // 👇 Create multiple users and get all
  Future<List<User>> createUsersReturning(List<UserInput> users) async {
    final companions = users.map((user) =>
      UsersCompanion.insert(
        name: user.name,
        email: user.email,
        age: Value(user.age),
      )
    ).toList();
    
    return await into(users).insertAllReturning(companions);
  }
  
  // 👇 Create order with returning
  Future<Order> createOrderReturning({
    required int userId,
    required List<OrderItemInput> items,
  }) async {
    return await transaction(() async {
      // Calculate total
      final total = items.fold(
        0.0, 
        (sum, item) => sum + item.price * item.quantity,
      );
      
      // Create order with returning
      final order = await into(orders).insertReturning(
        OrdersCompanion.insert(
          orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
          userId: userId,
          total: total,
          status: 'pending',
          shippingAddress: '',
          billingAddress: '',
          paymentMethod: 'cash',
          paymentStatus: 'unpaid',
          isPaid: false,
          isShipped: false,
          isDelivered: false,
        ),
      );
      
      // Create order items
      for (final item in items) {
        await into(orderItems).insert(
          OrderItemsCompanion.insert(
            orderId: order.id,
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: item.price,
            subtotal: item.price * item.quantity,
            total: item.price * item.quantity,
          ),
        );
      }
      
      return order;
    });
  }

  // ==================== UPDATE WITH RETURNING ====================
  
  // 👇 Update user and get full record
  Future<User?> updateUserReturning({
    required int userId,
    String? name,
    String? email,
    int? age,
  }) async {
    return await (update(users)..where((u) => u.id.equals(userId)))
      .writeReturning(
        UsersCompanion(
          name: name != null ? Value(name) : const Value.absent(),
          email: email != null ? Value(email) : const Value.absent(),
          age: age != null ? Value(age) : const Value.absent(),
          updatedAt: Value(DateTime.now()),
        ),
      );
  }
  
  // 👇 Update user status with returning
  Future<User?> updateUserStatusReturning({
    required int userId,
    required bool isActive,
  }) async {
    return await (update(users)..where((u) => u.id.equals(userId)))
      .writeReturning(
        UsersCompanion(
          isActive: Value(isActive),
          updatedAt: Value(DateTime.now()),
        ),
      );
  }
  
  // 👇 Update product stock with returning
  Future<Product?> updateProductStockReturning({
    required int productId,
    required int newStock,
  }) async {
    if (newStock < 0) {
      throw Exception('Stock cannot be negative');
    }
    
    return await (update(products)..where((p) => p.id.equals(productId)))
      .writeReturning(
        ProductsCompanion(
          stock: Value(newStock),
          updatedAt: Value(DateTime.now()),
        ),
      );
  }
  
  // 👇 Update order status with returning
  Future<Order?> updateOrderStatusReturning({
    required int orderId,
    required String newStatus,
  }) async {
    final validStatuses = ['pending', 'processing', 'paid', 'shipped', 'delivered', 'cancelled'];
    if (!validStatuses.contains(newStatus)) {
      throw Exception('Invalid status: $newStatus');
    }
    
    return await (update(orders)..where((o) => o.id.equals(orderId)))
      .writeReturning(
        OrdersCompanion(
          status: Value(newStatus),
          shippedDate: newStatus == 'shipped' 
              ? Value(DateTime.now()) 
              : const Value.absent(),
          deliveredDate: newStatus == 'delivered'
              ? Value(DateTime.now())
              : const Value.absent(),
          isPaid: newStatus == 'paid' || newStatus == 'shipped' 
              ? const Value(true)
              : const Value.absent(),
          isShipped: newStatus == 'shipped' || newStatus == 'delivered'
              ? const Value(true)
              : const Value.absent(),
          isDelivered: newStatus == 'delivered'
              : const Value(true)
              : const Value.absent(),
          updatedAt: Value(DateTime.now()),
        ),
      );
  }

  // ==================== DELETE WITH RETURNING ====================
  
  // 👇 Delete user and get deleted record
  Future<User?> deleteUserReturning(int userId) async {
    final results = await (delete(users)..where((u) => u.id.equals(userId)))
      .returning();
    return results.isNotEmpty ? results.first : null;
  }
  
  // 👇 Delete user and get specific fields
  Future<Map<String, dynamic>?> deleteUserReturningInfo(int userId) async {
    final results = await (delete(users)..where((u) => u.id.equals(userId)))
      .returning(returning: (u) => [u.id, u.name, u.email]);
    
    if (results.isEmpty) return null;
    
    return {
      'id': results[0][0],
      'name': results[0][1],
      'email': results[0][2],
    };
  }
  
  // 👇 Delete inactive users and return count
  Future<List<User>> deleteInactiveUsersReturning() async {
    return await (delete(users)
      ..where((u) => u.isActive.equals(false))
      ..where((u) => u.isVerified.equals(false)))
      .returning();
  }
  
  // 👇 Delete old orders with items (with returning)
  Future<List<Order>> deleteOldOrdersReturning(Duration olderThan) async {
    final cutoffDate = DateTime.now().subtract(olderThan);
    
    final orders = await (delete(orders)
      ..where((o) => 
        o.status.equals('delivered') |
        o.status.equals('cancelled')
      )
      ..where((o) => o.orderDate < const Variable(cutoffDate)))
      .returning();
    
    // Also delete associated items (cascade will handle)
    for (final order in orders) {
      await (delete(orderItems)..where((i) => i.orderId.equals(order.id))).go();
    }
    
    return orders;
  }

  // ==================== COMPLEX SCENARIOS ====================
  
  // 👇 Create order with returning and use data
  Future<void> createOrderAndNotify({
    required int userId,
    required List<OrderItemInput> items,
  }) async {
    final order = await createOrderReturning(
      userId: userId,
      items: items,
    );
    
    // Send notification with order data
    await _sendOrderNotification(
      orderId: order.id,
      orderNumber: order.orderNumber,
      total: order.total,
      userId: userId,
    );
    
    // Update analytics
    await _updateAnalytics(order);
  }
  
  // 👇 Update and use returned data
  Future<void> processOrderDelivery(int orderId) async {
    final order = await updateOrderStatusReturning(
      orderId: orderId,
      newStatus: 'delivered',
    );
    
    if (order != null) {
      // Send delivery confirmation
      await _sendDeliveryConfirmation(
        orderId: order.id,
        deliveredDate: order.deliveredDate!,
      );
      
      // Update user's order history
      await _updateUserHistory(order.userId);
    }
  }
  
  // 👇 Delete and use returned data
  Future<void> processUserDeletion(int userId) async {
    final user = await deleteUserReturning(userId);
    
    if (user != null) {
      // Log deletion for audit
      await _logUserDeletion(user);
      
      // Send notification
      await _sendUserDeletedNotification(user);
      
      // Clean up related data
      await _cleanupUserData(userId);
    }
  }
  
  // ==================== HELPER METHODS ====================
  
  Future<void> _sendOrderNotification({
    required int orderId,
    required String orderNumber,
    required double total,
    required int userId,
  }) async {
    print('📧 Order $orderNumber created for user $userId');
    print('Total: \$${total.toStringAsFixed(2)}');
  }
  
  Future<void> _updateAnalytics(Order order) async {
    print('📊 Analytics updated for order ${order.id}');
  }
  
  Future<void> _sendDeliveryConfirmation({
    required int orderId,
    required DateTime deliveredDate,
  }) async {
    print('📦 Order $orderId delivered on $deliveredDate');
  }
  
  Future<void> _updateUserHistory(int userId) async {
    print('👤 User history updated for $userId');
  }
  
  Future<void> _logUserDeletion(User user) async {
    print('🗑️ User ${user.name} (ID: ${user.id}) deleted for audit');
  }
  
  Future<void> _sendUserDeletedNotification(User user) async {
    print('📧 User ${user.name} deletion notified');
  }
  
  Future<void> _cleanupUserData(int userId) async {
    print('🧹 Cleanup completed for user $userId');
  }
}

// ==================== DATA CLASSES ====================

class UserInput {
  final String name;
  final String email;
  final int? age;
  
  UserInput({required this.name, required this.email, this.age});
}

class OrderItemInput {
  final int productId;
  final int quantity;
  final double price;
  
  OrderItemInput({
    required this.productId,
    required this.quantity,
    required this.price,
  });
}
```

```dart
// lib/ui/pages/checkout_complete.dart
class CheckoutCompletePage extends StatefulWidget {
  final AppDatabase db;
  final int userId;
  final List<CartItem> items;
  
  const CheckoutCompletePage({
    required this.db,
    required this.userId,
    required this.items,
  });
  
  @override
  _CheckoutCompletePageState createState() => _CheckoutCompletePageState();
}

class _CheckoutCompletePageState extends State<CheckoutCompletePage> {
  Order? _order;
  bool _loading = true;
  String? _error;
  
  @override
  void initState() {
    super.initState();
    _createOrder();
  }
  
  Future<void> _createOrder() async {
    setState(() => _loading = true);
    
    try {
      // Convert cart items to order items
      final orderItems = await Future.wait(
        widget.items.map((item) async {
          final product = await (widget.db.select(widget.db.products)
            ..where((p) => p.id.equals(item.productId)))
            .getSingle();
          
          return OrderItemInput(
            productId: item.productId,
            quantity: item.quantity,
            price: product.price,
          );
        }).toList(),
      );
      
      // 👇 Create order with returning
      final order = await widget.db.createOrderReturning(
        userId: widget.userId,
        items: orderItems,
      );
      
      setState(() {
        _order = order;
        _loading = false;
      });
      
    } catch (e) {
      setState(() {
        _error = e.toString();
        _loading = false;
      });
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Order Complete')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: _loading
            ? Center(child: CircularProgressIndicator())
            : _error != null
                ? _buildError()
                : _buildSuccess(),
      ),
    );
  }
  
  Widget _buildError() {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.error, size: 64, color: Colors.red),
          SizedBox(height: 16),
          Text('Error creating order: $_error'),
          SizedBox(height: 16),
          ElevatedButton(
            onPressed: _createOrder,
            child: Text('Retry'),
          ),
        ],
      ),
    );
  }
  
  Widget _buildSuccess() {
    if (_order == null) return SizedBox.shrink();
    
    final order = _order!;
    
    return Column(
      children: [
        Icon(Icons.check_circle, size: 80, color: Colors.green),
        SizedBox(height: 16),
        Text(
          'Order Complete!',
          style: TextStyle(
            fontSize: 24,
            fontWeight: FontWeight.bold,
          ),
        ),
        SizedBox(height: 8),
        Text('Order #${order.orderNumber}'),
        Text('Total: \$${order.total.toStringAsFixed(2)}'),
        SizedBox(height: 8),
        Text('Status: ${order.status}'),
        Text('Created: ${_formatDate(order.orderDate)}'),
        SizedBox(height: 24),
        ElevatedButton(
          onPressed: () {
            Navigator.popUntil(context, (route) => route.isFirst);
          },
          child: Text('Continue Shopping'),
        ),
      ],
    );
  }
  
  String _formatDate(DateTime date) {
    return '${date.month}/${date.day}/${date.year} ${date.hour}:${date.minute.toString().padLeft(2, '0')}';
  }
}
```

---

# Best Practices

- **Use returning for generated values** – IDs, timestamps
- **Use returning for audit trails** – Log what was changed
- **Use returning to avoid extra queries** – Get data in one operation
- **Use specific fields when possible** – Reduce data transfer
- **Handle empty results** – When no rows affected
- **Use returning for batch operations** – Get all affected rows
- **Use returning for validation** – Verify what was stored

---

# Common Mistakes

## Mistake 1: Not handling empty results

Wrong:
```dart
// 🚫 Assumes record exists
final user = await (delete(users)..where((u) => u.id.equals(999)))
  .returning();
print(user.first.name); // Throws if no user
```

Correct:
```dart
// ✅ Handle empty results
final users = await (delete(users)..where((u) => u.id.equals(999)))
  .returning();
if (users.isNotEmpty) {
  print(users.first.name);
} else {
  print('No user found');
}
```

## Mistake 2: Returning too much data

Wrong:
```dart
// 🚫 Returns all columns unnecessarily
final user = await insertReturning(companion);
// user has 20 columns but you only need 2
```

Correct:
```dart
// ✅ Only return needed fields
final result = await insertReturning(
  companion,
  returning: (u) => [u.id, u.name],
);
```

## Mistake 3: Not using returning for validation

Wrong:
```dart
// 🚫 Extra query to verify
await into(users).insert(companion);
final user = await (select(users)..where(...)).getSingle();
```

Correct:
```dart
// ✅ Get data in one operation
final user = await into(users).insertReturning(companion);
```

---

# Summary

| Operation | Method | Returns |
|-----------|--------|---------|
| **Insert** | `insertReturning` | Full/partial record |
| **Insert (bulk)** | `insertAllReturning` | List of records |
| **Update** | `writeReturning` | Full/partial record |
| **Update (bulk)** | `batchWriteReturning` | List of records |
| **Delete** | `returning` | List of records |

---

# Next Steps

Now you understand returning rows, let's dive deeper:

- [Batch Operations](link) – Advanced batch patterns
- [Transactions](link) – Transaction management
- [Streams](link) – Reactive queries

---

# Did You Know?

- **Returning requires SQLite 3.35+** – For full support

- **Returning can return any columns** – Including generated ones

- **Returning works with all operations** – Insert, update, delete

- **Returning is atomic** – Same transaction as the operation

- **Returning can be used in transactions** – With other operations

- **Returning is faster than select** – One round-trip vs two

- **Returning can return multiple rows** – For batch operations

- **Returning returns typed objects** – With full type safety

---

