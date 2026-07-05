## Mapping Joined Results

**Converting joined query results into meaningful data structures**

---

# What is it?

**Mapping Joined Results** is the process of transforming the raw tabular data returned from a join query into structured, nested Dart objects that reflect your domain model. When you join multiple tables, each row contains columns from all tables. Drift provides flexible ways to map these results into clean, type-safe objects.

> **Think of Mapping Joined Results like "assembling a puzzle"** – you have pieces from different boxes (tables), and you need to combine them into complete pictures (objects) that make sense in your app.

```dart
// 👇 Raw join results (flat structure)
final query = db.select(db.users).join([
  innerJoin(db.orders, db.orders.userId.equals(db.users.id)),
  innerJoin(db.products, db.products.id.equals(db.orderItems.productId)),
]);

final results = await query.get();
// Each row: [user columns, order columns, product columns]

// 👇 Mapped to nested object
final orders = results.map((row) {
  final user = row.readTable(db.users);
  final order = row.readTable(db.orders);
  final product = row.readTable(db.products);
  
  return OrderWithDetails(
    user: user,
    order: order,
    products: [product],
  );
}).toList();
```

 **What's happening here?**
- **Raw data** – Flat rows from join query
- **Read tables** – Extract data for each table
- **Map data** – Transform into nested objects
- **Group results** – Combine multiple rows per parent

---

# Why does it exist?

- **Domain Models** – Create clean object structures
- **API Responses** – Format data for API consumers
- **UI State** – Structure data for UI display
- **Business Logic** – Apply business rules during mapping
- **Data Transformation** – Convert database data to domain objects
- **Performance** – One query instead of many

---

# Basic Mapping

> **Simple result mapping**

## Join with Two Tables

```dart
// 👇 Map user and order join
final query = db.select(db.users).join([
  innerJoin(db.orders, db.orders.userId.equals(db.users.id)),
]);

query.where((u) => db.users.isActive.equals(true));
query.orderBy([(u) => OrderingTerm.desc(db.orders.orderDate)]);

final results = await query.get();

// 👇 Map to list of UserOrder objects
final userOrders = results.map((row) {
  final user = row.readTable(db.users);
  final order = row.readTable(db.orders);
  
  return UserOrder(
    user: user,
    order: order,
  );
}).toList();
```

## Join with Three Tables

```dart
// 👇 Map user, order, and product
final query = db.select(db.users).join([
  innerJoin(db.orders, db.orders.userId.equals(db.users.id)),
  innerJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
  innerJoin(db.products, db.products.id.equals(db.orderItems.productId)),
]);

final results = await query.get();

// 👇 Map to nested structure
final userOrders = results.map((row) {
  final user = row.readTable(db.users);
  final order = row.readTable(db.orders);
  final item = row.readTable(db.orderItems);
  final product = row.readTable(db.products);
  
  return UserOrderItem(
    user: user,
    order: order,
    orderItem: item,
    product: product,
  );
}).toList();
```

---

# Grouping Results

> **Handling one-to-many relationships in joined results**

## Group by Parent

```dart
// 👇 Group orders by user
Future<List<UserWithOrders>> getUserOrders() async {
  final query = db.select(db.users).join([
    innerJoin(db.orders, db.orders.userId.equals(db.users.id)),
  ]);
  
  query.orderBy([
    (u) => OrderingTerm.asc(db.users.name),
    (o) => OrderingTerm.desc(db.orders.orderDate),
  ]);
  
  final results = await query.get();
  
  // 👇 Group by user
  final userMap = <int, UserWithOrders>{};
  
  for (final row in results) {
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
}
```

## Group by Multiple Levels

