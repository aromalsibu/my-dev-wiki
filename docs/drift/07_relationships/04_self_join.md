## Self Join

**Joining a table with itself in Drift**

---

# What is it?

**Self Join** is a join where a table is joined with itself. This is essential for hierarchical data structures like employee-manager relationships, category trees, comment threads, and any data where records reference other records in the same table. In Drift, you use table aliases to create self joins.

> **Think of Self Join like "family trees"** – you have a list of people, and each person has a parent. To find out who someone's parent is, you look at the same list of people and find the one matching the parent ID.

```dart
// 👇 Self join: Employees with their managers
final e = alias(db.users, 'e'); // Employee alias
final m = alias(db.users, 'm'); // Manager alias

final query = db.select(e).join([
  leftJoin(
    m,
    e.managerId.equals(m.id),
  )
]);

final results = await query.get();
for (final row in results) {
  final employee = row.readTable(e);
  final manager = row.readTableOrNull(m);
  
  if (manager != null) {
    print('${employee.name} -> Manager: ${manager.name}');
  } else {
    print('${employee.name} -> No manager (CEO)');
  }
}

// Generated SQL:
// SELECT e.*, m.* 
// FROM users e 
// LEFT JOIN users m ON e.manager_id = m.id
```

> **What's happening here?**
> - **`alias()`** – Creates a named alias for the table
> - **Self reference** – Same table used twice
> - **Two aliases** – Different names for different roles
> - **Relationship** – Records reference other records

---

# Why does it exist?

- **Hierarchical Data** – Employee-manager relationships
- **Tree Structures** – Category trees, comment threads
- **Network Data** – Friends, connections
- **Recursive Relationships** – References within same table
- **Reporting** – Generate org charts
- **Data Analysis** – Analyze relationships

---

# Basic Self Join

> **Simple self join patterns**

## Inner Self Join

```dart
// 👇 Self join: All employees with managers
final e = alias(db.users, 'e');
final m = alias(db.users, 'm');

final query = db.select(e).join([
  innerJoin(
    m,
    e.managerId.equals(m.id),
  )
]);

// Returns employees who have managers
// Excludes CEO (no manager)
final results = await query.get();

for (final row in results) {
  final employee = row.readTable(e);
  final manager = row.readTable(m);
  print('${employee.name} -> ${manager.name}');
}
```

## Left Self Join

```dart
// 👇 Left self join: All employees (CEO included)
final e = alias(db.users, 'e');
final m = alias(db.users, 'm');

final query = db.select(e).join([
  leftJoin(
    m,
    e.managerId.equals(m.id),
  )
]);

// Returns ALL employees, CEO has NULL manager
final results = await query.get();

for (final row in results) {
  final employee = row.readTable(e);
  final manager = row.readTableOrNull(m);
  
  if (manager != null) {
    print('${employee.name} -> ${manager.name}');
  } else {
    print('${employee.name} -> CEO (No manager)');
  }
}
```

## Self Join with WHERE

```dart
// 👇 Self join with filters
final e = alias(db.users, 'e');
final m = alias(db.users, 'm');

final query = db.select(e).join([
  innerJoin(
    m,
    e.managerId.equals(m.id),
  )
]);

// Only active employees with active managers
query.where((u) => e.isActive.equals(true));
query.where((u) => m.isActive.equals(true));
query.where((u) => e.department.equals(m.department));

final results = await query.get();
```

---

# Advanced Self Join Patterns

> **Complex self join scenarios**

## Multiple Level Hierarchy

```dart
// 👇 Three-level hierarchy: Employee -> Manager -> Director
final e = alias(db.users, 'e');
final m = alias(db.users, 'm');
final d = alias(db.users, 'd');

final query = db.select(e).join([
  innerJoin(
    m,
    e.managerId.equals(m.id),
  ),
  leftJoin(
    d,
    m.managerId.equals(d.id),
  ),
]);

final results = await query.get();

for (final row in results) {
  final employee = row.readTable(e);
  final manager = row.readTable(m);
  final director = row.readTableOrNull(d);
  
  print('${employee.name} -> Manager: ${manager.name}');
  if (director != null) {
    print('  -> Director: ${director.name}');
  }
}
```

## Find All Subordinates

