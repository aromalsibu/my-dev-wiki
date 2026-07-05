## Batch vs Transaction

**Choosing the right approach for bulk operations in Drift**

---

# What is it?

**Batch vs Transaction** is about understanding when to use batch operations versus transactions for multiple database operations. While both can group operations together, they serve different purposes and have different performance characteristics. Choosing the right approach is crucial for application performance and data integrity.

> **Think of Batch vs Transaction like "loading a truck vs organizing a warehouse"** – batching is about efficiency (loading many boxes at once), while transactions are about integrity (making sure everything is organized correctly).

```dart
// 👇 BATCH: Efficient bulk operations
await into(users).batch((batch) {
  for (final user in users) {
    batch.insert(user);
  }
});
// All inserts happen in one transaction
// But operations are independent

// 👇 TRANSACTION: Atomic operations with dependencies
await transaction(() async {
  // 1️⃣ Must succeed together
  await insertUser(user);
  await insertProfile(profile);
  await updateUserStats(userId);
});
// Operations are atomic and interdependent
```

> **What's happening here?**
> - **Batch** – Efficient, independent operations
> - **Transaction** – Atomic, dependent operations
> - **Performance** – Batch is faster for bulk inserts
> - **Integrity** – Transaction ensures consistency

---

# Why does it exist?

- **Performance** – Batch operations are faster
- **Integrity** – Transactions ensure consistency
- **Different Needs** – Different use cases require different approaches
- **Optimization** – Choose the right tool for the job
- **Resource Management** – Efficient use of resources
- **Developer Choice** – Flexibility for different scenarios

---

# Understanding Batch Operations

> **What batch operations are and when to use them**

## Batch Insert

```dart
// 👇 BATCH: Fast bulk insert
await into(users).batch((batch) {
  for (final user in users) {
    batch.insert(
      UsersCompanion.insert(
        name: user.name,
        email: user.email,
      ),
    );
  }
});

// Performance: Extremely fast for large datasets
// Atomic? Yes, all or nothing
// Dependencies? Operations are independent
// Use Case: Bulk data import, migration
```

## Batch Update

```dart
// 👇 BATCH: Update multiple records
await into(users).batch((batch) {
  for (final user in users) {
    batch.update(
      users,
      UsersCompanion(
        name: Value(user.name),
        status: Value('active'),
      ),
      (u) => u.id.equals(user.id),
    );
  }
});

// Performance: Fast for multiple updates
// Atomic? Yes, all or nothing
// Dependencies? Operations are independent
// Use Case: Bulk status updates, batch processing
```

## Batch Delete

```dart
// 👇 BATCH: Delete multiple records
await into(users).batch((batch) {
  for (final id in userIds) {
    batch.delete(
      users,
      (u) => u.id.equals(id),
    );
  }
});

// Performance: Fast for multiple deletes
// Atomic? Yes, all or nothing
// Dependencies? Operations are independent
// Use Case: Bulk cleanup, data removal
```

---

# Understanding Transactions

> **What transactions are and when to use them**

## Transaction with Dependencies

```dart
// 👇 TRANSACTION: Operations depend on each other
await transaction(() async {
  // 1️⃣ Must succeed
  final userId = await insertUser(userData);
  
  // 2️⃣ Depends on 1️⃣
  await insertProfile(userId, profileData);
  
  // 3️⃣ Depends on 1️⃣ and 2️⃣
  await updateUserStatus(userId, 'active');
  
  // If any fails, all rollback
});

// Performance: Good, but overhead
// Atomic? Yes, all or nothing
// Dependencies? Operations are dependent
// Use Case: Complex business logic, data consistency
```

## Transaction with Conditions

```dart
// 👇 TRANSACTION: Conditional operations
await transaction(() async {
  // 1️⃣ Check balance
  final balance = await getBalance(userId);
  
  if (balance < amount) {
    throw Exception('Insufficient balance');
  }
  
  // 2️⃣ Deduct
  await updateBalance(userId, balance - amount);
  
  // 3️⃣ Add to recipient
  await updateBalance(recipientId, recipientBalance + amount);
  
  // 4️⃣ Record transaction
  await insertTransaction(fromId, toId, amount);
});

// Performance: Good for complex logic
// Atomic? Yes, all or nothing
// Dependencies? Operations are dependent
// Use Case: Financial operations, complex validations
```

