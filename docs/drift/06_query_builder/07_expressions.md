## Expressions

**Building advanced queries with Drift expressions**

---

# What is it?

**Expressions** are the building blocks of Drift queries. They represent SQL expressions in a type-safe way, allowing you to perform calculations, transformations, and complex logic directly in your queries. You can use built-in functions, create custom expressions, and combine them with operators.

> **Think of Expressions like "formulas in a spreadsheet"** – you combine values, operators, and functions to calculate new values or filter data in sophisticated ways.

```dart
// 👇 Basic arithmetic expression
final discountedUsers = await (select(users)
  ..where((u) => 
    u.age > const Variable(18) &
    u.balance > const Variable(100)
  ))
  .get();

// 👇 Expression with mathematical operations
final results = await select(products)
  .map((p) => [
    p.name,
    p.price * p.stock, // Calculate total inventory value
  ])
  .get();

// 👇 String concatenation
final fullNames = await select(users)
  .map((u) => [
    u.firstName + ' ' + u.lastName,
  ])
  .get();
```

> **What's happening here?**
> - **Arithmetic** – `+`, `-`, `*`, `/` on columns
> - **Comparisons** – `>`, `<`, `=`, `!=`, `LIKE`
> - **Logical** – `&` (AND), `|` (OR), `!` (NOT)
> - **Functions** – Custom SQL functions
> - **Type safety** – All expressions are type-checked

---

# Why does it exist?

- **Calculations** – Perform math directly in queries
- **Transformations** – Modify data on the fly
- **Complex Logic** – Build sophisticated WHERE clauses
- **Performance** – Let the database do the heavy lifting
- **Type Safety** – Compile-time validation
- **Flexibility** – Create custom expressions

---

# Arithmetic Expressions

> **Mathematical operations in queries**

## Basic Arithmetic

```dart
// 👇 Calculate total value
final results = await select(products)
  .map((p) => [
    p.name,
    p.price * p.stock, // Total inventory value
  ])
  .get();

// 👇 Price with tax
final results = await select(products)
  .map((p) => [
    p.name,
    p.price * const Constant(1.1), // 10% tax
  ])
  .get();

// 👇 Discount calculation
final results = await select(products)
  .map((p) => [
    p.name,
    p.price * (const Constant(1) - p.discountPercent / const Constant(100)),
  ])
  .get();
```

## Comparison Expressions

```dart
// 👇 Basic comparison
..where((u) => u.age > const Variable(18))

// 👇 Complex comparison
..where((u) => 
  u.age > const Variable(18) &
  u.balance > const Variable(100)
)

// 👇 String comparison
..where((u) => 
  u.firstName + ' ' + u.lastName == const Variable('John Doe')
)
```

---

# String Expressions

> **Manipulating text in queries**

## Concatenation

```dart
// 👇 Concatenate first and last name
final results = await select(users)
  .map((u) => [
    u.firstName + ' ' + u.lastName,
  ])
  .get();

// 👇 With string literals
final results = await select(users)
  .map((u) => [
    u.firstName + ' ' + u.lastName + ' (Age: ' + u.age.castString() + ')',
  ])
  .get();
```

## String Functions

```dart
// 👇 UPPER/LOWER
..where((u) => u.firstName.upper() == const Variable('JOHN'))

// 👇 Substring
..where((u) => u.email.substring(0, 3) == const Variable('jo'))

// 👇 Length
..where((u) => u.firstName.length() > const Variable(3))

// 👇 Trim
..where((u) => u.firstName.trim().contains('John'))
```

---

# Conditional Expressions

> **If-then-else logic in queries**

## CASE Expressions

```dart
// 👇 Using CASE in custom SQL
final results = await customSelect('''
  SELECT 
    id,
    name,
    age,
    CASE 
      WHEN age < 18 THEN 'Minor'
      WHEN age < 65 THEN 'Adult'
      ELSE 'Senior'
    END as age_group
  FROM users
''').get();

// 👇 More complex CASE
final results = await customSelect('''
  SELECT 
    name,
    balance,
    CASE 
      WHEN balance > 1000 THEN 'Gold'
      WHEN balance > 500 THEN 'Silver'
      WHEN balance > 100 THEN 'Bronze'
      ELSE 'Standard'
    END as membership_level
  FROM users
''').get();
```

---

# Date Expressions

> **Working with dates in queries**

## Date Calculations

