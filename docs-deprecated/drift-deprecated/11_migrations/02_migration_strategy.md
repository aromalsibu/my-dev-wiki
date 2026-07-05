## Migration Strategy

**Building robust migration strategies for Drift**

---

# What is it?

**Migration Strategy** is a comprehensive approach to handling database schema changes over time. It encompasses planning, testing, and executing migrations safely. A good migration strategy ensures that data is preserved, applications remain compatible, and rollbacks are possible. Drift provides a flexible `MigrationStrategy` class that allows you to define how your database evolves.

> **Think of Migration Strategy like a "renovation plan for your house"** – you don't just start knocking down walls. You plan each phase, ensure the structure stays safe, have a backup plan, and keep the house livable during renovations.

```dart
// 👇 Complete migration strategy
@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (migrator) async {
    // First time setup
    await migrator.createAll();
    await seedInitialData();
  },
  onUpgrade: (migrator, from, to) async {
    // Progressive upgrades
    if (from == 1) await migrateV1toV2();
    if (from == 2) await migrateV2toV3();
    // ... etc
  },
  onDowngrade: (migrator, from, to) async {
    // Handle rollbacks
    throw Exception('Downgrades not supported');
  },
  beforeOpen: (details) async {
    // Pre-open validation
    await validateSchema();
  },
);
```

> **What's happening here?**
> - **onCreate** – Fresh database setup
> - **onUpgrade** – Version-to-version migration
> - **onDowngrade** – Rollback handling
> - **beforeOpen** – Schema validation

---

# Why does it exist?

- **Data Preservation** – Keep data during schema changes
- **Version Compatibility** – Support multiple app versions
- **Risk Management** – Minimize migration risks
- **Rollback Capability** – Revert to previous versions
- **Testing** – Validate migrations before deployment
- **Documentation** – Track schema evolution

---

# Migration Strategy Components

> **Building a complete migration strategy**

## Progressive Migration

```dart
// 👇 Progressive migration approach
@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (migrator) async {
    // Build from scratch
    await migrator.createAll();
    await seedInitialData();
  },
  onUpgrade: (migrator, from, to) async {
    // 👇 Migrate step by step
    if (from < 2) await _migrateToV2();
    if (from < 3) await _migrateToV3();
    if (from < 4) await _migrateToV4();
    if (from < 5) await _migrateToV5();
    // ... etc
  },
);

Future<void> _migrateToV2() async {
  print('📦 Migrating to version 2...');
  // Add columns
  await addColumn(users, users.age);
  await addColumn(users, users.status);
  // Set defaults
  await updateUserDefaults();
}

Future<void> _migrateToV3() async {
  print('📦 Migrating to version 3...');
  // Create new tables
  await createTable(profiles);
  await createTable(userSettings);
  // Add foreign keys
  await addForeignKey(profiles, profiles.userId, users, users.id);
}
```

---

## Conditional Migration

```dart
// 👇 Conditional migration based on app version
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (migrator, from, to) async {
    // Check if migration is needed
    if (from == to) {
      print('✅ Schema up to date');
      return;
    }
    
    // Check app version compatibility
    final appVersion = await getAppVersion();
    if (appVersion < requiredMinVersion) {
      throw Exception('App version too old for database migration');
    }
    
    // Perform migrations
    await _performMigrations(from, to);
  },
);
```

---

# Best Practices

> **Essential migration patterns**

## 1. Always Backup First

```dart
// 👇 Backup before migration
Future<void> _migrateWithBackup(Migrator migrator) async {
  // Create backup table
  await db.customSelect('''
    CREATE TABLE IF NOT EXISTS users_backup_${DateTime.now().millisecondsSinceEpoch} 
    AS SELECT * FROM users
  ''').go();
  
  try {
    // Perform migration
    await migrator.addColumn(users, users.newColumn);
    await migrateData();
  } catch (e) {
    // Restore from backup if needed
    await db.customSelect('''
      INSERT OR REPLACE INTO users 
      SELECT * FROM users_backup_${DateTime.now().millisecondsSinceEpoch}
    ''').go();
    rethrow;
  }
}
```

## 2. Validate After Migration

