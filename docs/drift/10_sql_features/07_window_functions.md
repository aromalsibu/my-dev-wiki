# Window Functions

**Window functions** perform calculations across a set of related rows while still returning individual rows, making them ideal for rankings, running totals, moving averages, and analytical queries.

---

# What is it?

Unlike aggregate functions such as `SUM()` or `COUNT()`, which combine multiple rows into a single result, window functions calculate values **for each row** based on a group of related rows (called a *window*).

For example, instead of calculating the total sales for all customers:

| Customer | Sales |
| -------- | ----: |
| Alice    |  2500 |
| Bob      |  1800 |
| Charlie  |  3200 |

A window function can produce:

| Customer | Sales | Rank |
| -------- | ----: | ---: |
| Charlie  |  3200 |    1 |
| Alice    |  2500 |    2 |
| Bob      |  1800 |    3 |

Each row is preserved while additional analytical information is calculated.

---

# Why does it exist?

Suppose you're building a leaderboard.

Using normal SQL, calculating each player's rank can be complicated and often requires subqueries or application-side processing.

Window functions let SQLite calculate these values directly.

They're commonly used for:

* Rankings
* Running totals
* Percentiles
* Moving averages
* Previous and next row comparisons

Instead of processing data in Dart, you can let SQLite handle these calculations efficiently.

---

# Syntax

Calculate a rank.

```dart
final results = await customSelect(
  '''
  SELECT
    name,
    score,
    RANK() OVER (
      ORDER BY score DESC
    ) AS rank
  FROM players;
  ''',
  readsFrom: {players},
).get();
```

Explanation:

* `RANK()` assigns a ranking to each row.
* `OVER` defines the window used for the calculation.
* The rows remain unchanged; only a new `rank` column is added.

---

Calculate a running total.

```dart
final results = await customSelect(
  '''
  SELECT
    id,
    amount,
    SUM(amount) OVER (
      ORDER BY id
    ) AS running_total
  FROM payments;
  ''',
  readsFrom: {payments},
).get();
```

Explanation:

* `SUM()` becomes a window function when used with `OVER`.
* Each row includes the cumulative total up to that point.

---

# Mental Model

Think of a window function as looking through a moving window over your data.

```text
Rows

10
20
30
40
50
```

SQLite examines related rows while keeping each row separate.

```text
Current Row
     │
     ▼
┌─────────────┐
│10 20 30 40 │
└─────────────┘
      │
      ▼
Calculate Value
      │
      ▼
Return Current Row + Result
```

Unlike `GROUP BY`, no rows disappear.

---

# Examples

## Simple Example

Rank products by price.

```dart
final products = await customSelect(
  '''
  SELECT
    name,
    price,
    RANK() OVER (
      ORDER BY price DESC
    ) AS price_rank
  FROM products;
  ''',
  readsFrom: {products},
).get();
```

Explanation:

* Every product receives a rank.
* The original rows remain intact.

---

## Real-World Example

Suppose you're building a sales dashboard that shows each employee's cumulative sales throughout the month.

```dart
final sales = await customSelect(
  '''
  SELECT
    employee_id,
    sale_date,
    amount,
    SUM(amount) OVER (
      PARTITION BY employee_id
      ORDER BY sale_date
    ) AS cumulative_sales
  FROM sales;
  ''',
  readsFrom: {sales},
).get();
```

Explanation:

* `PARTITION BY` creates a separate window for each employee.
* `ORDER BY` calculates the running total in chronological order.
* Each sale shows the employee's total sales up to that date.

---

# When to Use

Use window functions when you need:

* Rankings
* Running totals
* Cumulative statistics
* Moving averages
* Previous or next row values
* Analytical reports
* Leaderboards

---

# When NOT to Use

Avoid window functions when:

* A simple aggregate query is sufficient.
* You're performing basic CRUD operations.
* The query builder already expresses the query clearly.
* The calculation doesn't require row-by-row analysis.

Remember that window functions are primarily analytical tools.

---

# Best Practices

* Use meaningful aliases for calculated columns.
* Keep window definitions simple and readable.
* Prefer SQLite calculations over Dart loops for analytics.
* Combine window functions with CTEs for complex reports.
* Use the query builder for ordinary queries and reserve custom SQL for advanced analytics.

---

# Common Mistakes

## Confusing Window Functions with Aggregates

Wrong:

```sql
SELECT
SUM(amount)
FROM sales;
```

Explanation:

* This returns a single row containing the total.

Correct:

```sql
SELECT
amount,
SUM(amount) OVER (...)
FROM sales;
```

Explanation:

* Every row is returned along with its calculated value.

---

## Forgetting `ORDER BY` for Running Totals

Wrong:

```sql
SUM(amount) OVER ()
```

Explanation:

* Without an ordering, the cumulative progression is undefined.

Correct:

```sql
SUM(amount) OVER (
  ORDER BY sale_date
)
```

Explanation:

* Rows are processed in a predictable order.

---

## Performing Analytics in Dart

Wrong:

```dart
final sales = await getSales();

// Calculate rankings manually.
```

Explanation:

* SQLite is optimized for these calculations.

Correct:

```dart
await customSelect(
  '''
  SELECT
    ...,
    RANK() OVER (...)
  ''',
).get();
```

Explanation:

* Let the database perform analytical computations efficiently.

---

# Related APIs

* Custom SQL
* Common Table Expressions (CTE)
* SQL Variables
* `customSelect()`
* Aggregate Functions

---

# Summary

Window functions allow SQLite to perform powerful analytical calculations while preserving individual rows. They're ideal for rankings, running totals, cumulative statistics, and other reporting scenarios that would otherwise require complex SQL or additional Dart code. Since Drift's query builder doesn't cover all window function features, they're typically used through custom SQL for advanced reporting and analytics.
