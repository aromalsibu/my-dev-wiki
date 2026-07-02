## Window Functions

**Advanced analytics with window functions in Drift**

---

# What is it?

**Window Functions** are a powerful SQL feature that perform calculations across a set of rows related to the current row, without collapsing them into a single result. Unlike aggregate functions (which reduce multiple rows to one), window functions return a value for every row while still allowing you to compute running totals, rankings, moving averages, and more.

> **Think of Window Functions like "looking through a window"** – you can see the rows around you (the "window"), perform calculations on them, and still see each individual row clearly.

```dart
// 👇 Window function: Rank products by revenue
final results = await customSelect('''
  SELECT 
    p.name,
    SUM(oi.total) as revenue,
    RANK() OVER (ORDER BY SUM(oi.total) DESC) as revenue_rank,
    DENSE_RANK() OVER (ORDER BY SUM(oi.total) DESC) as dense_rank,
    NTILE(4) OVER (ORDER BY SUM(oi.total) DESC) as quartile
  FROM products p
  LEFT JOIN order_items oi ON p.id = oi.product_id
  GROUP BY p.id
  ORDER BY revenue DESC
''').get();
```

> **What's happening here?**
> - **RANK()** – Ranks products by revenue
> - **OVER** – Defines the window of rows
> - **ORDER BY** – Orders rows within the window
> - **Per-row result** – Each product gets its own rank

---

# Why does it exist?

- **Ranking** – Assign ranks to rows
- **Running Totals** – Cumulative sums
- **Moving Averages** – Smooth data over time
- **Comparison** – Compare to previous/next rows
- **Percentiles** – Distributions and quartiles
- **Analytics** – Advanced business intelligence

---

# Ranking Window Functions

> **Assigning ranks to rows**

## RANK()

```dart
// 👇 Rank users by total spent
final results = await customSelect('''
  SELECT 
    u.name,
    SUM(o.total) as total_spent,
    RANK() OVER (ORDER BY SUM(o.total) DESC) as rank
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id
  ORDER BY total_spent DESC
''').get();

// Results:
// User A: $1000, Rank 1
// User B: $800, Rank 2
// User C: $800, Rank 2  (tied)
// User D: $500, Rank 4  (skips 3)
```

## DENSE_RANK()

```dart
// 👇 Dense rank (no gaps)
final results = await customSelect('''
  SELECT 
    u.name,
    SUM(o.total) as total_spent,
    DENSE_RANK() OVER (ORDER BY SUM(o.total) DESC) as dense_rank
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id
  ORDER BY total_spent DESC
''').get();

// Results:
// User A: $1000, Rank 1
// User B: $800, Rank 2  (tied)
// User C: $800, Rank 2  (tied)
// User D: $500, Rank 3  (no gap)
```

## ROW_NUMBER()

```dart
// 👇 Sequential row numbers
final results = await customSelect('''
  SELECT 
    u.name,
    o.order_number,
    o.total,
    ROW_NUMBER() OVER (ORDER BY o.order_date DESC) as row_num
  FROM users u
  INNER JOIN orders o ON u.id = o.user_id
  ORDER BY o.order_date DESC
''').get();

// Results:
// User A, Order 10, $100, Row 1
// User B, Order 9, $50, Row 2
// User C, Order 8, $200, Row 3
// ... sequential numbers
```

---

# Aggregate Window Functions

> **Running totals and moving averages**

## Running Total (SUM OVER)

```dart
// 👇 Running total of orders
final results = await customSelect('''
  SELECT 
    DATE(order_date) as date,
    total,
    SUM(total) OVER (ORDER BY order_date) as running_total,
    SUM(total) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as cumulative
  FROM orders
  WHERE status = 'completed'
  ORDER BY order_date
''').get();
```

## Moving Average

```dart
// 👇 7-day moving average
final results = await customSelect('''
  SELECT 
    DATE(order_date) as date,
    SUM(total) as daily_revenue,
    AVG(SUM(total)) OVER (
      ORDER BY DATE(order_date) 
      ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7days,
    AVG(SUM(total)) OVER (
      ORDER BY DATE(order_date) 
      ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) as moving_avg_30days
  FROM orders
  WHERE status = 'completed'
  GROUP BY DATE(order_date)
  ORDER BY date DESC
''').get();
```

