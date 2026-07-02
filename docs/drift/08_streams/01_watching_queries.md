## Watching Queries

**Building reactive queries with Drift streams**

---

# What is it?

**Watching Queries** is one of Drift's most powerful features. By converting any query into a `Stream`, you can automatically receive updates whenever the underlying data changes. This enables real-time UI updates without manual refreshes, polling, or complex state management.

> **Think of Watching Queries like "live sports scores"** – instead of constantly refreshing the page to see if anything changed, you just watch the screen and it updates automatically whenever a goal is scored.

```dart
// 👇 Watch all active users
final stream = (select(users)
  ..where((u) => u.isActive.equals(true)))
  .watch();

// 👇 Listen for changes
stream.listen((users) {
  print('Active users updated: ${users.length}');
  // UI automatically updates with new data
});

// Whenever a user is inserted, updated, or deleted:
// The stream emits a new list automatically!
```

> **What's happening here?**
> - **`.watch()`** – Converts any query to a Stream
> - **Automatic updates** – Stream emits on data changes
> - **Efficient** – Only emits when data actually changes
> - **Type-safe** – Stream emits typed lists
> - **Real-time** – Perfect for reactive UIs

---

# Why does it exist?

- **Real-time UI** – Update UI automatically
- **Reactive Programming** – Respond to data changes
- **Simplified State Management** – No manual refresh
- **Performance** – Only update when data changes
- **User Experience** – Always show latest data
- **Reduced Boilerplate** – Less manual update code

---

# Basic Watching

> **Simple watch operations**

## Watch All Records

```dart
// 👇 Watch all users
final stream = select(users).watch();

stream.listen((users) {
  print('All users: ${users.length}');
});

// 👇 Watch active users only
final activeStream = (select(users)
  ..where((u) => u.isActive.equals(true)))
  .watch();

activeStream.listen((users) {
  print('Active users: ${users.length}');
});
```

## Watch Single Record

```dart
// 👇 Watch a specific user
final userStream = (select(users)
  ..where((u) => u.id.equals(1)))
  .watchSingle();

userStream.listen((user) {
  print('User updated: ${user.name}');
});

// 👇 Watch with null (safe)
final userStreamOrNull = (select(users)
  ..where((u) => u.id.equals(999)))
  .watchSingleOrNull();

userStreamOrNull.listen((user) {
  if (user != null) {
    print('User found: ${user.name}');
  } else {
    print('User not found');
  }
});
```

## Watch with Filtering

```dart
// 👇 Watch users with complex filters
final stream = (select(users)
  ..where((u) => 
    u.age > const Variable(18) &
    u.isActive.equals(true) &
    u.isVerified.equals(true)
  )
  ..orderBy([(u) => OrderingTerm.desc(u.createdAt)]))
  .watch();

stream.listen((users) {
  print('Verified active adults: ${users.length}');
});
```

---

# Watch with Joins

> **Watching joined queries**

## Watch Joined Query

```dart
// 👇 Watch users with their orders
final query = db.select(db.users).join([
  innerJoin(db.orders, db.orders.userId.equals(db.users.id)),
]);

final stream = query.watch();

stream.listen((rows) {
  for (final row in rows) {
    final user = row.readTable(db.users);
    final order = row.readTable(db.orders);
    print('${user.name} -> ${order.orderNumber}');
  }
});
```

## Watch Nested Results

```dart
// 👇 Watch grouped user orders
final query = db.select(db.users).join([
  innerJoin(db.orders, db.orders.userId.equals(db.users.id)),
]);

query.orderBy([
  (u) => OrderingTerm.asc(db.users.name),
  (o) => OrderingTerm.desc(db.orders.orderDate),
]);

// Convert to nested structure on each update
final stream = query.watch().map((rows) {
  final userMap = <int, UserWithOrders>{};
  
  for (final row in rows) {
    final user = row.readTable(db.users);
    
    if (!userMap.containsKey(user.id)) {
      userMap[user.id] = UserWithOrders(
        user: user,
        orders: [],
      );
    }
    
    final order = row.readTable(db.orders);
    userMap[user.id]!.orders.add(order);
  }
  
  return userMap.values.toList();
});

stream.listen((usersWithOrders) {
  // UI updates with nested data
  for (final userData in usersWithOrders) {
    print('User: ${userData.user.name}');
    print('Orders: ${userData.orders.length}');
  }
});
```

---

# Real-World Example

> **Complete e-commerce watch system**

