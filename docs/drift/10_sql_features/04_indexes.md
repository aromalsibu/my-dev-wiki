## Indexes

**Optimizing database performance with indexes in Drift**

---

# What is it?

**Indexes** are special database objects that improve the speed of data retrieval operations. They work like a book's index – instead of scanning every page (row) to find what you need, you quickly look up the location in the index and go directly to the right place. Drift allows you to define indexes in your table definitions, making queries dramatically faster.

> **Think of Indexes like "a book's table of contents"** – without it, you'd have to flip through every page to find a topic. With it, you go directly to the right page, saving enormous time.

```dart
// 👇 Adding indexes to a table
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get userId => integer().named('user_id')();
  TextColumn get status => text()();
  DateTimeColumn get orderDate => dateTime().named('order_date')();
  
  // 👇 Add indexes for frequently queried columns
  @override
  List<Index> get indexes => [
    // Simple index on one column
    Index('idx_orders_user', 'user_id'),
    
    // Index on multiple columns (composite)
    Index('idx_orders_status_date', ['status', 'order_date']),
    
    // Unique index
    Index('idx_orders_number', 'order_number', unique: true),
  ];
}

// This makes queries like these MUCH faster:
// SELECT * FROM orders WHERE user_id = 1
// SELECT * FROM orders WHERE status = 'pending' ORDER BY order_date
// SELECT * FROM orders WHERE order_number = 'ORD-123'
```

> **What's happening here?**
> - **Index definition** – Specifies which columns to index
> - **Performance** – Dramatically speeds up queries
> - **Composite indexes** – Index multiple columns together
> - **Unique indexes** – Enforce uniqueness with performance

---

# Why does it exist?

- **Query Performance** – Speeds up SELECT queries
- **Sorting** – Faster ORDER BY operations
- **Lookups** – Faster WHERE clause evaluation
- **Joins** – Speeds up JOIN operations
- **Uniqueness** – Enforce unique constraints efficiently
- **Data Integrity** – Prevent duplicate values

---

# Creating Indexes

> **Different ways to create indexes**

## Simple Single-Column Index

```dart
// 👇 Index on a single column
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get email => text().unique()();
  TextColumn get name => text()();
  
  @override
  List<Index> get indexes => [
    // 👇 Index on name for faster lookups
    Index('idx_users_name', 'name'),
  ];
}

// Generated SQL:
// CREATE INDEX idx_users_name ON users(name)
```

## Composite Index (Multiple Columns)

```dart
// 👇 Index on multiple columns
class Orders extends Table {
  IntColumn get userId => integer().named('user_id')();
  TextColumn get status => text()();
  DateTimeColumn get orderDate => dateTime().named('order_date')();
  
  @override
  List<Index> get indexes => [
    // 👇 Composite index for common query patterns
    Index('idx_orders_user_status', ['user_id', 'status']),
    Index('idx_orders_status_date', ['status', 'order_date']),
  ];
}

// Generated SQL:
// CREATE INDEX idx_orders_user_status ON orders(user_id, status)
// CREATE INDEX idx_orders_status_date ON orders(status, order_date)
```

## Unique Index

```dart
// 👇 Unique index (enforces uniqueness + provides index)
class Products extends Table {
  TextColumn get sku => text().named('sku')();
  
  @override
  List<Index> get indexes => [
    // 👇 Unique index on SKU (like unique constraint but indexed)
    Index('idx_products_sku', 'sku', unique: true),
  ];
}

// Generated SQL:
// CREATE UNIQUE INDEX idx_products_sku ON products(sku)
```

## Case-Insensitive Index

```dart
// 👇 Case-insensitive index for text columns
class Users extends Table {
  TextColumn get email => text().named('email')();
  
  @override
  List<Index> get indexes => [
    // 👇 Case-insensitive index for email lookups
    Index('idx_users_email', 'email', collate: 'NOCASE'),
  ];
}

// Generated SQL:
// CREATE INDEX idx_users_email ON users(email COLLATE NOCASE)
```

---

# Index Best Practices

> **When and how to use indexes**

## Identify Common Query Patterns

