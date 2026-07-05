# Custom Objects

**Custom object conversion** allows Drift to store fully structured Dart classes in SQLite by converting them into a storable format and reconstructing them automatically when reading.

---

# What is it?

So far, we’ve converted simple types like:

* `Enum → String / int`
* `DateTime → int`
* `Map/List → JSON String`

Custom object conversion goes one step further:

> It lets you store your own Dart classes directly in a database column.

For example:

```dart id="c1x9qp"
class Address {
  final String city;
  final String street;
  final int zipCode;

  const Address({
    required this.city,
    required this.street,
    required this.zipCode,
  });
}
```

SQLite cannot store this object directly, so Drift uses a `TypeConverter` to serialize and deserialize it.

---

# Why does it exist?

Without custom object conversion, you would flatten everything into primitive fields:

```dart id="m8q2vn"
city: 'London',
street: 'Baker Street',
zipCode: 12345,
```

This leads to:

* Repetitive column definitions
* Poor grouping of related data
* Hard-to-maintain table schemas for complex models

Custom object conversion allows you to:

* Treat related fields as a single object
* Keep your domain model clean
* Reuse the same object across multiple tables

---

# Syntax

A custom object converter typically uses JSON internally.

```dart id="z9k2qp"
import 'dart:convert';

class Address {
  final String city;
  final String street;
  final int zipCode;

  const Address({
    required this.city,
    required this.street,
    required this.zipCode,
  });

  Map<String, dynamic> toJson() {
    return {
      'city': city,
      'street': street,
      'zipCode': zipCode,
    };
  }

  factory Address.fromJson(Map<String, dynamic> json) {
    return Address(
      city: json['city'] as String,
      street: json['street'] as String,
      zipCode: json['zipCode'] as int,
    );
  }
}
```

Now create a converter:

```dart id="q2v9lm"
class AddressConverter
    extends TypeConverter<Address, String> {
  const AddressConverter();

  @override
  Address fromSql(String fromDb) {
    return Address.fromJson(
      jsonDecode(fromDb) as Map<String, dynamic>,
    );
  }

  @override
  String toSql(Address value) {
    return jsonEncode(value.toJson());
  }
}
```

Apply it to a table:

```dart id="v7m1qp"
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  TextColumn get address =>
      text().map(const AddressConverter())();
}
```

**Explanation:**

* The entire `Address` object is stored in one column
* Internally stored as a JSON string
* Exposed as a fully typed Dart object

---

# Mental Model

```text id="t3k9qp"
Address Object
{
  city: "London",
  street: "Baker Street",
  zipCode: 12345
}
        │
        ▼
  AddressConverter
        │
        ▼
'{"city":"London","street":"Baker Street","zipCode":12345}'
        │
        ▼
      SQLite
```

Reading:

```text id="k9v2qp"
JSON String
        │
        ▼
  AddressConverter
        │
        ▼
Address Object
```

Your app never works with raw JSON or strings.

---

# Examples

## Example 1: Store Address Object

```dart id="a8m2qp"
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  TextColumn get address =>
      text().map(const AddressConverter())();
}
```

Insert user:

```dart id="u9x3qp"
await into(users).insert(
  UsersCompanion.insert(
    name: 'Alice',
    address: const Address(
      city: 'London',
      street: 'Baker Street',
      zipCode: 12345,
    ),
  ),
);
```

Read user:

```dart id="p2m8qp"
final user = await select(users).getSingle();

print(user.address.city); // London
```

**Explanation:**

* The object is stored as a single column
* Retrieved as a fully reconstructed `Address` instance

---

## Example 2: Store Complex Nested Object

```dart id="c7v2qp"
class Location {
  final double lat;
  final double lng;

  const Location(this.lat, this.lng);

  Map<String, dynamic> toJson() => {
        'lat': lat,
        'lng': lng,
      };

  factory Location.fromJson(Map<String, dynamic> json) {
    return Location(
      json['lat'] as double,
      json['lng'] as double,
    );
  }
}

class Place {
  final String name;
  final Location location;

  const Place({
    required this.name,
    required this.location,
  });

  Map<String, dynamic> toJson() => {
        'name': name,
        'location': location.toJson(),
      };

  factory Place.fromJson(Map<String, dynamic> json) {
    return Place(
      name: json['name'] as String,
      location: Location.fromJson(
        json['location'] as Map<String, dynamic>,
      ),
    );
  }
}
```

