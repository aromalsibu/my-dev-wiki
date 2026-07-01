## Companions

**Understanding Drift's companion classes for inserts and updates**

---

# What is it?

**Companion Classes** are generated classes in Drift that provide a type-safe way to insert, update, and delete records. They use the `Value<T>` wrapper to handle optional fields, default values, and partial updates. Every table automatically gets a companion class named `{TableName}Companion`.

> **Think of Companions like "order forms"** – you fill in the fields you want to set, leave others blank to use defaults, and submit it to create or update a record.

```dart
// 👇 Using a companion
await into(users).insert(
  UsersCompanion.insert(
    name: 'John Doe',           // Required field
    email: 'john@example.com',  // Required field
    age: Value(25),             // Optional field (provided)
    // isActive: Value(true)    // Optional field (uses default)
  ),
);

// Generated Companion (simplified)
class UsersCompanion extends UpdateCompanion<User> {
  final Value<int> id;
  final Value<String> name;
  final Value<String> email;
  final Value<int?> age;
  final Value<bool> isActive;
  
  const UsersCompanion({
    this.id = const Value.absent(),
    this.name = const Value.absent(),
    this.email = const Value.absent(),
    this.age = const Value.absent(),
    this.isActive = const Value.absent(),
  });
  
  UsersCompanion.insert({
    this.id = const Value.absent(),
    required String name,
    required String email,
    this.age = const Value.absent(),
    this.isActive = const Value.absent(),
  }) : super.insert();
}
```

> **What's happening here?**
> - **Companion Class** – Generated for each table
> - **Value<T>** – Wraps optional and default values
> - **insert constructor** – For creating new records
> - **Absent values** – Use default or skip during update

---

# Why does it exist?

- **Type Safety** – All fields are strongly typed
- **Partial Updates** – Update only specific fields
- **Default Handling** – Automatically use defaults
- **Null Handling** – Explicitly set NULL values
- **IDE Support** – Autocomplete and validation
- **Error Prevention** – Required fields must be provided

---

# Companion Structure

> **Understanding the generated companion**

```dart
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  IntColumn get age => integer().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// Generated UsersCompanion (complete)
class UsersCompanion extends UpdateCompanion<User> {
  // 👇 All fields as Value<T>
  final Value<int> id;
  final Value<String> name;
  final Value<String> email;
  final Value<int?> age;
  final Value<bool> isActive;
  final Value<DateTime> createdAt;

  // 👇 Default constructor (all absent)
  const UsersCompanion({
    this.id = const Value.absent(),
    this.name = const Value.absent(),
    this.email = const Value.absent(),
    this.age = const Value.absent(),
    this.isActive = const Value.absent(),
    this.createdAt = const Value.absent(),
  });

  // 👇 Insert constructor (required fields)
  UsersCompanion.insert({
    this.id = const Value.absent(),
    required String name,
    required String email,
    this.age = const Value.absent(),
    this.isActive = const Value.absent(),
    this.createdAt = const Value.absent(),
  }) : super.insert();

  // 👇 Methods for serialization
  @override
  Map<String, Expression> toColumns(bool nullToAbsent) {
    // ... internal mapping
  }
}
```

---

# Insert Operations

> **Using companions for inserts**

## Basic Insert

```dart
// 👇 Insert with required fields only
await into(users).insert(
  UsersCompanion.insert(
    name: 'John Doe',
    email: 'john@example.com',
    // age: uses default (NULL)
    // isActive: uses default (true)
    // createdAt: uses default (current time)
  ),
);

// Generated SQL:
// INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com')
// (Defaults used for other columns)
```

## Insert with All Fields

```dart
// 👇 Insert with all fields explicitly
await into(users).insert(
  UsersCompanion.insert(
    name: 'Jane Smith',
    email: 'jane@example.com',
    age: Value(28),
    isActive: Value(true),
    createdAt: Value(DateTime(2024, 1, 1)),
  ),
);

// Generated SQL:
// INSERT INTO users (name, email, age, is_active, created_at) 
// VALUES ('Jane Smith', 'jane@example.com', 28, 1, 1704067200)
```

