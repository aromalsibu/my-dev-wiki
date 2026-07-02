## Common Table Expressions (CTEs)

**Simplifying complex queries with Common Table Expressions**

---

# What is it?

**Common Table Expressions (CTEs)** are temporary result sets that exist only during the execution of a single SQL query. They allow you to break down complex queries into manageable parts, improve readability, and enable recursive queries. CTEs are defined using the `WITH` clause and can be referenced multiple times in the main query.

> **Think of CTEs like "scratch paper for complex math problems"** – instead of trying to solve everything in your head at once, you write down intermediate results, use them in the next step, and end up with the final answer.

```dart
// 👇 CTE: Get users with high order values
final results = await customSelect('''
  WITH user_stats AS (
    SELECT 
      user_id,
      COUNT(*) as order_count,
      SUM(total) as total_spent,
      AVG(total) as avg_order_value
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id
  )
  SELECT 
    u.name,
    u.email,
    us.order_count,
    us.total_spent,
    us.avg_order_value
  FROM users u
  INNER JOIN user_stats us ON u.id = us.user_id
  WHERE us.order_count > 5
  ORDER BY us.total_spent DESC
''').get();
```

> **What's happening here?**
> - **WITH clause** – Defines the CTE
> - **user_stats** – Name of the CTE
> - **Main query** – Uses the CTE like a table
> - **Complex logic** – Simplified and readable

---

# Why does it exist?

- **Complex Queries** – Break down complex logic
- **Readability** – Improve query maintainability
- **Reusability** – Reference CTE multiple times
- **Recursive Queries** – Handle hierarchical data
- **Performance** – Optimize complex queries
- **Organization** – Structure SQL logically

---

# Basic CTEs

> **Simple CTE patterns**

## Single CTE

```dart
// 👇 Single CTE for user order stats
final results = await customSelect('''
  WITH user_order_stats AS (
    SELECT 
      user_id,
      COUNT(*) as total_orders,
      SUM(total) as total_spent
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id
  )
  SELECT 
    u.name,
    u.email,
    uos.total_orders,
    uos.total_spent
  FROM users u
  LEFT JOIN user_order_stats uos ON u.id = uos.user_id
  ORDER BY uos.total_spent DESC
  LIMIT 10
''').get();
```

## Multiple CTEs

```dart
// 👇 Multiple CTEs in one query
final results = await customSelect('''
  WITH 
    user_stats AS (
      SELECT 
        user_id,
        COUNT(*) as order_count,
        SUM(total) as total_spent
      FROM orders
      GROUP BY user_id
    ),
    product_stats AS (
      SELECT 
        product_id,
        COUNT(*) as sold_count,
        SUM(quantity) as total_quantity
      FROM order_items
      GROUP BY product_id
    )
  SELECT 
    u.name as user_name,
    u.email,
    us.order_count,
    us.total_spent,
    p.name as top_product,
    ps.total_quantity
  FROM users u
  LEFT JOIN user_stats us ON u.id = us.user_id
  LEFT JOIN order_items oi ON u.id = oi.order_id
  LEFT JOIN products p ON oi.product_id = p.id
  LEFT JOIN product_stats ps ON p.id = ps.product_id
  ORDER BY us.total_spent DESC
  LIMIT 10
''').get();
```

## CTE with Filtering

```dart
// 👇 CTE with WHERE clause
final results = await customSelect('''
  WITH active_users AS (
    SELECT id, name, email
    FROM users
    WHERE is_active = 1 AND is_verified = 1
  ),
  recent_orders AS (
    SELECT user_id, total, order_date
    FROM orders
    WHERE order_date > datetime('now', '-30 days')
    AND status = 'completed'
  )
  SELECT 
    au.name,
    au.email,
    ro.total,
    ro.order_date
  FROM active_users au
  INNER JOIN recent_orders ro ON au.id = ro.user_id
  ORDER BY ro.order_date DESC
''').get();
```

---

# Advanced CTEs

> **Complex CTE patterns**

## CTE with Aggregation

