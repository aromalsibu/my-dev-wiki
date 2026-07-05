# Rollback Behavior

**Rollback behavior** is the process of automatically undoing all changes made during a transaction when an error occurs before the transaction completes.

---

# What is it?

When you execute multiple database operations inside a transaction, Drift treats them as a single unit of work.

If every operation succeeds, the transaction is **committed**, making the changes permanent.

If any operation throws an exception or the transaction fails, Drift **rolls back** the transaction, discarding every change made within it.

This ensures that your database is never left in a partially updated state.

---

# Why does it exist?

Imagine registering a new user.

The workflow involves:

1. Creating the user.
2. Creating their profile.
3. Assigning a default role.

If creating the profile fails after the user has already been inserted, the database would contain a user without a profile.

Without rollback:

```text
Insert User        ✔
Insert Profile     ✖
Assign Role        ✖

Database is inconsistent.
```

With rollback:

```text
Insert User        ✔
Insert Profile     ✖
Rollback           ✔

Database returns to its original state.
```

Rollback guarantees that either the **entire operation succeeds** or **nothing changes**.

---

# Syntax

A transaction that succeeds.

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

* Both inserts complete successfully.
* Drift commits the transaction.

---

A transaction that fails.

```dart
await transaction(() async {
  await into(users).insert(
    UsersCompanion.insert(
      name: 'Alice',
      age: 25,
    ),
  );

  throw Exception('Something went wrong');
});
```

Explanation:

* The exception causes the transaction to fail.
* Drift automatically rolls back the inserted user.

---

# Mental Model

Think of a transaction as writing on a whiteboard instead of permanent paper.

```text
Start Transaction
        │
        ▼
Write Change 1
Write Change 2
Write Change 3
        │
        ▼
Everything OK?
```

If yes:

```text
Commit
    │
    ▼
Permanent Database
```

If no:

```text
Erase Whiteboard
    │
    ▼
Original Database
```

Nothing becomes permanent until the transaction is committed.

---

# Examples

## Simple Example

Create a user and profile.

```dart
await transaction(() async {
  final id = await into(users).insert(
    UsersCompanion.insert(
      name: 'John',
      age: 30,
    ),
  );

  await into(profiles).insert(
    ProfilesCompanion.insert(
      userId: id,
      bio: 'Developer',
    ),
  );
});
```

Explanation:

* If both inserts succeed, both records are saved.
* If either insert fails, neither record remains in the database.

---

## Real-World Example

Suppose you're processing an online purchase.

```dart
await transaction(() async {
  await createOrder();

  await reserveInventory();

  await processPayment();

  await createShipment();
});
```

Explanation:

* Every operation is part of one business process.
* If payment processing fails, the order, inventory reservation, and shipment are all rolled back.
* The database remains consistent.

---

# When to Use

Rollback behavior is important whenever you're using transactions for:

* Financial operations
* User registration
* Order processing
* Inventory management
* Updating multiple related tables
* Any workflow where partial success would leave inconsistent data

---

# When NOT to Use

Rollback only applies to transactions.

It isn't relevant when:

* Performing a single insert.
* Running read-only queries.
* Executing independent operations that don't need to succeed together.

---

# Best Practices

* Let exceptions propagate so Drift can roll back the transaction.
* Keep transactions focused on a single business operation.
* Keep transactions short.
* Validate input before starting a transaction when possible.
* Catch exceptions outside the transaction if you need to show an error to the user.

---

# Common Mistakes

## Catching and Ignoring Exceptions

Wrong:

```dart
await transaction(() async {
  try {
    await createUser();
    await createProfile();
  } catch (_) {}
});
```

Explanation:

* Swallowing the exception may allow the transaction to complete even though part of the work failed.

Correct:

```dart
try {
  await transaction(() async {
    await createUser();
    await createProfile();
  });
} catch (e) {
  // Handle the error.
}
```

Explanation:

* If an operation fails, the transaction rolls back.
* The exception can then be handled by the caller.

---

## Assuming Previous Operations Are Saved

Wrong assumption:

> "The first insert already happened, so it won't be undone."

Explanation:

* Until the transaction commits, none of its changes are permanent.

Correct understanding:

* Every change inside the transaction is rolled back if the transaction fails.

---

## Mixing Database and External Operations

Wrong:

```dart
await transaction(() async {
  await createOrder();

  await sendEmail();

  await updateInventory();
});
```

Explanation:

* Rollback only affects database operations.
* Sending an email cannot be undone if the transaction later fails.

Correct:

```dart
await transaction(() async {
  await createOrder();
  await updateInventory();
});

await sendEmail();
```

Explanation:

* Complete the transaction first.
* Perform external side effects only after the transaction succeeds.

---

# Related APIs

* `transaction()`
* Nested Transactions
* Batch vs Transaction
* Insert
* Update
* Delete

---

# Summary

Rollback behavior is a core feature of transactions in Drift. If any operation inside a transaction fails, Drift automatically undoes every change made during that transaction. This ensures your database remains consistent and prevents partially completed operations from leaving invalid or incomplete data.
