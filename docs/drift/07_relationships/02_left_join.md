## Left Join

**Including all records from the left table with Drift**

---

# What is it?

**Left Join** (or LEFT OUTER JOIN) returns all rows from the left table, and matching rows from the right table. If no match exists in the right table, NULL values are returned for the right table's columns. This is essential when you want to see all records from one table, regardless of whether they have related records in another table.

> **Think of Left Join like "checking attendance"** – you have a list of all students (left table) and want to see who has submitted homework (right table). You see all students, but those who didn't submit show "No homework" (NULL).

```dart
// 👇 Left join: All users with optional orders
final query = db.select(db.users).join([
  leftJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

// Returns ALL users, even those without orders
// Users without orders have null order data
final results = await query.get();

// Generated SQL:
// SELECT users.*, orders.* 
// FROM users 
// LEFT JOIN orders ON orders.user_id = users.id
```

> **What's happening here?**
> - **All left rows** – Every user is included
> - **Matching right rows** – Orders are included if they exist
> - **NULL for missing** – Users without orders get NULL order data
> - **Safe reading** – Use `readTableOrNull()` for optional tables

---

# Why does it exist?

- **Include All Records** – See all items from primary table
- **Missing Relationships** – Identify records without related data
- **Reports** – Show all users with or without orders
- **Data Auditing** – Find orphaned records
- **User Experience** – Show comprehensive lists
- **Data Analysis** – Analyze complete datasets

---

# Basic Left Join

> **Simple left join patterns**

## Single Left Join

```dart
// 👇 Left join: All users with optional orders
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
    print('${user.name} -> Order: ${order.orderNumber}');
  } else {
    print('${user.name} -> No orders');
  }
}

// Generated SQL:
// SELECT users.*, orders.* 
// FROM users 
// LEFT JOIN orders ON orders.user_id = users.id
```

## Left Join with WHERE on Left Table

```dart
// 👇 Left join with filter on left table
final query = db.select(db.users).join([
  leftJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

// Filter only active users
query.where((u) => db.users.isActive.equals(true));

final results = await query.get();

// All active users are included, with or without orders
```

## Left Join with WHERE on Right Table

```dart
// 👇 Left join with filter on right table
final query = db.select(db.users).join([
  leftJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

// Filter orders (applies after join)
query.where((u) => db.users.isActive.equals(true));
query.where((o) => db.orders.status.equals('completed'));

// All active users included
// If user has completed orders, they appear
// If user doesn't have completed orders, order data is NULL
final results = await query.get();
```

---

# Multiple Left Joins

> **Joining with multiple tables**

## Two Left Joins

```dart
// 👇 Users with optional orders and products
final query = db.select(db.users).join([
  leftJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  ),
  leftJoin(
    db.orderItems,
    db.orderItems.orderId.equals(db.orders.id),
  ),
]);

query.where((u) => db.users.isActive.equals(true));

final results = await query.get();

for (final row in results) {
  final user = row.readTable(db.users);
  final order = row.readTableOrNull(db.orders);
  final item = row.readTableOrNull(db.orderItems);
  
  if (order != null && item != null) {
    print('${user.name} -> ${order.orderNumber} -> Item #${item.id}');
  } else if (order != null) {
    print('${user.name} -> ${order.orderNumber} -> No items');
  } else {
    print('${user.name} -> No orders');
  }
}
```

## Left Join with Mixed Joins

```dart
// 👇 Left join + Inner join combined
final query = db.select(db.users).join([
  leftJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  ),
  innerJoin(
    db.orderItems,
    db.orderItems.orderId.equals(db.orders.id),
  ),
]);

// This query:
// - All users are included (left join)
// - Orders are included if they exist (left join)
// - Order items are included ONLY IF order exists (inner join)
// - Users without orders have no order items
```

---

# Left Join Use Cases

> **Common patterns with left join**

## Finding Records Without Relationships

```dart
// 👇 Find users without orders
final query = db.select(db.users).join([
  leftJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

// 👇 Using custom SQL with NULL check
final results = await db.customSelect('''
  SELECT u.id, u.name, u.email
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  WHERE o.id IS NULL
''').get();

// Returns users with NO orders
```

## Complete Dataset with Summary

