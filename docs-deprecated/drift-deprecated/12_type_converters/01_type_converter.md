## TypeConverters

**Customizing data mapping between Dart and SQLite in Drift**

---

# What is it?

**TypeConverters** are Drift's powerful mechanism for mapping Dart types to SQLite column types and back. They allow you to store custom Dart objects, enums, lists, maps, and other complex types in your database. Drift provides a flexible `TypeConverter` interface that handles the serialization and deserialization automatically.

> **Think of TypeConverters like "universal translators"** – they convert between how you want to work with data (Dart objects) and how the database understands it (SQLite primitives).

```dart
// 👇 Custom TypeConverter for JSON data
class JsonConverter extends TypeConverter<Map<String, dynamic>, String> {
  const JsonConverter();
  
  @override
  Map<String, dynamic> fromSql(String fromDb) {
    return jsonDecode(fromDb) as Map<String, dynamic>;
  }
  
  @override
  String toSql(Map<String, dynamic> value) {
    return jsonEncode(value);
  }
}

// 👇 Using the converter in a table
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 👇 Store preferences as JSON
  TextColumn get preferences => text().map(const JsonConverter())();
}

// Now you can store Map objects directly!
final user = User(
  id: 1,
  name: 'John',
  preferences: {'theme': 'dark', 'language': 'en'},
);
```

> **What's happening here?**
> - **TypeConverter<S, T>** – S = Dart type, T = SQLite type
> - **fromSql** – Converts database value to Dart object
> - **toSql** – Converts Dart object to database value
> - **`.map()`** – Applies converter to a column

---

# Why does it exist?

- **Custom Objects** – Store complex Dart objects
- **Enums** – Store enum values as strings/integers
- **Lists** – Store lists in a single column
- **Maps** – Store key-value pairs
- **Dates** – Custom date formats
- **JSON** – Store JSON data
- **Encryption** – Automatically encrypt/decrypt

---

# Basic TypeConverters

> **Simple converter implementations**

## Enum Converter

```dart
// 👇 Define enum
enum UserStatus {
  active,
  inactive,
  pending,
  banned,
}

// 👇 Enum converter
class UserStatusConverter extends TypeConverter<UserStatus, String> {
  const UserStatusConverter();
  
  @override
  UserStatus fromSql(String fromDb) {
    return UserStatus.values.firstWhere(
      (e) => e.name == fromDb,
      orElse: () => UserStatus.pending,
    );
  }
  
  @override
  String toSql(UserStatus value) {
    return value.name;
  }
}

// 👇 Using in table
class Users extends Table {
  TextColumn get status => text().map(const UserStatusConverter())();
}
```

## DateTime Converter

```dart
// 👇 ISO 8601 date converter
class IsoDateConverter extends TypeConverter<DateTime, String> {
  const IsoDateConverter();
  
  @override
  DateTime fromSql(String fromDb) {
    return DateTime.parse(fromDb);
  }
  
  @override
  String toSql(DateTime value) {
    return value.toIso8601String();
  }
}

// 👇 Unix timestamp converter
class UnixTimestampConverter extends TypeConverter<DateTime, int> {
  const UnixTimestampConverter();
  
  @override
  DateTime fromSql(int fromDb) {
    return DateTime.fromMillisecondsSinceEpoch(fromDb * 1000);
  }
  
  @override
  int toSql(DateTime value) {
    return value.millisecondsSinceEpoch ~/ 1000;
  }
}
```

---

# Advanced TypeConverters

> **Complex converter implementations**

## JSON Converter

```dart
// 👇 Generic JSON converter
class JsonConverter<T> extends TypeConverter<T, String> {
  const JsonConverter();
  
  @override
  T fromSql(String fromDb) {
    return jsonDecode(fromDb) as T;
  }
  
  @override
  String toSql(T value) {
    return jsonEncode(value);
  }
}

// 👇 Using with different types
class Products extends Table {
  // 👇 Map type
  TextColumn get metadata => text().map(const JsonConverter<Map<String, dynamic>>())();
  
  // 👇 List type
  TextColumn get tags => text().map(const JsonConverter<List<String>>())();
}
```

