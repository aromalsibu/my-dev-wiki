## Upsert

**Insert or update records in Drift with conflict resolution**

---

# What is it?

**Upsert** (INSERT + UPDATE) is an operation that either inserts a new record or updates an existing one if a conflict occurs (usually on a unique constraint or primary key). Drift provides the `DoUpdate` and `DoNothing` conflict resolution strategies for fine-grained control.

> **Think of Upsert like "update or insert"** – you try to add a new record, but if one already exists with the same key, you update it instead. It's like saying "If you're already here, update yourself; if not, come in!"

```dart
// 👇 Upsert with conflict resolution
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1), // Existing ID
    name: 'John Updated',
    email: 'john@example.com',
  ),
  onConflict: DoUpdate(
    target: users.id, // 👈 Check conflict on id
    update: UsersCompanion(
      name: Value('John Updated'),
      email: Value('john@example.com'),
    ),
  ),
);

// What happens:
// If user with ID 1 exists: UPDATE user
// If user with ID 1 doesn't exist: INSERT new user
```

> **What's happening here?**
> - **`DoUpdate`** – Update on conflict
> - **`target`** – Which column(s) to check for conflicts
> - **`update`** – What to update when conflict occurs
> - **Atomic operation** – All or nothing

---

# Why does it exist?

- **Idempotent Operations** – Safe to run multiple times
- **Data Synchronization** – Sync external data sources
- **Cache Updates** – Update cached data
- **Avoid Duplicates** – Prevent duplicate records
- **Data Migration** – Insert or update existing data
- **Conflict Resolution** – Handle duplicates automatically

---

# Basic Upsert Operations

> **Simple upsert operations**

## Upsert by Primary Key

```dart
// 👇 Upsert user by ID
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1), // 👈 Primary key
    name: 'John Doe',
    email: 'john.doe@example.com',
    age: Value(30),
  ),
  onConflict: DoUpdate(
    target: users.id,
    update: UsersCompanion(
      name: Value('John Doe'),
      email: Value('john.doe@example.com'),
      age: Value(30),
    ),
  ),
);

// Generated SQL:
// INSERT INTO users (id, name, email, age) 
// VALUES (1, 'John Doe', 'john.doe@example.com', 30)
// ON CONFLICT(id) DO UPDATE 
// SET name = 'John Doe', email = 'john.doe@example.com', age = 30
```

---

## Upsert by Unique Constraint

```dart
// 👇 Upsert by unique email
await into(users).insert(
  UsersCompanion.insert(
    email: 'john@example.com', // 👈 Unique constraint
    name: 'John Updated',
    age: Value(25),
  ),
  onConflict: DoUpdate(
    target: users.email,
    update: UsersCompanion(
      name: Value('John Updated'),
      age: Value(25),
    ),
  ),
);

// If email 'john@example.com' exists: UPDATE that record
// If email 'john@example.com' doesn't exist: INSERT new record
```

---

## Upsert with Partial Update

```dart
// 👇 Only update specific fields on conflict
await into(users).insert(
  UsersCompanion.insert(
    email: 'john@example.com',
    name: 'John Updated',
    age: Value(25),
  ),
  onConflict: DoUpdate(
    target: users.email,
    update: UsersCompanion(
      name: Value('John Updated'), // Only update name
      // Age and other fields remain unchanged
    ),
  ),
);
```

---

## Upsert with DoNothing (Skip on Conflict)

```dart
// 👇 Insert or ignore (skip if exists)
await into(users).insert(
  UsersCompanion.insert(
    email: 'john@example.com',
    name: 'John Doe',
  ),
  onConflict: DoNothing(),
);

// If email exists: SKIP (no changes)
// If email doesn't exist: INSERT new record
```

---

# Advanced Upsert Patterns

> **Complex upsert operations**

## Pattern 1: Upsert with Timestamp

```dart
// 👇 Upsert with updated_at timestamp
await into(users).insert(
  UsersCompanion.insert(
    email: 'john@example.com',
    name: 'John',
    age: Value(30),
  ),
  onConflict: DoUpdate(
    target: users.email,
    update: UsersCompanion(
      name: Value('John'),
      age: Value(30),
      updatedAt: Value(DateTime.now()), // Update timestamp
    ),
  ),
);
```

---

## Pattern 2: Upsert with Excluded Values

```dart
// 👇 Using excluded values (SQLite feature)
// Drift provides helper methods for this
await into(users).insert(
  UsersCompanion.insert(
    email: 'john@example.com',
    name: 'John',
    age: Value(30),
    lastSync: Value(DateTime.now()),
  ),
  onConflict: DoUpdate(
    target: users.email,
    update: UsersCompanion(
      name: Value('John'),
      age: Value(30),
      lastSync: Value(DateTime.now()),
      // You can reference the inserted values
    ),
  ),
);
```

