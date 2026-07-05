## JSON Conversion

**Storing and retrieving JSON data in Drift**

---

# What is it?

**JSON Conversion** is the process of storing Dart objects, maps, and lists as JSON strings in your SQLite database. This allows you to store complex, nested data structures in a single column without creating multiple tables. Drift's `TypeConverter` handles the serialization and deserialization automatically.

> **Think of JSON Conversion like "packing for a trip"** – you take all your items (Dart objects), pack them into a single suitcase (JSON string), and unpack them when you arrive (back to Dart objects).

```dart
// 👇 JSON converter for maps
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

// 👇 Using in a table
class Users extends Table {
  TextColumn get preferences => text().map(const JsonMapConverter())();
}

// Now you can store Map objects directly!
final user = User(
  id: 1,
  name: 'John',
  preferences: {'theme': 'dark', 'language': 'en', 'notifications': true},
);
```

> **What's happening here?**
> - **`toSql`** – Converts Dart object to JSON string
> - **`fromSql`** – Converts JSON string back to Dart object
> - **Type safety** – Database operations use typed objects
> - **Flexibility** – Store any JSON-serializable data

---

# Why does it exist?

- **Store Complex Data** – Lists, maps, nested objects
- **Flexible Schema** – No need for separate tables
- **Performance** – Single column vs multiple joins
- **API Integration** – Store API responses directly
- **Configuration** – User preferences, settings
- **Dynamic Data** – Store dynamic structures

---

# Basic JSON Conversion

> **Simple JSON converters**

## Map Converter

```dart
// 👇 Map converter
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

// 👇 Using in table
class Settings extends Table {
  TextColumn get preferences => text()
    .withDefault(const Constant('{}'))
    .map(const JsonMapConverter())();
}
```

## List Converter

```dart
// 👇 List converter
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

// 👇 Typed list converter
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
```

---

# Advanced JSON Conversion

> **Complex JSON converters**

## Generic JSON Converter

```dart
// 👇 Generic converter for any type
class JsonConverter<T> extends TypeConverter<T, String> {
  final T Function(Map<String, dynamic>) fromJson;
  final Map<String, dynamic> Function(T) toJson;
  
  const JsonConverter({
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

// 👇 Custom Dart class
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

// 👇 Using generic converter
class Users extends Table {
  TextColumn get address => text().map(
    const JsonConverter<Address>(
      fromJson: Address.fromJson,
      toJson: (a) => a.toJson(),
    ),
  )();
}
```

## Nested JSON Converter

```dart
// 👇 Nested JSON structure
class ProductVariant {
  final String sku;
  final String size;
  final String color;
  final double price;
  
  ProductVariant({
    required this.sku,
    required this.size,
    required this.color,
    required this.price,
  });
  
  Map<String, dynamic> toJson() => {
    'sku': sku,
    'size': size,
    'color': color,
    'price': price,
  };
  
  factory ProductVariant.fromJson(Map<String, dynamic> json) => ProductVariant(
    sku: json['sku'] as String,
    size: json['size'] as String,
    color: json['color'] as String,
    price: json['price'] as double,
  );
}

class ProductVariantsConverter extends TypeConverter<List<ProductVariant>, String> {
  const ProductVariantsConverter();
  
  @override
  List<ProductVariant> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => ProductVariant.fromJson(e as Map<String, dynamic>)).toList();
  }
  
  @override
  String toSql(List<ProductVariant> value) {
    return jsonEncode(value.map((v) => v.toJson()).toList());
  }
}
```

---

# Real-World Example

> **Complete e-commerce JSON conversion system**

```dart
// lib/database/converters/json_converters.dart
import 'package:drift/drift.dart';
import 'dart:convert';

// 👇 Map converter
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

// 👇 String list converter
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

// 👇 Custom object converter
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
    if (fromDb.isEmpty) return null;
    final json = jsonDecode(fromDb) as Map<String, dynamic>;
    return Address.fromJson(json);
  }
  
  @override
  String toSql(Address value) {
    return jsonEncode(value.toJson());
  }
}

// 👇 Metadata converter (nested JSON)
class MetadataConverter extends TypeConverter<Map<String, dynamic>, String> {
  const MetadataConverter();
  
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

// lib/database/tables/products.dart
import '../converters/json_converters.dart';

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get sku => text().unique()();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  RealColumn get price => real()();
  IntColumn get stock => integer()();

  // 👇 JSON metadata
  TextColumn get metadata => text()
    .withDefault(const Constant('{}'))
    .map(const MetadataConverter())();

  // 👇 JSON tags
  TextColumn get tags => text()
    .withDefault(const Constant('[]'))
    .map(const StringListConverter())();

  // 👇 Address (custom object)
  TextColumn get warehouseAddress => text()
    .nullable()
    .map(const AddressConverter())
    .named('warehouse_address')();

  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/tables/users.dart
import '../converters/json_converters.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();

  // 👇 Preferences (Map)
  TextColumn get preferences => text()
    .withDefault(const Constant('{}'))
    .map(const JsonMapConverter())();

  // 👇 Address (custom object)
  TextColumn get address => text()
    .nullable()
    .map(const AddressConverter())();

  // 👇 Tags (List)
  TextColumn get interests => text()
    .withDefault(const Constant('[]'))
    .map(const StringListConverter())();

  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}
```

