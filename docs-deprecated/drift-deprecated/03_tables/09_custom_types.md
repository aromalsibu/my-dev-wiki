## Custom Types

**Extending Drift with custom type converters**

---

# What is it?

**Custom Types** in Drift allow you to store complex Dart objects in SQLite columns by converting them to and from primitive database types (TEXT, INTEGER, BLOB, etc.). This is achieved through `TypeConverter` classes that handle the serialization and deserialization logic.

> **Think of Custom Types like "translators"** – they convert between your Dart objects and what SQLite can understand, allowing you to store complex data like JSON objects, enums, or custom classes in a simple text column.

```dart
// 👇 Custom converter for JSON data
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
  
  // 👇 Store Map<String, dynamic> as JSON in TEXT column
  TextColumn get preferences => text().map(const JsonConverter())();
}

// Now you can store and retrieve Map objects directly!
final user = User(
  id: 1,
  name: 'John',
  preferences: {'theme': 'dark', 'language': 'en'}, // 👈 Map
);
```

> **What's happening here?**
> - **TypeConverter<S, T>** – S = Dart type, T = SQLite type
> - **fromSql** – Converts from database to Dart
> - **toSql** – Converts from Dart to database
> - **`.map()`** – Applies the converter to a column

---

# Why does it exist?

- **Complex Data** – Store objects, lists, maps in a single column
- **Enum Support** – Store enums as strings or integers
- **Custom Objects** – Convert custom Dart classes to database types
- **Encryption** – Encrypt/decrypt data automatically
- **Serialization** – Handle JSON, Protocol Buffers, etc.
- **Code Reuse** – Share converters across tables

---

# Basic TypeConverters

> **Simple converters for common use cases**

## JSON Converter

```dart
import 'dart:convert';

// 👇 Store any JSON-serializable object
class JsonConverter<T extends Object> extends TypeConverter<T, String> {
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

// Usage
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 👇 Store Map<String, dynamic>
  TextColumn get metadata => text().map(const JsonConverter<Map<String, dynamic>>())();
  
  // 👇 Store List<dynamic>
  TextColumn get tags => text().map(const JsonConverter<List<String>>())();
}

// Generated SQL:
// CREATE TABLE products (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   name TEXT NOT NULL,
//   metadata TEXT NOT NULL,
//   tags TEXT NOT NULL
// )
```

---

## Enum Converter (String-based)

```dart
// 👇 Define enum
enum Status {
  active,
  inactive,
  pending,
  deleted,
}

// 👇 Convert enum to/from string
class StatusConverter extends TypeConverter<Status, String> {
  const StatusConverter();
  
  @override
  Status fromSql(String fromDb) {
    return Status.values.firstWhere(
      (e) => e.name == fromDb,
      orElse: () => Status.pending,
    );
  }
  
  @override
  String toSql(Status value) {
    return value.name;
  }
}

// Usage
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text()();
  
  // 👇 Store enum as TEXT
  TextColumn get status => text().map(const StatusConverter())();
}
```

---

## Enum Converter (Integer-based)

```dart
// 👇 Define enum with integer values
enum Priority {
  low(0),
  medium(1),
  high(2),
  critical(3);
  
  final int value;
  const Priority(this.value);
  
  static Priority fromValue(int value) {
    return Priority.values.firstWhere(
      (e) => e.value == value,
      orElse: () => Priority.medium,
    );
  }
}

// 👇 Convert enum to/from integer
class PriorityConverter extends TypeConverter<Priority, int> {
  const PriorityConverter();
  
  @override
  Priority fromSql(int fromDb) {
    return Priority.fromValue(fromDb);
  }
  
  @override
  int toSql(Priority value) {
    return value.value;
  }
}

// Usage
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  
  // 👇 Store enum as INTEGER
  IntColumn get priority => integer().map(const PriorityConverter())();
}
```

---

## DateTime Converters

