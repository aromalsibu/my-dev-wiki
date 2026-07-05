# Migration Examples

**Migration examples** show how to perform common database schema changes while preserving existing user data.

---

# What is it?

As your application evolves, your database schema will change too. Drift's `Migrator` API provides methods to safely apply those changes during `onUpgrade()`.

Some migrations are simple, such as adding a column. Others, like deleting a column or changing its type, require rebuilding the table. Drift's `TableMigration` API automates much of that complex process. ([Drift][1])

---

# Why does it exist?

Existing users already have a database on their device.

When your schema changes, Drift needs instructions to transform that database into the new structure without losing data.

Migration examples demonstrate the recommended way to perform these updates safely.

---

# Syntax

Every migration is written inside `onUpgrade()`.

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 2) {
      // Migration logic
    }
  },
);
```

**Explanation:**

* `from` is the current database version.
* `to` is the latest schema version.
* Each migration should only run when needed.

---

# Mental Model

Think of each migration as one small upgrade step.

```text
Version 1
     │
     ▼
Migration
     │
     ▼
Version 2
     │
     ▼
Migration
     │
     ▼
Version 3
```

Instead of rebuilding the entire database every time, Drift applies only the missing steps.

---

# Examples

## Adding a Column

This is the most common migration.

Suppose you add an `email` column to the `Users` table.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().nullable()();
}
```

Update the migration:

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 2) {
      await m.addColumn(users, users.email);
    }
  },
);
```

**Explanation:**

* `addColumn()` issues an `ALTER TABLE ADD COLUMN`.
* Existing rows remain unchanged.
* The new nullable column is added safely. ([Drift][2])

---

## Creating a New Table

Suppose you introduce a new table.

```dart
class Categories extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}
```

Migration:

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 3) {
      await m.createTable(categories);
    }
  },
);
```

**Explanation:**

* `createTable()` creates only the specified table.
* Existing tables and data are left untouched. ([Dart packages][3])

---

## Renaming a Column

Suppose you rename `username` to `displayName`.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get displayName => text()();
}
```

Migration:

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 4) {
      await m.renameColumn(
        users,
        'username',
        users.displayName,
      );
    }
  },
);
```

**Explanation:**

* `renameColumn()` changes the column name without recreating the table when supported.
* Existing values are preserved. ([Dart packages][3])

---

## Removing a Column

SQLite doesn't directly support removing columns.

Drift uses `TableMigration` to rebuild the table.

Suppose the `nickname` column is removed.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}
```

Migration:

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 5) {
      await m.alterTable(
        TableMigration(users),
      );
    }
  },
);
```

**Explanation:**

* Drift creates a temporary table.
* Existing data is copied.
* The old table is replaced with the new schema automatically. ([Drift][1])

---

## Adding a Required Column

Adding a non-nullable column without a database default isn't possible with `addColumn()` alone.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get status => text()();
}
```

Migration:

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 6) {
      await m.alterTable(
        TableMigration(
          users,
          newColumns: [users.status],
          columnTransformer: {
            users.status: const Constant('active'),
          },
        ),
      );
    }
  },
);
```

**Explanation:**

* `newColumns` tells Drift this column doesn't exist yet.
* `columnTransformer` provides a value for existing rows.
* New inserts still follow the table definition. ([Drift][1])

---

## Real-World Example

A production migration often contains several independent upgrade steps.

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 2) {
      await m.addColumn(users, users.email);
    }

    if (from < 3) {
      await m.createTable(categories);
    }

    if (from < 4) {
      await m.renameColumn(
        users,
        'username',
        users.displayName,
      );
    }
  },
);
```

**Explanation:**

* Each block handles one schema version.
* Users upgrading from any previous version receive only the migrations they need.
* Every migration is independent, making future maintenance easier.

---

# When to Use

Write migrations when you:

* Add tables.
* Remove tables.
* Add columns.
* Remove columns.
* Rename columns.
* Change column types.
* Add indexes.
* Modify constraints.

---

# When NOT to Use

Don't write migrations for:

* Query changes.
* Repository changes.
* DAO refactoring.
* UI updates.
* Business logic changes.

Only database schema changes require migrations.

---

# Best Practices

* Keep one migration per schema version.
* Use `addColumn()` whenever possible.
* Use `TableMigration` only for complex schema changes.
* Keep migrations small and independent.
* Never modify old released migrations.
* Test every migration before release.

---

# Common Mistakes

## Running Every Migration Unconditionally

**Wrong**

```dart
onUpgrade: (m, from, to) async {
  await m.addColumn(users, users.email);
}
```

This attempts to add the column even if it already exists.

**Correct**

```dart
onUpgrade: (m, from, to) async {
  if (from < 2) {
    await m.addColumn(users, users.email);
  }
}
```

Only execute migrations that haven't already been applied.

---

## Using `TableMigration` for Simple Changes

**Wrong**

```dart
await m.alterTable(
  TableMigration(users),
);
```

Adding a nullable column doesn't require rebuilding the table.

**Correct**

```dart
await m.addColumn(
  users,
  users.email,
);
```

Prefer the simpler API whenever it supports your change. ([Drift][1])

---

## Forgetting Existing Rows

**Wrong**

```dart
TextColumn get status => text()();
```

```dart
await m.addColumn(users, users.status);
```

Existing rows don't have a value for the new required column.

**Correct**

```dart
await m.alterTable(
  TableMigration(
    users,
    newColumns: [users.status],
    columnTransformer: {
      users.status: const Constant('active'),
    },
  ),
);
```

Provide a value for existing rows during the migration. ([Drift][1])

---

# Related APIs

* Schema Versions
* MigrationStrategy
* Migrator
* TableMigration
* CustomExpression

---

# Summary

Drift provides different migration APIs depending on the type of schema change. Simple changes like adding a column or creating a table use dedicated `Migrator` methods, while complex changes such as removing columns or transforming data use `TableMigration`. Choosing the appropriate API keeps migrations simple, safe, and easy to maintain.

[1]: https://drift.simonbinder.eu/migrations/api/?utm_source=chatgpt.com "The migrator API"
[2]: https://drift.simonbinder.eu/migrations/?utm_source=chatgpt.com "Migrations"
[3]: https://pub.dev/documentation/drift/latest/drift/Migrator-class.html?utm_source=chatgpt.com "Migrator class - drift library - Dart API"
