## Nested Transactions

**Managing complex operations with nested transactions and savepoints**

---

# What is it?

**Nested Transactions** (also known as **Savepoints**) allow you to create sub-transactions within a larger transaction. This gives you fine-grained control over transaction rollbacks – you can roll back part of a transaction while keeping the rest. Drift supports savepoints through the `savepoint()` and `rollbackToSavepoint()` methods.

> **Think of Nested Transactions like "having multiple undo levels"** – instead of only being able to undo everything, you can undo just the last few steps while keeping the earlier work safe.

```dart
// 👇 Transaction with savepoint
await transaction(() async {
  // 1️⃣ Outer transaction
  final userId = await createUser('John');
  await createProfile(userId, 'John Doe');
  
  // 2️⃣ Create savepoint before risky operation
  await db.savepoint('before_posts');
  
  try {
    // 3️⃣ Risky operation that might fail
    await createPosts(userId, ['Post 1', 'Post 2']);
  } catch (e) {
    // 4️⃣ Rollback to savepoint (undo only posts)
    await db.rollbackToSavepoint('before_posts');
    print('Posts creation failed, but user and profile saved');
  }
  
  // 5️⃣ Outer transaction continues
  await updateUserStatus(userId, 'active');
});
```

> **What's happening here?**
> - **`savepoint()`** – Creates a checkpoint in the transaction
> - **`rollbackToSavepoint()`** – Undoes changes since savepoint
> - **Nested control** – Rollback part of transaction
> - **Recovery** – Continue after partial failure

---

# Why does it exist?

- **Partial Rollback** – Undo part of a transaction
- **Error Recovery** – Recover from errors without losing all work
- **Complex Operations** – Handle multi-step processes
- **Batch Processing** – Process items with individual recovery
- **Data Validation** – Validate and rollback specific changes
- **Performance** – Reduce full transaction rollbacks

---

# Basic Nested Transactions

> **Simple savepoint patterns**

## Basic Savepoint

```dart
// 👇 Basic savepoint usage
await transaction(() async {
  // Outer transaction
  await insertUser('John');
  
  // Create savepoint
  await db.savepoint('after_user');
  
  try {
    // Operations that might fail
    await insertPosts();
    await updateUserStats();
  } catch (e) {
    // Rollback to savepoint (undo posts, keep user)
    await db.rollbackToSavepoint('after_user');
    print('Posts failed, but user is saved');
  }
  
  // Continue transaction
  await finalizeUser('John');
});
```

## Multiple Savepoints

```dart
// 👇 Multiple savepoints for granular control
await transaction(() async {
  // Step 1: Create user
  await insertUser('John');
  await db.savepoint('user_created');
  
  // Step 2: Create profile
  await insertProfile(1, 'John Doe');
  await db.savepoint('profile_created');
  
  // Step 3: Create posts (risky)
  try {
    await insertPosts(1, ['Post 1', 'Post 2']);
    await db.savepoint('posts_created');
  } catch (e) {
    // Rollback to profile (keep user and profile)
    await db.rollbackToSavepoint('profile_created');
    print('Posts failed, but user and profile saved');
  }
  
  // Step 4: Create settings
  await insertSettings(1);
  await db.savepoint('settings_created');
});
```

---

# Advanced Nested Transaction Patterns

> **Complex savepoint scenarios**

## Pattern 1: Batch Processing with Individual Rollbacks

```dart
// 👇 Process batch items with individual rollbacks
Future<void> processOrderBatch(List<OrderData> orders) async {
  await transaction(() async {
    for (var i = 0; i < orders.length; i++) {
      final order = orders[i];
      final savepointName = 'order_${i}_${order.id}';
      
      try {
        // Create savepoint for each order
        await db.savepoint(savepointName);
        
        // Process order
        await validateOrder(order);
        await insertOrder(order);
        await updateInventory(order.items);
        
        // Success - keep this order
        print('✅ Order ${order.id} processed');
        
      } catch (e) {
        // Rollback only this order
        await db.rollbackToSavepoint(savepointName);
        print('❌ Order ${order.id} failed: $e');
        // Continue with next order
      }
    }
    
    // All orders processed successfully or skipped
    await updateBatchStatus('completed');
  });
}
```

---

## Pattern 2: Try-Catch with Savepoint

