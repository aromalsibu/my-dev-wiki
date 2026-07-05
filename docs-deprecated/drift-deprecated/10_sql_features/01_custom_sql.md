## Custom SQL

**Writing raw SQL in Drift for maximum flexibility**

---

# What is it?

**Custom SQL** allows you to write raw SQL queries directly in Drift when the built-in query builder isn't flexible enough. This gives you access to all SQLite features, including complex functions, CTEs, window functions, and advanced optimizations. Drift provides methods like `customSelect()`, `customUpdate()`, `customInsert()`, and `customDelete()` for executing raw SQL.

> **Think of Custom SQL like "using a power tool"** – sometimes you need more power and precision than the standard tools (query builder) can provide, and you need to go straight to the source.

```dart
// 👇 Custom SQL with CTE (Common Table Expression)
final results = await customSelect('''
  WITH user_stats AS (
    SELECT 
      user_id,
      COUNT(*) as order_count,
      SUM(total) as total_spent
    FROM orders
    GROUP BY user_id
  )
  SELECT 
    u.name,
    u.email,
    us.order_count,
    us.total_spent
  FROM users u
  LEFT JOIN user_stats us ON u.id = us.user_id
  WHERE us.order_count > 5
  ORDER BY us.total_spent DESC
''').get();
```

> **What's happening here?**
> - **`customSelect()`** – Execute raw SELECT queries
> - **CTE** – Complex query with Common Table Expression
> - **Joins** – Raw SQL with table aliases
> - **Aggregations** – Advanced grouping and sorting

---

# Why does it exist?

- **Complex Queries** – Queries beyond query builder capabilities
- **Performance** – Write optimized SQL directly
- **Features** – Use SQLite-specific functions
- **Migration** – Reuse existing SQL queries
- **Legacy Integration** – Work with existing database schemas
- **Full Control** – Complete control over SQL generation

---

# Custom Select

> **Reading data with raw SQL**

## Basic Custom Select

```dart
// 👇 Simple custom select
final results = await customSelect('''
  SELECT id, name, email 
  FROM users 
  WHERE is_active = 1
  ORDER BY name
''').get();

for (final row in results) {
  print('ID: ${row.data['id']}, Name: ${row.data['name']}');
}
```

## Custom Select with Variables

```dart
// 👇 Custom select with variables
final minAge = 18;
final maxAge = 65;

final results = await customSelect(
  '''
  SELECT * FROM users 
  WHERE age BETWEEN ? AND ?
  AND is_active = 1
  ''',
  variables: [
    Variable.withInt(minAge),
    Variable.withInt(maxAge),
  ],
).get();
```

## Custom Select with Complex Joins

```dart
// 👇 Complex custom select with multiple joins
final results = await customSelect('''
  SELECT 
    o.id as order_id,
    o.order_number,
    o.total,
    u.name as user_name,
    u.email as user_email,
    GROUP_CONCAT(p.name) as products
  FROM orders o
  INNER JOIN users u ON o.user_id = u.id
  INNER JOIN order_items oi ON o.id = oi.order_id
  INNER JOIN products p ON oi.product_id = p.id
  WHERE o.status = 'completed'
  GROUP BY o.id, o.order_number, o.total, u.name, u.email
  ORDER BY o.total DESC
  LIMIT 10
''').get();
```

---

# Custom Update

> **Updating data with raw SQL**

## Basic Custom Update

```dart
// 👇 Simple custom update
await customUpdate('''
  UPDATE users 
  SET status = 'inactive' 
  WHERE last_login < datetime('now', '-1 year')
''').go();
```

## Custom Update with Variables

