## Testing Migrations

**Ensuring migration safety with comprehensive testing in Drift**

---

# What is it?

**Testing Migrations** is the practice of verifying that your database migrations work correctly before deploying them to production. This involves testing both the upgrade path (moving from one version to another) and the downgrade path (rolling back to a previous version). Drift's in-memory database support makes it easy to test migrations in isolation.

> **Think of Testing Migrations like "test driving a new route"** – before taking the highway with your family, you drive it alone first to check for construction, traffic patterns, and potential issues. Similarly, you test migrations before running them on production data.

```dart
// 👇 Testing a migration
void testMigration() {
  // Create an in-memory database at version 1
  final db = AppDatabase(NativeDatabase.memory());
  
  // Run the migration
  await db.migrateTo(2);
  
  // Verify the migration worked
  final hasColumn = await db.customSelect(
    'PRAGMA table_info(users)'
  ).get();
  
  // Assert the new column exists
  expect(hasColumn.any((c) => c.data['name'] == 'status'), true);
}
```

> **What's happening here?**
> - **In-memory database** – Fast and isolated testing
> - **Version targeting** – Test specific migrations
> - **Verification** – Check schema and data integrity
> - **Isolation** – No impact on real data

---

# Why does it exist?

- **Safety** – Catch errors before production
- **Confidence** – Ensure migrations work
- **Data Integrity** – Verify data preservation
- **Performance** – Check migration speed
- **Rollback** – Test downgrade path
- **Automation** – CI/CD integration

---

# Basic Migration Testing

> **Simple migration tests**

## Testing a Single Migration

```dart
// test/migrations/migration_001_test.dart
import 'package:test/test.dart';
import 'package:drift/native.dart';
import '../lib/database/database.dart';

void main() {
  test('Migration 1: Add status column', () async {
    // 1️⃣ Create database at version 1
    final db = AppDatabase(NativeDatabase.memory());
    await db.ensureOpen();
    
    // 2️⃣ Verify schema before migration
    final beforeColumns = await db.customSelect(
      'PRAGMA table_info(users)'
    ).get();
    expect(beforeColumns.length, 3);
    
    // 3️⃣ Run migration to version 2
    await db.migrateTo(2);
    
    // 4️⃣ Verify schema after migration
    final afterColumns = await db.customSelect(
      'PRAGMA table_info(users)'
    ).get();
    expect(afterColumns.length, 4);
    
    // 5️⃣ Verify data integrity
    final hasStatus = afterColumns.any((c) => c.data['name'] == 'status');
    expect(hasStatus, true);
    
    // 6️⃣ Verify data migration
    final rows = await db.select(db.users).get();
    for (final row in rows) {
      expect(row.status, 'active'); // Default value set
    }
    
    await db.close();
  });
}
```

---

## Testing Multiple Migrations

```dart
// test/migrations/full_migration_test.dart
void main() {
  test('Full migration path: v1 -> v5', () async {
    final db = AppDatabase(NativeDatabase.memory());
    await db.ensureOpen();
    
    // Start at version 1
    expect(db.schemaVersion, 1);
    
    // Migrate to version 5
    await db.migrateTo(5);
    expect(db.schemaVersion, 5);
    
    // Verify all tables exist
    final tables = await db.customSelect(
      'SELECT name FROM sqlite_master WHERE type = "table"'
    ).get();
    final tableNames = tables.map((t) => t.data['name'] as String).toList();
    
    expect(tableNames.contains('users'), true);
    expect(tableNames.contains('orders'), true);
    expect(tableNames.contains('profiles'), true);
    expect(tableNames.contains('reviews'), true);
    
    // Verify all columns exist
    final columns = await db.customSelect(
      'PRAGMA table_info(users)'
    ).get();
    final columnNames = columns.map((c) => c.data['name'] as String).toList();
    
    expect(columnNames.contains('status'), true);
    expect(columnNames.contains('last_login'), true);
    expect(columnNames.contains('is_deleted'), true);
    
    await db.close();
  });
}
```

---

