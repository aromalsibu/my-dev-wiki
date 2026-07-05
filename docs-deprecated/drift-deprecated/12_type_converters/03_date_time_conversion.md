## DateTime Conversion

**Handling dates and times in Drift with custom converters**

---

# What is it?

**DateTime Conversion** is the process of storing and retrieving Dart `DateTime` objects in your SQLite database. SQLite doesn't have a native DateTime type, so dates are typically stored as TEXT (ISO 8601 strings), INTEGER (Unix timestamps), or REAL (Julian day numbers). Drift's `TypeConverter` allows you to choose the best format for your needs.

> **Think of DateTime Conversion like "translating between calendar formats"** – the database stores dates in a specific format, while your app uses Dart's powerful DateTime objects. The converter handles the translation seamlessly.

```dart
// 👇 Unix timestamp converter (stored as INTEGER)
class UnixDateTimeConverter extends TypeConverter<DateTime, int> {
  const UnixDateTimeConverter();
  
  @override
  DateTime fromSql(int fromDb) {
    return DateTime.fromMillisecondsSinceEpoch(fromDb * 1000);
  }
  
  @override
  int toSql(DateTime value) {
    return value.millisecondsSinceEpoch ~/ 1000;
  }
}

// 👇 Using in a table
class Events extends Table {
  // Store as Unix timestamp
  IntColumn get eventDate => integer().map(const UnixDateTimeConverter())();
}

// Now you can use DateTime objects directly!
final event = Event(
  id: 1,
  name: 'Conference',
  eventDate: DateTime(2024, 12, 15), // 👈 Dart DateTime!
);
```

> **What's happening here?**
> - **Format choice** – Unix timestamp, ISO string, or Julian day
> - **`fromSql`** – Converts database value to `DateTime`
> - **`toSql`** – Converts `DateTime` to database value
> - **Type safety** – Database operations use `DateTime`

---

# Why does it exist?

- **Date Storage** – Store dates in SQLite
- **Timezone Handling** – Manage timezones
- **Date Arithmetic** – Perform calculations
- **Format Flexibility** – Choose storage format
- **Compatibility** – Work with existing databases
- **Precision** – Store milliseconds or just dates

---

# Unix Timestamp Conversion

> **Storing dates as Unix timestamps (INTEGER)**

## Basic Unix Converter

```dart
// 👇 Unix timestamp (seconds since epoch)
class UnixDateTimeConverter extends TypeConverter<DateTime, int> {
  const UnixDateTimeConverter();
  
  @override
  DateTime fromSql(int fromDb) {
    return DateTime.fromMillisecondsSinceEpoch(fromDb * 1000);
  }
  
  @override
  int toSql(DateTime value) {
    return value.millisecondsSinceEpoch ~/ 1000;
  }
}

// 👇 Unix timestamp with milliseconds
class UnixMillisDateTimeConverter extends TypeConverter<DateTime, int> {
  const UnixMillisDateTimeConverter();
  
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

## Using Unix Converter

```dart
// 👇 Using in table
class Orders extends Table {
  IntColumn get createdAt => integer()
    .map(const UnixDateTimeConverter())
    .named('created_at')();
  
  IntColumn get updatedAt => integer()
    .nullable()
    .map(const UnixDateTimeConverter())
    .named('updated_at')();
}

// Usage
final order = Order(
  id: 1,
  createdAt: DateTime.now(), // 👈 Automatic conversion
);
```

---

# ISO 8601 String Conversion

> **Storing dates as ISO 8601 strings (TEXT)**

## Basic ISO Converter

```dart
// 👇 ISO 8601 string converter
class IsoDateTimeConverter extends TypeConverter<DateTime, String> {
  const IsoDateTimeConverter();
  
  @override
  DateTime fromSql(String fromDb) {
    return DateTime.parse(fromDb);
  }
  
  @override
  String toSql(DateTime value) {
    return value.toIso8601String();
  }
}

// 👇 ISO 8601 date only (no time)
class IsoDateConverter extends TypeConverter<DateTime, String> {
  const IsoDateConverter();
  
  @override
  DateTime fromSql(String fromDb) {
    return DateTime.parse(fromDb);
  }
  
  @override
  String toSql(DateTime value) {
    return value.toIso8601String().substring(0, 10);
  }
}
```

## Using ISO Converter

```dart
// 👇 Using in table
class Users extends Table {
  TextColumn get birthDate => text()
    .nullable()
    .map(const IsoDateConverter())
    .named('birth_date')();
  
