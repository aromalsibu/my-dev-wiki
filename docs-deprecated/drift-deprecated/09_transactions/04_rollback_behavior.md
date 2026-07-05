## Rollback Behavior

**Understanding how Drift handles transaction rollbacks and error recovery**

---

# What is it?

**Rollback Behavior** refers to how Drift handles transaction failures, error recovery, and the rollback process itself. Understanding rollback behavior is crucial for building robust applications that can recover gracefully from errors and maintain data consistency. Drift provides automatic rollback on exceptions, manual rollback control, and nested transaction support through savepoints.

> **Think of Rollback Behavior like "emergency protocols"** – when something goes wrong, you need to know exactly what happens, how to recover, and what state your data will be in.

```dart
// 👇 Automatic rollback on exception
await transaction(() async {
  await step1(); // ✅ Succeeds
  await step2(); // ❌ Throws exception
  await step3(); // ⏭️ Never executed
});
// All changes from step1 rolled back automatically
// Database remains in consistent state
```

> **What's happening here?**
> - **Automatic rollback** – Exception triggers rollback
> - **All or nothing** – No partial changes
> - **Atomic** – Changes are applied atomically
> - **Recovery** – Database returns to previous state

---

# Why does it exist?

- **Data Integrity** – Prevent partial updates
- **Error Recovery** – Recover from failures gracefully
- **Consistency** – Maintain database consistency
- **Error Handling** – Know how errors are handled
- **Testing** – Verify rollback behavior
- **Debugging** – Understand why data isn't saved

---

# Automatic Rollback

> **Understanding automatic rollback behavior**

## Automatic Rollback on Exception

```dart
// 👇 Automatic rollback on exception
await transaction(() async {
  // 1️⃣ Success
  await into(users).insert(
    UsersCompanion.insert(name: 'John Doe'),
  );
  
  // 2️⃣ This will throw
  await into(users).insert(
    UsersCompanion.insert(name: 'Jane', email: 'invalid'), // ❌
  );
  
  // 3️⃣ This never executes
  await into(users).insert(
    UsersCompanion.insert(name: 'Bob'),
  );
});
// user 'John Doe' is NOT saved (rollback)
// user 'Jane' is NOT saved (failed)
// user 'Bob' is NOT saved (never reached)
```

## Rollback with Nested Operations

```dart
// 👇 Nested operations rollback together
await transaction(() async {
  // 1️⃣ Success
  final orderId = await into(orders).insert(
    OrdersCompanion.insert(orderNumber: 'ORD-123'),
  );
  
  // 2️⃣ Success
  await into(orderItems).insert(
    OrderItemsCompanion.insert(orderId: orderId, productId: 1, quantity: 2),
  );
  
  // 3️⃣ This will throw (invalid product)
  await into(orderItems).insert(
    OrderItemsCompanion.insert(orderId: orderId, productId: 999, quantity: 1),
  );
  
  // 4️⃣ Never reached
  await updateOrderTotal(orderId);
});
// Order and order items are NOT saved
// All changes rolled back
```

---

# Manual Rollback

> **Controlling rollback manually**

## Explicit Rollback

```dart
// 👇 Manual rollback
await db.beginTransaction();

try {
  await insertUser('John');
  
  if (!await isUserValid()) {
    // Manually rollback
    await db.rollbackTransaction();
    throw Exception('User validation failed');
  }
  
  await insertProfile();
  await db.commitTransaction();
} catch (e) {
  await db.rollbackTransaction();
  rethrow;
}
```

## Conditional Rollback

```dart
// 👇 Rollback based on condition
await transaction(() async {
  final userId = await insertUser('John');
  
  // Validate after insert
  final user = await getUser(userId);
  if (user.age < 18) {
    // Rollback transaction
    throw Exception('User must be 18 or older');
  }
  
  await insertProfile(userId);
  await insertSettings(userId);
});
// If age check fails, user is NOT saved
```

---

# Error Handling and Rollback

> **Different error handling strategies**

## Try-Catch Pattern

```dart
// 👇 Try-catch with rollback
try {
  await transaction(() async {
    await insertUser('John');
    await insertProfile(1);
  });
  print('✅ Transaction successful');
} catch (e) {
  print('❌ Transaction failed: $e');
  // Changes already rolled back
}
```

