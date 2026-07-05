## Database Connection

**Understanding how Drift connects to your database**

---

# What is it?

**Database Connection** in Drift is the bridge between your Dart code and the actual SQLite database file. It's managed through the `QueryExecutor` interface, which handles opening, closing, and executing queries on the database.

> **Think of the Database Connection as the "phone line"** – it's the communication channel between your app and the database file, handling all the low-level details of reading and writing data.

```dart
// The connection flow in Drift
// 1. Define your QueryExecutor
QueryExecutor createExecutor() {
  return driftDatabase(name: 'my_app');
}

// 2. Pass it to your database
final db = AppDatabase(createExecutor());

// 3. Database opens connection when needed
final users = await db.select(db.users).get(); // Connection opens here

// 4. Close when done
await db.close(); // Connection closes
```

> **What's happening here?**
> - **QueryExecutor** – Interface that handles all database communication
> - **LazyDatabase** – Opens connection only on first use
> - **NativeDatabase** – Implementation using `dart:ffi` and SQLite
> - **Connection lifecycle** – Open → Use → Close

---

# Why does it exist?

- **Resource Management** – Properly open and close database connections
- **Platform Abstraction** – Same API works on all platforms
- **Performance** – Connection pooling for concurrent access
- **Background Processing** – Move database work off UI thread
- **Error Handling** – Graceful handling of connection errors
- **Testing** – Swap real connections with in-memory ones

---

# Connection Types

> **Different ways to connect to your database**

## 1. NativeDatabase (Default)
```dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

QueryExecutor createNativeConnection() {
  return LazyDatabase(() async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File(p.join(dir.path, 'app.db'));
    
    final db = await databaseFactoryIo.openDatabase(file.path);
    return NativeDatabase(db);
  });
}
```

## 2. NativeDatabase with Background Isolate
```dart
QueryExecutor createBackgroundConnection() {
  return NativeDatabase.createInBackground(
    File(p.join(getApplicationDocumentsDirectory().path, 'app.db')),
    setup: (db) async {
      await db.execute('PRAGMA journal_mode = WAL');
      await db.execute('PRAGMA foreign_keys = ON');
    },
    readPool: 4, // 4 read-only isolates
  );
}
```

## 3. In-Memory Database (Testing)
```dart
QueryExecutor createMemoryConnection() {
  return NativeDatabase.memory();
}

// Usage in tests
final db = AppDatabase(NativeDatabase.memory());
```

## 4. Web Database
```dart
import 'package:drift/wasm.dart';

QueryExecutor createWebConnection() {
  return driftDatabase(
    name: 'app_db',
    web: DriftWebOptions(
      sqlite3Wasm: Uri.parse('sqlite3.wasm'),
      driftWorker: Uri.parse('drift_worker.js'),
    ),
  );
}
```

## 5. Multiple Connections (Read/Write Split)
```dart
class DatabaseConnections {
  static QueryExecutor createWriteConnection() {
    return NativeDatabase.createInBackground(
      File('app.db'),
      setup: (db) async {
        await db.execute('PRAGMA journal_mode = WAL');
        await db.execute('PRAGMA synchronous = FULL');
      },
    );
  }

  static QueryExecutor createReadConnection() {
    return NativeDatabase(
      File('app.db'),
      openReadOnly: true,
    );
  }
}

// Usage
class AppDatabase extends _$AppDatabase {
  AppDatabase({required QueryExecutor read, required QueryExecutor write}) 
    : super(write, readConnection: read);
  
  @override
  int get schemaVersion => 1;
}

final db = AppDatabase(
  read: DatabaseConnections.createReadConnection(),
  write: DatabaseConnections.createWriteConnection(),
);
```

> **Key insights:**
> - **LazyDatabase** – Opens connection only when first used
> - **Background Isolate** – Moves database work off UI thread
> - **readPool** – Creates multiple read-only isolates for parallel queries
> - **In-Memory** – Perfect for tests, data is lost on close
> - **Web** – Uses WebAssembly SQLite
> - **Read/Write Split** – Separate connections for reads and writes

