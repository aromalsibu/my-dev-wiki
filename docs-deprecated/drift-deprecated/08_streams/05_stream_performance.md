## Stream Performance

**Optimizing Drift stream queries for maximum efficiency**

---

# What is it?

**Stream Performance** refers to techniques and best practices for ensuring your reactive queries run efficiently. While Drift streams are powerful, poorly optimized streams can cause performance issues like UI jank, excessive database queries, and high memory usage. Understanding how to optimize streams is crucial for building responsive applications.

> **Think of Stream Performance like "fuel efficiency"** – a well-tuned engine (optimized stream) gives you better mileage (performance) without wasting resources (CPU, memory, database queries).

```dart
// 👇 Inefficient stream (does too much work)
final slowStream = select(users)
  .watch()
  .map((users) {
    // Heavy computation on every change
    return users.map((u) => expensiveTransform(u)).toList();
  });

// 👇 Optimized stream (does only necessary work)
final fastStream = select(users)
  .watch()
  .distinct()
  .debounce(Duration(milliseconds: 300))
  .map((users) {
    // Lightweight transformation
    return users.map((u) => lightweightTransform(u)).toList();
  });
```

> **What's happening here?**
> - **`.distinct()`** – Prevents duplicate emissions
> - **`.debounce()`** – Batches rapid changes
> - **Lightweight transforms** – Minimize processing
> - **Efficient queries** – Database-level optimization

---

# Why does it exist?

- **UI Performance** – Prevent jank and lag
- **Resource Efficiency** – Reduce CPU and memory usage
- **Database Load** – Minimize database queries
- **User Experience** – Responsive interfaces
- **Scalability** – Handle large datasets
- **Battery Life** – Reduce power consumption

---

# Optimization Techniques

> **Strategies for optimizing streams**

## 1. Use Distinct to Avoid Duplicate Emissions

```dart
// 👇 Inefficient - emits on every change
final stream = select(users).watch();

// 👇 Efficient - only emits when data actually changes
final efficientStream = select(users)
  .watch()
  .distinct();
```

## 2. Use Debounce for Rapid Changes

```dart
// 👇 Inefficient - floods UI with updates
final searchStream = searchController.stream
  .switchMap((query) {
    return (select(users)
      ..where((u) => u.name.like('%$query%')))
      .watch();
  });

// 👇 Efficient - waits for typing to pause
final optimizedSearch = searchController.stream
  .debounce(Duration(milliseconds: 300))
  .switchMap((query) {
    if (query.isEmpty) return Stream.value([]);
    return (select(users)
      ..where((u) => u.name.like('%$query%')))
      .watch();
  });
```

## 3. Use Throttle for High-Frequency Updates

```dart
// 👇 Inefficient - processes every update
final stream = select(orders)
  .watch()
  .map((orders) => calculateStats(orders));

// 👇 Efficient - limits updates
final throttledStream = select(orders)
  .watch()
  .throttle(Duration(milliseconds: 500))
  .map((orders) => calculateStats(orders));
```

## 4. Optimize Query Filtering

```dart
// 👇 Inefficient - loads all data, filters in Dart
final stream = select(orders)
  .watch()
  .map((orders) => orders.where((o) => o.status == 'pending').toList());

// 👇 Efficient - filters at database level
final efficientStream = (select(orders)
  ..where((o) => o.status.equals('pending')))
  .watch();
```

## 5. Limit Result Sets

```dart
// 👇 Inefficient - loads all records
final stream = select(orders).watch();

// 👇 Efficient - only loads recent orders
final efficientStream = (select(orders)
  ..orderBy([(o) => OrderingTerm.desc(o.orderDate)])
  ..limit(100))
  .watch();
```

---

# Advanced Optimization

> **Deep-dive performance techniques**

## Indexing for Stream Queries

```dart
// 👇 Add indexes for frequently watched queries
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get status => text()();
  IntColumn get userId => integer()();
  DateTimeColumn get orderDate => dateTime()();
  
  @override
  List<Index> get indexes => [
    // 👇 Indexes for common watch queries
    Index('idx_orders_status', 'status'),
    Index('idx_orders_user_id', 'user_id'),
    Index('idx_orders_order_date', 'order_date'),
    // 👇 Composite index for complex queries
    Index('idx_orders_user_status', ['user_id', 'status']),
  ];
}
```

## Selective Column Selection

```dart
// 👇 Inefficient - loads all columns
final stream = select(users).watch();

// 👇 Efficient - only loads needed columns
final efficientStream = select(users)
  .map((u) => [u.id, u.name, u.email])
  .watch()
  .map((rows) {
    return rows.map((row) {
      return UserSummary(
        id: row[0] as int,
        name: row[1] as String,
        email: row[2] as String,
      );
    }).toList();
  });
```

