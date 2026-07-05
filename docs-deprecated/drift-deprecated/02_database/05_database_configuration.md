## Database Configuration

**Fine-tuning your Drift database for optimal performance and behavior**

---

# What is it?

**Database Configuration** refers to all the settings, PRAGMA statements, and options you can set on your Drift database to control its behavior, performance, and features. This includes everything from journal modes to cache sizes, foreign key constraints, and custom functions.

> **Think of Database Configuration like "tuning a race car"** – you adjust various parameters to get the best performance for your specific use case.

```dart
// Complete database configuration example
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();
    
    // 👇 All the configuration settings
    await customSelect('PRAGMA journal_mode = WAL').get();
    await customSelect('PRAGMA synchronous = NORMAL').get();
    await customSelect('PRAGMA foreign_keys = ON').get();
    await customSelect('PRAGMA cache_size = -10000').get();
    await customSelect('PRAGMA temp_store = MEMORY').get();
    await customSelect('PRAGMA mmap_size = 268435456').get();
    await customSelect('PRAGMA auto_vacuum = INCREMENTAL').get();
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **What's happening here?**
> - **PRAGMA statements** – Configure database behavior
> - **Journal mode** – Controls how changes are written
> - **Synchronous** – Controls durability vs performance
> - **Cache size** – Memory allocation for performance
> - **Foreign keys** – Enable referential integrity
> - **MMAP size** – Memory-mapped I/O for faster reads

---

# Why does it exist?

- **Performance** – Optimize for speed or memory
- **Durability** – Balance data safety vs performance
- **Concurrency** – Enable multiple readers/writers
- **Integrity** – Enforce foreign key constraints
- **Features** – Enable specific SQLite capabilities
- **Customization** – Tune for specific use cases

---

# Key Configuration Options

> **All the important PRAGMA statements you can use**

## 1. Journal Mode

> **Controls how SQLite handles transaction logging**

```dart
// lib/database/database.dart
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    // 👇 Choose your journal mode
    
    // Option 1: WAL (Write-Ahead Logging) - RECOMMENDED
    await customSelect('PRAGMA journal_mode = WAL').get();
    // ✅ Concurrent reads and writes
    // ✅ Faster performance
    // ✅ Better for multi-isolate
    
    // Option 2: DELETE (Default)
    // await customSelect('PRAGMA journal_mode = DELETE').get();
    // ✅ Traditional, widely compatible
    // ❌ Slower for concurrent access
    
    // Option 3: MEMORY
    // await customSelect('PRAGMA journal_mode = MEMORY').get();
    // ✅ Fastest
    // ❌ Data loss on crash
    
    // Option 4: OFF
    // await customSelect('PRAGMA journal_mode = OFF').get();
    // ✅ Maximum speed
    // ❌ No crash recovery
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **WAL mode** – Best for most apps, enables concurrent access
> - **DELETE mode** – Traditional, safe but slower
> - **MEMORY mode** – Fast but risks data loss
> - **OFF mode** – Maximum speed, no recovery

---

## 2. Synchronous Mode

> **Controls how aggressively SQLite writes to disk**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    // 👇 Choose your synchronous mode
    
    // Option 1: FULL (Safest - RECOMMENDED for financial data)
    await customSelect('PRAGMA synchronous = FULL').get();
    // ✅ Maximum durability
    // ❌ Slower performance
    
    // Option 2: NORMAL (Balanced - RECOMMENDED for most apps)
    await customSelect('PRAGMA synchronous = NORMAL').get();
    // ✅ Good performance
    // ✅ Good durability
    
    // Option 3: OFF (Fastest - Use with caution)
    // await customSelect('PRAGMA synchronous = OFF').get();
    // ✅ Maximum speed
    // ❌ Risk of corruption on crash
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **FULL mode** – Every write is synced to disk, safest
> - **NORMAL mode** – Good balance, recommended for most apps
> - **OFF mode** – Fastest but risky, not recommended

---

## 3. Cache Size

> **Controls memory allocated for caching**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    // 👇 Configure cache size
    
    // Option 1: Set to specific number of pages
    await customSelect('PRAGMA cache_size = 10000').get();
    // 10,000 pages × 4KB = 40MB cache
    
    // Option 2: Use negative number (KB)
    await customSelect('PRAGMA cache_size = -20000').get();
    // 20MB cache
    
    // Option 3: Large cache for performance
    // await customSelect('PRAGMA cache_size = -50000').get();
    // 50MB cache
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **Positive numbers** – Pages (typically 4KB each)
> - **Negative numbers** – Kilobytes (KB)
> - **Larger cache** – Better performance, more memory
> - **Balance** – Cache size vs memory usage

