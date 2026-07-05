## What is a DAO?

**Understanding the Data Access Object pattern with Drift's modern API**

---

# What is it?

**DAO (Data Access Object)** is a design pattern that provides an abstract interface to a database. In Drift, a DAO encapsulates all database operations for a specific entity using **type-safe query builder** methods instead of raw SQL strings. This gives you full type safety, IDE autocomplete, refactoring support, and compile-time validation.

> **Think of a DAO like a "type-safe API client"** – instead of writing raw HTTP requests (SQL), you call well-defined methods with typed parameters and get typed responses back. The underlying complexity is completely hidden.

```dart
// 👇 Modern DAO with type-safe query builder
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // 👇 Type-safe query builder (no SQL strings!)
  Future<List<User>> getActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .get();
  }

  Future<User?> getUserById(int id) {
    return (select(users)
      ..where((u) => u.id.equals(id)))
      .getSingleOrNull();
  }

  // 👇 Reactive query (stream)
  Stream<List<User>> watchActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .watch();
  }
}

// 👇 Using the DAO (clean, type-safe API)
final userDao = UserDao(db);
final users = await userDao.getActiveUsers(); // List<User>
final user = await userDao.getUserById(1);    // User?
final stream = userDao.watchActiveUsers();     // Stream<List<User>>
```

> **What's happening here?**
> - **`@DriftAccessor`** – Marks the class as a DAO
> - **Type-safe queries** – Dart methods instead of SQL strings
> - **IDE support** – Autocomplete and refactoring
> - **Reactive** – `.watch()` for streams
> - **Type safety** – Compile-time validation

---

# Why does it exist?

- **Type Safety** – Catch errors at compile-time, not runtime
- **IDE Support** – Autocomplete, refactoring, navigation
- **Abstraction** – Hide database complexity from business logic
- **Maintainability** – Easy to find and update queries
- **Reusability** – Same DAO used throughout the app
- **Testability** – DAOs can be easily mocked
- **Organization** – Queries grouped by entity

---

# Benefits of the Modern DAO Approach

> **Why use the query builder instead of raw SQL?**

## 1. Type Safety

```dart
// ❌ CLASSIC: Raw SQL (no type safety)
@Query('SELECT * FROM users WHERE age > :minAge')
Future<List<User>> getUsersOlderThan(int minAge);
// SQL errors only caught at runtime!

// ✅ MODERN: Query builder (compile-time safety)
Future<List<User>> getUsersOlderThan(int minAge) {
  return (select(users)
    ..where((u) => u.age > const Variable(minAge))) // 👈 Type-safe!
    .get();
}
// Errors caught at compile-time!
```

## 2. IDE Support

```dart
// ✅ MODERN: Full IDE support
Future<List<User>> getActiveUsers() {
  return (select(users)
    ..where((u) => u.isActive.equals(true))) // 👈 Autocomplete works!
    .get();
}
```

## 3. Refactoring

```dart
// ✅ MODERN: Safe refactoring
// If you rename a column, the query builder catches it
Future<List<User>> getActiveUsers() {
  return (select(users)
    ..where((u) => u.isActive.equals(true))) // 👈 Refactoring works!
    .get();
}
```

## 4. Composability

```dart
// ✅ MODERN: Queries are composable
Future<List<User>> getActiveUsers() {
  final query = select(users)
    ..where((u) => u.isActive.equals(true));
  return query.get();
}

Future<List<User>> getActiveAdults() {
  final query = select(users)
    ..where((u) => u.isActive.equals(true))
    ..where((u) => u.age > const Variable(18));
  return query.get();
}
```

---

# DAO vs Repository

> **Understanding the difference**

