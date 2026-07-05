## Aliases

**Creating column and table aliases in Drift queries**

---

# What is it?

**Aliases** allow you to rename tables or columns in your SQL queries. This is useful for readability, avoiding name conflicts in joins, and creating virtual columns in expressions. In Drift, you can create aliases for tables and use column aliases in `map()` or custom SQL queries.

> **Think of Aliases like "nicknames"** – just as you might call someone "Bob" instead of "Robert" for convenience, you can give your tables and columns shorter or clearer names in your queries.

```dart
// 👇 Table alias for self-join
final employees = alias(users, 'employees');
final managers = alias(users, 'managers');

final results = await (select(employees)
  ..join([
    innerJoin(
      managers,
      employees.managerId.equals(managers.id),
    )
  ]))
  .get();

// 👇 Column alias in custom SQL
final results = await customSelect('''
  SELECT name as user_name, age as user_age 
  FROM users
''').get();
```

> **What's happening here?**
> - **Table aliases** – Rename tables in queries
> - **Column aliases** – Rename columns in results
> - **Disambiguation** – Resolve name conflicts in joins
> - **Readability** – Make complex queries clearer

---

# Why does it exist?

- **Self-Joins** – Join a table to itself
- **Readability** – Make complex queries clearer
- **Conflict Resolution** – Avoid ambiguous column names
- **Custom Queries** – Rename columns in results
- **Expressions** – Name calculated columns
- **Code Organization** – Organize complex queries

---

# Table Aliases

> **Renaming tables in queries**

## Basic Table Alias

```dart
// 👇 Simple table alias
final usersAlias = alias(users, 'u');

// Use the alias in a query
final results = await (select(usersAlias)
  ..where((u) => u.isActive.equals(true)))
  .get();

// Generated SQL:
// SELECT * FROM users AS u WHERE u.is_active = 1
```

## Table Alias for Self-Join

```dart
// 👇 Self-join for employee-manager relationship
final employees = alias(users, 'employees');
final managers = alias(users, 'managers');

final results = await (select(employees)
  ..join([
    innerJoin(
      managers,
      employees.managerId.equals(managers.id),
    )
  ])
  ..where((e) => e.isActive.equals(true)))
  .get();

// Generated SQL:
// SELECT employees.*, managers.* 
// FROM users AS employees 
// INNER JOIN users AS managers 
// ON employees.manager_id = managers.id 
// WHERE employees.is_active = 1
```

## Table Alias with Where Clauses

```dart
// 👇 Using aliases in WHERE clauses
final products = alias(db.products, 'p');
final categories = alias(db.categories, 'c');

final results = await (select(products)
  ..join([
    innerJoin(
      categories,
      products.categoryId.equals(categories.id),
    )
  ])
  ..where((p) => p.isActive.equals(true))
  ..where((c) => c.name.equals('Electronics')))
  .get();
```

---

# Column Aliases

> **Renaming columns in results**

## Column Alias in Custom SQL

```dart
// 👇 Column aliases in custom select
final results = await customSelect('''
  SELECT 
    name as user_name,
    email as user_email,
    age as user_age
  FROM users
  WHERE is_active = 1
''').get();

// Access by alias
for (final row in results) {
  print(row.data['user_name']);
  print(row.data['user_email']);
}
```

## Column Alias for Expressions

```dart
// 👇 Alias for calculated columns
final results = await customSelect('''
  SELECT 
    name,
    age,
    (age * 2) as double_age,
    (age + 10) as age_in_ten_years
  FROM users
''').get();

for (final row in results) {
  print('Name: ${row.data['name']}, Double Age: ${row.data['double_age']}');
}
```

## Column Alias with Aggregations

```dart
// 👇 Alias for aggregate functions
final results = await customSelect('''
  SELECT 
    status,
    COUNT(*) as total_users,
    AVG(age) as average_age,
    MIN(age) as youngest_age,
    MAX(age) as oldest_age
  FROM users
  GROUP BY status
''').get();

for (final row in results) {
  print('Status: ${row.data['status']}');
  print('Total: ${row.data['total_users']}');
  print('Average Age: ${row.data['average_age']}');
}
```

---

# Aliases with Joins

> **Resolving ambiguous column names**

## Joins with Aliases

