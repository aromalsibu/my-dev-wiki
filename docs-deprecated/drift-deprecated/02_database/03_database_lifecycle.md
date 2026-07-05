## Database Lifecycle

**Understanding the complete lifecycle of your Drift database**

---

# What is it?

**Database Lifecycle** refers to the entire journey of your database from creation to destruction – including initialization, opening, migration, usage, and proper cleanup. Drift provides hooks and methods to manage each stage of this lifecycle.

> **Think of the Database Lifecycle as the "birth to death" of your database** – it covers everything from the first time your app creates the database file to the moment it's properly closed and cleaned up.

```dart
// The complete database lifecycle
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  // 1️⃣ CREATION: First time database is created
  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (migrator) async {
      await migrator.createAll(); // Tables are created
      await seedInitialData();    // Seed data
    },
  );

  // 2️⃣ OPENING: Every time database opens
  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();
    await customSelect('PRAGMA foreign_keys = ON').get();
  }

  // 3️⃣ USAGE: Query operations
  Future<List<User>> getUsers() async {
    return await select(users).get();
  }

  // 4️⃣ CLOSING: When database is closed
  @override
  Future<void> close() async {
    // Cleanup resources
    await super.close();
  }
}
```

> **What's happening here?**
> - **Creation** – First time database file is created
> - **Opening** – Each time database connection is established
> - **Migration** – Schema changes between versions
> - **Usage** – Queries, inserts, updates, deletes
> - **Closing** – Proper resource cleanup

---

# Why does it exist?

- **Resource Management** – Properly allocate and free resources
- **Schema Evolution** – Handle database changes over time
- **Performance** – Initialize and optimize at the right time
- **Data Integrity** – Ensure database is in a valid state
- **Testing** – Control lifecycle in tests
- **Error Recovery** – Handle failures gracefully

---

# Lifecycle Stages

> **The complete journey of a Drift database**

## Stage 1: Database Creation

> **First time the database file is created**

