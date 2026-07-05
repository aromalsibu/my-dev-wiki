## Custom Objects

**Storing complex Dart objects in Drift with custom converters**

---

# What is it?

**Custom Objects** are user-defined Dart classes that you want to store in your database. Drift's `TypeConverter` allows you to serialize these objects to a primitive SQLite type (usually TEXT as JSON) and deserialize them back when querying. This gives you complete control over how your objects are stored.

> **Think of Custom Objects like "building custom containers"** – instead of putting your items in standard boxes, you design your own container (converter) that knows exactly how to pack and unpack your specific items.

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
  
  // Convert to JSON for storage
  Map<String, dynamic> toJson() => {
    'street': street,
    'city': city,
    'state': state,
    'zipCode': zipCode,
    'country': country,
  };
  
  // Convert from JSON
  factory Address.fromJson(Map<String, dynamic> json) => Address(
    street: json['street'] as String,
    city: json['city'] as String,
    state: json['state'] as String,
    zipCode: json['zipCode'] as String,
    country: json['country'] as String,
  );
}

// 👇 Custom object converter
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

// 👇 Using in table
class Users extends Table {
  TextColumn get address => text()
    .nullable()
    .map(const AddressConverter())();
}

// Now you can store Address objects directly!
final user = User(
  id: 1,
  name: 'John',
  address: Address( // 👈 Dart object!
    street: '123 Main St',
    city: 'New York',
    state: 'NY',
    zipCode: '10001',
    country: 'USA',
  ),
);
```

> **What's happening here?**
> - **Custom class** – Your Dart object
> - **Serialization** – `toJson()` converts to JSON
> - **Deserialization** – `fromJson()` converts from JSON
> - **Converter** – Handles the transformation
> - **Type safety** – Database operations use typed objects

---

# Why does it exist?

- **Domain Models** – Store business objects
- **Complex Data** – Nested structures
- **Code Organization** – Keep data with behavior
- **Type Safety** – Full type checking
- **Reusability** – Share objects across app
- **Testing** – Easy to mock and test

---

# Basic Custom Objects

> **Simple custom object converters**

## Single Object Converter

```dart
// 👇 Custom Dart class
class Contact {
  final String phone;
  final String email;
  final String? website;
  
  Contact({
    required this.phone,
    required this.email,
    this.website,
  });
  
  Map<String, dynamic> toJson() => {
    'phone': phone,
    'email': email,
    'website': website,
  };
  
  factory Contact.fromJson(Map<String, dynamic> json) => Contact(
    phone: json['phone'] as String,
    email: json['email'] as String,
    website: json['website'] as String?,
  );
}

// 👇 Converter
class ContactConverter extends TypeConverter<Contact, String> {
  const ContactConverter();
  
  @override
  Contact fromSql(String fromDb) {
    if (fromDb.isEmpty) return null;
    final json = jsonDecode(fromDb) as Map<String, dynamic>;
    return Contact.fromJson(json);
  }
  
  @override
  String toSql(Contact value) {
    return jsonEncode(value.toJson());
  }
}

// 👇 Using in table
class Users extends Table {
  TextColumn get contact => text()
    .nullable()
    .map(const ContactConverter())();
}
```

## List of Custom Objects

```dart
// 👇 Custom object list converter
class OrderItem {
  final int productId;
  final int quantity;
  final double price;
  
  OrderItem({
    required this.productId,
    required this.quantity,
    required this.price,
  });
  
  Map<String, dynamic> toJson() => {
    'productId': productId,
    'quantity': quantity,
    'price': price,
  };
  
  factory OrderItem.fromJson(Map<String, dynamic> json) => OrderItem(
    productId: json['productId'] as int,
    quantity: json['quantity'] as int,
    price: json['price'] as double,
  );
}

class OrderItemListConverter extends TypeConverter<List<OrderItem>, String> {
  const OrderItemListConverter();
  
  @override
  List<OrderItem> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => OrderItem.fromJson(e as Map<String, dynamic>)).toList();
  }
  
  @override
  String toSql(List<OrderItem> value) {
    return jsonEncode(value.map((e) => e.toJson()).toList());
  }
}
```

---

# Advanced Custom Objects

> **Complex object converters**

## Nested Objects

```dart
// 👇 Nested custom objects
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

class Product {
  final String name;
  final String description;
  final List<ProductVariant> variants;
  
  Product({
    required this.name,
    required this.description,
    required this.variants,
  });
  
