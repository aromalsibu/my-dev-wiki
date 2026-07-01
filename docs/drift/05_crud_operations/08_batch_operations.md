## Batch Operations

**Advanced batch processing for multiple database operations**

---

# What is it?

**Batch Operations** allow you to group multiple database operations (inserts, updates, deletes) into a single transaction. This ensures atomicity, improves performance, and simplifies complex data manipulations. Drift provides a powerful batch API that supports mixed operations and returning results.

> **Think of Batch Operations like "mass editing"** – instead of updating one document at a time, you can select multiple documents and apply changes to all of them simultaneously.

```dart
// 👇 Batch multiple operations
await into(users).batch((batch) {
  // Insert new users
  batch.insert(UsersCompanion.insert(name: 'User 1', email: 'user1@example.com'));
  batch.insert(UsersCompanion.insert(name: 'User 2', email: 'user2@example.com'));
  
  // Update existing users
  batch.update(
    users,
    UsersCompanion(isActive: Value(false)),
    (u) => u.isActive.equals(true),
  );
  
  // Delete old users
  batch.delete(users, (u) => u.createdAt < const Variable(DateTime(2023, 1, 1)));
});

// All operations execute in a single transaction:
// BEGIN TRANSACTION;
// INSERT INTO users ...
// INSERT INTO users ...
// UPDATE users SET is_active = 0 WHERE is_active = 1;
// DELETE FROM users WHERE created_at < '2023-01-01';
// COMMIT;
```

> **What's happening here?**
> - **Single Transaction** – All operations in one transaction
> - **Mixed Operations** – Inserts, updates, deletes together
> - **Atomic** – All succeed or all fail
> - **Performance** – One round-trip to the database

---

# Why does it exist?

- **Atomic Operations** – All or nothing execution
- **Performance** – One database round-trip
- **Data Integrity** – Operations complete together
- **Complex Updates** – Multiple related changes
- **Batch Processing** – Process many records efficiently
- **Error Handling** – Rollback on any failure

---

# Basic Batch Operations

> **Simple batch patterns**

## Batch Insert Only

```dart
// 👇 Batch insert multiple records
await into(users).batch((batch) {
  batch.insert(
    UsersCompanion.insert(
      name: 'User 1',
      email: 'user1@example.com',
    ),
  );
  batch.insert(
    UsersCompanion.insert(
      name: 'User 2',
      email: 'user2@example.com',
    ),
  );
  batch.insert(
    UsersCompanion.insert(
      name: 'User 3',
      email: 'user3@example.com',
    ),
  );
});
```

## Batch Update Only

```dart
// 👇 Batch update multiple records
await into(users).batch((batch) {
  // Deactivate all inactive users
  batch.update(
    users,
    UsersCompanion(
      isActive: Value(false),
      updatedAt: Value(DateTime.now()),
    ),
    (u) => u.isActive.equals(false),
  );
  
  // Update status for old users
  batch.update(
    users,
    UsersCompanion(
      status: Value('senior'),
      updatedAt: Value(DateTime.now()),
    ),
    (u) => u.age > const Variable(60),
  );
});
```

---

## Batch Delete Only

```dart
// 👇 Batch delete multiple records
await into(users).batch((batch) {
  // Delete inactive users older than 1 year
  batch.delete(
    users,
    (u) => 
      u.isActive.equals(false) &
      u.createdAt < const Variable(
        DateTime.now().subtract(Duration(days: 365))
      ),
  );
  
  // Delete unverified users
  batch.delete(
    users,
    (u) => 
      u.isVerified.equals(false) &
      u.createdAt < const Variable(
        DateTime.now().subtract(Duration(days: 30))
      ),
  );
});
```

---

# Advanced Batch Patterns

> **Complex batch operations**

## Pattern 1: Mixed Operations in Batch

```dart
// 👇 Insert, update, and delete in one batch
Future<void> syncUsersWithBatch(List<UserData> users) async {
  final existingIds = users.map((u) => u.id).toList();
  
  await into(users).batch((batch) {
    // 1️⃣ Insert new users
    for (final user in users.where((u) => u.isNew)) {
      batch.insert(
        UsersCompanion.insert(
          name: user.name,
          email: user.email,
          age: Value(user.age),
        ),
      );
    }
    
    // 2️⃣ Update existing users
    for (final user in users.where((u) => !u.isNew && u.isUpdated)) {
      batch.update(
        users,
        UsersCompanion(
          name: Value(user.name),
          age: Value(user.age),
          updatedAt: Value(DateTime.now()),
        ),
        (u) => u.id.equals(user.id),
      );
    }
    
    // 3️⃣ Delete users not in the list
    batch.delete(
      users,
      (u) => u.id.isIn(existingIds).not(),
    );
  });
}
```