```dart
// 👇 Custom date format (e.g., ISO 8601)
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

// 👇 Date only (no time)
class DateOnlyConverter extends TypeConverter<DateTime, String> {
  const DateOnlyConverter();
  
  @override
  DateTime fromSql(String fromDb) {
    return DateTime.parse(fromDb);
  }
  
  @override
  String toSql(DateTime value) {
    return value.toIso8601String().substring(0, 10);
  }
}

// 👇 Unix timestamp (seconds since epoch)
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

// Usage
class Events extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 👇 Custom date format
  TextColumn get eventDate => text().map(const IsoDateConverter())();
  
  // 👇 Date only
  TextColumn get birthDate => text().map(const DateOnlyConverter())();
  
  // 👇 Unix timestamp
  IntColumn get createdAt => integer().map(const UnixTimestampConverter())();
}
```

---

# Complex TypeConverters

> **Converting complex objects**

## Custom Object Converter

```dart
// 👇 Custom Dart class
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
  
  // 👇 To JSON
  Map<String, dynamic> toJson() => {
    'street': street,
    'city': city,
    'state': state,
    'zipCode': zipCode,
    'country': country,
  };
  
  // 👇 From JSON
  factory Address.fromJson(Map<String, dynamic> json) => Address(
    street: json['street'] as String,
    city: json['city'] as String,
    state: json['state'] as String,
    zipCode: json['zipCode'] as String,
    country: json['country'] as String,
  );
}

// 👇 Converter
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

// Usage
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 👇 Store Address object as JSON
  TextColumn get address => text().map(const AddressConverter())();
}
```

---

## List Converter

```dart
// 👇 Store list of strings as comma-separated text
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

// 👇 Store list of integers as JSON
class IntListConverter extends TypeConverter<List<int>, String> {
  const IntListConverter();
  
  @override
  List<int> fromSql(String fromDb) {
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => e as int).toList();
  }
  
  @override
  String toSql(List<int> value) {
    return jsonEncode(value);
  }
}

// 👇 Store list of custom objects
class ProductListConverter extends TypeConverter<List<Product>, String> {
  const ProductListConverter();
  
  @override
  List<Product> fromSql(String fromDb) {
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => Product.fromJson(e as Map<String, dynamic>)).toList();
  }
  
  @override
  String toSql(List<Product> value) {
    return jsonEncode(value.map((p) => p.toJson()).toList());
  }
}

// Usage
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text()();
  
  // 👇 List of strings
  TextColumn get tags => text().map(const StringListConverter())();
  
  // 👇 List of integers
  TextColumn get productIds => text().map(const IntListConverter())();
  
  // 👇 List of custom objects
  TextColumn get products => text().map(const ProductListConverter())();
}
```

---

## Encrypted Converter

```dart
import 'dart:convert';
import 'package:encrypt/encrypt.dart' as encrypt;

// 👇 Encrypt/decrypt data automatically
class EncryptedStringConverter extends TypeConverter<String, String> {
  final encrypt.Key key;
  final encrypt.IV iv;
  final encrypt.Encrypter encrypter;
  
  EncryptedStringConverter({
    required String keyString,
    required String ivString,
  }) : key = encrypt.Key.fromUtf8(keyString),
       iv = encrypt.IV.fromUtf8(ivString),
       encrypter = encrypt.Encrypter(encrypt.AES(encrypt.Key.fromUtf8(keyString)));
  
  @override
  String fromSql(String fromDb) {
    try {
      final encrypted = encrypt.Encrypted.fromBase64(fromDb);
      return encrypter.decrypt(encrypted, iv: iv);
    } catch (e) {
      // Handle decryption errors
      return fromDb;
    }
  }
  
  @override
  String toSql(String value) {
    final encrypted = encrypter.encrypt(value, iv: iv);
    return encrypted.base64;
  }
}

// 👇 Encrypt JSON data
class EncryptedJsonConverter extends TypeConverter<Map<String, dynamic>, String> {
  final EncryptedStringConverter stringConverter;
  
  const EncryptedJsonConverter(this.stringConverter);
  
  @override
  Map<String, dynamic> fromSql(String fromDb) {
    final decrypted = stringConverter.fromSql(fromDb);
    return jsonDecode(decrypted) as Map<String, dynamic>;
  }
  
  @override
  String toSql(Map<String, dynamic> value) {
    final json = jsonEncode(value);
    return stringConverter.toSql(json);
  }
}

// Usage
class SecureData extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Encrypted text
  TextColumn get secretData => text().map(
    EncryptedStringConverter(
      keyString: 'my-secret-key-123',
      ivString: 'my-iv-12345678',
    ),
  )();
  
  // 👇 Encrypted JSON
  TextColumn get secureConfig => text().map(
    EncryptedJsonConverter(
      EncryptedStringConverter(
        keyString: 'my-secret-key-123',
        ivString: 'my-iv-12345678',
      ),
    ),
  )();
}
```

