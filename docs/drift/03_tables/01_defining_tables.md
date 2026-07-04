# Defining Tables

Define the structure of your database by creating tables with columns and constraints.

---

# What is it?

A table represents a collection of related data in your database. Every row in a table represents a single record, while each column stores a specific piece of information about that record.

In Drift, tables are defined as Dart classes by extending the `Table` class.

Instead of writing SQL like `CREATE TABLE`, you describe the table using Dart APIs, and Drift generates the corresponding SQLite schema.

---

# Why does it exist?

Traditionally, database tables are created using SQL statements.

For example:

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL
);
```

While SQL is powerful, it doesn't integrate well with Dart's type system.

Drift lets you define tables in Dart, which provides:

* Compile-time validation.
* Strong typing.
* Automatic code generation.
* Better IDE support.
* Easier refactoring.

---

# Syntax

## Creating a Table

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();
}
```

Explanation:

* `extends Table` tells Drift this class defines a database table.
* `integer()` creates an integer column.
* `text()` creates a text column.
* `autoIncrement()` marks the column as an auto-incrementing primary key.
* The final `()` completes the column definition.

---

## Registering the Table

After defining a table, register it in your database.

```dart
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase(super.executor);

  @override
  int get schemaVersion => 1;
}
```

Explanation:

* Only registered tables become part of the database.
* Drift generates APIs for every registered table.

---

# Mental Model

Think of a table as a spreadsheet.

```text
Users
+----+---------+
| ID | Name    |
+----+---------+
| 1  | Alice   |
| 2  | Bob     |
| 3  | Charlie |
+----+---------+
```

The table definition describes the structure, while the rows contain the actual data.

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

* Each getter defines one column.
* Drift generates the SQL and corresponding Dart classes automatically.

---

## Real-World Example

A task management application.

```dart
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get title => text()();

  TextColumn get description => text().nullable()();

  BoolColumn get completed => boolean().withDefault(const Constant(false))();

  DateTimeColumn get createdAt => dateTime()();
}
```

Explanation:

* `nullable()` allows a column to store `NULL`.
* `withDefault()` provides a default value for new rows.
* Drift generates a `Task` data class and a `TasksCompanion` for inserts and updates.

---

# When to Use

Define a table whenever you need to store a new type of data.

Examples include:

* Users
* Products
* Orders
* Notes
* Messages
* Settings

Each table should represent a single type of entity.

---

# When NOT to Use

Avoid creating a new table when:

* The data naturally belongs in an existing table.
* A simple column is sufficient.
* You're storing temporary, in-memory data.

Also, don't create one table for every screen in your application. Tables should model your data, not your UI.

---

# Best Practices

* Use singular concepts for each table (e.g., a `Users` table stores user records).
* Give columns descriptive names.
* Keep tables focused on a single entity.
* Add only the columns you need.
* Use appropriate column types.
* Organize table definitions into separate files as your project grows.

---

# Common Mistakes

## Forgetting to extend `Table`

**Wrong**

```dart
class Users {}
```

Explanation:

* Drift won't recognize this as a database table.

**Correct**

```dart
class Users extends Table {}
```

Explanation:

* Extending `Table` enables Drift's table generation.

---

## Forgetting to register the table

**Wrong**

```dart
class Users extends Table {
  // ...
}
```

But not adding it to the database.

Explanation:

* Drift only generates APIs for registered tables.

**Correct**

```dart
@DriftDatabase(
  tables: [Users],
)
```

Explanation:

* Register every table in the `@DriftDatabase` annotation.

---

## Mixing unrelated data

**Wrong**

```text
Users Table
------------
id
name
email
productName
orderDate
invoiceNumber
```

Explanation:

* A table should represent one entity, not multiple unrelated concepts.

**Correct**

```text
Users
Orders
Products
Invoices
```

Explanation:

* Split different entities into separate tables and relate them when necessary.

---

# Related APIs

* Columns
* Constraints
* Primary Keys
* Foreign Keys
* Generated Data Classes
* GeneratedDatabase

---

# Summary

Tables define the structure of your database. In Drift, you create tables by extending the `Table` class and defining columns using Dart APIs. Drift then generates the SQL schema, data classes, and query helpers, giving you a type-safe and maintainable way to model your application's data.
