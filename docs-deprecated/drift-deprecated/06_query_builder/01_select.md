
## Select

**Retrieving data from your Drift database**

---

# What is it?

**Select** is the fundamental operation for reading data from your database. Drift provides a powerful, type-safe query builder that allows you to write complex SQL queries using Dart syntax with full IDE support and compile-time validation. Instead of writing raw SQL strings, you chain methods to build your queries.

> **Think of Select like "searching for files"** – you specify what you're looking for, filter by criteria, sort the results, and get back exactly the data you need.

```dart
// 👇 Basic select without conditions
final allUsers = await select(users).get();
// Returns: List<User>

// 👇 Select with WHERE filter
final activeUsers = await (select(users)
  ..where((u) => u.isActive.equals(true)))
  .get();
// Returns: List<User> with only active users

// 👇 Select with multiple conditions and sorting
final youngUsers = await (select(users)
  ..where((u) => u.age < const Variable(30))
  ..where((u) => u.isActive.equals(true))
  ..orderBy([(u) => OrderingTerm(expression: u.age, mode: OrderingMode.desc)]))
  .get();
```

> **What's happening here?**
> - **`select(table)`** – Start a query on a table
> - **`where()`** – Filter results with conditions
> - **`orderBy()`** – Sort results
> - **`limit()`** – Limit number of results
> - **`get()`** – Execute the query and return results

---

# Why does it exist?

- **Read Data** – Retrieve data from the database
- **Type Safety** – Compile-time checking of queries
- **IDE Support** – Autocomplete and refactoring
- **Flexibility** – Complex queries with simple syntax
- **Performance** – Optimized SQL generation
- **Reactive** – Convert to streams for real-time updates

---

# Basic Select Operations

> **Simple query patterns**

## Select All Records

```dart
// 👇 Get all users
final allUsers = await select(users).get();

// 👇 Get all active users
final activeUsers = await (select(users)
  ..where((u) => u.isActive.equals(true)))
  .get();

// 👇 Get first 10 users
final firstTen = await (select(users)
  ..limit(10))
  .get();
```

## Select with WHERE Conditions

```dart
// 👇 Single condition
final usersOver18 = await (select(users)
  ..where((u) => u.age > const Variable(18)))
  .get();

// 👇 Multiple conditions (AND)
final activeAdults = await (select(users)
  ..where((u) => u.age > const Variable(18))
  ..where((u) => u.isActive.equals(true)))
  .get();

// 👇 OR condition
final specialUsers = await (select(users)
  ..where((u) => u.age < const Variable(13) | u.age > const Variable(65)))
  .get();

// 👇 Complex condition with AND & OR
final users = await (select(users)
  ..where((u) => 
    (u.age > const Variable(18) & u.isActive.equals(true)) |
    (u.age < const Variable(13))
  ))
  .get();
```

---

# Advanced Where Conditions

> **All filtering possibilities**

## Comparison Operators

```dart
// 👇 Equal
await (select(users)..where((u) => u.name.equals('John'))).get();

// 👇 Not equal
await (select(users)..where((u) => u.name.isNotValue('John'))).get();

// 👇 Greater than
await (select(users)..where((u) => u.age > const Variable(18))).get();

// 👇 Greater than or equal
await (select(users)..where((u) => u.age.isBiggerOrEqualValue(18))).get();

// 👇 Less than
await (select(users)..where((u) => u.age < const Variable(18))).get();

// 👇 Less than or equal
await (select(users)..where((u) => u.age.isSmallerOrEqualValue(18))).get();

// 👇 Between
await (select(users)..where((u) => u.age.isBetweenValues(18, 65))).get();

// 👇 In list
await (select(users)..where((u) => u.id.isIn([1, 2, 3, 4, 5]))).get();

// 👇 Not in list
await (select(users)..where((u) => u.id.isNotIn([1, 2, 3]))).get();
```

## String Conditions

