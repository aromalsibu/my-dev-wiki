## Dynamic Queries

**Building flexible queries at runtime in Drift**

---

# What is it?

**Dynamic Queries** are queries built at runtime based on conditions, user input, or application state. Instead of hardcoding WHERE clauses, ORDER BY, or LIMIT, you construct them dynamically using Drift's flexible query builder. This is essential for search functionality, filters, and user-configurable views.

> **Think of Dynamic Queries like "building a custom sandwich"** – you start with a base (SELECT), then add ingredients (conditions) based on what the customer wants, resulting in a unique creation every time.

```dart
// 👇 Building a query dynamically
Future<List<User>> searchUsers({
  String? name,
  int? minAge,
  int? maxAge,
  bool? isActive,
  String? sortBy,
  bool ascending = true,
  int? limit,
}) async {
  // 👇 Start with a base query
  final query = db.select(db.users);
  
  // 👇 Add filters dynamically
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
  
  // 👇 Add sorting dynamically
  if (sortBy != null) {
    final mode = ascending ? OrderingMode.asc : OrderingMode.desc;
    switch (sortBy) {
      case 'name':
        query.orderBy([(u) => OrderingTerm(expression: u.name, mode: mode)]);
        break;
      case 'age':
        query.orderBy([(u) => OrderingTerm(expression: u.age, mode: mode)]);
        break;
      case 'createdAt':
        query.orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: mode)]);
        break;
    }
  }
  
  // 👇 Add limit dynamically
  if (limit != null) {
    query.limit(limit);
  }
  
  return await query.get();
}
```

> **What's happening here?**
> - **Base query** – Start with `select()`
> - **Conditional building** – Add parts based on conditions
> - **Chained modifications** – `where()`, `orderBy()`, `limit()`
> - **Runtime decisions** – Query structure changes at runtime

---

# Why does it exist?

- **Search/Filter** – User-configurable search
- **User Preferences** – Custom sorting options
- **Pagination** – Different page sizes
- **Reporting** – User-defined reports
- **API Integration** – Dynamic API endpoints
- **Admin Panels** – Configurable views

---

# Basic Dynamic Queries

> **Simple dynamic query patterns**

## Dynamic WHERE Clauses

```dart
Future<List<User>> dynamicWhere({
  String? name,
  int? minAge,
  int? maxAge,
  bool? isActive,
}) async {
  final query = db.select(db.users);
  
  // 👇 Add WHERE clauses conditionally
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
  
  return await query.get();
}

// Usage
final youngUsers = await dynamicWhere(maxAge: 30);
final activeUsers = await dynamicWhere(isActive: true);
final johnUsers = await dynamicWhere(name: 'John');
```

## Dynamic ORDER BY

```dart
Future<List<User>> dynamicOrderBy({
  String? sortBy,
  bool ascending = true,
}) async {
  final query = db.select(db.users);
  
  // 👇 Build ORDER BY dynamically
  if (sortBy != null) {
    final mode = ascending ? OrderingMode.asc : OrderingMode.desc;
    
    switch (sortBy) {
      case 'name':
        query.orderBy([(u) => OrderingTerm(expression: u.name, mode: mode)]);
        break;
      case 'age':
        query.orderBy([(u) => OrderingTerm(expression: u.age, mode: mode)]);
        break;
      case 'createdAt':
        query.orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: mode)]);
        break;
      default:
        query.orderBy([(u) => OrderingTerm(expression: u.id, mode: mode)]);
    }
  }
  
  return await query.get();
}
```

## Dynamic LIMIT and OFFSET

```dart
Future<List<User>> dynamicPagination({
  int page = 0,
  int pageSize = 20,
}) async {
  final query = db.select(db.users);
  
  // 👇 Add pagination dynamically
  query
    ..orderBy([(u) => OrderingTerm(expression: u.id)])
    ..limit(pageSize, offset: page * pageSize);
  
  return await query.get();
}
```

---

# Advanced Dynamic Queries

> **Complex dynamic query patterns**

## Pattern 1: Search with Multiple Filters

