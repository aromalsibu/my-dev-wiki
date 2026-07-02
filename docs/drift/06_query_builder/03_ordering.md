## Ordering

**Sorting query results in Drift**

---

# What is it?

**Ordering** is the process of sorting query results by one or more columns in ascending or descending order. Drift provides a flexible API for ordering that supports single-column, multi-column, and custom sorting with full type safety.

> **Think of Ordering like "organizing a bookshelf"** – you arrange books by title, author, or publication date, either from A to Z or Z to A, to find what you need easily.

```dart
// 👇 Basic ordering (ascending)
final usersByName = await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.name)]))
  .get();

// 👇 Descending order
final usersByAgeDesc = await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.age, mode: OrderingMode.desc)]))
  .get();

// 👇 Multiple columns
final sortedUsers = await (select(users)
  ..orderBy([
    OrderingTerm(expression: u.age, mode: OrderingMode.desc),
    OrderingTerm(expression: u.name, mode: OrderingMode.asc),
  ]))
  .get();
```

> **What's happening here?**
> - **`orderBy()`** – Specifies sorting criteria
> - **`OrderingTerm`** – Defines a sort column and direction
> - **`OrderingMode.asc`** – Ascending (A-Z, 0-9)
> - **`OrderingMode.desc`** – Descending (Z-A, 9-0)
> - **Multi-column** – Sort by multiple columns sequentially

---

# Why does it exist?

- **Data Organization** – Present data in meaningful order
- **User Experience** – Show data in expected order
- **Business Logic** – Enforce business rules (most recent first)
- **Performance** – Use indexes for sorted queries
- **Analysis** – Easier data analysis
- **Consistency** – Predictable result ordering

---

# Basic Ordering

> **Simple sorting patterns**

## Single Column Ascending

```dart
// 👇 Sort by name (A-Z)
final usersByName = await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.name)]))
  .get();

// 👇 Sort by age (youngest first)
final usersByAgeAsc = await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.age)]))
  .get();

// 👇 Sort by date (oldest first)
final oldestUsers = await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.createdAt)]))
  .get();
```

## Single Column Descending

```dart
// 👇 Sort by name (Z-A)
final usersByNameDesc = await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.name, mode: OrderingMode.desc)]))
  .get();

// 👇 Sort by age (oldest first)
final usersByAgeDesc = await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.age, mode: OrderingMode.desc)]))
  .get();

// 👇 Sort by date (newest first)
final newestUsers = await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: OrderingMode.desc)]))
  .get();
```

---

# Multi-Column Ordering

> **Sorting by multiple columns**

## Two Columns

```dart
// 👇 Sort by age (descending), then name (ascending)
final sortedUsers = await (select(users)
  ..orderBy([
    OrderingTerm(expression: u.age, mode: OrderingMode.desc),
    OrderingTerm(expression: u.name, mode: OrderingMode.asc),
  ]))
  .get();

// 👇 Sort by status, then creation date
final sortedByStatus = await (select(users)
  ..orderBy([
    OrderingTerm(expression: u.status, mode: OrderingMode.asc),
    OrderingTerm(expression: u.createdAt, mode: OrderingMode.desc),
  ]))
  .get();
```

## Three or More Columns

```dart
// 👇 Sort by multiple columns
final complexSorted = await (select(users)
  ..orderBy([
    OrderingTerm(expression: u.isActive, mode: OrderingMode.desc),
    OrderingTerm(expression: u.age, mode: OrderingMode.desc),
    OrderingTerm(expression: u.name, mode: OrderingMode.asc),
  ]))
  .get();
```

---

# Ordering Helpers

> **Convenient ordering syntax**

## Using .asc() and .desc()

```dart
// 👇 Using .asc() helper
final usersByName = await (select(users)
  ..orderBy([(u) => OrderingTerm.asc(u.name)]))
  .get();

// 👇 Using .desc() helper
final usersByAgeDesc = await (select(users)
  ..orderBy([(u) => OrderingTerm.desc(u.age)]))
  .get();

// 👇 Mixed helpers
final sortedUsers = await (select(users)
  ..orderBy([
    OrderingTerm.desc(u.age),
    OrderingTerm.asc(u.name),
  ]))
  .get();
```

