## Composite Keys

**Mastering multi-column primary keys in Drift**

---

# What is it?

**Composite Keys** are primary keys that consist of two or more columns combined to uniquely identify a row. Instead of a single column, the combination of multiple columns must be unique. This is essential for junction tables and natural relationships where a single column isn't enough for unique identification.

> **Think of Composite Keys like "coordinates on a map"** – a single coordinate (like latitude alone) isn't unique, but the combination of latitude AND longitude pinpoints exactly one location.

```dart
class OrderItems extends Table {
  // 👇 Composite primary key: orderId + productId
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  IntColumn get orderId => integer()();
  IntColumn get productId => integer()();
  IntColumn get quantity => integer()();
}

// This ensures:
// (orderId, productId) is unique
// Example: (1, 1), (1, 2), (2, 1) - all valid
//          (1, 1) - cannot be inserted twice
```

> **What's happening here?**
> - **Multiple columns** – Two or more columns form the primary key
> - **Uniqueness** – The combination of all columns must be unique
> - **Indexing** – Composite key creates a multi-column index
> - **Foreign Keys** – Can be referenced by composite foreign keys

---

# Why does it exist?

- **Natural Relationships** – Many-to-many relationships need composite keys
- **Uniqueness** – Sometimes a single column isn't enough to identify a row
- **Data Integrity** – Prevents duplicate combinations
- **Performance** – Composite indexes for common query patterns
- **Business Logic** – Reflects real-world unique constraints

---

# Defining Composite Keys

> **Different ways to define composite primary keys**

## Simple Composite Key

```dart
class OrderItems extends Table {
  // 👇 Composite key with two columns
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  IntColumn get orderId => integer()();
  IntColumn get productId => integer()();
  IntColumn get quantity => integer()();
  RealColumn get price => real()();
}

// Generated SQL:
// CREATE TABLE order_items (
//   order_id INTEGER NOT NULL,
//   product_id INTEGER NOT NULL,
//   quantity INTEGER NOT NULL,
//   price REAL NOT NULL,
//   PRIMARY KEY (order_id, product_id)
// )
```

## Composite Key with Three Columns

```dart
class Enrollments extends Table {
  // 👇 Composite key with three columns
  @override
  Set<Column> get primaryKey => {studentId, courseId, semester};
  
  IntColumn get studentId => integer().named('student_id')();
  IntColumn get courseId => integer().named('course_id')();
  TextColumn get semester => text().named('semester')();
  TextColumn get grade => text().nullable()();
  DateTimeColumn get enrollmentDate => dateTime()
    .withDefault(currentDateAndTime)
    .named('enrollment_date')();
}

// Generated SQL:
// CREATE TABLE enrollments (
//   student_id INTEGER NOT NULL,
//   course_id INTEGER NOT NULL,
//   semester TEXT NOT NULL,
//   grade TEXT,
//   enrollment_date INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   PRIMARY KEY (student_id, course_id, semester)
// )
```

## Composite Key with Custom Column Names

```dart
class ProductAttributes extends Table {
  @override
  Set<Column> get primaryKey => {productCode, attributeName};
  
  TextColumn get productCode => text()
    .named('product_code')
    .customConstraint('CHECK (LENGTH(product_code) >= 6)')();
  
  TextColumn get attributeName => text()
    .named('attribute_name')();
  
  TextColumn get attributeValue => text()
    .named('attribute_value')();
}

// Generated SQL:
// CREATE TABLE product_attributes (
//   product_code TEXT NOT NULL CHECK (LENGTH(product_code) >= 6),
//   attribute_name TEXT NOT NULL,
//   attribute_value TEXT NOT NULL,
//   PRIMARY KEY (product_code, attribute_name)
// )
```

---

# Composite Foreign Keys

> **Referencing composite primary keys**

## Self-Contained Example

