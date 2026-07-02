## Organizing Queries

**Structuring DAO queries for maintainability and clarity**

---

# What is it?

**Organizing Queries** is the practice of structuring your DAO methods in a logical, consistent, and maintainable way. This includes naming conventions, grouping related queries, separating read and write operations, and organizing by entity or domain. Good organization makes your code easier to understand, maintain, and extend.

> **Think of Organizing Queries like "organizing a library"** – books (queries) are arranged by category (entity), author (operation type), and title (purpose), making them easy to find and understand.

```dart
// 👇 Well-organized DAO
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // ==================== READ OPERATIONS ====================
  // 👇 All read queries grouped together
  
  @Query('SELECT * FROM users')
  Future<List<User>> getAllUsers();

  @Query('SELECT * FROM users WHERE id = :id')
  Future<User?> getUserById(int id);

  @Query('SELECT * FROM users WHERE is_active = true')
  Future<List<User>> getActiveUsers();

  // ==================== WATCH OPERATIONS ====================
  // 👇 All reactive queries grouped together

  @Query('SELECT * FROM users WHERE is_active = true')
  Stream<List<User>> watchActiveUsers();

  // ==================== WRITE OPERATIONS ====================
  // 👇 All write operations grouped together

  Future<User> createUser(String name, String email) async { ... }
  
  Future<void> updateUser(User user) async { ... }
  
  Future<void> deleteUser(int userId) async { ... }
}
```

> **What's happening here?**
> - **Grouping** – Read, watch, and write operations separated
> - **Naming** – Clear, descriptive method names
> - **Ordering** – Logical progression from simple to complex
> - **Consistency** – Similar patterns across DAOs

---

# Why does it exist?

- **Readability** – Easy to find specific queries
- **Maintainability** – Quick to update and refactor
- **Consistency** – Same patterns across DAOs
- **Onboarding** – New developers understand quickly
- **Collaboration** – Multiple developers can work efficiently
- **Scalability** – Easy to add new queries

---

# Organization Strategies

> **Different ways to organize queries**

## 1. By Operation Type

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // 👇 READ OPERATIONS
  // ==================== READ OPERATIONS ====================
  
  @Query('SELECT * FROM users')
  Future<List<User>> getAllUsers();

  @Query('SELECT * FROM users WHERE id = :id')
  Future<User?> getUserById(int id);

  @Query('SELECT * FROM users WHERE is_active = true')
  Future<List<User>> getActiveUsers();

  // 👇 WATCH OPERATIONS
  // ==================== WATCH OPERATIONS ====================
  
  @Query('SELECT * FROM users WHERE is_active = true')
  Stream<List<User>> watchActiveUsers();

  @Query('SELECT * FROM users WHERE id = :id')
  Stream<User?> watchUserById(int id);

  // 👇 WRITE OPERATIONS
  // ==================== WRITE OPERATIONS ====================
  
  Future<User> createUser(String name, String email) async { ... }
  
  Future<void> updateUser(User user) async { ... }
  
  Future<void> deleteUser(int userId) async { ... }

  // 👇 BULK OPERATIONS
  // ==================== BULK OPERATIONS ====================
  
  Future<void> createMultipleUsers(List<User> users) async { ... }
  
  Future<void> deleteMultipleUsers(List<int> userIds) async { ... }
}
```

---

## 2. By Entity/Domain

```dart
// 👇 One DAO per entity
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // All user-related queries
}

@DriftAccessor(tables: [Orders, OrderItems])
class OrderDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  OrderDao(super.db);

  // All order-related queries
}

@DriftAccessor(tables: [Products])
class ProductDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  ProductDao(super.db);

  // All product-related queries
}
```

---

## 3. By Functionality

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);

  // 👇 BASIC CRUD
  // ==================== BASIC CRUD ====================
  
  Future<List<User>> getAllUsers() { ... }
  Future<User?> getUserById(int id) { ... }
  Future<User> createUser(String name, String email) { ... }
  Future<void> updateUser(User user) { ... }
  Future<void> deleteUser(int userId) { ... }

  // 👇 AUTHENTICATION
  // ==================== AUTHENTICATION ====================
  
  Future<User?> getUserByEmail(String email) { ... }
  Future<User?> getUserByAuthToken(String token) { ... }
  Future<void> updateLastLogin(int userId) { ... }

  // 👇 USER MANAGEMENT
  // ==================== USER MANAGEMENT ====================
  
  Future<void> activateUser(int userId) { ... }
  Future<void> deactivateUser(int userId) { ... }
  Future<void> verifyUser(int userId) { ... }
  Future<List<User>> getActiveUsers() { ... }

  // 👇 SEARCH
  // ==================== SEARCH ====================
  
  Future<List<User>> searchUsers(String query) { ... }
  Future<List<User>> getUsersByAgeRange(int min, int max) { ... }
}
```