Converter:

```dart id="l4x8qp"
class PlaceConverter
    extends TypeConverter<Place, String> {
  const PlaceConverter();

  @override
  Place fromSql(String fromDb) {
    return Place.fromJson(
      jsonDecode(fromDb) as Map<String, dynamic>,
    );
  }

  @override
  String toSql(Place value) {
    return jsonEncode(value.toJson());
  }
}
```

Table:

```dart id="n8q2vp"
class Places extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get place =>
      text().map(const PlaceConverter())();
}
```

**Explanation:**

* Supports nested object structures
* Entire graph stored in a single column
* Useful for encapsulating complex domain models

---

## Real-World Example

A shopping app stores shipping information as a single object.

```dart id="y8m3qp"
class ShippingInfo {
  final String recipientName;
  final Address address;
  final String phone;

  const ShippingInfo({
    required this.recipientName,
    required this.address,
    required this.phone,
  });

  Map<String, dynamic> toJson() => {
        'recipientName': recipientName,
        'address': address.toJson(),
        'phone': phone,
      };

  factory ShippingInfo.fromJson(Map<String, dynamic> json) {
    return ShippingInfo(
      recipientName: json['recipientName'] as String,
      address: Address.fromJson(
        json['address'] as Map<String, dynamic>,
      ),
      phone: json['phone'] as String,
    );
  }
}
```

Converter:

```dart id="s3k9qp"
class ShippingInfoConverter
    extends TypeConverter<ShippingInfo, String> {
  const ShippingInfoConverter();

  @override
  ShippingInfo fromSql(String fromDb) {
    return ShippingInfo.fromJson(
      jsonDecode(fromDb) as Map<String, dynamic>,
    );
  }

  @override
  String toSql(ShippingInfo value) {
    return jsonEncode(value.toJson());
  }
}
```

Table:

```dart id="v2m8qp"
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get shippingInfo =>
      text().map(const ShippingInfoConverter())();
}
```

Insert order:

```dart id="k9q2mp"
await into(orders).insert(
  OrdersCompanion.insert(
    shippingInfo: ShippingInfo(
      recipientName: 'Alice',
      address: const Address(
        city: 'London',
        street: 'Baker Street',
        zipCode: 12345,
      ),
      phone: '+44 123456789',
    ),
  ),
);
```

---

# When to Use

Use custom object conversion when:

* A field logically represents a single domain concept
* You want to encapsulate multiple related fields
* Data is not frequently queried individually
* Object structure is stable
* You prefer cleaner domain models

---

# When NOT to Use

Avoid custom object conversion when:

* You need to filter or sort by internal fields
* Data must be indexed efficiently
* Relationships should be normalized (use tables instead)
* Object structure changes frequently

Example:

❌ Bad:

```dart id="m2v8qp"
where: (o) => o.shippingInfo.equals(...)
```

You cannot efficiently query inside the object.

---

# Best Practices

* Use JSON internally for simplicity
* Keep objects immutable
* Provide `toJson()` / `fromJson()` methods
* Avoid deeply nested objects if performance matters
* Prefer relational tables for query-heavy data

---

# Common Mistakes

## Using Custom Objects for Queryable Data

**Wrong**

```dart id="q7m2qp"
shippingInfo.city == 'London'
```

You cannot query inside a serialized object.

**Correct**

```dart id="p8v2qp"
TextColumn get city => text()();
```

Use separate columns for searchable fields.

---

## Over-Nesting Objects

**Wrong**

```dart id="z1k9qp"
Order -> ShippingInfo -> Address -> Location -> Metadata
```

Too deep nesting makes debugging and updates difficult.

**Correct**

Keep object graphs shallow when stored in a database.

---

## Forgetting Version Compatibility

**Wrong**

Changing JSON structure without migration handling.

**Correct**

Maintain backward compatibility in `fromJson()` when evolving models.

---

# Related APIs

* TypeConverter
* JSON Conversion
* Enum Conversion
* DateTime Conversion
* Reusable Converters

---

# Summary

Custom object conversion allows Drift to store entire Dart classes in a single SQLite column by serializing them (usually as JSON). It provides a clean, type-safe way to model complex domain objects while keeping database schema simple. However, it should be used only for non-queryable, self-contained data structures.