## Insert with Explicit NULL

```dart
// 👇 Insert with NULL for optional field
await into(users).insert(
  UsersCompanion.insert(
    name: 'Bob Wilson',
    email: 'bob@example.com',
    age: const Value(null), // 👈 Explicitly set NULL
  ),
);

// Generated SQL:
// INSERT INTO users (name, email, age) VALUES ('Bob Wilson', 'bob@example.com', NULL)
```

## Insert with Multiple Records

```dart
// 👇 Batch insert
await into(users).insertAll([
  UsersCompanion.insert(
    name: 'User 1',
    email: 'user1@example.com',
  ),
  UsersCompanion.insert(
    name: 'User 2',
    email: 'user2@example.com',
  ),
  UsersCompanion.insert(
    name: 'User 3',
    email: 'user3@example.com',
    age: Value(30),
  ),
]);
```

---

# Update Operations

> **Using companions for updates**

## Full Update

```dart
// 👇 Update all fields
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'),
    email: Value('john.updated@example.com'),
    age: Value(26),
    isActive: Value(false),
    createdAt: Value(DateTime(2024, 1, 1)),
  ));

// Generated SQL:
// UPDATE users SET 
//   name = 'John Updated',
//   email = 'john.updated@example.com',
//   age = 26,
//   is_active = 0,
//   created_at = 1704067200
// WHERE id = 1
```

## Partial Update

```dart
// 👇 Update only specific fields
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    name: Value('John Updated'), // 👈 Only update name
    // Other fields use Value.absent()
  ));

// Generated SQL:
// UPDATE users SET name = 'John Updated' WHERE id = 1
// (Other columns unchanged)
```

## Conditional Update

```dart
// 👇 Update based on condition
await (update(users)..where((u) => u.isActive.equals(true)))
  .write(UsersCompanion(
    isActive: Value(false),
    updatedAt: Value(DateTime.now()), // If you had an updatedAt column
  ));
```

## Update with NULL

```dart
// 👇 Set field to NULL
await (update(users)..where((u) => u.id.equals(1)))
  .write(UsersCompanion(
    age: const Value(null), // 👈 Set age to NULL
  ));
```

---

# Delete Operations

> **Deleting records (companions are optional)**

## Simple Delete

```dart
// 👇 Delete by condition
await (delete(users)..where((u) => u.id.equals(1))).go();

// 👇 Delete all
await delete(users).go();

// 👇 Delete with multiple conditions
await (delete(users)
  ..where((u) => u.isActive.equals(false))
  ..where((u) => u.age.isBiggerOrEqualValue(100)))
  .go();
```

## Delete with Returning

```dart
// 👇 Delete and return deleted rows
final deleted = await (delete(users)..where((u) => u.id.equals(1)))
  .returning();

print('Deleted user: ${deleted.first}');
```

---

# Advanced Companion Patterns

> **Complex patterns with companions**

## Pattern 1: Conditional Insert

```dart
Future<int> createOrUpdateUser({
  required int id,
  required String name,
  String? email,
  int? age,
  bool? isActive,
}) async {
  // Check if user exists
  final existing = await (select(users)..where((u) => u.id.equals(id)))
      .getSingleOrNull();
  
  if (existing == null) {
    // Insert new
    return await into(users).insert(
      UsersCompanion.insert(
        name: name,
        email: email ?? '$name@example.com',
        age: Value(age),
        isActive: Value(isActive ?? true),
      ),
    );
  } else {
    // Update existing
    await (update(users)..where((u) => u.id.equals(id)))
      .write(UsersCompanion(
        name: Value(name),
        email: email != null ? Value(email) : const Value.absent(),
        age: age != null ? Value(age) : const Value.absent(),
        isActive: isActive != null ? Value(isActive) : const Value.absent(),
      ));
    return id;
  }
}
```

## Pattern 2: Bulk Update with Companion