---

## Pattern 2: Batch with Returning

```dart
// 👇 Batch with returning results
final results = await into(users).batchReturning(
  (batch) {
    batch.insert(
      UsersCompanion.insert(name: 'User 1', email: 'user1@example.com'),
    );
    batch.insert(
      UsersCompanion.insert(name: 'User 2', email: 'user2@example.com'),
    );
    batch.update(
      users,
      UsersCompanion(status: Value('active')),
      (u) => u.isActive.equals(true),
    );
  },
  returning: (u) => [u.id, u.name],
);

for (final result in results) {
  print('Affected: ID ${result[0]}, Name ${result[1]}');
}
```

---

## Pattern 3: Conditional Batch Operations

```dart
// 👇 Batch with conditions
Future<void> conditionalBatchUpdate({
  required String status,
  int? minAge,
  int? maxAge,
  bool? isActive,
}) async {
  await into(users).batch((batch) {
    final query = users.select()..where((u) {
      var conditions = Expression<bool>.fromConstant(true);
      
      if (minAge != null) {
        conditions = conditions & (u.age > const Variable(minAge));
      }
      
      if (maxAge != null) {
        conditions = conditions & (u.age < const Variable(maxAge));
      }
      
      if (isActive != null) {
        conditions = conditions & (u.isActive.equals(isActive));
      }
      
      return conditions;
    });
    
    batch.update(
      users,
      UsersCompanion(
        status: Value(status),
        updatedAt: Value(DateTime.now()),
      ),
      query.where,
    );
  });
}
```

---

## Pattern 4: Bulk Import with Validation

```dart
// 👇 Batch import with validation
class BatchImporter {
  final AppDatabase db;
  
  BatchImporter(this.db);
  
  Future<ImportResult> importUsers(List<UserData> users) async {
    var inserted = 0;
    var updated = 0;
    final errors = <String>[];
    
    try {
      await db.transaction(() async {
        for (final user in users) {
          try {
            // Validate user data
            _validateUser(user);
            
            // Check if user exists
            final existing = await (db.select(db.users)
              ..where((u) => u.email.equals(user.email)))
              .getSingleOrNull();
            
            if (existing == null) {
              // Insert new user
              await db.into(db.users).insert(
                UsersCompanion.insert(
                  name: user.name,
                  email: user.email,
                  age: Value(user.age),
                ),
              );
              inserted++;
            } else {
              // Update existing user
              await (db.update(db.users)..where((u) => u.id.equals(existing.id)))
                .write(UsersCompanion(
                  name: Value(user.name),
                  age: Value(user.age),
                  updatedAt: Value(DateTime.now()),
                ));
              updated++;
            }
          } catch (e) {
            errors.add('User ${user.email}: $e');
          }
        }
      });
      
      return ImportResult(
        inserted: inserted,
        updated: updated,
        errors: errors,
      );
    } catch (e) {
      return ImportResult(
        inserted: inserted,
        updated: updated,
        errors: [...errors, 'Transaction failed: $e'],
      );
    }
  }
  
  void _validateUser(UserData user) {
    if (user.name.isEmpty) {
      throw Exception('Name is required');
    }
    if (!user.email.contains('@')) {
      throw Exception('Invalid email format');
    }
    if (user.age != null && (user.age! < 0 || user.age! > 150)) {
      throw Exception('Invalid age');
    }
  }
}

class ImportResult {
  final int inserted;
  final int updated;
  final List<String> errors;
  
  ImportResult({
    required this.inserted,
    required this.updated,
    this.errors = const [],
  });
  
  bool get hasErrors => errors.isNotEmpty;
  int get total => inserted + updated;
  String get summary => 'Inserted: $inserted, Updated: $updated, Errors: ${errors.length}';
}
```

---

## Pattern 5: Chunked Batch Processing

