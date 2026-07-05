# Returning Rows

Returning rows lets you obtain the row that was inserted, updated, or deleted as part of a write operation.

---

# What is it?

Normally, write operations such as `insert()`, `update()`, and `delete()` only tell you whether the operation succeeded.

Sometimes, however, you also need the affected row.

For example:

* Insert a user and immediately display it.
* Update a task and return its latest values.
* Delete a record but keep a copy for logging.

SQLite supports the `RETURNING` clause, and Drift exposes this through its returning APIs.

---

# Why does it exist?

Without returning rows, you typically perform two database operations:

```text
Insert
   │
   ▼
Run another SELECT query
```

This means:

* An extra database query.
* Additional code.
* Slightly lower performance.

Returning rows combines these into a single operation.

---

# Syntax

## Returning an Inserted Row

```dart
final user = await into(users).insertReturning(
  UsersCompanion.insert(
    name: 'Alice',
    age: 25,
  ),
);
```

Explanation:

* `insertReturning()` inserts a new row.
* It immediately returns the generated data class for the inserted row.
* Auto-generated values, such as IDs, are already populated.

---

## Returning an Updated Row

```dart
final updatedTask = await (update(tasks)
      ..where((t) => t.id.equals(1)))
    .writeReturning(
      TasksCompanion(
        completed: const Value(true),
      ),
    );
```

Explanation:

* `writeReturning()` updates matching rows.
* It returns the updated row or rows after the operation.

---

## Returning Deleted Rows

```dart
final deletedUsers = await (delete(users)
      ..where((u) => u.isActive.equals(false)))
    .goAndReturn();
```

Explanation:

* `goAndReturn()` deletes matching rows.
* The deleted rows are returned before being removed from the database.

---

# Mental Model

Instead of performing two separate operations:

```text
Write
   │
   ▼
Select
```

Returning APIs perform both together.

```text
Write
   │
   ▼
Return Updated Row
```

One operation modifies the database and gives you the affected data immediately.

---

# Examples

## Simple Example

Create a product and obtain its generated ID.

```dart
final product = await into(products).insertReturning(
  ProductsCompanion.insert(
    name: 'Laptop',
    price: 999.99,
  ),
);

print(product.id);
```

Explanation:

* SQLite generates the ID.
* The returned object already contains it.
* No additional query is required.

---

## Real-World Example

Update a user's profile and return the latest version.

```dart
final updatedUser = await (update(users)
      ..where((u) => u.id.equals(5)))
    .writeReturning(
      UsersCompanion(
        email: const Value('alice@example.com'),
      ),
    );
```

Explanation:

* The email is updated.
* The latest database row is returned immediately.

---

# When to Use

Use returning rows when you need:

* Auto-generated IDs after inserts.
* Updated objects after writes.
* Deleted records for logging or auditing.
* To avoid an extra `SELECT` query.

---

# When NOT to Use

Avoid returning rows when:

* You don't need the affected data.
* Performance is critical and the returned objects won't be used.
* You're targeting SQLite versions that don't support the `RETURNING` clause.

A normal insert, update, or delete is sufficient in these cases.

---

# Best Practices

* Use returning APIs when you immediately need the affected row.
* Prefer them over performing a separate query.
* Be aware of SQLite version requirements.
* Return only what your application actually needs.
* Test returning behavior on all supported platforms.

---

# Common Mistakes

## Performing an unnecessary follow-up query

**Wrong**

```dart
await into(users).insert(
  UsersCompanion.insert(
    name: 'Alice',
    age: 25,
  ),
);

final user = await (select(users)
      ..orderBy([
        (u) => OrderingTerm.desc(u.id),
      ]))
    .getSingle();
```

Explanation:

* This requires two database operations.
* The second query is unnecessary.

**Correct**

```dart
final user = await into(users).insertReturning(
  UsersCompanion.insert(
    name: 'Alice',
    age: 25,
  ),
);
```

Explanation:

* The inserted row is returned immediately.

---

## Assuming every database supports returning

**Wrong**

Using returning APIs without considering SQLite compatibility.

Explanation:

* Older SQLite versions may not support the `RETURNING` clause.

**Correct**

Verify that your target SQLite version supports returning operations.

---

## Ignoring the returned data

**Wrong**

Using a returning API but discarding the result.

Explanation:

* If the returned row isn't needed, a regular write operation is simpler.

**Correct**

Use returning APIs only when the returned data provides value.

---

# Related APIs

* Insert
* Update
* Delete
* Generated Data Classes
* Select

---

# Summary

Returning rows allow Drift to return the rows affected by insert, update, or delete operations in the same database command. This avoids additional queries, simplifies code, and is especially useful when you need generated IDs or the latest version of modified data immediately after a write operation.
