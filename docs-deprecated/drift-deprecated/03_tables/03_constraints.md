## Constraints

**Enforcing data integrity with Drift constraints**

---

# What is it?

**Constraints** are rules that enforce data integrity at the database level. They ensure that only valid data is stored in your tables, preventing corruption and maintaining consistency. Drift supports all SQLite constraint types including PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, and NOT NULL.

> **Think of Constraints like "bouncers at a club"** – they check every piece of data trying to enter your database and only let in the ones that meet the rules.

```dart
class Users extends Table {
  // NOT NULL constraint (implicit)
  TextColumn get name => text()();
  
  // UNIQUE constraint
  TextColumn get email => text().unique()();
  
  // CHECK constraint
  IntColumn get age => integer()
    .customConstraint('CHECK (age >= 0 AND age <= 150)')();
  
  // PRIMARY KEY constraint
  IntColumn get id => integer().autoIncrement()();
  
  // FOREIGN KEY constraint
  IntColumn get roleId => integer().references(Roles, #id)();
}
```

> **What's happening here?**
> - **NOT NULL** – Column cannot contain NULL values
> - **UNIQUE** – All values in column must be different
> - **CHECK** – Values must satisfy a condition
> - **PRIMARY KEY** – Uniquely identifies each row
> - **FOREIGN KEY** – Links to another table

---

# Why does it exist?

- **Data Integrity** – Ensure data is valid and consistent
- **Data Quality** – Prevent garbage data from entering
- **Performance** – Constraints can be used as indexes
- **Documentation** – Self-documenting data rules
- **Error Prevention** – Catch data issues early
- **Referential Integrity** – Maintain relationships between tables

---

# NOT NULL Constraints

> **Ensuring columns always have values**

```dart
class NotNullExample extends Table {
  // 👇 Implicit NOT NULL (default)
  TextColumn get name => text()();
  IntColumn get age => integer()();
  BoolColumn get isActive => boolean()();
  
  // 👇 Explicit NOT NULL (same as default)
  TextColumn get email => text().customConstraint('NOT NULL')();
  
  // 👇 NULL allowed
  TextColumn get description => text().nullable()();
  
  // 👇 NOT NULL with default
  TextColumn get status => text()
    .withDefault(const Constant('active'))();
}

// Generated SQL:
// CREATE TABLE not_null_example (
//   name TEXT NOT NULL,
//   age INTEGER NOT NULL,
//   is_active INTEGER NOT NULL,
//   email TEXT NOT NULL,
//   description TEXT,
//   status TEXT NOT NULL DEFAULT 'active'
// )
```

> **Key insights:**
> - **NOT NULL is the default** – For all columns
> - **`.nullable()` removes NOT NULL**
> - **NOT NULL with default** – Always has a value
> - **Required fields** – Should be NOT NULL

---

# UNIQUE Constraints

> **Preventing duplicate values**

```dart
class UniqueExample extends Table {
  // 👇 Simple unique constraint
  TextColumn get email => text().unique()();
  
  // 👇 Unique with custom name
  TextColumn get username => text()
    .unique()
    .named('user_name')();
  
  // 👇 Unique with custom constraint
  TextColumn get phone => text()
    .customConstraint('UNIQUE (phone)')();
  
  // 👇 Composite unique (multiple columns)
  @override
  List<Modifier> get constraints => [
    const Modifier.unique(['first_name', 'last_name'], 'unique_full_name')
  ];
  
  TextColumn get firstName => text().named('first_name')();
  TextColumn get lastName => text().named('last_name')();
}

// Generated SQL:
// CREATE TABLE unique_example (
//   email TEXT NOT NULL UNIQUE,
//   user_name TEXT NOT NULL UNIQUE,
//   phone TEXT NOT NULL UNIQUE,
//   first_name TEXT NOT NULL,
//   last_name TEXT NOT NULL,
//   CONSTRAINT unique_full_name UNIQUE (first_name, last_name)
// )
```

---

# CHECK Constraints

> **Validating data with conditions**