## Stream Caching

```dart
// 👇 Cache stream results
class CachedStream<T> {
  final Stream<T> source;
  T? _cachedValue;
  bool _hasValue = false;
  
  CachedStream(this.source);
  
  Stream<T> get stream {
    return source.doOnData((data) {
      _cachedValue = data;
      _hasValue = true;
    });
  }
  
  T? get cachedValue => _hasValue ? _cachedValue : null;
}

// Usage
final cachedUsers = CachedStream(select(users).watch());
cachedUsers.stream.listen((users) {
  // Update UI
});
// Later, access cached data without subscribing
final users = cachedUsers.cachedValue;
```

## Stream Sharing

```dart
// 👇 Share a single stream subscription
final sharedStream = select(users)
  .watch()
  .shareReplay(1);

// Multiple subscribers share the same stream
sharedStream.listen((users) => updateWidget1(users));
sharedStream.listen((users) => updateWidget2(users));
// Only one database query is performed
```

---

# Real-World Example

> **Complete e-commerce stream optimization system**

```dart
// lib/database/stream_optimization_service.dart
import 'package:drift/drift.dart';
import 'package:rxdart/rxdart.dart';

class StreamOptimizationService {
  final AppDatabase db;
  final Map<String, StreamSubscription> _subscriptions = {};
  final Map<String, dynamic> _cache = {};
  
  StreamOptimizationService(this.db);

  // ==================== OPTIMIZED USER STREAMS ====================
  
  // 👇 Optimized active users stream
  Stream<List<User>> getOptimizedActiveUsers() {
    return (db.select(db.users)
      ..where((u) => u.isActive.equals(true))
      ..orderBy([(u) => OrderingTerm.asc(u.name)])
      ..limit(1000)) // 👈 Limit result size
      .watch()
      .distinct() // 👈 Avoid duplicate emissions
      .debounce(Duration(milliseconds: 200)) // 👈 Debounce rapid changes
      .map((users) {
        // 👈 Lightweight transformation
        return users.map((u) {
          // Only map what's needed
          return User(
            id: u.id,
            name: u.name,
            email: u.email,
            isActive: u.isActive,
            // Skip heavy fields if not needed
          );
        }).toList();
      })
      .shareReplay(1); // 👈 Share stream
  }

  // 👇 Optimized search stream
  Stream<List<User>> getOptimizedSearch(
    BehaviorSubject<String> searchSubject,
  ) {
    return searchSubject
      .debounce(Duration(milliseconds: 300)) // 👈 Wait for typing to pause
      .distinct() // 👈 Only search on new values
      .where((query) => query.length >= 2 || query.isEmpty) // 👈 Minimum length
      .switchMap((query) { // 👈 Cancel previous searches
        if (query.isEmpty) {
          return Stream.value([]);
        }
        
        // 👈 Optimized database query
        final searchTerm = '%$query%';
        return (db.select(db.users)
          ..where((u) => 
            u.name.like(searchTerm) |
            u.email.like(searchTerm)
          )
          ..limit(50)) // 👈 Limit results
          .watch()
          .distinct();
      })
      .shareReplay(1);
  }

  // ==================== OPTIMIZED ORDER STREAMS ====================
  
  // 👇 Optimized order dashboard stream
  Stream<DashboardStats> getOptimizedDashboardStats() {
    // 👈 Use separate optimized streams
    final usersStream = (db.select(db.users)
      ..limit(10000)) // 👈 Limit for count queries
      .watch()
      .distinct()
      .throttle(Duration(seconds: 5)) // 👈 Throttle dashboard updates
      .map((users) => users.length);
    
    final ordersStream = (db.select(db.orders)
      ..limit(10000))
      .watch()
      .distinct()
      .throttle(Duration(seconds: 5))
      .map((orders) {
        return OrderStats(
          total: orders.length,
          pending: orders.where((o) => o.status == 'pending').length,
          totalRevenue: orders.fold(0.0, (sum, o) => sum + o.total),
        );
      });
    
    return Rx.combineLatest2(
      usersStream,
      ordersStream,
      (userCount, orderStats) {
        return DashboardStats(
          totalUsers: userCount,
          totalOrders: orderStats.total,
          pendingOrders: orderStats.pending,
          totalRevenue: orderStats.totalRevenue,
        );
      },
    ).distinct() // 👈 Avoid duplicate emissions
     .shareReplay(1);
  }

  // ==================== OPTIMIZED PRODUCT STREAMS ====================
  
  // 👇 Optimized product catalog with pagination
  Stream<PaginatedProducts> getOptimizedProductCatalog({
    int pageSize = 20,
    String? category,
  }) {
    final query = db.select(db.products)
      ..where((p) => p.isActive.equals(true))
      ..orderBy([(p) => OrderingTerm.asc(p.name)]);
    
    if (category != null && category.isNotEmpty) {
      query.where((p) => p.category.equals(category));
    }
    
    // 👈 Use limit for pagination
    return query
      .watch()
      .distinct()
      .debounce(Duration(milliseconds: 200))
      .map((allProducts) {
        // 👈 Paginate in memory (or do at database level)
        final total = allProducts.length;
        final paginated = allProducts.take(pageSize).toList();
        
        return PaginatedProducts(
          items: paginated,
          total: total,
          hasMore: total > pageSize,
        );
      })
      .shareReplay(1);
  }

  // ==================== OPTIMIZED JOIN STREAMS ====================
  
  // 👇 Optimized order details stream
  Stream<OrderWithItems> getOptimizedOrderDetails(int orderId) {
    final query = db.select(db.orders).join([
      innerJoin(db.users, db.users.id.equals(db.orders.userId)),
      leftJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
      leftJoin(db.products, db.products.id.equals(db.orderItems.productId)),
    ]);
    
    query.where((o) => db.orders.id.equals(orderId));
    query.limit(1); // 👈 Only need one order
    
    return query
      .watch()
      .distinct()
      .map((rows) {
        if (rows.isEmpty) {
          throw Exception('Order not found');
        }
        
        final order = rows.first.readTable(db.orders);
        final user = rows.first.readTable(db.users);
        
        final items = <OrderItemWithProduct>[];
        for (final row in rows) {
          final item = row.readTableOrNull(db.orderItems);
          if (item != null) {
            final product = row.readTable(db.products);
            items.add(
              OrderItemWithProduct(
                orderItem: item,
                product: product,
              )
            );
          }
        }
        
        return OrderWithItems(
          order: order,
          user: user,
          items: items,
        );
      })
      .shareReplay(1);
  }

  // ==================== PERFORMANCE MONITORING ====================
  
  // 👇 Monitor stream performance
  Stream<T> monitorStream<T>(
    Stream<T> stream,
    String name,
  ) {
    return stream.doOnListen(() {
      print('📊 Stream "$name" started');
    }).doOnData((data) {
      print('📊 Stream "$name" emitted data');
    }).doOnCancel(() {
      print('📊 Stream "$name" cancelled');
    }).doOnError((error) {
      print('❌ Stream "$name" error: $error');
    });
  }
  
  // 👇 Performance tracked stream
  Stream<List<User>> getPerformanceTrackedUsers() {
    return monitorStream(
      getOptimizedActiveUsers(),
      'ActiveUsers',
    );
  }

  // ==================== BATCH UPDATE OPTIMIZATION ====================
  
  // 👇 Optimize batch updates
  Future<void> optimizedBatchUpdate(List<UserUpdate> updates) async {
    // 👈 Use batch for multiple updates
    await db.into(db.users).batch((batch) {
      for (final update in updates) {
        batch.update(
          db.users,
          UsersCompanion(
            name: update.name != null ? Value(update.name!) : const Value.absent(),
            isActive: update.isActive != null 
                ? Value(update.isActive!) 
                : const Value.absent(),
          ),
          (u) => u.id.equals(update.id),
        );
      }
    });
    
    // 👈 Streams will automatically emit once for all updates
  }

  // ==================== CLEANUP ====================
  
  void dispose() {
    for (final subscription in _subscriptions.values) {
      subscription.cancel();
    }
    _subscriptions.clear();
    _cache.clear();
    print('🧹 Stream optimization service disposed');
  }
}

// ==================== DATA CLASSES ====================

class UserSummary {
  final int id;
  final String name;
  final String email;
  
  UserSummary({
    required this.id,
    required this.name,
    required this.email,
  });
}

class OrderStats {
  final int total;
  final int pending;
  final double totalRevenue;
  
  OrderStats({
    required this.total,
    required this.pending,
    required this.totalRevenue,
  });
}

class PaginatedProducts {
  final List<Product> items;
  final int total;
  final bool hasMore;
  
  PaginatedProducts({
    required this.items,
    required this.total,
    required this.hasMore,
  });
}
```

