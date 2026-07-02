## SQLite Functions

**Leveraging SQLite's built-in functions in Drift**

---

# What is it?

**SQLite Functions** are the extensive library of built-in functions that SQLite provides for manipulating data, performing calculations, and transforming values. Drift allows you to use any of these functions in your queries through custom SQL, giving you access to powerful data processing capabilities directly in the database.

> **Think of SQLite Functions like a "Swiss Army knife"** – there's a tool for almost any data manipulation task you can think of, from simple string operations to complex date calculations and mathematical functions.

```dart
// 👇 Using SQLite functions in Drift
final results = await customSelect('''
  SELECT 
    name,
    UPPER(name) as uppercase_name,
    LENGTH(name) as name_length,
    SUBSTR(email, 1, INSTR(email, '@') - 1) as username,
    STRFTIME('%Y-%m', created_at) as signup_month,
    DATE('now') - DATE(created_at) as days_since_signup
  FROM users
''').get();
```

> **What's happening here?**
> - **String functions** – `UPPER()`, `LENGTH()`, `SUBSTR()`
> - **Date functions** – `STRFTIME()`, `DATE()`
> - **Search functions** – `INSTR()`
> - **Calculations** – Date arithmetic

---

# Why does it exist?

- **Data Manipulation** – Transform data efficiently
- **Performance** – Process data at database level
- **Convenience** – Built-in functions for common tasks
- **Power** – Complex operations with simple functions
- **Standardization** – SQL-standard functions
- **Integration** – Work with other SQL features

---

# String Functions

> **Manipulating text data**

## Basic String Functions

```dart
// 👇 String manipulation
final results = await customSelect('''
  SELECT 
    name,
    UPPER(name) as uppercase,
    LOWER(name) as lowercase,
    LENGTH(name) as length,
    TRIM(name) as trimmed,
    SUBSTR(name, 1, 5) as first_five,
    REPLACE(name, ' ', '_') as with_underscores
  FROM users
''').get();
```

## Search and Extraction

```dart
// 👇 Finding and extracting text
final results = await customSelect('''
  SELECT 
    email,
    INSTR(email, '@') as at_position,
    SUBSTR(email, 1, INSTR(email, '@') - 1) as username,
    SUBSTR(email, INSTR(email, '@') + 1) as domain,
    LENGTH(email) - LENGTH(REPLACE(email, '.', '')) as dot_count
  FROM users
''').get();
```

## String Aggregation

```dart
// 👇 Group concatenation
final results = await customSelect('''
  SELECT 
    category_id,
    GROUP_CONCAT(name) as all_names,
    GROUP_CONCAT(DISTINCT name) as unique_names,
    GROUP_CONCAT(name, '; ') as separated_names,
    COUNT(*) as count
  FROM products
  GROUP BY category_id
''').get();
```

---

# Date and Time Functions

> **Working with dates and times**

## Date Extraction

```dart
// 👇 Extract parts from dates
final results = await customSelect('''
  SELECT 
    order_date,
    DATE(order_date) as date_only,
    TIME(order_date) as time_only,
    STRFTIME('%Y', order_date) as year,
    STRFTIME('%m', order_date) as month,
    STRFTIME('%d', order_date) as day,
    STRFTIME('%Y-%m', order_date) as year_month,
    STRFTIME('%W', order_date) as week_number
  FROM orders
''').get();
```

## Date Arithmetic

```dart
// 👇 Date calculations
final results = await customSelect('''
  SELECT 
    created_at,
    DATE('now') as today,
    DATE('now', 'start of month') as month_start,
    DATE('now', '-7 days') as week_ago,
    JULIANDAY('now') - JULIANDAY(created_at) as days_old,
    STRFTIME('%Y-%m', created_at) as month
  FROM users
''').get();
```

## Date Formatting

```dart
// 👇 Custom date formatting
final results = await customSelect('''
  SELECT 
    order_date,
    STRFTIME('%Y-%m-%d %H:%M:%S', order_date) as iso_format,
    STRFTIME('%m/%d/%Y', order_date) as us_format,
    STRFTIME('%d %b %Y', order_date) as text_format,
    STRFTIME('%H:%M', order_date) as time_only
  FROM orders
''').get();
```