# Data Integrity Testing

> **Verifying data survives migrations**

```dart
// test/migrations/data_integrity_test.dart
void main() {
  test('Data integrity: User data preserved', () async {
    final db = AppDatabase(NativeDatabase.memory());
    await db.ensureOpen();
    
    // 1️⃣ Insert test data at version 1
    await db.into(db.users).insert(
      UsersCompanion.insert(
        name: 'John Doe',
        email: 'john@example.com',
      ),
    );
    
    final userId = await db.into(db.users).insert(
      UsersCompanion.insert(
        name: 'Jane Doe',
        email: 'jane@example.com',
      ),
    );
    
    // 2️⃣ Verify data exists
    var users = await db.select(db.users).get();
    expect(users.length, 2);
    
    // 3️⃣ Migrate to version 2
    await db.migrateTo(2);
    
    // 4️⃣ Verify data still exists
    users = await db.select(db.users).get();
    expect(users.length, 2);
    
    // 5️⃣ Verify specific user data
    final jane = users.firstWhere((u) => u.id == userId);
    expect(jane.name, 'Jane Doe');
    expect(jane.email, 'jane@example.com');
    
    // 6️⃣ Verify new column data
    expect(jane.status, 'active'); // Default value set
    
    await db.close();
  });
}
```

---

# Advanced Migration Testing

> **Complex test scenarios**

## Testing Rollbacks

```dart
// test/migrations/rollback_test.dart
void main() {
  test('Rollback: v2 -> v1', () async {
    final db = AppDatabase(NativeDatabase.memory());
    await db.ensureOpen();
    
    // 1️⃣ Migrate to version 2
    await db.migrateTo(2);
    expect(db.schemaVersion, 2);
    
    // 2️⃣ Insert data using new schema
    await db.into(db.users).insert(
      UsersCompanion.insert(
        name: 'Test User',
        email: 'test@example.com',
        status: Value('active'),
      ),
    );
    
    // 3️⃣ Rollback to version 1
    await db.migrateTo(1);
    expect(db.schemaVersion, 1);
    
    // 4️⃣ Verify schema reverted
    final columns = await db.customSelect(
      'PRAGMA table_info(users)'
    ).get();
    final columnNames = columns.map((c) => c.data['name'] as String).toList();
    expect(columnNames.contains('status'), false);
    
    // 5️⃣ Verify data preserved (status column dropped)
    final users = await db.select(db.users).get();
    expect(users.length, 1);
    
    await db.close();
  });
}
```

---

## Testing Destructive Migrations

```dart
// test/migrations/destructive_test.dart
void main() {
  test('Destructive migration: Drop column', () async {
    final db = AppDatabase(NativeDatabase.memory());
    await db.ensureOpen();
    
    // 1️⃣ Migrate to version with column to drop
    await db.migrateTo(2);
    
    // 2️⃣ Insert data
    await db.into(db.users).insert(
      UsersCompanion.insert(
        name: 'Test User',
        email: 'test@example.com',
        status: Value('active'),
      ),
    );
    
    // 3️⃣ Verify column exists
    var columns = await db.customSelect(
      'PRAGMA table_info(users)'
    ).get();
    expect(columns.any((c) => c.data['name'] == 'status'), true);
    
    // 4️⃣ Migrate to version 3 (drops status)
    await db.migrateTo(3);
    
    // 5️⃣ Verify column dropped
    columns = await db.customSelect(
      'PRAGMA table_info(users)'
    ).get();
    expect(columns.any((c) => c.data['name'] == 'status'), false);
    
    // 6️⃣ Verify data preserved (without dropped column)
    final users = await db.select(db.users).get();
    expect(users.length, 1);
    expect(users.first.name, 'Test User');
    expect(users.first.email, 'test@example.com');
    
    await db.close();
  });
}
```

---

# Performance Testing

> **Checking migration speed**

