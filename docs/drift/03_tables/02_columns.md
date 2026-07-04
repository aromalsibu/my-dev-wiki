# Columns

Columns define the individual pieces of data stored in a table.

---

# What is it?

A column represents a single attribute of the data stored in a table. Every column has a name, a data type, and optional constraints that determine what values it can store.

For example, a `Users` table might contain columns such as:

* `id`
* `name`
* `email`
* `age`
* `createdAt`

Each row in the table stores one value for every column.

In Drift, columns are defined using builder methods such as `integer()`, `text()`, `boolean()`, and `dateTime()`.

---

# Why does it exist?

Columns define the structure and rules of your data.

Without columns, a table would have no way of knowing:

* What information it stores.
* What type of data is allowed.
* Whether values are required.
* Whether default values should be applied.

Drift uses column definitions to generate:

* SQLite table definitions.
* Data classes.
* Companion classes.
* Type-safe query APIs.

---

# Syntax

## Defining Columns

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  IntColumn get age => integer()();

  BoolColumn get isActive => boolean()();

  DateTimeColumn get createdAt => dateTime()();
}
```

Explanation:

* `integer()` creates an integer column.
* `text()` creates a text column.
* `boolean()` creates a boolean column.
* `dateTime()` creates a `DateTime` column.
* `autoIncrement()` makes the column the table's auto-incrementing primary key.

---

## Nullable Columns

```dart
TextColumn get bio => text().nullable()();
```

Explanation:

* `nullable()` allows the column to contain `NULL`.
* Without it, the column is required.

---

## Default Values

```dart
BoolColumn get isVerified =>
    boolean().withDefault(const Constant(false))();
```

Explanation:

* `withDefault()` assigns a value when none is provided during insertion.

---

# Mental Model

Think of columns as the fields on a form.

```text
User

Name        ___________

Email       ___________

Age         ___________

Active      ☑
```

Each field corresponds to one column in the database.

Every row fills in those fields with actual values.

---

# Examples

## Simple Example

A products table.

```dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  RealColumn get price => real()();
}
```

Explanation:

* Each getter defines one column in the table.
* Drift generates the corresponding SQLite schema automatically.

---

## Real-World Example

A blog posts table.

```dart
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get title => text()();

  TextColumn get content => text()();

  BoolColumn get published =>
      boolean().withDefault(const Constant(false))();

  DateTimeColumn get createdAt => dateTime()();

  DateTimeColumn get updatedAt =>
      dateTime().nullable()();
}
```

Explanation:

* `published` defaults to `false`.
* `updatedAt` is nullable because a new post may not have been edited yet.

---

# Common Column Types

| Drift Type   | SQLite Type | Dart Type   |
| ------------ | ----------- | ----------- |
| `integer()`  | INTEGER     | `int`       |
| `text()`     | TEXT        | `String`    |
| `real()`     | REAL        | `double`    |
| `boolean()`  | INTEGER     | `bool`      |
| `dateTime()` | INTEGER     | `DateTime`  |
| `blob()`     | BLOB        | `Uint8List` |

---

# When to Use

Define a column for every piece of information that belongs to an entity.

Examples include:

* User name
* Product price
* Order status
* Creation date
* Email address
* Profile picture

Each column should represent a single value.

---

# When NOT to Use

Avoid:

* Combining multiple values into one column.
* Creating duplicate columns.
* Storing unrelated data together.

For example, instead of:

```text
address = "New York, USA, 10001"
```

Prefer separate columns when the values need to be queried independently.

```text
city
country
zipCode
```

---

# Best Practices

* Choose the correct column type.
* Keep column names descriptive.
* Make columns nullable only when necessary.
* Use default values for predictable behavior.
* Avoid storing multiple pieces of information in one column.
* Keep columns focused on a single attribute.

---

# Common Mistakes

## Using the wrong column type

**Wrong**

```dart
TextColumn get age => text()();
```

Explanation:

* Age is numeric data and should not be stored as text.

**Correct**

```dart
IntColumn get age => integer()();
```

Explanation:

* Numeric values should use integer columns.

---

## Making everything nullable

**Wrong**

```dart
TextColumn get name => text().nullable()();
```

Explanation:

* If every column is nullable, your data becomes less reliable.

**Correct**

```dart
TextColumn get name => text()();
```

Explanation:

* Only allow `NULL` when the value is genuinely optional.

---

## Using text for boolean values

**Wrong**

```dart
TextColumn get active => text()();
```

Explanation:

* Storing `"true"` and `"false"` as text wastes space and complicates queries.

**Correct**

```dart
BoolColumn get active => boolean()();
```

Explanation:

* Use the appropriate column type for the data being stored.

---

# Related APIs

* Defining Tables
* Constraints
* Primary Keys
* Foreign Keys
* Default Values
* Generated Columns
* Custom Types

---

# Summary

Columns define the individual fields within a table. Each column has a specific data type and optional constraints that determine how data is stored and validated. Choosing the right column types and keeping each column focused on a single piece of information leads to a well-structured and maintainable database schema.