```dart
// 👇 Process large datasets in chunks
class ChunkedBatchProcessor {
  final AppDatabase db;
  final int chunkSize;
  
  ChunkedBatchProcessor({
    required this.db,
    this.chunkSize = 1000,
  });
  
  Future<ChunkedProcessResult> processUsers(
    List<UserData> users,
    Function(int, int) onProgress,
  ) async {
    final total = users.length;
    var processed = 0;
    var inserted = 0;
    var updated = 0;
    final errors = <String>[];
    
    for (var i = 0; i < users.length; i += chunkSize) {
      final chunk = users.sublist(
        i,
        i + chunkSize > users.length ? users.length : i + chunkSize,
      );
      
      try {
        await db.transaction(() async {
          for (final user in chunk) {
            try {
              // Process each user in chunk
              final existing = await (db.select(db.users)
                ..where((u) => u.email.equals(user.email)))
                .getSingleOrNull();
              
              if (existing == null) {
                await db.into(db.users).insert(
                  UsersCompanion.insert(
                    name: user.name,
                    email: user.email,
                    age: Value(user.age),
                  ),
                );
                inserted++;
              } else {
                await (db.update(db.users)..where((u) => u.id.equals(existing.id)))
                  .write(UsersCompanion(
                    name: Value(user.name),
                    age: Value(user.age),
                    updatedAt: Value(DateTime.now()),
                  ));
                updated++;
              }
            } catch (e) {
              errors.add('User ${user.email}: $e');
            }
          }
        });
      } catch (e) {
        errors.add('Chunk ${i ~/ chunkSize + 1}: Transaction failed - $e');
      }
      
      processed += chunk.length;
      onProgress(processed, total);
    }
    
    return ChunkedProcessResult(
      inserted: inserted,
      updated: updated,
      errors: errors,
      totalProcessed: processed,
      totalUsers: total,
    );
  }
}

class ChunkedProcessResult {
  final int inserted;
  final int updated;
  final List<String> errors;
  final int totalProcessed;
  final int totalUsers;
  
  ChunkedProcessResult({
    required this.inserted,
    required this.updated,
    this.errors = const [],
    required this.totalProcessed,
    required this.totalUsers,
  });
  
  bool get hasErrors => errors.isNotEmpty;
  bool get isComplete => totalProcessed == totalUsers;
  String get summary => '✅ Inserted: $inserted, Updated: $updated, Errors: ${errors.length}';
}
```

---

# Real-World Example

> **Complete e-commerce batch system**