```dart
// test/migrations/performance_test.dart
void main() {
  test('Migration performance: v1 -> v5 with 10k users', () async {
    final db = AppDatabase(NativeDatabase.memory());
    await db.ensureOpen();
    
    // 1️⃣ Insert 10,000 users
    print('Inserting 10,000 users...');
    final insertStart = DateTime.now();
    
    for (var i = 0; i < 10000; i++) {
      await db.into(db.users).insert(
        UsersCompanion.insert(
          name: 'User $i',
          email: 'user$i@example.com',
        ),
      );
    }
    
    final insertTime = DateTime.now().difference(insertStart);
    print('✅ Inserted 10,000 users in ${insertTime.inSeconds}s');
    
    // 2️⃣ Run migration
    print('Running migration v1 -> v5...');
    final migrationStart = DateTime.now();
    
    await db.migrateTo(5);
    
    final migrationTime = DateTime.now().difference(migrationStart);
    print('✅ Migration completed in ${migrationTime.inSeconds}s');
    
    // 3️⃣ Validate results
    final users = await db.select(db.users).get();
    expect(users.length, 10000);
    
    // 4️⃣ Check performance constraints
    expect(migrationTime.inSeconds, lessThan(10));
    
    await db.close();
  });
}
```

---

# Real-World Example

> **Complete e-commerce migration testing system**

