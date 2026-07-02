## Distinct

**Getting unique values from your Drift queries**

---

# What is it?

**Distinct** is a query modifier that removes duplicate rows from your result set, returning only unique values. This is particularly useful when you want to find all unique values in a column, or get unique combinations of multiple columns.

> **Think of Distinct like "removing duplicates from a list"** – if you have a list of names with repeats, you filter out the duplicates to see each name only once.

```dart
// 👇 Get all unique user statuses
final uniqueStatuses = await select(users)
  .map((u) => u.status)
  .distinct()
  .get();

// 👇 Get all unique age values
final uniqueAges = await select(users)
  .map((u) => u.age)
  .distinct()
  .get();

// 👇 Get unique combinations of age and status
final uniqueCombinations = await select(users)
  .map((u) => [u.age, u.status])
  .distinct()
  .get();
```

> **What's happening here?**
> - **`.distinct()`** – Removes duplicate rows
> - **`.map()`** – Select specific columns
> - **Unique values** – Only unique values are returned
> - **Performance** – Database-level deduplication

---

# Why does it exist?

- **Data Analysis** – Find unique values in your data
- **Reporting** – Generate reports with unique values
- **User Experience** – Show unique options in filters
- **Data Quality** – Identify duplicate data
- **Performance** – Reduce data transfer
- **Aggregation** – Count unique values

---

# Basic Distinct

> **Simple distinct operations**

## Distinct on Single Column

```dart
// 👇 Get all unique statuses
final uniqueStatuses = await select(users)
  .map((u) => u.status)
  .distinct()
  .get();

// 👇 Get all unique email domains
final uniqueDomains = await select(users)
  .map((u) => u.email.split('@').last)
  .distinct()
  .get();

// 👇 Get all unique ages
final uniqueAges = await select(users)
  .map((u) => u.age)
  .distinct()
  .get();
```

## Distinct with WHERE Clause

```dart
// 👇 Get unique statuses of active users
final activeStatuses = await (select(users)
  ..where((u) => u.isActive.equals(true)))
  .map((u) => u.status)
  .distinct()
  .get();

// 👇 Get unique ages of verified users
final verifiedAges = await (select(users)
  ..where((u) => u.isVerified.equals(true)))
  .map((u) => u.age)
  .distinct()
  .get();
```

---

# Distinct on Multiple Columns

> **Getting unique combinations**

## Multiple Columns

```dart
// 👇 Get unique combinations of age and status
final uniqueCombinations = await select(users)
  .map((u) => [u.age, u.status])
  .distinct()
  .get();

// 👇 Get unique combinations of category and status
final uniqueCategoryStatus = await select(products)
  .map((p) => [p.category, p.status])
  .distinct()
  .get();
```

## Distinct with Map

```dart
// 👇 Convert to custom objects
final uniqueUsers = await select(users)
  .map((u) => [u.age, u.status])
  .distinct()
  .get();

// Process results
for (final combo in uniqueUsers) {
  final age = combo[0] as int?;
  final status = combo[1] as String;
  print('Age: $age, Status: $status');
}
```

---

# Count Distinct

> **Counting unique values**

## Count with Distinct

```dart
// 👇 Count unique statuses
final uniqueStatusCount = await select(users)
  .map((u) => u.status)
  .distinct()
  .count();

// 👇 Count unique ages
final uniqueAgeCount = await select(users)
  .map((u) => u.age)
  .distinct()
  .count();

// 👇 Count unique email domains
final domainCount = await select(users)
  .map((u) => u.email.split('@').last)
  .distinct()
  .count();
```

---

# Real-World Example

> **Complete e-commerce distinct system**

