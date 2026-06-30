

# Your First Query

**Write and execute your first type-safe database query**

---

# What is it?

**Your First Query** is the exciting moment when you write your first database interaction using Drift's type-safe query builder. Instead of writing raw SQL strings, you use Dart code that's **type-safe**, **auto-completing**, and **compile-time validated**.

> **Think of Drift's query builder as "SQL in Dart"** – you write queries using Dart syntax, get autocomplete, and catch errors before runtime!

```dart
// ❌ Raw SQL (error-prone, no type safety)
final result = await db.rawQuery(
  'SELECT * FROM users WHERE age > ?',
  [18]
);
final name = result[0]['name'] as String; // Manual cast 😰

// ✅ Drift Query Builder (type-safe, auto-completing)
final users = await (select(users)
  ..where((u) => u.age > const Variable(18)))
  .get();
final name = users.first.name; // Type-safe! 🎯
```

> **What's happening here?**
> - **`select(users)`** – Start a query on the `users` table
> - **`where((u) => u.age > const Variable(18))`** – Filter users older than 18
> - **`get()`** – Execute the query and return `List<User>`
> - **`users.first.name`** – Fully typed, no casting needed!

---

# Why does it exist?

- **Type Safety** – Queries are checked at compile-time
- **IDE Support** – Autocomplete, refactoring, and navigation
- **Readability** – Clean, expressive Dart syntax
- **Composability** – Build complex queries piece by piece
- **Reactive Support** – Easily convert any query to a stream
- **SQL Knowledge** – Works like SQL but with Dart's type system

---

# Setting Up Your First Query

> **Complete step-by-step example**

## Step 1: Define Your Table
```dart
// lib/database/tables/users.dart
import 'package:drift/drift.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().withLength(min: 2, max: 50)();
  TextColumn get email => text().unique()();
  IntColumn get age => integer()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}
```

## Step 2: Create Your Database
```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables/users.dart';

part 'database.g.dart';

@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
}
```

## Step 3: Run Code Generation
```bash
flutter pub run build_runner build
```

## Step 4: Write Your First Query
```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'database/database.dart';

void main() async {
  final db = AppDatabase();
  
  // ✅ INSERT: Add a user
  final userId = await db.into(db.users).insert(
    UsersCompanion.insert(
      name: 'John Doe',
      email: 'john@example.com',
      age: 25,
    ),
  );
  print('Inserted user with ID: $userId');
  
  // ✅ SELECT: Get all users
  final allUsers = await db.select(db.users).get();
  print('All users: $allUsers');
  
  // ✅ SELECT with WHERE: Get active users
  final activeUsers = await (db.select(db.users)
    ..where((u) => u.isActive.equals(true)))
    .get();
  print('Active users: ${activeUsers.length}');
  
  // ✅ SELECT with multiple conditions
  final adultUsers = await (db.select(db.users)
    ..where((u) => u.age.isBiggerOrEqualValue(18))
    ..where((u) => u.isActive.equals(true)))
    .get();
  print('Adult active users: ${adultUsers.length}');
  
  // ✅ UPDATE: Update a user
  await (db.update(db.users)
    ..where((u) => u.id.equals(userId)))
    .write(UsersCompanion(
      age: Value(26),
    ));
  
  // ✅ DELETE: Delete a user
  await (db.delete(db.users)
    ..where((u) => u.id.equals(userId)))
    .go();
  
  await db.close();
}
```

> **What's happening here?**
> - **`into(users).insert()`** – Insert a new row
> - **`select(users).get()`** – Fetch all rows
> - **`where()`** – Add filter conditions
> - **`update(users).write()`** – Update rows
> - **`delete(users).go()`** – Delete rows

---

# Query Builder Methods

> **Complete reference of query builder methods**

