## Views

**Creating and using database views in Drift**

---

# What is it?

**Views** are virtual tables that are defined by a SQL query. Unlike regular tables, views don't store data physically; instead, they present a transformed, filtered, or aggregated view of the underlying data. Drift supports creating and querying views, allowing you to simplify complex queries, improve security, and create reusable data abstractions.

> **Think of Views like "custom lenses"** – instead of looking at the raw data directly, you put on a lens that shows only what you want to see, in exactly the format you want it.

```dart
// 👇 Create a view for order summaries
final orderSummaryView = await db.customSelect('''
  CREATE VIEW order_summary AS
  SELECT 
    o.id as order_id,
    o.order_number,
    o.total,
    u.name as user_name,
    u.email as user_email,
    COUNT(oi.id) as item_count,
    SUM(oi.quantity) as total_items
  FROM orders o
  INNER JOIN users u ON o.user_id = u.id
  LEFT JOIN order_items oi ON o.id = oi.order_id
  WHERE o.status = 'completed'
  GROUP BY o.id
''').go();

// 👇 Query the view like a table
final results = await customSelect('''
  SELECT * FROM order_summary 
  WHERE total > 100
  ORDER BY total DESC
''').get();
```

> **What's happening here?**
> - **CREATE VIEW** – Define the view
> - **Virtual table** – No physical storage
> - **Queryable** – Use like a regular table
> - **Always up-to-date** – Reflects current data

---

# Why does it exist?

- **Simplify Complex Queries** – Encapsulate complex joins
- **Data Abstraction** – Hide complexity
- **Security** – Restrict access to specific columns
- **Reusability** – Use query logic in multiple places
- **Performance** – Pre-optimize complex queries
- **Data Transformation** – Present data in required format

---

# Creating Views

> **Different ways to create views**

## Basic View

```dart
// 👇 Simple view
await db.customSelect('''
  CREATE VIEW active_users AS
  SELECT id, name, email, created_at
  FROM users
  WHERE is_active = 1
''').go();

// Query the view
final results = await customSelect('''
  SELECT * FROM active_users
  ORDER BY name
''').get();
```

## Complex View with Joins

```dart
// 👇 View with multiple joins
await db.customSelect('''
  CREATE VIEW order_details AS
  SELECT 
    o.id as order_id,
    o.order_number,
    o.total,
    o.status,
    o.order_date,
    u.id as user_id,
    u.name as user_name,
    u.email as user_email,
    COUNT(DISTINCT oi.id) as item_count,
    SUM(oi.quantity) as total_items,
    GROUP_CONCAT(DISTINCT p.name) as product_names
  FROM orders o
  INNER JOIN users u ON o.user_id = u.id
  LEFT JOIN order_items oi ON o.id = oi.order_id
  LEFT JOIN products p ON oi.product_id = p.id
  WHERE o.status != 'cancelled'
  GROUP BY o.id
''').go();
```

## View with Aggregations

```dart
// 👇 Aggregated view
await db.customSelect('''
  CREATE VIEW user_stats AS
  SELECT 
    u.id as user_id,
    u.name as user_name,
    u.email as user_email,
    COUNT(o.id) as total_orders,
    SUM(o.total) as total_spent,
    AVG(o.total) as avg_order_value,
    MAX(o.total) as max_order_value,
    MIN(o.total) as min_order_value,
    JULIANDAY('now') - JULIANDAY(MAX(o.order_date)) as days_since_last_order
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  WHERE o.status = 'completed'
  GROUP BY u.id
''').go();
```

---

# Dropping Views

> **Removing views when no longer needed**

```dart
// 👇 Drop a view
await db.customSelect('''
  DROP VIEW IF EXISTS order_details
''').go();
```

---

# Querying Views

> **Using views in queries**

## Simple View Query

```dart
// 👇 Query a view
final results = await db.customSelect('''
  SELECT * FROM active_users
''').get();

for (final row in results) {
  print('User: ${row.data['name']} (${row.data['email']})');
}
```

## View with WHERE Clause

```dart
// 👇 View with conditions
final minSpent = 1000.0;
final results = await db.customSelect(
  '''
  SELECT * FROM user_stats
  WHERE total_spent > ?
  ORDER BY total_spent DESC
  ''',
  variables: [Variable.withDouble(minSpent)],
).get();
```

