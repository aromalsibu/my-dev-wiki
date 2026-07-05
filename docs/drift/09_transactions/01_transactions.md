# Transactions

**A transaction** is a group of database operations that are executed as a single unit, ensuring that either all operations succeed or none of them are applied.

---

# What is it?

A transaction lets you combine multiple database operations into one atomic operation.

Within a transaction, you can perform inserts, updates, deletes, or any other queries. If every operation succeeds, Drift commits the transaction and permanently saves the changes.

If any operation fails, Drift rolls back the transaction, restoring the database to the state it was in before the transaction started.

This guarantees that your database never ends up in a partially updated state.

---

# Why does it exist?

Consider transferring money between two bank accounts.

The process involves two operations:

1. Subtract money from Account A.
2. Add money to Account B.

If the app crashes after the first operation but before the second, money disappears.

Without a transaction:

```text
Withdraw ✔
Deposit ✖

Database becomes inconsistent.
```

With a transaction:

```text
Withdraw ✔
Deposit ✖
Rollback ✔

Database returns to its original state.
```

Transactions ensure that related operations either complete together or not at all.

---

# Syntax

Execute multiple operations in a transaction.

```dart
await transaction(() async {
  await into(users).insert(
    UsersCompanion.insert(
      name: 'Alice',
      age: 25,
    ),
  );

  await into(profiles).insert(
    ProfilesCompanion.insert(
      userId: 1,
      bio: 'Flutter Developer',
    ),
  );
});
```

Explanation:

* `transaction()` starts a new database transaction.
* If both inserts succeed, the transaction is committed.
* If either insert fails, both changes are rolled back.

---

A transaction can contain any number of database operations.

```dart
await transaction(() async {
  await updateUser();
  await createProfile();
  await deleteOldData();
});
```

Explanation:

* Every operation runs within the same transaction.
* All operations succeed together or fail together.

---

# Mental Model

Think of a transaction as a temporary workspace.

```text
Database
    │
    ▼
Start Transaction
    │
    ▼
Operation 1
Operation 2
Operation 3
    │
    ▼
Everything succeeds?
```

If yes:

```text
Commit
    │
    ▼
Database Updated
```

If no:

```text
Rollback
    │
    ▼
Database Restored
```

The database is only permanently changed after the transaction is committed.

---

# Examples

## Simple Example

Insert a user and their profile.

```dart
await transaction(() async {
  final userId = await into(users).insert(
    UsersCompanion.insert(
      name: 'John',
      age: 30,
    ),
  );

  await into(profiles).insert(
    ProfilesCompanion.insert(
      userId: userId,
      bio: 'Mobile Developer',
    ),
  );
});
```

Explanation:

* Both records are created together.
* If creating the profile fails, the user is also removed.

---

## Real-World Example

Suppose you're building an e-commerce application.

When placing an order, you need to:

* Create the order.
* Insert all order items.
* Reduce product inventory.

```dart
await transaction(() async {
  final orderId = await into(orders).insert(
    OrdersCompanion.insert(
      customerId: customerId,
    ),
  );

  for (final item in cartItems) {
    await into(orderItems).insert(
      OrderItemsCompanion.insert(
        orderId: orderId,
        productId: item.productId,
        quantity: item.quantity,
      ),
    );

    await (update(products)
          ..where((p) => p.id.equals(item.productId)))
        .write(
      ProductsCompanion(
        stock: Value(item.remainingStock),
      ),
    );
  }
});
```

Explanation:

* The entire order is processed as one unit.
* If inserting one item or updating stock fails, the entire order is rolled back.
* This prevents incomplete orders and incorrect inventory.

---

# When to Use

Use transactions when:

* Multiple operations depend on each other.
* Creating related records.
* Updating several tables.
* Moving data between tables.
* Processing payments or financial operations.
* Maintaining data consistency.

---

# When NOT to Use

A transaction isn't necessary when:

* Performing a single insert.
* Updating one row.
* Executing independent operations.
* Reading data without modifying it.

Avoid wrapping every database operation in a transaction unnecessarily.

---

# Best Practices

* Keep transactions as short as possible.
* Perform only database operations inside a transaction.
* Avoid long-running computations during a transaction.
* Group only related operations together.
* Handle exceptions appropriately.
* Let Drift automatically commit or roll back the transaction.

---

# Common Mistakes

## Using Separate Operations Instead of a Transaction

Wrong:

```dart
await createUser();
await createProfile();
```

Explanation:

* If the second operation fails, the first one has already been committed.

Correct:

```dart
await transaction(() async {
  await createUser();
  await createProfile();
});
```

Explanation:

* Both operations succeed or fail together.

---

## Performing Slow Work Inside a Transaction

Wrong:

```dart
await transaction(() async {
  await fetchDataFromApi();

  await into(users).insert(...);
});
```

Explanation:

* Long-running tasks keep the transaction open longer than necessary.

Correct:

```dart
final data = await fetchDataFromApi();

await transaction(() async {
  await into(users).insert(...);
});
```

Explanation:

* Complete non-database work before starting the transaction.

---

## Using Transactions for Unrelated Operations

Wrong:

```dart
await transaction(() async {
  await insertUser();
  await updateSettings();
  await clearLogs();
});
```

Explanation:

* These operations are unrelated and don't need to succeed or fail together.

Correct:

* Group only operations that represent a single logical unit of work.

---

# Related APIs

* `transaction()`
* Batch Operations
* Nested Transactions
* Rollback Behavior
* Insert
* Update
* Delete

---

# Summary

Transactions allow multiple database operations to be executed as a single atomic unit. If every operation succeeds, the changes are committed. If any operation fails, Drift automatically rolls back the entire transaction, ensuring your database remains consistent and preventing partial updates.