---

# Comparison Window Functions

> **Comparing with previous/next rows**

## LAG() and LEAD()

```dart
// 👇 Compare sales with previous month
final results = await customSelect('''
  WITH monthly_sales AS (
    SELECT 
      STRFTIME('%Y-%m', order_date) as month,
      SUM(total) as revenue
    FROM orders
    WHERE status = 'completed'
    GROUP BY STRFTIME('%Y-%m', order_date)
  )
  SELECT 
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) as previous_month,
    LEAD(revenue) OVER (ORDER BY month) as next_month,
    (revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month) * 100 as growth_percentage
  FROM monthly_sales
  ORDER BY month DESC
''').get();
```

---

# Partitioning Window Functions

> **Grouping within windows**

## Partition By

```dart
// 👇 Rank products within each category
final results = await customSelect('''
  SELECT 
    p.name,
    p.category,
    p.price,
    RANK() OVER (
      PARTITION BY p.category 
      ORDER BY p.price DESC
    ) as rank_in_category,
    DENSE_RANK() OVER (
      PARTITION BY p.category 
      ORDER BY p.price DESC
    ) as dense_rank_in_category
  FROM products p
  WHERE p.is_active = 1
  ORDER BY p.category, p.price DESC
''').get();

// Each category has its own ranking starting from 1
```

## Multiple Partitions

```dart
// 👇 Sales by category and region
final results = await customSelect('''
  SELECT 
    category,
    region,
    SUM(total) as sales,
    ROW_NUMBER() OVER (
      PARTITION BY category 
      ORDER BY SUM(total) DESC
    ) as rank_in_category,
    ROW_NUMBER() OVER (
      PARTITION BY region 
      ORDER BY SUM(total) DESC
    ) as rank_in_region
  FROM orders o
  INNER JOIN products p ON o.product_id = p.id
  INNER JOIN stores s ON o.store_id = s.id
  GROUP BY category, region
  ORDER BY category, sales DESC
''').get();
```

---

# Real-World Example

> **Complete e-commerce window functions system**

