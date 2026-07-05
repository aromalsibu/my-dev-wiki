## Relationships

**Connecting tables with joins in Drift**

---

# What is it?

**Relationships** are the connections between tables in your database. Drift provides powerful join capabilities to combine data from multiple tables, allowing you to fetch related data in a single query. This is essential for relational data models like users with orders, products with categories, and many-to-many relationships.

> **Think of Relationships like "family connections"** – just as family members are connected by relationships (parent-child, siblings), database tables are connected by foreign keys and joins to create meaningful connections between data.

```dart
// 👇 One-to-Many: User and their orders
final query = db.select(db.users).join([
  innerJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

// 👇 Results combine user and order data
final results = await query.get();
for (final row in results) {
  final user = row.readTable(db.users);
  final order = row.readTable(db.orders);
  print('${user.name} ordered #${order.orderNumber}');
}
```

> **What's happening here?**
> - **`join()`** – Combines multiple tables
> - **`innerJoin()`** – Only matching rows
> - **`leftJoin()`** – All rows from left table
> - **`crossJoin()`** – Cartesian product
> - **`readTable()`** – Extract data from joined tables

---

# Why does it exist?

- **Data Retrieval** – Get related data in one query
- **Performance** – Reduce database round-trips
- **Data Integrity** – Maintain relationships
- **Business Logic** – Model real-world relationships
- **Reporting** – Generate complex reports
- **Data Analysis** – Analyze related data

---

# Inner Join

> **Getting only matching records from both tables**

## Basic Inner Join

```dart
// 👇 Inner join users with orders
final query = db.select(db.users).join([
  innerJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

// 👇 Get all users who have orders
final results = await query.get();
for (final row in results) {
  final user = row.readTable(db.users);
  final order = row.readTable(db.orders);
  print('${user.name} -> ${order.orderNumber}');
}

// Generated SQL:
// SELECT * FROM users 
// INNER JOIN orders ON orders.user_id = users.id
```

## Inner Join with Conditions

```dart
// 👇 Inner join with additional conditions
final query = db.select(db.users).join([
  innerJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

query.where((u) => u.isActive.equals(true));
query.where((o) => o.status.equals('completed'));

final results = await query.get();
```

---

# Left Join

> **All rows from left table, matching rows from right**

## Basic Left Join

```dart
// 👇 Left join users with orders (all users, even without orders)
final query = db.select(db.users).join([
  leftJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

final results = await query.get();
for (final row in results) {
  final user = row.readTable(db.users);
  final order = row.readTableOrNull(db.orders); // 👈 May be null
  if (order != null) {
    print('${user.name} -> ${order.orderNumber}');
  } else {
    print('${user.name} -> No orders');
  }
}

// Generated SQL:
// SELECT * FROM users 
// LEFT JOIN orders ON orders.user_id = users.id
```

## Left Join with Multiple Tables

```dart
// 👇 Left join users, orders, and order items
final query = db.select(db.users).join([
  leftJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  ),
  leftJoin(
    db.orderItems,
    db.orderItems.orderId.equals(db.orders.id),
  ),
  leftJoin(
    db.products,
    db.products.id.equals(db.orderItems.productId),
  ),
]);

final results = await query.get();
for (final row in results) {
  final user = row.readTable(db.users);
  final order = row.readTableOrNull(db.orders);
  final product = row.readTableOrNull(db.products);
  
  if (order != null && product != null) {
    print('${user.name} ordered ${product.name}');
  }
}
```

---

# Multiple Joins

> **Combining many tables**

## Joining Multiple Tables

