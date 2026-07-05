# SQL Variables

**SQL variables** are placeholders used in custom SQL statements to safely pass values to a query without embedding them directly into the SQL string.

---

# What is it?

When writing custom SQL, you'll often need to insert dynamic values such as user IDs, names, dates, or search terms.

Instead of building SQL strings manually, Drift lets you use **variables**. A variable is represented by a `?` placeholder in the SQL statement, with the actual value supplied separately.

This approach:

* Prevents SQL injection.
* Keeps SQL readable.
* Allows SQLite to reuse prepared statements for better performance.

---

# Why does it exist?

Suppose you want to find a user by ID.

A common mistake is to insert the value directly into the SQL string.

```sql
SELECT * FROM users WHERE id = 5
```

This becomes dangerous when the value comes from user input.

Instead, use a placeholder:

```sql
SELECT * FROM users WHERE id = ?
```

The value is then bound separately by Drift, ensuring it is treated as data rather than executable SQL.

---

# Syntax

Use a variable in a SELECT query.

```dart
final result = await customSelect(
  'SELECT * FROM users WHERE id = ?',
  variables: [
    Variable.withInt(userId),
  ],
  readsFrom: {users},
).get();
```

Explanation:

* `?` is a placeholder for a value.
* `Variable.withInt()` binds an integer to the placeholder.
* Drift safely passes the value to SQLite.

---

Bind multiple variables.

```dart
final result = await customSelect(
  '''
  SELECT *
  FROM users
  WHERE age >= ?
    AND city = ?
  ''',
  variables: [
    Variable.withInt(18),
    Variable.withString('London'),
  ],
  readsFrom: {users},
).get();
```

Explanation:

* Variables are matched to placeholders in order.
* Each `?` must have a corresponding variable.

---

Use variables with a custom update.

```dart
await customStatement(
  'UPDATE users SET age = ? WHERE id = ?',
  [
    30,
    userId,
  ],
);
```

Explanation:

* The first value replaces the first `?`.
* The second value replaces the second `?`.

---

# Mental Model

Think of a SQL statement as a template.

```text
SELECT *
FROM users
WHERE id = ?
```

The variable fills the placeholder.

```text
Template
     │
     ▼
Placeholder (?)
     │
     ▼
Bound Value
     │
     ▼
SQLite executes the query
```

The SQL structure never changes—only the values do.

---

# Examples

## Simple Example

Find a user by email.

```dart
final result = await customSelect(
  '''
  SELECT *
  FROM users
  WHERE email = ?
  ''',
  variables: [
    Variable.withString(email),
  ],
  readsFrom: {users},
).get();
```

Explanation:

* The email is safely bound to the query.
* No string interpolation is required.

---

## Real-World Example

Search for products within a price range.

```dart
final result = await customSelect(
  '''
  SELECT *
  FROM products
  WHERE price BETWEEN ? AND ?
  ORDER BY price
  ''',
  variables: [
    Variable.withReal(minPrice),
    Variable.withReal(maxPrice),
  ],
  readsFrom: {products},
).get();
```

Explanation:

* Both price values are passed separately.
* SQLite executes the query using the supplied variables.
* The SQL remains readable and secure.

---

# When to Use

Use SQL variables whenever your custom SQL contains dynamic values, such as:

* IDs
* Names
* Dates
* Prices
* Search terms
* Filter values
* User input

As a general rule, every dynamic value should be passed as a variable.

---

# When NOT to Use

Variables aren't necessary for:

* Static SQL statements.
* Hardcoded values that never change.
* Queries written using Drift's query builder (it handles parameter binding automatically).

Avoid manually concatenating values into SQL strings.

---

# Best Practices

* Always use variables instead of string interpolation.
* Match the variable type to the database column type.
* Keep SQL and variables separate.
* Use multiline strings for longer SQL statements.
* Let Drift handle escaping and parameter binding.

---

# Common Mistakes

## Using String Interpolation

Wrong:

```dart
await customSelect(
  'SELECT * FROM users WHERE id = $userId',
).get();
```

Explanation:

* Direct interpolation can introduce SQL injection vulnerabilities.

Correct:

```dart
await customSelect(
  'SELECT * FROM users WHERE id = ?',
  variables: [
    Variable.withInt(userId),
  ],
  readsFrom: {users},
).get();
```

Explanation:

* The value is safely bound to the query.

---

## Mismatched Placeholders and Variables

Wrong:

```dart
await customSelect(
  '''
  SELECT *
  FROM users
  WHERE age > ?
    AND city = ?
  ''',
  variables: [
    Variable.withInt(18),
  ],
).get();
```

Explanation:

* There are two placeholders but only one variable.

Correct:

```dart
variables: [
  Variable.withInt(18),
  Variable.withString('London'),
]
```

Explanation:

* Every placeholder must have a matching variable.

---

## Using the Wrong Variable Type

Wrong:

```dart
Variable.withString('25')
```

Explanation:

* The value is stored as a string instead of an integer.

Correct:

```dart
Variable.withInt(25)
```

Explanation:

* Use the variable type that matches the column type for better correctness and performance.

---

# Related APIs

* `Variable`
* `customSelect()`
* `customStatement()`
* `customInsert()`
* `customUpdate()`
* Custom SQL

---

# Summary

SQL variables provide a safe and efficient way to pass dynamic values to custom SQL statements. By using `?` placeholders and binding values separately, you protect your application from SQL injection, improve query readability, and allow SQLite to optimize query execution. Whenever you're writing custom SQL with dynamic data, prefer SQL variables over string interpolation.
