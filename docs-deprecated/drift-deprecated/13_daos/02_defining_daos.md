## Defining DAOs (Modern API)

**Creating type-safe DAOs with Drift's query builder**

---

# What is it?

**Defining DAOs** is the process of creating Data Access Object classes that encapsulate all database operations for a specific entity using Drift's **type-safe query builder**. Instead of writing raw SQL strings with `@Query`, you define methods that use the query builder API, giving you full type safety, IDE autocomplete, and compile-time validation.

> **Think of Defining DAOs like "creating a typed API client"** – you define methods with clear signatures, and Drift generates all the underlying implementation code. The methods you write are pure Dart with full type safety.

```dart
// 👇 Defining a modern DAO
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // 👇 Type-safe query (no SQL strings!)
  Future<List<User>> getActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .get();
  }

  // 👇 With parameters
  Future<User?> getUserById(int id) {
    return (select(users)
      ..where((u) => u.id.equals(id)))
      .getSingleOrNull();
  }

  // 👇 Reactive query
  Stream<List<User>> watchActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .watch();
  }
}
```

> **What's happening here?**
> - **`@DriftAccessor`** – Marks the class as a DAO
> - **`select(users)`** – Starts a type-safe query
> - **`.where()`** – Adds type-safe filter conditions
> - **`.get()`** – Executes and returns typed results
> - **`.watch()`** – Returns a reactive stream

---

# Why does it exist?

- **Type Safety** – Catch errors at compile-time
- **IDE Support** – Autocomplete and refactoring
- **Maintainability** – No SQL strings to maintain
- **Reusability** – Query builder methods are composable
- **Testability** – Easier to test and mock
- **Security** – Built-in SQL injection protection

---

# Basic DAO Definition

> **The simplest DAO with query builder**

## Single Table DAO

```dart
// lib/database/daos/user_dao.dart
import 'package:drift/drift.dart';
import '../database.dart';
import '../tables/users.dart';

// 👇 Mark as DAO
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  // 👇 Constructor receives database
  UserDao(super.db);

  // ==================== QUERIES ====================

  // 👇 Get all users
  Future<List<User>> getAllUsers() {
    return select(users).get();
  }

  // 👇 Get user by ID
  Future<User?> getUserById(int id) {
    return (select(users)
      ..where((u) => u.id.equals(id)))
      .getSingleOrNull();
  }

  // 👇 Get active users
  Future<List<User>> getActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .get();
  }

  // 👇 Count users
  Future<int> getUserCount() {
    return select(users).count();
  }
}

// lib/database/database.dart
@DriftDatabase(
  tables: [Users],
  daos: [UserDao], // 👈 Register the DAO
)
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  @override
  int get schemaVersion => 1;
}
```

---

# Query Types in DAOs

> **Different types of queries you can define**

## 1. Future Queries (One-time)

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // 👇 Returns a Future (executes once)
  Future<List<User>> getAllUsers() {
    return select(users).get();
  }

  Future<User?> getUserById(int id) {
    return (select(users)
      ..where((u) => u.id.equals(id)))
      .getSingleOrNull();
  }

  Future<int> getUserCount() {
    return select(users).count();
  }
}
```

## 2. Stream Queries (Reactive)

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // 👇 Returns a Stream (emits on changes)
  Stream<List<User>> watchAllUsers() {
    return select(users).watch();
  }

  Stream<List<User>> watchActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .watch();
  }

  Stream<User?> watchUserById(int id) {
    return (select(users)
      ..where((u) => u.id.equals(id)))
      .watchSingleOrNull();
  }

  Stream<UserStats> watchUserStats() {
    return select(users)
      .watch()
      .map((users) => UserStats(
        total: users.length,
        active: users.where((u) => u.isActive).length,
      ));
  }
}
```

## 3. Custom Methods (Dart Logic)

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // 👇 Custom Dart logic with type safety
  Future<User> createUser(String name, String email) async {
    final id = await into(users).insert(
      UsersCompanion.insert(
        name: name,
        email: email,
        isActive: true,
      ),
    );
    return await getUserById(id);
  }

  Future<void> activateUser(int userId) async {
    await (update(users)
      ..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(isActive: const Value(true)));
  }

  Future<void> deleteUser(int userId) async {
    await (delete(users)
      ..where((u) => u.id.equals(userId)))
      .go();
  }
}
```

---

# Advanced DAO Definition

> **Complex DAO patterns**

## Multiple Tables

```dart
// lib/database/daos/order_dao.dart
import 'package:drift/drift.dart';
import '../database.dart';
import '../tables/orders.dart';
import '../tables/order_items.dart';
import '../tables/products.dart';

