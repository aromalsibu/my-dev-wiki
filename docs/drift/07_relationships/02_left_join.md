# Left Join

A left join returns all rows from the left table and the matching rows from the right table.

---

# What is it?

A left join includes **every row from the primary (left) table**, even when there is no matching row in the joined table.

If no matching record exists, the columns from the right table are returned as `NULL`.

This makes left joins useful for optional relationships.

---

# Why does it exist?

Consider a blogging application.

Some posts have authors, while others were imported and don't yet have an assigned author.

```text
Posts
+----+-----------+----------+
| ID | Title     | AuthorId |
+----+-----------+----------+
| 1  | Drift     |    10    |
| 2  | SQLite    |   NULL   |
| 3  | Flutter   |    11    |
+----+-----------+----------+

Users
+----+--------+
| ID | Name   |
+----+--------+
|10  | Alice  |
|11  | Bob    |
+----+--------+
```

With an inner join, the second post would be excluded.

A left join keeps every post and includes author information only when available.

---

# Syntax

## Performing a Left Join

```dart
final query = select(posts).join([
  leftOuterJoin(
    users,
    users.id.equalsExp(posts.authorId),
  ),
]);

final results = await query.get();
```

Explanation:

* `leftOuterJoin()` keeps every row from `posts`.
* Matching users are included when available.
* Missing matches produce `NULL` values for the joined table.

---

## Reading Optional Rows

```dart
for (final row in results) {
  final post = row.readTable(posts);
  final author = row.readTableOrNull(users);

  print(author?.name ?? 'Unknown Author');
}
```

Explanation:

* `readTable()` reads the left table because it always exists.
* `readTableOrNull()` safely reads the joined table, returning `null` when no matching row exists.

---

# Mental Model

Think of a left join as inviting everyone from one group, whether or not they have a partner.

```text
Posts                Users

Post A  ─────────► Alice
Post B  ─────────► (No Match)
Post C  ─────────► Bob

↓

Result

Post A + Alice
Post B + NULL
Post C + Bob
```

Every post appears in the final result.

---

# Examples

## Simple Example

Retrieve every product with its category.

```dart
final query = select(products).join([
  leftOuterJoin(
    categories,
    categories.id.equalsExp(products.categoryId),
  ),
]);

final rows = await query.get();
```

Explanation:

* Products without a category are still returned.
* Their category data is `null`.

---

## Real-World Example

Load all orders, even those without an assigned delivery driver.

```dart
final query = select(orders).join([
  leftOuterJoin(
    drivers,
    drivers.id.equalsExp(orders.driverId),
  ),
]);

final rows = await query.get();
```

Explanation:

* Every order is returned.
* Orders waiting for assignment have a `null` driver.

---

# When to Use

Use a left join when you need to:

* Include every row from one table.
* Handle optional relationships.
* Display missing related data.
* Build reports with incomplete relationships.
* Preserve the primary dataset.

---

# When NOT to Use

Avoid a left join when:

* Every row must have a matching related record.
* Unmatched rows should be excluded.

In these cases, an inner join is usually the better choice.

---

# Best Practices

* Use left joins for optional foreign keys.
* Access joined tables with `readTableOrNull()`.
* Check for `null` before using joined data.
* Join using indexed key columns.
* Keep join conditions simple and readable.

---

# Common Mistakes

## Using `readTable()` on an optional table

**Wrong**

```dart
final author = row.readTable(users);
```

Explanation:

* If no matching user exists, this will fail because the joined table contains `NULL` values.

**Correct**

```dart
final author = row.readTableOrNull(users);
```

Explanation:

* Safely returns `null` when the joined row doesn't exist.

---

## Using an inner join for optional relationships

**Wrong**

```dart
innerJoin(
  users,
  users.id.equalsExp(posts.authorId),
)
```

Explanation:

* Posts without authors are excluded entirely.

**Correct**

```dart
leftOuterJoin(
  users,
  users.id.equalsExp(posts.authorId),
)
```

Explanation:

* Every post is included, regardless of whether an author exists.

---

## Forgetting to handle `null`

**Wrong**

```dart
print(author.name);
```

Explanation:

* `author` may be `null` after a left join.

**Correct**

```dart
print(author?.name ?? 'Unknown');
```

Explanation:

* Handles missing related records safely.

---

# Related APIs

* Inner Join
* Cross Join
* Self Join
* Mapping Joined Results
* Foreign Keys

---

# Summary

A left join returns every row from the left table while including matching rows from the right table when available. If no related record exists, the joined columns are `NULL`, making left joins ideal for optional relationships where the primary data should always be preserved.