---

# Math Functions

> **Performing calculations**

## Basic Math

```dart
// 👇 Mathematical operations
final results = await customSelect('''
  SELECT 
    price,
    ROUND(price, 2) as rounded,
    ABS(price) as absolute,
    CEIL(price) as ceiling,
    FLOOR(price) as floor,
    ROUND(price * 1.1, 2) as with_tax,
    price * RANDOM() as random_value
  FROM products
''').get();
```

## Advanced Math

```dart
// 👇 Advanced calculations
final results = await customSelect('''
  SELECT 
    x, y,
    ABS(x - y) as difference,
    MAX(x, y) as max_value,
    MIN(x, y) as min_value,
    POW(x, 2) as x_squared,
    SQRT(x) as sqrt_x
  FROM coordinates
''').get();
```

---

# Aggregate Functions

> **Summary calculations**

## Basic Aggregates

```dart
// 👇 Summary statistics
final results = await customSelect('''
  SELECT 
    COUNT(*) as total,
    SUM(total) as sum,
    AVG(total) as average,
    MAX(total) as max,
    MIN(total) as min,
    TOTAL(total) as total_exact
  FROM orders
''').get();
```

## Grouped Aggregates

```dart
// 👇 Aggregates by group
final results = await customSelect('''
  SELECT 
    category,
    COUNT(*) as count,
    AVG(price) as avg_price,
    MAX(price) as max_price,
    MIN(price) as min_price,
    SUM(stock) as total_stock,
    GROUP_CONCAT(name) as product_names
  FROM products
  GROUP BY category
  HAVING COUNT(*) > 1
  ORDER BY avg_price DESC
''').get();
```

---

# Conditional Functions

> **Logic and branching**

## CASE Expressions

```dart
// 👇 Conditional logic with CASE
final results = await customSelect('''
  SELECT 
    name,
    price,
    CASE 
      WHEN price < 10 THEN 'Budget'
      WHEN price < 50 THEN 'Mid-Range'
      WHEN price < 100 THEN 'Premium'
      ELSE 'Luxury'
    END as price_tier,
    CASE 
      WHEN stock = 0 THEN 'Out of Stock'
      WHEN stock < 10 THEN 'Low Stock'
      ELSE 'In Stock'
    END as stock_status
  FROM products
''').get();
```

## IIF (SQLite-specific)

```dart
// 👇 Simple conditional
final results = await customSelect('''
  SELECT 
    name,
    price,
    IIF(stock = 0, 'Out of Stock', 'In Stock') as stock_status,
    IIF(price > 100, 'Expensive', 'Affordable') as price_category
  FROM products
''').get();
```

---

# Real-World Example

> **Complete e-commerce SQLite functions system**

