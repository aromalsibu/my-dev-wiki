## Multiple Databases

**Working with multiple Drift databases in a single application**

---

# What is it?

**Multiple Databases** refers to the ability to have separate Drift database instances in your application, each with its own schema, connections, and data. This is useful for separating concerns, managing different types of data, or implementing features like multi-tenant architectures.

> **Think of Multiple Databases like "having multiple filing cabinets"** – each cabinet holds different types of documents, and you can organize them separately for better management.

```dart
// Multiple databases in one app

// 1️⃣ Main database for user data
@DriftDatabase(tables: [Users, Posts])
class MainDatabase extends _$MainDatabase {
  MainDatabase() : super(_openConnection('main.db'));
  @override
  int get schemaVersion => 1;
  static QueryExecutor _openConnection(String name) {
    return driftDatabase(name: name);
  }
}

// 2️⃣ Settings database for app preferences
@DriftDatabase(tables: [Settings])
class SettingsDatabase extends _$SettingsDatabase {
  SettingsDatabase() : super(_openConnection('settings.db'));
  @override
  int get schemaVersion => 1;
  static QueryExecutor _openConnection(String name) {
    return driftDatabase(name: name);
  }
}

// Usage
final mainDb = MainDatabase();
final settingsDb = SettingsDatabase();
```

> **What's happening here?**
> - **Separate files** – Each database has its own file
> - **Independent schemas** – Different tables in each database
> - **Separate connections** – Each database opens its own connection
> - **Isolation** – Data is completely separated

---

# Why does it exist?

- **Separation of Concerns** – Different data for different features
- **Performance** – Smaller databases for faster operations
- **Maintainability** – Easier to manage and update
- **Security** – Different access levels for different data
- **Testing** – Test each database independently
- **Modularity** – Plug-and-play database modules

---

# Creating Multiple Databases

> **How to set up and use multiple databases**

## Example 1: Feature-Based Separation

> **Different databases for different features**

```dart
// lib/database/main_database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables/users.dart';
import 'tables/posts.dart';
import 'tables/comments.dart';

part 'main_database.g.dart';

@DriftDatabase(tables: [Users, Posts, Comments])
class MainDatabase extends _$MainDatabase {
  MainDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'main',
      native: const DriftNativeOptions(
        databaseDirectory: getApplicationSupportDirectory,
      ),
    );
  }

  // Queries specific to main database
  Future<List<User>> getActiveUsers() async {
    return await (select(users)..where((u) => u.isActive.equals(true))).get();
  }
}

// lib/database/settings_database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables/settings.dart';
import 'tables/preferences.dart';

part 'settings_database.g.dart';

@DriftDatabase(tables: [Settings, Preferences])
class SettingsDatabase extends _$SettingsDatabase {
  SettingsDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'settings',
      native: const DriftNativeOptions(
        databaseDirectory: getApplicationSupportDirectory,
      ),
    );
  }

  // Settings-specific queries
  Future<Setting?> getSetting(String key) async {
    return await (select(settings)..where((s) => s.key.equals(key)))
        .getSingleOrNull();
  }
}

// lib/database/analytics_database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables/events.dart';
import 'tables/sessions.dart';

part 'analytics_database.g.dart';

@DriftDatabase(tables: [Events, Sessions])
class AnalyticsDatabase extends _$AnalyticsDatabase {
  AnalyticsDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'analytics',
      native: const DriftNativeOptions(
        databaseDirectory: getApplicationSupportDirectory,
      ),
    );
  }

  // Analytics-specific queries
  Future<List<Event>> getEventsToday() async {
    final today = DateTime.now();
    final startOfDay = DateTime(today.year, today.month, today.day);
    
    return await (select(events)
      ..where((e) => e.timestamp.isBiggerOrEqualValue(startOfDay)))
      .get();
  }
}
```

> **Key insights:**
> - **Main database** – Core app data (users, posts, comments)
> - **Settings database** – App preferences and settings
> - **Analytics database** – Usage tracking and events
> - **Separate files** – `main.db`, `settings.db`, `analytics.db`

---

## Example 2: Multi-Tenant Architecture

> **Different databases for different tenants**

