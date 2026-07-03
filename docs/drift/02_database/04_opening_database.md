# Opening the Database

Open a SQLite database so Drift can read and write data.

---

# What is it?

Opening the database is the process of creating a connection between your Drift database and the underlying SQLite database.

This is typically done once when your application starts. After the database is opened, all queries, inserts, updates, and deletes use the same connection.

The way you open a database depends on your platform and storage requirements.

---

# Why does it exist?

Drift separates **defining a database** from **opening a database**.

Your database class describes:

* Tables
* DAOs
* Schema version
* Migrations

Opening the database determines:

* Where the database is stored.
* How SQLite is initialized.
* Which executor is used.
* How the connection is managed.

This separation makes Drift flexible enough to support Flutter, Dart, web, testing, and custom database implementations.

---

# Syntax

## Using `drift_flutter`

The simplest way to open a database in a Flutter application is with `drift_flutter`.

```dart
final database = AppDatabase(
  driftDatabase(
    name: 'app.db',
  ),
);
```

Explanation:

* `driftDatabase()` creates a SQLite database connection.
* `name` specifies the database file.
* `AppDatabase` uses this connection for all operations.

---

## Using an In-Memory Database

```dart
final database = AppDatabase(
  NativeDatabase.memory(),
);
```

Explanation:

* Creates a temporary database stored entirely in memory.
* All data is lost when the database is closed.
* Commonly used for testing.

---

# Mental Model

Opening the database is like opening a file before editing it.

```text
Application
      │
      ▼
Open Database
      │
      ▼
SQLite Connection
      │
      ▼
Read / Write Data
```

Once the connection is established, every database operation uses it.

---

# Examples

## Simple Example

Create a persistent local database.

```dart
final db = AppDatabase(
  driftDatabase(name: 'todo.db'),
);
```

Explanation:

* Creates (or opens) `todo.db`.
* Existing data is preserved between app launches.

---

## Real-World Example

A Flutter application usually creates the database once during startup.

```dart
final database = AppDatabase(
  driftDatabase(name: 'company.db'),
);
```

The database instance is then shared with repositories, DAOs, or dependency injection.

Explanation:

* Opening the database once avoids repeatedly creating new connections.

---

# When to Use

Open the database when:

* Your application starts.
* Initializing dependency injection.
* Creating the application's database instance.
* Setting up an in-memory database for tests.

---

# When NOT to Use

Avoid opening the database:

* Before every query.
* Inside widgets that rebuild frequently.
* Inside methods that are called repeatedly.

Instead, create one shared database instance and reuse it.

---

# Best Practices

* Open the database only once.
* Reuse the same database instance.
* Choose the appropriate executor for your platform.
* Use an in-memory database for testing.
* Close the database when it's no longer needed.

---

# Common Mistakes

## Opening the database repeatedly

**Wrong**

```dart
Future<void> loadData() async {
  final db = AppDatabase(
    driftDatabase(name: 'app.db'),
  );

  await db.select(db.users).get();
}
```

Explanation:

* A new connection is created every time the method runs.

**Correct**

```dart
final db = AppDatabase(
  driftDatabase(name: 'app.db'),
);

Future<void> loadData() async {
  await db.select(db.users).get();
}
```

Explanation:

* A single database instance is reused throughout the application.

---

## Opening the database inside a widget

**Wrong**

```dart
class HomePage extends StatelessWidget {
  final db = AppDatabase(
    driftDatabase(name: 'app.db'),
  );
}
```

Explanation:

* Widgets shouldn't be responsible for creating database connections.
* This makes dependency management more difficult.

**Correct**

Create the database during application initialization and inject it where needed.

---

## Using an in-memory database unintentionally

**Wrong**

```dart
AppDatabase(
  NativeDatabase.memory(),
);
```

Explanation:

* All data disappears when the application exits.

**Correct**

Use a persistent database for production and reserve in-memory databases for testing.

---

# Related APIs

* Database Connection
* Database Lifecycle
* GeneratedDatabase
* Multiple Databases
* In-Memory Database

---

# Summary

Opening the database establishes the connection between Drift and SQLite. This connection is typically created once during application startup and reused throughout the app. By separating database configuration from connection creation, Drift supports multiple platforms and connection strategies while keeping your database code clean and maintainable.
