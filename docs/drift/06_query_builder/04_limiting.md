## Limiting

**Controlling result set size and pagination in Drift**

---

# What is it?

**Limiting** is the process of restricting the number of rows returned by a query, and optionally skipping a certain number of rows (offset). This is essential for pagination, performance optimization, and preventing excessive data loading.

> **Think of Limiting like "reading a book one chapter at a time"** – instead of reading the entire book at once, you read a few pages (limit) and start from where you left off (offset).

```dart
// 👇 Get first 10 users
final firstPage = await (select(users)
  ..limit(10))
  .get();

// 👇 Get users 11-20 (page 2)
final secondPage = await (select(users)
  ..limit(10, offset: 10))
  .get();

// 👇 Limit with sorting for consistent pagination
final sortedPage = await (select(users)
  ..orderBy([(u) => OrderingTerm.asc(u.id)])
  ..limit(10, offset: 20))
  .get();
```

> **What's happening here?**
> - **`limit(count)`** – Restrict results to `count` rows
> - **`offset`** – Skip first `offset` rows
> - **Paginated results** – Combine `limit` and `offset` for pages
> - **Performance** – Reduces data transfer and processing

---

# Why does it exist?

- **Performance** – Load only what you need
- **Pagination** – Break large datasets into pages
- **User Experience** – Faster loading times
- **Bandwidth** – Reduce data transfer
- **Memory** – Lower memory usage
- **Scalability** – Handle large datasets efficiently

---

# Basic Limiting

> **Simple limit patterns**

## Limit Without Offset

```dart
// 👇 Get first 5 users
final firstFive = await (select(users)
  ..limit(5))
  .get();

// 👇 Get top 10 oldest users
final oldestTen = await (select(users)
  ..orderBy([(u) => OrderingTerm.desc(u.age)])
  ..limit(10))
  .get();

// 👇 Get 20 most recent orders
final recentOrders = await (select(orders)
  ..orderBy([(o) => OrderingTerm.desc(o.orderDate)])
  ..limit(20))
  .get();
```

## Limit with Offset

```dart
// 👇 Get users 6-10 (skip first 5)
final secondBatch = await (select(users)
  ..limit(5, offset: 5))
  .get();

// 👇 Get users 11-20 (page 3 with pageSize 10)
final pageThree = await (select(users)
  ..limit(10, offset: 20))
  .get();
```

---

# Pagination

> **Implementing pagination in queries**

## Basic Pagination

```dart
class PaginationHelper {
  final AppDatabase db;
  
  PaginationHelper(this.db);
  
  Future<PaginatedResult<User>> getUsers({
    required int page,
    int pageSize = 10,
    String? search,
  }) async {
    final query = db.select(db.users);
    
    // Apply filters
    if (search != null && search.isNotEmpty) {
      query.where((u) => u.name.like('%$search%'));
    }
    
    // Get total count
    final total = await query.count();
    
    // Get paginated results with sorting
    final items = await (query
      ..orderBy([(u) => OrderingTerm.asc(u.id)])
      ..limit(pageSize, offset: page * pageSize))
      .get();
    
    return PaginatedResult(
      items: items,
      total: total,
      page: page,
      pageSize: pageSize,
      hasMore: (page + 1) * pageSize < total,
    );
  }
}

// Usage
final result = await helper.getUsers(page: 0, pageSize: 10);
print('Showing ${result.items.length} of ${result.total} users');
```

---

## Cursor-Based Pagination

```dart
class CursorPaginationHelper {
  final AppDatabase db;
  
  CursorPaginationHelper(this.db);
  
  Future<CursorResult<User>> getUsersAfter({
    int? cursorId, // Last seen user ID
    int limit = 10,
    String? search,
  }) async {
    final query = db.select(db.users);
    
    // Apply search filter
    if (search != null && search.isNotEmpty) {
      query.where((u) => u.name.like('%$search%'));
    }
    
    // Apply cursor (get items after this ID)
    if (cursorId != null) {
      query.where((u) => u.id > const Variable(cursorId));
    }
    
    // Get items with limit
    final items = await (query
      ..orderBy([(u) => OrderingTerm.asc(u.id)])
      ..limit(limit + 1)) // Get one extra to check if more exist
      .get();
    
    final hasMore = items.length > limit;
    final resultItems = hasMore ? items.sublist(0, limit) : items;
    final nextCursor = hasMore && resultItems.isNotEmpty 
        ? resultItems.last.id 
        : null;
    
    return CursorResult(
      items: resultItems,
      nextCursor: nextCursor,
      hasMore: hasMore,
    );
  }
}

// Usage
var result = await helper.getUsersAfter(limit: 10);
// Show items, store result.nextCursor
result = await helper.getUsersAfter(cursorId: result.nextCursor, limit: 10);
```

