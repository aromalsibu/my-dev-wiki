## Custom SQL Expressions

**Writing raw SQL expressions in Drift queries**

---

# What is it?

**Custom SQL Expressions** allow you to write raw SQL directly in your Drift queries when the built-in query builder isn't flexible enough. This gives you full access to SQLite's capabilities while maintaining type safety through Drift's mapping features.

> **Think of Custom SQL Expressions like "using a master key"** – when the regular tools (query builder) can't open the door, you bring out the master key (raw SQL) to access everything the database has to offer.

```dart
// 👇 Custom SQL expression in a query
final results = await (select(users)
  ..where((u) => 
    u.age > const Variable(18) &
    u.email.customExpression('LIKE "%@gmail.com"')
  ))
  .get();

// 👇 Using custom expression for complex operations
final results = await customSelect('''
  SELECT 
    *,
    RANDOM() as random_order
  FROM users
  ORDER BY random_order
  LIMIT 10
''').get();
```

> **What's happening here?**
> - **`customExpression()`** – Raw SQL in WHERE clauses
> - **`customSelect()`** – Complete raw SQL queries
> - **`customUpdate()`** – Raw SQL for updates
> - **`customInsert()`** – Raw SQL for inserts

---

# Why does it exist?

- **Complex Logic** – SQL functions not available in query builder
- **Performance** – Write optimized SQL directly
- **Flexibility** – Use any SQLite feature
- **Legacy SQL** – Reuse existing SQL queries
- **Custom Functions** – Use SQLite extensions
- **Migration** – Complex schema changes

---

# Custom Expressions in WHERE

> **Using raw SQL in WHERE clauses**

## Basic Custom Expression

```dart
// 👇 Custom expression for email domain filter
final results = await (select(users)
  ..where((u) => 
    u.email.customExpression('LIKE "%@gmail.com"')
  ))
  .get();

// 👇 Custom expression with JSON functions
final results = await (select(users)
  ..where((u) => 
    u.metadata.customExpression("->>'$.age' = '25'")
  ))
  .get();

// 👇 Custom expression with regular expressions
final results = await (select(users)
  ..where((u) => 
    u.name.customExpression("REGEXP '^[A-Z]'")
  ))
  .get();
```

## Custom Expressions with Variables

```dart
// 👇 Using variables with custom expressions
final searchTerm = 'John';
final results = await (select(users)
  ..where((u) => 
    u.name.customExpression(
      'LIKE ?',
      variables: [Variable.withString('%$searchTerm%')]
    )
  ))
  .get();

// 👇 Multiple variables
final minAge = 18;
final maxAge = 30;
final results = await (select(users)
  ..where((u) => 
    u.age.customExpression(
      'BETWEEN ? AND ?',
      variables: [
        Variable.withInt(minAge),
        Variable.withInt(maxAge),
      ]
    )
  ))
  .get();
```

---

# Custom Select Queries

> **Full raw SQL SELECT queries**

## Basic Custom Select

```dart
// 👇 Simple custom select
final results = await customSelect('''
  SELECT 
    id, 
    name, 
    email,
    LENGTH(name) as name_length
  FROM users
  WHERE is_active = 1
  ORDER BY name_length DESC
''').get();

// Access results
for (final row in results) {
  print('ID: ${row.data['id']}');
  print('Name: ${row.data['name']} (${row.data['name_length']} chars)');
  print('Email: ${row.data['email']}');
}
```

## Custom Select with Joins

```dart
// 👇 Complex join with subqueries
final results = await customSelect('''
  SELECT 
    u.id as user_id,
    u.name as user_name,
    COUNT(o.id) as order_count,
    SUM(o.total) as total_spent,
    (
      SELECT COUNT(*) 
      FROM reviews r 
      WHERE r.user_id = u.id
    ) as review_count
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  WHERE u.is_active = 1
  GROUP BY u.id
  HAVING order_count > 5
  ORDER BY total_spent DESC
  LIMIT 10
''').get();
```

## Custom Select with Variables

```dart
// 👇 Using variables in custom select
final minOrders = 5;
final results = await customSelect(
  '''
  SELECT 
    u.id,
    u.name,
    COUNT(o.id) as order_count,
    SUM(o.total) as total_spent
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id
  HAVING order_count > ?
  ORDER BY total_spent DESC
  ''',
  variables: [Variable.withInt(minOrders)],
).get();
```

---

# Custom Update Queries

> **Raw SQL for updates**

## Basic Custom Update