```dart
// 👇 Users with order summary (including zero orders)
final results = await db.customSelect('''
  SELECT 
    u.id as user_id,
    u.name as user_name,
    COUNT(o.id) as order_count,
    COALESCE(SUM(o.total), 0) as total_spent,
    COALESCE(AVG(o.total), 0) as avg_order_value,
    CASE 
      WHEN COUNT(o.id) > 0 THEN 'Has Orders'
      ELSE 'No Orders'
    END as order_status
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id, u.name
  ORDER BY total_spent DESC
''').get();

for (final row in results) {
  print('${row.data['user_name']}');
  print('Orders: ${row.data['order_count']}');
  print('Total Spent: \$${row.data['total_spent']}');
  print('Status: ${row.data['order_status']}');
}
```

---

# Real-World Example

> **Complete e-commerce left join system**

```dart
// lib/database/left_join_service.dart
import 'package:drift/drift.dart';

class LeftJoinService {
  final AppDatabase db;
  
  LeftJoinService(this.db);

  // ==================== USER ORDER ANALYSIS ====================
  
  // 👇 Get all users with order status
  Future<List<UserOrderStatus>> getAllUsersWithOrderStatus() async {
    final results = await db.customSelect('''
      SELECT 
        u.id as user_id,
        u.name as user_name,
        u.email as user_email,
        u.is_active as is_active,
        COUNT(o.id) as order_count,
        COALESCE(SUM(o.total), 0) as total_spent,
        COALESCE(AVG(o.total), 0) as avg_order_value,
        MAX(o.order_date) as last_order_date,
        CASE 
          WHEN COUNT(o.id) = 0 THEN 'No Orders'
          WHEN COUNT(o.id) < 5 THEN 'Occasional'
          WHEN COUNT(o.id) < 20 THEN 'Regular'
          ELSE 'Frequent'
        END as order_frequency,
        CASE 
          WHEN JULIANDAY('now') - JULIANDAY(MAX(o.order_date)) < 30 AND COUNT(o.id) > 0 THEN 'Active'
          WHEN COUNT(o.id) > 0 THEN 'Inactive'
          ELSE 'Never Ordered'
        END as activity_status
      FROM users u
      LEFT JOIN orders o ON u.id = o.user_id
      GROUP BY u.id, u.name, u.email, u.is_active
      ORDER BY total_spent DESC
    ''').get();
    
    return results.map((row) {
      return UserOrderStatus(
        userId: row.data['user_id'] as int,
        userName: row.data['user_name'] as String,
        userEmail: row.data['user_email'] as String,
        isActive: (row.data['is_active'] as int) == 1,
        orderCount: row.data['order_count'] as int,
        totalSpent: row.data['total_spent'] as double,
        avgOrderValue: row.data['avg_order_value'] as double,
        lastOrderDate: row.data['last_order_date'] != null
            ? DateTime.parse(row.data['last_order_date'] as String)
            : null,
        orderFrequency: row.data['order_frequency'] as String,
        activityStatus: row.data['activity_status'] as String,
      );
    }).toList();
  }

  // ==================== PRODUCT CATEGORY ANALYSIS ====================
  
  // 👇 Get all products with category info (including uncategorized)
  Future<List<ProductCategoryInfo>> getAllProductsWithCategory() async {
    final results = await db.customSelect('''
      SELECT 
        p.id as product_id,
        p.name as product_name,
        p.sku as product_sku,
        p.price as product_price,
        p.stock as stock,
        p.is_active as is_active,
        c.id as category_id,
        c.name as category_name,
        CASE 
          WHEN c.id IS NULL THEN 'Uncategorized'
          ELSE c.name
        END as category_display
      FROM products p
      LEFT JOIN categories c ON p.category_id = c.id
      ORDER BY category_display, p.name
    ''').get();
    
    return results.map((row) {
      return ProductCategoryInfo(
        productId: row.data['product_id'] as int,
        productName: row.data['product_name'] as String,
        productSku: row.data['product_sku'] as String,
        productPrice: row.data['product_price'] as double,
        stock: row.data['stock'] as int,
        isActive: (row.data['is_active'] as int) == 1,
        categoryId: row.data['category_id'] as int?,
        categoryName: row.data['category_name'] as String?,
        categoryDisplay: row.data['category_display'] as String,
      );
    }).toList();
  }

  // ==================== ORDER ITEM ANALYSIS ====================
  
  // 👇 Get order details with product information
  Future<OrderWithItemDetails> getOrderWithItemDetails(int orderId) async {
    final query = db.select(db.orders).join([
      leftJoin(
        db.orderItems,
        db.orderItems.orderId.equals(db.orders.id),
      ),
      leftJoin(
        db.products,
        db.products.id.equals(db.orderItems.productId),
      ),
    ]);
    
    query.where((o) => db.orders.id.equals(orderId));
    
    final results = await query.get();
    
    if (results.isEmpty) {
      throw Exception('Order not found');
    }
    
    final order = results.first.readTable(db.orders);
    final items = <OrderItemWithProductInfo>[];
    
    for (final row in results) {
      final item = row.readTableOrNull(db.orderItems);
      if (item != null) {
        final product = row.readTableOrNull(db.products);
        items.add(OrderItemWithProductInfo(
          orderItem: item,
          product: product,
        ));
      }
    }
    
    return OrderWithItemDetails(
      order: order,
      items: items,
    );
  }

  // ==================== USER REVIEWS ANALYSIS ====================
  
  // 👇 Get users with their reviews
  Future<List<UserWithReviews>> getUsersWithReviews() async {
    final query = db.select(db.users).join([
      leftJoin(
        db.reviews,
        db.reviews.userId.equals(db.users.id),
      ),
      leftJoin(
        db.products,
        db.products.id.equals(db.reviews.productId),
      ),
    ]);
    
    query.where((u) => db.users.isActive.equals(true));
    
    final results = await query.get();
    
    // 👇 Group by user
    final userMap = <int, UserWithReviews>{};
    
    for (final row in results) {
      final user = row.readTable(db.users);
      
      if (!userMap.containsKey(user.id)) {
        userMap[user.id] = UserWithReviews(
          user: user,
          reviews: [],
        );
      }
      
      final review = row.readTableOrNull(db.reviews);
      if (review != null) {
        final product = row.readTable(db.products);
        userMap[user.id]!.reviews.add(
          ReviewWithProduct(
            review: review,
            product: product,
          )
        );
      }
    }
    
    return userMap.values.toList();
  }

  // ==================== LEFT JOIN WITH AGGREGATION ====================
  
  // 👇 Get category performance with left join
  Future<List<CategoryPerformance>> getCategoryPerformance() async {
    final results = await db.customSelect('''
      SELECT 
        c.id as category_id,
        c.name as category_name,
        COUNT(p.id) as total_products,
        COUNT(CASE WHEN p.is_active = 1 THEN 1 END) as active_products,
        COALESCE(AVG(p.price), 0) as avg_price,
        COALESCE(SUM(p.stock), 0) as total_stock,
        COALESCE(SUM(p.price * p.stock), 0) as inventory_value,
        COUNT(oi.id) as items_sold,
        COALESCE(SUM(oi.total), 0) as total_revenue,
        CASE 
          WHEN COUNT(p.id) = 0 THEN 'No Products'
          WHEN COUNT(oi.id) = 0 THEN 'No Sales'
          ELSE 'Has Sales'
        END as performance_status
      FROM categories c
      LEFT JOIN products p ON c.id = p.category_id
      LEFT JOIN order_items oi ON p.id = oi.product_id
      GROUP BY c.id, c.name
      ORDER BY total_revenue DESC
    ''').get();
    
    return results.map((row) {
      return CategoryPerformance(
        categoryId: row.data['category_id'] as int,
        categoryName: row.data['category_name'] as String,
        totalProducts: row.data['total_products'] as int,
        activeProducts: row.data['active_products'] as int,
        avgPrice: row.data['avg_price'] as double,
        totalStock: row.data['total_stock'] as int,
        inventoryValue: row.data['inventory_value'] as double,
        itemsSold: row.data['items_sold'] as int,
        totalRevenue: row.data['total_revenue'] as double,
        performanceStatus: row.data['performance_status'] as String,
      );
    }).toList();
  }

  // ==================== ADDRESS ANALYSIS ====================
  
  // 👇 Get users with addresses (including no address)
  Future<List<UserWithAddress>> getUsersWithAddresses() async {
    final query = db.select(db.users).join([
      leftJoin(
        db.addresses,
        db.addresses.userId.equals(db.users.id),
      )
    ]);
    
    query.where((u) => db.users.isActive.equals(true));
    query.orderBy([(u) => OrderingTerm.asc(db.users.name)]);
    
    final results = await query.get();
    
    final userMap = <int, UserWithAddress>{};
    
    for (final row in results) {
      final user = row.readTable(db.users);
      
      if (!userMap.containsKey(user.id)) {
        userMap[user.id] = UserWithAddress(
          user: user,
          addresses: [],
        );
      }
      
      final address = row.readTableOrNull(db.addresses);
      if (address != null) {
        userMap[user.id]!.addresses.add(address);
      }
    }
    
    return userMap.values.toList();
  }

  // ==================== INACTIVE USER ANALYSIS ====================
  
  // 👇 Find inactive users with no orders
  Future<List<User>> getInactiveUsersWithNoOrders() async {
    final results = await db.customSelect('''
      SELECT u.*
      FROM users u
      LEFT JOIN orders o ON u.id = o.user_id
      WHERE u.is_active = 0
      GROUP BY u.id
      HAVING COUNT(o.id) = 0
    ''').get();
    
    // Map results to User objects (simplified)
    return results.map((row) {
      return User(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        email: row.data['email'] as String,
        isActive: (row.data['is_active'] as int) == 1,
        // ... other fields
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class UserOrderStatus {
  final int userId;
  final String userName;
  final String userEmail;
  final bool isActive;
  final int orderCount;
  final double totalSpent;
  final double avgOrderValue;
  final DateTime? lastOrderDate;
  final String orderFrequency;
  final String activityStatus;
  
  UserOrderStatus({
    required this.userId,
    required this.userName,
    required this.userEmail,
    required this.isActive,
    required this.orderCount,
    required this.totalSpent,
    required this.avgOrderValue,
    this.lastOrderDate,
    required this.orderFrequency,
    required this.activityStatus,
  });
}

class ProductCategoryInfo {
  final int productId;
  final String productName;
  final String productSku;
  final double productPrice;
  final int stock;
  final bool isActive;
  final int? categoryId;
  final String? categoryName;
  final String categoryDisplay;
  
  ProductCategoryInfo({
    required this.productId,
    required this.productName,
    required this.productSku,
    required this.productPrice,
    required this.stock,
    required this.isActive,
    this.categoryId,
    this.categoryName,
    required this.categoryDisplay,
  });
}

class OrderWithItemDetails {
  final Order order;
  final List<OrderItemWithProductInfo> items;
  
  OrderWithItemDetails({
    required this.order,
    required this.items,
  });
}

class OrderItemWithProductInfo {
  final OrderItem orderItem;
  final Product? product;
  
  OrderItemWithProductInfo({
    required this.orderItem,
    this.product,
  });
}

class UserWithReviews {
  final User user;
  final List<ReviewWithProduct> reviews;
  
  UserWithReviews({
    required this.user,
    required this.reviews,
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

class CategoryPerformance {
  final int categoryId;
  final String categoryName;
  final int totalProducts;
  final int activeProducts;
  final double avgPrice;
  final int totalStock;
  final double inventoryValue;
  final int itemsSold;
  final double totalRevenue;
  final String performanceStatus;
  
  CategoryPerformance({
    required this.categoryId,
    required this.categoryName,
    required this.totalProducts,
    required this.activeProducts,
    required this.avgPrice,
    required this.totalStock,
    required this.inventoryValue,
    required this.itemsSold,
    required this.totalRevenue,
    required this.performanceStatus,
  });
}

class UserWithAddress {
  final User user;
  final List<Address> addresses;
  
  UserWithAddress({
    required this.user,
    required this.addresses,
  });
}
```

