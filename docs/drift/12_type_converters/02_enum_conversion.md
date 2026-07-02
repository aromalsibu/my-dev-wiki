
---

## Enum Conversion

**Converting between Dart enums and SQLite values in Drift**

---

# What is it?

**Enum Conversion** is the process of storing Dart enum values in your SQLite database and retrieving them as strongly-typed enums. Drift's `TypeConverter` allows you to map enums to their string names or integer indices. This gives you full type safety while working with enums in your database.

> **Think of Enum Conversion like "translating between names"** – your app knows "active", "inactive", "pending" as distinct states, but the database just sees text like 'active' or numbers like 1, 2, 3. The converter translates between them seamlessly.

```dart
// 👇 Define an enum
enum UserStatus {
  active,
  inactive,
  pending,
  banned,
}

// 👇 String-based enum converter
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

// 👇 Using in a table
class Users extends Table {
  // Store enum as TEXT
  TextColumn get status => text().map(const UserStatusConverter())();
}

// Now you can use enums directly!
final user = User(
  id: 1,
  name: 'John',
  status: UserStatus.active, // 👈 Enum, not string!
);
```

> **What's happening here?**
> - **Enum definition** – The Dart enum
> - **Converter** – Maps enum ↔ SQLite value
> - **`fromSql`** – Converts database value to enum
> - **`toSql`** – Converts enum to database value
> - **Type safety** – Database operations use typed enums

---

# Why does it exist?

- **Type Safety** – Use enums instead of strings/ints
- **IDE Support** – Autocomplete for enum values
- **Compile-time Validation** – Catch invalid values
- **Business Logic** – Enums represent domain concepts
- **Maintainability** – Single source of truth
- **Readability** – Self-documenting code

---

# String-Based Enum Conversion

> **Storing enums as TEXT values**

## Basic String Converter

```dart
// 👇 Define enum
enum ProductStatus {
  draft,
  published,
  archived,
  deleted,
}

// 👇 String converter
class ProductStatusConverter extends TypeConverter<ProductStatus, String> {
  const ProductStatusConverter();

  @override
  ProductStatus fromSql(String fromDb) {
    return ProductStatus.values.firstWhere(
      (e) => e.name == fromDb,
      orElse: () => ProductStatus.draft,
    );
  }

  @override
  String toSql(ProductStatus value) {
    return value.name;
  }
}

// 👇 Using in table
class Products extends Table {
  TextColumn get status => text().map(const ProductStatusConverter())();
}
```

## Generic String Converter

```dart
// 👇 Generic converter for any enum
class EnumStringConverter<T extends Enum> extends TypeConverter<T, String> {
  final List<T> values;
  final T defaultValue;

  const EnumStringConverter(this.values, this.defaultValue);

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

// 👇 Using generic converter
class Orders extends Table {
  TextColumn get status => text().map(
    const EnumStringConverter<OrderStatus>(OrderStatus.values, OrderStatus.pending),
  )();
}
```

---

# Integer-Based Enum Conversion

> **Storing enums as INTEGER values**

## Basic Integer Converter

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

// 👇 Integer converter
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

// 👇 Using in table
class Tasks extends Table {
  IntColumn get priority => integer().map(const PriorityConverter())();
}
```

## Generic Integer Converter

```dart
// 👇 Generic integer converter
class EnumIntConverter<T extends Enum> extends TypeConverter<T, int> {
  final List<T> values;
  final T defaultValue;
  final int Function(T) toInt;
  final T Function(int) fromInt;

  const EnumIntConverter({
    required this.values,
    required this.defaultValue,
    required this.toInt,
    required this.fromInt,
  });

  @override
  T fromSql(int fromDb) {
    try {
      return fromInt(fromDb);
    } catch (e) {
      return defaultValue;
    }
  }

  @override
  int toSql(T value) {
    return toInt(value);
  }
}

// 👇 Define enum with integer mapping
enum Role {
  user(1),
  moderator(2),
  admin(3),
  superAdmin(4);

  final int value;
  const Role(this.value);

  static Role fromValue(int value) {
    return Role.values.firstWhere(
      (e) => e.value == value,
      orElse: () => Role.user,
    );
  }
}

