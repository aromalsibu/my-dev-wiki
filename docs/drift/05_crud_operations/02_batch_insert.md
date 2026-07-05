# Batch Insert

Batch insert adds multiple rows to a table in a single operation.

---

# What is it?

A batch insert allows you to insert many records at once instead of performing separate insert operations for each row.

Rather than sending multiple database operations, Drift groups them into a single batch, making inserts much faster and more efficient.

---

# Why does it exist?

Imagine importing 1,000 contacts.

Without batching:

```text
Insert Contact 1
Insert Contact 2
Insert Contact 3
...
Insert Contact 1000
```

This requires 1,000 separate database operations.

With batch inserts:

```text
Batch
├── Contact 1
├── Contact 2
├── Contact 3
...
└── Contact 1000
```

The database processes them together, reducing overhead and improving performance.

---

# Syntax

## Basic Batch Insert

```dart
await batch((batch) {
  batch.insertAll(
    users,
    [
      UsersCompanion.insert(
        name: 'Alice',
        age: 25,
      ),
      UsersCompanion.insert(
        name: 'Bob',
        age: 30,
      ),
    ],
  );
});
```

Explanation:

* `batch()` creates a batch operation.
* `insertAll()` inserts multiple rows into the table.
* Each item in the list is a companion object representing one row.

---

## Batch Insert with Multiple Tables

```dart
await batch((batch) {
  batch.insertAll(users, usersList);

  batch.insertAll(products, productsList);
});
```

Explanation:

* A single batch can contain operations for multiple tables.
* Drift executes them efficiently as one grouped operation.

---

# Mental Model

Think of mailing letters.

Without batching:

```text
Post Office

Trip 1 → One letter
Trip 2 → One letter
Trip 3 → One letter
```

With batching:

```text
One Trip

📦
├── Letter 1
├── Letter 2
├── Letter 3
└── Letter 4
```

Instead of making many small trips, you deliver everything together.

---

# Examples

## Simple Example

Insert multiple tasks.

```dart
await batch((batch) {
  batch.insertAll(
    tasks,
    [
      TasksCompanion.insert(title: 'Buy milk'),
      TasksCompanion.insert(title: 'Read book'),
      TasksCompanion.insert(title: 'Exercise'),
    ],
  );
});
```

Explanation:

* Three tasks are inserted in one database operation.
* This is more efficient than three separate inserts.

---

## Real-World Example

Importing products from an API.

```dart
await batch((batch) {
  batch.insertAll(
    products,
    productCompanions,
  );
});
```

Explanation:

* `productCompanions` contains many products.
* Drift inserts them together, making bulk imports much faster.

---

# When to Use

Use batch inserts when:

* Importing large datasets.
* Syncing data from a server.
* Restoring backups.
* Seeding a database.
* Inserting many records at once.

---

# When NOT to Use

Avoid batch inserts when:

* Inserting only one or two rows.
* Each insert depends on the result of the previous insert.
* Individual error handling is required for each insert.

For small numbers of inserts, a normal `insert()` is often simpler.

---

# Best Practices

* Batch as many related inserts as practical.
* Use companion classes for batch inserts.
* Group operations that belong together.
* Use batches for imports and synchronization tasks.
* Avoid creating unnecessarily large batches if memory usage becomes a concern.

---

# Common Mistakes

## Repeating individual inserts

**Wrong**

```dart
for (final user in usersList) {
  await into(users).insert(user);
}
```

Explanation:

* Each iteration performs a separate database operation.
* This is slower for large collections.

**Correct**

```dart
await batch((batch) {
  batch.insertAll(users, usersList);
});
```

Explanation:

* All rows are inserted together.

---

## Using batches for a single insert

**Wrong**

```dart
await batch((batch) {
  batch.insert(
    users,
    UsersCompanion.insert(
      name: 'Alice',
      age: 25,
    ),
  );
});
```

Explanation:

* A batch adds unnecessary complexity for a single operation.

**Correct**

Use `into(users).insert(...)` for individual inserts.

---

## Mixing unrelated operations unnecessarily

**Wrong**

Adding unrelated database operations to the same batch simply because they occur close together in code.

Explanation:

* Batches should group operations that naturally belong together.
* Overusing batches can make code harder to understand.

**Correct**

Use batches for logically related bulk operations.

---

# Related APIs

* Insert
* Batch Operations
* Transactions
* Update
* Delete

---

# Summary

Batch insert allows multiple rows to be inserted efficiently in a single database operation. It reduces database overhead, improves performance, and is ideal for bulk imports, synchronization, and large datasets. For individual records, a normal insert is usually the simpler choice.
