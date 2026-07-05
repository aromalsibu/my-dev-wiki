# Filtering

Filtering limits a query to only the rows that match a specific condition.

---

# What is it?

Filtering is the process of selecting only the records you need instead of retrieving every row from a table.

In Drift, filtering is done using the `where()` method.

Without filtering, a query returns all rows in the table.

---

# Why does it exist?

Imagine a `users` table with thousands of records.

Most of the time, you don't need every user. You might only want:

* A specific user.
* Active users.
* Products under a certain price.
* Completed tasks.

Filtering allows SQLite to return only the rows that satisfy your conditions.

---

# Syntax

## Filtering by a Single Condition

```dart
final user = await (select(users)
      ..where((u) => u.id.equals(1)))
    .getSingle();
```

Explanation:

* `where()` adds a condition to the query.
* `u.id.equals(1)` matches rows whose `id` is `1`.
* Only matching rows are returned.

---

## Filtering Multiple Rows

```dart
final completedTasks = await (select(tasks)
      ..where((t) => t.completed.equals(true)))
    .get();
```

Explanation:

* Every completed task is returned.
* Rows that don't match the condition are excluded.

---

## Combining Conditions

```dart
final usersList = await (select(users)
      ..where(
        (u) =>
            u.age.isBiggerOrEqualValue(18) &
            u.isActive.equals(true),
      ))
    .get();
```

Explanation:

* `&` represents a logical **AND**.
* Both conditions must be true for a row to match.

---

# Mental Model

Think of filtering as placing a sieve over your data.

```text
Users

Alice   ✓
Bob     ✗
Carol   ✓
David   ✗

        │
        ▼
  Active Users

Alice
Carol
```

The filter removes rows that don't satisfy the condition.

---

# Examples

## Simple Example

Find a product by ID.

```dart
final product = await (select(products)
      ..where((p) => p.id.equals(5)))
    .getSingle();
```

Explanation:

* Returns the product whose ID is `5`.

---

## Real-World Example

Load all unread notifications.

```dart
final unread = await (select(notifications)
      ..where((n) => n.isRead.equals(false)))
    .get();
```

Explanation:

* Only unread notifications are returned.
* Read notifications are excluded.

---

# When to Use

Use filtering when you need to:

* Search for a specific record.
* Display subsets of data.
* Apply business rules.
* Retrieve only relevant rows.
* Improve query performance by avoiding unnecessary data.

---

# When NOT to Use

Avoid filtering when:

* You genuinely need every row.
* The filtering logic is better handled after retrieving a very small dataset.

For large datasets, filtering in the database is almost always preferable.

---

# Best Practices

* Filter as early as possible in the query.
* Use indexed columns for frequently filtered fields.
* Combine related conditions into a single `where()` clause.
* Keep filter expressions simple and readable.
* Let SQLite perform filtering instead of filtering in Dart.

---

# Common Mistakes

## Loading all rows and filtering in Dart

**Wrong**

```dart
final usersList = await select(users).get();

final activeUsers = usersList.where(
  (u) => u.isActive,
);
```

Explanation:

* Every row is loaded into memory.
* Filtering happens after the database query, which is inefficient.

**Correct**

```dart
final activeUsers = await (select(users)
      ..where((u) => u.isActive.equals(true)))
    .get();
```

Explanation:

* SQLite filters the rows before returning them.

---

## Using multiple `where()` calls incorrectly

**Wrong**

```dart
final query = select(users)
  ..where((u) => u.isActive.equals(true))
  ..where((u) => u.age.isBiggerThanValue(18));
```

Explanation:

* A select query supports a single `where()` clause.
* Multiple conditions should be combined into one expression.

**Correct**

```dart
final query = select(users)
  ..where(
    (u) =>
        u.isActive.equals(true) &
        u.age.isBiggerThanValue(18),
  );
```

Explanation:

* Both conditions are combined into a single filter.

---

## Forgetting comparison methods

**Wrong**

```dart
u.id == 1
```

Explanation:

* Drift expressions use SQL operators, not Dart's comparison operators.

**Correct**

```dart
u.id.equals(1)
```

Explanation:

* `equals()` generates the appropriate SQL comparison.

---

# Related APIs

* Select
* Ordering
* Limiting
* Expressions
* Dynamic Queries

---

# Summary

Filtering allows you to retrieve only the rows that match specific conditions using the `where()` method. By pushing filtering into SQLite instead of Dart, your queries become more efficient, use less memory, and return only the data your application actually needs.
