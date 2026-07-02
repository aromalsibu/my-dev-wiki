## Watching Multiple Tables

**Tracking changes across multiple tables with Drift streams**

---

# What is it?

**Watching Multiple Tables** involves creating reactive streams that depend on data from several tables simultaneously. Drift's stream system automatically tracks which tables a query depends on and emits updates when any of those tables change. This is essential for complex views that combine data from multiple sources.

> **Think of Watching Multiple Tables like "monitoring multiple news sources"** – instead of checking each source individually, you have a dashboard that updates automatically when ANY of the sources publish new information.

```dart
// 👇 This query depends on multiple tables
final query = db.select(db.orders).join([
  innerJoin(db.users, db.users.id.equals(db.orders.userId)),
  innerJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
  innerJoin(db.products, db.products.id.equals(db.orderItems.productId)),
]);

// 👇 The stream will emit when ANY of these tables change:
// - orders table
// - users table  
// - orderItems table
// - products table
final stream = query.watch();
```

> **What's happening here?**
> - **Dependency tracking** – Drift tracks all tables in the query
> - **Multi-table updates** – Any change triggers emission
> - **Complex queries** – Joins across many tables
> - **Automatic management** – No manual tracking needed

---

# Why does it exist?

- **Complex Views** – Dashboards with multiple data sources
- **Real-time Updates** – Keep UI in sync across tables
- **Data Consistency** – Reflect all changes immediately
- **Performance** – One stream for multiple tables
- **User Experience** – Always show latest data
- **Simplified Code** – Automatic dependency management

---

# Multi-Table Watch Basics

> **Simple multi-table watching patterns**

## Watch with Single Join

```dart
// 👇 Watch users with their orders
final query = db.select(db.users).join([
  innerJoin(db.orders, db.orders.userId.equals(db.users.id)),
]);

// 👇 Updates when users OR orders change
final stream = query.watch();

stream.listen((rows) {
  for (final row in rows) {
    final user = row.readTable(db.users);
    final order = row.readTable(db.orders);
    print('${user.name} -> ${order.orderNumber}');
  }
});
```

## Watch with Multiple Joins

```dart
// 👇 Watch complete order details
final query = db.select(db.orders).join([
  innerJoin(db.users, db.users.id.equals(db.orders.userId)),
  innerJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
  innerJoin(db.products, db.products.id.equals(db.orderItems.productId)),
]);

// 👇 Updates when ANY of the 4 tables change
final stream = query.watch();

stream.listen((rows) {
  for (final row in rows) {
    final order = row.readTable(db.orders);
    final user = row.readTable(db.users);
    final item = row.readTable(db.orderItems);
    final product = row.readTable(db.products);
    
    print('Order ${order.orderNumber} by ${user.name}');
    print('  ${product.name} x ${item.quantity}');
  }
});
```

---

# Multi-Table Watch Strategies

> **Different approaches to watching multiple tables**

## Strategy 1: Direct Join Watch

```dart
// 👇 Direct watch with joins (simplest)
Future<Stream<List<OrderWithDetails>>> watchOrdersWithDetails() {
  final query = db.select(db.orders).join([
    innerJoin(db.users, db.users.id.equals(db.orders.userId)),
    leftJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
    leftJoin(db.products, db.products.id.equals(db.orderItems.productId)),
  ]);
  
  return query.watch().map((rows) {
    final orderMap = <int, OrderWithDetails>{};
    
    for (final row in rows) {
      final order = row.readTable(db.orders);
      
      if (!orderMap.containsKey(order.id)) {
        orderMap[order.id] = OrderWithDetails(
          order: order,
          user: row.readTable(db.users),
          items: [],
        );
      }
      
      final item = row.readTableOrNull(db.orderItems);
      if (item != null) {
        final product = row.readTable(db.products);
        orderMap[order.id]!.items.add(
          OrderItemWithProduct(
            orderItem: item,
            product: product,
          )
        );
      }
    }
    
    return orderMap.values.toList();
  });
}
```

## Strategy 2: Separate Watches Combined

