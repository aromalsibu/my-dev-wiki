# Upsert

Upsert inserts a new row if it doesn't exist, or updates the existing row if it does.

---

# What is it?

An upsert combines two operations into one:

* **Insert** if the row doesn't exist.
* **Update** if the row already exists.

Instead of checking whether a record exists before deciding between `insert()` and `update()`, Drift lets SQLite handle that logic in a single operation.

---

# Why does it exist?

Consider synchronizing users from a server.

For each user, you don't know whether:

* It's a new user that should be inserted.
* It's an existing user that should be updated.

Without upsert, you'd typically write:

```text
Does the row exist?
        │
   Yes ─┴─ No
    │        │
 Update   Insert
```

This requires an additional query and more application logic.

Upsert performs both operations in a single database command.

---

# Syntax

## Basic Upsert

```dart
await into(users).insert(
  UsersCompanion.insert(
    id: const Value(1),
    name: 'Alice',
    age: 26,
  ),
  mode: InsertMode.insertOrReplace,
);
```

Explanation:

* `mode` controls how conflicts are handled.
* `InsertMode.insertOrReplace` replaces an existing row when a primary key or unique constraint conflict occurs.
* If no conflict exists, a new row is inserted.

> **Note:** Drift also provides more advanced conflict handling APIs such as `onConflict`, which are covered in advanced sections.

---

# Mental Model

Think of upsert as "save."

```text
Database
    │
    ▼

Does row exist?

      │
 ┌────┴────┐
 │         │
No        Yes
 │         │
 ▼         ▼
Insert   Update
```

Your application simply asks the database to save the row, and SQLite decides whether to insert or update.

---

# Examples

## Simple Example

Save a product.

```dart
await into(products).insert(
  ProductsCompanion.insert(
    id: const Value(5),
    name: 'Laptop',
    price: 899.99,
  ),
  mode: InsertMode.insertOrReplace,
);
```

Explanation:

* If product `5` doesn't exist, it is inserted.
* If it already exists, it is replaced with the new values.

---

## Real-World Example

Synchronizing data from a remote API.

```dart
for (final user in serverUsers) {
  await into(users).insert(
    user,
    mode: InsertMode.insertOrReplace,
  );
}
```

Explanation:

* New users are inserted.
* Existing users are updated automatically.
* No existence check is needed before saving.

---

# When to Use

Use upsert when:

* Synchronizing remote data.
* Importing records.
* Caching API responses.
* Saving data with known primary keys.
* You don't know whether the row already exists.

---

# When NOT to Use

Avoid upsert when:

* You only want to insert new rows.
* You only want to update existing rows.
* Replacing an entire row could unintentionally overwrite data.

In those situations, use `insert()` or `update()` directly.

---

# Best Practices

* Ensure the table has a primary key or unique constraint.
* Use upsert for synchronization and caching.
* Understand how your chosen conflict strategy behaves.
* Be cautious when replacing rows containing important data.
* Test conflict scenarios during development.

---

# Common Mistakes

## Expecting upsert without a unique constraint

**Wrong**

Using upsert on a table without a primary key or unique constraint.

Explanation:

* SQLite needs a conflict target to determine whether a row already exists.

**Correct**

Define a primary key or unique constraint before using upsert.

---

## Using upsert for partial updates

**Wrong**

Using an upsert when you only want to update one field.

Explanation:

* Conflict strategies like `insertOrReplace` replace the entire row.
* Existing values may be overwritten.

**Correct**

Use `update()` with a companion for partial updates.

---

## Confusing `insertOrReplace` with `replace()`

**Wrong**

Assuming both APIs behave the same.

Explanation:

* `replace()` updates an existing row identified by its primary key.
* `insertOrReplace` first attempts an insert and only replaces the row if a conflict occurs.

**Correct**

Choose the API based on whether you're explicitly updating an existing row or performing an insert-or-update operation.

---

# Related APIs

* Insert
* Replace
* Update
* Batch Insert
* Primary Keys

---

# Summary

Upsert combines insert and update into a single operation. It allows SQLite to insert new rows or resolve conflicts by updating existing ones, making it ideal for synchronization, caching, and import workflows. It requires a primary key or unique constraint so SQLite can detect when a row already exists.