```dart
// 👇 Validate after migration
Future<void> _migrateAndValidate() async {
  await _performMigration();
  
  // Validate schema
  final isValid = await _validateSchema();
  if (!isValid) {
    throw Exception('Migration validation failed');
  }
  
  // Validate data
  final dataValid = await _validateData();
  if (!dataValid) {
    throw Exception('Data validation failed');
  }
}

Future<bool> _validateSchema() async {
  // Check if all columns exist
  final columns = await getTableColumns('users');
  return columns.contains('new_column');
}

Future<bool> _validateData() async {
  // Check if data is valid
  final invalidCount = await db.customSelect('''
    SELECT COUNT(*) as count FROM users WHERE new_column IS NULL
  ''').get();
  return invalidCount.first.data['count'] == 0;
}
```

## 3. Use Transactions

```dart
// 👇 Migration in transaction
Future<void> _safeMigrate() async {
  await db.transaction(() async {
    // All changes in transaction
    await _migrateV1toV2();
    await _migrateV2toV3();
    await _updateIndexes();
    
    // If any fails, all rollback
  });
}
```

---

# Real-World Example

> **Complete e-commerce migration strategy**

```dart
// lib/database/migration_strategy_service.dart
import 'package:drift/drift.dart';

class MigrationStrategyService {
  final AppDatabase db;
  int _currentVersion = 1;
  
  MigrationStrategyService(this.db);

  // ==================== COMPLETE MIGRATION STRATEGY ====================
  
  MigrationStrategy get strategy => MigrationStrategy(
    onCreate: _onCreate,
    onUpgrade: _onUpgrade,
    onDowngrade: _onDowngrade,
    beforeOpen: _beforeOpen,
  );

  // ==================== ON CREATE ====================
  
  Future<void> _onCreate(Migrator migrator) async {
    print('📦 Creating fresh database...');
    
    try {
      // 1️⃣ Create all tables
      await migrator.createAll();
      print('  ✅ All tables created');
      
      // 2️⃣ Add indexes
      await _createInitialIndexes(migrator);
      
      // 3️⃣ Add foreign keys
      await _createForeignKeys(migrator);
      
      // 4️⃣ Seed initial data
      await _seedInitialData();
      
      // 5️⃣ Log creation
      await _logMigration('create', 1, 1);
      
      print('✅ Database creation complete!');
      
    } catch (e) {
      print('❌ Database creation failed: $e');
      rethrow;
    }
  }

  // ==================== ON UPGRADE ====================
  
  Future<void> _onUpgrade(Migrator migrator, int from, int to) async {
    print('🔄 Upgrading from v$from to v$to...');
    
    // 👇 Create backup
    await _createBackup();
    
    try {
      // 👇 Progressive migrations
      if (from <= 1 && to > 1) await _migrateV1ToV2(migrator);
      if (from <= 2 && to > 2) await _migrateV2ToV3(migrator);
      if (from <= 3 && to > 3) await _migrateV3ToV4(migrator);
      if (from <= 4 && to > 4) await _migrateV4ToV5(migrator);
      
      // 👇 Validate after migration
      await _validateAfterUpgrade(migrator, to);
      
      // 👇 Log migration
      await _logMigration('upgrade', from, to);
      
      print('✅ Migration to v$to complete!');
      
    } catch (e) {
      print('❌ Migration failed: $e');
      await _restoreBackup();
      rethrow;
    }
  }

  // ==================== MIGRATION STEPS ====================
  
  // V1 -> V2: Add user columns
  Future<void> _migrateV1ToV2(Migrator migrator) async {
    print('  📦 v1 -> v2: Adding user columns');
    
    // Add columns
    await migrator.addColumn(db.users, db.users.age);
    await migrator.addColumn(db.users, db.users.status);
    await migrator.addColumn(db.users, db.users.phone);
    
    // Set defaults
    await db.customUpdate('''
      UPDATE users 
      SET status = 'active', age = 18 
      WHERE status IS NULL
    ''').go();
    
    // Add index
    await migrator.addIndex(db.users, 'idx_users_status', [db.users.status]);
    
    print('  ✅ v1 -> v2 complete');
  }
  
  // V2 -> V3: Create orders
  Future<void> _migrateV2ToV3(Migrator migrator) async {
    print('  📦 v2 -> v3: Creating orders');
    
    // Create tables
    await migrator.createTable(db.orders);
    await migrator.createTable(db.orderItems);
    
    // Add foreign keys
    await migrator.addForeignKey(
      db.orderItems,
      db.orderItems.orderId,
      db.orders,
      db.orders.id,
    );
    await migrator.addForeignKey(
      db.orderItems,
      db.orderItems.productId,
      db.products,
      db.products.id,
    );
    
    // Add indexes
    await migrator.addIndex(db.orders, 'idx_orders_user', [db.orders.userId]);
    await migrator.addIndex(db.orderItems, 'idx_order_items_order', [db.orderItems.orderId]);
    
    print('  ✅ v2 -> v3 complete');
  }
  
  // V3 -> V4: Add reviews
  Future<void> _migrateV3ToV4(Migrator migrator) async {
    print('  📦 v3 -> v4: Adding reviews');
    
    await migrator.createTable(db.reviews);
    
    await migrator.addForeignKey(
      db.reviews,
      db.reviews.userId,
      db.users,
      db.users.id,
    );
    await migrator.addForeignKey(
      db.reviews,
      db.reviews.productId,
      db.products,
      db.products.id,
    );
    
    await migrator.addIndex(db.reviews, 'idx_reviews_user', [db.reviews.userId]);
    await migrator.addIndex(db.reviews, 'idx_reviews_product', [db.reviews.productId]);
    
    print('  ✅ v3 -> v4 complete');
  }
  
  // V4 -> V5: Add indexes and constraints
  Future<void> _migrateV4ToV5(Migrator migrator) async {
    print('  📦 v4 -> v5: Adding indexes');
    
    await migrator.addIndex(db.users, 'idx_users_email', [db.users.email]);
    await migrator.addIndex(db.orders, 'idx_orders_status', [db.orders.status]);
    await migrator.addIndex(db.orders, 'idx_orders_date', [db.orders.orderDate]);
    await migrator.addIndex(db.products, 'idx_products_price', [db.products.price]);
    await migrator.addIndex(db.products, 'idx_products_stock', [db.products.stock]);
    
    print('  ✅ v4 -> v5 complete');
  }

  // ==================== ON DOWNGRADE ====================
  
  Future<void> _onDowngrade(Migrator migrator, int from, int to) async {
    print('⬇️ Downgrading from v$from to v$to...');
    
    try {
      // 👇 Progressive downgrades
      if (from == 5 && to < 5) await _downgradeV5ToV4(migrator);
      if (from == 4 && to < 4) await _downgradeV4ToV3(migrator);
      
      await _logMigration('downgrade', from, to);
      print('✅ Downgrade to v$to complete!');
      
    } catch (e) {
      print('❌ Downgrade failed: $e');
      rethrow;
    }
  }
  
  Future<void> _downgradeV5ToV4(Migrator migrator) async {
    print('  ⬇️ v5 -> v4: Removing indexes');
    
    // Drop indexes (SQLite doesn't support DROP INDEX directly in migrator)
    await db.customSelect('DROP INDEX IF EXISTS idx_users_email').go();
    await db.customSelect('DROP INDEX IF EXISTS idx_orders_status').go();
    await db.customSelect('DROP INDEX IF EXISTS idx_orders_date').go();
    await db.customSelect('DROP INDEX IF EXISTS idx_products_price').go();
    await db.customSelect('DROP INDEX IF EXISTS idx_products_stock').go();
    
    print('  ✅ v5 -> v4 complete');
  }
  
  Future<void> _downgradeV4ToV3(Migrator migrator) async {
    print('  ⬇️ v4 -> v3: Removing reviews');
    
    await db.customSelect('DROP TABLE IF EXISTS reviews').go();
    
    print('  ✅ v4 -> v3 complete');
  }

  // ==================== BEFORE OPEN ====================
  
  Future<void> _beforeOpen(OpeningDetails details) async {
    print('📂 Opening database v${details.version}');
    
    // 👇 Check if schema is valid
    if (!await _isSchemaValid()) {
      print('⚠️ Schema validation failed');
    }
    
    // 👇 Run integrity check
    await _integrityCheck();
    
    // 👇 Optimize for performance
    await db.customSelect('PRAGMA optimize').get();
  }

  // ==================== HELPER METHODS ====================
  
  Future<void> _createInitialIndexes(Migrator migrator) async {
    print('  📇 Creating initial indexes');
    await migrator.addIndex(db.users, 'idx_users_email', [db.users.email]);
    await migrator.addIndex(db.orders, 'idx_orders_user', [db.orders.userId]);
    await migrator.addIndex(db.orderItems, 'idx_order_items_order', [db.orderItems.orderId]);
  }
  
  Future<void> _createForeignKeys(Migrator migrator) async {
    print('  🔗 Creating foreign keys');
    await migrator.addForeignKey(
      db.orderItems,
      db.orderItems.orderId,
      db.orders,
      db.orders.id,
    );
  }
  
  Future<void> _seedInitialData() async {
    print('  🌱 Seeding initial data');
    await db.into(db.users).insert(
      UsersCompanion.insert(
        name: 'Admin',
        email: 'admin@example.com',
        status: Value('active'),
        age: Value(30),
      ),
    );
  }
  
  Future<void> _createBackup() async {
    print('  💾 Creating backup');
    final timestamp = DateTime.now().millisecondsSinceEpoch;
    await db.customSelect('''
      CREATE TABLE IF NOT EXISTS backup_${timestamp} AS 
      SELECT * FROM users
    ''').go();
  }
  
  Future<void> _restoreBackup() async {
    print('  ↩️ Restoring from backup');
    // Restore logic
  }
  
  Future<void> _validateAfterUpgrade(Migrator migrator, int to) async {
    print('  ✅ Validating migration');
    // Validation logic
  }
  
  Future<void> _logMigration(String action, int from, int to) async {
    await db.customInsert('''
      INSERT INTO migration_log (action, from_version, to_version, timestamp)
      VALUES (?, ?, ?, datetime('now'))
    ''', variables: [
      Variable.withString(action),
      Variable.withInt(from),
      Variable.withInt(to),
    ]).go();
  }
  
  Future<bool> _isSchemaValid() async {
    try {
      await db.customSelect('SELECT 1 FROM users LIMIT 1').get();
      return true;
    } catch (e) {
      return false;
    }
  }
  
  Future<void> _integrityCheck() async {
    final result = await db.customSelect('PRAGMA integrity_check').get();
    final status = result.first.data['integrity_check'] as String;
    if (status == 'ok') {
      print('  ✅ Schema integrity check passed');
    } else {
      print('  ⚠️ Schema integrity check: $status');
    }
  }
}
```