```dart
// 1️⃣ Parent table with composite primary key
class OrderItems extends Table {
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  IntColumn get orderId => integer().named('order_id')();
  IntColumn get productId => integer().named('product_id')();
  IntColumn get quantity => integer()();
}

// 2️⃣ Child table referencing composite key
class OrderItemDetails extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Composite foreign key referencing OrderItems
  @override
  List<ForeignKey> get foreignKeys => [
    ForeignKey(
      columns: [orderId, productId],
      references: [OrderItems.orderId, OrderItems.productId],
      onDelete: KeyAction.cascade,
      onUpdate: KeyAction.cascade,
    ),
  ];
  
  IntColumn get orderId => integer().named('order_id')();
  IntColumn get productId => integer().named('product_id')();
  TextColumn get notes => text().nullable()();
  DateTimeColumn get processedAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('processed_at')();
}

// Generated SQL:
// CREATE TABLE order_item_details (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   order_id INTEGER NOT NULL,
//   product_id INTEGER NOT NULL,
//   notes TEXT,
//   processed_at INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   FOREIGN KEY (order_id, product_id) 
//     REFERENCES order_items(order_id, product_id) 
//     ON DELETE CASCADE ON UPDATE CASCADE
// )
```

## Complete Many-to-Many Example

```dart
// 1️⃣ Users table
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
}

// 2️⃣ Roles table
class Roles extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().unique()();
  TextColumn get description => text().nullable()();
}

// 3️⃣ Junction table with composite primary key
class UserRoles extends Table {
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
  
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))
    .named('is_active')();
  
  // 👇 Additional indexes for performance
  @override
  List<Index> get indexes => [
    Index('idx_user_roles_user', 'user_id'),
    Index('idx_user_roles_role', 'role_id'),
    Index('idx_user_roles_active', 'is_active'),
  ];
}

// Generated SQL:
// CREATE TABLE user_roles (
//   user_id INTEGER NOT NULL REFERENCES users(id),
//   role_id INTEGER NOT NULL REFERENCES roles(id),
//   assigned_at INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   is_active INTEGER NOT NULL DEFAULT 1,
//   PRIMARY KEY (user_id, role_id)
// );
// CREATE INDEX idx_user_roles_user ON user_roles(user_id);
// CREATE INDEX idx_user_roles_role ON user_roles(role_id);
// CREATE INDEX idx_user_roles_active ON user_roles(is_active);
```

---

# Composite Unique Constraints

> **Alternative to composite primary keys**

```dart
class ProductVariants extends Table {
  // 👇 Use auto-increment primary key
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Composite unique constraint instead of primary key
  @override
  List<Modifier> get constraints => [
    const Modifier.unique(
      ['productId', 'size', 'color'], 
      'unique_variant',
    ),
  ];
  
  IntColumn get productId => integer()
    .references(Products, #id)
    .named('product_id')();
  
  TextColumn get size => text()
    .named('size')
    .customConstraint("CHECK (size IN ('S', 'M', 'L', 'XL'))")();
  
  TextColumn get color => text()
    .named('color')();
  
  IntColumn get stock => integer()
    .withDefault(const Constant(0))();
  
  RealColumn get price => real()
    .customConstraint('CHECK (price >= 0)')();
}

// Generated SQL:
// CREATE TABLE product_variants (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   product_id INTEGER NOT NULL REFERENCES products(id),
//   size TEXT NOT NULL CHECK (size IN ('S', 'M', 'L', 'XL')),
//   color TEXT NOT NULL,
//   stock INTEGER NOT NULL DEFAULT 0,
//   price REAL NOT NULL CHECK (price >= 0),
//   CONSTRAINT unique_variant UNIQUE (product_id, size, color)
// )
```

---

# Composite Key Queries

> **Querying tables with composite keys**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'tables/order_items.dart';
import 'tables/user_roles.dart';