```dart
// 👇 Joins with table aliases
final users = alias(db.users, 'u');
final orders = alias(db.orders, 'o');
final products = alias(db.products, 'p');

final results = await customSelect('''
  SELECT 
    u.name as user_name,
    o.order_number as order_number,
    p.name as product_name,
    oi.quantity as quantity,
    oi.total as total_price
  FROM users u
  INNER JOIN orders o ON u.id = o.user_id
  INNER JOIN order_items oi ON o.id = oi.order_id
  INNER JOIN products p ON oi.product_id = p.id
  WHERE o.status = 'completed'
  ORDER BY o.order_date DESC
''').get();

for (final row in results) {
  print('User: ${row.data['user_name']}');
  print('Order: ${row.data['order_number']}');
  print('Product: ${row.data['product_name']}');
  print('Quantity: ${row.data['quantity']}');
  print('Total: \$${row.data['total_price']}');
}
```

---

# Aliases in Group By

> **Using aliases in GROUP BY clauses**

```dart
// 👇 Aliases in GROUP BY
final results = await customSelect('''
  SELECT 
    EXTRACT(YEAR FROM created_at) as year,
    EXTRACT(MONTH FROM created_at) as month,
    COUNT(*) as total_orders,
    SUM(total) as total_revenue
  FROM orders
  GROUP BY year, month
  ORDER BY year DESC, month DESC
''').get();

for (final row in results) {
  print('Year: ${row.data['year']}, Month: ${row.data['month']}');
  print('Orders: ${row.data['total_orders']}');
  print('Revenue: \$${row.data['total_revenue']}');
}
```

---

# Real-World Example

> **Complete e-commerce aliases system**