```dart
// 👇 Update with custom SQL
await customUpdate('''
  UPDATE users 
  SET status = 'inactive'
  WHERE is_active = 0
  AND last_login < datetime('now', '-1 year')
''').go();

// 👇 Update with variables
final newStatus = 'premium';
final minOrders = 10;
await customUpdate(
  '''
  UPDATE users 
  SET status = ?
  WHERE id IN (
    SELECT user_id 
    FROM orders 
    GROUP BY user_id 
    HAVING COUNT(*) > ?
  )
  ''',
  variables: [
    Variable.withString(newStatus),
    Variable.withInt(minOrders),
  ],
).go();
```

---

# Custom Delete Queries

> **Raw SQL for deletes**

```dart
// 👇 Delete with custom SQL
await customDelete('''
  DELETE FROM users 
  WHERE is_active = 0 
  AND created_at < datetime('now', '-2 years')
''').go();

// 👇 Delete with variables
final cutoffDate = DateTime.now().subtract(Duration(days: 365));
await customDelete(
  '''
  DELETE FROM users 
  WHERE is_deleted = 1 
  AND deleted_at < ?
  ''',
  variables: [Variable.withString(cutoffDate.toIso8601String())],
).go();
```

---

# Custom Insert Queries

> **Raw SQL for inserts**

```dart
// 👇 Bulk insert with custom SQL
await customInsert('''
  INSERT INTO users (name, email, created_at)
  VALUES 
    ('User 1', 'user1@example.com', datetime('now')),
    ('User 2', 'user2@example.com', datetime('now')),
    ('User 3', 'user3@example.com', datetime('now'))
''').go();

// 👇 Insert with variables
await customInsert(
  '''
  INSERT INTO users (name, email, created_at)
  VALUES (?, ?, datetime('now'))
  ''',
  variables: [
    Variable.withString('John Doe'),
    Variable.withString('john@example.com'),
  ],
).go();
```

---

# Real-World Example

> **Complete e-commerce custom SQL system**

