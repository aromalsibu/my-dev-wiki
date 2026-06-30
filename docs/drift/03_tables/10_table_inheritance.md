## Table Inheritance

**Reusing table definitions through inheritance in Drift**

---

# What is it?

**Table Inheritance** is a pattern where you define a base table class with common columns, and then extend it to create specialized tables that share those columns. This promotes code reuse, maintains consistency, and reduces duplication across your database schema.

> **Think of Table Inheritance like "family traits"** – children inherit common characteristics from their parents, but also have their own unique features.

```dart
// 👇 Base table with common columns
abstract class BaseTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
  BoolColumn get isDeleted => boolean().withDefault(const Constant(false))();
}

// 👇 Child tables inherit base columns
class Users extends BaseTable {
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
}

class Posts extends BaseTable {
  TextColumn get title => text()();
  TextColumn get content => text()();
  IntColumn get userId => integer().references(Users, #id)();
}

// Generated SQL for Users:
// CREATE TABLE users (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   created_at INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   updated_at INTEGER,
//   is_deleted INTEGER NOT NULL DEFAULT 0,
//   name TEXT NOT NULL,
//   email TEXT NOT NULL UNIQUE
// )
```

> **What's happening here?**
> - **BaseTable** – Defines common columns
> - **Inheritance** – Child tables extend BaseTable
> - **Code Reuse** – No duplicate column definitions
> - **Consistency** – All tables have the same base columns

---

# Why does it exist?

- **Code Reuse** – Don't repeat common column definitions
- **Consistency** – Ensure all tables have the same base columns
- **Maintainability** – Change base columns in one place
- **DRY Principle** – Don't Repeat Yourself
- **Pattern Enforcement** – Standardize table structures
- **Reduced Boilerplate** – Less code to write and maintain

---

# Basic Inheritance

> **Simple inheritance patterns**

## Single Inheritance

```dart
// 👇 Base table with audit columns
abstract class AuditableTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
  IntColumn get createdBy => integer().nullable().named('created_by')();
  IntColumn get updatedBy => integer().nullable().named('updated_by')();
}

// 👇 Users inherit audit columns
class Users extends AuditableTable {
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get fullName => text().nullable().named('full_name')();
}

// 👇 Products inherit audit columns
class Products extends AuditableTable {
  TextColumn get sku => text().unique()();
  TextColumn get name => text()();
  RealColumn get price => real()();
  IntColumn get stock => integer()();
}

// Generated SQL for Users:
// CREATE TABLE users (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   created_at INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   updated_at INTEGER,
//   created_by INTEGER,
//   updated_by INTEGER,
//   username TEXT NOT NULL UNIQUE,
//   email TEXT NOT NULL UNIQUE,
//   full_name TEXT
// )
```

---

## Multi-Level Inheritance

```dart
// 👇 Level 1: Base table
abstract class BaseTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// 👇 Level 2: Soft delete
abstract class SoftDeleteTable extends BaseTable {
  BoolColumn get isDeleted => boolean().withDefault(const Constant(false))();
  DateTimeColumn get deletedAt => dateTime().nullable().named('deleted_at')();
}

// 👇 Level 3: Audit trail
abstract class AuditableTable extends SoftDeleteTable {
  DateTimeColumn get updatedAt => dateTime().nullable().named('updated_at')();
  IntColumn get createdBy => integer().nullable().named('created_by')();
  IntColumn get updatedBy => integer().nullable().named('updated_by')();
}

// 👇 Final table inherits all levels
class Orders extends AuditableTable {
  TextColumn get orderNumber => text().unique().named('order_number')();
  IntColumn get userId => integer().references(Users, #id).named('user_id')();
  RealColumn get total => real()();
  TextColumn get status => text()();
}

// Generated SQL for Orders:
// CREATE TABLE orders (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   created_at INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   is_deleted INTEGER NOT NULL DEFAULT 0,
//   deleted_at INTEGER,
//   updated_at INTEGER,
//   created_by INTEGER,
//   updated_by INTEGER,
//   order_number TEXT NOT NULL UNIQUE,
//   user_id INTEGER NOT NULL REFERENCES users(id),
//   total REAL NOT NULL,
//   status TEXT NOT NULL
// )
```