## Basic Operations
```dart
// 1️⃣ SELECT all users
final allUsers = await db.select(db.users).get();

// 2️⃣ SELECT first user (single result)
final firstUser = await db.select(db.users).getSingle();

// 3️⃣ SELECT count
final count = await db.select(db.users).count();

// 4️⃣ SELECT with WHERE clause
final youngUsers = await (db.select(db.users)
  ..where((u) => u.age < const Variable(18)))
  .get();

// 5️⃣ SELECT with multiple WHERE conditions
final users = await (db.select(db.users)
  ..where((u) => u.age > const Variable(18))
  ..where((u) => u.isActive.equals(true)))
  .get();

// 6️⃣ SELECT with OR condition
final users = await (db.select(db.users)
  ..where((u) => u.age < const Variable(18) | u.age > const Variable(65)))
  .get();

// 7️⃣ SELECT with ORDER BY
final users = await (db.select(db.users)
  ..orderBy([(u) => OrderingTerm(expression: u.age)]))
  .get();

// 8️⃣ SELECT with ORDER BY DESC
final users = await (db.select(db.users)
  ..orderBy([(u) => OrderingTerm(expression: u.age, mode: OrderingMode.desc)]))
  .get();

// 9️⃣ SELECT with LIMIT
final users = await (db.select(db.users)
  ..limit(10))
  .get();

// 🔟 SELECT with OFFSET
final users = await (db.select(db.users)
  ..limit(10, offset: 20))
  .get();
```

---

# Common Filter Conditions

> **All the ways you can filter queries**

```dart
// lib/database/database.dart
class AppDatabase extends _$AppDatabase {
  // ... setup code

  // ✅ EQUAL
  Future<List<User>> getUsersByAge(int age) {
    return (db.select(db.users)
      ..where((u) => u.age.equals(age)))
      .get();
  }

  // ✅ NOT EQUAL
  Future<List<User>> getUsersNotByName(String name) {
    return (db.select(db.users)
      ..where((u) => u.name.isNotValue(name)))
      .get();
  }

  // ✅ GREATER THAN
  Future<List<User>> getUsersOlderThan(int age) {
    return (db.select(db.users)
      ..where((u) => u.age.isBiggerOrEqualValue(age)))
      .get();
  }

  // ✅ LESS THAN
  Future<List<User>> getUsersYoungerThan(int age) {
    return (db.select(db.users)
      ..where((u) => u.age.isSmallerOrEqualValue(age)))
      .get();
  }

  // ✅ BETWEEN
  Future<List<User>> getUsersBetweenAges(int min, int max) {
    return (db.select(db.users)
      ..where((u) => u.age.isBetweenValues(min, max)))
      .get();
  }

  // ✅ LIKE (pattern matching)
  Future<List<User>> searchUsersByName(String pattern) {
    return (db.select(db.users)
      ..where((u) => u.name.like('%$pattern%')))
      .get();
  }

  // ✅ STARTS WITH
  Future<List<User>> getUsersStartingWith(String prefix) {
    return (db.select(db.users)
      ..where((u) => u.name.startsWith(prefix)))
      .get();
  }

  // ✅ ENDS WITH
  Future<List<User>> getUsersEndingWith(String suffix) {
    return (db.select(db.users)
      ..where((u) => u.name.endsWith(suffix)))
      .get();
  }

  // ✅ IN LIST
  Future<List<User>> getUsersByIds(List<int> ids) {
    return (db.select(db.users)
      ..where((u) => u.id.isIn(ids)))
      .get();
  }

  // ✅ NULL CHECK
  Future<List<User>> getUsersWithEmail() {
    return (db.select(db.users)
      ..where((u) => u.email.isNotNull()))
      .get();
  }

  // ✅ NOT NULL CHECK
  Future<List<User>> getUsersWithoutEmail() {
    return (db.select(db.users)
      ..where((u) => u.email.isNull()))
      .get();
  }

  // ✅ COMPLEX: OR with AND
  Future<List<User>> getSpecialUsers() {
    return (db.select(db.users)
      ..where((u) => 
        (u.age > const Variable(18) & u.isActive.equals(true)) |
        (u.age < const Variable(13))
      ))
      .get();
  }
}
```

---

# Combining Queries

> **Building complex queries step by step**