```dart
// lib/database/watch_service.dart
import 'package:drift/drift.dart';

class WatchService {
  final AppDatabase db;
  
  WatchService(this.db);

  // ==================== USER WATCHES ====================
  
  // 👇 Watch all active users
  Stream<List<User>> watchActiveUsers() {
    return (db.select(db.users)
      ..where((u) => u.isActive.equals(true))
      ..orderBy([(u) => OrderingTerm.asc(u.name)]))
      .watch();
  }
  
  // 👇 Watch a single user
  Stream<User?> watchUser(int userId) {
    return (db.select(db.users)
      ..where((u) => u.id.equals(userId)))
      .watchSingleOrNull();
  }
  
  // 👇 Watch users by search
  Stream<List<User>> watchUsersBySearch(String query) {
    return (db.select(db.users)
      ..where((u) => 
        u.name.like('%$query%') |
        u.email.like('%$query%')
      )
      ..orderBy([(u) => OrderingTerm.asc(u.name)]))
      .watch();
  }

  // ==================== ORDER WATCHES ====================
  
  // 👇 Watch orders for a user
  Stream<List<Order>> watchUserOrders(int userId) {
    return (db.select(db.orders)
      ..where((o) => o.userId.equals(userId))
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
      .watch();
  }
  
  // 👇 Watch orders by status
  Stream<List<Order>> watchOrdersByStatus(String status) {
    return (db.select(db.orders)
      ..where((o) => o.status.equals(status))
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
      .watch();
  }
  
  // 👇 Watch order with details
  Stream<OrderWithItems> watchOrderDetails(int orderId) {
    final query = db.select(db.orders).join([
      innerJoin(db.users, db.users.id.equals(db.orders.userId)),
      leftJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
      leftJoin(db.products, db.products.id.equals(db.orderItems.productId)),
    ]);
    
    query.where((o) => db.orders.id.equals(orderId));
    
    return query.watch().map((rows) {
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
    });
  }

  // ==================== PRODUCT WATCHES ====================
  
  // 👇 Watch active products
  Stream<List<Product>> watchActiveProducts() {
    return (db.select(db.products)
      ..where((p) => p.isActive.equals(true))
      ..orderBy([(p) => OrderingTerm.asc(p.name)]))
      .watch();
  }
  
  // 👇 Watch products in category
  Stream<List<Product>> watchProductsByCategory(int categoryId) {
    final query = db.select(db.products).join([
      innerJoin(
        db.productCategories,
        db.productCategories.productId.equals(db.products.id),
      )
    ]);
    
    query.where((pc) => db.productCategories.categoryId.equals(categoryId));
    query.where((p) => db.products.isActive.equals(true));
    query.orderBy([(p) => OrderingTerm.asc(db.products.name)]);
    
    return query.watch().map((rows) {
      return rows.map((row) => row.readTable(db.products)).toList();
    });
  }
  
  // 👇 Watch low stock products
  Stream<List<Product>> watchLowStockProducts(int threshold) {
    return (db.select(db.products)
      ..where((p) => p.stock < const Variable(threshold))
      ..where((p) => p.isActive.equals(true))
      ..orderBy([(p) => OrderingTerm.asc(p.stock)]))
      .watch();
  }

  // ==================== DASHBOARD WATCHES ====================
  
  // 👇 Watch dashboard stats
  Stream<DashboardStats> watchDashboardStats() {
    final query = db.select(db.users)
      .join([
        leftJoin(db.orders, db.orders.userId.equals(db.users.id)),
      ]);
    
    return query.watch().map((rows) {
      final totalUsers = rows.length;
      final activeUsers = rows.where((r) => 
        r.readTable(db.users).isActive).length;
      
      final orders = rows
        .map((r) => r.readTableOrNull(db.orders))
        .where((o) => o != null)
        .cast<Order>()
        .toList();
      
      final totalOrders = orders.length;
      final totalRevenue = orders.fold(0.0, (sum, o) => sum + o.total);
      
      return DashboardStats(
        totalUsers: totalUsers,
        activeUsers: activeUsers,
        totalOrders: totalOrders,
        totalRevenue: totalRevenue,
      );
    });
  }
  
  // 👇 Watch real-time notifications
  Stream<List<Notification>> watchNotifications(int userId) {
    return (db.select(db.notifications)
      ..where((n) => n.userId.equals(userId))
      ..where((n) => n.isRead.equals(false))
      ..orderBy([(n) => OrderingTerm.desc(n.createdAt)]))
      .watch();
  }

  // ==================== ADVANCED WATCHES ====================
  
  // 👇 Watch with debounce
  Stream<List<User>> watchUsersWithDebounce({
    String? search,
    Duration debounceTime = const Duration(milliseconds: 500),
  }) {
    final query = db.select(db.users)
      ..where((u) => u.isActive.equals(true));
    
    if (search != null && search.isNotEmpty) {
      query.where((u) => u.name.like('%$search%'));
    }
    
    return query.watch().debounce(debounceTime);
  }
  
  // 👇 Watch and transform
  Stream<List<UserBasicInfo>> watchUserBasicInfo() {
    return (db.select(db.users)
      ..where((u) => u.isActive.equals(true))
      ..orderBy([(u) => OrderingTerm.asc(u.name)]))
      .watch()
      .map((users) {
        return users.map((user) {
          return UserBasicInfo(
            id: user.id,
            name: user.name,
            email: user.email,
            initials: user.name.split(' ').map((s) => s[0]).join(),
          );
        }).toList();
      });
  }
  
  // 👇 Watch with error handling
  Stream<List<User>> watchUsersSafe() {
    return (db.select(db.users)
      ..where((u) => u.isActive.equals(true)))
      .watch()
      .handleError((error) {
        print('Watch error: $error');
        return <User>[];
      });
  }
}

// ==================== DATA CLASSES ====================

class OrderWithItems {
  final Order order;
  final User user;
  final List<OrderItemWithProduct> items;
  
  OrderWithItems({
    required this.order,
    required this.user,
    required this.items,
  });
}

class DashboardStats {
  final int totalUsers;
  final int activeUsers;
  final int totalOrders;
  final double totalRevenue;
  
  DashboardStats({
    required this.totalUsers,
    required this.activeUsers,
    required this.totalOrders,
    required this.totalRevenue,
  });
}

class UserBasicInfo {
  final int id;
  final String name;
  final String email;
  final String initials;
  
  UserBasicInfo({
    required this.id,
    required this.name,
    required this.email,
    required this.initials,
  });
}
```