```dart
// lib/database/custom_sql_service.dart
import 'package:drift/drift.dart';

class CustomSqlService {
  final AppDatabase db;
  
  CustomSqlService(this.db);

  // ==================== COMPLEX REPORTS ====================
  
  // 👇 Get user segmentation report with custom SQL
  Future<List<UserSegment>> getUserSegments() async {
    final results = await db.customSelect('''
      WITH user_stats AS (
        SELECT 
          u.id,
          u.name,
          u.email,
          COUNT(o.id) as order_count,
          SUM(o.total) as total_spent,
          AVG(o.total) as avg_order_value,
          JULIANDAY('now') - JULIANDAY(MAX(o.order_date)) as days_since_last_order
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        WHERE o.status = 'completed'
        GROUP BY u.id
      )
      SELECT 
        *,
        CASE 
          WHEN total_spent > 1000 THEN 'VIP'
          WHEN total_spent > 500 THEN 'Premium'
          WHEN total_spent > 100 THEN 'Regular'
          WHEN order_count > 0 THEN 'Occasional'
          ELSE 'Inactive'
        END as segment,
        CASE 
          WHEN days_since_last_order < 30 THEN 'Active'
          WHEN days_since_last_order < 90 THEN 'At Risk'
          ELSE 'Inactive'
        END as engagement_status
      FROM user_stats
      ORDER BY total_spent DESC
    ''').get();
    
    return results.map((row) {
      return UserSegment(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        email: row.data['email'] as String,
        orderCount: row.data['order_count'] as int? ?? 0,
        totalSpent: row.data['total_spent'] as double? ?? 0.0,
        avgOrderValue: row.data['avg_order_value'] as double? ?? 0.0,
        daysSinceLastOrder: row.data['days_since_last_order'] as double? ?? 999,
        segment: row.data['segment'] as String,
        engagementStatus: row.data['engagement_status'] as String,
      );
    }).toList();
  }
  
  // 👇 Get product performance with custom SQL
  Future<List<ProductPerformance>> getProductPerformance() async {
    final results = await db.customSelect('''
      WITH product_stats AS (
        SELECT 
          p.id,
          p.name,
          p.sku,
          p.price,
          p.stock,
          COUNT(oi.id) as order_count,
          SUM(oi.quantity) as total_sold,
          SUM(oi.total) as total_revenue,
          AVG(oi.quantity) as avg_order_size,
          (
            SELECT COUNT(*) 
            FROM reviews r 
            WHERE r.product_id = p.id
          ) as review_count,
          (
            SELECT AVG(rating) 
            FROM reviews r 
            WHERE r.product_id = p.id
          ) as avg_rating
        FROM products p
        LEFT JOIN order_items oi ON p.id = oi.product_id
        GROUP BY p.id
      )
      SELECT 
        *,
        CASE 
          WHEN total_revenue > 10000 THEN 'Best Seller'
          WHEN total_revenue > 5000 THEN 'Top Seller'
          WHEN total_revenue > 1000 THEN 'Good Seller'
          WHEN total_revenue > 0 THEN 'Slow Seller'
          ELSE 'New'
        END as performance_tier,
        CASE 
          WHEN stock < reorder_level THEN 'Low Stock'
          WHEN stock < reorder_level * 2 THEN 'Warning'
          ELSE 'Healthy'
        END as stock_status,
        CASE 
          WHEN avg_rating > 4.5 THEN 'Highly Rated'
          WHEN avg_rating > 4.0 THEN 'Rated'
          WHEN avg_rating > 3.0 THEN 'Average'
          ELSE 'Needs Review'
        END as rating_status
      FROM product_stats
      ORDER BY total_revenue DESC
    ''').get();
    
    return results.map((row) {
      return ProductPerformance(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        sku: row.data['sku'] as String,
        price: row.data['price'] as double,
        stock: row.data['stock'] as int,
        orderCount: row.data['order_count'] as int? ?? 0,
        totalSold: row.data['total_sold'] as int? ?? 0,
        totalRevenue: row.data['total_revenue'] as double? ?? 0.0,
        avgOrderSize: row.data['avg_order_size'] as double? ?? 0.0,
        reviewCount: row.data['review_count'] as int? ?? 0,
        avgRating: row.data['avg_rating'] as double? ?? 0.0,
        performanceTier: row.data['performance_tier'] as String,
        stockStatus: row.data['stock_status'] as String,
        ratingStatus: row.data['rating_status'] as String,
      );
    }).toList();
  }

  // ==================== ADVANCED ANALYTICS ====================
  
  // 👇 Get cohort analysis with custom SQL
  Future<List<CohortData>> getCohortAnalysis() async {
    final results = await db.customSelect('''
      WITH cohorts AS (
        SELECT 
          u.id as user_id,
          STRFTIME('%Y-%m', u.created_at) as cohort_month,
          o.id as order_id,
          o.order_date,
          o.total
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        WHERE o.status = 'completed'
      ),
      cohort_orders AS (
        SELECT 
          cohort_month,
          STRFTIME('%Y-%m', order_date) as order_month,
          COUNT(DISTINCT user_id) as active_users,
          COUNT(order_id) as total_orders,
          SUM(total) as total_revenue
        FROM cohorts
        WHERE order_month >= cohort_month
        GROUP BY cohort_month, order_month
      ),
      cohort_bases AS (
        SELECT 
          cohort_month,
          COUNT(DISTINCT user_id) as base_users
        FROM cohorts
        GROUP BY cohort_month
      )
      SELECT 
        co.*,
        cb.base_users,
        ROUND(CAST(co.active_users AS REAL) / cb.base_users * 100, 2) as retention_rate
      FROM cohort_orders co
      JOIN cohort_bases cb ON co.cohort_month = cb.cohort_month
      ORDER BY co.cohort_month, co.order_month
    ''').get();
    
    return results.map((row) {
      return CohortData(
        cohortMonth: row.data['cohort_month'] as String,
        orderMonth: row.data['order_month'] as String,
        activeUsers: row.data['active_users'] as int,
        totalOrders: row.data['total_orders'] as int,
        totalRevenue: row.data['total_revenue'] as double? ?? 0.0,
        baseUsers: row.data['base_users'] as int,
        retentionRate: row.data['retention_rate'] as double? ?? 0.0,
      );
    }).toList();
  }
  
  // 👇 Get sales forecast with custom SQL
  Future<List<SalesForecast>> getSalesForecast() async {
    final results = await db.customSelect('''
      WITH monthly_sales AS (
        SELECT 
          STRFTIME('%Y-%m', order_date) as month,
          SUM(total) as revenue,
          COUNT(*) as orders
        FROM orders
        WHERE status = 'completed'
        GROUP BY month
        ORDER BY month DESC
        LIMIT 12
      ),
      avg_sales AS (
        SELECT 
          AVG(revenue) as avg_revenue,
          AVG(orders) as avg_orders
        FROM monthly_sales
      )
      SELECT 
        'Next Month' as forecast_period,
        avg_revenue * 1.1 as forecast_revenue,
        avg_orders * 1.05 as forecast_orders,
        'Based on last 12 months with 10% growth' as forecast_notes
      FROM avg_sales
      UNION ALL
      SELECT 
        'Next Quarter' as forecast_period,
        avg_revenue * 3.2 as forecast_revenue,
        avg_orders * 3.1 as forecast_orders,
        'Based on last 12 months with 5% growth' as forecast_notes
      FROM avg_sales
      UNION ALL
      SELECT 
        'Next Year' as forecast_period,
        avg_revenue * 12.5 as forecast_revenue,
        avg_orders * 12 as forecast_orders,
        'Based on last 12 months with 2% growth' as forecast_notes
      FROM avg_sales
    ''').get();
    
    return results.map((row) {
      return SalesForecast(
        forecastPeriod: row.data['forecast_period'] as String,
        forecastRevenue: row.data['forecast_revenue'] as double,
        forecastOrders: row.data['forecast_orders'] as int,
        forecastNotes: row.data['forecast_notes'] as String,
      );
    }).toList();
  }

  // ==================== DATA MAINTENANCE ====================
  
  // 👇 Clean up duplicate records
  Future<int> cleanupDuplicateEmails() async {
    final result = await db.customDelete('''
      DELETE FROM users 
      WHERE id NOT IN (
        SELECT MIN(id) 
        FROM users 
        GROUP BY email
      )
      AND email IN (
        SELECT email 
        FROM users 
        GROUP BY email 
        HAVING COUNT(*) > 1
      )
    ''').go();
    
    return result;
  }
  
  // 👇 Archive old data with custom SQL
  Future<int> archiveOldData(DateTime cutoffDate) async {
    // Create archive table if not exists
    await db.customSelect('''
      CREATE TABLE IF NOT EXISTS archived_orders AS 
      SELECT * FROM orders WHERE 1 = 0
    ''').go();
    
    // Move old orders to archive
    final result = await db.customInsert(
      '''
      INSERT INTO archived_orders 
      SELECT * FROM orders 
      WHERE order_date < ? 
      AND status IN ('completed', 'cancelled')
      ''',
      variables: [
        Variable.withString(cutoffDate.toIso8601String()),
      ],
    ).go();
    
    // Delete archived orders
    await db.customDelete(
      '''
      DELETE FROM orders 
      WHERE order_date < ? 
      AND status IN ('completed', 'cancelled')
      ''',
      variables: [
        Variable.withString(cutoffDate.toIso8601String()),
      ],
    ).go();
    
    return result;
  }
  
  // 👇 Update product prices with calculation
  Future<int> updatePricesWithCustomLogic() async {
    final result = await db.customUpdate('''
      UPDATE products 
      SET 
        price = CASE 
          WHEN stock < 10 THEN price * 1.2
          WHEN stock < 50 THEN price * 1.1
          WHEN stock < 100 THEN price * 1.05
          ELSE price
        END,
        updated_at = datetime('now')
      WHERE is_active = 1
    ''').go();
    
    return result;
  }

  // ==================== COMPLEX SEARCH ====================
  
  // 👇 Full-text search across multiple tables
  Future<List<SearchResult>> fullTextSearch(String query) async {
    final searchTerm = '%$query%';
    
    final results = await db.customSelect('''
      SELECT 
        'user' as type,
        u.id as id,
        u.name as name,
        u.email as detail,
        u.created_at as date,
        'User' as category
      FROM users u
      WHERE u.name LIKE ? OR u.email LIKE ?
      
      UNION ALL
      
      SELECT 
        'product' as type,
        p.id as id,
        p.name as name,
        p.sku as detail,
        p.created_at as date,
        p.category as category
      FROM products p
      WHERE p.name LIKE ? OR p.sku LIKE ? OR p.description LIKE ?
      
      UNION ALL
      
      SELECT 
        'order' as type,
        o.id as id,
        o.order_number as name,
        CAST(o.total AS TEXT) as detail,
        o.order_date as date,
        'Order' as category
      FROM orders o
      WHERE o.order_number LIKE ? OR o.status LIKE ?
      
      ORDER BY date DESC
      LIMIT 50
    ''', variables: [
      // User search
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
      // Product search
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
      // Order search
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
    ]).get();
    
    return results.map((row) {
      return SearchResult(
        type: row.data['type'] as String,
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        detail: row.data['detail'] as String?,
        date: DateTime.fromMillisecondsSinceEpoch(row.data['date'] as int),
        category: row.data['category'] as String?,
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class UserSegment {
  final int id;
  final String name;
  final String email;
  final int orderCount;
  final double totalSpent;
  final double avgOrderValue;
  final double daysSinceLastOrder;
  final String segment;
  final String engagementStatus;
  
  UserSegment({
    required this.id,
    required this.name,
    required this.email,
    required this.orderCount,
    required this.totalSpent,
    required this.avgOrderValue,
    required this.daysSinceLastOrder,
    required this.segment,
    required this.engagementStatus,
  });
}

class ProductPerformance {
  final int id;
  final String name;
  final String sku;
  final double price;
  final int stock;
  final int orderCount;
  final int totalSold;
  final double totalRevenue;
  final double avgOrderSize;
  final int reviewCount;
  final double avgRating;
  final String performanceTier;
  final String stockStatus;
  final String ratingStatus;
  
  ProductPerformance({
    required this.id,
    required this.name,
    required this.sku,
    required this.price,
    required this.stock,
    required this.orderCount,
    required this.totalSold,
    required this.totalRevenue,
    required this.avgOrderSize,
    required this.reviewCount,
    required this.avgRating,
    required this.performanceTier,
    required this.stockStatus,
    required this.ratingStatus,
  });
}

class CohortData {
  final String cohortMonth;
  final String orderMonth;
  final int activeUsers;
  final int totalOrders;
  final double totalRevenue;
  final int baseUsers;
  final double retentionRate;
  
  CohortData({
    required this.cohortMonth,
    required this.orderMonth,
    required this.activeUsers,
    required this.totalOrders,
    required this.totalRevenue,
    required this.baseUsers,
    required this.retentionRate,
  });
}

class SalesForecast {
  final String forecastPeriod;
  final double forecastRevenue;
  final int forecastOrders;
  final String forecastNotes;
  
  SalesForecast({
    required this.forecastPeriod,
    required this.forecastRevenue,
    required this.forecastOrders,
    required this.forecastNotes,
  });
}
```

