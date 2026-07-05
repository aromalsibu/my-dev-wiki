# Reusable Converters

**Reusable converters** are `TypeConverter`s that are defined once and shared across multiple tables and columns to keep your Drift code consistent, clean, and maintainable.

---

# What is it?

A reusable converter is a **single converter instance or class** that is used in more than one place in your database schema.

Instead of redefining the same conversion logic repeatedly in each table, you centralize it and reuse it wherever needed.

For example:

* A `DateTimeConverter` used in multiple tables
* An `EnumConverter` shared across entities
* A `JsonConverter` used for multiple configuration fields

---

# Why does it exist?

Without reusable converters, you end up duplicating logic:

```dart id="r1k9qp"
text().map(const DateTimeConverter())();
text().map(const DateTimeConverter())();
text().map(const DateTimeConverter())();
```

Problems with this approach:

* Repetition across tables
* Harder maintenance
* Inconsistent conversion logic risk
* Difficult schema updates later

Reusable converters solve this by providing:

* Single source of truth
* Consistent serialization behavior
* Easier refactoring
* Cleaner schema definitions

---

# Syntax

A reusable converter is just a normal `TypeConverter` declared once and reused.

```dart id="c9m2qp"
class DateTimeConverter
    extends TypeConverter<DateTime, int> {
  const DateTimeConverter();

  @override
  DateTime fromSql(int fromDb) {
    return DateTime.fromMillisecondsSinceEpoch(fromDb);
  }

  @override
  int toSql(DateTime value) {
    return value.millisecondsSinceEpoch;
  }
}
```

Now reuse it across multiple tables:

```dart id="v2k9qp"
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get createdAt =>
      text().map(const DateTimeConverter())();

  TextColumn get updatedAt =>
      text().map(const DateTimeConverter())();
}
```

```dart id="m8v3qp"
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get placedAt =>
      text().map(const DateTimeConverter())();

  TextColumn get shippedAt =>
      text().map(const DateTimeConverter())();
}
```

**Explanation:**

* Same converter is reused everywhere
* All timestamps behave consistently
* No duplicated logic

---

# Mental Model

```text id="t9k2qp"
           ┌────────────────────┐
           │ DateTimeConverter  │
           └─────────┬──────────┘
                     │
     ┌───────────────┼───────────────┐
     ▼                               ▼
Users.createdAt               Orders.shippedAt
     ▼                               ▼
  INTEGER                       INTEGER
```

One converter, many columns.

---

# Examples

## Example 1: Shared Enum Converter

```dart id="e2m9qp"
enum UserRole {
  admin,
  user,
  guest,
}

class UserRoleConverter
    extends TypeConverter<UserRole, String> {
  const UserRoleConverter();

  @override
  UserRole fromSql(String fromDb) {
    return UserRole.values.byName(fromDb);
  }

  @override
  String toSql(UserRole value) {
    return value.name;
  }
}
```

Reuse across tables:

```dart id="u7k2qp"
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get role =>
      text().map(const UserRoleConverter())();
}
```

```dart id="w1m8qp"
class AdminLogs extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get actorRole =>
      text().map(const UserRoleConverter())();
}
```

**Explanation:**

* Same enum mapping everywhere
* Prevents inconsistent storage formats

---

## Example 2: Shared JSON Converter

```dart id="q8m2qp"
import 'dart:convert';

class JsonMapConverter
    extends TypeConverter<Map<String, dynamic>, String> {
  const JsonMapConverter();

  @override
  Map<String, dynamic> fromSql(String fromDb) {
    return jsonDecode(fromDb) as Map<String, dynamic>;
  }

  @override
  String toSql(Map<String, dynamic> value) {
    return jsonEncode(value);
  }
}
```

Reuse:

```dart id="n3v9qp"
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get preferences =>
      text().map(const JsonMapConverter())();
}
```

```dart id="k7m2qp"
class AppSettings extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get config =>
      text().map(const JsonMapConverter())();
}
```

**Explanation:**

* Same JSON structure handling everywhere
* Ensures consistent encoding/decoding behavior

---

## Example 3: Central DateTime Converter in Large App

```dart id="x9m2qp"
class DateTimeConverter
    extends TypeConverter<DateTime, int> {
  const DateTimeConverter();

  @override
  DateTime fromSql(int fromDb) =>
      DateTime.fromMillisecondsSinceEpoch(fromDb);

  @override
  int toSql(DateTime value) =>
      value.millisecondsSinceEpoch;
}
```

Used across many tables:

```dart id="p2k8qp"
class Messages extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get sentAt =>
      text().map(const DateTimeConverter())();
}
```

```dart id="v8m2qp"
class Notifications extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get createdAt =>
      text().map(const DateTimeConverter())();
}
```

```dart id="s1k9qp"
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get dueAt =>
      text().map(const DateTimeConverter())();
}
```

---

# When to Use

Use reusable converters when:

* Same type appears in multiple tables
* You want consistent storage format across app
* You expect schema growth over time
* You want centralized conversion logic
* You are building production-scale apps

---

# When NOT to Use

Avoid forcing reuse when:

* Conversion logic is table-specific
* Data formats differ slightly per use case
* You are experimenting or prototyping
* You need specialized per-column behavior

---

# Best Practices

* Always mark converters as `const`
* Keep converters stateless
* Centralize them in a `converters/` directory
* Reuse instead of copying logic
* Avoid modifying behavior after release
* Keep naming explicit (`DateTimeConverter`, not `Converter1`)

---

# Common Mistakes

## Duplicating Converter Logic

**Wrong**

```dart id="m9k2qp"
class AConverter extends TypeConverter<DateTime, int> { ... }
class BConverter extends TypeConverter<DateTime, int> { ... }
```

Same logic duplicated across codebase.

**Correct**

```dart id="c2m9qp"
const dateTimeConverter = DateTimeConverter();
```

Reuse a single converter.

---

## Slightly Different Converters for Same Type

**Wrong**

```dart id="z8k2qp"
millisecondsSinceEpoch in one place
secondsSinceEpoch in another
```

This creates inconsistent data.

**Correct**

Always standardize one format across the app.

---

## Inline Anonymous Conversion Logic

**Wrong**

```dart id="q1m8qp"
text().map(TypeConverter.from(...))
```

Hard to reuse and maintain.

**Correct**

Define a named converter class.

---

# Related APIs

* TypeConverter
* Enum Conversion
* DateTime Conversion
* JSON Conversion
* Custom Objects

---

# Summary

Reusable converters allow you to define conversion logic once and apply it across multiple tables and columns. This ensures consistency, reduces duplication, and makes your Drift database easier to maintain and scale over time.