```dart
// lib/ui/pages/dashboard_page.dart
class DashboardPage extends StatefulWidget {
  final WatchService watchService;
  
  const DashboardPage({required this.watchService});
  
  @override
  _DashboardPageState createState() => _DashboardPageState();
}

class _DashboardPageState extends State<DashboardPage> {
  late Stream<DashboardStats> _statsStream;
  late Stream<List<User>> _usersStream;
  late Stream<List<Order>> _ordersStream;
  
  @override
  void initState() {
    super.initState();
    _statsStream = widget.watchService.watchDashboardStats();
    _usersStream = widget.watchService.watchActiveUsers();
    _ordersStream = widget.watchService.watchOrdersByStatus('pending');
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Dashboard'),
        actions: [
          IconButton(
            icon: Icon(Icons.refresh),
            onPressed: () {
              // Streams automatically update
              // No manual refresh needed!
            },
          ),
        ],
      ),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // Stats Cards
            StreamBuilder<DashboardStats>(
              stream: _statsStream,
              builder: (context, snapshot) {
                if (!snapshot.hasData) {
                  return CircularProgressIndicator();
                }
                
                final stats = snapshot.data!;
                return Row(
                  children: [
                    Expanded(
                      child: _buildStatCard(
                        'Users',
                        '${stats.totalUsers}',
                        Icons.people,
                        Colors.blue,
                      ),
                    ),
                    Expanded(
                      child: _buildStatCard(
                        'Active',
                        '${stats.activeUsers}',
                        Icons.person,
                        Colors.green,
                      ),
                    ),
                    Expanded(
                      child: _buildStatCard(
                        'Orders',
                        '${stats.totalOrders}',
                        Icons.shopping_cart,
                        Colors.orange,
                      ),
                    ),
                    Expanded(
                      child: _buildStatCard(
                        'Revenue',
                        '\$${stats.totalRevenue.toStringAsFixed(0)}',
                        Icons.attach_money,
                        Colors.purple,
                      ),
                    ),
                  ],
                );
              },
            ),
            SizedBox(height: 16),
            
            // Active Users
            Card(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      'Active Users',
                      style: TextStyle(
                        fontSize: 18,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    SizedBox(height: 8),
                    StreamBuilder<List<User>>(
                      stream: _usersStream,
                      builder: (context, snapshot) {
                        if (!snapshot.hasData) {
                          return CircularProgressIndicator();
                        }
                        
                        final users = snapshot.data!;
                        return Column(
                          children: users.take(5).map((user) {
                            return ListTile(
                              leading: CircleAvatar(
                                child: Text(user.name[0]),
                              ),
                              title: Text(user.name),
                              subtitle: Text(user.email),
                            );
                          }).toList(),
                        );
                      },
                    ),
                  ],
                ),
              ),
            ),
            SizedBox(height: 16),
            
            // Pending Orders
            Card(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      'Pending Orders',
                      style: TextStyle(
                        fontSize: 18,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    SizedBox(height: 8),
                    StreamBuilder<List<Order>>(
                      stream: _ordersStream,
                      builder: (context, snapshot) {
                        if (!snapshot.hasData) {
                          return CircularProgressIndicator();
                        }
                        
                        final orders = snapshot.data!;
                        return Column(
                          children: orders.take(5).map((order) {
                            return ListTile(
                              leading: Icon(Icons.receipt),
                              title: Text(order.orderNumber),
                              subtitle: Text('\$${order.total.toStringAsFixed(2)}'),
                              trailing: Text(order.status),
                            );
                          }).toList(),
                        );
                      },
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildStatCard(
    String label,
    String value,
    IconData icon,
    Color color,
  ) {
    return Container(
      margin: EdgeInsets.all(4),
      padding: EdgeInsets.all(12),
      decoration: BoxDecoration(
        color: color.withOpacity(0.1),
        borderRadius: BorderRadius.circular(8),
        border: Border.all(color: color.withOpacity(0.3)),
      ),
      child: Column(
        children: [
          Icon(icon, color: color),
          SizedBox(height: 4),
          Text(
            value,
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
              color: color,
            ),
          ),
          Text(
            label,
            style: TextStyle(fontSize: 12, color: Colors.grey[600]),
          ),
        ],
      ),
    );
  }
}
```

