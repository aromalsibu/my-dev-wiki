# Batch Operations

Batch operations group multiple database writes into a single efficient operation.

---

# What is it?

A batch operation allows you to execute multiple inserts, updates, deletes, or combinations of these operations together.

Instead of sending each operation to SQLite individually, Drift collects them into a single batch, reducing database overhead.

Batch operations are primarily a performance optimization.

---

# Why does it exist?

Suppose you need to:

* Insert 100 users.
* Update 50 products.
* Delete 20 old notifications.

Without batching:

```text
Insert
Insert
Insert
Update
Update
Delete
...
```

Each operation requires communication with the database.

With batching:

```text
Batch
├── Insert
├── Insert
├── Update
├── Delete
└── ...
```

Drift sends them together, resulting in fewer database round trips and improved performance.

---

# Syntax

## Creating a Batch

```dart
await batch((batch) {
  batch.insert(
    users,
    UsersCompanion.insert(
      name: 'Alice',
      age: 25,
    ),
  );

  batch.insert(
    users,
    UsersCompanion.insert(
      name: 'Bob',
      age: 30,
    ),
  );
});
```

Explanation:

* `batch()` creates a batch operation.
* `batch.insert()` queues insert operations.
* All queued operations are executed together.

---

## Mixing Different Operations

```dart
await batch((batch) {
  batch.insert(
    users,
    UsersCompanion.insert(
      name: 'Charlie',
      age: 28,
    ),
  );

  batch.update(
    products,
    ProductsCompanion(
      price: const Value(899.99),
    ),
    where: (p) => p.id.equals(1),
  );

  batch.deleteWhere(
    tasks,
    (t) => t.completed.equals(true),
  );
});
```

Explanation:

* A single batch can contain inserts, updates, and deletes.
* Operations are queued and executed efficiently.

---

# Mental Model

Think of batch operations as preparing several tasks before sending them to the database.

```text
Without Batch

Task 1 → Database
Task 2 → Database
Task 3 → Database

--------------

With Batch

Tasks
│
├── Insert
├── Update
├── Delete
│
▼

Database
```

Instead of making multiple trips, everything is sent together.

---

# Examples

## Simple Example

Insert multiple categories.

```dart
await batch((batch) {
  batch.insertAll(
    categories,
    [
      CategoriesCompanion.insert(name: 'Books'),
      CategoriesCompanion.insert(name: 'Electronics'),
      CategoriesCompanion.insert(name: 'Clothing'),
    ],
  );
});
```

Explanation:

* `insertAll()` inserts multiple rows in one operation.
* This is more efficient than separate inserts.

---

## Real-World Example

Synchronizing server data.

```dart
await batch((batch) {
  batch.insertAll(users, usersToInsert);

  batch.insertAll(products, productsToInsert);

  batch.deleteWhere(
    notifications,
    (n) => n.isRead.equals(true),
  );
});
```

Explanation:

* Multiple related operations are executed together.
* This improves synchronization performance.

---

# When to Use

Use batch operations when:

* Importing large datasets.
* Synchronizing data.
* Performing many inserts.
* Performing many updates.
* Performing many deletes.
* Executing several related write operations together.

---

# When NOT to Use

Avoid batch operations when:

* Only one database operation is needed.
* Each operation depends on the result of the previous one.
* Individual error handling is required between operations.

For a single write, normal CRUD APIs are usually simpler.

---

# Best Practices

* Group related operations into a single batch.
* Use `insertAll()` for multiple inserts.
* Keep batches reasonably sized.
* Use batches to improve performance, not simply to reduce lines of code.
* Consider transactions when atomicity is required.

---

# Common Mistakes

## Using a batch for one operation

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

* A batch adds unnecessary complexity for a single insert.

**Correct**

```dart
await into(users).insert(
  UsersCompanion.insert(
    name: 'Alice',
    age: 25,
  ),
);
```

Explanation:

* A normal insert is simpler.

---

## Executing many separate writes

**Wrong**

```dart
for (final user in usersList) {
  await into(users).insert(user);
}
```

Explanation:

* Every iteration performs a separate database operation.

**Correct**

```dart
await batch((batch) {
  batch.insertAll(users, usersList);
});
```

Explanation:

* All inserts are grouped into one efficient batch.

---

## Confusing batches with transactions

**Wrong**

Assuming a batch guarantees that all operations succeed or fail together.

Explanation:

* A batch improves efficiency.
* It does not automatically provide transactional guarantees.

**Correct**

Use a transaction when all operations must either succeed together or roll back together.

---

# Related APIs

* Batch Insert
* Insert
* Update
* Delete
* Transactions

---

# Summary

Batch operations allow multiple database writes to be grouped into a single efficient operation. They reduce communication with the database, improve performance, and are ideal for bulk inserts, updates, deletes, and synchronization tasks. When atomicity is required, combine or replace batch operations with transactions.