## List Converter

```dart
// 👇 Comma-separated list converter
class StringListConverter extends TypeConverter<List<String>, String> {
  const StringListConverter();
  
  @override
  List<String> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    return fromDb.split(',').where((s) => s.isNotEmpty).toList();
  }
  
  @override
  String toSql(List<String> value) {
    return value.join(',');
  }
}

// 👇 JSON list converter
class JsonListConverter<T> extends TypeConverter<List<T>, String> {
  const JsonListConverter();
  
  @override
  List<T> fromSql(String fromDb) {
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => e as T).toList();
  }
  
  @override
  String toSql(List<T> value) {
    return jsonEncode(value);
  }
}
```

## Custom Object Converter

```dart
// 👇 Custom Dart class
class Address {
  final String street;
  final String city;
  final String country;
  final String zipCode;
  
  Address({
    required this.street,
    required this.city,
    required this.country,
    required this.zipCode,
  });
  
  Map<String, dynamic> toJson() => {
    'street': street,
    'city': city,
    'country': country,
    'zipCode': zipCode,
  };
  
  factory Address.fromJson(Map<String, dynamic> json) => Address(
    street: json['street'] as String,
    city: json['city'] as String,
    country: json['country'] as String,
    zipCode: json['zipCode'] as String,
  );
}

// 👇 Address converter
class AddressConverter extends TypeConverter<Address, String> {
  const AddressConverter();
  
  @override
  Address fromSql(String fromDb) {
    final json = jsonDecode(fromDb) as Map<String, dynamic>;
    return Address.fromJson(json);
  }
  
  @override
  String toSql(Address value) {
    return jsonEncode(value.toJson());
  }
}
```

---

# Real-World Example

> **Complete e-commerce type converter system**