## Retry Pattern

```dart
// 👇 Retry transaction on specific errors
Future<void> retryTransaction() async {
  int retries = 3;
  
  while (retries > 0) {
    try {
      await transaction(() async {
        await insertUser('John');
        await insertProfile(1);
      });
      break; // Success
      
    } on DatabaseException catch (e) {
      retries--;
      if (retries == 0) rethrow;
      print('Retrying... ($retries left)');
      await Future.delayed(Duration(seconds: 1));
    }
  }
}
```

## Partial Success Pattern

```dart
// 👇 Handle partial success with savepoints
await transaction(() async {
  // 1️⃣ Always keep
  await insertUser('John');
  await db.savepoint('user_created');
  
  try {
    // 2️⃣ Attempt but might fail
    await insertProfile(1);
    await db.savepoint('profile_created');
    
    try {
      // 3️⃣ Attempt but might fail
      await insertPosts(1);
    } catch (e) {
      // Rollback only posts
      await db.rollbackToSavepoint('profile_created');
      print('Posts failed, but user and profile saved');
    }
    
  } catch (e) {
    // Rollback profile (keep user)
    await db.rollbackToSavepoint('user_created');
    print('Profile failed, but user saved');
  }
});
```

---

# Real-World Example

> **Complete e-commerce rollback system**