```dart
// lib/database/database.dart
class AppDatabase extends _$AppDatabase {
  // ... setup code

  // ✅ Method 1: Chained conditions
  Future<List<User>> getActiveAdultUsers() async {
    return await (db.select(db.users)
      ..where((u) => u.isActive.equals(true))
      ..where((u) => u.age > const Variable(18))
      ..orderBy([(u) => OrderingTerm(expression: u.name)])
      ..limit(50))
    .get();
  }

  // ✅ Method 2: Reusable query parts
  Expression<bool> isActiveUser(Users u) => u.isActive.equals(true);
  Expression<bool> isAdult(Users u) => u.age > const Variable(18);

  Future<List<User>> getActiveAdultUsersReusable() async {
    return await (db.select(db.users)
      ..where(isActiveUser)
      ..where(isAdult)
      ..orderBy([(u) => OrderingTerm(expression: u.name)]))
    .get();
  }

  // ✅ Method 3: Dynamic queries
  Future<List<User>> searchUsers({
    String? name,
    int? minAge,
    int? maxAge,
    bool? isActive,
  }) async {
    final query = db.select(db.users);
    
    if (name != null) {
      query.where((u) => u.name.like('%$name%'));
    }
    
    if (minAge != null) {
      query.where((u) => u.age > const Variable(minAge));
    }
    
    if (maxAge != null) {
      query.where((u) => u.age < const Variable(maxAge));
    }
    
    if (isActive != null) {
      query.where((u) => u.isActive.equals(isActive));
    }
    
    return await query.get();
  }

  // ✅ Method 4: Query with pagination
  Future<List<User>> getUsersPaginated(int page, int pageSize) async {
    return await (db.select(db.users)
      ..limit(pageSize, offset: page * pageSize)
      ..orderBy([(u) => OrderingTerm(expression: u.id)]))
    .get();
  }
}
```

---

# Aggregation Queries

> **Count, sum, average, min, max**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift/extensions.dart';

class AppDatabase extends _$AppDatabase {
  // ... setup code

  // ✅ COUNT
  Future<int> getTotalUsers() async {
    return await db.select(db.users).count();
  }

  // ✅ COUNT with WHERE
  Future<int> getActiveUserCount() async {
    return await (db.select(db.users)
      ..where((u) => u.isActive.equals(true)))
      .count();
  }

  // ✅ COUNT DISTINCT
  Future<int> getDistinctEmailCount() async {
    return await db.select(db.users).map((u) => u.email).distinct().count();
  }

  // ✅ MIN
  Future<int?> getMinimumAge() async {
    return await db.select(db.users).map((u) => u.age).min();
  }

  // ✅ MAX
  Future<int?> getMaximumAge() async {
    return await db.select(db.users).map((u) => u.age).max();
  }

  // ✅ SUM
  Future<int?> getTotalAge() async {
    return await db.select(db.users).map((u) => u.age).sum();
  }

  // ✅ AVERAGE
  Future<double?> getAverageAge() async {
    return await db.select(db.users).map((u) => u.age).avg();
  }

  // ✅ Custom aggregation with alias
  Future<int> getActiveUserCountCustom() async {
    // Use raw SQL for complex aggregations
    final result = await db.customSelect(
      'SELECT COUNT(*) as count FROM users WHERE is_active = ?',
      variables: [Variable.withInt(1)],
    ).get();
    
    return result.first.data['count'] as int;
  }
}
```

---

# Real-World Example

> **Complete user management system with queries**

```dart
// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables/users.dart';
import 'tables/posts.dart';

part 'database.g.dart';

