# Limiting

Limiting restricts the number of rows returned by a query.

---

# What is it?

A limit tells SQLite to return only a specific number of rows, even if the query matches many more.

In Drift, this is done using the `limit()` method.

Limiting is commonly used for:

* Pagination
* Recent activity
* Search results
* Preview lists
* Performance optimization

---

# Why does it exist?

Imagine a table containing 100,000 users.

If your application only needs the first 20 users for a page, retrieving all 100,000 rows would waste:

* Memory
* CPU time
* Database resources

Instead, SQLite can stop after returning the required number of rows.

---

# Syntax

## Limiting the Number of Rows

```dart
final usersList = await (select(users)
      ..limit(10))
    .get();
```

Explanation:

* `limit(10)` returns at most 10 rows.
* If fewer than 10 rows exist, all available rows are returned.

---

## Using an Offset

```dart
final nextPage = await (select(users)
      ..limit(20, offset: 20))
    .get();
```

Explanation:

* Returns up to 20 rows.
* Skips the first 20 matching rows.
* Useful for pagination.

---

## Combining Ordering and Limiting

```dart
final latestUsers = await (select(users)
      ..orderBy([
        (u) => OrderingTerm.desc(u.createdAt),
      ])
      ..limit(5))
    .get();
```

Explanation:

* Rows are first ordered by creation date.
* Only the five newest users are returned.

---

# Mental Model

Think of limiting as taking only the first few books from a shelf.

```text
Shelf

Book 1
Book 2
Book 3
Book 4
Book 5
Book 6
Book 7

Limit 3

↓

Book 1
Book 2
Book 3
```

The remaining books stay on the shelf but aren't returned.

---

# Examples

## Simple Example

Retrieve the five newest products.

```dart
final productsList = await (select(products)
      ..orderBy([
        (p) => OrderingTerm.desc(p.id),
      ])
      ..limit(5))
    .get();
```

Explanation:

* Products are sorted by ID.
* Only the latest five products are returned.

---

## Real-World Example

Implement pagination.

```dart
const pageSize = 20;
const page = 2;

final usersPage = await (select(users)
      ..orderBy([
        (u) => OrderingTerm.asc(u.name),
      ])
      ..limit(
        pageSize,
        offset: page * pageSize,
      ))
    .get();
```

Explanation:

* Retrieves one page of users.
* `offset` skips rows from previous pages.
* `limit` controls the page size.

---

# When to Use

Use limiting when you need to:

* Build paginated lists.
* Show recent items.
* Display previews.
* Reduce memory usage.
* Improve query performance.

---

# When NOT to Use

Avoid limiting when:

* You genuinely need every matching row.
* Performing calculations that require the full dataset.
* Exporting complete data.

---

# Best Practices

* Combine `limit()` with `orderBy()` for predictable results.
* Use `offset` for pagination.
* Keep page sizes reasonable.
* Let SQLite perform the limiting instead of truncating lists in Dart.
* Use indexes when limiting ordered queries on large tables.

---

# Common Mistakes

## Limiting without ordering

**Wrong**

```dart
final usersList = await (select(users)
      ..limit(10))
    .get();
```

Explanation:

* SQLite doesn't guarantee which 10 rows will be returned.
* The results may change over time.

**Correct**

```dart
final usersList = await (select(users)
      ..orderBy([
        (u) => OrderingTerm.asc(u.name),
      ])
      ..limit(10))
    .get();
```

Explanation:

* Rows are sorted before limiting, producing predictable results.

---

## Loading everything and trimming in Dart

**Wrong**

```dart
final usersList = await select(users).get();

final firstTen = usersList.take(10);
```

Explanation:

* Every row is loaded into memory.
* SQLite could have returned only the required rows.

**Correct**

Use `limit(10)` in the query.

---

## Using a large offset for deep pagination

**Wrong**

Using an extremely large offset, such as tens of thousands of rows.

Explanation:

* SQLite must still skip all preceding rows.
* Performance can degrade for deep pagination.

**Correct**

For very large datasets, consider keyset (cursor-based) pagination instead of relying on large offsets.

---

# Related APIs

* Select
* Filtering
* Ordering
* Distinct
* Dynamic Queries

---

# Summary

`limit()` restricts the number of rows returned by a query, making it an essential tool for pagination, previews, and performance optimization. It is most effective when combined with `orderBy()` to ensure the returned rows are consistent and predictable.