```dart
class SearchCriteria {
  String? name;
  String? email;
  int? minAge;
  int? maxAge;
  bool? isActive;
  bool? isVerified;
  String? status;
  DateTime? fromDate;
  DateTime? toDate;
  List<int>? excludeIds;
  String? sortBy;
  bool ascending = true;
  int? limit;
  int? offset;
}

Future<List<User>> advancedSearch(SearchCriteria criteria) async {
  final query = db.select(db.users);
  
  // 👇 Apply all filters dynamically
  if (criteria.name != null && criteria.name!.isNotEmpty) {
    query.where((u) => u.name.like('%${criteria.name}%'));
  }
  
  if (criteria.email != null && criteria.email!.isNotEmpty) {
    query.where((u) => u.email.like('%${criteria.email}%'));
  }
  
  if (criteria.minAge != null) {
    query.where((u) => u.age > const Variable(criteria.minAge!));
  }
  
  if (criteria.maxAge != null) {
    query.where((u) => u.age < const Variable(criteria.maxAge!));
  }
  
  if (criteria.isActive != null) {
    query.where((u) => u.isActive.equals(criteria.isActive!));
  }
  
  if (criteria.isVerified != null) {
    query.where((u) => u.isVerified.equals(criteria.isVerified!));
  }
  
  if (criteria.status != null && criteria.status!.isNotEmpty) {
    query.where((u) => u.status.equals(criteria.status!));
  }
  
  if (criteria.fromDate != null) {
    query.where((u) => u.createdAt > const Variable(criteria.fromDate!));
  }
  
  if (criteria.toDate != null) {
    query.where((u) => u.createdAt < const Variable(criteria.toDate!));
  }
  
  if (criteria.excludeIds != null && criteria.excludeIds!.isNotEmpty) {
    query.where((u) => u.id.isNotIn(criteria.excludeIds!));
  }
  
  // 👇 Dynamic sorting
  if (criteria.sortBy != null) {
    final mode = criteria.ascending ? OrderingMode.asc : OrderingMode.desc;
    switch (criteria.sortBy) {
      case 'name':
        query.orderBy([(u) => OrderingTerm(expression: u.name, mode: mode)]);
        break;
      case 'age':
        query.orderBy([(u) => OrderingTerm(expression: u.age, mode: mode)]);
        break;
      case 'createdAt':
        query.orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: mode)]);
        break;
    }
  }
  
  // 👇 Pagination
  if (criteria.limit != null) {
    query.limit(criteria.limit!, offset: criteria.offset ?? 0);
  }
  
  return await query.get();
}
```

---

## Pattern 2: Dynamic Query Builder Class

```dart
class DynamicQueryBuilder<T extends Table, D> {
  final Selectable<T, D> baseQuery;
  final List<Function> _filters = [];
  final List<Function> _orderBy = [];
  int? _limit;
  int? _offset;
  
  DynamicQueryBuilder(this.baseQuery);
  
  // 👇 Add filter
  DynamicQueryBuilder<T, D> where(Function condition) {
    _filters.add(condition);
    return this;
  }
  
  // 👇 Add ORDER BY
  DynamicQueryBuilder<T, D> orderBy(Function order) {
    _orderBy.add(order);
    return this;
  }
  
  // 👇 Add LIMIT
  DynamicQueryBuilder<T, D> limit(int limit, {int? offset}) {
    _limit = limit;
    _offset = offset;
    return this;
  }
  
  // 👇 Build and execute
  Future<List<D>> get() async {
    var query = baseQuery;
    
    for (final filter in _filters) {
      query = query..where(filter);
    }
    
    for (final order in _orderBy) {
      query = query..orderBy([order]);
    }
    
    if (_limit != null) {
      query = query..limit(_limit!, offset: _offset ?? 0);
    }
    
    return await query.get();
  }
  
  // 👇 Get count
  Future<int> count() async {
    var query = baseQuery;
    
    for (final filter in _filters) {
      query = query..where(filter);
    }
    
    return await query.count();
  }
}

// Usage
final builder = DynamicQueryBuilder(db.select(db.users))
  .where((u) => u.isActive.equals(true))
  .where((u) => u.age > const Variable(18))
  .orderBy((u) => OrderingTerm.desc(u.createdAt))
  .limit(20);

final users = await builder.get();
final total = await builder.count();
```

