## GeneratedDatabase

**The core class that powers your Drift database**

---

# What is it?

**GeneratedDatabase** is the base class that Drift generates for your database. It extends `GeneratedDatabase` and provides all the infrastructure for table management, schema versioning, and query execution.

> **Think of GeneratedDatabase as the "engine room"** – it handles all the low-level database operations, manages table references, and provides the foundation for your type-safe queries.

```dart
// When you write this
@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  @override
  int get schemaVersion => 1;
}

// Drift generates this (simplified)
abstract class _$AppDatabase extends GeneratedDatabase {
  _$AppDatabase(QueryExecutor e) : super(e);
  
  // Table references
  late final UsersTable users = UsersTable(this);
  late final PostsTable posts = PostsTable(this);
  
  @override
  int get schemaVersion => 1;
  
  @override
  Set<TableInfo> get allTables => {users, posts};
}
```

> **What's happening here?**
> - **`GeneratedDatabase`** – The base class for all Drift databases
> - **`_$AppDatabase`** – The generated class that extends `GeneratedDatabase`
> - **`allTables`** – Registry of all tables in the database
> - **`schemaVersion`** – Tracks database schema for migrations

---

# Why does it exist?

- **Table Registry** – Maintains a central registry of all tables
- **Schema Management** – Handles schema versioning and migrations
- **Query Execution** – Provides the infrastructure for executing queries
- **Connection Management** – Manages database connections through QueryExecutor
- **Code Organization** – Generated code stays separate from your business logic
- **Type Safety** – Ensures all table operations are type-checked

---

# Key Properties

> **The essential properties you'll work with**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables/users.dart';
import 'tables/posts.dart';

part 'database.g.dart';

@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  // 👇 REQUIRED: Schema version
  @override
  int get schemaVersion => 1;

  // 👇 OPTIONAL: Migration strategy
  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (migrator) async {
      await migrator.createAll();
    },
    onUpgrade: (migrator, from, to) async {
      if (from == 1) {
        // Migrate to version 2
        // await migrator.addColumn(users, users.newColumn);
      }
    },
  );

  // 👇 OPTIONAL: Custom database name (for logging)
  @override
  String get databaseName => 'AppDatabase';

  // 👇 OPTIONAL: Configure before open
  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();
    // Run PRAGMA statements or setup
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **`schemaVersion`** – Must be defined, starts at 1
> - **`migration`** – Handles schema changes between versions
> - **`databaseName`** – Used for logging and debugging
> - **`beforeOpen()`** – Runs before database is opened
> - **`allTables`** – Auto-generated from `@DriftDatabase` annotation

---

# Table Access

> **How to access your tables through GeneratedDatabase**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Generated table references
  // The Users table - accessible as `db.users`
  UsersTable get users => _users;
  
  // The Posts table - accessible as `db.posts`
  PostsTable get posts => _posts;
  
  // 👇 All tables registry
  @override
  Set<TableInfo> get allTables => {users, posts};

  // 👇 Custom query using table reference
  Future<List<User>> getActiveUsers() async {
    return await (select(users)
      ..where((u) => u.isActive.equals(true)))
      .get();
  }

  // 👇 Insert using table reference
  Future<int> createUser(String name, String email) async {
    return await into(users).insert(
      UsersCompanion.insert(
        name: name,
        email: email,
      ),
    );
  }
}