```dart
// lib/ui/pages/user_list_page.dart
class UserListPage extends StatefulWidget {
  final LeftJoinService joinService;
  
  const UserListPage({required this.joinService});
  
  @override
  _UserListPageState createState() => _UserListPageState();
}

class _UserListPageState extends State<UserListPage> {
  List<UserOrderStatus> _users = [];
  bool _isLoading = true;
  
  @override
  void initState() {
    super.initState();
    _loadUsers();
  }
  
  Future<void> _loadUsers() async {
    setState(() => _isLoading = true);
    
    try {
      final users = await widget.joinService.getAllUsersWithOrderStatus();
      
      setState(() {
        _users = users;
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
        title: Text('Users with Order Status'),
      ),
      body: _isLoading
          ? Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: _users.length,
              itemBuilder: (context, index) {
                final user = _users[index];
                return Card(
                  margin: EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                  child: ListTile(
                    title: Text(user.userName),
                    subtitle: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text('Email: ${user.userEmail}'),
                        Text('Orders: ${user.orderCount}'),
                        Text('Total Spent: \$${user.totalSpent.toStringAsFixed(2)}'),
                        Text('Status: ${user.activityStatus}'),
                      ],
                    ),
                    trailing: Container(
                      padding: EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                      decoration: BoxDecoration(
                        color: user.isActive ? Colors.green : Colors.red,
                        borderRadius: BorderRadius.circular(12),
                      ),
                      child: Text(
                        user.isActive ? 'Active' : 'Inactive',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                );
              },
            ),
    );
  }
}
```