---

## Pattern 3: Dynamic Join Queries

```dart
Future<List<OrderWithUser>> dynamicOrderSearch({
  int? userId,
  String? status,
  double? minTotal,
  double? maxTotal,
  DateTime? fromDate,
  DateTime? toDate,
  String? sortBy,
  bool ascending = true,
}) async {
  // 👇 Build dynamic query with joins
  final query = db.select(db.orders).join([
    innerJoin(db.users, db.users.id.equals(db.orders.userId)),
  ]);
  
  // 👇 Dynamic filters
  if (userId != null) {
    query.where((o) => db.orders.userId.equals(userId));
  }
  
  if (status != null && status.isNotEmpty) {
    query.where((o) => db.orders.status.equals(status));
  }
  
  if (minTotal != null) {
    query.where((o) => db.orders.total > const Variable(minTotal));
  }
  
  if (maxTotal != null) {
    query.where((o) => db.orders.total < const Variable(maxTotal));
  }
  
  if (fromDate != null) {
    query.where((o) => db.orders.orderDate > const Variable(fromDate));
  }
  
  if (toDate != null) {
    query.where((o) => db.orders.orderDate < const Variable(toDate));
  }
  
  // 👇 Dynamic sorting
  if (sortBy != null) {
    final mode = ascending ? OrderingMode.asc : OrderingMode.desc;
    switch (sortBy) {
      case 'orderDate':
        query.orderBy([(o) => OrderingTerm(expression: db.orders.orderDate, mode: mode)]);
        break;
      case 'total':
        query.orderBy([(o) => OrderingTerm(expression: db.orders.total, mode: mode)]);
        break;
    }
  }
  
  final results = await query.get();
  
  // 👇 Map results
  return results.map((row) {
    return OrderWithUser(
      order: row.readTable(db.orders),
      user: row.readTable(db.users),
    );
  }).toList();
}
```

---

# Real-World Example

> **Complete e-commerce dynamic query system**