```dart
// 👇 CTE with multiple aggregations
final results = await customSelect('''
  WITH monthly_sales AS (
    SELECT 
      STRFTIME('%Y-%m', order_date) as month,
      COUNT(*) as order_count,
      SUM(total) as revenue,
      AVG(total) as avg_order_value,
      COUNT(DISTINCT user_id) as unique_customers
    FROM orders
    WHERE status = 'completed'
    GROUP BY STRFTIME('%Y-%m', order_date)
  )
  SELECT 
    month,
    order_count,
    revenue,
    avg_order_value,
    unique_customers,
    LAG(revenue) OVER (ORDER BY month) as previous_month_revenue,
    (revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month) * 100 as growth_percentage
  FROM monthly_sales
  ORDER BY month DESC
''').get();
```

## CTE with Joins

```dart
// 👇 CTE with multiple joins
final results = await customSelect('''
  WITH customer_lifetime AS (
    SELECT 
      u.id as user_id,
      u.name as user_name,
      u.email,
      COUNT(o.id) as order_count,
      SUM(o.total) as lifetime_value,
      MAX(o.order_date) as last_order_date
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE o.status = 'completed'
    GROUP BY u.id
  ),
  product_preferences AS (
    SELECT 
      oi.user_id,
      p.id as product_id,
      p.name as product_name,
      COUNT(oi.id) as purchase_count,
      SUM(oi.quantity) as total_quantity
    FROM order_items oi
    INNER JOIN products p ON oi.product_id = p.id
    INNER JOIN orders o ON oi.order_id = o.id
    WHERE o.status = 'completed'
    GROUP BY oi.user_id, p.id
  )
  SELECT 
    cl.user_id,
    cl.user_name,
    cl.lifetime_value,
    cl.order_count,
    cl.last_order_date,
    pp.product_name as favorite_product,
    pp.purchase_count
  FROM customer_lifetime cl
  LEFT JOIN product_preferences pp ON cl.user_id = pp.user_id
  ORDER BY cl.lifetime_value DESC
  LIMIT 50
''').get();
```

---

# Recursive CTEs

> **Handling hierarchical data**

## Basic Recursive CTE

```dart
// 👇 Employee hierarchy
final results = await customSelect('''
  WITH RECURSIVE employee_tree AS (
    -- Base case: CEO (no manager)
    SELECT 
      id,
      name,
      manager_id,
      0 as level,
      name as path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: employees with managers
    SELECT 
      e.id,
      e.name,
      e.manager_id,
      et.level + 1,
      et.path || ' > ' || e.name
    FROM employees e
    INNER JOIN employee_tree et ON e.manager_id = et.id
  )
  SELECT 
    id,
    name,
    level,
    path,
    CASE 
      WHEN level = 0 THEN 'CEO'
      WHEN level = 1 THEN 'Executive'
      WHEN level = 2 THEN 'Manager'
      ELSE 'Employee'
    END as role
  FROM employee_tree
  ORDER BY path
''').get();
```

## Category Hierarchy

```dart
// 👇 Product category tree
final results = await customSelect('''
  WITH RECURSIVE category_tree AS (
    -- Base case: root categories
    SELECT 
      id,
      name,
      parent_id,
      0 as level,
      name as path
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Recursive case: subcategories
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
    level,
    path,
    SUBSTR('..........', 1, level * 2) || name as indented_name
  FROM category_tree
  ORDER BY path
''').get();
```

## Comment Threads

```dart
// 👇 Comment thread with nesting
final results = await customSelect('''
  WITH RECURSIVE comment_thread AS (
    -- Base case: top-level comments
    SELECT 
      id,
      content,
      user_id,
      parent_id,
      0 as level,
      id as thread_id,
      datetime(created_at) as created_at
    FROM comments
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Recursive case: replies
    SELECT 
      c.id,
      c.content,
      c.user_id,
      c.parent_id,
      ct.level + 1,
      ct.thread_id,
      datetime(c.created_at) as created_at
    FROM comments c
    INNER JOIN comment_thread ct ON c.parent_id = ct.id
  )
  SELECT 
    id,
    content,
    user_id,
    level,
    SUBSTR('..........', 1, level * 2) || content as indented_content,
    CASE 
      WHEN level = 0 THEN 'Top Level'
      WHEN level = 1 THEN 'Reply'
      ELSE 'Deep Reply'
    END as comment_type
  FROM comment_thread
  ORDER BY thread_id, level, created_at
''').get();
```

---

# Real-World Example

> **Complete e-commerce CTE system**