---

# Left Join vs Inner Join

> **When to use left join vs inner join**

| Feature | Left Join | Inner Join |
|---------|-----------|------------|
| **Includes** | All left rows | Only matching rows |
| **Missing Data** | Shows NULL | Excludes rows |
| **Use Case** | "Show all users" | "Show users with orders" |
| **NULL Handling** | `readTableOrNull()` | `readTable()` |
| **Performance** | Slightly slower | Faster |
| **Purpose** | Complete datasets | Matched data |

```dart
// 👇 Inner Join: Users who HAVE placed orders
final innerResult = await db.select(db.users).join([
  innerJoin(db.orders, db.orders.userId.equals(db.users.id))
]).get(); // Only users with orders

// 👇 Left Join: All users (with or without orders)
final leftResult = await db.select(db.users).join([
  leftJoin(db.orders, db.orders.userId.equals(db.users.id))
]).get(); // All users
```

---

# Best Practices

- **Use Left Join for optional relationships** – Data may not exist
- **Always use `readTableOrNull()`** – For optional right tables
- **Check for NULL** – Before accessing optional data
- **Use COALESCE** – For default values in reports
- **Add WHERE on left table first** – For performance
- **Use with COUNT** – To count related records
- **Use with aggregations** – For complete summaries
- **Test with data** – Verify NULL handling

