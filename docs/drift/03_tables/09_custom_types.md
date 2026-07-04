# Custom Types

Custom types allow you to store Dart types that SQLite doesn't support natively.

---

# What is it?

SQLite supports only a small set of built-in data types, such as:

* Integer
* Text
* Real
* Blob
* Null

However, applications often work with richer Dart types like:

* Enums
* Objects
* JSON models
* Custom classes

Custom types allow these Dart types to be stored in SQLite by converting them to and from supported database types.

In Drift, this is done using **type converters**.

---

# Why does it exist?

SQLite doesn't understand Dart-specific types.

For example, SQLite cannot directly store:

* `Color`
* `UserRole`
* `Address`
* `Settings`
* `Money`

Without custom types, you would need to manually convert these values every time you insert or read data.

Type converters automate this conversion, allowing you to work with your custom Dart types while Drift handles the database representation.

---

# Syntax

## Using a Custom Type

```dart
enum UserRole {
  admin,
  user,
  guest,
}

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get role =>
      text().map(const UserRoleConverter())();
}
```

Explanation:

* `map()` attaches a type converter to the column.
* Drift exposes the column as `UserRole` in Dart.
* SQLite stores the converted value.

> **Note:** Creating custom converters is covered in the **TypeConverter** section.

---

# Mental Model

Think of a type converter as a translator.

```text
Dart Object
     │
     ▼
Type Converter
     │
     ▼
SQLite Value
```

When writing data:

```text
UserRole.admin
        │
        ▼
    "admin"
        │
        ▼
     SQLite
```

When reading data:

```text
SQLite
   │
"admin"
   │
   ▼
UserRole.admin
```

---

# Examples

## Simple Example

Store an enum.

```dart
enum Priority {
  low,
  medium,
  high,
}

class Tasks extends Table {
  TextColumn get priority =>
      text().map(const PriorityConverter())();
}
```

Explanation:

* Drift automatically converts between `Priority` and the stored database value.
* Your application works directly with the enum instead of raw strings.

---

## Real-World Example

Store a custom settings object.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get settings =>
      text().map(const SettingsConverter())();
}
```

Explanation:

* The converter transforms the `Settings` object into a storable format, such as JSON.
* Reading the column automatically recreates the original object.

---

# When to Use

Use custom types when working with:

* Enums
* JSON models
* Value objects
* Domain models
* Custom Dart classes

They help keep your application code strongly typed while remaining compatible with SQLite.

---

# When NOT to Use

Avoid custom types when:

* SQLite already provides a suitable built-in type.
* A simple `text()`, `integer()`, or `boolean()` column is sufficient.
* The conversion adds unnecessary complexity.

Don't create custom types for data that can be stored naturally.

---

# Best Practices

* Keep converters simple and predictable.
* Use converters for reusable domain types.
* Prefer enums over raw strings when appropriate.
* Ensure conversions are reversible.
* Reuse converters across multiple tables when possible.

---

# Common Mistakes

## Storing enums as raw strings everywhere

**Wrong**

```dart
TextColumn get role => text()();
```

Explanation:

* Every query must manually convert between strings and enums.
* Invalid string values become harder to detect.

**Correct**

```dart
TextColumn get role =>
    text().map(const UserRoleConverter())();
```

Explanation:

* Drift handles the conversion automatically.

---

## Writing conversion logic repeatedly

**Wrong**

```dart
if (role == UserRole.admin) {
  // Convert manually
}
```

Explanation:

* Manual conversions scattered throughout the application are difficult to maintain.

**Correct**

Use a reusable type converter.

---

## Using custom types unnecessarily

**Wrong**

Creating a converter for an integer value that doesn't require special handling.

Explanation:

* Built-in SQLite types should be used whenever possible.

**Correct**

Reserve custom types for values that SQLite cannot represent directly.

---

# Related APIs

* TypeConverter
* Enum Conversion
* JSON Conversion
* Custom Objects
* Reusable Converters
* Columns

---

# Summary

Custom types let you work with Dart-specific types while storing compatible values in SQLite. By using type converters, Drift automatically converts data between your application's types and SQLite's supported types, resulting in cleaner, safer, and more expressive code.