```dart
// lib/database/rollback_service.dart
import 'package:drift/drift.dart';

class RollbackService {
  final AppDatabase db;
  
  RollbackService(this.db);

  // ==================== ORDER ROLLBACK ====================
  
  // 👇 Order with rollback scenarios
  Future<OrderResult> createOrderWithRollback({
    required int userId,
    required List<OrderItem> items,
    required bool simulateFailure,
  }) async {
    try {
      return await db.transaction(() async {
        // 1️⃣ Create order
        final orderId = await db.into(db.orders).insert(
          OrdersCompanion.insert(
            orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
            userId: userId,
            total: 0,
            status: 'pending',
            shippingAddress: '123 Main St',
          ),
        );
        
        // 2️⃣ Add items
        double total = 0;
        for (final item in items) {
          final product = await db.getProduct(item.productId);
          
          // Simulate failure for testing
          if (simulateFailure && item.productId == 2) {
            throw Exception('Product 2 out of stock');
          }
          
          await db.into(db.orderItems).insert(
            OrderItemsCompanion.insert(
              orderId: orderId,
              productId: item.productId,
              quantity: item.quantity,
              unitPrice: product.price,
              subtotal: product.price * item.quantity,
              total: product.price * item.quantity,
            ),
          );
          
          total += product.price * item.quantity;
        }
        
        // 3️⃣ Update order total
        await (db.update(db.orders)..where((o) => o.id.equals(orderId)))
          .write(OrdersCompanion(
            total: Value(total),
          ));
        
        return OrderResult(
          orderId: orderId,
          success: true,
        );
      });
      
    } catch (e) {
      // Transaction rolled back automatically
      print('Order creation failed: $e');
      
      // Check what was saved (should be nothing)
      final orderCount = await db.select(db.orders).count();
      print('Total orders after rollback: $orderCount');
      
      return OrderResult(
        orderId: 0,
        success: false,
        error: e.toString(),
      );
    }
  }

  // ==================== INVENTORY ROLLBACK ====================
  
  // 👇 Inventory reservation with rollback
  Future<ReservationResult> reserveInventory(
    List<InventoryRequest> requests,
  ) async {
    final reserved = <ReservedItem>[];
    final failed = <FailedItem>[];
    
    try {
      await db.transaction(() async {
        for (final request in requests) {
          // Get product
          final product = await db.getProduct(request.productId);
          
          // Check stock
          if (product.stock < request.quantity) {
            failed.add(FailedItem(
              productId: request.productId,
              reason: 'Insufficient stock. Available: ${product.stock}',
            ));
            continue; // Continue with other items
          }
          
          // Reserve stock
          await (db.update(db.products)..where((p) => p.id.equals(product.id)))
            .write(ProductsCompanion(
              stock: Value(product.stock - request.quantity),
              reservedStock: Value(product.reservedStock + request.quantity),
              updatedAt: Value(DateTime.now()),
            ));
          
          reserved.add(ReservedItem(
            productId: request.productId,
            quantity: request.quantity,
          ));
        }
        
        // If all items failed, rollback
        if (reserved.isEmpty && failed.isNotEmpty) {
          throw Exception('No items could be reserved');
        }
        
        // If some failed but some succeeded, keep successful ones
        // Failed items are logged but not rolled back
      });
      
      return ReservationResult(
        reserved: reserved,
        failed: failed,
        success: reserved.isNotEmpty,
      );
      
    } catch (e) {
      // Rollback all reservations
      return ReservationResult(
        reserved: [],
        failed: failed,
        success: false,
        error: e.toString(),
      );
    }
  }

  // ==================== DATA MIGRATION ROLLBACK ====================
  
  // 👇 Migration with full rollback
  Future<MigrationResult> migrateWithRollback() async {
    final steps = <MigrationStep>[];
    
    try {
      await db.transaction(() async {
        // Step 1: Backup
        steps.add(MigrationStep('Backup started', false));
        await _createBackup();
        steps[0] = MigrationStep('Backup completed', true);
        
        // Step 2: Schema change
        steps.add(MigrationStep('Schema change started', false));
        await _addNewColumns();
        steps[1] = MigrationStep('Schema change completed', true);
        
        // Step 3: Data migration
        steps.add(MigrationStep('Data migration started', false));
        await _migrateData();
        steps[2] = MigrationStep('Data migration completed', true);
        
        // Step 4: Validate
        steps.add(MigrationStep('Validation started', false));
        final valid = await _validateMigration();
        if (!valid) {
          throw Exception('Validation failed');
        }
        steps[3] = MigrationStep('Validation completed', true);
      });
      
      return MigrationResult(
        steps: steps,
        success: true,
      );
      
    } catch (e) {
      // Full rollback on any error
      print('Migration failed: $e');
      
      return MigrationResult(
        steps: steps,
        success: false,
        error: e.toString(),
      );
    }
  }

  // ==================== ROLLBACK TESTING ====================
  
  // 👇 Test rollback behavior
  Future<RollbackTestResult> testRollbackBehavior() async {
    final results = <String, dynamic>{};
    
    // 1️⃣ Test automatic rollback
    try {
      await db.transaction(() async {
        await db.into(db.users).insert(
          UsersCompanion.insert(name: 'Rollback User', email: 'rollback@test.com'),
        );
        throw Exception('Test error');
      });
      results['autoRollback'] = 'Failed: Should have rolled back';
    } catch (e) {
      // Check if user was inserted
      final count = await (db.select(db.users)
        ..where((u) => u.email.equals('rollback@test.com')))
        .count();
      
      results['autoRollback'] = count == 0 
          ? '✅ Rollback successful' 
          : '❌ Rollback failed: User exists ($count)';
    }
    
    // 2️⃣ Test savepoint rollback
    try {
      await db.transaction(() async {
        final userId = await db.into(db.users).insert(
          UsersCompanion.insert(name: 'Savepoint User', email: 'savepoint@test.com'),
        );
        
        await db.savepoint('before_profile');
        
        try {
          await db.into(db.profiles).insert(
            ProfilesCompanion.insert(
              userId: userId,
              fullName: Value('Test User'),
            ),
          );
          throw Exception('Profile error');
        } catch (e) {
          await db.rollbackToSavepoint('before_profile');
        }
        
        // User remains, profile rolled back
      });
      
      // Check results
      final user = await (db.select(db.users)
        ..where((u) => u.email.equals('savepoint@test.com')))
        .getSingleOrNull();
      
      final profile = await (db.select(db.profiles)
        ..where((p) => p.userId.equals(user?.id ?? 0)))
        .getSingleOrNull();
      
      results['savepointRollback'] = user != null && profile == null
          ? '✅ Savepoint rollback successful'
          : '❌ Savepoint rollback failed';
      
    } catch (e) {
      results['savepointRollback'] = '❌ Error: $e';
    }
    
    return RollbackTestResult(results: results);
  }

  // ==================== HELPER METHODS ====================
  
  Future<void> _createBackup() async {
    await db.customSelect('''
      CREATE TABLE IF NOT EXISTS migration_backup AS 
      SELECT * FROM users WHERE 1=0
    ''').get();
  }
  
  Future<void> _addNewColumns() async {
    await db.customSelect('''
      ALTER TABLE users ADD COLUMN new_field TEXT
    ''').get();
  }
  
  Future<void> _migrateData() async {
    await db.customSelect('''
      UPDATE users SET new_field = 'migrated'
    ''').get();
  }
  
  Future<bool> _validateMigration() async {
    final count = await db.select(db.users)
      .where((u) => u.newField.isNotNull())
      .count();
    return count > 0;
  }
}

// ==================== DATA CLASSES ====================

class OrderResult {
  final int orderId;
  final bool success;
  final String? error;
  
  OrderResult({
    required this.orderId,
    required this.success,
    this.error,
  });
}

class InventoryRequest {
  final int productId;
  final int quantity;
  
  InventoryRequest({
    required this.productId,
    required this.quantity,
  });
}

class ReservedItem {
  final int productId;
  final int quantity;
  
  ReservedItem({
    required this.productId,
    required this.quantity,
  });
}

class FailedItem {
  final int productId;
  final String reason;
  
  FailedItem({
    required this.productId,
    required this.reason,
  });
}

class ReservationResult {
  final List<ReservedItem> reserved;
  final List<FailedItem> failed;
  final bool success;
  final String? error;
  
  ReservationResult({
    required this.reserved,
    required this.failed,
    required this.success,
    this.error,
  });
}

class MigrationStep {
  final String description;
  final bool completed;
  final String? error;
  
  MigrationStep(this.description, this.completed, [this.error]);
}

class MigrationResult {
  final List<MigrationStep> steps;
  final bool success;
  final String? error;
  
  MigrationResult({
    required this.steps,
    required this.success,
    this.error,
  });
}

class RollbackTestResult {
  final Map<String, dynamic> results;
  
  RollbackTestResult({required this.results});
}
```

