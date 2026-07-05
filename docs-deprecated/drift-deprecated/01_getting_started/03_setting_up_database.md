

# Setting Up a Database

**The complete guide to Drift database setup with QueryExecutor**

---

# What is it?

**Setting up a database** in Drift means creating your database class, defining tables, and configuring the connection. The **modern approach** uses `drift_flutter` with the `driftDatabase()` helper, making it **platform-aware**, **flexible**, and **production-ready** out of the box.

> **Think of `driftDatabase()` as your database "wizard"** – it handles all the platform-specific details so you don't have to!

```dart
// The modern, clean way with drift_flutter
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';
import 'package:path_provider/path_provider.dart';

part 'database.g.dart';

// 1. Define your table
class TodoItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 6, max: 32)();
  TextColumn get content => text().named('body')();
  DateTimeColumn get createdAt => dateTime().nullable()();
}

// 2. Create the database
@DriftDatabase(tables: [TodoItems])
class AppDatabase extends _$AppDatabase {
  // 👇 Accept optional executor, use default if not provided
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 3. Connection setup with drift_flutter
  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'my_database',
      native: const DriftNativeOptions(
        // 👇 Customize storage location
        databaseDirectory: getApplicationSupportDirectory,
      ),
      // 👇 Optional: Web support
      web: DriftWebOptions(
        sqlite3Wasm: Uri.parse('sqlite3.wasm'),
        driftWorker: Uri.parse('drift_worker.js'),
      ),
    );
  }
}

// Usage
void main() async {
  final db = AppDatabase(); // Uses default connection
  // Or with custom executor
  final testDb = AppDatabase(NativeDatabase.memory());
}
```

> **What's happening here?**
> - **`driftDatabase()`** – Platform-aware connection helper from `drift_flutter`
> - **`name: 'my_database'`** – Database filename (creates `my_database.db`)
> - **`DriftNativeOptions`** – Configure native platform settings
> - **`databaseDirectory`** – Where to store the database file
> - **Optional executor** – Constructor accepts any `QueryExecutor` for testing
> - **Web support** – Configure WASM and worker files

---

# Why does it exist?

- **Platform Abstraction** – Same code works on mobile, desktop, and web
- **Zero Boilerplate** – `driftDatabase()` handles all the platform-specific setup
- **Flexible Storage** – Choose where to store your database file
- **Testability** – Inject in-memory or custom executors
- **Production Optimized** – Sets up WAL mode, foreign keys, and caching automatically
- **Future-Proof** – Easy to add new platforms or configurations

---

# The Modern Way: Using `drift_flutter`

> **The recommended approach for Flutter apps**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';
import 'package:path_provider/path_provider.dart';

import 'tables/todo_items.dart';
import 'tables/users.dart';
import 'tables/posts.dart';

part 'database.g.dart';

@DriftDatabase(tables: [TodoItems, Users, Posts])
class AppDatabase extends _$AppDatabase {
  // 👇 Flexible constructor
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Static connection method
  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'my_app_database',
      native: const DriftNativeOptions(
        // 👇 Choose storage location
        databaseDirectory: getApplicationDocumentsDirectory,
        // 👇 Optional: Use a specific file name
        // file: 'custom_name.db', // Overrides name
        // 👇 Optional: Enable background isolate
        // enableBackgroundIsolate: true,
      ),
      // 👇 Optional: Web configuration
      web: DriftWebOptions(
        sqlite3Wasm: Uri.parse('sqlite3.wasm'),
        driftWorker: Uri.parse('drift_worker.js'),
      ),
    );
  }

  // Custom queries
  Future<List<TodoItem>> getRecentTodos() async {
    return await (select(todoItems)
      ..orderBy([(t) => OrderingTerm(expression: t.createdAt, mode: OrderingMode.desc)])
      ..limit(10))
    .get();
  }

  Stream<List<TodoItem>> watchAllTodos() {
    return select(todoItems).watch();
  }
}
```

> **Key insights:**
> - **`getApplicationDocumentsDirectory`** – Standard location for user documents
> - **`getApplicationSupportDirectory`** – Better for app data (not user-visible)
> - **`enableBackgroundIsolate`** – Moves database work off UI thread automatically
> - **Web support** – Handles WASM loading and worker setup

---

# Alternative 1: NativeDatabase (Direct)

> **The simplest approach without drift_flutter**

```dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';
import 'package:sqlite3_flutter_libs/sqlite3_flutter_libs.dart';

