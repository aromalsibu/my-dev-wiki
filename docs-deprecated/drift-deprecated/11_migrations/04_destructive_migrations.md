## Destructive Migrations

**Handling dangerous database changes safely in Drift**

---

# What is it?

**Destructive Migrations** are schema changes that delete or modify data in ways that cannot be easily undone. These include dropping tables, removing columns, changing data types in ways that lose precision, or any operation that could result in permanent data loss. Handling these migrations requires extra caution, thorough testing, and careful planning.

> **Think of Destructive Migrations like "demolishing part of a building"** – you need to be absolutely sure before you knock down walls, have a detailed plan, backup everything, and know exactly how to rebuild if something goes wrong.

```dart
// 👇 Destructive migration: Dropping a table
await db.transaction(() async {
  // 1️⃣ Create backup
  await db.customSelect('''
    CREATE TABLE backup_old_table AS SELECT * FROM old_table
  ''').go();
  
  // 2️⃣ Verify backup
  final count = await db.select(db.backup).count();
  print('✅ Backed up $count rows');
  
  // 3️⃣ Drop the table
  await db.customSelect('DROP TABLE old_table').go();
  
  // 4️⃣ Create new structure
  await db.customSelect('''
    CREATE TABLE new_table (...)
  ''').go();
  
  // 5️⃣ Migrate data
  await db.customInsert('''
    INSERT INTO new_table SELECT * FROM backup_old_table
  ''').go();
});
```

> **What's happening here?**
> - **Backup first** – Always create a backup
> - **Verify backup** – Ensure data is safe
> - **Perform destructive operation** – With caution
> - **Rebuild** – Create new structure
> - **Data migration** – Transfer data to new structure

---

# Why does it exist?

- **Schema Cleanup** – Remove obsolete structures
- **Optimization** – Improve performance
- **Data Architecture Changes** – Restructure for new requirements
- **Feature Removal** – Remove deprecated features
- **Data Normalization** – Improve data design
- **Technical Debt** – Clean up old design decisions

---

# Types of Destructive Operations

> **Identifying destructive schema changes**

## Dropping Tables

```dart
// 👇 Safely dropping a table
Future<void> dropTableSafely(
  String tableName,
  AppDatabase db,
) async {
  await db.transaction(() async {
    // 1️⃣ Create backup
    final backupName = 'backup_${tableName}_${DateTime.now().millisecondsSinceEpoch}';
    await db.customSelect('''
      CREATE TABLE $backupName AS SELECT * FROM $tableName
    ''').go();
    
    // 2️⃣ Verify backup
    final count = await db.customSelect('''
      SELECT COUNT(*) as count FROM $backupName
    ''').get();
    print('✅ Backed up ${count.first.data['count']} rows');
    
    // 3️⃣ Check dependencies
    final dependencies = await _getTableDependencies(tableName, db);
    if (dependencies.isNotEmpty) {
      print('⚠️ Table has dependencies: ${dependencies.join(', ')}');
      // Optionally handle dependencies
    }
    
    // 4️⃣ Drop the table
    await db.customSelect('DROP TABLE $tableName').go();
    print('✅ Table dropped successfully');
  });
}

Future<List<String>> _getTableDependencies(
  String tableName,
  AppDatabase db,
) async {
  final results = await db.customSelect('''
    SELECT sql FROM sqlite_master 
    WHERE sql LIKE '%REFERENCES $tableName%'
  ''').get();
  return results.map((r) => r.data['sql'] as String).toList();
}
```

## Removing Columns

```dart
// 👇 Safely removing a column
Future<void> removeColumnSafely(
  String tableName,
  String columnName,
  AppDatabase db,
) async {
  await db.transaction(() async {
    // 1️⃣ Verify column exists
    final columns = await db.customSelect('''
      PRAGMA table_info($tableName)
    ''').get();
    
    final columnExists = columns.any((c) => c.data['name'] == columnName);
    if (!columnExists) {
      print('⚠️ Column $columnName does not exist in $tableName');
      return;
    }
    
    // 2️⃣ Backup specific column data
    final backupName = 'backup_${tableName}_${columnName}_${DateTime.now().millisecondsSinceEpoch}';
    await db.customSelect('''
      CREATE TABLE $backupName AS 
      SELECT id, $columnName FROM $tableName
    ''').go();
    
    // 3️⃣ Get all column names except the one to remove
    final columnNames = columns
        .where((c) => c.data['name'] != columnName)
        .map((c) => c.data['name'] as String)
        .join(', ');
    
    // 4️⃣ Create new table without the column
    final tempTable = '${tableName}_new';
    await db.customSelect('''
      CREATE TABLE $tempTable AS 
      SELECT $columnNames FROM $tableName
    ''').go();
    
    // 5️⃣ Swap tables
    await db.customSelect('DROP TABLE $tableName').go();
    await db.customSelect('ALTER TABLE $tempTable RENAME TO $tableName').go();
    
    print('✅ Column $columnName removed from $tableName');
  });
}
```

## Changing Data Types