@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }

  // ============ USER QUERIES ============
  
  // ✅ Get all users
  Future<List<User>> getAllUsers() {
    return db.select(db.users).get();
  }

  // ✅ Get user by ID
  Future<User?> getUserById(int id) {
    return (db.select(db.users)..where((u) => u.id.equals(id)))
        .getSingleOrNull();
  }

  // ✅ Get user by email
  Future<User?> getUserByEmail(String email) {
    return (db.select(db.users)..where((u) => u.email.equals(email)))
        .getSingleOrNull();
  }

  // ✅ Search users by name (case-insensitive)
  Future<List<User>> searchUsersByName(String query) {
    return (db.select(db.users)
      ..where((u) => u.name.like('%$query%')))
      .get();
  }

  // ✅ Get active users
  Future<List<User>> getActiveUsers() {
    return (db.select(db.users)
      ..where((u) => u.isActive.equals(true)))
      .get();
  }

  // ✅ Get users by age range
  Future<List<User>> getUsersByAgeRange(int minAge, int maxAge) {
    return (db.select(db.users)
      ..where((u) => u.age.isBetweenValues(minAge, maxAge)))
      .get();
  }

  // ✅ Get users sorted by creation date (newest first)
  Future<List<User>> getNewestUsers() {
    return (db.select(db.users)
      ..orderBy([(u) => OrderingTerm(expression: u.createdAt, mode: OrderingMode.desc)])
      ..limit(10))
      .get();
  }

  // ✅ Get user statistics
  Future<Map<String, dynamic>> getUserStats() async {
    final total = await db.select(db.users).count();
    final active = await (db.select(db.users)..where((u) => u.isActive.equals(true)))
        .count();
    final avgAge = await db.select(db.users).map((u) => u.age).avg();
    final minAge = await db.select(db.users).map((u) => u.age).min();
    final maxAge = await db.select(db.users).map((u) => u.age).max();
    
    return {
      'total': total,
      'active': active,
      'inactive': total - active,
      'averageAge': avgAge ?? 0,
      'minAge': minAge ?? 0,
      'maxAge': maxAge ?? 0,
    };
  }

  // ============ POST QUERIES ============
  
  // ✅ Get all posts
  Future<List<Post>> getAllPosts() {
    return db.select(db.posts).get();
  }

  // ✅ Get posts by user
  Future<List<Post>> getPostsByUser(int userId) {
    return (db.select(db.posts)..where((p) => p.userId.equals(userId)))
        .get();
  }

  // ✅ Get recent posts with limit
  Future<List<Post>> getRecentPosts(int limit) {
    return (db.select(db.posts)
      ..orderBy([(p) => OrderingTerm(expression: p.publishedAt, mode: OrderingMode.desc)])
      ..limit(limit))
      .get();
  }

  // ✅ Search posts by title
  Future<List<Post>> searchPostsByTitle(String query) {
    return (db.select(db.posts)
      ..where((p) => p.title.like('%$query%')))
      .get();
  }

  // ✅ Get posts with user details (JOIN query)
  Future<List<Map<String, dynamic>>> getPostsWithUserInfo() async {
    final results = await db.customSelect('''
      SELECT posts.*, users.name as author_name, users.email as author_email
      FROM posts
      INNER JOIN users ON posts.user_id = users.id
      ORDER BY posts.published_at DESC
    ''').get();
    
    return results.map((row) => row.data).toList();
  }

  // ============ COMPLEX QUERIES ============
  
  // ✅ Get active users with their post count
  Future<List<Map<String, dynamic>>> getActiveUsersWithPostCount() async {
    final results = await db.customSelect('''
      SELECT 
        users.*,
        COUNT(posts.id) as post_count
      FROM users
      LEFT JOIN posts ON users.id = posts.user_id
      WHERE users.is_active = 1
      GROUP BY users.id
      HAVING post_count > 0
      ORDER BY post_count DESC
    ''').get();
    
    return results.map((row) => row.data).toList();
  }

  // ✅ Get user engagement statistics
  Future<List<Map<String, dynamic>>> getUserEngagement() async {
    final results = await db.customSelect('''
      SELECT 
        users.id,
        users.name,
        COUNT(posts.id) as total_posts,
        MAX(posts.published_at) as last_post_date
      FROM users
      LEFT JOIN posts ON users.id = posts.user_id
      GROUP BY users.id
      ORDER BY total_posts DESC
    ''').get();
    
    return results.map((row) => row.data).toList();
  }
}
```

```dart
// lib/main.dart - Usage example
import 'package:flutter/material.dart';
import 'database/database.dart';