---

# Common Base Table Patterns

> **Reusable base table patterns**

## Pattern 1: Timestamp Base

```dart
// 👇 Base with timestamps only
abstract class TimestampedTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
  DateTimeColumn get updatedAt => dateTime()
    .nullable()
    .named('updated_at')();
}

// Usage
class Comments extends TimestampedTable {
  TextColumn get content => text()();
  IntColumn get postId => integer().references(Posts, #id).named('post_id')();
  IntColumn get userId => integer().references(Users, #id).named('user_id')();
}
```

---

## Pattern 2: Soft Delete Base

```dart
// 👇 Base with soft delete
abstract class SoftDeletableTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  BoolColumn get isDeleted => boolean()
    .withDefault(const Constant(false))
    .named('is_deleted')();
  DateTimeColumn get deletedAt => dateTime()
    .nullable()
    .named('deleted_at')();
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
}

// Usage with default queries
class Products extends SoftDeletableTable {
  TextColumn get name => text()();
  RealColumn get price => real()();
  
  // 👇 Helper for active products
  static Expression<bool> isActive(Products t) => 
    t.isDeleted.equals(false);
}

// Query active products
final activeProducts = await (select(products)
  ..where(Products.isActive))
  .get();
```

---

## Pattern 3: Tenant-Aware Base

```dart
// 👇 Base with multi-tenant support
abstract class TenantTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get tenantId => integer()
    .named('tenant_id')
    .customConstraint('CHECK (tenant_id IS NOT NULL)')();
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
  
  // 👇 Helper for tenant queries
  static Expression<bool> isTenant<T extends TenantTable>(
    T table, 
    int tenantId,
  ) => table.tenantId.equals(tenantId);
}

// Usage
class Orders extends TenantTable {
  TextColumn get orderNumber => text().unique().named('order_number')();
  RealColumn get total => real()();
  TextColumn get status => text()();
}

// Query tenant-specific orders
final tenantOrders = await (select(orders)
  ..where(TenantTable.isTenant(orders, tenantId)))
  .get();
```

---

## Pattern 4: Versioned Base

```dart
// 👇 Base with versioning support
abstract class VersionedTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get version => integer()
    .withDefault(const Constant(1))
    .customConstraint('CHECK (version >= 1)')();
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
  DateTimeColumn get updatedAt => dateTime()
    .nullable()
    .named('updated_at')();
  IntColumn get previousVersionId => integer()
    .nullable()
    .named('previous_version_id')();
}

// Usage with version tracking
class Documents extends VersionedTable {
  TextColumn get title => text()();
  TextColumn get content => text()();
  IntColumn get userId => integer().references(Users, #id).named('user_id')();
}

// Update with version increment
Future<void> updateDocument(int id, String newContent) async {
  await transaction(() async {
    // Get current version
    final current = await (select(documents)
      ..where((d) => d.id.equals(id)))
      .getSingle();
    
    // Create new version
    final newId = await into(documents).insert(
      DocumentsCompanion.insert(
        title: current.title,
        content: newContent,
        userId: current.userId,
        version: current.version + 1,
        previousVersionId: current.id,
      ),
    );
    
    // Update references to point to new version
    // ...
  });
}
```

---

# Real-World Example

> **Complete e-commerce schema with inheritance**