```dart
// 👇 Safely changing a data type
Future<void> changeColumnTypeSafely({
  required String tableName,
  required String columnName,
  required String newType,
  required String conversionSql,
  required AppDatabase db,
}) async {
  await db.transaction(() async {
    // 1️⃣ Get table info
    final columns = await db.customSelect('''
      PRAGMA table_info($tableName)
    ''').get();
    
    // 2️⃣ Create backup
    final backupName = 'backup_${tableName}_${DateTime.now().millisecondsSinceEpoch}';
    await db.customSelect('''
      CREATE TABLE $backupName AS SELECT * FROM $tableName
    ''').go();
    
    // 3️⃣ Verify data can be converted
    final invalidCount = await db.customSelect('''
      SELECT COUNT(*) as count FROM $tableName
      WHERE $columnName IS NOT NULL
      AND NOT ($conversionSql)
    ''').get();
    
    if (invalidCount.first.data['count'] > 0) {
      print('⚠️ Found ${invalidCount.first.data['count']} rows that cannot be converted');
      // Handle invalid data
    }
    
    // 4️⃣ Create new table with changed type
    final columnDefs = columns.map((c) {
      final name = c.data['name'] as String;
      final type = name == columnName ? newType : c.data['type'];
      return '$name $type';
    }).join(', ');
    
    final tempTable = '${tableName}_new';
    await db.customSelect('''
      CREATE TABLE $tempTable ($columnDefs)
    ''').go();
    
    // 5️⃣ Convert and copy data
    final selectColumns = columns.map((c) {
      final name = c.data['name'] as String;
      if (name == columnName) {
        return '$conversionSql as $name';
      }
      return name;
    }).join(', ');
    
    await db.customInsert('''
      INSERT INTO $tempTable ($selectColumns)
      SELECT $selectColumns FROM $tableName
    ''').go();
    
    // 6️⃣ Swap tables
    await db.customSelect('DROP TABLE $tableName').go();
    await db.customSelect('ALTER TABLE $tempTable RENAME TO $tableName').go();
    
    print('✅ Column $columnName type changed to $newType');
  });
}
```

---

# Real-World Example

> **Complete e-commerce destructive migration system**

```dart
// lib/database/destructive_migration_service.dart
import 'package:drift/drift.dart';

class DestructiveMigrationService {
  final AppDatabase db;
  
  DestructiveMigrationService(this.db);

  // ==================== BACKUP SYSTEM ====================
  
  // 👇 Comprehensive backup
  Future<BackupResult> createBackup({
    required String tableName,
    bool includeSchema = true,
  }) async {
    final timestamp = DateTime.now().millisecondsSinceEpoch;
    final backupName = 'backup_${tableName}_$timestamp';
    
    try {
      // 1️⃣ Create backup table
      await db.customSelect('''
        CREATE TABLE $backupName AS SELECT * FROM $tableName
      ''').go();
      
      // 2️⃣ Backup schema if needed
      if (includeSchema) {
        await db.customSelect('''
          CREATE TABLE ${backupName}_schema AS 
          SELECT * FROM sqlite_master WHERE tbl_name = '$tableName'
        ''').go();
      }
      
      // 3️⃣ Verify backup
      final count = await db.customSelect('''
        SELECT COUNT(*) as count FROM $backupName
      ''').get();
      
      return BackupResult(
        backupName: backupName,
        rowCount: count.first.data['count'] as int,
        timestamp: DateTime.now(),
        success: true,
      );
      
    } catch (e) {
      return BackupResult(
        backupName: backupName,
        rowCount: 0,
        timestamp: DateTime.now(),
        success: false,
        error: e.toString(),
      );
    }
  }
  
  // 👇 Restore from backup
  Future<bool> restoreFromBackup({
    required String backupName,
    required String tableName,
  }) async {
    try {
      // Check if backup exists
      final exists = await db.customSelect('''
        SELECT name FROM sqlite_master 
        WHERE type = 'table' AND name = '$backupName'
      ''').get();
      
      if (exists.isEmpty) {
        throw Exception('Backup $backupName not found');
      }
      
      // Restore data
      await db.customInsert('''
        INSERT OR REPLACE INTO $tableName 
        SELECT * FROM $backupName
      ''').go();
      
      return true;
      
    } catch (e) {
      print('❌ Restore failed: $e');
      return false;
    }
  }

  // ==================== DESTRUCTIVE MIGRATIONS ====================
  
  // 👇 Version 5 -> 6: Drop deprecated tables
  Future<void> migrateV5ToV6() async {
    print('📦 Running destructive migration: v5 -> v6');
    
    await db.transaction(() async {
      // 1️⃣ Backup deprecated tables
      print('  💾 Creating backups...');
      final backupResult = await createBackup(tableName: 'old_orders');
      if (!backupResult.success) {
        throw Exception('Backup failed: ${backupResult.error}');
      }
      
      // 2️⃣ Check dependencies
      print('  🔗 Checking dependencies...');
      final deps = await _getTableDependencies('old_orders');
      if (deps.isNotEmpty) {
        print('  ⚠️ Found dependencies: $deps');
        // Handle dependencies
        await _handleDependencies(deps);
      }
      
      // 3️⃣ Drop old tables
      print('  🗑️ Dropping old tables...');
      await db.customSelect('DROP TABLE IF EXISTS old_orders').go();
      await db.customSelect('DROP TABLE IF EXISTS old_order_items').go();
      
      // 4️⃣ Create new optimized tables
      print('  📝 Creating new tables...');
      await _createNewOrderTables();
      
      // 5️⃣ Migrate data
      print('  📊 Migrating data...');
      await _migrateOrderData(backupResult.backupName);
      
      print('✅ Destructive migration complete');
    });
  }
  
  // 👇 Version 6 -> 7: Remove column
  Future<void> migrateV6ToV7() async {
    print('📦 Running destructive migration: v6 -> v7');
    
    await db.transaction(() async {
      // 1️⃣ Backup data
      print('  💾 Creating backups...');
      await createBackup(tableName: 'users');
      
      // 2️⃣ Remove deprecated column
      print('  🗑️ Removing column...');
      await _removeColumn('users', 'old_status');
      
      // 3️⃣ Add new column
      print('  ➕ Adding new column...');
      await _addColumn('users', 'user_status');
      
      // 4️⃣ Migrate data
      print('  📊 Migrating data...');
      await _migrateUserStatus();
      
      print('✅ Destructive migration complete');
    });
  }

  // ==================== HELPER METHODS ====================
  
  Future<List<String>> _getTableDependencies(String tableName) async {
    final results = await db.customSelect('''
      SELECT sql FROM sqlite_master 
      WHERE type = 'table' 
      AND sql LIKE '%REFERENCES $tableName%'
    ''').get();
    return results.map((r) => r.data['sql'] as String).toList();
  }
  
  Future<void> _handleDependencies(List<String> dependencies) async {
    for (final dep in dependencies) {
      print('    Handling dependency: $dep');
      // Handle each dependency
    }
  }
  
  Future<void> _createNewOrderTables() async {
    // Create optimized order tables
  }
  
  Future<void> _migrateOrderData(String backupName) async {
    // Migrate data from backup to new tables
  }
  
  Future<void> _removeColumn(String tableName, String columnName) async {
    // Remove column implementation
  }
  
  Future<void> _addColumn(String tableName, String columnName) async {
    // Add column implementation
  }
  
  Future<void> _migrateUserStatus() async {
    // Migrate user status data
  }
}

// ==================== DATA CLASSES ====================

class BackupResult {
  final String backupName;
  final int rowCount;
  final DateTime timestamp;
  final bool success;
  final String? error;
  
  BackupResult({
    required this.backupName,
    required this.rowCount,
    required this.timestamp,
    required this.success,
    this.error,
  });
}
```