---

# Ordering with Filtering

> **Combining sorting with WHERE clauses**

```dart
// 👇 Filter then sort
final activeUsersByName = await (select(users)
  ..where((u) => u.isActive.equals(true))
  ..orderBy([(u) => OrderingTerm(expression: u.name)]))
  .get();

// 👇 Filter, sort, and limit
final recentActiveUsers = await (select(users)
  ..where((u) => u.isActive.equals(true))
  ..orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: OrderingMode.desc)])
  ..limit(10))
  .get();

// 👇 Complex filter with sorting
final specificUsers = await (select(users)
  ..where((u) => u.age > const Variable(18))
  ..where((u) => u.isVerified.equals(true))
  ..orderBy([
    OrderingTerm.desc(u.createdAt),
    OrderingTerm.asc(u.name),
  ]))
  .get();
```

---

# Random Ordering

> **Randomizing query results**

```dart
// 👇 Random ordering (SQLite RANDOM())
final randomUsers = await (select(users)
  ..orderBy([(u) => OrderingTerm.asc(u.id, collate: 'RANDOM')]))
  .get();

// 👇 Custom random ordering using raw SQL
final randomUsersCustom = await customSelect('''
  SELECT * FROM users ORDER BY RANDOM() LIMIT 10
''').get();
```

---

# Real-World Example

> **Complete e-commerce ordering system**