@DriftDatabase(tables: [OrderItems, UserRoles])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Query with composite key (both columns)
  Future<OrderItem?> getOrderItem(int orderId, int productId) async {
    return await (select(orderItems)
      ..where((o) => o.orderId.equals(orderId) & o.productId.equals(productId)))
      .getSingleOrNull();
  }

  // 👇 Query by part of composite key
  Future<List<OrderItem>> getOrderItemsByOrder(int orderId) async {
    return await (select(orderItems)
      ..where((o) => o.orderId.equals(orderId)))
      .get();
  }

  // 👇 Query with multiple conditions
  Future<List<OrderItem>> getOrderItemsWithPriceRange(
    int orderId,
    double minPrice,
    double maxPrice,
  ) async {
    return await (select(orderItems)
      ..where((o) => o.orderId.equals(orderId))
      ..where((o) => o.price.isBetweenValues(minPrice, maxPrice)))
      .get();
  }

  // 👇 Query user roles
  Future<List<UserRole>> getUserRoles(int userId) async {
    return await (select(userRoles)
      ..where((ur) => ur.userId.equals(userId)))
      .get();
  }

  // 👇 Check if user has role
  Future<bool> userHasRole(int userId, int roleId) async {
    final result = await (select(userRoles)
      ..where((ur) => ur.userId.equals(userId) & ur.roleId.equals(roleId)))
      .get();
    return result.isNotEmpty;
  }

  // 👇 Assign role (insert with composite key)
  Future<void> assignRole(int userId, int roleId) async {
    await into(userRoles).insert(
      UserRolesCompanion.insert(
        userId: userId,
        roleId: roleId,
        isActive: true,
      ),
    );
  }

  // 👇 Remove role (delete by composite key)
  Future<void> removeRole(int userId, int roleId) async {
    await (delete(userRoles)
      ..where((ur) => ur.userId.equals(userId) & ur.roleId.equals(roleId)))
      .go();
  }
}
```

---

# Real-World Example

> **Complete e-commerce system with composite keys**

```dart
// lib/database/tables/orders.dart
import 'package:drift/drift.dart';
import 'users.dart';

class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().unique()();
  IntColumn get userId => integer().references(Users, #id)();
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .customConstraint("CHECK (status IN ('pending', 'paid', 'shipped', 'delivered', 'cancelled'))")();
  DateTimeColumn get orderDate => dateTime().withDefault(currentDateAndTime)();
  RealColumn get total => real().customConstraint('CHECK (total >= 0)')();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
}

// lib/database/tables/products.dart
import 'package:drift/drift.dart';

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get sku => text().unique().customConstraint('CHECK (LENGTH(sku) >= 6)')();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  RealColumn get price => real().customConstraint('CHECK (price >= 0)')();
  IntColumn get stock => integer().withDefault(const Constant(0))();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
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
    .references(Products, #id, onDelete: KeyAction.restrict)
    .named('product_id')();
  
  // 👇 Data columns
  IntColumn get quantity => integer()
    .customConstraint('CHECK (quantity > 0)')();
  
  RealColumn get unitPrice => real()
    .customConstraint('CHECK (unit_price >= 0)')
    .named('unit_price')();
  
  RealColumn get subtotal => real()
    .customConstraint('CHECK (subtotal >= 0)')
    .named('subtotal')();
  
  RealColumn get discount => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (discount >= 0)')
    .named('discount')();
  
  RealColumn get total => real()
    .customConstraint('CHECK (total >= 0)')
    .named('total')();
  
  // 👇 Additional index
  @override
  List<Index> get indexes => [
    Index('idx_order_items_order', 'order_id'),
    Index('idx_order_items_product', 'product_id'),
  ];
}

// lib/database/tables/order_item_details.dart
import 'package:drift/drift.dart';
import 'order_items.dart';

class OrderItemDetails extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Composite foreign key referencing OrderItems
  @override
  List<ForeignKey> get foreignKeys => [
    ForeignKey(
      columns: [orderId, productId],
      references: [OrderItems.orderId, OrderItems.productId],
      onDelete: KeyAction.cascade,
      onUpdate: KeyAction.cascade,
    ),
  ];
  
  IntColumn get orderId => integer().named('order_id')();
  IntColumn get productId => integer().named('product_id')();
  TextColumn get notes => text().nullable()();
  TextColumn get shippingAddress => text().named('shipping_address')();
  DateTimeColumn get processedAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('processed_at')();
  BoolColumn get isGift => boolean()
    .withDefault(const Constant(false))
    .named('is_gift')();
}