---

# Destructive Migration Checklist

| Check | Description | Priority |
|-------|-------------|----------|
| **Backup** | Create full backup | Critical |
| **Verify Backup** | Check backup integrity | Critical |
| **Test** | Test on staging | Critical |
| **Dependencies** | Check for dependencies | High |
| **Data Validation** | Validate data before/after | High |
| **Rollback Plan** | Have rollback ready | High |
| **Documentation** | Document the change | Medium |
| **Performance** | Monitor performance | Medium |

---

# Common Mistakes

## Mistake 1: No backup

Wrong:
```dart
// 🚫 Directly dropping table
await db.customSelect('DROP TABLE old_table').go();
```

Correct:
```dart
// ✅ Always backup
await createBackup('old_table');
await db.customSelect('DROP TABLE old_table').go();
```

## Mistake 2: Ignoring dependencies

Wrong:
```dart
// 🚫 Dropping table with foreign keys
await db.customSelect('DROP TABLE users').go();
```

Correct:
```dart
// ✅ Handle dependencies first
await db.customSelect('DROP TABLE user_roles').go();
await db.customSelect('DROP TABLE users').go();
```

## Mistake 3: No rollback plan

Wrong:
```dart
// 🚫 No way to undo
await db.customSelect('DROP TABLE old_table').go();
```

Correct:
```dart
// ✅ Have rollback ready
final backupName = await createBackup('old_table');
await db.customSelect('DROP TABLE old_table').go();
// Can restore from backupName if needed
```

---

# Summary

| Operation | Risk | Mitigation |
|-----------|------|------------|
| **Drop Table** | High | Backup, verify, check dependencies |
| **Remove Column** | High | Backup, verify data |
| **Change Type** | Medium | Verify data, convert safely |
| **Modify Constraint** | Medium | Test thoroughly |
| **Drop Index** | Low | Can be recreated |

---

# Next Steps

Now you understand destructive migrations, let's dive deeper:

- [Testing Migrations](link) – Migration testing strategies
- [Best Practices](link) – Migration best practices
- [Type Converters](link) – Custom type converters

---

# Did You Know?

- **Destructive migrations are permanent** – Without backup

- **Backups are essential** – For recovery

- **Dependencies are important** – Check foreign keys

- **Data validation is key** – Verify before dropping

- **Testing is critical** – Test on staging first

- **Rollback plans are necessary** – Always plan for failure

- **Documentation is important** – Track what changed

- **Destructive migrations are risky** – Handle with care

---