```dart
// 👇 Current date/time
..where((o) => o.createdAt > currentDateAndTime)

// 👇 Date difference
final results = await customSelect('''
  SELECT 
    name,
    created_at,
    JULIANDAY('now') - JULIANDAY(created_at) as days_old
  FROM users
''').get();

// 👇 Extract parts
final results = await customSelect('''
  SELECT 
    name,
    STRFTIME('%Y', created_at) as year,
    STRFTIME('%m', created_at) as month,
    STRFTIME('%d', created_at) as day
  FROM users
''').get();
```

---

# Real-World Example

> **Complete e-commerce expressions system**

```dart
// lib/database/expression_service.dart
import 'package:drift/drift.dart';

class ExpressionService {
  final AppDatabase db;
  
  ExpressionService(this.db);

  // ==================== PRICE EXPRESSIONS ====================
  
  // 👇 Get products with calculated price
  Future<List<ProductPrice>> getProductPrices() async {
    final results = await customSelect('''
      SELECT 
        id,
        name,
        price,
        price * 1.1 as price_with_tax,
        price * (1 - discount_percent / 100) as discount_price,
        stock * price as inventory_value,
        price * 1.1 * (1 - discount_percent / 100) as final_price
      FROM products
      WHERE is_active = 1
    ''').get();
    
    return results.map((row) {
      return ProductPrice(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        basePrice: row.data['price'] as double,
        priceWithTax: row.data['price_with_tax'] as double,
        discountPrice: row.data['discount_price'] as double,
        inventoryValue: row.data['inventory_value'] as double,
        finalPrice: row.data['final_price'] as double,
      );
    }).toList();
  }
  
  // 👇 Get products with dynamic pricing
  Future<List<DynamicProduct>> getDynamicPricing() async {
    final results = await customSelect('''
      SELECT 
        name,
        price,
        stock,
        CASE 
          WHEN stock > 100 THEN price * 0.8  -- Bulk discount
          WHEN stock > 50 THEN price * 0.9
          ELSE price
        END as dynamic_price,
        CASE 
          WHEN stock > 100 THEN 'Bulk'
          WHEN stock > 50 THEN 'Standard'
          ELSE 'Limited'
        END as pricing_tier
      FROM products
      WHERE is_active = 1
    ''').get();
    
    return results.map((row) {
      return DynamicProduct(
        name: row.data['name'] as String,
        price: row.data['price'] as double,
        stock: row.data['stock'] as int,
        dynamicPrice: row.data['dynamic_price'] as double,
        pricingTier: row.data['pricing_tier'] as String,
      );
    }).toList();
  }

  // ==================== USER AGE EXPRESSIONS ====================
  
  // 👇 Get age calculations
  Future<List<UserAge>> getUserAges() async {
    final results = await customSelect('''
      SELECT 
        name,
        age,
        CASE 
          WHEN age < 18 THEN 'Minor'
          WHEN age < 30 THEN 'Young Adult'
          WHEN age < 45 THEN 'Adult'
          WHEN age < 60 THEN 'Middle Age'
          ELSE 'Senior'
        END as age_category,
        CASE 
          WHEN age < 18 THEN 'Junior'
          WHEN age < 65 THEN 'Working Age'
          ELSE 'Retirement'
        END as age_group,
        65 - age as years_to_retirement
      FROM users
      WHERE age IS NOT NULL
    ''').get();
    
    return results.map((row) {
      return UserAge(
        name: row.data['name'] as String,
        age: row.data['age'] as int,
        ageCategory: row.data['age_category'] as String,
        ageGroup: row.data['age_group'] as String,
        yearsToRetirement: row.data['years_to_retirement'] as int,
      );
    }).toList();
  }

  // ==================== ORDER EXPRESSIONS ====================
  
  // 👇 Order analytics with expressions
  Future<List<OrderAnalytic>> getOrderAnalytics() async {
    final results = await customSelect('''
      SELECT 
        o.id,
        o.order_number,
        o.order_date,
        u.name as user_name,
        u.email as user_email,
        o.total,
        o.total * 0.1 as tax_amount,
        o.total * 0.9 as taxable_amount,
        o.total - (o.total * 0.1) as subtotal,
        CASE 
          WHEN o.total > 100 THEN o.total * 0.95
          WHEN o.total > 50 THEN o.total * 0.98
          ELSE o.total
        END as discounted_total,
        SUM(oi.quantity) as total_items,
        AVG(oi.total / oi.quantity) as avg_item_price
      FROM orders o
      INNER JOIN users u ON o.user_id = u.id
      INNER JOIN order_items oi ON o.id = oi.order_id
      WHERE o.status = 'completed'
      GROUP BY o.id
    ''').get();
    
    return results.map((row) {
      return OrderAnalytic(
        orderId: row.data['id'] as int,
        orderNumber: row.data['order_number'] as String,
        orderDate: row.data['order_date'] as String,
        userName: row.data['user_name'] as String,
        userEmail: row.data['user_email'] as String,
        total: row.data['total'] as double,
        taxAmount: row.data['tax_amount'] as double,
        taxableAmount: row.data['taxable_amount'] as double,
        subtotal: row.data['subtotal'] as double,
        discountedTotal: row.data['discounted_total'] as double,
        totalItems: row.data['total_items'] as int,
        avgItemPrice: row.data['avg_item_price'] as double,
      );
    }).toList();
  }

  // ==================== AGGREGATE EXPRESSIONS ====================
  
  // 👇 Customer lifetime value with expressions
  Future<List<CustomerLifetime>> getCustomerLifetimeValue() async {
    final results = await customSelect('''
      SELECT 
        u.id as user_id,
        u.name as user_name,
        u.email as user_email,
        COUNT(o.id) as order_count,
        SUM(o.total) as total_spent,
        AVG(o.total) as avg_order_value,
        MAX(o.total) as max_order_value,
        MIN(o.total) as min_order_value,
        JULIANDAY('now') - JULIANDAY(MAX(o.order_date)) as days_since_last_order,
        SUM(o.total) / COUNT(o.id) as value_per_order
      FROM users u
      LEFT JOIN orders o ON u.id = o.user_id
      WHERE o.status = 'completed'
      GROUP BY u.id
      HAVING order_count > 0
    ''').get();
    
    return results.map((row) {
      return CustomerLifetime(
        userId: row.data['user_id'] as int,
        userName: row.data['user_name'] as String,
        userEmail: row.data['user_email'] as String,
        orderCount: row.data['order_count'] as int,
        totalSpent: row.data['total_spent'] as double,
        avgOrderValue: row.data['avg_order_value'] as double,
        maxOrderValue: row.data['max_order_value'] as double,
        minOrderValue: row.data['min_order_value'] as double,
        daysSinceLastOrder: row.data['days_since_last_order'] as double,
        valuePerOrder: row.data['value_per_order'] as double,
      );
    }).toList();
  }

  // ==================== INVENTORY EXPRESSIONS ====================
  
  // 👇 Inventory health with expressions
  Future<List<InventoryHealth>> getInventoryHealth() async {
    final results = await customSelect('''
      SELECT 
        p.id,
        p.name,
        p.sku,
        p.stock,
        p.reorder_level,
        p.stock - p.reorder_level as stock_buffer,
        p.stock * p.price as stock_value,
        p.reorder_level * p.price as reorder_value,
        CASE 
          WHEN p.stock <= 0 THEN 'Out of Stock'
          WHEN p.stock < p.reorder_level THEN 'Low Stock'
          WHEN p.stock < p.reorder_level * 2 THEN 'Warning'
          ELSE 'Healthy'
        END as stock_status,
        (p.stock / (p.reorder_level * 2)) * 100 as health_score
      FROM products p
      WHERE p.is_active = 1
    ''').get();
    
    return results.map((row) {
      return InventoryHealth(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        sku: row.data['sku'] as String,
        stock: row.data['stock'] as int,
        reorderLevel: row.data['reorder_level'] as int,
        stockBuffer: row.data['stock_buffer'] as int,
        stockValue: row.data['stock_value'] as double,
        reorderValue: row.data['reorder_value'] as double,
        stockStatus: row.data['stock_status'] as String,
        healthScore: row.data['health_score'] as double,
      );
    }).toList();
  }

  // ==================== STRING EXPRESSIONS ====================
  
  // 👇 User name analysis with string expressions
  Future<List<UserNameAnalysis>> getUserNameAnalysis() async {
    final results = await customSelect('''
      SELECT 
        name,
        email,
        LENGTH(name) as name_length,
        LENGTH(email) as email_length,
        INSTR(email, '@') as email_position,
        SUBSTR(email, INSTR(email, '@') + 1) as email_domain,
        UPPER(name) as name_uppercase,
        LOWER(name) as name_lowercase,
        CASE 
          WHEN email LIKE '%gmail%' THEN 'Gmail'
          WHEN email LIKE '%yahoo%' THEN 'Yahoo'
          WHEN email LIKE '%outlook%' THEN 'Outlook'
          ELSE 'Other'
        END as email_provider
      FROM users
    ''').get();
    
    return results.map((row) {
      return UserNameAnalysis(
        name: row.data['name'] as String,
        email: row.data['email'] as String,
        nameLength: row.data['name_length'] as int,
        emailLength: row.data['email_length'] as int,
        emailPosition: row.data['email_position'] as int,
        emailDomain: row.data['email_domain'] as String,
        nameUppercase: row.data['name_uppercase'] as String,
        nameLowercase: row.data['name_lowercase'] as String,
        emailProvider: row.data['email_provider'] as String,
      );
    }).toList();
  }

  // ==================== PERFORMANCE EXPRESSIONS ====================
  
  // 👇 Sales performance with expressions
  Future<List<SalesPerformance>> getSalesPerformance() async {
    final results = await customSelect('''
      SELECT 
        u.id as user_id,
        u.name as user_name,
        SUM(o.total) as total_sales,
        COUNT(o.id) as total_orders,
        AVG(o.total) as avg_sale,
        (SELECT MAX(total) FROM orders WHERE user_id = u.id) as max_sale,
        (SELECT MIN(total) FROM orders WHERE user_id = u.id) as min_sale,
        SUM(o.total) / COUNT(o.id) as sale_value_per_order,
        COUNT(o.id) / (JULIANDAY('now') - JULIANDAY(MIN(o.order_date))) as orders_per_day
      FROM users u
      LEFT JOIN orders o ON u.id = o.user_id
      WHERE o.status = 'completed'
      GROUP BY u.id
      HAVING total_sales > 0
      ORDER BY total_sales DESC
    ''').get();
    
    return results.map((row) {
      return SalesPerformance(
        userId: row.data['user_id'] as int,
        userName: row.data['user_name'] as String,
        totalSales: row.data['total_sales'] as double,
        totalOrders: row.data['total_orders'] as int,
        avgSale: row.data['avg_sale'] as double,
        maxSale: row.data['max_sale'] as double,
        minSale: row.data['min_sale'] as double,
        saleValuePerOrder: row.data['sale_value_per_order'] as double,
        ordersPerDay: row.data['orders_per_day'] as double,
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class ProductPrice {
  final int id;
  final String name;
  final double basePrice;
  final double priceWithTax;
  final double discountPrice;
  final double inventoryValue;
  final double finalPrice;
  
  ProductPrice({
    required this.id,
    required this.name,
    required this.basePrice,
    required this.priceWithTax,
    required this.discountPrice,
    required this.inventoryValue,
    required this.finalPrice,
  });
}

class DynamicProduct {
  final String name;
  final double price;
  final int stock;
  final double dynamicPrice;
  final String pricingTier;
  
  DynamicProduct({
    required this.name,
    required this.price,
    required this.stock,
    required this.dynamicPrice,
    required this.pricingTier,
  });
}

class UserAge {
  final String name;
  final int age;
  final String ageCategory;
  final String ageGroup;
  final int yearsToRetirement;
  
  UserAge({
    required this.name,
    required this.age,
    required this.ageCategory,
    required this.ageGroup,
    required this.yearsToRetirement,
  });
}

class OrderAnalytic {
  final int orderId;
  final String orderNumber;
  final String orderDate;
  final String userName;
  final String userEmail;
  final double total;
  final double taxAmount;
  final double taxableAmount;
  final double subtotal;
  final double discountedTotal;
  final int totalItems;
  final double avgItemPrice;
  
  OrderAnalytic({
    required this.orderId,
    required this.orderNumber,
    required this.orderDate,
    required this.userName,
    required this.userEmail,
    required this.total,
    required this.taxAmount,
    required this.taxableAmount,
    required this.subtotal,
    required this.discountedTotal,
    required this.totalItems,
    required this.avgItemPrice,
  });
}

class CustomerLifetime {
  final int userId;
  final String userName;
  final String userEmail;
  final int orderCount;
  final double totalSpent;
  final double avgOrderValue;
  final double maxOrderValue;
  final double minOrderValue;
  final double daysSinceLastOrder;
  final double valuePerOrder;
  
  CustomerLifetime({
    required this.userId,
    required this.userName,
    required this.userEmail,
    required this.orderCount,
    required this.totalSpent,
    required this.avgOrderValue,
    required this.maxOrderValue,
    required this.minOrderValue,
    required this.daysSinceLastOrder,
    required this.valuePerOrder,
  });
}

class InventoryHealth {
  final int id;
  final String name;
  final String sku;
  final int stock;
  final int reorderLevel;
  final int stockBuffer;
  final double stockValue;
  final double reorderValue;
  final String stockStatus;
  final double healthScore;
  
  InventoryHealth({
    required this.id,
    required this.name,
    required this.sku,
    required this.stock,
    required this.reorderLevel,
    required this.stockBuffer,
    required this.stockValue,
    required this.reorderValue,
    required this.stockStatus,
    required this.healthScore,
  });
}

class UserNameAnalysis {
  final String name;
  final String email;
  final int nameLength;
  final int emailLength;
  final int emailPosition;
  final String emailDomain;
  final String nameUppercase;
  final String nameLowercase;
  final String emailProvider;
  
  UserNameAnalysis({
    required this.name,
    required this.email,
    required this.nameLength,
    required this.emailLength,
    required this.emailPosition,
    required this.emailDomain,
    required this.nameUppercase,
    required this.nameLowercase,
    required this.emailProvider,
  });
}

class SalesPerformance {
  final int userId;
  final String userName;
  final double totalSales;
  final int totalOrders;
  final double avgSale;
  final double maxSale;
  final double minSale;
  final double saleValuePerOrder;
  final double ordersPerDay;
  
  SalesPerformance({
    required this.userId,
    required this.userName,
    required this.totalSales,
    required this.totalOrders,
    required this.avgSale,
    required this.maxSale,
    required this.minSale,
    required this.saleValuePerOrder,
    required this.ordersPerDay,
  });
}
```