```dart
// lib/database/alias_service.dart
import 'package:drift/drift.dart';

class AliasService {
  final AppDatabase db;
  
  AliasService(this.db);

  // ==================== USER-ORDER JOINS ====================
  
  // 👇 Get users with their order stats using aliases
  Future<List<UserOrderStats>> getUserOrderStats() async {
    final u = alias(db.users, 'u');
    final o = alias(db.orders, 'o');
    
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
      FROM ${u.actualTableName} u
      LEFT JOIN ${o.actualTableName} o ON u.id = o.user_id
      GROUP BY u.id, u.name, u.email
      HAVING total_orders > 0
      ORDER BY total_spent DESC
    ''').get();
    
    return results.map((row) {
      return UserOrderStats(
        userId: row.data['user_id'] as int,
        userName: row.data['user_name'] as String,
        userEmail: row.data['user_email'] as String,
        totalOrders: row.data['total_orders'] as int,
        totalSpent: row.data['total_spent'] as double? ?? 0.0,
        avgOrderValue: row.data['avg_order_value'] as double? ?? 0.0,
        maxOrderValue: row.data['max_order_value'] as double? ?? 0.0,
        minOrderValue: row.data['min_order_value'] as double? ?? 0.0,
        completedOrders: row.data['completed_orders'] as int,
      );
    }).toList();
  }

  // ==================== PRODUCT CATEGORY REPORT ====================
  
  // 👇 Get product categories report with aliases
  Future<List<CategoryReport>> getCategoryReport() async {
    final p = alias(db.products, 'p');
    final c = alias(db.categories, 'c');
    
    final results = await db.customSelect('''
      SELECT 
        c.id as category_id,
        c.name as category_name,
        COUNT(p.id) as product_count,
        SUM(p.stock) as total_stock,
        AVG(p.price) as avg_price,
        MIN(p.price) as min_price,
        MAX(p.price) as max_price,
        SUM(CASE WHEN p.is_active = 1 THEN 1 ELSE 0 END) as active_products,
        SUM(CASE WHEN p.stock > 0 AND p.is_active = 1 THEN 1 ELSE 0 END) as available_products
      FROM ${c.actualTableName} c
      LEFT JOIN ${p.actualTableName} p ON c.id = p.category_id
      GROUP BY c.id, c.name
      ORDER BY product_count DESC
    ''').get();
    
    return results.map((row) {
      return CategoryReport(
        categoryId: row.data['category_id'] as int,
        categoryName: row.data['category_name'] as String,
        productCount: row.data['product_count'] as int,
        totalStock: row.data['total_stock'] as int,
        avgPrice: row.data['avg_price'] as double? ?? 0.0,
        minPrice: row.data['min_price'] as double? ?? 0.0,
        maxPrice: row.data['max_price'] as double? ?? 0.0,
        activeProducts: row.data['active_products'] as int,
        availableProducts: row.data['available_products'] as int,
      );
    }).toList();
  }

  // ==================== ORDER DETAILS WITH ALIASES ====================
  
  // 👇 Get detailed order information with aliases
  Future<List<OrderDetail>> getOrderDetails({
    required int orderId,
  }) async {
    final o = alias(db.orders, 'o');
    final u = alias(db.users, 'u');
    final oi = alias(db.orderItems, 'oi');
    final p = alias(db.products, 'p');
    
    final results = await db.customSelect('''
      SELECT 
        o.id as order_id,
        o.order_number as order_number,
        o.order_date as order_date,
        o.status as order_status,
        o.total as order_total,
        o.is_paid as is_paid,
        o.is_shipped as is_shipped,
        o.is_delivered as is_delivered,
        u.id as user_id,
        u.name as user_name,
        u.email as user_email,
        oi.id as item_id,
        oi.quantity as quantity,
        oi.unit_price as unit_price,
        oi.discount as discount,
        oi.total as item_total,
        p.id as product_id,
        p.name as product_name,
        p.sku as product_sku
      FROM ${o.actualTableName} o
      INNER JOIN ${u.actualTableName} u ON o.user_id = u.id
      INNER JOIN ${oi.actualTableName} oi ON o.id = oi.order_id
      INNER JOIN ${p.actualTableName} p ON oi.product_id = p.id
      WHERE o.id = ?
    ''', variables: [
      Variable.withInt(orderId),
    ]).get();
    
    final orderDetails = <OrderDetail>[];
    for (final row in results) {
      orderDetails.add(OrderDetail(
        orderId: row.data['order_id'] as int,
        orderNumber: row.data['order_number'] as String,
        orderDate: DateTime.parse(row.data['order_date'] as String),
        orderStatus: row.data['order_status'] as String,
        orderTotal: row.data['order_total'] as double,
        isPaid: (row.data['is_paid'] as int) == 1,
        isShipped: (row.data['is_shipped'] as int) == 1,
        isDelivered: (row.data['is_delivered'] as int) == 1,
        userId: row.data['user_id'] as int,
        userName: row.data['user_name'] as String,
        userEmail: row.data['user_email'] as String,
        itemId: row.data['item_id'] as int,
        quantity: row.data['quantity'] as int,
        unitPrice: row.data['unit_price'] as double,
        discount: row.data['discount'] as double,
        itemTotal: row.data['item_total'] as double,
        productId: row.data['product_id'] as int,
        productName: row.data['product_name'] as String,
        productSku: row.data['product_sku'] as String,
      ));
    }
    return orderDetails;
  }

  // ==================== EMPLOYEE HIERARCHY ====================
  
  // 👇 Get employee hierarchy with self-join aliases
  Future<List<EmployeeHierarchy>> getEmployeeHierarchy() async {
    final e = alias(db.users, 'e');
    final m = alias(db.users, 'm');
    
    final results = await db.customSelect('''
      SELECT 
        e.id as employee_id,
        e.name as employee_name,
        e.email as employee_email,
        m.id as manager_id,
        m.name as manager_name,
        m.email as manager_email,
        e.department as department,
        e.hire_date as hire_date
      FROM ${e.actualTableName} e
      LEFT JOIN ${m.actualTableName} m ON e.manager_id = m.id
      WHERE e.is_active = 1
      ORDER BY e.department, e.name
    ''').get();
    
    return results.map((row) {
      return EmployeeHierarchy(
        employeeId: row.data['employee_id'] as int,
        employeeName: row.data['employee_name'] as String,
        employeeEmail: row.data['employee_email'] as String,
        managerId: row.data['manager_id'] as int?,
        managerName: row.data['manager_name'] as String?,
        managerEmail: row.data['manager_email'] as String?,
        department: row.data['department'] as String,
        hireDate: DateTime.parse(row.data['hire_date'] as String),
      );
    }).toList();
  }

  // ==================== SALES REPORT ====================
  
  // 👇 Get monthly sales report with aliases
  Future<List<MonthlySales>> getMonthlySalesReport({
    required int year,
  }) async {
    final o = alias(db.orders, 'o');
    
    final results = await db.customSelect('''
      SELECT 
        STRFTIME('%m', order_date) as month,
        COUNT(*) as total_orders,
        SUM(total) as total_revenue,
        AVG(total) as avg_order_value,
        SUM(CASE WHEN is_paid = 1 THEN total ELSE 0 END) as paid_revenue,
        COUNT(CASE WHEN is_paid = 1 THEN 1 END) as paid_orders,
        SUM(CASE WHEN status = 'cancelled' THEN total ELSE 0 END) as cancelled_revenue
      FROM ${o.actualTableName} o
      WHERE STRFTIME('%Y', order_date) = ?
      GROUP BY STRFTIME('%m', order_date)
      ORDER BY month
    ''', variables: [
      Variable.withString(year.toString()),
    ]).get();
    
    return results.map((row) {
      return MonthlySales(
        month: row.data['month'] as String,
        totalOrders: row.data['total_orders'] as int,
        totalRevenue: row.data['total_revenue'] as double? ?? 0.0,
        avgOrderValue: row.data['avg_order_value'] as double? ?? 0.0,
        paidRevenue: row.data['paid_revenue'] as double? ?? 0.0,
        paidOrders: row.data['paid_orders'] as int,
        cancelledRevenue: row.data['cancelled_revenue'] as double? ?? 0.0,
      );
    }).toList();
  }

  // ==================== TOP PRODUCTS ====================
  
  // 👇 Get top selling products with aliases
  Future<List<TopProduct>> getTopSellingProducts({
    int limit = 10,
  }) async {
    final oi = alias(db.orderItems, 'oi');
    final p = alias(db.products, 'p');
    
    final results = await db.customSelect('''
      SELECT 
        p.id as product_id,
        p.name as product_name,
        p.sku as product_sku,
        p.price as product_price,
        COUNT(oi.id) as order_count,
        SUM(oi.quantity) as total_sold,
        SUM(oi.total) as total_revenue,
        AVG(oi.total) as avg_order_value
      FROM ${p.actualTableName} p
      INNER JOIN ${oi.actualTableName} oi ON p.id = oi.product_id
      GROUP BY p.id, p.name, p.sku, p.price
      ORDER BY total_revenue DESC
      LIMIT ?
    ''', variables: [
      Variable.withInt(limit),
    ]).get();
    
    return results.map((row) {
      return TopProduct(
        productId: row.data['product_id'] as int,
        productName: row.data['product_name'] as String,
        productSku: row.data['product_sku'] as String,
        productPrice: row.data['product_price'] as double,
        orderCount: row.data['order_count'] as int,
        totalSold: row.data['total_sold'] as int,
        totalRevenue: row.data['total_revenue'] as double? ?? 0.0,
        avgOrderValue: row.data['avg_order_value'] as double? ?? 0.0,
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class UserOrderStats {
  final int userId;
  final String userName;
  final String userEmail;
  final int totalOrders;
  final double totalSpent;
  final double avgOrderValue;
  final double maxOrderValue;
  final double minOrderValue;
  final int completedOrders;
  
  UserOrderStats({
    required this.userId,
    required this.userName,
    required this.userEmail,
    required this.totalOrders,
    required this.totalSpent,
    required this.avgOrderValue,
    required this.maxOrderValue,
    required this.minOrderValue,
    required this.completedOrders,
  });
}

class CategoryReport {
  final int categoryId;
  final String categoryName;
  final int productCount;
  final int totalStock;
  final double avgPrice;
  final double minPrice;
  final double maxPrice;
  final int activeProducts;
  final int availableProducts;
  
  CategoryReport({
    required this.categoryId,
    required this.categoryName,
    required this.productCount,
    required this.totalStock,
    required this.avgPrice,
    required this.minPrice,
    required this.maxPrice,
    required this.activeProducts,
    required this.availableProducts,
  });
}

class OrderDetail {
  final int orderId;
  final String orderNumber;
  final DateTime orderDate;
  final String orderStatus;
  final double orderTotal;
  final bool isPaid;
  final bool isShipped;
  final bool isDelivered;
  final int userId;
  final String userName;
  final String userEmail;
  final int itemId;
  final int quantity;
  final double unitPrice;
  final double discount;
  final double itemTotal;
  final int productId;
  final String productName;
  final String productSku;
  
  OrderDetail({
    required this.orderId,
    required this.orderNumber,
    required this.orderDate,
    required this.orderStatus,
    required this.orderTotal,
    required this.isPaid,
    required this.isShipped,
    required this.isDelivered,
    required this.userId,
    required this.userName,
    required this.userEmail,
    required this.itemId,
    required this.quantity,
    required this.unitPrice,
    required this.discount,
    required this.itemTotal,
    required this.productId,
    required this.productName,
    required this.productSku,
  });
}

class EmployeeHierarchy {
  final int employeeId;
  final String employeeName;
  final String employeeEmail;
  final int? managerId;
  final String? managerName;
  final String? managerEmail;
  final String department;
  final DateTime hireDate;
  
  EmployeeHierarchy({
    required this.employeeId,
    required this.employeeName,
    required this.employeeEmail,
    this.managerId,
    this.managerName,
    this.managerEmail,
    required this.department,
    required this.hireDate,
  });
}

class MonthlySales {
  final String month;
  final int totalOrders;
  final double totalRevenue;
  final double avgOrderValue;
  final double paidRevenue;
  final int paidOrders;
  final double cancelledRevenue;
  
  MonthlySales({
    required this.month,
    required this.totalOrders,
    required this.totalRevenue,
    required this.avgOrderValue,
    required this.paidRevenue,
    required this.paidOrders,
    required this.cancelledRevenue,
  });
}

class TopProduct {
  final int productId;
  final String productName;
  final String productSku;
  final double productPrice;
  final int orderCount;
  final int totalSold;
  final double totalRevenue;
  final double avgOrderValue;
  
  TopProduct({
    required this.productId,
    required this.productName,
    required this.productSku,
    required this.productPrice,
    required this.orderCount,
    required this.totalSold,
    required this.totalRevenue,
    required this.avgOrderValue,
  });
}
```