```dart
// lib/database/cte_service.dart
import 'package:drift/drift.dart';

class CteService {
  final AppDatabase db;
  
  CteService(this.db);

  // ==================== CUSTOMER ANALYSIS ====================
  
  // 👇 Customer segmentation with CTEs
  Future<List<CustomerSegment>> getCustomerSegments() async {
    final results = await db.customSelect('''
      WITH 
        -- Calculate customer metrics
        customer_metrics AS (
          SELECT 
            u.id as user_id,
            u.name as user_name,
            u.email,
            COUNT(o.id) as order_count,
            SUM(o.total) as total_spent,
            AVG(o.total) as avg_order_value,
            MAX(o.order_date) as last_order_date,
            COUNT(DISTINCT p.id) as unique_products,
            JULIANDAY('now') - JULIANDAY(MAX(o.order_date)) as days_since_last_order
          FROM users u
          LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'completed'
          LEFT JOIN order_items oi ON o.id = oi.order_id
          LEFT JOIN products p ON oi.product_id = p.id
          GROUP BY u.id
        ),
        -- Add segment labels
        customer_segments AS (
          SELECT 
            *,
            CASE 
              WHEN total_spent > 10000 THEN 'VIP'
              WHEN total_spent > 5000 THEN 'Premium'
              WHEN total_spent > 1000 THEN 'Regular'
              WHEN order_count > 0 THEN 'Occasional'
              ELSE 'Inactive'
            END as segment,
            CASE 
              WHEN days_since_last_order < 30 THEN 'Active'
              WHEN days_since_last_order < 90 THEN 'At Risk'
              ELSE 'Inactive'
            END as engagement_status
          FROM customer_metrics
        )
      SELECT 
        user_id,
        user_name,
        email,
        order_count,
        total_spent,
        avg_order_value,
        unique_products,
        segment,
        engagement_status,
        days_since_last_order
      FROM customer_segments
      WHERE total_spent > 0
      ORDER BY total_spent DESC
    ''').get();
    
    return results.map((row) {
      return CustomerSegment(
        userId: row.data['user_id'] as int,
        userName: row.data['user_name'] as String,
        email: row.data['email'] as String,
        orderCount: row.data['order_count'] as int,
        totalSpent: row.data['total_spent'] as double,
        avgOrderValue: row.data['avg_order_value'] as double,
        uniqueProducts: row.data['unique_products'] as int,
        segment: row.data['segment'] as String,
        engagementStatus: row.data['engagement_status'] as String,
        daysSinceLastOrder: row.data['days_since_last_order'] as double?,
      );
    }).toList();
  }

  // ==================== SALES FORECASTING ====================
  
  // 👇 Sales forecast with CTEs
  Future<SalesForecast> getSalesForecast() async {
    final results = await db.customSelect('''
      WITH 
        -- Monthly sales history
        monthly_sales AS (
          SELECT 
            STRFTIME('%Y-%m', order_date) as month,
            SUM(total) as revenue,
            COUNT(*) as orders
          FROM orders
          WHERE status = 'completed'
          AND order_date >= datetime('now', '-12 months')
          GROUP BY STRFTIME('%Y-%m', order_date)
          ORDER BY month
        ),
        -- Moving averages
        moving_averages AS (
          SELECT 
            month,
            revenue,
            orders,
            AVG(revenue) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as ma_3mo,
            AVG(revenue) OVER (ORDER BY month ROWS BETWEEN 5 PRECEDING AND CURRENT ROW) as ma_6mo,
            AVG(revenue) OVER (ORDER BY month ROWS BETWEEN 11 PRECEDING AND CURRENT ROW) as ma_12mo
          FROM monthly_sales
        ),
        -- Growth rates
        growth_rates AS (
          SELECT 
            month,
            revenue,
            ma_3mo,
            ma_6mo,
            ma_12mo,
            (revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month) * 100 as growth_rate
          FROM moving_averages
        ),
        -- Forecast
        forecast AS (
          SELECT 
            'Next Month' as period,
            AVG(ma_3mo) * 1.1 as forecast_revenue,
            'Based on 3-month average with 10% growth' as note
          FROM growth_rates
          WHERE month >= datetime('now', '-3 months')
          UNION ALL
          SELECT 
            'Next Quarter' as period,
            AVG(ma_6mo) * 1.05 as forecast_revenue,
            'Based on 6-month average with 5% growth' as note
          FROM growth_rates
          WHERE month >= datetime('now', '-6 months')
          UNION ALL
          SELECT 
            'Next Year' as period,
            AVG(ma_12mo) * 1.02 as forecast_revenue,
            'Based on 12-month average with 2% growth' as note
          FROM growth_rates
        )
      SELECT * FROM forecast
    ''').get();
    
    return SalesForecast(
      nextMonthForecast: results[0].data['forecast_revenue'] as double,
      nextMonthNote: results[0].data['note'] as String,
      nextQuarterForecast: results[1].data['forecast_revenue'] as double,
      nextQuarterNote: results[1].data['note'] as String,
      nextYearForecast: results[2].data['forecast_revenue'] as double,
      nextYearNote: results[2].data['note'] as String,
    );
  }

  // ==================== PRODUCT RECOMMENDATIONS ====================
  
  // 👇 Product recommendation with CTEs
  Future<List<ProductRecommendation>> getProductRecommendations(
    int userId,
  ) async {
    final results = await db.customSelect(
      '''
      WITH 
        -- Products bought by the user
        user_products AS (
          SELECT DISTINCT product_id
          FROM order_items oi
          INNER JOIN orders o ON oi.order_id = o.id
          WHERE o.user_id = ? AND o.status = 'completed'
        ),
        -- Users who bought the same products
        similar_users AS (
          SELECT DISTINCT o.user_id
          FROM order_items oi
          INNER JOIN orders o ON oi.order_id = o.id
          WHERE oi.product_id IN (SELECT product_id FROM user_products)
          AND o.user_id != ?
          AND o.status = 'completed'
        ),
        -- Products bought by similar users
        recommended_products AS (
          SELECT 
            p.id as product_id,
            p.name as product_name,
            p.price,
            COUNT(DISTINCT o.user_id) as user_count,
            SUM(oi.quantity) as total_quantity
          FROM products p
          INNER JOIN order_items oi ON p.id = oi.product_id
          INNER JOIN orders o ON oi.order_id = o.id
          WHERE o.user_id IN (SELECT user_id FROM similar_users)
          AND p.id NOT IN (SELECT product_id FROM user_products)
          AND o.status = 'completed'
          GROUP BY p.id
        )
      SELECT 
        product_id,
        product_name,
        price,
        user_count,
        total_quantity,
        (user_count * 1.0 / (SELECT COUNT(*) FROM similar_users)) * total_quantity as recommendation_score
      FROM recommended_products
      ORDER BY recommendation_score DESC
      LIMIT 10
      ''',
      variables: [
        Variable.withInt(userId),
        Variable.withInt(userId),
      ],
    ).get();
    
    return results.map((row) {
      return ProductRecommendation(
        productId: row.data['product_id'] as int,
        productName: row.data['product_name'] as String,
        price: row.data['price'] as double,
        userCount: row.data['user_count'] as int,
        totalQuantity: row.data['total_quantity'] as int,
        recommendationScore: row.data['recommendation_score'] as double,
      );
    }).toList();
  }

  // ==================== INVENTORY ANALYSIS ====================
  
  // 👇 Inventory turnover analysis
  Future<List<InventoryTurnover>> getInventoryTurnover() async {
    final results = await db.customSelect('''
      WITH 
        -- Product sales
        product_sales AS (
          SELECT 
            p.id as product_id,
            p.name as product_name,
            p.sku,
            p.stock,
            COALESCE(SUM(oi.quantity), 0) as total_sold,
            COALESCE(COUNT(DISTINCT oi.order_id), 0) as order_count
          FROM products p
          LEFT JOIN order_items oi ON p.id = oi.product_id
          LEFT JOIN orders o ON oi.order_id = o.id
          WHERE o.status = 'completed' OR o.status IS NULL
          GROUP BY p.id
        ),
        -- Inventory metrics
        inventory_metrics AS (
          SELECT 
            product_id,
            product_name,
            sku,
            stock,
            total_sold,
            order_count,
            CASE 
              WHEN stock > 0 AND total_sold > 0 THEN total_sold * 1.0 / stock
              ELSE 0
            END as turnover_rate,
            CASE 
              WHEN total_sold > 0 THEN stock * 1.0 / total_sold
              ELSE 0
            END as months_of_inventory,
            CASE 
              WHEN stock < 10 AND total_sold > 0 THEN 'Low Stock'
              WHEN stock > 100 AND total_sold = 0 THEN 'Slow Moving'
              WHEN stock > 0 AND total_sold > 0 THEN 'Healthy'
              ELSE 'Inactive'
            END as inventory_status
          FROM product_sales
        )
      SELECT 
        product_id,
        product_name,
        sku,
        stock,
        total_sold,
        turnover_rate,
        months_of_inventory,
        inventory_status
      FROM inventory_metrics
      WHERE stock > 0
      ORDER BY turnover_rate DESC
    ''').get();
    
    return results.map((row) {
      return InventoryTurnover(
        productId: row.data['product_id'] as int,
        productName: row.data['product_name'] as String,
        sku: row.data['sku'] as String,
        stock: row.data['stock'] as int,
        totalSold: row.data['total_sold'] as int,
        turnoverRate: row.data['turnover_rate'] as double,
        monthsOfInventory: row.data['months_of_inventory'] as double,
        inventoryStatus: row.data['inventory_status'] as String,
      );
    }).toList();
  }

  // ==================== COHORT RETENTION ====================
  
  // 👇 Customer cohort retention analysis
  Future<List<CohortRetention>> getCohortRetention() async {
    final results = await db.customSelect('''
      WITH 
        -- Define cohorts by signup month
        cohorts AS (
          SELECT 
            id as user_id,
            STRFTIME('%Y-%m', created_at) as signup_month
          FROM users
        ),
        -- User orders with month
        user_orders AS (
          SELECT 
            o.user_id,
            c.signup_month,
            STRFTIME('%Y-%m', o.order_date) as order_month,
            o.total
          FROM orders o
          INNER JOIN cohorts c ON o.user_id = c.user_id
          WHERE o.status = 'completed'
        ),
        -- Calculate retention
        cohort_retention AS (
          SELECT 
            signup_month,
            order_month,
            COUNT(DISTINCT user_id) as active_users,
            SUM(total) as revenue
          FROM user_orders
          WHERE order_month >= signup_month
          GROUP BY signup_month, order_month
        ),
        -- Base users per cohort
        cohort_sizes AS (
          SELECT 
            signup_month,
            COUNT(DISTINCT user_id) as total_users
          FROM cohorts
          GROUP BY signup_month
        )
      SELECT 
        cr.signup_month,
        cr.order_month,
        cr.active_users,
        cr.revenue,
        cs.total_users as cohort_size,
        (cr.active_users * 100.0 / cs.total_users) as retention_percentage,
        STRFTIME('%Y-%m', datetime(cr.order_month || '-01')) as display_month
      FROM cohort_retention cr
      INNER JOIN cohort_sizes cs ON cr.signup_month = cs.signup_month
      ORDER BY cr.signup_month, cr.order_month
    ''').get();
    
    return results.map((row) {
      return CohortRetention(
        signupMonth: row.data['signup_month'] as String,
        orderMonth: row.data['order_month'] as String,
        activeUsers: row.data['active_users'] as int,
        revenue: row.data['revenue'] as double,
        cohortSize: row.data['cohort_size'] as int,
        retentionPercentage: row.data['retention_percentage'] as double,
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class CustomerSegment {
  final int userId;
  final String userName;
  final String email;
  final int orderCount;
  final double totalSpent;
  final double avgOrderValue;
  final int uniqueProducts;
  final String segment;
  final String engagementStatus;
  final double? daysSinceLastOrder;
  
  CustomerSegment({
    required this.userId,
    required this.userName,
    required this.email,
    required this.orderCount,
    required this.totalSpent,
    required this.avgOrderValue,
    required this.uniqueProducts,
    required this.segment,
    required this.engagementStatus,
    this.daysSinceLastOrder,
  });
}

class SalesForecast {
  final double nextMonthForecast;
  final String nextMonthNote;
  final double nextQuarterForecast;
  final String nextQuarterNote;
  final double nextYearForecast;
  final String nextYearNote;
  
  SalesForecast({
    required this.nextMonthForecast,
    required this.nextMonthNote,
    required this.nextQuarterForecast,
    required this.nextQuarterNote,
    required this.nextYearForecast,
    required this.nextYearNote,
  });
}

class ProductRecommendation {
  final int productId;
  final String productName;
  final double price;
  final int userCount;
  final int totalQuantity;
  final double recommendationScore;
  
  ProductRecommendation({
    required this.productId,
    required this.productName,
    required this.price,
    required this.userCount,
    required this.totalQuantity,
    required this.recommendationScore,
  });
}

class InventoryTurnover {
  final int productId;
  final String productName;
  final String sku;
  final int stock;
  final int totalSold;
  final double turnoverRate;
  final double monthsOfInventory;
  final String inventoryStatus;
  
  InventoryTurnover({
    required this.productId,
    required this.productName,
    required this.sku,
    required this.stock,
    required this.totalSold,
    required this.turnoverRate,
    required this.monthsOfInventory,
    required this.inventoryStatus,
  });
}

class CohortRetention {
  final String signupMonth;
  final String orderMonth;
  final int activeUsers;
  final double revenue;
  final int cohortSize;
  final double retentionPercentage;
  
  CohortRetention({
    required this.signupMonth,
    required this.orderMonth,
    required this.activeUsers,
    required this.revenue,
    required this.cohortSize,
    required this.retentionPercentage,
  });
}
```

