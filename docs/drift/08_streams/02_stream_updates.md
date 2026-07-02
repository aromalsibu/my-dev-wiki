## Stream Updates

**Understanding how Drift streams update and behave**

---

# What is it?

**Stream Updates** refer to how Drift's reactive streams emit new data when the underlying database changes. Understanding when and how streams update is crucial for building efficient reactive applications. Drift uses a sophisticated dependency tracking system to know exactly when to emit new data.

> **Think of Stream Updates like "live notifications"** – you only get notified when something actually changes, not when nothing happens. And you get the complete, updated data every time.

```dart
// 👇 A stream that updates on database changes
final stream = (select(users)
  ..where((u) => u.isActive.equals(true)))
  .watch();

stream.listen((users) {
  // 👇 This runs when:
  // 1. A user is inserted
  // 2. A user is updated (and changes affect the result)
  // 3. A user is deleted (and changes affect the result)
  // ❌ DOES NOT run when:
  // - A user is updated but still matches the filter
  // - A user outside the filter changes
  print('Users updated: ${users.length}');
});

// Insert new active user -> Stream emits
await db.into(db.users).insert(UsersCompanion.insert(
  name: 'New User',
  email: 'new@example.com',
  isActive: true,
));

// Update user to inactive -> Stream DOES NOT emit
// (User no longer matches filter)
await (db.update(db.users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(isActive: Value(false)));
```

> **What's happening here?**
> - **Smart updates** – Stream only emits when relevant data changes
> - **Dependency tracking** – Drift tracks which tables/queries are affected
> - **Filter awareness** – Updates respect WHERE clauses
> - **Complete data** – Each emission contains the full updated dataset

---

# Why does it exist?

- **Efficiency** – Only emit when necessary
- **Performance** – Reduce unnecessary UI updates
- **Accuracy** – Always reflect current data state
- **Smart Tracking** – Know exactly what affects each query
- **Resource Optimization** – Minimize database queries
- **User Experience** – Smooth, responsive UI

---

# When Streams Update

> **Understanding stream emission triggers**

## Insert Triggers

```dart
// 👇 This stream watches active users
final stream = (select(users)
  ..where((u) => u.isActive.equals(true)))
  .watch();

// ✅ Inserts that affect the stream:
// 1. Insert active user -> Stream emits
await db.into(db.users).insert(
  UsersCompanion.insert(
    name: 'Active User',
    email: 'active@example.com',
    isActive: true,
  ),
);

// ❌ Inserts that DO NOT affect the stream:
// 2. Insert inactive user -> Stream DOES NOT emit
await db.into(db.users).insert(
  UsersCompanion.insert(
    name: 'Inactive User',
    email: 'inactive@example.com',
    isActive: false,
  ),
);
```

## Update Triggers

```dart
// 👇 Watch users over 18
final stream = (select(users)
  ..where((u) => u.age > const Variable(18)))
  .watch();

// ✅ Updates that affect the stream:
// 1. User age changes from 17 to 19 -> Stream emits
await (db.update(db.users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(age: Value(19)));

// ❌ Updates that DO NOT affect the stream:
// 2. User name changes (age unchanged) -> Stream DOES NOT emit
await (db.update(db.users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(name: Value('New Name')));
```

## Delete Triggers

```dart
// 👇 Watch active verified users
final stream = (select(users)
  ..where((u) => 
    u.isActive.equals(true) &
    u.isVerified.equals(true)
  ))
  .watch();

// ✅ Deletes that affect the stream:
// 1. Delete active verified user -> Stream emits
await (db.delete(db.users)..where((u) => u.id.equals(1))).go();

// ❌ Deletes that DO NOT affect the stream:
// 2. Delete inactive user -> Stream DOES NOT emit
await (db.delete(db.users)..where((u) => u.id.equals(999))).go();
```

---

# Stream Behavior Patterns

> **Common stream update patterns**

## Pattern 1: Distinct Emissions

