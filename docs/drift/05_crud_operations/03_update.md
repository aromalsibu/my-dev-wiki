# Update

Update modifies existing rows in a table.

---

# What is it?

An update operation changes the values of one or more columns in existing database rows.

Unlike an insert, which creates a new row, an update modifies data that already exists.

In Drift, updates are performed using the `update()` API.

---

# Why does it exist?

Data changes over time.

For example:

* A user updates their email address.
* A task is marked as completed.
* A product's price changes.
* An order's status is updated.

Without update operations, you would have to delete the old row and insert a new one, which is inefficient and can break relationships.

---

# Syntax

## Updating a Row

```dart id="d0rj5g"
await (update(users)
      ..where((u) => u.id.equals(1)))
    .write(
      UsersCompanion(
        age: const Value(26),
      ),
    );
```

Explanation:

* `update(users)` targets the `users` table.
* `where()` selects the rows to update.
* `write()` applies the new values.
* `Value(26)` marks the `age` column for updating.

---

## Updating Multiple Columns

```dart id="8zqgqt"
await (update(users)
      ..where((u) => u.id.equals(1)))
    .write(
      UsersCompanion(
        name: const Value('Alice Smith'),
        age: const Value(26),
      ),
    );
```

Explanation:

* Both `name` and `age` are updated.
* Any omitted columns remain unchanged.

---

# Mental Model

Think of updating as editing a row in a spreadsheet.

```text id="zjlwmm"
Before

+----+-------+-----+
| ID | Name  | Age |
+----+-------+-----+
| 1  | Alice | 25  |
+----+-------+-----+

        │
        ▼
     Update Age

        │
        ▼

After

+----+-------+-----+
| ID | Name  | Age |
+----+-------+-----+
| 1  | Alice | 26  |
+----+-------+-----+
```

The row stays the same; only the selected columns change.

---

# Examples

## Simple Example

Mark a task as completed.

```dart id="yvc5nt"
await (update(tasks)
      ..where((t) => t.id.equals(5)))
    .write(
      TasksCompanion(
        completed: const Value(true),
      ),
    );
```

Explanation:

* Only the `completed` column changes.
* The rest of the task remains unchanged.

---

## Real-World Example

Update a customer's contact information.

```dart id="zrd7p0"
await (update(customers)
      ..where((c) => c.id.equals(10)))
    .write(
      CustomersCompanion(
        email: const Value('alice@example.com'),
        phone: const Value('1234567890'),
      ),
    );
```

Explanation:

* Both contact fields are updated.
* Other customer information remains untouched.

---

# When to Use

Use update when you need to:

* Edit user information.
* Mark tasks as complete.
* Change product prices.
* Update order statuses.
* Modify existing records.

---

# When NOT to Use

Don't use update when:

* Creating new records.
* Replacing an entire row.
* Deleting data.

Instead, use:

* `insert()` for new rows.
* `replace()` to replace an entire record.
* `delete()` to remove rows.

---

# Best Practices

* Always use a `where()` clause unless you intentionally want to update every row.
* Update only the columns that need to change.
* Use companion classes for updates.
* Wrap updated values in `Value()`.
* Validate user input before updating the database.

---

# Common Mistakes

## Forgetting the `where()` clause

**Wrong**

```dart id="5g4v1k"
await update(users).write(
  UsersCompanion(
    age: const Value(26),
  ),
);
```

Explanation:

* Every row in the table is updated.
* This is a common and potentially serious mistake.

**Correct**

```dart id="h2p4fd"
await (update(users)
      ..where((u) => u.id.equals(1)))
    .write(
      UsersCompanion(
        age: const Value(26),
      ),
    );
```

Explanation:

* Only the intended row is modified.

---

## Forgetting `Value()`

**Wrong**

```dart id="g6x1je"
UsersCompanion(
  age: 26,
);
```

Explanation:

* Companion fields require `Value<T>` for updates.

**Correct**

```dart id="wn2s7x"
UsersCompanion(
  age: const Value(26),
);
```

Explanation:

* `Value()` explicitly marks the field for updating.

---

## Using update to create new data

**Wrong**

Attempting to add a new user with `update()`.

Explanation:

* Updates only affect existing rows.
* If no matching row exists, nothing is updated.

**Correct**

Use `insert()` to create new records.

---

# Related APIs

* Insert
* Replace
* Delete
* Value<T>
* Companions

---

# Summary

Update modifies existing rows in a Drift table without creating new ones. By combining `update()`, `where()`, and companion classes, you can safely change specific columns while leaving the rest of the row unchanged. Always use a `where()` clause to ensure only the intended rows are updated.
