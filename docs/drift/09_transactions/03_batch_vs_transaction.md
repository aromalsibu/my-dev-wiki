# Batch vs Transaction

**Batches** and **transactions** both execute multiple database operations, but they solve different problems. Batches optimize performance, while transactions guarantee atomicity.

---

# What is it?

At first glance, batches and transactions seem similar because both involve multiple database operations.

However, their purposes are different:

* A **batch** groups operations together to reduce communication with the database and improve performance.
* A **transaction** groups operations together to ensure they either all succeed or all fail.

Although Drift executes a batch inside a transaction internally, you should choose between them based on **why** you're grouping the operations.

---

# Why does it exist?

Suppose you need to insert 10,000 products into your database.

Doing this one insert at a time is slow because each insert requires separate communication with SQLite.

A batch solves this by grouping the inserts together.

Now consider transferring money between two accounts.

Here, speed isn't the priority—data consistency is.

If one operation fails, both operations must be undone.

A transaction is the correct choice.

---

# Syntax

## Batch

```dart
await batch((batch) {
  batch.insertAll(users, [
    UsersCompanion.insert(name: 'Alice', age: 25),
    UsersCompanion.insert(name: 'Bob', age: 30),
  ]);
});
```

Explanation:

* `batch()` groups multiple operations together.
* SQLite executes them efficiently.
* Ideal for bulk inserts, updates, and deletes.

---

## Transaction

```dart
await transaction(() async {
  await createUser();
  await createProfile();
});
```

Explanation:

* `transaction()` guarantees that both operations succeed or both fail.
* Used when maintaining data consistency is more important than raw performance.

---

# Mental Model

Think of a batch as a delivery truck.

Instead of making many trips:

```text
Package
Package
Package
Package
```

Everything is delivered together.

```text
Truck
 ├── Package
 ├── Package
 ├── Package
 └── Package
```

This reduces overhead.

---

Think of a transaction as a contract.

```text
Step 1 ✔
Step 2 ✔
Step 3 ✔

Commit
```

If one step fails:

```text
Step 1 ✔
Step 2 ✖

Rollback Everything
```

Nothing is permanently saved.

---

# Examples

## Simple Example

### Batch Insert

```dart
await batch((batch) {
  batch.insertAll(tasks, [
    TasksCompanion.insert(title: 'Task 1'),
    TasksCompanion.insert(title: 'Task 2'),
    TasksCompanion.insert(title: 'Task 3'),
  ]);
});
```

Explanation:

* All inserts are sent together.
* Faster than calling `insert()` three separate times.

---

### Transaction

```dart
await transaction(() async {
  await into(users).insert(
    UsersCompanion.insert(
      name: 'John',
      age: 30,
    ),
  );

  await into(profiles).insert(
    ProfilesCompanion.insert(
      userId: 1,
      bio: 'Developer',
    ),
  );
});
```

Explanation:

* Both records represent a single business operation.
* If either insert fails, neither record is saved.

---

## Real-World Example

### Batch

Import thousands of products from a CSV file.

```dart
await batch((batch) {
  batch.insertAll(products, importedProducts);
});
```

Explanation:

* Performance is the primary concern.
* Batch processing significantly reduces database overhead.

---

### Transaction

Process a customer order.

```dart
await transaction(() async {
  await createOrder();

  await reserveInventory();

  await createInvoice();

  await recordPayment();
});
```

Explanation:

* Every step depends on the previous one.
* A failure anywhere rolls back the entire order.

---

# When to Use

## Use a Batch when:

* Importing large datasets.
* Bulk inserting records.
* Bulk updating records.
* Bulk deleting records.
* Performance is the primary goal.

---

## Use a Transaction when:

* Multiple operations form one business process.
* Maintaining data consistency.
* Updating multiple related tables.
* Preventing partial writes.
* Financial or inventory operations.

---

# When NOT to Use

Don't use a batch:

* For unrelated operations that must succeed together.
* When rollback behavior is your primary concern.

Don't use a transaction:

* Simply to improve performance.
* For large bulk imports where batching is more appropriate.

Choose the tool based on your goal.

---

# Best Practices

* Use batches for bulk operations.
* Use transactions for business logic.
* Keep transactions short.
* Avoid unnecessary batches for small operations.
* Avoid wrapping every batch inside another transaction unless required.
* Design your code around intent rather than implementation details.

---

# Common Mistakes

## Using Transactions for Bulk Imports

Wrong:

```dart
await transaction(() async {
  for (final product in products) {
    await into(productsTable).insert(product);
  }
});
```

Explanation:

* This works, but performs many individual insert operations.

Correct:

```dart
await batch((batch) {
  batch.insertAll(productsTable, products);
});
```

Explanation:

* Batch operations are optimized for bulk data.

---

## Using a Batch for Business Logic

Wrong:

```dart
await batch((batch) {
  batch.insert(...);
  batch.update(...);
  batch.delete(...);
});
```

Explanation:

* A batch expresses bulk database work, not a business workflow.

Correct:

```dart
await transaction(() async {
  await createOrder();

  await updateInventory();

  await createInvoice();
});
```

Explanation:

* Transactions better represent operations that must succeed or fail together.

---

## Choosing Based Only on Performance

Wrong assumption:

> "Batch and transaction are interchangeable."

Explanation:

* They solve different problems.

Correct understanding:

* Use **batch** for efficiency.
* Use **transaction** for consistency.

---

# Related APIs

* `batch()`
* `transaction()`
* Batch Insert
* Batch Operations
* Rollback Behavior
* Insert
* Update
* Delete

---

# Summary

Batches and transactions both group multiple database operations, but they have different purposes. Use **batches** to improve performance during bulk operations, and use **transactions** to ensure multiple related operations succeed or fail as a single unit. Choosing the right tool depends on whether your priority is **efficiency** or **data consistency**.
