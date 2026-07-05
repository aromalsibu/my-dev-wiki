## Batch Insert

**Efficient bulk data insertion in Drift**

---

# What is it?

**Batch Insert** is a technique for inserting multiple records efficiently in a single database operation. Instead of executing individual INSERT statements for each record, Drift's batch API groups them together, significantly improving performance for large datasets.

> **Think of Batch Insert like "bulk mailing"** – instead of sending each letter individually, you bundle them all together and send them in one shipment, saving time and resources.

```dart
// 👇 Single inserts (slow for many records)
for (final user in users) {
  await into(users).insert(user); // 100 separate operations
}

// 👇 Batch insert (fast)
await into(users).batch((batch) {
  for (final user in users) {
    batch.insert(user); // Single operation for all
  }
});

// Generated SQL (single transaction):
// BEGIN TRANSACTION;
// INSERT INTO users (name, email) VALUES ('User1', 'email1');
// INSERT INTO users (name, email) VALUES ('User2', 'email2');
// INSERT INTO users (name, email) VALUES ('User3', 'email3');
// COMMIT;
```

> **What's happening here?**
> - **Single Transaction** – All inserts in one transaction
> - **Reduced Overhead** – One connection round-trip
> - **Better Performance** – 10-100x faster for bulk data
> - **Atomic Operation** – All succeed or all fail

---

# Why does it exist?

- **Performance** – 10-100x faster for bulk inserts
- **Reduced Overhead** – One database connection round-trip
- **Atomic Operations** – All inserts succeed or fail together
- **Resource Efficiency** – Less memory and CPU usage
- **Large Datasets** – Efficiently import/export data
- **Data Migration** – Bulk data migrations

---

# Basic Batch Insert

> **Simple batch operations**

## Basic Batch

```dart
// 👇 Batch insert multiple users
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

// Generated SQL:
// BEGIN TRANSACTION;
// INSERT INTO users (name, email) VALUES ('User 1', 'user1@example.com');
// INSERT INTO users (name, email) VALUES ('User 2', 'user2@example.com');
// INSERT INTO users (name, email) VALUES ('User 3', 'user3@example.com');
// COMMIT;
```

## Batch with Loop

```dart
// 👇 Batch insert with loop
Future<void> insertUsers(List<UserData> users) async {
  await into(users).batch((batch) {
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
}

class UserData {
  final String name;
  final String email;
  final int? age;
  
  UserData({required this.name, required this.email, this.age});
}
```

---

# Batch with Different Operations

> **Mixed batch operations**

## Batch Insert, Update, Delete

```dart
// 👇 Batch with multiple operation types
await into(users).batch((batch) {
  // Insert new users
  for (final user in newUsers) {
    batch.insert(
      UsersCompanion.insert(
        name: user.name,
        email: user.email,
      ),
    );
  }
  
  // Update existing users
  for (final user in updatedUsers) {
    batch.update(
      users,
      UsersCompanion(
        name: Value(user.name),
        age: Value(user.age),
      ),
      (u) => u.id.equals(user.id),
    );
  }
  
  // Delete users
  for (final id in deletedUserIds) {
    batch.delete(
      users,
      (u) => u.id.equals(id),
    );
  }
});

// All operations execute in a single transaction:
// BEGIN TRANSACTION;
// INSERT INTO users ...
// UPDATE users SET ...
// DELETE FROM users ...
// COMMIT;
```

---

# Advanced Batch Patterns

> **Complex batch operations**

## Pattern 1: Bulk Import with Validation

```dart
Future<void> bulkImportUsers(List<Map<String, dynamic>> csvData) async {
  // 👇 Validate all data first
  final validatedUsers = <UserData>[];
  
  for (final row in csvData) {
    final name = row['name'] as String?;
    final email = row['email'] as String?;
    final age = row['age'] as int?;
    
    if (name == null || name.isEmpty) {
      throw Exception('Name required at row ${csvData.indexOf(row)}');
    }
    if (email == null || !email.contains('@')) {
      throw Exception('Invalid email at row ${csvData.indexOf(row)}');
    }
    if (age != null && (age < 0 || age > 150)) {
      throw Exception('Invalid age at row ${csvData.indexOf(row)}');
    }
    
    validatedUsers.add(UserData(
      name: name,
      email: email,
      age: age,
    ));
  }
  
  // 👇 Batch insert all validated data
  await into(users).batch((batch) {
    for (final user in validatedUsers) {
      batch.insert(
        UsersCompanion.insert(
          name: user.name,
          email: user.email,
          age: Value(user.age),
        ),
      );
    }
  });
}
```