```dart
// 👇 Try multiple strategies with savepoints
Future<void> processPaymentWithRetry(int orderId) async {
  await transaction(() async {
    // Savepoint before payment attempt
    await db.savepoint('payment_attempt');
    
    try {
      // Try primary payment method
      await processPrimaryPayment(orderId);
      
    } catch (e) {
      // Rollback to before primary attempt
      await db.rollbackToSavepoint('payment_attempt');
      
      try {
        // Try secondary payment method
        await processSecondaryPayment(orderId);
        
      } catch (e2) {
        // Rollback both attempts
        await db.rollbackToSavepoint('payment_attempt');
        throw Exception('Both payment methods failed');
      }
    }
    
    // Payment successful
    await finalizeOrder(orderId);
  });
}
```

---

## Pattern 3: Validating Data with Savepoints

```dart
// 👇 Validate and rollback invalid data
Future<void> importUsers(List<UserData> users) async {
  await transaction(() async {
    final validUsers = <UserData>[];
    final errors = <String>[];
    
    for (var i = 0; i < users.length; i++) {
      final user = users[i];
      final savepointName = 'user_$i';
      
      await db.savepoint(savepointName);
      
      try {
        // Validate user
        if (user.email.isEmpty) {
          throw Exception('Email required');
        }
        if (user.name.length < 2) {
          throw Exception('Name too short');
        }
        
        // Insert valid user
        await into(users).insert(
          UsersCompanion.insert(
            name: user.name,
            email: user.email,
          ),
        );
        
        validUsers.add(user);
        
      } catch (e) {
        // Rollback invalid user
        await db.rollbackToSavepoint(savepointName);
        errors.add('User ${i + 1}: $e');
      }
    }
    
    if (validUsers.isEmpty) {
      throw Exception('No valid users to import');
    }
    
    print('✅ Imported ${validUsers.length} users');
    if (errors.isNotEmpty) {
      print('⚠️ Errors: ${errors.join(', ')}');
    }
  });
}
```

---

# Real-World Example

> **Complete e-commerce nested transaction system**