## Join View with Table

```dart
// 👇 Join view with regular table
final results = await db.customSelect('''
  SELECT 
    us.*,
    u.last_login
  FROM user_stats us
  INNER JOIN users u ON us.user_id = u.id
  WHERE us.total_orders > 5
  ORDER BY us.total_spent DESC
''').get();
```

---

# Real-World Example

> **Complete e-commerce view system**

```dart
// lib/database/view_service.dart
import 'package:drift/drift.dart';

class ViewService {
  final AppDatabase db;
  
  ViewService(this.db);

  // ==================== INITIALIZE VIEWS ====================
  
  Future<void> initializeViews() async {
    await _createActiveUserView();
    await _createOrderSummaryView();
    await _createProductStatsView();
    await _createUserStatsView();
    await _createDailySalesView();
  }
  
  // 👇 Active user view
  Future<void> _createActiveUserView() async {
    await db.customSelect('''
      CREATE VIEW IF NOT EXISTS active_users AS
      SELECT 
        u.id,
        u.name,
        u.email,
        u.is_active,
        u.is_verified,
        u.created_at,
        u.last_login,
        u.login_count,
        p.full_name,
        p.phone,
        p.city,
        p.country
      FROM users u
      LEFT JOIN user_profiles p ON u.id = p.user_id
      WHERE u.is_active = 1
    ''').go();
  }
  
  // 👇 Order summary view
  Future<void> _createOrderSummaryView() async {
    await db.customSelect('''
      CREATE VIEW IF NOT EXISTS order_summary AS
      SELECT 
        o.id as order_id,
        o.order_number,
        o.total,
        o.status,
        o.is_paid,
        o.is_shipped,
        o.is_delivered,
        o.order_date,
        o.shipped_date,
        o.delivered_date,
        u.id as user_id,
        u.name as user_name,
        u.email as user_email,
        COUNT(oi.id) as item_count,
        SUM(oi.quantity) as total_items,
        GROUP_CONCAT(DISTINCT p.name) as product_names,
        SUM(oi.total) as subtotal,
        o.total as grand_total
      FROM orders o
      INNER JOIN users u ON o.user_id = u.id
      LEFT JOIN order_items oi ON o.id = oi.order_id
      LEFT JOIN products p ON oi.product_id = p.id
      WHERE o.status != 'cancelled'
      GROUP BY o.id
    ''').go();
  }
  
  // 👇 Product stats view
  Future<void> _createProductStatsView() async {
    await db.customSelect('''
      CREATE VIEW IF NOT EXISTS product_stats AS
      SELECT 
        p.id as product_id,
        p.name as product_name,
        p.sku,
        p.price,
        p.stock,
        p.is_active,
        c.id as category_id,
        c.name as category_name,
        COUNT(DISTINCT oi.order_id) as order_count,
        SUM(oi.quantity) as total_sold,
        SUM(oi.total) as total_revenue,
        AVG(oi.total) as avg_order_value,
        COALESCE(AVG(r.rating), 0) as avg_rating,
        COUNT(DISTINCT r.id) as review_count
      FROM products p
      LEFT JOIN categories c ON p.category_id = c.id
      LEFT JOIN order_items oi ON p.id = oi.product_id
      LEFT JOIN reviews r ON p.id = r.product_id
      WHERE p.is_active = 1
      GROUP BY p.id
    ''').go();
  }
  
  // 👇 User stats view
  Future<void> _createUserStatsView() async {
    await db.customSelect('''
      CREATE VIEW IF NOT EXISTS user_stats AS
      SELECT 
        u.id as user_id,
        u.name as user_name,
        u.email as user_email,
        u.is_active,
        u.is_verified,
        u.created_at as user_created_at,
        COUNT(DISTINCT o.id) as total_orders,
        SUM(o.total) as total_spent,
        AVG(o.total) as avg_order_value,
        MAX(o.total) as max_order_value,
        MIN(o.total) as min_order_value,
        COUNT(DISTINCT r.id) as total_reviews,
        COALESCE(AVG(r.rating), 0) as avg_rating,
        COUNT(DISTINCT p.id) as unique_products_purchased,
        JULIANDAY('now') - JULIANDAY(MAX(o.order_date)) as days_since_last_order,
        SUM(o.total) / NULLIF(COUNT(DISTINCT o.id), 0) as value_per_order,
        COUNT(DISTINCT o.id) / NULLIF(JULIANDAY('now') - JULIANDAY(MIN(o.order_date)), 0) as orders_per_day
      FROM users u
      LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'completed'
      LEFT JOIN order_items oi ON o.id = oi.order_id
      LEFT JOIN products p ON oi.product_id = p.id
      LEFT JOIN reviews r ON u.id = r.user_id
      GROUP BY u.id
    ''').go();
  }
  
  // 👇 Daily sales view
  Future<void> _createDailySalesView() async {
    await db.customSelect('''
      CREATE VIEW IF NOT EXISTS daily_sales AS
      SELECT 
        DATE(o.order_date) as sale_date,
        COUNT(DISTINCT o.id) as order_count,
        COUNT(DISTINCT o.user_id) as unique_customers,
        SUM(o.total) as total_revenue,
        SUM(oi.quantity) as total_items,
        SUM(oi.total) as total_items_revenue,
        AVG(o.total) as avg_order_value,
        MAX(o.total) as max_order_value,
        MIN(o.total) as min_order_value,
        COUNT(DISTINCT p.id) as unique_products
      FROM orders o
      INNER JOIN order_items oi ON o.id = oi.order_id
      INNER JOIN products p ON oi.product_id = p.id
      WHERE o.status = 'completed'
      GROUP BY DATE(o.order_date)
      HAVING order_count > 0
      ORDER BY sale_date DESC
    ''').go();
  }

  // ==================== QUERY VIEWS ====================
  
  // 👇 Get active users from view
  Future<List<ActiveUser>> getActiveUsers() async {
    final results = await db.customSelect('''
      SELECT * FROM active_users
      ORDER BY name
    ''').get();
    
    return results.map((row) {
      return ActiveUser(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        email: row.data['email'] as String,
        isActive: (row.data['is_active'] as int) == 1,
        isVerified: (row.data['is_verified'] as int) == 1,
        createdAt: DateTime.fromMillisecondsSinceEpoch(row.data['created_at'] as int),
        lastLogin: row.data['last_login'] != null
            ? DateTime.fromMillisecondsSinceEpoch(row.data['last_login'] as int)
            : null,
        loginCount: row.data['login_count'] as int,
        fullName: row.data['full_name'] as String?,
        phone: row.data['phone'] as String?,
        city: row.data['city'] as String?,
        country: row.data['country'] as String?,
      );
    }).toList();
  }
  
  // 👇 Get order summary with filters
  Future<List<OrderSummary>> getOrderSummary({
    String? status,
    double? minTotal,
    double? maxTotal,
  }) async {
    final conditions = <String>[];
    final variables = <Variable>[];
    
    if (status != null && status.isNotEmpty) {
      conditions.add('status = ?');
      variables.add(Variable.withString(status));
    }
    
    if (minTotal != null) {
      conditions.add('grand_total >= ?');
      variables.add(Variable.withDouble(minTotal));
    }
    
    if (maxTotal != null) {
      conditions.add('grand_total <= ?');
      variables.add(Variable.withDouble(maxTotal));
    }
    
    final whereClause = conditions.isEmpty 
        ? '' 
        : 'WHERE ${conditions.join(' AND ')}';
    
    final results = await db.customSelect(
      '''
      SELECT * FROM order_summary
      $whereClause
      ORDER BY order_date DESC
      ''',
      variables: variables,
    ).get();
    
    return results.map((row) {
      return OrderSummary(
        orderId: row.data['order_id'] as int,
        orderNumber: row.data['order_number'] as String,
        total: row.data['grand_total'] as double,
        status: row.data['status'] as String,
        isPaid: (row.data['is_paid'] as int) == 1,
        isShipped: (row.data['is_shipped'] as int) == 1,
        isDelivered: (row.data['is_delivered'] as int) == 1,
        orderDate: DateTime.fromMillisecondsSinceEpoch(row.data['order_date'] as int),
        userId: row.data['user_id'] as int,
        userName: row.data['user_name'] as String,
        userEmail: row.data['user_email'] as String,
        itemCount: row.data['item_count'] as int,
        totalItems: row.data['total_items'] as int,
        productNames: row.data['product_names'] as String?,
        subtotal: row.data['subtotal'] as double,
      );
    }).toList();
  }
  
  // 👇 Get product stats with filters
  Future<List<ProductStats>> getProductStats({
    String? category,
    bool? inStock,
    int? minOrders,
  }) async {
    final conditions = <String>[];
    final variables = <Variable>[];
    
    if (category != null && category.isNotEmpty) {
      conditions.add('category_name = ?');
      variables.add(Variable.withString(category));
    }
    
    if (inStock != null) {
      conditions.add(inStock ? 'stock > 0' : 'stock = 0');
    }
    
    if (minOrders != null) {
      conditions.add('order_count >= ?');
      variables.add(Variable.withInt(minOrders));
    }
    
    final whereClause = conditions.isEmpty 
        ? '' 
        : 'WHERE ${conditions.join(' AND ')}';
    
    final results = await db.customSelect(
      '''
      SELECT * FROM product_stats
      $whereClause
      ORDER BY total_revenue DESC
      ''',
      variables: variables,
    ).get();
    
    return results.map((row) {
      return ProductStats(
        productId: row.data['product_id'] as int,
        productName: row.data['product_name'] as String,
        sku: row.data['sku'] as String,
        price: row.data['price'] as double,
        stock: row.data['stock'] as int,
        isActive: (row.data['is_active'] as int) == 1,
        categoryId: row.data['category_id'] as int?,
        categoryName: row.data['category_name'] as String?,
        orderCount: row.data['order_count'] as int,
        totalSold: row.data['total_sold'] as int,
        totalRevenue: row.data['total_revenue'] as double,
        avgOrderValue: row.data['avg_order_value'] as double,
        avgRating: row.data['avg_rating'] as double,
        reviewCount: row.data['review_count'] as int,
      );
    }).toList();
  }
  
  // 👇 Get user stats
  Future<List<UserStats>> getUserStats({
    int? minOrders,
    double? minSpent,
    double? minRating,
  }) async {
    final conditions = <String>[];
    final variables = <Variable>[];
    
    if (minOrders != null) {
      conditions.add('total_orders >= ?');
      variables.add(Variable.withInt(minOrders));
    }
    
    if (minSpent != null) {
      conditions.add('total_spent >= ?');
      variables.add(Variable.withDouble(minSpent));
    }
    
    if (minRating != null) {
      conditions.add('avg_rating >= ?');
      variables.add(Variable.withDouble(minRating));
    }
    
    final whereClause = conditions.isEmpty 
        ? '' 
        : 'WHERE ${conditions.join(' AND ')}';
    
    final results = await db.customSelect(
      '''
      SELECT * FROM user_stats
      $whereClause
      ORDER BY total_spent DESC
      ''',
      variables: variables,
    ).get();
    
    return results.map((row) {
      return UserStats(
        userId: row.data['user_id'] as int,
        userName: row.data['user_name'] as String,
        userEmail: row.data['user_email'] as String,
        isActive: (row.data['is_active'] as int) == 1,
        isVerified: (row.data['is_verified'] as int) == 1,
        userCreatedAt: DateTime.fromMillisecondsSinceEpoch(row.data['user_created_at'] as int),
        totalOrders: row.data['total_orders'] as int,
        totalSpent: row.data['total_spent'] as double,
        avgOrderValue: row.data['avg_order_value'] as double,
        maxOrderValue: row.data['max_order_value'] as double,
        minOrderValue: row.data['min_order_value'] as double,
        totalReviews: row.data['total_reviews'] as int,
        avgRating: row.data['avg_rating'] as double,
        uniqueProductsPurchased: row.data['unique_products_purchased'] as int,
        daysSinceLastOrder: row.data['days_since_last_order'] as double?,
        valuePerOrder: row.data['value_per_order'] as double?,
        ordersPerDay: row.data['orders_per_day'] as double?,
      );
    }).toList();
  }
  
  // 👇 Get daily sales
  Future<List<DailySales>> getDailySales({
    DateTime? fromDate,
    DateTime? toDate,
  }) async {
    final conditions = <String>[];
    final variables = <Variable>[];
    
    if (fromDate != null) {
      conditions.add('sale_date >= ?');
      variables.add(Variable.withDateTime(fromDate));
    }
    
    if (toDate != null) {
      conditions.add('sale_date <= ?');
      variables.add(Variable.withDateTime(toDate));
    }
    
    final whereClause = conditions.isEmpty 
        ? '' 
        : 'WHERE ${conditions.join(' AND ')}';
    
    final results = await db.customSelect(
      '''
      SELECT * FROM daily_sales
      $whereClause
      ORDER BY sale_date DESC
      ''',
      variables: variables,
    ).get();
    
    return results.map((row) {
      return DailySales(
        saleDate: DateTime.parse(row.data['sale_date'] as String),
        orderCount: row.data['order_count'] as int,
        uniqueCustomers: row.data['unique_customers'] as int,
        totalRevenue: row.data['total_revenue'] as double,
        totalItems: row.data['total_items'] as int,
        totalItemsRevenue: row.data['total_items_revenue'] as double,
        avgOrderValue: row.data['avg_order_value'] as double,
        maxOrderValue: row.data['max_order_value'] as double,
        minOrderValue: row.data['min_order_value'] as double,
        uniqueProducts: row.data['unique_products'] as int,
      );
    }).toList();
  }

  // ==================== DROP VIEWS ====================
  
  Future<void> dropAllViews() async {
    await db.customSelect('DROP VIEW IF EXISTS active_users').go();
    await db.customSelect('DROP VIEW IF EXISTS order_summary').go();
    await db.customSelect('DROP VIEW IF EXISTS product_stats').go();
    await db.customSelect('DROP VIEW IF EXISTS user_stats').go();
    await db.customSelect('DROP VIEW IF EXISTS daily_sales').go();
  }
}

// ==================== DATA CLASSES ====================

class ActiveUser {
  final int id;
  final String name;
  final String email;
  final bool isActive;
  final bool isVerified;
  final DateTime createdAt;
  final DateTime? lastLogin;
  final int loginCount;
  final String? fullName;
  final String? phone;
  final String? city;
  final String? country;
  
  ActiveUser({
    required this.id,
    required this.name,
    required this.email,
    required this.isActive,
    required this.isVerified,
    required this.createdAt,
    this.lastLogin,
    required this.loginCount,
    this.fullName,
    this.phone,
    this.city,
    this.country,
  });
}

class OrderSummary {
  final int orderId;
  final String orderNumber;
  final double total;
  final String status;
  final bool isPaid;
  final bool isShipped;
  final bool isDelivered;
  final DateTime orderDate;
  final int userId;
  final String userName;
  final String userEmail;
  final int itemCount;
  final int totalItems;
  final String? productNames;
  final double subtotal;
  
  OrderSummary({
    required this.orderId,
    required this.orderNumber,
    required this.total,
    required this.status,
    required this.isPaid,
    required this.isShipped,
    required this.isDelivered,
    required this.orderDate,
    required this.userId,
    required this.userName,
    required this.userEmail,
    required this.itemCount,
    required this.totalItems,
    this.productNames,
    required this.subtotal,
  });
}

class ProductStats {
  final int productId;
  final String productName;
  final String sku;
  final double price;
  final int stock;
  final bool isActive;
  final int? categoryId;
  final String? categoryName;
  final int orderCount;
  final int totalSold;
  final double totalRevenue;
  final double avgOrderValue;
  final double avgRating;
  final int reviewCount;
  
  ProductStats({
    required this.productId,
    required this.productName,
    required this.sku,
    required this.price,
    required this.stock,
    required this.isActive,
    this.categoryId,
    this.categoryName,
    required this.orderCount,
    required this.totalSold,
    required this.totalRevenue,
    required this.avgOrderValue,
    required this.avgRating,
    required this.reviewCount,
  });
}

class UserStats {
  final int userId;
  final String userName;
  final String userEmail;
  final bool isActive;
  final bool isVerified;
  final DateTime userCreatedAt;
  final int totalOrders;
  final double totalSpent;
  final double avgOrderValue;
  final double maxOrderValue;
  final double minOrderValue;
  final int totalReviews;
  final double avgRating;
  final int uniqueProductsPurchased;
  final double? daysSinceLastOrder;
  final double? valuePerOrder;
  final double? ordersPerDay;
  
  UserStats({
    required this.userId,
    required this.userName,
    required this.userEmail,
    required this.isActive,
    required this.isVerified,
    required this.userCreatedAt,
    required this.totalOrders,
    required this.totalSpent,
    required this.avgOrderValue,
    required this.maxOrderValue,
    required this.minOrderValue,
    required this.totalReviews,
    required this.avgRating,
    required this.uniqueProductsPurchased,
    this.daysSinceLastOrder,
    this.valuePerOrder,
    this.ordersPerDay,
  });
}

class DailySales {
  final DateTime saleDate;
  final int orderCount;
  final int uniqueCustomers;
  final double totalRevenue;
  final int totalItems;
  final double totalItemsRevenue;
  final double avgOrderValue;
  final double maxOrderValue;
  final double minOrderValue;
  final int uniqueProducts;
  
  DailySales({
    required this.saleDate,
    required this.orderCount,
    required this.uniqueCustomers,
    required this.totalRevenue,
    required this.totalItems,
    required this.totalItemsRevenue,
    required this.avgOrderValue,
    required this.maxOrderValue,
    required this.minOrderValue,
    required this.uniqueProducts,
  });
}
```

