# What is Drift?

Drift is a reactive persistence library for Dart and Flutter that provides a type-safe API for working with SQLite databases.

---

# What is it?

Drift is an Object Relational Mapping (ORM) library built on top of SQLite. It allows you to define your database schema in Dart, generate strongly typed APIs, and interact with your database without writing most SQL manually.

Instead of dealing with raw SQL strings and manually mapping query results, Drift generates Dart classes and query builders that are checked at compile time.

Drift combines the reliability of SQLite with the productivity of modern Dart development.

---

# Why does it exist?

Working directly with SQLite can become repetitive and error-prone.

Without Drift, developers often need to:

- Write SQL queries as strings
- Manually convert database rows into Dart objects
- Handle database updates manually
- Maintain SQL and Dart code separately

Drift solves these problems by providing:

- Type-safe queries
- Automatic code generation
- Generated data classes
- Reactive query streams
- Compile-time validation for many database operations

This results in cleaner, safer, and more maintainable database code.

---

# Syntax

A simple Drift database starts with defining tables and creating a database class.

```dart
@DriftDatabase(tables: [Todos])
class AppDatabase extends _$AppDatabase {
  AppDatabase(super.executor);

  @override
  int get schemaVersion => 1;
}
```

Explanation:

- `@DriftDatabase` tells Drift which tables belong to the database.
- `tables` lists all table definitions.
- `_$AppDatabase` is generated during code generation.
- `schemaVersion` specifies the current database schema version.

---

# Mental Model

Think of Drift as a translator between your Dart code and SQLite.

```
Flutter App
      │
      ▼
   Drift APIs
      │
      ▼
Generated Code
      │
      ▼
   SQLite Database
```

You interact with Dart objects and query builders.

Drift generates the SQL needed to communicate with SQLite.

This allows you to work in Dart while still benefiting from SQLite's performance and reliability.

---

# Examples

## Simple Example

Suppose you have a table of users.

Instead of writing SQL manually, you can query it using Drift's generated APIs.

```dart
final users = await db.select(db.users).get();
```

Explanation:

- `select()` creates a type-safe query.
- `get()` executes the query and returns all matching rows.

---

## Real-World Example

Imagine a notes application.

You might have tables for:

- Notes
- Categories
- Attachments

Drift can:

- Generate data classes for each table.
- Build type-safe queries.
- Automatically update the UI when notes change.
- Handle relationships between tables.
- Support database migrations as the app evolves.

This allows you to build complex local databases without managing SQL manually.

---

# When to Use

Use Drift when:

- Building offline-first applications.
- Using SQLite in Flutter.
- You want type-safe database access.
- Your project contains complex queries.
- Your application requires relationships between tables.
- You need reactive database updates.
- You expect your database schema to evolve over time.

---

# When NOT to Use

Drift may not be the best choice when:

- Your app only stores a few simple key-value pairs.
- No relational data is required.
- You don't need SQL features.

In those cases, simpler storage solutions may be sufficient.

---

# Best Practices

- Organize tables into separate files.
- Keep queries inside DAOs.
- Prefer Drift's query builder over raw SQL.
- Use migrations when changing schemas.
- Watch queries only when reactive updates are needed.
- Keep generated files out of manual edits.

---

# Common Mistakes

## Writing unnecessary raw SQL

**Wrong**

```dart
customSelect('SELECT * FROM users');
```

Explanation:

- Raw SQL bypasses many of Drift's type-safe APIs.

**Correct**

```dart
select(users).get();
```

Explanation:

- Drift generates safer, strongly typed queries.

---

## Editing generated files

**Wrong**

```dart
// Editing app_database.g.dart
```

Explanation:

- Generated files are overwritten during code generation.

**Correct**

Modify your table or database definitions instead and regenerate the code.

---

## Ignoring schema versions

**Wrong**

Changing tables without updating the schema version.

Explanation:

- Drift cannot apply migrations correctly.

**Correct**

Increase the schema version and provide the appropriate migration.

---

# Related APIs

- Installation
- Setting Up a Database
- GeneratedDatabase
- Defining Tables
- Code Generation
- Generated Data Classes
- Select
- Watching Queries
- Transactions
- Migration Strategy

---

# Summary

Drift is a type-safe SQLite ORM for Dart and Flutter. It generates strongly typed database APIs, supports reactive queries, simplifies CRUD operations, and provides migration support, making it easier to build reliable and maintainable local databases.