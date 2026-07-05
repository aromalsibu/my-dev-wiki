## One-to-Many

**Mastering one-to-many relationships in Drift**

---

# What is it?

**One-to-Many** is the most common type of database relationship where a single record in one table relates to multiple records in another table. In Drift, this is implemented using a foreign key in the "many" table that references the primary key of the "one" table. This relationship is fundamental to most database designs.

> **Think of One-to-Many like "a parent with multiple children"** – one parent can have many children, but each child has only one parent. The parent is the "one" side, and the children are the "many" side.

```dart
// 👇 One-to-Many: A user has many posts
// Users (one side)
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

// Posts (many side)
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  
  // 👇 Foreign key to Users
  IntColumn get userId => integer()
    .references(Users, #id)
    .named('user_id')();
}

// Query: Get all posts for a user
final userId = 1;
final userPosts = await (db.select(db.posts)
  ..where((p) => p.userId.equals(userId)))
  .get();
```

> **What's happening here?**
> - **One side** – Users (each user has many posts)
> - **Many side** – Posts (each post belongs to one user)
> - **Foreign key** – `userId` in Posts references Users
> - **Query** – Filter posts by userId

---

# Why does it exist?

- **Real-world relationships** – User-posts, category-products
- **Data Organization** – Group related data
- **Data Integrity** – Maintain valid relationships
- **Query Efficiency** – Efficient data retrieval
- **Business Logic** – Model business rules
- **Reporting** – Generate hierarchical reports

---

# Defining One-to-Many

> **Creating the relationship in tables**

## Basic One-to-Many

```dart
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/tables/posts.dart
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  
  // 👇 Foreign key to Users
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)
    .named('user_id')();
  
  BoolColumn get isPublished => boolean()
    .withDefault(const Constant(false))
    .named('is_published')();
  
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
  
  DateTimeColumn get publishedAt => dateTime()
    .nullable()
    .named('published_at')();
  
  // 👇 Index for performance
  @override
  List<Index> get indexes => [
    Index('idx_posts_user', 'user_id'),
    Index('idx_posts_published', 'is_published'),
    Index('idx_posts_created', 'created_at'),
  ];
}
```

---

# Querying One-to-Many

> **Retrieving related data**

## Get Many Side Records

```dart
// 👇 Get all posts for a user
Future<List<Post>> getUserPosts(int userId) async {
  return await (db.select(db.posts)
    ..where((p) => p.userId.equals(userId))
    ..where((p) => p.isPublished.equals(true))
    ..orderBy([(p) => OrderingTerm.desc(p.createdAt)]))
    .get();
}

// 👇 Get published posts for a user
Future<List<Post>> getUserPublishedPosts(int userId) async {
  return await (db.select(db.posts)
    ..where((p) => p.userId.equals(userId))
    ..where((p) => p.isPublished.equals(true))
    ..orderBy([(p) => OrderingTerm.desc(p.publishedAt)]))
    .get();
}

// 👇 Get recent posts for a user
Future<List<Post>> getUserRecentPosts(int userId, int limit) async {
  return await (db.select(db.posts)
    ..where((p) => p.userId.equals(userId))
    ..where((p) => p.isPublished.equals(true))
    ..orderBy([(p) => OrderingTerm.desc(p.createdAt)])
    ..limit(limit))
    .get();
}
```

## Get One Side Record with Count

```dart
// 👇 Get users with post counts
Future<List<UserWithPostCount>> getUsersWithPostCounts() async {
  final results = await db.customSelect('''
    SELECT 
      u.id as user_id,
      u.name as user_name,
      u.email as user_email,
      COUNT(p.id) as post_count,
      COUNT(CASE WHEN p.is_published = 1 THEN 1 END) as published_count,
      MAX(p.created_at) as last_post_date
    FROM users u
    LEFT JOIN posts p ON u.id = p.user_id
    GROUP BY u.id, u.name, u.email
    ORDER BY post_count DESC
  ''').get();
  
  return results.map((row) {
    return UserWithPostCount(
      userId: row.data['user_id'] as int,
      userName: row.data['user_name'] as String,
      userEmail: row.data['user_email'] as String,
      postCount: row.data['post_count'] as int,
      publishedCount: row.data['published_count'] as int,
      lastPostDate: row.data['last_post_date'] != null
          ? DateTime.parse(row.data['last_post_date'] as String)
          : null,
    );
  }).toList();
}
```

## Get One Side with Related Data