```dart
// lib/database/nested_transaction_service.dart
import 'package:drift/drift.dart';

class NestedTransactionService {
  final AppDatabase db;
  
  NestedTransactionService(this.db);

  // ==================== BULK ORDER PROCESSING ====================
  
  // 👇 Process multiple orders with individual rollbacks
  Future<BatchOrderResult> processBulkOrders(
    List<OrderInput> orders,
  ) async {
    final results = <OrderResult>[];
    
    await db.transaction(() async {
      for (var i = 0; i < orders.length; i++) {
        final order = orders[i];
        final savepointName = 'order_${i}_${order.id}';
        
        try {
          await db.savepoint(savepointName);
          
          // Process order
          final result = await _processSingleOrder(order);
          results.add(result);
          
        } catch (e) {
          // Rollback only this order
          await db.rollbackToSavepoint(savepointName);
          results.add(OrderResult.failed(order.id, e.toString()));
        }
      }
    });
    
    return BatchOrderResult(
      total: orders.length,
      successful: results.where((r) => r.success).length,
      failed: results.where((r) => !r.success).length,
      results: results,
    );
  }
  
  Future<OrderResult> _processSingleOrder(OrderInput order) async {
    // Validate inventory
    for (final item in order.items) {
      final product = await db.getProduct(item.productId);
      if (product.stock < item.quantity) {
        throw Exception('Insufficient stock for ${product.name}');
      }
    }
    
    // Create order
    final orderId = await db.into(db.orders).insert(
      OrdersCompanion.insert(
        orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
        userId: order.userId,
        total: _calculateTotal(order.items),
        status: 'pending',
        shippingAddress: order.shippingAddress,
        billingAddress: order.billingAddress,
      ),
    );
    
    // Add items
    for (final item in order.items) {
      final product = await db.getProduct(item.productId);
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
    }
    
    return OrderResult.success(orderId, order.id);
  }

  // ==================== BULK PRODUCT UPDATE ====================
  
  // 👇 Update products with validation and rollback
  Future<BulkUpdateResult> bulkUpdateProducts(
    List<ProductUpdate> updates,
  ) async {
    final results = <ProductUpdateResult>[];
    
    await db.transaction(() async {
      for (var i = 0; i < updates.length; i++) {
        final update = updates[i];
        final savepointName = 'product_$i';
        
        try {
          await db.savepoint(savepointName);
          
          // Validate and update
          final product = await db.getProduct(update.id);
          
          if (update.price != null && update.price! < 0) {
            throw Exception('Invalid price: ${update.price}');
          }
          
          await (db.update(db.products)..where((p) => p.id.equals(product.id)))
            .write(ProductsCompanion(
              price: update.price != null 
                  ? Value(update.price!) 
                  : const Value.absent(),
              stock: update.stock != null 
                  ? Value(update.stock!) 
                  : const Value.absent(),
              updatedAt: Value(DateTime.now()),
            ));
          
          results.add(ProductUpdateResult.success(update.id));
          
        } catch (e) {
          await db.rollbackToSavepoint(savepointName);
          results.add(ProductUpdateResult.failed(update.id, e.toString()));
        }
      }
    });
    
    return BulkUpdateResult(
      total: updates.length,
      successful: results.where((r) => r.success).length,
      failed: results.where((r) => !r.success).length,
      results: results,
    );
  }

  // ==================== DATA MIGRATION WITH SAVEPOINTS ====================
  
  // 👇 Complex data migration with rollback points
  Future<MigrationResult> migrateData() async {
    final steps = <MigrationStep>[];
    
    await db.transaction(() async {
      // Step 1: Backup data
      await db.savepoint('backup_created');
      await _createBackupTables();
      steps.add(MigrationStep('Backup created', true));
      
      // Step 2: Add new columns
      try {
        await db.savepoint('columns_added');
        await _addNewColumns();
        steps.add(MigrationStep('Columns added', true));
      } catch (e) {
        await db.rollbackToSavepoint('backup_created');
        steps.add(MigrationStep('Columns failed', false, e.toString()));
        rethrow;
      }
      
      // Step 3: Migrate data
      try {
        await db.savepoint('data_migrated');
        await _migrateExistingData();
        steps.add(MigrationStep('Data migrated', true));
      } catch (e) {
        await db.rollbackToSavepoint('columns_added');
        steps.add(MigrationStep('Data migration failed', false, e.toString()));
        rethrow;
      }
      
      // Step 4: Clean up
      try {
        await db.savepoint('cleanup_done');
        await _cleanupOldData();
        steps.add(MigrationStep('Cleanup done', true));
      } catch (e) {
        await db.rollbackToSavepoint('data_migrated');
        steps.add(MigrationStep('Cleanup failed', false, e.toString()));
        rethrow;
      }
    });
    
    return MigrationResult(
      steps: steps,
      success: steps.every((s) => s.success),
    );
  }
  
  Future<void> _createBackupTables() async {
    await db.customSelect('''
      CREATE TABLE IF NOT EXISTS backup_products AS SELECT * FROM products
    ''').get();
  }
  
  Future<void> _addNewColumns() async {
    await db.customSelect('''
      ALTER TABLE products ADD COLUMN new_price REAL
    ''').get();
  }
  
  Future<void> _migrateExistingData() async {
    await db.customSelect('''
      UPDATE products SET new_price = price * 1.1
    ''').get();
  }
  
  Future<void> _cleanupOldData() async {
    await db.customSelect('''
      DELETE FROM products WHERE is_active = 0
    ''').get();
  }

  // ==================== ORDER FULFILLMENT WITH RETRIES ====================
  
  // 👇 Fulfill order with retry strategies
  Future<FulfillmentResult> fulfillOrder(int orderId) async {
    return await db.transaction(() async {
      // Get order
      final order = await db.getOrder(orderId);
      
      // Savepoint before fulfillment
      await db.savepoint('fulfillment_start');
      
      // Try warehouse A
      try {
        await db.savepoint('warehouse_a');
        await _fulfillFromWarehouse(order, 'A');
        
      } catch (e) {
        await db.rollbackToSavepoint('warehouse_a');
        
        // Try warehouse B
        try {
          await db.savepoint('warehouse_b');
          await _fulfillFromWarehouse(order, 'B');
          
        } catch (e2) {
          await db.rollbackToSavepoint('warehouse_b');
          
          // Try warehouse C
          try {
            await db.savepoint('warehouse_c');
            await _fulfillFromWarehouse(order, 'C');
            
          } catch (e3) {
            await db.rollbackToSavepoint('warehouse_c');
            throw Exception('All warehouses failed');
          }
        }
      }
      
      // Update order status
      await (db.update(db.orders)..where((o) => o.id.equals(orderId)))
        .write(OrdersCompanion(
          status: Value('fulfilled'),
          fulfilledAt: Value(DateTime.now()),
        ));
      
      return FulfillmentResult(
        orderId: orderId,
        status: 'fulfilled',
        timestamp: DateTime.now(),
      );
    });
  }
  
  Future<void> _fulfillFromWarehouse(Order order, String warehouseId) async {
    // Mock warehouse fulfillment
    if (warehouseId == 'A' && order.total > 1000) {
      throw Exception('Warehouse A: Order too large');
    }
    if (warehouseId == 'B' && order.items.length > 10) {
      throw Exception('Warehouse B: Too many items');
    }
    // Warehouse C always succeeds
    print('✅ Fulfilled from warehouse $warehouseId');
  }
}

// ==================== DATA CLASSES ====================

class OrderInput {
  final String id;
  final int userId;
  final List<CartItem> items;
  final String shippingAddress;
  final String billingAddress;
  
  OrderInput({
    required this.id,
    required this.userId,
    required this.items,
    required this.shippingAddress,
    required this.billingAddress,
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

class OrderResult {
  final String orderId;
  final bool success;
  final String? error;
  
  OrderResult._({required this.orderId, required this.success, this.error});
  
  factory OrderResult.success(int id, String orderId) {
    return OrderResult._(orderId: '$id', success: true);
  }
  
  factory OrderResult.failed(String orderId, String error) {
    return OrderResult._(orderId: orderId, success: false, error: error);
  }
}

class BatchOrderResult {
  final int total;
  final int successful;
  final int failed;
  final List<OrderResult> results;
  
  BatchOrderResult({
    required this.total,
    required this.successful,
    required this.failed,
    required this.results,
  });
}

class ProductUpdate {
  final int id;
  final double? price;
  final int? stock;
  
  ProductUpdate({
    required this.id,
    this.price,
    this.stock,
  });
}

class ProductUpdateResult {
  final int productId;
  final bool success;
  final String? error;
  
  ProductUpdateResult._({required this.productId, required this.success, this.error});
  
  factory ProductUpdateResult.success(int id) {
    return ProductUpdateResult._(productId: id, success: true);
  }
  
  factory ProductUpdateResult.failed(int id, String error) {
    return ProductUpdateResult._(productId: id, success: false, error: error);
  }
}

class BulkUpdateResult {
  final int total;
  final int successful;
  final int failed;
  final List<ProductUpdateResult> results;
  
  BulkUpdateResult({
    required this.total,
    required this.successful,
    required this.failed,
    required this.results,
  });
}

class MigrationStep {
  final String name;
  final bool success;
  final String? error;
  
  MigrationStep(this.name, this.success, [this.error]);
}

class MigrationResult {
  final List<MigrationStep> steps;
  final bool success;
  
  MigrationResult({
    required this.steps,
    required this.success,
  });
}

class FulfillmentResult {
  final int orderId;
  final String status;
  final DateTime timestamp;
  
  FulfillmentResult({
    required this.orderId,
    required this.status,
    required this.timestamp,
  });
}
```