```dart
// lib/database/window_service.dart
import 'package:drift/drift.dart';

class WindowService {
  final AppDatabase db;
  
  WindowService(this.db);

  // ==================== SALES ANALYTICS ====================
  
  // 👇 Comprehensive sales dashboard
  Future<SalesDashboard> getSalesDashboard() async {
    // Daily sales with moving averages
    final dailySales = await db.customSelect('''
      WITH daily_revenue AS (
        SELECT 
          DATE(order_date) as date,
          COUNT(*) as orders,
          SUM(total) as revenue
        FROM orders
        WHERE status = 'completed'
        GROUP BY DATE(order_date)
      )
      SELECT 
        date,
        orders,
        revenue,
        SUM(revenue) OVER (ORDER BY date) as cumulative_revenue,
        AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as avg_7day,
        AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) as avg_30day,
        ROW_NUMBER() OVER (ORDER BY revenue DESC) as rank,
        (revenue - LAG(revenue) OVER (ORDER BY date)) / LAG(revenue) OVER (ORDER BY date) * 100 as daily_growth
      FROM daily_revenue
      ORDER BY date DESC
      LIMIT 30
    ''').get();
    
    // Category performance
    final categoryPerformance = await db.customSelect('''
      SELECT 
        c.name as category,
        COUNT(DISTINCT o.id) as orders,
        SUM(oi.total) as revenue,
        RANK() OVER (ORDER BY SUM(oi.total) DESC) as revenue_rank,
        DENSE_RANK() OVER (ORDER BY SUM(oi.total) DESC) as dense_revenue_rank,
        PERCENT_RANK() OVER (ORDER BY SUM(oi.total) DESC) * 100 as revenue_percentile
      FROM categories c
      INNER JOIN products p ON c.id = p.category_id
      INNER JOIN order_items oi ON p.id = oi.product_id
      INNER JOIN orders o ON oi.order_id = o.id
      WHERE o.status = 'completed'
      GROUP BY c.id
      ORDER BY revenue DESC
    ''').get();
    
    return SalesDashboard(
      dailySales: dailySales.map((row) {
        return DailySale(
          date: DateTime.parse(row.data['date'] as String),
          orders: row.data['orders'] as int,
          revenue: row.data['revenue'] as double,
          cumulativeRevenue: row.data['cumulative_revenue'] as double,
          avg7Day: row.data['avg_7day'] as double? ?? 0.0,
          avg30Day: row.data['avg_30day'] as double? ?? 0.0,
          rank: row.data['rank'] as int,
          dailyGrowth: row.data['daily_growth'] as double? ?? 0.0,
        );
      }).toList(),
      categoryPerformance: categoryPerformance.map((row) {
        return CategoryPerformance(
          category: row.data['category'] as String,
          orders: row.data['orders'] as int,
          revenue: row.data['revenue'] as double,
          revenueRank: row.data['revenue_rank'] as int,
          denseRevenueRank: row.data['dense_revenue_rank'] as int,
          revenuePercentile: row.data['revenue_percentile'] as double,
        );
      }).toList(),
    );
  }

  // ==================== CUSTOMER RANKINGS ====================
  
  // 👇 Customer tier system with window functions
  Future<List<CustomerTier>> getCustomerTiers() async {
    final results = await db.customSelect('''
      WITH customer_metrics AS (
        SELECT 
          u.id,
          u.name,
          u.email,
          COUNT(o.id) as order_count,
          SUM(o.total) as total_spent,
          AVG(o.total) as avg_order_value,
          MAX(o.order_date) as last_order_date,
          COUNT(DISTINCT DATE(o.order_date)) as days_with_orders
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'completed'
        GROUP BY u.id
      ),
      ranked_customers AS (
        SELECT 
          *,
          RANK() OVER (ORDER BY total_spent DESC) as revenue_rank,
          DENSE_RANK() OVER (ORDER BY order_count DESC) as order_count_rank,
          NTILE(4) OVER (ORDER BY total_spent DESC) as quartile,
          PERCENT_RANK() OVER (ORDER BY total_spent DESC) as percentile
        FROM customer_metrics
        WHERE total_spent > 0
      )
      SELECT 
        id,
        name,
        email,
        order_count,
        total_spent,
        avg_order_value,
        last_order_date,
        revenue_rank,
        order_count_rank,
        quartile,
        percentile,
        CASE 
          WHEN total_spent > 10000 THEN 'VIP'
          WHEN total_spent > 5000 THEN 'Gold'
          WHEN total_spent > 1000 THEN 'Silver'
          ELSE 'Bronze'
        END as tier
      FROM ranked_customers
      ORDER BY total_spent DESC
    ''').get();
    
    return results.map((row) {
      return CustomerTier(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        email: row.data['email'] as String,
        orderCount: row.data['order_count'] as int,
        totalSpent: row.data['total_spent'] as double,
        avgOrderValue: row.data['avg_order_value'] as double,
        lastOrderDate: row.data['last_order_date'] != null
            ? DateTime.parse(row.data['last_order_date'] as String)
            : null,
        revenueRank: row.data['revenue_rank'] as int,
        orderCountRank: row.data['order_count_rank'] as int,
        quartile: row.data['quartile'] as int,
        percentile: row.data['percentile'] as double,
        tier: row.data['tier'] as String,
      );
    }).toList();
  }

  // ==================== PRODUCT ANALYTICS ====================
  
  // 👇 Product performance with window functions
  Future<List<ProductPerformance>> getProductPerformance() async {
    final results = await db.customSelect('''
      WITH product_metrics AS (
        SELECT 
          p.id,
          p.name,
          p.sku,
          p.price,
          p.stock,
          p.category,
          COALESCE(SUM(oi.quantity), 0) as total_sold,
          COALESCE(SUM(oi.total), 0) as total_revenue,
          COALESCE(COUNT(DISTINCT oi.order_id), 0) as order_count,
          COALESCE(AVG(oi.total), 0) as avg_order_value
        FROM products p
        LEFT JOIN order_items oi ON p.id = oi.product_id
        LEFT JOIN orders o ON oi.order_id = o.id AND o.status = 'completed'
        GROUP BY p.id
      ),
      ranked_products AS (
        SELECT 
          *,
          RANK() OVER (ORDER BY total_revenue DESC) as revenue_rank,
          RANK() OVER (ORDER BY total_sold DESC) as sales_rank,
          RANK() OVER (ORDER BY price DESC) as price_rank,
          NTILE(5) OVER (ORDER BY total_revenue DESC) as revenue_quintile,
          (total_revenue - LAG(total_revenue) OVER (ORDER BY total_revenue DESC)) / NULLIF(LAG(total_revenue) OVER (ORDER BY total_revenue DESC), 0) as revenue_gap,
          total_sold * 1.0 / NULLIF(SUM(total_sold) OVER (), 0) * 100 as sales_percentage
        FROM product_metrics
        WHERE total_revenue > 0
      )
      SELECT 
        id,
        name,
        sku,
        price,
        stock,
        category,
        total_sold,
        total_revenue,
        order_count,
        avg_order_value,
        revenue_rank,
        sales_rank,
        price_rank,
        revenue_quintile,
        revenue_gap,
        sales_percentage,
        CASE 
          WHEN revenue_rank <= 5 THEN 'Top Seller'
          WHEN revenue_rank <= 20 THEN 'Best Seller'
          WHEN revenue_rank <= 50 THEN 'Good Seller'
          ELSE 'Standard'
        END as performance_tier,
        CASE 
          WHEN stock < 10 AND total_sold > 0 THEN 'Low Stock'
          WHEN total_sold = 0 AND stock > 0 THEN 'No Sales'
          WHEN stock > 0 AND total_sold > 0 THEN 'Active'
          ELSE 'Inactive'
        END as status
      FROM ranked_products
      ORDER BY revenue_rank
    ''').get();
    
    return results.map((row) {
      return ProductPerformance(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        sku: row.data['sku'] as String,
        price: row.data['price'] as double,
        stock: row.data['stock'] as int,
        category: row.data['category'] as String,
        totalSold: row.data['total_sold'] as int,
        totalRevenue: row.data['total_revenue'] as double,
        orderCount: row.data['order_count'] as int,
        avgOrderValue: row.data['avg_order_value'] as double,
        revenueRank: row.data['revenue_rank'] as int,
        salesRank: row.data['sales_rank'] as int,
        priceRank: row.data['price_rank'] as int,
        revenueQuintile: row.data['revenue_quintile'] as int,
        revenueGap: row.data['revenue_gap'] as double?,
        salesPercentage: row.data['sales_percentage'] as double,
        performanceTier: row.data['performance_tier'] as String,
        status: row.data['status'] as String,
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class DailySale {
  final DateTime date;
  final int orders;
  final double revenue;
  final double cumulativeRevenue;
  final double avg7Day;
  final double avg30Day;
  final int rank;
  final double dailyGrowth;
  
  DailySale({
    required this.date,
    required this.orders,
    required this.revenue,
    required this.cumulativeRevenue,
    required this.avg7Day,
    required this.avg30Day,
    required this.rank,
    required this.dailyGrowth,
  });
}

class CategoryPerformance {
  final String category;
  final int orders;
  final double revenue;
  final int revenueRank;
  final int denseRevenueRank;
  final double revenuePercentile;
  
  CategoryPerformance({
    required this.category,
    required this.orders,
    required this.revenue,
    required this.revenueRank,
    required this.denseRevenueRank,
    required this.revenuePercentile,
  });
}

class SalesDashboard {
  final List<DailySale> dailySales;
  final List<CategoryPerformance> categoryPerformance;
  
  SalesDashboard({
    required this.dailySales,
    required this.categoryPerformance,
  });
}

class CustomerTier {
  final int id;
  final String name;
  final String email;
  final int orderCount;
  final double totalSpent;
  final double avgOrderValue;
  final DateTime? lastOrderDate;
  final int revenueRank;
  final int orderCountRank;
  final int quartile;
  final double percentile;
  final String tier;
  
  CustomerTier({
    required this.id,
    required this.name,
    required this.email,
    required this.orderCount,
    required this.totalSpent,
    required this.avgOrderValue,
    this.lastOrderDate,
    required this.revenueRank,
    required this.orderCountRank,
    required this.quartile,
    required this.percentile,
    required this.tier,
  });
}

class ProductPerformance {
  final int id;
  final String name;
  final String sku;
  final double price;
  final int stock;
  final String category;
  final int totalSold;
  final double totalRevenue;
  final int orderCount;
  final double avgOrderValue;
  final int revenueRank;
  final int salesRank;
  final int priceRank;
  final int revenueQuintile;
  final double? revenueGap;
  final double salesPercentage;
  final String performanceTier;
  final String status;
  
  ProductPerformance({
    required this.id,
    required this.name,
    required this.sku,
    required this.price,
    required this.stock,
    required this.category,
    required this.totalSold,
    required this.totalRevenue,
    required this.orderCount,
    required this.avgOrderValue,
    required this.revenueRank,
    required this.salesRank,
    required this.priceRank,
    required this.revenueQuintile,
    this.revenueGap,
    required this.salesPercentage,
    required this.performanceTier,
    required this.status,
  });
}
```