```dart
// 👇 User -> Orders -> Items -> Products
final query = db.select(db.users).join([
  innerJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  ),
  innerJoin(
    db.orderItems,
    db.orderItems.orderId.equals(db.orders.id),
  ),
  innerJoin(
    db.products,
    db.products.id.equals(db.orderItems.productId),
  ),
]);

query.where((o) => db.orders.status.equals('completed'));

final results = await query.get();

// 👇 Map results to DTO
final orderDetails = results.map((row) {
  final user = row.readTable(db.users);
  final order = row.readTable(db.orders);
  final item = row.readTable(db.orderItems);
  final product = row.readTable(db.products);
  
  return OrderDetailDto(
    userName: user.name,
    orderNumber: order.orderNumber,
    productName: product.name,
    quantity: item.quantity,
    total: item.total,
  );
}).toList();
```

---

# Real-World Example

> **Complete e-commerce relationship system**

```dart
// lib/database/relationship_service.dart
import 'package:drift/drift.dart';

class RelationshipService {
  final AppDatabase db;
  
  RelationshipService(this.db);

  // ==================== ONE-TO-MANY ====================
  
  // 👇 Get users with their orders
  Future<List<UserWithOrders>> getUsersWithOrders() async {
    final query = db.select(db.users).join([
      leftJoin(
        db.orders,
        db.orders.userId.equals(db.users.id),
      )
    ]);
    
    query.orderBy([(u) => OrderingTerm.asc(db.users.name)]);
    
    final results = await query.get();
    
    // 👇 Group results by user
    final userMap = <int, UserWithOrders>{};
    
    for (final row in results) {
      final user = row.readTable(db.users);
      
      if (!userMap.containsKey(user.id)) {
        userMap[user.id] = UserWithOrders(
          user: user,
          orders: [],
        );
      }
      
      final order = row.readTableOrNull(db.orders);
      if (order != null) {
        userMap[user.id]!.orders.add(order);
      }
    }
    
    return userMap.values.toList();
  }
  
  // 👇 Get user with orders and items
  Future<UserWithFullOrders> getUserWithFullOrders(int userId) async {
    final query = db.select(db.users).join([
      innerJoin(
        db.orders,
        db.orders.userId.equals(db.users.id),
      ),
      innerJoin(
        db.orderItems,
        db.orderItems.orderId.equals(db.orders.id),
      ),
      innerJoin(
        db.products,
        db.products.id.equals(db.orderItems.productId),
      ),
    ]);
    
    query.where((u) => db.users.id.equals(userId));
    
    final results = await query.get();
    
    if (results.isEmpty) {
      throw Exception('User not found');
    }
    
    final user = results.first.readTable(db.users);
    final ordersMap = <int, OrderWithItems>{};
    
    for (final row in results) {
      final order = row.readTable(db.orders);
      final item = row.readTable(db.orderItems);
      final product = row.readTable(db.products);
      
      if (!ordersMap.containsKey(order.id)) {
        ordersMap[order.id] = OrderWithItems(
          order: order,
          items: [],
        );
      }
      
      ordersMap[order.id]!.items.add(
        OrderItemWithProduct(
          orderItem: item,
          product: product,
        ),
      );
    }
    
    return UserWithFullOrders(
      user: user,
      orders: ordersMap.values.toList(),
    );
  }

  // ==================== MANY-TO-ONE ====================
  
  // 👇 Get products with their categories
  Future<List<ProductWithCategory>> getProductsWithCategories() async {
    final query = db.select(db.products).join([
      innerJoin(
        db.categories,
        db.categories.id.equals(db.products.categoryId),
      )
    ]);
    
    query.where((p) => db.products.isActive.equals(true));
    query.orderBy([(p) => OrderingTerm.asc(db.products.name)]);
    
    final results = await query.get();
    
    return results.map((row) {
      final product = row.readTable(db.products);
      final category = row.readTable(db.categories);
      
      return ProductWithCategory(
        product: product,
        category: category,
      );
    }).toList();
  }
  
  // 👇 Get product with all details
  Future<ProductWithDetails> getProductDetails(int productId) async {
    final query = db.select(db.products).join([
      innerJoin(
        db.categories,
        db.categories.id.equals(db.products.categoryId),
      ),
      leftJoin(
        db.reviews,
        db.reviews.productId.equals(db.products.id),
      ),
      leftJoin(
        db.users,
        db.users.id.equals(db.reviews.userId),
      ),
    ]);
    
    query.where((p) => db.products.id.equals(productId));
    
    final results = await query.get();
    
    if (results.isEmpty) {
      throw Exception('Product not found');
    }
    
    final product = results.first.readTable(db.products);
    final category = results.first.readTable(db.categories);
    final reviews = <ReviewWithUser>[];
    
    for (final row in results) {
      final review = row.readTableOrNull(db.reviews);
      if (review != null) {
        final user = row.readTable(db.users);
        reviews.add(ReviewWithUser(
          review: review,
          user: user,
        ));
      }
    }
    
    return ProductWithDetails(
      product: product,
      category: category,
      reviews: reviews,
    );
  }

  // ==================== MANY-TO-MANY ====================
  
  // 👇 Get products by tag (many-to-many)
  Future<List<Product>> getProductsByTag(String tagName) async {
    final query = db.select(db.products).join([
      innerJoin(
        db.productTags,
        db.productTags.productId.equals(db.products.id),
      ),
      innerJoin(
        db.tags,
        db.tags.id.equals(db.productTags.tagId),
      ),
    ]);
    
    query.where((t) => db.tags.name.equals(tagName));
    query.where((p) => db.products.isActive.equals(true));
    
    final results = await query.get();
    
    return results.map((row) => row.readTable(db.products)).toList();
  }
  
  // 👇 Get tags by product (many-to-many)
  Future<List<Tag>> getTagsByProduct(int productId) async {
    final query = db.select(db.tags).join([
      innerJoin(
        db.productTags,
        db.productTags.tagId.equals(db.tags.id),
      ),
      innerJoin(
        db.products,
        db.products.id.equals(db.productTags.productId),
      ),
    ]);
    
    query.where((p) => db.products.id.equals(productId));
    
    final results = await query.get();
    
    return results.map((row) => row.readTable(db.tags)).toList();
  }
  
  // 👇 Get products with their tags
  Future<List<ProductWithTags>> getProductsWithTags() async {
    final query = db.select(db.products).join([
      leftJoin(
        db.productTags,
        db.productTags.productId.equals(db.products.id),
      ),
      leftJoin(
        db.tags,
        db.tags.id.equals(db.productTags.tagId),
      ),
    ]);
    
    query.where((p) => db.products.isActive.equals(true));
    query.orderBy([(p) => OrderingTerm.asc(db.products.name)]);
    
    final results = await query.get();
    
    // 👇 Group by product
    final productMap = <int, ProductWithTags>{};
    
    for (final row in results) {
      final product = row.readTable(db.products);
      
      if (!productMap.containsKey(product.id)) {
        productMap[product.id] = ProductWithTags(
          product: product,
          tags: [],
        );
      }
      
      final tag = row.readTableOrNull(db.tags);
      if (tag != null) {
        productMap[product.id]!.tags.add(tag);
      }
    }
    
    return productMap.values.toList();
  }

  // ==================== AGGREGATE JOINS ====================
  
  // 👇 Get users with order statistics
  Future<List<UserWithStats>> getUsersWithOrderStats() async {
    final results = await db.customSelect('''
      SELECT 
        u.id as user_id,
        u.name as user_name,
        u.email as user_email,
        COUNT(o.id) as total_orders,
        SUM(o.total) as total_spent,
        AVG(o.total) as avg_order_value,
        MAX(o.total) as max_order_value,
        MIN(o.total) as min_order_value,
        COUNT(CASE WHEN o.status = 'completed' THEN 1 END) as completed_orders
      FROM users u
      LEFT JOIN orders o ON u.id = o.user_id
      WHERE u.is_active = 1
      GROUP BY u.id
      HAVING total_orders > 0
      ORDER BY total_spent DESC
    ''').get();
    
    return results.map((row) {
      return UserWithStats(
        id: row.data['user_id'] as int,
        name: row.data['user_name'] as String,
        email: row.data['user_email'] as String,
        totalOrders: row.data['total_orders'] as int,
        totalSpent: row.data['total_spent'] as double,
        avgOrderValue: row.data['avg_order_value'] as double,
        maxOrderValue: row.data['max_order_value'] as double,
        minOrderValue: row.data['min_order_value'] as double,
        completedOrders: row.data['completed_orders'] as int,
      );
    }).toList();
  }
  
  // 👇 Get products with sales stats
  Future<List<ProductWithSalesStats>> getProductsWithSalesStats() async {
    final results = await db.customSelect('''
      SELECT 
        p.id as product_id,
        p.name as product_name,
        p.sku as product_sku,
        p.price as product_price,
        COUNT(oi.id) as order_count,
        SUM(oi.quantity) as total_sold,
        SUM(oi.total) as total_revenue,
        AVG(oi.total) as avg_order_value,
        MAX(oi.total) as max_order_value,
        MIN(oi.total) as min_order_value,
        AVG(r.rating) as avg_rating,
        COUNT(r.id) as review_count
      FROM products p
      LEFT JOIN order_items oi ON p.id = oi.product_id
      LEFT JOIN reviews r ON p.id = r.product_id
      WHERE p.is_active = 1
      GROUP BY p.id
      HAVING order_count > 0
      ORDER BY total_revenue DESC
    ''').get();
    
    return results.map((row) {
      return ProductWithSalesStats(
        id: row.data['product_id'] as int,
        name: row.data['product_name'] as String,
        sku: row.data['product_sku'] as String,
        price: row.data['product_price'] as double,
        orderCount: row.data['order_count'] as int,
        totalSold: row.data['total_sold'] as int,
        totalRevenue: row.data['total_revenue'] as double,
        avgOrderValue: row.data['avg_order_value'] as double,
        maxOrderValue: row.data['max_order_value'] as double,
        minOrderValue: row.data['min_order_value'] as double,
        avgRating: row.data['avg_rating'] as double? ?? 0.0,
        reviewCount: row.data['review_count'] as int,
      );
    }).toList();
  }

  // ==================== SELF-JOIN ====================
  
  // 👇 Get employee hierarchy (self-join)
  Future<List<EmployeeWithManager>> getEmployeeHierarchy() async {
    final e = alias(db.users, 'e');
    final m = alias(db.users, 'm');
    
    final query = db.select(e).join([
      leftJoin(
        m,
        e.managerId.equals(m.id),
      )
    ]);
    
    query.where((u) => e.isActive.equals(true));
    query.orderBy([(u) => OrderingTerm.asc(e.department)]);
    
    final results = await query.get();
    
    return results.map((row) {
      final employee = row.readTable(e);
      final manager = row.readTableOrNull(m);
      
      return EmployeeWithManager(
        employee: employee,
        manager: manager,
      );
    }).toList();
  }
  
  // 👇 Get manager with subordinates
  Future<ManagerWithSubordinates> getManagerWithTeam(int managerId) async {
    final query = db.select(db.users)
      ..where((u) => u.managerId.equals(managerId))
      ..where((u) => u.isActive.equals(true))
      ..orderBy([(u) => OrderingTerm.asc(u.name)]);
    
    final subordinates = await query.get();
    
    final manager = await (db.select(db.users)
      ..where((u) => u.id.equals(managerId)))
      .getSingle();
    
    return ManagerWithSubordinates(
      manager: manager,
      subordinates: subordinates,
    );
  }
}

// ==================== DATA CLASSES ====================

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
  final List<OrderWithItems> orders;
  
  UserWithFullOrders({
    required this.user,
    required this.orders,
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

class ProductWithCategory {
  final Product product;
  final Category category;
  
  ProductWithCategory({
    required this.product,
    required this.category,
  });
}

class ProductWithDetails {
  final Product product;
  final Category category;
  final List<ReviewWithUser> reviews;
  
  ProductWithDetails({
    required this.product,
    required this.category,
    required this.reviews,
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

class ProductWithTags {
  final Product product;
  final List<Tag> tags;
  
  ProductWithTags({
    required this.product,
    required this.tags,
  });
}

class UserWithStats {
  final int id;
  final String name;
  final String email;
  final int totalOrders;
  final double totalSpent;
  final double avgOrderValue;
  final double maxOrderValue;
  final double minOrderValue;
  final int completedOrders;
  
  UserWithStats({
    required this.id,
    required this.name,
    required this.email,
    required this.totalOrders,
    required this.totalSpent,
    required this.avgOrderValue,
    required this.maxOrderValue,
    required this.minOrderValue,
    required this.completedOrders,
  });
}

class ProductWithSalesStats {
  final int id;
  final String name;
  final String sku;
  final double price;
  final int orderCount;
  final int totalSold;
  final double totalRevenue;
  final double avgOrderValue;
  final double maxOrderValue;
  final double minOrderValue;
  final double avgRating;
  final int reviewCount;
  
  ProductWithSalesStats({
    required this.id,
    required this.name,
    required this.sku,
    required this.price,
    required this.orderCount,
    required this.totalSold,
    required this.totalRevenue,
    required this.avgOrderValue,
    required this.maxOrderValue,
    required this.minOrderValue,
    required this.avgRating,
    required this.reviewCount,
  });
}

class EmployeeWithManager {
  final User employee;
  final User? manager;
  
  EmployeeWithManager({
    required this.employee,
    this.manager,
  });
}

class ManagerWithSubordinates {
  final User manager;
  final List<User> subordinates;
  
  ManagerWithSubordinates({
    required this.manager,
    required this.subordinates,
  });
}
```