```dart
class CheckExample extends Table {
  // 👇 Simple CHECK constraint
  IntColumn get age => integer()
    .customConstraint('CHECK (age >= 0)')();
  
  // 👇 Multiple CHECK constraints
  IntColumn get score => integer()
    .customConstraint('CHECK (score >= 0)')
    .customConstraint('CHECK (score <= 100)')();
  
  // 👇 CHECK with string validation
  TextColumn get email => text()
    .customConstraint('CHECK (email LIKE "%@%")')();
  
  // 👇 CHECK with enum-like validation
  TextColumn get status => text()
    .customConstraint("CHECK (status IN ('active', 'inactive', 'pending'))")();
  
  // 👇 CHECK with multiple conditions
  RealColumn get price => real()
    .customConstraint('CHECK (price >= 0 AND price <= 999999.99)')();
  
  // 👇 Complex CHECK with logic
  TextColumn get zipCode => text()
    .customConstraint('CHECK (zipCode GLOB "[0-9][0-9][0-9][0-9][0-9]")')();
}

// Generated SQL:
// CREATE TABLE check_example (
//   age INTEGER NOT NULL CHECK (age >= 0),
//   score INTEGER NOT NULL CHECK (score >= 0) CHECK (score <= 100),
//   email TEXT NOT NULL CHECK (email LIKE "%@%"),
//   status TEXT NOT NULL CHECK (status IN ('active', 'inactive', 'pending')),
//   price REAL NOT NULL CHECK (price >= 0 AND price <= 999999.99),
//   zip_code TEXT NOT NULL CHECK (zipCode GLOB "[0-9][0-9][0-9][0-9][0-9]")
// )
```

> **Key insights:**
> - **CHECK constraints** – Validate data before insert/update
> - **Multiple CHECK constraints** – Can be added
> - **Complex conditions** – Use AND, OR, LIKE, GLOB
> - **String validation** – Validate format with LIKE
> - **Enum validation** – Restrict to allowed values

---

# PRIMARY KEY Constraints

> **Uniquely identifying rows**

```dart
class PrimaryKeyExample extends Table {
  // 👇 Single column primary key (auto-increment)
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Single column primary key (custom)
  @override
  ColumnKey get primaryKey => const ColumnKey('user_id');
  
  TextColumn get userId => text().named('user_id')();
  
  // 👇 Composite primary key
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  IntColumn get orderId => integer()();
  IntColumn get productId => integer()();
}

// Generated SQL:
// CREATE TABLE primary_key_example1 (
//   id INTEGER PRIMARY KEY AUTOINCREMENT
// )
//
// CREATE TABLE primary_key_example2 (
//   user_id TEXT PRIMARY KEY
// )
//
// CREATE TABLE primary_key_example3 (
//   order_id INTEGER NOT NULL,
//   product_id INTEGER NOT NULL,
//   PRIMARY KEY (order_id, product_id)
// )
```

> **Key insights:**
> - **Auto-increment** – Automatically generates IDs
> - **Composite keys** – Multiple columns as primary key
> - **Custom keys** – Use business keys as primary keys
> - **Performance** – Primary keys are automatically indexed

---

# FOREIGN KEY Constraints

> **Maintaining relationships between tables**

```dart
class ForeignKeyExample extends Table {
  // 👇 Simple foreign key
  IntColumn get userId => integer()
    .references(Users, #id)();
  
  // 👇 Foreign key with delete action
  IntColumn get categoryId => integer()
    .references(Categories, #id, onDelete: KeyAction.cascade)();
  
  // 👇 Foreign key with update action
  IntColumn get createdBy => integer()
    .references(Users, #id, onUpdate: KeyAction.setNull)();
  
  // 👇 Foreign key with both actions
  IntColumn get parentId => integer()
    .references(
      Categories, 
      #id, 
      onDelete: KeyAction.cascade,
      onUpdate: KeyAction.cascade,
    )();
  
  // 👇 Composite foreign key
  @override
  List<ForeignKey> get foreignKeys => [
    ForeignKey(
      columns: [orderId, productId],
      references: [OrderItems.id, OrderItems.productId],
      onDelete: KeyAction.cascade,
      onUpdate: KeyAction.cascade,
    ),
  ];
  
  IntColumn get orderId => integer()();
  IntColumn get productId => integer()();
}

// Generated SQL:
// CREATE TABLE foreign_key_example (
//   user_id INTEGER NOT NULL REFERENCES users(id),
//   category_id INTEGER NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
//   created_by INTEGER NOT NULL REFERENCES users(id) ON UPDATE SET NULL,
//   parent_id INTEGER NOT NULL REFERENCES categories(id) ON DELETE CASCADE ON UPDATE CASCADE,
//   order_id INTEGER NOT NULL,
//   product_id INTEGER NOT NULL,
//   FOREIGN KEY (order_id, product_id) REFERENCES order_items(id, product_id) ON DELETE CASCADE ON UPDATE CASCADE
// )
```

> **Key insights:**
> - **`onDelete`** – Action when referenced row is deleted
> - **`onUpdate`** – Action when referenced column is updated
> - **`KeyAction.cascade`** – Delete/update child rows
> - **`KeyAction.setNull`** – Set foreign key to NULL
> - **`KeyAction.restrict`** – Prevent delete/update
> - **`KeyAction.setDefault`** – Set to default value

---

# DEFERRABLE Constraints

> **Controlling when constraints are checked**

