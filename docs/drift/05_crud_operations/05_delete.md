# Delete

Delete removes rows from a table.

---

# What is it?

A delete operation permanently removes one or more rows from a database table.

Once a row is deleted, it is no longer returned by queries unless it is inserted again.

In Drift, deletions are performed using the `delete()` API.

---

# Why does it exist?

Applications often need to remove data.

For example:

* A user deletes a note.
* A completed task is removed.
* An old log entry is cleaned up.
* An expired session is deleted.

Without delete operations, the database would continue to grow with unwanted or outdated data.

---

# Syntax

## Deleting a Single Row

```dart
await (delete(users)
      ..where((u) => u.id.equals(1)))
    .go();
```

Explanation:

* `delete(users)` targets the `users` table.
* `where()` specifies which rows to remove.
* `go()` executes the delete operation.

---

## Deleting Multiple Rows

```dart
await (delete(tasks)
      ..where((t) => t.completed.equals(true)))
    .go();
```

Explanation:

* Every completed task is deleted.
* All rows matching the condition are removed.

---

# Mental Model

Think of deleting as removing rows from a spreadsheet.

```text
Before

+----+-------+
| ID | Name  |
+----+-------+
| 1  | Alice |
| 2  | Bob   |
| 3  | Carol |
+----+-------+

Delete ID = 2

        │
        ▼

After

+----+-------+
| ID | Name  |
+----+-------+
| 1  | Alice |
| 3  | Carol |
+----+-------+
```

The deleted row no longer exists in the table.

---

# Examples

## Simple Example

Delete a product.

```dart
await (delete(products)
      ..where((p) => p.id.equals(5)))
    .go();
```

Explanation:

* Removes the product whose ID is `5`.

---

## Real-World Example

Delete expired notifications.

```dart
await (delete(notifications)
      ..where((n) => n.expiresAt.isSmallerThanValue(DateTime.now())))
    .go();
```

Explanation:

* All expired notifications are removed.
* Active notifications remain untouched.

---

# When to Use

Use delete when you need to:

* Remove unwanted records.
* Delete user-created content.
* Clean up old data.
* Remove expired records.
* Implement data cleanup tasks.

---

# When NOT to Use

Don't use delete when:

* Updating existing data.
* Temporarily hiding records.
* Archiving data.

For these cases:

* Use `update()` to modify data.
* Consider a soft-delete flag such as `isDeleted`.
* Archive important records instead of permanently removing them.

---

# Best Practices

* Always use a `where()` clause unless you intentionally want to delete every row.
* Confirm destructive actions with users when appropriate.
* Consider soft deletes for important business data.
* Use transactions when deleting related records.
* Back up important data before bulk deletions.

---

# Common Mistakes

## Forgetting the `where()` clause

**Wrong**

```dart
await delete(users).go();
```

Explanation:

* Every row in the table is deleted.
* This is one of the most common and dangerous database mistakes.

**Correct**

```dart
await (delete(users)
      ..where((u) => u.id.equals(1)))
    .go();
```

Explanation:

* Only the intended row is removed.

---

## Deleting instead of updating

**Wrong**

Deleting a user just to change their email address.

Explanation:

* The user's data and relationships are lost.

**Correct**

Use `update()` to modify existing records.

---

## Accidentally deleting related data

**Wrong**

Deleting a parent record without considering related child records.

Explanation:

* This may violate foreign key constraints or leave orphaned data, depending on your schema.

**Correct**

Understand your table relationships and use transactions or cascading rules when appropriate.

---

# Related APIs

* Insert
* Update
* Replace
* Batch Operations
* Transactions

---

# Summary

Delete permanently removes rows from a Drift table. By combining `delete()`, `where()`, and `go()`, you can safely remove specific records while leaving the rest of the table unchanged. Always use a `where()` clause unless you intentionally want to delete every row, and consider soft deletes for data that may need to be recovered later.