---

# Rollback Behavior Checklist

| Scenario | Behavior | Best Practice |
|----------|----------|---------------|
| **Exception** | Automatic rollback | Use try-catch |
| **Manual Rollback** | Explicit rollback | Use with conditions |
| **Savepoint Rollback** | Partial rollback | Use for complex operations |
| **Retry** | Retry on failure | Use with retry count |
| **Testing** | Verify rollback | Use test framework |

---

# Common Mistakes

## Mistake 1: Not handling rollback errors

Wrong:
```dart
// 🚫 Rollback may fail
await db.rollbackTransaction();
```

Correct:
```dart
// ✅ Handle rollback errors
try {
  await db.rollbackTransaction();
} catch (e) {
  print('Rollback failed: $e');
}
```

## Mistake 2: Trying to commit after rollback

Wrong:
```dart
// 🚫 Can't commit after rollback
await db.rollbackTransaction();
await db.commitTransaction(); // Error!
```

Correct:
```dart
// ✅ Commit only on success
try {
  await transaction(() async {
    await operation();
  });
  // Auto-commit on success
} catch (e) {
  // Auto-rollback on error
}
```

## Mistake 3: Not verifying rollback

Wrong:
```dart
// 🚫 Assume rollback worked
await rollback();
// Continue without verifying
```

Correct:
```dart
// ✅ Verify rollback state
await rollback();
final count = await select(users).count();
if (count > 0) {
  // Handle unexpected state
}
```

---

# Summary

| Behavior | Description | Best Practice |
|----------|-------------|---------------|
| **Auto Rollback** | Exception triggers rollback | Try-catch pattern |
| **Manual Rollback** | Explicit rollback | Use conditionally |
| **Savepoint** | Partial rollback | Complex operations |
| **Retry** | Retry on failure | With retry count |
| **Testing** | Verify rollback | Test framework |

---

# Next Steps

Now you understand rollback behavior, let's dive deeper:

- [Custom SQL](link) – Advanced SQL features
- [Views](link) – Database views
- [Indexes](link) – Performance optimization

---

# Did You Know?

- **Rollbacks are atomic** – All or nothing

- **Rollbacks are fast** – Database handles efficiently

- **Rollbacks are automatic** – On any exception

- **Rollbacks are reversible** – Can retry after rollback

- **Rollbacks are essential** – For data integrity

- **Rollbacks are tested** – By Drift automatically

- **Rollbacks work with savepoints** – Nested rollbacks

- **Rollbacks are transaction-safe** – Consistent state

---

