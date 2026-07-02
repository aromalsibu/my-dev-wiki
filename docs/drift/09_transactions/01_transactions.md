## Transactions

**Ensuring data integrity with atomic operations in Drift**

---

# What is it?

**Transactions** are a fundamental database concept that groups multiple operations into a single, atomic unit of work. Either all operations in the transaction succeed and are committed to the database, or none of them take effect (rollback). Drift provides both automatic and manual transaction control.

> **Think of Transactions like a "bank transfer"** – when you transfer money between accounts, both the withdrawal and deposit must succeed together. If one fails, neither happens.

```dart
// 👇 Transaction ensures all operations succeed or fail together
await transaction(() async {
  // 1️⃣ Deduct from sender's account
  await (update(accounts)..where((a) => a.id.equals(senderId)))
    .write(AccountsCompanion(balance: Value(senderBalance - amount)));
  
  // 2️⃣ Add to receiver's account
  await (update(accounts)..where((a) => a.id.equals(receiverId)))
    .write(AccountsCompanion(balance: Value(receiverBalance + amount)));
  
  // 3️⃣ Record the transaction
  await into(transactions).insert(
    TransactionsCompanion.insert(
      senderId: senderId,
      receiverId: receiverId,
      amount: amount,
    ),
  );
  
  // All succeed -> COMMIT
  // Any fails -> ROLLBACK
});
```

> **What's happening here?**
> - **Atomic** – All or nothing
> - **Consistent** – Database remains in a valid state
> - **Isolated** – Operations are isolated from others
> - **Durable** – Committed changes persist

---

# Why does it exist?

- **Data Integrity** – Maintain database consistency
- **Atomic Operations** – All or nothing execution
- **Error Recovery** – Rollback on failure
- **Complex Updates** – Multiple related changes
- **Business Logic** – Enforce business rules
- **Performance** – Batch operations in one transaction

---

# Basic Transactions

> **Simple transaction patterns**

## Automatic Transaction

```dart
// 👇 Using transaction() helper
await transaction(() async {
  // All operations in this block are atomic
  await into(users).insert(UsersCompanion.insert(name: 'John'));
  await into(posts).insert(PostsCompanion.insert(title: 'Hello', userId: 1));
  await (update(users)..where((u) => u.id.equals(1)))
    .write(UsersCompanion(postsCount: Value(1)));
});

// If any operation fails, all are rolled back
// If all succeed, all are committed
```

## Manual Transaction Control

```dart
// 👇 Manual transaction control
final userDao = UserDao(db);
await db.beginTransaction();

try {
  // Perform operations
  await userDao.createUser(name: 'John');
  await userDao.createPost(title: 'Hello', userId: 1);
  
  // Commit if successful
  await db.commitTransaction();
} catch (e) {
  // Rollback on error
  await db.rollbackTransaction();
  rethrow;
}
```

---

## Transaction with Return Value

```dart
// 👇 Transaction returning a value
Future<int> createUserWithPosts() async {
  return await transaction(() async {
    // Insert user
    final userId = await into(users).insert(
      UsersCompanion.insert(name: 'John Doe'),
    );
    
    // Insert posts
    await into(posts).insert(
      PostsCompanion.insert(title: 'Post 1', userId: userId),
    );
    await into(posts).insert(
      PostsCompanion.insert(title: 'Post 2', userId: userId),
    );
    
    // Return user ID
    return userId;
  });
}
```

---

# Transaction Isolation Levels

> **Controlling transaction isolation**

## Default Isolation (Serializable)

```dart
// 👇 Default isolation (most restrictive)
await transaction(() async {
  // Operations are fully isolated
  // Other transactions cannot see intermediate changes
  await updateUser(1);
  await updateUser(2);
});
```

## Defer Constraints

```dart
// 👇 Defer constraint checking until commit
await transaction(() async {
  // Foreign key constraints are checked at commit
  // Allows circular references to be resolved
  await db.deferConstraints(() async {
    await into(posts).insert(PostsCompanion.insert(
      title: 'Post',
      userId: 1, // User may not exist yet
    ));
    
    await into(users).insert(UsersCompanion.insert(
      id: Value(1),
      name: 'John',
    ));
  });
});
```

---

# Nested Transactions

> **Transactions within transactions**

## Using Nested Transactions