---

# Reusable Converters

> **Creating converters you can reuse across your app**

```dart
// lib/database/converters/converters.dart
import 'package:drift/drift.dart';
import 'dart:convert';

// 1️⃣ JSON Converter
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

// 2️⃣ JSON List Converter
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

// 3️⃣ Enum Converter (generic)
class EnumConverter<T extends Enum> extends TypeConverter<T, String> {
  final List<T> values;
  final T defaultValue;
  
  const EnumConverter(this.values, this.defaultValue);
  
  @override
  T fromSql(String fromDb) {
    return values.cast<T>().firstWhere(
      (e) => e.name == fromDb,
      orElse: () => defaultValue,
    );
  }
  
  @override
  String toSql(T value) {
    return value.name;
  }
}

// 4️⃣ DateTime ISO Converter
class DateTimeIsoConverter extends TypeConverter<DateTime, String> {
  const DateTimeIsoConverter();
  
  @override
  DateTime fromSql(String fromDb) {
    return DateTime.parse(fromDb);
  }
  
  @override
  String toSql(DateTime value) {
    return value.toIso8601String();
  }
}

// 5️⃣ DateTime Unix Converter
class DateTimeUnixConverter extends TypeConverter<DateTime, int> {
  const DateTimeUnixConverter();
  
  @override
  DateTime fromSql(int fromDb) {
    return DateTime.fromMillisecondsSinceEpoch(fromDb * 1000);
  }
  
  @override
  int toSql(DateTime value) {
    return value.millisecondsSinceEpoch ~/ 1000;
  }
}

// 6️⃣ String List Converter
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

// 7️⃣ Int List Converter
class IntListConverter extends TypeConverter<List<int>, String> {
  const IntListConverter();
  
  @override
  List<int> fromSql(String fromDb) {
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => e as int).toList();
  }
  
  @override
  String toSql(List<int> value) {
    return jsonEncode(value);
  }
}

// 8️⃣ Custom Object Converter (generic)
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

// lib/database/tables/users.dart
import 'package:drift/drift.dart';
import '../converters/converters.dart';

// 👇 Define enum
enum UserStatus { active, inactive, pending, banned }

// 👇 Custom Address class
class Address {
  final String street;
  final String city;
  final String country;
  
  Address({required this.street, required this.city, required this.country});
  
  Map<String, dynamic> toJson() => {
    'street': street,
    'city': city,
    'country': country,
  };
  
  factory Address.fromJson(Map<String, dynamic> json) => Address(
    street: json['street'] as String,
    city: json['city'] as String,
    country: json['country'] as String,
  );
}

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  
  // 👇 Reusable JSON converter
  TextColumn get preferences => text().map(const JsonMapConverter())();
  
  // 👇 Reusable enum converter
  TextColumn get status => text().map(
    const EnumConverter<UserStatus>(UserStatus.values, UserStatus.pending),
  )();
  
  // 👇 Reusable object converter
  TextColumn get address => text().map(
    const ObjectConverter<Address>(
      fromJson: Address.fromJson,
      toJson: (a) => a.toJson(),
    ),
  )();
  
  // 👇 Reusable list converter
  TextColumn get tags => text().map(const StringListConverter())();
  
  // 👇 Reusable datetime converter
  TextColumn get birthDate => text().map(const DateTimeIsoConverter())();
}
```

