## Inner Join

**Combining related data with inner joins in Drift**

---

# What is it?

**Inner Join** is a type of join that returns only the rows that have matching values in both tables. It's the most commonly used join type for retrieving related data where records exist on both sides of the relationship. In Drift, you use `innerJoin()` to create inner joins between tables.

> **Think of Inner Join like "common friends on social media"** – you only see people who follow you AND you follow them back (mutual connection). If you follow someone but they don't follow back, they're not included.

```dart
// 👇 Inner join: Users who have placed orders
final query = db.select(db.users).join([
  innerJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

// Returns users who have at least one order
// Users without orders are excluded
final results = await query.get();

// Generated SQL:
// SELECT users.*, orders.* 
// FROM users 
// INNER JOIN orders ON orders.user_id = users.id
```

> **What's happening here?**
> - **Matching rows only** – Both tables must match
> - **Condition** – `ON orders.user_id = users.id`
> - **Excludes** – Users without orders, orders without users
> - **Result** – Combined data from both tables

---

# Why does it exist?

- **Get Related Data** – Retrieve connected records
- **Data Integrity** – Only returns valid relationships
- **Performance** – Single query instead of multiple
- **Reporting** – Generate combined reports
- **Business Logic** – Implement business rules
- **Analysis** – Analyze related data

---

# Basic Inner Join

> **Simple inner join patterns**

## Single Inner Join

```dart
// 👇 Join users with their orders
final query = db.select(db.users).join([
  innerJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

final results = await query.get();
for (final row in results) {
  final user = row.readTable(db.users);
  final order = row.readTable(db.orders);
  print('${user.name} -> Order: ${order.orderNumber}');
}

// Generated SQL:
// SELECT users.*, orders.* 
// FROM users 
// INNER JOIN orders ON orders.user_id = users.id
```

## Inner Join with WHERE

```dart
// 👇 Join with additional condition
final query = db.select(db.users).join([
  innerJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

// Only active users with completed orders
query.where((u) => db.users.isActive.equals(true));
query.where((o) => db.orders.status.equals('completed'));

final results = await query.get();
```

## Inner Join with Specific Columns

```dart
// 👇 Select specific columns from joined tables
final query = db.select(db.users)
  .join([
    innerJoin(
      db.orders,
      db.orders.userId.equals(db.users.id),
    )
  ])
  .map((u, o) => [
    u.id,
    u.name,
    o.orderNumber,
    o.total,
  ]);

final results = await query.get();
for (final row in results) {
  final userId = row[0] as int;
  final userName = row[1] as String;
  final orderNumber = row[2] as String;
  final total = row[3] as double;
  
  print('$userName (ID: $userId) - Order: $orderNumber, Total: \$$total');
}
```

---

# Multiple Inner Joins

> **Joining more than two tables**

## Three-Table Inner Join

```dart
// 👇 Users -> Orders -> Order Items
final query = db.select(db.users).join([
  innerJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  ),
  innerJoin(
    db.orderItems,
    db.orderItems.orderId.equals(db.orders.id),
  ),
]);

final results = await query.get();
for (final row in results) {
  final user = row.readTable(db.users);
  final order = row.readTable(db.orders);
  final item = row.readTable(db.orderItems);
  
  print('${user.name} ordered item #${item.id} in order ${order.orderNumber}');
}
```

## Four-Table Inner Join

```dart
// 👇 Users -> Orders -> Items -> Products
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

// 👇 Map to DTO
final orderProducts = results.map((row) {
  final user = row.readTable(db.users);
  final order = row.readTable(db.orders);
  final item = row.readTable(db.orderItems);
  final product = row.readTable(db.products);
  
  return OrderProductDto(
    userName: user.name,
    orderNumber: order.orderNumber,
    productName: product.name,
    quantity: item.quantity,
    total: item.total,
  );
}).toList();
```

---

# Inner Join with Aggregations

> **Joining and aggregating data**

## Group By with Inner Join

```dart
// 👇 Users with order counts
final query = db.select(db.users).join([
  innerJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]);

// 👇 Use custom SQL for aggregation
final results = await db.customSelect('''
  SELECT 
    u.id as user_id,
    u.name as user_name,
    COUNT(o.id) as order_count,
    SUM(o.total) as total_spent,
    AVG(o.total) as avg_order_value
  FROM users u
  INNER JOIN orders o ON u.id = o.user_id
  WHERE u.is_active = 1
  GROUP BY u.id, u.name
  HAVING order_count > 5
  ORDER BY total_spent DESC
''').get();

for (final row in results) {
  print('User: ${row.data['user_name']}');
  print('Orders: ${row.data['order_count']}');
  print('Total Spent: \$${row.data['total_spent']}');
  print('Avg: \$${row.data['avg_order_value']}');
}
```

