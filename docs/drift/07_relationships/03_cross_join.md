## Cross Join

**Creating Cartesian products with Drift cross joins**

---

# What is it?

**Cross Join** (or CARTESIAN JOIN) combines every row from the first table with every row from the second table, creating a Cartesian product. This means if Table A has 10 rows and Table B has 5 rows, the result will have 50 rows (10 × 5). Cross joins are rarely used in production applications but can be useful for generating test data, creating combinations, and certain reporting scenarios.

> **Think of Cross Join like "creating all possible pairs"** – if you have 3 shirts and 4 pants, a cross join gives you all 12 possible outfits (3 × 4 combinations).

```dart
// 👇 Cross join: Every user with every product
final query = db.select(db.users).join([
  crossJoin(db.products),
]);

// Returns: users × products (all combinations)
// If 100 users and 1000 products = 100,000 rows!
final results = await query.get();

// Generated SQL:
// SELECT users.*, products.* 
// FROM users 
// CROSS JOIN products
```

> **What's happening here?**
> - **Cartesian product** – Every row × every row
> - **No condition** – No ON clause needed
> - **Explosive results** – Can generate huge datasets
> - **Use with caution** – Can be extremely large

---

# Why does it exist?

- **Test Data Generation** – Create test combinations
- **Combination Analysis** – Generate all possible pairs
- **Report Generation** – Cross-tabulation reports
- **Matrix Creation** – Build matrix-like structures
- **Template Generation** – Create all combinations of options
- **Data Sampling** – Create sample datasets

---

# Basic Cross Join

> **Simple cross join patterns**

## Basic Cross Join

```dart
// 👇 Cross join: Users and products
final query = db.select(db.users).join([
  crossJoin(db.products),
]);

final results = await query.get();
print('Total combinations: ${results.length}');

// Generated SQL:
// SELECT users.*, products.* 
// FROM users 
// CROSS JOIN products
```

## Cross Join with WHERE

```dart
// 👇 Cross join with filter
final query = db.select(db.users).join([
  crossJoin(db.products),
]);

// 👇 Filter results after cross join
query.where((u) => db.users.isActive.equals(true));
query.where((p) => db.products.isActive.equals(true));
query.where((p) => db.products.price > const Variable(50));

// This still generates all combinations first,
// then filters them
final results = await query.get();
```

## Cross Join with Limit

```dart
// 👇 Cross join with limit (safer)
final query = db.select(db.users).join([
  crossJoin(db.products),
]);

final results = await (query
  ..limit(100))
  .get();

// Only returns first 100 combinations
```

---

# Cross Join Use Cases

> **Practical applications of cross joins**

## Generating Test Data

```dart
// 👇 Generate user-product test data
Future<void> generateTestData() async {
  // Get all users and products
  final users = await db.select(db.users).get();
  final products = await db.select(db.products).get();
  
  // Generate all combinations
  final query = db.select(db.users).join([
    crossJoin(db.products),
  ]);
  
  final results = await query.limit(1000).get();
  
  // Create test records for each combination
  for (final row in results) {
    final user = row.readTable(db.users);
    final product = row.readTable(db.products);
    
    // Create test interactions
    // For example: user viewed product, added to cart, etc.
  }
}
```

## Creating Combination Reports

```dart
// 👇 Generate product-category combinations
Future<List<ProductCategoryCombination>> getProductCategoryCombos() async {
  // Instead of joining, show all products even if no category
  final query = db.select(db.products).join([
    crossJoin(db.categories),
  ]);
  
  final results = await query.get();
  
  return results.map((row) {
    final product = row.readTable(db.products);
    final category = row.readTable(db.categories);
    
    return ProductCategoryCombination(
      product: product,
      category: category,
    );
  }).toList();
}
```

## Creating Pricing Matrix

```dart
// 👇 Product × Currency cross join for pricing
Future<List<ProductPriceMatrix>> getProductPricingMatrix() async {
  final query = db.select(db.products).join([
    crossJoin(db.currencies),
  ]);
  
  query.where((p) => db.products.isActive.equals(true));
  query.where((c) => db.currencies.isActive.equals(true));
  
  final results = await query.get();
  
  return results.map((row) {
    final product = row.readTable(db.products);
    final currency = row.readTable(db.currencies);
    
    return ProductPriceMatrix(
      product: product,
      currency: currency,
      priceInCurrency: product.price * currency.exchangeRate,
    );
  }).toList();
}
```

---

# Real-World Example

> **Complete e-commerce cross join system**