```dart
// 👇 Group orders and items by user
Future<List<UserWithFullOrders>> getUserFullOrders() async {
  final query = db.select(db.users).join([
    innerJoin(db.orders, db.orders.userId.equals(db.users.id)),
    innerJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
    innerJoin(db.products, db.products.id.equals(db.orderItems.productId)),
  ]);
  
  query.orderBy([
    (u) => OrderingTerm.asc(db.users.name),
    (o) => OrderingTerm.desc(db.orders.orderDate),
  ]);
  
  final results = await query.get();
  
  // 👇 Nested grouping
  final userMap = <int, UserWithFullOrders>{};
  
  for (final row in results) {
    final user = row.readTable(db.users);
    final order = row.readTable(db.orders);
    final item = row.readTable(db.orderItems);
    final product = row.readTable(db.products);
    
    // Get or create user
    if (!userMap.containsKey(user.id)) {
      userMap[user.id] = UserWithFullOrders(
        user: user,
        orders: [],
      );
    }
    
    final userData = userMap[user.id]!;
    
    // Find or create order within user
    var orderData = userData.orders.firstWhere(
      (o) => o.order.id == order.id,
      orElse: () {
        final newOrder = UserOrderWithItems(
          order: order,
          items: [],
        );
        userData.orders.add(newOrder);
        return newOrder;
      },
    );
    
    // Add item to order
    orderData.items.add(
      OrderItemWithProduct(
        orderItem: item,
        product: product,
      )
    );
  }
  
  return userMap.values.toList();
}
```

---

# Real-World Example

> **Complete mapping system**