---

# Best Practices

- **Use meaningful aliases** – `u` for users, `o` for orders
- **Be consistent** – Use the same alias conventions
- **Document complex queries** – Explain what aliases mean
- **Use aliases for self-joins** – Required for clarity
- **Use aliases in reports** – Make results clearer
- **Use aliases for aggregations** – Name calculated columns
- **Use table aliases in custom SQL** – Always for clarity

---

# Common Mistakes

## Mistake 1: Ambiguous column names without aliases

Wrong:
```dart
// 🚫 Ambiguous column name 'id'
final results = await customSelect('''
  SELECT id, name FROM users u
  INNER JOIN orders o ON u.id = o.user_id
''').get();
```

Correct:
```dart
// ✅ Use aliases to disambiguate
final results = await customSelect('''
  SELECT u.id as user_id, u.name as user_name, o.id as order_id
  FROM users u
  INNER JOIN orders o ON u.id = o.user_id
''').get();
```

## Mistake 2: Not using table aliases in joins

Wrong:
```dart
// 🚫 No aliases in self-join
final results = await customSelect('''
  SELECT * FROM users 
  INNER JOIN users ON users.manager_id = users.id
''').get();
```

Correct:
```dart
// ✅ Use aliases for self-joins
final results = await customSelect('''
  SELECT e.*, m.* 
  FROM users e
  INNER JOIN users m ON e.manager_id = m.id
''').get();
```

---

# Summary

| Alias Type | Syntax | Use Case |
|------------|--------|----------|
| **Table Alias** | `alias(table, 'alias')` | Self-joins, readability |
| **Column Alias** | `column as alias` | Custom SQL |
| **Aggregation Alias** | `COUNT(*) as total` | Reports |

---

# Next Steps

Now you understand aliases, let's dive deeper:

- [Expressions](link) – Custom expressions
- [Custom SQL Expressions](link) – Raw SQL expressions
- [Dynamic Queries](link) – Building queries dynamically

---

# Did You Know?

- **Aliases are temporary** – Only exist for the query

- **Table aliases are required for self-joins** – To avoid ambiguity

- **Column aliases can be used in ORDER BY** – In some SQLite versions

- **Aliases can be used in GROUP BY** – For grouping by calculated columns

- **Aliases are optional** – But recommended for clarity

- **Aliases can be any valid identifier** – Letters, numbers, underscores

- **Aliases are case-insensitive** – In most SQL implementations

- **Aliases are not stored in the database** – They're query-time only

---