---

# Limiting with Filtering

> **Combining limits with WHERE clauses**

```dart
// 👇 Get top 5 oldest active users
final oldestActive = await (select(users)
  ..where((u) => u.isActive.equals(true))
  ..orderBy([(u) => OrderingTerm.desc(u.age)])
  ..limit(5))
  .get();

// 👇 Get 10 most recent verified users
final recentVerified = await (select(users)
  ..where((u) => u.isVerified.equals(true))
  ..orderBy([(u) => OrderingTerm.desc(u.createdAt)])
  ..limit(10))
  .get();

// 👇 Get paginated results with filters
Future<List<User>> getActiveUsersPaginated(int page, int pageSize) async {
  return await (select(users)
    ..where((u) => u.isActive.equals(true))
    ..orderBy([(u) => OrderingTerm.asc(u.id)])
    ..limit(pageSize, offset: page * pageSize))
    .get();
}
```

---

# Limiting with Joins

> **Limiting results in joined queries**

```dart
// 👇 Get recent orders with user details
final recentOrdersWithUsers = await (select(orders)
  ..join([innerJoin(users, users.id.equals(orders.userId))])
  ..orderBy([(o) => OrderingTerm.desc(o.orderDate)])
  ..limit(10))
  .get();

// 👇 Get top 5 users with most orders
final topUsers = await customSelect('''
  SELECT 
    u.id,
    u.name,
    COUNT(o.id) as order_count
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id
  ORDER BY order_count DESC
  LIMIT 5
''').get();
```

---

# Performance with Limits

> **Optimizing queries with limits**

## Query Optimization

```dart
// 👇 Bad: Loading all records
final allUsers = await select(users).get(); // Millions of records!

// 👇 Good: Load only what you need
final recentUsers = await (select(users)
  ..orderBy([(u) => OrderingTerm.desc(u.createdAt)])
  ..limit(100))
  .get();

// 👇 Best: Use pagination
final pageUsers = await (select(users)
  ..orderBy([(u) => OrderingTerm.asc(u.id)])
  ..limit(50, offset: (page - 1) * 50))
  .get();
```

## Count + Limit Pattern

```dart
Future<PaginatedResult<User>> getUsersWithCount({
  required int page,
  int pageSize = 20,
  String? search,
}) async {
  final query = db.select(db.users);
  
  // Apply filters
  if (search != null && search.isNotEmpty) {
    query.where((u) => u.name.like('%$search%'));
  }
  
  // Count total (for pagination info)
  final total = await query.count();
  
  // Get paginated results
  final items = await (query
    ..orderBy([(u) => OrderingTerm.asc(u.id)])
    ..limit(pageSize, offset: page * pageSize))
    .get();
  
  return PaginatedResult(
    items: items,
    total: total,
    page: page,
    pageSize: pageSize,
    hasMore: (page + 1) * pageSize < total,
  );
}
```

---

# Real-World Example

> **Complete e-commerce pagination system**

