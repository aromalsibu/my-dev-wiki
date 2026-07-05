## Foreign Keys

**Building relationships between tables with Drift**

---

# What is it?

**Foreign Keys** are constraints that link rows in one table to rows in another table. They ensure referential integrity by preventing orphaned records and maintaining relationships between your data. Drift provides a type-safe way to define foreign keys with configurable actions for updates and deletes.

> **Think of Foreign Keys like "family connections"** – a child has a parent, and you can't have a child without a parent. If the parent is removed, the child either goes with them (CASCADE) or is left without a parent (SET NULL).

```dart
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  
  // 👇 Foreign key linking to Users table
  IntColumn get userId => integer()
    .references(Users, #id, 
      onDelete: KeyAction.cascade,
      onUpdate: KeyAction.cascade,
    )
    .named('user_id')();
}

// This ensures:
// 1. userId must exist in Users table
// 2. If user is deleted, their posts are deleted
// 3. If user ID changes, posts are updated
```

> **What's happening here?**
> - **Reference** – Links to another table's column
> - **On Delete** – Action when referenced row is deleted
> - **On Update** – Action when referenced column is updated
> - **Integrity** – Prevents orphaned records

---

# Why does it exist?

- **Referential Integrity** – Maintain valid relationships between tables
- **Data Consistency** – Prevent orphaned records
- **Automatic Cleanup** – Cascade deletes remove related data
- **Query Optimization** – Foreign keys are often indexed
- **Documentation** – Self-documenting relationships
- **Error Prevention** – Catch invalid references early

---

# Basic Foreign Keys

> **Simple foreign key definitions**

## Single Column Foreign Key

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
}

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  
  // 👇 Basic foreign key
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

## Foreign Key with NULL

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  
  // 👇 Optional foreign key (allows NULL)
  IntColumn get userId => integer()
    .references(Users, #id)
    .nullable()
    .named('user_id')();
}

// Generated SQL:
// CREATE TABLE posts (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   title TEXT NOT NULL,
//   content TEXT NOT NULL,
//   user_id INTEGER REFERENCES users(id)
// )
```

---

# Foreign Key Actions

> **Controlling what happens when referenced data changes**

## ON DELETE CASCADE

> **Delete child records when parent is deleted**

```dart
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().unique()();
}

class OrderItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 CASCADE delete
  IntColumn get orderId => integer()
    .references(
      Orders, 
      #id,
      onDelete: KeyAction.cascade, // 👈 Delete items when order is deleted
    )
    .named('order_id')();
  
  IntColumn get productId => integer()();
  IntColumn get quantity => integer()();
}

// When an order is deleted:
// DELETE FROM orders WHERE id = 1;
// 🔄 Automatically: DELETE FROM order_items WHERE order_id = 1;
```

## ON DELETE RESTRICT

> **Prevent deletion if child records exist**

```dart
class Categories extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().unique()();
}

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 👇 RESTRICT delete
  IntColumn get categoryId => integer()
    .references(
      Categories, 
      #id,
      onDelete: KeyAction.restrict, // 👈 Prevent deleting categories with products
    )
    .named('category_id')();
}

// When trying to delete a category with products:
// DELETE FROM categories WHERE id = 1;
// ❌ Foreign key constraint violation: products still reference this category
```

## ON DELETE SET NULL

> **Set foreign key to NULL when parent is deleted**

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  
  // 👇 SET NULL on delete
  IntColumn get userId => integer()
    .references(
      Users, 
      #id,
      onDelete: KeyAction.setNull, // 👈 Set to NULL when user is deleted
    )
    .nullable()
    .named('user_id')();
}

// When a user is deleted:
// DELETE FROM users WHERE id = 1;
// 🔄 Automatically: UPDATE posts SET user_id = NULL WHERE user_id = 1;
```

## ON DELETE SET DEFAULT

> **Set foreign key to default value when parent is deleted**

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  
  // 👇 SET DEFAULT on delete
  IntColumn get userId => integer()
    .references(
      Users, 
      #id,
      onDelete: KeyAction.setDefault, // 👈 Set to default when user is deleted
    )
    .withDefault(const Constant(1)) // 👈 Default user ID
    .named('user_id')();
}

