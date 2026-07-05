## Replace

**Insert or replace records in Drift**

---

# What is it?

**Replace** is an insert operation that either inserts a new record or replaces an existing record if a conflict occurs (usually on a unique constraint or primary key). When a conflict happens, the existing record is deleted and the new record is inserted in its place.

> **Think of Replace like "overwriting a file"** – if the file exists, you delete the old one and save the new version; if it doesn't exist, you just save the new file.

```dart
// 👇 Replace a user
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1), // Existing ID
    name: 'John Updated',
    email: 'john.new@example.com',
  ),
  mode: InsertMode.replace, // 👈 Replace on conflict
);

// What happens:
// If user with ID 1 exists: DELETE old, INSERT new
// If user with ID 1 doesn't exist: INSERT new

// Generated SQL:
// INSERT OR REPLACE INTO users (id, name, email) 
// VALUES (1, 'John Updated', 'john.new@example.com')
```

> **What's happening here?**
> - **`InsertMode.replace`** – Replace on conflict
> - **Conflict detection** – Based on PRIMARY KEY or UNIQUE constraints
> - **Delete + Insert** – Existing record removed, new record added
> - **Atomic operation** – Happens in a single database operation

---

# Why does it exist?

- **Upsert Simplicity** – Insert or replace without complex logic
- **Data Synchronization** – Sync external data sources
- **Cache Updates** – Update cached data
- **Idempotent Operations** – Safe to run multiple times
- **Data Migration** – Replace existing records
- **Conflict Resolution** – Handle duplicates automatically

---

# Basic Replace Operations

> **Simple replace operations**

## Replace by Primary Key

```dart
// 👇 Replace user by ID
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1), // 👈 Primary key
    name: 'John Doe',
    email: 'john.doe@example.com',
    age: Value(30),
  ),
  mode: InsertMode.replace,
);

// Generated SQL:
// INSERT OR REPLACE INTO users (id, name, email, age) 
// VALUES (1, 'John Doe', 'john.doe@example.com', 30)
```

## Replace by Unique Constraint

```dart
// 👇 Replace by unique email
await into(users).insert(
  UsersCompanion.insert(
    email: 'john@example.com', // 👈 Unique constraint
    name: 'John Updated',
    age: Value(25),
  ),
  mode: InsertMode.replace,
);

// If email 'john@example.com' exists:
// 1. Delete existing record with that email
// 2. Insert new record with new data
```

---

## Replace with All Fields

```dart
// 👇 Full replace (all fields provided)
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'Jane Smith',
    email: 'jane@example.com',
    age: Value(28),
    isActive: Value(true),
    isVerified: Value(true),
    createdAt: Value(DateTime(2024, 1, 1)),
  ),
  mode: InsertMode.replace,
);
```

## Replace with Partial Data

```dart
// 👇 Replace with partial data (missing fields use defaults)
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'John Updated',
    email: 'john.updated@example.com',
    // Missing fields will use database defaults
  ),
  mode: InsertMode.replace,
);
```

---

# Replace vs Other Operations

> **Understanding the differences**

## Replace vs Insert with Conflict

```dart
// 👇 Insert with conflict (insert or ignore)
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'John',
    email: 'john@example.com',
  ),
  onConflict: DoNothing(), // Skip if conflict
);

// 👇 Insert with update (upsert)
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'John Updated',
    email: 'john@example.com',
  ),
  onConflict: DoUpdate(
    target: users.id,
    update: UsersCompanion(
      name: Value('John Updated'),
    ),
  ),
);

// 👇 Replace (delete + insert)
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'John Updated',
    email: 'john.new@example.com',
  ),
  mode: InsertMode.replace,
);
```

---

## Replace vs Update

```dart
// 👇 Update (only changes specified fields)
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'),
    // Other fields remain unchanged
  ));

// 👇 Replace (completely replaces record)
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'John Updated',
    email: 'john.new@example.com',
    // All fields must be provided or use defaults
  ),
  mode: InsertMode.replace,
);
```

---

# Advanced Replace Patterns

> **Complex replace operations**

## Pattern 1: Replace with Cascade

```dart
// 👇 Replace automatically handles cascade deletes
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)();
}

// When replacing a user, old posts are deleted (cascade)
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'New User',
    email: 'new@example.com',
  ),
  mode: InsertMode.replace,
);

// What happens:
// 1. Old user with ID 1 is deleted (posts cascade delete)
// 2. New user with ID 1 is inserted
```

---

## Pattern 2: Replace with Timestamp

```dart
// 👇 Replace with updated timestamp
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}

// Replace will keep or reset timestamps
await into(users).insert(
  UsersCompanion.insert(
    email: 'john@example.com',
    name: 'John',
    createdAt: Value(DateTime(2024, 1, 1)), // Keep original
    updatedAt: Value(DateTime.now()),      // Set new update time
  ),
  mode: InsertMode.replace,
);
```