```dart
// lib/database/database.dart
@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    // 👇 Called when database file doesn't exist
    onCreate: (migrator) async {
      print('📦 Creating fresh database...');
      
      // 1. Create all tables
      await migrator.createAll();
      
      // 2. Create indexes
      await migrator.addIndex(
        users,
        'idx_users_email',
        [users.email],
      );
      
      // 3. Seed initial data
      await seedInitialData();
      
      // 4. Set up triggers (optional)
      await createTriggers();
      
      print('✅ Database creation complete!');
    },
    
    // 👇 Called when opening existing database
    onOpen: (details) async {
      print('📂 Opening existing database');
    },
  );

  // 👇 Seed initial data
  Future<void> seedInitialData() async {
    print('🌱 Seeding initial data...');
    
    // Insert default users
    await into(users).insertAll([
      UsersCompanion.insert(
        name: 'Admin User',
        email: 'admin@example.com',
        age: 30,
        isActive: true,
      ),
      UsersCompanion.insert(
        name: 'Test User',
        email: 'test@example.com',
        age: 25,
        isActive: true,
      ),
    ]);
    
    // Insert default posts
    await into(posts).insertAll([
      PostsCompanion.insert(
        title: 'Welcome Post',
        content: 'Welcome to the app!',
        userId: 1,
        isPublished: true,
        publishedAt: DateTime.now(),
      ),
    ]);
  }

  Future<void> createTriggers() async {
    await customSelect('''
      CREATE TRIGGER update_timestamp 
      AFTER UPDATE ON users 
      FOR EACH ROW 
      BEGIN
        UPDATE users SET updated_at = CURRENT_TIMESTAMP 
        WHERE id = NEW.id;
      END;
    ''').get();
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **`onCreate`** – Runs only when database is first created
> - **`createAll()`** – Creates all tables defined in `@DriftDatabase`
> - **`createTable()`** – Creates a specific table
> - **`addIndex()`** – Creates indexes for performance
> - **`seedInitialData()`** – Populates database with default data
> - **Triggers** – Can be created using raw SQL

---

## Stage 2: Database Opening

> **Each time the database connection is established**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Called BEFORE database is opened
  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();
    
    print('🔓 Preparing to open database...');
    
    // Configure database settings
    await customSelect('PRAGMA foreign_keys = ON').get();
    await customSelect('PRAGMA journal_mode = WAL').get();
    await customSelect('PRAGMA synchronous = NORMAL').get();
    await customSelect('PRAGMA cache_size = -10000').get();
    
    print('✅ Database settings configured');
  }

  // 👇 Called AFTER database is opened
  @override
  MigrationStrategy get migration => MigrationStrategy(
    onOpen: (details) async {
      print('📂 Database opened successfully');
      
      // Perform integrity check
      await checkDatabaseIntegrity();
      
      // Optimize for performance
      await optimizeDatabase();
    },
  );

  // 👇 Check database integrity
  Future<void> checkDatabaseIntegrity() async {
    final result = await customSelect('PRAGMA integrity_check').get();
    final status = result.first.data['integrity_check'] as String;
    
    if (status == 'ok') {
      print('✅ Database integrity check passed');
    } else {
      print('❌ Database integrity check failed: $status');
    }
  }

  // 👇 Optimize database
  Future<void> optimizeDatabase() async {
    print('⚡ Optimizing database...');
    await customSelect('PRAGMA optimize').get();
    await customSelect('PRAGMA wal_checkpoint(TRUNCATE)').get();
    print('✅ Database optimized');
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **`beforeOpen()`** – Runs before connection is established
> - **`onOpen`** – Runs after connection is established
> - **PRAGMA statements** – Configure database behavior
> - **Integrity check** – Verify database is not corrupted
> - **Optimization** – Run `PRAGMA optimize` for better performance
> - **WAL checkpoint** – Clean up WAL file

---

## Stage 3: Database Migration

> **When schema version changes**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  // 👇 Increment version when schema changes
  @override
  int get schemaVersion => 3;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    // 👇 Called when upgrading to newer version
    onUpgrade: (migrator, from, to) async {
      print('🔄 Upgrading database from $from to $to');
      
      try {
        // Version 1 → 2: Add posts table
        if (from == 1 && to == 2) {
          print('➕ Creating posts table');
          await migrator.createTable(posts);
          await migrator.addIndex(
            posts,
            'idx_posts_user_id',
            [posts.userId],
          );
        }
        
        // Version 2 → 3: Add comments and categories
        if (from == 2 && to == 3) {
          print('➕ Creating comments table');
          await migrator.createTable(comments);
          
          print('➕ Creating categories table');
          await migrator.createTable(categories);
          
          print('➕ Adding category_id to posts');
          await migrator.addColumn(posts, posts.categoryId);
          
          print('➕ Adding foreign key constraints');
          await migrator.addForeignKey(
            posts,
            posts.categoryId,
            categories,
            categories.id,
          );
          
          // Migrate existing data
          await migrateExistingData();
        }
        
        print('✅ Migration complete!');
      } catch (e) {
        print('❌ Migration failed: $e');
        rethrow;
      }
    },
    
    // 👇 Called when downgrading to older version
    onDowngrade: (migrator, from, to) async {
      print('⬇️ Downgrading from $from to $to');
      
      // Handle downgrade carefully (data loss possible)
      if (from == 3 && to == 2) {
        print('⚠️ Removing categories and comments');
        await migrator.dropTable(comments);
        await migrator.dropTable(categories);
        await migrator.removeColumn(posts, posts.categoryId);
      }
    },
    
    // 👇 Called before each migration
    beforeOpen: (details) async {
      print('🗄️ Opening database version ${details.version}');
      
      // Backup database before migration
      if (details.version != schemaVersion) {
        await backupDatabase(details.version);
      }
    },
    
    // 👇 Called after migration
    onOpen: (details) async {
      print('✅ Database ready (version ${details.version})');
    },
  );

  // 👇 Migrate existing data
  Future<void> migrateExistingData() async {
    print('📊 Migrating existing data...');
    
    // Set default category for existing posts
    final defaultCategory = await into(categories).insert(
      CategoriesCompanion.insert(name: 'General'),
    );
    
    await (update(posts)..where((p) => p.categoryId.isNull()))
      .write(PostsCompanion(categoryId: Value(defaultCategory)));
    
    print('✅ Data migration complete');
  }

  // 👇 Backup database
  Future<void> backupDatabase(int fromVersion) async {
    print('💾 Creating backup before migration...');
    
    final dir = await getApplicationDocumentsDirectory();
    final backupPath = p.join(
      dir.path,
      'backup_v${fromVersion}_${DateTime.now().millisecondsSinceEpoch}.db',
    );
    
    await customSelect(
      'VACUUM INTO ?',
      variables: [Variable.withString(backupPath)],
    ).get();
    
    print('✅ Backup created at: $backupPath');
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **`onUpgrade`** – Called when increasing schema version
> - **`onDowngrade`** – Called when decreasing schema version
> - **`createTable()`** – Add new tables during migration
> - **`addColumn()`** – Add new columns to existing tables
> - **`dropTable()`** – Remove tables (use with caution)
> - **`removeColumn()`** – Remove columns (use with caution)
> - **`addForeignKey()`** – Add foreign key constraints
> - **`addIndex()`** – Add indexes for performance

---

## Stage 4: Database Usage

> **Normal database operations during app runtime**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Query operations
  Future<List<User>> getActiveUsers() async {
    return await (select(users)
      ..where((u) => u.isActive.equals(true)))
      .get();
  }

  // 👇 Insert operations
  Future<int> createUser(String name, String email) async {
    return await into(users).insert(
      UsersCompanion.insert(
        name: name,
        email: email,
        age: 25,
        isActive: true,
      ),
    );
  }

  // 👇 Update operations
  Future<void> updateUser(int id, String name) async {
    await (update(users)..where((u) => u.id.equals(id)))
      .write(UsersCompanion(name: Value(name)));
  }

  // 👇 Delete operations
  Future<void> deleteUser(int id) async {
    await (delete(users)..where((u) => u.id.equals(id))).go();
  }

  // 👇 Transaction operations
  Future<void> transferData(int fromUserId, int toUserId) async {
    await transaction(() async {
      // All operations in one transaction
      final fromUser = await getUserById(fromUserId);
      final toUser = await getUserById(toUserId);
      
      // Transfer data...
      await updateUser(fromUserId, 'Updated Name');
      await updateUser(toUserId, 'Another Name');
    });
  }

  // 👇 Reactive streams
  Stream<List<User>> watchActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .watch();
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **Queries** – SELECT operations with type safety
> - **Inserts** – Add new rows to tables
> - **Updates** – Modify existing rows
> - **Deletes** – Remove rows from tables
> - **Transactions** – Group operations atomically
> - **Streams** – Reactive query results

---

## Stage 5: Database Closing

> **Properly closing the database connection**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Called when database is closed
  @override
  Future<void> close() async {
    print('🔒 Closing database...');
    
    try {
      // 1. Flush any pending changes
      await customSelect('PRAGMA wal_checkpoint(TRUNCATE)').get();
      
      // 2. Vacuum database (optional)
      // await customSelect('VACUUM').get();
      
      // 3. Close connection
      await super.close();
      
      print('✅ Database closed successfully');
    } catch (e) {
      print('❌ Error closing database: $e');
      rethrow;
    }
  }

  // 👇 Custom cleanup method
  Future<void> cleanupAndClose() async {
    print('🧹 Cleaning up before closing...');
    
    // Clear any caches
    _cache.clear();
    
    // Close active streams
    _streamSubscriptions.forEach((sub) => sub.cancel());
    _streamSubscriptions.clear();
    
    // Then close
    await close();
  }

  // 👇 Force close (ignore errors)
  Future<void> forceClose() async {
    print('💥 Force closing database...');
    try {
      await close();
    } catch (e) {
      print('Error while closing: $e');
      // Ignore errors
    }
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **WAL checkpoint** – Flush WAL file to database
> - **VACUUM** – Rebuild database to free space (optional)
> - **Cancel streams** – Prevent memory leaks
> - **Clear caches** – Free memory
> - **Error handling** – Handle closing errors gracefully
> - **Force close** – Ignore errors if needed

---

# Lifecycle Management

> **Complete lifecycle management in your app**

```dart
// lib/services/database_service.dart
import 'package:flutter/widgets.dart';
import '../database/database.dart';