```dart
// lib/database/converters/converters.dart
import 'package:drift/drift.dart';
import 'dart:convert';

// ==================== ENUM CONVERTERS ====================

enum OrderStatus {
  pending,
  processing,
  paid,
  shipped,
  delivered,
  cancelled,
  refunded,
}

class OrderStatusConverter extends TypeConverter<OrderStatus, String> {
  const OrderStatusConverter();
  
  @override
  OrderStatus fromSql(String fromDb) {
    return OrderStatus.values.firstWhere(
      (e) => e.name == fromDb,
      orElse: () => OrderStatus.pending,
    );
  }
  
  @override
  String toSql(OrderStatus value) {
    return value.name;
  }
}

// ==================== JSON CONVERTERS ====================

class JsonMapConverter extends TypeConverter<Map<String, dynamic>, String> {
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

class JsonListConverter extends TypeConverter<List<dynamic>, String> {
  const JsonListConverter();
  
  @override
  List<dynamic> fromSql(String fromDb) {
    return jsonDecode(fromDb) as List<dynamic>;
  }
  
  @override
  String toSql(List<dynamic> value) {
    return jsonEncode(value);
  }
}

// ==================== DATE CONVERTERS ====================

class IsoDateConverter extends TypeConverter<DateTime, String> {
  const IsoDateConverter();
  
  @override
  DateTime fromSql(String fromDb) {
    return DateTime.parse(fromDb);
  }
  
  @override
  String toSql(DateTime value) {
    return value.toIso8601String();
  }
}

// ==================== LIST CONVERTERS ====================

class StringListConverter extends TypeConverter<List<String>, String> {
  const StringListConverter();
  
  @override
  List<String> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    return fromDb.split(',').where((s) => s.isNotEmpty).toList();
  }
  
  @override
  String toSql(List<String> value) {
    return value.join(',');
  }
}

// ==================== CUSTOM OBJECT CONVERTERS ====================

class Address {
  final String street;
  final String city;
  final String state;
  final String zipCode;
  final String country;
  
  Address({
    required this.street,
    required this.city,
    required this.state,
    required this.zipCode,
    required this.country,
  });
  
  Map<String, dynamic> toJson() => {
    'street': street,
    'city': city,
    'state': state,
    'zipCode': zipCode,
    'country': country,
  };
  
  factory Address.fromJson(Map<String, dynamic> json) => Address(
    street: json['street'] as String,
    city: json['city'] as String,
    state: json['state'] as String,
    zipCode: json['zipCode'] as String,
    country: json['country'] as String,
  );
}

class AddressConverter extends TypeConverter<Address, String> {
  const AddressConverter();
  
  @override
  Address fromSql(String fromDb) {
    final json = jsonDecode(fromDb) as Map<String, dynamic>;
    return Address.fromJson(json);
  }
  
  @override
  String toSql(Address value) {
    return jsonEncode(value.toJson());
  }
}

// ==================== REUSABLE CONVERTERS ====================

// 👇 Generic converter for any object
class ObjectConverter<T> extends TypeConverter<T, String> {
  final T Function(Map<String, dynamic>) fromJson;
  final Map<String, dynamic> Function(T) toJson;
  
  const ObjectConverter({
    required this.fromJson,
    required this.toJson,
  });
  
  @override
  T fromSql(String fromDb) {
    final json = jsonDecode(fromDb) as Map<String, dynamic>;
    return fromJson(json);
  }
  
  @override
  String toSql(T value) {
    return jsonEncode(toJson(value));
  }
}

// ==================== TABLES WITH CONVERTERS ====================

// lib/database/tables/users.dart
import '../converters/converters.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get passwordHash => text().named('password_hash')();
  
  // 👇 Enum converter
  TextColumn get status => text().map(const UserStatusConverter())();
  
  // 👇 JSON converter
  TextColumn get preferences => text().map(const JsonMapConverter())();
  
  // 👇 List converter
  TextColumn get tags => text().map(const StringListConverter())();
  
  // 👇 Custom object converter
  TextColumn get address => text().map(const AddressConverter())();
  
  // 👇 Date converter
  TextColumn get birthDate => text()
    .nullable()
    .map(const IsoDateConverter())
    .named('birth_date')();
  
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}

// lib/database/tables/products.dart
import '../converters/converters.dart';

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get sku => text().unique()();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  RealColumn get price => real()();
  
  // 👇 JSON metadata
  TextColumn get metadata => text()
    .map(const JsonMapConverter())
    .withDefault(const Constant('{}'))();
  
  // 👇 JSON tags
  TextColumn get tags => text()
    .map(const JsonListConverter())
    .withDefault(const Constant('[]'))();
  
  // 👇 Custom attributes with object converter
  TextColumn get attributes => text().map(
    const ObjectConverter<Map<String, dynamic>>(
      fromJson: (json) => json,
      toJson: (obj) => obj,
    ),
  )();
  
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/database.dart
@DriftDatabase(tables: [Users, Products])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'app_database');
  }
}
```