---

## Pattern 3: Bulk Upsert

```dart
// 👇 Bulk upsert with batch
Future<void> bulkUpsertUsers(List<UserData> users) async {
  await into(users).batch((batch) {
    for (final user in users) {
      batch.insert(
        UsersCompanion.insert(
          email: user.email,
          name: user.name,
          age: Value(user.age),
        ),
        onConflict: DoUpdate(
          target: users.email,
          update: UsersCompanion(
            name: Value(user.name),
            age: Value(user.age),
            updatedAt: Value(DateTime.now()),
          ),
        ),
      );
    }
  });
}
```

---

## Pattern 4: Upsert with Returning

```dart
// 👇 Upsert and return the record
final result = await into(users).insertReturning(
  UsersCompanion.insert(
    email: 'john@example.com',
    name: 'John Doe',
    age: Value(30),
  ),
  onConflict: DoUpdate(
    target: users.email,
    update: UsersCompanion(
      name: Value('John Doe'),
      age: Value(30),
      updatedAt: Value(DateTime.now()),
    ),
  ),
);

print('Upserted user: ${result.name}, ID: ${result.id}');
```

---

## Pattern 5: Conditional Upsert

```dart
// 👇 Upsert with condition (only update if condition met)
class AppDatabase extends _$AppDatabase {
  // Upsert only if new data is newer
  Future<void> upsertIfNewer({
    required String email,
    required String name,
    required DateTime lastSync,
  }) async {
    await into(users).insert(
      UsersCompanion.insert(
        email: email,
        name: name,
        lastSync: Value(lastSync),
      ),
      onConflict: DoUpdate(
        target: users.email,
        update: UsersCompanion(
          name: Value(name),
          lastSync: Value(lastSync),
          updatedAt: Value(DateTime.now()),
        ),
        // Only update if new sync is newer
        where: (u) => u.lastSync < const Variable(lastSync),
      ),
    );
  }
}
```

---

# Real-World Example

> **Complete e-commerce upsert system**