  Map<String, dynamic> toJson() => {
    'name': name,
    'description': description,
    'variants': variants.map((v) => v.toJson()).toList(),
  };
  
  factory Product.fromJson(Map<String, dynamic> json) => Product(
    name: json['name'] as String,
    description: json['description'] as String,
    variants: (json['variants'] as List<dynamic>)
        .map((e) => ProductVariant.fromJson(e as Map<String, dynamic>))
        .toList(),
  );
}

// 👇 Nested converter
class ProductConverter extends TypeConverter<Product, String> {
  const ProductConverter();
  
  @override
  Product fromSql(String fromDb) {
    if (fromDb.isEmpty) return null;
    final json = jsonDecode(fromDb) as Map<String, dynamic>;
    return Product.fromJson(json);
  }
  
  @override
  String toSql(Product value) {
    return jsonEncode(value.toJson());
  }
}
```

## Generic Object Converter

```dart
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
class Users extends Table {
  TextColumn get profile => text().nullable().map(
    const ObjectConverter<UserProfile>(
      fromJson: UserProfile.fromJson,
      toJson: (p) => p.toJson(),
    ),
  )();
}
```

---

# Real-World Example

> **Complete e-commerce custom objects system**

```dart
// lib/database/models/address.dart
class Address {
  final String street;
  final String city;
  final String state;
  final String zipCode;
  final String country;
  final String? unit;
  final double? latitude;
  final double? longitude;
  
  Address({
    required this.street,
    required this.city,
    required this.state,
    required this.zipCode,
    required this.country,
    this.unit,
    this.latitude,
    this.longitude,
  });
  
  Map<String, dynamic> toJson() => {
    'street': street,
    'city': city,
    'state': state,
    'zipCode': zipCode,
    'country': country,
    if (unit != null) 'unit': unit,
    if (latitude != null) 'latitude': latitude,
    if (longitude != null) 'longitude': longitude,
  };
  
  factory Address.fromJson(Map<String, dynamic> json) => Address(
    street: json['street'] as String,
    city: json['city'] as String,
    state: json['state'] as String,
    zipCode: json['zipCode'] as String,
    country: json['country'] as String,
    unit: json['unit'] as String?,
    latitude: json['latitude'] as double?,
    longitude: json['longitude'] as double?,
  );
  
  String get fullAddress => [street, unit, city, state, zipCode, country]
      .where((part) => part != null && part.isNotEmpty)
      .join(', ');
  
  bool get isValid => street.isNotEmpty && city.isNotEmpty && state.isNotEmpty;
}

// lib/database/models/product_variant.dart
class ProductVariant {
  final String id;
  final String name;
  final String sku;
  final double price;
  final int stock;
  final Map<String, String> attributes;
  final bool isActive;
  
  ProductVariant({
    required this.id,
    required this.name,
    required this.sku,
    required this.price,
    required this.stock,
    required this.attributes,
    this.isActive = true,
  });
  
  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'sku': sku,
    'price': price,
    'stock': stock,
    'attributes': attributes,
    'isActive': isActive,
  };
  
  factory ProductVariant.fromJson(Map<String, dynamic> json) => ProductVariant(
    id: json['id'] as String,
    name: json['name'] as String,
    sku: json['sku'] as String,
    price: json['price'] as double,
    stock: json['stock'] as int,
    attributes: Map<String, String>.from(json['attributes'] as Map),
    isActive: json['isActive'] as bool? ?? true,
  );
  
  bool get isInStock => stock > 0;
}

// lib/database/converters/custom_object_converters.dart
import 'dart:convert';
import 'package:drift/drift.dart';
import '../models/address.dart';
import '../models/product_variant.dart';

// 👇 Address converter
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

// 👇 Product variant list converter
class ProductVariantListConverter extends TypeConverter<List<ProductVariant>, String> {
  const ProductVariantListConverter();
  
  @override
  List<ProductVariant> fromSql(String fromDb) {
    if (fromDb.isEmpty) return [];
    final list = jsonDecode(fromDb) as List<dynamic>;
    return list.map((e) => ProductVariant.fromJson(e as Map<String, dynamic>)).toList();
  }
  
  @override
  String toSql(List<ProductVariant> value) {
    return jsonEncode(value.map((e) => e.toJson()).toList());
  }
}

// 👇 Generic object converter
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

// lib/database/tables/products.dart
import '../converters/custom_object_converters.dart';
import '../models/product_variant.dart';

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();

  // 👇 List of variants
  TextColumn get variants => text()
    .withDefault(const Constant('[]'))
    .map(const ProductVariantListConverter())();

  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/tables/users.dart