```dart
// lib/database/sqlite_functions_service.dart
import 'package:drift/drift.dart';

class SqliteFunctionsService {
  final AppDatabase db;
  
  SqliteFunctionsService(this.db);

  // ==================== DATA CLEANING ====================
  
  // 👇 Clean user data
  Future<List<CleanedUser>> getCleanedUsers() async {
    final results = await db.customSelect('''
      SELECT 
        id,
        TRIM(name) as cleaned_name,
        LOWER(TRIM(email)) as normalized_email,
        REPLACE(phone, ' ', '') as cleaned_phone,
        REPLACE(UPPER(address), '\n', ', ') as cleaned_address,
        LENGTH(TRIM(bio)) as bio_length,
        INSTR(email, '@') as email_validation
      FROM users
      WHERE TRIM(name) != ''
      AND INSTR(email, '@') > 0
    ''').get();
    
    return results.map((row) {
      return CleanedUser(
        id: row.data['id'] as int,
        cleanedName: row.data['cleaned_name'] as String,
        normalizedEmail: row.data['normalized_email'] as String,
        cleanedPhone: row.data['cleaned_phone'] as String?,
        cleanedAddress: row.data['cleaned_address'] as String?,
        bioLength: row.data['bio_length'] as int,
        emailValidation: row.data['email_validation'] as int,
      );
    }).toList();
  }

  // ==================== DATE ANALYTICS ====================
  
  // 👇 Comprehensive date analytics
  Future<DateAnalytics> getDateAnalytics() async {
    // User signups by month
    final signups = await db.customSelect('''
      SELECT 
        STRFTIME('%Y-%m', created_at) as month,
        COUNT(*) as signups,
        SUM(COUNT(*)) OVER (ORDER BY STRFTIME('%Y-%m', created_at)) as cumulative_signups,
        (COUNT(*) * 100.0 / (SELECT COUNT(*) FROM users)) as percentage
      FROM users
      GROUP BY STRFTIME('%Y-%m', created_at)
      ORDER BY month DESC
      LIMIT 12
    ''').get();
    
    // Order patterns by day
    final patterns = await db.customSelect('''
      SELECT 
        STRFTIME('%w', order_date) as day_of_week,
        CASE STRFTIME('%w', order_date)
          WHEN '0' THEN 'Sunday'
          WHEN '1' THEN 'Monday'
          WHEN '2' THEN 'Tuesday'
          WHEN '3' THEN 'Wednesday'
          WHEN '4' THEN 'Thursday'
          WHEN '5' THEN 'Friday'
          WHEN '6' THEN 'Saturday'
        END as day_name,
        COUNT(*) as orders,
        SUM(total) as revenue,
        AVG(total) as avg_order,
        ROUND(AVG(total), 2) as rounded_avg
      FROM orders
      WHERE status = 'completed'
      GROUP BY STRFTIME('%w', order_date)
      ORDER BY day_of_week
    ''').get();
    
    return DateAnalytics(
      monthlySignups: signups.map((row) {
        return MonthlySignup(
          month: row.data['month'] as String,
          signups: row.data['signups'] as int,
          cumulativeSignups: row.data['cumulative_signups'] as int,
          percentage: row.data['percentage'] as double,
        );
      }).toList(),
      orderPatterns: patterns.map((row) {
        return OrderPattern(
          dayName: row.data['day_name'] as String,
          orders: row.data['orders'] as int,
          revenue: row.data['revenue'] as double,
          avgOrder: row.data['avg_order'] as double,
        );
      }).toList(),
    );
  }

  // ==================== TEXT ANALYSIS ====================
  
  // 👇 Analyze product descriptions
  Future<List<ProductTextAnalysis>> analyzeProductTexts() async {
    final results = await db.customSelect('''
      SELECT 
        id,
        name,
        description,
        LENGTH(description) as description_length,
        LENGTH(REPLACE(description, ' ', '')) as text_without_spaces,
        LENGTH(description) - LENGTH(REPLACE(description, ' ', '')) as word_count,
        INSTR(description, 'price') as price_mentioned,
        REPLACE(REPLACE(description, '.', ''), ',', '') as clean_description,
        UPPER(SUBSTR(name, 1, 1)) as first_letter,
        LOWER(name) as lowercase_name,
        REPLACE(name, ' ', '-') as url_slug
      FROM products
      WHERE LENGTH(description) > 0
    ''').get();
    
    return results.map((row) {
      return ProductTextAnalysis(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        descriptionLength: row.data['description_length'] as int,
        wordCount: row.data['word_count'] as int,
        priceMentioned: (row.data['price_mentioned'] as int) > 0,
        cleanDescription: row.data['clean_description'] as String,
        urlSlug: row.data['url_slug'] as String,
      );
    }).toList();
  }

  // ==================== RANDOM SAMPLING ====================
  
  // 👇 Get random sample of records
  Future<List<Product>> getRandomProducts(int limit) async {
    final results = await db.customSelect(
      '''
      SELECT * FROM products
      WHERE is_active = 1
      ORDER BY RANDOM()
      LIMIT ?
      ''',
      variables: [Variable.withInt(limit)],
    ).get();
    
    return results.map((row) {
      return Product(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        sku: row.data['sku'] as String,
        price: row.data['price'] as double,
        stock: row.data['stock'] as int,
        // ... other fields
      );
    }).toList();
  }

  // ==================== DATA VALIDATION ====================
  
  // 👇 Validate user data with functions
  Future<List<UserValidation>> validateUsers() async {
    final results = await db.customSelect('''
      SELECT 
        id,
        name,
        email,
        phone,
        CASE 
          WHEN TRIM(name) == '' THEN 'Invalid'
          WHEN LENGTH(TRIM(name)) < 2 THEN 'Too Short'
          WHEN INSTR(email, '@') == 0 THEN 'Invalid Email'
          WHEN LENGTH(email) > 100 THEN 'Email Too Long'
          WHEN LENGTH(REPLACE(REPLACE(phone, ' ', ''), '-', '')) < 10 THEN 'Invalid Phone'
          ELSE 'Valid'
        END as validation_status,
        INSTR(email, '@') as at_position,
        LENGTH(phone) - LENGTH(REPLACE(REPLACE(phone, ' ', ''), '-', '')) as phone_digits,
        IIF(LENGTH(TRIM(name)) > 0, 'Has Name', 'Missing Name') as name_status
      FROM users
    ''').get();
    
    return results.map((row) {
      return UserValidation(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        email: row.data['email'] as String,
        phone: row.data['phone'] as String?,
        validationStatus: row.data['validation_status'] as String,
        atPosition: row.data['at_position'] as int,
        phoneDigits: row.data['phone_digits'] as int,
        nameStatus: row.data['name_status'] as String,
      );
    }).toList();
  }

  // ==================== HIERARCHY WITH DATE ====================
  
  // 👇 Find all subcategories with depth and path
  Future<List<CategoryPath>> getCategoryPaths() async {
    final results = await db.customSelect('''
      WITH RECURSIVE category_tree AS (
        SELECT 
          id,
          name,
          parent_id,
          0 as depth,
          name as path,
          GROUP_CONCAT(id) as id_path
        FROM categories
        WHERE parent_id IS NULL
        
        UNION ALL
        
        SELECT 
          c.id,
          c.name,
          c.parent_id,
          ct.depth + 1,
          ct.path || ' > ' || c.name,
          ct.id_path || ',' || c.id
        FROM categories c
        INNER JOIN category_tree ct ON c.parent_id = ct.id
      )
      SELECT 
        id,
        name,
        depth,
        path,
        REPLACE(path, ' > ', '/') as url_path,
        LENGTH(path) - LENGTH(REPLACE(path, '>', '')) as path_length,
        IIF(depth = 0, 'Root', IIF(depth = 1, 'Parent', 'Child')) as level_type
      FROM category_tree
      ORDER BY path
    ''').get();
    
    return results.map((row) {
      return CategoryPath(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        depth: row.data['depth'] as int,
        path: row.data['path'] as String,
        urlPath: row.data['url_path'] as String,
        pathLength: row.data['path_length'] as int,
        levelType: row.data['level_type'] as String,
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class CleanedUser {
  final int id;
  final String cleanedName;
  final String normalizedEmail;
  final String? cleanedPhone;
  final String? cleanedAddress;
  final int bioLength;
  final int emailValidation;
  
  CleanedUser({
    required this.id,
    required this.cleanedName,
    required this.normalizedEmail,
    this.cleanedPhone,
    this.cleanedAddress,
    required this.bioLength,
    required this.emailValidation,
  });
}

class MonthlySignup {
  final String month;
  final int signups;
  final int cumulativeSignups;
  final double percentage;
  
  MonthlySignup({
    required this.month,
    required this.signups,
    required this.cumulativeSignups,
    required this.percentage,
  });
}

class OrderPattern {
  final String dayName;
  final int orders;
  final double revenue;
  final double avgOrder;
  
  OrderPattern({
    required this.dayName,
    required this.orders,
    required this.revenue,
    required this.avgOrder,
  });
}

class DateAnalytics {
  final List<MonthlySignup> monthlySignups;
  final List<OrderPattern> orderPatterns;
  
  DateAnalytics({
    required this.monthlySignups,
    required this.orderPatterns,
  });
}

class ProductTextAnalysis {
  final int id;
  final String name;
  final int descriptionLength;
  final int wordCount;
  final bool priceMentioned;
  final String cleanDescription;
  final String urlSlug;
  
  ProductTextAnalysis({
    required this.id,
    required this.name,
    required this.descriptionLength,
    required this.wordCount,
    required this.priceMentioned,
    required this.cleanDescription,
    required this.urlSlug,
  });
}

class UserValidation {
  final int id;
  final String name;
  final String email;
  final String? phone;
  final String validationStatus;
  final int atPosition;
  final int phoneDigits;
  final String nameStatus;
  
  UserValidation({
    required this.id,
    required this.name,
    required this.email,
    this.phone,
    required this.validationStatus,
    required this.atPosition,
    required this.phoneDigits,
    required this.nameStatus,
  });
}

class CategoryPath {
  final int id;
  final String name;
  final int depth;
  final String path;
  final String urlPath;
  final int pathLength;
  final String levelType;
  
  CategoryPath({
    required this.id,
    required this.name,
    required this.depth,
    required this.path,
    required this.urlPath,
    required this.pathLength,
    required this.levelType,
  });
}
```