```dart
// 👇 Avoid duplicate emissions
final stream = (select(users)
  ..where((u) => u.isActive.equals(true)))
  .watch()
  .distinct();

// Even if multiple changes happen quickly,
// only the final state is emitted once
```

## Pattern 2: Debounced Updates

```dart
// 👇 Debounce rapid updates
final stream = (select(users)
  ..where((u) => u.isActive.equals(true)))
  .watch()
  .debounce(Duration(milliseconds: 300));

// If 10 users are inserted in quick succession,
// only one emission after 300ms of no changes
```

## Pattern 3: Throttled Updates

```dart
// 👇 Throttle updates
final stream = (select(users)
  ..where((u) => u.isActive.equals(true)))
  .watch()
  .throttle(Duration(milliseconds: 500));

// Emits at most once every 500ms,
// even if changes happen more frequently
```

---

# Real-World Example

> **Complete e-commerce stream updates system**

```dart
// lib/database/stream_update_service.dart
import 'package:drift/drift.dart';
import 'package:rxdart/rxdart.dart';

class StreamUpdateService {
  final AppDatabase db;
  final Map<String, StreamSubscription> _subscriptions = {};
  
  StreamUpdateService(this.db);

  // ==================== UNDERSTANDING STREAM UPDATES ====================
  
  // 👇 Demo: See when streams update
  Future<void> demonstrateStreamUpdates() async {
    print('===== DEMONSTRATING STREAM UPDATES =====');
    
    // 1️⃣ Watch active users
    final activeStream = (db.select(db.users)
      ..where((u) => u.isActive.equals(true)))
      .watch();
    
    activeStream.listen((users) {
      print('📢 Active users updated: ${users.length} users');
    });
    
    // 2️⃣ Watch verified users
    final verifiedStream = (db.select(db.users)
      ..where((u) => u.isVerified.equals(true)))
      .watch();
    
    verifiedStream.listen((users) {
      print('📢 Verified users updated: ${users.length} users');
    });
    
    // 3️⃣ Insert new active user -> Both streams update
    print('\n1️⃣ Insert active user...');
    await db.into(db.users).insert(
      UsersCompanion.insert(
        name: 'Active User',
        email: 'active@example.com',
        isActive: true,
        isVerified: false,
      ),
    );
    // Both streams emit because user is active
    // Verified stream emits because count changed (even though user is not verified)
    
    // 4️⃣ Insert inactive user -> Only verified stream updates
    print('\n2️⃣ Insert inactive but verified user...');
    await db.into(db.users).insert(
      UsersCompanion.insert(
        name: 'Inactive Verified',
        email: 'inactive@example.com',
        isActive: false,
        isVerified: true,
      ),
    );
    // Active stream DOES NOT emit (user not active)
    // Verified stream emits (user is verified)
    
    // 5️⃣ Update user from inactive to active -> Both streams update
    print('\n3️⃣ Update user to active...');
    await (db.update(db.users)..where((u) => u.id.equals(2)))
      .write(UsersCompanion(
        isActive: Value(true),
        isVerified: Value(true),
      ));
    // Both streams emit (user now active AND verified)
  }

  // ==================== ORDER STREAM UPDATES ====================
  
  // 👇 Watch pending orders with timestamps
  Stream<List<Order>> watchPendingOrders() {
    return (db.select(db.orders)
      ..where((o) => o.status.equals('pending'))
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
      .watch()
      .doOnListen(() => print('📡 Watching pending orders started'))
      .doOnCancel(() => print('📡 Watching pending orders stopped'))
      .doOnData((orders) {
        print('📦 Pending orders updated: ${orders.length} orders');
      })
      .doOnError((error) {
        print('❌ Error in pending orders stream: $error');
      });
  }
  
  // 👇 Watch high-value orders with debounce
  Stream<List<Order>> watchHighValueOrders(
    double minValue,
    Duration debounceTime,
  ) {
    return (db.select(db.orders)
      ..where((o) => o.total > const Variable(minValue))
      ..where((o) => o.status.isNotValue('cancelled'))
      ..orderBy([(o) => OrderingTerm.desc(o.total)]))
      .watch()
      .debounce(debounceTime)
      .doOnData((orders) {
        print('💰 High-value orders updated: ${orders.length} orders');
        if (orders.isNotEmpty) {
          print('   Highest: \$${orders.first.total}');
        }
      });
  }

  // ==================== PRODUCT STREAM UPDATES ====================
  
  // 👇 Watch low stock with throttle
  Stream<List<Product>> watchLowStockThrottled(
    int threshold,
    Duration throttleTime,
  ) {
    return (db.select(db.products)
      ..where((p) => p.stock < const Variable(threshold))
      ..where((p) => p.isActive.equals(true))
      ..orderBy([(p) => OrderingTerm.asc(p.stock)]))
      .watch()
      .throttle(throttleTime)
      .doOnData((products) {
        print('⚠️ Low stock updated: ${products.length} products');
        for (final product in products.take(3)) {
          print('   ${product.name}: ${product.stock} units');
        }
      });
  }
  
  // 👇 Watch product with price changes
  Stream<ProductPriceHistory> watchProductPriceChanges(int productId) {
    return (db.select(db.products)
      ..where((p) => p.id.equals(productId)))
      .watchSingle()
      .scan<ProductPriceHistory>(
        ProductPriceHistory(initial: true, price: 0.0, changes: []),
        (history, product) {
          final newPrice = product.price;
          final priceHistory = history.changes;
          
          if (!history.initial && history.price != newPrice) {
            priceHistory.add(PriceChange(
              oldPrice: history.price,
              newPrice: newPrice,
              timestamp: DateTime.now(),
            ));
          }
          
          return ProductPriceHistory(
            initial: false,
            price: newPrice,
            changes: priceHistory,
          );
        },
      )
      .doOnData((history) {
        if (history.changes.isNotEmpty) {
          final lastChange = history.changes.last;
          print('💲 Price changed: \$${lastChange.oldPrice} -> \$${lastChange.newPrice}');
        }
      });
  }

  // ==================== USER STREAM UPDATES ====================
  
  // 👇 Watch user profile changes
  Stream<UserProfileUpdate> watchUserProfileChanges(int userId) {
    final query = db.select(db.users).join([
      leftJoin(db.userProfiles, db.userProfiles.userId.equals(db.users.id)),
    ]);
    
    query.where((u) => db.users.id.equals(userId));
    
    return query.watch().map((rows) {
      if (rows.isEmpty) return null;
      
      final user = rows.first.readTable(db.users);
      final profile = rows.first.readTableOrNull(db.userProfiles);
      
      return UserProfileUpdate(
        user: user,
        profile: profile,
        timestamp: DateTime.now(),
      );
    }).where((update) => update != null).cast<UserProfileUpdate>()
      .distinct((a, b) => 
        a.user.id == b.user.id &&
        a.profile?.fullName == b.profile?.fullName &&
        a.profile?.bio == b.profile?.bio
      )
      .doOnData((update) {
        print('👤 User ${update.user.name} profile updated');
        if (update.profile != null) {
          print('   Full Name: ${update.profile!.fullName}');
        }
      });
  }
  
  // 👇 Watch user activity (combined streams)
  Stream<UserActivity> watchUserActivity(int userId) {
    final ordersStream = (db.select(db.orders)
      ..where((o) => o.userId.equals(userId))
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
      .watch();
    
    final reviewsStream = (db.select(db.reviews)
      ..where((r) => r.userId.equals(userId))
      ..orderBy([(r) => OrderingTerm.desc(r.createdAt)]))
      .watch();
    
    return Rx.combineLatest2<List<Order>, List<Review>, UserActivity>(
      ordersStream,
      reviewsStream,
      (orders, reviews) {
        return UserActivity(
          userId: userId,
          recentOrders: orders.take(5).toList(),
          recentReviews: reviews.take(5).toList(),
          totalOrders: orders.length,
          totalReviews: reviews.length,
          lastActivity: _getLastActivity(orders, reviews),
          timestamp: DateTime.now(),
        );
      },
    ).distinct((a, b) =>
      a.totalOrders == b.totalOrders &&
      a.totalReviews == b.totalReviews &&
      a.lastActivity == b.lastActivity
    );
  }
  
  DateTime _getLastActivity(List<Order> orders, List<Review> reviews) {
    final orderDate = orders.isNotEmpty ? orders.first.orderDate : DateTime(1970);
    final reviewDate = reviews.isNotEmpty ? reviews.first.createdAt : DateTime(1970);
    return orderDate.isAfter(reviewDate) ? orderDate : reviewDate;
  }

  // ==================== CLEANUP ====================
  
  void dispose() {
    for (final subscription in _subscriptions.values) {
      subscription.cancel();
    }
    _subscriptions.clear();
    print('🧹 All stream subscriptions disposed');
  }
}

// ==================== DATA CLASSES ====================

class ProductPriceHistory {
  final bool initial;
  final double price;
  final List<PriceChange> changes;
  
  ProductPriceHistory({
    required this.initial,
    required this.price,
    required this.changes,
  });
}

class PriceChange {
  final double oldPrice;
  final double newPrice;
  final DateTime timestamp;
  
  PriceChange({
    required this.oldPrice,
    required this.newPrice,
    required this.timestamp,
  });
}

class UserProfileUpdate {
  final User user;
  final UserProfile? profile;
  final DateTime timestamp;
  
  UserProfileUpdate({
    required this.user,
    this.profile,
    required this.timestamp,
  });
}

class UserActivity {
  final int userId;
  final List<Order> recentOrders;
  final List<Review> recentReviews;
  final int totalOrders;
  final int totalReviews;
  final DateTime lastActivity;
  final DateTime timestamp;
  
  UserActivity({
    required this.userId,
    required this.recentOrders,
    required this.recentReviews,
    required this.totalOrders,
    required this.totalReviews,
    required this.lastActivity,
    required this.timestamp,
  });
}
```