```dart
// 👇 DAO: Type-safe database operations
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // Type-safe queries
  Future<User?> getUserById(int id) {
    return (select(users)
      ..where((u) => u.id.equals(id)))
      .getSingleOrNull();
  }

  Future<User> createUser(String name, String email) async {
    final id = await into(users).insert(
      UsersCompanion.insert(name: name, email: email),
    );
    return await getUserById(id);
  }
}

// 👇 Repository: Business logic + DAO
class UserRepository {
  final UserDao _userDao;
  final CacheService _cacheService;

  UserRepository(this._userDao, this._cacheService);

  // 👇 Adds business logic (caching, validation)
  Future<User?> getUserById(int id) async {
    // Check cache first
    final cached = await _cacheService.getUser(id);
    if (cached != null) return cached;

    // Get from database
    final user = await _userDao.getUserById(id);

    // Cache for next time
    if (user != null) await _cacheService.cacheUser(user);

    return user;
  }
}
```

---

# Real-World Example

> **Complete e-commerce UserDao with modern API**

```dart
// lib/database/daos/user_dao.dart
import 'package:drift/drift.dart';
import '../database.dart';
import '../tables/users.dart';

@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // ==================== READ OPERATIONS ====================

  // 👇 All users
  Future<List<User>> getAllUsers() => select(users).get();

  // 👇 By ID
  Future<User?> getUserById(int id) {
    return (select(users)
      ..where((u) => u.id.equals(id)))
      .getSingleOrNull();
  }

  // 👇 By email
  Future<User?> getUserByEmail(String email) {
    return (select(users)
      ..where((u) => u.email.equals(email)))
      .getSingleOrNull();
  }

  // 👇 Active users
  Future<List<User>> getActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .get();
  }

  // 👇 Verified users
  Future<List<User>> getVerifiedUsers() {
    return (select(users)
      ..where((u) => u.isVerified.equals(true)))
      .get();
  }

  // 👇 Users by age range
  Future<List<User>> getUsersByAgeRange(int minAge, int maxAge) {
    return (select(users)
      ..where((u) => u.age.isBetweenValues(minAge, maxAge)))
      .get();
  }

  // 👇 Search users
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

  // 👇 Counts
  Future<int> getUserCount() => select(users).count();

  Future<int> getActiveUserCount() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .count();
  }

  // 👇 Recent users
  Future<List<User>> getRecentUsers(int limit) {
    return (select(users)
      ..orderBy([(u) => OrderingTerm.desc(u.createdAt)])
      ..limit(limit))
      .get();
  }

  // ==================== WATCH OPERATIONS ====================

  // 👇 Watch active users (reactive)
  Stream<List<User>> watchActiveUsers() {
    return (select(users)
      ..where((u) => u.isActive.equals(true)))
      .watch();
  }

  // 👇 Watch user by ID
  Stream<User?> watchUserById(int id) {
    return (select(users)
      ..where((u) => u.id.equals(id)))
      .watchSingleOrNull();
  }

  // 👇 Watch all users
  Stream<List<User>> watchAllUsers() => select(users).watch();

  // 👇 Watch user stats
  Stream<UserStats> watchUserStats() {
    return select(users)
      .watch()
      .map((users) {
        return UserStats(
          total: users.length,
          active: users.where((u) => u.isActive).length,
          verified: users.where((u) => u.isVerified).length,
        );
      });
  }

  // ==================== WRITE OPERATIONS ====================

  // 👇 Create user
  Future<User> createUser({
    required String username,
    required String email,
    required String password,
    String? fullName,
    int? age,
  }) async {
    final id = await into(users).insert(
      UsersCompanion.insert(
        username: username,
        email: email,
        passwordHash: _hashPassword(password),
        fullName: Value(fullName),
        age: Value(age),
        isActive: true,
        isVerified: false,
      ),
    );
    return await getUserById(id);
  }

  // 👇 Update user
  Future<void> updateUser(User user) async {
    await update(users).replace(user);
  }

  // 👇 Update user profile
  Future<void> updateUserProfile({
    required int userId,
    String? username,
    String? email,
    String? fullName,
    int? age,
  }) async {
    await (update(users)
      ..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        username: username != null ? Value(username) : const Value.absent(),
        email: email != null ? Value(email) : const Value.absent(),
        fullName: fullName != null ? Value(fullName) : const Value.absent(),
        age: age != null ? Value(age) : const Value.absent(),
        updatedAt: Value(DateTime.now()),
      ));
  }

  // 👇 Status updates
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

  Future<void> verifyUser(int userId) async {
    await (update(users)
      ..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(isVerified: const Value(true)));
  }

  // 👇 Delete user
  Future<void> deleteUser(int userId) async {
    await (delete(users)
      ..where((u) => u.id.equals(userId)))
      .go();
  }

  // 👇 Batch operations
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

  // 👇 Bulk delete
  Future<void> deleteMultipleUsers(List<int> userIds) async {
    await into(users).batch((batch) {
      for (final id in userIds) {
        batch.delete(users, (u) => u.id.equals(id));
      }
    });
  }

  // 👇 Upsert
  Future<User> upsertUser(User user) async {
    await into(users).insert(
      user,
      onConflict: DoUpdate(
        target: users.id,
        update: UsersCompanion.fromUser(user),
      ),
    );
    return await getUserById(user.id);
  }

  // ==================== TRANSACTIONS ====================

  Future<void> activateMultipleUsersWithTransaction(List<int> userIds) async {
    await db.transaction(() async {
      for (final id in userIds) {
        await activateUser(id);
      }
    });
  }

  // ==================== PRIVATE HELPERS ====================

  String _hashPassword(String password) {
    return 'hashed_$password';
  }
}
```