```dart
// 👇 Find all employees who report to a manager
final e = alias(db.users, 'e');
final m = alias(db.users, 'm');

Future<List<User>> getSubordinates(int managerId) async {
  final query = db.select(db.users)
    ..where((u) => u.managerId.equals(managerId))
    ..where((u) => u.isActive.equals(true))
    ..orderBy([(u) => OrderingTerm.asc(u.name)]);
  
  return await query.get();
}

// 👇 Find all subordinates recursively (using custom SQL)
Future<List<User>> getAllSubordinatesRecursive(int managerId) async {
  final results = await db.customSelect('''
    WITH RECURSIVE subordinates AS (
      SELECT * FROM users WHERE id = ?
      UNION ALL
      SELECT u.* FROM users u
      INNER JOIN subordinates s ON u.manager_id = s.id
    )
    SELECT * FROM subordinates WHERE id != ?
  ''', variables: [
    Variable.withInt(managerId),
    Variable.withInt(managerId),
  ]).get();
  
  return results.map((row) {
    return User(
      id: row.data['id'] as int,
      name: row.data['name'] as String,
      // ... other fields
    );
  }).toList();
}
```

---

# Real-World Example

> **Complete e-commerce self join system**

```dart
// lib/database/self_join_service.dart
import 'package:drift/drift.dart';

class SelfJoinService {
  final AppDatabase db;
  
  SelfJoinService(this.db);

  // ==================== EMPLOYEE HIERARCHY ====================
  
  // 👇 Get complete org chart
  Future<List<OrganizationNode>> getOrgChart() async {
    final e = alias(db.users, 'e');
    final m = alias(db.users, 'm');
    
    final query = db.select(e).join([
      leftJoin(
        m,
        e.managerId.equals(m.id),
      )
    ]);
    
    query.where((u) => e.isActive.equals(true));
    query.orderBy([
      (u) => OrderingTerm.asc(e.managerId),
      (u) => OrderingTerm.asc(e.name),
    ]);
    
    final results = await query.get();
    final employees = <OrganizationNode>[];
    
    for (final row in results) {
      final employee = row.readTable(e);
      final manager = row.readTableOrNull(m);
      
      employees.add(OrganizationNode(
        employee: employee,
        manager: manager,
      ));
    }
    
    return employees;
  }
  
  // 👇 Get manager with direct reports
  Future<ManagerWithReports> getManagerWithReports(int managerId) async {
    final e = alias(db.users, 'e');
    final m = alias(db.users, 'm');
    
    // Get manager info
    final managerQuery = db.select(m)
      ..where((u) => m.id.equals(managerId));
    final manager = await managerQuery.getSingle();
    
    // Get reports
    final reportsQuery = db.select(e)
      ..where((u) => e.managerId.equals(managerId))
      ..where((u) => e.isActive.equals(true))
      ..orderBy([(u) => OrderingTerm.asc(e.name)]);
    
    final reports = await reportsQuery.get();
    
    return ManagerWithReports(
      manager: manager,
      reports: reports,
    );
  }
  
  // 👇 Get full reporting chain
  Future<List<ReportingChain>> getReportingChain(int employeeId) async {
    final results = await db.customSelect('''
      WITH RECURSIVE reporting_chain AS (
        SELECT 
          id,
          name,
          manager_id,
          0 as level
        FROM users 
        WHERE id = ?
        
        UNION ALL
        
        SELECT 
          u.id,
          u.name,
          u.manager_id,
          rc.level + 1
        FROM users u
        INNER JOIN reporting_chain rc ON u.id = rc.manager_id
      )
      SELECT 
        id,
        name,
        manager_id,
        level,
        CASE 
          WHEN level = 0 THEN 'Employee'
          WHEN level = 1 THEN 'Direct Manager'
          WHEN level = 2 THEN 'Second Level Manager'
          ELSE 'Higher Level Manager'
        END as role
      FROM reporting_chain
      ORDER BY level ASC
    ''', variables: [
      Variable.withInt(employeeId),
    ]).get();
    
    return results.map((row) {
      return ReportingChain(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        managerId: row.data['manager_id'] as int?,
        level: row.data['level'] as int,
        role: row.data['role'] as String,
      );
    }).toList();
  }

  // ==================== CATEGORY HIERARCHY ====================
  
  // 👇 Get category tree
  Future<List<CategoryNode>> getCategoryTree() async {
    final c = alias(db.categories, 'c');
    final p = alias(db.categories, 'p');
    
    final query = db.select(c).join([
      leftJoin(
        p,
        c.parentId.equals(p.id),
      )
    ]);
    
    query.where((u) => c.isActive.equals(true));
    query.orderBy([
      (u) => OrderingTerm.asc(c.parentId),
      (u) => OrderingTerm.asc(c.name),
    ]);
    
    final results = await query.get();
    
    return results.map((row) {
      final category = row.readTable(c);
      final parent = row.readTableOrNull(p);
      
      return CategoryNode(
        category: category,
        parent: parent,
      );
    }).toList();
  }
  
  // 👇 Get subcategories recursively
  Future<List<Category>> getSubcategoriesRecursive(int categoryId) async {
    final results = await db.customSelect('''
      WITH RECURSIVE subcategories AS (
        SELECT * FROM categories WHERE id = ?
        UNION ALL
        SELECT c.* FROM categories c
        INNER JOIN subcategories sc ON c.parent_id = sc.id
      )
      SELECT * FROM subcategories WHERE id != ?
    ''', variables: [
      Variable.withInt(categoryId),
      Variable.withInt(categoryId),
    ]).get();
    
    return results.map((row) {
      return Category(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        parentId: row.data['parent_id'] as int?,
        // ... other fields
      );
    }).toList();
  }

  // ==================== COMMENT THREADS ====================
  
  // 👇 Get comment thread with parent
  Future<List<CommentThread>> getCommentThread(int postId) async {
    final c = alias(db.comments, 'c');
    final p = alias(db.comments, 'p');
    
    final query = db.select(c).join([
      leftJoin(
        p,
        c.parentCommentId.equals(p.id),
      )
    ]);
    
    query.where((u) => c.postId.equals(postId));
    query.where((u) => c.isActive.equals(true));
    query.orderBy([(u) => OrderingTerm.asc(c.createdAt)]);
    
    final results = await query.get();
    
    return results.map((row) {
      final comment = row.readTable(c);
      final parent = row.readTableOrNull(p);
      
      return CommentThread(
        comment: comment,
        parent: parent,
      );
    }).toList();
  }
  
  // 👇 Get comment with all replies
  Future<CommentWithReplies> getCommentWithReplies(int commentId) async {
    // Get parent comment
    final parentQuery = db.select(db.comments)
      ..where((c) => c.id.equals(commentId));
    final parent = await parentQuery.getSingle();
    
    // Get replies
    final repliesQuery = db.select(db.comments)
      ..where((c) => c.parentCommentId.equals(commentId))
      ..where((c) => c.isActive.equals(true))
      ..orderBy([(c) => OrderingTerm.asc(c.createdAt)]);
    
    final replies = await repliesQuery.get();
    
    return CommentWithReplies(
      comment: parent,
      replies: replies,
    );
  }

  // ==================== FRIEND NETWORKS ====================
  
  // 👇 Get friends of friends (degree 2)
  Future<List<User>> getFriendsOfFriends(int userId) async {
    final results = await db.customSelect('''
      SELECT DISTINCT f2.*
      FROM friends f1
      INNER JOIN friends f2 ON f1.friend_id = f2.user_id
      WHERE f1.user_id = ?
      AND f2.friend_id != ?
      AND f2.friend_id NOT IN (
        SELECT friend_id FROM friends WHERE user_id = ?
      )
    ''', variables: [
      Variable.withInt(userId),
      Variable.withInt(userId),
      Variable.withInt(userId),
    ]).get();
    
    return results.map((row) {
      return User(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        // ... other fields
      );
    }).toList();
  }
  
  // 👇 Get friends count for all users
  Future<List<UserFriendCount>> getFriendCounts() async {
    final results = await db.customSelect('''
      SELECT 
        u.id as user_id,
        u.name as user_name,
        COUNT(f.friend_id) as friend_count
      FROM users u
      LEFT JOIN friends f ON u.id = f.user_id
      GROUP BY u.id, u.name
      ORDER BY friend_count DESC
    ''').get();
    
    return results.map((row) {
      return UserFriendCount(
        userId: row.data['user_id'] as int,
        userName: row.data['user_name'] as String,
        friendCount: row.data['friend_count'] as int,
      );
    }).toList();
  }

  // ==================== PRODUCT RELATIONSHIPS ====================
  
  // 👇 Get product recommendations (products bought together)
  Future<List<ProductRecommendation>> getProductRecommendations(int productId) async {
    final results = await db.customSelect('''
      WITH product_orders AS (
        SELECT DISTINCT order_id
        FROM order_items
        WHERE product_id = ?
      )
      SELECT 
        p.id as product_id,
        p.name as product_name,
        COUNT(oi.order_id) as bought_together_count
      FROM products p
      INNER JOIN order_items oi ON p.id = oi.product_id
      WHERE p.id != ?
      AND oi.order_id IN product_orders
      GROUP BY p.id, p.name
      ORDER BY bought_together_count DESC
      LIMIT 10
    ''', variables: [
      Variable.withInt(productId),
      Variable.withInt(productId),
    ]).get();
    
    return results.map((row) {
      return ProductRecommendation(
        productId: row.data['product_id'] as int,
        productName: row.data['product_name'] as String,
        boughtTogetherCount: row.data['bought_together_count'] as int,
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class OrganizationNode {
  final User employee;
  final User? manager;
  
  OrganizationNode({
    required this.employee,
    this.manager,
  });
}

class ManagerWithReports {
  final User manager;
  final List<User> reports;
  
  ManagerWithReports({
    required this.manager,
    required this.reports,
  });
}

class ReportingChain {
  final int id;
  final String name;
  final int? managerId;
  final int level;
  final String role;
  
  ReportingChain({
    required this.id,
    required this.name,
    required this.managerId,
    required this.level,
    required this.role,
  });
}

class CategoryNode {
  final Category category;
  final Category? parent;
  
  CategoryNode({
    required this.category,
    this.parent,
  });
}

class CommentThread {
  final Comment comment;
  final Comment? parent;
  
  CommentThread({
    required this.comment,
    this.parent,
  });
}

class CommentWithReplies {
  final Comment comment;
  final List<Comment> replies;
  
  CommentWithReplies({
    required this.comment,
    required this.replies,
  });
}

class UserFriendCount {
  final int userId;
  final String userName;
  final int friendCount;
  
  UserFriendCount({
    required this.userId,
    required this.userName,
    required this.friendCount,
  });
}

class ProductRecommendation {
  final int productId;
  final String productName;
  final int boughtTogetherCount;
  
  ProductRecommendation({
    required this.productId,
    required this.productName,
    required this.boughtTogetherCount,
  });
}
```