---

# Real-World Example

> **Complete e-commerce data with custom types**

```dart
// lib/database/converters/converters.dart
import 'package:drift/drift.dart';
import 'dart:convert';

// 👇 Product Variant
class ProductVariant {
  final String sku;
  final String size;
  final String color;
  final double price;
  final int stock;
  
  ProductVariant({
    required this.sku,
    required this.size,
    required this.color,
    required this.price,
    required this.stock,
  });
  
  Map<String, dynamic> toJson() => {
    'sku': sku,
    'size': size,
    'color': color,
    'price': price,
    'stock': stock,
  };
  
  factory ProductVariant.fromJson(Map<String, dynamic> json) => ProductVariant(
    sku: json['sku'] as String,
    size: json['size'] as String,
    color: json['color'] as String,
    price: json['price'] as double,
    stock: json['stock'] as int,
  );
}

// 👇 Product Variant List Converter
class ProductVariantListConverter extends TypeConverter<List<ProductVariant>, String> {
  const ProductVariantListConverter();
  
  @override
  List<ProductVariant> fromSql(String fromDb) {
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => ProductVariant.fromJson(e as Map<String, dynamic>)).toList();
  }
  
  @override
  String toSql(List<ProductVariant> value) {
    return jsonEncode(value.map((v) => v.toJson()).toList());
  }
}

// 👇 Review
class Review {
  final int userId;
  final String userName;
  final int rating;
  final String? comment;
  final DateTime date;
  
  Review({
    required this.userId,
    required this.userName,
    required this.rating,
    this.comment,
    required this.date,
  });
  
  Map<String, dynamic> toJson() => {
    'userId': userId,
    'userName': userName,
    'rating': rating,
    'comment': comment,
    'date': date.toIso8601String(),
  };
  
  factory Review.fromJson(Map<String, dynamic> json) => Review(
    userId: json['userId'] as int,
    userName: json['userName'] as String,
    rating: json['rating'] as int,
    comment: json['comment'] as String?,
    date: DateTime.parse(json['date'] as String),
  );
}

// 👇 Review List Converter
class ReviewListConverter extends TypeConverter<List<Review>, String> {
  const ReviewListConverter();
  
  @override
  List<Review> fromSql(String fromDb) {
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => Review.fromJson(e as Map<String, dynamic>)).toList();
  }
  
  @override
  String toSql(List<Review> value) {
    return jsonEncode(value.map((r) => r.toJson()).toList());
  }
}

// 👇 Order Status Enum
enum OrderStatus {
  pending,
  processing,
  shipped,
  delivered,
  cancelled,
  refunded,
}

// 👇 Enum Converter
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

// 👇 Shipping Address
class ShippingAddress {
  final String street;
  final String city;
  final String state;
  final String zipCode;
  final String country;
  final String? phone;
  
  ShippingAddress({
    required this.street,
    required this.city,
    required this.state,
    required this.zipCode,
    required this.country,
    this.phone,
  });
  
  Map<String, dynamic> toJson() => {
    'street': street,
    'city': city,
    'state': state,
    'zipCode': zipCode,
    'country': country,
    'phone': phone,
  };
  
  factory ShippingAddress.fromJson(Map<String, dynamic> json) => ShippingAddress(
    street: json['street'] as String,
    city: json['city'] as String,
    state: json['state'] as String,
    zipCode: json['zipCode'] as String,
    country: json['country'] as String,
    phone: json['phone'] as String?,
  );
}

// 👇 Address Converter
class ShippingAddressConverter extends TypeConverter<ShippingAddress, String> {
  const ShippingAddressConverter();
  
  @override
  ShippingAddress fromSql(String fromDb) {
    final json = jsonDecode(fromDb) as Map<String, dynamic>;
    return ShippingAddress.fromJson(json);
  }
  
  @override
  String toSql(ShippingAddress value) {
    return jsonEncode(value.toJson());
  }
}

// 👇 Payment Info
class PaymentInfo {
  final String method;
  final String status;
  final double amount;
  final DateTime date;
  final String? transactionId;
  
  PaymentInfo({
    required this.method,
    required this.status,
    required this.amount,
    required this.date,
    this.transactionId,
  });
  
  Map<String, dynamic> toJson() => {
    'method': method,
    'status': status,
    'amount': amount,
    'date': date.toIso8601String(),
    'transactionId': transactionId,
  };
  
  factory PaymentInfo.fromJson(Map<String, dynamic> json) => PaymentInfo(
    method: json['method'] as String,
    status: json['status'] as String,
    amount: json['amount'] as double,
    date: DateTime.parse(json['date'] as String),
    transactionId: json['transactionId'] as String?,
  );
}

// 👇 Payment Converter
class PaymentInfoConverter extends TypeConverter<PaymentInfo, String> {
  const PaymentInfoConverter();
  
  @override
  PaymentInfo fromSql(String fromDb) {
    final json = jsonDecode(fromDb) as Map<String, dynamic>;
    return PaymentInfo.fromJson(json);
  }
  
  @override
  String toSql(PaymentInfo value) {
    return jsonEncode(value.toJson());
  }
}

// lib/database/tables/products.dart
import 'package:drift/drift.dart';
import '../converters/converters.dart';

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get sku => text().unique()();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  
  // 👇 Store variants as JSON
  TextColumn get variants => text().map(const ProductVariantListConverter())();
  
  // 👇 Store reviews as JSON
  TextColumn get reviews => text()
    .withDefault(const Constant('[]'))
    .map(const ReviewListConverter())();
  
  // 👇 Store tags as list
  TextColumn get tags => text()
    .withDefault(const Constant(''))
    .map(const StringListConverter())();
  
  // 👇 Store metadata as JSON
  TextColumn get metadata => text()
    .withDefault(const Constant('{}'))
    .map(const JsonMapConverter())();
  
  // 👇 Store rating
  RealColumn get averageRating => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (average_rating >= 0 AND average_rating <= 5)')
    .named('average_rating')();
  
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))
    .named('is_active')();
  
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
  
  @override
  List<Index> get indexes => [
    Index('idx_products_sku', 'sku'),
    Index('idx_products_is_active', 'is_active'),
  ];
}

// lib/database/tables/orders.dart
import 'package:drift/drift.dart';
import '../converters/converters.dart';

class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().unique()();
  IntColumn get userId => integer().references(Users, #id)();
  
  // 👇 Store status as enum
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .map(const OrderStatusConverter())();
  
  // 👇 Store shipping address
  TextColumn get shippingAddress => text().map(const ShippingAddressConverter())();
  
  // 👇 Store payment info
  TextColumn get paymentInfo => text().map(const PaymentInfoConverter())();
  
  // 👇 Store items as JSON
  TextColumn get items => text().map(const JsonListConverter())();
  
  // 👇 Store timestamps
  DateTimeColumn get orderDate => dateTime()
    .withDefault(currentDateAndTime)
    .named('order_date')();
  
  DateTimeColumn get shippedDate => dateTime()
    .nullable()
    .named('shipped_date')();
  
  DateTimeColumn get deliveredDate => dateTime()
    .nullable()
    .named('delivered_date')();
  
  // 👇 Store totals
  RealColumn get subtotal => real()
    .customConstraint('CHECK (subtotal >= 0)')();
  
  RealColumn get tax => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (tax >= 0)')();
  
  RealColumn get shippingCost => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (shipping_cost >= 0)')
    .named('shipping_cost')();
  
  RealColumn get total => real()
    .customConstraint('CHECK (total >= 0)')();
  
  // 👇 Flags
  BoolColumn get isPaid => boolean()
    .withDefault(const Constant(false))
    .named('is_paid')();
  
  BoolColumn get isShipped => boolean()
    .withDefault(const Constant(false))
    .named('is_shipped')();
  
  BoolColumn get isDelivered => boolean()
    .withDefault(const Constant(false))
    .named('is_delivered')();
  
  @override
  List<Index> get indexes => [
    Index('idx_orders_user', 'user_id'),
    Index('idx_orders_status', 'status'),
    Index('idx_orders_order_date', 'order_date'),
  ];
}
```

