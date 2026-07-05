## Primary Keys

**Understanding and implementing primary keys in Drift**

---

# What is it?

**Primary Keys** are unique identifiers for each row in a database table. They ensure that every record can be uniquely identified and retrieved. Drift provides flexible ways to define primary keys, from simple auto-incrementing integers to complex composite keys.

> **Think of Primary Keys like "social security numbers"** – each person has a unique one that identifies them and only them, and you can always find someone using their SSN.

```dart
class Users extends Table {
  // The most common primary key: auto-incrementing integer
  IntColumn get id => integer().autoIncrement()();
  
  // The rest of the columns...
  TextColumn get name => text()();
  TextColumn get email => text()();
}
```

> **What's happening here?**
> - **Auto-increment** – Automatically generates sequential IDs
> - **Primary Key** – `autoIncrement()` implies PRIMARY KEY
> - **Unique** – Each ID is unique across the table
> - **Indexed** – Primary keys are automatically indexed for fast lookups

---

# Why does it exist?

- **Unique Identification** – Every row has a unique identifier
- **Fast Lookups** – Primary keys are automatically indexed
- **Relationships** – Foreign keys reference primary keys
- **Data Integrity** – Prevents duplicate rows
- **Performance** – Primary key lookups are O(log n) or better
- **Ordering** – Sequential primary keys maintain insertion order

---

# Single Column Primary Keys

> **The most common type of primary key**

## Auto-incrementing Integer (Recommended)

```dart
class Users extends Table {
  // 👇 Most common primary key approach
  IntColumn get id => integer().autoIncrement()();
  
  TextColumn get name => text()();
  TextColumn get email => text()();
}

// Generated SQL:
// CREATE TABLE users (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   name TEXT NOT NULL,
//   email TEXT NOT NULL
// )
```

## Custom Integer Primary Key

```dart
class Products extends Table {
  // 👇 Custom primary key (not auto-incrementing)
  @override
  ColumnKey get primaryKey => const ColumnKey('product_id');
  
  IntColumn get productId => integer().named('product_id')();
  TextColumn get name => text()();
  RealColumn get price => real()();
}

// Generated SQL:
// CREATE TABLE products (
//   product_id INTEGER PRIMARY KEY,
//   name TEXT NOT NULL,
//   price REAL NOT NULL
// )
```

## Text Primary Key

```dart
class Countries extends Table {
  // 👇 Text primary key (ISO country codes)
  @override
  ColumnKey get primaryKey => const ColumnKey('code');
  
  TextColumn get code => text().named('code')();
  TextColumn get name => text()();
  TextColumn get currency => text()();
}

// Generated SQL:
// CREATE TABLE countries (
//   code TEXT PRIMARY KEY,
//   name TEXT NOT NULL,
//   currency TEXT NOT NULL
// )
```

---

# Composite Primary Keys

> **Primary keys that use multiple columns**

## Simple Composite Key

```dart
class OrderItems extends Table {
  // 👇 Composite primary key (orderId + productId)
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  IntColumn get orderId => integer()();
  IntColumn get productId => integer()();
  IntColumn get quantity => integer()();
  RealColumn get unitPrice => real()();
}

// Generated SQL:
// CREATE TABLE order_items (
//   order_id INTEGER NOT NULL,
//   product_id INTEGER NOT NULL,
//   quantity INTEGER NOT NULL,
//   unit_price REAL NOT NULL,
//   PRIMARY KEY (order_id, product_id)
// )
```

## Composite Key with Custom Names

```dart
class StudentCourses extends Table {
  // 👇 Composite key with named columns
  @override
  Set<Column> get primaryKey => {studentId, courseId, semester};
  
  IntColumn get studentId => integer().named('student_id')();
  IntColumn get courseId => integer().named('course_id')();
  TextColumn get semester => text().named('semester')();
  TextColumn get grade => text().nullable()();
}

// Generated SQL:
// CREATE TABLE student_courses (
//   student_id INTEGER NOT NULL,
//   course_id INTEGER NOT NULL,
//   semester TEXT NOT NULL,
//   grade TEXT,
//   PRIMARY KEY (student_id, course_id, semester)
// )
```