part 'database.g.dart';

@DriftDatabase(tables: [TodoItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return LazyDatabase(() async {
      final dir = await getApplicationDocumentsDirectory();
      final file = p.join(dir.path, 'app.db');
      
      // Open database with custom options
      final db = await databaseFactoryIo.openDatabase(
        file,
        options: OpenDatabaseOptions(
          setup: (db) async {
            await db.execute('PRAGMA journal_mode = WAL');
            await db.execute('PRAGMA foreign_keys = ON');
          },
        ),
      );
      
      return NativeDatabase(db);
    });
  }
}
```

> **Pros:** Full control over configuration
> **Cons:** More boilerplate, manual setup required

---

# Alternative 2: NativeDatabase with Background Isolate

> **For performance-critical apps**

```dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

part 'database.g.dart';

@DriftDatabase(tables: [TodoItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return NativeDatabase.createInBackground(
      File(p.join(getApplicationDocumentsDirectory().path, 'app.db')),
      setup: (db) async {
        await db.execute('PRAGMA journal_mode = WAL');
        await db.execute('PRAGMA synchronous = NORMAL');
        await db.execute('PRAGMA foreign_keys = ON');
        await db.execute('PRAGMA cache_size = 10000');
      },
      // 👇 Enable read pool for concurrent queries
      readPool: 4,
    );
  }
}
```

> **Pros:** Background isolate, concurrent reads
> **Cons:** More complex setup, WAL mode required

---

# Alternative 3: In-Memory Database (Testing)

> **For unit tests and fast prototyping**

```dart
// test/database_test.dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:test/test.dart';
import '../lib/database/database.dart';

void main() {
  late AppDatabase db;

  setUp(() {
    // 👇 In-memory database
    db = AppDatabase(NativeDatabase.memory());
  });

  tearDown(() async {
    await db.close();
  });

  test('should insert and retrieve todo', () async {
    await db.into(db.todoItems).insert(
      TodoItemsCompanion(
        title: Value('Test Todo'),
        content: Value('Test Content'),
      ),
    );
    
    final todos = await db.select(db.todoItems).get();
    expect(todos.length, 1);
    expect(todos.first.title, 'Test Todo');
  });
}
```

> **Pros:** Fast, isolated, no cleanup needed
> **Cons:** Data not persisted

---

# Alternative 4: SqfliteQueryExecutor (Legacy)

> **For apps using sqflite**

```dart
import 'package:drift/drift.dart';
import 'package:drift/sqflite.dart';
import 'package:sqflite/sqflite.dart' as sqflite;
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

part 'database.g.dart';

@DriftDatabase(tables: [TodoItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return LazySqfliteExecutor(
      databaseFactory: sqflite.databaseFactory,
      path: p.join(getApplicationDocumentsDirectory().path, 'app.db'),
      onConfigure: (db) async {
        await db.execute('PRAGMA foreign_keys = ON');
      },
    );
  }
}
```

> **Pros:** Compatible with existing sqflite code
> **Cons:** Older, less performant, no isolate support

---

# Alternative 5: Web Platform (Pure)

> **For web-only apps**

```dart
import 'package:drift/drift.dart';
import 'package:drift/wasm.dart';

part 'database.g.dart';

@DriftDatabase(tables: [TodoItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return LazyDatabase(() async {
      final db = await databaseFactoryWeb.openDatabase(
        'app.db',
        options: OpenDatabaseOptions(
          setup: (db) async {
            await db.execute('PRAGMA foreign_keys = ON');
          },
        ),
      );
      return WasmDatabase(db);
    });
  }
}
```

> **Pros:** Lightweight, pure web
> **Cons:** Manual WASM setup

---

# Alternative 6: Multiple Databases

> **For apps with separate databases per feature**

```dart
// lib/database/main_database.dart
@DriftDatabase(tables: [Users, Posts])
class MainDatabase extends _$MainDatabase {
  MainDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());
  
  @override
  int get schemaVersion => 1;
  
  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'main');
  }
}

// lib/database/settings_database.dart
@DriftDatabase(tables: [Settings])
class SettingsDatabase extends _$SettingsDatabase {
  SettingsDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());
  
  @override
  int get schemaVersion => 1;
  
  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'settings');
  }
}

