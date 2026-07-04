# Table Inheritance

Table inheritance lets multiple tables share common column definitions through Dart inheritance.

---

# What is it?

As your application grows, you may have several tables that contain the same columns.

For example:

* `createdAt`
* `updatedAt`
* `id`
* `deletedAt`

Instead of defining these columns in every table, you can place them in a base table class and have other tables inherit from it.

This reduces duplication and keeps table definitions consistent.

---

# Why does it exist?

Without inheritance, shared columns must be copied into every table.

For example:

```text
Users
├── createdAt
├── updatedAt

Products
├── createdAt
├── updatedAt

Orders
├── createdAt
├── updatedAt
```

If you later decide to modify one of these shared columns, you'll need to update every table individually.

Table inheritance allows common columns to be defined once and reused across multiple tables.

---

# Syntax

## Creating a Base Table

```dart
abstract class BaseTable extends Table {
  IntColumn get id => integer().autoIncrement()();

  DateTimeColumn get createdAt => dateTime()();

  DateTimeColumn get updatedAt => dateTime()();
}
```

Explanation:

* `BaseTable` contains columns shared by multiple tables.
* Declaring it as `abstract` indicates it isn't meant to become a table on its own.

---

## Inheriting the Base Table

```dart
class Users extends BaseTable {
  TextColumn get name => text()();
}

class Products extends BaseTable {
  TextColumn get title => text()();
}
```

Explanation:

* `Users` and `Products` inherit all columns from `BaseTable`.
* Each table can define its own additional columns.

---

# Mental Model

Think of table inheritance as a template.

```text
          BaseTable
        ┌─────────────┐
        │ id          │
        │ createdAt   │
        │ updatedAt   │
        └──────┬──────┘
               │
      ┌────────┴────────┐
      ▼                 ▼
    Users           Products
      │                 │
     name             title
```

Each table receives the shared columns while adding its own unique fields.

---

# Examples

## Simple Example

A base table for timestamps.

```dart
abstract class TimestampedTable extends Table {
  DateTimeColumn get createdAt => dateTime()();

  DateTimeColumn get updatedAt => dateTime()();
}

class Notes extends TimestampedTable {
  TextColumn get title => text()();
}
```

Explanation:

* `Notes` automatically includes both timestamp columns.
* Only table-specific columns need to be added.

---

## Real-World Example

An e-commerce application.

```dart
abstract class EntityTable extends Table {
  IntColumn get id => integer().autoIncrement()();

  DateTimeColumn get createdAt => dateTime()();

  DateTimeColumn get updatedAt => dateTime()();
}

class Customers extends EntityTable {
  TextColumn get name => text()();
}

class Orders extends EntityTable {
  DateTimeColumn get orderDate => dateTime()();
}

class Products extends EntityTable {
  TextColumn get title => text()();
}
```

Explanation:

* Every table shares a common structure.
* Changes to shared columns only need to be made in one place.

---

# When to Use

Use table inheritance when:

* Multiple tables share the same columns.
* You want to reduce duplicated code.
* Common fields should remain consistent across tables.
* Building large applications with many tables.

---

# When NOT to Use

Avoid table inheritance when:

* Only one table uses the shared columns.
* The shared columns are unrelated.
* Inheritance makes the table hierarchy harder to understand.

Don't create deep inheritance hierarchies just to avoid a few repeated lines of code.

---

# Best Practices

* Keep base tables focused on genuinely shared columns.
* Make base table classes `abstract`.
* Avoid multiple levels of inheritance.
* Use descriptive names such as `BaseTable` or `TimestampedTable`.
* Keep table-specific columns in the derived table.

---

# Common Mistakes

## Duplicating shared columns

**Wrong**

```dart
class Users extends Table {
  DateTimeColumn get createdAt => dateTime()();

  DateTimeColumn get updatedAt => dateTime()();
}

class Products extends Table {
  DateTimeColumn get createdAt => dateTime()();

  DateTimeColumn get updatedAt => dateTime()();
}
```

Explanation:

* The same columns are repeated across multiple tables.

**Correct**

```dart
abstract class TimestampedTable extends Table {
  DateTimeColumn get createdAt => dateTime()();

  DateTimeColumn get updatedAt => dateTime()();
}
```

Explanation:

* Shared columns are defined once and inherited where needed.

---

## Putting unrelated columns in the base table

**Wrong**

```dart
abstract class BaseTable extends Table {
  TextColumn get productName => text()();

  TextColumn get customerName => text()();
}
```

Explanation:

* Not every table needs these columns.

**Correct**

Only include columns that are truly common to all inheriting tables.

---

## Overusing inheritance

**Wrong**

```text
BaseTable
    │
TimestampedTable
    │
AuditableTable
    │
SoftDeleteTable
    │
Users
```

Explanation:

* Deep inheritance hierarchies make table definitions harder to follow.

**Correct**

Prefer a simple, shallow inheritance structure.

---

# Related APIs

* Defining Tables
* Columns
* Generated Data Classes
* TypeConverter
* Constraints

---

# Summary

Table inheritance allows multiple Drift tables to share common column definitions through Dart inheritance. It helps reduce duplication, improves consistency, and makes large database schemas easier to maintain by centralizing shared fields in a reusable base table.