---

# Naming Conventions

> **Consistent naming for DAO methods**

## Read Methods

```dart
// 👇 Get all records
Future<List<User>> getAllUsers();

// 👇 Get by ID
Future<User?> getUserById(int id);

// 👇 Get by unique field
Future<User?> getUserByEmail(String email);

// 👇 Get by condition
Future<List<User>> getActiveUsers();
Future<List<User>> getUsersByAgeRange(int min, int max);

// 👇 Get limited results
Future<List<User>> getRecentUsers(int limit);
Future<List<User>> getTopUsers(int limit);

// 👇 Search
Future<List<User>> searchUsers(String query);
```

## Watch Methods

```dart
// 👇 Watch all records
Stream<List<User>> watchAllUsers();

// 👇 Watch by ID
Stream<User?> watchUserById(int id);

// 👇 Watch by condition
Stream<List<User>> watchActiveUsers();
Stream<List<User>> watchUsersByAgeRange(int min, int max);
```

## Write Methods

```dart
// 👇 Create
Future<User> createUser(String name, String email);
Future<List<User>> createMultipleUsers(List<User> users);

// 👇 Update
Future<void> updateUser(User user);
Future<void> updateMultipleUsers(List<User> users);
Future<void> updateUserStatus(int userId, bool active);

// 👇 Delete
Future<void> deleteUser(int userId);
Future<void> deleteMultipleUsers(List<int> userIds);

// 👇 Upsert
Future<User> upsertUser(User user);
```

---

# Real-World Example