```dart
// lib/database/distinct_service.dart
import 'package:drift/drift.dart';

class DistinctService {
  final AppDatabase db;
  
  DistinctService(this.db);

  // ==================== USER DISTINCTS ====================
  
  // 👇 Get all unique user statuses
  Future<List<String>> getUniqueUserStatuses() async {
    return await select(users)
      .map((u) => u.status)
      .distinct()
      .get();
  }
  
  // 👇 Get all unique user ages (non-null)
  Future<List<int>> getUniqueUserAges() async {
    final results = await (select(users)
      ..where((u) => u.age.isNotNull()))
      .map((u) => u.age)
      .distinct()
      .get();
    
    return results.whereType<int>().toList();
  }
  
  // 👇 Get unique age groups
  Future<List<String>> getUniqueAgeGroups() async {
    final results = await select(users)
      .map((u) => u.age)
      .distinct()
      .get();
    
    final ageGroups = <String>{};
    for (final age in results) {
      if (age == null) continue;
      if (age < 18) ageGroups.add('Under 18');
      else if (age < 30) ageGroups.add('18-29');
      else if (age < 45) ageGroups.add('30-44');
      else if (age < 60) ageGroups.add('45-59');
      else ageGroups.add('60+');
    }
    return ageGroups.toList()..sort();
  }
  
  // 👇 Get unique statuses for active users
  Future<List<String>> getUniqueActiveUserStatuses() async {
    return await (select(users)
      ..where((u) => u.isActive.equals(true)))
      .map((u) => u.status)
      .distinct()
      .get();
  }

  // ==================== PRODUCT DISTINCTS ====================
  
  // 👇 Get all unique categories
  Future<List<String>> getUniqueCategories() async {
    return await select(products)
      .map((p) => p.category)
      .distinct()
      .get();
  }
  
  // 👇 Get all unique product statuses
  Future<List<String>> getUniqueProductStatuses() async {
    return await select(products)
      .map((p) => p.status)
      .distinct()
      .get();
  }
  
  // 👇 Get unique categories with product count
  Future<List<CategoryStats>> getCategoryStats() async {
    final results = await db.customSelect('''
      SELECT 
        category,
        COUNT(*) as count,
        AVG(price) as avg_price,
        MIN(price) as min_price,
        MAX(price) as max_price
      FROM products
      GROUP BY category
      ORDER BY count DESC
    ''').get();
    
    return results.map((row) {
      return CategoryStats(
        category: row.data['category'] as String,
        count: row.data['count'] as int,
        avgPrice: row.data['avg_price'] as double? ?? 0.0,
        minPrice: row.data['min_price'] as double? ?? 0.0,
        maxPrice: row.data['max_price'] as double? ?? 0.0,
      );
    }).toList();
  }
  
  // 👇 Get unique category and status combinations
  Future<List<CategoryStatusCombo>> getUniqueCategoryStatusCombos() async {
    final results = await select(products)
      .map((p) => [p.category, p.status])
      .distinct()
      .get();
    
    return results.map((combo) {
      return CategoryStatusCombo(
        category: combo[0] as String,
        status: combo[1] as String,
      );
    }).toList();
  }

  // ==================== ORDER DISTINCTS ====================
  
  // 👇 Get all unique order statuses
  Future<List<String>> getUniqueOrderStatuses() async {
    return await select(orders)
      .map((o) => o.status)
      .distinct()
      .get();
  }
  
  // 👇 Get all unique payment methods
  Future<List<String>> getUniquePaymentMethods() async {
    return await (select(orders)
      ..where((o) => o.paymentMethod.isNotNull()))
      .map((o) => o.paymentMethod)
      .distinct()
      .get();
  }
  
  // 👇 Get unique order years
  Future<List<int>> getUniqueOrderYears() async {
    final results = await select(orders)
      .map((o) => o.orderDate.year)
      .distinct()
      .get();
    return results..sort();
  }

  // ==================== ANALYSIS DISTINCTS ====================
  
  // 👇 Get user distribution by age
  Future<List<AgeDistribution>> getAgeDistribution() async {
    final results = await db.customSelect('''
      SELECT 
        CASE 
          WHEN age IS NULL THEN 'Unknown'
          WHEN age < 18 THEN 'Under 18'
          WHEN age < 30 THEN '18-29'
          WHEN age < 45 THEN '30-44'
          WHEN age < 60 THEN '45-59'
          ELSE '60+'
        END as age_group,
        COUNT(*) as count,
        (COUNT(*) * 100.0 / (SELECT COUNT(*) FROM users)) as percentage
      FROM users
      GROUP BY age_group
      ORDER BY age_group
    ''').get();
    
    return results.map((row) {
      return AgeDistribution(
        group: row.data['age_group'] as String,
        count: row.data['count'] as int,
        percentage: row.data['percentage'] as double,
      );
    }).toList();
  }
  
  // 👇 Get order summary by status
  Future<List<OrderSummary>> getOrderSummaryByStatus() async {
    final results = await db.customSelect('''
      SELECT 
        status,
        COUNT(*) as count,
        SUM(total) as total_revenue,
        AVG(total) as avg_order_value,
        MIN(total) as min_order_value,
        MAX(total) as max_order_value
      FROM orders
      GROUP BY status
      ORDER BY status
    ''').get();
    
    return results.map((row) {
      return OrderSummary(
        status: row.data['status'] as String,
        count: row.data['count'] as int,
        totalRevenue: row.data['total_revenue'] as double? ?? 0.0,
        avgOrderValue: row.data['avg_order_value'] as double? ?? 0.0,
        minOrderValue: row.data['min_order_value'] as double? ?? 0.0,
        maxOrderValue: row.data['max_order_value'] as double? ?? 0.0,
      );
    }).toList();
  }
  
  // 👇 Get product count by category
  Future<List<CategorySummary>> getProductCategorySummary() async {
    final results = await db.customSelect('''
      SELECT 
        category,
        COUNT(*) as total,
        SUM(CASE WHEN is_active = 1 THEN 1 ELSE 0 END) as active,
        SUM(CASE WHEN stock > 0 THEN 1 ELSE 0 END) as in_stock,
        AVG(price) as avg_price
      FROM products
      GROUP BY category
      ORDER BY total DESC
    ''').get();
    
    return results.map((row) {
      return CategorySummary(
        category: row.data['category'] as String,
        total: row.data['total'] as int,
        active: row.data['active'] as int,
        inStock: row.data['in_stock'] as int,
        avgPrice: row.data['avg_price'] as double? ?? 0.0,
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class CategoryStats {
  final String category;
  final int count;
  final double avgPrice;
  final double minPrice;
  final double maxPrice;
  
  CategoryStats({
    required this.category,
    required this.count,
    required this.avgPrice,
    required this.minPrice,
    required this.maxPrice,
  });
}

class CategoryStatusCombo {
  final String category;
  final String status;
  
  CategoryStatusCombo({
    required this.category,
    required this.status,
  });
}

class AgeDistribution {
  final String group;
  final int count;
  final double percentage;
  
  AgeDistribution({
    required this.group,
    required this.count,
    required this.percentage,
  });
}

class OrderSummary {
  final String status;
  final int count;
  final double totalRevenue;
  final double avgOrderValue;
  final double minOrderValue;
  final double maxOrderValue;
  
  OrderSummary({
    required this.status,
    required this.count,
    required this.totalRevenue,
    required this.avgOrderValue,
    required this.minOrderValue,
    required this.maxOrderValue,
  });
}

class CategorySummary {
  final String category;
  final int total;
  final int active;
  final int inStock;
  final double avgPrice;
  
  CategorySummary({
    required this.category,
    required this.total,
    required this.active,
    required this.inStock,
    required this.avgPrice,
  });
}
```