```dart
// 👇 Nested transactions (savepoints)
await transaction(() async {
  // Outer transaction
  final userId = await createUser('John');
  
  await transaction(() async {
    // Inner transaction (savepoint)
    // Can rollback independently
    await createPost(userId, 'Hello');
    await createComment(postId, 'Great post!');
  });
  
  // Outer transaction continues
  await updateUserStatus(userId, 'active');
});
```

## Savepoints

```dart
// 👇 Manual savepoints
await db.transaction(() async {
  // Start transaction
  await db.beginTransaction();
  
  try {
    // Create savepoint
    await db.savepoint('before_posts');
    
    // Operations that might fail
    await createPost(1, 'Title');
    await createComment(1, 'Comment');
    
    // If something fails, rollback to savepoint
    // await db.rollbackToSavepoint('before_posts');
    
    await db.commitTransaction();
  } catch (e) {
    await db.rollbackTransaction();
    rethrow;
  }
});
```

---

# Advanced Transaction Patterns

> **Complex transaction scenarios**

## Pattern 1: Transfer Money with Balance Check

```dart
Future<void> transferMoney({
  required int fromUserId,
  required int toUserId,
  required double amount,
}) async {
  await transaction(() async {
    // 1️⃣ Get sender's balance
    final sender = await (select(users)
      ..where((u) => u.id.equals(fromUserId)))
      .getSingle();
    
    if (sender.balance < amount) {
      throw Exception('Insufficient balance');
    }
    
    // 2️⃣ Deduct from sender
    await (update(users)..where((u) => u.id.equals(fromUserId)))
      .write(UsersCompanion(
        balance: Value(sender.balance - amount),
        updatedAt: Value(DateTime.now()),
      ));
    
    // 3️⃣ Get receiver's balance
    final receiver = await (select(users)
      ..where((u) => u.id.equals(toUserId)))
      .getSingle();
    
    // 4️⃣ Add to receiver
    await (update(users)..where((u) => u.id.equals(toUserId)))
      .write(UsersCompanion(
        balance: Value(receiver.balance + amount),
        updatedAt: Value(DateTime.now()),
      ));
    
    // 5️⃣ Record transaction
    await into(transactions).insert(
      TransactionsCompanion.insert(
        fromUserId: fromUserId,
        toUserId: toUserId,
        amount: amount,
        timestamp: DateTime.now(),
      ),
    );
  });
}
```

---

## Pattern 2: Order Processing with Inventory

```dart
Future<Order> processOrder(int userId, List<OrderItem> items) async {
  return await transaction(() async {
    // 1️⃣ Validate all items
    for (final item in items) {
      final product = await (select(products)
        ..where((p) => p.id.equals(item.productId)))
        .getSingle();
      
      if (product.stock < item.quantity) {
        throw Exception('Insufficient stock for ${product.name}');
      }
    }
    
    // 2️⃣ Create order
    final orderId = await into(orders).insert(
      OrdersCompanion.insert(
        userId: userId,
        orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
        status: 'pending',
        total: 0,
      ),
    );
    
    // 3️⃣ Process each item
    double total = 0;
    for (final item in items) {
      final product = await (select(products)
        ..where((p) => p.id.equals(item.productId)))
        .getSingle();
      
      // Add order item
      await into(orderItems).insert(
        OrderItemsCompanion.insert(
          orderId: orderId,
          productId: item.productId,
          quantity: item.quantity,
          unitPrice: product.price,
          total: product.price * item.quantity,
        ),
      );
      
      // Update stock
      await (update(products)..where((p) => p.id.equals(product.id)))
        .write(ProductsCompanion(
          stock: Value(product.stock - item.quantity),
          updatedAt: Value(DateTime.now()),
        ));
      
      total += product.price * item.quantity;
    }
    
    // 4️⃣ Update order total
    await (update(orders)..where((o) => o.id.equals(orderId)))
      .write(OrdersCompanion(
        total: Value(total),
        status: Value('completed'),
      ));
    
    // 5️⃣ Return the complete order
    return await (select(orders)
      ..where((o) => o.id.equals(orderId)))
      .getSingle();
  });
}
```

---

## Pattern 3: Audit Trail with Transaction

