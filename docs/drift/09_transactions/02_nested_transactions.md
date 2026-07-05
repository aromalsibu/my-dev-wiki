# Nested Transactions

**Nested transactions** allow a transaction to be started inside another transaction while preserving the atomic behavior of the outer transaction.

---

# What is it?

Sometimes, a function that performs database operations is already wrapped in a transaction, but that function itself also starts a transaction.

This creates a **nested transaction**.

Drift supports nested transactions, allowing reusable database functions to safely participate in larger transactions without breaking consistency.

From the caller's perspective, everything still behaves as a single transaction.

---

# Why does it exist?

Imagine you have a reusable function that creates a user.

```dart
Future<int> createUser() async {
  return transaction(() async {
    ...
  });
}
```

Later, you build a registration workflow.

```dart
await transaction(() async {
  await createUser();
  await createProfile();
});
```

Without nested transactions, this would either:

* Throw an error
* Start an unrelated transaction
* Commit part of the work too early

Drift handles this correctly so reusable functions can safely be used inside larger transactions.

---

# Syntax

Start a transaction inside another transaction.

```dart
await transaction(() async {
  await createUser();

  await transaction(() async {
    await createProfile();
  });
});
```

Explanation:

* The inner transaction executes within the context of the outer transaction.
* The outer transaction is still responsible for the final commit or rollback.

---

Nested transactions also work through function calls.

```dart
Future<void> createUserAndProfile() async {
  await transaction(() async {
    await createUser();
    await createProfile();
  });
}
```

Explanation:

* Helper methods can safely use transactions without knowing whether a transaction is already active.

---

# Mental Model

Think of nested transactions as layers.

```text
Outer Transaction
│
├── Insert User
│
├── Inner Transaction
│      │
│      ├── Insert Profile
│      └── Update Settings
│
└── Insert Log
```

Nothing is permanently written until the outer transaction finishes.

```text
Outer Transaction
        │
        ▼
Everything succeeds?
      /     \
    Yes      No
     │        │
 Commit   Rollback
```

The outer transaction controls the final outcome.

---

# Examples

## Simple Example

A reusable helper function.

```dart
Future<void> createProfile(int userId) async {
  await transaction(() async {
    await into(profiles).insert(
      ProfilesCompanion.insert(
        userId: userId,
        bio: '',
      ),
    );
  });
}
```

Explanation:

* The helper manages its own transaction.
* If it's called inside another transaction, Drift nests it automatically.

---

Use the helper inside another transaction.

```dart
await transaction(() async {
  final userId = await into(users).insert(
    UsersCompanion.insert(
      name: 'Alice',
      age: 25,
    ),
  );

  await createProfile(userId);
});
```

Explanation:

* Although `createProfile()` starts a transaction, the entire workflow behaves as a single atomic operation.

---

## Real-World Example

Suppose you're implementing user registration.

Several reusable services perform database operations.

```dart
await transaction(() async {
  final userId = await userDao.createUser(user);

  await profileDao.createProfile(userId);

  await settingsDao.createDefaultSettings(userId);

  await logDao.recordRegistration(userId);
});
```

Each DAO method may internally use a transaction.

Explanation:

* Drift safely nests these transactions.
* If any step fails, the entire registration process is rolled back.
* No partial user data remains in the database.

---

# When to Use

Nested transactions are useful when:

* Building reusable DAO methods.
* Creating helper functions that modify the database.
* Writing service-layer methods.
* Combining multiple reusable operations into one workflow.

Most of the time, nested transactions happen naturally—you don't need to design for them explicitly.

---

# When NOT to Use

Don't create nested transactions simply because you can.

Avoid unnecessary nesting when:

* All operations are already inside the same transaction.
* A helper function doesn't need transaction boundaries.
* A single transaction around the entire workflow is sufficient.

Excessive nesting can make code harder to understand.

---

# Best Practices

* Prefer one transaction around a complete business operation.
* Write reusable DAO methods that can safely be called from other transactions.
* Keep nested transactions short.
* Avoid deep levels of nesting unless necessary.
* Let the outer transaction control the final commit.

---

# Common Mistakes

## Assuming the Inner Transaction Commits Immediately

Wrong assumption:

> "The inner transaction is committed as soon as it finishes."

Explanation:

* Changes are not permanently committed until the outer transaction succeeds.

Correct understanding:

* The outer transaction determines whether all changes are committed or rolled back.

---

## Starting Transactions Everywhere

Wrong:

```dart
await transaction(() async {
  await transaction(() async {
    await transaction(() async {
      await insertUser();
    });
  });
});
```

Explanation:

* Deep nesting adds complexity without providing additional benefits.

Correct:

```dart
await transaction(() async {
  await insertUser();
  await createProfile();
});
```

Explanation:

* Use a single transaction unless reusable functions naturally introduce nesting.

---

## Catching Exceptions Without Re-throwing

Wrong:

```dart
await transaction(() async {
  try {
    await createUser();
  } catch (_) {}
});
```

Explanation:

* Swallowing the exception may allow the transaction to complete successfully even though part of the work failed.

Correct:

```dart
await transaction(() async {
  await createUser();
});
```

Or rethrow the exception after handling it if appropriate.

Explanation:

* Allow failures to propagate so Drift can roll back the transaction.

---

# Related APIs

* `transaction()`
* Rollback Behavior
* Batch Operations
* DAOs
* Insert
* Update
* Delete

---

# Summary

Nested transactions allow transaction-aware functions to be safely composed into larger workflows. Even when a transaction is started inside another transaction, Drift treats the entire operation as a single atomic unit. The outer transaction controls the final commit or rollback, ensuring your database remains consistent.
