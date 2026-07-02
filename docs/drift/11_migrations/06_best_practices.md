## Migration Best Practices

**Essential guidelines for safe and effective database migrations**

---

# What is it?

**Migration Best Practices** are the proven guidelines, patterns, and techniques for managing database schema changes safely and efficiently. They cover everything from planning and testing to deployment and rollback, ensuring that your migrations are reliable, maintainable, and minimally disruptive to your users.

> **Think of Migration Best Practices like "construction safety protocols"** – just as builders follow safety rules to prevent accidents, you follow migration best practices to prevent data loss, downtime, and production issues.

---

# Why does it exist?

- **Reliability** – Ensure migrations work consistently
- **Safety** – Prevent data loss and corruption
- **Minimal Downtime** – Keep applications running
- **Maintainability** – Easy to understand and modify
- **Testability** – Verify migrations work
- **Rollback Ability** – Undo changes if needed

---

# Core Principles

> **The fundamental rules of migration management**

## 1. Always Backup First

```dart
// 👇 Always create a backup before any migration
Future<void> migrateWithBackup() async {
  // 1️⃣ Create backup
  final backupName = await createBackup('users');
  
  try {
    // 2️⃣ Perform migration
    await performMigration();
    
    // 3️⃣ Verify migration
    await validateMigration();
    
  } catch (e) {
    // 4️⃣ Restore from backup
    await restoreFromBackup(backupName);
    rethrow;
  }
}
```

## 2. Use Transactions

```dart
// 👇 Always use transactions for migrations
await db.transaction(() async {
  // All changes in a single transaction
  await addColumn(users, users.newColumn);
  await migrateData();
  await addIndex(users, 'idx_new_column', [users.newColumn]);
  
  // If any fails, all rollback
});
```

## 3. Test Before Deploying

```dart
// test/migrations/migration_test.dart
void main() {
  test('Migration v2 works correctly', () async {
    final db = AppDatabase(NativeDatabase.memory());
    await db.ensureOpen();
    
    // 1️⃣ Insert test data
    await insertTestData(db);
    
    // 2️⃣ Run migration
    await db.migrateTo(2);
    
    // 3️⃣ Verify results
    await verifyMigration(db);
    
    await db.close();
  });
}
```

---

# Version Management

> **Handling schema versions properly**

## Incremental Versioning

```dart
// 👇 Use incremental version numbers
@override
int get schemaVersion => 5; // Current version

// Each migration has a unique version number
// v1 -> v2: Add status column
// v2 -> v3: Create orders table
// v3 -> v4: Add reviews table
// v4 -> v5: Add indexes
```

## Version Mapping

```dart
// 👇 Map versions to migration steps
class MigrationRegistry {
  static final Map<int, MigrationStep> steps = {
    1: MigrationStep(
      version: 1,
      description: 'Initial schema',
      up: _migrateV1ToV2,
      down: _rollbackV1ToV2,
    ),
    2: MigrationStep(
      version: 2,
      description: 'Add orders table',
      up: _migrateV2ToV3,
      down: _rollbackV2ToV3,
    ),
    3: MigrationStep(
      version: 3,
      description: 'Add reviews table',
      up: _migrateV3ToV4,
      down: _rollbackV3ToV4,
    ),
  };
  
  static Future<void> migrateTo(int targetVersion) async {
    for (final step in steps.values) {
      if (step.version < targetVersion) {
        await step.up();
      }
    }
  }
}
```

---

# Data Migration Patterns

> **Moving data safely between versions**

## Pattern 1: Add with Default

```dart
// 👇 Add column with default value
await migrator.addColumn(users, users.status);

// Set default for existing rows
await db.customUpdate('''
  UPDATE users 
  SET status = 'active' 
  WHERE status IS NULL
''').go();

// Now make it NOT NULL
await db.customSelect('''
  CREATE TABLE users_new (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'active'
  );
  
  INSERT INTO users_new SELECT * FROM users;
  DROP TABLE users;
  ALTER TABLE users_new RENAME TO users;
''').go();
```