```dart
// lib/database/order_service.dart
import 'package:drift/drift.dart';

class OrderService {
  final AppDatabase db;
  
  OrderService(this.db);
  
  // ==================== USER ORDERING ====================
  
  // 👇 Get users with various sorting
  Future<List<User>> getUsers({
    SortOption? sortBy,
    bool ascending = true,
  }) async {
    final query = db.select(db.users);
    
    if (sortBy != null) {
      final mode = ascending ? OrderingMode.asc : OrderingMode.desc;
      
      switch (sortBy) {
        case SortOption.name:
          query.orderBy([(u) => OrderingTerm(expression: u.name, mode: mode)]);
          break;
        case SortOption.age:
          query.orderBy([(u) => OrderingTerm(expression: u.age, mode: mode)]);
          break;
        case SortOption.createdAt:
          query.orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: mode)]);
          break;
        case SortOption.status:
          query.orderBy([(u) => OrderingTerm(expression: u.status, mode: mode)]);
          break;
        case SortOption.active:
          query.orderBy([(u) => OrderingTerm(expression: u.isActive, mode: mode)]);
          break;
      }
    }
    
    return await query.get();
  }
  
  // 👇 Get users with advanced sorting
  Future<List<User>> getUsersAdvanced({
    String? filter,
    UserSort sortBy = UserSort.nameAsc,
    int? limit,
  }) async {
    final query = db.select(db.users);
    
    // Apply filter
    if (filter != null && filter.isNotEmpty) {
      query.where((u) => 
        u.name.like('%$filter%') |
        u.email.like('%$filter%')
      );
    }
    
    // Apply sorting
    switch (sortBy) {
      case UserSort.nameAsc:
        query.orderBy([(u) => OrderingTerm.asc(u.name)]);
        break;
      case UserSort.nameDesc:
        query.orderBy([(u) => OrderingTerm.desc(u.name)]);
        break;
      case UserSort.ageAsc:
        query.orderBy([(u) => OrderingTerm.asc(u.age)]);
        break;
      case UserSort.ageDesc:
        query.orderBy([(u) => OrderingTerm.desc(u.age)]);
        break;
      case UserSort.newest:
        query.orderBy([(u) => OrderingTerm.desc(u.createdAt)]);
        break;
      case UserSort.oldest:
        query.orderBy([(u) => OrderingTerm.asc(u.createdAt)]);
        break;
      case UserSort.activeFirst:
        query.orderBy([
          OrderingTerm.desc(u.isActive),
          OrderingTerm.asc(u.name),
        ]);
        break;
    }
    
    // Apply limit
    if (limit != null) {
      query.limit(limit);
    }
    
    return await query.get();
  }

  // ==================== PRODUCT ORDERING ====================
  
  // 👇 Get products with sorting
  Future<List<Product>> getProducts({
    String? category,
    ProductSort sortBy = ProductSort.nameAsc,
  }) async {
    final query = db.select(db.products);
    
    if (category != null && category.isNotEmpty) {
      query.where((p) => p.category.equals(category));
    }
    
    switch (sortBy) {
      case ProductSort.nameAsc:
        query.orderBy([(p) => OrderingTerm.asc(p.name)]);
        break;
      case ProductSort.nameDesc:
        query.orderBy([(p) => OrderingTerm.desc(p.name)]);
        break;
      case ProductSort.priceAsc:
        query.orderBy([(p) => OrderingTerm.asc(p.price)]);
        break;
      case ProductSort.priceDesc:
        query.orderBy([(p) => OrderingTerm.desc(p.price)]);
        break;
      case ProductSort.popularity:
        query.orderBy([(p) => OrderingTerm.desc(p.soldCount)]);
        break;
      case ProductSort.rating:
        query.orderBy([
          OrderingTerm.desc(p.rating),
          OrderingTerm.asc(p.name),
        ]);
        break;
      case ProductSort.newest:
        query.orderBy([(p) => OrderingTerm.desc(p.createdAt)]);
        break;
    }
    
    return await query.get();
  }
  
  // 👇 Get products with price range and sorting
  Future<List<Product>> getProductsFiltered({
    double? minPrice,
    double? maxPrice,
    ProductSort sortBy = ProductSort.priceAsc,
  }) async {
    final query = db.select(db.products);
    
    if (minPrice != null) {
      query.where((p) => p.price > const Variable(minPrice));
    }
    if (maxPrice != null) {
      query.where((p) => p.price < const Variable(maxPrice));
    }
    
    switch (sortBy) {
      case ProductSort.priceAsc:
        query.orderBy([(p) => OrderingTerm.asc(p.price)]);
        break;
      case ProductSort.priceDesc:
        query.orderBy([(p) => OrderingTerm.desc(p.price)]);
        break;
      default:
        query.orderBy([(p) => OrderingTerm.asc(p.name)]);
    }
    
    return await query.get();
  }

  // ==================== ORDER ORDERING ====================
  
  // 👇 Get orders with sorting
  Future<List<Order>> getOrders({
    int? userId,
    OrderSort sortBy = OrderSort.newest,
  }) async {
    final query = db.select(db.orders);
    
    if (userId != null) {
      query.where((o) => o.userId.equals(userId));
    }
    
    switch (sortBy) {
      case OrderSort.newest:
        query.orderBy([(o) => OrderingTerm.desc(o.orderDate)]);
        break;
      case OrderSort.oldest:
        query.orderBy([(o) => OrderingTerm.asc(o.orderDate)]);
        break;
      case OrderSort.highestTotal:
        query.orderBy([(o) => OrderingTerm.desc(o.total)]);
        break;
      case OrderSort.lowestTotal:
        query.orderBy([(o) => OrderingTerm.asc(o.total)]);
        break;
      case OrderSort.status:
        query.orderBy([
          OrderingTerm.asc(o.status),
          OrderingTerm.desc(o.orderDate),
        ]);
        break;
    }
    
    return await query.get();
  }

  // ==================== ADVANCED ORDERING ====================
  
  // 👇 Get top users by order count
  Future<List<UserStats>> getTopUsersByOrders(int limit) async {
    final results = await db.customSelect('''
      SELECT 
        u.id,
        u.name,
        COUNT(o.id) as order_count,
        SUM(o.total) as total_spent,
        AVG(o.total) as average_order
      FROM users u
      LEFT JOIN orders o ON u.id = o.user_id
      GROUP BY u.id
      ORDER BY order_count DESC, total_spent DESC
      LIMIT ?
    ''', variables: [Variable.withInt(limit)]).get();
    
    return results.map((row) {
      return UserStats(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        orderCount: row.data['order_count'] as int,
        totalSpent: row.data['total_spent'] as double? ?? 0.0,
        averageOrder: row.data['average_order'] as double? ?? 0.0,
      );
    }).toList();
  }
  
  // 👇 Get product sales ranking
  Future<List<ProductSales>> getProductSalesRanking(int limit) async {
    final results = await db.customSelect('''
      SELECT 
        p.id,
        p.name,
        p.sku,
        COUNT(oi.id) as order_count,
        SUM(oi.quantity) as total_sold,
        SUM(oi.total) as total_revenue
      FROM products p
      LEFT JOIN order_items oi ON p.id = oi.product_id
      GROUP BY p.id
      ORDER BY total_revenue DESC, total_sold DESC
      LIMIT ?
    ''', variables: [Variable.withInt(limit)]).get();
    
    return results.map((row) {
      return ProductSales(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        sku: row.data['sku'] as String,
        orderCount: row.data['order_count'] as int,
        totalSold: row.data['total_sold'] as int,
        totalRevenue: row.data['total_revenue'] as double? ?? 0.0,
      );
    }).toList();
  }
}

// ==================== ENUMS ====================

enum SortOption {
  name,
  age,
  createdAt,
  status,
  active,
}

enum UserSort {
  nameAsc,
  nameDesc,
  ageAsc,
  ageDesc,
  newest,
  oldest,
  activeFirst,
}

enum ProductSort {
  nameAsc,
  nameDesc,
  priceAsc,
  priceDesc,
  popularity,
  rating,
  newest,
}

enum OrderSort {
  newest,
  oldest,
  highestTotal,
  lowestTotal,
  status,
}

// ==================== DATA CLASSES ====================

class UserStats {
  final int id;
  final String name;
  final int orderCount;
  final double totalSpent;
  final double averageOrder;
  
  UserStats({
    required this.id,
    required this.name,
    required this.orderCount,
    required this.totalSpent,
    required this.averageOrder,
  });
}

class ProductSales {
  final int id;
  final String name;
  final String sku;
  final int orderCount;
  final int totalSold;
  final double totalRevenue;
  
  ProductSales({
    required this.id,
    required this.name,
    required this.sku,
    required this.orderCount,
    required this.totalSold,
    required this.totalRevenue,
  });
}
```

