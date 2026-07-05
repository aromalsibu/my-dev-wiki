# Destructive Migrations

A **destructive migration** replaces an existing database with a new one by deleting some or all of its schema and data instead of preserving it.

---

# What is it?

Normally, a migration updates the existing database while keeping the user's data intact.

A destructive migration takes the opposite approach:

1. Remove existing tables or the entire database.
2. Recreate the latest schema.
3. Start with an empty database.

This approach is much simpler than writing complex migrations, but **all existing data is permanently lost**.

For most production applications, destructive migrations should be avoided.

---

# Why does it exist?

Sometimes the data stored in a database isn't valuable because it can be recreated.

Examples include:

* API cache
* Search indexes
* Temporary offline data
* Session data
* Downloaded content

In these cases, deleting and recreating the database is often easier than maintaining complex migration logic.

---

# Syntax

A destructive migration is usually implemented inside `onUpgrade()`.

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    await m.deleteTable('cached_products');
    await m.createTable(cachedProducts);
  },
);
```

Explanation:

* `deleteTable()` removes both the table and its data.
* `createTable()` recreates the table using the latest schema.

---

# Mental Model

Normal migration:

```text
Old Database
      │
      ▼
Update Schema
      │
      ▼
Existing Data Preserved
```

Destructive migration:

```text
Old Database
      │
      ▼
Delete Everything
      │
      ▼
Create New Database
      │
      ▼
Empty Database
```

Instead of transforming existing data, you simply start over.

---

# Examples

## Example 1: Recreate a Cache Table

Suppose your application caches products downloaded from an API.

```dart
class CachedProducts extends Table {
  IntColumn get id => integer()();
  TextColumn get name => text()();

  @override
  Set<Column> get primaryKey => {id};
}
```

Migration:

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 2) {
      await m.deleteTable('cached_products');
      await m.createTable(cachedProducts);
    }
  },
);
```

Explanation:

* Existing cache data is removed.
* The table is recreated using the latest schema.
* Fresh data will be downloaded from the server.

---

## Example 2: Rebuild Multiple Cache Tables

Sometimes multiple cache tables can safely be recreated.

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 3) {
      await m.deleteTable('cached_products');
      await m.deleteTable('cached_categories');
      await m.deleteTable('cached_images');

      await m.createTable(cachedProducts);
      await m.createTable(cachedCategories);
      await m.createTable(cachedImages);
    }
  },
);
```

Explanation:

* Every cache table is removed.
* Each table is recreated with the latest schema.
* The next synchronization repopulates the database.

---

## Real-World Example

Suppose an application stores only weather forecasts downloaded from an API.

```dart
class WeatherForecasts extends Table {
  IntColumn get id => integer()();
  TextColumn get city => text()();
  TextColumn get forecast => text()();

  @override
  Set<Column> get primaryKey => {id};
}
```

A new release completely redesigns the table structure.

Instead of migrating thousands of cached rows:

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
  onUpgrade: (m, from, to) async {
    if (from < 2) {
      await m.deleteTable('weather_forecasts');
      await m.createTable(weatherForecasts);
    }
  },
);
```

Explanation:

* Cached forecasts are discarded.
* The new schema is created immediately.
* Fresh forecasts are downloaded after the application starts.

This keeps the migration simple because no user-generated data is involved.

---

# When to Use

Use destructive migrations when:

* The database stores only cache data.
* Data can be downloaded again.
* Data is temporary.
* You're still in early development.
* Resetting the database is acceptable.

---

# When NOT to Use

Do **not** use destructive migrations for databases containing:

* User accounts
* Notes
* Messages
* Orders
* Shopping carts
* Offline-created content
* User preferences
* Financial records

Users expect this data to survive application updates.

---

# Best Practices

* Prefer preserving user data whenever possible.
* Clearly separate cache tables from user data.
* Use destructive migrations only when data is recoverable.
* Keep users informed if important local data will be removed.
* Test recreated tables after the migration.

---

# Common Mistakes

## Using Destructive Migrations for User Data

**Wrong**

```dart
await m.deleteTable('users');
await m.createTable(users);
```

Every user account stored locally is permanently deleted.

**Correct**

```dart
await m.addColumn(
  users,
  users.email,
);
```

Preserve user data whenever possible.

---

## Forgetting to Recreate the Table

**Wrong**

```dart
await m.deleteTable('cached_products');
```

The table is removed but never recreated.

**Correct**

```dart
await m.deleteTable('cached_products');
await m.createTable(cachedProducts);
```

Always recreate any table that the application expects to exist.

---

## Using Destructive Migrations Because They're Easier

**Wrong**

```dart
onUpgrade: (m, from, to) async {
  await m.deleteTable('orders');
  await m.createTable(orders);
}
```

This discards valuable user data just to avoid writing a migration.

**Correct**

```dart
onUpgrade: (m, from, to) async {
  await m.addColumn(
    orders,
    orders.status,
  );
}
```

Choose schema migrations whenever user data should be preserved.

---

# Related APIs

* MigrationStrategy
* Migrator
* TableMigration
* Schema Versions
* Testing Migrations

---

# Summary

Destructive migrations recreate part or all of a database instead of updating it in place. They are appropriate for cache or temporary data that can be regenerated, but they should be avoided for databases containing valuable user information. In most production applications, preserving existing data through proper migrations is the preferred approach.
