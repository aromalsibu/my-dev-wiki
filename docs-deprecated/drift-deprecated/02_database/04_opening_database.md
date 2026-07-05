## Opening the Database

**Different strategies for opening your Drift database**

---

# What is it?

**Opening the Database** refers to the process of establishing a connection to your SQLite database file. Drift provides multiple ways to open databases – from lazy opening on first use to explicit pre-opening during app startup.

> **Think of opening a database like "picking up the phone"** – it's establishing the communication channel so you can start making calls (queries).

```dart
// Different ways to open a database

// 1️⃣ Lazy opening (default)
final db = AppDatabase(); // No connection yet
final users = await db.select(db.users).get(); // Opens here

// 2️⃣ Explicit opening
final db = AppDatabase();
await db.ensureOpen(); // Opens immediately

// 3️⃣ Pre-opening on startup
void main() async {
  final db = AppDatabase();
  await db.open(); // Open before app starts
  runApp(MyApp(db: db));
}
```

> **What's happening here?**
> - **Lazy opening** – Connection opens on first query
> - **Explicit opening** – Manually trigger connection
> - **Pre-opening** – Open during app initialization
> - **Connection state** – Track whether database is open

---

# Why does it exist?

- **Performance** – Open connections only when needed
- **Startup Speed** – Defer database work for faster app launch
- **Resource Management** – Control when resources are allocated
- **User Experience** – Avoid UI jank by opening in background
- **Testing** – Control database state in tests
- **Error Handling** – Catch connection errors early

---

# Opening Strategies

> **Different ways to open your database**

## Strategy 1: Lazy Opening (Default)

> **Database opens on first query**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    // 👇 LazyDatabase - opens on first use
    return driftDatabase(
      name: 'my_app',
      native: const DriftNativeOptions(
        enableBackgroundIsolate: true,
      ),
    );
  }
}

// Usage
void main() async {
  final db = AppDatabase();
  
  // ✅ Database is NOT open yet
  
  // 👇 This query triggers the connection
  final users = await db.select(db.users).get();
  
  // ✅ Database is now open
}
```

> **Key insights:**
> - **LazyDatabase** – Wraps connection and opens on first use
> - **First query triggers open** – Database opens when needed
> - **No explicit open required** – Just use the database
> - **Good for simple apps** – No complexity needed

---

## Strategy 2: Explicit Opening

> **Manually control when database opens**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Track opening state
  bool _isOpen = false;
  bool get isOpen => _isOpen;

  // 👇 Explicit open method
  Future<void> open() async {
    if (_isOpen) return;
    
    print('🔓 Opening database...');
    
    try {
      // 👇 Force connection by making a simple query
      await customSelect('SELECT 1').get();
      _isOpen = true;
      print('✅ Database opened successfully');
    } catch (e) {
      print('❌ Failed to open database: $e');
      rethrow;
    }
  }

  // 👇 Ensure open (open if not already)
  Future<void> ensureOpen() async {
    if (!_isOpen) {
      await open();
    }
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}

// Usage
void main() async {
  final db = AppDatabase();
  
  // 👇 Explicitly open database
  await db.open();
  
  // ✅ Database is now open
  // All queries will work immediately
  final users = await db.select(db.users).get();
}
```

> **Key insights:**
> - **`open()`** – Explicitly opens the connection
> - **`ensureOpen()`** – Opens if not already open
> - **`isOpen`** – Track connection state
> - **Health check** – `SELECT 1` validates connection
> - **Error handling** – Catch opening errors

---

## Strategy 3: Pre-Opening on Startup

> **Open database before app starts**

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'database/database.dart';

void main() async {
  // 👇 Ensure Flutter is initialized
  WidgetsFlutterBinding.ensureInitialized();
  
  // 👇 Create and open database before app starts
  final db = AppDatabase();
  await db.open();
  
  // 👇 Run app with database
  runApp(MyApp(db: db));
}

class MyApp extends StatelessWidget {
  final AppDatabase db;
  
  const MyApp({required this.db});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: HomePage(db: db),
    );
  }
}

class HomePage extends StatelessWidget {
  final AppDatabase db;
  