```dart
// lib/database/tables/base_tables.dart
import 'package:drift/drift.dart';

// 👇 Level 1: Core Base
abstract class BaseTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
}

// 👇 Level 2: Soft Delete
abstract class SoftDeletableTable extends BaseTable {
  BoolColumn get isDeleted => boolean()
    .withDefault(const Constant(false))
    .named('is_deleted')();
  DateTimeColumn get deletedAt => dateTime()
    .nullable()
    .named('deleted_at')();
}

// 👇 Level 3: Audit Trail
abstract class AuditableTable extends SoftDeletableTable {
  DateTimeColumn get updatedAt => dateTime()
    .nullable()
    .named('updated_at')();
  IntColumn get createdBy => integer()
    .nullable()
    .references(Users, #id)
    .named('created_by')();
  IntColumn get updatedBy => integer()
    .nullable()
    .references(Users, #id)
    .named('updated_by')();
}

// 👇 Level 4: Tenant-Aware
abstract class TenantTable extends AuditableTable {
  IntColumn get tenantId => integer()
    .nullable()
    .named('tenant_id')();
}

// 👇 Level 5: Versioned
abstract class VersionedTable extends TenantTable {
  IntColumn get version => integer()
    .withDefault(const Constant(1))
    .customConstraint('CHECK (version >= 1)')();
  IntColumn get previousVersionId => integer()
    .nullable()
    .named('previous_version_id')();
}

// lib/database/tables/users.dart
import 'package:drift/drift.dart';
import 'base_tables.dart';

// 👇 Users use audit trail (no tenant)
class Users extends AuditableTable {
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get fullName => text().nullable().named('full_name')();
  TextColumn get passwordHash => text().named('password_hash')();
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))
    .named('is_active')();
  BoolColumn get isAdmin => boolean()
    .withDefault(const Constant(false))
    .named('is_admin')();
  DateTimeColumn get lastLogin => dateTime()
    .nullable()
    .named('last_login')();
}

// lib/database/tables/products.dart
import 'package:drift/drift.dart';
import 'base_tables.dart';

// 👇 Products use all features
class Products extends VersionedTable {
  // Business keys
  TextColumn get sku => text().unique().named('sku')();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  
  // Pricing
  RealColumn get price => real()
    .customConstraint('CHECK (price >= 0)')();
  RealColumn get cost => real()
    .nullable()
    .customConstraint('CHECK (cost >= 0)')();
  
  // Inventory
  IntColumn get stock => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (stock >= 0)')();
  IntColumn get reorderLevel => integer()
    .withDefault(const Constant(10))
    .customConstraint('CHECK (reorder_level >= 0)')
    .named('reorder_level')();
  IntColumn get reservedStock => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (reserved_stock >= 0)')
    .named('reserved_stock')();
  
  // Status
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))
    .named('is_active')();
  BoolColumn get isFeatured => boolean()
    .withDefault(const Constant(false))
    .named('is_featured')();
  
  // Ratings
  RealColumn get averageRating => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (average_rating >= 0 AND average_rating <= 5)')
    .named('average_rating')();
  IntColumn get reviewCount => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (review_count >= 0)')
    .named('review_count')();
  
  // Indexes
  @override
  List<Index> get indexes => [
    Index('idx_products_sku', 'sku'),
    Index('idx_products_tenant_id', 'tenant_id'),
    Index('idx_products_is_active', 'is_active'),
    Index('idx_products_is_deleted', 'is_deleted'),
    Index('idx_products_created_at', 'created_at'),
  ];
}

// lib/database/tables/orders.dart
import 'package:drift/drift.dart';
import 'base_tables.dart';
import 'users.dart';

// 👇 Orders use multi-level inheritance
class Orders extends VersionedTable {
  // Order identification
  TextColumn get orderNumber => text()
    .unique()
    .customConstraint('CHECK (LENGTH(order_number) >= 8)')
    .named('order_number')();
  
  // Relations
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.restrict)
    .named('user_id')();
  
  // Financial
  RealColumn get subtotal => real()
    .customConstraint('CHECK (subtotal >= 0)')();
  RealColumn get tax => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (tax >= 0)')();
  RealColumn get shippingCost => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (shipping_cost >= 0)')
    .named('shipping_cost')();
  RealColumn get discount => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (discount >= 0)')();
  RealColumn get total => real()
    .customConstraint('CHECK (total >= 0)')();
  
  // Status
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .customConstraint("CHECK (status IN ('pending', 'processing', 'paid', 'shipped', 'delivered', 'cancelled', 'refunded'))")();
  
  // Shipping
  TextColumn get shippingAddress => text().named('shipping_address')();
  TextColumn get billingAddress => text().named('billing_address')();
  
  // Tracking
  TextColumn get trackingNumber => text()
    .nullable()
    .named('tracking_number')();
  
  // Dates
  DateTimeColumn get orderDate => dateTime()
    .withDefault(currentDateAndTime)
    .named('order_date')();
  DateTimeColumn get shippedDate => dateTime()
    .nullable()
    .named('shipped_date')();
  DateTimeColumn get deliveredDate => dateTime()
    .nullable()
    .named('delivered_date')();
  
  // Flags
  BoolColumn get isPaid => boolean()
    .withDefault(const Constant(false))
    .named('is_paid')();
  BoolColumn get isShipped => boolean()
    .withDefault(const Constant(false))
    .named('is_shipped')();
  BoolColumn get isDelivered => boolean()
    .withDefault(const Constant(false))
    .named('is_delivered')();
  
  // Indexes
  @override
  List<Index> get indexes => [
    Index('idx_orders_order_number', 'order_number'),
    Index('idx_orders_user_id', 'user_id'),
    Index('idx_orders_tenant_id', 'tenant_id'),
    Index('idx_orders_status', 'status'),
    Index('idx_orders_order_date', 'order_date'),
    Index('idx_orders_is_deleted', 'is_deleted'),
  ];
}

// lib/database/tables/order_items.dart
import 'package:drift/drift.dart';
import 'base_tables.dart';
import 'orders.dart';
import 'products.dart';

// 👇 Order items use composition (no inheritance)
class OrderItems extends Table {
  // Composite primary key
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  // Relationships
  IntColumn get orderId => integer()
    .references(Orders, #id, onDelete: KeyAction.cascade)
    .named('order_id')();
  
  IntColumn get productId => integer()
    .references(Products, #id, onDelete: KeyAction.restrict)
    .named('product_id')();
  
  // Item details
  IntColumn get quantity => integer()
    .customConstraint('CHECK (quantity > 0)')();
  RealColumn get unitPrice => real()
    .customConstraint('CHECK (unit_price >= 0)')
    .named('unit_price')();
  RealColumn get discountPercent => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (discount_percent >= 0 AND discount_percent <= 100)')
    .named('discount_percent')();
  
  // Generated totals
  RealColumn get subtotal => real()
    .generatedAs(
      const Constant("quantity * unit_price"),
      stored: true,
    )();
  RealColumn get discountAmount => real()
    .generatedAs(
      const Constant("subtotal * discount_percent / 100"),
      stored: true,
    )
    .named('discount_amount')();
  RealColumn get total => real()
    .generatedAs(
      const Constant("subtotal - discount_amount"),
      stored: true,
    )();
  
  // Status
  BoolColumn get isShipped => boolean()
    .withDefault(const Constant(false))
    .named('is_shipped')();
  BoolColumn get isDelivered => boolean()
    .withDefault(const Constant(false))
    .named('is_delivered')();
  
  // Indexes
  @override
  List<Index> get indexes => [
    Index('idx_order_items_order', 'order_id'),
    Index('idx_order_items_product', 'product_id'),
  ];
}

// lib/database/database.dart
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

  // 👇 Using inherited features
  Future<List<User>> getActiveUsers() async {
    return await (select(users)
      ..where((u) => u.isActive.equals(true))
      ..where((u) => u.isDeleted.equals(false)))
      .get();
  }

  // 👇 Query with soft delete
  Future<List<Product>> getActiveProducts() async {
    return await (select(products)
      ..where((p) => p.isDeleted.equals(false))
      ..where((p) => p.isActive.equals(true)))
      .get();
  }

  // 👇 Query by tenant
  Future<List<Order>> getTenantOrders(int tenantId) async {
    return await (select(orders)
      ..where((o) => o.tenantId.equals(tenantId))
      ..where((o) => o.isDeleted.equals(false))
      ..orderBy([(o) => OrderingTerm(expression: o.createdAt, mode: OrderingMode.desc)]))
      .get();
  }

  // 👇 Create with audit fields
  Future<int> createOrder({
    required int userId,
    required int tenantId,
    required List<CartItem> items,
  }) async {
    return await transaction(() async {
      // ... order creation logic
      final orderId = await into(orders).insert(
        OrdersCompanion.insert(
          orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
          userId: userId,
          tenantId: tenantId,
          createdBy: userId,
          subtotal: 0,
          total: 0,
        ),
      );
      
      // Add items...
      return orderId;
    });
  }

  // 👇 Soft delete
  Future<void> softDeleteOrder(int orderId) async {
    await (update(orders)..where((o) => o.id.equals(orderId)))
      .write(OrdersCompanion(
        isDeleted: const Value(true),
        deletedAt: Value(DateTime.now()),
      ));
  }

  // 👇 Versioned update
  Future<void> updateProductVersion(int productId, String newName) async {
    await transaction(() async {
      // Get current version
      final current = await (select(products)
        ..where((p) => p.id.equals(productId)))
        .getSingle();
      
      // Create new version
      await into(products).insert(
        ProductsCompanion.insert(
          sku: current.sku,
          name: newName,
          description: Value(current.description),
          price: current.price,
          cost: Value(current.cost),
          stock: current.stock,
          tenantId: current.tenantId,
          createdBy: current.createdBy,
          version: current.version + 1,
          previousVersionId: current.id,
        ),
      );
      
      // Mark old version as deleted
      await (update(products)..where((p) => p.id.equals(productId)))
        .write(ProductsCompanion(
          isDeleted: const Value(true),
          deletedAt: Value(DateTime.now()),
        ));
    });
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

- **Use abstract classes** – For base tables (not instantiated)
- **Keep inheritance shallow** – Maximum 3-4 levels
- **Use descriptive names** – `AuditableTable`, `SoftDeletableTable`
- **Document base tables** – Explain what they provide
- **Add helper methods** – For common queries
- **Use composition when needed** – Not everything needs inheritance
- **Test inheritance** – Ensure columns are generated correctly
- **Consider performance** – More columns = more data
- **Plan migrations** – Changes to base tables affect all children

---

# Common Mistakes

## Mistake 1: Non-abstract base tables

Wrong:
```dart
// 🚫 Can be instantiated
class BaseTable extends Table {
  IntColumn get id => integer().autoIncrement()();
}
```

Correct:
```dart
// ✅ Must be abstract
abstract class BaseTable extends Table {
  IntColumn get id => integer().autoIncrement()();
}
```

## Mistake 2: Over-using inheritance

Wrong:
```dart
// 🚫 Everything inherits from one giant base
abstract class MegaTable extends Table {
  // 20+ columns that only apply to some tables
}
```

Correct:
```dart
// ✅ Multiple focused base tables
abstract class TimestampedTable extends Table { ... }
abstract class SoftDeletableTable extends TimestampedTable { ... }
```

## Mistake 3: Forgetting to import base table

Wrong:
```dart
// 🚫 Base table not imported
class Users extends BaseTable { // Error: BaseTable not found
}
```

Correct:
```dart
// ✅ Import base table
import 'base_tables.dart';
class Users extends BaseTable { ... }
```

---

# Summary

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Audit Trail** | Tracking changes | `createdBy`, `updatedBy` |
| **Soft Delete** | Recoverable deletion | `isDeleted`, `deletedAt` |
| **Timestamping** | Time tracking | `createdAt`, `updatedAt` |
| **Multi-Tenant** | Data isolation | `tenantId` |
| **Versioning** | Change history | `version`, `previousVersionId` |

---

# Next Steps

Now you understand table inheritance, let's dive deeper:

- [Indexes](link) – Advanced indexing strategies
- [Views](link) – Creating database views
- [Triggers](link) – Database triggers

---

# Did You Know?

- **Drift supports multi-level inheritance** – Up to any depth

- **Abstract tables are not created** – Only concrete tables

- **Base tables can have constraints** – They apply to all children

- **Column naming is consistent** – Across all inherited tables

- **Migrations handle inheritance** – Changes cascade automatically

- **Helper methods can be added** – To base tables for common queries

- **Inheritance reduces code by 50%+** – For large schemas

- **Composition is sometimes better** – For unrelated shared columns

---
