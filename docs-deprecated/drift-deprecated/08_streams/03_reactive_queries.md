## Reactive Queries

**Building powerful reactive data flows with Drift streams**

---

# What is it?

**Reactive Queries** are the combination of Drift's stream capabilities with reactive programming patterns to create sophisticated, real-time data flows. By combining streams with operators like `combineLatest`, `merge`, `switchMap`, and more, you can build complex reactive data pipelines that automatically respond to changes across multiple data sources.

> **Think of Reactive Queries like "orchestrating a live show"** – instead of each instrument playing independently, you have a conductor (reactive logic) that coordinates all the instruments (data streams) to create a harmonious performance (reactive UI).

```dart
// 👇 Combining two streams reactively
final activeUsersStream = (select(users)
  ..where((u) => u.isActive.equals(true)))
  .watch();

final recentOrdersStream = (select(orders)
  ..orderBy([(o) => OrderingTerm.desc(o.orderDate)])
  ..limit(10))
  .watch();

// 👇 Combine both streams into one reactive flow
final dashboardStream = Rx.combineLatest2(
  activeUsersStream,
  recentOrdersStream,
  (users, orders) {
    return DashboardData(
      totalUsers: users.length,
      recentOrders: orders,
      // ... compute derived data
    );
  },
);

// Now dashboard automatically updates when EITHER stream changes!
```

> **What's happening here?**
> - **Combine streams** – Merge multiple data sources
> - **Reactive operators** – Transform and combine streams
> - **Derived data** – Compute from multiple sources
> - **Automatic updates** – Any change triggers recomputation

---

# Why does it exist?

- **Complex UIs** – Combine multiple data sources
- **Derived State** – Compute from multiple streams
- **Real-time Dashboards** – Update all widgets together
- **Search with Filters** – Combine search terms with data
- **Data Consistency** – Keep related data in sync
- **Performance** – Reduce unnecessary computations

---

# Combining Streams

> **Merging multiple reactive streams**

## Combine Two Streams

```dart
// 👇 Combine active users and recent orders
final activeUsersStream = (select(users)
  ..where((u) => u.isActive.equals(true)))
  .watch();

final recentOrdersStream = (select(orders)
  ..orderBy([(o) => OrderingTerm.desc(o.orderDate)])
  ..limit(5))
  .watch();

final combinedStream = Rx.combineLatest2(
  activeUsersStream,
  recentOrdersStream,
  (users, orders) {
    return DashboardSummary(
      activeUsers: users.length,
      recentOrders: orders,
      totalRevenue: orders.fold(0.0, (sum, o) => sum + o.total),
    );
  },
);

// UI updates when either active users OR recent orders change
```

## Combine Three Streams

```dart
// 👇 Combine users, orders, and products
final usersStream = select(users).watch();
final ordersStream = select(orders).watch();
final productsStream = select(products).watch();

final combinedStream = Rx.combineLatest3(
  usersStream,
  ordersStream,
  productsStream,
  (users, orders, products) {
    return SystemStats(
      totalUsers: users.length,
      totalOrders: orders.length,
      totalProducts: products.length,
      activeUsers: users.where((u) => u.isActive).length,
      pendingOrders: orders.where((o) => o.status == 'pending').length,
    );
  },
);
```

## Combine Multiple Streams (List)

```dart
// 👇 Combine any number of streams
final streams = [
  select(users).watch(),
  select(orders).watch(),
  select(products).watch(),
  select(categories).watch(),
];

final combinedStream = Rx.combineLatest(
  streams,
  (values) {
    final users = values[0] as List<User>;
    final orders = values[1] as List<Order>;
    final products = values[2] as List<Product>;
    final categories = values[3] as List<Category>;
    
    return {
      'users': users.length,
      'orders': orders.length,
      'products': products.length,
      'categories': categories.length,
    };
  },
);
```

---

# Stream Transformation

> **Transforming streams with reactive operators**

## Mapping Streams