---

## Pattern 3: Sync External Data

```dart
// 👇 Sync users from external API
Future<void> syncUsersFromApi(List<ApiUser> apiUsers) async {
  for (final apiUser in apiUsers) {
    await into(users).insert(
      UsersCompanion.insert(
        id: Value(apiUser.id),
        name: apiUser.name,
        email: apiUser.email,
        age: Value(apiUser.age),
        isActive: Value(apiUser.isActive),
        updatedAt: Value(DateTime.now()),
      ),
      mode: InsertMode.replace,
    );
  }
}

// This will:
// - Insert new users
// - Replace existing users with new data
// - Maintain same IDs
```

---

## Pattern 4: Replace with Validation

```dart
// 👇 Replace with data validation
Future<void> replaceUserWithValidation({
  required int id,
  required String name,
  required String email,
  int? age,
}) async {
  // Validate data
  if (name.length < 2) {
    throw ArgumentError('Name must be at least 2 characters');
  }
  if (!email.contains('@')) {
    throw ArgumentError('Invalid email format');
  }
  if (age != null && (age < 0 || age > 150)) {
    throw ArgumentError('Invalid age');
  }
  
  // Replace user
  await into(users).insert(
    UsersCompanion.insert(
      id: Value(id),
      name: name,
      email: email,
      age: Value(age),
      updatedAt: Value(DateTime.now()),
    ),
    mode: InsertMode.replace,
  );
}
```

---

# Real-World Example

> **Complete e-commerce replace system**