// lib/database/database.dart - Usage
import 'package:drift/drift.dart';
import 'tables/orders.dart';
import 'tables/products.dart';
import 'tables/order_items.dart';
import 'tables/order_item_details.dart';

@DriftDatabase(tables: [
  Orders,
  Products,
  OrderItems,
  OrderItemDetails,
])
class EcommerceDatabase extends _$EcommerceDatabase {
  EcommerceDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Create order with items
  Future<int> createOrder(
    int userId,
    List<CartItem> items,
  ) async {
    return await transaction(() async {
      // 1️⃣ Create order
      final orderId = await into(orders).insert(
        OrdersCompanion.insert(
          orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
          userId: userId,
          total: 0,
        ),
      );
      
      double orderTotal = 0;
      
      // 2️⃣ Add items
      for (final item in items) {
        final product = await getProduct(item.productId);
        
        // Check stock
        if (product.stock < item.quantity) {
          throw Exception('Insufficient stock for product: ${product.name}');
        }
        
        final subtotal = product.price * item.quantity;
        final total = subtotal - item.discount;
        orderTotal += total;
        
        // Insert order item
        await into(orderItems).insert(
          OrderItemsCompanion.insert(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: product.price,
            subtotal: subtotal,
            discount: item.discount,
            total: total,
          ),
        );
        
        // Add item details
        if (item.notes != null || item.isGift) {
          await into(orderItemDetails).insert(
            OrderItemDetailsCompanion.insert(
              orderId: orderId,
              productId: item.productId,
              notes: Value(item.notes),
              isGift: item.isGift,
            ),
          );
        }
        
        // Update stock
        await (update(products)..where((p) => p.id.equals(product.id)))
          .write(ProductsCompanion(stock: Value(product.stock - item.quantity)));
      }
      
      // 3️⃣ Update order total
      await (update(orders)..where((o) => o.id.equals(orderId)))
        .write(OrdersCompanion(total: Value(orderTotal)));
      
      return orderId;
    });
  }

  // 👇 Get order with all items
  Future<OrderWithItems> getOrder(int orderId) async {
    final order = await (select(orders)..where((o) => o.id.equals(orderId)))
        .getSingleOrNull();
    
    if (order == null) throw Exception('Order not found');
    
    final items = await (select(orderItems)
      ..where((i) => i.orderId.equals(orderId)))
      .get();
    
    // Get details for each item
    final details = <int, OrderItemDetail>{};
    for (final item in items) {
      final detail = await (select(orderItemDetails)
        ..where((d) => 
          d.orderId.equals(orderId) & 
          d.productId.equals(item.productId)
        ))
        .getSingleOrNull();
      
      if (detail != null) {
        details[item.productId] = detail;
      }
    }
    
    return OrderWithItems(
      order: order,
      items: items,
      details: details,
    );
  }

  // 👇 Get order items with product details
  Future<List<OrderItemWithProduct>> getOrderItemsWithProducts(int orderId) async {
    final results = await customSelect('''
      SELECT 
        oi.*,
        p.name as product_name,
        p.sku as product_sku,
        p.description as product_description
      FROM order_items oi
      INNER JOIN products p ON oi.product_id = p.id
      WHERE oi.order_id = ?
      ORDER BY oi.product_id
    ''', variables: [Variable.withInt(orderId)]).get();
    
    return results.map((row) {
      return OrderItemWithProduct(
        orderItem: OrderItem(
          orderId: row.data['order_id'] as int,
          productId: row.data['product_id'] as int,
          quantity: row.data['quantity'] as int,
          unitPrice: row.data['unit_price'] as double,
          subtotal: row.data['subtotal'] as double,
          discount: row.data['discount'] as double,
          total: row.data['total'] as double,
        ),
        productName: row.data['product_name'] as String,
        productSku: row.data['product_sku'] as String,
        productDescription: row.data['product_description'] as String?,
      );
    }).toList();
  }

