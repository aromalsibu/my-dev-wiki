## SQL Variables

**Using parameters and variables in Drift custom SQL queries**

---

# What is it?

**SQL Variables** (also known as parameters or bind variables) are placeholders in your SQL queries that are replaced with actual values when the query is executed. Drift provides type-safe variable support through the `Variable` class, which prevents SQL injection and handles type conversion automatically.

> **Think of SQL Variables like "fill-in-the-blank forms"** – you create a template with blanks (variables), and then fill in the blanks with actual values when you're ready to execute.

```dart
// 👇 Using variables in custom SQL
final minAge = 18;
final maxAge = 65;
final status = 'active';

final results = await customSelect(
  '''
  SELECT * FROM users 
  WHERE age BETWEEN ? AND ?
  AND status = ?
  ''',
  variables: [
    Variable.withInt(minAge),
    Variable.withInt(maxAge),
    Variable.withString(status),
  ],
).get();

// Generated SQL:
// SELECT * FROM users WHERE age BETWEEN 18 AND 65 AND status = 'active'
```

> **What's happening here?**
> - **`?`** – Placeholder in the SQL query
> - **`Variable`** – Type-safe wrapper for values
> - **Positional binding** – Variables match by position
> - **Type safety** – Compile-time type checking
> - **SQL injection prevention** – Values are properly escaped

---

# Why does it exist?

- **SQL Injection Prevention** – Automatically escape values
- **Type Safety** – Compile-time type checking
- **Reusability** – Use same query with different values
- **Performance** – Query plans are reused
- **Dynamic Queries** – Build queries with variable values
- **Security** – Protect against attacks

---

# Variable Types

> **All Variable types available in Drift**

## Basic Variable Types

```dart
// 👇 String variable
Variable.withString('John Doe')

// 👇 Integer variable
Variable.withInt(25)

// 👇 Double variable
Variable.withDouble(99.99)

// 👇 Boolean variable
Variable.withBool(true)

// 👇 DateTime variable
Variable.withDateTime(DateTime.now())

// 👇 Null variable
Variable.withNull()
```

## Using Variables in Queries

```dart
// 👇 Multiple variable types
final name = 'John';
final age = 25;
final isActive = true;
final createdAt = DateTime.now();

final results = await customSelect(
  '''
  SELECT * FROM users 
  WHERE name = ?
  AND age > ?
  AND is_active = ?
  AND created_at > ?
  ''',
  variables: [
    Variable.withString(name),
    Variable.withInt(age),
    Variable.withBool(isActive),
    Variable.withDateTime(createdAt),
  ],
).get();
```

---

# Variable Operators

> **Using variables with different SQL operators**

## Comparison Operators

```dart
// 👇 Equals
final email = 'john@example.com';
await customSelect(
  'SELECT * FROM users WHERE email = ?',
  variables: [Variable.withString(email)],
).get();

// 👇 Greater than
final minAge = 18;
await customSelect(
  'SELECT * FROM users WHERE age > ?',
  variables: [Variable.withInt(minAge)],
).get();

// 👇 Less than
final maxAge = 65;
await customSelect(
  'SELECT * FROM users WHERE age < ?',
  variables: [Variable.withInt(maxAge)],
).get();

// 👇 LIKE pattern
final searchTerm = '%John%';
await customSelect(
  'SELECT * FROM users WHERE name LIKE ?',
  variables: [Variable.withString(searchTerm)],
).get();
```

## IN Operator with Variables

```dart
// 👇 IN with multiple values
final userIds = [1, 2, 3, 4, 5];
final placeholders = userIds.map((_) => '?').join(',');

await customSelect(
  'SELECT * FROM users WHERE id IN ($placeholders)',
  variables: userIds.map((id) => Variable.withInt(id)).toList(),
).get();

// 👇 IN with subquery (using variable)
final minOrders = 5;
await customSelect(
  '''
  SELECT * FROM users 
  WHERE id IN (
    SELECT user_id FROM orders 
    GROUP BY user_id 
    HAVING COUNT(*) > ?
  )
  ''',
  variables: [Variable.withInt(minOrders)],
).get();
```