---

# SQLite Function Categories

| Category | Functions | Use Case |
|----------|-----------|----------|
| **String** | `UPPER()`, `LOWER()`, `LENGTH()`, `SUBSTR()`, `INSTR()`, `REPLACE()` | Text manipulation |
| **Date/Time** | `DATE()`, `TIME()`, `STRFTIME()`, `JULIANDAY()` | Date operations |
| **Math** | `ABS()`, `ROUND()`, `CEIL()`, `FLOOR()`, `RANDOM()` | Calculations |
| **Aggregate** | `COUNT()`, `SUM()`, `AVG()`, `MAX()`, `MIN()`, `TOTAL()` | Summary stats |
| **Conditional** | `CASE`, `IIF()`, `COALESCE()` | Branching logic |
| **Conversion** | `CAST()`, `TYPEOF()` | Type conversion |

---

# Best Practices

- **Use functions at database level** – Better performance
- **Combine functions** – Chain operations
- **Use STRFTIME for formatting** – Flexible date formatting
- **Use COALESCE for defaults** – Handle NULL values
- **Use ROUND for precision** – Control decimal places
- **Use LENGTH with TRIM** – Accurate string lengths
- **Index function results** – For performance
- **Test functions** – Verify results

---

# Common Mistakes

## Mistake 1: Wrong date format