```dart
// lib/database/tenant_database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

import 'tables/users.dart';
import 'tables/products.dart';
import 'tables/orders.dart';

part 'tenant_database.g.dart';

@DriftDatabase(tables: [Users, Products, Orders])
class TenantDatabase extends _$TenantDatabase {
  final String tenantId;

  TenantDatabase(this.tenantId, [QueryExecutor? executor]) 
      : super(executor ?? _openConnection(tenantId));

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection(String tenantId) {
    return driftDatabase(
      name: 'tenant_$tenantId', // Different file per tenant
      native: const DriftNativeOptions(
        databaseDirectory: getApplicationSupportDirectory,
      ),
    );
  }

  // Tenant-specific queries
  Future<List<User>> getTenantUsers() async {
    return await select(users).get();
  }

  Future<List<Product>> getAvailableProducts() async {
    return await (select(products)
      ..where((p) => p.isAvailable.equals(true)))
      .get();
  }
}

// lib/services/tenant_manager.dart
class TenantManager {
  static final Map<String, TenantDatabase> _databases = {};
  
  static TenantDatabase getDatabase(String tenantId) {
    if (!_databases.containsKey(tenantId)) {
      _databases[tenantId] = TenantDatabase(tenantId);
    }
    return _databases[tenantId]!;
  }
  
  static Future<void> closeAll() async {
    for (final db in _databases.values) {
      await db.close();
    }
    _databases.clear();
  }
}

// Usage
final tenant1Db = TenantManager.getDatabase('tenant_123');
final tenant2Db = TenantManager.getDatabase('tenant_456');

// Each tenant has its own database file
// tenant_123.db and tenant_456.db
```

> **Key insights:**
> - **One database per tenant** – Complete isolation
> - **Dynamic naming** – `tenant_{id}.db`
> - **Caching** – Cache database instances
> - **Cleanup** – Close all when done

---

## Example 3: Read-Only vs Read-Write Separation

> **Separate databases for different access patterns**

```dart
// lib/database/readonly_database.dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

import 'tables/products.dart';
import 'tables/categories.dart';

part 'readonly_database.g.dart';

@DriftDatabase(tables: [Products, Categories])
class ReadOnlyDatabase extends _$ReadOnlyDatabase {
  ReadOnlyDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return LazyDatabase(() async {
      final dir = await getApplicationDocumentsDirectory();
      final path = p.join(dir.path, 'products.db');
      
      final db = await databaseFactoryIo.openDatabase(
        path,
        options: OpenDatabaseOptions(
          readOnly: true, // 👈 Read-only mode
        ),
      );
      
      return NativeDatabase(db);
    });
  }

  // Read-only queries
  Future<List<Product>> getAllProducts() async {
    return await select(products).get();
  }

  Future<Product?> getProduct(int id) async {
    return await (select(products)..where((p) => p.id.equals(id)))
        .getSingleOrNull();
  }
}

// lib/database/readwrite_database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables/products.dart';
import 'tables/categories.dart';

part 'readwrite_database.g.dart';

@DriftDatabase(tables: [Products, Categories])
class ReadWriteDatabase extends _$ReadWriteDatabase {
  ReadWriteDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'products',
      native: const DriftNativeOptions(
        databaseDirectory: getApplicationSupportDirectory,
      ),
    );
  }

  // Write operations
  Future<int> addProduct(String name, double price) async {
    return await into(products).insert(
      ProductsCompanion.insert(
        name: name,
        price: price,
      ),
    );
  }

  Future<void> updateProduct(int id, String name) async {
    await (update(products)..where((p) => p.id.equals(id)))
      .write(ProductsCompanion(name: Value(name)));
  }
}

// Usage
final readDb = ReadOnlyDatabase(); // For searching/viewing
final writeDb = ReadWriteDatabase(); // For adding/editing

// ✅ Read from read-only database (faster)
final products = await readDb.getAllProducts();

// ✅ Write to read-write database
await writeDb.addProduct('New Product', 99.99);
```

> **Key insights:**
> - **Read-only database** – Optimized for reads, faster
> - **Read-write database** – Handles all writes
> - **Shared tables** – Same schema in both databases
> - **Different connections** – Each has its own connection

---

# Managing Multiple Databases

> **Complete management system for multiple databases**