```dart
// test/migrations/migration_test_suite.dart
import 'package:test/test.dart';
import 'package:drift/native.dart';
import '../../lib/database/database.dart';
import '../../lib/database/migrations/migration_service.dart';

void main() {
  group('Migration Tests', () {
    late AppDatabase db;
    
    setUp(() async {
      // Create fresh in-memory database for each test
      db = AppDatabase(NativeDatabase.memory());
      await db.ensureOpen();
    });
    
    tearDown(() async {
      await db.close();
    });

    // ==================== VERSION TESTS ====================
    
    test('Initial version is 1', () async {
      expect(db.schemaVersion, 1);
    });

    test('Can migrate to each version', () async {
      final versions = [1, 2, 3, 4, 5];
      
      for (final version in versions) {
        await db.migrateTo(version);
        expect(db.schemaVersion, version);
        
        // Verify schema for each version
        await _verifySchema(db, version);
      }
    });

    // ==================== SCHEMA TESTS ====================
    
    test('v1 schema: Users table only', () async {
      await db.migrateTo(1);
      await _verifyV1Schema(db);
    });

    test('v2 schema: Users with status column', () async {
      await db.migrateTo(2);
      await _verifyV2Schema(db);
    });

    test('v3 schema: Orders and order items', () async {
      await db.migrateTo(3);
      await _verifyV3Schema(db);
    });

    test('v4 schema: Reviews table', () async {
      await db.migrateTo(4);
      await _verifyV4Schema(db);
    });

    test('v5 schema: All indexes', () async {
      await db.migrateTo(5);
      await _verifyV5Schema(db);
    });

    // ==================== DATA TESTS ====================
    
    test('Data preserved across migrations', () async {
      // 1️⃣ Insert data at v1
      await db.migrateTo(1);
      await db.into(db.users).insert(
        UsersCompanion.insert(
          name: 'Test User',
          email: 'test@example.com',
        ),
      );
      
      // 2️⃣ Migrate through all versions
      await db.migrateTo(5);
      
      // 3️⃣ Verify data
      final users = await db.select(db.users).get();
      expect(users.length, 1);
      expect(users.first.name, 'Test User');
      expect(users.first.email, 'test@example.com');
      
      // 4️⃣ Verify new fields have defaults
      expect(users.first.status, 'active');
    });

    test('Data migrated correctly: v1 -> v2', () async {
      await db.migrateTo(1);
      
      // Insert users before migration
      await db.into(db.users).insertAll([
        UsersCompanion.insert(name: 'User 1', email: 'user1@example.com'),
        UsersCompanion.insert(name: 'User 2', email: 'user2@example.com'),
      ]);
      
      // Migrate to v2
      await db.migrateTo(2);
      
      // Verify status column set
      final users = await db.select(db.users).get();
      for (final user in users) {
        expect(user.status, 'active');
      }
    });

    // ==================== PERFORMANCE TESTS ====================
    
    test('Migration performance: v1 -> v5 with 1000 users', () async {
      await db.migrateTo(1);
      
      // Insert 1000 users
      for (var i = 0; i < 1000; i++) {
        await db.into(db.users).insert(
          UsersCompanion.insert(
            name: 'User $i',
            email: 'user$i@example.com',
          ),
        );
      }
      
      final start = DateTime.now();
      await db.migrateTo(5);
      final duration = DateTime.now().difference(start);
      
      print('Migration time: ${duration.inMilliseconds}ms');
      expect(duration.inSeconds, lessThan(5));
    });

    // ==================== ROLLBACK TESTS ====================
    
    test('Rollback from v2 to v1', () async {
      await db.migrateTo(2);
      
      // Insert user with status
      await db.into(db.users).insert(
        UsersCompanion.insert(
          name: 'Test User',
          email: 'test@example.com',
          status: Value('active'),
        ),
      );
      
      // Rollback to v1
      await db.migrateTo(1);
      
      // Verify status column removed
      final columns = await db.customSelect(
        'PRAGMA table_info(users)'
      ).get();
      expect(columns.any((c) => c.data['name'] == 'status'), false);
      
      // Verify data preserved
      final users = await db.select(db.users).get();
      expect(users.length, 1);
      expect(users.first.name, 'Test User');
    });

    // ==================== INTEGRITY TESTS ====================
    
    test('Foreign key integrity preserved', () async {
      await db.migrateTo(5);
      
      // Create user
      final userId = await db.into(db.users).insert(
        UsersCompanion.insert(
          name: 'Test User',
          email: 'test@example.com',
        ),
      );
      
      // Create order
      final orderId = await db.into(db.orders).insert(
        OrdersCompanion.insert(
          orderNumber: 'ORD-001',
          userId: userId,
          total: 100.0,
          status: 'pending',
        ),
      );
      
      // Create order items
      await db.into(db.orderItems).insert(
        OrderItemsCompanion.insert(
          orderId: orderId,
          productId: 1,
          quantity: 1,
          unitPrice: 100.0,
        ),
      );
      
      // Verify relationships
      final orders = await db.select(db.orders)
        .where((o) => o.userId.equals(userId))
        .get();
      expect(orders.length, 1);
      
      final items = await db.select(db.orderItems)
        .where((i) => i.orderId.equals(orderId))
        .get();
      expect(items.length, 1);
    });

    // ==================== ERROR HANDLING TESTS ====================
    
    test('Invalid migration throws error', () async {
      // Attempt to migrate to non-existent version
      expect(
        () => db.migrateTo(99),
        throwsException,
      );
    });
  });
}

// ==================== VERIFICATION HELPERS ====================

Future<void> _verifyV1Schema(AppDatabase db) async {
  final tables = await db.customSelect(
    'SELECT name FROM sqlite_master WHERE type = "table"'
  ).get();
  final tableNames = tables.map((t) => t.data['name'] as String).toList();
  expect(tableNames.contains('users'), true);
  expect(tableNames.contains('orders'), false);
}

Future<void> _verifyV2Schema(AppDatabase db) async {
  final columns = await db.customSelect(
    'PRAGMA table_info(users)'
  ).get();
  final columnNames = columns.map((c) => c.data['name'] as String).toList();
  expect(columnNames.contains('status'), true);
  expect(columnNames.contains('last_login'), false);
}

Future<void> _verifyV3Schema(AppDatabase db) async {
  final tables = await db.customSelect(
    'SELECT name FROM sqlite_master WHERE type = "table"'
  ).get();
  final tableNames = tables.map((t) => t.data['name'] as String).toList();
  expect(tableNames.contains('orders'), true);
  expect(tableNames.contains('order_items'), true);
}

Future<void> _verifyV4Schema(AppDatabase db) async {
  final tables = await db.customSelect(
    'SELECT name FROM sqlite_master WHERE type = "table"'
  ).get();
  final tableNames = tables.map((t) => t.data['name'] as String).toList();
  expect(tableNames.contains('reviews'), true);
}

Future<void> _verifyV5Schema(AppDatabase db) async {
  final indexes = await db.customSelect(
    "SELECT name FROM sqlite_master WHERE type = 'index'"
  ).get();
  final indexNames = indexes.map((i) => i.data['name'] as String).toList();
  expect(indexNames.any((i) => i.contains('idx_users_email')), true);
  expect(indexNames.any((i) => i.contains('idx_orders_user')), true);
}

Future<void> _verifySchema(AppDatabase db, int version) async {
  switch (version) {
    case 1:
      await _verifyV1Schema(db);
      break;
    case 2:
      await _verifyV2Schema(db);
      break;
    case 3:
      await _verifyV3Schema(db);
      break;
    case 4:
      await _verifyV4Schema(db);
      break;
    case 5:
      await _verifyV5Schema(db);
      break;
  }
}
```