```dart
// 👇 Like (pattern matching)
await (select(users)..where((u) => u.name.like('%John%'))).get();

// 👇 Starts with
await (select(users)..where((u) => u.name.startsWith('Jo'))).get();

// 👇 Ends with
await (select(users)..where((u) => u.name.endsWith('hn'))).get();

// 👇 Contains (LIKE %value%)
await (select(users)..where((u) => u.name.contains('oh'))).get();

// 👇 Case-insensitive search
await (select(users)..where((u) => u.name.like('%john%', caseSensitive: false))).get();

// 👇 Empty string check
await (select(users)..where((u) => u.name.equals(''))).get();
```

## Null Checks

```dart
// 👇 IS NULL
await (select(users)..where((u) => u.age.isNull())).get();

// 👇 IS NOT NULL
await (select(users)..where((u) => u.age.isNotNull())).get();

// 👇 Combined with other conditions
await (select(users)
  ..where((u) => u.age.isNotNull())
  ..where((u) => u.age > const Variable(18)))
  .get();
```

---

# Sorting Results

> **Ordering query results**

## Single Column Sorting

```dart
// 👇 Ascending (default)
await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.name)]))
  .get();

// 👇 Descending
await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.name, mode: OrderingMode.desc)]))
  .get();

// 👇 By date (newest first)
await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: OrderingMode.desc)]))
  .get();
```

## Multi-Column Sorting

```dart
// 👇 Sort by multiple columns
await (select(users)
  ..orderBy([
    OrderingTerm(expression: u.age, mode: OrderingMode.desc),
    OrderingTerm(expression: u.name),
  ]))
  .get();

// 👇 Sort by age desc, then name asc
await (select(users)
  ..orderBy([
    (u) => OrderingTerm.desc(u.age),
    (u) => OrderingTerm.asc(u.name),
  ]))
  .get();
```

---

# Pagination

> **Limiting and offsetting results**

## Basic Pagination

```dart
// 👇 Get first 10 users
await (select(users)
  ..limit(10))
  .get();

// 👇 Get users 11-20 (page 2)
await (select(users)
  ..limit(10, offset: 10))
  .get();

// 👇 Pagination with sorting
await (select(users)
  ..orderBy([(u) => OrderingTerm(expression: u.id)])
  ..limit(10, offset: 20))
  .get();
```

## Page-Based Pagination

```dart
class PaginatedResult<T> {
  final List<T> items;
  final int total;
  final int page;
  final int pageSize;
  final bool hasMore;
  
  PaginatedResult({
    required this.items,
    required this.total,
    required this.page,
    required this.pageSize,
    this.hasMore = false,
  });
}

Future<PaginatedResult<User>> getUsersPaginated({
  required int page,
  int pageSize = 10,
  String? search,
}) async {
  // Build query
  final query = select(users);
  
  if (search != null && search.isNotEmpty) {
    query.where((u) => u.name.like('%$search%'));
  }
  
  // Get total count for pagination
  final total = await query.count();
  
  // Get paginated results
  final items = await (query
    ..orderBy([(u) => OrderingTerm(expression: u.id)])
    ..limit(pageSize, offset: page * pageSize))
    .get();
  
  final hasMore = (page + 1) * pageSize < total;
  
  return PaginatedResult(
    items: items,
    total: total,
    page: page,
    pageSize: pageSize,
    hasMore: hasMore,
  );
}

// Usage
final result = await getUsersPaginated(page: 0, pageSize: 10);
print('Users: ${result.items.length} of ${result.total}');
```

---

# Selecting Single Records

> **Getting one record**

## Get Single Record

```dart
// 👇 Get first record (throws if none)
final firstUser = await select(users).getSingle();

// 👇 Get single record by ID
final user = await (select(users)
  ..where((u) => u.id.equals(1)))
  .getSingle();

// 👇 Get single record or null (safe)
final user = await (select(users)
  ..where((u) => u.id.equals(999)))
  .getSingleOrNull();

if (user != null) {
  print('Found user: ${user.name}');
} else {
  print('User not found');
}

// 👇 Get first matching record
final firstActiveUser = await (select(users)
  ..where((u) => u.isActive.equals(true)))
  .getSingleOrNull();
```