---

# Real-World Example

> **Complete e-commerce inner join system**

```dart
// lib/database/inner_join_service.dart
import 'package:drift/drift.dart';

class InnerJoinService {
  final AppDatabase db;
  
  InnerJoinService(this.db);

  // ==================== USER-ORDER JOINS ====================
  
  // 👇 Get active users with orders
  Future<List<UserOrderSummary>> getActiveUsersWithOrders() async {
    final query = db.select(db.users).join([
      innerJoin(
        db.orders,
        db.orders.userId.equals(db.users.id),
      )
    ]);
    
    query.where((u) => db.users.isActive.equals(true));
    query.where((o) => db.orders.status.equals('completed'));
    query.orderBy([(u) => OrderingTerm.desc(db.orders.orderDate)]);
    
    final results = await query.get();
    
    // 👇 Group by user
    final userMap = <int, UserOrderSummary>{};
    
    for (final row in results) {
      final user = row.readTable(db.users);
      
      if (!userMap.containsKey(user.id)) {
        userMap[user.id] = UserOrderSummary(
          id: user.id,
          name: user.name,
          email: user.email,
          orders: [],
          totalOrders: 0,
          totalSpent: 0.0,
        );
      }
      
      final order = row.readTable(db.orders);
      userMap[user.id]!.orders.add(order);
      userMap[user.id]!.totalOrders++;
      userMap[user.id]!.totalSpent += order.total;
    }
    
    return userMap.values.toList();
  }

  // 👇 Get user with order details (Inner Join)
  Future<UserOrderDetail> getUserOrderDetails(int userId) async {
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
    query.where((o) => db.orders.status.equals('completed'));
    
    final results = await query.get();
    
    if (results.isEmpty) {
      throw Exception('User not found');
    }
    
    final user = results.first.readTable(db.users);
    final ordersMap = <int, OrderWithProducts>{};
    
    for (final row in results) {
      final order = row.readTable(db.orders);
      final item = row.readTable(db.orderItems);
      final product = row.readTable(db.products);
      
      if (!ordersMap.containsKey(order.id)) {
        ordersMap[order.id] = OrderWithProducts(
          order: order,
          items: [],
          totalItems: 0,
          totalAmount: 0.0,
        );
      }
      
      ordersMap[order.id]!.items.add(
        OrderItemDetail(
          productName: product.name,
          sku: product.sku,
          quantity: item.quantity,
          unitPrice: item.unitPrice,
          total: item.total,
        )
      );
      ordersMap[order.id]!.totalItems += item.quantity;
      ordersMap[order.id]!.totalAmount += item.total;
    }
    
    return UserOrderDetail(
      user: user,
      orders: ordersMap.values.toList(),
    );
  }

  // ==================== PRODUCT-CATEGORY JOINS ====================
  
  // 👇 Get products with categories
  Future<List<ProductCategory>> getProductsWithCategories() async {
    final query = db.select(db.products).join([
      innerJoin(
        db.categories,
        db.categories.id.equals(db.products.categoryId),
      )
    ]);
    
    query.where((p) => db.products.isActive.equals(true));
    query.orderBy([(p) => OrderingTerm.asc(db.categories.name)]);
    query.orderBy([(p) => OrderingTerm.asc(db.products.name)]);
    
    final results = await query.get();
    
    return results.map((row) {
      final product = row.readTable(db.products);
      final category = row.readTable(db.categories);
      
      return ProductCategory(
        product: product,
        category: category,
      );
    }).toList();
  }
  
  // 👇 Get products with categories and tags
  Future<List<ProductTagCategory>> getProductsWithCategoriesAndTags() async {
    final query = db.select(db.products).join([
      innerJoin(
        db.categories,
        db.categories.id.equals(db.products.categoryId),
      ),
      innerJoin(
        db.productTags,
        db.productTags.productId.equals(db.products.id),
      ),
      innerJoin(
        db.tags,
        db.tags.id.equals(db.productTags.tagId),
      ),
    ]);
    
    query.where((p) => db.products.isActive.equals(true));
    
    final results = await query.get();
    
    final productMap = <int, ProductTagCategory>{};
    
    for (final row in results) {
      final product = row.readTable(db.products);
      
      if (!productMap.containsKey(product.id)) {
        final category = row.readTable(db.categories);
        productMap[product.id] = ProductTagCategory(
          product: product,
          category: category,
          tags: [],
        );
      }
      
      final tag = row.readTable(db.tags);
      productMap[product.id]!.tags.add(tag);
    }
    
    return productMap.values.toList();
  }

  // ==================== ORDER-ITEM-PRODUCT JOINS ====================
  
  // 👇 Get order details with products
  Future<OrderWithFullDetails> getOrderDetails(int orderId) async {
    final query = db.select(db.orders).join([
      innerJoin(
        db.orderItems,
        db.orderItems.orderId.equals(db.orders.id),
      ),
      innerJoin(
        db.products,
        db.products.id.equals(db.orderItems.productId),
      ),
      innerJoin(
        db.users,
        db.users.id.equals(db.orders.userId),
      ),
    ]);
    
    query.where((o) => db.orders.id.equals(orderId));
    
    final results = await query.get();
    
    if (results.isEmpty) {
      throw Exception('Order not found');
    }
    
    final order = results.first.readTable(db.orders);
    final user = results.first.readTable(db.users);
    final items = results.map((row) {
      final item = row.readTable(db.orderItems);
      final product = row.readTable(db.products);
      
      return OrderItemDetail(
        productName: product.name,
        sku: product.sku,
        quantity: item.quantity,
        unitPrice: item.unitPrice,
        total: item.total,
      );
    }).toList();
    
    return OrderWithFullDetails(
      order: order,
      user: user,
      items: items,
    );
  }

  // ==================== SELF INNER JOIN ====================
  
  // 👇 Get employees with managers
  Future<List<EmployeeWithManager>> getEmployeesWithManagers() async {
    final e = alias(db.users, 'e');
    final m = alias(db.users, 'm');
    
    final query = db.select(e).join([
      innerJoin(
        m,
        e.managerId.equals(m.id),
      )
    ]);
    
    query.where((u) => e.isActive.equals(true));
    query.where((u) => m.isActive.equals(true));
    query.orderBy([(u) => OrderingTerm.asc(e.name)]);
    
    final results = await query.get();
    
    return results.map((row) {
      final employee = row.readTable(e);
      final manager = row.readTable(m);
      
      return EmployeeWithManager(
        employee: employee,
        manager: manager,
      );
    }).toList();
  }

  // ==================== ADVANCED INNER JOINS ====================
  
  // 👇 Get top products with sales (Inner Join to get only sold products)
  Future<List<TopProductSales>> getTopSellingProducts() async {
    final results = await db.customSelect('''
      SELECT 
        p.id as product_id,
        p.name as product_name,
        p.sku as product_sku,
        COUNT(oi.id) as order_count,
        SUM(oi.quantity) as total_sold,
        SUM(oi.total) as total_revenue,
        AVG(oi.quantity) as avg_quantity_per_order
      FROM products p
      INNER JOIN order_items oi ON p.id = oi.product_id
      WHERE p.is_active = 1
      GROUP BY p.id, p.name, p.sku
      ORDER BY total_revenue DESC
      LIMIT 20
    ''').get();
    
    return results.map((row) {
      return TopProductSales(
        productId: row.data['product_id'] as int,
        productName: row.data['product_name'] as String,
        productSku: row.data['product_sku'] as String,
        orderCount: row.data['order_count'] as int,
        totalSold: row.data['total_sold'] as int,
        totalRevenue: row.data['total_revenue'] as double,
        avgQuantityPerOrder: row.data['avg_quantity_per_order'] as double,
      );
    }).toList();
  }

  // 👇 Get customer lifetime value (Inner Join on orders)
  Future<List<CustomerLifetimeValue>> getCustomerLifetimeValue() async {
    final results = await db.customSelect('''
      SELECT 
        u.id as user_id,
        u.name as user_name,
        u.email as user_email,
        COUNT(DISTINCT o.id) as total_orders,
        SUM(o.total) as total_spent,
        AVG(o.total) as avg_order_value,
        MAX(o.total) as max_order_value,
        MIN(o.total) as min_order_value,
        DATE('now') - DATE(MAX(o.order_date)) as days_since_last_order
      FROM users u
      INNER JOIN orders o ON u.id = o.user_id
      WHERE u.is_active = 1
        AND o.status = 'completed'
        AND o.total > 0
      GROUP BY u.id, u.name, u.email
      HAVING total_orders > 5
      ORDER BY total_spent DESC
    ''').get();
    
    return results.map((row) {
      return CustomerLifetimeValue(
        userId: row.data['user_id'] as int,
        userName: row.data['user_name'] as String,
        userEmail: row.data['user_email'] as String,
        totalOrders: row.data['total_orders'] as int,
        totalSpent: row.data['total_spent'] as double,
        avgOrderValue: row.data['avg_order_value'] as double,
        maxOrderValue: row.data['max_order_value'] as double,
        minOrderValue: row.data['min_order_value'] as double,
        daysSinceLastOrder: row.data['days_since_last_order'] as int,
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class UserOrderSummary {
  final int id;
  final String name;
  final String email;
  final List<Order> orders;
  int totalOrders;
  double totalSpent;
  
  UserOrderSummary({
    required this.id,
    required this.name,
    required this.email,
    required this.orders,
    this.totalOrders = 0,
    this.totalSpent = 0.0,
  });
}

class UserOrderDetail {
  final User user;
  final List<OrderWithProducts> orders;
  
  UserOrderDetail({
    required this.user,
    required this.orders,
  });
}

class OrderWithProducts {
  final Order order;
  final List<OrderItemDetail> items;
  int totalItems;
  double totalAmount;
  
  OrderWithProducts({
    required this.order,
    required this.items,
    this.totalItems = 0,
    this.totalAmount = 0.0,
  });
}

class OrderItemDetail {
  final String productName;
  final String sku;
  final int quantity;
  final double unitPrice;
  final double total;
  
  OrderItemDetail({
    required this.productName,
    required this.sku,
    required this.quantity,
    required this.unitPrice,
    required this.total,
  });
}

class ProductCategory {
  final Product product;
  final Category category;
  
  ProductCategory({
    required this.product,
    required this.category,
  });
}

class ProductTagCategory {
  final Product product;
  final Category category;
  final List<Tag> tags;
  
  ProductTagCategory({
    required this.product,
    required this.category,
    required this.tags,
  });
}

class OrderWithFullDetails {
  final Order order;
  final User user;
  final List<OrderItemDetail> items;
  
  OrderWithFullDetails({
    required this.order,
    required this.user,
    required this.items,
  });
}

class EmployeeWithManager {
  final User employee;
  final User manager;
  
  EmployeeWithManager({
    required this.employee,
    required this.manager,
  });
}

class TopProductSales {
  final int productId;
  final String productName;
  final String productSku;
  final int orderCount;
  final int totalSold;
  final double totalRevenue;
  final double avgQuantityPerOrder;
  
  TopProductSales({
    required this.productId,
    required this.productName,
    required this.productSku,
    required this.orderCount,
    required this.totalSold,
    required this.totalRevenue,
    required this.avgQuantityPerOrder,
  });
}

class CustomerLifetimeValue {
  final int userId;
  final String userName;
  final String userEmail;
  final int totalOrders;
  final double totalSpent;
  final double avgOrderValue;
  final double maxOrderValue;
  final double minOrderValue;
  final int daysSinceLastOrder;
  
  CustomerLifetimeValue({
    required this.userId,
    required this.userName,
    required this.userEmail,
    required this.totalOrders,
    required this.totalSpent,
    required this.avgOrderValue,
    required this.maxOrderValue,
    required this.minOrderValue,
    required this.daysSinceLastOrder,
  });
}

class OrderProductDto {
  final String userName;
  final String orderNumber;
  final String productName;
  final int quantity;
  final double total;
  
  OrderProductDto({
    required this.userName,
    required this.orderNumber,
    required this.productName,
    required this.quantity,
    required this.total,
  });
}
```