---

# Variable in Different Query Types

> **Using variables in all custom query types**

## Custom Select with Variables

```dart
// 👇 Select with multiple variables
final status = 'active';
final minAge = 18;
final maxAge = 65;

final results = await customSelect(
  '''
  SELECT * FROM users 
  WHERE status = ? 
  AND age BETWEEN ? AND ?
  ''',
  variables: [
    Variable.withString(status),
    Variable.withInt(minAge),
    Variable.withInt(maxAge),
  ],
).get();
```

## Custom Update with Variables

```dart
// 👇 Update with variables
final newStatus = 'inactive';
final cutoffDate = DateTime.now().subtract(Duration(days: 30));

await customUpdate(
  '''
  UPDATE users 
  SET status = ?, updated_at = datetime('now')
  WHERE last_login < ? AND status = 'active'
  ''',
  variables: [
    Variable.withString(newStatus),
    Variable.withDateTime(cutoffDate),
  ],
).go();
```

## Custom Insert with Variables

```dart
// 👇 Insert with variables
final name = 'John Doe';
final email = 'john@example.com';
final age = 25;

await customInsert(
  '''
  INSERT INTO users (name, email, age, created_at)
  VALUES (?, ?, ?, datetime('now'))
  ''',
  variables: [
    Variable.withString(name),
    Variable.withString(email),
    Variable.withInt(age),
  ],
).go();
```

## Custom Delete with Variables

```dart
// 👇 Delete with variables
final minAge = 65;
final status = 'inactive';

await customDelete(
  '''
  DELETE FROM users 
  WHERE age > ? AND status = ?
  ''',
  variables: [
    Variable.withInt(minAge),
    Variable.withString(status),
  ],
).go();
```

---

# Real-World Example

> **Complete e-commerce variable system**