```dart
// lib/ui/pages/user_list_page.dart
class UserListPage extends StatefulWidget {
  final OrderService orderService;
  
  const UserListPage({required this.orderService});
  
  @override
  _UserListPageState createState() => _UserListPageState();
}

class _UserListPageState extends State<UserListPage> {
  UserSort _currentSort = UserSort.nameAsc;
  List<User> _users = [];
  bool _isLoading = true;
  
  @override
  void initState() {
    super.initState();
    _loadUsers();
  }
  
  Future<void> _loadUsers() async {
    setState(() => _isLoading = true);
    
    try {
      final users = await widget.orderService.getUsersAdvanced(
        sortBy: _currentSort,
      );
      
      setState(() {
        _users = users;
        _isLoading = false;
      });
    } catch (e) {
      setState(() => _isLoading = false);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Users'),
        bottom: PreferredSize(
          preferredSize: Size.fromHeight(50),
          child: Container(
            padding: EdgeInsets.symmetric(horizontal: 16),
            child: Row(
              children: [
                Icon(Icons.sort),
                SizedBox(width: 8),
                Expanded(
                  child: DropdownButton<UserSort>(
                    isExpanded: true,
                    value: _currentSort,
                    items: UserSort.values.map((sort) {
                      return DropdownMenuItem(
                        value: sort,
                        child: Text(_getSortLabel(sort)),
                      );
                    }).toList(),
                    onChanged: (value) {
                      if (value != null) {
                        setState(() {
                          _currentSort = value;
                        });
                        _loadUsers();
                      }
                    },
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
      body: _isLoading
          ? Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: _users.length,
              itemBuilder: (context, index) {
                final user = _users[index];
                return ListTile(
                  title: Text(user.name),
                  subtitle: Text('Age: ${user.age ?? "N/A"}'),
                  trailing: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      if (user.isActive)
                        Icon(Icons.check_circle, color: Colors.green, size: 20),
                      Text(
                        user.createdAt.toString().substring(0, 10),
                        style: TextStyle(fontSize: 12),
                      ),
                    ],
                  ),
                );
              },
            ),
    );
  }
  
  String _getSortLabel(UserSort sort) {
    switch (sort) {
      case UserSort.nameAsc: return 'Name (A-Z)';
      case UserSort.nameDesc: return 'Name (Z-A)';
      case UserSort.ageAsc: return 'Age (Youngest)';
      case UserSort.ageDesc: return 'Age (Oldest)';
      case UserSort.newest: return 'Newest First';
      case UserSort.oldest: return 'Oldest First';
      case UserSort.activeFirst: return 'Active First';
    }
  }
}
```