```dart
// lib/ui/pages/bulk_order_page.dart
class BulkOrderPage extends StatefulWidget {
  final NestedTransactionService transactionService;
  
  const BulkOrderPage({required this.transactionService});
  
  @override
  _BulkOrderPageState createState() => _BulkOrderPageState();
}

class _BulkOrderPageState extends State<BulkOrderPage> {
  BatchOrderResult? _result;
  bool _isProcessing = false;
  
  Future<void> _processBulkOrders() async {
    setState(() => _isProcessing = true);
    
    try {
      // Mock orders
      final orders = [
        OrderInput(
          id: '1',
          userId: 1,
          items: [CartItem(productId: 1, quantity: 2)],
          shippingAddress: '123 Main St',
          billingAddress: '123 Main St',
        ),
        OrderInput(
          id: '2',
          userId: 2,
          items: [CartItem(productId: 2, quantity: 1)],
          shippingAddress: '456 Oak Ave',
          billingAddress: '456 Oak Ave',
        ),
        OrderInput(
          id: '3',
          userId: 3,
          items: [CartItem(productId: 3, quantity: 5)],
          shippingAddress: '789 Pine Rd',
          billingAddress: '789 Pine Rd',
        ),
      ];
      
      final result = await widget.transactionService.processBulkOrders(orders);
      
      setState(() {
        _result = result;
        _isProcessing = false;
      });
      
    } catch (e) {
      setState(() => _isProcessing = false);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Bulk Order Processing'),
        backgroundColor: Colors.purple[700],
      ),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // Status
            Card(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: Column(
                  children: [
                    Text(
                      _isProcessing ? 'Processing...' : 'Ready',
                      style: TextStyle(
                        fontSize: 18,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    SizedBox(height: 8),
                    if (_result != null) ...[
                      Text('Total: ${_result!.total}'),
                      Text('✅ Successful: ${_result!.successful}'),
                      Text('❌ Failed: ${_result!.failed}'),
                    ],
                  ],
                ),
              ),
            ),
            SizedBox(height: 16),
            
            // Process button
            ElevatedButton(
              onPressed: _isProcessing ? null : _processBulkOrders,
              style: ElevatedButton.styleFrom(
                minimumSize: Size(double.infinity, 50),
              ),
              child: _isProcessing
                  ? CircularProgressIndicator()
                  : Text('Process Bulk Orders'),
            ),
            SizedBox(height: 16),
            
            // Results
            if (_result != null)
              Expanded(
                child: ListView.builder(
                  itemCount: _result!.results.length,
                  itemBuilder: (context, index) {
                    final result = _result!.results[index];
                    return Card(
                      color: result.success ? Colors.green[50] : Colors.red[50],
                      child: ListTile(
                        leading: Icon(
                          result.success ? Icons.check_circle : Icons.error,
                          color: result.success ? Colors.green : Colors.red,
                        ),
                        title: Text('Order ${result.orderId}'),
                        subtitle: result.error != null 
                            ? Text('Error: ${result.error}')
                            : Text('Processed successfully'),
                      ),
                    );
                  },
                ),
              ),
          ],
        ),
      ),
    );
  }
}
```