```dart
// lib/database/database.dart
@DriftDatabase(tables: [
  Users,
  Products,
  Categories,
  Orders,
  OrderItems,
  Reviews,
])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 5;

  @override
  MigrationStrategy get migration => MigrationStrategyService(this).strategy;

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'app_database');
  }
}
```

---

# Migration Strategy Checklist

| Component | Purpose | Best Practice |
|-----------|---------|---------------|
| **onCreate** | Fresh database | Create all tables, seed data |
| **onUpgrade** | Version upgrade | Progressive steps |
| **onDowngrade** | Version rollback | Safe removal |
| **beforeOpen** | Pre-open validation | Check schema |
| **Backup** | Data protection | Before migrations |
| **Validation** | Verify migration | After migration |
| **Logging** | Track changes | Log all migrations |

---

# Common Mistakes

## Mistake 1: No backup before migration

Wrong:
```dart
// 🚫 Direct migration without backup
await migrator.addColumn(users, users.newColumn);
```

Correct:
```dart
// ✅ Backup before migration
await _createBackup();
await migrator.addColumn(users, users.newColumn);
```

## Mistake 2: Not handling errors

Wrong:
```dart
// 🚫 No error handling
await _performMigration();
```

Correct:
```dart
// ✅ Handle errors
try {
  await _performMigration();
} catch (e) {
  await _rollbackMigration();
  rethrow;
}
```

## Mistake 3: Skipping validation

Wrong:
```dart
// 🚫 No validation after migration
await migrator.addColumn(users, users.newColumn);
```

Correct:
```dart
// ✅ Validate after migration
await migrator.addColumn(users, users.newColumn);
await _validateMigration();
```

---

# Summary

| Aspect | Description | Best Practice |
|--------|-------------|---------------|
| **Backup** | Create before migration | Always |
| **Validation** | Verify after migration | Always |
| **Transaction** | Use for safety | Always |
| **Rollback** | Handle failures | Plan for |
| **Logging** | Track changes | Implement |
| **Testing** | Test migrations | Thorough |

---

# Next Steps

Now you understand migration strategy, let's dive deeper:

- [Migration Examples](link) – Real-world examples
- [Destructive Migrations](link) – Handling destructive changes
- [Testing Migrations](link) – Testing strategies

---

# Did You Know?

- **Migrations can be complex** – Data transformation
- **Migrations should be reversible** – Plan rollbacks
- **Migrations are transactional** – All or nothing
- **Migrations can be tested** – With test databases
- **Migrations are critical** – For data integrity
- **Migrations require planning** – Think ahead
- **Migrations should be documented** – Track changes
- **Migrations are production-critical** – Handle with care

---