```dart
// 👇 Transform user stream to UI model
final userViewModels = select(users)
  .watch()
  .map((users) {
    return users.map((user) {
      return UserViewModel(
        id: user.id,
        displayName: user.name,
        email: user.email,
        isActive: user.isActive,
        initials: _getInitials(user.name),
      );
    }).toList();
  });

// 👇 Transform and filter
final activeUserEmails = select(users)
  .watch()
  .map((users) => users.where((u) => u.isActive))
  .map((users) => users.map((u) => u.email).toList());
```

## Debouncing Streams

```dart
// 👇 Debounce search input
final searchController = StreamController<String>();

final searchResults = searchController.stream
  .debounce(Duration(milliseconds: 500))
  .switchMap((query) {
    if (query.isEmpty) {
      return Stream.value([]);
    }
    
    // Return a stream that watches the search
    return (select(users)
      ..where((u) => u.name.like('%$query%')))
      .watch();
  });

// Usage
searchController.add('John'); // Will wait 500ms before searching
```

## SwitchMap Pattern

```dart
// 👇 Switch between different streams based on input
class FilterState {
  final String category;
  final String sortBy;
  final bool ascending;
}

final filterStream = BehaviorSubject<FilterState>();

final productsStream = filterStream.switchMap((filter) {
  final query = select(products)
    ..where((p) => p.isActive.equals(true));
  
  if (filter.category.isNotEmpty) {
    query.where((p) => p.category.equals(filter.category));
  }
  
  final mode = filter.ascending ? OrderingMode.asc : OrderingMode.desc;
  
  switch (filter.sortBy) {
    case 'name':
      query.orderBy([(p) => OrderingTerm(expression: p.name, mode: mode)]);
      break;
    case 'price':
      query.orderBy([(p) => OrderingTerm(expression: p.price, mode: mode)]);
      break;
  }
  
  return query.watch();
});

// When filter changes, the stream automatically switches to the new query
filterStream.add(FilterState(
  category: 'Electronics',
  sortBy: 'price',
  ascending: true,
));
```

---

# Real-World Example

> **Complete e-commerce reactive system**