---

# Best Practices

- **Use descriptive converter names** – `JsonMapConverter`, `StatusEnumConverter`
- **Keep converters in one place** – `lib/database/converters/`
- **Handle null values** – Check for null in `fromSql`
- **Handle errors gracefully** – Provide fallback values
- **Use const constructors** – For better performance
- **Document converters** – Explain serialization format
- **Test converters** – Verify serialization/deserialization
- **Use reusable converters** – Share across tables
- **Consider performance** – JSON serialization has overhead
- **Consider migration** – Changing converter format requires migrations

---

# Common Mistakes

## Mistake 1: Not handling null values

Wrong:
```dart
// 🚫 Crashes on null
@override
T fromSql(String fromDb) {
  return jsonDecode(fromDb) as T; // Null throws
}
```

Correct:
```dart
// ✅ Handle null
@override
T fromSql(String fromDb) {
  if (fromDb.isEmpty) return defaultValue;
  return jsonDecode(fromDb) as T;
}
```

## Mistake 2: Storing large objects

Wrong:
```dart
// 🚫 Storing 10MB JSON in a column
class Products extends Table {
  TextColumn get metadata => text().map(const JsonMapConverter())();
}
```

Correct:
```dart
// ✅ Consider separate table for large data
class ProductMetadata extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get productId => integer().references(Products, #id)();
  TextColumn get metadata => text().map(const JsonMapConverter())();
}
```

