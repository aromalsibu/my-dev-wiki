## Generated Columns

**Creating computed columns in Drift**

---

# What is it?

**Generated Columns** are columns whose values are computed from other columns in the same table, rather than being explicitly inserted. They can be either stored in the database (taking up space but faster to read) or virtual (computed on-the-fly when queried). Drift supports both types with a simple, type-safe API.

> **Think of Generated Columns like "spreadsheet formulas"** – instead of manually calculating a value, you define the formula and the value is automatically computed whenever the source data changes.

```dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get firstName => text()();
  TextColumn get lastName => text()();
  
  // 👇 Generated column computing full name
  TextColumn get fullName => text()
    .generatedAs(
      // 👇 SQL expression
      const Constant("firstName || ' ' || lastName"),
      // 👇 Store in database (vs virtual)
      stored: true,
    )();
}

// When you insert:
// firstName = 'John', lastName = 'Doe'
// fullName = 'John Doe' (automatically computed)
```

> **What's happening here?**
> - **Expression** – SQL formula for computing the value
> - **Stored** – Value is persisted in the database
> - **Virtual** – Value is computed when queried
> - **Automatic** – No manual calculation needed

---

# Why does it exist?

- **Data Consistency** – Always computed from source data
- **Performance** – Pre-compute expensive calculations
- **Simplicity** – No need to calculate in application code
- **Indexing** – Generated columns can be indexed
- **Data Normalization** – Derive values from normalized data
- **Business Logic** – Enforce business rules at database level

---

# Stored vs Virtual Columns

> **Understanding the difference between stored and virtual generated columns**

## Stored Generated Columns

> **Values are physically stored in the database**

```dart
class StoredExample extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get firstName => text()();
  TextColumn get lastName => text()();
  
  // 👇 STORED generated column
  TextColumn get fullName => text()
    .generatedAs(
      const Constant("firstName || ' ' || lastName"),
      stored: true, // 👈 Persisted in database
    )();
  
  // 👇 STORED with numeric calculation
  IntColumn get age => integer()
    .generatedAs(
      const Constant("(strftime('%Y', 'now') - strftime('%Y', birthDate))"),
      stored: true,
    )();
  
  DateTimeColumn get birthDate => dateTime()();
}

// Generated SQL:
// CREATE TABLE stored_example (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   first_name TEXT NOT NULL,
//   last_name TEXT NOT NULL,
//   full_name TEXT GENERATED ALWAYS AS (first_name || ' ' || last_name) STORED,
//   age INTEGER GENERATED ALWAYS AS ((strftime('%Y', 'now') - strftime('%Y', birth_date))) STORED,
//   birth_date INTEGER NOT NULL
// )
```

> **Key insights:**
> - **STORED** – Value takes up disk space
> - **Faster reads** – No calculation on query
> - **Can be indexed** – For faster queries
> - **Updates automatically** – When source columns change

---

## Virtual Generated Columns

> **Values are computed when queried**

```dart
class VirtualExample extends Table {
  IntColumn get id => integer().autoIncrement()();
  RealColumn get price => real()();
  IntColumn get quantity => integer()();
  
  // 👇 VIRTUAL generated column (default)
  RealColumn get total => real()
    .generatedAs(
      const Constant("price * quantity"),
      // stored defaults to false
    )();
  
  // 👇 VIRTUAL text column
  TextColumn get status => text()
    .generatedAs(
      const Constant("CASE WHEN quantity > 0 THEN 'in stock' ELSE 'out of stock' END"),
    )();
}

// Generated SQL:
// CREATE TABLE virtual_example (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   price REAL NOT NULL,
//   quantity INTEGER NOT NULL,
//   total REAL GENERATED ALWAYS AS (price * quantity) VIRTUAL,
//   status TEXT GENERATED ALWAYS AS (CASE WHEN quantity > 0 THEN 'in stock' ELSE 'out of stock' END) VIRTUAL
// )
```

> **Key insights:**
> - **VIRTUAL** – No disk space used
> - **Slower reads** – Computed each time
> - **Cannot be indexed** – On virtual columns
> - **Updates automatically** – When source columns change

---