```dart
// lib/ui/pages/product_detail_page.dart
class ProductDetailPage extends StatelessWidget {
  final AppDatabase db;
  final int productId;

  const ProductDetailPage({required this.db, required this.productId});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: _loadProduct(),
      builder: (context, snapshot) {
        if (!snapshot.hasData) return CircularProgressIndicator();

        final product = snapshot.data!;

        return Scaffold(
          appBar: AppBar(title: Text(product.name)),
          body: Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('SKU: ${product.sku}'),
                Text('Price: \$${product.price}'),
                Text('Stock: ${product.stock}'),

                // 👇 Metadata (Map)
                if (product.metadata.isNotEmpty) ...[
                  SizedBox(height: 16),
                  Text('Details:', style: TextStyle(fontWeight: FontWeight.bold)),
                  ...product.metadata.entries.map((entry) {
                    return Text('${entry.key}: ${entry.value}');
                  }).toList(),
                ],

                // 👇 Tags (List)
                if (product.tags.isNotEmpty) ...[
                  SizedBox(height: 16),
                  Text('Tags:', style: TextStyle(fontWeight: FontWeight.bold)),
                  Wrap(
                    spacing: 8,
                    children: product.tags.map((tag) {
                      return Chip(label: Text(tag));
                    }).toList(),
                  ),
                ],

                // 👇 Address (Custom object)
                if (product.warehouseAddress != null) ...[
                  SizedBox(height: 16),
                  Text('Warehouse:', style: TextStyle(fontWeight: FontWeight.bold)),
                  Text(product.warehouseAddress!.street),
                  Text('${product.warehouseAddress!.city}, ${product.warehouseAddress!.state}'),
                  Text('${product.warehouseAddress!.zipCode}, ${product.warehouseAddress!.country}'),
                ],
              ],
            ),
          ),
        );
      },
    );
  }

  Future<Product> _loadProduct() async {
    return await (db.select(db.products)
      ..where((p) => p.id.equals(productId)))
      .getSingle();
  }
}
```

---

# JSON Conversion Best Practices

- **Use JSON for complex data** – Maps, lists, nested objects
- **Handle empty values** – Default to `{}` or `[]`
- **Validate data** – Ensure JSON is valid
- **Use const constructors** – Better performance
- **Document structure** – Explain JSON schema
- **Test conversion** – Verify serialization
- **Consider performance** – JSON has overhead
- **Use typed converters** – Type-safe access

---

# JSON Conversion Checklist

| Practice | Description | Priority |
|----------|-------------|----------|
| **Default Values** | Handle empty | High |
| **Validation** | Ensure valid JSON | High |
| **Const Constructor** | Performance | High |
| **Documentation** | Explain schema | Medium |
| **Testing** | Verify conversion | High |
| **Performance** | Monitor overhead | Medium |
| **Type Safety** | Use typed converters | High |

---

# Common Mistakes

## Mistake 1: Not handling empty values

Wrong:
```dart
// 🚫 Empty string causes error
@override
Map fromSql(String fromDb) {
  return jsonDecode(fromDb);
}
```

Correct:
```dart
// ✅ Handle empty
@override
Map fromSql(String fromDb) {
  if (fromDb.isEmpty) return {};
  return jsonDecode(fromDb);
}
```

## Mistake 2: Incorrect type mapping

Wrong:
```dart
// 🚫 Type mismatch
List<String> tags = jsonDecode(tagsJson); // Dynamic list
```

Correct:
```dart
// ✅ Correct type casting
final list = jsonDecode(tagsJson) as List<dynamic>;
return list.map((e) => e as String).toList();
```

## Mistake 3: Storing large JSON

Wrong:
```dart
// 🚫 Storing 10MB JSON in a column
TextColumn get largeData => text().map(const JsonMapConverter())();
```

Correct:
```dart
// ✅ Separate table for large data
class LargeDataTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get data => text().map(const JsonMapConverter())();
}
```

---

# Summary

| Converter | Use Case | Example |
|-----------|----------|---------|
| **Map** | Key-value pairs | Preferences, settings |
| **List** | Arrays | Tags, categories |
| **Custom Object** | Structured data | Address, variants |
| **Generic** | Any type | Reusable converters |

---

# Next Steps

Now you understand JSON conversion, let's dive deeper:

- [Custom Objects](link) – Complex object mapping
- [Reusable Converters](link) – Sharing converters

---

# Did You Know?

- **JSON is flexible** – Store any structure

- **JSON is readable** – Human-readable format

- **JSON is compact** – When minified

- **JSON is standard** – Universal format

- **JSON can be nested** – Deep structures

- **JSON has overhead** – Parsing cost

- **JSON is typed** – Strings, numbers, booleans

- **JSON is common** – Used in most apps

---

