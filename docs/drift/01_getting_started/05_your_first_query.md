# Your First Query

Retrieve data from your database using Drift's type-safe query builder.

---

# What is it?

A query is a request to read, insert, update, or delete data from your database. In Drift, queries are built using Dart APIs instead of writing raw SQL strings.

Your first query is typically a simple `SELECT` statement that retrieves all rows from a table.

Drift converts these Dart APIs into SQL behind the scenes, while ensuring type safety and compile-time validation.

---

# Why does it exist?

Writing SQL directly as strings has several drawbacks:

* SQL syntax errors are only discovered at runtime.
* Column names are manually typed and prone to mistakes.
* Mapping rows to Dart objects requires extra code.
* Refactoring becomes difficult.

Drift's query builder solves these problems by providing a fluent, strongly typed API that integrates naturally with Dart.

---

# Syntax

## Selecting All Rows

```dart
final users = await select(users).get();
```

Explanation:

* `select(users)` creates a query for the `users` table.
* `get()` executes the query.
* The result is returned as a list of generated data classes.

---

## Selecting a Single Row

```dart
final user = await (select(users)
      ..where((u) => u.id.equals(1)))
    .getSingle();
```

Explanation:

* `where()` adds a filter to the query.
* `equals()` creates a SQL equality condition.
* `getSingle()` expects exactly one matching row.

---

## Watching a Query

```dart
final stream = select(users).watch();
```

Explanation:

* `watch()` returns a stream instead of executing the query immediately.
* The stream emits new results whenever the table changes.

---

# Mental Model

Think of the query builder as constructing a SQL statement step by step.

```text
select(users)
      │
      ▼
Add filters
      │
      ▼
Add ordering
      │
      ▼
Execute
      │
      ▼
SQLite
      │
      ▼
Generated Data Classes
```

You describe *what* data you want, and Drift generates the corresponding SQL.

---

# Examples

## Simple Example

Retrieve every user from the database.

```dart
final allUsers = await select(users).get();
```

Explanation:

* Returns every row from the `users` table as a list of generated `User` objects.

---

## Real-World Example

Load all incomplete tasks.

```dart
final pendingTasks = await (select(tasks)
      ..where((t) => t.completed.equals(false)))
    .get();
```

Explanation:

* Filters the `tasks` table to include only incomplete tasks.
* Returns the matching rows as generated data classes.

---

# When to Use

Use Drift queries when you need to:

* Retrieve data from a table.
* Filter specific records.
* Load data for your UI.
* Fetch related information.
* Build reactive interfaces using streams.

---

# When NOT to Use

Avoid using the query builder when:

* You need SQL features that Drift doesn't support directly. In such cases, use custom SQL.
* You're performing database schema changes, which belong in migrations.

---

# Best Practices

* Prefer the query builder over raw SQL whenever possible.
* Use `watch()` for data that should automatically update in the UI.
* Keep complex queries inside DAOs.
* Fetch only the data you need.
* Reuse query logic instead of duplicating it throughout the application.

---

# Common Mistakes

## Using `getSingle()` for multiple rows

**Wrong**

```dart
final user = await select(users).getSingle();
```

Explanation:

* `getSingle()` throws an exception unless exactly one row exists.

**Correct**

```dart
final usersList = await select(users).get();
```

Explanation:

* `get()` is intended for queries that return multiple rows.

---

## Using `get()` when reactive updates are needed

**Wrong**

```dart
final users = await select(users).get();
```

Explanation:

* The result is fetched only once and won't update automatically.

**Correct**

```dart
final usersStream = select(users).watch();
```

Explanation:

* `watch()` keeps the UI synchronized with database changes.

---

## Writing raw SQL unnecessarily

**Wrong**

```dart
customSelect('SELECT * FROM users');
```

Explanation:

* Raw SQL bypasses Drift's type-safe APIs and compile-time checks.

**Correct**

```dart
select(users).get();
```

Explanation:

* The query builder is safer, easier to refactor, and integrates with generated types.

---

# Related APIs

* Select
* Filtering
* Ordering
* Limiting
* Watching Queries
* Insert
* Update
* Delete
* Custom SQL

---

# Summary

Drift's query builder provides a type-safe way to retrieve data from your database. Instead of writing SQL strings, you build queries using Dart APIs, and Drift generates the SQL automatically. This approach reduces boilerplate, improves type safety, and makes database code easier to read and maintain.