```dart
// lib/database/dynamic_query_service.dart
import 'package:drift/drift.dart';

class DynamicQueryService {
  final AppDatabase db;
  
  DynamicQueryService(this.db);

  // ==================== USER DYNAMIC QUERIES ====================
  
  // 👇 Complete dynamic user search
  Future<PaginatedResult<User>> dynamicUserSearch({
    String? search,
    int? minAge,
    int? maxAge,
    bool? isActive,
    bool? isVerified,
    String? status,
    DateTime? fromDate,
    DateTime? toDate,
    String? sortBy,
    bool ascending = true,
    int page = 0,
    int pageSize = 20,
  }) async {
    final query = db.select(db.users);
    
    // 👇 Apply search filter
    if (search != null && search.isNotEmpty) {
      query.where((u) => 
        u.name.like('%$search%') |
        u.email.like('%$search%')
      );
    }
    
    // 👇 Age filter
    if (minAge != null) {
      query.where((u) => u.age > const Variable(minAge));
    }
    if (maxAge != null) {
      query.where((u) => u.age < const Variable(maxAge));
    }
    
    // 👇 Boolean filters
    if (isActive != null) {
      query.where((u) => u.isActive.equals(isActive));
    }
    if (isVerified != null) {
      query.where((u) => u.isVerified.equals(isVerified));
    }
    
    // 👇 Status filter
    if (status != null && status.isNotEmpty) {
      query.where((u) => u.status.equals(status));
    }
    
    // 👇 Date range
    if (fromDate != null) {
      query.where((u) => u.createdAt > const Variable(fromDate));
    }
    if (toDate != null) {
      query.where((u) => u.createdAt < const Variable(toDate));
    }
    
    // 👇 Get total count
    final total = await query.count();
    
    // 👇 Dynamic sorting
    if (sortBy != null) {
      final mode = ascending ? OrderingMode.asc : OrderingMode.desc;
      switch (sortBy) {
        case 'name':
          query.orderBy([(u) => OrderingTerm(expression: u.name, mode: mode)]);
          break;
        case 'age':
          query.orderBy([(u) => OrderingTerm(expression: u.age, mode: mode)]);
          break;
        case 'createdAt':
          query.orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: mode)]);
          break;
        case 'status':
          query.orderBy([(u) => OrderingTerm(expression: u.status, mode: mode)]);
          break;
        default:
          query.orderBy([(u) => OrderingTerm(expression: u.id, mode: mode)]);
      }
    }
    
    // 👇 Pagination
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

  // ==================== PRODUCT DYNAMIC QUERIES ====================
  
  // 👇 Dynamic product search with filters
  Future<PaginatedResult<Product>> dynamicProductSearch({
    String? search,
    String? category,
    double? minPrice,
    double? maxPrice,
    int? minStock,
    int? maxStock,
    bool? isActive,
    List<String>? tags,
    String? sortBy,
    bool ascending = true,
    int page = 0,
    int pageSize = 20,
  }) async {
    final query = db.select(db.products);
    
    // 👇 Search
    if (search != null && search.isNotEmpty) {
      query.where((p) => 
        p.name.like('%$search%') |
        p.sku.like('%$search%') |
        p.description.like('%$search%')
      );
    }
    
    // 👇 Category
    if (category != null && category.isNotEmpty) {
      query.where((p) => p.category.equals(category));
    }
    
    // 👇 Price range
    if (minPrice != null) {
      query.where((p) => p.price > const Variable(minPrice));
    }
    if (maxPrice != null) {
      query.where((p) => p.price < const Variable(maxPrice));
    }
    
    // 👇 Stock range
    if (minStock != null) {
      query.where((p) => p.stock > const Variable(minStock));
    }
    if (maxStock != null) {
      query.where((p) => p.stock < const Variable(maxStock));
    }
    
    // 👇 Active status
    if (isActive != null) {
      query.where((p) => p.isActive.equals(isActive));
    }
    
    // 👇 Tags
    if (tags != null && tags.isNotEmpty) {
      for (final tag in tags) {
        query.where((p) => p.tags.like('%$tag%'));
      }
    }
    
    // 👇 Count
    final total = await query.count();
    
    // 👇 Sorting
    if (sortBy != null) {
      final mode = ascending ? OrderingMode.asc : OrderingMode.desc;
      switch (sortBy) {
        case 'name':
          query.orderBy([(p) => OrderingTerm(expression: p.name, mode: mode)]);
          break;
        case 'price':
          query.orderBy([(p) => OrderingTerm(expression: p.price, mode: mode)]);
          break;
        case 'stock':
          query.orderBy([(p) => OrderingTerm(expression: p.stock, mode: mode)]);
          break;
        case 'createdAt':
          query.orderBy([(p) => OrderingTerm(expression: p.createdAt, mode: mode)]);
          break;
        default:
          query.orderBy([(p) => OrderingTerm(expression: p.id, mode: mode)]);
      }
    }
    
    // 👇 Pagination
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

  // ==================== ORDER DYNAMIC QUERIES ====================
  
  // 👇 Dynamic order search
  Future<PaginatedResult<Order>> dynamicOrderSearch({
    int? userId,
    String? status,
    String? paymentStatus,
    double? minTotal,
    double? maxTotal,
    DateTime? fromDate,
    DateTime? toDate,
    bool? isPaid,
    bool? isShipped,
    bool? isDelivered,
    String? sortBy,
    bool ascending = true,
    int page = 0,
    int pageSize = 20,
  }) async {
    final query = db.select(db.orders);
    
    // 👇 Filters
    if (userId != null) {
      query.where((o) => o.userId.equals(userId));
    }
    if (status != null && status.isNotEmpty) {
      query.where((o) => o.status.equals(status));
    }
    if (paymentStatus != null && paymentStatus.isNotEmpty) {
      query.where((o) => o.paymentStatus.equals(paymentStatus));
    }
    if (minTotal != null) {
      query.where((o) => o.total > const Variable(minTotal));
    }
    if (maxTotal != null) {
      query.where((o) => o.total < const Variable(maxTotal));
    }
    if (fromDate != null) {
      query.where((o) => o.orderDate > const Variable(fromDate));
    }
    if (toDate != null) {
      query.where((o) => o.orderDate < const Variable(toDate));
    }
    if (isPaid != null) {
      query.where((o) => o.isPaid.equals(isPaid));
    }
    if (isShipped != null) {
      query.where((o) => o.isShipped.equals(isShipped));
    }
    if (isDelivered != null) {
      query.where((o) => o.isDelivered.equals(isDelivered));
    }
    
    final total = await query.count();
    
    // 👇 Sorting
    if (sortBy != null) {
      final mode = ascending ? OrderingMode.asc : OrderingMode.desc;
      switch (sortBy) {
        case 'orderDate':
          query.orderBy([(o) => OrderingTerm(expression: o.orderDate, mode: mode)]);
          break;
        case 'total':
          query.orderBy([(o) => OrderingTerm(expression: o.total, mode: mode)]);
          break;
        case 'status':
          query.orderBy([(o) => OrderingTerm(expression: o.status, mode: mode)]);
          break;
        default:
          query.orderBy([(o) => OrderingTerm(expression: o.id, mode: mode)]);
      }
    }
    
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

  // ==================== ADVANCED DYNAMIC QUERIES ====================
  
  // 👇 Build query from configuration
  Future<List> dynamicQueryFromConfig(DynamicQueryConfig config) async {
    if (config.queryType == 'users') {
      return await _dynamicUserQuery(config);
    } else if (config.queryType == 'products') {
      return await _dynamicProductQuery(config);
    } else if (config.queryType == 'orders') {
      return await _dynamicOrderQuery(config);
    }
    return [];
  }
  
  Future<List> _dynamicUserQuery(DynamicQueryConfig config) async {
    final query = db.select(db.users);
    
    // Apply all configured filters
    for (final filter in config.filters) {
      switch (filter.field) {
        case 'name':
          query.where((u) => u.name.like('%${filter.value}%'));
          break;
        case 'age':
          final age = int.tryParse(filter.value);
          if (age != null) {
            if (filter.operator == '>') {
              query.where((u) => u.age > const Variable(age));
            } else if (filter.operator == '<') {
              query.where((u) => u.age < const Variable(age));
            } else {
              query.where((u) => u.age.equals(age));
            }
          }
          break;
        case 'isActive':
          final boolValue = filter.value.toLowerCase() == 'true';
          query.where((u) => u.isActive.equals(boolValue));
          break;
      }
    }
    
    // Apply sorting
    if (config.sortBy != null) {
      final mode = config.ascending ? OrderingMode.asc : OrderingMode.desc;
      query.orderBy([(u) => 
        OrderingTerm(expression: u.id, mode: mode)
      ]);
    }
    
    // Apply limit
    if (config.limit != null) {
      query.limit(config.limit!, offset: config.offset ?? 0);
    }
    
    return await query.get();
  }
  
  Future<List> _dynamicProductQuery(DynamicQueryConfig config) async {
    // Similar dynamic building for products
    final query = db.select(db.products);
    // ... apply filters from config
    return await query.get();
  }
  
  Future<List> _dynamicOrderQuery(DynamicQueryConfig config) async {
    // Similar dynamic building for orders
    final query = db.select(db.orders);
    // ... apply filters from config
    return await query.get();
  }

  // ==================== DYNAMIC FILTER PRESETS ====================
  
  // 👇 Pre-configured filters
  Future<List<User>> getUsersByPreset(String preset) async {
    switch (preset) {
      case 'active_users':
        return await dynamicUserSearch(isActive: true).then((r) => r.items);
      case 'verified_users':
        return await dynamicUserSearch(isVerified: true).then((r) => r.items);
      case 'new_users':
        return await dynamicUserSearch(sortBy: 'createdAt', ascending: false, pageSize: 10).then((r) => r.items);
      case 'inactive_users':
        return await dynamicUserSearch(isActive: false).then((r) => r.items);
      default:
        return await db.select(db.users).get();
    }
  }
}

// ==================== DATA CLASSES ====================

class DynamicQueryConfig {
  final String queryType;
  final List<Filter> filters;
  final String? sortBy;
  final bool ascending;
  final int? limit;
  final int? offset;
  
  DynamicQueryConfig({
    required this.queryType,
    this.filters = const [],
    this.sortBy,
    this.ascending = true,
    this.limit,
    this.offset,
  });
}

class Filter {
  final String field;
  final String operator;
  final String value;
  
  Filter({
    required this.field,
    required this.operator,
    required this.value,
  });
}

class OrderWithUser {
  final Order order;
  final User user;
  
  OrderWithUser({
    required this.order,
    required this.user,
  });
}
```