```dart
// 👇 Add indexes based on how you query
class Orders extends Table {
  IntColumn get userId => integer().named('user_id')();
  TextColumn get status => text()();
  DateTimeColumn get orderDate => dateTime().named('order_date')();
  RealColumn get total => real()();
  
  @override
  List<Index> get indexes => [
    // 👇 Common queries:
    // - Get user's orders
    Index('idx_orders_user', 'user_id'),
    
    // - Get pending orders
    Index('idx_orders_status', 'status'),
    
    // - Get orders by date range
    Index('idx_orders_date', 'order_date'),
    
    // - Get high-value orders for user
    // Composite index for complex conditions
    Index('idx_orders_user_total', ['user_id', 'total']),
  ];
}
```

## Index for Foreign Keys

```dart
// 👇 Always index foreign key columns
class Posts extends Table {
  IntColumn get userId => integer()
    .references(Users, #id)
    .named('user_id')();
  
  @override
  List<Index> get indexes => [
    // 👇 Foreign key index for faster joins
    Index('idx_posts_user', 'user_id'),
  ];
}
```

---

# Real-World Example

> **Complete e-commerce indexing system**

```dart
// lib/database/tables/indexed_tables.dart
import 'package:drift/drift.dart';

// 1️⃣ USERS TABLE
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique().named('username')();
  TextColumn get email => text().unique().named('email')();
  TextColumn get fullName => text().nullable().named('full_name')();
  BoolColumn get isActive => boolean().withDefault(const Constant(true)).named('is_active')();
  BoolColumn get isVerified => boolean().withDefault(const Constant(false)).named('is_verified')();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime).named('created_at')();
  DateTimeColumn get lastLogin => dateTime().nullable().named('last_login')();
  
  @override
  List<Index> get indexes => [
    // 👇 Username is already unique but index helps
    Index('idx_users_username', 'username'),
    
    // 👇 Email lookups are common
    Index('idx_users_email', 'email'),
    
    // 👇 Active users filter
    Index('idx_users_active', 'is_active'),
    
    // 👇 Verification status
    Index('idx_users_verified', 'is_verified'),
    
    // 👇 Recent users
    Index('idx_users_created', 'created_at'),
    
    // 👇 Users who haven't logged in recently
    Index('idx_users_last_login', 'last_login'),
    
    // 👇 Active and verified (common filter)
    Index('idx_users_active_verified', ['is_active', 'is_verified']),
  ];
}

// 2️⃣ PRODUCTS TABLE
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get sku => text().unique().named('sku')();
  TextColumn get name => text().named('name')();
  TextColumn get description => text().nullable().named('description')();
  IntColumn get categoryId => integer().references(Categories, #id).nullable().named('category_id')();
  RealColumn get price => real().customConstraint('CHECK (price >= 0)').named('price')();
  IntColumn get stock => integer().withDefault(const Constant(0)).named('stock')();
  BoolColumn get isActive => boolean().withDefault(const Constant(true)).named('is_active')();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime).named('created_at')();
  
  @override
  List<Index> get indexes => [
    // 👇 SKU lookups
    Index('idx_products_sku', 'sku'),
    
    // 👇 Product name search
    Index('idx_products_name', 'name'),
    
    // 👇 Category filtering
    Index('idx_products_category', 'category_id'),
    
    // 👇 Price range queries
    Index('idx_products_price', 'price'),
    
    // 👇 Stock status
    Index('idx_products_stock', 'stock'),
    
    // 👇 Active products
    Index('idx_products_active', 'is_active'),
    
    // 👇 Common filters
    Index('idx_products_category_active', ['category_id', 'is_active']),
    
    // 👇 Price with active status
    Index('idx_products_active_price', ['is_active', 'price']),
    
    // 👇 Name search with active
    Index('idx_products_name_active', ['name', 'is_active']),
  ];
}

// 3️⃣ ORDERS TABLE
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().unique().named('order_number')();
  IntColumn get userId => integer().references(Users, #id).named('user_id')();
  TextColumn get status => text().named('status')();
  RealColumn get total => real().customConstraint('CHECK (total >= 0)').named('total')();
  DateTimeColumn get orderDate => dateTime().withDefault(currentDateAndTime).named('order_date')();
  BoolColumn get isPaid => boolean().withDefault(const Constant(false)).named('is_paid')();
  BoolColumn get isShipped => boolean().withDefault(const Constant(false)).named('is_shipped')();
  BoolColumn get isDelivered => boolean().withDefault(const Constant(false)).named('is_delivered')();
  
  @override
  List<Index> get indexes => [
    // 👇 Order number lookups
    Index('idx_orders_number', 'order_number'),
    
    // 👇 User orders
    Index('idx_orders_user', 'user_id'),
    
    // 👇 Status filtering
    Index('idx_orders_status', 'status'),
    
    // 👇 Date range queries
    Index('idx_orders_date', 'order_date'),
    
    // 👇 Total range queries
    Index('idx_orders_total', 'total'),
    
    // 👇 Payment status
    Index('idx_orders_paid', 'is_paid'),
    
    // 👇 Shipping status
    Index('idx_orders_shipped', 'is_shipped'),
    
    // 👇 Common query pattern: user + status
    Index('idx_orders_user_status', ['user_id', 'status']),
    
    // 👇 Status + date (pending orders sorted by date)
    Index('idx_orders_status_date', ['status', 'order_date']),
    
    // 👇 User + date (user's orders sorted by date)
    Index('idx_orders_user_date', ['user_id', 'order_date']),
    
    // 👇 Paid + shipped + delivered status
    Index('idx_orders_status_paid', ['status', 'is_paid']),
    
    // 👇 High-value orders for a user
    Index('idx_orders_user_total', ['user_id', 'total']),
  ];
}

// 4️⃣ ORDER ITEMS TABLE
class OrderItems extends Table {
  @override
  Set<Column> get primaryKey => {orderId, productId};
  
  IntColumn get orderId => integer().references(Orders, #id, onDelete: KeyAction.cascade).named('order_id')();
  IntColumn get productId => integer().references(Products, #id).named('product_id')();
  IntColumn get quantity => integer().customConstraint('CHECK (quantity > 0)').named('quantity')();
  RealColumn get unitPrice => real().customConstraint('CHECK (unit_price >= 0)').named('unit_price')();
  RealColumn get total => real().customConstraint('CHECK (total >= 0)').named('total')();
  
  @override
  List<Index> get indexes => [
    // 👇 Order items lookup
    Index('idx_order_items_order', 'order_id'),
    
    // 👇 Product sales analysis
    Index('idx_order_items_product', 'product_id'),
    
    // 👇 Order + product (primary key is already indexed)
    Index('idx_order_items_order_product', ['order_id', 'product_id']),
    
    // 👇 Product + order (for product sales analysis)
    Index('idx_order_items_product_order', ['product_id', 'order_id']),
    
    // 👇 Price and quantity analysis
    Index('idx_order_items_total', 'total'),
  ];
}

// 5️⃣ REVIEWS TABLE
class Reviews extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get userId => integer().references(Users, #id).named('user_id')();
  IntColumn get productId => integer().references(Products, #id).named('product_id')();
  IntColumn get rating => integer().customConstraint('CHECK (rating >= 1 AND rating <= 5)').named('rating')();
  TextColumn get content => text().nullable().named('content')();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime).named('created_at')();
  
  @override
  List<Index> get indexes => [
    // 👇 User reviews
    Index('idx_reviews_user', 'user_id'),
    
    // 👇 Product reviews
    Index('idx_reviews_product', 'product_id'),
    
    // 👇 Rating filter
    Index('idx_reviews_rating', 'rating'),
    
    // 👇 Recent reviews
    Index('idx_reviews_created', 'created_at'),
    
    // 👇 Product + rating (common filter)
    Index('idx_reviews_product_rating', ['product_id', 'rating']),
    
    // 👇 User + rating
    Index('idx_reviews_user_rating', ['user_id', 'rating']),
  ];
}

// lib/database/index_service.dart
import 'package:drift/drift.dart';

class IndexService {
  final AppDatabase db;
  
  IndexService(this.db);

  // 👇 Check if indexes exist
  Future<List<String>> getIndexes(String tableName) async {
    final results = await db.customSelect(
      '''
      SELECT name FROM sqlite_master 
      WHERE type = 'index' 
      AND tbl_name = ?
      ''',
      variables: [Variable.withString(tableName)],
    ).get();
    
    return results.map((row) => row.data['name'] as String).toList();
  }
  
  // 👇 Analyze query performance
  Future<void> analyzeQuery(String query) async {
    // Use EXPLAIN QUERY PLAN to see if indexes are used
    final results = await db.customSelect('''
      EXPLAIN QUERY PLAN $query
    ''').get();
    
    for (final row in results) {
      print('Query plan: ${row.data}');
    }
  }
  
  // 👇 Rebuild indexes
  Future<void> rebuildIndexes() async {
    await db.customSelect('REINDEX').go();
    print('✅ All indexes rebuilt');
  }
}
```