  const HomePage({required this.db});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('App')),
      body: FutureBuilder(
        // 👇 Database is already open, query immediately
        future: db.select(db.users).get(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return Center(child: CircularProgressIndicator());
          }
          
          final users = snapshot.data!;
          return ListView.builder(
            itemCount: users.length,
            itemBuilder: (context, index) {
              return ListTile(title: Text(users[index].name));
            },
          );
        },
      ),
    );
  }
}
```

> **Key insights:**
> - **Pre-open** – Database ready before app starts
> - **No waiting** – Queries run immediately
> - **Better UX** – No loading spinner on first query
> - **Error handling** – Catch issues before app starts

---

## Strategy 4: Background Opening

> **Open database in background to avoid UI jank**

```dart
// lib/services/database_service.dart
import 'package:flutter/material.dart';
import '../database/database.dart';

class DatabaseService extends ChangeNotifier {
  static final DatabaseService _instance = DatabaseService._internal();
  factory DatabaseService() => _instance;
  DatabaseService._internal();

  AppDatabase? _database;
  bool _isOpen = false;
  bool _isOpening = false;
  String? _error;

  // 👇 Open in background
  Future<void> openInBackground() async {
    if (_isOpen || _isOpening) return;
    
    _isOpening = true;
    notifyListeners();
    
    // 👇 Use compute to open in background
    await compute(_openDatabaseInBackground, null);
    
    _isOpening = false;
    _isOpen = true;
    notifyListeners();
  }

  static Future<void> _openDatabaseInBackground(void _) async {
    // This runs in a background isolate
    final db = AppDatabase();
    await db.open();
    // Store reference globally or in singleton
    _instance._database = db;
  }

  // 👇 Get database (waits if opening)
  Future<AppDatabase> getDatabase() async {
    if (_error != null) {
      throw Exception(_error);
    }
    
    if (!_isOpen) {
      await openInBackground();
    }
    
    return _database!;
  }

  // 👇 Check if database is ready
  bool get isReady => _isOpen && !_isOpening;
}

// Usage
class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final DatabaseService _dbService = DatabaseService();

  @override
  void initState() {
    super.initState();
    // 👇 Start opening in background
    _dbService.openInBackground();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: ListenableBuilder(
        listenable: _dbService,
        builder: (context, child) {
          if (!_dbService.isReady) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  CircularProgressIndicator(),
                  SizedBox(height: 16),
                  Text('Opening database...'),
                ],
              ),
            );
          }
          
          return FutureBuilder(
            future: _dbService.getDatabase(),
            builder: (context, snapshot) {
              if (!snapshot.hasData) {
                return Center(child: CircularProgressIndicator());
              }
              
              final db = snapshot.data!;
              return UsersList(db: db);
            },
          );
        },
      ),
    );
  }
}
```

> **Key insights:**
> - **Background opening** – Uses isolates to open without UI jank
> - **Status tracking** – Show loading state
> - **Ready check** – Know when database is available
> - **Error handling** – Capture and display errors

---

## Strategy 5: Connection Pool Opening

> **Open multiple connections with pooling**

```dart
// lib/database/connection_pool.dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

class ConnectionPool {
  static final ConnectionPool _instance = ConnectionPool._internal();
  factory ConnectionPool() => _instance;
  ConnectionPool._internal();

  QueryExecutor? _writeExecutor;
  QueryExecutor? _readExecutor;
  bool _isOpen = false;

  // 👇 Open all connections
  Future<void> openPool() async {
    if (_isOpen) return;
    
    print('🔓 Opening connection pool...');
    
    // Open write connection
    _writeExecutor = await _createWriteExecutor();
    
    // Open read connection pool
    _readExecutor = await _createReadExecutor();
    
    _isOpen = true;
    print('✅ Connection pool opened');
  }

  Future<QueryExecutor> _createWriteExecutor() async {
    final dir = await getApplicationDocumentsDirectory();
    final path = p.join(dir.path, 'app.db');
    
    return NativeDatabase.createInBackground(
      File(path),
      setup: (db) async {
        await db.execute('PRAGMA journal_mode = WAL');
        await db.execute('PRAGMA foreign_keys = ON');
        await db.execute('PRAGMA synchronous = FULL');
      },
      readPool: 1,
    );
  }

  Future<QueryExecutor> _createReadExecutor() async {
    final dir = await getApplicationDocumentsDirectory();
    final path = p.join(dir.path, 'app.db');
    
    return NativeDatabase.createInBackground(
      File(path),
      setup: (db) async {
        await db.execute('PRAGMA journal_mode = WAL');
        await db.execute('PRAGMA foreign_keys = ON');
        await db.execute('PRAGMA query_only = ON');
        await db.execute('PRAGMA synchronous = NORMAL');
      },
      readPool: 4, // 4 read connections
    );
  }