---

# Count Queries

> **Counting records**

## Basic Count

```dart
// 👇 Count all users
final totalUsers = await select(users).count();

// 👇 Count active users
final activeCount = await (select(users)
  ..where((u) => u.isActive.equals(true)))
  .count();

// 👇 Count users over 18
final adultCount = await (select(users)
  ..where((u) => u.age > const Variable(18)))
  .count();
```

## Aggregated Count

```dart
// 👇 Count distinct emails
final distinctEmails = await select(users)
  .map((u) => u.email)
  .distinct()
  .count();

// 👇 Count by group (using custom SQL)
final countByAge = await customSelect('''
  SELECT age, COUNT(*) as count 
  FROM users 
  GROUP BY age
''').get();
```

---

# Real-World Example

> **Complete e-commerce query system**

```dart
// lib/database/query_service.dart
import 'package:drift/drift.dart';

class QueryService {
  final AppDatabase db;
  
  QueryService(this.db);
  
  // ==================== USER QUERIES ====================
  
  // 👇 Get user by ID
  Future<User?> getUserById(int id) async {
    return await (db.select(db.users)
      ..where((u) => u.id.equals(id)))
      .getSingleOrNull();
  }
  
  // 👇 Get user by email
  Future<User?> getUserByEmail(String email) async {
    return await (db.select(db.users)
      ..where((u) => u.email.equals(email)))
      .getSingleOrNull();
  }
  
  // 👇 Search users by name
  Future<List<User>> searchUsers(String query) async {
    return await (db.select(db.users)
      ..where((u) => u.name.like('%$query%')))
      .get();
  }
  
  // 👇 Get active users sorted by creation
  Future<List<User>> getActiveUsers() async {
    return await (db.select(db.users)
      ..where((u) => u.isActive.equals(true))
      ..orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: OrderingMode.desc)]))
      .get();
  }
  
  // 👇 Get users with filters
  Future<List<User>> filterUsers({
    String? name,
    int? minAge,
    int? maxAge,
    bool? isActive,
    bool? isVerified,
  }) async {
    final query = db.select(db.users);
    
    if (name != null && name.isNotEmpty) {
      query.where((u) => u.name.like('%$name%'));
    }
    
    if (minAge != null) {
      query.where((u) => u.age > const Variable(minAge));
    }
    
    if (maxAge != null) {
      query.where((u) => u.age < const Variable(maxAge));
    }
    
    if (isActive != null) {
      query.where((u) => u.isActive.equals(isActive));
    }
    
    if (isVerified != null) {
      query.where((u) => u.isVerified.equals(isVerified));
    }
    
    return await query.get();
  }
  
  // 👇 Get user statistics
  Future<Map<String, dynamic>> getUserStats() async {
    final total = await db.select(db.users).count();
    final active = await (db.select(db.users)
      ..where((u) => u.isActive.equals(true)))
      .count();
    
    final verified = await (db.select(db.users)
      ..where((u) => u.isVerified.equals(true)))
      .count();
    
    final avgAge = await db.select(db.users)
      .map((u) => u.age)
      .avg();
    
    final minAge = await db.select(db.users)
      .map((u) => u.age)
      .min();
    
    final maxAge = await db.select(db.users)
      .map((u) => u.age)
      .max();
    
    return {
      'total': total,
      'active': active,
      'inactive': total - active,
      'verified': verified,
      'unverified': total - verified,
      'averageAge': avgAge ?? 0,
      'minAge': minAge ?? 0,
      'maxAge': maxAge ?? 0,
    };
  }

  // ==================== PRODUCT QUERIES ====================
  
  // 👇 Get product with details
  Future<Product?> getProductById(int id) async {
    return await (db.select(db.products)
      ..where((p) => p.id.equals(id)))
      .getSingleOrNull();
  }
  
  // 👇 Get product by SKU
  Future<Product?> getProductBySku(String sku) async {
    return await (db.select(db.products)
      ..where((p) => p.sku.equals(sku)))
      .getSingleOrNull();
  }
  
  // 👇 Get active products with filters
  Future<List<Product>> getProducts({
    String? name,
    double? minPrice,
    double? maxPrice,
    int? minStock,
    bool? isActive,
  }) async {
    final query = db.select(db.products);
    
    if (name != null && name.isNotEmpty) {
      query.where((p) => p.name.like('%$name%'));
    }
    
    if (minPrice != null) {
      query.where((p) => p.price > const Variable(minPrice));
    }
    
    if (maxPrice != null) {
      query.where((p) => p.price < const Variable(maxPrice));
    }
    
    if (minStock != null) {
      query.where((p) => p.stock > const Variable(minStock));
    }
    
    if (isActive != null) {
      query.where((p) => p.isActive.equals(isActive));
    }
    
    return await query.get();
  }
  
  // 👇 Get products by category
  Future<List<Product>> getProductsByCategory(int categoryId) async {
    return await (db.select(db.products)
      ..where((p) => p.categoryId.equals(categoryId))
      ..orderBy([(p) => OrderingTerm(expression: p.name)]))
      .get();
  }
  
  // 👇 Get low stock products
  Future<List<Product>> getLowStockProducts(int threshold) async {
    return await (db.select(db.products)
      ..where((p) => p.stock < const Variable(threshold))
      ..orderBy([(p) => OrderingTerm(expression: p.stock)]))
      .get();
  }
  
  // 👇 Get product price range stats
  Future<Map<String, dynamic>> getProductPriceStats() async {
    final avgPrice = await db.select(db.products)
      .map((p) => p.price)
      .avg();
    
    final minPrice = await db.select(db.products)
      .map((p) => p.price)
      .min();
    
    final maxPrice = await db.select(db.products)
      .map((p) => p.price)
      .max();
    
    return {
      'averagePrice': avgPrice ?? 0,
      'minPrice': minPrice ?? 0,
      'maxPrice': maxPrice ?? 0,
    };
  }

  // ==================== ORDER QUERIES ====================
  
  // 👇 Get order with items
  Future<OrderWithItems> getOrderWithItems(int orderId) async {
    final order = await (db.select(db.orders)
      ..where((o) => o.id.equals(orderId)))
      .getSingle();
    
    final items = await (db.select(db.orderItems)
      ..where((i) => i.orderId.equals(orderId)))
      .get();
    
    return OrderWithItems(order: order, items: items);
  }
  
  // 👇 Get user's orders
  Future<List<Order>> getUserOrders(int userId) async {
    return await (db.select(db.orders)
      ..where((o) => o.userId.equals(userId))
      ..orderBy([(o) => OrderingTerm(expression: o.orderDate, mode: OrderingMode.desc)]))
      .get();
  }
  
  // 👇 Get orders by status
  Future<List<Order>> getOrdersByStatus(String status) async {
    return await (db.select(db.orders)
      ..where((o) => o.status.equals(status))
      ..orderBy([(o) => OrderingTerm(expression: o.orderDate, mode: OrderingMode.desc)]))
      .get();
  }
  
  // 👇 Get recent orders with items
  Future<List<OrderWithItems>> getRecentOrders(int limit) async {
    final orders = await (db.select(db.orders)
      ..orderBy([(o) => OrderingTerm(expression: o.orderDate, mode: OrderingMode.desc)])
      ..limit(limit))
      .get();
    
    final result = <OrderWithItems>[];
    for (final order in orders) {
      final items = await (db.select(db.orderItems)
        ..where((i) => i.orderId.equals(order.id)))
        .get();
      
      // Get product details for each item
      final itemWithProducts = <OrderItemWithProduct>[];
      for (final item in items) {
        final product = await (db.select(db.products)
          ..where((p) => p.id.equals(item.productId)))
          .getSingle();
        
        itemWithProducts.add(
          OrderItemWithProduct(orderItem: item, product: product),
        );
      }
      
      result.add(OrderWithItems(
        order: order,
        items: itemWithProducts,
      ));
    }
    
    return result;
  }
  
  // 👇 Get order statistics
  Future<Map<String, dynamic>> getOrderStats() async {
    final total = await db.select(db.orders).count();
    final pending = await (db.select(db.orders)
      ..where((o) => o.status.equals('pending')))
      .count();
    
    final completed = await (db.select(db.orders)
      ..where((o) => o.status.equals('delivered')))
      .count();
    
    final cancelled = await (db.select(db.orders)
      ..where((o) => o.status.equals('cancelled')))
      .count();
    
    final totalRevenue = await db.select(db.orders)
      .map((o) => o.total)
      .sum();
    
    final avgOrderValue = total > 0 
        ? (totalRevenue ?? 0) / total 
        : 0.0;
    
    return {
      'total': total,
      'pending': pending,
      'completed': completed,
      'cancelled': cancelled,
      'totalRevenue': totalRevenue ?? 0,
      'averageOrderValue': avgOrderValue,
    };
  }

  // ==================== COMPLEX QUERIES ====================
  
  // 👇 Search across multiple tables
  Future<List<SearchResult>> globalSearch(String query) async {
    final results = await db.customSelect('''
      SELECT 
        'user' as type,
        u.id as id,
        u.name as name,
        u.email as detail
      FROM users u
      WHERE u.name LIKE ? OR u.email LIKE ?
      
      UNION
      
      SELECT 
        'product' as type,
        p.id as id,
        p.name as name,
        p.sku as detail
      FROM products p
      WHERE p.name LIKE ? OR p.sku LIKE ?
      
      UNION
      
      SELECT 
        'order' as type,
        o.id as id,
        o.order_number as name,
        CAST(o.total AS TEXT) as detail
      FROM orders o
      WHERE o.order_number LIKE ?
    ''', variables: [
      Variable.withString('%$query%'),
      Variable.withString('%$query%'),
      Variable.withString('%$query%'),
      Variable.withString('%$query%'),
      Variable.withString('%$query%'),
    ]).get();
    
    return results.map((row) {
      return SearchResult(
        type: row.data['type'] as String,
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        detail: row.data['detail'] as String?,
      );
    }).toList();
  }
  
  // 👇 Get dashboard data
  Future<DashboardData> getDashboardData() async {
    final userStats = await getUserStats();
    final productStats = await getProductPriceStats();
    final orderStats = await getOrderStats();
    
    final recentOrders = await getRecentOrders(5);
    
    return DashboardData(
      userStats: userStats,
      productStats: productStats,
      orderStats: orderStats,
      recentOrders: recentOrders,
    );
  }
}

// ==================== DATA CLASSES ====================

class OrderWithItems {
  final Order order;
  final List<OrderItemWithProduct> items;
  
  OrderWithItems({required this.order, required this.items});
}

class OrderItemWithProduct {
  final OrderItem orderItem;
  final Product product;
  
  OrderItemWithProduct({required this.orderItem, required this.product});
}

class SearchResult {
  final String type;
  final int id;
  final String name;
  final String? detail;
  
  SearchResult({
    required this.type,
    required this.id,
    required this.name,
    this.detail,
  });
}

class DashboardData {
  final Map<String, dynamic> userStats;
  final Map<String, dynamic> productStats;
  final Map<String, dynamic> orderStats;
  final List<OrderWithItems> recentOrders;
  
  DashboardData({
    required this.userStats,
    required this.productStats,
    required this.orderStats,
    required this.recentOrders,
  });
}
```