```dart
// lib/database/variable_service.dart
import 'package:drift/drift.dart';

class VariableService {
  final AppDatabase db;
  
  VariableService(this.db);

  // ==================== USER QUERIES ====================
  
  // 👇 Dynamic user search with variables
  Future<List<User>> searchUsers({
    String? name,
    String? email,
    int? minAge,
    int? maxAge,
    bool? isActive,
  }) async {
    final conditions = <String>[];
    final variables = <Variable>[];
    
    if (name != null && name.isNotEmpty) {
      conditions.add('name LIKE ?');
      variables.add(Variable.withString('%$name%'));
    }
    
    if (email != null && email.isNotEmpty) {
      conditions.add('email LIKE ?');
      variables.add(Variable.withString('%$email%'));
    }
    
    if (minAge != null) {
      conditions.add('age >= ?');
      variables.add(Variable.withInt(minAge));
    }
    
    if (maxAge != null) {
      conditions.add('age <= ?');
      variables.add(Variable.withInt(maxAge));
    }
    
    if (isActive != null) {
      conditions.add('is_active = ?');
      variables.add(Variable.withBool(isActive));
    }
    
    final whereClause = conditions.isEmpty 
        ? '' 
        : 'WHERE ${conditions.join(' AND ')}';
    
    final results = await db.customSelect(
      '''
      SELECT * FROM users 
      $whereClause
      ORDER BY name
      ''',
      variables: variables,
    ).get();
    
    return results.map((row) {
      return User(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        email: row.data['email'] as String,
        age: row.data['age'] as int?,
        isActive: (row.data['is_active'] as int) == 1,
        // ... other fields
      );
    }).toList();
  }

  // ==================== ORDER QUERIES ====================
  
  // 👇 Filter orders with variables
  Future<List<Order>> filterOrders({
    int? userId,
    String? status,
    double? minTotal,
    double? maxTotal,
    DateTime? fromDate,
    DateTime? toDate,
  }) async {
    final conditions = <String>[];
    final variables = <Variable>[];
    
    if (userId != null) {
      conditions.add('user_id = ?');
      variables.add(Variable.withInt(userId));
    }
    
    if (status != null && status.isNotEmpty) {
      conditions.add('status = ?');
      variables.add(Variable.withString(status));
    }
    
    if (minTotal != null) {
      conditions.add('total >= ?');
      variables.add(Variable.withDouble(minTotal));
    }
    
    if (maxTotal != null) {
      conditions.add('total <= ?');
      variables.add(Variable.withDouble(maxTotal));
    }
    
    if (fromDate != null) {
      conditions.add('order_date >= ?');
      variables.add(Variable.withDateTime(fromDate));
    }
    
    if (toDate != null) {
      conditions.add('order_date <= ?');
      variables.add(Variable.withDateTime(toDate));
    }
    
    final whereClause = conditions.isEmpty 
        ? '' 
        : 'WHERE ${conditions.join(' AND ')}';
    
    final results = await db.customSelect(
      '''
      SELECT * FROM orders 
      $whereClause
      ORDER BY order_date DESC
      ''',
      variables: variables,
    ).get();
    
    return results.map((row) {
      return Order(
        id: row.data['id'] as int,
        orderNumber: row.data['order_number'] as String,
        userId: row.data['user_id'] as int,
        total: row.data['total'] as double,
        status: row.data['status'] as String,
        orderDate: DateTime.fromMillisecondsSinceEpoch(row.data['order_date'] as int),
        // ... other fields
      );
    }).toList();
  }

  // ==================== PRODUCT QUERIES ====================
  
  // 👇 Advanced product search with variables
  Future<List<Product>> searchProducts({
    String? search,
    String? category,
    double? minPrice,
    double? maxPrice,
    int? minStock,
    int? maxStock,
    bool? isActive,
  }) async {
    final conditions = <String>[];
    final variables = <Variable>[];
    
    if (search != null && search.isNotEmpty) {
      conditions.add('(name LIKE ? OR sku LIKE ? OR description LIKE ?)');
      final searchTerm = '%$search%';
      variables.addAll([
        Variable.withString(searchTerm),
        Variable.withString(searchTerm),
        Variable.withString(searchTerm),
      ]);
    }
    
    if (category != null && category.isNotEmpty) {
      conditions.add('category = ?');
      variables.add(Variable.withString(category));
    }
    
    if (minPrice != null) {
      conditions.add('price >= ?');
      variables.add(Variable.withDouble(minPrice));
    }
    
    if (maxPrice != null) {
      conditions.add('price <= ?');
      variables.add(Variable.withDouble(maxPrice));
    }
    
    if (minStock != null) {
      conditions.add('stock >= ?');
      variables.add(Variable.withInt(minStock));
    }
    
    if (maxStock != null) {
      conditions.add('stock <= ?');
      variables.add(Variable.withInt(maxStock));
    }
    
    if (isActive != null) {
      conditions.add('is_active = ?');
      variables.add(Variable.withBool(isActive));
    }
    
    final whereClause = conditions.isEmpty 
        ? '' 
        : 'WHERE ${conditions.join(' AND ')}';
    
    final results = await db.customSelect(
      '''
      SELECT * FROM products 
      $whereClause
      ORDER BY name
      ''',
      variables: variables,
    ).get();
    
    return results.map((row) {
      return Product(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        sku: row.data['sku'] as String,
        price: row.data['price'] as double,
        stock: row.data['stock'] as int,
        category: row.data['category'] as String,
        isActive: (row.data['is_active'] as int) == 1,
        // ... other fields
      );
    }).toList();
  }

  // ==================== DYNAMIC VARIABLE BUILDING ====================
  
  // 👇 Build variables dynamically
  Future<List<Map<String, dynamic>>> dynamicQuery({
    required String table,
    required Map<String, dynamic> filters,
    String? orderBy,
    int? limit,
  }) async {
    final conditions = <String>[];
    final variables = <Variable>[];
    
    for (final entry in filters.entries) {
      conditions.add('$entry.key = ?');
      
      // Handle different types
      final value = entry.value;
      if (value is String) {
        variables.add(Variable.withString(value));
      } else if (value is int) {
        variables.add(Variable.withInt(value));
      } else if (value is double) {
        variables.add(Variable.withDouble(value));
      } else if (value is bool) {
        variables.add(Variable.withBool(value));
      } else if (value is DateTime) {
        variables.add(Variable.withDateTime(value));
      } else {
        variables.add(Variable.withNull());
      }
    }
    
    final whereClause = conditions.isEmpty 
        ? '' 
        : 'WHERE ${conditions.join(' AND ')}';
    
    final orderClause = orderBy != null 
        ? 'ORDER BY $orderBy' 
        : '';
    
    final limitClause = limit != null 
        ? 'LIMIT $limit' 
        : '';
    
    return await db.customSelect(
      '''
      SELECT * FROM $table 
      $whereClause 
      $orderClause 
      $limitClause
      ''',
      variables: variables,
    ).get();
  }

  // ==================== BATCH OPERATIONS ====================
  
  // 👇 Batch update with variables
  Future<int> batchUpdateStatus({
    required List<int> userIds,
    required String newStatus,
  }) async {
    final placeholders = userIds.map((_) => '?').join(',');
    final variables = <Variable>[
      Variable.withString(newStatus),
      ...userIds.map((id) => Variable.withInt(id)),
    ];
    
    final result = await db.customUpdate(
      '''
      UPDATE users 
      SET status = ?, updated_at = datetime('now')
      WHERE id IN ($placeholders)
      ''',
      variables: variables,
    ).go();
    
    return result;
  }

  // ==================== REPORT QUERIES ====================
  
  // 👇 Sales report with variables
  Future<SalesReport> getSalesReport({
    required DateTime startDate,
    required DateTime endDate,
    String? category,
    int? minOrders,
  }) async {
    final conditions = <String>[
      'o.order_date >= ?',
      'o.order_date <= ?',
      'o.status = "completed"',
    ];
    final variables = <Variable>[
      Variable.withDateTime(startDate),
      Variable.withDateTime(endDate),
    ];
    
    if (category != null && category.isNotEmpty) {
      conditions.add('p.category = ?');
      variables.add(Variable.withString(category));
    }
    
    if (minOrders != null) {
      conditions.add('COUNT(DISTINCT o.id) >= ?');
      variables.add(Variable.withInt(minOrders));
    }
    
    final whereClause = conditions.join(' AND ');
    
    final results = await db.customSelect(
      '''
      SELECT 
        DATE(o.order_date) as date,
        COUNT(DISTINCT o.id) as order_count,
        SUM(oi.quantity) as items_sold,
        SUM(oi.total) as revenue
      FROM orders o
      INNER JOIN order_items oi ON o.id = oi.order_id
      INNER JOIN products p ON oi.product_id = p.id
      WHERE $whereClause
      GROUP BY DATE(o.order_date)
      HAVING items_sold > 0
      ORDER BY date
      ''',
      variables: variables,
    ).get();
    
    final dailyReports = results.map((row) {
      return DailyReport(
        date: DateTime.parse(row.data['date'] as String),
        orderCount: row.data['order_count'] as int,
        itemsSold: row.data['items_sold'] as int,
        revenue: row.data['revenue'] as double,
      );
    }).toList();
    
    // Calculate totals
    final totalOrders = dailyReports.fold(0, (sum, r) => sum + r.orderCount);
    final totalItems = dailyReports.fold(0, (sum, r) => sum + r.itemsSold);
    final totalRevenue = dailyReports.fold(0.0, (sum, r) => sum + r.revenue);
    
    return SalesReport(
      dailyReports: dailyReports,
      totalOrders: totalOrders,
      totalItems: totalItems,
      totalRevenue: totalRevenue,
      days: dailyReports.length,
    );
  }
}

// ==================== DATA CLASSES ====================

class DailyReport {
  final DateTime date;
  final int orderCount;
  final int itemsSold;
  final double revenue;
  
  DailyReport({
    required this.date,
    required this.orderCount,
    required this.itemsSold,
    required this.revenue,
  });
}

class SalesReport {
  final List<DailyReport> dailyReports;
  final int totalOrders;
  final int totalItems;
  final double totalRevenue;
  final int days;
  
  SalesReport({
    required this.dailyReports,
    required this.totalOrders,
    required this.totalItems,
    required this.totalRevenue,
    required this.days,
  });
}
```