---

# Best Practices

- **Use `watch()` for UI data** – Automatic updates
- **Use `watchSingle()` for detail views** – Single record updates
- **Use `watchSingleOrNull()`** – For optional records
- **Transform streams** – Map to UI-friendly models
- **Debounce streams** – For search/filter
- **Handle errors** – Catch and handle stream errors
- **Dispose subscriptions** – Avoid memory leaks
- **Use StreamBuilder** – For Flutter widgets
- **Use distinct** – Avoid duplicate emissions

---

# Common Mistakes

## Mistake 1: Not disposing subscriptions

Wrong:
```dart
// 🚫 Memory leak
final subscription = stream.listen((data) {});
// Never disposed
```

Correct:
```dart
// ✅ Dispose on dispose
late StreamSubscription subscription;

@override
void initState() {
  super.initState();
  subscription = stream.listen((data) {});
}

@override
void dispose() {
  subscription.cancel();
  super.dispose();
}
```

## Mistake 2: Watching with expensive transformations

Wrong:
```dart
// 🚫 Expensive transformation on every update
stream.watch().map((data) {
  // Heavy computation here
  return expensiveTransform(data);
});
```

Correct:
```dart
// ✅ Use distinct to avoid unnecessary updates
stream
  .distinct()
  .map((data) => expensiveTransform(data));
```

## Mistake 3: Not handling errors

Wrong:
```dart
// 🚫 Stream errors crash the app
stream.listen((data) {});
```

Correct:
```dart
// ✅ Handle errors
stream.listen(
  (data) {},
  onError: (error) {
    print('Stream error: $error');
    // Show error UI
  },
);
```

---

# Summary

| Method | Purpose | Returns |
|--------|---------|---------|
| `.watch()` | Watch multiple records | `Stream<List<T>>` |
| `.watchSingle()` | Watch single record | `Stream<T>` |
| `.watchSingleOrNull()` | Watch optional record | `Stream<T?>` |
| `.watch()` with join | Watch joined data | `Stream<List<Row>>` |

---

# Next Steps

Now you understand watching queries, let's dive deeper:

- [Stream Updates](link) – Understanding stream behavior
- [Reactive Queries](link) – Advanced reactive patterns
- [Stream Performance](link) – Performance optimization

---

# Did You Know?

- **Watch queries are efficient** – Only emit on actual changes

- **Watch queries track dependencies** – Across multiple tables

- **Watch queries can be combined** – With RxDart operators

- **Watch queries are lazy** – Only start on subscription

- **Watch queries work with joins** – Complex data relationships

- **Watch queries support transactions** – Emit after commit

- **Watch queries are production-ready** – Used in large apps

- **Watch queries are the main feature** – Of Drift's reactive nature

---