  Future<Product> getProduct(int id) async {
    final product = await (select(products)..where((p) => p.id.equals(id)))
        .getSingleOrNull();
    if (product == null) throw Exception('Product not found');
    return product;
  }
}

// 👇 Data classes
class CartItem {
  final int productId;
  final int quantity;
  final double discount;
  final String? notes;
  final bool isGift;
  
  CartItem({
    required this.productId,
    required this.quantity,
    this.discount = 0,
    this.notes,
    this.isGift = false,
  });
}

class OrderWithItems {
  final Order order;
  final List<OrderItem> items;
  final Map<int, OrderItemDetail> details;
  
  OrderWithItems({
    required this.order,
    required this.items,
    required this.details,
  });
}

class OrderItemWithProduct {
  final OrderItem orderItem;
  final String productName;
  final String productSku;
  final String? productDescription;
  
  OrderItemWithProduct({
    required this.orderItem,
    required this.productName,
    required this.productSku,
    required this.productDescription,
  });
}
```

---

# Best Practices

- **Use composite keys only when necessary** – Single column keys are simpler
- **Order columns by query patterns** – Most frequently queried column first
- **Add indexes on composite key parts** – For partial queries
- **Use meaningful column names** – In composite keys
- **Consider performance** – Composite keys are slower than single keys
- **Document composite relationships** – Explain why they're composite
- **Use FOREIGN KEY constraints** – Maintain referential integrity
- **Test query performance** – With composite keys
- **Consider alternatives** – Surrogate keys with unique constraints
- **Use composite keys for junction tables** – Natural fit

---

# Common Mistakes

## Mistake 1: Wrong column order in composite keys

Wrong:
```dart
// 🚫 Less common query pattern first
@override
Set<Column> get primaryKey => {productId, orderId};
```

Correct:
```dart
// ✅ Most frequent query column first
@override
Set<Column> get primaryKey => {orderId, productId};
```

## Mistake 2: Not handling part of composite key queries

Wrong:
```dart
// 🚫 Query by part of composite key is slow
final items = await select(orderItems).get(); // Then filter in Dart
```

Correct:
```dart
// ✅ Use WHERE clause for partial key
final items = await (select(orderItems)
  ..where((i) => i.orderId.equals(orderId)))
  .get();
```

## Mistake 3: Missing foreign key references

Wrong:
```dart
// 🚫 No foreign key constraint
@override
Set<Column> get primaryKey => {orderId, productId};
```

Correct:
```dart
// ✅ Add foreign key constraints
@override
List<ForeignKey> get foreignKeys => [
  ForeignKey(
    columns: [orderId],
    references: [Orders.id],
  ),
  ForeignKey(
    columns: [productId],
    references: [Products.id],
  ),
];
```

---

# Summary

| Aspect | Best Practice | Purpose |
|--------|---------------|---------|
| **Key Definition** | `primaryKey: {col1, col2}` | Uniqueness |
| **Column Order** | Most queried first | Performance |
| **Foreign Keys** | Reference composite keys | Integrity |
| **Indexes** | On key parts | Query speed |
| **Junction Tables** | Natural for many-to-many | Relationships |

---

# Next Steps

Now you understand composite keys, let's dive deeper:

- [Foreign Keys](link) – Advanced foreign key relationships
- [Default Values](link) – Setting default values
- [Generated Columns](link) – Computed columns

---

# Did You Know?

- **Composite keys create multi-column indexes** – For fast lookups

- **SQLite can use partial composite indexes** – For performance

- **Composite foreign keys must match exactly** – Same columns and order

- **The order of columns in composite keys matters** – For query performance

- **AUTOINCREMENT is not allowed on composite keys** – Only single columns

- **Composite keys can be up to 64 columns** – But not recommended

- **Each column in a composite key is NOT NULL** – By default

- **Composite unique constraints are similar** – But not primary keys

---