@DriftAccessor(tables: [Orders, OrderItems, Products])
class OrderDao extends DatabaseAccessor<AppDatabase> with _$OrderDaoMixin {
  OrderDao(super.db);

  // 👇 Simple queries
  Future<List<Order>> getAllOrders() => select(orders).get();

  Future<Order?> getOrderById(int id) {
    return (select(orders)
      ..where((o) => o.id.equals(id)))
      .getSingleOrNull();
  }

  // 👇 Queries with joins
  Future<OrderWithUser> getOrderWithUser(int orderId) {
    final query = select(orders).join([
      innerJoin(
        users,
        users.id.equals(orders.userId),
      ),
    ]);

    query.where((o) => orders.id.equals(orderId));

    return query.map((row) {
      return OrderWithUser(
        order: row.readTable(orders),
        user: row.readTable(users),
      );
    }).getSingle();
  }

  // 👇 Complex query with multiple joins
  Future<List<OrderWithItems>> getOrdersWithItems() {
    final query = select(orders).join([
      innerJoin(users, users.id.equals(orders.userId)),
      leftJoin(orderItems, orderItems.orderId.equals(orders.id)),
      leftJoin(products, products.id.equals(orderItems.productId)),
    ]);

    query.orderBy([(o) => OrderingTerm.desc(orders.orderDate)]);

    // Group results
    return query.map((row) {
      final order = row.readTable(orders);
      final user = row.readTable(users);
      final item = row.readTableOrNull(orderItems);
      final product = row.readTableOrNull(products);

      return OrderWithItems(
        order: order,
        user: user,
        items: item != null ? [item] : [],
        products: product != null ? [product] : [],
      );
    }).get();
  }
}
```

---

# Real-World Example

> **Complete e-commerce DAO definitions**

```dart
// lib/database/daos/user_dao.dart
import 'package:drift/drift.dart';
import '../database.dart';
import '../tables/users.dart';

@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // ==================== READ ====================

  Future<List<User>> getAllUsers() => select(users).get();

  Future<User?> getUserById(int id) {
    return (select(users)
      ..where((u) => u.id.equals(id)))
      .getSingleOrNull();
  }

  Future<User?> getUserByEmail(String email) {
    return (select(users)
      ..where((u) => u.email.equals(email)))
      .getSingleOrNull();
  }

  Future<List<User>> getActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .get();
  }

  Future<List<User>> getUsersByAgeRange(int minAge, int maxAge) {
    return (select(users)
      ..where((u) => u.age.isBetweenValues(minAge, maxAge)))
      .get();
  }

  Future<List<User>> searchUsers(String query) {
    final searchTerm = '%$query%';
    return (select(users)
      ..where((u) => 
        u.username.like(searchTerm) |
        u.email.like(searchTerm) |
        u.fullName.like(searchTerm)
      ))
      .get();
  }

  Future<int> getUserCount() => select(users).count();

  // ==================== WATCH ====================

  Stream<List<User>> watchActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .watch();
  }

  Stream<User?> watchUserById(int id) {
    return (select(users)
      ..where((u) => u.id.equals(id)))
      .watchSingleOrNull();
  }

  Stream<UserStats> watchUserStats() {
    return select(users)
      .watch()
      .map((users) => UserStats(
        total: users.length,
        active: users.where((u) => u.isActive).length,
        verified: users.where((u) => u.isVerified).length,
      ));
  }

  // ==================== WRITE ====================

  Future<User> createUser({
    required String username,
    required String email,
    required String password,
  }) async {
    final id = await into(users).insert(
      UsersCompanion.insert(
        username: username,
        email: email,
        passwordHash: _hashPassword(password),
        isActive: true,
      ),
    );
    return await getUserById(id);
  }

  Future<void> updateUser(User user) async {
    await update(users).replace(user);
  }

  Future<void> activateUser(int userId) async {
    await (update(users)
      ..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(isActive: const Value(true)));
  }

  Future<void> deactivateUser(int userId) async {
    await (update(users)
      ..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(isActive: const Value(false)));
  }

  Future<void> deleteUser(int userId) async {
    await (delete(users)
      ..where((u) => u.id.equals(userId)))
      .go();
  }

  Future<void> activateMultipleUsers(List<int> userIds) async {
    await into(users).batch((batch) {
      for (final id in userIds) {
        batch.update(
          users,
          UsersCompanion(isActive: const Value(true)),
          (u) => u.id.equals(id),
        );
      }
    });
  }

  // ==================== TRANSACTIONS ====================

  Future<void> transferUserData({
    required int fromUserId,
    required int toUserId,
  }) async {
    await db.transaction(() async {
      // Get both users
      final fromUser = await getUserById(fromUserId);
      final toUser = await getUserById(toUserId);

      if (fromUser == null || toUser == null) {
        throw Exception('User not found');
      }

      // Transfer data
      await (update(users)
        ..where((u) => u.id.equals(fromUserId)))
        .write(UsersCompanion(
          isActive: const Value(false),
        ));

      await (update(users)
        ..where((u) => u.id.equals(toUserId)))
        .write(UsersCompanion(
          isActive: const Value(true),
        ));
    });
  }

  // ==================== PRIVATE ====================

  String _hashPassword(String password) {
    return 'hashed_$password';
  }
}
```

```dart
// lib/database/daos/order_dao.dart
import 'package:drift/drift.dart';
import '../database.dart';
import '../tables/orders.dart';
import '../tables/order_items.dart';
import '../tables/products.dart';

