# Enum Conversion

**Enum conversion** allows Drift to store Dart enums in SQLite while automatically converting them back when reading data.

---

# What is it?

Enums are commonly used to represent a fixed set of values in Dart.

For example:

```dart
enum OrderStatus {
  pending,
  processing,
  shipped,
  delivered,
  cancelled,
}
```

SQLite has no concept of enums—it can only store primitive values like `TEXT` or `INTEGER`.

A `TypeConverter` maps the enum to one of these SQLite types so Drift can store and retrieve it automatically.

---

# Why does it exist?

Without a converter, every database operation would require manual conversion.

```dart
await into(orders).insert(
  OrdersCompanion.insert(
    status: OrderStatus.processing.name,
  ),
);

final order = await select(orders).getSingle();

final status = OrderStatus.values.byName(order.status);
```

This conversion logic would need to be repeated throughout your application.

Enum conversion centralizes that logic so your application always works with enums instead of raw strings or integers.

---

# Syntax

Create a converter for the enum.

```dart
enum OrderStatus {
  pending,
  processing,
  shipped,
  delivered,
  cancelled,
}

class OrderStatusConverter
    extends TypeConverter<OrderStatus, String> {
  const OrderStatusConverter();

  @override
  OrderStatus fromSql(String fromDb) {
    return OrderStatus.values.byName(fromDb);
  }

  @override
  String toSql(OrderStatus value) {
    return value.name;
  }
}
```

**Explanation:**

* SQLite stores a `String`.
* Drift automatically converts it into an `OrderStatus`.

Apply the converter to a column.

```dart
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get status =>
      text().map(const OrderStatusConverter())();
}
```

**Explanation:**

* `map()` registers the converter.
* Reads and writes now use `OrderStatus`.

---

# Mental Model

```text
OrderStatus.processing
          │
          ▼
  OrderStatusConverter
          │
          ▼
     "processing"
      (SQLite)
```

When reading:

```text
"processing"
     │
     ▼
OrderStatusConverter
     │
     ▼
OrderStatus.processing
```

Your application only sees the enum.

---

# Examples

## Example 1: Store Enum as Text

```dart
enum Priority {
  low,
  medium,
  high,
}

class PriorityConverter
    extends TypeConverter<Priority, String> {
  const PriorityConverter();

  @override
  Priority fromSql(String fromDb) {
    return Priority.values.byName(fromDb);
  }

  @override
  String toSql(Priority value) {
    return value.name;
  }
}

class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get priority =>
      text().map(const PriorityConverter())();
}
```

Insert a task.

```dart
await into(tasks).insert(
  TasksCompanion.insert(
    priority: Priority.high,
  ),
);
```

**Explanation:**

* SQLite stores `"high"`.
* Drift returns `Priority.high` when reading.

---

## Example 2: Store Enum as Integer

Enums can also be stored as integers.

```dart
enum ThemeMode {
  light,
  dark,
  system,
}

class ThemeModeConverter
    extends TypeConverter<ThemeMode, int> {
  const ThemeModeConverter();

  @override
  ThemeMode fromSql(int fromDb) {
    return ThemeMode.values[fromDb];
  }

  @override
  int toSql(ThemeMode value) {
    return value.index;
  }
}
```

Apply the converter.

```dart
class Settings extends Table {
  IntColumn get id => integer().autoIncrement()();

  IntColumn get themeMode =>
      integer().map(const ThemeModeConverter())();
}
```

**Explanation:**

* SQLite stores `0`, `1`, or `2`.
* Drift converts the value back into a `ThemeMode`.

---

## Real-World Example

An order management system stores the status of every order.

```dart
enum OrderStatus {
  pending,
  processing,
  shipped,
  delivered,
  cancelled,
}

class OrderStatusConverter
    extends TypeConverter<OrderStatus, String> {
  const OrderStatusConverter();

  @override
  OrderStatus fromSql(String fromDb) {
    return OrderStatus.values.byName(fromDb);
  }

  @override
  String toSql(OrderStatus value) {
    return value.name;
  }
}

class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get customerName => text()();

  TextColumn get status =>
      text().map(const OrderStatusConverter())();
}
```

Insert an order.

```dart
await into(orders).insert(
  OrdersCompanion.insert(
    customerName: 'Alice',
    status: OrderStatus.processing,
  ),
);
```

Read the order.

```dart
final order = await select(orders).getSingle();

if (order.status == OrderStatus.processing) {
  print('Preparing shipment...');
}
```

**Explanation:**

* No manual serialization is required.
* Business logic works directly with the enum.
* The database stores only a simple string.

---

# String vs Integer Storage

| Store as String                | Store as Integer                       |
| ------------------------------ | -------------------------------------- |
| Human-readable                 | More compact                           |
| Easier to debug                | Slightly smaller storage               |
| Safer if enum order changes    | Can break if enum values are reordered |
| Preferred in most applications | Useful for fixed numeric values        |

In most cases, storing enums as **strings** is recommended because it's easier to read and less error-prone.

---

# When to Use

Use enum conversion when storing:

* Order status
* User roles
* Task priority
* Payment status
* Notification type
* Theme mode
* Any finite set of predefined values

---

# When NOT to Use

Avoid enums when:

* Values change frequently.
* New values are added dynamically.
* The data belongs in a database table.

For example, product categories are usually better stored in a table than in an enum.

---

# Best Practices

* Prefer storing enums as strings.
* Make converters `const`.
* Reuse converters across tables.
* Keep conversion logic symmetrical.
* Avoid changing enum names after release if they're stored as strings.

---

# Common Mistakes

## Storing Enum Indexes Unintentionally

**Wrong**

```dart
@override
int toSql(OrderStatus value) {
  return value.index;
}
```

Changing the enum order later changes the stored meaning.

For example:

```dart
enum OrderStatus {
  pending,
  processing,
  shipped,
}
```

Later:

```dart
enum OrderStatus {
  processing,
  pending,
  shipped,
}
```

Existing database values now map to the wrong enum.

**Correct**

```dart
@override
String toSql(OrderStatus value) {
  return value.name;
}
```

Store the enum name unless you specifically need integer storage.

---

## Forgetting to Attach the Converter

**Wrong**

```dart
TextColumn get status => text()();
```

Drift treats the value as a plain string.

**Correct**

```dart
TextColumn get status =>
    text().map(const OrderStatusConverter())();
```

Always attach the converter to the column.

---

## Performing Manual Conversion

**Wrong**

```dart
status: OrderStatus.processing.name,
```

Manual conversions become repetitive and easy to forget.

**Correct**

```dart
status: OrderStatus.processing,
```

Let the converter handle the serialization automatically.

---

# Related APIs

* TypeConverter
* DateTime Conversion
* JSON Conversion
* Custom Objects
* Reusable Converters

---

# Summary

Enum conversion allows Drift to seamlessly store Dart enums as SQLite values while exposing them as enums throughout your application. By using a `TypeConverter`, you eliminate repetitive serialization code, improve type safety, and keep your business logic focused on meaningful enum values instead of raw database representations.