// When a user is deleted:
// DELETE FROM users WHERE id = 1;
// 🔄 Automatically: UPDATE posts SET user_id = 1 WHERE user_id = 1;
```

---

# Foreign Key Actions Reference

> **Complete reference of all actions**

| Action | Effect | Use Case |
|--------|--------|----------|
| **CASCADE** | Delete/update child records | Orders and order items |
| **RESTRICT** | Prevent parent deletion | Categories with products |
| **SET NULL** | Set foreign key to NULL | Posts when author is deleted |
| **SET DEFAULT** | Set to default value | Assign to default user |
| **NO ACTION** | Like RESTRICT but deferred | Complex relationships |

## ON UPDATE Actions

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
}

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  
  // 👇 CASCADE on update
  IntColumn get userId => integer()
    .references(
      Users, 
      #id,
      onUpdate: KeyAction.cascade, // 👈 Update user ID in posts when user ID changes
    )
    .named('user_id')();
}

// When a user ID is updated:
// UPDATE users SET id = 100 WHERE id = 1;
// 🔄 Automatically: UPDATE posts SET user_id = 100 WHERE user_id = 1;
```

---

# Composite Foreign Keys

> **Foreign keys referencing composite primary keys**

```dart
// 1️⃣ Parent table with composite primary key
class OrderItems extends Table {
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  IntColumn get orderId => integer().named('order_id')();
  IntColumn get productId => integer().named('product_id')();
  IntColumn get quantity => integer()();
}

// 2️⃣ Child table with composite foreign key
class OrderItemDetails extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Composite foreign key
  @override
  List<ForeignKey> get foreignKeys => [
    ForeignKey(
      columns: [orderId, productId],  // 👈 Child columns
      references: [OrderItems.orderId, OrderItems.productId], // 👈 Parent columns
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

---

# Self-Referencing Foreign Keys

> **Foreign keys that reference the same table**

```dart
class Employees extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  
  // 👇 Self-referencing foreign key (manager)
  IntColumn get managerId => integer()
    .references(Employees, #id, onDelete: KeyAction.setNull)
    .nullable()
    .named('manager_id')();
  
  TextColumn get department => text()();
  DateTimeColumn get hireDate => dateTime().withDefault(currentDateAndTime)();
  
  @override
  List<Index> get indexes => [
    Index('idx_employees_manager', 'manager_id'),
    Index('idx_employees_department', 'department'),
  ];
}

// This creates a hierarchy:
// Employee 1: John (manager_id = null)
//   Employee 2: Alice (manager_id = 1)
//     Employee 3: Bob (manager_id = 2)
//   Employee 4: Charlie (manager_id = 1)
```

---

# Real-World Example

> **Complete e-commerce database with foreign keys**

```dart
// lib/database/tables/users.dart
import 'package:drift/drift.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get fullName => text().nullable()();
  TextColumn get passwordHash => text().named('password_hash')();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  BoolColumn get isVerified => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get lastLogin => dateTime().nullable()();
}

// lib/database/tables/addresses.dart
import 'package:drift/drift.dart';
import 'users.dart';

class Addresses extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Foreign key with cascade
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)
    .named('user_id')();
  
  TextColumn get street => text()();
  TextColumn get city => text()();
  TextColumn get state => text()();
  TextColumn get zipCode => text().named('zip_code')();
  TextColumn get country => text()();
  BoolColumn get isDefault => boolean()
    .withDefault(const Constant(false))
    .named('is_default')();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/tables/categories.dart
import 'package:drift/drift.dart';