```dart
// lib/ui/pages/user_orders_page.dart
class UserOrdersPage extends StatefulWidget {
  final InnerJoinService joinService;
  final int userId;
  
  const UserOrdersPage({
    required this.joinService,
    required this.userId,
  });
  
  @override
  _UserOrdersPageState createState() => _UserOrdersPageState();
}

class _UserOrdersPageState extends State<UserOrdersPage> {
  UserOrderDetail? _orderDetail;
  bool _isLoading = true;
  String? _error;
  
  @override
  void initState() {
    super.initState();
    _loadData();
  }
  
  Future<void> _loadData() async {
    setState(() => _isLoading = true);
    
    try {
      final data = await widget.joinService.getUserOrderDetails(
        widget.userId,
      );
      
      setState(() {
        _orderDetail = data;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Order History'),
      ),
      body: _isLoading
          ? Center(child: CircularProgressIndicator())
          : _error != null
              ? _buildError()
              : _buildContent(),
    );
  }
  
  Widget _buildError() {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.error, size: 64, color: Colors.red),
          SizedBox(height: 16),
          Text('Error: $_error'),
          SizedBox(height: 16),
          ElevatedButton(
            onPressed: _loadData,
            child: Text('Retry'),
          ),
        ],
      ),
    );
  }
  
  Widget _buildContent() {
    final data = _orderDetail!;
    
    return ListView(
      padding: EdgeInsets.all(16),
      children: [
        // User info
        Card(
          child: ListTile(
            title: Text(data.user.name),
            subtitle: Text(data.user.email),
          ),
        ),
        SizedBox(height: 16),
        
        // Orders
        for (final order in data.orders)
          Card(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Text(
                        'Order #${order.order.orderNumber}',
                        style: TextStyle(fontWeight: FontWeight.bold),
                      ),
                      Text('\$${order.totalAmount.toStringAsFixed(2)}'),
                    ],
                  ),
                  Text('Status: ${order.order.status}'),
                  Text('Items: ${order.totalItems}'),
                  SizedBox(height: 8),
                  ...order.items.map((item) => Padding(
                    padding: EdgeInsets.symmetric(vertical: 4),
                    child: Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Text('${item.quantity}x ${item.productName}'),
                        Text('\$${item.total.toStringAsFixed(2)}'),
                      ],
                    ),
                  )),
                ],
              ),
            ),
          ),
      ],
    );
  }
}
```