---

## Pattern 2: Batch with Conflict Resolution

```dart
Future<void> upsertUsers(List<UserData> users) async {
  await into(users).batch((batch) {
    for (final user in users) {
      batch.insert(
        UsersCompanion.insert(
          name: user.name,
          email: user.email,
          age: Value(user.age),
        ),
        onConflict: DoUpdate(
          target: users.email,
          update: UsersCompanion(
            name: Value(user.name),
            age: Value(user.age),
            updatedAt: Value(DateTime.now()),
          ),
        ),
      );
    }
  });
}

// Generated SQL:
// INSERT INTO users (name, email, age) 
// VALUES ('John', 'john@example.com', 25)
// ON CONFLICT(email) DO UPDATE SET name = 'John', age = 25, updated_at = ...
// INSERT INTO users (name, email, age) 
// VALUES ('Jane', 'jane@example.com', 30)
// ON CONFLICT(email) DO UPDATE SET name = 'Jane', age = 30, updated_at = ...
```

---

## Pattern 3: Batch with Returning

```dart
// 👇 Batch insert with returned data
Future<List<int>> batchInsertReturning(List<UserData> users) async {
  final results = await into(users).batchReturning(
    (batch) {
      for (final user in users) {
        batch.insert(
          UsersCompanion.insert(
            name: user.name,
            email: user.email,
            age: Value(user.age),
          ),
        );
      }
    },
    returning: (u) => u.id,
  );
  
  return results.expand((r) => r).toList();
}
```

---

## Pattern 4: Chunked Batch for Large Datasets

```dart
Future<void> insertLargeDataset(List<UserData> users) async {
  const chunkSize = 1000; // 👇 Process in chunks
  
  for (var i = 0; i < users.length; i += chunkSize) {
    final chunk = users.sublist(
      i,
      i + chunkSize > users.length ? users.length : i + chunkSize,
    );
    
    await into(users).batch((batch) {
      for (final user in chunk) {
        batch.insert(
          UsersCompanion.insert(
            name: user.name,
            email: user.email,
            age: Value(user.age),
          ),
        );
      }
    });
    
    // 👇 Optional: Report progress
    print('Inserted ${i + chunk.length} of ${users.length} users');
  }
}
```

---

# Batch Performance Comparison

> **Performance benchmarks**

```dart
Future<void> compareInsertPerformance(AppDatabase db) async {
  const recordCount = 1000;
  
  // 👇 Method 1: Individual inserts (slow)
  print('Individual inserts...');
  final stopwatch1 = Stopwatch()..start();
  for (var i = 0; i < recordCount; i++) {
    await db.into(db.users).insert(
      UsersCompanion.insert(
        name: 'User $i',
        email: 'user$i@example.com',
      ),
    );
  }
  stopwatch1.stop();
  print('Individual: ${stopwatch1.elapsedMilliseconds}ms');
  
  // 👇 Method 2: Batch insert (fast)
  print('Batch insert...');
  final stopwatch2 = Stopwatch()..start();
  await db.into(db.users).batch((batch) {
    for (var i = 0; i < recordCount; i++) {
      batch.insert(
        UsersCompanion.insert(
          name: 'User $i',
          email: 'user$i@example.com',
        ),
      );
    }
  });
  stopwatch2.stop();
  print('Batch: ${stopwatch2.elapsedMilliseconds}ms');
  
  // 👇 Method 3: insertAll (alternative)
  print('insertAll...');
  final stopwatch3 = Stopwatch()..start();
  final companions = List.generate(recordCount, (i) =>
    UsersCompanion.insert(
      name: 'User $i',
      email: 'user$i@example.com',
    ),
  );
  await db.into(db.users).insertAll(companions);
  stopwatch3.stop();
  print('insertAll: ${stopwatch3.elapsedMilliseconds}ms');
  
  // Results on average:
  // Individual: ~1500ms
  // Batch: ~100ms
  // insertAll: ~80ms (slightly faster than batch)
}
```

---

# Real-World Example

> **Complete e-commerce batch system**