---

# Connection Configuration

> **Fine-tuning your database connection**

```dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';
import 'package:sqlite3_flutter_libs/sqlite3_flutter_libs.dart';

QueryExecutor createOptimizedConnection() {
  return LazyDatabase(() async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File(p.join(dir.path, 'optimized.db'));
    
    // Ensure directory exists
    await file.parent.create(recursive: true);
    
    // Open with custom options
    final db = await databaseFactoryIo.openDatabase(
      file.path,
      options: OpenDatabaseOptions(
        readOnly: false,
        // 👇 Setup runs after database is opened
        setup: (db) async {
          // Performance optimizations
          await db.execute('PRAGMA journal_mode = WAL');
          await db.execute('PRAGMA synchronous = NORMAL');
          await db.execute('PRAGMA foreign_keys = ON');
          await db.execute('PRAGMA cache_size = 10000');
          await db.execute('PRAGMA temp_store = MEMORY');
          await db.execute('PRAGMA mmap_size = 268435456'); // 256MB
          
          // Enable foreign key constraints
          await db.execute('PRAGMA foreign_keys = ON');
          
          // Optimize for speed
          await db.execute('PRAGMA journal_size_limit = 67108864'); // 64MB
        },
      ),
    );
    
    return NativeDatabase(db);
  });
}
```

> **What's happening here?**
> - **WAL Mode** – Write-Ahead Logging for better concurrency
> - **Synchronous NORMAL** – Balanced performance vs durability
> - **Foreign Keys** – Enforce referential integrity
> - **Cache Size** – More memory for better performance
> - **Memory Mapped** – Faster reads with mmap
> - **Journal Limit** – Prevents journal file from growing too large

---

# Connection Pooling

> **Managing multiple connections for performance**

```dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';

class ConnectionPool {
  static final ConnectionPool _instance = ConnectionPool._internal();
  factory ConnectionPool() => _instance;
  ConnectionPool._internal();

  QueryExecutor? _writeExecutor;
  QueryExecutor? _readExecutor;
  
  // Write connection (single)
  QueryExecutor get writeExecutor {
    _writeExecutor ??= NativeDatabase.createInBackground(
      File('app.db'),
      setup: (db) async {
        await db.execute('PRAGMA journal_mode = WAL');
        await db.execute('PRAGMA synchronous = NORMAL');
      },
      readPool: 1, // Only one write connection
    );
    return _writeExecutor!;
  }

  // Read connections (pool)
  QueryExecutor get readExecutor {
    _readExecutor ??= NativeDatabase.createInBackground(
      File('app.db'),
      setup: (db) async {
        await db.execute('PRAGMA journal_mode = WAL');
        await db.execute('PRAGMA synchronous = NORMAL');
        await db.execute('PRAGMA query_only = ON'); // Read-only
      },
      readPool: 4, // 4 read connections
    );
    return _readExecutor!;
  }
}

// Usage
class AppDatabase extends _$AppDatabase {
  AppDatabase()
      : super(
          ConnectionPool().writeExecutor,
          readConnection: ConnectionPool().readExecutor,
        );

  @override
  int get schemaVersion => 1;
}

final db = AppDatabase();

// Write operations use write connection
Future<void> insertUser() async {
  await db.into(db.users).insert(...);
}

// Read operations use read connection pool
Future<List<User>> getUsers() async {
  return await db.select(db.users).get(); // Uses read pool
}
```

> **Key insights:**
> - **Write connection** – Single connection for writes (prevents conflicts)
> - **Read pool** – Multiple connections for parallel reads
> - **Query only** – Read-only mode for read connections
> - **WAL mode** – Required for concurrent access
> - **Performance** – Read connections don't block writes

---

# Connection Lifecycle