---

# CTE Best Practices

- **Use descriptive names** – Meaningful CTE names
- **Break down complex queries** – One CTE per logical step
- **Add comments** – Explain each CTE's purpose
- **Avoid too many CTEs** – Balance readability and performance
- **Test CTEs individually** – Verify each part works
- **Use for recursive data** – Perfect for hierarchies
- **Use for complex analytics** – Simplify advanced queries
- **Index CTE columns** – For performance

---

# CTE Checklist

| Practice | Description | Impact |
|----------|-------------|--------|
| **Descriptive Names** | Clear purpose | Medium |
| **Logical Breakdown** | One step per CTE | High |
| **Comments** | Explain purpose | Medium |
| **Testing** | Verify each CTE | High |
| **Performance** | Index CTE columns | High |
| **Recursion** | Use for hierarchies | High |

---

# Common Mistakes

## Mistake 1: Recursive CTE without termination

Wrong:
```dart
// 🚫 Infinite recursion
WITH RECURSIVE infinite AS (
  SELECT 1 as n
  UNION ALL
  SELECT n + 1 FROM infinite
)
SELECT * FROM infinite
```

Correct:
```dart
// ✅ Limited recursion
WITH RECURSIVE limited AS (
  SELECT 1 as n
  UNION ALL
  SELECT n + 1 FROM limited
  WHERE n < 10
)
SELECT * FROM limited
```

