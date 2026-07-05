# Common Table Expressions (CTE)

**A Common Table Expression (CTE)** is a temporary named result set that exists only for the duration of a single SQL statement, making complex queries easier to read and maintain.

---

# What is it?

A Common Table Expression, or **CTE**, lets you break a complex SQL query into smaller, reusable parts.

Instead of writing one large query with deeply nested subqueries, you can define intermediate results using the `WITH` clause and reference them later in the same query.

Think of a CTE as a temporary table that exists only while the query is executing.

---

# Why does it exist?

Consider a report that needs to:

1. Calculate total sales for each customer.
2. Find customers whose sales exceed $10,000.
3. Sort them by total sales.

Without a CTE, this often requires nested subqueries that are difficult to read.

```text
SELECT ...
FROM (
    SELECT ...
    FROM (
        ...
    )
)
```

With a CTE, each step has a clear name.

```text
WITH customer_sales AS (...)
SELECT ...
FROM customer_sales
```

This makes complex SQL much easier to understand and maintain.

---

# Syntax

A simple CTE.

```dart
final results = await customSelect(
  '''
  WITH adult_users AS (
    SELECT *
    FROM users
    WHERE age >= 18
  )
  SELECT *
  FROM adult_users;
  ''',
  readsFrom: {users},
).get();
```

Explanation:

* `WITH` creates a temporary named result set.
* `adult_users` behaves like a temporary table.
* The CTE exists only while this query runs.

---

Using multiple CTEs.

```dart
final results = await customSelect(
  '''
  WITH
    active_users AS (
      SELECT *
      FROM users
      WHERE active = 1
    ),
    premium_users AS (
      SELECT *
      FROM active_users
      WHERE premium = 1
    )

  SELECT *
  FROM premium_users;
  ''',
  readsFrom: {users},
).get();
```

Explanation:

* One CTE can reference another.
* This allows complex queries to be built in small, readable steps.

---

# Mental Model

Think of a CTE as a temporary worksheet.

```text
Users Table
      │
      ▼
Create Temporary Result
      │
      ▼
Use Temporary Result
      │
      ▼
Return Final Result
```

Unlike a view:

* A CTE exists only during one query.
* It isn't stored in the database.
* It can't be queried later.

---

# Examples

## Simple Example

Find all expensive products.

```dart
final products = await customSelect(
  '''
  WITH expensive_products AS (
    SELECT *
    FROM products
    WHERE price > 1000
  )

  SELECT *
  FROM expensive_products;
  ''',
  readsFrom: {products},
).get();
```

Explanation:

* The CTE isolates the filtering logic.
* The final query becomes easier to read.

---

## Real-World Example

Suppose you're building a sales dashboard.

First, calculate each customer's total spending.

Then display only the top customers.

```dart
final customers = await customSelect(
  '''
  WITH customer_totals AS (
    SELECT
      customer_id,
      SUM(total) AS total_spent
    FROM orders
    GROUP BY customer_id
  )

  SELECT *
  FROM customer_totals
  WHERE total_spent >= 10000
  ORDER BY total_spent DESC;
  ''',
  readsFrom: {orders},
).get();
```

Explanation:

* The CTE computes customer totals once.
* The final query filters and sorts those results.
* This approach is much clearer than nesting multiple subqueries.

---

# When to Use

Use CTEs when:

* Writing complex SQL.
* Replacing nested subqueries.
* Breaking large queries into logical steps.
* Creating intermediate calculations.
* Writing reporting or analytics queries.
* Improving query readability.

---

# When NOT to Use

Avoid CTEs when:

* A simple query is sufficient.
* Drift's query builder expresses the query clearly.
* The query consists of only one or two straightforward operations.

Using a CTE for very simple queries can make them unnecessarily verbose.

---

# Best Practices

* Give CTEs meaningful names.
* Keep each CTE focused on one task.
* Use CTEs to improve readability, not just shorten code.
* Prefer multiple small CTEs over one deeply nested query.
* Use custom SQL only when the query builder isn't sufficient.

---

# Common Mistakes

## Using Deeply Nested Subqueries

Wrong:

```sql
SELECT *
FROM (
    SELECT *
    FROM (
        SELECT ...
    )
);
```

Explanation:

* Deep nesting is difficult to read and maintain.

Correct:

```sql
WITH filtered_data AS (...)
SELECT *
FROM filtered_data;
```

Explanation:

* Named steps are easier to understand.

---

## Expecting a CTE to Persist

Wrong assumption:

> "I created a CTE, so I can query it later."

Explanation:

* A CTE exists only for the current SQL statement.

Correct understanding:

* If you need reusable query logic across multiple queries, consider using a **View** instead.

---

## Using a CTE for Every Query

Wrong:

```sql
WITH users_cte AS (
    SELECT * FROM users
)
SELECT *
FROM users_cte;
```

Explanation:

* The CTE doesn't simplify the query.

Correct:

```sql
SELECT *
FROM users;
```

Explanation:

* Use CTEs only when they improve clarity or solve a more complex problem.

---

# Related APIs

* Custom SQL
* SQL Variables
* Views
* Window Functions
* `customSelect()`

---

# Summary

Common Table Expressions (CTEs) let you create temporary named result sets within a single SQL statement. They make complex queries easier to read, organize, and maintain by breaking them into logical steps. While they don't replace tables or views, they're an excellent tool for reporting, analytics, and other advanced SQL queries where readability is important.