```dart
// lib/services/database_manager.dart
import 'package:flutter/foundation.dart';
import '../database/main_database.dart';
import '../database/settings_database.dart';
import '../database/analytics_database.dart';

enum DatabaseType {
  main,
  settings,
  analytics,
}

class DatabaseManager extends ChangeNotifier {
  static final DatabaseManager _instance = DatabaseManager._internal();
  factory DatabaseManager() => _instance;
  DatabaseManager._internal();

  MainDatabase? _mainDb;
  SettingsDatabase? _settingsDb;
  AnalyticsDatabase? _analyticsDb;

  bool _isInitialized = false;
  final Map<DatabaseType, bool> _status = {};

  // 👇 Initialize all databases
  Future<void> initializeAll() async {
    if (_isInitialized) return;

    try {
      print('📦 Initializing all databases...');

      // Initialize main database
      _mainDb = MainDatabase();
      await _mainDb!.ensureOpen();
      _status[DatabaseType.main] = true;

      // Initialize settings database
      _settingsDb = SettingsDatabase();
      await _settingsDb!.ensureOpen();
      _status[DatabaseType.settings] = true;

      // Initialize analytics database
      _analyticsDb = AnalyticsDatabase();
      await _analyticsDb!.ensureOpen();
      _status[DatabaseType.analytics] = true;

      _isInitialized = true;
      notifyListeners();

      print('✅ All databases initialized');

    } catch (e) {
      print('❌ Failed to initialize databases: $e');
      rethrow;
    }
  }

  // 👇 Get specific database
  T getDatabase<T>(DatabaseType type) {
    if (!_isInitialized) {
      throw Exception('Databases not initialized. Call initializeAll() first.');
    }

    switch (type) {
      case DatabaseType.main:
        return _mainDb as T;
      case DatabaseType.settings:
        return _settingsDb as T;
      case DatabaseType.analytics:
        return _analyticsDb as T;
    }
  }

  // 👇 Check database status
  bool isReady(DatabaseType type) {
    return _status[type] ?? false;
  }

  // 👇 Close all databases
  Future<void> closeAll() async {
    print('🔒 Closing all databases...');

    try {
      await _mainDb?.close();
      await _settingsDb?.close();
      await _analyticsDb?.close();

      _mainDb = null;
      _settingsDb = null;
      _analyticsDb = null;
      _isInitialized = false;
      _status.clear();

      notifyListeners();
      print('✅ All databases closed');

    } catch (e) {
      print('❌ Error closing databases: $e');
      rethrow;
    }
  }

  // 👇 Reset all databases (for testing)
  Future<void> resetAll() async {
    await closeAll();
    await initializeAll();
  }

  // 👇 Get statistics
  Map<String, dynamic> getStats() {
    return {
      'initialized': _isInitialized,
      'databases': {
        'main': _status[DatabaseType.main] ?? false,
        'settings': _status[DatabaseType.settings] ?? false,
        'analytics': _status[DatabaseType.analytics] ?? false,
      },
    };
  }
}
```

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'services/database_manager.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize all databases
  await DatabaseManager().initializeAll();
  
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
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
        final stats = DatabaseManager().getStats();
        final allReady = stats['databases'].values.every((v) => v == true);
        
        if (!allReady) {
          return Scaffold(
            body: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  CircularProgressIndicator(),
                  SizedBox(height: 16),
                  Text('Initializing databases...'),
                ],
              ),
            ),
          );
        }
        
        return child;
      },
    );
  }
}

class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final dbManager = DatabaseManager();
    
    return Scaffold(
      appBar: AppBar(title: Text('Multi-Database App')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // Main database section
            _DatabaseCard(
              title: 'Main Database',
              subtitle: 'Users, Posts, Comments',
              status: dbManager.isReady(DatabaseType.main),
              onTap: () {
                final mainDb = dbManager.getDatabase<MainDatabase>(DatabaseType.main);
                // Use mainDb...
              },
            ),
            SizedBox(height: 16),
            
            // Settings database section
            _DatabaseCard(
              title: 'Settings Database',
              subtitle: 'App preferences, Settings',
              status: dbManager.isReady(DatabaseType.settings),
              onTap: () {
                final settingsDb = dbManager.getDatabase<SettingsDatabase>(DatabaseType.settings);
                // Use settingsDb...
              },
            ),
            SizedBox(height: 16),
            
            // Analytics database section
            _DatabaseCard(
              title: 'Analytics Database',
              subtitle: 'Events, Sessions',
              status: dbManager.isReady(DatabaseType.analytics),
              onTap: () {
                final analyticsDb = dbManager.getDatabase<AnalyticsDatabase>(DatabaseType.analytics);
                // Use analyticsDb...
              },
            ),
            SizedBox(height: 16),
            
            ElevatedButton(
              onPressed: () async {
                await dbManager.closeAll();
                await dbManager.initializeAll();
              },
              child: Text('Reset All'),
            ),
          ],
        ),
      ),
    );
  }
}

class _DatabaseCard extends StatelessWidget {
  final String title;
  final String subtitle;
  final bool status;
  final VoidCallback onTap;

