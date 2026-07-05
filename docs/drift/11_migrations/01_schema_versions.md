# Schema Versions

A **schema version** is a number that represents the current structure of your database.

---

# What is it?

Every Drift database has a `schemaVersion` getter that returns an integer. This number tells Drift which version of your database schema the application expects.

Whenever you make changes to your database structure—such as adding tables, removing columns, or modifying constraints—you should increment the schema version.

When Drift opens the database, it compares:

* The schema version stored in the existing database.
* The schema version defined in your application.

If they differ, Drift knows that a migration is required.

---

# Why does it exist?

Applications evolve over time. New features often require changes to the database schema.

For example:

* Add a new table
* Add a new column
* Remove an old column
* Rename a table
* Create new indexes

Existing users already have a database on their device. Simply changing your table definitions doesn't update their database automatically.

Schema versions allow Drift to detect these changes and run the appropriate migration steps.

Without schema versions, users could end up with an outdated database structure, causing queries to fail or data to become inaccessible.

---

# Syntax

Every `GeneratedDatabase` must define a schema version.

```dart
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase(super.executor);

  @override
  int get schemaVersion => 1;
}
```

Explanation:

* `schemaVersion` specifies the expected database schema.
* New databases are created using this version.
* Existing databases compare their stored version against this value.

---

When you change the database schema, increment the version.

```dart
@override
int get schemaVersion => 2;
```

Explanation:

* Increasing the version tells Drift that the schema has changed.
* Drift will invoke your migration strategy when opening an existing database.

---

# Mental Model

Think of the schema version as a version number for your database blueprint.

```text
Version 1
┌─────────────────────┐
│ Users               │
│ id                  │
│ name                │
└─────────────────────┘

        ↓ Update App

Version 2
┌─────────────────────┐
│ Users               │
│ id                  │
│ name                │
│ email               │
└─────────────────────┘
```

When the app starts, Drift checks:

```text
Database on device: Version 1
Application expects: Version 2

        ↓

Run migration

        ↓

Database becomes Version 2
```

The schema version acts as a checkpoint that keeps the database structure synchronized with your application.

---

# Examples

## Simple Example

Version 1:

```dart
@override
int get schemaVersion => 1;
```

Explanation:

* The application expects the initial database structure.

---

Later, you add a new `email` column.

```dart
@override
int get schemaVersion => 2;
```

Explanation:

* The schema has changed.
* Drift will look for a migration from version 1 to version 2.

---

## Real-World Example

Imagine you're building a task management app.

### Version 1

```dart
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
}
```

```dart
@override
int get schemaVersion => 1;
```

Explanation:

* Tasks only store a title.

---

Months later, you introduce due dates.

```dart
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  DateTimeColumn get dueDate => dateTime().nullable()();
}
```

```dart
@override
int get schemaVersion => 2;
```

Explanation:

* The table structure has changed.
* Existing users need a migration to add the new column.
* New users receive the latest schema directly.

---

# When to Use

Use schema versions whenever your database schema changes, including:

* Adding tables
* Removing tables
* Adding columns
* Removing columns
* Renaming tables
* Renaming columns
* Changing constraints
* Adding indexes
* Modifying views or triggers

---

# When NOT to Use

Do **not** change the schema version when:

* Updating query logic.
* Changing DAO methods.
* Refactoring Dart code.
* Modifying business logic.
* Changing repository implementations.

Only increment the version when the actual SQLite schema changes.

---

# Best Practices

* Start with version `1`.
* Increment the version by one for each schema change.
* Keep migration logic synchronized with schema versions.
* Never skip writing migrations for released versions.
* Test migrations before releasing updates.
* Treat schema versions as part of your application's public data model.

---

# Common Mistakes

## Forgetting to Increment the Version

**Wrong**

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get email => text()();
}

@override
int get schemaVersion => 1;
```

The table changed, but the version didn't.

As a result, Drift won't run a migration, and existing databases won't have the `email` column.

**Correct**

```dart
@override
int get schemaVersion => 2;
```

Increment the version whenever the schema changes.

---

## Incrementing the Version Without a Migration

**Wrong**

```dart
@override
int get schemaVersion => 3;
```

If users upgrade from an older version, Drift detects the version change, but without migration logic, the database can't be safely updated.

**Correct**

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (migrator, from, to) async {
    // Apply schema changes here.
  },
);
```

Always provide migration logic for released schema changes.

---

## Incrementing the Version for Non-Schema Changes

**Wrong**

```dart
Future<List<Task>> getCompletedTasks() {
  // Improved query
}
```

```dart
@override
int get schemaVersion => 4;
```

Only application code changed, not the database schema.

**Correct**

Leave the schema version unchanged.

Increment it only when the database structure itself changes.

---

# Related APIs

* MigrationStrategy
* Migrator
* GeneratedDatabase
* Table
* TableMigration

---

# Summary

The `schemaVersion` identifies the current structure of your database. Drift compares this version with the one stored in the existing database to determine whether a migration is needed. Every structural database change should be accompanied by a schema version increment and the corresponding migration logic to ensure existing users can safely upgrade their databases.