```dart
// lib/database/database.dart
@DriftDatabase(tables: [Users, Products, Orders, OrderItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // ==================== USER REPLACES ====================
  
  // 👇 Replace user (sync from external source)
  Future<void> replaceUserFromSync({
    required int id,
    required String name,
    required String email,
    int? age,
    bool isActive = true,
  }) async {
    await into(users).insert(
      UsersCompanion.insert(
        id: Value(id),
        name: name,
        email: email,
        age: Value(age),
        isActive: Value(isActive),
        updatedAt: Value(DateTime.now()),
      ),
      mode: InsertMode.replace,
    );
  }
  
  // 👇 Replace user with returning
  Future<User?> replaceUserReturning({
    required int id,
    required String name,
    required String email,
    int? age,
  }) async {
    final results = await into(users).insertReturning(
      UsersCompanion.insert(
        id: Value(id),
        name: name,
        email: email,
        age: Value(age),
        updatedAt: Value(DateTime.now()),
      ),
      mode: InsertMode.replace,
    );
    return results.isNotEmpty ? results.first : null;
  }

  // ==================== PRODUCT REPLACES ====================
  
  // 👇 Replace product (from inventory sync)
  Future<void> replaceProductFromInventory({
    required int id,
    required String sku,
    required String name,
    required double price,
    required int stock,
    String? description,
  }) async {
    await into(products).insert(
      ProductsCompanion.insert(
        id: Value(id),
        sku: sku,
        name: name,
        price: price,
        stock: Value(stock),
        description: Value(description),
        updatedAt: Value(DateTime.now()),
      ),
      mode: InsertMode.replace,
    );
  }
  
  // 👇 Replace multiple products
  Future<void> replaceProductsBatch(List<ProductSyncData> products) async {
    await transaction(() async {
      for (final product in products) {
        await into(products).insert(
          ProductsCompanion.insert(
            id: Value(product.id),
            sku: product.sku,
            name: product.name,
            price: product.price,
            stock: Value(product.stock),
            description: Value(product.description),
            updatedAt: Value(DateTime.now()),
          ),
          mode: InsertMode.replace,
        );
      }
    });
  }
  
  // 👇 Replace product with stock adjustment
  Future<void> replaceProductWithStockHistory({
    required int id,
    required String sku,
    required String name,
    required double price,
    required int newStock,
  }) async {
    // Get old stock before replace
    final oldProduct = await (select(products)
      ..where((p) => p.id.equals(id)))
      .getSingleOrNull();
    
    final oldStock = oldProduct?.stock ?? 0;
    
    // Replace product
    await into(products).insert(
      ProductsCompanion.insert(
        id: Value(id),
        sku: sku,
        name: name,
        price: price,
        stock: Value(newStock),
        updatedAt: Value(DateTime.now()),
      ),
      mode: InsertMode.replace,
    );
    
    // Record stock change (if you have a stock history table)
    if (oldStock != newStock) {
      await _recordStockChange(id, oldStock, newStock);
    }
  }
  
  Future<void> _recordStockChange(int productId, int oldStock, int newStock) async {
    // Record stock change in history table
    await customInsert(
      '''
      INSERT INTO stock_history (product_id, old_stock, new_stock, changed_at)
      VALUES (?, ?, ?, ?)
      ''',
      variables: [
        Variable.withInt(productId),
        Variable.withInt(oldStock),
        Variable.withInt(newStock),
        Variable.withString(DateTime.now().toIso8601String()),
      ],
    ).go();
  }

  // ==================== BULK REPLACE ====================
  
  // 👇 Bulk replace (sync entire catalog)
  Future<void> syncProductCatalog(List<CatalogItem> catalog) async {
    await transaction(() async {
      // Replace all products
      for (final item in catalog) {
        await into(products).insert(
          ProductsCompanion.insert(
            id: Value(item.id),
            sku: item.sku,
            name: item.name,
            price: item.price,
            stock: Value(item.stock),
            description: Value(item.description),
            isActive: Value(item.isActive),
            updatedAt: Value(DateTime.now()),
          ),
          mode: InsertMode.replace,
        );
      }
    });
  }
  
  // 👇 Batch replace with insertAll
  Future<void> syncUsersBatch(List<UserSyncData> users) async {
    final companions = users.map((user) =>
      UsersCompanion.insert(
        id: Value(user.id),
        name: user.name,
        email: user.email,
        age: Value(user.age),
        isActive: Value(user.isActive),
        updatedAt: Value(DateTime.now()),
      )
    ).toList();
    
    // insertAll with replace mode
    await into(users).insertAll(
      companions,
      mode: InsertMode.replace,
    );
  }
  
  // 👇 Replace or insert with conflict handling
  Future<void> replaceOrUpdateProduct({
    required int id,
    required String sku,
    required String name,
    required double price,
    int? stock,
  }) async {
    try {
      // Try replace first
      await into(products).insert(
        ProductsCompanion.insert(
          id: Value(id),
          sku: sku,
          name: name,
          price: price,
          stock: stock != null ? Value(stock) : const Value.absent(),
          updatedAt: Value(DateTime.now()),
        ),
        mode: InsertMode.replace,
      );
    } catch (e) {
      // If replace fails, update instead
      await (update(products)..where((p) => p.id.equals(id)))
        .write(ProductsCompanion(
          name: Value(name),
          price: Value(price),
          stock: stock != null ? Value(stock) : const Value.absent(),
          updatedAt: Value(DateTime.now()),
        ));
    }
  }

  // ==================== RELATIONSHIP REPLACE ====================
  
  // 👇 Replace order with all its items
  Future<void> replaceOrderWithItems({
    required int orderId,
    required int userId,
    required List<OrderItemData> items,
    required String shippingAddress,
  }) async {
    await transaction(() async {
      // Calculate total
      final total = items.fold(
        0.0, 
        (sum, item) => sum + item.price * item.quantity,
      );
      
      // Replace order
      await into(orders).insert(
        OrdersCompanion.insert(
          id: Value(orderId),
          orderNumber: Value('ORD-$orderId'),
          userId: userId,
          total: total,
          shippingAddress: shippingAddress,
          status: Value('pending'),
          updatedAt: Value(DateTime.now()),
        ),
        mode: InsertMode.replace,
      );
      
      // Replace order items (delete old, insert new)
      // First, delete existing items
      await (delete(orderItems)..where((i) => i.orderId.equals(orderId))).go();
      
      // Then insert new items
      for (final item in items) {
        await into(orderItems).insert(
          OrderItemsCompanion.insert(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: item.price,
            subtotal: item.price * item.quantity,
            total: item.price * item.quantity,
          ),
        );
      }
    });
  }
  
  // ==================== CACHE MANAGEMENT ====================
  
  // 👇 Replace cache data
  Future<void> updateProductCache(List<ProductCacheData> products) async {
    // Replace all products in cache table
    await into(productCache).insertAll(
      products.map((p) => ProductCacheCompanion.insert(
        id: Value(p.id),
        name: p.name,
        price: p.price,
        stock: p.stock,
        lastUpdated: Value(DateTime.now()),
      )).toList(),
      mode: InsertMode.replace,
    );
  }
  
  // 👇 Incremental cache update
  Future<void> updateCacheIncremental(List<ProductCacheData> products) async {
    for (final product in products) {
      // Check if exists
      final existing = await (select(productCache)
        ..where((c) => c.id.equals(product.id)))
        .getSingleOrNull();
      
      if (existing == null) {
        // Insert new
        await into(productCache).insert(
          ProductCacheCompanion.insert(
            id: product.id,
            name: product.name,
            price: product.price,
            stock: product.stock,
          ),
        );
      } else {
        // Replace with latest
        await into(productCache).insert(
          ProductCacheCompanion.insert(
            id: product.id,
            name: product.name,
            price: product.price,
            stock: product.stock,
            lastUpdated: Value(DateTime.now()),
          ),
          mode: InsertMode.replace,
        );
      }
    }
  }
}

// ==================== DATA CLASSES ====================

class ProductSyncData {
  final int id;
  final String sku;
  final String name;
  final double price;
  final int stock;
  final String? description;
  
  ProductSyncData({
    required this.id,
    required this.sku,
    required this.name,
    required this.price,
    required this.stock,
    this.description,
  });
}

class UserSyncData {
  final int id;
  final String name;
  final String email;
  final int? age;
  final bool isActive;
  
  UserSyncData({
    required this.id,
    required this.name,
    required this.email,
    this.age,
    this.isActive = true,
  });
}

class CatalogItem {
  final int id;
  final String sku;
  final String name;
  final double price;
  final int stock;
  final String? description;
  final bool isActive;
  
  CatalogItem({
    required this.id,
    required this.sku,
    required this.name,
    required this.price,
    required this.stock,
    this.description,
    this.isActive = true,
  });
}

class OrderItemData {
  final int productId;
  final int quantity;
  final double price;
  
  OrderItemData({
    required this.productId,
    required this.quantity,
    required this.price,
  });
}

class ProductCacheData {
  final int id;
  final String name;
  final double price;
  final int stock;
  
  ProductCacheData({
    required this.id,
    required this.name,
    required this.price,
    required this.stock,
  });
}
```