// Usage
final db = AppDatabase();
final allUsers = await db.select(db.users).get();
final user = await db.getUserById(1);
```

> **What's happening here?**
> - **`db.users`** – Access the Users table
> - **`db.posts`** – Access the Posts table
> - **`allTables`** – Registry of all tables (used for migrations)
> - **Table classes** – Generated from your Table definitions

---

# Schema Versioning

> **Managing database schema changes**

```dart
// lib/database/database.dart
@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  // 👇 Start with version 1
  @override
  int get schemaVersion => 3;

  // 👇 Migration strategy
  @override
  MigrationStrategy get migration => MigrationStrategy(
    // Called when database is first created
    onCreate: (migrator) async {
      print('Creating fresh database...');
      await migrator.createAll();
    },
    
    // Called when upgrading from version to version
    onUpgrade: (migrator, from, to) async {
      print('Upgrading from $from to $to');
      
      // Version 1 → 2: Add new column
      if (from == 1 && to == 2) {
        print('Adding category column to posts');
        await migrator.addColumn(posts, posts.category);
      }
      
      // Version 2 → 3: Create new table
      if (from == 2 && to == 3) {
        print('Creating comments table');
        await migrator.createTable(comments);
      }
    },
    
    // Called when downgrading (rare)
    onDowngrade: (migrator, from, to) async {
      print('Downgrading from $from to $to');
      // Handle downgrade or throw error
      throw Exception('Downgrades not supported');
    },
    
    // Called before each migration
    beforeOpen: (details) async {
      print('Opening database version ${details.version}');
    },
  );

  // 👇 Optional: Verify schema after migrations
  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();
    // Run verification queries
    final count = await customSelect('SELECT COUNT(*) FROM users').get().first;
    print('Users count: ${count.data['COUNT(*)']}');
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **`onCreate`** – Runs when database is first created
> - **`onUpgrade`** – Runs when schema version increases
> - **`onDowngrade`** – Runs when schema version decreases
> - **`beforeOpen`** – Runs before database is opened
> - **`migrator`** – Helper for adding columns, tables, indexes

---

# Table Info and Relationships

> **Understanding how tables are registered**

```dart
// lib/database/tables/users.dart
import 'package:drift/drift.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  IntColumn get age => integer()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  
  // 👇 Custom column naming
  @override
  List<String> get customConstraints => [
    'CHECK (age >= 0 AND age <= 150)',
  ];
}

// lib/database/tables/posts.dart
import 'package:drift/drift.dart';
import 'users.dart';

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  IntColumn get userId => integer().references(Users, #id)();
  DateTimeColumn get publishedAt => dateTime().nullable()();
  BoolColumn get isPublished => boolean().withDefault(const Constant(false))();
}

// lib/database/database.dart
@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Access table info
  void inspectTables() {
    print('📊 Table Registry:');
    
    for (final table in allTables) {
      print('  - ${table.actualTableName}');
      print('    Dart class: ${table.runtimeType}');
      
      // Access columns
      final columns = table.columns;
      print('    Columns: ${columns.map((c) => c.name).join(', ')}');
    }
  }

  // 👇 Check if table exists
  Future<bool> tableExists(String tableName) async {
    final result = await customSelect(
      'SELECT name FROM sqlite_master WHERE type="table" AND name=?',
      variables: [Variable.withString(tableName)],
    ).get();
    return result.isNotEmpty;
  }

  // 👇 Get table schema
  Future<List<Map<String, dynamic>>> getTableSchema(String tableName) async {
    final results = await customSelect(
      'PRAGMA table_info($tableName)',
    ).get();
    return results.map((row) => row.data).toList();
  }
}
```

> **What's happening here?**
> - **`allTables`** – Set of all registered tables
> - **`actualTableName`** – Name of the table in SQLite
> - **`columns`** – List of columns in the table
> - **`customConstraints`** – Additional SQL constraints
> - **`references`** – Foreign key relationship

---

# Real-World Example

> **Complete GeneratedDatabase setup with advanced features**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables/users.dart';
import 'tables/posts.dart';
import 'tables/comments.dart';
import 'tables/categories.dart';
import 'tables/tags.dart';
import 'tables/post_tags.dart';

part 'database.g.dart';