```dart
// 👇 Get user with all posts (using join)
Future<UserWithPosts> getUserWithPosts(int userId) async {
  // Get user
  final user = await (db.select(db.users)
    ..where((u) => u.id.equals(userId)))
    .getSingle();
  
  // Get posts
  final posts = await (db.select(db.posts)
    ..where((p) => p.userId.equals(userId))
    ..where((p) => p.isPublished.equals(true))
    ..orderBy([(p) => OrderingTerm.desc(p.createdAt)]))
    .get();
  
  return UserWithPosts(
    user: user,
    posts: posts,
  );
}
```

---

# Managing One-to-Many

> **Creating, updating, and deleting related data**

## Create Related Records

```dart
// 👇 Create post for user
Future<int> createPost({
  required int userId,
  required String title,
  required String content,
}) async {
  return await db.into(db.posts).insert(
    PostsCompanion.insert(
      userId: userId,
      title: title,
      content: content,
    ),
  );
}

// 👇 Create post with immediate publish
Future<int> createPublishedPost({
  required int userId,
  required String title,
  required String content,
}) async {
  return await db.into(db.posts).insert(
    PostsCompanion.insert(
      userId: userId,
      title: title,
      content: content,
      isPublished: const Value(true),
      publishedAt: Value(DateTime.now()),
    ),
  );
}

// 👇 Create multiple posts
Future<List<int>> createPosts({
  required int userId,
  required List<PostInput> posts,
}) async {
  final ids = <int>[];
  
  await db.transaction(() async {
    for (final post in posts) {
      final id = await db.into(db.posts).insert(
        PostsCompanion.insert(
          userId: userId,
          title: post.title,
          content: post.content,
          isPublished: Value(post.isPublished),
          publishedAt: post.isPublished 
              ? Value(DateTime.now()) 
              : const Value.absent(),
        ),
      );
      ids.add(id);
    }
  });
  
  return ids;
}
```

## Update Related Records

```dart
// 👇 Update post
Future<void> updatePost({
  required int postId,
  String? title,
  String? content,
  bool? isPublished,
}) async {
  await (db.update(db.posts)..where((p) => p.id.equals(postId)))
    .write(PostsCompanion(
      title: title != null ? Value(title) : const Value.absent(),
      content: content != null ? Value(content) : const Value.absent(),
      isPublished: isPublished != null 
          ? Value(isPublished) 
          : const Value.absent(),
      publishedAt: isPublished == true 
          ? Value(DateTime.now()) 
          : const Value.absent(),
    ));
}

// 👇 Update all posts for a user
Future<void> updateAllUserPosts(int userId, bool isPublished) async {
  await (db.update(db.posts)..where((p) => p.userId.equals(userId)))
    .write(PostsCompanion(
      isPublished: Value(isPublished),
      publishedAt: isPublished 
          ? Value(DateTime.now()) 
          : const Value.absent(),
    ));
}
```

## Delete Related Records

```dart
// 👇 Delete single post
Future<void> deletePost(int postId) async {
  await (db.delete(db.posts)..where((p) => p.id.equals(postId))).go();
}

// 👇 Delete all posts for a user
Future<void> deleteAllUserPosts(int userId) async {
  await (db.delete(db.posts)..where((p) => p.userId.equals(userId))).go();
}

// 👇 Delete unpublished posts
Future<int> deleteUnpublishedPosts() async {
  return await (db.delete(db.posts)
    ..where((p) => p.isPublished.equals(false))
    ..where((p) => p.createdAt < const Variable(
      DateTime.now().subtract(Duration(days: 30))
    )))
    .go();
}
```

---

# Real-World Example

> **Complete e-commerce one-to-many system**