## Mistake 3: Missing TypeConverter in imports

Wrong:
```dart
// 🚫 TypeConverter not imported
class Products extends Table {
  // Error: JsonMapConverter not found
  TextColumn get metadata => text().map(const JsonMapConverter())();
}
```

Correct:
```dart
// ✅ Import converter
import '../converters/converters.dart';

class Products extends Table {
  TextColumn get metadata => text().map(const JsonMapConverter())();
}
```

---

# Summary

| Converter Type | Purpose | Example |
|----------------|---------|---------|
| **JSON** | Store maps/objects | `JsonMapConverter` |
| **Enum** | Store enum values | `EnumConverter<Status>` |
| **List** | Store lists | `StringListConverter` |
| **Object** | Store custom classes | `AddressConverter` |
| **Encrypted** | Secure data | `EncryptedStringConverter` |
| **DateTime** | Custom date formats | `IsoDateConverter` |

---

# Next Steps

Now you understand custom types, let's dive deeper:

- [Table Inheritance](link) – Reusing table definitions
- [Indexes](link) – Advanced indexing strategies
- [Type Converters Advanced](link) – Advanced converter patterns

---

# Did You Know?

- **TypeConverters apply globally** – Throughout your app

- **JSON in SQLite is TEXT** – With no native JSON support

- **Custom types can be nested** – Lists of objects with converters

- **Enums are type-safe** – When using enum converters

- **Encrypted converters** – Can handle encryption automatically

- **Converters are reusable** – Across multiple tables

- **JSON serialization has overhead** – Use for infrequent queries

- **Custom types can be indexed** – But only the underlying SQL type

---

## 🎯 **That's Topic #20: Custom Types!**

---

**Ready for Topic #21: Table Inheritance?** Say "**NEXT**" and let's keep this ELITE knowledge flowing! 🔥