```dart
// lib/database/cross_join_service.dart
import 'package:drift/drift.dart';

class CrossJoinService {
  final AppDatabase db;
  
  CrossJoinService(this.db);

  // ==================== PRODUCT COMBINATIONS ====================
  
  // 👇 Generate product-combo report
  Future<List<ProductCombination>> getProductCombinations() async {
    final query = db.select(db.products)
      .join([
        crossJoin(db.products)
      ]);
    
    query.where((p1) => db.products.isActive.equals(true));
    query.where((p2) => db.productsAlias2.isActive.equals(true));
    query.where((p1) => db.products.id < db.productsAlias2.id); // Avoid duplicates
    
    final results = await query.limit(100).get();
    
    return results.map((row) {
      final product1 = row.readTable(db.products);
      final product2 = row.readTable(db.productsAlias2);
      
      return ProductCombination(
        product1: product1,
        product2: product2,
        combinedStock: product1.stock + product2.stock,
        combinedPrice: product1.price + product2.price,
      );
    }).toList();
  }
  
  // 👇 Generate product-variant matrix
  Future<List<ProductVariantMatrix>> getProductVariantMatrix() async {
    final query = db.select(db.products)
      .join([
        crossJoin(db.variants)
      ]);
    
    query.where((p) => db.products.isActive.equals(true));
    query.where((v) => db.variants.isActive.equals(true));
    
    final results = await query.get();
    
    return results.map((row) {
      final product = row.readTable(db.products);
      final variant = row.readTable(db.variants);
      
      return ProductVariantMatrix(
        product: product,
        variant: variant,
        sku: '${product.sku}-${variant.code}',
        price: product.basePrice + variant.priceAdjustment,
      );
    }).toList();
  }

  // ==================== USER-ROLE MATRIX ====================
  
  // 👇 Generate all user-role assignments
  Future<List<UserRoleAssignment>> getAllUserRoleAssignments() async {
    final query = db.select(db.users)
      .join([
        crossJoin(db.roles)
      ]);
    
    query.where((u) => db.users.isActive.equals(true));
    
    final results = await query.get();
    
    return results.map((row) {
      final user = row.readTable(db.users);
      final role = row.readTable(db.roles);
      
      return UserRoleAssignment(
        user: user,
        role: role,
        isAssigned: false, // Default, can be checked later
      );
    }).toList();
  }

  // ==================== PERMISSION MATRIX ====================
  
  // 👇 Generate permission matrix
  Future<List<PermissionMatrix>> getPermissionMatrix() async {
    final query = db.select(db.users)
      .join([
        crossJoin(db.permissions)
      ]);
    
    final results = await query.get();
    
    return results.map((row) {
      final user = row.readTable(db.users);
      final permission = row.readTable(db.permissions);
      
      return PermissionMatrix(
        user: user,
        permission: permission,
        hasPermission: false, // Default, checked later
      );
    }).toList();
  }

  // ==================== STORE-PRODUCT MATRIX ====================
  
  // 👇 Generate store-product availability
  Future<List<StoreProductAvailability>> getStoreProductMatrix() async {
    final query = db.select(db.stores)
      .join([
        crossJoin(db.products)
      ]);
    
    query.where((s) => db.stores.isActive.equals(true));
    query.where((p) => db.products.isActive.equals(true));
    
    final results = await query.limit(1000).get();
    
    return results.map((row) {
      final store = row.readTable(db.stores);
      final product = row.readTable(db.products);
      
      return StoreProductAvailability(
        store: store,
        product: product,
        inStock: false, // Default, checked later
        price: product.price + store.margin,
      );
    }).toList();
  }

  // ==================== DATE-PRODUCT COMBINATIONS ====================
  
  // 👇 Generate schedule matrix (products for each day)
  Future<List<DailyProductSchedule>> getDailyProductSchedule(
    DateTime startDate,
    DateTime endDate,
  ) async {
    // 👇 Generate date range
    final dates = _generateDateRange(startDate, endDate);
    
    final query = db.select(db.products)
      .join([
        crossJoin(db.datesTemp)
      ]);
    
    query.where((p) => db.products.isActive.equals(true));
    
    final results = await query.get();
    
    return results.map((row) {
      final product = row.readTable(db.products);
      final date = row.readTable(db.datesTemp);
      
      return DailyProductSchedule(
        product: product,
        date: date.date,
        isScheduled: false, // Default
      );
    }).toList();
  }
  
  List<Map<String, dynamic>> _generateDateRange(
    DateTime start,
    DateTime end,
  ) {
    // Generate date range for cross join
    final dates = <Map<String, dynamic>>[];
    var current = start;
    while (current.isBefore(end) || current.isAtSameMomentAs(end)) {
      dates.add({'date': current});
      current = current.add(Duration(days: 1));
    }
    return dates;
  }
}

// ==================== DATA CLASSES ====================

class ProductCombination {
  final Product product1;
  final Product product2;
  final int combinedStock;
  final double combinedPrice;
  
  ProductCombination({
    required this.product1,
    required this.product2,
    required this.combinedStock,
    required this.combinedPrice,
  });
}

class ProductVariantMatrix {
  final Product product;
  final Variant variant;
  final String sku;
  final double price;
  
  ProductVariantMatrix({
    required this.product,
    required this.variant,
    required this.sku,
    required this.price,
  });
}

class UserRoleAssignment {
  final User user;
  final Role role;
  final bool isAssigned;
  
  UserRoleAssignment({
    required this.user,
    required this.role,
    required this.isAssigned,
  });
}

class PermissionMatrix {
  final User user;
  final Permission permission;
  final bool hasPermission;
  
  PermissionMatrix({
    required this.user,
    required this.permission,
    required this.hasPermission,
  });
}

class StoreProductAvailability {
  final Store store;
  final Product product;
  final bool inStock;
  final double price;
  
  StoreProductAvailability({
    required this.store,
    required this.product,
    required this.inStock,
    required this.price,
  });
}

class DailyProductSchedule {
  final Product product;
  final DateTime date;
  final bool isScheduled;
  
  DailyProductSchedule({
    required this.product,
    required this.date,
    required this.isScheduled,
  });
}
```

