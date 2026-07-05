# Custom SQL Expressions

Custom SQL expressions allow you to use raw SQL snippets inside Drift's type-safe query builder.

---

# What is it?

Drift provides a rich set of expression APIs, but occasionally you may need SQL functionality that isn't directly exposed.

Custom SQL expressions let you embed raw SQL expressions while still using the rest of Drift's query builder.

This gives you the flexibility of SQL without abandoning Drift's typed APIs.

---

# Why does it exist?

Most queries can be written using Drift's built-in expression methods.

However, SQLite supports many advanced features that may not have dedicated Dart APIs.

For example, you might want to:

* Call a SQLite function.
* Use a complex SQL expression.
* Perform custom calculations.
* Access SQLite-specific features.

Custom SQL expressions bridge this gap.

---

# Syntax

## Using a Custom Expression

```dart
final discountedPrice = CustomExpression<double>(
  'price * 0.9',
);

final query = selectOnly(products)
  ..addColumns([discountedPrice]);
```

Explanation:

* `CustomExpression` represents a raw SQL expression.
* The generic type (`double`) tells Drift the expected result type.
* Drift includes the expression in the generated SQL.

---

## Using a SQLite Function

```dart
final currentTime = const CustomExpression<DateTime>(
  'CURRENT_TIMESTAMP',
);

final query = selectOnly(users)
  ..addColumns([currentTime]);
```

Explanation:

* The SQL expression is inserted directly into the query.
* SQLite evaluates the function when the query runs.

---

# Mental Model

Think of custom expressions as an escape hatch.

```text
Drift Query Builder
        │
        ▼
Built-in Expressions
        │
        ▼
Need something unavailable?
        │
        ▼
Custom SQL Expression
```

Most of the query remains type-safe, while a small part is written as raw SQL.

---

# Examples

## Simple Example

Convert a name to uppercase.

```dart
final upperName = CustomExpression<String>(
  'UPPER(name)',
);

final query = selectOnly(users)
  ..addColumns([upperName]);
```

Explanation:

* SQLite's `UPPER()` function converts text to uppercase.
* The result is returned as a `String`.

---

## Real-World Example

Calculate a discounted product price.

```dart
final discountedPrice = CustomExpression<double>(
  'price * 0.85',
);

final query = selectOnly(products)
  ..addColumns([
    products.name,
    discountedPrice,
  ]);
```

Explanation:

* SQLite calculates the discounted price.
* The computed value is returned alongside the product name.

---

# When to Use

Use custom SQL expressions when:

* Drift doesn't expose the SQL feature you need.
* Calling SQLite functions.
* Performing advanced calculations.
* Using SQLite-specific capabilities.
* Gradually migrating existing SQL to Drift.

---

# When NOT to Use

Avoid custom SQL expressions when:

* A built-in Drift expression already exists.
* The query can be expressed using Drift's type-safe APIs.
* The SQL fragment is difficult to maintain.

Prefer Drift's native APIs whenever possible, as they provide compile-time validation and better readability.

---

# Best Practices

* Use built-in Drift expressions first.
* Keep custom SQL snippets small and focused.
* Specify the correct generic type.
* Avoid duplicating complex SQL throughout the codebase.
* Document non-obvious SQL expressions.

---

# Common Mistakes

## Using raw SQL unnecessarily

**Wrong**

```dart
CustomExpression<bool>(
  'age > 18',
)
```

Explanation:

* Drift already provides type-safe comparison methods.

**Correct**

```dart
users.age.isBiggerThanValue(18)
```

Explanation:

* Built-in expressions are safer and easier to refactor.

---

## Providing the wrong result type

**Wrong**

```dart
CustomExpression<String>(
  'COUNT(*)',
)
```

Explanation:

* `COUNT(*)` returns an integer, not a string.

**Correct**

```dart
CustomExpression<int>(
  'COUNT(*)',
)
```

Explanation:

* The generic type should match the SQL expression's result.

---

## Writing large SQL fragments

**Wrong**

Embedding an entire complex query inside a `CustomExpression`.

Explanation:

* This reduces readability and bypasses many benefits of Drift's query builder.

**Correct**

Use custom expressions only for the parts that require raw SQL, while letting Drift build the rest of the query.

---

# Related APIs

* Expressions
* Custom SQL
* SQLite Functions
* Select
* Dynamic Queries

---

# Summary

Custom SQL expressions provide a way to embed raw SQL fragments inside Drift's query builder when built-in APIs aren't sufficient. They offer flexibility for advanced SQLite features while allowing the rest of the query to remain type-safe and maintainable. Use them sparingly, and prefer Drift's native expression APIs whenever possible.