```dart
// lib/database/tables/orders.dart
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().unique()();
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)
    .named('user_id')();
  
  RealColumn get total => real().customConstraint('CHECK (total >= 0)')();
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .customConstraint("CHECK (status IN ('pending', 'processing', 'paid', 'shipped', 'delivered', 'cancelled'))")();
  
  TextColumn get shippingAddress => text().named('shipping_address')();
  TextColumn get billingAddress => text().named('billing_address')();
  TextColumn get paymentMethod => text().nullable().named('payment_method')();
  TextColumn get paymentStatus => text()
    .withDefault(const Constant('unpaid'))
    .named('payment_status')();
  
  BoolColumn get isPaid => boolean()
    .withDefault(const Constant(false))
    .named('is_paid')();
  
  DateTimeColumn get orderDate => dateTime()
    .withDefault(currentDateAndTime)
    .named('order_date')();
  
  DateTimeColumn get shippedDate => dateTime()
    .nullable()
    .named('shipped_date')();
  
  @override
  List<Index> get indexes => [
    Index('idx_orders_user', 'user_id'),
    Index('idx_orders_status', 'status'),
    Index('idx_orders_order_date', 'order_date'),
  ];
}

// lib/database/tables/order_items.dart (Many side)
class OrderItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get orderId => integer()
    .references(Orders, #id, onDelete: KeyAction.cascade)
    .named('order_id')();
  
  IntColumn get productId => integer()
    .references(Products, #id, onDelete: KeyAction.restrict)
    .named('product_id')();
  
  IntColumn get quantity => integer()
    .customConstraint('CHECK (quantity > 0)')();
  
  RealColumn get unitPrice => real()
    .customConstraint('CHECK (unit_price >= 0)')
    .named('unit_price')();
  
  RealColumn get subtotal => real()
    .generatedAs(
      const Constant("quantity * unit_price"),
      stored: true,
    )();
  
  RealColumn get total => real()
    .generatedAs(
      const Constant("quantity * unit_price"),
      stored: true,
    )();
  
  @override
  List<Index> get indexes => [
    Index('idx_order_items_order', 'order_id'),
    Index('idx_order_items_product', 'product_id'),
  ];
}

// lib/database/one_to_many_service.dart
import 'package:drift/drift.dart';

class OneToManyService {
  final AppDatabase db;
  
  OneToManyService(this.db);

  // ==================== ORDER MANAGEMENT ====================
  
  // 👇 Create order with items
  Future<Order> createOrder({
    required int userId,
    required String shippingAddress,
    required String billingAddress,
    required List<CartItem> items,
  }) async {
    return await db.transaction(() async {
      // Calculate total
      double total = 0;
      for (final item in items) {
        final product = await db.getProduct(item.productId);
        total += product.price * item.quantity;
      }
      
      // Create order
      final orderId = await db.into(db.orders).insert(
        OrdersCompanion.insert(
          userId: userId,
          orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
          total: total,
          shippingAddress: shippingAddress,
          billingAddress: billingAddress,
          paymentMethod: 'pending',
          paymentStatus: 'unpaid',
          isPaid: false,
        ),
      );
      
      // Create order items
      for (final item in items) {
        final product = await db.getProduct(item.productId);
        await db.into(db.orderItems).insert(
          OrderItemsCompanion.insert(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: product.price,
          ),
        );
        
        // Update stock
        await db.updateProductStock(item.productId, product.stock - item.quantity);
      }
      
      return await db.getOrder(orderId);
    });
  }
  
  // 👇 Get user orders
  Future<List<Order>> getUserOrders(int userId) async {
    return await (db.select(db.orders)
      ..where((o) => o.userId.equals(userId))
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
      .get();
  }
  
  // 👇 Get order with items
  Future<OrderWithItems> getOrderWithItems(int orderId) async {
    final order = await db.getOrder(orderId);
    final items = await (db.select(db.orderItems)
      ..where((i) => i.orderId.equals(orderId)))
      .get();
    
    // Get product details for each item
    final itemsWithProducts = <OrderItemWithProduct>[];
    for (final item in items) {
      final product = await db.getProduct(item.productId);
      itemsWithProducts.add(
        OrderItemWithProduct(
          orderItem: item,
          product: product,
        )
      );
    }
    
    return OrderWithItems(
      order: order,
      items: itemsWithProducts,
    );
  }
  
  // 👇 Get user with order summary
  Future<UserOrderSummary> getUserOrderSummary(int userId) async {
    final user = await db.getUser(userId);
    final orders = await getUserOrders(userId);
    
    final totalOrders = orders.length;
    final totalSpent = orders.fold(0.0, (sum, order) => sum + order.total);
    final avgOrderValue = totalOrders > 0 ? totalSpent / totalOrders : 0.0;
    final completedOrders = orders.where((o) => o.status == 'delivered').length;
    
    return UserOrderSummary(
      user: user,
      orders: orders,
      totalOrders: totalOrders,
      totalSpent: totalSpent,
      avgOrderValue: avgOrderValue,
      completedOrders: completedOrders,
    );
  }
  
  // 👇 Get order statistics for user
  Future<Map<String, dynamic>> getUserOrderStats(int userId) async {
    final results = await db.customSelect('''
      SELECT 
        COUNT(*) as total_orders,
        SUM(total) as total_spent,
        AVG(total) as avg_order_value,
        MAX(total) as max_order_value,
        MIN(total) as min_order_value,
        SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END) as completed_orders,
        SUM(CASE WHEN status = 'cancelled' THEN 1 ELSE 0 END) as cancelled_orders
      FROM orders
      WHERE user_id = ?
    ''', variables: [
      Variable.withInt(userId),
    ]).get();
    
    final row = results.first;
    return {
      'totalOrders': row.data['total_orders'] as int,
      'totalSpent': row.data['total_spent'] as double,
      'avgOrderValue': row.data['avg_order_value'] as double,
      'maxOrderValue': row.data['max_order_value'] as double,
      'minOrderValue': row.data['min_order_value'] as double,
      'completedOrders': row.data['completed_orders'] as int,
      'cancelledOrders': row.data['cancelled_orders'] as int,
    };
  }
  
  // ==================== ORDER ITEM MANAGEMENT ====================
  
  // 👇 Update order item quantity
  Future<void> updateOrderItemQuantity({
    required int orderId,
    required int productId,
    required int newQuantity,
  }) async {
    if (newQuantity <= 0) {
      throw Exception('Quantity must be positive');
    }
    
    await db.transaction(() async {
      final item = await (db.select(db.orderItems)
        ..where((i) => i.orderId.equals(orderId) & i.productId.equals(productId)))
        .getSingle();
      
      final product = await db.getProduct(productId);
      final stockDiff = item.quantity - newQuantity;
      
      // Update stock
      await db.updateProductStock(productId, product.stock + stockDiff);
      
      // Update item
      await (db.update(db.orderItems)
        ..where((i) => i.orderId.equals(orderId) & i.productId.equals(productId)))
        .write(OrderItemsCompanion(
          quantity: Value(newQuantity),
        ));
      
      // Recalculate order total
      await _recalculateOrderTotal(orderId);
    });
  }
  
  // 👇 Remove item from order
  Future<void> removeOrderItem({
    required int orderId,
    required int productId,
  }) async {
    await db.transaction(() async {
      final item = await (db.select(db.orderItems)
        ..where((i) => i.orderId.equals(orderId) & i.productId.equals(productId)))
        .getSingle();
      
      final product = await db.getProduct(productId);
      
      // Restore stock
      await db.updateProductStock(productId, product.stock + item.quantity);
      
      // Delete item
      await (db.delete(db.orderItems)
        ..where((i) => i.orderId.equals(orderId) & i.productId.equals(productId)))
        .go();
      
      // Recalculate order total
      await _recalculateOrderTotal(orderId);
    });
  }
  
  Future<void> _recalculateOrderTotal(int orderId) async {
    final items = await (db.select(db.orderItems)
      ..where((i) => i.orderId.equals(orderId)))
      .get();
    
    final total = items.fold(0.0, (sum, item) => sum + item.total);
    
    await (db.update(db.orders)..where((o) => o.id.equals(orderId)))
      .write(OrdersCompanion(
        total: Value(total),
      ));
  }

  // ==================== CATEGORY-PRODUCT (One-to-Many) ====================
  
  // 👇 Get products by category
  Future<List<Product>> getCategoryProducts(int categoryId) async {
    return await (db.select(db.products)
      ..where((p) => p.categoryId.equals(categoryId))
      ..where((p) => p.isActive.equals(true))
      ..orderBy([(p) => OrderingTerm.asc(p.name)]))
      .get();
  }
  
  // 👇 Get category with products
  Future<CategoryWithProducts> getCategoryWithProducts(int categoryId) async {
    final category = await db.getCategory(categoryId);
    final products = await getCategoryProducts(categoryId);
    
    return CategoryWithProducts(
      category: category,
      products: products,
    );
  }
  
  // 👇 Get category stats
  Future<CategoryStats> getCategoryStats(int categoryId) async {
    final results = await db.customSelect('''
      SELECT 
        COUNT(*) as product_count,
        AVG(price) as avg_price,
        MAX(price) as max_price,
        MIN(price) as min_price,
        SUM(stock) as total_stock,
        SUM(CASE WHEN is_active = 1 THEN 1 ELSE 0 END) as active_count
      FROM products
      WHERE category_id = ?
    ''', variables: [
      Variable.withInt(categoryId),
    ]).get();
    
    final row = results.first;
    return CategoryStats(
      productCount: row.data['product_count'] as int,
      avgPrice: row.data['avg_price'] as double,
      maxPrice: row.data['max_price'] as double,
      minPrice: row.data['min_price'] as double,
      totalStock: row.data['total_stock'] as int,
      activeCount: row.data['active_count'] as int,
    );
  }
  
  // ====================== BULK ONE-TO-MANY ======================
  
  // 👇 Bulk update order status
  Future<void> bulkUpdateOrderStatus({
    required int userId,
    required String newStatus,
  }) async {
    await (db.update(db.orders)
      ..where((o) => o.userId.equals(userId))
      ..where((o) => o.status.isNotValue('delivered')))
      .write(OrdersCompanion(
        status: Value(newStatus),
        shippedDate: newStatus == 'shipped' 
            ? Value(DateTime.now()) 
            : const Value.absent(),
        isPaid: newStatus == 'paid' || newStatus == 'shipped' 
            ? const Value(true) 
            : const Value.absent(),
      ));
  }
  
  // 👇 Delete all user orders
  Future<int> deleteUserOrders(int userId) async {
    return await (db.delete(db.orders)
      ..where((o) => o.userId.equals(userId))
      ..where((o) => o.status.isIn(['cancelled', 'delivered'])))
      .go();
  }
}

// ==================== DATA CLASSES ====================

class UserWithPosts {
  final User user;
  final List<Post> posts;
  
  UserWithPosts({
    required this.user,
    required this.posts,
  });
}

class UserWithPostCount {
  final int userId;
  final String userName;
  final String userEmail;
  final int postCount;
  final int publishedCount;
  final DateTime? lastPostDate;
  
  UserWithPostCount({
    required this.userId,
    required this.userName,
    required this.userEmail,
    required this.postCount,
    required this.publishedCount,
    this.lastPostDate,
  });
}

class OrderWithItems {
  final Order order;
  final List<OrderItemWithProduct> items;
  
  OrderWithItems({
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

class UserOrderSummary {
  final User user;
  final List<Order> orders;
  final int totalOrders;
  final double totalSpent;
  final double avgOrderValue;
  final int completedOrders;
  
  UserOrderSummary({
    required this.user,
    required this.orders,
    required this.totalOrders,
    required this.totalSpent,
    required this.avgOrderValue,
    required this.completedOrders,
  });
}

class CategoryWithProducts {
  final Category category;
  final List<Product> products;
  
  CategoryWithProducts({
    required this.category,
    required this.products,
  });
}

class CategoryStats {
  final int productCount;
  final double avgPrice;
  final double maxPrice;
  final double minPrice;
  final int totalStock;
  final int activeCount;
  
  CategoryStats({
    required this.productCount,
    required this.avgPrice,
    required this.maxPrice,
    required this.minPrice,
    required this.totalStock,
    required this.activeCount,
  });
}
```