---

# View Best Practices

- **Use descriptive names** – Clear view names
- **Create with `IF NOT EXISTS`** – Prevent errors
- **Use views for complex queries** – Simplify application code
- **Keep views focused** – Single purpose per view
- **Test view performance** – Ensure efficiency
- **Document view purpose** – Explain what they do
- **Use views for security** – Restrict data access
- **Drop views when no longer needed** – Clean up

---

# View Checklist

| Practice | Description | Impact |
|----------|-------------|--------|
| **IF NOT EXISTS** | Prevent errors | High |
| **Descriptive Names** | Clear purpose | Medium |
| **Focused Views** | Single purpose | Medium |
| **Performance** | Test efficiency | High |
| **Documentation** | Explain purpose | Medium |
| **Security** | Data restriction | High |

---

# Common Mistakes

## Mistake 1: View too complex

Wrong:
```dart
// 🚫 View with too many joins
CREATE VIEW everything AS
SELECT * FROM users
JOIN orders ON ...
JOIN products ON ...
JOIN categories ON ...
```

Correct:
```dart
// ✅ Split into multiple views
CREATE VIEW user_orders AS ...
CREATE VIEW order_products AS ...
```

## Mistake 2: Not using IF NOT EXISTS

Wrong:
```dart
// 🚫 Error if view exists
CREATE VIEW my_view AS ...
```

Correct:
```dart
// ✅ Safe creation
CREATE VIEW IF NOT EXISTS my_view AS ...
```

## Mistake 3: Dropping views incorrectly

Wrong:
```dart
// 🚫 Error if view doesn't exist
DROP VIEW my_view
```

Correct:
```dart
// ✅ Safe drop
DROP VIEW IF EXISTS my_view
```

---

# Summary

| Feature | Description | Use Case |
|---------|-------------|----------|
| **CREATE VIEW** | Define virtual table | Complex queries |
| **IF NOT EXISTS** | Safe creation | Prevent errors |
| **DROP VIEW** | Remove view | Cleanup |
| **Query View** | Use like table | Data access |

---

# Next Steps

Now you understand views, let's dive deeper:

- [Indexes](link) – Performance optimization
- [Triggers](link) – Database triggers
- [CTEs](link) – Common Table Expressions

---

# Did You Know?

- **Views are virtual** – No physical storage

- **Views are always current** – Reflect live data

- **Views can be indexed** – With materialized views

- **Views can be used in joins** – Like regular tables

- **Views can have WHERE clauses** – When queried

- **Views can be nested** – View based on another view

- **Views improve security** – Restrict data access

- **Views simplify queries** – Encapsulate complexity

---