# Common Generated Column Patterns

> **Typical use cases for generated columns**

## Pattern 1: Full Name from First and Last Name

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get firstName => text().named('first_name')();
  TextColumn get lastName => text().named('last_name')();
  
  // 👇 Stored full name for faster queries
  TextColumn get fullName => text()
    .generatedAs(
      const Constant("TRIM(first_name || ' ' || last_name)"),
      stored: true,
    )
    .named('full_name')();
  
  // 👇 Virtual initials for quick display
  TextColumn get initials => text()
    .generatedAs(
      const Constant("SUBSTR(first_name, 1, 1) || SUBSTR(last_name, 1, 1)"),
    )
    .named('initials')();
}

// Generated SQL:
// CREATE TABLE users (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   first_name TEXT NOT NULL,
//   last_name TEXT NOT NULL,
//   full_name TEXT GENERATED ALWAYS AS (TRIM(first_name || ' ' || last_name)) STORED,
//   initials TEXT GENERATED ALWAYS AS (SUBSTR(first_name, 1, 1) || SUBSTR(last_name, 1, 1)) VIRTUAL
// )
```

---

## Pattern 2: Computed Totals

```dart
class OrderItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get quantity => integer()();
  RealColumn get unitPrice => real().named('unit_price')();
  RealColumn get discountPercent => real()
    .withDefault(const Constant(0.0))
    .named('discount_percent')();
  
  // 👇 Subtotal without discount
  RealColumn get subtotal => real()
    .generatedAs(
      const Constant("quantity * unit_price"),
      stored: true,
    )();
  
  // 👇 Discount amount
  RealColumn get discountAmount => real()
    .generatedAs(
      const Constant("subtotal * discount_percent / 100"),
      stored: true,
    )
    .named('discount_amount')();
  
  // 👇 Final total with discount
  RealColumn get total => real()
    .generatedAs(
      const Constant("subtotal - discount_amount"),
      stored: true,
    )();
  
  // 👇 Virtual tax calculation
  RealColumn get tax => real()
    .generatedAs(
      const Constant("total * 0.1"),
    )
    .named('tax')();
}

// Generated SQL:
// CREATE TABLE order_items (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   quantity INTEGER NOT NULL,
//   unit_price REAL NOT NULL,
//   discount_percent REAL NOT NULL DEFAULT 0.0,
//   subtotal REAL GENERATED ALWAYS AS (quantity * unit_price) STORED,
//   discount_amount REAL GENERATED ALWAYS AS (subtotal * discount_percent / 100) STORED,
//   total REAL GENERATED ALWAYS AS (subtotal - discount_amount) STORED,
//   tax REAL GENERATED ALWAYS AS (total * 0.1) VIRTUAL
// )
```

---

## Pattern 3: Conditional Status Based on Data

```dart
class InventoryItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  IntColumn get stock => integer()();
  IntColumn get reorderLevel => integer()
    .withDefault(const Constant(10))
    .named('reorder_level')();
  
  // 👇 Status based on stock levels
  TextColumn get status => text()
    .generatedAs(
      const Constant('''
        CASE 
          WHEN stock <= 0 THEN 'out_of_stock'
          WHEN stock < reorder_level THEN 'low_stock'
          ELSE 'in_stock'
        END
      '''),
      stored: true,
    )();
  
  // 👇 Virtual urgency indicator
  TextColumn get urgency => text()
    .generatedAs(
      const Constant('''
        CASE 
          WHEN stock <= 0 THEN 'immediate'
          WHEN stock < reorder_level THEN 'soon'
          ELSE 'none'
        END
      '''),
    )
    .named('urgency')();
  
  // 👇 Stock percentage (for UI progress bars)
  RealColumn get stockPercentage => real()
    .generatedAs(
      const Constant('''
        CASE 
          WHEN reorder_level = 0 THEN 100.0
          ELSE (stock * 100.0 / (reorder_level * 2))
        END
      '''),
      stored: true,
    )
    .named('stock_percentage')
    .customConstraint('CHECK (stock_percentage >= 0)')();
}