```dart
// lib/ui/widgets/stream_demo_widget.dart
class StreamDemoWidget extends StatefulWidget {
  final StreamUpdateService service;
  
  const StreamDemoWidget({required this.service});
  
  @override
  _StreamDemoWidgetState createState() => _StreamDemoWidgetState();
}

class _StreamDemoWidgetState extends State<StreamDemoWidget> {
  final Map<String, dynamic> _logs = {};
  final List<String> _messages = [];
  final ScrollController _scrollController = ScrollController();
  
  @override
  void initState() {
    super.initState();
    _setupStreams();
  }
  
  void _setupStreams() {
    // Watch pending orders
    widget.service.watchPendingOrders().listen((orders) {
      setState(() {
        _logs['pendingOrders'] = orders.length;
        _addMessage('📦 Pending orders updated: ${orders.length}');
      });
    });
    
    // Watch high-value orders
    widget.service.watchHighValueOrders(100.0, Duration(seconds: 1)).listen((orders) {
      setState(() {
        _logs['highValueOrders'] = orders.length;
        if (orders.isNotEmpty) {
          _addMessage('💰 High value order: \$${orders.first.total}');
        }
      });
    });
    
    // Watch low stock
    widget.service.watchLowStockThrottled(10, Duration(seconds: 2)).listen((products) {
      setState(() {
        _logs['lowStock'] = products.length;
        if (products.isNotEmpty) {
          _addMessage('⚠️ Low stock: ${products.first.name} (${products.first.stock})');
        }
      });
    });
  }
  
  void _addMessage(String message) {
    _messages.insert(0, '[${_formatTime(DateTime.now())}] $message');
    if (_messages.length > 50) {
      _messages.removeLast();
    }
    _scrollController.animateTo(
      0,
      duration: Duration(milliseconds: 300),
      curve: Curves.easeOut,
    );
  }
  
  String _formatTime(DateTime time) {
    return '${time.hour.toString().padLeft(2, '0')}:${time.minute.toString().padLeft(2, '0')}:${time.second.toString().padLeft(2, '0')}';
  }
  
  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Stream Updates Live Demo',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            SizedBox(height: 8),
            Wrap(
              spacing: 16,
              children: [
                _buildStat('Pending Orders', _logs['pendingOrders'] ?? 0),
                _buildStat('High Value Orders', _logs['highValueOrders'] ?? 0),
                _buildStat('Low Stock Items', _logs['lowStock'] ?? 0),
              ],
            ),
            SizedBox(height: 12),
            Container(
              height: 200,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey[300]!),
                borderRadius: BorderRadius.circular(8),
              ),
              child: ListView.builder(
                controller: _scrollController,
                reverse: true,
                itemCount: _messages.length,
                itemBuilder: (context, index) {
                  return Padding(
                    padding: EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                    child: Text(
                      _messages[index],
                      style: TextStyle(fontSize: 12),
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
  
  Widget _buildStat(String label, int value) {
    return Container(
      padding: EdgeInsets.all(8),
      decoration: BoxDecoration(
        color: Colors.grey[100],
        borderRadius: BorderRadius.circular(4),
      ),
      child: Row(
        children: [
          Text(
            '$label: ',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),
          Text('$value'),
        ],
      ),
    );
  }
}
```