```dart
// lib/ui/pages/search_page.dart
class SearchPage extends StatefulWidget {
  final QueryService queryService;
  
  const SearchPage({required this.queryService});
  
  @override
  _SearchPageState createState() => _SearchPageState();
}

class _SearchPageState extends State<SearchPage> {
  final _searchController = TextEditingController();
  List<SearchResult> _results = [];
  bool _isLoading = false;
  
  Future<void> _search() async {
    final query = _searchController.text.trim();
    if (query.isEmpty) {
      setState(() => _results = []);
      return;
    }
    
    setState(() => _isLoading = true);
    
    try {
      final results = await widget.queryService.globalSearch(query);
      setState(() {
        _results = results;
        _isLoading = false;
      });
    } catch (e) {
      setState(() => _isLoading = false);
      // Handle error
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Search')),
      body: Column(
        children: [
          Padding(
            padding: EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _searchController,
                    decoration: InputDecoration(
                      hintText: 'Search users, products, orders...',
                      border: OutlineInputBorder(),
                    ),
                    onSubmitted: (_) => _search(),
                  ),
                ),
                SizedBox(width: 8),
                ElevatedButton(
                  onPressed: _search,
                  child: Text('Search'),
                ),
              ],
            ),
          ),
          if (_isLoading)
            Center(child: CircularProgressIndicator())
          else
            Expanded(
              child: ListView.builder(
                itemCount: _results.length,
                itemBuilder: (context, index) {
                  final result = _results[index];
                  return ListTile(
                    leading: Icon(_getIcon(result.type)),
                    title: Text(result.name),
                    subtitle: Text(result.detail ?? ''),
                    trailing: Text(result.type),
                  );
                },
              ),
            ),
        ],
      ),
    );
  }
  
  IconData _getIcon(String type) {
    switch (type) {
      case 'user': return Icons.person;
      case 'product': return Icons.inventory;
      case 'order': return Icons.shopping_cart;
      default: return Icons.search;
    }
  }
}
```