---

# Index Performance Considerations

> **When indexes help and when they hurt**

## When Indexes Help

```dart
// ✅ These queries benefit from indexes
// 1. WHERE clause lookups
// 2. ORDER BY sorting
// 3. GROUP BY aggregation
// 4. JOIN conditions

// Example: User lookups benefit from index
final user = await (select(users)
  ..where((u) => u.email.equals('john@example.com')))
  .getSingle(); // Index on email makes this fast

// Example: Date range queries
final recentOrders = await (select(orders)
  ..where((o) => o.orderDate > const Variable(DateTime.now().subtract(Duration(days: 30))))
  ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
  .get(); // Index on orderDate helps both WHERE and ORDER BY
```

## When Indexes Hurt

```dart
// ❌ These operations are slower with indexes
// 1. INSERT operations (must update index)
// 2. UPDATE operations (must update index)
// 3. DELETE operations (must update index)

// Example: Bulk insert is slower with many indexes
// If you have 10 indexes, each insert updates all 10
```

---

# Index Decision Guide

| Query Type | Index Recommendation | Priority |
|------------|---------------------|----------|
| **WHERE on column** | Index that column | High |
| **ORDER BY** | Index sort column | High |
| **JOIN condition** | Index foreign key | High |
| **WHERE on multiple columns** | Composite index | Medium |
| **COUNT(*) with WHERE** | Index filter column | Medium |
| **Full text search** | FTS extension | Medium |
| **High write volume** | Fewer indexes | Low |

