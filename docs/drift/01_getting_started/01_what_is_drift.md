

# What is Drift?

**Understand the reactive SQLite ORM for Flutter & Dart**

---

# What is it?

**Drift** (formerly known as `moor`) is a **reactive persistence library** for Dart and Flutter that provides a type-safe way to work with SQLite databases. It combines the power of raw SQL with the safety and convenience of modern Dart features.

> **Think of Drift as your database "superpower"** – it catches SQL errors at compile-time, automatically generates boilerplate code, and turns your database into reactive streams that update your UI automatically.

```dart
// Without Drift (raw SQLite)
final result = await database.rawQuery(
  'SELECT * FROM users WHERE age > ?',
  [18]
);
final name = result[0]['name'] as String; // Manual casting 😰

// With Drift (type-safe)
final users = await (select(users)
  ..where((u) => u.age > const Variable(18)))
  .get();
final name = users.first.name; // Type-safe! 🎯
```

> **What's happening here?**
> - **Raw SQLite**: Returns `List<Map<String, dynamic>>` – no type safety, manual casting required
> - **Drift**: Returns `List<User>` – fully typed, auto-generated data classes
> - **Compile-time safety**: SQL errors are caught during development, not at runtime
> - **Automatic mapping**: Drift handles the conversion between SQL rows and Dart objects

---

# Why does it exist?

- **Type Safety** – Catch SQL syntax and type errors at compile-time, not runtime. No more `type 'int' is not a subtype of 'String'` crashes!

- **Reactive Queries** – Turn any query into a `Stream` that automatically emits new results when the database changes. Your UI stays in sync without manual refresh logic.

- **Boilerplate Reduction** – Drift generates all the boring CRUD code, data classes, and table mappings for you. Write less code, do more.

- **SQL Flexibility** – Write complex SQL queries with JOINs, CTEs, and window functions, but with Dart's type safety and autocomplete.

- **Dart-First Design** – Everything is designed to feel natural in Dart – use Dart syntax for queries, streams for reactivity, and Futures for async operations.

- **Platform Agnostic** – Works everywhere Dart runs: Flutter (iOS, Android, Web, Desktop), pure Dart CLI apps, and server-side applications.

---

# Core Architecture

> **[Understanding the building blocks of Drift]**

```dart
// 1. Define your tables
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  IntColumn get age => integer()();
}

// 2. Create the database
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  
  @override
  int get schemaVersion => 1;
}

// 3. Write type-safe queries
Future<List<User>> getAdultUsers() async {
  return await (select(users)
    ..where((u) => u.age > const Variable(18)))
    .get();
}

// 4. Reactive streams
Stream<List<User>> watchAllUsers() {
  return select(users).watch();
}
```

> **Key insights:**
> - **Tables are Dart classes** extending `Table` – each column is a property
> - **Database class** uses `@DriftDatabase` annotation to generate all the code
> - **Queries are composable** – you chain methods like `where()`, `orderBy()`, `limit()`
> - **Streams are built-in** – every query can become a reactive stream with `.watch()`
> - **Code generation** happens at build time – zero runtime reflection

---

# Real-World Example

> **Practical use case: A Todo App with real-time updates**