```dart
// lib/database/batch_service.dart
import 'package:drift/drift.dart';

class BatchService {
  final AppDatabase db;
  
  BatchService(this.db);
  
  // ==================== PRODUCT BATCH OPERATIONS ====================
  
  // 👇 Bulk product import
  Future<BatchResult> bulkImportProducts(List<ProductData> products) async {
    var imported = 0;
    final errors = <String>[];
    
    await db.transaction(() async {
      for (final product in products) {
        try {
          // Validate
          if (product.sku.isEmpty) {
            throw Exception('SKU required for ${product.name}');
          }
          if (product.price < 0) {
            throw Exception('Invalid price for ${product.name}');
          }
          
          // Check if exists
          final existing = await (db.select(db.products)
            ..where((p) => p.sku.equals(product.sku)))
            .getSingleOrNull();
          
          if (existing == null) {
            await db.into(db.products).insert(
              ProductsCompanion.insert(
                sku: product.sku,
                name: product.name,
                price: product.price,
                description: Value(product.description),
                stock: Value(product.stock),
                isActive: Value(product.isActive),
              ),
            );
            imported++;
          } else {
            await (db.update(db.products)..where((p) => p.id.equals(existing.id)))
              .write(ProductsCompanion(
                name: Value(product.name),
                price: Value(product.price),
                description: Value(product.description),
                stock: Value(product.stock),
                isActive: Value(product.isActive),
                updatedAt: Value(DateTime.now()),
              ));
            imported++;
          }
        } catch (e) {
          errors.add('${product.sku}: $e');
        }
      }
    });
    
    return BatchResult(
      processed: imported,
      errors: errors,
      total: products.length,
    );
  }
  
  // 👇 Bulk price update
  Future<BatchResult> bulkUpdatePrices(
    List<PriceUpdateData> updates,
  ) async {
    var updated = 0;
    final errors = <String>[];
    
    await db.transaction(() async {
      for (final update in updates) {
        try {
          final result = await (db.update(db.products)
            ..where((p) => p.id.equals(update.productId)))
            .write(ProductsCompanion(
              price: Value(update.newPrice),
              updatedAt: Value(DateTime.now()),
            ));
          
          if (result > 0) {
            updated++;
          }
        } catch (e) {
          errors.add('Product ${update.productId}: $e');
        }
      }
    });
    
    return BatchResult(
      processed: updated,
      errors: errors,
      total: updates.length,
    );
  }
  
  // 👇 Bulk stock adjustment
  Future<BatchResult> bulkAdjustStock(List<StockAdjustmentData> adjustments) async {
    var adjusted = 0;
    final errors = <String>[];
    
    await db.transaction(() async {
      for (final adj in adjustments) {
        try {
          // Get current product
          final product = await (db.select(db.products)
            ..where((p) => p.id.equals(adj.productId)))
            .getSingle();
          
          final newStock = product.stock + adj.adjustment;
          if (newStock < 0) {
            throw Exception('Insufficient stock');
          }
          
          await (db.update(db.products)..where((p) => p.id.equals(product.id)))
            .write(ProductsCompanion(
              stock: Value(newStock),
              updatedAt: Value(DateTime.now()),
            ));
          
          adjusted++;
        } catch (e) {
          errors.add('Product ${adj.productId}: $e');
        }
      }
    });
    
    return BatchResult(
      processed: adjusted,
      errors: errors,
      total: adjustments.length,
    );
  }

  // ==================== ORDER BATCH OPERATIONS ====================
  
  // 👇 Bulk order status update
  Future<BatchResult> bulkUpdateOrderStatus(
    List<OrderStatusUpdateData> updates,
  ) async {
    var updated = 0;
    final errors = <String>[];
    
    await db.transaction(() async {
      for (final update in updates) {
        try {
          final result = await (db.update(db.orders)
            ..where((o) => o.id.equals(update.orderId)))
            .write(OrdersCompanion(
              status: Value(update.newStatus),
              shippedDate: update.newStatus == 'shipped' 
                  ? Value(DateTime.now()) 
                  : const Value.absent(),
              deliveredDate: update.newStatus == 'delivered'
                  ? Value(DateTime.now())
                  : const Value.absent(),
              updatedAt: Value(DateTime.now()),
            ));
          
          if (result > 0) {
            updated++;
          }
        } catch (e) {
          errors.add('Order ${update.orderId}: $e');
        }
      }
    });
    
    return BatchResult(
      processed: updated,
      errors: errors,
      total: updates.length,
    );
  }
  
  // 👇 Bulk order cancellation with stock restoration
  Future<BatchResult> bulkCancelOrders(List<int> orderIds) async {
    var cancelled = 0;
    final errors = <String>[];
    
    await db.transaction(() async {
      for (final orderId in orderIds) {
        try {
          // Get order items
          final items = await (db.select(db.orderItems)
            ..where((i) => i.orderId.equals(orderId)))
            .get();
          
          // Restore stock
          for (final item in items) {
            final product = await (db.select(db.products)
              ..where((p) => p.id.equals(item.productId)))
              .getSingle();
            
            await (db.update(db.products)..where((p) => p.id.equals(product.id)))
              .write(ProductsCompanion(
                stock: Value(product.stock + item.quantity),
                updatedAt: Value(DateTime.now()),
              ));
          }
          
          // Cancel order
          await (db.update(db.orders)..where((o) => o.id.equals(orderId)))
            .write(OrdersCompanion(
              status: Value('cancelled'),
              isPaid: const Value(false),
              paymentStatus: Value('refunded'),
              updatedAt: Value(DateTime.now()),
            ));
          
          cancelled++;
        } catch (e) {
          errors.add('Order $orderId: $e');
        }
      }
    });
    
    return BatchResult(
      processed: cancelled,
      errors: errors,
      total: orderIds.length,
    );
  }

  // ==================== USER BATCH OPERATIONS ====================
  
  // 👇 Bulk user status update
  Future<BatchResult> bulkUpdateUserStatus(
    List<UserStatusUpdateData> updates,
  ) async {
    var updated = 0;
    final errors = <String>[];
    
    await db.transaction(() async {
      for (final update in updates) {
        try {
          final result = await (db.update(db.users)
            ..where((u) => u.id.equals(update.userId)))
            .write(UsersCompanion(
              isActive: Value(update.isActive),
              updatedAt: Value(DateTime.now()),
            ));
          
          if (result > 0) {
            updated++;
          }
        } catch (e) {
          errors.add('User ${update.userId}: $e');
        }
      }
    });
    
    return BatchResult(
      processed: updated,
      errors: errors,
      total: updates.length,
    );
  }
  
  // 👇 Bulk user verification
  Future<BatchResult> bulkVerifyUsers(List<int> userIds) async {
    var verified = 0;
    final errors = <String>[];
    
    await db.transaction(() async {
      for (final userId in userIds) {
        try {
          final result = await (db.update(db.users)
            ..where((u) => u.id.equals(userId)))
            .write(UsersCompanion(
              isVerified: const Value(true),
              updatedAt: Value(DateTime.now()),
            ));
          
          if (result > 0) {
            verified++;
          }
        } catch (e) {
          errors.add('User $userId: $e');
        }
      }
    });
    
    return BatchResult(
      processed: verified,
      errors: errors,
      total: userIds.length,
    );
  }

  // ==================== DATA CLEANUP BATCHES ====================
  
  // 👇 Archive old data
  Future<BatchResult> archiveOldData({
    Duration? olderThan,
    bool includeUsers = false,
    bool includeOrders = false,
  }) async {
    var archived = 0;
    final errors = <String>[];
    final cutoffDate = DateTime.now().subtract(olderThan ?? Duration(days: 365));
    
    await db.transaction(() async {
      // Archive old orders
      if (includeOrders) {
        try {
          final orders = await (db.select(db.orders)
            ..where((o) => 
              o.status.equals('delivered') |
              o.status.equals('cancelled')
            )
            ..where((o) => o.orderDate < const Variable(cutoffDate)))
            .get();
          
          for (final order in orders) {
            // Archive order items first
            final items = await (db.select(db.orderItems)
              ..where((i) => i.orderId.equals(order.id)))
              .get();
            
            for (final item in items) {
              await db.into(db.archivedOrderItems).insert(
                ArchivedOrderItemsCompanion.insert(
                  orderId: item.orderId,
                  productId: item.productId,
                  quantity: item.quantity,
                  unitPrice: item.unitPrice,
                  subtotal: item.subtotal,
                  total: item.total,
                  archivedAt: DateTime.now(),
                ),
              );
            }
            
            await (db.delete(db.orderItems)
              ..where((i) => i.orderId.equals(order.id)))
              .go();
            
            // Archive order
            await db.into(db.archivedOrders).insert(
              ArchivedOrdersCompanion.insert(
                orderNumber: order.orderNumber,
                userId: order.userId,
                total: order.total,
                status: order.status,
                orderDate: order.orderDate,
                archivedAt: DateTime.now(),
              ),
            );
            
            await (db.delete(db.orders)
              ..where((o) => o.id.equals(order.id)))
              .go();
            
            archived++;
          }
        } catch (e) {
          errors.add('Order archiving: $e');
        }
      }
      
      // Archive old users
      if (includeUsers) {
        try {
          final users = await (db.select(db.users)
            ..where((u) => u.isDeleted.equals(true))
            ..where((u) => u.deletedAt < const Variable(cutoffDate)))
            .get();
          
          for (final user in users) {
            await db.into(db.archivedUsers).insert(
              ArchivedUsersCompanion.insert(
                username: user.username,
                email: user.email,
                fullName: Value(user.fullName),
                archivedAt: DateTime.now(),
              ),
            );
            
            await (db.delete(db.users)
              ..where((u) => u.id.equals(user.id)))
              .go();
            
            archived++;
          }
        } catch (e) {
          errors.add('User archiving: $e');
        }
      }
    });
    
    return BatchResult(
      processed: archived,
      errors: errors,
      total: 0, // Not tracked for cleanup operations
    );
  }

  // ==================== PERFORMANCE OPTIMIZATION ====================
  
  // 👇 Use batch for large operations
  Future<BatchResult> optimizeBatchOperation(
    List<Operation> operations,
  ) async {
    var processed = 0;
    final errors = <String>[];
    
    await db.transaction(() async {
      for (final op in operations) {
        try {
          switch (op.type) {
            case OperationType.insert:
              await db.into(db.users).insert(op.companion);
              break;
            case OperationType.update:
              await (db.update(db.users)..where(op.where))
                .write(op.companion);
              break;
            case OperationType.delete:
              await (db.delete(db.users)..where(op.where)).go();
              break;
          }
          processed++;
        } catch (e) {
          errors.add('Operation $processed: $e');
        }
      }
    });
    
    return BatchResult(
      processed: processed,
      errors: errors,
      total: operations.length,
    );
  }
}

// ==================== DATA CLASSES ====================

class BatchResult {
  final int processed;
  final List<String> errors;
  final int total;
  
  BatchResult({
    required this.processed,
    this.errors = const [],
    required this.total,
  });
  
  bool get hasErrors => errors.isNotEmpty;
  double get successRate => total > 0 ? processed / total : 0;
  
  String get summary {
    var summary = '✅ $processed processed';
    if (total > 0) {
      summary += ' of $total (${(successRate * 100).toStringAsFixed(0)}%)';
    }
    if (hasErrors) {
      summary += '\n❌ ${errors.length} errors: ${errors.join(', ')}';
    }
    return summary;
  }
}

class ProductData {
  final String sku;
  final String name;
  final double price;
  final String? description;
  final int stock;
  final bool isActive;
  
  ProductData({
    required this.sku,
    required this.name,
    required this.price,
    this.description,
    this.stock = 0,
    this.isActive = true,
  });
}

class PriceUpdateData {
  final int productId;
  final double newPrice;
  
  PriceUpdateData({required this.productId, required this.newPrice});
}

class StockAdjustmentData {
  final int productId;
  final int adjustment;
  
  StockAdjustmentData({required this.productId, required this.adjustment});
}

class OrderStatusUpdateData {
  final int orderId;
  final String newStatus;
  
  OrderStatusUpdateData({required this.orderId, required this.newStatus});
}

class UserStatusUpdateData {
  final int userId;
  final bool isActive;
  
  UserStatusUpdateData({required this.userId, required this.isActive});
}

enum OperationType { insert, update, delete }

class Operation {
  final OperationType type;
  final Insertable? companion;
  final Expression<bool> Function(dynamic)? where;
  
  Operation.insert(this.companion)
      : type = OperationType.insert,
        where = null;
  
  Operation.update(this.where, this.companion)
      : type = OperationType.update;
  
  Operation.delete(this.where)
      : type = OperationType.delete,
        companion = null;
}
```