---

# Window Function Checklist

| Function | Purpose | Example |
|----------|---------|---------|
| **RANK()** | Rank with gaps | Competition rankings |
| **DENSE_RANK()** | Rank without gaps | Tier systems |
| **ROW_NUMBER()** | Sequential numbers | Row indexing |
| **SUM() OVER** | Running totals | Cumulative revenue |
| **AVG() OVER** | Moving averages | Trend analysis |
| **LAG()** | Previous row | Month-over-month |
| **LEAD()** | Next row | Future projections |
| **NTILE()** | Percentiles | Quartiles, deciles |

---

# Best Practices

- **Use PARTITION BY for groups** – Per-group calculations
- **Use ORDER BY for ordering** – Define window order
- **Use ROWS/RANGE for frames** – Define window size
- **Test window functions** – Verify results
- **Use CTEs with window functions** – Cleaner queries
- **Index window columns** – For performance
- **Limit result sets** – Avoid large windows
- **Document complex windows** – Explain logic

---

# Common Mistakes

## Mistake 1: Missing ORDER BY in window

Wrong:
```dart
// 🚫 Order matters for RANK
RANK() OVER ()
```

Correct:
```dart
// ✅ Always order for ranking
RANK() OVER (ORDER BY revenue DESC)
```

## Mistake 2: Wrong frame specification