```dart
// todos.dart - Table definition
import 'package:drift/drift.dart';

class Todos extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text().nullable()();
  BoolColumn get isCompleted => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime()();
}

// database.dart - Database setup
@DriftDatabase(tables: [Todos])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  
  @override
  int get schemaVersion => 1;
  
  // Insert a new todo
  Future<int> insertTodo(Insertable<Todo> todo) {
    return into(todos).insert(todo);
  }
  
  // Watch all incomplete todos (reactive!)
  Stream<List<Todo>> watchIncompleteTodos() {
    return (select(todos)
      ..where((t) => t.isCompleted.equals(false))
      ..orderBy([(t) => OrderingTerm(expression: t.createdAt)]))
    .watch();
  }
  
  // Mark todo as complete
  Future<void> completeTodo(int id) async {
    await (update(todos)..where((t) => t.id.equals(id)))
      .write(TodosCompanion(isCompleted: const Value(true)));
  }
}

// main.dart - Usage in Flutter
class TodoList extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final db = Provider.of<AppDatabase>(context);
    
    return StreamBuilder<List<Todo>>(
      stream: db.watchIncompleteTodos(),
      builder: (context, snapshot) {
        if (!snapshot.hasData) return CircularProgressIndicator();
        
        final todos = snapshot.data!;
        return ListView.builder(
          itemCount: todos.length,
          itemBuilder: (context, index) {
            final todo = todos[index];
            return ListTile(
              title: Text(todo.title),
              subtitle: Text(todo.content ?? 'No content'),
              trailing: Checkbox(
                value: todo.isCompleted,
                onChanged: (_) => db.completeTodo(todo.id),
              ),
            );
          },
        );
      },
    );
  }
}
```

> **Benefits of this approach:**
> - **Auto-updating UI** – The `StreamBuilder` rebuilds whenever todos change
> - **No manual refresh** – Drift handles all the reactive wiring
> - **Type-safe everywhere** – `Todo` objects have proper types
> - **Separation of concerns** – Database logic stays in its own layer
> - **Testable** – Database can be mocked or use in-memory for tests

---

# Best Practices

- **Use `@DriftDatabase`** – Let Drift generate the boilerplate for you
- **Keep tables in separate files** – Better organization and maintainability
- **Use `Companion` objects for inserts** – They handle optional fields properly
- **Prefer streams over `get()` for UI** – Streams keep your UI reactive
- **Name your tables and columns clearly** – Good names make queries self-documenting
- **Use `Value<T>` for nullable fields** – It clearly distinguishes null vs absent values

---

# Common Mistakes

## Mistake 1: Forgetting to run `build_runner`

Wrong:
```bash
# Nothing happens, code not generated
flutter run
```

Correct:
```bash
# Generate the code first
flutter pub run build_runner build
# Or watch for changes
flutter pub run build_runner watch
```

## Mistake 2: Using `get()` when you need `watch()`

Wrong:
```dart
// UI won't update when data changes
final todos = await select(users).get();
```

Correct:
```dart
// UI updates automatically
final stream = select(users).watch();
```

## Mistake 3: Not handling schema migrations

Wrong:
```dart
@override
int get schemaVersion => 2; // Changed version, no migration
```

Correct:
```dart
@override
int get schemaVersion => 2;

@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (migrator, from, to) async {
    if (from == 1) {
      await migrator.addColumn(users, users.age);
    }
  },
);
```

---

# Summary

| Concept | Key Takeaway |
|---------|--------------|
| **What is Drift?** | A type-safe, reactive ORM for SQLite in Dart |
| **Why use it?** | Compile-time safety, reactive streams, less boilerplate |
| **Core features** | Type-safe queries, reactive streams, code generation |
| **Best for** | Flutter apps needing local data with real-time UI updates |
| **Not for** | Very simple apps where raw SQLite is sufficient |

---

# Next Steps

Now that you understand what Drift is, let's get it installed and running:

- [Installation](link) – Add Drift to your project
- [Setting Up a Database](link) – Create your first database
- [Code Generation](link) – Set up build_runner

---

# Did You Know?

- **Drift was originally called "moor"** – named after the moorland in the UK, but renamed to Drift in 2021 to avoid confusion with the Moor testing framework

- **Drift supports SQLite, Postgres, and MySQL** – though SQLite is the most popular for Flutter

- **The reactive streams are powered by `StreamController`** – Drift uses a clever system to detect changes and only emit new results when data actually changes

- **Drift can generate CRUD operations automatically** – with `@UseRowClass`, you don't even need to write queries for basic operations

- **Drift uses `sqlite3` under the hood** – not the built-in SQLite in Flutter, giving you more features and better performance

---