```dart
// lib/database/pagination_service.dart
import 'package:drift/drift.dart';

class PaginationService {
  final AppDatabase db;
  
  PaginationService(this.db);
  
  // ==================== USER PAGINATION ====================
  
  // 👇 Simple user pagination
  Future<PaginatedResult<User>> getUsers({
    required int page,
    int pageSize = 20,
    String? search,
    bool? isActive,
  }) async {
    final query = db.select(db.users);
    
    // Filters
    if (search != null && search.isNotEmpty) {
      query.where((u) => 
        u.name.like('%$search%') |
        u.email.like('%$search%')
      );
    }
    if (isActive != null) {
      query.where((u) => u.isActive.equals(isActive));
    }
    
    // Count
    final total = await query.count();
    
    // Results
    final items = await (query
      ..orderBy([(u) => OrderingTerm.asc(u.id)])
      ..limit(pageSize, offset: page * pageSize))
      .get();
    
    return PaginatedResult(
      items: items,
      total: total,
      page: page,
      pageSize: pageSize,
      hasMore: (page + 1) * pageSize < total,
    );
  }
  
  // ==================== PRODUCT PAGINATION ====================
  
  // 👇 Advanced product pagination
  Future<PaginatedResult<Product>> getProducts({
    required int page,
    int pageSize = 20,
    String? search,
    String? category,
    double? minPrice,
    double? maxPrice,
    bool? isActive,
    ProductSort sortBy = ProductSort.nameAsc,
  }) async {
    final query = db.select(db.products);
    
    // Apply filters
    if (search != null && search.isNotEmpty) {
      query.where((p) => 
        p.name.like('%$search%') |
        p.sku.like('%$search%') |
        p.description.like('%$search%')
      );
    }
    if (category != null && category.isNotEmpty) {
      query.where((p) => p.category.equals(category));
    }
    if (minPrice != null) {
      query.where((p) => p.price > const Variable(minPrice));
    }
    if (maxPrice != null) {
      query.where((p) => p.price < const Variable(maxPrice));
    }
    if (isActive != null) {
      query.where((p) => p.isActive.equals(isActive));
    }
    
    // Count
    final total = await query.count();
    
    // Sort
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
    }
    
    // Results
    final items = await (query
      ..limit(pageSize, offset: page * pageSize))
      .get();
    
    return PaginatedResult(
      items: items,
      total: total,
      page: page,
      pageSize: pageSize,
      hasMore: (page + 1) * pageSize < total,
    );
  }
  
  // ==================== ORDER PAGINATION ====================
  
  // 👇 Order pagination with user filter
  Future<PaginatedResult<Order>> getOrders({
    required int page,
    int pageSize = 10,
    int? userId,
    String? status,
    DateTime? fromDate,
    DateTime? toDate,
    OrderSort sortBy = OrderSort.newest,
  }) async {
    final query = db.select(db.orders);
    
    // Filters
    if (userId != null) {
      query.where((o) => o.userId.equals(userId));
    }
    if (status != null && status.isNotEmpty) {
      query.where((o) => o.status.equals(status));
    }
    if (fromDate != null) {
      query.where((o) => o.orderDate > const Variable(fromDate));
    }
    if (toDate != null) {
      query.where((o) => o.orderDate < const Variable(toDate));
    }
    
    // Count
    final total = await query.count();
    
    // Sort
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
    }
    
    // Results
    final items = await (query
      ..limit(pageSize, offset: page * pageSize))
      .get();
    
    return PaginatedResult(
      items: items,
      total: total,
      page: page,
      pageSize: pageSize,
      hasMore: (page + 1) * pageSize < total,
    );
  }
  
  // ==================== ORDER ITEMS PAGINATION ====================
  
  // 👇 Get order items with product details
  Future<PaginatedResult<OrderItemWithProduct>> getOrderItems({
    required int orderId,
    required int page,
    int pageSize = 10,
  }) async {
    final query = db.select(db.orderItems)
      ..where((i) => i.orderId.equals(orderId));
    
    final total = await query.count();
    
    final items = await (query
      ..orderBy([(i) => OrderingTerm.asc(i.productId)])
      ..limit(pageSize, offset: page * pageSize))
      .get();
    
    // Get product details for each item
    final itemsWithProducts = <OrderItemWithProduct>[];
    for (final item in items) {
      final product = await (db.select(db.products)
        ..where((p) => p.id.equals(item.productId)))
        .getSingle();
      
      itemsWithProducts.add(
        OrderItemWithProduct(orderItem: item, product: product),
      );
    }
    
    return PaginatedResult(
      items: itemsWithProducts,
      total: total,
      page: page,
      pageSize: pageSize,
      hasMore: (page + 1) * pageSize < total,
    );
  }
  
  // ==================== SEARCH PAGINATION ====================
  
  // 👇 Global search with pagination
  Future<PaginatedResult<SearchResult>> globalSearch({
    required String query,
    required int page,
    int pageSize = 20,
  }) async {
    if (query.isEmpty) {
      return PaginatedResult(
        items: [],
        total: 0,
        page: page,
        pageSize: pageSize,
        hasMore: false,
      );
    }
    
    final searchTerm = '%$query%';
    
    // Use UNION with LIMIT for pagination
    final results = await db.customSelect('''
      SELECT * FROM (
        SELECT 
          'user' as type,
          id,
          name,
          email as detail,
          created_at as date
        FROM users 
        WHERE name LIKE ? OR email LIKE ?
        
        UNION
        
        SELECT 
          'product' as type,
          id,
          name,
          sku as detail,
          created_at as date
        FROM products 
        WHERE name LIKE ? OR sku LIKE ?
        
        UNION
        
        SELECT 
          'order' as type,
          id,
          order_number as name,
          CAST(total AS TEXT) as detail,
          order_date as date
        FROM orders 
        WHERE order_number LIKE ?
      )
      ORDER BY date DESC
      LIMIT ? OFFSET ?
    ''', variables: [
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
      Variable.withInt(pageSize),
      Variable.withInt(page * pageSize),
    ]).get();
    
    // Get total count (separate query)
    final countResult = await db.customSelect('''
      SELECT COUNT(*) as total FROM (
        SELECT id FROM users WHERE name LIKE ? OR email LIKE ?
        UNION
        SELECT id FROM products WHERE name LIKE ? OR sku LIKE ?
        UNION
        SELECT id FROM orders WHERE order_number LIKE ?
      )
    ''', variables: [
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
      Variable.withString(searchTerm),
    ]).get();
    
    final total = countResult.first.data['total'] as int;
    
    final items = results.map((row) {
      return SearchResult(
        type: row.data['type'] as String,
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        detail: row.data['detail'] as String?,
        date: DateTime.fromMillisecondsSinceEpoch(row.data['date'] as int),
      );
    }).toList();
    
    return PaginatedResult(
      items: items,
      total: total,
      page: page,
      pageSize: pageSize,
      hasMore: (page + 1) * pageSize < total,
    );
  }
}

// ==================== ENUMS ====================

enum ProductSort {
  nameAsc,
  nameDesc,
  priceAsc,
  priceDesc,
  popularity,
}

enum OrderSort {
  newest,
  oldest,
  highestTotal,
  lowestTotal,
}

// ==================== DATA CLASSES ====================

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
  
  int get totalPages => (total / pageSize).ceil();
  
  bool get isFirstPage => page == 0;
  
  bool get isLastPage => !hasMore;
  
  String get summary => 'Showing ${items.length} of $total items (Page ${page + 1}/$totalPages)';
}

class OrderItemWithProduct {
  final OrderItem orderItem;
  final Product product;
  
  OrderItemWithProduct({
    required this.orderItem,
    required this.product,
  });
}

class SearchResult {
  final String type;
  final int id;
  final String name;
  final String? detail;
  final DateTime date;
  
  SearchResult({
    required this.type,
    required this.id,
    required this.name,
    this.detail,
    required this.date,
  });
}
```