```dart
// lib/database/mapping_service.dart
import 'package:drift/drift.dart';

class MappingService {
  final AppDatabase db;
  
  MappingService(this.db);

  // ==================== ORDER MAPPING ====================
  
  // 👇 Map order with user and items
  Future<OrderFull> getOrderFull(int orderId) async {
    final query = db.select(db.orders).join([
      innerJoin(db.users, db.users.id.equals(db.orders.userId)),
      leftJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
      leftJoin(db.products, db.products.id.equals(db.orderItems.productId)),
    ]);
    
    query.where((o) => db.orders.id.equals(orderId));
    
    final results = await query.get();
    
    if (results.isEmpty) {
      throw Exception('Order not found');
    }
    
    // Extract order and user
    final firstRow = results.first;
    final order = firstRow.readTable(db.orders);
    final user = firstRow.readTable(db.users);
    
    // Extract items
    final items = <OrderItemFull>[];
    for (final row in results) {
      final item = row.readTableOrNull(db.orderItems);
      if (item != null) {
        final product = row.readTable(db.products);
        items.add(
          OrderItemFull(
            orderItem: item,
            product: product,
          )
        );
      }
    }
    
    return OrderFull(
      order: order,
      user: user,
      items: items,
    );
  }
  
  // 👇 Map all orders with users
  Future<List<OrderWithUser>> getAllOrdersWithUsers() async {
    final query = db.select(db.orders).join([
      innerJoin(db.users, db.users.id.equals(db.orders.userId)),
    ]);
    
    query.orderBy([(o) => OrderingTerm.desc(db.orders.orderDate)]);
    
    final results = await query.get();
    
    return results.map((row) {
      return OrderWithUser(
        order: row.readTable(db.orders),
        user: row.readTable(db.users),
      );
    }).toList();
  }

  // ==================== USER MAPPING ====================
  
  // 👇 Map user with complete history
  Future<UserFull> getUserFull(int userId) async {
    // Get user
    final user = await (db.select(db.users)
      ..where((u) => u.id.equals(userId)))
      .getSingle();
    
    // Get profile
    final profile = await (db.select(db.userProfiles)
      ..where((p) => p.userId.equals(userId)))
      .getSingleOrNull();
    
    // Get settings
    final settings = await (db.select(db.userSettings)
      ..where((s) => s.userId.equals(userId)))
      .getSingleOrNull();
    
    // Get orders
    final orderQuery = db.select(db.orders).join([
      leftJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id)),
      leftJoin(db.products, db.products.id.equals(db.orderItems.productId)),
    ]);
    
    orderQuery.where((o) => db.orders.userId.equals(userId));
    orderQuery.orderBy([(o) => OrderingTerm.desc(db.orders.orderDate)]);
    
    final orderResults = await orderQuery.get();
    
    // Group orders
    final orders = <UserOrderWithItems>[];
    final orderMap = <int, UserOrderWithItems>{};
    
    for (final row in orderResults) {
      final order = row.readTable(db.orders);
      
      if (!orderMap.containsKey(order.id)) {
        final newOrder = UserOrderWithItems(
          order: order,
          items: [],
        );
        orderMap[order.id] = newOrder;
        orders.add(newOrder);
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
    
    return UserFull(
      user: user,
      profile: profile,
      settings: settings,
      orders: orders,
    );
  }

  // ==================== PRODUCT MAPPING ====================
  
  // 👇 Map product with all details
  Future<ProductFull> getProductFull(int productId) async {
    // Get product
    final product = await (db.select(db.products)
      ..where((p) => p.id.equals(productId)))
      .getSingle();
    
    // Get categories
    final categoryQuery = db.select(db.categories).join([
      innerJoin(
        db.productCategories,
        db.productCategories.categoryId.equals(db.categories.id),
      )
    ]);
    
    categoryQuery.where((pc) => db.productCategories.productId.equals(productId));
    
    final categories = await categoryQuery.get()
      .then((results) => results.map((r) => r.readTable(db.categories)).toList());
    
    // Get tags
    final tagQuery = db.select(db.tags).join([
      innerJoin(
        db.productTags,
        db.productTags.tagId.equals(db.tags.id),
      )
    ]);
    
    tagQuery.where((pt) => db.productTags.productId.equals(productId));
    
    final tags = await tagQuery.get()
      .then((results) => results.map((r) => r.readTable(db.tags)).toList());
    
    // Get reviews
    final reviews = await (db.select(db.reviews)
      ..where((r) => r.productId.equals(productId))
      ..join([
        innerJoin(db.users, db.users.id.equals(db.reviews.userId)),
      ]))
      .get();
    
    final reviewList = reviews.map((row) {
      return ReviewWithUser(
        review: row.readTable(db.reviews),
        user: row.readTable(db.users),
      );
    }).toList();
    
    return ProductFull(
      product: product,
      categories: categories,
      tags: tags,
      reviews: reviewList,
    );
  }
  
  // 👇 Map all products with basic details
  Future<List<ProductBasic>> getAllProductsBasic() async {
    final query = db.select(db.products)
      .where((p) => p.isActive.equals(true))
      .join([
        leftJoin(
          db.productCategories,
          db.productCategories.productId.equals(db.products.id),
        ),
        leftJoin(
          db.categories,
          db.categories.id.equals(db.productCategories.categoryId),
        ),
      ]);
    
    query.orderBy([(p) => OrderingTerm.asc(db.products.name)]);
    
    final results = await query.get();
    
    // Group by product
    final productMap = <int, ProductBasic>{};
    
    for (final row in results) {
      final product = row.readTable(db.products);
      
      if (!productMap.containsKey(product.id)) {
        productMap[product.id] = ProductBasic(
          product: product,
          categories: [],
        );
      }
      
      final category = row.readTableOrNull(db.categories);
      if (category != null) {
        productMap[product.id]!.categories.add(category);
      }
    }
    
    return productMap.values.toList();
  }

  // ==================== DASHBOARD MAPPING ====================
  
  // 👇 Map complete dashboard data
  Future<DashboardData> getDashboardData() async {
    // Recent orders
    final recentOrdersQuery = db.select(db.orders).join([
      innerJoin(db.users, db.users.id.equals(db.orders.userId)),
    ]);
    
    recentOrdersQuery
      ..orderBy([(o) => OrderingTerm.desc(db.orders.orderDate)])
      ..limit(10);
    
    final recentOrders = await recentOrdersQuery.get()
      .then((results) => results.map((row) {
        return OrderWithUser(
          order: row.readTable(db.orders),
          user: row.readTable(db.users),
        );
      }).toList());
    
    // Top products
    final topProductsQuery = db.customSelect('''
      SELECT 
        p.id,
        p.name,
        p.sku,
        SUM(oi.quantity) as total_sold,
        SUM(oi.total) as total_revenue
      FROM products p
      INNER JOIN order_items oi ON p.id = oi.product_id
      WHERE p.is_active = 1
      GROUP BY p.id
      ORDER BY total_revenue DESC
      LIMIT 5
    ''');
    
    final topProducts = await topProductsQuery.get()
      .then((results) => results.map((row) {
        return TopProduct(
          id: row.data['id'] as int,
          name: row.data['name'] as String,
          sku: row.data['sku'] as String,
          totalSold: row.data['total_sold'] as int,
          totalRevenue: row.data['total_revenue'] as double,
        );
      }).toList());
    
    // Stats
    final stats = await db.customSelect('''
      SELECT 
        (SELECT COUNT(*) FROM orders WHERE status = 'completed') as completed_orders,
        (SELECT COUNT(*) FROM orders WHERE status = 'pending') as pending_orders,
        (SELECT SUM(total) FROM orders WHERE status = 'completed') as total_revenue,
        (SELECT COUNT(*) FROM users WHERE is_active = 1) as active_users,
        (SELECT COUNT(*) FROM products WHERE is_active = 1 AND stock > 0) as in_stock_products
    ''').get();
    
    final statsRow = stats.first;
    
    return DashboardData(
      recentOrders: recentOrders,
      topProducts: topProducts,
      completedOrders: statsRow.data['completed_orders'] as int,
      pendingOrders: statsRow.data['pending_orders'] as int,
      totalRevenue: statsRow.data['total_revenue'] as double,
      activeUsers: statsRow.data['active_users'] as int,
      inStockProducts: statsRow.data['in_stock_products'] as int,
    );
  }

  // ==================== REPORT MAPPING ====================
  
  // 👇 Map sales report
  Future<SalesReport> getSalesReport({
    required DateTime startDate,
    required DateTime endDate,
  }) async {
    final results = await db.customSelect('''
      SELECT 
        DATE(o.order_date) as date,
        COUNT(DISTINCT o.id) as order_count,
        COUNT(DISTINCT o.user_id) as unique_customers,
        SUM(o.total) as revenue,
        SUM(oi.quantity) as items_sold,
        AVG(o.total) as avg_order_value
      FROM orders o
      INNER JOIN order_items oi ON o.id = oi.order_id
      WHERE o.status = 'completed'
        AND o.order_date >= ?
        AND o.order_date <= ?
      GROUP BY DATE(o.order_date)
      ORDER BY date
    ''', variables: [
      Variable.withString(startDate.toIso8601String()),
      Variable.withString(endDate.toIso8601String()),
    ]).get();
    
    final dailyReports = results.map((row) {
      return DailySalesReport(
        date: DateTime.parse(row.data['date'] as String),
        orderCount: row.data['order_count'] as int,
        uniqueCustomers: row.data['unique_customers'] as int,
        revenue: row.data['revenue'] as double,
        itemsSold: row.data['items_sold'] as int,
        avgOrderValue: row.data['avg_order_value'] as double,
      );
    }).toList();
    
    // Calculate totals
    final totalOrders = dailyReports.fold(0, (sum, r) => sum + r.orderCount);
    final totalRevenue = dailyReports.fold(0.0, (sum, r) => sum + r.revenue);
    final totalItems = dailyReports.fold(0, (sum, r) => sum + r.itemsSold);
    final avgDailyRevenue = dailyReports.isNotEmpty 
        ? totalRevenue / dailyReports.length 
        : 0.0;
    
    return SalesReport(
      dailyReports: dailyReports,
      totalOrders: totalOrders,
      totalRevenue: totalRevenue,
      totalItemsSold: totalItems,
      avgDailyRevenue: avgDailyRevenue,
      days: dailyReports.length,
    );
  }
}

// ==================== DATA CLASSES ====================

class UserOrder {
  final User user;
  final Order order;
  
  UserOrder({
    required this.user,
    required this.order,
  });
}

class UserOrderItem {
  final User user;
  final Order order;
  final OrderItem orderItem;
  final Product product;
  
  UserOrderItem({
    required this.user,
    required this.order,
    required this.orderItem,
    required this.product,
  });
}

class UserWithOrders {
  final User user;
  final List<Order> orders;
  
  UserWithOrders({
    required this.user,
    required this.orders,
  });
}

class UserWithFullOrders {
  final User user;
  final List<UserOrderWithItems> orders;
  
  UserWithFullOrders({
    required this.user,
    required this.orders,
  });
}

class UserOrderWithItems {
  final Order order;
  final List<OrderItemWithProduct> items;
  
  UserOrderWithItems({
    required this.order,
    required this.items,
  });
}

class OrderItemWithProduct {
  final OrderItem orderItem;
  final Product product;
  
  OrderItemWithProduct({
    required this.orderItem,
    required this.product,
  });
}

class OrderFull {
  final Order order;
  final User user;
  final List<OrderItemFull> items;
  
  OrderFull({
    required this.order,
    required this.user,
    required this.items,
  });
}

class OrderItemFull {
  final OrderItem orderItem;
  final Product product;
  
  OrderItemFull({
    required this.orderItem,
    required this.product,
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

class UserFull {
  final User user;
  final UserProfile? profile;
  final UserSettings? settings;
  final List<UserOrderWithItems> orders;
  
  UserFull({
    required this.user,
    this.profile,
    this.settings,
    required this.orders,
  });
}

class ProductFull {
  final Product product;
  final List<Category> categories;
  final List<Tag> tags;
  final List<ReviewWithUser> reviews;
  
  ProductFull({
    required this.product,
    required this.categories,
    required this.tags,
    required this.reviews,
  });
}

class ProductBasic {
  final Product product;
  final List<Category> categories;
  
  ProductBasic({
    required this.product,
    required this.categories,
  });
}

class ReviewWithUser {
  final Review review;
  final User user;
  
  ReviewWithUser({
    required this.review,
    required this.user,
  });
}

class TopProduct {
  final int id;
  final String name;
  final String sku;
  final int totalSold;
  final double totalRevenue;
  
  TopProduct({
    required this.id,
    required this.name,
    required this.sku,
    required this.totalSold,
    required this.totalRevenue,
  });
}

class DashboardData {
  final List<OrderWithUser> recentOrders;
  final List<TopProduct> topProducts;
  final int completedOrders;
  final int pendingOrders;
  final double totalRevenue;
  final int activeUsers;
  final int inStockProducts;
  
  DashboardData({
    required this.recentOrders,
    required this.topProducts,
    required this.completedOrders,
    required this.pendingOrders,
    required this.totalRevenue,
    required this.activeUsers,
    required this.inStockProducts,
  });
}

class SalesReport {
  final List<DailySalesReport> dailyReports;
  final int totalOrders;
  final double totalRevenue;
  final int totalItemsSold;
  final double avgDailyRevenue;
  final int days;
  
  SalesReport({
    required this.dailyReports,
    required this.totalOrders,
    required this.totalRevenue,
    required this.totalItemsSold,
    required this.avgDailyRevenue,
    required this.days,
  });
}

class DailySalesReport {
  final DateTime date;
  final int orderCount;
  final int uniqueCustomers;
  final double revenue;
  final int itemsSold;
  final double avgOrderValue;
  
  DailySalesReport({
    required this.date,
    required this.orderCount,
    required this.uniqueCustomers,
    required this.revenue,
    required this.itemsSold,
    required this.avgOrderValue,
  });
}
```