```dart
// lib/database/database.dart
@DriftDatabase(
  tables: [Users],
  daos: [UserDao],
)
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'app_database');
  }
}
```

---

# DAO Best Practices (Modern API)

- **Use query builder** – Never use raw SQL strings
- **Use `.watch()`** – For reactive queries
- **Use `.get()`** – For one-time queries
- **Use `.getSingleOrNull()`** – For optional results
- **Use transactions** – For multiple related operations
- **Use batch operations** – For bulk operations
- **Use `@DriftAccessor`** – For DAO classes
- **Use type-safe Companions** – For inserts and updates

---

# Classic vs Modern DAO Comparison

| Feature | Classic (@Query) | Modern (Query Builder) |
|---------|------------------|----------------------|
| **Type Safety** | ❌ Runtime errors | ✅ Compile-time errors |
| **IDE Support** | ❌ No autocomplete | ✅ Full autocomplete |
| **Refactoring** | ❌ Manual updates | ✅ Automatic refactoring |
| **SQL Injection** | ⚠️ Risk if not careful | ✅ Built-in protection |
| **Reactive** | ⚠️ Limited | ✅ Full `.watch()` support |
| **Code Clarity** | ⚠️ Mix of Dart + SQL | ✅ Pure Dart code |
| **Testing** | ❌ Harder to test | ✅ Easy to test |

---

# Summary

| Aspect | Description | Benefit |
|--------|-------------|---------|
| **Type-Safe** | Query builder | Compile-time safety |
| **IDE Support** | Autocomplete | Faster development |
| **Refactoring** | Safe renaming | Easier maintenance |
| **Reactive** | `.watch()` | Real-time updates |
| **Clean Code** | No SQL strings | Better readability |

---

# Next Steps

Now you understand what a modern DAO is, let's dive deeper:

- [Defining DAOs](link) – How to define DAOs
- [Organizing Queries](link) – Organizing queries in DAOs
- [Injecting Dependencies](link) – Dependency injection for DAOs

---

# Did You Know?

- **Query builder compiles to SQL** – At compile-time

- **Query builder is fully type-safe** – No runtime surprises

- **Query builder is the modern way** – Recommended by Drift team

- **Query builder supports all SQL features** – Joins, aggregations, CTEs

- **Query builder is composable** – Build complex queries piece by piece

- **Query builder has better performance** – Optimized SQL generation

- **Query builder is more maintainable** – Pure Dart code

- **Query builder is the future** – Of Drift development

---