class DatabaseService extends ChangeNotifier {
  static final DatabaseService _instance = DatabaseService._internal();
  factory DatabaseService() => _instance;
  DatabaseService._internal();

  AppDatabase? _database;
  bool _isInitialized = false;
  bool _isOpen = false;

  // 👇 Database lifecycle states
  enum LifecycleState {
    uninitialized,
    initializing,
    initialized,
    opening,
    open,
    closing,
    closed,
    error,
  }

  LifecycleState _state = LifecycleState.uninitialized;
  LifecycleState get state => _state;

  // 👇 Initialize database
  Future<void> initialize() async {
    if (_isInitialized) return;
    
    _state = LifecycleState.initializing;
    notifyListeners();
    
    try {
      _database = AppDatabase();
      _isInitialized = true;
      _state = LifecycleState.initialized;
      notifyListeners();
      
      print('✅ Database service initialized');
    } catch (e) {
      _state = LifecycleState.error;
      notifyListeners();
      print('❌ Failed to initialize: $e');
      rethrow;
    }
  }

  // 👇 Open database (explicit)
  Future<void> open() async {
    if (!_isInitialized) await initialize();
    if (_isOpen) return;
    
    _state = LifecycleState.opening;
    notifyListeners();
    
    try {
      // Ensure database is open by performing a simple query
      await _database!.customSelect('SELECT 1').get();
      _isOpen = true;
      _state = LifecycleState.open;
      notifyListeners();
      
      print('✅ Database opened');
    } catch (e) {
      _state = LifecycleState.error;
      notifyListeners();
      print('❌ Failed to open: $e');
      rethrow;
    }
  }