## Pattern 2: Split Column

```dart
// 👇 Split one column into two
await db.customSelect('''
  ALTER TABLE users ADD COLUMN first_name TEXT;
  ALTER TABLE users ADD COLUMN last_name TEXT;
  
  UPDATE users SET 
    first_name = SUBSTR(full_name, 1, INSTR(full_name, ' ') - 1),
    last_name = SUBSTR(full_name, INSTR(full_name, ' ') + 1)
  WHERE full_name LIKE '% %';
''').go();
```

## Pattern 3: Merge Columns

```dart
// 👇 Merge multiple columns into one
await db.customSelect('''
  ALTER TABLE users ADD COLUMN full_name TEXT;
  
  UPDATE users SET 
    full_name = first_name || ' ' || last_name
  WHERE first_name IS NOT NULL AND last_name IS NOT NULL;
''').go();
```

---

# Real-World Example

> **Complete e-commerce migration best practices system**

```dart
// lib/database/migration_best_practices.dart
import 'package:drift/drift.dart';

class MigrationBestPractices {
  final AppDatabase db;
  
  MigrationBestPractices(this.db);

  // ==================== MIGRATION PLANNING ====================
  
  // 👇 Migration plan template
  Future<void> executeMigrationPlan({
    required int targetVersion,
    required bool backupFirst,
    required bool validateAfter,
  }) async {
    print('📋 Starting migration plan to v$targetVersion');
    
    // 1️⃣ Backup
    if (backupFirst) {
      print('💾 Creating backup...');
      await _createFullBackup();
    }
    
    // 2️⃣ Check prerequisites
    print('🔍 Checking prerequisites...');
    await _checkPrerequisites();
    
    // 3️⃣ Run migration
    print('📦 Running migration...');
    await _runMigration(targetVersion);
    
    // 4️⃣ Validate
    if (validateAfter) {
      print('✅ Validating migration...');
      await _validateMigration();
    }
    
    // 5️⃣ Cleanup
    print('🧹 Cleaning up...');
    await _cleanup();
    
    print('✅ Migration complete!');
  }

  // ==================== BACKUP STRATEGIES ====================
  
  Future<void> _createFullBackup() async {
    final timestamp = DateTime.now().millisecondsSinceEpoch;
    final backupName = 'full_backup_$timestamp';
    
    // Backup all tables
    final tables = await db.customSelect('''
      SELECT name FROM sqlite_master 
      WHERE type = 'table' AND name NOT LIKE 'sqlite_%'
    ''').get();
    
    for (final table in tables) {
      final tableName = table.data['name'] as String;
      await db.customSelect('''
        CREATE TABLE ${backupName}_$tableName AS 
        SELECT * FROM $tableName
      ''').go();
    }
    
    // Verify backup
    await _verifyBackup(backupName);
  }
  
  Future<void> _verifyBackup(String backupName) async {
    final tables = await db.customSelect('''
      SELECT name FROM sqlite_master 
      WHERE type = 'table' AND name LIKE '$backupName%'
    ''').get();
    
    if (tables.isEmpty) {
      throw Exception('Backup verification failed');
    }
    
    print('✅ Backup verified: ${tables.length} tables');
  }

  // ==================== PREREQUISITE CHECKS ====================
  
  Future<void> _checkPrerequisites() async {
    // 1️⃣ Check database integrity
    final integrity = await db.customSelect('PRAGMA integrity_check').get();
    final status = integrity.first.data['integrity_check'] as String;
    if (status != 'ok') {
      throw Exception('Database integrity check failed: $status');
    }
    
    // 2️⃣ Check foreign keys
    final fk = await db.customSelect('PRAGMA foreign_keys').get();
    if (fk.first.data['foreign_keys'] != 1) {
      await db.customSelect('PRAGMA foreign_keys = ON').get();
    }
    
    // 3️⃣ Check available space
    final pageCount = await db.customSelect('PRAGMA page_count').get();
    final pageSize = await db.customSelect('PRAGMA page_size').get();
    final size = (pageCount.first.data['page_count'] as int) * 
                 (pageSize.first.data['page_size'] as int);
    if (size > 100000000) { // 100MB
      print('⚠️ Database size: ${size ~/ 1000000}MB');
    }
  }

  // ==================== MIGRATION EXECUTION ====================
  
  Future<void> _runMigration(int targetVersion) async {
    await db.transaction(() async {
      // Version 1 -> 2
      if (targetVersion >= 2 && db.schemaVersion < 2) {
        await _migrateV1ToV2();
      }
      
      // Version 2 -> 3
      if (targetVersion >= 3 && db.schemaVersion < 3) {
        await _migrateV2ToV3();
      }
      
      // Version 3 -> 4
      if (targetVersion >= 4 && db.schemaVersion < 4) {
        await _migrateV3ToV4();
      }
    });
  }
  
  Future<void> _migrateV1ToV2() async {
    print('  📦 v1 -> v2: Add user status');
    
    // Add column with default
    await db.customSelect('''
      ALTER TABLE users ADD COLUMN status TEXT DEFAULT 'active'
    ''').go();
    
    // Update existing users
    await db.customUpdate('''
      UPDATE users SET status = 'active' WHERE status IS NULL
    ''').go();
    
    print('  ✅ v1 -> v2 complete');
  }
  
  Future<void> _migrateV2ToV3() async {
    print('  📦 v2 -> v3: Create orders');
    
    // Create tables
    await db.customSelect('''
      CREATE TABLE orders (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id INTEGER NOT NULL,
        total REAL NOT NULL,
        status TEXT NOT NULL,
        created_at INTEGER NOT NULL,
        FOREIGN KEY (user_id) REFERENCES users(id)
      );
      
      CREATE TABLE order_items (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        order_id INTEGER NOT NULL,
        product_id INTEGER NOT NULL,
        quantity INTEGER NOT NULL,
        price REAL NOT NULL,
        FOREIGN KEY (order_id) REFERENCES orders(id)
      );
    ''').go();
    
    print('  ✅ v2 -> v3 complete');
  }
  
  Future<void> _migrateV3ToV4() async {
    print('  📦 v3 -> v4: Add indexes');
    
    await db.customSelect('''
      CREATE INDEX idx_users_email ON users(email);
      CREATE INDEX idx_orders_user ON orders(user_id);
      CREATE INDEX idx_order_items_order ON order_items(order_id);
    ''').go();
    
    print('  ✅ v3 -> v4 complete');
  }

  // ==================== VALIDATION ====================
  
  Future<void> _validateMigration() async {
    // 1️⃣ Schema validation
    final tables = await db.customSelect('''
      SELECT name FROM sqlite_master 
      WHERE type = 'table'
    ''').get();
    
    final requiredTables = ['users', 'orders', 'order_items'];
    for (final table in requiredTables) {
      if (!tables.any((t) => t.data['name'] == table)) {
        throw Exception('Table $table not found');
      }
    }
    
    // 2️⃣ Data validation
    final userCount = await db.select(db.users).count();
    if (userCount == 0) {
      print('⚠️ No users found in database');
    }
    
    // 3️⃣ Foreign key validation
    final fk = await db.customSelect('PRAGMA foreign_key_check').get();
    if (fk.isNotEmpty) {
      print('⚠️ Foreign key violations found: ${fk.length}');
    }
    
    // 4️⃣ Index validation
    final indexes = await db.customSelect('''
      SELECT name FROM sqlite_master WHERE type = 'index'
    ''').get();
    print('✅ ${indexes.length} indexes found');
  }

  // ==================== CLEANUP ====================
  
  Future<void> _cleanup() async {
    // 1️⃣ Optimize database
    await db.customSelect('PRAGMA optimize').get();
    
    // 2️⃣ Check WAL file
    await db.customSelect('PRAGMA wal_checkpoint(TRUNCATE)').get();
    
    // 3️⃣ Clean up temporary tables
    await db.customSelect('''
      DROP TABLE IF EXISTS migration_backup
    ''').go();
    
    print('🧹 Cleanup complete');
  }

  // ==================== ROLLBACK STRATEGY ====================
  
  Future<void> _rollbackMigration() async {
    print('⬇️ Rolling back migration...');
    
    // 1️⃣ Restore from backup
    final latestBackup = await _getLatestBackup();
    if (latestBackup != null) {
      await _restoreFromBackup(latestBackup);
    }
    
    // 2️⃣ Drop new tables
    await db.customSelect('DROP TABLE IF EXISTS orders').go();
    await db.customSelect('DROP TABLE IF EXISTS order_items').go();
    
    // 3️⃣ Remove new columns
    await db.customSelect('''
      CREATE TABLE users_new AS 
      SELECT id, name, email, created_at FROM users
    ''').go();
    await db.customSelect('DROP TABLE users').go();
    await db.customSelect('ALTER TABLE users_new RENAME TO users').go();
    
    print('✅ Rollback complete');
  }
  
  Future<String?> _getLatestBackup() async {
    final backups = await db.customSelect('''
      SELECT name FROM sqlite_master 
      WHERE type = 'table' AND name LIKE 'backup_%'
      ORDER BY name DESC
    ''').get();
    return backups.isNotEmpty ? backups.first.data['name'] as String : null;
  }
  
  Future<void> _restoreFromBackup(String backupName) async {
    await db.customInsert('''
      INSERT OR REPLACE INTO users 
      SELECT * FROM $backupName
    ''').go();
  }
}
```