---

# Index Best Practices

- **Index columns used in WHERE** – Primary targets
- **Index foreign keys** – For JOIN performance
- **Index columns used in ORDER BY** – For sorting
- **Use composite indexes wisely** – Match query patterns
- **Avoid indexing low-cardinality columns** – e.g., booleans
- **Balance read/write performance** – Indexes slow writes
- **Monitor index usage** – Remove unused indexes
- **Test with realistic data** – Verify performance

---

# Common Mistakes

## Mistake 1: Too many indexes

Wrong:
```dart
// 🚫 Indexing every column
@override
List<Index> get indexes => [
  Index('idx_1', 'col1'),
  Index('idx_2', 'col2'),
  // ... 20 more indexes
];
```

Correct:
```dart
// ✅ Index only frequently queried columns
@override
List<Index> get indexes => [
  Index('idx_email', 'email'), // WHERE email = ?
  Index('idx_user_id', 'user_id'), // JOIN ON user_id
];
```

## Mistake 2: Wrong column order in composite index

Wrong:
```dart
// 🚫 Less selective column first
Index('idx_status_date', ['status', 'order_date']);
// Query: WHERE status = 'pending' AND order_date > '2024-01-01'
```

Correct:
```dart
// ✅ More selective column first
Index('idx_status_date', ['order_date', 'status']);
// Query: WHERE order_date > '2024-01-01' AND status = 'pending'
```

## Mistake 3: Indexing low-cardinality columns

Wrong:
```dart
// 🚫 Boolean column (only two values)
Index('idx_is_active', 'is_active');
```

Correct:
```dart
// ✅ Composite index with boolean
Index('idx_active_status', ['is_active', 'status']);
```

---

# Summary

| Index Type | Purpose | Example |
|------------|---------|---------|
| **Single-Column** | Simple lookups | `Index('idx_email', 'email')` |
| **Composite** | Multiple columns | `Index('idx_user_date', ['user_id', 'date'])` |
| **Unique** | Enforce uniqueness | `Index('idx_sku', 'sku', unique: true)` |
| **Case-Insensitive** | Text search | `Index('idx_name', 'name', collate: 'NOCASE')` |

---

# Next Steps

Now you understand indexes, let's dive deeper:

- [Triggers](link) – Database triggers
- [CTEs](link) – Common Table Expressions
- [Window Functions](link) – Advanced analytics

---

# Did You Know?

- **Indexes can speed up queries by 100x** – Dramatic improvement

- **Indexes slow down writes** – INSERT, UPDATE, DELETE

- **Composite indexes order matters** – Put most selective column first

- **SQLite creates indexes automatically** – For PRIMARY KEY and UNIQUE

- **Indexes use B-tree structure** – For efficient lookups

- **Covering indexes** – Index includes all needed columns

- **Indexes can be partial** – Index only certain rows

- **Too many indexes is bad** – Balance performance

---