---

# Cross Join vs Other Joins

> **When to use cross join vs other joins**

```dart
// 👇 Cross Join: All combinations (no condition)
final crossResult = await db.select(db.users)
  .join([crossJoin(db.products)])
  .get(); // Every user × every product

// 👇 Inner Join: Only matching (with condition)
final innerResult = await db.select(db.users)
  .join([
    innerJoin(
      db.orders,
      db.orders.userId.equals(db.users.id),
    )
  ])
  .get(); // Only users with orders

// 👇 Left Join: All left rows (with optional matches)
final leftResult = await db.select(db.users)
  .join([
    leftJoin(
      db.orders,
      db.orders.userId.equals(db.users.id),
    )
  ])
  .get(); // All users, orders if exist
```

---

# Best Practices

- **Use cross join with caution** – Can generate huge datasets
- **Always add LIMIT** – To control result size
- **Add WHERE filters** – To reduce combinations
- **Use for test data only** – Not for production
- **Consider alternatives** – Inner join might be better
- **Test with small datasets** – Verify performance
- **Use cross join for combinations** – Matrix-like data
- **Add meaningful aliases** – For clarity

---

# Common Mistakes

## Mistake 1: Unlimited cross join

Wrong:
```dart
// 🚫 Generates millions of rows
final results = await db.select(db.users)
  .join([crossJoin(db.products)])
  .get();
```

Correct:
```dart
// ✅ Add limit
final results = await db.select(db.users)
  .join([crossJoin(db.products)])
  .limit(100)
  .get();
```

## Mistake 2: Using cross join for relationships

Wrong:
```dart
// 🚫 Wrong approach for user-orders
final query = db.select(db.users).join([
  crossJoin(db.orders),
]);
```

Correct:
```dart
// ✅ Use inner join for relationships
final query = db.select(db.users).join([
  innerJoin(db.orders, db.orders.userId.equals(db.users.id)),
]);
```

## Mistake 3: No WHERE conditions

Wrong:
```dart
// 🚫 All combinations without filtering
final results = await db.select(db.users)
  .join([crossJoin(db.products)])
  .get();
```

Correct:
```dart
// ✅ Filter to reduce combinations
final results = await db.select(db.users)
  .join([crossJoin(db.products)])
  .where((u) => db.users.isActive.equals(true))
  .where((p) => db.products.isActive.equals(true))
  .limit(100)
  .get();
```

---

# Summary

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Cross Join** | Cartesian product | Combinations |
| **No Condition** | Every row × every row | Matrix data |
| **Explosive** | Can be huge | Use with caution |
| **Test Data** | Generate combinations | Testing |

---

# Next Steps

Now you understand cross join, let's dive deeper:

- [Self Join](link) – Self join
- [Many-to-Many](link) – Many-to-many relationships
- [One-to-Many](link) – One-to-many relationships

---

# Did You Know?

- **Cross join has no ON clause** – No condition needed

- **Cross join can be huge** – 100 × 100 = 10,000 rows

- **Cross join is symmetric** – Table order doesn't matter

- **Cross join is rarely used** – In production applications

- **Cross join is useful for test data** – Generate combinations

- **Cross join can be slow** – For large tables

- **Cross join is also called CARTESIAN JOIN** – Same thing

- **Cross join can be filtered** – With WHERE clause

---