  QueryExecutor get writeExecutor {
    if (!_isOpen) throw Exception('Connection pool not open');
    return _writeExecutor!;
  }

  QueryExecutor get readExecutor {
    if (!_isOpen) throw Exception('Connection pool not open');
    return _readExecutor!;
  }

  // 👇 Close all connections
  Future<void> closePool() async {
    if (!_isOpen) return;
    
    print('🔒 Closing connection pool...');
    
    await _writeExecutor?.close();
    await _readExecutor?.close();
    
    _isOpen = false;
    print('✅ Connection pool closed');
  }
}

// lib/database/database.dart
class AppDatabase extends _$AppDatabase {
  AppDatabase({required QueryExecutor read, required QueryExecutor write})
      : super(write, readConnection: read);

  @override
  int get schemaVersion => 1;
}

// Usage
void main() async {
  // 👇 Open connection pool
  await ConnectionPool().openPool();
  
  // 👇 Create database with pooled connections
  final db = AppDatabase(
    read: ConnectionPool().readExecutor,
    write: ConnectionPool().writeExecutor,
  );
  
  runApp(MyApp(db: db));
}
```

> **Key insights:**
> - **Multiple connections** – Separate read and write connections
> - **Connection pool** – Multiple read connections for parallel queries
> - **Write connection** – Single connection for writes
> - **Query only** – Read connections are read-only
> - **Pool management** – Open and close all at once

---

# Opening with Options

> **Customizing the open process**

```dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

class DatabaseOpener {
  // 👇 Open with custom options
  static Future<AppDatabase> openWithOptions({
    bool readOnly = false,
    bool enableWAL = true,
    int cacheSize = 10000,
    int readPoolSize = 4,
  }) async {
    final dir = await getApplicationDocumentsDirectory();
    final path = p.join(dir.path, 'app.db');
    
    final executor = await _createExecutor(
      path: path,
      readOnly: readOnly,
      enableWAL: enableWAL,
      cacheSize: cacheSize,
      readPoolSize: readPoolSize,
    );
    
    return AppDatabase(executor);
  }

  static Future<QueryExecutor> _createExecutor({
    required String path,
    required bool readOnly,
    required bool enableWAL,
    required int cacheSize,
    required int readPoolSize,
  }) async {
    return NativeDatabase.createInBackground(
      File(path),
      setup: (db) async {
        if (enableWAL) {
          await db.execute('PRAGMA journal_mode = WAL');
        }
        
        if (readOnly) {
          await db.execute('PRAGMA query_only = ON');
        }
        
        await db.execute('PRAGMA foreign_keys = ON');
        await db.execute('PRAGMA cache_size = $cacheSize');
        await db.execute('PRAGMA synchronous = ${readOnly ? "NORMAL" : "FULL"}');
      },
      readPool: readOnly ? readPoolSize : 1,
    );
  }

  // 👇 Open with timeout
  static Future<AppDatabase> openWithTimeout(Duration timeout) async {
    return await Future.timeout(
      openWithOptions(),
      onTimeout: () {
        throw TimeoutException('Database opening timed out');
      },
    );
  }

  // 👇 Open with retry
  static Future<AppDatabase> openWithRetry({
    int maxRetries = 3,
    Duration retryDelay = const Duration(seconds: 1),
  }) async {
    int attempts = 0;
    
    while (attempts < maxRetries) {
      try {
        return await openWithOptions();
      } catch (e) {
        attempts++;
        if (attempts >= maxRetries) rethrow;
        
        print('⚠️ Open attempt $attempts failed, retrying...');
        await Future.delayed(retryDelay);
      }
    }
    
    throw Exception('Failed to open database after $maxRetries attempts');
  }
}
```

> **Key insights:**
> - **Custom options** – Configure read-only, WAL, cache
> - **Read-only mode** – Prevent accidental writes
> - **Timeout handling** – Fail if open takes too long
> - **Retry logic** – Retry on failure

---

# Real-World Example

> **Complete production database opening system**

```dart
// lib/services/database_manager.dart
import 'package:flutter/material.dart';
import 'package:connectivity_plus/connectivity_plus.dart';
import '../database/database.dart';
import '../database/connection_pool.dart';