---

# Best Practices

- **Use variables** – Prevent SQL injection
- **Document complex queries** – Explain what they do
- **Test queries** – Verify they work correctly
- **Use query builder when possible** – Better type safety
- **Keep queries readable** – Use proper formatting
- **Use CTEs** – For complex queries
- **Add indexes** – For performance
- **Test with realistic data** – Ensure performance

---

# Common Mistakes

## Mistake 1: SQL injection risk

Wrong:
```dart
// 🚫 Risk of SQL injection
final name = "Robert'); DROP TABLE users; --";
await customSelect('SELECT * FROM users WHERE name = "$name"').get();
```

Correct:
```dart
// ✅ Use variables to prevent injection
final name = "Robert'); DROP TABLE users; --";
await customSelect(
  'SELECT * FROM users WHERE name = ?',
  variables: [Variable.withString(name)],
).get();
```

## Mistake 2: Incorrect variable types

Wrong:
```dart
// 🚫 Variable type mismatch
final age = '25'; // String, but needs int
await customSelect(
  'SELECT * FROM users WHERE age > ?',
  variables: [Variable.withString(age)],
).get();
```

Correct:
```dart
// ✅ Use correct variable type
final age = 25;
await customSelect(
  'SELECT * FROM users WHERE age > ?',
  variables: [Variable.withInt(age)],
).get();
```