---

# Best Practices

- **Use batch for 10+ operations** – Better performance
- **Keep batch sizes manageable** – 100-1000 operations
- **Use transactions** – All or nothing
- **Validate before batch** – Catch errors early
- **Handle errors gracefully** – Log and continue
- **Chunk large datasets** – Process in smaller batches
- **Use returning when needed** – Get results back
- **Test batch operations** – Verify atomicity

---

# Common Mistakes

## Mistake 1: Huge batch sizes

Wrong:
```dart
// 🚫 100,000 operations in one batch (memory issues)
await db.batch((batch) {
  for (final item in hugeList) {
    batch.insert(item);
  }
});
```

Correct:
```dart
// ✅ Chunk into manageable sizes
const chunkSize = 1000;
for (var i = 0; i < hugeList.length; i += chunkSize) {
  final chunk = hugeList.sublist(i, min(i + chunkSize, hugeList.length));
  await db.batch((batch) {
    for (final item in chunk) {
      batch.insert(item);
    }
  });
}
```

## Mistake 2: Not handling errors in batch

Wrong:
```dart
// 🚫 One error fails entire batch
await db.batch((batch) {
  for (final user in users) {
    batch.insert(user); // One invalid user fails everything
  }
});
```

Correct:
```dart
// ✅ Validate before batch
for (final user in users) {
  if (!isValid(user)) {
    throw Exception('Invalid user: ${user.email}');
  }
}
await db.batch((batch) {
  for (final user in users) {
    batch.insert(user);
  }
});
```

---

# Summary

| Feature | Purpose | Best For |
|---------|---------|----------|
| **Batch** | Multiple operations | Bulk updates |
| **Transaction** | Atomic operations | Related changes |
| **Returning** | Get results | Audit trails |
| **Chunking** | Large datasets | Performance |

---

# Next Steps

Now you understand batch operations, let's dive deeper:

- [Transactions](link) – Transaction management
- [Streams](link) – Reactive queries
- [Performance](link) – Performance optimization

---

# Did You Know?

- **Batch operations are 10-100x faster** – For bulk operations

- **Batch operations are atomic** – All or nothing

- **Batch uses one transaction** – Under the hood

- **Batch supports mixed operations** – Insert, update, delete

- **Batch can return results** – With batchReturning

- **Chunking prevents memory issues** – For large datasets

- **Batch operations are transaction-safe** – Rollback on failure

- **Batch is the most efficient way** – To process bulk data

---