class Categories extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().unique()();
  TextColumn get description => text().nullable()();
  
  // 👇 Self-referencing foreign key (subcategories)
  IntColumn get parentId => integer()
    .references(Categories, #id, onDelete: KeyAction.setNull)
    .nullable()
    .named('parent_id')();
  
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/tables/products.dart
import 'package:drift/drift.dart';
import 'categories.dart';

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get sku => text().unique()();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  TextColumn get shortDescription => text().nullable().named('short_description')();
  
  // 👇 Foreign key with restrict (prevent deletion)
  IntColumn get categoryId => integer()
    .references(Categories, #id, onDelete: KeyAction.restrict)
    .named('category_id')();
  
  RealColumn get price => real().customConstraint('CHECK (price >= 0)')();
  RealColumn get cost => real().nullable().customConstraint('CHECK (cost >= 0)')();
  IntColumn get stock => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (stock >= 0)')();
  IntColumn get reorderLevel => integer()
    .withDefault(const Constant(10))
    .named('reorder_level')();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}

// lib/database/tables/orders.dart
import 'package:drift/drift.dart';
import 'users.dart';
import 'addresses.dart';

class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().unique()();
  
  // 👇 Foreign key with restrict
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.restrict)
    .named('user_id')();
  
  // 👇 Foreign key with set null
  IntColumn get billingAddressId => integer()
    .references(Addresses, #id, onDelete: KeyAction.setNull)
    .nullable()
    .named('billing_address_id')();
  
  IntColumn get shippingAddressId => integer()
    .references(Addresses, #id, onDelete: KeyAction.setNull)
    .nullable()
    .named('shipping_address_id')();
  
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .customConstraint("CHECK (status IN ('pending', 'processing', 'paid', 'shipped', 'delivered', 'cancelled'))")();
  
  TextColumn get paymentStatus => text()
    .withDefault(const Constant('unpaid'))
    .named('payment_status')
    .customConstraint("CHECK (payment_status IN ('unpaid', 'paid', 'refunded'))")();
  
  TextColumn get paymentMethod => text()
    .nullable()
    .named('payment_method')();
  
  RealColumn get subtotal => real().customConstraint('CHECK (subtotal >= 0)')();
  RealColumn get tax => real().withDefault(const Constant(0.0)).customConstraint('CHECK (tax >= 0)')();
  RealColumn get shippingCost => real()
    .withDefault(const Constant(0.0))
    .named('shipping_cost')
    .customConstraint('CHECK (shipping_cost >= 0)')();
  RealColumn get discount => real().withDefault(const Constant(0.0)).customConstraint('CHECK (discount >= 0)')();
  RealColumn get total => real().customConstraint('CHECK (total >= 0)')();
  
  DateTimeColumn get orderDate => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get shippedDate => dateTime().nullable().named('shipped_date')();
  DateTimeColumn get deliveredDate => dateTime().nullable().named('delivered_date')();
}

// lib/database/tables/order_items.dart
import 'package:drift/drift.dart';
import 'orders.dart';
import 'products.dart';

class OrderItems extends Table {
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  // 👇 Foreign key with cascade
  IntColumn get orderId => integer()
    .references(Orders, #id, onDelete: KeyAction.cascade)
    .named('order_id')();
  
  // 👇 Foreign key with restrict
  IntColumn get productId => integer()
    .references(Products, #id, onDelete: KeyAction.restrict)
    .named('product_id')();
  
  IntColumn get quantity => integer()
    .customConstraint('CHECK (quantity > 0)')();
  
  RealColumn get unitPrice => real()
    .customConstraint('CHECK (unit_price >= 0)')
    .named('unit_price')();
  
  RealColumn get subtotal => real()
    .customConstraint('CHECK (subtotal >= 0)')();
  
  RealColumn get discount => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (discount >= 0)')();
  
  RealColumn get total => real()
    .customConstraint('CHECK (total >= 0)')();
  
  @override
  List<Index> get indexes => [
    Index('idx_order_items_order', 'order_id'),
    Index('idx_order_items_product', 'product_id'),
  ];
}

// lib/database/tables/reviews.dart
import 'package:drift/drift.dart';
import 'users.dart';
import 'products.dart';
import 'orders.dart';

class Reviews extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 👇 Foreign key with cascade
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)
    .named('user_id')();
  
  // 👇 Foreign key with restrict
  IntColumn get productId => integer()
    .references(Products, #id, onDelete: KeyAction.restrict)
    .named('product_id')();
  
  // 👇 Foreign key with set null
  IntColumn get orderId => integer()
    .references(Orders, #id, onDelete: KeyAction.setNull)
    .nullable()
    .named('order_id')();
  
  IntColumn get rating => integer()
    .customConstraint('CHECK (rating >= 1 AND rating <= 5)')();
  
  TextColumn get title => text().nullable()();
  TextColumn get content => text().nullable()();
  BoolColumn get isVerifiedPurchase => boolean()
    .withDefault(const Constant(false))
    .named('is_verified_purchase')();
  BoolColumn get isRecommended => boolean().nullable().named('is_recommended')();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
  
  @override
  List<Index> get indexes => [
    Index('idx_reviews_user', 'user_id'),
    Index('idx_reviews_product', 'product_id'),
    Index('idx_reviews_rating', 'rating'),
    Index('idx_reviews_created', 'created_at'),
  ];
}
```

```dart
// lib/database/database.dart - Usage with foreign keys
import 'package:drift/drift.dart';
import 'tables/users.dart';
import 'tables/addresses.dart';
import 'tables/categories.dart';
import 'tables/products.dart';
import 'tables/orders.dart';
import 'tables/order_items.dart';
import 'tables/reviews.dart';

@DriftDatabase(tables: [
  Users,
  Addresses,
  Categories,
  Products,
  Orders,
  OrderItems,
  Reviews,
])
class EcommerceDatabase extends _$EcommerceDatabase {
  EcommerceDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Enable foreign keys
  @override
  Future<void> beforeOpen() async {
    await super.beforeOpen();
    await customSelect('PRAGMA foreign_keys = ON').get();
  }

  // 👇 Create order with foreign key handling
  Future<int> createOrder(
    int userId,
    int billingAddressId,
    int shippingAddressId,
    List<OrderItemInput> items,
  ) async {
    return await transaction(() async {
      // Validate foreign keys exist
      final user = await (select(users)..where((u) => u.id.equals(userId)))
          .getSingleOrNull();
      if (user == null) throw Exception('User not found');
      
      final billingAddress = await (select(addresses)
        ..where((a) => a.id.equals(billingAddressId) & a.userId.equals(userId)))
        .getSingleOrNull();
      if (billingAddress == null) throw Exception('Billing address not found');
      
      // Check stock
      for (final item in items) {
        final product = await (select(products)
          ..where((p) => p.id.equals(item.productId)))
          .getSingleOrNull();
        if (product == null) throw Exception('Product ${item.productId} not found');
        if (product.stock < item.quantity) {
          throw Exception('Insufficient stock for product: ${product.name}');
        }
      }
      
      // Create order
      final orderId = await into(orders).insert(
        OrdersCompanion.insert(
          orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
          userId: userId,
          billingAddressId: billingAddressId,
          shippingAddressId: shippingAddressId,
          subtotal: 0,
          total: 0,
        ),
      );
      
      // Add items
      double subtotal = 0;
      for (final item in items) {
        final product = await getProduct(item.productId);
        final itemSubtotal = product.price * item.quantity;
        subtotal += itemSubtotal;
        
        await into(orderItems).insert(
          OrderItemsCompanion.insert(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: product.price,
            subtotal: itemSubtotal,
            total: itemSubtotal,
          ),
        );
        
        // Update stock
        await (update(products)..where((p) => p.id.equals(product.id)))
          .write(ProductsCompanion(stock: Value(product.stock - item.quantity)));
      }
      
      // Update order totals
      await (update(orders)..where((o) => o.id.equals(orderId)))
        .write(OrdersCompanion(
          subtotal: Value(subtotal),
          total: Value(subtotal),
        ));
      
      return orderId;
    });
  }

  // 👇 Delete user with cascade (auto-deletes addresses and reviews)
  Future<void> deleteUser(int userId) async {
    // Foreign key CASCADE will automatically delete:
    // - Addresses
    // - Reviews
    // - Orders (but restrict prevents if orders exist)
    
    await (delete(users)..where((u) => u.id.equals(userId))).go();
  }

  // 👇 Get user with addresses (using foreign key)
  Future<Map<String, dynamic>> getUserWithDetails(int userId) async {
    final user = await (select(users)..where((u) => u.id.equals(userId)))
        .getSingleOrNull();
    if (user == null) throw Exception('User not found');
    
    final addresses = await (select(addresses)
      ..where((a) => a.userId.equals(userId)))
      .get();
    
    final orders = await (select(orders)
      ..where((o) => o.userId.equals(userId)))
      .get();
    
    return {
      'user': user,
      'addresses': addresses,
      'orderCount': orders.length,
    };
  }

  // 👇 Get product with reviews
  Future<Map<String, dynamic>> getProductWithReviews(int productId) async {
    final product = await (select(products)
      ..where((p) => p.id.equals(productId)))
      .getSingleOrNull();
    if (product == null) throw Exception('Product not found');
    
    final reviews = await (select(reviews)
      ..where((r) => r.productId.equals(productId))
      ..orderBy([(r) => OrderingTerm(expression: r.createdAt, mode: OrderingMode.desc)]))
      .get();
    
    final avgRating = reviews.isNotEmpty
        ? reviews.map((r) => r.rating).reduce((a, b) => a + b) / reviews.length
        : 0.0;
    
    return {
      'product': product,
      'reviews': reviews,
      'ratingCount': reviews.length,
      'averageRating': avgRating,
    };
  }

  Future<Product> getProduct(int id) async {
    final product = await (select(products)..where((p) => p.id.equals(id)))
        .getSingleOrNull();
    if (product == null) throw Exception('Product not found');
    return product;
  }
}

class OrderItemInput {
  final int productId;
  final int quantity;
  
  OrderItemInput({required this.productId, required this.quantity});
}
```

---

# Best Practices

- **Always enable foreign keys** – `PRAGMA foreign_keys = ON`
- **Use CASCADE carefully** – Can delete a lot of data
- **Use RESTRICT for important references** – Prevent accidental deletion
- **Use SET NULL for optional references** – Keep child records
- **Add indexes on foreign key columns** – For query performance
- **Test cascade behavior** – Ensure it works as expected
- **Document foreign key actions** – Explain what happens
- **Use appropriate action for each relationship** – Based on business needs
- **Consider performance impact** – Foreign keys add overhead

---

# Common Mistakes

## Mistake 1: Forgetting to enable foreign keys

Wrong:
```dart
// 🚫 Foreign keys won't work
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

## Mistake 2: Wrong column order in composite foreign keys

Wrong:
```dart
// 🚫 Column order must match
ForeignKey(
  columns: [productId, orderId], // Wrong order
  references: [OrderItems.orderId, OrderItems.productId],
)
```

Correct:
```dart
// ✅ Same order as primary key
ForeignKey(
  columns: [orderId, productId], // Correct order
  references: [OrderItems.orderId, OrderItems.productId],
)
```

## Mistake 3: Using RESTRICT with NULL values

Wrong:
```dart
// 🚫 RESTRICT doesn't work with NULL
IntColumn get userId => integer()
  .references(Users, #id, onDelete: KeyAction.restrict)
  .nullable()();
```

Correct:
```dart
// ✅ Use SET NULL for optional references
IntColumn get userId => integer()
  .references(Users, #id, onDelete: KeyAction.setNull)
  .nullable()();
```

---

# Summary

| Action | Effect | Use Case |
|--------|--------|----------|
| **CASCADE** | Delete/update child records | Orders → OrderItems |
| **RESTRICT** | Prevent parent deletion | Categories → Products |
| **SET NULL** | Set foreign key to NULL | Users → Posts (author) |
| **SET DEFAULT** | Set to default value | Users → Posts (default user) |
| **NO ACTION** | Like RESTRICT | Complex relationships |

---

# Next Steps

Now you understand foreign keys, let's dive deeper:

- [Many-to-Many](link) – Implementing many-to-many relationships
- [One-to-Many](link) – One-to-many relationships
- [One-to-One](link) – One-to-one relationships

---

# Did You Know?

- **Foreign keys are OFF by default** – Must enable with PRAGMA

- **CASCADE deletes can trigger more cascades** – Chain reactions

- **RESTRICT is checked before the operation** – Prevents the delete

- **SET NULL only works on nullable columns** – Must use `.nullable()`

- **Composite foreign keys require matching column order** – Must match primary key

- **Self-referencing foreign keys** – For hierarchies (employees, categories)

- **Foreign keys can reference UNIQUE constraints** – Not just PRIMARY KEY

- **Indexes on foreign key columns** – Improve JOIN performance

---