```dart
// lib/database/reactive_service.dart
import 'package:drift/drift.dart';
import 'package:rxdart/rxdart.dart';

class ReactiveService {
  final AppDatabase db;
  
  ReactiveService(this.db);

  // ==================== COMBINED DASHBOARD ====================
  
  // 👇 Complete reactive dashboard
  Stream<ReactiveDashboard> getReactiveDashboard() {
    // 1️⃣ User stats stream
    final usersStream = db.select(db.users)
      .watch()
      .map((users) {
        return UserStats(
          total: users.length,
          active: users.where((u) => u.isActive).length,
          verified: users.where((u) => u.isVerified).length,
        );
      });
    
    // 2️⃣ Order stats stream
    final ordersStream = db.select(db.orders)
      .watch()
      .map((orders) {
        return OrderStats(
          total: orders.length,
          pending: orders.where((o) => o.status == 'pending').length,
          completed: orders.where((o) => o.status == 'delivered').length,
          totalRevenue: orders.fold(0.0, (sum, o) => sum + o.total),
        );
      });
    
    // 3️⃣ Product stats stream
    final productsStream = db.select(db.products)
      .watch()
      .map((products) {
        return ProductStats(
          total: products.length,
          active: products.where((p) => p.isActive).length,
          lowStock: products.where((p) => p.stock < 10).length,
        );
      });
    
    // 4️⃣ Recent activity stream
    final recentOrdersStream = (db.select(db.orders)
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)])
      ..limit(10))
      .watch();
    
    // 👇 Combine all streams
    return Rx.combineLatest4(
      usersStream,
      ordersStream,
      productsStream,
      recentOrdersStream,
      (users, orders, products, recentOrders) {
        return ReactiveDashboard(
          userStats: users,
          orderStats: orders,
          productStats: products,
          recentOrders: recentOrders,
          timestamp: DateTime.now(),
        );
      },
    ).distinct((a, b) =>
      a.userStats.total == b.userStats.total &&
      a.orderStats.totalRevenue == b.orderStats.totalRevenue &&
      a.productStats.total == b.productStats.total &&
      a.recentOrders.length == b.recentOrders.length
    );
  }

  // ==================== REACTIVE SEARCH ====================
  
  // 👇 Reactive search with filters
  class ProductSearchFilters {
    final String query;
    final String category;
    final double? minPrice;
    final double? maxPrice;
    final bool? inStock;
    final String sortBy;
    final bool ascending;
    
    ProductSearchFilters({
      this.query = '',
      this.category = '',
      this.minPrice,
      this.maxPrice,
      this.inStock,
      this.sortBy = 'name',
      this.ascending = true,
    });
  }
  
  Stream<List<Product>> reactiveProductSearch(
    ProductSearchFilters filters,
  ) {
    return Rx.combineLatest2(
      _buildProductsStream(filters),
      _buildCategoriesStream(),
      (products, categories) {
        // Filter and sort products
        var filtered = products;
        
        if (filters.query.isNotEmpty) {
          filtered = filtered.where((p) =>
            p.name.toLowerCase().contains(filters.query.toLowerCase()) ||
            p.sku.toLowerCase().contains(filters.query.toLowerCase())
          ).toList();
        }
        
        if (filters.category.isNotEmpty) {
          filtered = filtered.where((p) =>
            p.category == filters.category
          ).toList();
        }
        
        if (filters.minPrice != null) {
          filtered = filtered.where((p) =>
            p.price >= filters.minPrice!
          ).toList();
        }
        
        if (filters.maxPrice != null) {
          filtered = filtered.where((p) =>
            p.price <= filters.maxPrice!
          ).toList();
        }
        
        if (filters.inStock != null) {
          filtered = filtered.where((p) =>
            p.stock > 0 == filters.inStock!
          ).toList();
        }
        
        // Sort
        switch (filters.sortBy) {
          case 'name':
            filtered.sort((a, b) => filters.ascending
              ? a.name.compareTo(b.name)
              : b.name.compareTo(a.name));
            break;
          case 'price':
            filtered.sort((a, b) => filters.ascending
              ? a.price.compareTo(b.price)
              : b.price.compareTo(a.price));
            break;
          case 'stock':
            filtered.sort((a, b) => filters.ascending
              ? a.stock.compareTo(b.stock)
              : b.stock.compareTo(a.stock));
            break;
        }
        
        return filtered;
      },
    );
  }
  
  Stream<List<Product>> _buildProductsStream(ProductSearchFilters filters) {
    final query = db.select(db.products)
      ..where((p) => p.isActive.equals(true));
    
    if (filters.category.isNotEmpty) {
      query.where((p) => p.category.equals(filters.category));
    }
    
    return query.watch();
  }
  
  Stream<List<String>> _buildCategoriesStream() {
    return db.select(db.products)
      .map((p) => p.category)
      .distinct()
      .watch()
      .map((categories) => categories.toList());
  }

  // ==================== REACTIVE ORDER PROCESSING ====================
  
  // 👇 Reactive order pipeline
  Stream<OrderProcessingState> reactiveOrderProcessing(int orderId) {
    final orderStream = (db.select(db.orders)
      ..where((o) => o.id.equals(orderId)))
      .watchSingle();
    
    final itemsStream = (db.select(db.orderItems)
      ..where((i) => i.orderId.equals(orderId)))
      .watch();
    
    final productsStream = itemsStream.switchMap((items) {
      if (items.isEmpty) return Stream.value([]);
      
      final productQueries = items.map((item) {
        return (db.select(db.products)
          ..where((p) => p.id.equals(item.productId)))
          .watchSingle();
      });
      
      return Rx.combineLatest(productQueries, (values) {
        return values.cast<Product>().toList();
      });
    });
    
    return Rx.combineLatest3(
      orderStream,
      itemsStream,
      productsStream,
      (order, items, products) {
        final itemDetails = items.map((item) {
          final product = products.firstWhere(
            (p) => p.id == item.productId,
            orElse: () => null,
          );
          return OrderItemDetail(
            item: item,
            product: product,
          );
        }).toList();
        
        return OrderProcessingState(
          order: order,
          items: itemDetails,
          totalItems: items.length,
          totalAmount: itemDetails.fold(0.0, (sum, d) => sum + d.item.total),
          status: order.status,
          isComplete: order.status == 'delivered',
        );
      },
    );
  }

  // ==================== REACTIVE USER PROFILE ====================
  
  // 👇 Complete reactive user profile
  Stream<ReactiveUserProfile> reactiveUserProfile(int userId) {
    final userStream = (db.select(db.users)
      ..where((u) => u.id.equals(userId)))
      .watchSingle();
    
    final profileStream = (db.select(db.userProfiles)
      ..where((p) => p.userId.equals(userId)))
      .watchSingleOrNull();
    
    final ordersStream = (db.select(db.orders)
      ..where((o) => o.userId.equals(userId))
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
      .watch();
    
    final reviewsStream = (db.select(db.reviews)
      ..where((r) => r.userId.equals(userId))
      ..orderBy([(r) => OrderingTerm.desc(r.createdAt)]))
      .watch();
    
    return Rx.combineLatest4(
      userStream,
      profileStream,
      ordersStream,
      reviewsStream,
      (user, profile, orders, reviews) {
        return ReactiveUserProfile(
          user: user,
          profile: profile,
          recentOrders: orders.take(5).toList(),
          recentReviews: reviews.take(5).toList(),
          orderCount: orders.length,
          reviewCount: reviews.length,
          totalSpent: orders.fold(0.0, (sum, o) => sum + o.total),
          averageRating: reviews.isEmpty
              ? 0.0
              : reviews.map((r) => r.rating).reduce((a, b) => a + b) / reviews.length,
        );
      },
    ).distinct((a, b) =>
      a.user.id == b.user.id &&
      a.orderCount == b.orderCount &&
      a.reviewCount == b.reviewCount &&
      a.totalSpent == b.totalSpent
    );
  }

  // ==================== REACTIVE NOTIFICATIONS ====================
  
  // 👇 Real-time notification system
  Stream<ReactiveNotification> reactiveNotificationSystem(int userId) {
    // Watch for new orders
    final newOrdersStream = (db.select(db.orders)
      ..where((o) => o.userId.equals(userId))
      ..where((o) => o.status.equals('pending')))
      .watch()
      .map((orders) => orders.length)
      .pairwise()
      .where((pair) => pair.first < pair.last)
      .map((pair) => ReactiveNotification(
        type: NotificationType.newOrder,
        title: 'New Order',
        message: 'You have a new order waiting',
        timestamp: DateTime.now(),
      ));
    
    // Watch for low stock alerts
    final lowStockStream = (db.select(db.products)
      ..where((p) => p.stock < const Variable(5))
      ..where((p) => p.isActive.equals(true)))
      .watch()
      .map((products) => products.length)
      .pairwise()
      .where((pair) => pair.first < pair.last)
      .map((pair) => ReactiveNotification(
        type: NotificationType.lowStock,
        title: 'Low Stock Alert',
        message: 'Products are running low on stock',
        timestamp: DateTime.now(),
      ));
    
    // Watch for price drops
    final priceDropStream = (db.select(db.products)
      ..where((p) => p.isActive.equals(true)))
      .watch()
      .map((products) => products.map((p) => p.price).toList())
      .pairwise()
      .where((pair) {
        // Check if any price dropped
        for (var i = 0; i < pair.first.length; i++) {
          if (pair.first[i] > pair.second[i]) return true;
        }
        return false;
      })
      .map((_) => ReactiveNotification(
        type: NotificationType.priceDrop,
        title: 'Price Drop Alert',
        message: 'Some products have dropped in price',
        timestamp: DateTime.now(),
      ));
    
    // Merge all notification streams
    return Rx.merge([
      newOrdersStream,
      lowStockStream,
      priceDropStream,
    ]);
  }
}

// ==================== DATA CLASSES ====================

class UserStats {
  final int total;
  final int active;
  final int verified;
  
  UserStats({
    required this.total,
    required this.active,
    required this.verified,
  });
}

class OrderStats {
  final int total;
  final int pending;
  final int completed;
  final double totalRevenue;
  
  OrderStats({
    required this.total,
    required this.pending,
    required this.completed,
    required this.totalRevenue,
  });
}

class ProductStats {
  final int total;
  final int active;
  final int lowStock;
  
  ProductStats({
    required this.total,
    required this.active,
    required this.lowStock,
  });
}

class ReactiveDashboard {
  final UserStats userStats;
  final OrderStats orderStats;
  final ProductStats productStats;
  final List<Order> recentOrders;
  final DateTime timestamp;
  
  ReactiveDashboard({
    required this.userStats,
    required this.orderStats,
    required this.productStats,
    required this.recentOrders,
    required this.timestamp,
  });
}

class OrderProcessingState {
  final Order order;
  final List<OrderItemDetail> items;
  final int totalItems;
  final double totalAmount;
  final String status;
  final bool isComplete;
  
  OrderProcessingState({
    required this.order,
    required this.items,
    required this.totalItems,
    required this.totalAmount,
    required this.status,
    required this.isComplete,
  });
}

class OrderItemDetail {
  final OrderItem item;
  final Product? product;
  
  OrderItemDetail({
    required this.item,
    this.product,
  });
}

class ReactiveUserProfile {
  final User user;
  final UserProfile? profile;
  final List<Order> recentOrders;
  final List<Review> recentReviews;
  final int orderCount;
  final int reviewCount;
  final double totalSpent;
  final double averageRating;
  
  ReactiveUserProfile({
    required this.user,
    this.profile,
    required this.recentOrders,
    required this.recentReviews,
    required this.orderCount,
    required this.reviewCount,
    required this.totalSpent,
    required this.averageRating,
  });
}

enum NotificationType {
  newOrder,
  lowStock,
  priceDrop,
}

class ReactiveNotification {
  final NotificationType type;
  final String title;
  final String message;
  final DateTime timestamp;
  
  ReactiveNotification({
    required this.type,
    required this.title,
    required this.message,
    required this.timestamp,
  });
}
```