```dart
// lib/ui/pages/user_profile_page.dart
class UserProfilePage extends StatelessWidget {
  final AppDatabase db;
  final int userId;
  
  const UserProfilePage({required this.db, required this.userId});
  
  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: _loadUser(),
      builder: (context, snapshot) {
        if (!snapshot.hasData) return CircularProgressIndicator();
        
        final user = snapshot.data!;
        
        return Scaffold(
          appBar: AppBar(title: Text(user.username)),
          body: Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // Basic info
                Text('Email: ${user.email}'),
                Text('Status: ${user.status.name}'),
                
                // Address
                if (user.address != null) ...[
                  SizedBox(height: 16),
                  Text('Address:'),
                  Text(user.address!.street),
                  Text('${user.address!.city}, ${user.address!.state}'),
                  Text('${user.address!.zipCode}, ${user.address!.country}'),
                ],
                
                // Preferences (Map)
                if (user.preferences.isNotEmpty) ...[
                  SizedBox(height: 16),
                  Text('Preferences:'),
                  ...user.preferences.entries.map((entry) {
                    return Text('${entry.key}: ${entry.value}');
                  }).toList(),
                ],
                
                // Tags (List)
                if (user.tags.isNotEmpty) ...[
                  SizedBox(height: 16),
                  Text('Tags:'),
                  Wrap(
                    spacing: 8,
                    children: user.tags.map((tag) {
                      return Chip(label: Text(tag));
                    }).toList(),
                  ),
                ],
              ],
            ),
          ),
        );
      },
    );
  }
  
  Future<User> _loadUser() async {
    return await (db.select(db.users)
      ..where((u) => u.id.equals(userId)))
      .getSingle();
  }
}
```

---

# TypeConverter Best Practices

- **Use const constructors** – For better performance
- **Handle null values** – Check for null in `fromSql`
- **Validate data** – Ensure data is valid
- **Document converters** – Explain serialization
- **Test converters** – Verify serialization/deserialization
- **Reuse converters** – Share across tables
- **Use JSON for complex objects** – Flexible storage
- **Consider performance** – JSON has overhead

---

# TypeConverter Checklist

| Practice | Description | Priority |
|----------|-------------|----------|
| **Const Constructor** | Performance | High |
| **Null Handling** | Prevent errors | High |
| **Validation** | Data integrity | High |
| **Documentation** | Explain usage | Medium |
| **Testing** | Verify behavior | High |
| **Reuse** | Share converters | Medium |
| **JSON** | Complex objects | Medium |
| **Performance** | Monitor overhead | Medium |

---

# Common Mistakes

## Mistake 1: Not handling null

Wrong:
```dart
// 🚫 Null throws error
@override
T fromSql(String fromDb) {
  return jsonDecode(fromDb) as T; // Null error
}
```

Correct:
```dart
// ✅ Handle null
@override
T fromSql(String fromDb) {
  if (fromDb == null) return defaultValue;
  return jsonDecode(fromDb) as T;
}
```

## Mistake 2: Using wrong type

Wrong:
```dart
// 🚫 Type mismatch
TextColumn get preferences => text().map(const JsonMapConverter())();
// Map<String, dynamic> stored as text
```

Correct:
```dart
// ✅ Correct type
TextColumn get preferences => text().map(const JsonMapConverter())();
// Works correctly because JSON is stored as text
```

## Mistake 3: Missing import

Wrong:
```dart
// 🚫 Converter not imported
class Users extends Table {
  TextColumn get status => text().map(const UserStatusConverter())();
}
```

Correct:
```dart
// ✅ Import converter
import 'converters/converters.dart';
class Users extends Table {
  TextColumn get status => text().map(const UserStatusConverter())();
}
```

---

# Summary

| Converter Type | Use Case | Example |
|----------------|----------|---------|
| **Enum** | Store enums | `UserStatus.active` |
| **DateTime** | Custom date format | `DateTime` to ISO string |
| **JSON** | Complex objects | `Map<String, dynamic>` |
| **List** | Lists | `List<String>` |
| **Custom** | Custom objects | `Address` object |

---

# Next Steps

Now you understand type converters, let's dive deeper:

- [Enum Conversion](link) – Advanced enum handling
- [DateTime Conversion](link) – Date/time strategies
- [Custom Objects](link) – Complex object mapping

---

# Did You Know?

- **TypeConverters are powerful** – Store any Dart object

- **TypeConverters are reusable** – Share across tables

- **TypeConverters are safe** – Type-checked conversions

- **JSON is flexible** – Store any structure

- **Enums are type-safe** – Use enum converters

- **Lists are common** – Store multiple values

- **Dates need conversion** – Custom formats

- **TypeConverters are essential** – For real-world apps

---