---

# Variable Best Practices

- **Always use variables** – Prevent SQL injection
- **Use appropriate Variable types** – Match SQL column types
- **Position matters** – Match variables to ? positions
- **Use type-safe variables** – `Variable.withString()`, etc.
- **Handle null values** – Use `Variable.withNull()`
- **Use in all custom queries** – Select, update, insert, delete
- **Build dynamically** – For dynamic queries
- **Test with different values** – Verify query works

---

# Variable Checklist

| Practice | Description | Impact |
|----------|-------------|--------|
| **Always Use Variables** | Prevent SQL injection | High |
| **Correct Type** | Match SQL column | High |
| **Position Matching** | Match ? order | High |
| **Null Handling** | Use withNull | Medium |
| **Dynamic Building** | For dynamic queries | Medium |
| **Testing** | Different values | High |

---

# Common Mistakes

## Mistake 1: String concatenation

Wrong:
```dart
// 🚫 SQL injection risk
final name = "Robert'; DROP TABLE users; --";
await customSelect('SELECT * FROM users WHERE name = "$name"').get();
```

Correct:
```dart
// ✅ Use variables
final name = "Robert'; DROP TABLE users; --";
await customSelect(
  'SELECT * FROM users WHERE name = ?',
  variables: [Variable.withString(name)],
).get();
```