> **Complete e-commerce query organization system**

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
  @Query('SELECT * FROM users')
  Future<List<User>> getAllUsers();

  // 👇 By unique identifiers
  @Query('SELECT * FROM users WHERE id = :id')
  Future<User?> getUserById(int id);

  @Query('SELECT * FROM users WHERE email = :email')
  Future<User?> getUserByEmail(String email);

  @Query('SELECT * FROM users WHERE username = :username')
  Future<User?> getUserByUsername(String username);

  // 👇 By status
  @Query('SELECT * FROM users WHERE is_active = true')
  Future<List<User>> getActiveUsers();

  @Query('SELECT * FROM users WHERE is_active = false')
  Future<List<User>> getInactiveUsers();

  @Query('SELECT * FROM users WHERE is_verified = true')
  Future<List<User>> getVerifiedUsers();

  // 👇 By date
  @Query('SELECT * FROM users ORDER BY created_at DESC LIMIT :limit')
  Future<List<User>> getRecentUsers(int limit);

  @Query('''
    SELECT * FROM users 
    WHERE created_at > :date
    ORDER BY created_at DESC
  ''')
  Future<List<User>> getUsersCreatedAfter(DateTime date);

  // 👇 By age
  @Query('''
    SELECT * FROM users 
    WHERE age > :minAge AND age < :maxAge
    ORDER BY age
  ''')
  Future<List<User>> getUsersByAgeRange(int minAge, int maxAge);

  // 👇 Search
  @Query('''
    SELECT * FROM users 
    WHERE name LIKE :search 
    OR email LIKE :search
    OR username LIKE :search
  ''')
  Future<List<User>> searchUsers(String search);

  // 👇 Counts
  @Query('SELECT COUNT(*) FROM users')
  Future<int> getUserCount();

  @Query('SELECT COUNT(*) FROM users WHERE is_active = true')
  Future<int> getActiveUserCount();

  // ==================== WATCH OPERATIONS ====================
  
  @Query('SELECT * FROM users WHERE is_active = true')
  Stream<List<User>> watchActiveUsers();

  @Query('SELECT * FROM users WHERE id = :id')
  Stream<User?> watchUserById(int id);

  @Query('SELECT * FROM users WHERE email = :email')
  Stream<User?> watchUserByEmail(String email);

  @Query('SELECT * FROM users')
  Stream<List<User>> watchAllUsers();

  // ==================== WRITE OPERATIONS ====================
  
  // 👇 Create
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
        isVerified: false,
      ),
    );
    return await getUserById(id);
  }

  // 👇 Update
  Future<User> updateUser(User user) async {
    await update(users).write(user);
    return await getUserById(user.id);
  }

  Future<void> updateUserProfile({
    required int userId,
    String? username,
    String? email,
    int? age,
  }) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(
        username: username != null ? Value(username) : const Value.absent(),
        email: email != null ? Value(email) : const Value.absent(),
        age: age != null ? Value(age) : const Value.absent(),
        updatedAt: Value(DateTime.now()),
      ));
  }

  // 👇 Status management
  Future<void> activateUser(int userId) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(isActive: const Value(true)));
  }

  Future<void> deactivateUser(int userId) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(isActive: const Value(false)));
  }

  Future<void> verifyUser(int userId) async {
    await (update(users)..where((u) => u.id.equals(userId)))
      .write(UsersCompanion(isVerified: const Value(true)));
  }

  // 👇 Delete
  Future<void> deleteUser(int userId) async {
    await (delete(users)..where((u) => u.id.equals(userId))).go();
  }

  Future<void> deleteMultipleUsers(List<int> userIds) async {
    await into(users).batch((batch) {
      for (final id in userIds) {
        batch.delete(users, (u) => u.id.equals(id));
      }
    });
  }

  // 👇 Bulk operations
  Future<void> activateMultipleUsers(List<int> userIds) async {
    await db.transaction(() async {
      for (final id in userIds) {
        await activateUser(id);
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

  // ==================== HELPER METHODS ====================

  String _hashPassword(String password) {
    return 'hashed_$password';
  }
}
```

---

# Organization Best Practices

- **Group by operation type** – Read, watch, write
- **Use consistent naming** – `get`, `watch`, `create`, `update`, `delete`
- **One DAO per entity** – Users, orders, products
- **Keep methods focused** – One responsibility per method
- **Document with comments** – Explain purpose
- **Order methods logically** – Simple to complex
- **Use sections** – Visual separation with comments
- **Follow team conventions** – Consistent across the team

---

# Organization Checklist

| Practice | Description | Priority |
|----------|-------------|----------|
| **Group by Type** | Read, watch, write | High |
| **Consistent Naming** | `get`, `watch`, `create` | High |
| **One per Entity** | Separate DAOs | High |
| **Focused Methods** | One responsibility | High |
| **Comments** | Explain purpose | Medium |
| **Logical Order** | Simple to complex | Medium |
| **Visual Separation** | Sections | Medium |

---

# Common Mistakes

## Mistake 1: Random method order

Wrong:
```dart
// 🚫 No logical order
Future<void> deleteUser(int id) { ... }
Future<User> getUserById(int id) { ... }
Future<void> updateUser(User user) { ... }
Future<List<User>> getAllUsers() { ... }
```

Correct:
```dart
// ✅ Grouped by type
// Read operations
Future<User> getUserById(int id) { ... }
Future<List<User>> getAllUsers() { ... }

// Write operations
Future<void> updateUser(User user) { ... }
Future<void> deleteUser(int id) { ... }
```

## Mistake 2: Inconsistent naming

Wrong:
```dart
// 🚫 Inconsistent
Future<User> fetchUser(int id) { ... }
Future<List<User>> getAllUsers() { ... }
Future<User> getUserByEmail(String email) { ... }
```

Correct:
```dart
// ✅ Consistent
Future<User> getUserById(int id) { ... }
Future<List<User>> getAllUsers() { ... }
Future<User> getUserByEmail(String email) { ... }
```

## Mistake 3: Too many methods in one DAO

Wrong:
```dart
// 🚫 One DAO for everything
@DriftAccessor(tables: [Users, Orders, Products])
class MegaDao { ... } // 100+ methods
```

Correct:
```dart
// ✅ Separate DAOs
@DriftAccessor(tables: [Users])
class UserDao { ... }

@DriftAccessor(tables: [Orders])
class OrderDao { ... }

@DriftAccessor(tables: [Products])
class ProductDao { ... }
```

---

# Summary

| Organization | Description | Example |
|--------------|-------------|---------|
| **By Operation** | Read, watch, write | `// READ OPERATIONS` |
| **By Entity** | One per entity | `UserDao`, `OrderDao` |
| **By Naming** | Consistent verbs | `get`, `watch`, `create` |
| **By Section** | Visual separation | Comments and spacing |

---

# Next Steps

Now you understand organizing queries, let's dive deeper:

- [Injecting Dependencies](link) – Dependency injection for DAOs
- [Best Practices](link) – DAO best practices
- [Performance](link) – Performance optimization

---

# Did You Know?

- **Good organization saves time** – Easy to find queries

- **Consistent naming helps** – Predictable method names

- **Grouping improves readability** – Clear structure

- **Comments are valuable** – Explain complex queries

- **Sections help navigation** – Quick visual scanning

- **Organization is a skill** – Improves with practice

- **Good organization scales** – Works for large projects

- **Organization is collaborative** – Team standards

---