---

# Batch vs Transaction Comparison

> **When to use each approach**

| Feature | Batch | Transaction |
|---------|-------|-------------|
| **Performance** | Extremely fast | Good |
| **Atomic** | Yes | Yes |
| **Dependencies** | Independent | Dependent |
| **Complex Logic** | Limited | Full support |
| **Error Handling** | All or nothing | All or nothing |
| **Use Case** | Bulk data | Complex logic |
| **Resources** | Lower overhead | Higher overhead |
| **Return Values** | Limited | Full access |

---

# Real-World Example

> **Complete comparison system**

```dart
// lib/database/batch_vs_transaction_service.dart
import 'package:drift/drift.dart';

class BatchVsTransactionService {
  final AppDatabase db;
  
  BatchVsTransactionService(this.db);

  // ==================== BATCH APPROACH ====================
  
  // 👇 Batch approach: Bulk user import
  Future<ImportResult> batchImportUsers(List<UserData> users) async {
    var imported = 0;
    final errors = <String>[];
    
    try {
      await db.into(db.users).batch((batch) {
        for (final user in users) {
          try {
            batch.insert(
              UsersCompanion.insert(
                name: user.name,
                email: user.email,
                age: Value(user.age),
              ),
            );
            imported++;
          } catch (e) {
            errors.add('User ${user.email}: $e');
          }
        }
      });
    } catch (e) {
      // Batch failed entirely
      return ImportResult(
        imported: 0,
        errors: ['Batch import failed: $e'],
      );
    }
    
    return ImportResult(
      imported: imported,
      errors: errors,
    );
  }
  
  // 👇 Batch approach: Bulk status update
  Future<int> batchUpdateStatus(
    List<int> userIds,
    String newStatus,
  ) async {
    var updated = 0;
    
    await db.into(db.users).batch((batch) {
      for (final id in userIds) {
        batch.update(
          db.users,
          UsersCompanion(
            status: Value(newStatus),
            updatedAt: Value(DateTime.now()),
          ),
          (u) => u.id.equals(id),
        );
        updated++;
      }
    });
    
    return updated;
  }

  // ==================== TRANSACTION APPROACH ====================
  
  // 👇 Transaction approach: Complex user import
  Future<TransactionImportResult> transactionImportUsers(
    List<UserData> users,
  ) async {
    final imported = <User>[];
    final errors = <String>[];
    
    try {
      await db.transaction(() async {
        for (final user in users) {
          try {
            // Validate
            if (user.email.isEmpty) {
              throw Exception('Email required for ${user.name}');
            }
            
            // Check for duplicates
            final existing = await (db.select(db.users)
              ..where((u) => u.email.equals(user.email)))
              .getSingleOrNull();
            
            if (existing != null) {
              // Update existing user
              await (db.update(db.users)..where((u) => u.id.equals(existing.id)))
                .write(UsersCompanion(
                  name: Value(user.name),
                  age: Value(user.age),
                  updatedAt: Value(DateTime.now()),
                ));
              
              imported.add(existing);
            } else {
              // Insert new user
              final id = await db.into(db.users).insert(
                UsersCompanion.insert(
                  name: user.name,
                  email: user.email,
                  age: Value(user.age),
                ),
              );
              
              final newUser = await (db.select(db.users)
                ..where((u) => u.id.equals(id)))
                .getSingle();
              
              imported.add(newUser);
            }
            
          } catch (e) {
            errors.add('User ${user.email}: $e');
            // Continue with next user, transaction continues
          }
        }
        
        // If all users failed, rollback
        if (imported.isEmpty && errors.isNotEmpty) {
          throw Exception('All users failed to import');
        }
      });
      
    } catch (e) {
      // Transaction rolled back
      return TransactionImportResult(
        imported: [],
        errors: ['Transaction failed: $e'],
      );
    }
    
    return TransactionImportResult(
      imported: imported,
      errors: errors,
    );
  }
  
  // 👇 Transaction approach: Money transfer
  Future<TransferResult> transactionTransferMoney({
    required int fromUserId,
    required int toUserId,
    required double amount,
  }) async {
    try {
      await db.transaction(() async {
        // 1️⃣ Get balances
        final fromUser = await (db.select(db.users)
          ..where((u) => u.id.equals(fromUserId)))
          .getSingle();
        
        final toUser = await (db.select(db.users)
          ..where((u) => u.id.equals(toUserId)))
          .getSingle();
        
        // 2️⃣ Check balance
        if (fromUser.balance < amount) {
          throw Exception('Insufficient balance');
        }
        
        // 3️⃣ Deduct
        await (db.update(db.users)..where((u) => u.id.equals(fromUserId)))
          .write(UsersCompanion(
            balance: Value(fromUser.balance - amount),
            updatedAt: Value(DateTime.now()),
          ));
        
        // 4️⃣ Add
        await (db.update(db.users)..where((u) => u.id.equals(toUserId)))
          .write(UsersCompanion(
            balance: Value(toUser.balance + amount),
            updatedAt: Value(DateTime.now()),
          ));
        
        // 5️⃣ Record
        await db.into(db.transactions).insert(
          TransactionsCompanion.insert(
            fromUserId: fromUserId,
            toUserId: toUserId,
            amount: amount,
            timestamp: DateTime.now(),
          ),
        );
      });
      
      return TransferResult.success(
        fromUserId: fromUserId,
        toUserId: toUserId,
        amount: amount,
      );
      
    } catch (e) {
      return TransferResult.failure(e.toString());
    }
  }

  // ==================== PERFORMANCE COMPARISON ====================
  
  // 👇 Compare batch vs transaction performance
  Future<ComparisonResult> comparePerformance(int recordCount) async {
    // 1️⃣ Generate test data
    final users = List.generate(recordCount, (i) {
      return UserData(
        name: 'User $i',
        email: 'user$i@example.com',
        age: 20 + (i % 30),
      );
    });
    
    // 2️⃣ Time batch import
    final batchStart = DateTime.now();
    try {
      await batchImportUsers(users);
    } catch (e) {
      // Ignore errors for comparison
    }
    final batchDuration = DateTime.now().difference(batchStart);
    
    // 3️⃣ Time transaction import
    final txStart = DateTime.now();
    try {
      await transactionImportUsers(users);
    } catch (e) {
      // Ignore errors for comparison
    }
    final txDuration = DateTime.now().difference(txStart);
    
    return ComparisonResult(
      recordCount: recordCount,
      batchDuration: batchDuration,
      transactionDuration: txDuration,
      ratio: batchDuration.inMilliseconds / txDuration.inMilliseconds,
    );
  }

  // ==================== HYBRID APPROACH ====================
  
  // 👇 Combine batch and transaction for best results
  Future<HybridResult> hybridImport(List<UserData> users) async {
    var inserted = 0;
    var updated = 0;
    final errors = <String>[];
    
    await db.transaction(() async {
      // 1️⃣ Validate all users
      final validUsers = <UserData>[];
      for (final user in users) {
        if (user.email.isEmpty) {
          errors.add('User ${user.name}: Email required');
          continue;
        }
        validUsers.add(user);
      }
      
      if (validUsers.isEmpty) {
        throw Exception('No valid users to import');
      }
      
      // 2️⃣ Batch insert new users
      final newUsers = validUsers.where((u) => !u.exists).toList();
      if (newUsers.isNotEmpty) {
        await db.into(db.users).batch((batch) {
          for (final user in newUsers) {
            batch.insert(
              UsersCompanion.insert(
                name: user.name,
                email: user.email,
                age: Value(user.age),
              ),
            );
            inserted++;
          }
        });
      }
      
      // 3️⃣ Batch update existing users
      final existingUsers = validUsers.where((u) => u.exists).toList();
      if (existingUsers.isNotEmpty) {
        await db.into(db.users).batch((batch) {
          for (final user in existingUsers) {
            batch.update(
              db.users,
              UsersCompanion(
                name: Value(user.name),
                age: Value(user.age),
                updatedAt: Value(DateTime.now()),
              ),
              (u) => u.email.equals(user.email),
            );
            updated++;
          }
        });
      }
    });
    
    return HybridResult(
      inserted: inserted,
      updated: updated,
      errors: errors,
    );
  }
}

// ==================== DATA CLASSES ====================

class UserData {
  final String name;
  final String email;
  final int? age;
  final bool exists;
  
  UserData({
    required this.name,
    required this.email,
    this.age,
    this.exists = false,
  });
}

class ImportResult {
  final int imported;
  final List<String> errors;
  
  ImportResult({
    required this.imported,
    this.errors = const [],
  });
}

class TransactionImportResult {
  final List<User> imported;
  final List<String> errors;
  
  TransactionImportResult({
    required this.imported,
    this.errors = const [],
  });
}

class TransferResult {
  final bool success;
  final int? fromUserId;
  final int? toUserId;
  final double? amount;
  final String? error;
  
  TransferResult._({
    required this.success,
    this.fromUserId,
    this.toUserId,
    this.amount,
    this.error,
  });
  
  factory TransferResult.success({
    required int fromUserId,
    required int toUserId,
    required double amount,
  }) {
    return TransferResult._(
      success: true,
      fromUserId: fromUserId,
      toUserId: toUserId,
      amount: amount,
    );
  }
  
  factory TransferResult.failure(String error) {
    return TransferResult._(
      success: false,
      error: error,
    );
  }
}

class ComparisonResult {
  final int recordCount;
  final Duration batchDuration;
  final Duration transactionDuration;
  final double ratio;
  
  ComparisonResult({
    required this.recordCount,
    required this.batchDuration,
    required this.transactionDuration,
    required this.ratio,
  });
}

class HybridResult {
  final int inserted;
  final int updated;
  final List<String> errors;
  
  HybridResult({
    required this.inserted,
    required this.updated,
    this.errors = const [],
  });
}
```