```dart
class DeferrableExample extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Foreign key with deferrable
  IntColumn get userId => integer()
    .references(
      Users, 
      #id,
      onDelete: KeyAction.cascade,
      // 👇 Defer constraint checking to transaction end
      deferred: true,
      initiallyDeferred: true,
    )();
  
  // 👇 Custom deferrable constraint
  @override
  List<String> get customConstraints => [
    'CONSTRAINT unique_pair UNIQUE (col1, col2) DEFERRABLE INITIALLY IMMEDIATE',
  ];
  
  TextColumn get col1 => text()();
  TextColumn get col2 => text()();
}

// Generated SQL:
// CREATE TABLE deferrable_example (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE DEFERRABLE INITIALLY DEFERRED,
//   col1 TEXT NOT NULL,
//   col2 TEXT NOT NULL,
//   CONSTRAINT unique_pair UNIQUE (col1, col2) DEFERRABLE INITIALLY IMMEDIATE
// )
```

> **Key insights:**
> - **DEFERRABLE** – Constraint check can be deferred
> - **INITIALLY DEFERRED** – Check at transaction commit
> - **INITIALLY IMMEDIATE** – Check immediately (default)
> - **Useful for circular references** – Where order matters

---

# Real-World Example

> **Complete table with all constraints**

```dart
// lib/database/tables/orders.dart
import 'package:drift/drift.dart';
import 'users.dart';
import 'products.dart';

class Orders extends Table {
  // 1️⃣ Primary Key
  IntColumn get id => integer().autoIncrement()();
  
  // 2️⃣ Foreign Key
  IntColumn get userId => integer()
    .references(
      Users, 
      #id,
      onDelete: KeyAction.cascade,
      onUpdate: KeyAction.cascade,
    )
    .customConstraint('NOT NULL')();
  
  // 3️⃣ Check constraints
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .customConstraint("CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))")();
  
  RealColumn get totalAmount => real()
    .customConstraint('CHECK (total_amount >= 0)')
    .named('total_amount')();
  
  DateTimeColumn get orderDate => dateTime()
    .withDefault(currentDateAndTime)
    .customConstraint('CHECK (order_date IS NOT NULL)')
    .named('order_date')();
  
  // 4️⃣ Unique constraints
  TextColumn get orderNumber => text()
    .unique()
    .customConstraint('CHECK (LENGTH(order_number) >= 8)')
    .named('order_number')();
  
  // 5️⃣ Complex constraints
  @override
  List<String> get customConstraints => [
    'CHECK (total_amount > 0 OR status == "cancelled")',
    'CHECK (order_date <= CURRENT_TIMESTAMP)',
  ];
  
  // 6️⃣ Indexes
  @override
  List<Index> get indexes => [
    Index('idx_orders_user_id', 'user_id'),
    Index('idx_orders_status', 'status'),
    Index('idx_orders_order_date', 'order_date'),
    Index('idx_orders_user_status', ['user_id', 'status']),
  ];
}

// lib/database/tables/order_items.dart
import 'package:drift/drift.dart';
import 'orders.dart';
import 'products.dart';

class OrderItems extends Table {
  // 1️⃣ Composite primary key
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  // 2️⃣ Foreign keys
  IntColumn get orderId => integer()
    .references(
      Orders, 
      #id,
      onDelete: KeyAction.cascade,
    )();
  
  IntColumn get productId => integer()
    .references(
      Products, 
      #id,
      onDelete: KeyAction.restrict,
    )();
  
  // 3️⃣ Check constraints
  IntColumn get quantity => integer()
    .customConstraint('CHECK (quantity > 0)')
    .withDefault(const Constant(1))();
  
  RealColumn get unitPrice => real()
    .customConstraint('CHECK (unit_price >= 0)')
    .named('unit_price')();
  
  RealColumn get totalPrice => real()
    .customConstraint('CHECK (total_price >= 0)')
    .named('total_price')();
  
  // 4️⃣ Complex constraint
  @override
  List<String> get customConstraints => [
    'CHECK (total_price = quantity * unit_price)',
  ];
  
  // 5️⃣ Indexes
  @override
  List<Index> get indexes => [
    Index('idx_order_items_order', 'order_id'),
    Index('idx_order_items_product', 'product_id'),
  ];
}
```