@DriftDatabase(
  tables: [
    Users,
    Posts,
    Comments,
    Categories,
    Tags,
    PostTags,
  ],
)
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 5;

  @override
  String get databaseName => 'BlogDatabase';

  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (migrator) async {
      print('📦 Creating database with all tables...');
      await migrator.createAll();
      await insertInitialData();
    },
    
    onUpgrade: (migrator, from, to) async {
      print('🔄 Upgrading database from $from to $to');
      
      // Version 1 → 2: Add posts table
      if (from == 1 && to == 2) {
        print('➕ Creating posts table');
        await migrator.createTable(posts);
      }
      
      // Version 2 → 3: Add comments and categories
      if (from == 2 && to == 3) {
        print('➕ Creating comments table');
        await migrator.createTable(comments);
        print('➕ Creating categories table');
        await migrator.createTable(categories);
        print('➕ Adding category_id to posts');
        await migrator.addColumn(posts, posts.categoryId);
      }
      
      // Version 3 → 4: Add tags
      if (from == 3 && to == 4) {
        print('➕ Creating tags table');
        await migrator.createTable(tags);
        print('➕ Creating post_tags junction table');
        await migrator.createTable(postTags);
      }
      
      // Version 4 → 5: Add indexes
      if (from == 4 && to == 5) {
        print('📇 Adding indexes for performance');
        await migrator.addIndex(
          posts,
          'idx_posts_user_id',
          [posts.userId],
        );
        await migrator.addIndex(
          comments,
          'idx_comments_post_id',
          [comments.postId],
        );
      }
    },
    
    beforeOpen: (details) async {
      print('🗄️ Opening database (version ${details.version})');
      
      // Enable WAL mode for performance
      if (details.wasOpened) {
        await customSelect('PRAGMA optimize').get();
      }
    },
  );

  // 👇 Database initialization
  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();
    
    // Run PRAGMA statements
    await customSelect('PRAGMA foreign_keys = ON').get();
    await customSelect('PRAGMA journal_mode = WAL').get();
  }

  // 👇 Insert initial data
  Future<void> insertInitialData() async {
    print('🌱 Seeding initial data...');
    
    // Insert default categories
    await into(categories).insertAll([
      CategoriesCompanion.insert(name: 'Technology'),
      CategoriesCompanion.insert(name: 'Lifestyle'),
      CategoriesCompanion.insert(name: 'Health'),
    ]);
    
    // Insert admin user
    await into(users).insert(
      UsersCompanion.insert(
        name: 'Admin',
        email: 'admin@example.com',
        age: 30,
        isActive: true,
      ),
    );
  }

  // 👇 Database inspection
  Future<Map<String, dynamic>> getDatabaseInfo() async {
    final tables = allTables.map((t) => t.actualTableName).toList();
    
    final version = await customSelect('PRAGMA user_version').get().first;
    final pageSize = await customSelect('PRAGMA page_size').get().first;
    const journalMode = await customSelect('PRAGMA journal_mode').get().first;
    
    return {
      'tables': tables,
      'version': version.data['user_version'],
      'pageSize': pageSize.data['page_size'],
      'journalMode': journalMode.data['journal_mode'],
      'tableCount': tables.length,
    };
  }

  // 👇 Table operations
  Future<void> vacuumDatabase() async {
    await customSelect('VACUUM').get();
    print('🧹 Database vacuumed');
  }

  Future<void> optimizeDatabase() async {
    await customSelect('PRAGMA optimize').get();
    print('⚡ Database optimized');
  }

  Future<int> getDatabaseSize() async {
    final result = await customSelect(
      'SELECT page_count * page_size as size FROM pragma_page_count(), pragma_page_size()',
    ).get().first;
    return result.data['size'] as int;
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'blog_database',
      native: const DriftNativeOptions(
        databaseDirectory: getApplicationSupportDirectory,
        enableBackgroundIsolate: true,
      ),
      web: DriftWebOptions(
        sqlite3Wasm: Uri.parse('sqlite3.wasm'),
        driftWorker: Uri.parse('drift_worker.js'),
      ),
    );
  }
}
```

```dart
// lib/main.dart - Usage
import 'package:flutter/material.dart';
import 'database/database.dart';

