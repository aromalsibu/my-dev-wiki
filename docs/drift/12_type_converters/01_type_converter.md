# TypeConverter

A **TypeConverter** tells Drift how to convert a custom Dart type into a SQLite-compatible value and back again.

---

# What is it?

SQLite supports only a small set of storage types:

* `INTEGER`
* `REAL`
* `TEXT`
* `BLOB`
* `NULL`

However, your Dart application often works with richer types like:

* Enums
* `DateTime`
* `Uri`
* `Color`
* Custom classes
* JSON objects

A `TypeConverter` bridges this gap by defining how a Dart object is stored in the database and how it's reconstructed when reading data.

---

# Why does it exist?

Suppose you have the following model:

```dart
enum UserRole {
  admin,
  user,
  guest,
}
```

SQLite doesn't know what a `UserRole` is.

Without a converter, you would need to manually convert it every time.

```dart
await into(users).insert(
  UsersCompanion.insert(
    role: UserRole.admin.name,
  ),
);

final user = await select(users).getSingle();
final role = UserRole.values.byName(user.role);
```

This becomes repetitive and error-prone.

A `TypeConverter` automates the conversion so your application always works with `UserRole`, while SQLite stores a supported type like `TEXT` or `INTEGER`.

---

# Syntax

A custom converter extends `TypeConverter`.

```dart
class UserRoleConverter extends TypeConverter<UserRole, String> {
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

**Explanation:**

* `UserRole` is the Dart type.
* `String` is the SQLite storage type.
* `toSql()` converts Dart → SQLite.
* `fromSql()` converts SQLite → Dart.

Apply the converter to a column.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get role =>
      text().map(const UserRoleConverter())();
}
```

**Explanation:**

* `map()` attaches the converter to the column.
* Drift automatically performs conversions during reads and writes.

---

# Mental Model

Think of a `TypeConverter` as a translator.

```text
Dart Object
(UserRole.admin)
        │
        ▼
 TypeConverter
        │
        ▼
SQLite Value
("admin")
```

Reading works in reverse.

```text
SQLite Value
("admin")
        │
        ▼
 TypeConverter
        │
        ▼
Dart Object
(UserRole.admin)
```

Your application never has to perform these conversions manually.

---

# Examples

## Simple Example

Store an enum as text.

```dart
enum Priority {
  low,
  medium,
  high,
}

class PriorityConverter extends TypeConverter<Priority, String> {
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

**Explanation:**

* SQLite stores `"low"`, `"medium"` or `"high"`.
* Drift automatically returns a `Priority` enum.

---

## Real-World Example

Suppose an e-commerce app stores the status of an order.

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

Insert data:

```dart
await into(orders).insert(
  OrdersCompanion.insert(
    customerName: 'Alice',
    status: OrderStatus.processing,
  ),
);
```

**Explanation:**

* The application inserts an `OrderStatus`.
* Drift stores `"processing"` in SQLite.

Read data:

```dart
final order = await select(orders).getSingle();

print(order.status);
```

**Explanation:**

* `order.status` is an `OrderStatus`, not a `String`.
* No manual conversion is needed.

---

# When to Use

Use a `TypeConverter` when storing:

* Enums
* `DateTime`
* JSON objects
* Custom classes
* `Uri`
* `Duration`
* `Color`
* Any Dart type unsupported by SQLite

---

# When NOT to Use

Don't use a converter for SQLite's native types:

* `int`
* `double`
* `String`
* `bool`
* `Uint8List`

Drift already knows how to store these.

---

# Best Practices

* Keep converters stateless.
* Make converters `const` whenever possible.
* Reuse the same converter across multiple tables.
* Keep conversion logic simple.
* Ensure `toSql()` and `fromSql()` are inverse operations.

---

# Common Mistakes

## Performing Manual Conversion

**Wrong**

```dart
await into(users).insert(
  UsersCompanion.insert(
    role: UserRole.admin.name,
  ),
);
```

Now every insert must manually convert the enum.

**Correct**

```dart
await into(users).insert(
  UsersCompanion.insert(
    role: UserRole.admin,
  ),
);
```

Let the converter handle the transformation.

---

## Using Different Conversion Logic

**Wrong**

```dart
@override
String toSql(UserRole value) => value.name;

@override
UserRole fromSql(String value) {
  return UserRole.admin;
}
```

The conversion isn't reversible.

**Correct**

```dart
@override
UserRole fromSql(String value) {
  return UserRole.values.byName(value);
}
```

Both methods should accurately convert between Dart and SQLite.

---

## Creating Multiple Identical Converters

**Wrong**

```dart
text().map(UserRoleConverter())
```

Repeated in every table.

**Correct**

```dart
const converter = UserRoleConverter();

text().map(converter)
```

Or simply:

```dart
text().map(const UserRoleConverter())
```

Reuse converter instances whenever possible.

---

# Related APIs

* Enum Conversion
* DateTime Conversion
* JSON Conversion
* Custom Objects
* Reusable Converters

---

# Summary

A `TypeConverter` allows Drift to store custom Dart types in SQLite by converting them to supported database values and back again. It removes the need for manual serialization, keeps your models type-safe, and makes database code cleaner by letting Drift handle conversions automatically.