---

# Primary Key Options

> **Advanced primary key configurations**

## WITH AUTOINCREMENT

```dart
class AutoIncrementExample extends Table {
  // 👇 AUTOINCREMENT ensures IDs are never reused
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

// Generated SQL:
// CREATE TABLE auto_increment_example (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   name TEXT NOT NULL
// )
```

**When to use AUTOINCREMENT:**
- When you need IDs to be strictly increasing
- When you never want IDs to be reused
- For audit logs where sequence matters
- **Note:** AUTOINCREMENT adds overhead

## WITHOUT AUTOINCREMENT (Default)

```dart
class DefaultPrimaryKey extends Table {
  // 👇 ROWID acts as primary key (faster, more efficient)
  IntColumn get id => integer()(); // Not autoIncrement()
  TextColumn get name => text()();
}

// Generated SQL:
// CREATE TABLE default_primary_key (
//   id INTEGER PRIMARY KEY,  -- ROWID alias
//   name TEXT NOT NULL
// )
```

**When to skip AUTOINCREMENT:**
- For better performance
- When you don't need strict increasing order
- When you don't care about reused IDs
- For most tables (default is fine)

## Using ROWID Directly

```dart
class RowIdExample extends Table {
  // 👇 Use SQLite's internal ROWID as primary key
  // No need to define a primary key column
  TextColumn get name => text()();
  TextColumn get email => text()();
  
  // Access ROWID via custom query
  Future<int> getRowId(QueryExecutor executor, String name) async {
    final result = await executor.customSelect(
      'SELECT ROWID FROM row_id_example WHERE name = ?',
      variables: [Variable.withString(name)],
    ).get();
    return result.first.data['ROWID'] as int;
  }
}

// Generated SQL:
// CREATE TABLE row_id_example (
//   name TEXT NOT NULL,
//   email TEXT NOT NULL
// )
// Note: No explicit primary key, ROWID is used
```

---

# Primary Key Relationships

> **Using primary keys in relationships**

## One-to-One Relationship

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text()();
}

class Profiles extends Table {
  // 👇 Foreign key references Users primary key
  IntColumn get userId => integer()
    .references(Users, #id)
    .unique() // One-to-one: each user has one profile
    .named('user_id')();
  
  TextColumn get bio => text().nullable()();
  TextColumn get avatar => text().nullable()();
}

// Generated SQL:
// CREATE TABLE profiles (
//   user_id INTEGER NOT NULL UNIQUE REFERENCES users(id),
//   bio TEXT,
//   avatar TEXT
// )
```

## One-to-Many Relationship

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  
  // 👇 Foreign key references Users primary key
  IntColumn get userId => integer()
    .references(Users, #id)
    .named('user_id')();
}

// Generated SQL:
// CREATE TABLE posts (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   title TEXT NOT NULL,
//   content TEXT NOT NULL,
//   user_id INTEGER NOT NULL REFERENCES users(id)
// )
```

## Many-to-Many Relationship

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

class Roles extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

class UserRoles extends Table {
  // 👇 Composite primary key using both foreign keys
  @override
  Set<Column> get primaryKey => {userId, roleId};
  