// Generated SQL:
// CREATE TABLE inventory_items (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   name TEXT NOT NULL,
//   stock INTEGER NOT NULL,
//   reorder_level INTEGER NOT NULL DEFAULT 10,
//   status TEXT GENERATED ALWAYS AS (
//     CASE 
//       WHEN stock <= 0 THEN 'out_of_stock'
//       WHEN stock < reorder_level THEN 'low_stock'
//       ELSE 'in_stock'
//     END
//   ) STORED,
//   urgency TEXT GENERATED ALWAYS AS (
//     CASE 
//       WHEN stock <= 0 THEN 'immediate'
//       WHEN stock < reorder_level THEN 'soon'
//       ELSE 'none'
//     END
//   ) VIRTUAL,
//   stock_percentage REAL GENERATED ALWAYS AS (
//     CASE 
//       WHEN reorder_level = 0 THEN 100.0
//       ELSE (stock * 100.0 / (reorder_level * 2))
//     END
//   ) STORED CHECK (stock_percentage >= 0)
// )
```

---

## Pattern 4: Date and Time Calculations

```dart
class Events extends Table {
  IntColumn get id => integer().autoIncrement()();
  DateTimeColumn get eventDate => dateTime().named('event_date')();
  TextColumn get eventName => text().named('event_name')();
  
  // 👇 Extract year from date
  IntColumn get year => integer()
    .generatedAs(
      const Constant("strftime('%Y', eventDate)"),
      stored: true,
    )();
  
  // 👇 Extract month name
  TextColumn get month => text()
    .generatedAs(
      const Constant("strftime('%B', eventDate)"),
      stored: true,
    )
    .named('month')();
  
  // 👇 Extract day of week
  TextColumn get dayOfWeek => text()
    .generatedAs(
      const Constant("strftime('%A', eventDate)"),
      stored: true,
    )
    .named('day_of_week')();
  
  // 👇 Is event in the future?
  BoolColumn get isUpcoming => boolean()
    .generatedAs(
      const Constant("eventDate > datetime('now')"),
    )
    .named('is_upcoming')();
  
  // 👇 Virtual: days until event
  IntColumn get daysUntil => integer()
    .generatedAs(
      const Constant("julianday(eventDate) - julianday('now')"),
    )
    .named('days_until')();
}

// Generated SQL:
// CREATE TABLE events (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   event_date INTEGER NOT NULL,
//   event_name TEXT NOT NULL,
//   year INTEGER GENERATED ALWAYS AS (strftime('%Y', event_date)) STORED,
//   month TEXT GENERATED ALWAYS AS (strftime('%B', event_date)) STORED,
//   day_of_week TEXT GENERATED ALWAYS AS (strftime('%A', event_date)) STORED,
//   is_upcoming INTEGER GENERATED ALWAYS AS (event_date > datetime('now')) VIRTUAL,
//   days_until INTEGER GENERATED ALWAYS AS (julianday(event_date) - julianday('now')) VIRTUAL
// )
```

---

# Generated Columns with Indexes

> **Indexing generated columns for performance**

```dart
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().named('order_number')();
  RealColumn get subtotal => real()();
  RealColumn get taxPercent => real()
    .withDefault(const Constant(0.1))
    .named('tax_percent')();
  
  // 👇 Generated total
  RealColumn get total => real()
    .generatedAs(
      const Constant("subtotal * (1 + taxPercent)"),
      stored: true, // Must be STORED to index
    )
    .named('total')();
  
  // 👇 Generated order status
  TextColumn get status => text()
    .generatedAs(
      const Constant('''
        CASE 
          WHEN isPaid = 1 AND isShipped = 1 THEN 'delivered'
          WHEN isPaid = 1 AND isShipped = 0 THEN 'processing'
          WHEN isPaid = 0 THEN 'pending'
          ELSE 'unknown'
        END
      '''),
      stored: true,
    )();
  
  BoolColumn get isPaid => boolean()
    .withDefault(const Constant(false))
    .named('is_paid')();
  
  BoolColumn get isShipped => boolean()
    .withDefault(const Constant(false))
    .named('is_shipped')();
  
  // 👇 Indexes on generated columns
  @override
  List<Index> get indexes => [
    // ✅ Can index STORED generated columns
    Index('idx_orders_status', 'status'),
    Index('idx_orders_total', 'total'),
    // ✅ Can index source columns
    Index('idx_orders_order_number', 'order_number'),
  ];
}