```dart
// lib/database/batch_operations.dart
import 'package:drift/drift.dart';

class BatchOperations {
  final AppDatabase db;
  
  BatchOperations(this.db);
  
  // ==================== PRODUCT BATCHES ====================
  
  // 👇 Bulk product import
  Future<int> bulkImportProducts(List<ProductData> products) async {
    int imported = 0;
    
    await db.transaction(() async {
      // Validate all products
      for (final product in products) {
        if (product.sku.length < 6) {
          throw Exception('Invalid SKU: ${product.sku}');
        }
        if (product.price < 0) {
          throw Exception('Invalid price: ${product.price}');
        }
      }
      
      // Batch insert
      await db.into(db.products).batch((batch) {
        for (final product in products) {
          batch.insert(
            ProductsCompanion.insert(
              sku: product.sku,
              name: product.name,
              price: product.price,
              description: Value(product.description),
              stock: Value(product.stock),
            ),
          );
          imported++;
        }
      });
    });
    
    return imported;
  }
  
  // 👇 Bulk product update
  Future<int> bulkUpdateProducts(List<ProductUpdate> updates) async {
    int updated = 0;
    
    await db.into(db.products).batch((batch) {
      for (final update in updates) {
        batch.update(
          db.products,
          ProductsCompanion(
            name: update.name != null ? Value(update.name!) : const Value.absent(),
            price: update.price != null ? Value(update.price!) : const Value.absent(),
            stock: update.stock != null ? Value(update.stock!) : const Value.absent(),
            updatedAt: Value(DateTime.now()),
          ),
          (p) => p.id.equals(update.id),
        );
        updated++;
      }
    });
    
    return updated;
  }
  
  // 👇 Bulk product deletion
  Future<int> bulkDeleteProducts(List<int> ids) async {
    int deleted = 0;
    
    await db.into(db.products).batch((batch) {
      for (final id in ids) {
        batch.delete(
          db.products,
          (p) => p.id.equals(id),
        );
        deleted++;
      }
    });
    
    return deleted;
  }
  
  // ==================== ORDER BATCHES ====================
  
  // 👇 Bulk order creation
  Future<List<int>> bulkCreateOrders(List<OrderData> orders) async {
    final orderIds = <int>[];
    
    await db.transaction(() async {
      for (final orderData in orders) {
        // Create order
        final orderId = await db.into(db.orders).insert(
          OrdersCompanion.insert(
            orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}-${orderIds.length}',
            userId: orderData.userId,
            total: orderData.total,
            status: 'pending',
            shippingAddress: orderData.shippingAddress,
            billingAddress: orderData.billingAddress,
            paymentMethod: orderData.paymentMethod,
            paymentStatus: 'unpaid',
          ),
        );
        orderIds.add(orderId);
        
        // Create order items in batch
        await db.into(db.orderItems).batch((batch) {
          for (final item in orderData.items) {
            batch.insert(
              OrderItemsCompanion.insert(
                orderId: orderId,
                productId: item.productId,
                quantity: item.quantity,
                unitPrice: item.price,
                subtotal: item.price * item.quantity,
                total: item.price * item.quantity,
              ),
            );
          }
        });
        
        // Update stock in batch
        await db.into(db.products).batch((batch) {
          for (final item in orderData.items) {
            batch.update(
              db.products,
              ProductsCompanion(
                stock: Value(item.productStock - item.quantity),
                updatedAt: Value(DateTime.now()),
              ),
              (p) => p.id.equals(item.productId),
            );
          }
        });
      }
    });
    
    return orderIds;
  }
  
  // ==================== INVENTORY BATCHES ====================
  
  // 👇 Bulk stock adjustment
  Future<void> bulkAdjustStock(List<StockAdjustment> adjustments) async {
    await db.into(db.products).batch((batch) {
      for (final adj in adjustments) {
        final newStock = adj.currentStock + adj.adjustment;
        if (newStock < 0) {
          throw Exception('Insufficient stock for product ${adj.productId}');
        }
        
        batch.update(
          db.products,
          ProductsCompanion(
            stock: Value(newStock),
            updatedAt: Value(DateTime.now()),
          ),
          (p) => p.id.equals(adj.productId),
        );
      }
    });
  }
  
  // 👇 Bulk price update
  Future<void> bulkUpdatePrices(List<PriceUpdate> updates) async {
    await db.into(db.products).batch((batch) {
      for (final update in updates) {
        batch.update(
          db.products,
          ProductsCompanion(
            price: Value(update.newPrice),
            updatedAt: Value(DateTime.now()),
          ),
          (p) => p.id.equals(update.productId),
        );
      }
    });
  }
  
  // ==================== USER BATCHES ====================
  
  // 👇 Bulk user import from CSV
  Future<BatchResult> importUsersFromCSV(String csvContent) async {
    final lines = csvContent.split('\n');
    final headers = lines.first.split(',');
    
    final users = <UserData>[];
    int validCount = 0;
    int errorCount = 0;
    final errors = <String>[];
    
    for (var i = 1; i < lines.length; i++) {
      try {
        final values = lines[i].split(',');
        if (values.length < 2) continue;
        
        final user = UserData(
          name: values[0],
          email: values[1],
          age: values.length > 2 ? int.tryParse(values[2]) : null,
        );
        
        // Validate
        if (user.name.isEmpty) {
          throw Exception('Name required at row ${i + 1}');
        }
        if (!user.email.contains('@')) {
          throw Exception('Invalid email at row ${i + 1}');
        }
        
        users.add(user);
        validCount++;
      } catch (e) {
        errorCount++;
        errors.add('Row ${i + 1}: $e');
      }
    }
    
    // Batch insert valid users
    if (users.isNotEmpty) {
      await db.into(db.users).batch((batch) {
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
    }
    
    return BatchResult(
      inserted: validCount,
      errors: errorCount,
      errorMessages: errors,
    );
  }
  
  // ==================== REPORTING BATCHES ====================
  
  // 👇 Bulk status update
  Future<void> bulkUpdateOrderStatus(List<OrderStatusUpdate> updates) async {
    await db.into(db.orders).batch((batch) {
      for (final update in updates) {
        final companion = OrdersCompanion(
          status: Value(update.newStatus),
          shippedDate: update.newStatus == 'shipped' 
              ? Value(DateTime.now()) 
              : const Value.absent(),
          deliveredDate: update.newStatus == 'delivered'
              ? Value(DateTime.now())
              : const Value.absent(),
          isPaid: update.newStatus == 'paid' || update.newStatus == 'shipped'
              ? const Value(true)
              : const Value.absent(),
          isShipped: update.newStatus == 'shipped' || update.newStatus == 'delivered'
              ? const Value(true)
              : const Value.absent(),
          isDelivered: update.newStatus == 'delivered'
              ? const Value(true)
              : const Value.absent(),
        );
        
        batch.update(
          db.orders,
          companion,
          (o) => o.id.equals(update.orderId),
        );
      }
    });
  }
}

// ==================== DATA CLASSES ====================

class ProductData {
  final String sku;
  final String name;
  final double price;
  final String? description;
  final int stock;
  
  ProductData({
    required this.sku,
    required this.name,
    required this.price,
    this.description,
    this.stock = 0,
  });
}

class ProductUpdate {
  final int id;
  final String? name;
  final double? price;
  final int? stock;
  
  ProductUpdate({
    required this.id,
    this.name,
    this.price,
    this.stock,
  });
}

class OrderData {
  final int userId;
  final List<OrderItemData> items;
  final double total;
  final String shippingAddress;
  final String billingAddress;
  final String paymentMethod;
  
  OrderData({
    required this.userId,
    required this.items,
    required this.total,
    required this.shippingAddress,
    required this.billingAddress,
    required this.paymentMethod,
  });
}

class OrderItemData {
  final int productId;
  final int quantity;
  final double price;
  final int productStock;
  
  OrderItemData({
    required this.productId,
    required this.quantity,
    required this.price,
    required this.productStock,
  });
}

class StockAdjustment {
  final int productId;
  final int currentStock;
  final int adjustment;
  
  StockAdjustment({
    required this.productId,
    required this.currentStock,
    required this.adjustment,
  });
}

class PriceUpdate {
  final int productId;
  final double newPrice;
  
  PriceUpdate({
    required this.productId,
    required this.newPrice,
  });
}

class OrderStatusUpdate {
  final int orderId;
  final String newStatus;
  
  OrderStatusUpdate({
    required this.orderId,
    required this.newStatus,
  });
}

class UserData {
  final String name;
  final String email;
  final int? age;
  
  UserData({
    required this.name,
    required this.email,
    this.age,
  });
}

class BatchResult {
  final int inserted;
  final int errors;
  final List<String> errorMessages;
  
  BatchResult({
    required this.inserted,
    required this.errors,
    this.errorMessages = const [],
  });
}

// lib/database/database.dart - Adding batch methods
@DriftDatabase(tables: [Users, Products, Orders, OrderItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;
  
  // 👇 Expose batch operations
  BatchOperations get batchOps => BatchOperations(this);
}
```