```dart
// lib/database/database.dart
@DriftDatabase(tables: [Users, Products, Orders, OrderItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // ==================== USER UPSERTS ====================
  
  // 👇 Basic user upsert
  Future<void> upsertUser({
    required String email,
    required String name,
    int? age,
    bool isActive = true,
  }) async {
    await into(users).insert(
      UsersCompanion.insert(
        email: email,
        name: name,
        age: Value(age),
        isActive: Value(isActive),
        updatedAt: Value(DateTime.now()),
      ),
      onConflict: DoUpdate(
        target: users.email,
        update: UsersCompanion(
          name: Value(name),
          age: Value(age),
          isActive: Value(isActive),
          updatedAt: Value(DateTime.now()),
        ),
      ),
    );
  }
  
  // 👇 Upsert with returning
  Future<User> upsertUserReturning({
    required String email,
    required String name,
    int? age,
  }) async {
    return await into(users).insertReturning(
      UsersCompanion.insert(
        email: email,
        name: name,
        age: Value(age),
        updatedAt: Value(DateTime.now()),
      ),
      onConflict: DoUpdate(
        target: users.email,
        update: UsersCompanion(
          name: Value(name),
          age: Value(age),
          updatedAt: Value(DateTime.now()),
        ),
      ),
    );
  }
  
  // 👇 Upsert with condition (only update if newer)
  Future<void> syncUserFromAPI({
    required int id,
    required String email,
    required String name,
    required DateTime lastSync,
  }) async {
    await into(users).insert(
      UsersCompanion.insert(
        id: Value(id),
        email: email,
        name: name,
        lastSync: Value(lastSync),
        updatedAt: Value(DateTime.now()),
      ),
      onConflict: DoUpdate(
        target: users.id,
        update: UsersCompanion(
          email: Value(email),
          name: Value(name),
          lastSync: Value(lastSync),
          updatedAt: Value(DateTime.now()),
        ),
        where: (u) => u.lastSync < const Variable(lastSync),
      ),
    );
  }
  
  // 👇 Bulk user upsert
  Future<void> bulkUpsertUsers(List<UserUpsertData> users) async {
    await into(users).batch((batch) {
      for (final user in users) {
        batch.insert(
          UsersCompanion.insert(
            email: user.email,
            name: user.name,
            age: Value(user.age),
            isActive: Value(user.isActive),
            updatedAt: Value(DateTime.now()),
          ),
          onConflict: DoUpdate(
            target: users.email,
            update: UsersCompanion(
              name: Value(user.name),
              age: Value(user.age),
              isActive: Value(user.isActive),
              updatedAt: Value(DateTime.now()),
            ),
          ),
        );
      }
    });
  }

  // ==================== PRODUCT UPSERTS ====================
  
  // 👇 Product upsert by SKU
  Future<void> upsertProduct({
    required String sku,
    required String name,
    required double price,
    int stock = 0,
    String? description,
  }) async {
    await into(products).insert(
      ProductsCompanion.insert(
        sku: sku,
        name: name,
        price: price,
        stock: Value(stock),
        description: Value(description),
        updatedAt: Value(DateTime.now()),
      ),
      onConflict: DoUpdate(
        target: products.sku,
        update: ProductsCompanion(
          name: Value(name),
          price: Value(price),
          stock: Value(stock),
          description: Value(description),
          updatedAt: Value(DateTime.now()),
        ),
      ),
    );
  }
  
  // 👇 Upsert product with price history
  Future<void> upsertProductWithHistory({
    required String sku,
    required String name,
    required double price,
    int stock = 0,
  }) async {
    // Get existing product if any
    final existing = await (select(products)
      ..where((p) => p.sku.equals(sku)))
      .getSingleOrNull();
    
    final oldPrice = existing?.price;
    
    // Upsert product
    await into(products).insert(
      ProductsCompanion.insert(
        sku: sku,
        name: name,
        price: price,
        stock: Value(stock),
        updatedAt: Value(DateTime.now()),
      ),
      onConflict: DoUpdate(
        target: products.sku,
        update: ProductsCompanion(
          name: Value(name),
          price: Value(price),
          stock: Value(stock),
          updatedAt: Value(DateTime.now()),
        ),
      ),
    );
    
    // Record price change
    if (oldPrice != null && oldPrice != price) {
      await _recordPriceChange(sku, oldPrice, price);
    }
  }
  
  Future<void> _recordPriceChange(String sku, double oldPrice, double newPrice) async {
    await customInsert(
      '''
      INSERT INTO price_history (sku, old_price, new_price, changed_at)
      VALUES (?, ?, ?, ?)
      ''',
      variables: [
        Variable.withString(sku),
        Variable.withDouble(oldPrice),
        Variable.withDouble(newPrice),
        Variable.withString(DateTime.now().toIso8601String()),
      ],
    ).go();
  }
  
  // 👇 Bulk product upsert from catalog sync
  Future<void> syncProductCatalog(List<ProductSyncData> products) async {
    await into(products).batch((batch) {
      for (final product in products) {
        batch.insert(
          ProductsCompanion.insert(
            sku: product.sku,
            name: product.name,
            price: product.price,
            stock: Value(product.stock),
            description: Value(product.description),
            isActive: Value(product.isActive),
            updatedAt: Value(DateTime.now()),
          ),
          onConflict: DoUpdate(
            target: products.sku,
            update: ProductsCompanion(
              name: Value(product.name),
              price: Value(product.price),
              stock: Value(product.stock),
              description: Value(product.description),
              isActive: Value(product.isActive),
              updatedAt: Value(DateTime.now()),
            ),
          ),
        );
      }
    });
  }

  // ==================== INVENTORY UPSERTS ====================
  
  // 👇 Upsert inventory with stock adjustment
  Future<void> upsertInventory({
    required String sku,
    required int adjustment,
    required String reason,
  }) async {
    // Get current stock
    final current = await (select(products)
      ..where((p) => p.sku.equals(sku)))
      .getSingleOrNull();
    
    final currentStock = current?.stock ?? 0;
    final newStock = currentStock + adjustment;
    
    if (newStock < 0) {
      throw Exception('Insufficient stock for $sku');
    }
    
    // Upsert product with adjusted stock
    await into(products).insert(
      ProductsCompanion.insert(
        sku: sku,
        name: current?.name ?? 'Unknown',
        price: current?.price ?? 0.0,
        stock: Value(newStock),
        updatedAt: Value(DateTime.now()),
      ),
      onConflict: DoUpdate(
        target: products.sku,
        update: ProductsCompanion(
          stock: Value(newStock),
          updatedAt: Value(DateTime.now()),
        ),
      ),
    );
    
    // Record inventory adjustment
    await _recordInventoryAdjustment(sku, adjustment, reason, currentStock, newStock);
  }
  
  Future<void> _recordInventoryAdjustment(
    String sku,
    int adjustment,
    String reason,
    int oldStock,
    int newStock,
  ) async {
    await customInsert(
      '''
      INSERT INTO inventory_log (sku, adjustment, reason, old_stock, new_stock, created_at)
      VALUES (?, ?, ?, ?, ?, ?)
      ''',
      variables: [
        Variable.withString(sku),
        Variable.withInt(adjustment),
        Variable.withString(reason),
        Variable.withInt(oldStock),
        Variable.withInt(newStock),
        Variable.withString(DateTime.now().toIso8601String()),
      ],
    ).go();
  }

  // ==================== ORDER UPSERTS ====================
  
  // 👇 Upsert order with status update
  Future<void> upsertOrderStatus({
    required int orderId,
    required String newStatus,
  }) async {
    final validStatuses = ['pending', 'processing', 'paid', 'shipped', 'delivered', 'cancelled'];
    if (!validStatuses.contains(newStatus)) {
      throw Exception('Invalid status: $newStatus');
    }
    
    await into(orders).insert(
      OrdersCompanion.insert(
        id: Value(orderId),
        orderNumber: Value('ORD-$orderId'),
        status: Value(newStatus),
        updatedAt: Value(DateTime.now()),
        // Provide other fields with defaults
        userId: Value(0), // Should be updated separately
        total: Value(0.0),
        shippingAddress: Value(''),
        billingAddress: Value(''),
        paymentMethod: Value(''),
        paymentStatus: Value('unpaid'),
        isPaid: const Value(false),
        isShipped: const Value(false),
        isDelivered: const Value(false),
      ),
      onConflict: DoUpdate(
        target: orders.id,
        update: OrdersCompanion(
          status: Value(newStatus),
          shippedDate: newStatus == 'shipped' 
              ? Value(DateTime.now()) 
              : const Value.absent(),
          deliveredDate: newStatus == 'delivered'
              ? Value(DateTime.now())
              : const Value.absent(),
          isPaid: newStatus == 'paid' || newStatus == 'shipped' || newStatus == 'delivered'
              ? const Value(true)
              : const Value.absent(),
          isShipped: newStatus == 'shipped' || newStatus == 'delivered'
              ? const Value(true)
              : const Value.absent(),
          isDelivered: newStatus == 'delivered'
              ? const Value(true)
              : const Value.absent(),
          updatedAt: Value(DateTime.now()),
        ),
      ),
    );
  }
  
  // 👇 Upsert order from API
  Future<void> upsertOrderFromAPI({
    required int id,
    required int userId,
    required String orderNumber,
    required double total,
    required String status,
    required DateTime orderDate,
  }) async {
    await into(orders).insert(
      OrdersCompanion.insert(
        id: Value(id),
        userId: userId,
        orderNumber: orderNumber,
        total: total,
        status: Value(status),
        orderDate: Value(orderDate),
        updatedAt: Value(DateTime.now()),
        // Defaults for required fields
        shippingAddress: Value(''),
        billingAddress: Value(''),
        paymentMethod: Value(''),
        paymentStatus: Value('unpaid'),
        isPaid: const Value(false),
        isShipped: const Value(false),
        isDelivered: const Value(false),
      ),
      onConflict: DoUpdate(
        target: orders.id,
        update: OrdersCompanion(
          userId: Value(userId),
          total: Value(total),
          status: Value(status),
          updatedAt: Value(DateTime.now()),
        ),
      ),
    );
  }

  // ==================== CACHE MANAGEMENT ====================
  
  // 👇 Upsert cache entries
  Future<void> upsertCache({
    required String key,
    required String value,
    Duration? ttl,
  }) async {
    final expiresAt = ttl != null 
        ? DateTime.now().add(ttl)
        : null;
    
    await into(cache).insert(
      CacheCompanion.insert(
        key: key,
        value: value,
        expiresAt: Value(expiresAt),
        updatedAt: Value(DateTime.now()),
      ),
      onConflict: DoUpdate(
        target: cache.key,
        update: CacheCompanion(
          value: Value(value),
          expiresAt: Value(expiresAt),
          updatedAt: Value(DateTime.now()),
        ),
      ),
    );
  }
  
  // 👇 Upsert with TTL refresh
  Future<void> refreshCache(String key, String newValue) async {
    // Update or insert with extended TTL
    await upsertCache(
      key: key,
      value: newValue,
      ttl: Duration(hours: 1),
    );
  }
  
  // 👇 Upsert multiple cache entries
  Future<void> bulkUpsertCache(Map<String, String> entries) async {
    await into(cache).batch((batch) {
      for (final entry in entries.entries) {
        batch.insert(
          CacheCompanion.insert(
            key: entry.key,
            value: entry.value,
            expiresAt: Value(DateTime.now().add(Duration(hours: 1))),
            updatedAt: Value(DateTime.now()),
          ),
          onConflict: DoUpdate(
            target: cache.key,
            update: CacheCompanion(
              value: Value(entry.value),
              expiresAt: Value(DateTime.now().add(Duration(hours: 1))),
              updatedAt: Value(DateTime.now()),
            ),
          ),
        );
      }
    });
  }
}

// ==================== DATA CLASSES ====================

class UserUpsertData {
  final String email;
  final String name;
  final int? age;
  final bool isActive;
  
  UserUpsertData({
    required this.email,
    required this.name,
    this.age,
    this.isActive = true,
  });
}

class ProductSyncData {
  final String sku;
  final String name;
  final double price;
  final int stock;
  final String? description;
  final bool isActive;
  
  ProductSyncData({
    required this.sku,
    required this.name,
    required this.price,
    required this.stock,
    this.description,
    this.isActive = true,
  });
}
```