```dart
Future<void> bulkUpdateUsers(List<UserUpdate> updates) async {
  await transaction(() async {
    for (final update in updates) {
      await (update(users)..where((u) => u.id.equals(update.id)))
        .write(UsersCompanion(
          name: update.name != null ? Value(update.name!) : const Value.absent(),
          age: update.age != null ? Value(update.age!) : const Value.absent(),
          isActive: update.isActive != null 
              ? Value(update.isActive!) 
              : const Value.absent(),
        ));
    }
  });
}

class UserUpdate {
  final int id;
  final String? name;
  final int? age;
  final bool? isActive;
  
  UserUpdate({required this.id, this.name, this.age, this.isActive});
}
```

## Pattern 3: Upsert (Insert or Update)

```dart
Future<int> upsertUser(UsersCompanion companion) async {
  // 👇 Try to find user by email
  final email = companion.email.value;
  if (email == null) {
    return await into(users).insert(companion);
  }
  
  final existing = await (select(users)..where((u) => u.email.equals(email)))
      .getSingleOrNull();
  
  if (existing == null) {
    // Insert
    return await into(users).insert(companion);
  } else {
    // Update
    await (update(users)..where((u) => u.id.equals(existing.id)))
      .write(companion);
    return existing.id;
  }
}

// Usage
final id = await upsertUser(
  UsersCompanion.insert(
    name: 'John Doe',
    email: 'john@example.com',
    age: Value(30),
  ),
);
```

## Pattern 4: Safe Update with Validation

```dart
Future<void> safeUpdateUser(int id, UsersCompanion companion) async {
  // 👇 Validate before update
  if (companion.name.value != null) {
    final name = companion.name.value!;
    if (name.length < 2) {
      throw Exception('Name must be at least 2 characters');
    }
  }
  
  if (companion.email.value != null) {
    final email = companion.email.value!;
    if (!email.contains('@')) {
      throw Exception('Invalid email format');
    }
  }
  
  if (companion.age.value != null) {
    final age = companion.age.value!;
    if (age < 0 || age > 150) {
      throw Exception('Invalid age range');
    }
  }
  
  // 👇 Perform update
  await (update(users)..where((u) => u.id.equals(id)))
    .write(companion);
}

// Usage
await safeUpdateUser(1, UsersCompanion(
  name: Value('John'),
  age: Value(25),
));
```

---

# Companion with Relations

> **Using companions with relations**

```dart
// lib/database/tables/posts.dart
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  IntColumn get userId => integer().references(Users, #id)();
  BoolColumn get isPublished => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/database.dart
@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  // 👇 Create user and post in one transaction
  Future<int> createUserWithPost({
    required String userName,
    required String userEmail,
    required String postTitle,
    required String postContent,
  }) async {
    return await transaction(() async {
      // 1️⃣ Create user
      final userId = await into(users).insert(
        UsersCompanion.insert(
          name: userName,
          email: userEmail,
        ),
      );
      
      // 2️⃣ Create post for user
      await into(posts).insert(
        PostsCompanion.insert(
          title: postTitle,
          content: postContent,
          userId: userId,
        ),
      );
      
      return userId;
    });
  }

  // 👇 Update user and their posts
  Future<void> updateUserWithPosts(int userId, {
    required String userName,
    required String postTitle,
  }) async {
    await transaction(() async {
      // 1️⃣ Update user
      await (update(users)..where((u) => u.id.equals(userId)))
        .write(UsersCompanion(name: Value(userName)));
      
      // 2️⃣ Update user's posts
      await (update(posts)..where((p) => p.userId.equals(userId)))
        .write(PostsCompanion(title: Value(postTitle)));
    });
  }

  // 👇 Delete user and their posts (cascade)
  Future<void> deleteUserWithPosts(int userId) async {
    await transaction(() async {
      // Delete user - ON DELETE CASCADE will delete posts
      await (delete(users)..where((u) => u.id.equals(userId))).go();
    });
  }
}
```

---

# Real-World Example

> **Complete e-commerce companion usage**