  // 👇 Get database (lazy open)
  Future<AppDatabase> getDatabase() async {
    if (!_isInitialized) await initialize();
    if (!_isOpen) await open();
    return _database!;
  }

  // 👇 Health check
  Future<bool> isHealthy() async {
    try {
      final db = await getDatabase();
      await db.customSelect('SELECT 1').get();
      return true;
    } catch (e) {
      print('❌ Health check failed: $e');
      return false;
    }
  }

  // 👇 Close database
  Future<void> close() async {
    if (!_isOpen || _database == null) return;
    
    _state = LifecycleState.closing;
    notifyListeners();
    
    try {
      await _database!.close();
      _isOpen = false;
      _state = LifecycleState.closed;
      notifyListeners();
      
      print('✅ Database closed');
    } catch (e) {
      _state = LifecycleState.error;
      notifyListeners();
      print('❌ Failed to close: $e');
      rethrow;
    }
  }

  // 👇 Reset (for testing)
  Future<void> reset() async {
    await close();
    _database = null;
    _isInitialized = false;
    _isOpen = false;
    _state = LifecycleState.uninitialized;
    notifyListeners();
  }

  @override
  void dispose() {
    close();
    super.dispose();
  }
}
```

```dart
// lib/widgets/database_widget.dart
import 'package:flutter/material.dart';
import '../services/database_service.dart';

class DatabaseWidget extends StatefulWidget {
  final Widget child;
  
  const DatabaseWidget({required this.child});
  
  @override
  _DatabaseWidgetState createState() => _DatabaseWidgetState();
}

class _DatabaseWidgetState extends State<DatabaseWidget> {
  final DatabaseService _dbService = DatabaseService();

  @override
  void initState() {
    super.initState();
    _initializeDatabase();
  }

  Future<void> _initializeDatabase() async {
    try {
      await _dbService.initialize();
      
      // Check if database is healthy
      final healthy = await _dbService.isHealthy();
      if (!healthy) {
        print('⚠️ Database health check failed');
      }
    } catch (e) {
      print('❌ Database initialization failed: $e');
      // Show error UI
      setState(() {});
    }
  }