```dart
Future<void> updateUserWithAudit(int userId, UsersCompanion companion) async {
  await transaction(() async {
    // 1️⃣ Get old state
    final oldUser = await (select(users)
      ..where((u) => u.id.equals(userId)))
      .getSingle();
    
    // 2️⃣ Update user
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(companion);
    
    // 3️⃣ Get new state
    final newUser = await (select(users)
      ..where((u) => u.id.equals(userId)))
      .getSingle();
    
    // 4️⃣ Log audit
    await into(auditLogs).insert(
      AuditLogsCompanion.insert(
        userId: userId,
        action: 'UPDATE_USER',
        oldValue: Value(oldUser.toJson().toString()),
        newValue: Value(newUser.toJson().toString()),
        timestamp: DateTime.now(),
      ),
    );
  });
}
```

---

## Pattern 4: Conditional Transaction

```dart
Future<void> conditionalUpdate({
  required int userId,
  required String newStatus,
  bool requireAdminApproval = false,
}) async {
  await transaction(() async {
    // Get current user
    final user = await (select(users)
      ..where((u) => u.id.equals(userId)))
      .getSingle();
    
    // Check conditions
    if (requireAdminApproval && !user.isAdmin) {
      throw Exception('Admin approval required');
    }
    
    // Perform update
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        status: Value(newStatus),
        updatedAt: Value(DateTime.now()),
      ));
    
    // Log if status changed
    if (user.status != newStatus) {
      await into(statusLogs).insert(
        StatusLogsCompanion.insert(
          userId: userId,
          oldStatus: user.status,
          newStatus: newStatus,
          changedAt: DateTime.now(),
        ),
      );
    }
  });
}
```

---

# Transaction Error Handling

> **Handling errors in transactions**

## Basic Error Handling

```dart
Future<void> performTransaction() async {
  try {
    await transaction(() async {
      // Operations that might fail
      await operation1();
      await operation2();
      await operation3();
    });
  } catch (e) {
    print('Transaction failed: $e');
    // Handle error (retry, notify user, etc.)
  }
}
```

## Selective Error Recovery

```dart
Future<void> robustTransaction() async {
  var retries = 3;
  
  while (retries > 0) {
    try {
      await transaction(() async {
        await operation1();
        await operation2();
        await operation3();
      });
      break; // Success
    } catch (e) {
      retries--;
      if (retries == 0) rethrow;
      
      print('Retrying transaction... ($retries left)');
      await Future.delayed(Duration(seconds: 1));
    }
  }
}
```

---

# Real-World Example

> **Complete e-commerce transaction system**