```dart
// 👇 Watch tables separately and combine
Stream<CombinedData> watchCombinedData() {
  final usersStream = db.select(db.users)
    .where((u) => u.isActive.equals(true))
    .watch();
  
  final ordersStream = db.select(db.orders)
    .where((o) => o.status.equals('pending'))
    .watch();
  
  final productsStream = db.select(db.products)
    .where((p) => p.isActive.equals(true))
    .watch();
  
  // 👇 Combine all streams
  return Rx.combineLatest3(
    usersStream,
    ordersStream,
    productsStream,
    (users, orders, products) {
      return CombinedData(
        activeUsers: users,
        pendingOrders: orders,
        activeProducts: products,
        totalUsers: users.length,
        totalOrders: orders.length,
        totalProducts: products.length,
      );
    },
  );
}
```

## Strategy 3: Lazy Multi-Table Watch

```dart
// 👇 Only watch when needed
class LazyMultiTableWatcher {
  final AppDatabase db;
  StreamSubscription? _subscription;
  final BehaviorSubject<List<OrderWithDetails>> _subject = BehaviorSubject();
  
  LazyMultiTableWatcher(this.db);
  
  Stream<List<OrderWithDetails>> watch() {
    if (_subscription == null) {
      _subscription = _buildQuery().watch().listen((rows) {
        _subject.add(_mapRows(rows));
      });
    }
    return _subject.stream;
  }
  
  Query _buildQuery() {
    return db.select(db.orders).join([
      innerJoin(db.users, db.users.id.equals(db.orders.userId)),
      innerJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
      innerJoin(db.products, db.products.id.equals(db.orderItems.productId)),
    ]);
  }
  
  List<OrderWithDetails> _mapRows(List<Row> rows) {
    // Mapping logic
    return [];
  }
  
  void dispose() {
    _subscription?.cancel();
    _subject.close();
  }
}
```

---

# Real-World Example

> **Complete e-commerce multi-table watch system**