---

# Nested Transaction Best Practices

- **Use meaningful savepoint names** – Descriptive and unique
- **Rollback to savepoint when needed** – Recover from errors
- **Release savepoints when done** – Clean up resources
- **Use try-catch blocks** – Handle errors gracefully
- **Avoid too many savepoints** – Performance overhead
- **Test savepoint behavior** – Verify rollback works
- **Use savepoints for validation** – Validate and rollback invalid data
- **Document savepoint logic** – Explain complex transactions

---

# Nested Transaction Checklist

| Practice | Description | Impact |
|----------|-------------|--------|
| **Meaningful Names** | Descriptive savepoint names | High |
| **Try-Catch** | Handle errors gracefully | High |
| **Resource Cleanup** | Release savepoints | Medium |
| **Testing** | Verify rollback behavior | High |
| **Documentation** | Explain complex logic | Medium |
| **Performance** | Avoid too many savepoints | Medium |

---

# Common Mistakes

## Mistake 1: Rolling back wrong savepoint

Wrong:
```dart
// 🚫 Rolling back parent before child
await db.savepoint('parent');
await db.savepoint('child');
await db.rollbackToSavepoint('parent'); // Rolls back child too
```

Correct:
```dart
// ✅ Rollback child only
await db.savepoint('parent');
await db.savepoint('child');
await db.rollbackToSavepoint('child'); // Only child rolled back
```

## Mistake 2: Using savepoints after commit

Wrong:
```dart
// 🚫 Savepoint after commit
await db.commitTransaction();
await db.savepoint('after_commit'); // Error!
```

Correct:
```dart
// ✅ Savepoints before commit
await db.savepoint('before_commit');
await db.commitTransaction();
```

## Mistake 3: Not handling savepoint errors

Wrong:
```dart
// 🚫 Savepoint error crashes transaction
await db.savepoint('test');
// Operation fails, but no rollback
```

Correct:
```dart
// ✅ Handle errors with rollback
try {
  await db.savepoint('test');
  // Operation
} catch (e) {
  await db.rollbackToSavepoint('test');
}
```

---

# Summary

| Feature | Purpose | Example |
|---------|---------|---------|
| **Savepoint** | Create checkpoint | `db.savepoint('name')` |
| **Rollback To** | Undo to savepoint | `db.rollbackToSavepoint('name')` |
| **Nested** | Inner transactions | Savepoints within transaction |
| **Recovery** | Partial rollback | Error recovery |

---

# Next Steps

Now you understand nested transactions, let's dive deeper:

- [Batch vs Transaction](link) – Choosing the right approach
- [Rollback Behavior](link) – Understanding rollbacks
- [Custom SQL](link) – Advanced SQL features

---

# Did You Know?

- **Savepoints are nested** – You can have multiple levels

- **Rollback to savepoint** – Undoes changes since savepoint

- **Savepoints are database features** – Supported by SQLite

- **Savepoints have overhead** – Too many affects performance

- **Savepoints can be named** – For easy reference

- **Savepoints are optional** – Not all transactions need them

- **Savepoints are powerful** – For complex operations

- **Savepoints are used in nested transactions** – Internal implementation

---