```dart
// lib/database/transaction_service.dart
import 'package:drift/drift.dart';

class TransactionService {
  final AppDatabase db;
  
  TransactionService(this.db);
  
  // ==================== ORDER TRANSACTIONS ====================
  
  // 👇 Place order with inventory check
  Future<OrderResult> placeOrder({
    required int userId,
    required List<CartItem> items,
    required String shippingAddress,
    required String paymentMethod,
  }) async {
    return await db.transaction(() async {
      final orderItems = <OrderItemInfo>[];
      double total = 0;
      
      // 1️⃣ Validate and reserve inventory
      for (final item in items) {
        final product = await db.getProductWithLock(item.productId);
        
        if (product.stock < item.quantity) {
          throw InventoryException(
            'Insufficient stock for ${product.name}. '
            'Available: ${product.stock}'
          );
        }
        
        // Reserve stock
        await db.updateProductStock(
          item.productId, 
          product.stock - item.quantity,
        );
        
        final itemTotal = product.price * item.quantity;
        total += itemTotal;
        
        orderItems.add(OrderItemInfo(
          productId: item.productId,
          name: product.name,
          quantity: item.quantity,
          price: product.price,
          total: itemTotal,
        ));
      }
      
      // 2️⃣ Create order
      final orderId = await db.into(db.orders).insert(
        OrdersCompanion.insert(
          orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
          userId: userId,
          total: total,
          status: 'pending',
          shippingAddress: shippingAddress,
          billingAddress: shippingAddress,
          paymentMethod: paymentMethod,
          paymentStatus: 'unpaid',
          isPaid: false,
          isShipped: false,
          isDelivered: false,
        ),
      );
      
      // 3️⃣ Add order items
      for (final item in orderItems) {
        await db.into(db.orderItems).insert(
          OrderItemsCompanion.insert(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: item.price,
            subtotal: item.total,
            total: item.total,
          ),
        );
      }
      
      // 4️⃣ Record transaction history
      await db.into(db.orderHistory).insert(
        OrderHistoryCompanion.insert(
          orderId: orderId,
          action: 'ORDER_CREATED',
          details: 'Order placed with ${orderItems.length} items',
          createdAt: DateTime.now(),
        ),
      );
      
      return OrderResult(
        orderId: orderId,
        orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
        total: total,
        items: orderItems,
        status: 'pending',
      );
    });
  }
  
  // 👇 Process payment with transaction
  Future<PaymentResult> processPayment({
    required int orderId,
    required String paymentMethod,
    required double amount,
  }) async {
    return await db.transaction(() async {
      // 1️⃣ Get order
      final order = await (db.select(db.orders)
        ..where((o) => o.id.equals(orderId)))
        .getSingle();
      
      if (order.status != 'pending' && order.status != 'processing') {
        throw PaymentException('Order cannot be paid in status: ${order.status}');
      }
      
      // 2️⃣ Process payment (mock)
      final transactionId = await _processPaymentGateway(
        amount: amount,
        method: paymentMethod,
        orderId: orderId,
      );
      
      // 3️⃣ Update order
      await (db.update(db.orders)..where((o) => o.id.equals(orderId)))
        .write(OrdersCompanion(
          status: Value('paid'),
          paymentMethod: Value(paymentMethod),
          paymentStatus: Value('paid'),
          isPaid: const Value(true),
          updatedAt: Value(DateTime.now()),
        ));
      
      // 4️⃣ Record payment
      await db.into(db.payments).insert(
        PaymentsCompanion.insert(
          orderId: orderId,
          amount: amount,
          method: paymentMethod,
          transactionId: transactionId,
          status: 'completed',
          createdAt: DateTime.now(),
        ),
      );
      
      // 5️⃣ Update order history
      await db.into(db.orderHistory).insert(
        OrderHistoryCompanion.insert(
          orderId: orderId,
          action: 'PAYMENT_COMPLETED',
          details: 'Payment of \$${amount.toStringAsFixed(2)} processed',
          createdAt: DateTime.now(),
        ),
      );
      
      return PaymentResult(
        orderId: orderId,
        transactionId: transactionId,
        amount: amount,
        status: 'completed',
      );
    });
  }
  
  // 👇 Cancel order with stock restoration
  Future<CancelResult> cancelOrder({
    required int orderId,
    required String reason,
  }) async {
    return await db.transaction(() async {
      // 1️⃣ Get order
      final order = await (db.select(db.orders)
        ..where((o) => o.id.equals(orderId)))
        .getSingle();
      
      if (order.status == 'delivered') {
        throw CancelException('Order already delivered');
      }
      
      // 2️⃣ Get order items
      final items = await (db.select(db.orderItems)
        ..where((i) => i.orderId.equals(orderId)))
        .get();
      
      // 3️⃣ Restore stock
      for (final item in items) {
        final product = await db.getProduct(item.productId);
        await db.updateProductStock(
          item.productId, 
          product.stock + item.quantity,
        );
      }
      
      // 4️⃣ Update order
      await (db.update(db.orders)..where((o) => o.id.equals(orderId)))
        .write(OrdersCompanion(
          status: Value('cancelled'),
          isPaid: const Value(false),
          paymentStatus: Value('refunded'),
          updatedAt: Value(DateTime.now()),
        ));
      
      // 5️⃣ Record cancellation
      await db.into(db.orderHistory).insert(
        OrderHistoryCompanion.insert(
          orderId: orderId,
          action: 'ORDER_CANCELLED',
          details: 'Cancelled. Reason: $reason',
          createdAt: DateTime.now(),
        ),
      );
      
      return CancelResult(
        orderId: orderId,
        status: 'cancelled',
        reason: reason,
        itemsRestored: items.length,
      );
    });
  }
  
  // 👇 Ship order with tracking
  Future<ShippingResult> shipOrder({
    required int orderId,
    required String trackingNumber,
    required String carrier,
  }) async {
    return await db.transaction(() async {
      // 1️⃣ Get order
      final order = await (db.select(db.orders)
        ..where((o) => o.id.equals(orderId)))
        .getSingle();
      
      if (order.status != 'paid' && order.status != 'processing') {
        throw ShippingException('Order cannot be shipped in status: ${order.status}');
      }
      
      // 2️⃣ Update order
      await (db.update(db.orders)..where((o) => o.id.equals(orderId)))
        .write(OrdersCompanion(
          status: Value('shipped'),
          trackingNumber: Value(trackingNumber),
          isShipped: const Value(true),
          shippedDate: Value(DateTime.now()),
          updatedAt: Value(DateTime.now()),
        ));
      
      // 3️⃣ Update order items
      await (db.update(db.orderItems)..where((i) => i.orderId.equals(orderId)))
        .write(OrderItemsCompanion(
          isShipped: const Value(true),
        ));
      
      // 4️⃣ Record shipping
      await db.into(db.shippingLogs).insert(
        ShippingLogsCompanion.insert(
          orderId: orderId,
          trackingNumber: trackingNumber,
          carrier: carrier,
          status: 'shipped',
          createdAt: DateTime.now(),
        ),
      );
      
      // 5️⃣ Update order history
      await db.into(db.orderHistory).insert(
        OrderHistoryCompanion.insert(
          orderId: orderId,
          action: 'ORDER_SHIPPED',
          details: 'Shipped via $carrier. Tracking: $trackingNumber',
          createdAt: DateTime.now(),
        ),
      );
      
      return ShippingResult(
        orderId: orderId,
        trackingNumber: trackingNumber,
        carrier: carrier,
        status: 'shipped',
        estimatedDelivery: DateTime.now().add(Duration(days: 3)),
      );
    });
  }
  
  // ==================== USER TRANSACTIONS ====================
  
  // 👇 Create user with profile and settings
  Future<UserResult> createUserWithProfile({
    required String username,
    required String email,
    required String password,
    String? fullName,
    Map<String, dynamic>? preferences,
  }) async {
    return await db.transaction(() async {
      // 1️⃣ Create user
      final userId = await db.into(db.users).insert(
        UsersCompanion.insert(
          username: username,
          email: email,
          passwordHash: _hashPassword(password),
          fullName: Value(fullName),
          isActive: Value(true),
          isVerified: Value(false),
        ),
      );
      
      // 2️⃣ Create profile
      await db.into(db.profiles).insert(
        ProfilesCompanion.insert(
          userId: userId,
          displayName: Value(fullName ?? username),
          bio: Value(null),
          avatar: Value(null),
        ),
      );
      
      // 3️⃣ Create settings
      await db.into(db.userSettings).insert(
        UserSettingsCompanion.insert(
          userId: userId,
          theme: 'light',
          language: 'en',
          notifications: true,
          preferences: Value(preferences != null ? jsonEncode(preferences) : null),
        ),
      );
      
      // 4️⃣ Get created user
      final user = await (db.select(db.users)
        ..where((u) => u.id.equals(userId)))
        .getSingle();
      
      return UserResult(
        id: userId,
        username: user.username,
        email: user.email,
        createdAt: user.createdAt,
      );
    });
  }
  
  // ==================== HELPER METHODS ====================
  
  Future<String> _processPaymentGateway({
    required double amount,
    required String method,
    required int orderId,
  }) async {
    // Mock payment processing
    await Future.delayed(Duration(seconds: 1));
    return 'TRANS-${DateTime.now().millisecondsSinceEpoch}';
  }
  
  String _hashPassword(String password) {
    // Mock password hashing
    return 'hashed_$password';
  }
}

// ==================== DATA CLASSES ====================

class OrderResult {
  final int orderId;
  final String orderNumber;
  final double total;
  final List<OrderItemInfo> items;
  final String status;
  
  OrderResult({
    required this.orderId,
    required this.orderNumber,
    required this.total,
    required this.items,
    required this.status,
  });
}

class OrderItemInfo {
  final int productId;
  final String name;
  final int quantity;
  final double price;
  final double total;
  
  OrderItemInfo({
    required this.productId,
    required this.name,
    required this.quantity,
    required this.price,
    required this.total,
  });
}

class PaymentResult {
  final int orderId;
  final String transactionId;
  final double amount;
  final String status;
  
  PaymentResult({
    required this.orderId,
    required this.transactionId,
    required this.amount,
    required this.status,
  });
}

class CancelResult {
  final int orderId;
  final String status;
  final String reason;
  final int itemsRestored;
  
  CancelResult({
    required this.orderId,
    required this.status,
    required this.reason,
    required this.itemsRestored,
  });
}

class ShippingResult {
  final int orderId;
  final String trackingNumber;
  final String carrier;
  final String status;
  final DateTime estimatedDelivery;
  
  ShippingResult({
    required this.orderId,
    required this.trackingNumber,
    required this.carrier,
    required this.status,
    required this.estimatedDelivery,
  });
}

class UserResult {
  final int id;
  final String username;
  final String email;
  final DateTime createdAt;
  
  UserResult({
    required this.id,
    required this.username,
    required this.email,
    required this.createdAt,
  });
}

// ==================== CUSTOM EXCEPTIONS ====================

class InventoryException implements Exception {
  final String message;
  InventoryException(this.message);
  @override
  String toString() => 'InventoryException: $message';
}

class PaymentException implements Exception {
  final String message;
  PaymentException(this.message);
  @override
  String toString() => 'PaymentException: $message';
}

class CancelException implements Exception {
  final String message;
  CancelException(this.message);
  @override
  String toString() => 'CancelException: $message';
}

class ShippingException implements Exception {
  final String message;
  ShippingException(this.message);
  @override
  String toString() => 'ShippingException: $message';
}
```