---

# Self Join vs Subquery

> **When to use self join vs subquery**

```dart
// 👇 Self Join: Get employees with managers
final e = alias(db.users, 'e');
final m = alias(db.users, 'm');

final joinResult = await db.select(e)
  .join([leftJoin(m, e.managerId.equals(m.id))])
  .get();

// 👇 Subquery: Get employees with managers
final subqueryResult = await db.select(db.users)
  .where((u) => u.managerId.isNotNull())
  .get();
```

---

# Best Practices

- **Always use aliases** – For self joins
- **Use meaningful alias names** – `employee`, `manager`
- **Use left join for optional relationships** – Include all records
- **Use inner join for required relationships** – Only matching
- **Add indexes** – On foreign key columns
- **Use recursive CTE** – For deep hierarchies
- **Limit depth** – Prevent infinite recursion
- **Test with data** – Verify hierarchy

---

# Common Mistakes

## Mistake 1: Forgetting aliases

Wrong:
```dart
// 🚫 Error: Cannot join table to itself
innerJoin(db.users, db.users.managerId.equals(db.users.id))
```

Correct:
```dart
// ✅ Use aliases
final e = alias(db.users, 'e');
final m = alias(db.users, 'm');
innerJoin(m, e.managerId.equals(m.id))
```

## Mistake 2: Wrong column reference