```dart
// lib/ui/pages/analytics_page.dart
class AnalyticsPage extends StatefulWidget {
  final DistinctService distinctService;
  
  const AnalyticsPage({required this.distinctService});
  
  @override
  _AnalyticsPageState createState() => _AnalyticsPageState();
}

class _AnalyticsPageState extends State<AnalyticsPage> {
  List<AgeDistribution> _ageDistribution = [];
  List<OrderSummary> _orderSummary = [];
  List<CategoryStats> _categoryStats = [];
  bool _isLoading = true;
  
  @override
  void initState() {
    super.initState();
    _loadAnalytics();
  }
  
  Future<void> _loadAnalytics() async {
    setState(() => _isLoading = true);
    
    try {
      final ageDist = await widget.distinctService.getAgeDistribution();
      final orderSummary = await widget.distinctService.getOrderSummaryByStatus();
      final categoryStats = await widget.distinctService.getCategoryStats();
      
      setState(() {
        _ageDistribution = ageDist;
        _orderSummary = orderSummary;
        _categoryStats = categoryStats;
        _isLoading = false;
      });
    } catch (e) {
      setState(() => _isLoading = false);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Analytics')),
      body: _isLoading
          ? Center(child: CircularProgressIndicator())
          : ListView(
              padding: EdgeInsets.all(16),
              children: [
                _buildSection(
                  'Age Distribution',
                  _ageDistribution.map((dist) => ListTile(
                    title: Text(dist.group),
                    trailing: Text('${dist.count} (${dist.percentage.toStringAsFixed(1)}%)'),
                  )).toList(),
                ),
                SizedBox(height: 16),
                _buildSection(
                  'Order Summary by Status',
                  _orderSummary.map((summary) => ListTile(
                    title: Text(summary.status),
                    subtitle: Text('${summary.count} orders'),
                    trailing: Text('\$${summary.totalRevenue.toStringAsFixed(2)}'),
                  )).toList(),
                ),
                SizedBox(height: 16),
                _buildSection(
                  'Category Stats',
                  _categoryStats.map((stat) => ListTile(
                    title: Text(stat.category),
                    subtitle: Text('${stat.count} products, Avg: \$${stat.avgPrice.toStringAsFixed(2)}'),
                    trailing: Text('Min: \$${stat.minPrice}, Max: \$${stat.maxPrice}'),
                  )).toList(),
                ),
              ],
            ),
    );
  }
  
  Widget _buildSection(String title, List<Widget> children) {
    return Card(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Padding(
            padding: EdgeInsets.all(16),
            child: Text(
              title,
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          ...children,
        ],
      ),
    );
  }
}
```