---

## 4. Foreign Keys

> **Enforce referential integrity**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    // 👇 Always enable foreign keys (RECOMMENDED)
    await customSelect('PRAGMA foreign_keys = ON').get();
    
    // This ensures:
    // - Cannot delete a user if they have posts
    // - Cannot insert a post with invalid user_id
    // - Cascade deletes work properly
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **Always enable** – Prevents orphaned records
> - **ON by default** – In modern SQLite versions
> - **Required** – For foreign key constraints to work

---

## 5. Memory-Mapped I/O

> **Use memory mapping for faster reads**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    // 👇 Enable memory-mapped I/O
    
    // Option 1: Standard size
    await customSelect('PRAGMA mmap_size = 268435456').get();
    // 256MB memory map
    
    // Option 2: Large database
    // await customSelect('PRAGMA mmap_size = 1073741824').get();
    // 1GB memory map
    
    // Option 3: Disable
    // await customSelect('PRAGMA mmap_size = 0').get();
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **Memory mapping** – Maps database file to memory
> - **Faster reads** – Avoids system calls
> - **Requires enough memory** – Don't exceed available RAM
> - **Database size** – Set based on expected database size

---

## 6. Auto Vacuum

> **Controls database space management**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    // 👇 Choose auto-vacuum mode
    
    // Option 1: NONE (Default)
    // await customSelect('PRAGMA auto_vacuum = NONE').get();
    // ❌ Database never shrinks
    
    // Option 2: FULL (Shrink after each transaction)
    // await customSelect('PRAGMA auto_vacuum = FULL').get();
    // ✅ Keeps database small
    // ❌ Slower transactions
    
    // Option 3: INCREMENTAL (Gradual shrink - RECOMMENDED)
    await customSelect('PRAGMA auto_vacuum = INCREMENTAL').get();
    // ✅ Gradual space recovery
    // ✅ Good performance
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **INCREMENTAL** – Best balance for most apps
> - **FULL** – Smallest database, slower operations
> - **NONE** – Fastest, but database never shrinks

---

## 7. Temp Store

> **Where to store temporary tables and indices**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    // 👇 Choose temp store location
    
    // Option 1: DEFAULT (Use default location)
    // await customSelect('PRAGMA temp_store = DEFAULT').get();
    
    // Option 2: FILE (Use filesystem - safer)
    // await customSelect('PRAGMA temp_store = FILE').get();
    // ✅ Data persists if memory is low
    
    // Option 3: MEMORY (Fastest - RECOMMENDED)
    await customSelect('PRAGMA temp_store = MEMORY').get();
    // ✅ Faster
    // ❌ Uses memory
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **MEMORY** – Fastest, recommended for performance
> - **FILE** – Safer, uses less memory
> - **DEFAULT** – Platform default

---

## 8. Journal Size Limit

> **Control the size of the WAL journal file**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    // 👇 Set journal size limit
    
    // Option 1: Standard limit
    await customSelect('PRAGMA journal_size_limit = 67108864').get();
    // 64MB limit
    
    // Option 2: Large database
    // await customSelect('PRAGMA journal_size_limit = 268435456').get();
    // 256MB limit
    
    // Option 3: Unlimited
    // await customSelect('PRAGMA journal_size_limit = -1').get();
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **Limits WAL file growth** – Prevents disk space issues
> - **Balance** – Larger for more concurrent transactions
> - **Smaller** – Saves disk space

---

## 9. Page Size

> **Set the database page size**