// Generated SQL:
// CREATE TABLE orders (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   order_number TEXT NOT NULL,
//   subtotal REAL NOT NULL,
//   tax_percent REAL NOT NULL DEFAULT 0.1,
//   total REAL GENERATED ALWAYS AS (subtotal * (1 + taxPercent)) STORED,
//   status TEXT GENERATED ALWAYS AS (
//     CASE 
//       WHEN is_paid = 1 AND is_shipped = 1 THEN 'delivered'
//       WHEN is_paid = 1 AND is_shipped = 0 THEN 'processing'
//       WHEN is_paid = 0 THEN 'pending'
//       ELSE 'unknown'
//     END
//   ) STORED,
//   is_paid INTEGER NOT NULL DEFAULT 0,
//   is_shipped INTEGER NOT NULL DEFAULT 0
// );
// CREATE INDEX idx_orders_status ON orders(status);
// CREATE INDEX idx_orders_total ON orders(total);
// CREATE INDEX idx_orders_order_number ON orders(order_number);
```

---

# Real-World Example

> **Complete e-commerce product catalog with generated columns**

```dart
// lib/database/tables/products.dart
import 'package:drift/drift.dart';

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get sku => text().unique().customConstraint('CHECK (LENGTH(sku) >= 6)')();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  
  // 👇 Price information
  RealColumn get basePrice => real()
    .customConstraint('CHECK (base_price >= 0)')
    .named('base_price')();
  
  RealColumn get discountPercent => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (discount_percent >= 0 AND discount_percent <= 100)')
    .named('discount_percent')();
  
  // 👇 Inventory
  IntColumn get stock => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (stock >= 0)')();
  
  IntColumn get reservedStock => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (reserved_stock >= 0)')
    .named('reserved_stock')();
  
  // 👇 Generated: current price with discount
  RealColumn get currentPrice => real()
    .generatedAs(
      const Constant("base_price * (1 - discount_percent / 100)"),
      stored: true,
    )
    .customConstraint('CHECK (current_price >= 0)')
    .named('current_price')();
  
  // 👇 Generated: available stock
  IntColumn get availableStock => integer()
    .generatedAs(
      const Constant("stock - reserved_stock"),
      stored: true,
    )
    .customConstraint('CHECK (available_stock >= 0)')
    .named('available_stock')();
  
  // 👇 Generated: stock status
  TextColumn get stockStatus => text()
    .generatedAs(
      const Constant('''
        CASE 
          WHEN stock - reserved_stock <= 0 THEN 'out_of_stock'
          WHEN stock - reserved_stock < 5 THEN 'low_stock'
          ELSE 'in_stock'
        END
      '''),
      stored: true,
    )
    .named('stock_status')();
  
  // 👇 Generated: product status
  TextColumn get productStatus => text()
    .generatedAs(
      const Constant('''
        CASE 
          WHEN isActive = 0 THEN 'inactive'
          WHEN stock - reserved_stock <= 0 THEN 'out_of_stock'
          WHEN isFeatured = 1 THEN 'featured'
          ELSE 'active'
        END
      ''')
    )
    .named('product_status')();
  
  // 👇 Flags
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))
    .named('is_active')();
  
  BoolColumn get isFeatured => boolean()
    .withDefault(const Constant(false))
    .named('is_featured')();
  
  BoolColumn get isNew => boolean()
    .generatedAs(
      const Constant("julianday('now') - julianday(created_at) < 30"),
    )
    .named('is_new')();
  
  // 👇 Timestamps
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
  
  DateTimeColumn get updatedAt => dateTime()
    .nullable()
    .named('updated_at')();
  
  // 👇 Rating
  RealColumn get rating => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (rating >= 0 AND rating <= 5)')();
  
  IntColumn get reviewCount => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (review_count >= 0)')
    .named('review_count')();
  
  // 👇 Indexes
  @override
  List<Index> get indexes => [
    // ✅ Index on STORED generated columns
    Index('idx_products_current_price', 'current_price'),
    Index('idx_products_stock_status', 'stock_status'),
    Index('idx_products_product_status', 'product_status'),
    Index('idx_products_available_stock', 'available_stock'),
    // ✅ Index on regular columns
    Index('idx_products_sku', 'sku'),
    Index('idx_products_is_active', 'is_active'),
    Index('idx_products_is_featured', 'is_featured'),
    Index('idx_products_created_at', 'created_at'),
  ];
}