---

# Best Practices

- **Use appropriate join type** – Inner vs Left vs Cross
- **Add conditions** – On the join or in WHERE
- **Use table aliases** – For self-joins
- **Read with `readTable` or `readTableOrNull`** – Safe extraction
- **Group results** – Handle one-to-many relationships
- **Use indexes** – On foreign key columns
- **Limit results** – For performance
- **Test queries** – Verify data integrity

---

# Common Mistakes

## Mistake 1: Wrong join type

Wrong:
```dart
// 🚫 Users without orders are excluded
innerJoin(db.orders, db.orders.userId.equals(db.users.id))
```

Correct:
```dart
// ✅ Users without orders are included
leftJoin(db.orders, db.orders.userId.equals(db.users.id))
```

## Mistake 2: Not handling null with left join

Wrong:
```dart
// 🚫 Null pointer exception
final order = row.readTable(db.orders);
print(order.orderNumber);
```

Correct:
```dart
// ✅ Handle null safely
final order = row.readTableOrNull(db.orders);
if (order != null) {
  print(order.orderNumber);
}
```

## Mistake 3: Missing join conditions

Wrong:
```dart
// 🚫 Cartesian product (all users × all orders)
innerJoin(db.orders, null)
```

Correct:
```dart
// ✅ Always specify join condition
innerJoin(db.orders, db.orders.userId.equals(db.users.id))
```

---

# Summary

| Join Type | Purpose | Returns |
|-----------|---------|---------|
| **Inner Join** | Only matching rows | Matching rows |
| **Left Join** | All left rows | Left + matching right |
| **Cross Join** | All combinations | Cartesian product |
| **Self Join** | Table with itself | Related rows |

---

# Next Steps

Now you understand relationships, let's dive deeper:

- [Many-to-Many](link) – Many-to-many relationships
- [One-to-Many](link) – One-to-many relationships
- [One-to-One](link) – One-to-one relationships

---

# Did You Know?

- **Joins are performed at the database** – Highly optimized

- **Inner joins are most common** – For related data

- **Left joins include all left rows** – Even without matches

- **Join order affects performance** – Smaller tables first

- **Indexes speed up joins** – On join columns

- **Self joins are powerful** – For hierarchical data

- **Joins can be nested** – Multiple joins together

- **Drift joins are type-safe** – Compile-time validation

---