```dart
// lib/database/multi_table_watch_service.dart
import 'package:drift/drift.dart';
import 'package:rxdart/rxdart.dart';

class MultiTableWatchService {
  final AppDatabase db;
  
  MultiTableWatchService(this.db);

  // ==================== COMPLETE ORDER TRACKING ====================
  
  // 👇 Watch complete order lifecycle
  Stream<OrderLifecycle> watchOrderLifecycle(int orderId) {
    final query = db.select(db.orders).join([
      innerJoin(db.users, db.users.id.equals(db.orders.userId)),
      leftJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
      leftJoin(db.products, db.products.id.equals(db.orderItems.productId)),
      leftJoin(db.shippingLogs, db.shippingLogs.orderId.equals(db.orders.id)),
    ]);
    
    query.where((o) => db.orders.id.equals(orderId));
    query.orderBy([(i) => OrderingTerm.asc(db.orderItems.id)]);
    
    return query.watch().map((rows) {
      if (rows.isEmpty) {
        throw Exception('Order not found');
      }
      
      // Extract data
      final order = rows.first.readTable(db.orders);
      final user = rows.first.readTable(db.users);
      final shippingLog = rows.first.readTableOrNull(db.shippingLogs);
      
      // Extract items
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
      
      return OrderLifecycle(
        order: order,
        user: user,
        items: items,
        shippingLog: shippingLog,
        status: order.status,
        isPaid: order.isPaid,
        isShipped: order.isShipped,
        isDelivered: order.isDelivered,
        lastUpdate: DateTime.now(),
      );
    }).distinct((a, b) =>
      a.status == b.status &&
      a.isPaid == b.isPaid &&
      a.isShipped == b.isShipped &&
      a.isDelivered == b.isDelivered
    );
  }

  // ==================== COMPLETE PRODUCT CATALOG ====================
  
  // 👇 Watch product catalog with all relationships
  Stream<List<CompleteProduct>> watchCompleteProductCatalog() {
    final query = db.select(db.products).join([
      leftJoin(db.productCategories, db.productCategories.productId.equals(db.products.id)),
      leftJoin(db.categories, db.categories.id.equals(db.productCategories.categoryId)),
      leftJoin(db.productTags, db.productTags.productId.equals(db.products.id)),
      leftJoin(db.tags, db.tags.id.equals(db.productTags.tagId)),
      leftJoin(db.reviews, db.reviews.productId.equals(db.products.id)),
      leftJoin(db.users, db.users.id.equals(db.reviews.userId)),
    ]);
    
    query.where((p) => db.products.isActive.equals(true));
    query.orderBy([(p) => OrderingTerm.asc(db.products.name)]);
    
    return query.watch().map((rows) {
      final productMap = <int, CompleteProduct>{};
      
      for (final row in rows) {
        final product = row.readTable(db.products);
        
        if (!productMap.containsKey(product.id)) {
          productMap[product.id] = CompleteProduct(
            product: product,
            categories: [],
            tags: [],
            reviews: [],
          );
        }
        
        final category = row.readTableOrNull(db.categories);
        if (category != null && 
            !productMap[product.id]!.categories.any((c) => c.id == category.id)) {
          productMap[product.id]!.categories.add(category);
        }
        
        final tag = row.readTableOrNull(db.tags);
        if (tag != null &&
            !productMap[product.id]!.tags.any((t) => t.id == tag.id)) {
          productMap[product.id]!.tags.add(tag);
        }
        
        final review = row.readTableOrNull(db.reviews);
        if (review != null) {
          final user = row.readTable(db.users);
          if (!productMap[product.id]!.reviews.any((r) => r.review.id == review.id)) {
            productMap[product.id]!.reviews.add(
              ReviewWithUser(
                review: review,
                user: user,
              )
            );
          }
        }
      }
      
      return productMap.values.toList();
    });
  }

  // ==================== USER FULL ACTIVITY ====================
  
  // 👇 Watch complete user activity
  Stream<UserFullActivity> watchUserFullActivity(int userId) {
    // 1️⃣ Watch user profile
    final userStream = (db.select(db.users)
      ..where((u) => u.id.equals(userId)))
      .watchSingle();
    
    // 2️⃣ Watch user orders with items
    final ordersQuery = db.select(db.orders).join([
      innerJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
      innerJoin(db.products, db.products.id.equals(db.orderItems.productId)),
    ]);
    ordersQuery.where((o) => db.orders.userId.equals(userId));
    ordersQuery.orderBy([(o) => OrderingTerm.desc(db.orders.orderDate)]);
    final ordersStream = ordersQuery.watch();
    
    // 3️⃣ Watch user reviews
    final reviewsQuery = db.select(db.reviews).join([
      innerJoin(db.products, db.products.id.equals(db.reviews.productId)),
    ]);
    reviewsQuery.where((r) => db.reviews.userId.equals(userId));
    reviewsQuery.orderBy([(r) => OrderingTerm.desc(db.reviews.createdAt)]);
    final reviewsStream = reviewsQuery.watch();
    
    // 4️⃣ Combine all streams
    return Rx.combineLatest3(
      userStream,
      ordersStream,
      reviewsStream,
      (user, orderRows, reviewRows) {
        // Map orders
        final orderMap = <int, OrderWithItems>{};
        for (final row in orderRows) {
          final order = row.readTable(db.orders);
          
          if (!orderMap.containsKey(order.id)) {
            orderMap[order.id] = OrderWithItems(
              order: order,
              items: [],
            );
          }
          
          final item = row.readTable(db.orderItems);
          final product = row.readTable(db.products);
          orderMap[order.id]!.items.add(
            OrderItemWithProduct(
              orderItem: item,
              product: product,
            )
          );
        }
        
        // Map reviews
        final reviews = reviewRows.map((row) {
          return ReviewWithProduct(
            review: row.readTable(db.reviews),
            product: row.readTable(db.products),
          );
        }).toList();
        
        // Map profile (fetch separately if needed)
        return UserFullActivity(
          user: user,
          orders: orderMap.values.toList(),
          reviews: reviews,
          totalOrders: orderMap.length,
          totalReviews: reviews.length,
          totalSpent: orderMap.values.fold(0.0, (sum, o) => sum + o.order.total),
          lastActivity: _getLastActivity(orderMap.values, reviews),
        );
      },
    );
  }
  
  DateTime _getLastActivity(
    Iterable<OrderWithItems> orders,
    List<ReviewWithProduct> reviews,
  ) {
    final orderDates = orders.map((o) => o.order.orderDate);
    final reviewDates = reviews.map((r) => r.review.createdAt);
    final allDates = [...orderDates, ...reviewDates];
    return allDates.isEmpty ? DateTime.now() : allDates.reduce((a, b) => a.isAfter(b) ? a : b);
  }

  // ==================== COMPLETE ANALYTICS DASHBOARD ====================
  
  // 👇 Watch complete analytics
  Stream<AnalyticsDashboard> watchAnalyticsDashboard() {
    // 1️⃣ User metrics
    final userMetrics = db.select(db.users)
      .watch()
      .map((users) {
        return UserMetrics(
          total: users.length,
          active: users.where((u) => u.isActive).length,
          verified: users.where((u) => u.isVerified).length,
          newToday: users.where((u) =>
            u.createdAt.isAfter(DateTime.now().subtract(Duration(days: 1)))
          ).length,
        );
      });
    
    // 2️⃣ Order metrics
    final orderMetrics = db.select(db.orders)
      .watch()
      .map((orders) {
        return OrderMetrics(
          total: orders.length,
          pending: orders.where((o) => o.status == 'pending').length,
          completed: orders.where((o) => o.status == 'delivered').length,
          totalRevenue: orders.fold(0.0, (sum, o) => sum + o.total),
          avgOrderValue: orders.isEmpty ? 0.0 : 
            orders.fold(0.0, (sum, o) => sum + o.total) / orders.length,
        );
      });
    
    // 3️⃣ Product metrics
    final productMetrics = db.select(db.products)
      .watch()
      .map((products) {
        return ProductMetrics(
          total: products.length,
          active: products.where((p) => p.isActive).length,
          lowStock: products.where((p) => p.stock < 10).length,
          outOfStock: products.where((p) => p.stock == 0).length,
        );
      });
    
    // 4️⃣ Recent orders with users
    final recentOrders = (db.select(db.orders)
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)])
      ..limit(10))
      .join([
        innerJoin(db.users, db.users.id.equals(db.orders.userId)),
      ])
      .watch()
      .map((rows) {
        return rows.map((row) {
          return OrderWithUser(
            order: row.readTable(db.orders),
            user: row.readTable(db.users),
          );
        }).toList();
      });
    
    // 5️⃣ Combine all metrics
    return Rx.combineLatest4(
      userMetrics,
      orderMetrics,
      productMetrics,
      recentOrders,
      (users, orders, products, recent) {
        return AnalyticsDashboard(
          userMetrics: users,
          orderMetrics: orders,
          productMetrics: products,
          recentOrders: recent,
          timestamp: DateTime.now(),
        );
      },
    ).distinct((a, b) =>
      a.userMetrics.total == b.userMetrics.total &&
      a.orderMetrics.totalRevenue == b.orderMetrics.totalRevenue &&
      a.productMetrics.total == b.productMetrics.total
    );
  }

  // ==================== REAL-TIME MONITORING ====================
  
  // 👇 Monitor all critical tables
  Stream<SystemHealth> watchSystemHealth() {
    final users = db.select(db.users).watch();
    final orders = db.select(db.orders).watch();
    final products = db.select(db.products).watch();
    final payments = db.select(db.payments).watch();
    
    return Rx.combineLatest4(
      users,
      orders,
      products,
      payments,
      (u, o, p, pay) {
        return SystemHealth(
          totalUsers: u.length,
          totalOrders: o.length,
          totalProducts: p.length,
          totalPayments: pay.length,
          pendingOrders: o.where((ord) => ord.status == 'pending').length,
          failedPayments: pay.where((paym) => paym.status == 'failed').length,
          lowStockProducts: p.where((prod) => prod.stock < 10).length,
          averageOrderValue: o.isEmpty ? 0.0 :
            o.fold(0.0, (sum, ord) => sum + ord.total) / o.length,
          lastUpdated: DateTime.now(),
        );
      },
    );
  }
}

// ==================== DATA CLASSES ====================

class OrderLifecycle {
  final Order order;
  final User user;
  final List<OrderItemWithProduct> items;
  final ShippingLog? shippingLog;
  final String status;
  final bool isPaid;
  final bool isShipped;
  final bool isDelivered;
  final DateTime lastUpdate;
  
  OrderLifecycle({
    required this.order,
    required this.user,
    required this.items,
    this.shippingLog,
    required this.status,
    required this.isPaid,
    required this.isShipped,
    required this.isDelivered,
    required this.lastUpdate,
  });
}

class CompleteProduct {
  final Product product;
  final List<Category> categories;
  final List<Tag> tags;
  final List<ReviewWithUser> reviews;
  
  CompleteProduct({
    required this.product,
    required this.categories,
    required this.tags,
    required this.reviews,
  });
}

class UserFullActivity {
  final User user;
  final List<OrderWithItems> orders;
  final List<ReviewWithProduct> reviews;
  final int totalOrders;
  final int totalReviews;
  final double totalSpent;
  final DateTime lastActivity;
  
  UserFullActivity({
    required this.user,
    required this.orders,
    required this.reviews,
    required this.totalOrders,
    required this.totalReviews,
    required this.totalSpent,
    required this.lastActivity,
  });
}

class ReviewWithProduct {
  final Review review;
  final Product product;
  
  ReviewWithProduct({
    required this.review,
    required this.product,
  });
}

class UserMetrics {
  final int total;
  final int active;
  final int verified;
  final int newToday;
  
  UserMetrics({
    required this.total,
    required this.active,
    required this.verified,
    required this.newToday,
  });
}

class OrderMetrics {
  final int total;
  final int pending;
  final int completed;
  final double totalRevenue;
  final double avgOrderValue;
  
  OrderMetrics({
    required this.total,
    required this.pending,
    required this.completed,
    required this.totalRevenue,
    required this.avgOrderValue,
  });
}

class ProductMetrics {
  final int total;
  final int active;
  final int lowStock;
  final int outOfStock;
  
  ProductMetrics({
    required this.total,
    required this.active,
    required this.lowStock,
    required this.outOfStock,
  });
}

class AnalyticsDashboard {
  final UserMetrics userMetrics;
  final OrderMetrics orderMetrics;
  final ProductMetrics productMetrics;
  final List<OrderWithUser> recentOrders;
  final DateTime timestamp;
  
  AnalyticsDashboard({
    required this.userMetrics,
    required this.orderMetrics,
    required this.productMetrics,
    required this.recentOrders,
    required this.timestamp,
  });
}

class SystemHealth {
  final int totalUsers;
  final int totalOrders;
  final int totalProducts;
  final int totalPayments;
  final int pendingOrders;
  final int failedPayments;
  final int lowStockProducts;
  final double averageOrderValue;
  final DateTime lastUpdated;
  
  SystemHealth({
    required this.totalUsers,
    required this.totalOrders,
    required this.totalProducts,
    required this.totalPayments,
    required this.pendingOrders,
    required this.failedPayments,
    required this.lowStockProducts,
    required this.averageOrderValue,
    required this.lastUpdated,
  });
}
```