---

# Replace vs Update vs Upsert

> **When to use each operation**

```dart
// 👇 UPDATE: Modify specific fields
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'),
    // Other fields stay the same
  ));

// 👇 UPSERT: Insert or update specific fields
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'John Updated',
    age: Value(30),
  ),
  onConflict: DoUpdate(
    target: users.id,
    update: UsersCompanion(
      name: Value('John Updated'),
      age: Value(30),
    ),
  ),
);

// 👇 REPLACE: Delete and insert entire record
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'John Updated',
    email: 'john.new@example.com',
    age: Value(30),
  ),
  mode: InsertMode.replace,
);
```

---

# Best Practices

- **Use replace for synchronization** – External data sources
- **Use replace for complete updates** – When all fields change
- **Use update for partial updates** – When only some fields change
- **Use replace for cache updates** – Refresh cached data
- **Handle foreign keys** – Cascade deletes will trigger
- **Provide all required fields** – Or ensure defaults exist
- **Use transactions** – For multiple replaces
- **Test replace behavior** – Understand what gets replaced

---

# Common Mistakes

## Mistake 1: Using replace when update is better

Wrong:
```dart
// 🚫 Replaces entire record unnecessarily
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'John Updated',
    // Missing other fields will use defaults
  ),
  mode: InsertMode.replace,
);
```

Correct:
```dart
// ✅ Use update for partial changes
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'),
    // Other fields unchanged
  ));
```

## Mistake 2: Forgetting foreign key cascade

Wrong:
```dart
// 🚫 May accidentally delete related data
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'New Name',
  ),
  mode: InsertMode.replace,
);
// If ON DELETE CASCADE is set, related data is deleted!
```

Correct:
```dart
// ✅ Check foreign key constraints first
// Or use ON DELETE RESTRICT to prevent accidental deletion
```

## Mistake 3: Not providing all fields

Wrong:
```dart
// 🚫 Missing required fields may use defaults
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'John',
    // email not provided! May use default or fail
  ),
  mode: InsertMode.replace,
);
```

Correct:
```dart
// ✅ Provide all required fields
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    name: 'John',
    email: 'john@example.com', // Always provide
  ),
  mode: InsertMode.replace,
);
```

---

# Summary

| Operation | Behavior | Best For |
|-----------|----------|----------|
| **Update** | Modify specific fields | Partial changes |
| **Upsert** | Insert or update | Insert with conflict handling |
| **Replace** | Delete + Insert | Complete record replacement |

---

# Next Steps

Now you understand replace, let's dive deeper:

- [Upsert](link) – Insert or update with conflict handling
- [Update](link) – Updating existing records
- [Delete](link) – Deleting records

---

# Did You Know?

- **Replace is INSERT OR REPLACE** – Under the hood in SQLite

- **Replace deletes then inserts** – Not a true update

- **Replace triggers DELETE and INSERT triggers** – If you have them

- **Replace can cascade delete** – With foreign key constraints

- **Replace resets auto-increment** – When replacing primary keys

- **Replace is atomic** – Happens in one database operation

- **Replace is not SQL-standard** – It's a SQLite extension

- **Replace can be used with returning** – Returns inserted data

---