// Usage
final mainDb = MainDatabase();
final settingsDb = SettingsDatabase();
```

> **Pros:** Separation of concerns, better organization
> **Cons:** More connections, potential for race conditions

---

# Alternative 7: Custom Executor with Interceptors

> **For logging, monitoring, and debugging**

```dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

class LoggingQueryExecutor extends QueryExecutor {
  final QueryExecutor _base;
  
  LoggingQueryExecutor(this._base);
  
  @override
  Future<T> run<T>(QueryContext<T> context) async {
    print('🔍 Executing: ${context.sql}');
    print('📝 Arguments: ${context.arguments}');
    
    final start = DateTime.now();
    final result = await _base.run(context);
    final duration = DateTime.now().difference(start);
    
    print('✅ Completed in ${duration.inMilliseconds}ms');
    return result;
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
}

// Usage
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    final base = driftDatabase(name: 'my_app');
    return LoggingQueryExecutor(base); // 👈 Wrap with interceptor
  }
}
```

> **Pros:** Debugging, performance monitoring
> **Cons:** Overhead in production

---

# Real-World Example

> **Complete production setup with all the bells and whistles**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';
import 'package:path_provider/path_provider.dart';

import 'tables/todos.dart';
import 'tables/users.dart';
import 'tables/categories.dart';

part 'database.g.dart';

@DriftDatabase(tables: [Todos, Users, Categories])
class AppDatabase extends _$AppDatabase {
  // 👇 Accept optional executor for testing
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 2;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (migrator) async {
      await migrator.createAll();
      print('✅ Database created with all tables');
    },
    onUpgrade: (migrator, from, to) async {
      if (from == 1) {
        await migrator.addColumn(todos, todos.categoryId);
        print('✅ Migrated from version 1 to 2');
      }
    },
  );

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'production_database',
      native: const DriftNativeOptions(
        databaseDirectory: getApplicationSupportDirectory,
        // 👇 Enable background isolate for better performance
        enableBackgroundIsolate: true,
      ),
      web: DriftWebOptions(
        sqlite3Wasm: Uri.parse('sqlite3.wasm'),
        driftWorker: Uri.parse('drift_worker.js'),
        // 👇 Cache database in IndexedDB for web
        cacheId: 'my_app_cache',
      ),
    );
  }

  // 👇 Business methods
  Future<List<Todo>> getPendingTodos() async {
    return await (select(todos)
      ..where((t) => t.isCompleted.equals(false))
      ..orderBy([(t) => OrderingTerm(expression: t.dueDate)]))
    .get();
  }

  Stream<List<Todo>> watchTodosByCategory(int categoryId) {
    return (select(todos)
      ..where((t) => t.categoryId.equals(categoryId)))
    .watch();
  }

  Future<int> createTodo(String title, String content, int categoryId) async {
    return await into(todos).insert(
      TodosCompanion(
        title: Value(title),
        content: Value(content),
        categoryId: Value(categoryId),
        createdAt: Value(DateTime.now()),
        isCompleted: const Value(false),
      ),
    );
  }

  Future<void> markTodoComplete(int id) async {
    await (update(todos)..where((t) => t.id.equals(id)))
      .write(TodosCompanion(isCompleted: const Value(true)));
  }

  Future<void> deleteTodo(int id) async {
    await (delete(todos)..where((t) => t.id.equals(id))).go();
  }
}

// 👇 Easy access throughout app
class DatabaseProvider {
  static AppDatabase? _instance;
  
  static AppDatabase get instance {
    _instance ??= AppDatabase();
    return _instance!;
  }
}

// 👇 For testing
class TestDatabase extends AppDatabase {
  TestDatabase() : super(NativeDatabase.memory());
}
```

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'database/database.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize database
  final db = DatabaseProvider.instance;
  
  runApp(MyApp(db: db));
}

class MyApp extends StatelessWidget {
  final AppDatabase db;
  
  const MyApp({required this.db});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: TodoListPage(db: db),
    );
  }
}

class TodoListPage extends StatelessWidget {
  final AppDatabase db;
  
  const TodoListPage({required this.db});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Todos')),
      body: StreamBuilder(
        stream: db.watchTodosByCategory(1),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return Center(child: CircularProgressIndicator());
          }
          
          final todos = snapshot.data!;
          if (todos.isEmpty) {
            return Center(child: Text('No todos yet'));
          }
          