---

# Best Practices

- **Use indexes** – On sorted columns for performance
- **Limit results** – When sorting large datasets
- **Multi-column sorting** – For complex ordering
- **Consistent ordering** – Always use orderBy with pagination
- **Use .asc()/.desc() helpers** – For cleaner code
- **Consider NULL values** – NULLs appear first or last
- **Test with data** – Verify sorting works correctly

---

# Common Mistakes

## Mistake 1: Ordering without limit on large tables

Wrong:
```dart
// 🚫 Sorts all records (slow for large tables)
await (select(users)..orderBy([(u) => OrderingTerm.asc(u.name)])).get();
```

Correct:
```dart
// ✅ Add limit for performance
await (select(users)
  ..orderBy([(u) => OrderingTerm.asc(u.name)])
  ..limit(100))
  .get();
```

## Mistake 2: Wrong order in multi-column sort

Wrong:
```dart
// 🚫 Names sorted, then ages (less useful)
..orderBy([
  OrderingTerm.asc(u.name),
  OrderingTerm.desc(u.age),
])
```

Correct:
```dart
// ✅ Ages first, then names (more useful)
..orderBy([
  OrderingTerm.desc(u.age),
  OrderingTerm.asc(u.name),
])
```

## Mistake 3: Forgetting to include orderBy with pagination

Wrong:
```dart
// 🚫 Unpredictable order with pagination
await (select(users)..limit(10, offset: 0)).get();
```

Correct:
```dart
// ✅ Always order when paginating
await (select(users)
  ..orderBy([(u) => OrderingTerm.asc(u.id)])
  ..limit(10, offset: 0))
  .get();
```

---

# Summary

| Method | Purpose | Example |
|--------|---------|---------|
| `orderBy([term])` | Sort results | `orderBy([OrderingTerm.asc(u.name)])` |
| `.asc()` | Ascending order | `OrderingTerm.asc(u.age)` |
| `.desc()` | Descending order | `OrderingTerm.desc(u.age)` |
| Multiple terms | Multi-column sort | `orderBy([term1, term2])` |

---

# Next Steps

Now you understand ordering, let's dive deeper:

- [Limiting](link) – Pagination and limiting
- [Distinct](link) – Getting unique values
- [Aliases](link) – Column aliases

---

# Did You Know?

- **Ordering can use indexes** – For performance

- **NULL values sort first** – In ascending order

- **Multi-column sorts create composite indexes** – For performance

- **Ordering can use expressions** – Not just columns

- **Random ordering is possible** – Using `RANDOM()`

- **Ordering is applied after filtering** – For performance

- **Ordering can be case-sensitive** – Using collation

- **Ordering is required for pagination** – Consistent results

---