  TextColumn get lastLogin => text()
    .nullable()
    .map(const IsoDateTimeConverter())
    .named('last_login')();
}
```

---

# Custom Date Formats

> **Storing dates in custom formats**

```dart
// 👇 Custom date format converter
class CustomDateConverter extends TypeConverter<DateTime, String> {
  final String format;
  
  const CustomDateConverter({this.format = 'yyyy-MM-dd HH:mm:ss'});
  
  @override
  DateTime fromSql(String fromDb) {
    // Parse custom format
    final parts = fromDb.split(' ');
    final dateParts = parts[0].split('-');
    final timeParts = parts.length > 1 ? parts[1].split(':') : ['00', '00', '00'];
    
    return DateTime(
      int.parse(dateParts[0]),
      int.parse(dateParts[1]),
      int.parse(dateParts[2]),
      int.parse(timeParts[0]),
      int.parse(timeParts[1]),
      int.parse(timeParts[2]),
    );
  }
  
  @override
  String toSql(DateTime value) {
    return '${value.year}-${value.month.toString().padLeft(2, '0')}-${value.day.toString().padLeft(2, '0')} '
           '${value.hour.toString().padLeft(2, '0')}:${value.minute.toString().padLeft(2, '0')}:${value.second.toString().padLeft(2, '0')}';
  }
}
```

---

# Real-World Example

> **Complete e-commerce date/time conversion system**

```dart
// lib/database/converters/date_converters.dart
import 'package:drift/drift.dart';

// 👇 Unix timestamp converter (seconds)
class UnixDateTimeConverter extends TypeConverter<DateTime, int> {
  const UnixDateTimeConverter();
  
  @override
  DateTime fromSql(int fromDb) {
    return DateTime.fromMillisecondsSinceEpoch(fromDb * 1000);
  }
  
  @override
  int toSql(DateTime value) {
    return value.millisecondsSinceEpoch ~/ 1000;
  }
}

// 👇 ISO 8601 string converter
class IsoDateTimeConverter extends TypeConverter<DateTime, String> {
  const IsoDateTimeConverter();
  
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
class IsoDateConverter extends TypeConverter<DateTime, String> {
  const IsoDateConverter();
  
  @override
  DateTime fromSql(String fromDb) {
    return DateTime.parse(fromDb);
  }
  
  @override
  String toSql(DateTime value) {
    return value.toIso8601String().substring(0, 10);
  }
}

// 👇 Human-readable format
class HumanDateConverter extends TypeConverter<DateTime, String> {
  const HumanDateConverter();
  
  @override
  DateTime fromSql(String fromDb) {
    return DateTime.parse(fromDb);
  }
  
  @override
  String toSql(DateTime value) {
    return '${value.year}-${value.month.toString().padLeft(2, '0')}-${value.day.toString().padLeft(2, '0')}';
  }
}

// 👇 Month/year only converter
class MonthYearConverter extends TypeConverter<DateTime, String> {
  const MonthYearConverter();
  
  @override
  DateTime fromSql(String fromDb) {
    final parts = fromDb.split('-');
    return DateTime(int.parse(parts[0]), int.parse(parts[1]));
  }
  
  @override
  String toSql(DateTime value) {
    return '${value.year}-${value.month.toString().padLeft(2, '0')}';
  }
}

// lib/database/tables/orders.dart
import '../converters/date_converters.dart';

class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().unique()();
  IntColumn get userId => integer().references(Users, #id)();
  RealColumn get total => real()();
  TextColumn get status => text()();

  // 👇 Unix timestamp (compact, easy to sort)
  IntColumn get orderDate => integer()
    .withDefault(currentDateAndTime)
    .map(const UnixDateTimeConverter())
    .named('order_date')();

  // 👇 ISO string (human-readable)
  TextColumn get shippedDate => text()
    .nullable()
    .map(const IsoDateTimeConverter())
    .named('shipped_date')();

  // 👇 Date only (no time)
  TextColumn get deliveryDate => text()
    .nullable()
    .map(const IsoDateConverter())
    .named('delivery_date')();

  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}

// lib/database/tables/users.dart
import '../converters/date_converters.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();

  // 👇 ISO date (birthday, no time)
  TextColumn get birthDate => text()
    .nullable()
    .map(const IsoDateConverter())
    .named('birth_date')();

  // 👇 Last login (ISO with time)
  TextColumn get lastLogin => text()
    .nullable()
    .map(const IsoDateTimeConverter())
    .named('last_login')();

  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}