          return ListView.builder(
            itemCount: todos.length,
            itemBuilder: (context, index) {
              final todo = todos[index];
              return ListTile(
                title: Text(todo.title),
                subtitle: Text(todo.content),
                trailing: Checkbox(
                  value: todo.isCompleted,
                  onChanged: (_) => db.markTodoComplete(todo.id),
                ),
                onLongPress: () => db.deleteTodo(todo.id),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // Add new todo
          db.createTodo('New Todo', 'Todo content', 1);
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

> **Benefits of this approach:**
> - **Clean separation** – Database logic isolated from UI
> - **Platform-aware** – Works on mobile, desktop, and web
> - **Testable** – In-memory for unit tests
> - **Reactive** – Streams keep UI updated automatically
> - **Optimized** – Background isolate for performance
> - **Maintainable** – Easy to add new tables and queries

---

# Best Practices

- **Use `drift_flutter`** – The recommended way for Flutter apps
- **Accept optional executor** – Makes testing easy
- **Use `getApplicationSupportDirectory`** – Better for app data
- **Enable background isolates** – `enableBackgroundIsolate: true`
- **Set up migrations** – Handle schema changes gracefully
- **Use streams for UI** – Keep UI reactive
- **Close database** – Call `close()` when done
- **Singleton pattern** – One database instance per app
- **Separate tables** – Each table in its own file
- **Add logging** – Use interceptors during development

---

# Common Mistakes

## Mistake 1: Not using `drift_flutter`

Wrong:
```dart
// 🚫 Manual setup, more boilerplate
static QueryExecutor _openConnection() {
  return LazyDatabase(() async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File(p.join(dir.path, 'app.db'));
    // ... lots of code
  });
}
```

Correct:
```dart
// ✅ Platform-aware, less code
static QueryExecutor _openConnection() {
  return driftDatabase(name: 'my_app');
}
```

## Mistake 2: Forgetting web configuration

Wrong:
```dart
// 🚫 Web will crash
static QueryExecutor _openConnection() {
  return driftDatabase(name: 'my_app');
}
```

Correct:
```dart
// ✅ Web works
static QueryExecutor _openConnection() {
  return driftDatabase(
    name: 'my_app',
    web: DriftWebOptions(
      sqlite3Wasm: Uri.parse('sqlite3.wasm'),
      driftWorker: Uri.parse('drift_worker.js'),
    ),
  );
}
```

## Mistake 3: Not closing database

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
// ✅ Clean up
void main() async {
  final db = AppDatabase();
  // Use database...
  await db.close(); // Close when done
}
```

## Mistake 4: Using `get()` for reactive UI

Wrong:
```dart
// 🚫 Won't update
final todos = await db.select(db.todos).get();
```

Correct:
```dart
// ✅ Auto-updates
final stream = db.select(db.todos).watch();
```

---

# Summary

| Approach | Best For | Key Feature |
|----------|----------|-------------|
| **drift_flutter** | Flutter apps | Platform-aware, simple |
| **NativeDatabase** | Full control | Custom configuration |
| **Background Isolate** | Performance | Off-UI thread |
| **In-Memory** | Testing | Fast, isolated |
| **SqfliteExecutor** | Legacy apps | Sqflite compatibility |
| **Web** | Web-only | Lightweight |
| **Multiple DBs** | Large apps | Separation of concerns |
| **Interceptors** | Debugging | Logging, monitoring |

---

# Next Steps

Now your database is set up, let's dive deeper:

- [Code Generation](link) – Understand how Drift generates code
- [Your First Query](link) – Write and execute your first query
- [Tables Deep Dive](link) – Advanced table definitions

---

# Did You Know?

- **`drift_flutter` automatically handles** different platforms – you don't need conditional imports

- **`enableBackgroundIsolate: true`** moves all database operations to a background isolate, preventing UI jank

- **The `driftDatabase()` helper** creates a `LazyDatabase` internally, so the connection is only opened on first use

- **Drift supports both `getApplicationDocumentsDirectory` and `getApplicationSupportDirectory`** – choose based on whether users should see the database files

- **Web support requires the SQLite WASM file** – it's automatically handled when using `drift_flutter` with web configuration

- **You can have multiple databases** in the same app – just create separate database classes with different names

- **Drift's background isolates** automatically handle WAL mode – you don't need to configure it manually when using `drift_flutter`

---

## 🎯 **That's the Complete Guide: Setting Up a Database!**