// 👇 Using generic converter
class Users extends Table {
  IntColumn get role => integer().map(
    const EnumIntConverter<Role>(
      values: Role.values,
      defaultValue: Role.user,
      toInt: (role) => role.value,
      fromInt: Role.fromValue,
    ),
  )();
}
```

---

# Real-World Example

> **Complete e-commerce enum conversion system**

```dart
// lib/database/enums/enums.dart
enum UserStatus {
  active,
  inactive,
  pending,
  banned,
}

enum OrderStatus {
  pending,
  processing,
  paid,
  shipped,
  delivered,
  cancelled,
  refunded,
}

enum PaymentStatus {
  unpaid,
  paid,
  refunded,
  failed,
}

enum ProductCategory {
  electronics,
  clothing,
  books,
  home,
  sports,
  toys,
}

// lib/database/converters/enum_converters.dart
import 'package:drift/drift.dart';
import '../enums/enums.dart';

// 👇 UserStatus converter
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

// 👇 OrderStatus converter
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

// 👇 PaymentStatus converter
class PaymentStatusConverter extends TypeConverter<PaymentStatus, String> {
  const PaymentStatusConverter();

  @override
  PaymentStatus fromSql(String fromDb) {
    return PaymentStatus.values.firstWhere(
      (e) => e.name == fromDb,
      orElse: () => PaymentStatus.unpaid,
    );
  }

  @override
  String toSql(PaymentStatus value) {
    return value.name;
  }
}

// 👇 ProductCategory converter (integer-based)
class ProductCategoryConverter extends TypeConverter<ProductCategory, int> {
  const ProductCategoryConverter();

  @override
  ProductCategory fromSql(int fromDb) {
    return ProductCategory.values.firstWhere(
      (e) => e.index == fromDb,
      orElse: () => ProductCategory.electronics,
    );
  }

  @override
  int toSql(ProductCategory value) {
    return value.index;
  }
}

// 👇 Generic enum converter
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

// lib/database/tables/users.dart
import '../converters/enum_converters.dart';
import '../enums/enums.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();

  // 👇 UserStatus enum as TEXT
  TextColumn get status => text()
    .withDefault(const Constant('active'))
    .map(const UserStatusConverter())();

  // 👇 Using generic enum converter
  TextColumn get role => text()
    .withDefault(const Constant('user'))
    .map(const EnumConverter<Role>(Role.values, Role.user))();

  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}

// lib/database/tables/orders.dart
import '../converters/enum_converters.dart';
import '../enums/enums.dart';

class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().unique()();
  IntColumn get userId => integer().references(Users, #id)();
  RealColumn get total => real()();

  // 👇 OrderStatus enum as TEXT
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .map(const OrderStatusConverter())();

  // 👇 PaymentStatus enum as TEXT
  TextColumn get paymentStatus => text()
    .withDefault(const Constant('unpaid'))
    .map(const PaymentStatusConverter())();

  DateTimeColumn get orderDate => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get shippedDate => dateTime().nullable()();
}

// lib/database/tables/products.dart
import '../converters/enum_converters.dart';
import '../enums/enums.dart';

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get sku => text().unique()();
  TextColumn get name => text()();

  // 👇 ProductCategory enum as INTEGER
  IntColumn get category => integer()
    .withDefault(const Constant(0))
    .map(const ProductCategoryConverter())();

  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}
```

```dart
// lib/ui/pages/order_page.dart
class OrderPage extends StatelessWidget {
  final AppDatabase db;
  final int orderId;