```dart
// lib/ui/pages/performance_demo_page.dart
class PerformanceDemoPage extends StatefulWidget {
  final StreamOptimizationService optimizationService;
  
  const PerformanceDemoPage({required this.optimizationService});
  
  @override
  _PerformanceDemoPageState createState() => _PerformanceDemoPageState();
}

class _PerformanceDemoPageState extends State<PerformanceDemoPage> {
  final _searchController = BehaviorSubject<String>();
  final _textController = TextEditingController();
  List<User> _users = [];
  bool _isLoading = true;
  
  @override
  void initState() {
    super.initState();
    _setupStreams();
  }
  
  void _setupStreams() {
    // 👇 Optimized search stream
    widget.optimizationService
      .getOptimizedSearch(_searchController)
      .listen((users) {
        setState(() {
          _users = users;
          _isLoading = false;
        });
      });
  }
  
  @override
  void dispose() {
    _searchController.close();
    _textController.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Stream Performance Demo'),
        backgroundColor: Colors.green[800],
      ),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // Search
            TextField(
              controller: _textController,
              decoration: InputDecoration(
                hintText: 'Search users (debounced)',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.search),
              ),
              onChanged: (query) {
                _searchController.add(query);
              },
            ),
            SizedBox(height: 16),
            
            // Results count
            Text(
              '${_users.length} users found',
              style: TextStyle(
                fontSize: 16,
                fontWeight: FontWeight.bold,
              ),
            ),
            SizedBox(height: 8),
            
            // Results
            Expanded(
              child: _isLoading
                  ? Center(child: CircularProgressIndicator())
                  : ListView.builder(
                      itemCount: _users.length,
                      itemBuilder: (context, index) {
                        final user = _users[index];
                        return ListTile(
                          leading: CircleAvatar(
                            child: Text(user.name[0].toUpperCase()),
                          ),
                          title: Text(user.name),
                          subtitle: Text(user.email),
                          trailing: Icon(
                            user.isActive ? Icons.check_circle : Icons.cancel,
                            color: user.isActive ? Colors.green : Colors.red,
                          ),
                        );
                      },
                    ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

# Performance Checklist

| Technique | Description | Impact |
|-----------|-------------|--------|
| **Indexes** | Add indexes for watched columns | High |
| **Limit** | Use `limit()` for large tables | High |
| **Distinct** | Avoid duplicate emissions | Medium |
| **Debounce** | Batch rapid changes | Medium |
| **Throttle** | Limit update frequency | Medium |
| **Selective Columns** | Only select needed columns | Medium |
| **Share** | Reuse streams | Medium |
| **Caching** | Cache expensive results | High |
| **Batch Updates** | Batch multiple changes | High |

---

# Best Practices

- **Add indexes** – For frequently watched columns
- **Limit result sets** – Use `limit()` for large tables
- **Use distinct** – Avoid duplicate emissions
- **Use debounce** – For search/input streams
- **Use throttle** – For high-frequency updates
- **Use selective columns** – Only select what's needed
- **Share streams** – Reuse subscriptions
- **Batch updates** – Reduce stream emissions
- **Monitor performance** – Track stream behavior
- **Test with real data** – Verify performance

---

# Common Mistakes

## Mistake 1: No indexes on watched columns

Wrong:
```dart
// 🚫 No indexes, slow queries
class Orders extends Table {
  TextColumn get status => text()();
  // No index on status
}
```

Correct:
```dart
// ✅ Add indexes
class Orders extends Table {
  TextColumn get status => text()();
  
