# Database Lifecycle

Understand how a Drift database is created, used, and closed during your application's lifetime.

---

# What is it?

The database lifecycle refers to the different stages a Drift database goes through while your application is running.

A typical lifecycle looks like this:

```text
Create Database
       │
       ▼
Open Connection
       │
       ▼
Execute Queries
       │
       ▼
Watch Streams
       │
       ▼
Close Database
```

Most applications create the database once when the app starts, use it throughout the app's lifetime, and close it when it's no longer needed.

---

# Why does it exist?

Opening a SQLite database is relatively expensive compared to executing queries.

Continuously opening and closing the database for every operation can:

* Reduce performance.
* Waste system resources.
* Create multiple unnecessary connections.
* Make state management more difficult.

By treating the database as a long-lived object, Drift provides better performance and a more predictable architecture.

---

# Syntax

## Creating the Database

```dart
final db = AppDatabase(
  driftDatabase(name: 'app.db'),
);
```

Explanation:

* Creates the database instance.
* Opens the connection when needed.

---

## Using the Database

```dart
final users = await db.select(db.users).get();
```

Explanation:

* Executes queries through the existing database instance.
* No new connection is created.

---

## Closing the Database

```dart
await db.close();
```

Explanation:

* Closes the database connection.
* Releases any underlying resources.

---

# Mental Model

Think of the database like a file.

```text
Open File
    │
Read / Write
    │
Read / Write
    │
Read / Write
    │
Close File
```

You don't repeatedly open and close the file every time you read a line.

A database works the same way.

Open it once, use it many times, and close it when you're finished.

---

# Examples

## Simple Example

Application startup.

```dart
final db = AppDatabase(
  driftDatabase(name: 'todo.db'),
);
```

Throughout the application:

```dart
await db.select(db.tasks).get();

await db.into(db.tasks).insert(task);

await db.delete(db.tasks).go();
```

Finally:

```dart
await db.close();
```

Explanation:

* One database instance serves the entire application.
* The connection remains open until it's explicitly closed.

---

## Real-World Example

A Flutter application might create the database during dependency injection.

```text
App Starts
     │
     ▼
Create AppDatabase
     │
     ▼
Inject into DAOs
     │
     ▼
Repositories
     │
     ▼
UI
     │
     ▼
App Closes
     │
     ▼
Database Closed
```

Every layer shares the same database instance.

---

# When to Use

Follow the standard lifecycle when:

* Building Flutter applications.
* Using dependency injection.
* Working with repositories or DAOs.
* Managing a local SQLite database.

---

# When NOT to Use

Avoid:

* Creating a new database instance for every query.
* Closing the database immediately after each operation.
* Keeping multiple unnecessary database instances alive.

These patterns waste resources and complicate your application.

---

# Best Practices

* Create the database once.
* Share a single database instance.
* Close the database only when it's no longer needed.
* Inject the database into DAOs or repositories.
* Let the database live as long as your application requires.

---

# Common Mistakes

## Opening the database repeatedly

**Wrong**

```dart
Future<List<User>> loadUsers() async {
  final db = AppDatabase(
    driftDatabase(name: 'app.db'),
  );

  return db.select(db.users).get();
}
```

Explanation:

* A new database is created every time the method is called.

**Correct**

```dart
final db = AppDatabase(
  driftDatabase(name: 'app.db'),
);

Future<List<User>> loadUsers() {
  return db.select(db.users).get();
}
```

Explanation:

* The same database instance is reused.

---

## Closing the database too early

**Wrong**

```dart
await db.close();

await db.select(db.users).get();
```

Explanation:

* Queries cannot execute after the database has been closed.

**Correct**

Close the database only when the application is shutting down or when the database is no longer needed.

---

## Creating multiple shared databases

**Wrong**

```text
Widget A → AppDatabase()

Widget B → AppDatabase()

Widget C → AppDatabase()
```

Explanation:

* Multiple connections to the same database are unnecessary in most applications.

**Correct**

```text
AppDatabase
      │
 ┌────┼────┐
 │    │    │
DAO Repository UI
```

Explanation:

* Share a single database instance across the application.

---

# Related APIs

* Database Connection
* Opening the Database
* Multiple Databases
* Transactions
* DAOs
* In-Memory Database

---

# Summary

The database lifecycle describes how a Drift database is created, used, and eventually closed. In most applications, you should create a single database instance, reuse it throughout the app, and close it only when it's no longer needed. This approach improves performance, simplifies architecture, and ensures efficient resource management.