@DriftAccessor(tables: [Orders, OrderItems, Products])
class OrderDao extends DatabaseAccessor<AppDatabase> with _$OrderDaoMixin {
  OrderDao(super.db);

  // ==================== READ ====================

  Future<List<Order>> getAllOrders() => select(orders).get();

  Future<Order?> getOrderById(int id) {
    return (select(orders)
      ..where((o) => o.id.equals(id)))
      .getSingleOrNull();
  }

  Future<List<Order>> getOrdersByUser(int userId) {
    return (select(orders)
      ..where((o) => o.userId.equals(userId))
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
      .get();
  }

  Future<List<Order>> getOrdersByStatus(String status) {
    return (select(orders)
      ..where((o) => o.status.equals(status))
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
      .get();
  }

  // 👇 Order with user info (join)
  Future<OrderWithUser> getOrderWithUser(int orderId) {
    final query = select(orders).join([
      innerJoin(users, users.id.equals(orders.userId)),
    ]);

    query.where((o) => orders.id.equals(orderId));

    return query.map((row) {
      return OrderWithUser(
        order: row.readTable(orders),
        user: row.readTable(users),
      );
    }).getSingle();
  }

  // 👇 Order with items (complex join)
  Future<OrderWithItems> getOrderWithItems(int orderId) {
    final query = select(orders).join([
      innerJoin(users, users.id.equals(orders.userId)),
      leftJoin(orderItems, orderItems.orderId.equals(orders.id)),
      leftJoin(products, products.id.equals(orderItems.productId)),
    ]);

    query.where((o) => orders.id.equals(orderId));

    return query.map((row) {
      final order = row.readTable(orders);
      final user = row.readTable(users);
      final items = <OrderItemWithProduct>[];

      final item = row.readTableOrNull(orderItems);
      if (item != null) {
        final product = row.readTable(products);
        items.add(OrderItemWithProduct(
          orderItem: item,
          product: product,
        ));
      }

      return OrderWithItems(
        order: order,
        user: user,
        items: items,
      );
    }).getSingle();
  }

  // ==================== WATCH ====================

  Stream<List<Order>> watchUserOrders(int userId) {
    return (select(orders)
      ..where((o) => o.userId.equals(userId))
      ..orderBy([(o) => OrderingTerm.desc(o.orderDate)]))
      .watch();
  }

  Stream<OrderStats> watchOrderStats() {
    return select(orders)
      .watch()
      .map((orders) => OrderStats(
        total: orders.length,
        pending: orders.where((o) => o.status == 'pending').length,
        completed: orders.where((o) => o.status == 'completed').length,
        totalRevenue: orders.fold(0.0, (sum, o) => sum + o.total),
      ));
  }

  // ==================== WRITE ====================

  Future<Order> createOrder({
    required int userId,
    required List<OrderItemInput> items,
  }) async {
    return await db.transaction(() async {
      double total = 0;

      // Calculate total and validate stock
      for (final item in items) {
        final product = await (select(products)
          ..where((p) => p.id.equals(item.productId)))
          .getSingle();

        if (product.stock < item.quantity) {
          throw Exception('Insufficient stock for ${product.name}');
        }

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

      // Create order items
      for (final item in items) {
        final product = await (select(products)
          ..where((p) => p.id.equals(item.productId)))
          .getSingle();

        await into(orderItems).insert(
          OrderItemsCompanion.insert(
            orderId: orderId,
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: product.price,
          ),
        );

        // Update stock
        await (update(products)
          ..where((p) => p.id.equals(item.productId)))
          .write(ProductsCompanion(
            stock: Value(product.stock - item.quantity),
          ));
      }

      return await getOrderById(orderId);
    });
  }

  Future<void> updateOrderStatus(int orderId, String newStatus) async {
    await (update(orders)
      ..where((o) => o.id.equals(orderId)))
      .write(OrdersCompanion(status: Value(newStatus)));
  }

  Future<void> cancelOrder(int orderId) async {
    await db.transaction(() async {
      // Restore stock
      final items = await (select(orderItems)
        ..where((i) => i.orderId.equals(orderId)))
        .get();

      for (final item in items) {
        final product = await (select(products)
          ..where((p) => p.id.equals(item.productId)))
          .getSingle();

        await (update(products)
          ..where((p) => p.id.equals(item.productId)))
          .write(ProductsCompanion(
            stock: Value(product.stock + item.quantity),
          ));
      }

      // Cancel order
      await (update(orders)
        ..where((o) => o.id.equals(orderId)))
        .write(OrdersCompanion(status: Value('cancelled')));
    });
  }

  Future<void> deleteOrder(int orderId) async {
    await (delete(orders)
      ..where((o) => o.id.equals(orderId)))
      .go();
  }
}
```

---

# Modern DAO Definition Checklist

| Component | Purpose | Example |
|-----------|---------|---------|
| **@DriftAccessor** | Mark as DAO | `@DriftAccessor(tables: [Users])` |
| **DatabaseAccessor** | Provide database access | `extends DatabaseAccessor<AppDatabase>` |
| **Mixin** | Auto-generated code | `with _$UserDaoMixin` |
| **Constructor** | Receive database | `UserDao(super.db)` |
| **select()** | Start a query | `select(users)` |
| **where()** | Add filters | `..where((u) => u.isActive.equals(true))` |
| **get()** | Execute once | `.get()` |
| **watch()** | Reactive stream | `.watch()` |

---

# Common Mistakes

## Mistake 1: Missing @DriftAccessor

Wrong:
```dart
// ❌ No annotation
class UserDao extends DatabaseAccessor<AppDatabase> {
  UserDao(super.db);
}
```

Correct:
```dart
// ✅ With annotation
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);
}
```

## Mistake 2: Missing tables in @DriftAccessor

Wrong:
```dart
// ❌ Using table not declared
@DriftAccessor(tables: [Users])
class OrderDao extends DatabaseAccessor<AppDatabase> with _$OrderDaoMixin {
  // Uses orders table - error!
}
```

Correct:
```dart
// ✅ Declare all tables used
@DriftAccessor(tables: [Orders, OrderItems])
class OrderDao extends DatabaseAccessor<AppDatabase> with _$OrderDaoMixin {
  // Can use orders and order_items
}
```

## Mistake 3: Not registering DAO

Wrong:
```dart
// ❌ DAO not registered
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase { ... }
```

Correct:
```dart
// ✅ DAO registered
@DriftDatabase(
  tables: [Users],
  daos: [UserDao],
)
class AppDatabase extends _$AppDatabase { ... }
```

---

# Summary

| Feature | Purpose | Example |
|---------|---------|---------|
| **@DriftAccessor** | Mark as DAO | `@DriftAccessor(tables: [Users])` |
| **select()** | Start query | `select(users)` |
| **where()** | Filter | `..where((u) => u.isActive.equals(true))` |
| **get()** | Execute | `.get()` |
| **watch()** | Reactive | `.watch()` |
| **Custom Logic** | Business logic | `createUser()`, `activateUser()` |

---

# Next Steps

Now you understand defining DAOs with the modern API, let's dive deeper:

- [Organizing Queries](link) – Organizing queries in DAOs
- [Injecting Dependencies](link) – Dependency injection for DAOs
- [Best Practices](link) – DAO best practices

---

# Did You Know?

- **Query builder is fully type-safe** – Compile-time validation

- **Query builder has full IDE support** – Autocomplete, refactoring

- **Query builder is the modern way** – Recommended by Drift team

- **Query builder supports all SQL features** – Joins, aggregations

- **Query builder is composable** – Build complex queries piece by piece

- **Query builder is more maintainable** – Pure Dart code

- **Query builder is reactive** – Full `.watch()` support

- **Query builder is the future** – Of Drift development

---