void main() async {
  final db = AppDatabase();
  
  // ✅ Insert sample data
  await db.into(db.users).insert(
    UsersCompanion.insert(
      name: 'Alice Johnson',
      email: 'alice@example.com',
      age: 28,
      isActive: true,
    ),
  );
  
  await db.into(db.users).insert(
    UsersCompanion.insert(
      name: 'Bob Smith',
      email: 'bob@example.com',
      age: 22,
      isActive: true,
    ),
  );
  
  // ✅ Get all users
  final allUsers = await db.getAllUsers();
  print('Total users: ${allUsers.length}');
  
  // ✅ Search users
  final searchResults = await db.searchUsersByName('Alice');
  print('Search results: ${searchResults.length}');
  
  // ✅ Get user stats
  final stats = await db.getUserStats();
  print('Stats: $stats');
  
  // ✅ Get recent posts
  final recentPosts = await db.getRecentPosts(5);
  print('Recent posts: ${recentPosts.length}');
  
  // ✅ Complex query
  final engagement = await db.getUserEngagement();
  print('Engagement: $engagement');
  
  await db.close();
}
```

> **Benefits of this approach:**
> - **Clean organization** – All queries in one place
> - **Type-safe** – All methods return typed results
> - **Reusable** – Queries can be composed and reused
> - **Documented** – Method names describe what they do
> - **Testable** – Easy to mock and test

---

# Best Practices

- **Use `where` with variables** – Never concatenate strings (SQL injection risk)
- **Use `getSingle()` carefully** – Throws if no results or multiple results
- **Use `getSingleOrNull()`** – Safer for optional results
- **Name your queries descriptively** – `getActiveUsers()` not `query1()`
- **Use `limit()` for pagination** – Avoid loading large datasets
- **Use `orderBy()` for consistent results** – Especially with limit
- **Combine conditions with `&` (AND) and `|` (OR)** – For complex logic
- **Use `customSelect()` for complex joins** – When query builder is limiting
- **Add indexes for frequent queries** – Performance optimization
- **Test your queries** – Use in-memory database for tests

---

# Common Mistakes

## Mistake 1: Not using variables

Wrong:
```dart
// 🚫 SQL injection risk
final users = await db.customSelect(
  'SELECT * FROM users WHERE name = $name'
).get();
```

Correct:
```dart
// ✅ Safe with variables
final users = await db.customSelect(
  'SELECT * FROM users WHERE name = ?',
  variables: [Variable.withString(name)]
).get();
```

## Mistake 2: Forgetting to call `get()`

Wrong:
```dart
// 🚫 Returns query builder, not results
final users = db.select(db.users);
```

Correct:
```dart
// ✅ Executes query
final users = await db.select(db.users).get();
```

## Mistake 3: Using `getSingle()` on empty results

Wrong:
```dart
// 🚫 Throws if no user found
final user = await (db.select(db.users)..where((u) => u.id.equals(999)))
    .getSingle();
```

Correct:
```dart
// ✅ Handles empty case
final user = await (db.select(db.users)..where((u) => u.id.equals(999)))
    .getSingleOrNull();
```

## Mistake 4: Not handling null values

Wrong:
```dart
// 🚫 Null if no posts
final posts = await db.select(db.posts).get();
```

Correct:
```dart
// ✅ Handle empty list
final posts = await db.select(db.posts).get() ?? [];
```

---

# Summary

| Operation | Method | Example |
|-----------|--------|---------|
| **SELECT all** | `select(table).get()` | `db.select(db.users).get()` |
| **SELECT with WHERE** | `..where((t) => condition)` | `..where((u) => u.age > 18)` |
| **ORDER BY** | `..orderBy([...])` | `..orderBy([(u) => OrderingTerm(expression: u.name)])` |
| **LIMIT** | `..limit(n)` | `..limit(10)` |
| **Single result** | `.getSingle()` or `.getSingleOrNull()` | `.getSingle()` |
| **INSERT** | `into(table).insert()` | `db.into(db.users).insert(companion)` |
| **UPDATE** | `update(table).write()` | `db.update(db.users).write(companion)` |
| **DELETE** | `delete(table).go()` | `db.delete(db.users).go()` |
| **COUNT** | `.count()` | `db.select(db.users).count()` |
| **JOIN** | `customSelect()` | Raw SQL for complex joins |

---

# Next Steps

Now you've written your first query, let's dive deeper:

- [Tables Deep Dive](link) – Advanced table definitions
- [CRUD Operations](link) – Complete CRUD guide
- [Query Builder](link) – Advanced query builder techniques

---

# Did You Know?

- **Drift's query builder compiles to SQL** – Each Dart expression becomes SQL behind the scenes

- **The `where` conditions use `Expression` objects** – They're not evaluated at runtime in Dart

- **You can use Dart's `&` and `|` operators** – They map to SQL `AND` and `OR`

- **`get()` returns a `Future<List<T>>`** – It executes the query and maps results

- **`watch()` returns a `Stream<List<T>>`** – It emits new results when data changes

- **Drift uses `Variable` for parameters** – This prevents SQL injection

- **You can use custom SQL** – `customSelect()` for complex queries

- **All Drift queries are async** – They return `Future` or `Stream`

---