enum DatabaseStatus {
  closed,
  opening,
  open,
  error,
  retrying,
}

class DatabaseManager extends ChangeNotifier {
  static final DatabaseManager _instance = DatabaseManager._internal();
  factory DatabaseManager() => _instance;
  DatabaseManager._internal();

  AppDatabase? _database;
  DatabaseStatus _status = DatabaseStatus.closed;
  String? _errorMessage;
  int _retryCount = 0;
  static const int maxRetries = 3;

  // 👇 Status getters
  DatabaseStatus get status => _status;
  bool get isOpen => _status == DatabaseStatus.open;
  bool get isOpening => _status == DatabaseStatus.opening;
  bool get hasError => _status == DatabaseStatus.error;
  String? get errorMessage => _errorMessage;

  // 👇 Open database with full lifecycle
  Future<AppDatabase> openDatabase() async {
    // If already open, return existing instance
    if (_status == DatabaseStatus.open && _database != null) {
      return _database!;
    }

    // If opening, wait for completion
    if (_status == DatabaseStatus.opening) {
      // Wait for open to complete
      await Future.delayed(Duration(milliseconds: 100));
      return _database!;
    }

    _status = DatabaseStatus.opening;
    _errorMessage = null;
    notifyListeners();

    try {
      // Check connectivity (optional)
      final connectivity = await Connectivity().checkConnectivity();
      if (connectivity == ConnectivityResult.none) {
        throw Exception('No internet connection');
      }

      // Open connection pool
      await ConnectionPool().openPool();

      // Create database with pooled connections
      _database = AppDatabase(
        read: ConnectionPool().readExecutor,
        write: ConnectionPool().writeExecutor,
      );

      // Verify database is working
      await _database!.customSelect('SELECT 1').get();
      
      // Run integrity check
      final integrityResult = await _database!.customSelect(
        'PRAGMA integrity_check'
      ).get().first;
      
      if (integrityResult.data['integrity_check'] != 'ok') {
        throw Exception('Database integrity check failed');
      }

      _status = DatabaseStatus.open;
      _retryCount = 0;
      notifyListeners();
      
      print('✅ Database opened successfully');
      return _database!;

    } catch (e) {
      _status = DatabaseStatus.error;
      _errorMessage = e.toString();
      notifyListeners();
      
      print('❌ Failed to open database: $e');
      
      // Attempt retry
      if (_retryCount < maxRetries) {
        _retryCount++;
        _status = DatabaseStatus.retrying;
        notifyListeners();
        
        print('🔄 Retrying open (attempt $_retryCount/$maxRetries)...');
        await Future.delayed(Duration(seconds: _retryCount));
        return await openDatabase();
      }
      
      rethrow;
    }
  }

  // 👇 Close database
  Future<void> closeDatabase() async {
    if (_database == null) return;
    
    try {
      await ConnectionPool().closePool();
      await _database!.close();
      _database = null;
      _status = DatabaseStatus.closed;
      notifyListeners();
      
      print('✅ Database closed');
    } catch (e) {
      print('❌ Error closing database: $e');
    }
  }

  // 👇 Reset database (for testing)
  Future<void> resetDatabase() async {
    await closeDatabase();
    _retryCount = 0;
    _errorMessage = null;
    _status = DatabaseStatus.closed;
    notifyListeners();
  }

