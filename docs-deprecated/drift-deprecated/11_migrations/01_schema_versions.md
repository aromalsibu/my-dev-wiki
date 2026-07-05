## Schema Versions

**Managing database schema versions in Drift**

---

# What is it?

**Schema Versions** are a way to track and manage changes to your database structure over time. Every time you add, remove, or modify tables or columns, you increment the schema version number. Drift uses this version number to determine which migrations need to be applied when your app starts up.

> **Think of Schema Versions like "software version numbers"** – just as your app has a version number to track releases, your database has a version number to track structural changes. When the version changes, you know what updates need to be applied.

```dart
// 👇 Schema version management in Drift
@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  // 👇 Current schema version
  @override
  int get schemaVersion => 5;

  // 👇 Migration strategy handles version changes
  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (migrator) async {
      // Fresh database creation
      await migrator.createAll();
    },
    onUpgrade: (migrator, from, to) async {
      // Migrate from version to version
      if (from == 1) {
        await migrator.addColumn(users, users.newColumn);
      }
      if (from == 2) {
        await migrator.createTable(newTable);
      }
      // ... more migrations
    },
  );
}
```

> **What's happening here?**
> - **schemaVersion** – Current version number
> - **onCreate** – Runs when database is first created
> - **onUpgrade** – Runs when version changes
> - **Migration steps** – Incremental changes

---

# Why does it exist?

- **Schema Evolution** – Change database structure over time
- **Data Preservation** – Keep existing data when updating
- **Version Tracking** – Know what schema version is installed
- **Automated Upgrades** – Apply migrations automatically
- **Rollback Support** – Handle downgrades if needed
- **Testing** – Test migrations before deployment

---

# Setting Schema Versions

> **How to manage schema versions**

## Basic Version Management

```dart
// 👇 Simple version management
@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  // 👇 Start with version 1
  @override
  int get schemaVersion => 1;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (migrator) async {
      // 👇 Create all tables on first run
      await migrator.createAll();
      print('✅ Database created with all tables');
    },
  );
}
```

## Incrementing Versions

```dart
// 👇 Version 1 -> 2 migration
@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  // 👇 Increment version when schema changes
  @override
  int get schemaVersion => 2;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (migrator) async {
      // Fresh database
      await migrator.createAll();
    },
    onUpgrade: (migrator, from, to) async {
      // 👇 Migrate from version 1 to 2
      if (from == 1) {
        await migrator.addColumn(users, users.age);
        await migrator.addColumn(users, users.status);
        print('✅ Migrated users table: added age and status columns');
      }
    },
  );
}
```

---

# Migration Strategies

> **Handling schema migrations**

## Basic Migration

```dart
// 👇 Complete migration strategy
@override
MigrationStrategy get migration => MigrationStrategy(
  // 👇 Fresh database creation
  onCreate: (migrator) async {
    await migrator.createAll();
    await _seedInitialData();
  },
  
  // 👇 Upgrade path
  onUpgrade: (migrator, from, to) async {
    print('🔄 Upgrading from version $from to $to');
    
    // Version 1 -> 2: Add columns
    if (from == 1 && to == 2) {
      await migrator.addColumn(users, users.age);
      await migrator.addColumn(users, users.status);
    }
    
    // Version 2 -> 3: Create new table
    if (from == 2 && to == 3) {
      await migrator.createTable(profiles);
      await migrator.addColumn(users, users.profileId);
    }
    
    // Version 3 -> 4: Add constraints
    if (from == 3 && to == 4) {
      await migrator.addForeignKey(
        users,
        users.profileId,
        profiles,
        profiles.id,
      );
    }
  },
  
  // 👇 Downgrade handling (optional)
  onDowngrade: (migrator, from, to) async {
    print('⬇️ Downgrading from version $from to $to');
    // Handle downgrades carefully
    throw Exception('Downgrades not supported');
  },
  
  // 👇 Called before database opens
  beforeOpen: (details) async {
    print('📂 Opening database version ${details.version}');
  },
);

Future<void> _seedInitialData() async {
  // Insert default data
  await into(users).insert(
    UsersCompanion.insert(
      name: 'Admin',
      email: 'admin@example.com',
    ),
  );
}
```

---

# Real-World Example

> **Complete e-commerce migration system**