---

# Best Practices

- **Keep transactions short** – Don't keep them open
- **Use transactions for atomic operations** – Related changes
- **Handle errors properly** – Rollback on failure
- **Use savepoints for nested transactions** – Partial rollback
- **Test transaction behavior** – Verify atomicity
- **Avoid user interaction in transactions** – Don't wait for input
- **Use appropriate isolation levels** – Based on requirements
- **Monitor transaction duration** – Avoid long-running transactions

---

# Common Mistakes

## Mistake 1: Long-running transactions

Wrong:
```dart
// 🚫 Transaction with user input
await transaction(() async {
  await updateUser(1);
  
  // User prompt inside transaction (BAD!)
  final response = await showDialog(...);
  if (response) {
    await deleteUser(1);
  }
});
```

Correct:
```dart
// ✅ Get user input before transaction
final response = await showDialog(...);

await transaction(() async {
  await updateUser(1);
  if (response) {
    await deleteUser(1);
  }
});
```

## Mistake 2: Not handling errors

Wrong:
```dart
// 🚫 Transaction fails silently
await transaction(() async {
  await operation1();
  await operation2(); // If this fails, all rolls back
});
```

Correct:
```dart
// ✅ Handle errors
try {
  await transaction(() async {
    await operation1();
    await operation2();
  });
} catch (e) {
  print('Transaction failed: $e');
  // Handle error
}
```

