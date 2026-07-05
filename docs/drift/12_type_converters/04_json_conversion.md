# JSON Conversion

**JSON conversion** allows Drift to store complex Dart objects as JSON while automatically converting them back into structured Dart types when reading from the database.

---

# What is it?

SQLite does not understand Dart objects like maps, lists, or custom classes.

However, it can store **text** or **blob data**, which makes JSON a natural fit for representing structured data.

A `TypeConverter` can serialize Dart objects into JSON strings when writing to the database and deserialize them back into Dart objects when reading.

---

# Why does it exist?

Without JSON conversion, storing structured data becomes verbose and repetitive.

```dart id="k2m9qp"
await into(users).insert(
  UsersCompanion.insert(
    preferences: '{"darkMode":true,"fontSize":14}',
  ),
);

final row = await select(users).getSingle();

final prefs = jsonDecode(row.preferences);
```

This leads to:

* Manual encoding/decoding everywhere
* No type safety
* Hard-to-maintain string-based logic
* Increased risk of runtime errors

JSON conversion solves this by letting you work with **typed Dart objects directly**.

---

# Syntax

A JSON converter typically uses `String` as the SQLite type.

```dart id="q8x2mp"
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

**Explanation:**

* Dart object → JSON string when saving
* JSON string → Dart `Map` when reading

Apply it to a column:

```dart id="w1p9qs"
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  TextColumn get preferences =>
      text().map(const JsonMapConverter())();
}
```

**Explanation:**

* `preferences` is exposed as a `Map<String, dynamic>`
* SQLite stores it as a JSON string

---

# Mental Model

```text id="t7k3qn"
Dart Map
{darkMode: true, fontSize: 14}
        │
        ▼
   JSON Encode
        │
        ▼
'{"darkMode":true,"fontSize":14}'
        │
        ▼
      SQLite
```

Reading:

```text id="h2v8ld"
'{"darkMode":true,"fontSize":14}'
        │
        ▼
   JSON Decode
        │
        ▼
Dart Map
```

Your app always works with structured objects.

---

# Examples

## Example 1: Store User Preferences

```dart id="m8v2qp"
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  TextColumn get preferences =>
      text().map(const JsonMapConverter())();
}
```

Insert user:

```dart id="z3q8nt"
await into(users).insert(
  UsersCompanion.insert(
    name: 'Alice',
    preferences: {
      'darkMode': true,
      'fontSize': 16,
    },
  ),
);
```

Read user:

```dart id="c9k2mv"
final user = await select(users).getSingle();

print(user.preferences['darkMode']); // true
```

**Explanation:**

* No manual JSON encoding required
* Dart Map is stored and retrieved automatically

---

## Example 2: Store Lists in JSON

```dart id="v5n2lp"
class TagsConverter
    extends TypeConverter<List<String>, String> {
  const TagsConverter();

  @override
  List<String> fromSql(String fromDb) {
    return List<String>.from(jsonDecode(fromDb));
  }

  @override
  String toSql(List<String> value) {
    return jsonEncode(value);
  }
}

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get title => text()();

  TextColumn get tags =>
      text().map(const TagsConverter())();
}
```

Insert post:

```dart id="r1k7mq"
await into(posts).insert(
  PostsCompanion.insert(
    title: 'Drift Guide',
    tags: ['flutter', 'drift', 'sqlite'],
  ),
);
```

**Explanation:**

* Lists are stored as JSON arrays
* Retrieved as typed `List<String>`

---

## Real-World Example

A settings screen stores flexible configuration data.

```dart id="b9x2qp"
class AppSettings extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get config =>
      text().map(const JsonMapConverter())();
}
```

Insert settings:

```dart id="k3m9xt"
await into(appSettings).insert(
  AppSettingsCompanion.insert(
    config: {
      'theme': 'dark',
      'notifications': true,
      'layout': {
        'grid': true,
        'columns': 3,
      },
    },
  ),
);
```

Read settings:

```dart id="u8q1nv"
final settings = await select(appSettings).getSingle();

final theme = settings.config['theme'];
final grid = settings.config['layout']['grid'];
```

**Explanation:**

* Nested JSON structures are supported
* No schema changes required for flexible data
* Ideal for dynamic configuration

---

# When to Use

Use JSON conversion when storing:

* User preferences
* App settings
* Flexible metadata
* Lists of tags or IDs
* Nested configuration objects
* API response caching

---

# When NOT to Use

Avoid JSON conversion when:

* You need to query individual fields frequently
* Data is relational (use tables instead)
* You need indexing on inner fields
* Schema is stable and structured

Example:

❌ Bad:

```dart
preferences = '{"darkMode":true}'
```

If you need to query `darkMode`, use a proper column instead.

---

# Best Practices

* Prefer JSON only for flexible or semi-structured data
* Keep JSON structure stable once used in production
* Always validate decoded data if it comes from external sources
* Use strongly typed models instead of raw `Map` where possible
* Avoid deeply nested JSON when performance matters

---

# Common Mistakes

## Storing Complex Logic in JSON

**Wrong**

```dart id="x9q2kd"
preferences: {
  'theme': {
    'dark': true,
    'rules': {
      'autoSwitch': true,
    }
  }
}
```

Overly complex JSON becomes hard to maintain.

**Correct**

Flatten or normalize structure when possible.

```dart id="n4v8pq"
preferences: {
  'darkMode': true,
  'autoSwitchTheme': true,
}
```

---

## Forgetting Type Safety

**Wrong**

```dart id="v3m9kp"
final darkMode = user.preferences['darkMode'];
```

No type guarantees.

**Correct**

```dart id="q7n2ld"
final darkMode = user.preferences['darkMode'] as bool;
```

Or better: use a typed model instead of raw maps.

---

## Storing Query-Critical Data in JSON

**Wrong**

```dart id="k1p8qd"
TextColumn get preferences => text().map(JsonMapConverter())();
```

Then trying to filter:

```dart id="x8v3qp"
where: (u) => u.preferences.equals('dark')
```

This is inefficient and unsupported.

**Correct**

```dart id="m5q8vn"
BoolColumn get darkMode => boolean()();
```

Use proper columns for queryable fields.

---

# Related APIs

* TypeConverter
* Enum Conversion
* DateTime Conversion
* Custom Objects
* Reusable Converters

---

# Summary

JSON conversion allows Drift to store complex Dart objects inside SQLite by encoding them as JSON strings. It provides flexibility for dynamic or semi-structured data while keeping your application code type-safe and clean. However, it should be used carefully and avoided for data that needs to be queried or indexed frequently.
