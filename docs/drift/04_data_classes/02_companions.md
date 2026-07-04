# Companions

Companion classes provide a flexible way to insert and update rows in a Drift database.

---

# What is it?

For every table you define, Drift automatically generates a companion class.

If you have a table named `Users`, Drift generates:

* `User` — Represents a complete database row.
* `UsersCompanion` — Represents values used for inserts and updates.

Unlike generated data classes, companion classes allow columns to be omitted, making them ideal for creating and modifying records.

---

# Why does it exist?

When inserting or updating data, you don't always provide every column.

For example:

* Auto-increment IDs are generated automatically.
* Some columns have default values.
* Some columns are nullable.
* Updates often modify only a few columns.

Generated data classes represent complete rows, but companions let you specify only the values you want to insert or update.

---

# Syntax

## Inserting with a Companion

```dart id="go0r2z"
await into(users).insert(
  UsersCompanion.insert(
    name: 'Alice',
    age: 25,
  ),
);
```

Explanation:

* `UsersCompanion.insert()` creates a companion for insertion.
* Required columns are passed as parameters.
* Auto-generated and default columns can be omitted.

---

## Updating with a Companion

```dart id="9m04v6"
await (update(users)
      ..where((u) => u.id.equals(1)))
    .write(
      UsersCompanion(
        age: const Value(26),
      ),
    );
```

Explanation:

* Only the `age` column is updated.
* Other columns remain unchanged.
* `Value()` indicates that the column should be included in the update.

---

# Mental Model

Think of companions as partial rows.

```text id="wn34yf"
Generated Data Class

User
├── id
├── name
└── age

        │

Companion

UsersCompanion
├── name
└── age
```

A data class represents an entire row.

A companion represents only the values involved in an insert or update.

---

# Examples

## Simple Example

Insert a new product.

```dart id="6rj0i8"
await into(products).insert(
  ProductsCompanion.insert(
    name: 'Laptop',
    price: 999.99,
  ),
);
```

Explanation:

* The database generates the product ID automatically.
* Only the required values are supplied.

---

## Real-World Example

Update only a user's email address.

```dart id="4k7gm4"
await (update(users)
      ..where((u) => u.id.equals(5)))
    .write(
      UsersCompanion(
        email: const Value('alice@example.com'),
      ),
    );
```

Explanation:

* Only the `email` column changes.
* Every other column keeps its current value.

---

# When to Use

Use companion classes when:

* Inserting new rows.
* Updating existing rows.
* Omitting optional columns.
* Working with default values.
* Performing partial updates.

They are the recommended way to write data in Drift.

---

# When NOT to Use

Don't use companions when:

* Reading query results.
* Passing complete database records.
* Representing an entire row.

For these cases, use the generated data class instead.

---

# Best Practices

* Use `.insert()` constructors for inserts.
* Use companion objects for updates.
* Include only the columns you want to modify.
* Let SQLite handle auto-generated values.
* Prefer companions over manually implementing `Insertable`.

---

# Common Mistakes

## Using a generated data class for inserts

**Wrong**

```dart id="h9v0u8"
await into(users).insert(
  User(
    id: 1,
    name: 'Alice',
    age: 25,
  ),
);
```

Explanation:

* Generated data classes represent complete rows.
* Auto-generated columns often shouldn't be provided manually.

**Correct**

```dart id="lygmzt"
await into(users).insert(
  UsersCompanion.insert(
    name: 'Alice',
    age: 25,
  ),
);
```

Explanation:

* The companion lets Drift generate missing values automatically.

---

## Forgetting `Value()` during updates

**Wrong**

```dart id="rmwqhz"
UsersCompanion(
  age: 26,
);
```

Explanation:

* Companion fields expect `Value<T>` objects for updates.

**Correct**

```dart id="4hhjlwm"
UsersCompanion(
  age: const Value(26),
);
```

Explanation:

* `Value()` marks the field as intentionally included in the update.

---

## Using companions to represent database rows

**Wrong**

Passing a companion throughout your application as if it were a model.

Explanation:

* Companions are designed for write operations, not as domain models.

**Correct**

Use generated data classes when working with retrieved records.

---

# Related APIs

* Generated Data Classes
* Value<T>
* Insertable
* Insert
* Update

---

# Summary

Companion classes provide a flexible way to insert and update data in Drift. Unlike generated data classes, companions allow columns to be omitted, making them ideal for partial writes, default values, and auto-generated fields. They are the standard mechanism for write operations in Drift.
