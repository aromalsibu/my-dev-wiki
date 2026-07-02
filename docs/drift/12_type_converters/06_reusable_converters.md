## Reusable Converters

**Creating and sharing TypeConverter implementations across your Drift application**

---

# What is it?

**Reusable Converters** are TypeConverter implementations that are designed to be shared across multiple tables, databases, and projects. Instead of recreating the same converter logic each time, you can create a single, well-tested converter that can be used wherever needed, promoting code reuse, consistency, and maintainability.

> **Think of Reusable Converters like "standardized measurement tools"** – instead of each person making their own ruler, you create a standard ruler that everyone can use, ensuring consistent measurements across the organization.

```dart
// 👇 Reusable converter for JSON maps
class JsonMapConverter extends TypeConverter<Map<String, dynamic>, String> {
  const JsonMapConverter();
  
  @override
  Map<String, dynamic> fromSql(String fromDb) {
    if (fromDb.isEmpty) return {};
    return jsonDecode(fromDb) as Map<String, dynamic>;
  }
  
  @override
  String toSql(Map<String, dynamic> value) {
    return jsonEncode(value);
  }
}

// 👇 Using the same converter across multiple tables
class Users extends Table {
  TextColumn get preferences => text().map(const JsonMapConverter())();
}

class Products extends Table {
  TextColumn get metadata => text().map(const JsonMapConverter())();
}

class Orders extends Table {
  TextColumn get details => text().map(const JsonMapConverter())();
}
```

> **What's happening here?**
> - **Single source** – One converter definition
> - **Multiple uses** – Used across many tables
> - **Consistency** – Same behavior everywhere
> - **Maintainability** – Change once, update everywhere

---

# Why does it exist?

- **Code Reuse** – Don't repeat yourself
- **Consistency** – Same behavior across tables
- **Maintainability** – Change in one place
- **Testing** – Test once, use everywhere
- **Organization** – Centralized converter registry
- **Onboarding** – Easy for new developers

---

# Creating Reusable Converters

> **Building converters you can share**

## Base Converter Library

```dart
// lib/database/converters/base_converters.dart
import 'package:drift/drift.dart';
import 'dart:convert';

// 👇 1. JSON Map Converter
class JsonMapConverter extends TypeConverter<Map<String, dynamic>, String> {
  const JsonMapConverter();
  
  @override
  Map<String, dynamic> fromSql(String fromDb) {
    if (fromDb.isEmpty) return {};
    return jsonDecode(fromDb) as Map<String, dynamic>;
  }
  
  @override
  String toSql(Map<String, dynamic> value) {
    return jsonEncode(value);
  }
}

// 👇 2. JSON List Converter
class JsonListConverter extends TypeConverter<List<dynamic>, String> {
  const JsonListConverter();
  
  @override
  List<dynamic> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    return jsonDecode(fromDb) as List<dynamic>;
  }
  
  @override
  String toSql(List<dynamic> value) {
    return jsonEncode(value);
  }
}

// 👇 3. String List Converter
class StringListConverter extends TypeConverter<List<String>, String> {
  const StringListConverter();
  
  @override
  List<String> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => e as String).toList();
  }
  
  @override
  String toSql(List<String> value) {
    return jsonEncode(value);
  }
}

// 👇 4. Int List Converter
class IntListConverter extends TypeConverter<List<int>, String> {
  const IntListConverter();
  
  @override
  List<int> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => e as int).toList();
  }
  
  @override
  String toSql(List<int> value) {
    return jsonEncode(value);
  }
}

// 👇 5. Bool List Converter
class BoolListConverter extends TypeConverter<List<bool>, String> {
  const BoolListConverter();
  
  @override
  List<bool> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => e as bool).toList();
  }
  
  @override
  String toSql(List<bool> value) {
    return jsonEncode(value);
  }
}
```

---

# Advanced Reusable Converters

> **Generic and flexible converters**

## Enum Converter

```dart
// lib/database/converters/enum_converters.dart
import 'package:drift/drift.dart';

// 👇 Reusable enum converter
class EnumConverter<T extends Enum> extends TypeConverter<T, String> {
  final List<T> values;
  final T defaultValue;
  
  const EnumConverter(this.values, this.defaultValue);
  
  @override
  T fromSql(String fromDb) {
    return values.firstWhere(
      (e) => e.name == fromDb,
      orElse: () => defaultValue,
    );
  }
  
  @override
  String toSql(T value) {
    return value.name;
  }
}

// Usage
enum UserStatus { active, inactive, pending }

class Users extends Table {
  TextColumn get status => text()
    .map(const EnumConverter<UserStatus>(UserStatus.values, UserStatus.pending))();
}

enum OrderStatus { pending, processing, shipped, delivered }

class Orders extends Table {
  TextColumn get status => text()
    .map(const EnumConverter<OrderStatus>(OrderStatus.values, OrderStatus.pending))();
}
```

---

## Object Converter