---

# Migration Best Practices Checklist

| Practice | Description | Priority |
|----------|-------------|----------|
| **Backup** | Create before migration | Critical |
| **Transaction** | Use for safety | Critical |
| **Testing** | Test thoroughly | Critical |
| **Validation** | Verify after migration | High |
| **Rollback Plan** | Have a way back | High |
| **Documentation** | Track changes | Medium |
| **Monitoring** | Watch performance | Medium |
| **Automation** | CI/CD integration | Medium |

---

# Common Mistakes to Avoid

## 1. No Backup

Always backup before any migration, especially destructive ones.

## 2. No Transaction

Always use transactions to ensure atomicity.

## 3. No Testing

Always test migrations on staging before production.

## 4. No Validation

Always validate after migration to ensure success.

## 5. No Rollback Plan

Always have a way to undo changes.

## 6. No Documentation

Always document what changed and why.

## 7. No Performance Testing

Always check migration speed with realistic data.

## 8. No Monitoring

Always monitor migrations in production.

---

# Summary

| Principle | Why | How |
|-----------|-----|-----|
| **Backup First** | Prevent data loss | Always create backup |
| **Use Transactions** | Ensure atomicity | Wrap in transaction |
| **Test Thoroughly** | Catch errors early | In-memory testing |
| **Validate After** | Verify success | Schema/data checks |
| **Plan Rollback** | Undo if needed | Prepare reverse |
| **Document Changes** | Track history | Migration logs |
| **Monitor Performance** | Check speed | Performance tests |

---

# Next Steps

Now you understand migration best practices, let's dive deeper:

- [Type Converters](link) – Custom type converters
- [DAO](link) – Data Access Objects
- [Performance](link) – Performance optimization

---

# Did You Know?

- **Best practices prevent disasters** – Data loss is avoidable

- **Testing saves time** – Catches issues early

- **Backups are essential** – Always have a safety net

- **Transactions ensure atomicity** – All or nothing

- **Validation gives confidence** – Know it worked

- **Rollback plans are crucial** – Have a way back

- **Documentation helps others** – Share knowledge

- **Best practices evolve** – Stay updated

---