---

# Upsert vs Replace vs Update

> **When to use each operation**

```dart
// 👇 UPDATE: Modify specific fields
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'),
    // Other fields stay the same
  ));

// 👇 UPSERT: Insert or update
await into(users).insert(
  UsersCompanion.insert(
    email: 'john@example.com',
    name: 'John Updated',
  ),
  onConflict: DoUpdate(
    target: users.email,
    update: UsersCompanion(
      name: Value('John Updated'),
    ),
  ),
);

// 👇 REPLACE: Delete and insert entire record
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

# Best Practices

- **Use upsert for synchronization** – External data sources
- **Use DoUpdate for partial updates** – Only update changed fields
- **Use DoNothing for ignoring duplicates** – Avoid errors
- **Use unique constraints as target** – For conflict detection
- **Use timestamps** – Track when records are upserted
- **Use transactions** – For multiple upserts
- **Test upsert behavior** – Understand conflict handling
- **Use returning** – When you need the result

---

# Common Mistakes

## Mistake 1: Not specifying target for conflict

Wrong:
```dart
// 🚫 Missing target
await into(users).insert(
  UsersCompanion.insert(...),
  onConflict: DoUpdate(
    update: UsersCompanion(...), // No target specified!
  ),
);
```

Correct:
```dart
// ✅ Specify target column
await into(users).insert(
  UsersCompanion.insert(...),
  onConflict: DoUpdate(
    target: users.email, // 👈 Must specify target
    update: UsersCompanion(...),
  ),
);
```

## Mistake 2: Using upsert when update would work

Wrong:
```dart
// 🚫 Overkill for known record
await into(users).insert(
  UsersCompanion.insert(id: Value(1), ...),
  onConflict: DoUpdate(...),
);
```

Correct:
```dart
// ✅ Use update when record exists
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(...));
```

## Mistake 3: Not handling all constraints

Wrong:
```dart
// 🚫 Only checks one unique constraint
await into(users).insert(
  UsersCompanion.insert(
    id: Value(1),
    email: 'john@example.com',
  ),
  onConflict: DoUpdate(
    target: users.id, // Only checks ID
    update: UsersCompanion(...),
  ),
);
// Conflict on email would still fail
```

Correct:
```dart
// ✅ Check all unique constraints or use replace
// Or handle conflicts appropriately
await into(users).insert(
  UsersCompanion.insert(...),
  onConflict: DoUpdate(
    target: users.email, // Check more relevant constraint
    update: UsersCompanion(...),
  ),
);
```

---

# Summary

| Operation | Behavior | Best For |
|-----------|----------|----------|
| **Update** | Modify existing | Known records |
| **Upsert** | Insert or update | Unknown existence |
| **Replace** | Delete + Insert | Full refresh |

---

# Next Steps

Now you understand upsert, let's dive deeper:

- [Replace](link) – Insert or replace
- [Update](link) – Updating existing records
- [Delete](link) – Deleting records

---

# Did You Know?

- **Upsert is INSERT ON CONFLICT** – Under the hood in SQLite

- **Upsert is SQL-standard** – Supported by SQLite 3.24+

- **DoUpdate can use excluded values** – To reference inserted values

- **DoNothing is like INSERT OR IGNORE** – Skips on conflict

- **Upsert is atomic** – Happens in one database operation

- **Upsert can use WHERE** – For conditional updates

- **Upsert works with returning** – Returns the resulting record

- **Upsert is faster than REPLACE** – Doesn't delete and reinsert

---