---

# Best Practices

- **Use distinct on indexed columns** – For better performance
- **Use `map()` to select only needed columns** – Reduce data
- **Use distinct for reporting** – Get unique values
- **Combine with WHERE** – For filtered distinct values
- **Use count with distinct** – For unique value counts
- **Consider performance** – Distinct is a heavy operation

---

# Common Mistakes

## Mistake 1: Using distinct on all columns

Wrong:
```dart
// 🚫 Distinct on all columns (inefficient)
final users = await select(users).distinct().get();
```

Correct:
```dart
// ✅ Distinct on specific columns
final statuses = await select(users)
  .map((u) => u.status)
  .distinct()
  .get();
```

## Mistake 2: Not handling null values

Wrong:
```dart
// 🚫 Null values included
final ages = await select(users)
  .map((u) => u.age)
  .distinct()
  .get(); // Includes null
```

Correct:
```dart
// ✅ Handle null values
final ages = await (select(users)
  ..where((u) => u.age.isNotNull()))
  .map((u) => u.age)
  .distinct()
  .get();
```

## Mistake 3: Using distinct without map

Wrong:
```dart
// 🚫 Not supported (distinct must be after map)
final values = await select(users).distinct().get();
```

Correct:
```dart
// ✅ Use map before distinct
final values = await select(users)
  .map((u) => u.status)
  .distinct()
  .get();
```

---

# Summary

| Method | Purpose | Returns |
|--------|---------|---------|
| `.distinct()` | Remove duplicates | `List<T>` |
| `.map().distinct()` | Unique column values | `List<dynamic>` |
| `.map().distinct().count()` | Count unique values | `int` |

---

# Next Steps

Now you understand distinct, let's dive deeper:

- [Aliases](link) – Column aliases
- [Expressions](link) – Custom expressions
- [Custom SQL Expressions](link) – Raw SQL expressions

---

# Did You Know?

- **Distinct is performed at the database level** – Efficient

- **Distinct can be used on multiple columns** – Unique combinations

- **Distinct works with aggregations** – Count distinct values

- **Distinct can use indexes** – On selected columns

- **Distinct is useful for filters** – Generate filter options

- **Distinct returns a list** – Not a Set

- **Distinct with ORDER BY** – Can be combined

- **Distinct is SQL's DISTINCT** – Under the hood

---