> **Managing the lifecycle of your database connection**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'my_app',
      native: const DriftNativeOptions(
        enableBackgroundIsolate: true,
      ),
    );
  }

  // 👇 Connection state tracking
  bool _isOpen = false;
  bool get isOpen => _isOpen;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();
    _isOpen = true;
    print('🔓 Database connection opened');
  }

  // 👇 Custom open method
  Future<void> ensureOpen() async {
    if (!isOpen) {
      await beforeOpen();
    }
  }

  // 👇 Custom close with cleanup
  @override
  Future<void> close() async {
    _isOpen = false;
    await super.close();
    print('🔒 Database connection closed');
  }
}

// lib/main.dart - Managed lifecycle
void main() async {
  // 1. Create database
  final db = AppDatabase();
  
  // 2. Ensure it's open (optional - lazy opening works too)
  await db.ensureOpen();
  
  try {
    // 3. Use database
    final users = await db.select(db.users).get();
    print('Users: ${users.length}');
    
    // 4. More operations...
    await db.into(db.users).insert(...);
    
  } catch (e) {
    print('Database error: $e');
  } finally {
    // 5. Always close when done
    await db.close();
  }
}
```

> **What's happening here?**
> - **Lazy opening** – Connection opens on first query
> - **Manual opening** – Call `ensureOpen()` to pre-open
> - **State tracking** – Track if connection is open
> - **Clean close** – Always close database when done
> - **Error handling** – Catch and handle database errors

---

# Connection Monitoring

> **Debugging and monitoring your connections**

```dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';

class MonitoredDatabase extends GeneratedDatabase {
  MonitoredDatabase(QueryExecutor e) : super(e);
  
  @override
  int get schemaVersion => 1;
  
  @override
  String get databaseName => 'MonitoredDB';
}

class LoggingExecutor extends QueryExecutor {
  final QueryExecutor _base;
  int _queryCount = 0;
  int _errorCount = 0;
  Duration _totalTime = Duration.zero;
  
  LoggingExecutor(this._base);

  @override
  Future<T> run<T>(QueryContext<T> context) async {
    _queryCount++;
    
    final start = DateTime.now();
    print('🔍 Query ${_queryCount}: ${context.sql}');
    print('📝 Args: ${context.arguments}');
    
    try {
      final result = await _base.run(context);
      final duration = DateTime.now().difference(start);
      _totalTime += duration;
      
      print('✅ Completed in ${duration.inMilliseconds}ms');
      return result;
    } catch (e) {
      _errorCount++;
      print('❌ Query failed: $e');
      rethrow;
    }
  }

  // 👇 Required overrides
  @override
  Future<void> close() => _base.close();
  
  @override
  Future<int> delete(String table, Insertable entity) => _base.delete(table, entity);
  
  @override
  Future<int> insert(String table, Insertable entity, {InsertMode mode = InsertMode.insert}) =>
      _base.insert(table, entity, mode: mode);
  
  @override
  Future<int> update(String table, Insertable entity) => _base.update(table, entity);
  
  @override
  Future<List<Map<String, Object?>>> query(String sql, [List<Object?> arguments = const []]) =>
      _base.query(sql, arguments);

  // 👇 Statistics
  Map<String, dynamic> getStats() {
    return {
      'totalQueries': _queryCount,
      'errors': _errorCount,
      'totalTime': _totalTime.inMilliseconds,
      'avgTime': _queryCount > 0 ? (_totalTime.inMilliseconds / _queryCount).round() : 0,
    };
  }
}

// Usage
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    final base = driftDatabase(name: 'my_app');
    return LoggingExecutor(base); // 👈 Wrap with logger
  }
}

// Get stats
final db = AppDatabase();
final stats = (db.executor as LoggingExecutor).getStats();
print('Database stats: $stats');
```

> **Key insights:**
> - **Query logging** – Track all SQL queries
> - **Performance monitoring** – Measure query times
> - **Error tracking** – Count failed queries
> - **Statistics** – Get insights into database usage
> - **Debugging** – Identify slow queries

---

# Real-World Example

> **Complete production-ready connection setup**

```dart
// lib/database/connection.dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';
import 'package:sqlite3_flutter_libs/sqlite3_flutter_libs.dart';
import 'dart:developer' as dev;