```dart
// lib/ui/pages/advanced_search_page.dart
class AdvancedSearchPage extends StatefulWidget {
  final DynamicQueryService queryService;
  
  const AdvancedSearchPage({required this.queryService});
  
  @override
  _AdvancedSearchPageState createState() => _AdvancedSearchPageState();
}

class _AdvancedSearchPageState extends State<AdvancedSearchPage> {
  final _searchController = TextEditingController();
  int? _minAge;
  int? _maxAge;
  bool? _isActive;
  bool? _isVerified;
  String? _sortBy;
  bool _ascending = true;
  int _currentPage = 0;
  final int _pageSize = 20;
  
  List<User> _users = [];
  bool _isLoading = false;
  int _total = 0;
  
  @override
  void initState() {
    super.initState();
    _loadUsers();
  }
  
  Future<void> _loadUsers() async {
    setState(() => _isLoading = true);
    
    try {
      final result = await widget.queryService.dynamicUserSearch(
        search: _searchController.text,
        minAge: _minAge,
        maxAge: _maxAge,
        isActive: _isActive,
        isVerified: _isVerified,
        sortBy: _sortBy,
        ascending: _ascending,
        page: _currentPage,
        pageSize: _pageSize,
      );
      
      setState(() {
        _users = result.items;
        _total = result.total;
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
        title: Text('Advanced Search'),
        actions: [
          IconButton(
            icon: Icon(Icons.refresh),
            onPressed: _loadUsers,
          ),
        ],
      ),
      body: Column(
        children: [
          // Search bar
          Padding(
            padding: EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _searchController,
                    decoration: InputDecoration(
                      hintText: 'Search users...',
                      border: OutlineInputBorder(),
                    ),
                    onSubmitted: (_) => _loadUsers(),
                  ),
                ),
                SizedBox(width: 8),
                ElevatedButton(
                  onPressed: _loadUsers,
                  child: Text('Search'),
                ),
              ],
            ),
          ),
          
          // Filters
          Container(
            padding: EdgeInsets.all(16),
            color: Colors.grey[100],
            child: Column(
              children: [
                Row(
                  children: [
                    Expanded(
                      child: TextFormField(
                        decoration: InputDecoration(labelText: 'Min Age'),
                        keyboardType: TextInputType.number,
                        onChanged: (v) => _minAge = int.tryParse(v),
                      ),
                    ),
                    SizedBox(width: 16),
                    Expanded(
                      child: TextFormField(
                        decoration: InputDecoration(labelText: 'Max Age'),
                        keyboardType: TextInputType.number,
                        onChanged: (v) => _maxAge = int.tryParse(v),
                      ),
                    ),
                  ],
                ),
                Row(
                  children: [
                    Expanded(
                      child: DropdownButton<String>(
                        isExpanded: true,
                        hint: Text('Sort By'),
                        value: _sortBy,
                        items: ['name', 'age', 'createdAt'].map((s) {
                          return DropdownMenuItem(
                            value: s,
                            child: Text(s),
                          );
                        }).toList(),
                        onChanged: (v) {
                          setState(() => _sortBy = v);
                          _loadUsers();
                        },
                      ),
                    ),
                    SizedBox(width: 16),
                    Expanded(
                      child: DropdownButton<bool>(
                        isExpanded: true,
                        value: _ascending,
                        items: [
                          DropdownMenuItem(value: true, child: Text('Ascending')),
                          DropdownMenuItem(value: false, child: Text('Descending')),
                        ],
                        onChanged: (v) {
                          setState(() => _ascending = v ?? true);
                          _loadUsers();
                        },
                      ),
                    ),
                  ],
                ),
              ],
            ),
          ),
          
          // Results
          Expanded(
            child: _isLoading
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
                            if (!user.isActive)
                              Icon(Icons.cancel, color: Colors.red, size: 20),
                            if (user.isVerified)
                              Icon(Icons.verified, color: Colors.blue, size: 16),
                          ],
                        ),
                      );
                    },
                  ),
          ),
          
          // Pagination
          Container(
            padding: EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text('Total: $_total users'),
                Row(
                  children: [
                    IconButton(
                      icon: Icon(Icons.chevron_left),
                      onPressed: _currentPage > 0
                          ? () {
                              setState(() => _currentPage--);
                              _loadUsers();
                            }
                          : null,
                    ),
                    Text('Page ${_currentPage + 1}'),
                    IconButton(
                      icon: Icon(Icons.chevron_right),
                      onPressed: (_currentPage + 1) * _pageSize < _total
                          ? () {
                              setState(() => _currentPage++);
                              _loadUsers();
                            }
                          : null,
                    ),
                  ],
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

---

# Best Practices

- **Start with a base query** – Then add conditions
- **Use meaningful parameter names** – For dynamic values
- **Document query building logic** – Make it maintainable
- **Test all combinations** – Ensure correctness
- **Use indexes** – On frequently filtered columns
- **Limit result size** – Prevent performance issues
- **Use caching** – For frequently run queries
- **Log dynamic queries** – For debugging

---

# Common Mistakes

## Mistake 1: Building queries in loops

Wrong:
```dart
// 🚫 Inefficient query building
for (final filter in filters) {
  query = query..where((u) => u.name.equals(filter));
}
```

Correct:
```dart
// ✅ Build conditions once
for (final filter in filters) {
  query.where((u) => u.name.equals(filter));
}
```

## Mistake 2: Not handling empty conditions

Wrong:
```dart
// 🚫 Empty search returns everything
final query = db.select(db.users);
if (search != null) {
  query.where((u) => u.name.like('%$search%'));
}
```

Correct:
```dart
// ✅ Handle empty search
final query = db.select(db.users);
if (search != null && search.isNotEmpty) {
  query.where((u) => u.name.like('%$search%'));
}
```

## Mistake 3: Forgetting to add sorting with pagination

Wrong:
```dart
// 🚫 Unpredictable pagination
query.limit(pageSize, offset: page * pageSize);
```

Correct:
```dart
// ✅ Always order for pagination
query
  ..orderBy([(u) => OrderingTerm.asc(u.id)])
  ..limit(pageSize, offset: page * pageSize);
```

---

# Summary

| Pattern | Purpose | Example |
|---------|---------|---------|
| **Conditional WHERE** | Dynamic filters | `if (name != null) query.where(...)` |
| **Dynamic ORDER BY** | Custom sorting | `switch(sortBy) { ... }` |
| **Pagination** | Page navigation | `limit(pageSize, offset: page * pageSize)` |
| **Search** | Text search | `name.like('%$search%')` |
| **Builder Pattern** | Complex queries | `DynamicQueryBuilder` |

---

# Next Steps

Now you understand dynamic queries, let's dive deeper:

- [Joins](link) – Table joins
- [Views](link) – Database views
- [Indexes](link) – Performance optimization

---

# Did You Know?

- **Dynamic queries are essential for search** – User-driven search

- **Query builder is chainable** – For easy dynamic building

- **Conditions are evaluated at query time** – Not build time

- **Dynamic queries can be cached** – For performance

- **Query complexity grows with conditions** – Monitor performance

- **Indexes help dynamic queries** – On filtered columns

- **Dynamic queries can use subqueries** – For complex logic

- **Dynamic queries are type-safe** – Compile-time validation

---