---

# Common Mistakes

## Mistake 1: Using `readTable()` on optional data

Wrong:
```dart
// 🚫 Throws exception if no order exists
final order = row.readTable(db.orders);
print(order.orderNumber);
```

Correct:
```dart
// ✅ Handle potential NULL
final order = row.readTableOrNull(db.orders);
if (order != null) {
  print(order.orderNumber);
}
```

## Mistake 2: Filtering on right table

Wrong:
```dart
// 🚫 Right table filter may hide NULLs
query.where((o) => db.orders.status.equals('completed'));
// This creates an implicit INNER JOIN effect
```

Correct:
```dart
// ✅ Filter on left table
query.where((u) => db.users.isActive.equals(true));
// OR handle NULL after join
```

## Mistake 3: Not handling NULL in aggregations

Wrong:
```dart
// 🚫 SUM returns NULL for no rows
final total = await db.select(db.users).join([
  leftJoin(db.orders, ...)
]).map((o) => db.orders.total).sum();
```

Correct:
```dart
// ✅ Use COALESCE for default
final total = await db.customSelect('''
  SELECT COALESCE(SUM(total), 0) as total
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
''').get();
```

---

# Summary

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Left Join** | All left rows | Complete datasets |
| **NULL Handling** | `readTableOrNull()` | Optional data |
| **COALESCE** | Default values | Reports |
| **Multiple Left Joins** | Chain joins | Complex queries |

---

# Next Steps

Now you understand left join, let's dive deeper:

- [Cross Join](link) – Cross join
- [Self Join](link) – Self join
- [Many-to-Many](link) – Many-to-many relationships

---

# Did You Know?

- **Left join is also called LEFT OUTER JOIN** – Same thing

- **Left join is not symmetric** – Table order matters

- **Left join can be slower** – Than inner join

- **Left join shows all left rows** – Even without matches

- **Left join is essential** – For complete reports

- **Left join can be converted** – To inner join with WHERE

- **Left join handles NULL** – On right table columns

- **Left join is common** – In business reporting

---