```dart
// lib/ui/pages/analytics_dashboard_page.dart
class AnalyticsDashboardPage extends StatelessWidget {
  final MultiTableWatchService watchService;
  
  const AnalyticsDashboardPage({required this.watchService});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Real-Time Analytics'),
        backgroundColor: Colors.blue[900],
      ),
      body: StreamBuilder<AnalyticsDashboard>(
        stream: watchService.watchAnalyticsDashboard(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return Center(child: CircularProgressIndicator());
          }
          
          final data = snapshot.data!;
          
          return ListView(
            padding: EdgeInsets.all(16),
            children: [
              // User metrics
              _buildSection('User Metrics', [
                _buildMetric('Total Users', data.userMetrics.total.toString()),
                _buildMetric('Active', data.userMetrics.active.toString()),
                _buildMetric('Verified', data.userMetrics.verified.toString()),
                _buildMetric('New Today', data.userMetrics.newToday.toString()),
              ]),
              SizedBox(height: 16),
              
              // Order metrics
              _buildSection('Order Metrics', [
                _buildMetric('Total Orders', data.orderMetrics.total.toString()),
                _buildMetric('Pending', data.orderMetrics.pending.toString()),
                _buildMetric('Completed', data.orderMetrics.completed.toString()),
                _buildMetric('Revenue', '\$${data.orderMetrics.totalRevenue.toStringAsFixed(0)}'),
              ]),
              SizedBox(height: 16),
              
              // Product metrics
              _buildSection('Product Metrics', [
                _buildMetric('Total Products', data.productMetrics.total.toString()),
                _buildMetric('Active', data.productMetrics.active.toString()),
                _buildMetric('Low Stock', data.productMetrics.lowStock.toString()),
                _buildMetric('Out of Stock', data.productMetrics.outOfStock.toString()),
              ]),
              SizedBox(height: 16),
              
              // Recent orders
              Card(
                child: Padding(
                  padding: EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        'Recent Orders',
                        style: TextStyle(
                          fontSize: 18,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      SizedBox(height: 8),
                      ...data.recentOrders.map((order) {
                        return ListTile(
                          leading: Icon(Icons.receipt),
                          title: Text(order.order.orderNumber),
                          subtitle: Text(order.user.name),
                          trailing: Text('\$${order.order.total.toStringAsFixed(2)}'),
                        );
                      }),
                    ],
                  ),
                ),
              ),
              SizedBox(height: 16),
              Text(
                'Updated: ${_formatTime(data.timestamp)}',
                style: TextStyle(
                  fontSize: 12,
                  color: Colors.grey,
                ),
              ),
            ],
          );
        },
      ),
    );
  }
  
  Widget _buildSection(String title, List<Widget> children) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              title,
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            SizedBox(height: 12),
            Wrap(
              spacing: 16,
              children: children,
            ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildMetric(String label, String value) {
    return Container(
      padding: EdgeInsets.symmetric(horizontal: 12, vertical: 8),
      decoration: BoxDecoration(
        color: Colors.grey[100],
        borderRadius: BorderRadius.circular(8),
      ),
      child: Column(
        children: [
          Text(
            value,
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          Text(
            label,
            style: TextStyle(
              fontSize: 12,
              color: Colors.grey[600],
            ),
          ),
        ],
      ),
    );
  }
  
  String _formatTime(DateTime time) {
    return '${time.hour.toString().padLeft(2, '0')}:${time.minute.toString().padLeft(2, '0')}:${time.second.toString().padLeft(2, '0')}';
  }
}
```