Wrong:
```dart
// 🚫 Incorrect frame
AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND 1 FOLLOWING)
```

Correct:
```dart
// ✅ Correct frame for 7-day moving average
AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
```

## Mistake 3: Forgetting PARTITION BY

Wrong:
```dart
// 🚫 Ranks across ALL products
RANK() OVER (ORDER BY price DESC)
```

Correct:
```dart
// ✅ Ranks within each category
RANK() OVER (PARTITION BY category ORDER BY price DESC)
```

---

# Summary

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Ranking** | Assign ranks | Competition |
| **Aggregates** | Running totals | Cumulative |
| **Comparison** | LAG/LEAD | Trends |
| **Partition** | Group windows | Per-group |
| **Frames** | Window size | Moving averages |

---

# Next Steps

Now you understand window functions, let's dive deeper:

- [SQLite Functions](link) – SQLite function library
- [Migrations](link) – Schema management
- [Type Converters](link) – Custom type converters

---

# Did You Know?

- **Window functions are powerful** – For analytics

- **Window functions are efficient** – Single pass

- **Window functions are standard** – SQL standard

- **Window functions are supported** – In SQLite 3.25+

- **Window functions can be nested** - With CTEs

- **Window functions can be used** - With aggregations

- **Window functions are essential** - For business intelligence

- **Window functions are production-ready** - Used in large systems

---