  IntColumn get userId => integer()
    .references(Users, #id)
    .named('user_id')();
  
  IntColumn get roleId => integer()
    .references(Roles, #id)
    .named('role_id')();
  
  DateTimeColumn get assignedAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('assigned_at')();
}

// Generated SQL:
// CREATE TABLE user_roles (
//   user_id INTEGER NOT NULL REFERENCES users(id),
//   role_id INTEGER NOT NULL REFERENCES roles(id),
//   assigned_at INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   PRIMARY KEY (user_id, role_id)
// )
```

---

# Real-World Example

> **Complete primary key implementation**

```dart
// lib/database/tables/base_table.dart
import 'package:drift/drift.dart';

// 👇 Base table with common primary key pattern
abstract class BaseTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
  BoolColumn get isDeleted => boolean().withDefault(const Constant(false))();
}

// lib/database/tables/users.dart
import 'package:drift/drift.dart';
import 'base_table.dart';

class Users extends BaseTable {
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get fullName => text().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get lastLogin => dateTime().nullable()();
}

// lib/database/tables/products.dart
import 'package:drift/drift.dart';
import 'base_table.dart';

class Products extends BaseTable {
  // 👇 Custom primary key (SKU)
  @override
  ColumnKey get primaryKey => const ColumnKey('sku');
  
