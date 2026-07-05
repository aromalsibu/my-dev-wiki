# Expressions

Expressions represent SQL operations such as comparisons, arithmetic, logical conditions, and function calls.

---

# What is it?

In Drift, an expression is a piece of SQL that evaluates to a value.

Expressions are used throughout the query builder to create conditions and compute values.

For example, these are all expressions:

* `users.id.equals(1)`
* `products.price + const Constant(10)`
* `tasks.completed.equals(true)`
* `users.age.isBiggerThanValue(18)`

Instead of writing SQL as strings, Drift lets you build these expressions using Dart's type-safe APIs.

---

# Why does it exist?

SQL queries are built from expressions.

Consider this SQL:

```sql
WHERE age > 18 AND active = TRUE
```

Instead of writing raw SQL, Drift provides Dart methods that generate the same SQL while checking types at compile time.

This reduces syntax errors and makes queries easier to refactor.

---

# Syntax

## Comparison Expression

```dart
final adults = await (select(users)
      ..where(
        (u) => u.age.isBiggerThanValue(18),
      ))
    .get();
```

Explanation:

* `isBiggerThanValue()` creates a SQL comparison.
* The expression is translated into `age > 18`.

---

## Combining Expressions

```dart
final activeAdults = await (select(users)
      ..where(
        (u) =>
            u.age.isBiggerThanValue(18) &
            u.isActive.equals(true),
      ))
    .get();
```

Explanation:

* `&` represents SQL `AND`.
* Both expressions must evaluate to true.

---

## Arithmetic Expression

```dart
final expensiveProducts = await (select(products)
      ..where(
        (p) => (p.price + const Constant(50))
            .isBiggerThanValue(500),
      ))
    .get();
```

Explanation:

* `Constant(50)` represents a literal SQL value.
* Arithmetic expressions are evaluated by SQLite.

---

# Mental Model

Think of expressions as building blocks.

```text
Expression

Age > 18
     │
     ▼

Name = Alice
     │
     ▼

Active = true
     │
     ▼

Combine

↓

WHERE age > 18
AND name = 'Alice'
AND active = TRUE
```

Small expressions can be combined to build complex queries.

---

# Examples

## Simple Example

Find users named Alice.

```dart
final usersList = await (select(users)
      ..where(
        (u) => u.name.equals('Alice'),
      ))
    .get();
```

Explanation:

* `equals()` creates a string comparison expression.

---

## Real-World Example

Retrieve available products costing less than $500.

```dart
final availableProducts = await (select(products)
      ..where(
        (p) =>
            p.inStock.equals(true) &
            p.price.isSmallerThanValue(500),
      ))
    .get();
```

Explanation:

* Two expressions are combined with `AND`.
* SQLite evaluates both conditions before returning rows.

---

# When to Use

Use expressions whenever you need to:

* Compare values.
* Filter rows.
* Perform calculations.
* Build join conditions.
* Create dynamic SQL queries.

Expressions are the foundation of almost every Drift query.

---

# When NOT to Use

Avoid expressions when:

* You're working entirely with in-memory Dart objects.
* The operation doesn't involve the database.

For data already loaded into memory, normal Dart expressions are appropriate.

---

# Best Practices

* Prefer Drift expressions over raw SQL.
* Combine expressions instead of nesting multiple queries.
* Keep expressions readable.
* Use helper methods for repeated conditions.
* Let SQLite perform calculations whenever possible.

---

# Common Mistakes

## Using Dart comparison operators

**Wrong**

```dart
u.age > 18
```

Explanation:

* Dart operators don't generate SQL expressions.

**Correct**

```dart
u.age.isBiggerThanValue(18)
```

Explanation:

* Drift methods generate the appropriate SQL.

---

## Using Dart logical operators

**Wrong**

```dart
u.isActive.equals(true) &&
u.age.isBiggerThanValue(18)
```

Explanation:

* `&&` evaluates Dart booleans.
* Drift expressions must use SQL operators.

**Correct**

```dart
u.isActive.equals(true) &
u.age.isBiggerThanValue(18)
```

Explanation:

* `&` generates SQL `AND`.

---

## Moving database logic into Dart

**Wrong**

Loading every row and filtering afterward.

Explanation:

* More data is transferred than necessary.
* SQLite is optimized for evaluating expressions efficiently.

**Correct**

Build expressions directly in the query so filtering happens inside the database.

---

# Related APIs

* Filtering
* Select
* Ordering
* Custom SQL Expressions
* Dynamic Queries

---

# Summary

Expressions are the building blocks of Drift queries. They represent SQL comparisons, calculations, and logical operations using Dart's type-safe APIs. By composing expressions, you can build powerful queries without writing raw SQL, while benefiting from compile-time checking and better maintainability.