```dart
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get fullName => text().nullable()();
  TextColumn get passwordHash => text()();
  IntColumn get age => integer().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  BoolColumn get isVerified => boolean().withDefault(const Constant(false))();
  BoolColumn get isAdmin => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}

// lib/database/tables/products.dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get sku => text().unique()();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  RealColumn get price => real().customConstraint('CHECK (price >= 0)')();
  IntColumn get stock => integer().withDefault(const Constant(0))();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
}

// lib/database/tables/orders.dart
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get orderNumber => text().unique()();
  IntColumn get userId => integer().references(Users, #id)();
  RealColumn get total => real()();
  TextColumn get status => text()();
  DateTimeColumn get orderDate => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get shippedDate => dateTime().nullable()();
}

// lib/database/database.dart
@DriftDatabase(tables: [Users, Products, Orders])
class EcommerceDatabase extends _$EcommerceDatabase {
  EcommerceDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Create user
  Future<int> createUser({
    required String username,
    required String email,
    required String passwordHash,
    String? fullName,
    int? age,
    bool isAdmin = false,
  }) async {
    return await into(users).insert(
      UsersCompanion.insert(
        username: username,
        email: email,
        passwordHash: passwordHash,
        fullName: Value(fullName),
        age: Value(age),
        isAdmin: Value(isAdmin),
        // isActive: true (default)
        // isVerified: false (default)
      ),
    );
  }

  // 👇 Update user
  Future<void> updateUser(
    int userId, {
    String? username,
    String? email,
    String? fullName,
    int? age,
    bool? isActive,
    bool? isVerified,
  }) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        username: username != null ? Value(username) : const Value.absent(),
        email: email != null ? Value(email) : const Value.absent(),
        fullName: fullName != null ? Value(fullName) : const Value.absent(),
        age: age != null ? Value(age) : const Value.absent(),
        isActive: isActive != null ? Value(isActive) : const Value.absent(),
        isVerified: isVerified != null ? Value(isVerified) : const Value.absent(),
        updatedAt: Value(DateTime.now()),
      ));
  }

  // 👇 Update user by partial fields only
  Future<void> updateUserProfile(int userId, {
    String? fullName,
    int? age,
  }) async {
    final companion = UsersCompanion(
      fullName: fullName != null ? Value(fullName) : const Value.absent(),
      age: age != null ? Value(age) : const Value.absent(),
      updatedAt: Value(DateTime.now()),
    );
    
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(companion);
  }

  // 👇 Create product
  Future<int> createProduct({
    required String sku,
    required String name,
    String? description,
    required double price,
    int stock = 0,
  }) async {
    return await into(products).insert(
      ProductsCompanion.insert(
        sku: sku,
        name: name,
        description: Value(description),
        price: price,
        stock: Value(stock),
      ),
    );
  }

  // 👇 Update product stock
  Future<void> updateProductStock(int productId, int newStock) async {
    await (update(products)..where((p) => p.id.equals(productId)))
      .write(ProductsCompanion(
        stock: Value(newStock),
        updatedAt: Value(DateTime.now()),
      ));
  }

  // 👇 Create order with automatic total calculation
  Future<int> createOrder(
    int userId,
    List<OrderItemInput> items,
  ) async {
    return await transaction(() async {
      // Calculate total
      double total = 0;
      for (final item in items) {
        final product = await (select(products)
          ..where((p) => p.id.equals(item.productId)))
          .getSingle();
        total += product.price * item.quantity;
      }

      // Create order
      final orderId = await into(orders).insert(
        OrdersCompanion.insert(
          orderNumber: 'ORD-${DateTime.now().millisecondsSinceEpoch}',
          userId: userId,
          total: total,
          status: 'pending',
        ),
      );

      // Update product stock (reduce)
      for (final item in items) {
        final product = await (select(products)
          ..where((p) => p.id.equals(item.productId)))
          .getSingle();
        
        await (update(products)..where((p) => p.id.equals(product.id)))
          .write(ProductsCompanion(
            stock: Value(product.stock - item.quantity),
            updatedAt: Value(DateTime.now()),
          ));
      }

      return orderId;
    });
  }

  // 👇 Cancel order (restore stock)
  Future<void> cancelOrder(int orderId) async {
    await transaction(() async {
      // 1️⃣ Get order items (simplified - you'd have an OrderItems table)
      // 2️⃣ Restore stock
      // 3️⃣ Update order status
      
      await (update(orders)..where((o) => o.id.equals(orderId)))
        .write(OrdersCompanion(
          status: Value('cancelled'),
        ));
    });
  }

  // 👇 Complex upsert
  Future<int> upsertProduct(ProductsCompanion companion) async {
    final sku = companion.sku.value;
    if (sku == null) throw Exception('SKU is required');
    
    final existing = await (select(products)
      ..where((p) => p.sku.equals(sku)))
      .getSingleOrNull();
    
    if (existing == null) {
      // Insert new product
      return await into(products).insert(companion);
    } else {
      // Update existing product
      await (update(products)..where((p) => p.id.equals(existing.id)))
        .write(companion);
      return existing.id;
    }
  }

  // 👇 Safe delete with validation
  Future<void> safeDeleteUser(int userId) async {
    // Check if user has orders
    final hasOrders = await (select(orders)
      ..where((o) => o.userId.equals(userId)))
      .count() > 0;
    
    if (hasOrders) {
      throw Exception('Cannot delete user with existing orders');
    }
    
    await (delete(users)..where((u) => u.id.equals(userId))).go();
  }
}

class OrderItemInput {
  final int productId;
  final int quantity;
  
  OrderItemInput({required this.productId, required this.quantity});
}
```

