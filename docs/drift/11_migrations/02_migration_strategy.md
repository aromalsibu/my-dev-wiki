# Migration Strategy

A **MigrationStrategy** defines how Drift creates and upgrades your database when its schema version changes.

---

# What is it?

`MigrationStrategy` is a configuration object that tells Drift what to do in different stages of a database's lifecycle.

It contains callbacks that are automatically invoked when:

* A new database is created.
* An existing database needs to be upgraded.
* The database has finished opening.

Typically, you'll override the `migration` getter in your `GeneratedDatabase` and return a `MigrationStrategy`.

---

# Why does it exist?

Changing `schemaVersion` only tells Drift that the database structure has changed—it doesn't tell Drift *how* to perform the update.

For example, if you add a new column:

* Existing databases don't automatically gain that column.
* Drift needs instructions for updating the schema safely.

`MigrationStrategy` provides those instructions.

Without it, upgrading users could encounter missing tables, missing columns, or incompatible schemas.

---

# Syntax

Override the `migration` getter in your database.

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (migrator) async {
    await migrator.createAll();
  },
  onUpgrade: (migrator, from, to) async {
    // Handle schema upgrades.
  },
  beforeOpen: (details) async {
    // Run setup after opening.
  },
);
```

Explanation:

* `onCreate` runs when the database is created for the first time.
* `onUpgrade` runs when an existing database needs to move to a newer schema version.
* `beforeOpen` runs every time the database is opened, after creation or migration.

---

# Mental Model

Think of `MigrationStrategy` as the database's startup checklist.

```text
Open Database
      │
      ▼
Does database exist?
      │
 ┌────┴────┐
 │         │
No        Yes
 │         │
 ▼         ▼
onCreate   Compare schema versions
              │
      Versions different?
              │
        ┌─────┴─────┐
        │           │
       No          Yes
        │           │
        ▼           ▼
      beforeOpen  onUpgrade
                     │
                     ▼
                 beforeOpen
```

Each callback has a specific responsibility, ensuring the database is always in the expected state.

---

# Examples

## Simple Example

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (migrator) async {
    await migrator.createAll();
  },
);
```

Explanation:

* `createAll()` creates every table, view, trigger, and index defined in your database.
* This is enough if your app has never released a schema update.

---

## Real-World Example

Suppose version 2 adds an `email` column to the `Users` table.

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (migrator) async {
    await migrator.createAll();
  },
  onUpgrade: (migrator, from, to) async {
    if (from < 2) {
      await migrator.addColumn(users, users.email);
    }
  },
  beforeOpen: (details) async {
    await customStatement('PRAGMA foreign_keys = ON');
  },
);
```

Explanation:

* New users get the complete schema through `createAll()`.
* Existing users upgrading from version 1 receive the new `email` column.
* `beforeOpen()` enables SQLite foreign key enforcement every time the database opens.

---

# When to Use

Use `MigrationStrategy` whenever your database may need to:

* Create tables.
* Upgrade schemas.
* Perform initialization after opening.
* Support users upgrading from previous app versions.

In practice, almost every production Drift database should define one.

---

# When NOT to Use

Avoid leaving the migration strategy empty in production applications.

If your database schema is expected to evolve, you should always provide migration logic.

---

# Best Practices

* Always implement `onCreate()`.
* Use `migrator.createAll()` for initial database creation.
* Keep upgrade logic incremental.
* Check the `from` version before applying each migration.
* Keep migrations deterministic and repeatable.
* Test every released migration path.

---

# Common Mistakes

## Forgetting to Implement `onUpgrade`

**Wrong**

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (m) async => m.createAll(),
);
```

The database can be created, but existing users can't be upgraded safely.

**Correct**

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (m) async => m.createAll(),
  onUpgrade: (m, from, to) async {
    // Apply required migrations.
  },
);
```

Always provide upgrade logic once your app has released a schema.

---

## Ignoring the Previous Version

**Wrong**

```dart
onUpgrade: (m, from, to) async {
  await m.addColumn(users, users.email);
}
```

This tries to add the column regardless of the current version.

**Correct**

```dart
onUpgrade: (m, from, to) async {
  if (from < 2) {
    await m.addColumn(users, users.email);
  }
}
```

Only apply migrations that haven't already been executed.

---

## Putting Schema Changes in `beforeOpen`

**Wrong**

```dart
beforeOpen: (details) async {
  await migrator.addColumn(users, users.email);
}
```

`beforeOpen` isn't intended for schema migrations.

**Correct**

```dart
onUpgrade: (m, from, to) async {
  if (from < 2) {
    await m.addColumn(users, users.email);
  }
}
```

Keep schema modifications inside `onUpgrade()`.

---

# Related APIs

* Schema Versions
* Migrator
* TableMigration
* GeneratedDatabase
* DatabaseConnection

---

# Summary

`MigrationStrategy` is the central place for managing your database's lifecycle. It tells Drift how to create a new database, upgrade existing ones, and perform setup after opening. By defining clear migration logic, you ensure users can safely move between schema versions without losing data.