---

# Inner Join vs Other Joins

> **Understanding when to use inner join**

```dart
// 👇 Inner Join: Only active users with orders
final innerResult = await db.select(db.users).join([
  innerJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]).get(); // Only users with orders

// 👇 Left Join: All users with optional orders
final leftResult = await db.select(db.users).join([
  leftJoin(
    db.orders,
    db.orders.userId.equals(db.users.id),
  )
]).get(); // All users (orders may be null)

// 👇 Cross Join: All combinations
final crossResult = await db.select(db.users).join([
  crossJoin(db.orders),
]).get(); // Every user × every order
```

---

# Best Practices

- **Use inner join for required relationships** – Only when data exists
- **Always specify join condition** – Avoid Cartesian product
- **Use table aliases** – For self-joins
- **Add WHERE conditions** – Filter results
- **Use indexes** – On foreign key columns
- **Join in optimal order** – Smaller tables first
- **Select only needed columns** – For performance
- **Test with data** – Verify join results

---

# Common Mistakes

## Mistake 1: Missing join condition

Wrong:
```dart
// 🚫 Cartesian product (all users × all orders)
innerJoin(db.orders)
```

Correct:
```dart
// ✅ Always specify join condition
innerJoin(db.orders, db.orders.userId.equals(db.users.id))
```