---

# Best Practices

- **Use foreign keys** – Maintain referential integrity
- **Add indexes** – On foreign key columns
- **Use cascade delete** – Clean up related data
- **Use transactions** – For multiple operations
- **Preload related data** – Avoid N+1 queries
- **Use joins** – For efficient data retrieval
- **Add constraints** – Validate data at database level
- **Consider performance** – Index and query optimization

---

# Common Mistakes

## Mistake 1: Missing foreign key constraints

Wrong:
```dart
// 🚫 No foreign key constraint
IntColumn get userId => integer()();
```

Correct:
```dart
// ✅ Add foreign key constraint
IntColumn get userId => integer()
  .references(Users, #id, onDelete: KeyAction.cascade)();
```

## Mistake 2: Not handling N+1 queries

Wrong:
```dart
// 🚫 N+1 query problem
final users = await db.select(db.users).get();
for (final user in users) {
  final posts = await getPosts(user.id); // N+1 queries!
}
```

Correct:
```dart
// ✅ Use join or batch loading
final query = db.select(db.users).join([
  innerJoin(db.posts, db.posts.userId.equals(db.users.id)),
]);
final results = await query.get();
```

## Mistake 3: Not enabling foreign keys

Wrong:
```dart
// 🚫 Foreign keys disabled
@override
Future<void> beforeOpen() async {
  // Missing PRAGMA foreign_keys = ON
}
```

Correct:
```dart
// ✅ Enable foreign keys
@override
Future<void> beforeOpen() async {
  await customSelect('PRAGMA foreign_keys = ON').get();
}
```

---

# Summary

| Concept | One Side | Many Side |
|---------|----------|-----------|
| **Relationship** | Has many | Belongs to |
| **Foreign Key** | Primary key | References one |
| **Cascade** | ON DELETE CASCADE | Delete related |
| **Query** | Get related | Filter by parent |

---

# Next Steps

Now you understand one-to-many, let's dive deeper:

- [One-to-One](link) – One-to-one relationships
- [Mapping Joined Results](link) – Result mapping
- [Streams](link) – Reactive queries

---

# Did You Know?

- **One-to-many is the most common relationship** – In databases

- **Foreign keys maintain data integrity** – Prevent orphaned records

- **Indexes speed up one-to-many queries** – On foreign key columns

- **Cascade deletes automatically clean up** – Related records

- **N+1 queries are a common problem** – Use joins to avoid

- **One-to-many relationships are directional** – Parent to child

- **Aggregations are common** – Count, sum, average of related data

- **One-to-many can be nested** – Multiple levels of relationships