## Mistake 3: Nesting transactions incorrectly

Wrong:
```dart
// 🚫 Nested transactions with conflicting operations
await transaction(() async {
  await operation1();
  await transaction(() async {
    await operation1(); // Conflict!
  });
});
```

Correct:
```dart
// ✅ Use savepoints for nested operations
await transaction(() async {
  await operation1();
  await db.savepoint('savepoint1');
  try {
    await operation2();
  } catch (e) {
    await db.rollbackToSavepoint('savepoint1');
  }
});
```

---

# Summary

| Feature | Purpose | Best For |
|---------|---------|----------|
| **Transaction** | Atomic operations | Multiple related changes |
| **Savepoint** | Nested transactions | Partial rollback |
| **Isolation** | Concurrent access | Data consistency |
| **Defer constraints** | Circular references | Complex relationships |

---

# Next Steps

Now you understand transactions, let's dive deeper:

- [Streams](link) – Reactive queries
- [Performance](link) – Performance optimization
- [Testing](link) – Testing strategies

---

# Did You Know?

- **Transactions are ACID** – Atomic, Consistent, Isolated, Durable

- **Transactions can be nested** – Using savepoints

- **Transactions can be deferred** – For complex constraints

- **Transactions are isolated** – From other transactions

- **Transactions can be rolled back** – On any failure

- **Transactions are safe** – For concurrent access

- **Transactions are critical** – For data integrity

- **Transactions are used by Drift** – In batch operations

---

