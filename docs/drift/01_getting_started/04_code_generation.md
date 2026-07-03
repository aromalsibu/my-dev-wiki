# Code Generation

Generate type-safe Dart code from your Drift database definitions.

---

# What is it?

Code generation is the process where Drift analyzes your annotated database classes, tables, DAOs, and SQL files, then generates Dart code that you use in your application.

Instead of writing repetitive boilerplate code yourself, Drift creates it automatically.

The generated code includes:

* Database implementations
* Data classes
* Companion classes
* Query APIs
* Table metadata
* DAO implementations

You should never edit this generated code manually.

---

# Why does it exist?

Without code generation, you would have to manually write code for:

* Mapping database rows to Dart objects.
* Creating insert and update objects.
* Building query helpers.
* Validating table definitions.
* Connecting tables to the database.

This is repetitive, error-prone, and difficult to maintain.

Drift generates this code for you, allowing you to focus on defining your database instead of implementing its plumbing.

---

# Syntax

## Mark Classes for Generation

```dart
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase(super.executor);

  @override
  int get schemaVersion => 1;
}
```

Explanation:

* `@DriftDatabase` tells Drift to generate database code.
* `_$AppDatabase` is created during code generation.

---

## Include the Generated File

```dart
part 'app_database.g.dart';
```

Explanation:

* Connects your source file with the generated implementation.

---

## Generate the Code

```bash
dart run build_runner build
```

Explanation:

* Generates all missing or updated Drift files once.

---

## Automatically Regenerate Code

```bash
dart run build_runner watch
```

Explanation:

* Watches your project for changes and regenerates code automatically.

---

# Mental Model

Think of code generation as a compiler for your database definitions.

```text
Your Dart Code
    │
    ▼
Annotations
    │
    ▼
Drift Generator
    │
    ▼
Generated .g.dart Files
    │
    ▼
Your Application
```

You write the database schema.

Drift writes the implementation.

---

# Examples

## Simple Example

Suppose you define a table.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();
}
```

Explanation:

* This is all you write manually.

After running the generator, Drift creates:

* A data class (`User`)
* A companion (`UsersCompanion`)
* Table information
* Query helpers

All of this is generated automatically.

---

## Real-World Example

Imagine adding a new column.

```dart
TextColumn get email => text()();
```

After running code generation:

* The generated data class now includes `email`.
* Companion classes are updated.
* Query APIs understand the new column.
* Type safety is preserved automatically.

No manual updates are required.

---

# When to Use

Run code generation whenever you:

* Add a new table.
* Modify a table.
* Create or update a DAO.
* Change a database class.
* Add SQL query files.
* Update annotations used by Drift.

---

# When NOT to Use

You should not run code generation if nothing has changed in your annotated files.

Also, never attempt to replace generated files with manually written implementations.

---

# Best Practices

* Use `build_runner watch` during development.
* Commit generated files if it matches your team's workflow.
* Never modify generated files.
* Keep your annotations clean and organized.
* Regenerate code after every schema change.

---

# Common Mistakes

## Editing generated files

**Wrong**

```dart
// app_database.g.dart

class _$AppDatabase {
  // Modified manually
}
```

Explanation:

* Your changes will be overwritten the next time code generation runs.

**Correct**

Modify the source file instead.

```dart
class Users extends Table {
  // Update the table definition here
}
```

Explanation:

* Source files are the only files you should edit.

---

## Forgetting to regenerate

**Wrong**

Adding a new table but not running:

```bash
dart run build_runner build
```

Explanation:

* The generated APIs won't include your changes.

**Correct**

Run the generator after every change to annotated Drift code.

---

## Missing the `part` directive

**Wrong**

```dart
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {}
```

Explanation:

* The generated code cannot be linked without the `part` directive.

**Correct**

```dart
part 'app_database.g.dart';
```

Explanation:

* Includes the generated implementation in your source file.

---

# Related APIs

* Installation
* Setting Up a Database
* GeneratedDatabase
* Defining Tables
* Generated Data Classes
* Companions

---

# Summary

Code generation is a core part of Drift. It transforms your database definitions into strongly typed Dart code, eliminating boilerplate and providing compile-time safety. By defining your schema and letting Drift generate the implementation, you get a cleaner, safer, and more maintainable database layer.