---

# Migration Testing Best Practices

- **Test each migration** – Verify each step individually
- **Test full migration path** – From start to end
- **Test with data** – Verify data integrity
- **Test rollbacks** – Ensure downgrades work
- **Test performance** – Check migration speed
- **Test with realistic data** – Use production-like data
- **Test destructive operations** – Handle careful testing
- **Automate tests** – CI/CD integration

---

# Migration Testing Checklist

| Test Type | Purpose | Priority |
|-----------|---------|----------|
| **Schema Verification** | Check structure | High |
| **Data Preservation** | No data loss | High |
| **Data Migration** | Correct data | High |
| **Rollback** | Undo changes | High |
| **Performance** | Speed check | Medium |
| **Integrity** | Constraints | Medium |
| **Error Handling** | Graceful failure | Medium |
| **CI/CD** | Automated tests | Medium |

---

# Common Mistakes

## Mistake 1: Not testing with data

Wrong:
```dart
// 🚫 Empty database migration test
test('Migration works', () async {
  await db.migrateTo(2);
  // Only schema checked, no data
});
```

Correct:
```dart
// ✅ Test with data
test('Migration works with data', () async {
  await insertTestData(db);
  await db.migrateTo(2);
  await verifyDataIntegrity(db);
});
```

## Mistake 2: Not testing rollbacks

Wrong:
```dart
// 🚫 Only testing upgrade
test('Migration works', () async {
  await db.migrateTo(5);
});
```

Correct:
```dart
// ✅ Test both upgrade and downgrade
test('Migration and rollback work', () async {
  await db.migrateTo(5);
  await db.migrateTo(3);
  await verifySchemaVersion(db, 3);
});
```

## Mistake 3: Not testing with realistic data

Wrong:
```dart
// 🚫 Only 1-2 test records
for (var i = 0; i < 2; i++) {
  await insertUser(db);
}
```

Correct:
```dart
// ✅ Test with realistic data volume
for (var i = 0; i < 1000; i++) {
  await insertRealisticUser(db);
}
```

---

# Summary

| Test Type | Purpose | Example |
|-----------|---------|---------|
| **Schema** | Verify structure | Table/column existence |
| **Data** | Preserve data | Count/values check |
| **Rollback** | Undo changes | Version verification |
| **Performance** | Speed check | Migration timing |
| **Integrity** | Constraints | Foreign keys |
| **Error** | Graceful failure | Invalid migration |

---

# Next Steps

Now you understand testing migrations, let's dive deeper:

- [Best Practices](link) – Migration best practices
- [Type Converters](link) – Custom type converters
- [DAO](link) – Data Access Objects

---

# Did You Know?

- **In-memory databases are perfect** – For fast migration tests

- **Migration tests should be fast** – Run in CI/CD

- **Data volume matters** – Test with realistic data

- **Rollbacks are as important** – As upgrades

- **Automated testing is essential** – For confidence

- **Migration tests catch bugs** – Before production

- **Testing should be comprehensive** – Cover all paths

- **Migration testing saves time** – Prevents production issues

---