---

# Best Practices

- **Use `readTableOrNull()`** – For optional data
- **Group results** – For one-to-many relationships
- **Use nested mapping** – For complex structures
- **Order results** – For consistent grouping
- **Use transactions** – For multi-table mapping
- **Handle nulls** – For optional relationships
- **Use custom classes** – For clean domain models
- **Test mapping logic** – Verify correctness

---

# Common Mistakes

## Mistake 1: Not grouping one-to-many results

Wrong:
```dart
// 🚫 Flat list of user-order pairs
final flatResults = results.map((row) {
  return UserWithOrders(
    user: row.readTable(db.users),
    orders: [row.readTable(db.orders)],
  );
}).toList();
// Each user appears multiple times!
```

Correct:
```dart
// ✅ Group by user
final groupedResults = groupByUser(results);
```

## Mistake 2: Not handling null values

Wrong:
```dart
// 🚫 Crashes if profile is null
final profile = row.readTable(db.profiles);
```

Correct:
```dart
// ✅ Handle optional data
final profile = row.readTableOrNull(db.profiles);
```

## Mistake 3: Multiple database queries

Wrong:
```dart
// 🚫 N+1 query problem
final users = await db.select(db.users).get();
for (final user in users) {
  final orders = await getOrders(user.id); // N+1 queries!
}
```

Correct:
```dart
// ✅ One query with join
final results = await db.select(db.users).join([...]).get();
```

---

# Summary

| Technique | Purpose | Example |
|-----------|---------|---------|
| `readTable()` | Required data | Required relationship |
| `readTableOrNull()` | Optional data | Optional relationship |
| Grouping | One-to-many | Group by parent |
| Nesting | Complex structures | Multiple levels |
| Custom classes | Domain models | Clean objects |

---

# Next Steps

Now you understand mapping joined results, let's dive deeper:

- [Streams](link) – Reactive queries
- [Watching Queries](link) – Real-time updates
- [Stream Updates](link) – Stream management

---

# Did You Know?

- **Mapping can be done in one query** – Instead of many

- **Nested mapping improves performance** – Fewer round trips

- **Grouping is essential** – For one-to-many relationships

- **Null handling is important** – For optional relationships

- **Mapping can include business logic** – Apply rules during mapping

- **Mapping can be complex** – For deeply nested structures

- **Mapping is critical for APIs** – Clean JSON responses

- **Mapping can be optimized** – With proper queries

---