  TextColumn get sku => text()
    .unique()
    .named('sku')
    .customConstraint('CHECK (LENGTH(sku) >= 6)')();
  
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  RealColumn get price => real().customConstraint('CHECK (price >= 0)')();
  IntColumn get stock => integer().withDefault(const Constant(0))();
  TextColumn get category => text()();
}

// lib/database/tables/orders.dart
import 'package:drift/drift.dart';
import 'users.dart';
import 'base_table.dart';

class Orders extends BaseTable {
  // 👇 Use standard auto-increment
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Foreign key
  IntColumn get userId => integer()
    .references(Users, #id)
    .named('user_id')();
  
  // 👇 Unique order number (business key)
  TextColumn get orderNumber => text()
    .unique()
    .named('order_number')
    .customConstraint('CHECK (LENGTH(order_number) >= 8)')();
  
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .customConstraint("CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))")();
  
  RealColumn get totalAmount => real()
    .customConstraint('CHECK (total_amount >= 0)')
    .named('total_amount')();
  
  DateTimeColumn get orderDate => dateTime()
    .withDefault(currentDateAndTime)
    .named('order_date')();
}

// lib/database/tables/order_items.dart
import 'package:drift/drift.dart';
import 'orders.dart';
import 'products.dart';

class OrderItems extends Table {
  // 👇 Composite primary key
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  // 👇 Foreign keys
  IntColumn get orderId => integer()
    .references(Orders, #id, onDelete: KeyAction.cascade)
    .named('order_id')();
  
  IntColumn get productId => integer()
    .references(Products, #id)
    .named('product_id')();
  
  IntColumn get quantity => integer()
    .customConstraint('CHECK (quantity > 0)')();
  
  RealColumn get unitPrice => real()
    .customConstraint('CHECK (unit_price >= 0)')
    .named('unit_price')();
  
  RealColumn get totalPrice => real()
    .customConstraint('CHECK (total_price >= 0)')
    .named('total_price')();
  
  // 👇 Additional index for performance
  @override
  List<Index> get indexes => [
    Index('idx_order_items_order', 'order_id'),
    Index('idx_order_items_product', 'product_id'),
  ];
}

// lib/database/database.dart - Usage
import 'package:drift/drift.dart';
import 'tables/users.dart';
import 'tables/products.dart';
import 'tables/orders.dart';
import 'tables/order_items.dart';

@DriftDatabase(tables: [
  Users,
  Products,
  Orders,
  OrderItems,
])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Query by primary key
  Future<User?> getUserById(int id) async {
    return await (select(users)..where((u) => u.id.equals(id)))
        .getSingleOrNull();
  }

  // 👇 Query by custom primary key (SKU)
  Future<Product?> getProductBySku(String sku) async {
    return await (select(products)..where((p) => p.sku.equals(sku)))
        .getSingleOrNull();
  }

  // 👇 Query by business key
  Future<Order?> getOrderByNumber(String orderNumber) async {
    return await (select(orders)..where((o) => o.orderNumber.equals(orderNumber)))
        .getSingleOrNull();
  }

  // 👇 Insert with composite primary key
  Future<void> addOrderItem(
    int orderId,
    int productId,
    int quantity,
    double unitPrice,
  ) async {
    await into(orderItems).insert(
      OrderItemsCompanion.insert(
        orderId: orderId,
        productId: productId,
        quantity: quantity,
        unitPrice: unitPrice,
        totalPrice: quantity * unitPrice,
      ),
    );
  }

  // 👇 Get order with items using primary keys
  Future<Map<String, dynamic>> getOrderWithItems(int orderId) async {
    final order = await getOrderById(orderId);
    if (order == null) throw Exception('Order not found');
    
    final items = await (select(orderItems)
      ..where((i) => i.orderId.equals(orderId)))
      .get();
    
    return {
      'order': order,
      'items': items,
      'total': items.fold<double>(
        0, 
        (sum, item) => sum + item.totalPrice,
      ),
    };
  }

  Future<Order?> getOrderById(int id) async {
    return await (select(orders)..where((o) => o.id.equals(id)))
        .getSingleOrNull();
  }
}
```

---

# Best Practices

- **Use auto-incrementing integers** – For most tables
- **Use business keys** – For natural identifiers (SKU, order numbers)
- **Use composite keys** – For junction tables
- **Add indexes** – On foreign key columns
- **Consider performance** – AUTOINCREMENT adds overhead
- **Use meaningful names** – For primary key columns
- **Document custom keys** – Explain why they're used
- **Plan relationships** – Consider how tables relate
- **Test performance** – Ensure primary keys are efficient

---

# Common Mistakes

## Mistake 1: Using TEXT primary keys on large tables

Wrong:
```dart
// 🚫 Slow for large tables
@override
ColumnKey get primaryKey => const ColumnKey('email');
TextColumn get email => text()();
```

Correct:
```dart
// ✅ Use integer primary key
IntColumn get id => integer().autoIncrement()();
TextColumn get email => text().unique()();
```

## Mistake 2: Not using auto-increment when needed

Wrong:
```dart
// 🚫 IDs might be reused
IntColumn get id => integer()();
```

Correct:
```dart
// ✅ IDs are always unique and increasing
IntColumn get id => integer().autoIncrement()();
```

## Mistake 3: Making foreign keys primary keys by default

Wrong:
```dart
// 🚫 One-to-one relationship forced
@override
Set<Column> get primaryKey => {userId};
IntColumn get userId => integer().references(Users, #id)();
```

Correct:
```dart
// ✅ Let users have their own primary key
IntColumn get id => integer().autoIncrement()();
IntColumn get userId => integer().references(Users, #id).unique()();
```

---

# Summary

| Type | Example | Use Case |
|------|---------|----------|
| **Auto-increment** | `integer().autoIncrement()` | Most tables |
| **Custom Integer** | `ColumnKey('product_id')` | Business keys |
| **Text Key** | `TextColumn get code` | Natural identifiers |
| **Composite** | `primaryKey: {col1, col2}` | Junction tables |
| **ROWID** | No primary key column | Performance |

---

# Next Steps

Now you understand primary keys, let's dive deeper:

- [Composite Keys](link) – Advanced composite key patterns
- [Foreign Keys](link) – Advanced foreign key relationships
- [Default Values](link) – Setting default values

---

# Did You Know?

- **SQLite PRIMARY KEY is just a UNIQUE NOT NULL constraint** – With an implicit index

- **ROWID is always available** – Even without a primary key

- **AUTOINCREMENT guarantees increasing IDs** – But adds overhead

- **Text primary keys are slower** – For lookups than integers

- **Composite primary keys create multi-column indexes** – For fast lookups

- **Primary keys can be used in sorting** – They preserve insertion order

- **Drift auto-generates primary key getters** – For your convenience

- **Foreign keys must reference a PRIMARY KEY or UNIQUE** – Constraint