```dart
// lib/database/converters/object_converters.dart
import 'package:drift/drift.dart';
import 'dart:convert';

// 👇 Reusable object converter
class ObjectConverter<T> extends TypeConverter<T, String> {
  final T Function(Map<String, dynamic>) fromJson;
  final Map<String, dynamic> Function(T) toJson;
  
  const ObjectConverter({
    required this.fromJson,
    required this.toJson,
  });
  
  @override
  T fromSql(String fromDb) {
    if (fromDb.isEmpty) return null;
    final json = jsonDecode(fromDb) as Map<String, dynamic>;
    return fromJson(json);
  }
  
  @override
  String toSql(T value) {
    return jsonEncode(toJson(value));
  }
}

// Usage
class Address {
  final String street;
  final String city;
  
  Address({required this.street, required this.city});
  
  Map<String, dynamic> toJson() => {'street': street, 'city': city};
  factory Address.fromJson(Map<String, dynamic> json) =>
      Address(street: json['street'] as String, city: json['city'] as String);
}

class Users extends Table {
  TextColumn get address => text().nullable().map(
    const ObjectConverter<Address>(
      fromJson: Address.fromJson,
      toJson: (a) => a.toJson(),
    ),
  )();
}
```

---

# Centralized Converter Registry

> **Organizing converters in one place**

```dart
// lib/database/converters/converters.dart
import 'package:drift/drift.dart';
import 'dart:convert';

// ==================== BASE CONVERTERS ====================

class JsonMapConverter extends TypeConverter<Map<String, dynamic>, String> {
  const JsonMapConverter();
  @override Map<String, dynamic> fromSql(String fromDb) {
    if (fromDb.isEmpty) return {};
    return jsonDecode(fromDb) as Map<String, dynamic>;
  }
  @override String toSql(Map<String, dynamic> value) => jsonEncode(value);
}

class JsonListConverter extends TypeConverter<List<dynamic>, String> {
  const JsonListConverter();
  @override List<dynamic> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    return jsonDecode(fromDb) as List<dynamic>;
  }
  @override String toSql(List<dynamic> value) => jsonEncode(value);
}

// ==================== LIST CONVERTERS ====================

class StringListConverter extends TypeConverter<List<String>, String> {
  const StringListConverter();
  @override List<String> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => e as String).toList();
  }
  @override String toSql(List<String> value) => jsonEncode(value);
}

class IntListConverter extends TypeConverter<List<int>, String> {
  const IntListConverter();
  @override List<int> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => e as int).toList();
  }
  @override String toSql(List<int> value) => jsonEncode(value);
}

// ==================== DATE CONVERTERS ====================

class UnixDateTimeConverter extends TypeConverter<DateTime, int> {
  const UnixDateTimeConverter();
  @override DateTime fromSql(int fromDb) =>
      DateTime.fromMillisecondsSinceEpoch(fromDb * 1000);
  @override int toSql(DateTime value) =>
      value.millisecondsSinceEpoch ~/ 1000;
}

class IsoDateTimeConverter extends TypeConverter<DateTime, String> {
  const IsoDateTimeConverter();
  @override DateTime fromSql(String fromDb) => DateTime.parse(fromDb);
  @override String toSql(DateTime value) => value.toIso8601String();
}

class IsoDateConverter extends TypeConverter<DateTime, String> {
  const IsoDateConverter();
  @override DateTime fromSql(String fromDb) => DateTime.parse(fromDb);
  @override String toSql(DateTime value) =>
      value.toIso8601String().substring(0, 10);
}

// ==================== ENUM CONVERTERS ====================

class EnumConverter<T extends Enum> extends TypeConverter<T, String> {
  final List<T> values;
  final T defaultValue;
  const EnumConverter(this.values, this.defaultValue);
  @override T fromSql(String fromDb) {
    return values.firstWhere(
      (e) => e.name == fromDb,
      orElse: () => defaultValue,
    );
  }
  @override String toSql(T value) => value.name;
}

// ==================== OBJECT CONVERTERS ====================

class ObjectConverter<T> extends TypeConverter<T, String> {
  final T Function(Map<String, dynamic>) fromJson;
  final Map<String, dynamic> Function(T) toJson;
  const ObjectConverter({
    required this.fromJson,
    required this.toJson,
  });
  @override T fromSql(String fromDb) {
    if (fromDb.isEmpty) return null;
    final json = jsonDecode(fromDb) as Map<String, dynamic>;
    return fromJson(json);
  }
  @override String toSql(T value) => jsonEncode(toJson(value));
}
```

---

# Real-World Example

> **Complete e-commerce reusable converters system**