---

# Best Practices

- **Use type-safe query builder** – Avoid raw SQL when possible
- **Add indexes** – For frequently queried columns
- **Use limit** – For large result sets
- **Use orderBy** – For consistent results
- **Use getSingleOrNull** – For optional results
- **Use count for totals** – Before pagination
- **Use where with variables** – Prevent SQL injection
- **Test queries** – Verify performance

---

# Common Mistakes

## Mistake 1: Not using limit

Wrong:
```dart
// 🚫 Loads all records (performance issue)
final allUsers = await select(users).get();
```

Correct:
```dart
// ✅ Use limit for large tables
final recentUsers = await (select(users)
  ..limit(100))
  .get();
```

## Mistake 2: Not using indexes

Wrong:
```dart
// 🚫 Slow query without index
await (select(users)..where((u) => u.email.equals('john@example.com'))).get();
```

Correct:
```dart
// ✅ Add index in table definition
class Users extends Table {
  TextColumn get email => text().unique()();
  
  @override
  List<Index> get indexes => [
    Index('idx_users_email', 'email'), // Add index
  ];
}
```

## Mistake 3: Not handling null results

Wrong:
```dart
// 🚫 Throws if no result
final user = await (select(users)..where((u) => u.id.equals(999)))
  .getSingle();
```

Correct:
```dart
// ✅ Handle null
final user = await (select(users)..where((u) => u.id.equals(999)))
  .getSingleOrNull();
```

---

# Summary

| Method | Purpose | Returns |
|--------|---------|---------|
| `get()` | All matching records | `List<T>` |
| `getSingle()` | One record | `T` |
| `getSingleOrNull()` | Optional record | `T?` |
| `count()` | Number of records | `int` |
| `first()` | First record | `T` |
| `map()` | Custom columns | `List<dynamic>` |

---

# Next Steps

Now you understand select operations, let's dive deeper:

- [Filtering](link) – Advanced filtering
- [Joins](link) – Table joins
- [Aggregations](link) – Advanced aggregations

---

# Did You Know?

- **Select queries are lazy** – Execute only when `get()` is called

- **Where clauses are composable** – Chain multiple conditions

- **OrderBy can use multiple columns** – For complex sorting

- **Limit with offset** – For efficient pagination

- **Count queries are optimized** – Use `count()` for totals

- **Custom SQL is available** – For complex queries

- **Streams can be built from selects** – For real-time updates

- **Select operations are type-safe** – Compile-time checking

---

