# Database Configuration

Configure how your Drift database behaves, including its schema version, migrations, and lifecycle hooks.

---

# What is it?

Database configuration is the process of defining settings that control how your database is initialized and managed.

These settings determine things such as:

* The current schema version.
* How migrations are performed.
* What happens when the database is first created.
* How upgrades and downgrades are handled.

These configurations are typically defined by overriding properties or methods in your database class.

---

# Why does it exist?

As your application grows, your database schema will evolve. New tables, columns, and relationships may be added over time.

Database configuration allows Drift to:

* Keep track of schema changes.
* Upgrade existing databases safely.
* Run initialization logic.
* Maintain compatibility between app versions.

Without proper configuration, users could lose data or encounter database errors after updating the app.

---

# Syntax

## Setting the Schema Version

```dart
@override
int get schemaVersion => 1;
```

Explanation:

* `schemaVersion` identifies the current version of your database schema.
* Increase this value whenever the database structure changes.

---

## Configuring Migrations

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onCreate: (m) async {
    await m.createAll();
  },
  onUpgrade: (m, from, to) async {
    // Migration logic
  },
);
```

Explanation:

* `MigrationStrategy` defines how the database is created and upgraded.
* `onCreate` runs when the database is created for the first time.
* `onUpgrade` runs when moving between schema versions.

---

# Mental Model

Think of database configuration as the blueprint for your database's lifecycle.

```text
Database Starts
       │
       ▼
Read Configuration
       │
       ├── Schema Version
       ├── Migration Strategy
       ├── Creation Logic
       └── Upgrade Logic
       │
       ▼
Database Ready
```

Before Drift begins executing queries, it checks this configuration to ensure the database is in the expected state.

---

# Examples

## Simple Example

A basic configuration with only a schema version.

```dart
class AppDatabase extends _$AppDatabase {
  @override
  int get schemaVersion => 1;
}
```

Explanation:

* Drift creates the database using schema version `1`.
* No custom migration logic is needed until the schema changes.

---

## Real-World Example

Suppose version 2 of your app adds a new table.

```dart
@override
int get schemaVersion => 2;

@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 2) {
      await m.createTable(tasks);
    }
  },
);
```

Explanation:

* Existing users are upgraded from version 1 to version 2.
* Only the new table is created, preserving existing data.

---

# When to Use

Configure your database when:

* Creating a new Drift database.
* Changing your schema.
* Adding migrations.
* Initializing database data.
* Handling upgrades between app versions.

---

# When NOT to Use

Database configuration should not be used for:

* Business logic.
* CRUD operations.
* Complex queries.
* Application initialization unrelated to the database.

Keep configuration focused on database behavior.

---

# Best Practices

* Start with `schemaVersion` set to `1`.
* Increment the schema version whenever the schema changes.
* Keep migration logic small and well-tested.
* Never modify an existing migration after release.
* Test migrations before publishing updates.

---

# Common Mistakes

## Forgetting to increase the schema version

**Wrong**

Adding a new column without changing:

```dart
@override
int get schemaVersion => 1;
```

Explanation:

* Drift won't detect that the schema has changed.

**Correct**

```dart
@override
int get schemaVersion => 2;
```

Explanation:

* Increasing the schema version tells Drift that a migration is required.

---

## Putting business logic in migrations

**Wrong**

```dart
onUpgrade: (m, from, to) async {
  await sendAnalytics();
}
```

Explanation:

* Migrations should only modify the database schema or data.

**Correct**

Keep migrations focused on database changes only.

---

## Recreating tables during upgrades

**Wrong**

```dart
await m.createAll();
```

inside every upgrade.

Explanation:

* Recreating existing tables can cause migration failures or data loss.

**Correct**

Only apply the specific schema changes needed for that version.

---

# Related APIs

* Schema Versions
* Migration Strategy
* Database Lifecycle
* Opening the Database
* GeneratedDatabase
* Testing Migrations

---

# Summary

Database configuration defines how Drift initializes and manages your database. It includes the schema version, migration strategy, and lifecycle hooks that keep your database consistent as your application evolves. A well-configured database makes upgrades safer and ensures long-term maintainability.