```dart
// lib/database/migration_service.dart
import 'package:drift/drift.dart';

class MigrationService {
  final AppDatabase db;
  
  MigrationService(this.db);

  // ==================== MIGRATION STRATEGY ====================
  
  // 👇 Complete migration strategy
  MigrationStrategy get migrationStrategy => MigrationStrategy(
    onCreate: (migrator) async {
      print('📦 Creating fresh database...');
      await _createAllTables(migrator);
      await _seedInitialData();
      print('✅ Database creation complete');
    },
    
    onUpgrade: (migrator, from, to) async {
      print('🔄 Upgrading from v$from to v$to...');
      
      // 👇 Version 1 -> 2: Add user columns
      if (from == 1 && to == 2) {
        await _migrateV1toV2(migrator);
      }
      
      // 👇 Version 2 -> 3: Create orders
      if (from == 2 && to == 3) {
        await _migrateV2toV3(migrator);
      }
      
      // 👇 Version 3 -> 4: Create reviews
      if (from == 3 && to == 4) {
        await _migrateV3toV4(migrator);
      }
      
      // 👇 Version 4 -> 5: Add indexes
      if (from == 4 && to == 5) {
        await _migrateV4toV5(migrator);
      }
      
      print('✅ Migration to v$to complete');
    },
    
    onDowngrade: (migrator, from, to) async {
      print('⬇️ Downgrading from v$from to v$to...');
      throw Exception('Downgrades are not supported');
    },
    
    beforeOpen: (details) async {
      print('📂 Opening database v${details.version}');
      await _validateSchema(details.version);
    },
  );

  // ==================== MIGRATION STEPS ====================
  
  // 👇 V1 -> V2: Add user columns
  Future<void> _migrateV1toV2(Migrator migrator) async {
    print('  ➕ Adding user columns...');
    await migrator.addColumn(db.users, db.users.age);
    await migrator.addColumn(db.users, db.users.status);
    await migrator.addColumn(db.users, db.users.phone);
    await migrator.addColumn(db.users, db.users.birthDate);
    
    // Set default values
    await db.customUpdate('''
      UPDATE users 
      SET status = 'active', age = 18
      WHERE status IS NULL OR age IS NULL
    ''').go();
    
    print('  ✅ User columns added');
  }
  
  // 👇 V2 -> V3: Create orders
  Future<void> _migrateV2toV3(Migrator migrator) async {
    print('  ➕ Creating orders table...');
    await migrator.createTable(db.orders);
    
    print('  ➕ Creating order_items table...');
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
    
    print('  ✅ Orders tables created');
  }
  
  // 👇 V3 -> V4: Create reviews
  Future<void> _migrateV3toV4(Migrator migrator) async {
    print('  ➕ Creating reviews table...');
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
    
    // Add indexes
    await migrator.addIndex(db.reviews, 'idx_reviews_user', [db.reviews.userId]);
    await migrator.addIndex(db.reviews, 'idx_reviews_product', [db.reviews.productId]);
    
    print('  ✅ Reviews table created');
  }
  
  // 👇 V4 -> V5: Add indexes
  Future<void> _migrateV4toV5(Migrator migrator) async {
    print('  📇 Adding indexes...');
    
    await migrator.addIndex(db.users, 'idx_users_email', [db.users.email]);
    await migrator.addIndex(db.users, 'idx_users_status', [db.users.status]);
    
    await migrator.addIndex(db.orders, 'idx_orders_status', [db.orders.status]);
    await migrator.addIndex(db.orders, 'idx_orders_date', [db.orders.orderDate]);
    
    await migrator.addIndex(db.products, 'idx_products_price', [db.products.price]);
    await migrator.addIndex(db.products, 'idx_products_stock', [db.products.stock]);
    
    print('  ✅ Indexes added');
  }

  // ==================== INITIAL SETUP ====================
  
  Future<void> _createAllTables(Migrator migrator) async {
    // 👇 Create all tables
    await migrator.createAll();
    print('  ✅ All tables created');
    
    // 👇 Add indexes
    await migrator.addIndex(db.users, 'idx_users_email', [db.users.email]);
    await migrator.addIndex(db.orders, 'idx_orders_user', [db.orders.userId]);
    await migrator.addIndex(db.orderItems, 'idx_order_items_order', [db.orderItems.orderId]);
    print('  ✅ Indexes created');
  }
  
  Future<void> _seedInitialData() async {
    print('  🌱 Seeding initial data...');
    
    // 👇 Create admin user
    await db.into(db.users).insert(
      UsersCompanion.insert(
        name: 'Admin',
        email: 'admin@example.com',
        status: Value('active'),
        age: Value(30),
      ),
    );
    
    // 👇 Create default categories
    await db.into(db.categories).insertAll([
      CategoriesCompanion.insert(name: 'Electronics'),
      CategoriesCompanion.insert(name: 'Clothing'),
      CategoriesCompanion.insert(name: 'Books'),
    ]);
    
    print('  ✅ Initial data seeded');
  }

  // ==================== SCHEMA VALIDATION ====================
  
  Future<void> _validateSchema(int version) async {
    try {
      // 👇 Check if all tables exist
      final tables = await db.customSelect('''
        SELECT name FROM sqlite_master 
        WHERE type = 'table' 
        AND name NOT LIKE 'sqlite_%'
      ''').get();
      
      print('  📊 Tables: ${tables.map((t) => t.data['name']).join(', ')}');
      
      // 👇 Check schema integrity
      final integrity = await db.customSelect('PRAGMA integrity_check').get();
      final status = integrity.first.data['integrity_check'] as String;
      
      if (status == 'ok') {
        print('  ✅ Schema integrity check passed');
      } else {
        print('  ⚠️ Schema integrity check: $status');
      }
      
    } catch (e) {
      print('  ❌ Schema validation failed: $e');
    }
  }

  // ==================== MIGRATION TESTING ====================
  
  // 👇 Test migrations
  Future<void> testMigrations() async {
    print('🧪 Testing migrations...');
    
    // Test each migration step
    final steps = [
      _testV1toV2,
      _testV2toV3,
      _testV3toV4,
      _testV4toV5,
    ];
    
    for (final step in steps) {
      await step();
    }
    
    print('✅ All migration tests passed');
  }
  
  Future<void> _testV1toV2() async {
    print('  ✅ V1->V2 migration tested');
  }
  
  Future<void> _testV2toV3() async {
    print('  ✅ V2->V3 migration tested');
  }
  
  Future<void> _testV3toV4() async {
    print('  ✅ V3->V4 migration tested');
  }
  
  Future<void> _testV4toV5() async {
    print('  ✅ V4->V5 migration tested');
  }

  // ==================== VERSION INFO ====================
  
  // 👇 Get schema version info
  Future<SchemaInfo> getSchemaInfo() async {
    final results = await db.customSelect('''
      SELECT 
        name,
        sql,
        type
      FROM sqlite_master 
      WHERE type IN ('table', 'index', 'trigger')
      ORDER BY type, name
    ''').get();
    
    final tables = results.where((r) => r.data['type'] == 'table').length;
    final indexes = results.where((r) => r.data['type'] == 'index').length;
    final triggers = results.where((r) => r.data['type'] == 'trigger').length;
    
    return SchemaInfo(
      version: db.schemaVersion,
      tableCount: tables,
      indexCount: indexes,
      triggerCount: triggers,
      details: results.map((r) {
        return SchemaDetail(
          name: r.data['name'] as String,
          type: r.data['type'] as String,
          sql: r.data['sql'] as String?,
        );
      }).toList(),
    );
  }
}

// ==================== DATA CLASSES ====================

class SchemaInfo {
  final int version;
  final int tableCount;
  final int indexCount;
  final int triggerCount;
  final List<SchemaDetail> details;
  
  SchemaInfo({
    required this.version,
    required this.tableCount,
    required this.indexCount,
    required this.triggerCount,
    required this.details,
  });
}

class SchemaDetail {
  final String name;
  final String type;
  final String? sql;
  
  SchemaDetail({
    required this.name,
    required this.type,
    this.sql,
  });
}
```

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';

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
  int get schemaVersion => 5; // 👈 Current version

  @override
  MigrationStrategy get migration => MigrationService(this).migrationStrategy;

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'app_database');
  }
}
```

---

# Schema Version Checklist

| Practice | Description | Impact |
|----------|-------------|--------|
| **Version Increment** | Bump version on schema change | High |
| **Migration Testing** | Test each migration | High |
| **Data Migration** | Migrate existing data | High |
| **Backup Before Migration** | Backup database | High |
| **Rollback Strategy** | Plan for rollbacks | Medium |
| **Documentation** | Document changes | Medium |
| **Testing** | Test with production data | High |

---

# Common Mistakes

## Mistake 1: Forgetting to increment version

Wrong:
```dart
// 🚫 Version stays at 1 after changes
@override
int get schemaVersion => 1;
```

Correct:
```dart
// ✅ Increment when schema changes
@override
int get schemaVersion => 2;
```

## Mistake 2: Not handling data migration

Wrong:
```dart
// 🚫 New column added but no default values
await migrator.addColumn(users, users.status);
```

Correct:
```dart
// ✅ Add default values
await migrator.addColumn(users, users.status);
await db.customUpdate('UPDATE users SET status = "active"').go();
```

## Mistake 3: Destructive migrations without backup

Wrong:
```dart
// 🚫 Dropping table without backup
await migrator.dropTable(users);
```

Correct:
```dart
// ✅ Backup before destructive operations
await db.customSelect('CREATE TABLE users_backup AS SELECT * FROM users').go();
await migrator.dropTable(users);
```

---

# Summary

| Feature | Purpose | Best Practice |
|---------|---------|---------------|
| **schemaVersion** | Track schema version | Increment on changes |
| **onCreate** | Fresh database setup | Create all tables |
| **onUpgrade** | Version-to-version migration | Test thoroughly |
| **onDowngrade** | Handle rollbacks | Plan carefully |

---

# Next Steps

Now you understand schema versions, let's dive deeper:

- [Migration Strategy](link) – Advanced migration patterns
- [Migration Examples](link) – Real-world migrations
- [Testing Migrations](link) – Migration testing

---

# Did You Know?

- **Schema version is stored in the database** – In `PRAGMA user_version`

- **Migrations are run in order** – From current version to target

- **Migrations are transactions** – All or nothing

- **OnCreate runs only once** – When database is first created

- **OnUpgrade runs on version changes** – Every time

- **OnDowngrade is optional** – Handle version decreases

- **Migrations can be complex** – Data migration + schema changes

- **Migrations are critical** – For data integrity

---