```dart
// lib/ui/pages/comparison_page.dart
class ComparisonPage extends StatefulWidget {
  final BatchVsTransactionService service;
  
  const ComparisonPage({required this.service});
  
  @override
  _ComparisonPageState createState() => _ComparisonPageState();
}

class _ComparisonPageState extends State<ComparisonPage> {
  ComparisonResult? _result;
  bool _isRunning = false;
  
  Future<void> _runComparison() async {
    setState(() => _isRunning = true);
    
    try {
      final result = await widget.service.comparePerformance(1000);
      setState(() {
        _result = result;
        _isRunning = false;
      });
    } catch (e) {
      setState(() => _isRunning = false);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Batch vs Transaction Comparison'),
        backgroundColor: Colors.cyan[800],
      ),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            Card(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: Column(
                  children: [
                    Text(
                      'Performance Comparison',
                      style: TextStyle(
                        fontSize: 20,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    SizedBox(height: 8),
                    Text(
                      'Import 1,000 users',
                      style: TextStyle(color: Colors.grey),
                    ),
                    SizedBox(height: 16),
                    if (_result != null) ...[
                      _buildComparisonRow(
                        'Batch',
                        '${_result!.batchDuration.inMilliseconds}ms',
                        Colors.green,
                      ),
                      _buildComparisonRow(
                        'Transaction',
                        '${_result!.transactionDuration.inMilliseconds}ms',
                        Colors.orange,
                      ),
                      SizedBox(height: 8),
                      Text(
                        'Batch is ${_result!.ratio.toStringAsFixed(1)}x faster',
                        style: TextStyle(
                          fontSize: 16,
                          fontWeight: FontWeight.bold,
                          color: Colors.blue,
                        ),
                      ),
                    ],
                    SizedBox(height: 16),
                    ElevatedButton(
                      onPressed: _isRunning ? null : _runComparison,
                      style: ElevatedButton.styleFrom(
                        minimumSize: Size(double.infinity, 50),
                      ),
                      child: _isRunning
                          ? CircularProgressIndicator()
                          : Text('Run Comparison'),
                    ),
                  ],
                ),
              ),
            ),
            SizedBox(height: 16),
            _buildUseCases(),
          ],
        ),
      ),
    );
  }
  
  Widget _buildComparisonRow(String label, String value, Color color) {
    return Padding(
      padding: EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(label),
          Container(
            padding: EdgeInsets.symmetric(horizontal: 12, vertical: 4),
            decoration: BoxDecoration(
              color: color.withOpacity(0.2),
              borderRadius: BorderRadius.circular(8),
            ),
            child: Text(
              value,
              style: TextStyle(
                fontWeight: FontWeight.bold,
                color: color,
              ),
            ),
          ),
        ],
      ),
    );
  }
  
  Widget _buildUseCases() {
    return Expanded(
      child: Card(
        child: Padding(
          padding: EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                'When to Use Each:',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
              SizedBox(height: 8),
              ListTile(
                leading: Icon(Icons.speed, color: Colors.green),
                title: Text('Batch Operations'),
                subtitle: Text('Bulk inserts, updates, deletes'),
              ),
              ListTile(
                leading: Icon(Icons.gavel, color: Colors.orange),
                title: Text('Transactions'),
                subtitle: Text('Complex logic, dependencies, validations'),
              ),
              ListTile(
                leading: Icon(Icons.hybrid, color: Colors.blue),
                title: Text('Hybrid Approach'),
                subtitle: Text('Batch inside transaction for best results'),
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

- **Use batch for independent operations** – Bulk inserts, updates, deletes
- **Use transaction for dependent operations** – Complex business logic
- **Use hybrid approach** – Batch inside transaction when needed
- **Test both approaches** – Verify performance and integrity
- **Consider error handling** – All or nothing vs partial success
- **Monitor performance** – Choose the faster approach
- **Document your choice** – Explain why you chose one over the other

---

# Decision Guide

| Scenario | Recommendation | Reason |
|----------|---------------|---------|
| **Bulk Insert** | Batch | Much faster |
| **Bulk Update** | Batch | Much faster |
| **Bulk Delete** | Batch | Much faster |
| **Related Operations** | Transaction | Must succeed together |
| **Complex Validations** | Transaction | Need full control |
| **Mixed Operations** | Hybrid | Best of both |
| **Performance Critical** | Batch | Faster |
| **Integrity Critical** | Transaction | Safer |

---

# Common Mistakes

## Mistake 1: Using transaction for bulk operations

Wrong:
```dart
// 🚫 Slow for bulk operations
await transaction(() async {
  for (final user in users) {
    await db.into(db.users).insert(user);
  }
});
```

Correct:
```dart
// ✅ Use batch for bulk operations
await db.into(db.users).batch((batch) {
  for (final user in users) {
    batch.insert(user);
  }
});
```

## Mistake 2: Using batch for dependent operations

Wrong:
```dart
// 🚫 Can't handle dependencies
await db.into(db.orders).batch((batch) {
  batch.insert(order);
  // Can't use order ID in batch
});
```

Correct:
```dart
// ✅ Use transaction for dependencies
await transaction(() async {
  final orderId = await db.into(db.orders).insert(order);
  await db.into(db.orderItems).insertAll(items);
});
```

## Mistake 3: Not choosing the right tool

Wrong:
```dart
// 🚫 Using transaction for simple bulk operations
await transaction(() async {
  await db.into(db.users).insertAll(users);
});
```

Correct:
```dart
// ✅ Use batch for simpler, faster operations
await db.into(db.users).insertAll(users);
```

---

# Summary

| Approach | Best For | Performance |
|----------|----------|-------------|
| **Batch** | Bulk operations | Very fast |
| **Transaction** | Complex logic | Good |
| **Hybrid** | Both needs | Excellent |

---

# Next Steps

Now you understand batch vs transaction, let's dive deeper:

- [Rollback Behavior](link) – Understanding rollbacks
- [Custom SQL](link) – Advanced SQL features
- [Views](link) – Database views

---

# Did You Know?

- **Batch operations are 10-100x faster** – For bulk operations

- **Transactions are essential for integrity** – Complex operations

- **Hybrid approaches exist** – Batch inside transaction

- **Batch operations are still atomic** – All or nothing

- **Transactions have more overhead** – But more control

- **Choosing the right approach matters** – For performance

- **You can mix both approaches** – For optimal results

- **Test both approaches** – Verify which is faster

---