```dart
// lib/database/converters/converters.dart
export 'base_converters.dart';
export 'enum_converters.dart';
export 'list_converters.dart';
export 'json_converters.dart';
export 'date_converters.dart';
export 'object_converters.dart';

// lib/database/converters/base_converters.dart
import 'package:drift/drift.dart';
import 'dart:convert';

class JsonMapConverter extends TypeConverter<Map<String, dynamic>, String> {
  const JsonMapConverter();
  @override Map<String, dynamic> fromSql(String fromDb) {
    if (fromDb.isEmpty) return {};
    return jsonDecode(fromDb) as Map<String, dynamic>;
  }
  @override String toSql(Map<String, dynamic> value) => jsonEncode(value);
}

// lib/database/converters/list_converters.dart
class StringListConverter extends TypeConverter<List<String>, String> {
  const StringListConverter();
  @override List<String> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => e as String).toList();
  }
  @override String toSql(List<String> value) => jsonEncode(value);
}

// lib/database/converters/date_converters.dart
class IsoDateTimeConverter extends TypeConverter<DateTime, String> {
  const IsoDateTimeConverter();
  @override DateTime fromSql(String fromDb) => DateTime.parse(fromDb);
  @override String toSql(DateTime value) => value.toIso8601String();
}

// lib/database/converters/enum_converters.dart
enum UserStatus { active, inactive, pending }
enum OrderStatus { pending, processing, shipped, delivered }
enum PaymentStatus { unpaid, paid, refunded }

class EnumConverter<T extends Enum> extends TypeConverter<T, String> {
  final List<T> values;
  final T defaultValue;
  const EnumConverter(this.values, this.defaultValue);
  @override T fromSql(String fromDb) {
    return values.firstWhere(
      (e) => e.name == fromDb,
      orElse: () => defaultValue,
    );
  }
  @override String toSql(T value) => value.name;
}

// lib/database/tables/users.dart
import '../converters/converters.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();

  // 👇 Using reusable converters
  TextColumn get preferences => text()
    .withDefault(const Constant('{}'))
    .map(const JsonMapConverter())();

  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .map(const EnumConverter<UserStatus>(UserStatus.values, UserStatus.pending))();

  TextColumn get tags => text()
    .withDefault(const Constant('[]'))
    .map(const StringListConverter())();

  TextColumn get lastLogin => text()
    .nullable()
    .map(const IsoDateTimeConverter())
    .named('last_login')();

  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/tables/orders.dart
import '../converters/converters.dart';

class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().unique()();
  IntColumn get userId => integer().references(Users, #id)();
  RealColumn get total => real()();

  // 👇 Using reusable converters
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .map(const EnumConverter<OrderStatus>(OrderStatus.values, OrderStatus.pending))();

  TextColumn get paymentStatus => text()
    .withDefault(const Constant('unpaid'))
    .map(const EnumConverter<PaymentStatus>(PaymentStatus.values, PaymentStatus.unpaid))();

  TextColumn get metadata => text()
    .withDefault(const Constant('{}'))
    .map(const JsonMapConverter())();

  TextColumn get orderDate => text()
    .map(const IsoDateTimeConverter())
    .named('order_date')();

  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/database.dart
@DriftDatabase(tables: [Users, Orders])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'app_database');
  }
}
```

---

# Reusable Converter Best Practices

- **Create a central file** – `converters.dart`
- **Use const constructors** – Better performance
- **Document each converter** – Explain usage
- **Test converters thoroughly** – Verify behavior
- **Use descriptive names** – Clear purpose
- **Export converters** – Easy imports
- **Consider performance** – JSON has overhead
- **Handle edge cases** – Empty, null values

---

# Reusable Converter Checklist

| Practice | Description | Priority |
|----------|-------------|----------|
| **Central Location** | One file | High |
| **Const Constructor** | Performance | High |
| **Documentation** | Explain usage | Medium |
| **Testing** | Verify behavior | High |
| **Descriptive Names** | Clear purpose | High |
| **Export** | Easy imports | Medium |
| **Performance** | Monitor overhead | Medium |

---

# Common Mistakes

## Mistake 1: Scattered converters

Wrong:
```dart
// 🚫 Converters in multiple files
// lib/tables/users/converter.dart
// lib/tables/orders/converter.dart
```

Correct:
```dart
// ✅ Centralized
// lib/database/converters/converters.dart
// All converters in one place
```

## Mistake 2: Duplicate converter logic

Wrong:
```dart
// 🚫 Duplicate JSON map converters
class UserPreferencesConverter { ... }
class ProductMetadataConverter { ... }
```

Correct:
```dart
// ✅ Single reusable converter
class JsonMapConverter { ... }
// Used everywhere
```

## Mistake 3: Missing exports

Wrong:
```dart
// 🚫 Importing individual files
import 'converters/json_converter.dart';
import 'converters/enum_converter.dart';
```

Correct:
```dart
// ✅ Single import
import 'converters/converters.dart';
```

---

# Summary

| Type | Purpose | Example |
|------|---------|---------|
| **Base Converters** | Fundamental types | JSON, list converters |
| **Enum Converters** | Enum handling | Status, type converters |
| **Date Converters** | Date formats | ISO, Unix converters |
| **Object Converters** | Custom objects | Address, profile converters |

---

# Next Steps

Now you understand reusable converters, let's move to the next section:

- [DAO](link) – Data Access Objects

---

# Did You Know?

- **Reusable converters save time** – Write once, use everywhere

- **Centralized converters are easier to maintain** – One change affects all

- **Reusable converters ensure consistency** – Same behavior everywhere

- **Converters can be shared** – Across multiple databases

- **Converters are testable** – Write tests once

- **Converters improve onboarding** – Easier for new developers

- **Converters are essential** – For large applications

- **Converters promote DRY** – Don't repeat yourself

---