class DatabaseConnectionManager {
  static final DatabaseConnectionManager _instance = DatabaseConnectionManager._internal();
  factory DatabaseConnectionManager() => _instance;
  DatabaseConnectionManager._internal();

  AppDatabase? _database;
  bool _isInitialized = false;

  Future<AppDatabase> get database async {
    if (_database == null) {
      await initialize();
    }
    return _database!;
  }

  Future<void> initialize() async {
    if (_isInitialized) return;
    
    dev.log('Initializing database connection...');
    
    final executor = await _createOptimizedConnection();
    _database = AppDatabase(executor);
    _isInitialized = true;
    
    dev.log('Database connection initialized');
  }

  Future<QueryExecutor> _createOptimizedConnection() async {
    return LazyDatabase(() async {
      // Get documents directory
      final dir = await getApplicationDocumentsDirectory();
      final path = p.join(dir.path, 'production_app.db');
      
      dev.log('Opening database at: $path');
      
      // Open with production settings
      final db = await databaseFactoryIo.openDatabase(
        path,
        options: OpenDatabaseOptions(
          setup: (db) async {
            // 👇 Performance settings
            await db.execute('PRAGMA journal_mode = WAL');
            await db.execute('PRAGMA synchronous = NORMAL');
            await db.execute('PRAGMA foreign_keys = ON');
            await db.execute('PRAGMA cache_size = -10000'); // 10MB
            await db.execute('PRAGMA temp_store = MEMORY');
            await db.execute('PRAGMA mmap_size = 134217728'); // 128MB
            
            // 👇 Optimize for specific use cases
            await db.execute('PRAGMA page_size = 4096');
            await db.execute('PRAGMA journal_size_limit = 33554432'); // 32MB
            await db.execute('PRAGMA auto_vacuum = INCREMENTAL');
            
            dev.log('✅ Database optimized for production');
          },
        ),
      );
      
      return NativeDatabase(db);
    });
  }

  Future<void> closeConnection() async {
    if (_database != null) {
      dev.log('Closing database connection...');
      await _database!.close();
      _database = null;
      _isInitialized = false;
      dev.log('✅ Database connection closed');
    }
  }
}

// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';
import 'tables/users.dart';
import 'tables/posts.dart';

part 'database.g.dart';

@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase(QueryExecutor e) : super(e);

  @override
  int get schemaVersion => 2;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (migrator) async {
      await migrator.createAll();
      dev.log('📦 Database created');
    },
    onUpgrade: (migrator, from, to) async {
      dev.log('🔄 Upgrading from $from to $to');
      if (from == 1 && to == 2) {
        await migrator.addColumn(posts, posts.categoryId);
        dev.log('✅ Migration complete');
      }
    },
  );

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();
    dev.log('🔓 Database opening...');
  }

  @override
  Future<void> close() async {
    dev.log('🔒 Database closing...');
    await super.close();
  }

  // 👇 Health check
  Future<bool> healthCheck() async {
    try {
      await customSelect('SELECT 1').get();
      return true;
    } catch (e) {
      dev.log('❌ Health check failed: $e');
      return false;
    }
  }
}

