# Constraints

Constraints define rules that control what data can be stored in a table.

---

# What is it?

Constraints are rules applied to columns or tables to ensure that only valid data is stored in the database.

For example, a constraint can require that:

* A value must not be `NULL`.
* A value must be unique.
* A number must be positive.
* A referenced row must exist in another table.

By enforcing these rules at the database level, SQLite helps maintain data integrity regardless of where the data comes from.

---

# Why does it exist?

Without constraints, invalid or inconsistent data can easily enter your database.

For example:

* Two users could have the same email.
* An order could reference a customer that doesn't exist.
* A product could have a negative price.
* Required fields could be left empty.

Constraints prevent these situations automatically, reducing bugs and improving data reliability.

---

# Syntax

## NOT NULL

By default, Drift columns are required unless marked as nullable.

```dart
TextColumn get name => text()();
```

Explanation:

* The column cannot store `NULL`.
* Every row must provide a value for `name`.

---

## Nullable

```dart
TextColumn get bio => text().nullable()();
```

Explanation:

* `nullable()` allows the column to contain `NULL`.
* Use it only for optional data.

---

## UNIQUE

```dart
TextColumn get email => text().unique()();
```

Explanation:

* Every email value must be unique.
* Attempting to insert a duplicate value results in an error.

---

## CHECK

```dart
IntColumn get age =>
    integer().check(age.isBiggerOrEqualValue(0))();
```

Explanation:

* `check()` enforces a custom validation rule.
* Here, only non-negative ages are allowed.

---

# Mental Model

Think of constraints as security guards for your database.

```text
        Insert Row
            │
            ▼
     Constraint Checks
            │
   ┌────────┼────────┐
   │        │        │
 NOT NULL UNIQUE CHECK
   │        │        │
   └────────┼────────┘
            │
     Valid Data?
       │       │
      Yes      No
       │        │
       ▼        ▼
   Store Row   Reject Row
```

Before SQLite stores a row, it verifies that all constraints are satisfied.

---

# Examples

## Simple Example

A users table with required and unique fields.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  TextColumn get email => text().unique()();
}
```

Explanation:

* `name` is required.
* `email` must be unique across all users.

---

## Real-World Example

A products table with validation.

```dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  RealColumn get price =>
      real().check(price.isBiggerThanValue(0))();

  IntColumn get stock =>
      integer().check(stock.isBiggerOrEqualValue(0))();
}
```

Explanation:

* Product prices must be greater than zero.
* Stock cannot be negative.
* Invalid rows are rejected before they're stored.

---

# Common Constraints in Drift

| Constraint        | Purpose                                  |
| ----------------- | ---------------------------------------- |
| `nullable()`      | Allows `NULL` values                     |
| `unique()`        | Prevents duplicate values                |
| `check()`         | Validates values using an expression     |
| `autoIncrement()` | Creates an auto-incrementing primary key |
| `references()`    | Creates a foreign key relationship       |
| `withDefault()`   | Assigns a default value                  |

---

# When to Use

Use constraints whenever your data has rules that must always be enforced.

Examples include:

* Email addresses must be unique.
* Prices cannot be negative.
* Every task must have a title.
* Orders must reference existing customers.
* Status values must follow specific rules.

---

# When NOT to Use

Avoid using constraints for business rules that change frequently.

For example:

* User permission checks.
* Feature flags.
* Application-specific workflows.

These are usually better handled in your application's business logic rather than the database.

---

# Best Practices

* Let the database enforce data integrity.
* Make required fields non-nullable.
* Use `unique()` only when duplicates should never exist.
* Use `check()` for simple validation rules.
* Keep constraints simple and easy to understand.

---

# Common Mistakes

## Making required fields nullable

**Wrong**

```dart
TextColumn get username => text().nullable()();
```

Explanation:

* Usernames are typically required.
* Allowing `NULL` may lead to incomplete records.

**Correct**

```dart
TextColumn get username => text()();
```

Explanation:

* Required fields should remain non-nullable.

---

## Forgetting unique constraints

**Wrong**

```dart
TextColumn get email => text()();
```

Explanation:

* Multiple users could register with the same email address.

**Correct**

```dart
TextColumn get email => text().unique()();
```

Explanation:

* The database guarantees email uniqueness.

---

## Relying only on application validation

**Wrong**

Checking values only before insertion in Dart code.

Explanation:

* Data inserted from another source or future code changes could bypass these checks.

**Correct**

Use database constraints alongside application validation.

Explanation:

* The application improves the user experience.
* The database guarantees data integrity.

---

# Related APIs

* Columns
* Primary Keys
* Composite Keys
* Foreign Keys
* Default Values
* Generated Columns

---

# Summary

Constraints enforce rules that keep your database consistent and reliable. Drift provides APIs for defining common constraints such as required fields, uniqueness, and custom validation, allowing SQLite to reject invalid data before it's stored. Proper use of constraints helps protect your data and reduces the need for repetitive validation throughout your application.