// lib/database/tables/orders.dart
import 'package:drift/drift.dart';
import 'users.dart';
import 'products.dart';

class OrderItems extends Table {
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  IntColumn get orderId => integer()
    .references(Orders, #id, onDelete: KeyAction.cascade)
    .named('order_id')();
  
  IntColumn get productId => integer()
    .references(Products, #id, onDelete: KeyAction.restrict)
    .named('product_id')();
  
  IntColumn get quantity => integer()
    .customConstraint('CHECK (quantity > 0)')();
  
  // 👇 Price at time of order (snapshot)
  RealColumn get unitPrice => real()
    .customConstraint('CHECK (unit_price >= 0)')
    .named('unit_price')();
  
  RealColumn get discountPercent => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (discount_percent >= 0 AND discount_percent <= 100)')
    .named('discount_percent')();
  
  // 👇 Generated: subtotal
  RealColumn get subtotal => real()
    .generatedAs(
      const Constant("quantity * unit_price"),
      stored: true,
    )();
  
  // 👇 Generated: discount amount
  RealColumn get discountAmount => real()
    .generatedAs(
      const Constant("subtotal * discount_percent / 100"),
      stored: true,
    )
    .named('discount_amount')();
  
  // 👇 Generated: final total
  RealColumn get total => real()
    .generatedAs(
      const Constant("subtotal - discount_amount"),
      stored: true,
    )();
  
  // 👇 Generated: item status
  TextColumn get itemStatus => text()
    .generatedAs(
      const Constant('''
        CASE 
          WHEN isShipped = 1 AND isDelivered = 1 THEN 'delivered'
          WHEN isShipped = 1 AND isDelivered = 0 THEN 'shipped'
          ELSE 'pending'
        END
      ''')
    )
    .named('item_status')();
  
  BoolColumn get isShipped => boolean()
    .withDefault(const Constant(false))
    .named('is_shipped')();
  
  BoolColumn get isDelivered => boolean()
    .withDefault(const Constant(false))
    .named('is_delivered')();
  
  // 👇 Indexes
  @override
  List<Index> get indexes => [
    Index('idx_order_items_order', 'order_id'),
    Index('idx_order_items_product', 'product_id'),
    Index('idx_order_items_status', 'item_status'),
  ];
}
```

```dart
// lib/database/database.dart - Usage
import 'package:drift/drift.dart';
import 'tables/products.dart';
import 'tables/order_items.dart';

@DriftDatabase(tables: [Products, OrderItems])
class EcommerceDatabase extends _$EcommerceDatabase {
  EcommerceDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Using generated columns
  Future<List<Product>> getProductsInStock() async {
    // availableStock is generated automatically
    return await (select(products)
      ..where((p) => p.availableStock > const Variable(0))
      ..orderBy([(p) => OrderingTerm(expression: p.currentPrice)]))
      .get();
  }

  Future<List<Product>> getFeaturedProducts() async {
    return await (select(products)
      ..where((p) => p.isFeatured.equals(true))
      ..where((p) => p.isActive.equals(true))
      ..orderBy([(p) => OrderingTerm(expression: p.rating, mode: OrderingMode.desc)]))
      .get();
  }

  Future<List<Product>> getDiscountedProducts() async {
    return await (select(products)
      ..where((p) => p.discountPercent > const Variable(0.0))
      ..where((p) => p.availableStock > const Variable(0)))
      .get();
  }

  // 👇 Create order with generated columns
  Future<int> createOrder(int userId, List<CartItem> items) async {
    return await transaction(() async {
      // Create order
      final orderId = await into(orders).insert(
        OrdersCompanion.insert(
          userId: userId,
          orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
        ),
      );

      // Add items with generated columns
      for (final item in items) {
        final product = await getProduct(item.productId);
        
        // availableStock is automatically checked
        if (product.availableStock < item.quantity) {
          throw Exception('Insufficient stock for ${product.name}');
        }

        await into(orderItems).insert(
          OrderItemsCompanion.insert(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: product.currentPrice, // 👈 Use generated currentPrice
            discountPercent: product.discountPercent,
          ),
        );

        // Update stock (reserved)
        await (update(products)..where((p) => p.id.equals(product.id)))
          .write(ProductsCompanion(
            reservedStock: Value(product.reservedStock + item.quantity),
          ));
      }

      return orderId;
    });
  }

  Future<Product> getProduct(int id) async {
    final product = await (select(products)..where((p) => p.id.equals(id)))
        .getSingleOrNull();
    if (product == null) throw Exception('Product not found');
    return product;
  }
}

class CartItem {
  final int productId;
  final int quantity;
  
  CartItem({required this.productId, required this.quantity});
}
```

---

# Best Practices

- **Use STORED** – For frequently queried generated columns
- **Use VIRTUAL** – For rarely queried or expensive calculations
- **Index STORED columns** – For better query performance
- **Use meaningful expressions** – Make generated columns self-documenting
- **Test generated columns** – Verify they compute correctly
- **Consider performance** – STORED takes disk space, VIRTUAL takes CPU
- **Keep expressions simple** – Complex expressions are harder to maintain
- **Document calculations** – Explain business logic
- **Use CHECK constraints** – Validate generated values

---

# Common Mistakes

## Mistake 1: Trying to index VIRTUAL columns

Wrong:
```dart
// 🚫 Cannot index VIRTUAL generated columns
TextColumn get status => text()
  .generatedAs(const Constant("...")) // VIRTUAL by default

@override
List<Index> get indexes => [
  Index('idx_status', 'status'), // ❌ Will fail
];
```

Correct:
```dart
// ✅ Use STORED for indexed columns
TextColumn get status => text()
  .generatedAs(const Constant("..."), stored: true)();

@override
List<Index> get indexes => [
  Index('idx_status', 'status'), // ✅ Works
];
```

## Mistake 2: Using generated columns in INSERT

Wrong:
```dart
// 🚫 Cannot insert into generated columns
await into(products).insert(
  ProductsCompanion.insert(
    total: Value(100.0), // ❌ Generated column
  ),
);
```

Correct:
```dart
// ✅ Insert source columns only
await into(products).insert(
  ProductsCompanion.insert(
    price: 50.0,
    quantity: 2,
    // total is automatically computed
  ),
);
```

## Mistake 3: Complex expressions with SQL injection risk

Wrong:
```dart
// 🚫 String concatenation is risky
final expression = "price * $discount"; // Could be injection
```

Correct:
```dart
// ✅ Use Constant with safe SQL
const Constant("price * discount_percent / 100");
```

---

# Summary

| Aspect | STORED | VIRTUAL |
|--------|--------|---------|
| **Storage** | Takes disk space | No disk space |
| **Performance** | Faster reads | Slower reads |
| **Indexable** | Yes | No |
| **Update** | On source change | On query |
| **Use Case** | Frequent queries | Rare queries |

---

# Next Steps

Now you understand generated columns, let's dive deeper:

- [Custom Types](link) – Custom type converters
- [Table Inheritance](link) – Reusing table definitions
- [Indexes](link) – Advanced indexing strategies

---

# Did You Know?

- **Generated columns cannot be updated directly** – They're computed automatically

- **SQLite supports generated columns since version 3.31.0** – March 2020

- **STORED columns can be used in PRIMARY KEY** – But not VIRTUAL

- **Generated columns can reference other generated columns** – As long as they're STORED

- **Date calculations in SQLite** – Use `strftime()` and `julianday()` functions

- **CASE expressions are powerful** – For conditional generated values

- **Generated columns can have CHECK constraints** – To validate computed values

- **Expressions can include subqueries** – In some SQLite versions

---