Wrong:
```dart
// 🚫 Wrong date format
STRFTIME('%Y/%m/%d', order_date)
```

Correct:
```dart
// ✅ Correct format
STRFTIME('%Y-%m-%d', order_date)
```

## Mistake 2: Not handling NULL

Wrong:
```dart
// 🚫 NULL values cause issues
LENGTH(description) // NULL if description is NULL
```

Correct:
```dart
// ✅ Handle NULL
COALESCE(LENGTH(description), 0) as length
```

## Mistake 3: Case sensitivity

Wrong:
```dart
// 🚫 Case-sensitive comparison
WHERE name = 'john' // Won't match 'John'
```

Correct:
```dart
// ✅ Case-insensitive comparison
WHERE UPPER(name) = 'JOHN'
```

---

# Summary

| Function Type | Purpose | Example |
|---------------|---------|---------|
| **String** | Manipulate text | `UPPER()`, `LOWER()` |
| **Date** | Handle dates | `STRFTIME()`, `DATE()` |
| **Math** | Calculations | `ROUND()`, `ABS()` |
| **Aggregate** | Summarize | `COUNT()`, `SUM()` |
| **Conditional** | Branching | `CASE`, `IIF()` |

---

# Next Steps

Now you understand SQLite functions, let's dive deeper:

- [Migrations](link) – Schema management
- [Type Converters](link) – Custom type converters
- [DAO](link) – Data Access Objects

---

# Did You Know?

- **SQLite has 50+ built-in functions** – Extensive library

- **STRFTIME supports all date formats** – Highly flexible

- **GROUP_CONCAT is unique** – SQLite-specific function

- **RANDOM() is fast** – For sampling

- **Functions can be combined** – Chain operations

- **Functions can be used in indexes** – For performance

- **Functions are optimized** - Built into SQLite

- **Functions are production-ready** - Used in large systems

---