```dart
// lib/ui/widgets/paginated_list.dart
class PaginatedList<T> extends StatefulWidget {
  final Future<PaginatedResult<T>> Function(int page) fetchItems;
  final Widget Function(T item) itemBuilder;
  final Widget? emptyWidget;
  final Widget? loadingWidget;
  final int pageSize;
  
  const PaginatedList({
    required this.fetchItems,
    required this.itemBuilder,
    this.emptyWidget,
    this.loadingWidget,
    this.pageSize = 20,
  });
  
  @override
  _PaginatedListState createState() => _PaginatedListState();
}

class _PaginatedListState<T> extends State<PaginatedList<T>> {
  List<T> _items = [];
  bool _isLoading = true;
  bool _hasMore = true;
  int _page = 0;
  final ScrollController _scrollController = ScrollController();
  
  @override
  void initState() {
    super.initState();
    _loadMore();
    _scrollController.addListener(_onScroll);
  }
  
  void _onScroll() {
    if (_scrollController.position.pixels >= 
        _scrollController.position.maxScrollExtent - 200) {
      if (!_isLoading && _hasMore) {
        _loadMore();
      }
    }
  }
  
  Future<void> _loadMore() async {
    setState(() => _isLoading = true);
    
    try {
      final result = await widget.fetchItems(_page);
      
      setState(() {
        _items.addAll(result.items);
        _hasMore = result.hasMore;
        _page++;
        _isLoading = false;
      });
    } catch (e) {
      setState(() => _isLoading = false);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    if (_isLoading && _items.isEmpty) {
      return widget.loadingWidget ?? 
          Center(child: CircularProgressIndicator());
    }
    
    if (_items.isEmpty) {
      return widget.emptyWidget ?? 
          Center(child: Text('No items found'));
    }
    
    return ListView.builder(
      controller: _scrollController,
      itemCount: _items.length + (_hasMore ? 1 : 0),
      itemBuilder: (context, index) {
        if (index == _items.length) {
          return Padding(
            padding: EdgeInsets.all(16),
            child: Center(child: CircularProgressIndicator()),
          );
        }
        
        return widget.itemBuilder(_items[index]);
      },
    );
  }
  
  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }
}

// Usage
class UserListScreen extends StatelessWidget {
  final PaginationService paginationService;
  
  const UserListScreen({required this.paginationService});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Users')),
      body: PaginatedList<User>(
        fetchItems: (page) => paginationService.getUsers(
          page: page,
          pageSize: 20,
          isActive: true,
        ),
        itemBuilder: (user) => ListTile(
          title: Text(user.name),
          subtitle: Text(user.email),
          trailing: Text(user.isActive ? 'Active' : 'Inactive'),
        ),
      ),
    );
  }
}
```

