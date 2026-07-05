# Replace

Replace updates an existing row by replacing all of its column values.

---

# What is it?

`replace()` updates a row using an entire data object instead of updating individual columns.

Unlike `update()`, which modifies only selected columns, `replace()` treats the provided object as the complete replacement for the existing row.

The row is identified using its primary key.

---

# Why does it exist?

Sometimes you already have a complete object representing the latest state of a row.

For example:

* A user edits every field in a profile.
* A form submits all values at once.
* You retrieve an object, modify it, and save it back.

Using `replace()` is simpler than creating a companion and wrapping every changed field with `Value()`.

---

# Syntax

## Replacing a Row

```dart
final user = User(
  id: 1,
  name: 'Alice',
  age: 26,
);

await update(users).replace(user);
```

Explanation:

* `replace()` uses the row's primary key (`id`) to find the existing record.
* Every column in the object is written back to the database.
* The row must already exist.

---

## Updating an Existing Object

```dart
final existingUser = await (select(users)
      ..where((u) => u.id.equals(1)))
    .getSingle();

final updatedUser = existingUser.copyWith(
  age: 27,
);

await update(users).replace(updatedUser);
```

Explanation:

* The existing row is read from the database.
* `copyWith()` creates an updated version.
* `replace()` writes the new object back.

---

# Mental Model

Think of replacing as swapping an entire document.

```text
Before

User
├── id: 1
├── name: Alice
└── age: 25

        │
        ▼
     Replace

        │
        ▼

User
├── id: 1
├── name: Alice
└── age: 26
```

Instead of editing one field individually, the entire object becomes the new database row.

---

# Examples

## Simple Example

Update a product.

```dart
await update(products).replace(
  Product(
    id: 3,
    name: 'Laptop',
    price: 899.99,
  ),
);
```

Explanation:

* The product with ID `3` is replaced.
* Every property in the `Product` object is written to the database.

---

## Real-World Example

Saving an edited profile.

```dart
final updatedCustomer = customer.copyWith(
  email: 'alice@example.com',
  phone: '1234567890',
);

await update(customers).replace(updatedCustomer);
```

Explanation:

* The application edits the object in memory.
* The updated object replaces the corresponding database row.

---

# When to Use

Use `replace()` when:

* You already have the complete object.
* Saving data from an edit form.
* Updating most or all columns.
* Working with immutable models and `copyWith()`.

---

# When NOT to Use

Avoid `replace()` when:

* Only one or two columns need updating.
* You don't have all column values.
* You're performing partial updates.

In these situations, `update()` with a companion is usually more appropriate.

---

# Best Practices

* Ensure the object has a valid primary key.
* Use `copyWith()` to modify immutable data classes.
* Use `replace()` only when updating complete objects.
* Prefer `update()` for partial changes.
* Verify that the row exists before replacing if necessary.

---

# Common Mistakes

## Using `replace()` for partial updates

**Wrong**

```dart
await update(users).replace(
  User(
    id: 1,
    name: 'Alice',
    age: 0,
  ),
);
```

Explanation:

* Every field in the object is written to the database.
* Placeholder or incorrect values may overwrite existing data.

**Correct**

Use `update()` with a companion when only a few columns need to change.

---

## Forgetting the primary key

**Wrong**

```dart
await update(users).replace(
  User(
    id: 0,
    name: 'Alice',
    age: 26,
  ),
);
```

Explanation:

* `replace()` uses the primary key to locate the row.
* An incorrect or missing key may prevent the update.

**Correct**

Provide the correct primary key value for the row being replaced.

---

## Confusing `replace()` with SQL REPLACE

**Wrong**

Assuming `replace()` inserts a new row if one doesn't exist.

Explanation:

* Drift's `replace()` performs an update based on the primary key.
* It is not the same as SQLite's `INSERT OR REPLACE`.

**Correct**

Use `replace()` to update existing rows, and use `upsert()` when you need insert-or-update behavior.

---

# Related APIs

* Update
* Upsert
* Insert
* Generated Data Classes
* Companions

---

# Summary

`replace()` updates an existing row by writing an entire data object back to the database. It is most useful when you already have a complete model, such as after editing a form or using `copyWith()`. For partial updates, `update()` with companion classes is generally the better choice.