---

# Best Practices

- **Use join watches** – For complex data relationships
- **Use combineLatest** – For independent table streams
- **Use distinct** – Avoid duplicate emissions
- **Use throttle** – For high-frequency updates
- **Use caching** – For expensive transformations
- **Use error handling** – Graceful degradation
- **Use lazy loading** – Start when needed
- **Use proper disposal** – Prevent memory leaks

---

# Common Mistakes

## Mistake 1: Too many table watches

Wrong:
```dart
// 🚀 Too many separate watches
final stream1 = table1.watch();
final stream2 = table2.watch();
final stream3 = table3.watch();
// Combine them all
```

Correct:
```dart
// ✅ Single join watch when possible
final stream = db.select(table1).join([
  innerJoin(table2, ...),
  innerJoin(table3, ...),
]).watch();
```

## Mistake 2: Not tracking dependencies

Wrong:
```dart
// 🚫 Manual cache that doesn't update
List cachedData = [];
stream.listen((data) {
  cachedData = data;
});
// cachedData might be stale
```

Correct:
```dart
// ✅ Always read from stream
stream.listen((data) {
  // Always fresh data
  updateUI(data);
});
```

## Mistake 3: Heavy computation in stream

Wrong:
```dart
// 🚫 Heavy processing on every emission
stream.watch().map((data) {
  return processLargeDataset(data); // Slow
});
```

Correct:
```dart
// ✅ Use debounce/throttle
stream.watch()
  .debounce(Duration(milliseconds: 500))
  .map((data) => processLargeDataset(data));
```

---

# Summary

| Pattern | Purpose | Example |
|---------|---------|---------|
| **Join Watch** | Multiple tables | Complex queries |
| **CombineLatest** | Independent streams | Separate watches |
| **Debounce** | Reduce updates | Rapid changes |
| **Distinct** | Avoid duplicates | Unchanged data |
| **Lazy Watch** | On-demand loading | Resource optimization |

---

# Next Steps

Now you understand watching multiple tables, let's dive deeper:

- [Stream Performance](link) – Performance optimization
- [Transactions](link) – Transaction management
- [Custom SQL](link) – Advanced SQL features

---

# Did You Know?

- **Drift tracks table dependencies automatically** – No manual tracking

- **Multiple table streams are efficient** – Smart change detection

- **Join watches update when ANY table changes** – Comprehensive

- **CombineLatest updates when ALL streams update** – Synchronized

- **Multi-table watches reduce code** – One stream vs many

- **Multi-table watches improve UX** – Always up-to-date

- **Multi-table watches are scalable** – Handle many tables

- **Multi-table watches are essential** – For complex applications

---