void main() async {
  final db = AppDatabase();
  
  // ✅ Inspect database
  final info = await db.getDatabaseInfo();
  print('Database Info: $info');
  
  // ✅ Check table existence
  final hasUsers = await db.tableExists('users');
  print('Has users table? $hasUsers');
  
  // ✅ Get table schema
  final schema = await db.getTableSchema('users');
  print('Users schema: $schema');
  
  // ✅ Optimize database
  await db.optimizeDatabase();
  
  // ✅ Get database size
  final size = await db.getDatabaseSize();
  print('Database size: ${size / 1024} KB');
  
  // ✅ Vacuum database
  await db.vacuumDatabase();
  
  // ✅ Query using tables
  final users = await db.select(db.users).get();
  print('Total users: ${users.length}');
  
  await db.close();
}
```

> **Benefits of this approach:**
> - **Complete schema management** – Handles all versions
> - **Automated migrations** – Safe schema evolution
> - **Performance optimization** – PRAGMA statements and indexes
> - **Initial data seeding** – Populates database on creation
> - **Database inspection** – Debug and monitor database
> - **Maintenance operations** – Vacuum, optimize, size check

---

# Best Practices

- **Always increment schemaVersion** – When adding/removing columns or tables
- **Use migrations** – Never drop and recreate tables
- **Test migrations** – Use in-memory database for migration testing
- **Add indexes** – For frequently queried columns
- **Enable WAL mode** – For better concurrency
- **Enable foreign keys** – Maintain referential integrity
- **Run PRAGMA optimize** – For performance tuning
- **Use beforeOpen** – Configure database settings
- **Inspect allTables** – For debugging and monitoring
- **Close database properly** – Call close() when done

---

# Common Mistakes

## Mistake 1: Forgetting to increment schemaVersion

Wrong:
```dart
// 🚫 Version stays at 1 after adding columns
@override
int get schemaVersion => 1;
```

Correct:
```dart
// ✅ Increment for schema changes
@override
int get schemaVersion => 2;
```

## Mistake 2: Not handling migrations

Wrong:
```dart
// 🚫 Database crashes on upgrade
@override
MigrationStrategy get migration => MigrationStrategy();
```

Correct:
```dart
// ✅ Handle all migrations
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (migrator, from, to) async {
    if (from == 1) {
      await migrator.addColumn(posts, posts.categoryId);
    }
  },
);
```

## Mistake 3: Creating tables manually

Wrong:
```dart
// 🚫 Manual table creation causes errors
await db.customSelect('CREATE TABLE users (...)').get();
```

Correct:
```dart
// ✅ Use Drift's migrator
await migrator.createAll();
// Or for specific table
await migrator.createTable(users);
```

---

# Summary

| Concept | Purpose | Best Practice |
|---------|---------|---------------|
| **GeneratedDatabase** | Base class for all Drift databases | Let Drift generate it |
| **schemaVersion** | Track schema changes | Increment on changes |
| **migration** | Handle schema evolution | Test migrations thoroughly |
| **allTables** | Registry of all tables | Used by migrations |
| **beforeOpen** | Database setup | Enable WAL, foreign keys |
| **Table access** | `db.tableName` | Use generated getters |

---

# Next Steps

Now you understand GeneratedDatabase, let's dive deeper:

- [Database Connection](link) – Advanced connection options
- [Database Lifecycle](link) – Managing database lifecycle
- [Database Configuration](link) – Fine-tuning your database

---

# Did You Know?

- **GeneratedDatabase creates over 50 methods** – From just your table definitions

- **The `_$AppDatabase` class is fully generated** – You never write it yourself

- **`allTables` is used by Drift's migration system** – It knows which tables to create

- **`schemaVersion` is stored in `PRAGMA user_version`** – Persistent across app restarts

- **You can have multiple databases** – Each with its own GeneratedDatabase subclass

- **Drift's GeneratedDatabase is fully typed** – Table references are type-safe

- **The `beforeOpen` method runs on every open** – Perfect for PRAGMA statements

- **GeneratedDatabase handles connection pooling** – Through QueryExecutor configuration

---