import '../converters/custom_object_converters.dart';
import '../models/address.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();

  // 👇 Address object
  TextColumn get shippingAddress => text()
    .nullable()
    .map(const AddressConverter())
    .named('shipping_address')();

  TextColumn get billingAddress => text()
    .nullable()
    .map(const AddressConverter())
    .named('billing_address')();

  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}
```

```dart
// lib/ui/pages/checkout_page.dart
class CheckoutPage extends StatelessWidget {
  final AppDatabase db;
  final int userId;

  const CheckoutPage({required this.db, required this.userId});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: _loadUserData(),
      builder: (context, snapshot) {
        if (!snapshot.hasData) return CircularProgressIndicator();

        final user = snapshot.data!;

        return Scaffold(
          appBar: AppBar(title: Text('Checkout')),
          body: Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // 👇 Address object
                Text('Shipping Address:'),
                if (user.shippingAddress != null) ...[
                  Text(user.shippingAddress!.street),
                  Text(user.shippingAddress!.city),
                  Text(user.shippingAddress!.state),
                  Text(user.shippingAddress!.zipCode),
                  Text(user.shippingAddress!.country),
                ],
                SizedBox(height: 16),

                Text('Billing Address:'),
                if (user.billingAddress != null) ...[
                  Text(user.billingAddress!.street),
                  Text(user.billingAddress!.city),
                  Text(user.billingAddress!.state),
                  Text(user.billingAddress!.zipCode),
                  Text(user.billingAddress!.country),
                ],
                SizedBox(height: 16),

                // 👇 Address methods
                if (user.shippingAddress != null)
                  Text('Full Address: ${user.shippingAddress!.fullAddress}'),
              ],
            ),
          ),
        );
      },
    );
  }

  Future<User> _loadUserData() async {
    return await (db.select(db.users)
      ..where((u) => u.id.equals(userId)))
      .getSingle();
  }
}
```

---

# Custom Object Best Practices

- **Use JSON for serialization** – Standard format
- **Handle null values** – Use nullable fields
- **Add helper methods** – `fullAddress`, `isValid`
- **Use const constructors** – Better performance
- **Document object structure** – Explain fields
- **Test serialization** – Verify conversion
- **Use immutable objects** – Final fields where possible
- **Implement equality** – For testing and collections

---

# Custom Object Checklist

| Practice | Description | Priority |
|----------|-------------|----------|
| **Serialization** | toJson/fromJson | High |
| **Null Handling** | Nullable fields | High |
| **Helper Methods** | Domain logic | Medium |
| **Const Constructor** | Performance | High |
| **Testing** | Verify conversion | High |
| **Immutability** | Final fields | Medium |
| **Equality** | For collections | Medium |

---

# Common Mistakes

## Mistake 1: Missing default constructor

Wrong:
```dart
// 🚫 No default constructor
class Address {
  final String street;
  Address(this.street); // No default
}
```

Correct:
```dart
// ✅ Has default constructor
class Address {
  final String street;
  Address({required this.street}); // Named constructor
}
```

## Mistake 2: Not handling null

Wrong:
```dart
// 🚫 Null throws error
@override
Address fromSql(String fromDb) {
  return Address.fromJson(jsonDecode(fromDb));
}
```

Correct:
```dart
// ✅ Handle null
@override
Address fromSql(String fromDb) {
  if (fromDb.isEmpty) return null;
  return Address.fromJson(jsonDecode(fromDb));
}
```

## Mistake 3: Missing type casting

Wrong:
```dart
// 🚫 Type casting errors
final city = json['city']; // dynamic
```

Correct:
```dart
// ✅ Type casting
final city = json['city'] as String;
```

---

# Summary

| Type | Use Case | Example |
|------|----------|---------|
| **Single Object** | One-to-one data | Address, Contact |
| **List of Objects** | One-to-many data | Product variants |
| **Nested Objects** | Complex structures | Order with items |
| **Generic** | Reusable | Any object type |

---

# Next Steps

Now you understand custom objects, let's dive deeper:

- [Reusable Converters](link) – Sharing converters

---

# Did You Know?

- **Custom objects add behavior** – Methods and logic

- **JSON serialization is common** – Standard approach

- **Nested objects are powerful** – Complex structures

- **Generic converters are reusable** – DRY principle

- **Custom objects are type-safe** – Compile-time checking

- **Helper methods improve code** – Cleaner usage

- **Immutability is beneficial** – Safer objects

- **Custom objects are testable** – Unit tests

---

