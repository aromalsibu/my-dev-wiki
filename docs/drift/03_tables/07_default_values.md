# Default Values

Default values provide automatic values for columns when no value is explicitly supplied during insertion.

---

# What is it?

A default value is a value that SQLite automatically assigns to a column if an insert operation doesn't provide one.

Instead of requiring every insert to specify every column, you can define sensible defaults for commonly used values.

For example, a new task might automatically start as incomplete, or a new user account might be enabled by default.

---

# Why does it exist?

Without default values, every insert must explicitly provide values for every required column.

For example:

* New tasks should usually be incomplete.
* New users might be active by default.
* Counters often start at zero.

Rather than repeating these values in every insert statement, you can define them once in the table definition.

This reduces boilerplate and ensures consistent data.

---

# Syntax

## Defining a Default Value

```dart
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get title => text()();

  BoolColumn get completed =>
      boolean().withDefault(const Constant(false))();
}
```

Explanation:

* `withDefault()` assigns a default value to the column.
* `Constant(false)` tells SQLite to use `false` when no value is provided.
* The default is only applied during insertion.

---

## Overriding the Default

```dart
await into(tasks).insert(
  TasksCompanion.insert(
    title: 'Write documentation',
    completed: true,
  ),
);
```

Explanation:

* Providing a value explicitly overrides the default.
* SQLite only uses the default when the column is omitted.

---

# Mental Model

Think of default values as pre-filled answers on a form.

```text
Task Form

Title        ___________

Completed    ☑ false (default)
```

If you leave the field unchanged, the default value is used.

If you enter your own value, it replaces the default.

---

# Examples

## Simple Example

A users table.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  BoolColumn get isActive =>
      boolean().withDefault(const Constant(true))();
}
```

Explanation:

* Every new user is active unless another value is provided.

---

## Real-World Example

A products table.

```dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  IntColumn get stock =>
      integer().withDefault(const Constant(0))();

  BoolColumn get available =>
      boolean().withDefault(const Constant(true))();
}
```

Explanation:

* New products start with zero stock.
* Products are available by default.
* Insert operations only need to provide values that differ from these defaults.

---

# When to Use

Use default values when:

* A column usually starts with the same value.
* The value is predictable.
* You want to reduce repetitive insert code.
* The default makes sense for most new records.

Common examples include:

* `false`
* `true`
* `0`
* Empty strings
* Current timestamps (when appropriate)

---

# When NOT to Use

Avoid default values when:

* Every insert should explicitly provide the value.
* The value depends on application logic.
* Different situations require different initial values.

Don't use defaults to hide missing or incorrect application logic.

---

# Best Practices

* Use defaults for common initial values.
* Keep default values simple and predictable.
* Choose defaults that make sense for most records.
* Avoid unnecessary defaults on required business data.
* Document defaults so they're easy to understand.

---

# Common Mistakes

## Repeating the default during every insert

**Wrong**

```dart
await into(tasks).insert(
  TasksCompanion.insert(
    title: 'Buy milk',
    completed: false,
  ),
);
```

Explanation:

* `completed` already has a default value.
* Providing it every time adds unnecessary repetition.

**Correct**

```dart
await into(tasks).insert(
  TasksCompanion.insert(
    title: 'Buy milk',
  ),
);
```

Explanation:

* SQLite automatically uses the default value.

---

## Using the wrong default

**Wrong**

```dart
BoolColumn get isActive =>
    boolean().withDefault(const Constant(false))();
```

Explanation:

* If new users should normally be active, this default doesn't match the application's behavior.

**Correct**

Choose a default that reflects the most common case.

---

## Assuming defaults apply to updates

**Wrong**

Expecting a default value to be applied when updating an existing row.

Explanation:

* Default values are only used during insertion.
* Updates only change the columns you explicitly modify.

**Correct**

Provide the desired value during updates.

---

# Related APIs

* Columns
* Constraints
* Insert
* Companions
* Value<T>

---

# Summary

Default values allow SQLite to automatically assign values to columns during insertion when no value is provided. They reduce repetitive insert code, improve consistency, and make common initialization behavior part of the database schema rather than the application logic.