  @override
  void dispose() {
    _dbService.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ListenableBuilder(
      listenable: _dbService,
      builder: (context, child) {
        final state = _dbService.state;
        
        if (state == DatabaseService.LifecycleState.error) {
          return Scaffold(
            body: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.error, size: 64, color: Colors.red),
                  SizedBox(height: 16),
                  Text('Database Error'),
                  SizedBox(height: 8),
                  ElevatedButton(
                    onPressed: _initializeDatabase,
                    child: Text('Retry'),
                  ),
                ],
              ),
            ),
          );
        }
        
        if (state == DatabaseService.LifecycleState.initializing ||
            state == DatabaseService.LifecycleState.opening) {
          return Scaffold(
            body: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  CircularProgressIndicator(),
                  SizedBox(height: 16),
                  Text('Initializing database...'),
                ],
              ),
            ),
          );
        }
        
        return widget.child;
      },
    );
  }
}
```

> **Benefits of this approach:**
> - **Centralized management** – One service handles all lifecycle events
> - **State tracking** – Know exactly what state the database is in
> - **Error handling** – Graceful error recovery
> - **Lazy opening** – Open only when needed
> - **Proper cleanup** – Always close when done
> - **Testing support** – Reset for tests

---

# Best Practices

- **Always handle onCreate** – Seed initial data
- **Run PRAGMA statements in beforeOpen** – Configure database
- **Use migrations for schema changes** – Never drop and recreate
- **Test migrations thoroughly** – Use in-memory database for tests
- **Check integrity on open** – Detect corruption early
- **Optimize on open** – Run `PRAGMA optimize`
- **Close database on app exit** – Prevent resource leaks
- **Handle errors gracefully** – Show user-friendly messages
- **Use transactions for multiple operations** – Maintain consistency
- **Monitor lifecycle states** – Debug and log events

---

# Common Mistakes

## Mistake 1: Not handling onCreate

Wrong:
```dart
// 🚫 Database created but empty
@override
MigrationStrategy get migration => MigrationStrategy();
```

Correct:
```dart
// ✅ Create tables and seed data
@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (migrator) async {
    await migrator.createAll();
    await seedInitialData();
  },
);
```

## Mistake 2: Not closing database

Wrong:
```dart
// 🚫 Resource leak
void main() {
  final db = AppDatabase();
  // Never closed
}
```

Correct:
```dart
// ✅ Always close
void main() async {
  final db = AppDatabase();
  try {
    // Use database
  } finally {
    await db.close();
  }
}
```

## Mistake 3: Not handling migrations

Wrong:
```dart
// 🚫 App crashes on version change
@override
int get schemaVersion => 2;
// No migration strategy
```

Correct:
```dart
// ✅ Handle all version changes
@override
int get schemaVersion => 2;

@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (migrator, from, to) async {
    if (from == 1) {
      await migrator.addColumn(users, users.newColumn);
    }
  },
);
```

---

# Summary

| Lifecycle Stage | Key Methods | Purpose |
|-----------------|-------------|---------|
| **Creation** | `onCreate`, `createAll()` | First time setup |
| **Opening** | `beforeOpen`, `onOpen` | Configure and initialize |
| **Migration** | `onUpgrade`, `onDowngrade` | Schema changes |
| **Usage** | Queries, inserts, updates | Normal operations |
| **Closing** | `close()` | Resource cleanup |

---

# Next Steps

Now you understand the database lifecycle, let's dive deeper:

- [Opening the Database](link) – Manual opening strategies
- [Database Configuration](link) – Fine-tuning settings
- [Multiple Databases](link) – Working with multiple databases

---

# Did You Know?

- **Drift automatically creates your database file** – When first accessed

- **`onCreate` only runs once** – When the database file is first created

- **`beforeOpen` runs every time** – Including after onCreate

- **Migrations are atomic** – They run in a transaction

- **You can test migrations** – Using in-memory database with versions

- **WAL checkpoint flushes** – Moves data from WAL file to main database

- **PRAGMA optimize** – Analyzes database for better performance

- **Integrity check** – Verifies database is not corrupted

---