  @override
  List<Index> get indexes => [
    Index('idx_orders_status', 'status'),
  ];
}
```

## Mistake 2: Loading too much data

Wrong:
```dart
// 🚫 Loads all records
final stream = select(orders).watch();
```

Correct:
```dart
// ✅ Only load what's needed
final stream = (select(orders)
  ..limit(100))
  .watch();
```

## Mistake 3: No debounce for search

Wrong:
```dart
// 🚫 Database query on every keystroke
textStream.switchMap((query) {
  return (select(users)..where(...)).watch();
});
```

Correct:
```dart
// ✅ Debounce input
textStream
  .debounce(Duration(milliseconds: 300))
  .switchMap((query) {
    return (select(users)..where(...)).watch();
  });
```

---

# Summary

| Technique | Purpose | Example |
|-----------|---------|---------|
| **Indexes** | Speed up queries | `Index('idx_status', 'status')` |
| **Limit** | Reduce data | `limit(100)` |
| **Distinct** | Avoid duplicates | `.distinct()` |
| **Debounce** | Reduce frequency | `.debounce(Duration(milliseconds: 300))` |
| **Throttle** | Limit frequency | `.throttle(Duration(seconds: 1))` |
| **Share** | Reuse streams | `.shareReplay(1)` |

---

# Next Steps

Now you understand stream performance, let's dive deeper:

- [Transactions](link) – Transaction management
- [Custom SQL](link) – Advanced SQL features
- [Indexing](link) – Advanced indexing strategies

---

# Did You Know?

- **Indexes can improve stream performance by 100x** – On large tables

- **Debounce reduces database queries** – By batching rapid changes

- **Distinct prevents UI flicker** – By avoiding duplicate updates

- **Stream sharing reduces resource usage** – One stream for many subscribers

- **Limit prevents memory issues** – By controlling result size

- **Batch updates reduce stream emissions** – One emission for many changes

- **Selective columns reduce data transfer** – Only what's needed

- **Performance optimization is essential** – For production apps

---