## Mistake 2: Wrong variable type

Wrong:
```dart
// 🚫 Type mismatch
final age = '25'; // String
await customSelect(
  'SELECT * FROM users WHERE age > ?',
  variables: [Variable.withString(age)],
).get();
```

Correct:
```dart
// ✅ Correct type
final age = 25; // int
await customSelect(
  'SELECT * FROM users WHERE age > ?',
  variables: [Variable.withInt(age)],
).get();
```

## Mistake 3: Variable position mismatch

Wrong:
```dart
// 🚫 Wrong order
await customSelect(
  'SELECT * FROM users WHERE age > ? AND status = ?',
  variables: [
    Variable.withString(status), // Wrong order
    Variable.withInt(age),
  ],
).get();
```

Correct:
```dart
// ✅ Match ? positions
await customSelect(
  'SELECT * FROM users WHERE age > ? AND status = ?',
  variables: [
    Variable.withInt(age),
    Variable.withString(status),
  ],
).get();
```

---

# Summary

| Variable Type | Method | SQL Equivalent |
|---------------|--------|----------------|
| **String** | `Variable.withString()` | `'value'` |
| **Integer** | `Variable.withInt()` | `123` |
| **Double** | `Variable.withDouble()` | `99.99` |
| **Boolean** | `Variable.withBool()` | `1` or `0` |
| **DateTime** | `Variable.withDateTime()` | `'2024-01-01'` |
| **Null** | `Variable.withNull()` | `NULL` |

---

# Next Steps

Now you understand SQL variables, let's dive deeper:

- [Views](link) – Database views
- [Indexes](link) – Performance optimization
- [Triggers](link) – Database triggers

---

# Did You Know?

- **Variables prevent SQL injection** – Automatically escaped

- **Variables improve performance** – Query plan caching

- **Variables are type-safe** – Compile-time checking

- **Variables support all types** – String, int, double, bool, DateTime

- **Variables can be null** – Use `Variable.withNull()`

- **Variables are positional** – Match by position

- **Variables work in all custom queries** – Select, update, insert, delete

- **Variables are essential** – For secure dynamic queries

---