```dart
// 👇 Custom update with variables
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

## Complex Custom Update

```dart
// 👇 Complex update with calculations
await customUpdate('''
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
```

---

# Custom Insert

> **Inserting data with raw SQL**

## Basic Custom Insert

```dart
// 👇 Simple custom insert
await customInsert('''
  INSERT INTO users (name, email, created_at)
  VALUES ('John Doe', 'john@example.com', datetime('now'))
''').go();
```

## Bulk Custom Insert

```dart
// 👇 Bulk insert with custom SQL
await customInsert('''
  INSERT INTO users (name, email, created_at) VALUES
  ('User 1', 'user1@example.com', datetime('now')),
  ('User 2', 'user2@example.com', datetime('now')),
  ('User 3', 'user3@example.com', datetime('now'))
''').go();
```

## Conditional Custom Insert

```dart
// 👇 Insert with conflict handling
await customInsert('''
  INSERT OR REPLACE INTO users (id, name, email, updated_at)
  VALUES (?, ?, ?, datetime('now'))
  ''',
  variables: [
    Variable.withInt(1),
    Variable.withString('John Updated'),
    Variable.withString('john.updated@example.com'),
  ],
).go();
```

---

# Custom Delete

> **Deleting data with raw SQL**

## Basic Custom Delete

```dart
// 👇 Simple custom delete
await customDelete('''
  DELETE FROM users 
  WHERE is_active = 0 
  AND created_at < datetime('now', '-2 years')
''').go();
```

## Custom Delete with Variables

```dart
// 👇 Custom delete with variables
final cutoffDate = DateTime.now().subtract(Duration(days: 365));

await customDelete(
  '''
  DELETE FROM users 
  WHERE is_deleted = 1 
  AND deleted_at < ?
  ''',
  variables: [
    Variable.withString(cutoffDate.toIso8601String()),
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
  
  // 👇 User segmentation with CTE
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

  // ==================== ADVANCED ANALYTICS ====================
  
  // 👇 Cohort analysis
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
  
  // 👇 Archive old data
  Future<int> archiveOldData(DateTime cutoffDate) async {
    // Create archive table
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

  // ==================== FULL-TEXT SEARCH ====================
  
  // 👇 Global search across tables
  Future<List<SearchResult>> globalSearch(String query) async {
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

  // ==================== WINDOW FUNCTIONS ====================
  
  // 👇 Ranking with window functions
  Future<List<ProductRanking>> getProductRankings() async {
    final results = await db.customSelect('''
      SELECT 
        p.id,
        p.name,
        p.sku,
        p.price,
        SUM(oi.quantity) as total_sold,
        SUM(oi.total) as total_revenue,
        RANK() OVER (ORDER BY SUM(oi.total) DESC) as revenue_rank,
        DENSE_RANK() OVER (ORDER BY SUM(oi.quantity) DESC) as sales_rank,
        NTILE(4) OVER (ORDER BY SUM(oi.total) DESC) as quartile
      FROM products p
      LEFT JOIN order_items oi ON p.id = oi.product_id
      WHERE p.is_active = 1
      GROUP BY p.id
      HAVING total_sold > 0
      ORDER BY revenue_rank
      LIMIT 20
    ''').get();
    
    return results.map((row) {
      return ProductRanking(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        sku: row.data['sku'] as String,
        price: row.data['price'] as double,
        totalSold: row.data['total_sold'] as int,
        totalRevenue: row.data['total_revenue'] as double,
        revenueRank: row.data['revenue_rank'] as int,
        salesRank: row.data['sales_rank'] as int,
        quartile: row.data['quartile'] as int,
      );
    }).toList();
  }

  // ==================== RECURSIVE QUERIES ====================
  
  // 👇 Get category hierarchy with recursive CTE
  Future<List<CategoryNode>> getCategoryHierarchy() async {
    final results = await db.customSelect('''
      WITH RECURSIVE category_tree AS (
        SELECT 
          id,
          name,
          parent_id,
          0 as level,
          name as path
        FROM categories
        WHERE parent_id IS NULL
        
        UNION ALL
        
        SELECT 
          c.id,
          c.name,
          c.parent_id,
          ct.level + 1,
          ct.path || ' > ' || c.name
        FROM categories c
        INNER JOIN category_tree ct ON c.parent_id = ct.id
      )
      SELECT 
        id,
        name,
        parent_id,
        level,
        path,
        CASE 
          WHEN level = 0 THEN 'Root'
          WHEN level = 1 THEN 'Parent'
          ELSE 'Child'
        END as type
      FROM category_tree
      ORDER BY path
    ''').get();
    
    return results.map((row) {
      return CategoryNode(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        parentId: row.data['parent_id'] as int?,
        level: row.data['level'] as int,
        path: row.data['path'] as String,
        type: row.data['type'] as String,
      );
    }).toList();
  }

  // ==================== AGGREGATION WITH ROLLUP ====================
  
  // 👇 Sales summary with ROLLUP
  Future<List<SalesSummary>> getSalesSummary() async {
    final results = await db.customSelect('''
      SELECT 
        COALESCE(c.name, 'All Categories') as category,
        COALESCE(p.name, 'All Products') as product,
        COALESCE(STRFTIME('%Y-%m', o.order_date), 'All Periods') as period,
        COUNT(DISTINCT o.id) as order_count,
        SUM(oi.quantity) as total_items,
        SUM(oi.total) as total_revenue
      FROM orders o
      INNER JOIN order_items oi ON o.id = oi.order_id
      INNER JOIN products p ON oi.product_id = p.id
      LEFT JOIN categories c ON p.category_id = c.id
      WHERE o.status = 'completed'
      GROUP BY category, product, period WITH ROLLUP
      ORDER BY category, product, period
    ''').get();
    
    return results.map((row) {
      return SalesSummary(
        category: row.data['category'] as String,
        product: row.data['product'] as String,
        period: row.data['period'] as String,
        orderCount: row.data['order_count'] as int,
        totalItems: row.data['total_items'] as int,
        totalRevenue: row.data['total_revenue'] as double,
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

class SearchResult {
  final String type;
  final int id;
  final String name;
  final String? detail;
  final DateTime date;
  final String? category;
  
  SearchResult({
    required this.type,
    required this.id,
    required this.name,
    this.detail,
    required this.date,
    this.category,
  });
}

class ProductRanking {
  final int id;
  final String name;
  final String sku;
  final double price;
  final int totalSold;
  final double totalRevenue;
  final int revenueRank;
  final int salesRank;
  final int quartile;
  
  ProductRanking({
    required this.id,
    required this.name,
    required this.sku,
    required this.price,
    required this.totalSold,
    required this.totalRevenue,
    required this.revenueRank,
    required this.salesRank,
    required this.quartile,
  });
}

class CategoryNode {
  final int id;
  final String name;
  final int? parentId;
  final int level;
  final String path;
  final String type;
  
  CategoryNode({
    required this.id,
    required this.name,
    required this.parentId,
    required this.level,
    required this.path,
    required this.type,
  });
}

class SalesSummary {
  final String category;
  final String product;
  final String period;
  final int orderCount;
  final int totalItems;
  final double totalRevenue;
  
  SalesSummary({
    required this.category,
    required this.product,
    required this.period,
    required this.orderCount,
    required this.totalItems,
    required this.totalRevenue,
  });
}
```

---

# Custom SQL Best Practices

- **Use variables** – Prevent SQL injection
- **Use CTEs for complex queries** – Improve readability
- **Document complex queries** – Explain what they do
- **Test queries** – Verify they work correctly
- **Use query builder when possible** – Better type safety
- **Keep queries readable** – Use proper formatting
- **Add indexes** – For performance
- **Test with realistic data** – Ensure performance

---

# Custom SQL Checklist

| Practice | Description | Impact |
|----------|-------------|--------|
| **Variables** | Prevent SQL injection | High |
| **CTEs** | Complex queries | Medium |
| **Documentation** | Explain queries | Medium |
| **Testing** | Verify correctness | High |
| **Readability** | Format properly | Medium |
| **Indexes** | Optimize performance | High |

---

# Common Mistakes

## Mistake 1: SQL injection risk

Wrong:
```dart
// 🚫 Risk of SQL injection
final name = "Robert'; DROP TABLE users; --";
await customSelect('SELECT * FROM users WHERE name = "$name"').get();
```

Correct:
```dart
// ✅ Use variables to prevent injection
final name = "Robert'; DROP TABLE users; --";
await customSelect(
  'SELECT * FROM users WHERE name = ?',
  variables: [Variable.withString(name)],
).get();
```

## Mistake 2: Not handling results safely

Wrong:
```dart
// 🚫 Type cast errors
final id = row.data['id'];
final name = row.data['name'] as String;
```

Correct:
```dart
// ✅ Safe handling
final id = row.data['id'] as int? ?? 0;
final name = row.data['name'] as String? ?? 'Unknown';
```

## Mistake 3: Hardcoded values

Wrong:
```dart
// 🚫 Hardcoded values
await customSelect('SELECT * FROM users WHERE age > 18').get();
```

Correct:
```dart
// ✅ Use variables
final minAge = 18;
await customSelect(
  'SELECT * FROM users WHERE age > ?',
  variables: [Variable.withInt(minAge)],
).get();
```

---

# Summary

| Method | Purpose | Use Case |
|--------|---------|----------|
| `customSelect()` | Raw SELECT | Complex queries |
| `customUpdate()` | Raw UPDATE | Bulk updates |
| `customInsert()` | Raw INSERT | Bulk inserts |
| `customDelete()` | Raw DELETE | Bulk deletes |

---

# Next Steps

Now you understand custom SQL, let's dive deeper:

- [SQL Variables](link) – Using variables
- [Views](link) – Database views
- [Indexes](link) – Performance optimization

---

# Did You Know?

- **Custom SQL bypasses type safety** – Use with caution

- **Variables prevent SQL injection** – Always use them

- **CTEs (WITH) are powerful** – Complex queries made simple

- **Window functions are available** – For advanced analytics

- **Recursive CTEs** – For hierarchical data

- **JSON functions** – In modern SQLite

- **Full-text search** – With FTS5 extension

- **Custom SQL is essential** – For complex applications

---