  // 👇 Health check
  Future<bool> healthCheck() async {
    if (_database == null) return false;
    
    try {
      await _database!.customSelect('SELECT 1').get();
      return true;
    } catch (e) {
      return false;
    }
  }
}
```

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'services/database_manager.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // 👇 Pre-open database before app starts
  final dbManager = DatabaseManager();
  
  try {
    await dbManager.openDatabase();
  } catch (e) {
    print('Failed to open database: $e');
  }
  
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Database App',
      home: DatabaseWrapper(
        child: HomePage(),
      ),
    );
  }
}

class DatabaseWrapper extends StatelessWidget {
  final Widget child;
  
  const DatabaseWrapper({required this.child});
  
  @override
  Widget build(BuildContext context) {
    return ListenableBuilder(
      listenable: DatabaseManager(),
      builder: (context, _) {
        final status = DatabaseManager().status;
        
        if (status == DatabaseStatus.open) {
          return child;
        }
        
        if (status == DatabaseStatus.error) {
          return Scaffold(
            body: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.error, size: 64, color: Colors.red),
                  SizedBox(height: 16),
                  Text(
                    'Database Error',
                    style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text(
                    DatabaseManager().errorMessage ?? 'Unknown error',
                    style: TextStyle(fontSize: 16),
                    textAlign: TextAlign.center,
                  ),
                  SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () async {
                      await DatabaseManager().openDatabase();
                    },
                    child: Text('Retry'),
                  ),
                ],
              ),
            ),
          );
        }
        
        // Opening or retrying
        return Scaffold(
          body: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                CircularProgressIndicator(),
                SizedBox(height: 16),
                Text(
                  status == DatabaseStatus.retrying
                      ? 'Retrying (${DatabaseManager()._retryCount}/$maxRetries)...'
                      : 'Opening database...',
                  style: TextStyle(fontSize: 16),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: FutureBuilder(
        future: DatabaseManager().openDatabase(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return Center(child: CircularProgressIndicator());
          }
          
          final db = snapshot.data!;
          
          return FutureBuilder(
            future: db.select(db.users).get(),
            builder: (context, snapshot) {
              if (!snapshot.hasData) {
                return Center(child: CircularProgressIndicator());
              }
              
              final users = snapshot.data!;
              return ListView.builder(
                itemCount: users.length,
                itemBuilder: (context, index) {
                  return ListTile(title: Text(users[index].name));
                },
              );
            },
          );
        },
      ),
    );
  }
}
```

> **Benefits of this approach:**
> - **Status tracking** – Know database state at all times
> - **Error recovery** – Automatic retry on failure
> - **Health monitoring** – Check database health
> - **Clean lifecycle** – Proper open and close
> - **User feedback** – Show loading/error states
> - **Testing support** – Reset for tests

---

# Best Practices

- **Use lazy opening** – For simple apps
- **Pre-open on startup** – For better UX
- **Track opening state** – Show loading indicators
- **Handle errors gracefully** – User-friendly error messages
- **Retry on failure** – Transient errors can be retried
- **Health check** – Verify database is working
- **Background opening** – Avoid UI jank
- **Connection pooling** – For high-performance apps
- **Timeout handling** – Prevent hanging
- **Always close** – Clean up resources

---

# Common Mistakes

## Mistake 1: Not handling open errors

Wrong:
```dart
// 🚫 App crashes on open failure
final db = AppDatabase();
await db.open(); // If this fails, app crashes
```

Correct:
```dart
// ✅ Handle errors gracefully
try {
  await db.open();
} catch (e) {
  print('Failed to open: $e');
  // Show error UI
}
```

## Mistake 2: Opening on UI thread

Wrong:
```dart
// 🚫 Blocks UI
void main() {
  final db = AppDatabase();
  db.open(); // Blocks UI thread
  runApp(MyApp());
}
```

Correct:
```dart
// ✅ Open in background
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final db = AppDatabase();
  await db.open(); // Non-blocking
  runApp(MyApp());
}
```

## Mistake 3: Not checking if already open

Wrong:
```dart
// 🚫 Multiple opens
await db.open();
await db.open(); // Could cause issues
```

Correct:
```dart
// ✅ Check before opening
if (!db.isOpen) {
  await db.open();
}
```

---

# Summary

| Strategy | Use Case | Key Feature |
|----------|----------|-------------|
| **Lazy Opening** | Simple apps | Opens on first query |
| **Explicit Opening** | Need control | Manual open/close |
| **Pre-Opening** | Better UX | Open before app starts |
| **Background Opening** | Performance | No UI jank |
| **Connection Pool** | High performance | Multiple connections |

---

# Next Steps

Now you understand database opening, let's dive deeper:

- [Database Configuration](link) – Fine-tuning your database
- [Multiple Databases](link) – Working with multiple databases
- [Tables Deep Dive](link) – Advanced table definitions

---

# Did You Know?

- **LazyDatabase opens on first operation** – Not necessarily the first query

- **`SELECT 1` is the fastest way** – To check if database is open

- **WAL mode must be enabled** – For read pool to work

- **You can open databases in read-only mode** – For safety

- **Background isolates open databases faster** – Because they run in parallel

- **Connection pools can be shared** – Across multiple database instances

- **Opening errors are often recoverable** – With retry logic

---