```dart
// lib/ui/pages/reactive_dashboard_page.dart
class ReactiveDashboardPage extends StatelessWidget {
  final ReactiveService reactiveService;
  
  const ReactiveDashboardPage({required this.reactiveService});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Reactive Dashboard'),
        backgroundColor: Colors.blue[800],
      ),
      body: StreamBuilder<ReactiveDashboard>(
        stream: reactiveService.getReactiveDashboard(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return Center(child: CircularProgressIndicator());
          }
          
          final dashboard = snapshot.data!;
          
          return SingleChildScrollView(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                // Stats row
                Row(
                  children: [
                    _buildStatCard(
                      'Users',
                      dashboard.userStats.total.toString(),
                      'Active: ${dashboard.userStats.active}',
                      Icons.people,
                      Colors.blue,
                    ),
                    _buildStatCard(
                      'Revenue',
                      '\$${dashboard.orderStats.totalRevenue.toStringAsFixed(0)}',
                      'Orders: ${dashboard.orderStats.total}',
                      Icons.attach_money,
                      Colors.green,
                    ),
                  ],
                ),
                SizedBox(height: 16),
                Row(
                  children: [
                    _buildStatCard(
                      'Products',
                      dashboard.productStats.total.toString(),
                      'Active: ${dashboard.productStats.active}',
                      Icons.inventory,
                      Colors.orange,
                    ),
                    _buildStatCard(
                      'Pending',
                      dashboard.orderStats.pending.toString(),
                      'Orders waiting',
                      Icons.pending,
                      Colors.red,
                    ),
                  ],
                ),
                SizedBox(height: 16),
                
                // Recent Orders
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
                        ...dashboard.recentOrders.map((order) {
                          return ListTile(
                            leading: Icon(Icons.receipt),
                            title: Text(order.orderNumber),
                            subtitle: Text('Status: ${order.status}'),
                            trailing: Text('\$${order.total.toStringAsFixed(2)}'),
                          );
                        }),
                      ],
                    ),
                  ),
                ),
                SizedBox(height: 16),
                
                // Timestamp
                Text(
                  'Last updated: ${_formatTime(dashboard.timestamp)}',
                  style: TextStyle(
                    fontSize: 12,
                    color: Colors.grey,
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
  
  Widget _buildStatCard(
    String label,
    String value,
    String subtitle,
    IconData icon,
    Color color,
  ) {
    return Expanded(
      child: Container(
        margin: EdgeInsets.all(4),
        padding: EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: color.withOpacity(0.1),
          borderRadius: BorderRadius.circular(12),
          border: Border.all(color: color.withOpacity(0.3)),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Icon(icon, color: color, size: 20),
                SizedBox(width: 8),
                Text(
                  label,
                  style: TextStyle(
                    fontSize: 14,
                    color: Colors.grey[600],
                  ),
                ),
              ],
            ),
            SizedBox(height: 4),
            Text(
              value,
              style: TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
                color: color,
              ),
            ),
            Text(
              subtitle,
              style: TextStyle(
                fontSize: 12,
                color: Colors.grey,
              ),
            ),
          ],
        ),
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

- **Use combineLatest** – For dependent streams
- **Use switchMap** – For search/filter
- **Use debounce** – For rapid changes
- **Use distinct** – Avoid duplicate emissions
- **Use share** – Reuse streams
- **Use doOn** – For side effects
- **Use handleError** – For error handling
- **Use BehaviorSubject** – For state management

---

# Common Mistakes

## Mistake 1: Creating streams unnecessarily

Wrong:
```dart
// 🚫 New stream on every build
StreamBuilder(
  stream: (select(users)..where(...)).watch(),
)
```

Correct:
```dart
// ✅ Create once, reuse
final stream = (select(users)..where(...)).watch();
StreamBuilder(stream: stream)
```

## Mistake 2: Not disposing subjects

Wrong:
```dart
// 🚫 Memory leak
final subject = BehaviorSubject();
// Never closed
```

Correct:
```dart
// ✅ Dispose subjects
final subject = BehaviorSubject();
subject.close();
```

## Mistake 3: Combining non-reactive data

Wrong:
```dart
// 🚫 Static data in reactive flow
Rx.combineLatest2(stream1, stream2, (a, b) {
  final staticData = getStaticData(); // Won't update
  return combine(a, b, staticData);
});
```

Correct:
```dart
// ✅ Everything reactive
final staticDataStream = Stream.value(getStaticData());
Rx.combineLatest3(stream1, stream2, staticDataStream, (a, b, c) {
  return combine(a, b, c);
});
```

---

# Summary

| Operator | Purpose | Example |
|----------|---------|---------|
| `combineLatest` | Combine streams | Dashboard |
| `switchMap` | Switch streams | Search |
| `debounce` | Wait for pause | Typing |
| `distinct` | Remove duplicates | Updates |
| `merge` | Combine emissions | Notifications |
| `pairwise` | Compare values | Change detection |

---

# Next Steps

Now you understand reactive queries, let's dive deeper:

- [Watching Multiple Tables](link) – Multi-table watches
- [Stream Performance](link) – Performance optimization
- [Transactions](link) – Transaction management

---

# Did You Know?

- **Reactive queries are composable** – Build complex flows

- **Reactive queries are lazy** – Only start on subscription

- **Reactive queries are cancellable** – Free resources

- **Reactive queries are testable** – With stream testing

- **Reactive queries are efficient** – Only compute when needed

- **Reactive queries are powerful** – Combine any number of streams

- **Reactive queries are the future** – Of UI development

- **Reactive queries are production-ready** – Used in large apps

---