  const _DatabaseCard({
    required this.title,
    required this.subtitle,
    required this.status,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: Icon(
          status ? Icons.check_circle : Icons.error,
          color: status ? Colors.green : Colors.red,
        ),
        title: Text(title),
        subtitle: Text(subtitle),
        trailing: Icon(Icons.arrow_forward),
        onTap: onTap,
      ),
    );
  }
}
```

> **Key insights:**
> - **Centralized management** – One manager for all databases
> - **Status tracking** – Know each database's state
> - **Type-safe access** – Get typed database instances
> - **Lifecycle management** – Initialize and close together
> - **Error handling** – Catch and handle errors

---

# Sharing Data Between Databases

> **Techniques for sharing data across databases**

```dart
// lib/services/data_sync_service.dart
import '../database/main_database.dart';
import '../database/analytics_database.dart';

class DataSyncService {
  final MainDatabase _mainDb;
  final AnalyticsDatabase _analyticsDb;

  DataSyncService(this._mainDb, this._analyticsDb);

  // 👇 Sync user data to analytics
  Future<void> syncUserToAnalytics(int userId) async {
    // Get user from main database
    final user = await (_mainDb.select(_mainDb.users)
      ..where((u) => u.id.equals(userId)))
      .getSingle();

    // Create analytics event
    await _analyticsDb.into(_analyticsDb.events).insert(
      EventsCompanion.insert(
        userId: userId,
        eventType: 'user_viewed',
        timestamp: DateTime.now(),
        metadata: '{"name": "${user.name}"}',
      ),
    );
  }

  // 👇 Sync settings between databases
  Future<void> syncSettings(String key, String value) async {
    // Get current setting from settings database
    final setting = await (_analyticsDb.select(_analyticsDb.settings)
      ..where((s) => s.key.equals(key)))
      .getSingleOrNull();

    // If setting doesn't exist, create it
    if (setting == null) {
      await _analyticsDb.into(_analyticsDb.settings).insert(
        SettingsCompanion.insert(
          key: key,
          value: value,
        ),
      );
    } else if (setting.value != value) {
      // Update setting
      await (_analyticsDb.update(_analyticsDb.settings)
        ..where((s) => s.key.equals(key)))
        .write(SettingsCompanion(value: Value(value)));
    }
  }

  // 👇 Generate cross-database report
  Future<Map<String, dynamic>> generateReport() async {
    // Get data from main database
    final users = await _mainDb.select(_mainDb.users).get();
    final posts = await _mainDb.select(_mainDb.posts).get();

    // Get data from analytics database
    final events = await _analyticsDb.select(_analyticsDb.events).get();
    final sessions = await _analyticsDb.select(_analyticsDb.sessions).get();

    return {
      'totalUsers': users.length,
      'totalPosts': posts.length,
      'totalEvents': events.length,
      'totalSessions': sessions.length,
      'activeUsers': users.where((u) => u.isActive).length,
      'mostActiveUser': _findMostActiveUser(events, users),
    };
  }

  String? _findMostActiveUser(List<Event> events, List<User> users) {
    if (events.isEmpty) return null;
    
    final userIds = events.map((e) => e.userId).toList();
    final mostActiveId = userIds
        .groupBy((id) => id)
        .map((id, list) => MapEntry(id, list.length))
        .entries
        .reduce((a, b) => a.value > b.value ? a : b)
        .key;
    
    return users.firstWhere((u) => u.id == mostActiveId).name;
  }
}

// Extension for grouping
extension ListExtensions<T> on List<T> {
  Map<T, List<T>> groupBy(dynamic Function(T) key) {
    final map = <T, List<T>>{};
    for (final item in this) {
      final k = key(item) as T;
      map.putIfAbsent(k, () => []).add(item);
    }
    return map;
  }
}
```

> **Key insights:**
> - **Cross-database queries** – Combine data from multiple databases
> - **Sync operations** – Keep data consistent across databases
> - **Reports** – Generate insights from multiple sources
> - **Service layer** – Business logic separate from databases

---

# Testing Multiple Databases

> **Testing strategies for multiple databases**

```dart
// test/database_test.dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:test/test.dart';
import '../lib/database/main_database.dart';
import '../lib/database/settings_database.dart';
import '../lib/database/analytics_database.dart';