Wrong:
```dart
// 🚫 Wrong alias in condition
innerJoin(m, e.managerId.equals(e.id))
```

Correct:
```dart
// ✅ Correct alias usage
innerJoin(m, e.managerId.equals(m.id))
```

## Mistake 3: Not handling NULL in left join

Wrong:
```dart
// 🚫 CEO has no manager (NULL)
final manager = row.readTable(m);
```

Correct:
```dart
// ✅ Handle NULL
final manager = row.readTableOrNull(m);
```

---

# Summary

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Self Join** | Table with itself | Hierarchies |
| **Aliases** | Required | `alias(table, 'name')` |
| **Recursive CTE** | Deep hierarchies | Trees, networks |
| **Left Join** | Include all | Org charts |

---

# Next Steps

Now you understand self join, let's dive deeper:

- [Many-to-Many](link) – Many-to-many relationships
- [One-to-Many](link) – One-to-many relationships
- [One-to-One](link) – One-to-one relationships

---

# Did You Know?

- **Self joins are essential** – For hierarchical data

- **Self joins require aliases** – To differentiate roles

- **Recursive CTEs are powerful** – For deep hierarchies

- **Self joins can be nested** – Multiple levels

- **Self joins are common** – In org charts, category trees

- **Self joins are efficient** – Single table scan

- **Self joins can be combined** – With other joins

- **Self joins are recursive** – With CTE support

---