```dart
// lib/database/database.dart - Usage with constraints
import 'package:drift/drift.dart';
import 'tables/orders.dart';
import 'tables/order_items.dart';

@DriftDatabase(tables: [Orders, OrderItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // ✅ Insert with constraint validation
  Future<int> createOrder(int userId, List<OrderItemInput> items) async {
    return await transaction(() async {
      // 1️⃣ Create order
      final orderId = await into(orders).insert(
        OrdersCompanion.insert(
          userId: userId,
          orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
          totalAmount: 0.0, // Will be updated
        ),
      );
      
      // 2️⃣ Add items
      double total = 0.0;
      for (final item in items) {
        final product = await getProduct(item.productId);
        final price = product.price * item.quantity;
        total += price;
        
        await into(orderItems).insert(
          OrderItemsCompanion.insert(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: product.price,
            totalPrice: price,
          ),
        );
      }
      
      // 3️⃣ Update total
      await (update(orders)..where((o) => o.id.equals(orderId)))
        .write(OrdersCompanion(totalAmount: Value(total)));
      
      return orderId;
    });
  }

  // ✅ Handle constraint violations
  Future<void> cancelOrder(int orderId) async {
    try {
      await (update(orders)..where((o) => o.id.equals(orderId)))
        .write(const OrdersCompanion(status: Value('cancelled')));
    } catch (e) {
      if (e.toString().contains('CHECK constraint failed')) {
        throw Exception('Order cannot be cancelled in current status');
      }
      rethrow;
    }
  }

  Future<Product> getProduct(int id) async {
    // ... get product from database
    return Product(id: id, name: 'Product $id', price: 10.0);
  }
}

class OrderItemInput {
  final int productId;
  final int quantity;
  
  OrderItemInput({required this.productId, required this.quantity});
}

class Product {
  final int id;
  final String name;
  final double price;
  
  Product({required this.id, required this.name, required this.price});
}
```

---

# Best Practices

- **Always use FOREIGN KEYS** – Maintain referential integrity
- **Add CHECK constraints** – Validate data at database level
- **Use NOT NULL** – For required fields
- **Add UNIQUE constraints** – For unique identifiers
- **Use ON DELETE CASCADE** – Clean up related data
- **Use ON DELETE RESTRICT** – Prevent accidental deletes
- **Add indexes** – For frequently queried constraints
- **Use DEFERRABLE** – For circular references
- **Test constraints** – Verify they work as expected
- **Document constraints** – Explain complex rules

---

# Common Mistakes

## Mistake 1: Forgetting to enable foreign keys

Wrong:
```dart
// 🚫 Foreign keys not enabled
@override
Future<void> beforeOpen() async {
  // Missing PRAGMA foreign_keys = ON
}
```

Correct:
```dart
// ✅ Enable foreign keys
@override
Future<void> beforeOpen() async {
  await customSelect('PRAGMA foreign_keys = ON').get();
}
```

## Mistake 2: Using CHECK with NULL values

Wrong:
```dart
// 🚫 CHECK fails when value is NULL
IntColumn get age => integer()
  .nullable()
  .customConstraint('CHECK (age >= 0)')(); // NULL fails check
```

Correct:
```dart
// ✅ Handle NULL values in CHECK
IntColumn get age => integer()
  .nullable()
  .customConstraint('CHECK (age >= 0 OR age IS NULL)')();
```

## Mistake 3: Not handling constraint violations

Wrong:
```dart
// 🚫 App crashes on constraint violation
await db.into(users).insert(user);
```

Correct:
```dart
// ✅ Handle constraint errors
try {
  await db.into(users).insert(user);
} on DriftException catch (e) {
  if (e.message.contains('UNIQUE constraint failed')) {
    throw Exception('Email already exists');
  }
  rethrow;
}
```

---

# Summary

| Constraint | Purpose | Best Practice |
|------------|---------|---------------|
| **NOT NULL** | Required values | Use for required fields |
| **UNIQUE** | No duplicates | Use for emails, usernames |
| **CHECK** | Data validation | Use for business rules |
| **PRIMARY KEY** | Row identification | Auto-increment or business key |
| **FOREIGN KEY** | Relationships | Use CASCADE or RESTRICT |
| **DEFERRABLE** | Circular dependencies | Use sparingly |

---

# Next Steps

Now you understand constraints, let's dive deeper:

- [Primary Keys](link) – Complete primary key guide
- [Foreign Keys](link) – Advanced foreign key relationships
- [Default Values](link) – Advanced default values

---

# Did You Know?

- **SQLite supports up to 64 indexes** – On a single table

- **FOREIGN KEY constraints are OFF by default** – Must enable with PRAGMA

- **CHECK constraints are evaluated** – On INSERT and UPDATE

- **UNIQUE constraints create indexes** – Automatically for performance

- **DEFERRABLE constraints** – Allow circular references

- **PRIMARY KEY is automatically NOT NULL** – And UNIQUE

- **Foreign key actions** – CASCADE, RESTRICT, SET NULL, SET DEFAULT, NO ACTION

- **Constraint violations throw DriftException** – Catch and handle gracefully

---

