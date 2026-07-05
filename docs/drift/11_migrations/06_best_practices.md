# Best Practices

**Migration best practices** help you evolve your database safely while preserving user data and keeping migrations maintainable.

---

# What is it?

Database migrations are permanent. Once a version of your app is released, users may continue upgrading from that version for months or even years.

Because of this, migration code should be treated as production code—not temporary code that can be rewritten later.

Following a few best practices helps prevent data loss, failed upgrades, and difficult-to-maintain migration logic.

---

# Why does it exist?

A migration runs only once for each user, but if it fails, the consequences can be severe:

* User data may be lost.
* The database may become corrupted.
* The application may fail to start.
* Recovery can be difficult or impossible.

These best practices reduce those risks and make future schema changes easier to implement.

---

# Mental Model

Think of migrations as a staircase.

```text
Version 1
    │
    ▼
Version 2
    │
    ▼
Version 3
    │
    ▼
Version 4
```

Each step should be:

* Small
* Independent
* Tested
* Safe

Avoid taking giant leaps that combine many unrelated changes.

---

# Best Practices

## 1. Increment the Schema Version for Every Schema Change

Every structural database change should have its own schema version.

**Good**

```dart
@override
int get schemaVersion => 3;
```

Explanation:

* Drift detects schema changes by comparing version numbers.
* Forgetting to increment the version prevents migrations from running.

---

## 2. Keep One Migration Per Version

Instead of writing one large migration, write one block for each version.

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
      await m.createIndex(userEmailIndex);
    }
  },
);
```

Explanation:

* Each migration corresponds to a single schema version.
* Users upgrading from any previous version receive only the required changes.

---

## 3. Prefer the Simplest Migration API

Use the migration method that matches your schema change.

```dart
await m.addColumn(users, users.email);
```

Explanation:

* `addColumn()` is simpler than rebuilding the table.
* Reserve `TableMigration` for changes SQLite can't perform directly.

---

## 4. Preserve Existing Data

Whenever possible, modify the schema without deleting user data.

**Good**

```dart
await m.alterTable(
  TableMigration(users),
);
```

Explanation:

* Drift rebuilds the table while copying existing rows.
* User data remains intact.

Avoid deleting and recreating tables unless the data is disposable.

---

## 5. Keep Migrations Independent

Each migration should depend only on the current database version.

```dart
if (from < 5) {
  await m.createTable(settings);
}
```

Explanation:

* Users can upgrade from any supported version.
* Migrations remain predictable and easier to maintain.

---

## 6. Test Every Migration

Every released migration should have a corresponding test.

```dart
test(
  'migrates schema from version 2 to 3',
  () async {
    // Migration test
  },
);
```

Explanation:

* Verify both the updated schema and the preserved data.
* Catch migration issues before releasing your application.

---

## 7. Never Modify Released Migrations

Suppose version 2 has already been published.

**Don't** rewrite the version 2 migration.

Instead:

```text
Version 2
    │
    ▼
Version 3
```

Add a new migration for version 3.

Explanation:

* Existing users have already executed the version 2 migration.
* Changing it later can produce inconsistent databases.

---

## 8. Use Transactions for Complex Migrations

When multiple schema or data updates must succeed together, execute them inside a transaction.

```dart
onUpgrade: (m, from, to) async {
  await transaction(() async {
    await m.createTable(categories);
    await m.addColumn(tasks, tasks.categoryId);
  });
}
```

Explanation:

* Either all changes succeed, or none of them are applied.
* This prevents partially migrated databases.

---

# When to Avoid Destructive Migrations

Avoid deleting and recreating tables if they contain:

* User accounts
* Notes
* Messages
* Orders
* Offline-created data
* Application settings

Destructive migrations are best reserved for cache or temporary data that can be recreated.

---

# Common Mistakes

## Combining Multiple Versions Into One Migration

**Wrong**

```dart
onUpgrade: (m, from, to) async {
  await m.addColumn(users, users.email);
  await m.createTable(categories);
  await m.createIndex(userEmailIndex);
}
```

This ignores which version the user is upgrading from.

**Correct**

```dart
onUpgrade: (m, from, to) async {
  if (from < 2) {
    await m.addColumn(users, users.email);
  }

  if (from < 3) {
    await m.createTable(categories);
  }

  if (from < 4) {
    await m.createIndex(userEmailIndex);
  }
}
```

---

## Rewriting Old Migrations

**Wrong**

```text
Release v1.0

↓

Edit the version 2 migration later
```

Users who already upgraded have a different database state than new users.

**Correct**

Always create a new migration for each new schema version.

---

## Skipping Migration Tests

**Wrong**

Release the application without verifying the upgrade path.

A migration that works on a fresh database may fail for existing users.

**Correct**

Test every migration before publishing a new release.

---

# Related APIs

* Schema Versions
* MigrationStrategy
* Migrator
* TableMigration
* Testing Migrations

---

# Summary

Well-designed migrations are small, incremental, and thoroughly tested. Keep one migration per schema version, use the simplest migration API available, preserve existing data whenever possible, and never modify migrations that have already been released. Following these practices ensures users can upgrade safely as your application's database evolves.