```dart
// lib/ui/pages/import_page.dart
class ImportPage extends StatefulWidget {
  final AppDatabase db;
  
  const ImportPage({required this.db});
  
  @override
  _ImportPageState createState() => _ImportPageState();
}

class _ImportPageState extends State<ImportPage> {
  bool _isImporting = false;
  String _status = '';
  double _progress = 0;
  
  Future<void> _importCSV() async {
    setState(() {
      _isImporting = true;
      _status = 'Reading CSV...';
      _progress = 0;
    });
    
    try {
      // 👇 Simulate reading CSV
      final csvContent = await _readCSVFile();
      _progress = 0.3;
      _status = 'Importing users...';
      
      // 👇 Batch import
      final result = await widget.db.batchOps.importUsersFromCSV(csvContent);
      
      _progress = 1.0;
      _status = 'Import complete!';
      
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text(
            '✅ Inserted ${result.inserted} users, '
            '${result.errors} errors\n'
            '${result.errorMessages.join('\n')}'
          ),
          backgroundColor: result.errors > 0 ? Colors.orange : Colors.green,
          duration: Duration(seconds: 5),
        ),
      );
      
    } catch (e) {
      setState(() {
        _status = 'Error: $e';
        _isImporting = false;
      });
      
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('❌ Import failed: $e'),
          backgroundColor: Colors.red,
        ),
      );
    } finally {
      setState(() {
        _isImporting = false;
        _progress = 0;
      });
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Import Data')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            Text(_status, style: TextStyle(fontSize: 16)),
            if (_isImporting) ...[
              SizedBox(height: 16),
              LinearProgressIndicator(value: _progress),
              SizedBox(height: 8),
              Text('${(_progress * 100).round()}%'),
            ],
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: _isImporting ? null : _importCSV,
              child: Text('Import CSV'),
            ),
            if (_isImporting) ...[
              SizedBox(height: 8),
              CircularProgressIndicator(),
            ],
          ],
        ),
      ),
    );
  }
  
  Future<String> _readCSVFile() async {
    // Simulate reading CSV file
    await Future.delayed(Duration(seconds: 2));
    return '''
name,email,age
John Doe,john@example.com,30
Jane Smith,jane@example.com,25
Bob Wilson,bob@example.com,40
Alice Brown,alice@example.com,35
''';
  }
}
```