```dart
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    // 👇 Set page size (must be set before creating tables)
    
    // Option 1: Default (4096)
    // await customSelect('PRAGMA page_size = 4096').get();
    
    // Option 2: Optimized for large records
    // await customSelect('PRAGMA page_size = 8192').get();
    
    // Option 3: Optimized for small records
    // await customSelect('PRAGMA page_size = 2048').get();
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

> **Key insights:**
> - **Default is 4096** – Works for most apps
> - **Larger page** – Better for large records
> - **Smaller page** – Better for many small records
> - **Must be set** – Before creating tables

---

# Complete Configuration Example

> **All settings combined for production use**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';
import 'package:path_provider/path_provider.dart';

import 'tables/users.dart';
import 'tables/posts.dart';
import 'tables/comments.dart';

part 'database.g.dart';

@DriftDatabase(tables: [Users, Posts, Comments])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Complete database configuration
  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    print('⚙️ Configuring database...');

    // 1️⃣ Journal Mode - Enable WAL for concurrent access
    await customSelect('PRAGMA journal_mode = WAL').get();
    print('✅ Journal mode: WAL');

    // 2️⃣ Synchronous - Balance performance and durability
    await customSelect('PRAGMA synchronous = NORMAL').get();
    print('✅ Synchronous: NORMAL');

    // 3️⃣ Foreign Keys - Enforce referential integrity
    await customSelect('PRAGMA foreign_keys = ON').get();
    print('✅ Foreign keys: ON');

    // 4️⃣ Cache Size - Allocate 10MB for cache
    await customSelect('PRAGMA cache_size = -10000').get();
    print('✅ Cache size: 10MB');

    // 5️⃣ Temp Store - Use memory for temp tables
    await customSelect('PRAGMA temp_store = MEMORY').get();
    print('✅ Temp store: MEMORY');

    // 6️⃣ MMAP Size - Memory map 256MB for faster reads
    await customSelect('PRAGMA mmap_size = 268435456').get();
    print('✅ MMAP size: 256MB');

    // 7️⃣ Auto Vacuum - Incremental cleanup
    await customSelect('PRAGMA auto_vacuum = INCREMENTAL').get();
    print('✅ Auto vacuum: INCREMENTAL');

    // 8️⃣ Journal Size Limit - Prevent WAL file from growing too large
    await customSelect('PRAGMA journal_size_limit = 67108864').get();
    print('✅ Journal size limit: 64MB');

    // 9️⃣ Optimize for performance
    await customSelect('PRAGMA optimize').get();
    print('✅ Database optimized');

    // 🔟 Integrity check (optional)
    final result = await customSelect('PRAGMA integrity_check').get();
    final status = result.first.data['integrity_check'] as String;
    if (status == 'ok') {
      print('✅ Integrity check passed');
    } else {
      print('⚠️ Integrity check warning: $status');
    }

    print('✅ Database configuration complete!');
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'my_app',
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

> **Key insights:**
> - **All settings** – Applied in `beforeOpen()`
> - **Logging** – Track what's being configured
> - **Integrity check** – Verify database health
> - **Optimization** – Run `PRAGMA optimize`
> - **Platform awareness** – Different settings for different platforms

---

# Platform-Specific Configuration

> **Different settings for different platforms**

```dart
// lib/database/database_config.dart
import 'dart:io';
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

class DatabaseConfig {
  // 👇 Platform-specific configuration
  static Future<void> configure(AppDatabase db) async {
    if (Platform.isAndroid) {
      await _configureAndroid(db);
    } else if (Platform.isIOS) {
      await _configureIOS(db);
    } else if (Platform.isMacOS || Platform.isWindows || Platform.isLinux) {
      await _configureDesktop(db);
    } else if (Platform.isBrowser) {
      await _configureWeb(db);
    }
  }

  // Android configuration
  static Future<void> _configureAndroid(AppDatabase db) async {
    print('📱 Configuring for Android...');
    
    await db.customSelect('PRAGMA journal_mode = WAL').get();
    await db.customSelect('PRAGMA synchronous = NORMAL').get();
    await db.customSelect('PRAGMA foreign_keys = ON').get();
    await db.customSelect('PRAGMA cache_size = -10000').get(); // 10MB
    
    // Android-specific optimizations
    await db.customSelect('PRAGMA mmap_size = 134217728').get(); // 128MB
  }

  // iOS configuration
  static Future<void> _configureIOS(AppDatabase db) async {
    print('📱 Configuring for iOS...');
    
    await db.customSelect('PRAGMA journal_mode = WAL').get();
    await db.customSelect('PRAGMA synchronous = FULL').get(); // iOS prefers durability
    await db.customSelect('PRAGMA foreign_keys = ON').get();
    await db.customSelect('PRAGMA cache_size = -15000').get(); // 15MB
    
    // iOS-specific optimizations
    await db.customSelect('PRAGMA mmap_size = 268435456').get(); // 256MB
  }