void main() {
  group('Multiple Databases Tests', () {
    late MainDatabase mainDb;
    late SettingsDatabase settingsDb;
    late AnalyticsDatabase analyticsDb;

    setUp(() {
      // 👇 Use in-memory databases for testing
      mainDb = MainDatabase(NativeDatabase.memory());
      settingsDb = SettingsDatabase(NativeDatabase.memory());
      analyticsDb = AnalyticsDatabase(NativeDatabase.memory());
    });

    tearDown(() async {
      await mainDb.close();
      await settingsDb.close();
      await analyticsDb.close();
    });

    test('should work with all databases', () async {
      // Insert into main database
      final userId = await mainDb.into(mainDb.users).insert(
        UsersCompanion.insert(
          name: 'Test User',
          email: 'test@example.com',
        ),
      );

      // Insert into settings database
      await settingsDb.into(settingsDb.settings).insert(
        SettingsCompanion.insert(
          key: 'theme',
          value: 'dark',
        ),
      );

      // Insert into analytics database
      await analyticsDb.into(analyticsDb.events).insert(
        EventsCompanion.insert(
          userId: userId,
          eventType: 'login',
          timestamp: DateTime.now(),
        ),
      );

      // Verify all databases
      expect(await mainDb.select(mainDb.users).count(), 1);
      expect(await settingsDb.select(settingsDb.settings).count(), 1);
      expect(await analyticsDb.select(analyticsDb.events).count(), 1);
    });

    test('should handle cross-database operations', () async {
      // Setup data
      final userId = await mainDb.into(mainDb.users).insert(
        UsersCompanion.insert(
          name: 'Test User',
          email: 'test@example.com',
        ),
      );

      // Sync service test
      final syncService = DataSyncService(mainDb, analyticsDb);
      await syncService.syncUserToAnalytics(userId);

      // Verify sync worked
      final events = await analyticsDb.select(analyticsDb.events).get();
      expect(events.length, 1);
      expect(events.first.userId, userId);
      expect(events.first.eventType, 'user_viewed');
    });

    test('should handle errors gracefully', () async {
      // Create a corrupted database scenario
      // ... test error handling
    });
  });
}
```

> **Key insights:**
> - **In-memory databases** – Fast, isolated tests
> - **Setup/teardown** – Fresh databases for each test
> - **Cross-database tests** – Test sync operations
> - **Error handling** – Test failure scenarios

---

# Best Practices

- **Use separate databases for different concerns** – Keep data organized
- **Use consistent naming** – `{feature}_database.dart`
- **Centralize management** – Use a database manager
- **Cache database instances** – Reuse instead of recreating
- **Initialize all at startup** – Or use lazy initialization
- **Close all on app exit** – Proper cleanup
- **Use in-memory for testing** – Fast and isolated
- **Document relationships** – Make clear which data goes where
- **Monitor performance** – Multiple connections consume resources
- **Consider alternatives** – For simple apps, one database might be enough

---

# Common Mistakes

## Mistake 1: Not closing all databases

Wrong:
```dart
// 🚫 Only closes one database
void dispose() {
  mainDb.close();
  // settingsDb and analyticsDb stay open
}
```

Correct:
```dart
// ✅ Close all databases
void dispose() async {
  await mainDb.close();
  await settingsDb.close();
  await analyticsDb.close();
}
```

## Mistake 2: Sharing connections across databases

Wrong:
```dart
// 🚫 Same connection for different databases
final executor = NativeDatabase(File('app.db'));
final db1 = AppDatabase1(executor);
final db2 = AppDatabase2(executor); // Same connection!
```

Correct:
```dart
// ✅ Separate connections
final db1 = AppDatabase1(NativeDatabase(File('app1.db')));
final db2 = AppDatabase2(NativeDatabase(File('app2.db')));
```

## Mistake 3: Not initializing all databases

Wrong:
```dart
// 🚫 Some databases might not be ready
void main() {
  final mainDb = MainDatabase(); // Only main is ready
  runApp(MyApp());
}
```

Correct:
```dart
// ✅ Initialize all at startup
void main() async {
  await DatabaseManager().initializeAll();
  runApp(MyApp());
}
```

---

# Summary

| Aspect | Best Practice | Purpose |
|--------|---------------|---------|
| **Separation** | Feature-based or tenant-based | Organization |
| **Naming** | `{feature}_database.dart` | Clarity |
| **Management** | Centralized manager | Control |
| **Lifecycle** | Initialize and close together | Consistency |
| **Testing** | In-memory databases | Speed |

---

# Next Steps

Now you understand multiple databases, let's dive deeper:

- [Tables Deep Dive](link) – Advanced table definitions
- [Columns](link) – Advanced column configuration
- [Relationships](link) – Table relationships

---

# Did You Know?

- **You can have up to 30 databases** – Limited by SQLite's `sqlite3_open` calls

- **Multiple databases can share a connection pool** – For better performance

- **SQLite supports ATTACH** – You can query across databases

- **Each database has its own WAL file** – Separate WAL files per database

- **Database files are independent** – You can copy/delete one without affecting others

- **Testing with in-memory databases** – Each test gets a fresh database

- **Multi-tenant apps benefit** – Separate databases for isolation

- **Analytics databases** – Can be optimized differently than main databases

---