---

# Best Practices

- **Use batch for 10+ records** – Better performance
- **Chunk large datasets** – 1000-5000 records per batch
- **Validate before batch** – Catch errors early
- **Use transactions** – Atomic operations
- **Handle errors gracefully** – Skip invalid records
- **Report progress** – For long-running operations
- **Use appropriate chunk size** – Balance memory and performance
- **Test with different sizes** – Find optimal chunk size

---

# Common Mistakes

## Mistake 1: Too many records in one batch

Wrong:
```dart
// 🚫 100,000 records in one batch (memory issues)
await into(users).batch((batch) {
  for (final user in 100000_users) {
    batch.insert(user);
  }
});
```

Correct:
```dart
// ✅ Chunk into smaller batches
const chunkSize = 5000;
for (var i = 0; i < users.length; i += chunkSize) {
  final chunk = users.sublist(i, min(i + chunkSize, users.length));
  await into(users).batch((batch) {
    for (final user in chunk) {
      batch.insert(user);
    }
  });
}
```

## Mistake 2: Not validating before batch

Wrong:
```dart
// 🚫 Invalid data in batch (transaction fails)
await into(users).batch((batch) {
  for (final user in users) {
    batch.insert(user); // Invalid user data
  }
});
```

Correct:
```dart
// ✅ Validate all data first
for (final user in users) {
  if (user.name.isEmpty) throw Exception('Invalid user');
}
await into(users).batch((batch) {
  for (final user in users) {
    batch.insert(user);
  }
});
```

## Mistake 3: Using batch for single record

Wrong:
```dart
// 🚫 Overkill for one record
await into(users).batch((batch) {
  batch.insert(user);
});
```

Correct:
```dart
// ✅ Use regular insert
await into(users).insert(user);
```

---

# Summary

| Method | Use Case | Performance |
|--------|----------|-------------|
| **insert** | 1 record | Fast |
| **insertAll** | 2-100 records | Good |
| **batch** | 10-5000 records | Excellent |
| **chunked batch** | 5000+ records | Best |

---

# Next Steps

Now you understand batch insert, let's dive deeper:

- [Update](link) – Updating existing records
- [Delete](link) – Deleting records
- [Upsert](link) – Insert or update

---

# Did You Know?

- **Batch operations are 10-100x faster** – For bulk inserts

- **Batch uses single transaction** – All or nothing

- **Memory usage increases with batch size** – Balance performance

- **SQLite has no built-in batch** – Drift implements it

- **Batch supports mixed operations** – Insert, update, delete

- **Chunking prevents memory issues** – For very large datasets

- **Batch can return inserted data** – With returning

- **Batch operations are atomic** – Rollback on any error

---