```

```dart
// lib/ui/pages/order_history_page.dart
class OrderHistoryPage extends StatelessWidget {
  final AppDatabase db;
  final int userId;

  const OrderHistoryPage({required this.db, required this.userId});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Order History')),
      body: FutureBuilder(
        future: _loadOrders(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return CircularProgressIndicator();

          final orders = snapshot.data!;

          return ListView.builder(
            itemCount: orders.length,
            itemBuilder: (context, index) {
              final order = orders[index];

              return Card(
                margin: EdgeInsets.all(8),
                child: ListTile(
                  title: Text('Order #${order.orderNumber}'),
                  subtitle: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      // 👇 Formatted dates
                      Text('Ordered: ${_formatDate(order.orderDate)}'),
                      if (order.shippedDate != null)
                        Text('Shipped: ${_formatDate(order.shippedDate!)}'),
                      if (order.deliveryDate != null)
                        Text('Delivered: ${_formatDate(order.deliveryDate!)}'),
                      Text('Total: \$${order.total}'),
                    ],
                  ),
                  trailing: Text(order.status),
                ),
              );
            },
          );
        },
      ),
    );
  }

  Future<List<Order>> _loadOrders() async {
    return await (db.select(db.orders)
      ..where((o) => o.userId.equals(userId))
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
      .get();
  }

  String _formatDate(DateTime date) {
    return '${date.month}/${date.day}/${date.year} ${date.hour}:${date.minute.toString().padLeft(2, '0')}';
  }
}
```

---

# DateTime Conversion Best Practices

- **Choose appropriate format** – Unix for performance, ISO for readability
- **Handle null values** – Use nullable columns
- **Use timezone-aware** – Consider timezone handling
- **Use const constructors** – Better performance
- **Document format choice** – Explain why
- **Test conversions** – Verify date handling
- **Consider precision** – Milliseconds or seconds
- **Be consistent** – Use same format throughout

---

# DateTime Conversion Checklist

| Practice | Description | Priority |
|----------|-------------|----------|
| **Format Choice** | Unix, ISO, or custom | High |
| **Null Handling** | Use nullable | High |
| **Timezone** | Consider timezone | Medium |
| **Const Constructor** | Performance | High |
| **Testing** | Verify conversion | High |
| **Consistency** | Same format | Medium |

---

# Common Mistakes

## Mistake 1: Timezone issues

Wrong:
```dart
// 🚫 Losing timezone info
@override
DateTime fromSql(String fromDb) {
  return DateTime.parse(fromDb); // Local timezone may be wrong
}
```

Correct:
```dart
// ✅ Handle timezone
@override
DateTime fromSql(String fromDb) {
  return DateTime.parse(fromDb).toLocal();
}
```

## Mistake 2: Wrong format

Wrong:
```dart
// 🚫 Invalid date format
final date = DateTime.parse('2024-15-01'); // Month 15 invalid
```

Correct:
```dart
// ✅ Valid format
final date = DateTime.parse('2024-01-15');
```

## Mistake 3: Not handling null

Wrong:
```dart
// 🚫 Null throws error
@override
DateTime fromSql(String fromDb) {
  return DateTime.parse(fromDb); // Null error
}
```

Correct:
```dart
// ✅ Handle null
@override
DateTime fromSql(String fromDb) {
  if (fromDb == null) return null;
  return DateTime.parse(fromDb);
}
```

---

# Summary

| Format | Storage | Use Case | Example |
|--------|---------|----------|---------|
| **Unix Timestamp** | INTEGER | Sorting, performance | `1704067200` |
| **ISO 8601** | TEXT | Readability | `'2024-01-01T12:00:00'` |
| **Date Only** | TEXT | Dates only | `'2024-01-01'` |
| **Custom** | TEXT | Specific needs | `'01/01/2024'` |

---

# Next Steps

Now you understand DateTime conversion, let's dive deeper:

- [JSON Conversion](link) – JSON data
- [Custom Objects](link) – Complex object mapping
- [Reusable Converters](link) – Sharing converters

---

# Did You Know?

- **Unix timestamps are compact** – Only 4-8 bytes

- **ISO strings are human-readable** – Easy to debug

- **SQLite has date functions** – `DATE()`, `TIME()`, `STRFTIME()`

- **Timezones are important** – Consider UTC vs local

- **Date precision matters** – Seconds vs milliseconds

- **Date formats are flexible** – Use any format you want

- **Dates can be calculated** – In SQL or Dart

- **Date conversion is common** – Used in almost every app

---