---

# Best Practices

- **Use `.distinct()`** – Avoid duplicate emissions
- **Use `.debounce()`** – For search/filter
- **Use `.throttle()`** – For rapid updates
- **Use `.doOnData()`** – For logging/debugging
- **Use `.handleError()`** – For error handling
- **Cancel subscriptions** – Prevent memory leaks
- **Use `StreamBuilder`** – For Flutter widgets
- **Test stream updates** – Verify behavior

---

# Common Mistakes

## Mistake 1: Expecting immediate updates

Wrong:
```dart
// 🚫 Expects immediate emission
final stream = query.watch();
await insertData();
// Stream might not have emitted yet
```

Correct:
```dart
// ✅ Use listen or StreamBuilder
final stream = query.watch();
stream.listen((data) {
  // Handle update
});
await insertData();
// Stream will emit when data changes
```

## Mistake 2: Not handling stream lifecycle

Wrong:
```dart
// 🚫 Stream keeps running after widget dispose
@override
void initState() {
  super.initState();
  stream.listen((data) {});
}
```

Correct:
```dart
// ✅ Dispose subscription
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

## Mistake 3: Expensive operations in stream

Wrong:
```dart
// 🚫 Heavy computation on every emission
stream.watch().map((data) {
  return heavyComputation(data); // Runs on every update
});
```

Correct:
```dart
// ✅ Use distinct to prevent unnecessary work
stream
  .distinct()
  .map((data) => heavyComputation(data));
```

---

# Summary

| Trigger | Emits | Example |
|---------|-------|---------|
| **Insert** | If matches filter | Insert active user |
| **Update** | If filter changes | Age changes |
| **Delete** | If matches filter | Delete active user |
| **No Change** | Doesn't emit | Update name only |

---

# Next Steps

Now you understand stream updates, let's dive deeper:

- [Reactive Queries](link) – Advanced reactive patterns
- [Watching Multiple Tables](link) – Multi-table watches
- [Stream Performance](link) – Performance optimization

---

# Did You Know?

- **Stream updates are smart** – Only emit when relevant

- **Stream updates are automatic** – No manual refresh needed

- **Stream updates are efficient** – Track dependencies

- **Stream updates are atomic** – Emit after transaction

- **Stream updates are consistent** – Always full data

- **Stream updates are composable** – Combine with RxDart

- **Stream updates are production-ready** – Used in large apps

- **Stream updates are the key** – To Drift's reactivity

---

