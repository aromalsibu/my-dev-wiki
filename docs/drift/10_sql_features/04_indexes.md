# Indexes

**Indexes** are database structures that speed up queries by allowing SQLite to locate rows efficiently without scanning an entire table.

---

# What is it?

By default, SQLite searches a table row by row to find matching data. This is known as a **full table scan**.

As your tables grow, full table scans become increasingly expensive.

An index is a separate data structure that stores values from one or more columns in a sorted format, allowing SQLite to quickly locate matching rows.

For example, instead of checking every user to find one with a specific email, SQLite can use an index to jump directly to the matching row.

---

# Why does it exist?

Imagine a table with one million users.

Without an index:

```text
Find user by email
        │
        ▼
Check Row 1
Check Row 2
Check Row 3
...
Check Row 1,000,000
```

With an index:

```text
Find user by email
        │
        ▼
Use Index
        │
        ▼
Jump directly to matching row
```

Indexes dramatically reduce the amount of work SQLite needs to perform, making queries much faster.

The trade-off is that indexes consume additional storage and slightly slow down insert, update, and delete operations because the index must also be updated.

---

# Syntax

Create an index on a column.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get email => text()();

  @override
  List<Index> get indexes => [
        Index('email_index', [email]),
      ];
}
```

Explanation:

* `indexes` defines custom indexes for the table.
* `email` is indexed to speed up lookups by email.
* Drift creates the index when the database schema is created or migrated.

---

Create a composite index.

```dart
class Orders extends Table {
  IntColumn get customerId => integer()();

  DateTimeColumn get createdAt => dateTime()();

  @override
  List<Index> get indexes => [
        Index(
          'customer_date_index',
          [customerId, createdAt],
        ),
      ];
}
```

Explanation:

* The index covers both `customerId` and `createdAt`.
* Useful for queries that filter or sort using both columns.

---

# Mental Model

Think of an index like the index in a book.

Without an index:

```text
Open Book
      │
      ▼
Read every page
      │
      ▼
Find topic
```

With an index:

```text
Open Index
      │
      ▼
Find topic
      │
      ▼
Jump directly to page
```

SQLite uses indexes in the same way to locate rows efficiently.

---

# Examples

## Simple Example

Find a user by email.

```dart
final user = await (select(users)
      ..where((u) => u.email.equals(email)))
    .getSingle();
```

Explanation:

* If `email` is indexed, SQLite can locate the row much faster than scanning the entire table.

---

## Real-World Example

Suppose you're building an e-commerce application where users frequently view their order history.

```dart
final orders = await (select(orders)
      ..where((o) => o.customerId.equals(customerId))
      ..orderBy([
        (o) => OrderingTerm.desc(o.createdAt),
      ]))
    .get();
```

Explanation:

* A composite index on `(customerId, createdAt)` allows SQLite to efficiently filter by customer and sort by date.
* Without the index, SQLite may need to scan and sort a large number of rows.

---

# When to Use

Create indexes for columns that are:

* Frequently searched.
* Used in `WHERE` clauses.
* Used in `JOIN` conditions.
* Used in `ORDER BY`.
* Used in `GROUP BY`.
* Used to enforce uniqueness.

Common examples include:

* Email addresses
* Foreign keys
* User IDs
* Product IDs
* Order dates

---

# When NOT to Use

Avoid indexing:

* Very small tables.
* Columns that are rarely queried.
* Columns with very few distinct values (for example, a boolean flag).
* Every column in a table.

Too many indexes increase:

* Database size.
* Insert time.
* Update time.
* Delete time.

Indexes should be added intentionally, not automatically.

---

# Best Practices

* Index columns used frequently in queries.
* Index foreign keys when joining tables.
* Use composite indexes for common multi-column queries.
* Remove unused indexes.
* Measure performance before adding indexes everywhere.
* Balance read performance against write performance.

---

# Common Mistakes

## Indexing Every Column

Wrong:

```dart
// Every column has its own index.
```

Explanation:

* Extra indexes consume storage and slow write operations.

Correct:

* Index only the columns that benefit common queries.

---

## Forgetting to Index Foreign Keys

Wrong:

```dart
final orders = await (select(orders)
      ..where((o) => o.customerId.equals(id)))
    .get();
```

Explanation:

* Without an index, SQLite may scan the entire table.

Correct:

```dart
@override
List<Index> get indexes => [
  Index('customer_index', [customerId]),
];
```

Explanation:

* Indexing the foreign key improves lookup performance.

---

## Expecting Indexes to Speed Up Every Query

Wrong assumption:

> "Adding an index always makes queries faster."

Explanation:

* SQLite decides whether using an index is beneficial.
* Some queries are faster with a full table scan.

Correct understanding:

* Indexes optimize specific query patterns, not every query.

---

# Related APIs

* `Index`
* `WHERE`
* `ORDER BY`
* `GROUP BY`
* Foreign Keys
* Query Optimization

---

# Summary

Indexes improve query performance by allowing SQLite to locate rows efficiently without scanning entire tables. They're most useful for columns that are frequently filtered, joined, or sorted. While indexes can significantly speed up reads, they also increase storage usage and add overhead to write operations, so they should be used strategically where they provide the greatest benefit.
