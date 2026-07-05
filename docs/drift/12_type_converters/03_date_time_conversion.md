# DateTime Conversion

**DateTime conversion** allows Drift to store `DateTime` objects in SQLite and automatically convert them back when reading data.

---

# What is it?

SQLite does not have a native `DateTime` type. It only understands:

* `INTEGER`
* `TEXT`
* `REAL`
* `BLOB`

So when you store a `DateTime` in Drift, it must be converted into one of these formats.

A `TypeConverter` handles this transformation so your Dart code always works with `DateTime`, while SQLite stores a supported representation.

---

# Why does it exist?

Without conversion, you would need to manually transform every timestamp.

```dart id="d8s1qk"
await into(tasks).insert(
  TasksCompanion.insert(
    createdAt: DateTime.now().millisecondsSinceEpoch,
  ),
);

final row = await select(tasks).getSingle();

final createdAt =
    DateTime.fromMillisecondsSinceEpoch(row.createdAt);
```

This becomes repetitive and error-prone.

A converter ensures:

* You always work with `DateTime` in Dart
* SQLite stores a consistent format
* No manual conversion is required anywhere in your code

---

# Syntax

A `DateTime` converter typically stores values as Unix timestamps (`int`).

```dart id="xk2p9v"
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

**Explanation:**

* SQLite stores `int` (Unix time in milliseconds)
* Drift converts it into a `DateTime`
* Dart code never sees the raw integer

Apply the converter to a column:

```dart id="q7m0dn"
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get title => text()();

  IntColumn get createdAt =>
      integer().map(const DateTimeConverter())();
}
```

**Explanation:**

* `createdAt` is exposed as a `DateTime` in Dart
* Internally stored as an integer timestamp

---

# Mental Model

```text id="m2q8vp"
DateTime.now()
      │
      ▼
DateTimeConverter
      │
      ▼
1700000000000  (SQLite INTEGER)
```

Reading:

```text id="p9k3xl"
1700000000000
      │
      ▼
DateTimeConverter
      │
      ▼
DateTime object
```

Your application never deals with raw timestamps directly.

---

# Examples

## Example 1: Store Creation Time

```dart id="t3w8qa"
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get title => text()();

  IntColumn get createdAt =>
      integer().map(const DateTimeConverter())();
}
```

Insert a task:

```dart id="f1v9kc"
await into(tasks).insert(
  TasksCompanion.insert(
    title: 'Write Drift Docs',
    createdAt: DateTime.now(),
  ),
);
```

Read it:

```dart id="v0x7lz"
final task = await select(tasks).getSingle();

print(task.createdAt); // DateTime instance
```

---

## Example 2: Track Updates and Events

```dart id="h4s9cd"
class Events extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  IntColumn get eventTime =>
      integer().map(const DateTimeConverter())();
}
```

Insert an event:

```dart id="g6k2lm"
await into(events).insert(
  EventsCompanion.insert(
    name: 'User Logged In',
    eventTime: DateTime.now(),
  ),
);
```

Filter by time:

```dart id="z8r3tn"
final recentEvents = await (select(events)
      ..where((e) =>
          e.eventTime.isBiggerThanValue(
            DateTime.now().subtract(
              const Duration(hours: 1),
            ),
          )))
    .get();
```

**Explanation:**

* You can use `DateTime` directly in queries
* Drift handles conversion automatically

---

## Real-World Example

A messaging app stores when each message was sent.

```dart id="c9x1dp"
class Messages extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get sender => text()();

  TextColumn get content => text()();

  IntColumn get sentAt =>
      integer().map(const DateTimeConverter())();
}
```

Insert a message:

```dart id="k2m7rt"
await into(messages).insert(
  MessagesCompanion.insert(
    sender: 'Alice',
    content: 'Hello!',
    sentAt: DateTime.now(),
  ),
);
```

Query recent messages:

```dart id="n5q8yv"
final messages = await (select(messages)
      ..orderBy([
        (m) => OrderingTerm(
              expression: m.sentAt,
              mode: OrderingMode.desc,
            ),
      ])
      ..limit(20))
    .get();
```

**Explanation:**

* Messages are sorted by real `DateTime` values
* No manual timestamp handling is required
* Code remains readable and type-safe

---

# When to Use

Use `DateTime` conversion when storing:

* Created timestamps
* Updated timestamps
* Event logs
* Scheduled tasks
* Message times
* Audit logs

---

# When NOT to Use

Avoid `DateTime` conversion when:

* You need timezone-aware strings (use ISO 8601 text instead)
* You store only relative durations (use `Duration`)
* The value is not a timestamp (e.g., just a numeric counter)

---

# Best Practices

* Prefer storing `DateTime` as **millisecondsSinceEpoch (int)**
* Keep all timestamps in UTC internally
* Convert to local time only in the UI layer
* Reuse a single converter across tables
* Avoid mixing string-based and integer-based timestamps

---

# Common Mistakes

## Storing DateTime as String

**Wrong**

```dart id="y1k8pm"
class DateTimeConverter
    extends TypeConverter<DateTime, String> {
  const DateTimeConverter();

  @override
  DateTime fromSql(String fromDb) =>
      DateTime.parse(fromDb);

  @override
  String toSql(DateTime value) =>
      value.toIso8601String();
}
```

This works but is slower and harder to query.

**Correct**

```dart id="r7v3ql"
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

Integer storage is more efficient and query-friendly.

---

## Forgetting UTC Consistency

**Wrong**

```dart id="w3x9hb"
DateTime.now()
```

Stored values may vary depending on device time zones.

**Correct**

```dart id="u8k2ld"
DateTime.now().toUtc()
```

Always store timestamps in UTC.

---

## Manual Conversion in Queries

**Wrong**

```dart id="v4n6qp"
where: (t) => t.createdAt
    .equals(DateTime.now().millisecondsSinceEpoch)
```

This bypasses type safety.

**Correct**

```dart id="j9m2xq"
where: (t) => t.createdAt
    .isBiggerThanValue(DateTime.now().subtract(Duration(days: 1)))
```

Let Drift handle conversion automatically.

---

# Related APIs

* TypeConverter
* Enum Conversion
* JSON Conversion
* Custom Objects
* Reusable Converters

---

# Summary

DateTime conversion allows Drift to store timestamps in SQLite while exposing them as strongly-typed `DateTime` objects in Dart. By converting between `DateTime` and Unix timestamps, you get efficient storage, safe queries, and clean application code without manual serialization logic.
