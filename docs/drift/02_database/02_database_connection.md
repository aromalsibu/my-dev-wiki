# Database Connection

A database connection is the bridge between your Drift database and the underlying SQLite database.

---

# What is it?

Before Drift can read or write data, it needs a connection to a SQLite database. This connection is provided through a **database executor**, which knows how to open the database file and execute SQL statements.

When you create your `AppDatabase`, you pass it a database connection.

```dart
final db = AppDatabase(connection);
```

Without a connection, your database cannot perform any operations.

---

# Why does it exist?

Drift separates the **database logic** from the **database connection**.

This separation allows Drift to:

* Support multiple platforms.
* Support different SQLite implementations.
* Make databases easier to test.
* Allow custom connection strategies.

Instead of hardcoding how the database is opened, Drift lets you provide the connection when creating the database.

---

# Syntax

## Passing a Connection

```dart
final database = AppDatabase(connection);
```

Explanation:

* `connection` is responsible for communicating with SQLite.
* `AppDatabase` uses this connection for all database operations.

---

## Creating a Connection with `drift_flutter`

```dart
final database = AppDatabase(
  driftDatabase(
    name: 'app.db',
  ),
);
```

Explanation:

* `driftDatabase()` creates a SQLite database connection for Flutter.
* `name` specifies the database file name.
* Drift automatically manages opening the database.

> **Note:** There are multiple ways to create a database connection depending on your platform and storage requirements. This page focuses on the concept of a database connection. Specific connection methods are covered in **Opening the Database**.

---

# Mental Model

Think of the connection as a bridge.

```text
Flutter App
      │
      ▼
 AppDatabase
      │
      ▼
Database Connection
      │
      ▼
 SQLite
```

Your application never communicates with SQLite directly.

Every query flows through the database connection.

---

# Examples

## Simple Example

Creating a database instance.

```dart
final db = AppDatabase(
  driftDatabase(name: 'todo.db'),
);
```

Explanation:

* A connection is created for the `todo.db` database.
* The same connection is used for every query executed through `db`.

---

## Real-World Example

A larger application might create the database once and share it throughout the app.

```dart
final database = AppDatabase(
  driftDatabase(name: 'company.db'),
);
```

The database instance is then injected into repositories, DAOs, or services, ensuring every part of the application uses the same connection.

Explanation:

* Sharing a single database instance avoids opening multiple connections unnecessarily.

---

# When to Use

Create a database connection when:

* Initializing your application.
* Creating your database instance.
* Connecting to a SQLite database.
* Setting up an in-memory database for testing.

---

# When NOT to Use

Avoid creating a new connection every time you need to execute a query.

Instead, create one database instance and reuse it throughout your application.

---

# Best Practices

* Create a single database instance for your application.
* Reuse the same connection whenever possible.
* Close the database when it's no longer needed.
* Keep connection creation separate from business logic.
* Use different connection strategies for production and testing when appropriate.

---

# Common Mistakes

## Creating multiple database instances

**Wrong**

```dart
Future<void> loadUsers() async {
  final db = AppDatabase(
    driftDatabase(name: 'app.db'),
  );

  await db.select(db.users).get();
}
```

Explanation:

* A new connection is opened every time the method runs.
* This wastes resources and can lead to unexpected behavior.

**Correct**

```dart
final db = AppDatabase(
  driftDatabase(name: 'app.db'),
);

Future<void> loadUsers() async {
  await db.select(db.users).get();
}
```

Explanation:

* Reuse a single database instance throughout the application.

---

## Mixing connection logic with application logic

**Wrong**

Creating database connections in multiple widgets or services.

Explanation:

* Connection management becomes difficult.
* Multiple database instances may be created accidentally.

**Correct**

Create the database once during application startup and inject it where needed.

---

## Forgetting to close the database

**Wrong**

Creating a database connection without ever disposing it.

Explanation:

* Resources remain allocated longer than necessary.

**Correct**

Close the database when the application or test no longer needs it.

```dart
await db.close();
```

Explanation:

* Releases the underlying database resources.

---

# Related APIs

* GeneratedDatabase
* Opening the Database
* Database Lifecycle
* Multiple Databases
* In-Memory Database

---

# Summary

A database connection allows Drift to communicate with SQLite. It acts as the bridge between your application and the database engine. By creating a single shared connection and passing it to your database class, you ensure efficient, reliable, and maintainable database access throughout your application.