  // Desktop configuration
  static Future<void> _configureDesktop(AppDatabase db) async {
    print('💻 Configuring for Desktop...');
    
    await db.customSelect('PRAGMA journal_mode = WAL').get();
    await db.customSelect('PRAGMA synchronous = NORMAL').get();
    await db.customSelect('PRAGMA foreign_keys = ON').get();
    await db.customSelect('PRAGMA cache_size = -50000').get(); // 50MB
    
    // Desktop-specific optimizations
    await db.customSelect('PRAGMA mmap_size = 536870912').get(); // 512MB
    await db.customSelect('PRAGMA journal_size_limit = 268435456').get(); // 256MB
  }

  // Web configuration
  static Future<void> _configureWeb(AppDatabase db) async {
    print('🌐 Configuring for Web...');
    
    // Web has limited memory, use conservative settings
    await db.customSelect('PRAGMA journal_mode = MEMORY').get();
    await db.customSelect('PRAGMA synchronous = NORMAL').get();
    await db.customSelect('PRAGMA foreign_keys = ON').get();
    await db.customSelect('PRAGMA cache_size = -5000').get(); // 5MB
    await db.customSelect('PRAGMA temp_store = MEMORY').get();
  }
}
```

> **Key insights:**
> - **Platform detection** – Configure based on platform
> - **Android** – Focus on performance
> - **iOS** – Focus on durability
> - **Desktop** – More memory available
> - **Web** – Conservative settings for browser

---

# Real-World Example

> **Production database configuration with monitoring**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';
import 'package:path_provider/path_provider.dart';
import 'dart:developer' as dev;

import 'tables/users.dart';
import 'tables/posts.dart';
import 'tables/comments.dart';

part 'database.g.dart';

@DriftDatabase(tables: [Users, Posts, Comments])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Production configuration
  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();

    try {
      // 1️⃣ Transaction for atomic configuration
      await transaction(() async {
        // Performance settings
        await customSelect('PRAGMA journal_mode = WAL').get();
        await customSelect('PRAGMA synchronous = NORMAL').get();
        await customSelect('PRAGMA foreign_keys = ON').get();
        await customSelect('PRAGMA cache_size = -20000').get(); // 20MB
        await customSelect('PRAGMA temp_store = MEMORY').get();
        await customSelect('PRAGMA mmap_size = 268435456').get(); // 256MB
        await customSelect('PRAGMA auto_vacuum = INCREMENTAL').get();
        await customSelect('PRAGMA journal_size_limit = 67108864').get(); // 64MB
        
        // 2️⃣ Optimize
        await customSelect('PRAGMA optimize').get();
      });

      // 3️⃣ Health check
      await _performHealthCheck();

      dev.log('✅ Database configured successfully');

    } catch (e) {
      dev.log('❌ Database configuration failed: $e');
      rethrow;
    }
  }

  // 👇 Health check
  Future<void> _performHealthCheck() async {
    // Check integrity
    final integrity = await customSelect('PRAGMA integrity_check').get();
    final status = integrity.first.data['integrity_check'] as String;
    
    if (status != 'ok') {
      throw Exception('Database integrity check failed: $status');
    }

    // Check foreign keys
    final fk = await customSelect('PRAGMA foreign_keys').get();
    if (fk.first.data['foreign_keys'] != 1) {
      dev.log('⚠️ Foreign keys not enabled');
    }

    // Check journal mode
    final journal = await customSelect('PRAGMA journal_mode').get();
    dev.log('📝 Journal mode: ${journal.first.data['journal_mode']}');

    // Check cache size
    final cache = await customSelect('PRAGMA cache_size').get();
    dev.log('💾 Cache size: ${cache.first.data['cache_size']}');
  }

  // 👇 Get configuration report
  Future<Map<String, dynamic>> getConfigurationReport() async {
    final journal = await customSelect('PRAGMA journal_mode').get();
    final sync = await customSelect('PRAGMA synchronous').get();
    final fk = await customSelect('PRAGMA foreign_keys').get();
    final cache = await customSelect('PRAGMA cache_size').get();
    final mmap = await customSelect('PRAGMA mmap_size').get();
    final vacuum = await customSelect('PRAGMA auto_vacuum').get();
    final pageSize = await customSelect('PRAGMA page_size').get();
    
    return {
      'journal_mode': journal.first.data['journal_mode'],
      'synchronous': sync.first.data['synchronous'],
      'foreign_keys': fk.first.data['foreign_keys'] == 1,
      'cache_size': cache.first.data['cache_size'],
      'mmap_size': mmap.first.data['mmap_size'],
      'auto_vacuum': vacuum.first.data['auto_vacuum'],
      'page_size': pageSize.first.data['page_size'],
    };
  }

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'my_app',
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
// lib/main.dart
import 'package:flutter/material.dart';
import 'database/database.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  final db = AppDatabase();
  
  // Check configuration
  final config = await db.getConfigurationReport();
  print('📊 Database Configuration:');
  config.forEach((key, value) {
    print('  $key: $value');
  });
  
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
      appBar: AppBar(title: Text('Database Demo')),
      body: FutureBuilder(
        future: db.getConfigurationReport(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return Center(child: CircularProgressIndicator());
          }
          
          final config = snapshot.data!;
          return ListView(
            children: config.entries.map((entry) {
              return ListTile(
                title: Text(entry.key.replaceAll('_', ' ').toUpperCase()),
                subtitle: Text(entry.value.toString()),
              );
            }).toList(),
          );
        },
      ),
    );
  }
}
```

