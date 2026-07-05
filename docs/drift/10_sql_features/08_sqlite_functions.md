# SQLite Functions

**SQLite functions** are built-in functions that perform calculations, manipulate data, and transform values directly within SQL queries.

---

# What is it?

SQLite includes many built-in functions that can be used inside SQL statements.

Instead of fetching data into Dart and processing it there, you can let SQLite perform the work before returning the results.

SQLite functions are commonly used for:

* String manipulation
* Mathematical calculations
* Date and time operations
* Aggregate calculations
* Conditional expressions
* Handling `NULL` values

Using these functions keeps queries concise and often improves performance by reducing application-side processing.

---

# Why does it exist?

Suppose you want every user's name in uppercase.

Without SQLite functions:

```text
Database
    │
    ▼
Fetch Data
    │
    ▼
Convert to Uppercase in Dart
    │
    ▼
Display Result
```

With SQLite functions:

```text
Database
    │
    ▼
UPPER(name)
    │
    ▼
Return Result
```

The transformation happens inside SQLite, reducing the work your application needs to do.

---

# Syntax

Convert text to uppercase.

```dart
final result = await customSelect(
  '''
  SELECT
    UPPER(name) AS name
  FROM users;
  ''',
  readsFrom: {users},
).get();
```

Explanation:

* `UPPER()` converts text to uppercase.
* The transformed value is returned in the query result.

---

Calculate an average.

```dart
final result = await customSelect(
  '''
  SELECT
    AVG(price) AS average_price
  FROM products;
  ''',
  readsFrom: {products},
).get();
```

Explanation:

* `AVG()` calculates the average value of a column.
* SQLite performs the calculation before returning the result.

---

Get the current timestamp.

```dart
final result = await customSelect(
  '''
  SELECT CURRENT_TIMESTAMP;
  ''',
).get();
```

Explanation:

* `CURRENT_TIMESTAMP` returns the current date and time according to SQLite.

---

# Mental Model

Think of SQLite functions as built-in tools.

```text
Query
   │
   ▼
SQLite Function
   │
   ▼
Processed Value
   │
   ▼
Application
```

Instead of processing data after fetching it, SQLite processes it during query execution.

---

# Examples

## Simple Example

Capitalize usernames.

```dart
final users = await customSelect(
  '''
  SELECT
    UPPER(username) AS username
  FROM users;
  ''',
  readsFrom: {users},
).get();
```

Explanation:

* Every username is converted to uppercase before being returned.

---

## Real-World Example

Suppose you're building an analytics dashboard showing monthly revenue.

```dart
final revenue = await customSelect(
  '''
  SELECT
    strftime('%Y-%m', created_at) AS month,
    SUM(total) AS revenue
  FROM orders
  GROUP BY month
  ORDER BY month;
  ''',
  readsFrom: {orders},
).get();
```

Explanation:

* `strftime()` extracts the year and month from each order date.
* `SUM()` calculates the revenue for each month.
* SQLite performs all grouping and calculations before returning the results.

---

# Common SQLite Functions

## String Functions

| Function   | Description                         |
| ---------- | ----------------------------------- |
| `UPPER()`  | Converts text to uppercase          |
| `LOWER()`  | Converts text to lowercase          |
| `LENGTH()` | Returns the length of a string      |
| `TRIM()`   | Removes leading and trailing spaces |
| `SUBSTR()` | Extracts part of a string           |

---

## Numeric Functions

| Function  | Description     |
| --------- | --------------- |
| `ABS()`   | Absolute value  |
| `ROUND()` | Rounds a number |
| `MIN()`   | Smallest value  |
| `MAX()`   | Largest value   |
| `AVG()`   | Average value   |

---

## Aggregate Functions

| Function  | Description        |
| --------- | ------------------ |
| `COUNT()` | Counts rows        |
| `SUM()`   | Calculates total   |
| `AVG()`   | Calculates average |
| `MIN()`   | Finds minimum      |
| `MAX()`   | Finds maximum      |

---

## Date & Time Functions

| Function            | Description           |
| ------------------- | --------------------- |
| `DATE()`            | Returns a date        |
| `TIME()`            | Returns a time        |
| `DATETIME()`        | Returns date and time |
| `CURRENT_DATE`      | Today's date          |
| `CURRENT_TIMESTAMP` | Current date and time |
| `strftime()`        | Formats date and time |

---

## NULL Functions

| Function     | Description                            |
| ------------ | -------------------------------------- |
| `COALESCE()` | Returns the first non-NULL value       |
| `IFNULL()`   | Replaces NULL with another value       |
| `NULLIF()`   | Returns NULL when two values are equal |

---

# When to Use

Use SQLite functions when:

* Formatting text.
* Calculating totals or averages.
* Working with dates.
* Replacing `NULL` values.
* Generating reports.
* Performing calculations before returning data.
* Reducing Dart-side processing.

---

# When NOT to Use

Avoid SQLite functions when:

* Simple Dart code is more readable.
* The operation depends on application-specific logic.
* You're performing work unrelated to the query.
* The query builder already provides the required functionality.

Don't move every computation into SQL—use SQLite functions where they simplify the query or improve efficiency.

---

# Best Practices

* Prefer SQLite functions for database-related calculations.
* Use aliases for calculated columns.
* Keep SQL readable.
* Let SQLite handle filtering, grouping, and aggregation.
* Use custom SQL only when the query builder isn't sufficient.

---

# Common Mistakes

## Processing Database Data in Dart

Wrong:

```dart
final users = await select(users).get();

for (final user in users) {
  print(user.name.toUpperCase());
}
```

Explanation:

* Every row is processed after fetching.

Correct:

```dart
await customSelect(
  '''
  SELECT
    UPPER(name)
  FROM users;
  ''',
).get();
```

Explanation:

* SQLite performs the transformation before returning the data.

---

## Ignoring NULL Values

Wrong:

```sql
UPPER(nickname)
```

Explanation:

* If `nickname` is `NULL`, the result is also `NULL`.

Correct:

```sql
UPPER(COALESCE(nickname, 'Guest'))
```

Explanation:

* `COALESCE()` provides a default value before applying `UPPER()`.

---

## Reimplementing Built-in Functions

Wrong:

Fetching every row and manually calculating totals in Dart.

Explanation:

* SQLite already provides optimized aggregate functions.

Correct:

```sql
SELECT SUM(total)
FROM orders;
```

Explanation:

* Let SQLite perform aggregate calculations efficiently.

---

# Related APIs

* Custom SQL
* SQL Variables
* Window Functions
* Common Table Expressions (CTE)
* Aggregate Functions

---

# Summary

SQLite functions provide powerful built-in capabilities for transforming, calculating, and formatting data directly within SQL queries. They allow SQLite to handle tasks such as string manipulation, aggregation, date formatting, and `NULL` handling before data reaches your application. Using these functions appropriately results in cleaner queries, less Dart code, and more efficient data processing.