## Mistake 2: Wrong table order

Wrong:
```dart
// 🚫 Less efficient join order
innerJoin(db.orderItems, db.orders.id.equals(db.orderItems.orderId))
```

Correct:
```dart
// ✅ Join smaller to larger (if possible)
innerJoin(db.orderItems, db.orderItems.orderId.equals(db.orders.id))
```

## Mistake 3: Not handling joined data

Wrong:
```dart
// 🚫 Without reading all tables
final query = db.select(db.users).join([
  innerJoin(db.orders, ...)
]);
final results = await query.get();
final user = results.first.readTable(db.users); // ✅ User exists
// But forgot to read orders
```

Correct:
```dart
// ✅ Read all joined tables
final user = row.readTable(db.users);
final order = row.readTable(db.orders);
```

---

# Summary

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Inner Join** | Matching rows only | Required relationships |
| **Condition** | Join on equality | Foreign key matches |
| **Multiple Tables** | Chain joins | Complex queries |
| **Aggregation** | GROUP BY + JOIN | Reports |

---

# Next Steps

Now you understand inner join, let's dive deeper:

- [Left Join](link) – Left outer join
- [Cross Join](link) – Cross join
- [Self Join](link) – Self join

---

# Did You Know?

- **Inner join is the default join** – Most commonly used

- **Inner join can be optimized** – With proper indexing

- **Inner join excludes nulls** – On both sides

- **Inner join is associative** – Join order doesn't matter

- **Inner join is commutative** – Table order doesn't matter

- **Inner join is often used** – In reporting queries

- **Inner join can be nested** – For complex queries

- **Inner join is performant** – When properly indexed

---

## 🎯 **That's Topic: Inner Join!**

---

**Ready for the next topic: Left Join?** Say "**NEXT**" and let's keep this ELITE knowledge flowing! 🔥