---

# Best Practices

- **Use `insert` constructor** – For creating new records
- **Use `Value.absent()`** – To skip fields in updates
- **Use `Value(null)`** – To explicitly set NULL
- **Use transactions** – For multiple operations
- **Validate before update** – Use companion with validation
- **Use `copyWith`** – For data class modifications
- **Document companions** – Explain required fields
- **Test companions** – Verify insert/update behavior

---

# Common Mistakes

## Mistake 1: Using insert constructor for updates

Wrong:
```dart
// 🚫 Using insert constructor for updates
await update(users).write(
  UsersCompanion.insert( // Wrong constructor
    name: 'John',
  ),
);
```

Correct:
```dart
// ✅ Use default constructor for updates
await update(users).write(
  UsersCompanion(
    name: Value('John'),
  ),
);
```

## Mistake 2: Forgetting `Value` wrapper

Wrong:
```dart
// 🚫 Missing Value wrapper
await update(users).write(
  UsersCompanion(
    name: 'John', // ❌ Should be Value('John')
  ),
);
```

Correct:
```dart
// ✅ Use Value wrapper
await update(users).write(
  UsersCompanion(
    name: Value('John'),
  ),
);
```

## Mistake 3: Using `Value.absent()` in inserts

Wrong:
```dart
// 🚫 Absent doesn't work in insert
await into(users).insert(
  UsersCompanion.insert(
    name: Value.absent(), // ❌ Required field can't be absent
  ),
);
```

Correct:
```dart
// ✅ Provide all required fields
await into(users).insert(
  UsersCompanion.insert(
    name: 'John', // Required field
    email: 'john@example.com', // Required field
  ),
);
```

---

# Summary

| Method | Purpose | Use Case |
|--------|---------|----------|
| **insert constructor** | Create new record | `UsersCompanion.insert()` |
| **default constructor** | Update existing | `UsersCompanion()` |
| **Value.absent()** | Skip field | Updates only |
| **Value(value)** | Set field | Inserts and updates |
| **Value(null)** | Set NULL | Optional fields |

---

# Next Steps

Now you understand companions, let's dive deeper:

- [Value<T>](link) – Deep dive into Value wrapper
- [Insertable](link) – Custom insert logic
- [CRUD Operations](link) – Complete CRUD guide

---

# Did You Know?

- **Companions are generated** – Don't manually edit them

- **Insert constructor validates required fields** – At compile-time

- **Absent values in inserts use defaults** – Not NULL

- **Default constructor allows partial updates** – Only provided fields update

- **Value.absent() skips the field entirely** – In UPDATE statements

- **Value(null) explicitly sets NULL** – Even in optional fields

- **Companions are immutable** – Use `copyWith` to modify

- **Companions can be used in transactions** – Atomic operations

---