---

# Best Practices

- **Keep expressions simple** – Complex expressions are hard to maintain
- **Test expressions** – Verify they return correct results
- **Use indexes** – For columns used in expressions
- **Document complex logic** – Explain what expressions do
- **Use type safety** – Let Drift validate your expressions
- **Consider performance** – Complex expressions can be slow
- **Use custom SQL for very complex logic** – It's more flexible

---

# Common Mistakes

## Mistake 1: Type mismatches in expressions

Wrong:
```dart
// 🚫 Can't compare string to int
..where((u) => u.name > const Variable(18))
```

Correct:
```dart
// ✅ Compare compatible types
..where((u) => u.age > const Variable(18))
```

## Mistake 2: Forgetting `const Variable()`

Wrong:
```dart
// 🚫 Error: int can't be used in Expression
..where((u) => u.age > 18)
```

Correct:
```dart
// ✅ Use Variable wrapper
..where((u) => u.age > const Variable(18))
```

## Mistake 3: Using unsupported operators

Wrong:
```dart
// 🚫 Not all Dart operators work
..where((u) => u.age % 2 == 0)
```

Correct:
```dart
// ✅ Use supported operations or custom SQL
final results = await customSelect('''
  SELECT * FROM users WHERE age % 2 = 0
''').get();
```

---

# Summary

| Expression Type | Purpose | Example |
|-----------------|---------|---------|
| **Arithmetic** | Math operations | `price * stock` |
| **String** | Text manipulation | `firstName + ' ' + lastName` |
| **Conditional** | Logic | `CASE WHEN ... END` |
| **Date** | Date operations | `JULIANDAY('now')` |
| **Aggregate** | Calculations | `SUM(total)` |

---

# Next Steps

Now you understand expressions, let's dive deeper:

- [Custom SQL Expressions](link) – Raw SQL expressions
- [Dynamic Queries](link) – Building queries dynamically
- [Joins](link) – Table joins

---

# Did You Know?

- **Expressions are type-safe** – Compile-time validation

- **Expressions can be complex** – Nested calculations

- **Expressions are compiled to SQL** – By Drift

- **Expressions can use database functions** – `COUNT()`, `SUM()`, `AVG()`

- **Expressions can be used in WHERE, ORDER, SELECT** – Anywhere

- **Expressions are lazy** – Only evaluated when query runs

- **Expressions can be composed** – Chain operations

- **Expressions are powerful** – Let the database do the work