## Mistake 3: Not handling results safely

Wrong:
```dart
// 🚫 Type cast errors
final id = row.data['id']; // Could be null
final name = row.data['name'] as String; // Might fail
```

Correct:
```dart
// ✅ Safe handling
final id = row.data['id'] as int? ?? 0;
final name = row.data['name'] as String? ?? 'Unknown';
```

---

# Summary

| Method | Purpose | Use Case |
|--------|---------|----------|
| `customExpression()` | Raw SQL in WHERE | Complex filters |
| `customSelect()` | Raw SQL SELECT | Complex queries |
| `customUpdate()` | Raw SQL UPDATE | Bulk updates |
| `customDelete()` | Raw SQL DELETE | Bulk deletes |
| `customInsert()` | Raw SQL INSERT | Bulk inserts |

---

# Next Steps

Now you understand custom SQL expressions, let's dive deeper:

- [Dynamic Queries](link) – Building queries dynamically
- [Joins](link) – Table joins
- [Views](link) – Database views

---

# Did You Know?

- **Custom SQL bypasses type safety** – Use with caution

- **Variables prevent SQL injection** – Always use them

- **CTEs (WITH) are powerful** – Complex queries made simple

- **Full-text search is possible** – With FTS5 extension

- **JSON functions are available** – In modern SQLite

- **Regular expressions** – With the REGEXP extension

- **Window functions** – For advanced analytics

- **Recursive CTEs** – For hierarchical data

---