## Mistake 2: CTE too complex

Wrong:
```dart
// 🚫 Single CTE doing everything
WITH complex AS (
  -- 50+ lines of complex logic
)
```

Correct:
```dart
// ✅ Multiple focused CTEs
WITH
  step1 AS (...),
  step2 AS (...),
  step3 AS (...)
```

## Mistake 3: Not using CTEs for performance

Wrong:
```dart
// 🚫 Multiple subqueries repeated
SELECT * FROM table1
WHERE id IN (SELECT id FROM table2 WHERE ...)
AND EXISTS (SELECT id FROM table2 WHERE ...)
```

Correct:
```dart
// ✅ CTE once, reference multiple times
WITH filtered_table2 AS (
  SELECT id FROM table2 WHERE ...
)
SELECT * FROM table1
WHERE id IN (SELECT id FROM filtered_table2)
AND EXISTS (SELECT id FROM filtered_table2)
```

---

# Summary

| Feature | Description | Use Case |
|---------|-------------|----------|
| **WITH** | Define CTE | Complex queries |
| **Recursive** | Self-referencing | Hierarchies |
| **Multiple CTEs** | Logical steps | Complex analytics |
| **Reusable** | Reference multiple times | Performance |

---

# Next Steps

Now you understand CTEs, let's dive deeper:

- [Window Functions](link) – Advanced analytics
- [SQLite Functions](link) – SQLite function library
- [Migrations](link) – Schema management

---

# Did You Know?

- **CTEs are temporary** – Exist only during query execution

- **Recursive CTEs handle hierarchies** – Employee trees, categories

- **CTEs improve readability** – Break down complex queries

- **CTEs can be referenced multiple times** – In the main query

- **CTEs are optimized by SQLite** - Query planner

- **CTEs are part of SQL standard** - Supported by all databases

- **CTEs can be nested** - CTEs within CTEs

- **CTEs are powerful for analytics** - Complex calculations

---