---

# Best Practices

- **Always use limit with large tables** – For performance
- **Use ordering with pagination** – Consistent results
- **Count total for pagination info** – Show total pages
- **Use cursor-based pagination** – For large datasets
- **Implement infinite scroll** – For better UX
- **Cache results** – For frequently accessed pages
- **Use indexes** – On paginated/sorted columns
- **Test performance** – With realistic data sizes

---

# Common Mistakes

## Mistake 1: Loading all records

Wrong:
```dart
// 🚫 Loads all records (performance disaster)
final allUsers = await select(users).get();
```

Correct:
```dart
// ✅ Load only what you need
final users = await (select(users)
  ..limit(100))
  .get();
```

## Mistake 2: Pagination without ordering

Wrong:
```dart
// 🚫 Unpredictable results
final page = await (select(users)
  ..limit(10, offset: 10))
  .get();
```

Correct:
```dart
// ✅ Always order for consistent pagination
final page = await (select(users)
  ..orderBy([(u) => OrderingTerm.asc(u.id)])
  ..limit(10, offset: 10))
  .get();
```

## Mistake 3: Not counting total

Wrong:
```dart
// 🚫 No total count
final items = await query.get(); // Unknown total
```

Correct:
```dart
// ✅ Get total for pagination
final total = await query.count();
final items = await (query..limit(10)).get();
```

---

# Summary

| Pattern | Method | Use Case |
|---------|--------|----------|
| **Simple Limit** | `limit(10)` | First N records |
| **Offset Pagination** | `limit(10, offset: 20)` | Page 3 |
| **Cursor Pagination** | `where(id > cursor)` | Infinite scroll |
| **Count + Limit** | `count()` + `limit()` | With total |

---

# Next Steps

Now you understand limiting, let's dive deeper:

- [Distinct](link) – Getting unique values
- [Aliases](link) – Column aliases
- [Expressions](link) – Custom expressions

---

# Did You Know?

- **Limit without order can be inconsistent** – Always order

- **Large offsets can be slow** – Use cursor pagination

- **Count queries are optimized** – Don't fetch all rows

- **Limit is applied after WHERE and ORDER** – For efficiency

- **SQLite uses indexes for LIMIT** – With proper indexing

- **Limit can be combined with JOINs** – For paginated joins

- **Pagination is essential for APIs** – REST/GraphQL endpoints

- **Infinite scroll is common** – Mobile and web apps

---
