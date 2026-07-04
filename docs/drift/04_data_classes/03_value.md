# Value<T>

`Value<T>` tells Drift whether a column should be included in an insert or update operation.

---

# What is it?

`Value<T>` is a wrapper used inside companion classes to distinguish between:

* A column that should be updated.
* A column that should be left unchanged.

This distinction is important because `null` can itself be a valid value.

For example, during an update:

* Do you want to set `bio` to `null`?
* Or do you want to leave `bio` unchanged?

Without `Value<T>`, Drift wouldn't know the difference.

---

# Why does it exist?

Imagine a nullable column:

```dart
TextColumn get bio => text().nullable()();
```

Suppose you want to update only a user's name.

If you write:

```dart
bio: null
```

Does that mean:

* "Don't update the bio."

or

* "Update the bio and set it to NULL."

These are two completely different operations.

`Value<T>` removes this ambiguity by explicitly indicating whether a column should participate in the write operation.

---

# Syntax

## Updating a Value

```dart id="tb98z8"
await (update(users)
      ..where((u) => u.id.equals(1)))
    .write(
      UsersCompanion(
        age: const Value(26),
      ),
    );
```

Explanation:

* `Value(26)` tells Drift to update the `age` column.
* Columns not wrapped in `Value()` are left unchanged.

---

## Setting a Nullable Column to NULL

```dart id="j4umqk"
await (update(users)
      ..where((u) => u.id.equals(1)))
    .write(
      UsersCompanion(
        bio: const Value(null),
      ),
    );
```

Explanation:

* `Value(null)` explicitly sets the database column to `NULL`.
* This is different from omitting the column entirely.

---

## Leaving a Column Unchanged

```dart id="9o8v2z"
await (update(users)
      ..where((u) => u.id.equals(1)))
    .write(
      UsersCompanion(
        name: const Value('Alice'),
      ),
    );
```

Explanation:

* Only `name` is updated.
* Every other column remains unchanged.

---

# Mental Model

Think of `Value<T>` as an instruction.

```text id="5ujccw"
Value(25)
     │
     ▼
Update this column

--------------

Value(null)
     │
     ▼
Update this column to NULL

--------------

Column omitted
     │
     ▼
Don't touch this column
```

`Value<T>` doesn't just hold a value—it also tells Drift what action to take.

---

# Examples

## Simple Example

Update a product's price.

```dart id="tw4hpm"
await (update(products)
      ..where((p) => p.id.equals(3)))
    .write(
      ProductsCompanion(
        price: const Value(499.99),
      ),
    );
```

Explanation:

* Only the `price` column changes.
* All other columns keep their existing values.

---

## Real-World Example

Update a user's profile.

```dart id="8qj2lm"
await (update(users)
      ..where((u) => u.id.equals(8)))
    .write(
      UsersCompanion(
        name: const Value('Alice'),
        bio: const Value(null),
      ),
    );
```

Explanation:

* The user's name is updated.
* The bio is explicitly cleared.
* Other columns remain unchanged.

---

# When to Use

Use `Value<T>` when:

* Updating rows with companion classes.
* Setting nullable columns to `NULL`.
* Performing partial updates.
* Explicitly controlling which columns are written.

---

# When NOT to Use

You typically don't need `Value<T>` when:

* Reading data.
* Working with generated data classes.
* Using the `.insert()` constructor of a companion, where required fields are passed directly.

---

# Best Practices

* Wrap update values in `Value()`.
* Use `Value(null)` to intentionally store `NULL`.
* Omit fields you don't want to update.
* Don't use `null` directly inside companions.
* Think of `Value()` as indicating "include this column."

---

# Common Mistakes

## Forgetting `Value()`

**Wrong**

```dart id="ng1qje"
UsersCompanion(
  age: 30,
);
```

Explanation:

* Companion fields expect `Value<T>` objects for updates.

**Correct**

```dart id="y1hfq5"
UsersCompanion(
  age: const Value(30),
);
```

Explanation:

* `Value()` marks the column for updating.

---

## Using `null` instead of `Value(null)`

**Wrong**

```dart id="b4ntwh"
UsersCompanion(
  bio: null,
);
```

Explanation:

* This means the column is omitted from the update.
* The existing database value remains unchanged.

**Correct**

```dart id="zt4lhh"
UsersCompanion(
  bio: const Value(null),
);
```

Explanation:

* `Value(null)` explicitly updates the column to `NULL`.

---

## Wrapping every value unnecessarily

**Wrong**

Using `Value()` everywhere, including when constructing objects that don't require companions.

Explanation:

* `Value<T>` is specifically designed for companion classes and write operations.

**Correct**

Use `Value()` only where Drift expects it.

---

# Related APIs

* Companions
* Generated Data Classes
* Insertable
* Update
* Insert

---

# Summary

`Value<T>` is a wrapper that tells Drift whether a column should be included in an insert or update operation. It removes the ambiguity between "leave this column unchanged" and "set this column to `NULL`," making partial updates safe and explicit.
