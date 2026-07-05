# Testing Migrations

**Migration testing** verifies that your database can be upgraded from an older schema version to the latest one without losing data or producing an invalid database.

---

# What is it?

Whenever you change your database schema, you should test that existing users can upgrade successfully.

A migration test typically performs four steps:

1. Create a database with an old schema.
2. Insert sample data.
3. Run the migration.
4. Verify both the data and the new schema.

Unlike testing normal CRUD operations, migration testing focuses on the upgrade process itself.

---

# Why does it exist?

A migration may work perfectly on a fresh database but fail for existing users.

Common migration issues include:

* Missing columns
* Lost data
* Incorrect default values
* Broken foreign keys
* Failed startup after upgrading

Migration tests catch these problems before your users do.

---

# Typical Migration Test Flow

```text
Create Old Database
        │
        ▼
Insert Test Data
        │
        ▼
Run Migration
        │
        ▼
Verify Schema
        │
        ▼
Verify Existing Data
        │
        ▼
Verify New Features
```

A migration isn't successful until **both the schema and the existing data are correct**.

---

# Examples

## Example 1: Verify Existing Data After a Migration

Suppose version 2 adds an `email` column to the `Users` table.

```dart
test('migration preserves existing users', () async {
  final database = AppDatabase(NativeDatabase.memory());

  await database.customStatement('''
    CREATE TABLE users (
      id INTEGER PRIMARY KEY,
      name TEXT NOT NULL
    );
  ''');

  await database.customStatement("""
    INSERT INTO users (name)
    VALUES ('Alice');
  """);

  await database.migration.onUpgrade!(
    database.migrator,
    1,
    2,
  );

  final result = await database.customSelect(
    'SELECT name FROM users',
  ).getSingle();

  expect(result.data['name'], 'Alice');
});
```

Explanation:

* Create a database using the old schema.
* Insert representative data.
* Run the migration.
* Verify that the existing row still exists after the upgrade.

---

## Example 2: Verify a Newly Added Column

Suppose the migration adds an `email` column.

```dart
test('migration adds email column', () async {
  final database = AppDatabase(NativeDatabase.memory());

  await database.customStatement('''
    CREATE TABLE users (
      id INTEGER PRIMARY KEY,
      name TEXT NOT NULL
    );
  ''');

  await database.migration.onUpgrade!(
    database.migrator,
    1,
    2,
  );

  final columns = await database.customSelect(
    "PRAGMA table_info(users);",
  ).get();

  expect(
    columns.any((column) => column.data['name'] == 'email'),
    isTrue,
  );
});
```

Explanation:

* `PRAGMA table_info()` returns metadata about a table.
* The test verifies that the migration actually created the new column.

---

## Example 3: Verify Default Values

Suppose a migration introduces a required `status` column with a default value.

```dart
test('existing rows receive default status', () async {
  final database = AppDatabase(NativeDatabase.memory());

  await database.customStatement('''
    CREATE TABLE users (
      id INTEGER PRIMARY KEY,
      name TEXT NOT NULL
    );
  ''');

  await database.customStatement("""
    INSERT INTO users (name)
    VALUES ('John');
  """);

  await database.migration.onUpgrade!(
    database.migrator,
    1,
    2,
  );

  final row = await database.customSelect(
    '''
    SELECT status
    FROM users
    WHERE name = 'John'
    ''',
  ).getSingle();

  expect(row.data['status'], 'active');
});
```

Explanation:

* Existing rows should receive the transformed value defined during the migration.
* Verify that the migration populated the new column correctly.

---

## Real-World Example

A production migration often needs to preserve data while introducing new schema elements.

```dart
test('database upgrades safely', () async {
  final database = AppDatabase(NativeDatabase.memory());

  await database.customStatement('''
    CREATE TABLE tasks (
      id INTEGER PRIMARY KEY,
      title TEXT NOT NULL
    );
  ''');

  await database.customStatement("""
    INSERT INTO tasks(title)
    VALUES ('Finish Drift Handbook');
  """);

  await database.migration.onUpgrade!(
    database.migrator,
    1,
    2,
  );

  final task = await database.customSelect(
    '''
    SELECT title
    FROM tasks
    ''',
  ).getSingle();

  expect(task.data['title'], 'Finish Drift Handbook');

  final schema = await database.customSelect(
    "PRAGMA table_info(tasks);",
  ).get();

  expect(
    schema.any((column) => column.data['name'] == 'due_date'),
    isTrue,
  );
});
```

Explanation:

* Existing task data survives the migration.
* The new schema is available after upgrading.
* Both the data and the structure are verified.

---

# What Should You Test?

Every migration test should verify:

* Existing rows are preserved.
* New columns exist.
* New tables are created.
* Renamed columns contain the correct data.
* Default values are applied correctly.
* Foreign keys still work.
* Indexes are recreated if needed.
* Queries continue to work.

---

# When to Use

Write migration tests whenever you:

* Add a table.
* Remove a table.
* Add a column.
* Rename a column.
* Change a column type.
* Use `TableMigration`.
* Perform data transformations.

In short, **every released migration should have a corresponding test**.

---

# When NOT to Use

Migration tests aren't necessary when:

* Your database schema hasn't changed.
* You're still prototyping and haven't released the app.
* The database is recreated every time the app starts.

Once real users depend on your database, migration tests become essential.

---

# Best Practices

* Test every released migration.
* Insert realistic sample data before migrating.
* Verify both the schema and the data.
* Test upgrades from every supported version.
* Keep migration tests in your automated test suite.
* Never assume a migration works just because the app starts.

---

# Common Mistakes

## Testing Only an Empty Database

**Wrong**

```dart
test('migration works', () async {
  // Upgrade an empty database.
});
```

An empty database doesn't verify whether existing user data survives.

**Correct**

```dart
await database.customStatement("""
  INSERT INTO users(name)
  VALUES ('Alice');
""");
```

Always migrate databases that contain representative data.

---

## Only Checking That the Migration Completes

**Wrong**

```dart
await database.migration.onUpgrade!(
  database.migrator,
  1,
  2,
);
```

The migration may complete while silently losing data.

**Correct**

```dart
final users = await database.customSelect(
  'SELECT * FROM users',
).get();

expect(users.length, 1);
```

Always verify the migrated data.

---

## Forgetting to Verify the New Schema

**Wrong**

```dart
expect(users.single.data['name'], 'Alice');
```

The data survived, but the new schema might still be incorrect.

**Correct**

```dart
final columns = await database.customSelect(
  'PRAGMA table_info(users);',
).get();

expect(
  columns.any((c) => c.data['name'] == 'email'),
  isTrue,
);
```

Verify both the schema and the preserved data.

---

# Related APIs

* Schema Versions
* MigrationStrategy
* Migrator
* TableMigration
* In-Memory Database

---

# Summary

Migration testing ensures that users can safely upgrade to newer versions of your application. A good migration test creates an old database, inserts representative data, runs the migration, and verifies both the updated schema and the preserved data. By testing every released migration, you can confidently evolve your database without risking data loss or broken upgrades.
