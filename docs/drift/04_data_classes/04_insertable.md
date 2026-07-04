# Insertable

`Insertable` represents any object that can be inserted into a Drift table.

---

# What is it?

`Insertable` is an interface used by Drift for objects that can be written to a database table.

When you call an insert method, Drift expects an `Insertable`.

Several Drift-generated classes already implement this interface, including:

* Generated data classes
* Companion classes

This means both can be passed to insert operations, although companion classes are usually the preferred choice.

---

# Why does it exist?

Drift supports multiple ways of representing data that can be inserted into a table.

Instead of requiring insert methods to accept only one specific class, Drift uses the `Insertable` interface.

This provides flexibility while keeping the insert APIs consistent.

---

# Syntax

## Inserting with a Companion

```dart
await into(users).insert(
  UsersCompanion.insert(
    name: 'Alice',
    age: 25,
  ),
);
```

Explanation:

* `UsersCompanion` implements `Insertable`.
* Drift converts it into the values needed for the insert operation.

---

## Inserting with a Generated Data Class

```dart
await into(users).insert(
  User(
    id: 1,
    name: 'Alice',
    age: 25,
  ),
);
```

Explanation:

* Generated data classes also implement `Insertable`.
* The object can be inserted directly if it contains all required values.

---

# Mental Model

Think of `Insertable` as a common contract.

```text
            Insertable
                 ▲
        ┌────────┴────────┐
        │                 │
        │                 │
Generated Data      Companion
     Class             Class
        │                 │
        └────────┬────────┘
                 ▼
         into(table).insert(...)
```

Anything that implements `Insertable` can be passed to Drift's insert APIs.

---

# Examples

## Simple Example

Insert a new task.

```dart
await into(tasks).insert(
  TasksCompanion.insert(
    title: 'Write documentation',
  ),
);
```

Explanation:

* The companion implements `Insertable`.
* Drift extracts the column values automatically.

---

## Real-World Example

A helper method that accepts any insertable object.

```dart
Future<void> saveUser(
  Insertable<User> user,
) async {
  await into(users).insert(user);
}
```

Explanation:

* The method accepts any object implementing `Insertable<User>`.
* Callers can provide either a companion or another compatible implementation.

---

# When to Use

You'll encounter `Insertable` when:

* Reading Drift API signatures.
* Creating reusable insert methods.
* Writing generic repository code.
* Working with advanced Drift APIs.

In day-to-day development, you'll most commonly use companion classes instead of interacting with `Insertable` directly.

---

# When NOT to Use

Avoid implementing `Insertable` yourself unless you have a specific advanced use case.

For most applications:

* Use generated data classes to read data.
* Use companion classes to insert and update data.

Drift already generates the required implementations.

---

# Best Practices

* Prefer companion classes for inserts.
* Use generated data classes for reading query results.
* Treat `Insertable` as an implementation detail unless writing generic APIs.
* Let Drift-generated classes handle the interface.
* Avoid custom implementations unless necessary.

---

# Common Mistakes

## Implementing `Insertable` unnecessarily

**Wrong**

```dart
class MyUser implements Insertable<User> {
  // Custom implementation
}
```

Explanation:

* Most applications don't need custom `Insertable` implementations.
* This adds unnecessary complexity.

**Correct**

Use the generated companion class.

---

## Confusing `Insertable` with a model

**Wrong**

Treating `Insertable` as a data model for your application.

Explanation:

* `Insertable` is an interface for database write operations.
* It is not intended to represent business or domain models.

**Correct**

Use generated data classes or your own domain models where appropriate.

---

## Using generated data classes for every insert

**Wrong**

```dart
await into(users).insert(
  User(
    id: 1,
    name: 'Alice',
    age: 25,
  ),
);
```

Explanation:

* This requires providing every necessary value, including fields that may be auto-generated.

**Correct**

```dart
await into(users).insert(
  UsersCompanion.insert(
    name: 'Alice',
    age: 25,
  ),
);
```

Explanation:

* Companion classes are designed specifically for insert operations.

---

# Related APIs

* Companions
* Generated Data Classes
* Value<T>
* Insert
* Update

---

# Summary

`Insertable` is the interface that represents objects capable of being inserted into a Drift table. Companion classes and generated data classes already implement this interface, allowing them to be passed directly to insert methods. While you'll rarely interact with `Insertable` directly, understanding its role helps explain how Drift's insert APIs accept different kinds of objects in a consistent way.