  const OrderPage({required this.db, required this.orderId});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: _loadOrder(),
      builder: (context, snapshot) {
        if (!snapshot.hasData) return CircularProgressIndicator();

        final order = snapshot.data!;

        return Scaffold(
          appBar: AppBar(title: Text('Order ${order.orderNumber}')),
          body: Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // 👇 Using enum directly
                Text('Status: ${_getStatusDisplay(order.status)}'),
                Text('Payment: ${_getPaymentDisplay(order.paymentStatus)}'),
                Text('Total: \$${order.total}'),
                Text('Ordered: ${order.orderDate}'),

                // 👇 Enum comparison
                if (order.status == OrderStatus.shipped) {
                  Text('📦 Your order has been shipped!'),
                }

                if (order.status == OrderStatus.delivered) {
                  Text('✅ Your order has been delivered!'),
                }

                // 👇 Enum methods
                Text('Can cancel: ${order.status == OrderStatus.pending}'),
              ],
            ),
          ),
        );
      },
    );
  }

  Future<Order> _loadOrder() async {
    return await (db.select(db.orders)
      ..where((o) => o.id.equals(orderId)))
      .getSingle();
  }

  String _getStatusDisplay(OrderStatus status) {
    switch (status) {
      case OrderStatus.pending:
        return '⏳ Pending';
      case OrderStatus.processing:
        return '🔄 Processing';
      case OrderStatus.paid:
        return '💰 Paid';
      case OrderStatus.shipped:
        return '📦 Shipped';
      case OrderStatus.delivered:
        return '✅ Delivered';
      case OrderStatus.cancelled:
        return '❌ Cancelled';
      case OrderStatus.refunded:
        return '↩️ Refunded';
    }
  }

  String _getPaymentDisplay(PaymentStatus status) {
    switch (status) {
      case PaymentStatus.unpaid:
        return '❌ Unpaid';
      case PaymentStatus.paid:
        return '✅ Paid';
      case PaymentStatus.refunded:
        return '↩️ Refunded';
      case PaymentStatus.failed:
        return '❌ Failed';
    }
  }
}
```

---

# Enum Conversion Best Practices

- **Use string conversion** – More readable in database
- **Use integer conversion** – More compact storage
- **Handle unknown values** – Provide default
- **Use const constructors** – Better performance
- **Document enum meaning** – Explain values
- **Use enum methods** – Business logic in enum
- **Test conversion** – Verify mapping works
- **Consistent naming** – Match database values

---

# Enum Conversion Checklist

| Practice | Description | Priority |
|----------|-------------|----------|
| **String or Int** | Choose storage type | High |
| **Default Value** | Handle unknown | High |
| **Const Constructor** | Performance | High |
| **Documentation** | Explain values | Medium |
| **Testing** | Verify conversion | High |
| **Consistency** | Match naming | Medium |

---

# Common Mistakes

## Mistake 1: Not handling unknown values

Wrong:
```dart
// 🚫 Throws if unknown value
@override
T fromSql(String fromDb) {
  return values.firstWhere((e) => e.name == fromDb);
}
```

Correct:
```dart
// ✅ Handle unknown
@override
T fromSql(String fromDb) {
  return values.firstWhere(
    (e) => e.name == fromDb,
    orElse: () => defaultValue,
  );
}
```

## Mistake 2: Wrong column type

Wrong:
```dart
// 🚫 String converter but int column
IntColumn get status => integer().map(const StringConverter())();
```

Correct:
```dart
// ✅ Match column type
TextColumn get status => text().map(const StringConverter())();
```

## Mistake 3: Missing `.map()`

Wrong:
```dart
// 🚫 Enum not converted
TextColumn get status => text()();
// Stores enum directly (error)
```

Correct:
```dart
// ✅ Use .map()
TextColumn get status => text().map(const StatusConverter())();
```

---

# Summary

| Storage Type | Use Case | Example |
|--------------|----------|---------|
| **String** | Readable, flexible | `'active'`, `'pending'` |
| **Integer** | Compact, indexed | `0`, `1`, `2` |
| **Generic** | Reusable | Any enum type |

---

# Next Steps

Now you understand enum conversion, let's dive deeper:

- [DateTime Conversion](link) – Date/time strategies
- [JSON Conversion](link) – JSON data
- [Custom Objects](link) – Complex object mapping

---

# Did You Know?

- **Enums are type-safe** – Compile-time checking

- **String storage is readable** – Easier to debug

- **Integer storage is compact** – Less space

- **Enum methods add logic** – Business rules in enum

- **Default values are important** – Handle migration

- **Enum converters are reusable** – Share across tables

- **Enums are immutable** – Safe to use

- **Enum conversion is common** – Used in most apps

---