> **Benefits of this approach:**
> - **Atomic configuration** – All settings applied together
> - **Health monitoring** – Check database status
> - **Configuration report** – View current settings
> - **Error handling** – Catch configuration errors
> - **Logging** – Track configuration process
> - **Testing** – Verify settings in tests

---

# Best Practices

- **Apply settings in beforeOpen()** – Before database is used
- **Use WAL mode** – For concurrent access
- **Enable foreign keys** – Maintain data integrity
- **Set appropriate cache size** – Balance performance and memory
- **Use memory temp store** – For better performance
- **Run PRAGMA optimize** – Periodically for performance
- **Check integrity** – Verify database health
- **Test configuration** – Verify settings in tests
- **Log configuration** – Track what's applied
- **Platform-specific** – Different settings for different platforms

---

# Common Mistakes

## Mistake 1: Setting PRAGMA after tables are created

Wrong:
```dart
// 🚫 Page size must be set before tables
@override
Future<void> beforeOpen() async {
  await customSelect('PRAGMA page_size = 8192').get();
  // Tables already created with default page size
}
```

Correct:
```dart
// ✅ Set page size at creation time
@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (migrator) async {
    await customSelect('PRAGMA page_size = 8192').get();
    await migrator.createAll();
  },
);
```

## Mistake 2: Not enabling foreign keys

Wrong:
```dart
// 🚫 Foreign key constraints ignored
@override
Future<void> beforeOpen() async {
  // Missing foreign_keys = ON
}
```

Correct:
```dart
// ✅ Always enable foreign keys
@override
Future<void> beforeOpen() async {
  await customSelect('PRAGMA foreign_keys = ON').get();
}
```

## Mistake 3: Using incompatible settings

Wrong:
```dart
// 🚫 readPool requires WAL mode
@override
Future<void> beforeOpen() async {
  await customSelect('PRAGMA journal_mode = DELETE').get();
}
// readPool: 4 // Will cause "database locked" errors
```

Correct:
```dart
// ✅ Enable WAL for readPool
@override
Future<void> beforeOpen() async {
  await customSelect('PRAGMA journal_mode = WAL').get();
}
```

---

# Summary

| Setting | Recommended | Purpose |
|---------|-------------|---------|
| **Journal Mode** | WAL | Concurrent access |
| **Synchronous** | NORMAL | Performance vs durability |
| **Foreign Keys** | ON | Data integrity |
| **Cache Size** | -10000 or more | Performance |
| **Temp Store** | MEMORY | Faster temp operations |
| **MMAP Size** | 256MB+ | Faster reads |
| **Auto Vacuum** | INCREMENTAL | Space management |
| **Journal Limit** | 64MB+ | Prevent file growth |

---

# Next Steps

Now you understand database configuration, let's dive deeper:

- [Multiple Databases](link) – Working with multiple databases
- [Tables Deep Dive](link) – Advanced table definitions
- [Performance](link) – Advanced performance optimization

---

# Did You Know?

- **WAL mode enables up to 50% better performance** – For concurrent reads and writes

- **PRAGMA optimize analyzes all queries** – And creates indexes automatically

- **MMAP size can be larger than the database** – It maps the entire file

- **Cache size uses negative numbers for KB** – Positive numbers are pages

- **Foreign keys are OFF by default** – You must enable them

- **Auto vacuum can be run manually** – With `VACUUM` command

- **PRAGMA integrity_check** – Can detect corruption early

- **Settings persist** – They stay until the database is closed

---