// lib/main.dart
import 'package:flutter/material.dart';
import 'database/connection.dart';
import 'database/database.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize database connection
  await DatabaseConnectionManager().initialize();
  
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  late Future<AppDatabase> _dbFuture;
  bool _isConnected = false;

  @override
  void initState() {
    super.initState();
    _dbFuture = DatabaseConnectionManager().database;
    _checkConnection();
  }

  Future<void> _checkConnection() async {
    final db = await _dbFuture;
    final healthy = await db.healthCheck();
    setState(() {
      _isConnected = healthy;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Database Demo'),
        actions: [
          Icon(
            _isConnected ? Icons.check_circle : Icons.error,
            color: _isConnected ? Colors.green : Colors.red,
          ),
        ],
      ),
      body: FutureBuilder<AppDatabase>(
        future: _dbFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(child: CircularProgressIndicator());
          }
          
          if (snapshot.hasError) {
            return Center(child: Text('Error: ${snapshot.error}'));
          }
          
          final db = snapshot.data!;
          
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Text('Connection: ${_isConnected ? '✅ Connected' : '❌ Disconnected'}'),
                ElevatedButton(
                  onPressed: () async {
                    await db.into(db.users).insert(
                      UsersCompanion.insert(
                        name: 'Test User',
                        email: 'test@example.com',
                      ),
                    );
                    
                    final users = await db.select(db.users).get();
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(
                        content: Text('Added user. Total: ${users.length}'),
                      ),
                    );
                  },
                  child: Text('Add User'),
                ),
              ],
            ),
          );
        },
      ),
    );
  }

  @override
  void dispose() {
    // Close connection when app exits
    DatabaseConnectionManager().closeConnection();
    super.dispose();
  }
}
```

> **Benefits of this approach:**
> - **Centralized management** – Single source of truth for connections
> - **Optimized settings** – Production-ready configuration
> - **Health monitoring** – Check connection health
> - **Clean lifecycle** – Proper open and close
> - **Error handling** – Graceful failure handling
> - **Development logging** – Track database operations
> - **Platform-aware** – Works on all platforms

---

# Best Practices

- **Use LazyDatabase** – Open connection only when needed
- **Enable WAL mode** – Required for concurrent access
- **Configure cache size** – Improve performance
- **Enable foreign keys** – Maintain data integrity
- **Close connections** – Always close when done
- **Use connection pooling** – For high-performance apps
- **Monitor connections** – Track open/close events
- **Handle errors gracefully** – Connection errors can happen
- **Test with in-memory** – Use for unit tests
- **Optimize settings** – For your specific use case

---

# Common Mistakes

## Mistake 1: Not closing connections

Wrong:
```dart
// 🚫 Memory leak
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

## Mistake 2: Not enabling WAL mode with isolates

Wrong:
```dart
// 🚫 "Database is locked" errors
NativeDatabase.createInBackground(File('app.db'));
```

Correct:
```dart
// ✅ Enable WAL for concurrent access
NativeDatabase.createInBackground(
  File('app.db'),
  setup: (db) async {
    await db.execute('PRAGMA journal_mode = WAL');
  },
);
```

## Mistake 3: Using read pool without WAL

Wrong:
```dart
// 🚫 readPool requires WAL mode
NativeDatabase.createInBackground(
  File('app.db'),
  readPool: 4, // Will fail without WAL
);
```

Correct:
```dart
// ✅ Always enable WAL with readPool
NativeDatabase.createInBackground(
  File('app.db'),
  setup: (db) async {
    await db.execute('PRAGMA journal_mode = WAL');
  },
  readPool: 4,
);
```

---

# Summary

| Concept | Purpose | Best Practice |
|---------|---------|---------------|
| **QueryExecutor** | Database connection interface | Use `driftDatabase()` |
| **LazyDatabase** | Deferred connection opening | Open only when needed |
| **NativeDatabase** | SQLite implementation | Use with background isolate |
| **WAL Mode** | Concurrent access | Always enable |
| **Connection Pool** | Multiple connections | Read/write split |
| **In-Memory** | Testing | Use `NativeDatabase.memory()` |

---

# Next Steps

Now you understand database connections, let's dive deeper:

- [Database Lifecycle](link) – Managing database lifecycle
- [Opening the Database](link) – Manual opening strategies
- [Database Configuration](link) – Fine-tuning settings

---

# Did You Know?

- **Drift uses `dart:ffi`** – For native SQLite performance

- **WAL mode enables concurrent reads and writes** – Without blocking

- **`readPool: 4` creates 5 isolates total** – 1 write + 4 read

- **In-memory databases are fast** – No disk I/O, perfect for tests

- **Drift supports connection interceptors** – For logging and monitoring

- **You can open databases in read-only mode** – For safety

- **Connection errors are recoverable** – You can retry operations

- **Drift handles connection pooling automatically** – When using `readPool`

---
