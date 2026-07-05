# Distinct

`DISTINCT` removes duplicate rows from a query result.

---

# What is it?

Sometimes a query returns duplicate values.

The `DISTINCT` keyword tells SQLite to return only unique results.

Instead of returning every matching row, duplicate rows are filtered out before the results are returned.

---

# Why does it exist?

Imagine a products table with categories.

```text
Books
Books
Books
Electronics
Electronics
Clothing
```

If you only want the available categories, duplicates aren't useful.

Using `DISTINCT` produces:

```text
Books
Electronics
Clothing
```

This makes queries cleaner and avoids duplicate data in your application.

---

# Syntax

## Selecting Distinct Values

```dart
final categories = await (selectOnly(products)
      ..addColumns([products.category])
      ..distinct())
    .get();
```

Explanation:

* `selectOnly()` creates a query without selecting every column.
* `addColumns()` specifies the columns to retrieve.
* `distinct()` removes duplicate rows from the result.

---

## Distinct with Ordering

```dart
final categories = await (selectOnly(products)
      ..addColumns([products.category])
      ..distinct()
      ..orderBy([
        OrderingTerm.asc(products.category),
      ]))
    .get();
```

Explanation:

* Duplicate categories are removed.
* The remaining values are sorted alphabetically.

---

# Mental Model

Think of `DISTINCT` as removing repeated names from a list.

```text
Original

Apple
Apple
Orange
Banana
Orange
Apple

        │
        ▼

DISTINCT

Apple
Orange
Banana
```

Only unique values remain.

---

# Examples

## Simple Example

Retrieve all unique departments.

```dart
final departments = await (selectOnly(employees)
      ..addColumns([employees.department])
      ..distinct())
    .get();
```

Explanation:

* Each department appears only once.
* Duplicate department names are removed.

---

## Real-World Example

Load all available product brands for a filter menu.

```dart
final brands = await (selectOnly(products)
      ..addColumns([products.brand])
      ..distinct()
      ..orderBy([
        OrderingTerm.asc(products.brand),
      ]))
    .get();
```

Explanation:

* Duplicate brands are removed.
* The list is sorted for display in the UI.

---

# When to Use

Use `DISTINCT` when you need to:

* Populate dropdown menus.
* Display filter options.
* Retrieve unique values.
* Remove duplicate query results.
* Generate reports with unique entries.

---

# When NOT to Use

Avoid `DISTINCT` when:

* Every row is important.
* Duplicate rows are expected and meaningful.
* You don't actually need unique values.

Remember that removing duplicates requires additional work from SQLite.

---

# Best Practices

* Use `DISTINCT` only when duplicate removal is required.
* Combine it with `orderBy()` for predictable output.
* Select only the columns you need.
* Avoid using `DISTINCT` as a workaround for poorly designed queries.
* Consider indexes for frequently queried columns.

---

# Common Mistakes

## Selecting every column with `DISTINCT`

**Wrong**

```dart
final usersList = await (select(users)
      ..distinct())
    .get();
```

Explanation:

* Since every row usually has a unique primary key, nearly all rows remain distinct.
* This often doesn't achieve the intended result.

**Correct**

Use `selectOnly()` and retrieve only the columns whose unique values you need.

---

## Removing duplicates in Dart

**Wrong**

```dart
final categories = await select(products).get();

final uniqueCategories = categories
    .map((p) => p.category)
    .toSet();
```

Explanation:

* Every row is loaded before duplicates are removed.
* SQLite can perform this more efficiently.

**Correct**

Use `DISTINCT` in the database query.

---

## Expecting `DISTINCT` to group data

**Wrong**

Assuming `DISTINCT` performs aggregations like `GROUP BY`.

Explanation:

* `DISTINCT` only removes duplicate rows.
* It does not calculate totals, counts, or grouped summaries.

**Correct**

Use `GROUP BY` and aggregate functions when grouping data is required.

---

# Related APIs

* Select
* Select Only
* Ordering
* Filtering
* Expressions
* Custom SQL

---

# Summary

`DISTINCT` removes duplicate rows from query results, making it useful for retrieving unique values such as categories, brands, or departments. By combining it with `selectOnly()` and `orderBy()`, you can efficiently produce clean, unique datasets directly from SQLite without additional processing in Dart.
