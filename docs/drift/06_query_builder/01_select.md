# Select

Select retrieves rows from a database table.

---

# What is it?

A select query reads data from one or more tables.

Unlike insert, update, or delete operations, a select query does not modify the database. It simply returns the rows that match the query.

In Drift, the `select()` API is the starting point for most read operations.

---

# Why does it exist?

After storing data, your application needs a way to retrieve it.

For example:

* Display all users.
* Find a specific product.
* Show completed tasks.
* Load a user's orders.

Without select queries, data could be written to the database but never read back.

---

# Syntax

## Selecting All Rows

```dart
final allUsers = await select(users).get();
```

Explanation:

* `select(users)` creates a query for the `users` table.
* `get()` executes the query and returns all matching rows as generated data classes.

---

## Selecting a Single Row

```dart
final user = await (select(users)
      ..where((u) => u.id.equals(1)))
    .getSingle();
```

Explanation:

* `where()` filters the rows.
* `getSingle()` expects exactly one matching row.
* If zero or multiple rows are found, an exception is thrown.

---

## Selecting the First Matching Row

```dart
final user = await (select(users)
      ..where((u) => u.id.equals(1)))
    .getSingleOrNull();
```

Explanation:

* `getSingleOrNull()` returns the matching row.
* Returns `null` if no row matches.
* Throws an exception if multiple rows match.

---

# Mental Model

Think of a select query as searching a spreadsheet.

```text
Users

+----+--------+
| ID | Name   |
+----+--------+
| 1  | Alice  |
| 2  | Bob    |
| 3  | Carol  |
+----+--------+

        │
        ▼
Select ID = 2

        │
        ▼

Bob
```

The query searches the table and returns the matching rows.

---

# Examples

## Simple Example

Retrieve all products.

```dart
final products = await select(productsTable).get();

for (final product in products) {
  print(product.name);
}
```

Explanation:

* `get()` returns a list of generated data classes.
* Each item represents one database row.

---

## Real-World Example

Load a user's profile.

```dart
final profile = await (select(users)
      ..where((u) => u.email.equals(email)))
    .getSingleOrNull();
```

Explanation:

* Searches for a user by email.
* Returns `null` if the user doesn't exist.

---

# When to Use

Use select when you need to:

* Display records.
* Search for data.
* Load application state.
* Retrieve a specific row.
* Read data without modifying it.

---

# When NOT to Use

Don't use select when:

* Creating new records.
* Updating existing records.
* Deleting records.

Instead, use:

* `insert()` to create rows.
* `update()` to modify rows.
* `delete()` to remove rows.

---

# Best Practices

* Filter data using `where()` whenever possible.
* Use `getSingle()` only when exactly one row should exist.
* Use `getSingleOrNull()` for optional results.
* Retrieve only the data your application needs.
* Prefer reactive queries (`watch()`) when the UI should update automatically.

---

# Common Mistakes

## Loading an entire table unnecessarily

**Wrong**

```dart
final users = await select(users).get();
```

When you only need one user.

Explanation:

* Reads every row from the table.
* Can be inefficient for large datasets.

**Correct**

```dart
final user = await (select(users)
      ..where((u) => u.id.equals(1)))
    .getSingle();
```

Explanation:

* Retrieves only the required row.

---

## Using `getSingle()` when multiple rows may exist

**Wrong**

```dart
final task = await (select(tasks)
      ..where((t) => t.completed.equals(false)))
    .getSingle();
```

Explanation:

* Multiple incomplete tasks may exist.
* `getSingle()` will throw an exception.

**Correct**

```dart
final tasks = await (select(tasks)
      ..where((t) => t.completed.equals(false)))
    .get();
```

Explanation:

* `get()` returns all matching rows.

---

## Forgetting to execute the query

**Wrong**

```dart
final query = select(users);
```

Explanation:

* This only creates the query.
* No database operation occurs.

**Correct**

```dart
final usersList = await select(users).get();
```

Explanation:

* `get()` executes the query and returns the results.

---

# Related APIs

* Filtering
* Ordering
* Limiting
* Distinct
* Watching Queries
* Expressions

---

# Summary

`select()` is the foundation of reading data in Drift. It creates a query that can be refined with filters, ordering, limits, joins, and other query builder APIs before being executed with methods like `get()`, `getSingle()`, or `getSingleOrNull()`. It is the primary way to retrieve data from your database.
