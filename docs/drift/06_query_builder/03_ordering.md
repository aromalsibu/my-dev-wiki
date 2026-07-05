# Ordering

Ordering controls the sequence in which query results are returned.

---

# What is it?

By default, a database does **not** guarantee the order of returned rows unless you explicitly request one.

The `orderBy()` method lets you sort query results based on one or more columns.

For example, you can sort:

* Alphabetically
* By date
* By price
* By ID
* In ascending or descending order

---

# Why does it exist?

Imagine a list of tasks.

Without ordering:

```text
Task C
Task A
Task B
```

The order depends on how SQLite stores the data and may change over time.

With ordering:

```text
Task A
Task B
Task C
```

Ordering ensures your application displays data in a predictable way.

---

# Syntax

## Ordering in Ascending Order

```dart
final usersList = await (select(users)
      ..orderBy([
        (u) => OrderingTerm.asc(u.name),
      ]))
    .get();
```

Explanation:

* `orderBy()` defines how the results should be sorted.
* `OrderingTerm.asc()` sorts the `name` column in ascending order (A → Z).

---

## Ordering in Descending Order

```dart
final latestUsers = await (select(users)
      ..orderBy([
        (u) => OrderingTerm.desc(u.id),
      ]))
    .get();
```

Explanation:

* `OrderingTerm.desc()` sorts the rows in descending order.
* Larger IDs appear first.

---

## Ordering by Multiple Columns

```dart
final employees = await (select(users)
      ..orderBy([
        (u) => OrderingTerm.asc(u.department),
        (u) => OrderingTerm.asc(u.name),
      ]))
    .get();
```

Explanation:

* Rows are first sorted by department.
* Users within the same department are then sorted by name.

---

# Mental Model

Think of ordering as arranging books on a shelf.

```text
Before

Charlie
Alice
Bob

        │
        ▼
Sort by Name

Alice
Bob
Charlie
```

Ordering changes the sequence of rows, not the data itself.

---

# Examples

## Simple Example

Sort products by price.

```dart
final productsList = await (select(products)
      ..orderBy([
        (p) => OrderingTerm.asc(p.price),
      ]))
    .get();
```

Explanation:

* Products are returned from the lowest price to the highest.

---

## Real-World Example

Show the newest notifications first.

```dart
final notificationsList = await (select(notifications)
      ..orderBy([
        (n) => OrderingTerm.desc(n.createdAt),
      ]))
    .get();
```

Explanation:

* The most recently created notifications appear first.
* Older notifications appear later in the list.

---

# When to Use

Use ordering when you need to:

* Display alphabetical lists.
* Show the newest or oldest records.
* Rank results by score or price.
* Present data in a consistent order.
* Prepare data for pagination.

---

# When NOT to Use

Avoid ordering when:

* The order doesn't matter.
* You're working with very small datasets where ordering provides no benefit.
* The query is performance-critical and sorting is unnecessary.

Remember that sorting has a cost, especially for large datasets without indexes.

---

# Best Practices

* Always order lists shown to users.
* Order by indexed columns when possible.
* Use descending order for recent activity.
* Use multiple ordering terms for predictable results.
* Combine ordering with `limit()` for pagination.

---

# Common Mistakes

## Assuming rows are automatically ordered

**Wrong**

```dart
final usersList = await select(users).get();
```

Explanation:

* SQLite does not guarantee the order of returned rows.
* The order may change as data is inserted or deleted.

**Correct**

```dart
final usersList = await (select(users)
      ..orderBy([
        (u) => OrderingTerm.asc(u.name),
      ]))
    .get();
```

Explanation:

* The results are consistently sorted by name.

---

## Ordering in Dart instead of SQL

**Wrong**

```dart
final usersList = await select(users).get();

usersList.sort(
  (a, b) => a.name.compareTo(b.name),
);
```

Explanation:

* Every row is loaded before sorting.
* SQLite can perform the sort more efficiently.

**Correct**

Use `orderBy()` so the database returns already-sorted results.

---

## Using only one ordering criterion when multiple are needed

**Wrong**

Sorting employees only by department.

Explanation:

* Employees within the same department may appear in an unpredictable order.

**Correct**

Sort by department first, then by name.

---

# Related APIs

* Select
* Filtering
* Limiting
* Distinct
* Expressions

---

# Summary

`orderBy()` controls the order in which query results are returned. By using ascending, descending, or multiple ordering terms, you can produce predictable and user-friendly results while allowing SQLite to perform the sorting efficiently before the data reaches your application.
