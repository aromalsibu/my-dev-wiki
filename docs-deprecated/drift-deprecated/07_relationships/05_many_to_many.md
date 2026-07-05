## Many-to-Many

**Implementing many-to-many relationships in Drift**

---

# What is it?

**Many-to-Many** relationships occur when multiple records in one table relate to multiple records in another table. This requires a junction table (also called a bridge table or associative table) that stores the relationships between the two tables. Drift provides powerful ways to query and manage many-to-many relationships using joins and custom SQL.

> **Think of Many-to-Many like "students and courses"** – a student can take many courses, and a course can have many students. The enrollment table connects them, showing which students are in which courses.

```dart
// 👇 Three tables: Students, Courses, Enrollments (junction)
// Students
class Students extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

// Courses
class Courses extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
}

// Junction table (Enrollments)
class Enrollments extends Table {
  @override
  Set<Column> get primaryKey => {studentId, courseId};
  
  IntColumn get studentId => integer().references(Students, #id)();
  IntColumn get courseId => integer().references(Courses, #id)();
  DateTimeColumn get enrolledAt => dateTime().withDefault(currentDateAndTime)();
  TextColumn get grade => text().nullable()();
}

// 👇 Query: Get all courses for a student
final studentId = 1;
final query = db.select(db.courses).join([
  innerJoin(
    db.enrollments,
    db.enrollments.courseId.equals(db.courses.id),
  )
]);
query.where((e) => db.enrollments.studentId.equals(studentId));

final courses = await query.get();
```

> **What's happening here?**
> - **Two parent tables** – Students and Courses
> - **Junction table** – Enrollments connects them
> - **Composite primary key** – (studentId, courseId) is unique
> - **Foreign keys** – References to both parent tables
> - **Additional fields** – Enrollment date, grade, etc.

---

# Why does it exist?

- **Real-world relationships** – Students-courses, products-orders, users-roles
- **Flexible connections** – Many-to-many associations
- **Rich relationships** – Add metadata to connections
- **Data Normalization** – Avoid data duplication
- **Query Efficiency** – Efficient relationship queries
- **Data Integrity** – Enforce relationship constraints

---

# Defining Many-to-Many Tables

> **Creating the junction table**

## Basic Junction Table

```dart
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
}

// lib/database/tables/roles.dart
class Roles extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().unique()();
  TextColumn get description => text().nullable()();
}

// lib/database/tables/user_roles.dart (Junction)
class UserRoles extends Table {
  // 👇 Composite primary key
  @override
  Set<Column> get primaryKey => {userId, roleId};
  
  // 👇 Foreign keys
  IntColumn get userId => integer()
    .references(Users, #id, onDelete: KeyAction.cascade)
    .named('user_id')();
  
  IntColumn get roleId => integer()
    .references(Roles, #id, onDelete: KeyAction.cascade)
    .named('role_id')();
  
  // 👇 Additional metadata
  DateTimeColumn get assignedAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('assigned_at')();
  
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))
    .named('is_active')();
  
  // 👇 Indexes for performance
  @override
  List<Index> get indexes => [
    Index('idx_user_roles_user', 'user_id'),
    Index('idx_user_roles_role', 'role_id'),
    Index('idx_user_roles_active', 'is_active'),
  ];
}
```

---

# Querying Many-to-Many

> **Retrieving related data**

## Get Related Records

```dart
// 👇 Get all roles for a user
Future<List<Role>> getUserRoles(int userId) async {
  final query = db.select(db.roles).join([
    innerJoin(
      db.userRoles,
      db.userRoles.roleId.equals(db.roles.id),
    )
  ]);
  
  query.where((ur) => db.userRoles.userId.equals(userId));
  query.where((ur) => db.userRoles.isActive.equals(true));
  
  final results = await query.get();
  return results.map((row) => row.readTable(db.roles)).toList();
}

// 👇 Get all users for a role
Future<List<User>> getRoleUsers(int roleId) async {
  final query = db.select(db.users).join([
    innerJoin(
      db.userRoles,
      db.userRoles.userId.equals(db.users.id),
    )
  ]);
  
  query.where((ur) => db.userRoles.roleId.equals(roleId));
  query.where((ur) => db.userRoles.isActive.equals(true));
  
  final results = await query.get();
  return results.map((row) => row.readTable(db.users)).toList();
}
```

## Get Related Records with Metadata

```dart
// 👇 Get user roles with assignment details
Future<List<UserRoleAssignment>> getUserRoleAssignments(int userId) async {
  final query = db.select(db.users).join([
    innerJoin(
      db.userRoles,
      db.userRoles.userId.equals(db.users.id),
    ),
    innerJoin(
      db.roles,
      db.roles.id.equals(db.userRoles.roleId),
    ),
  ]);
  
  query.where((u) => db.users.id.equals(userId));
  query.where((ur) => db.userRoles.isActive.equals(true));
  
  final results = await query.get();
  
  return results.map((row) {
    final user = row.readTable(db.users);
    final role = row.readTable(db.roles);
    final userRole = row.readTable(db.userRoles);
    
    return UserRoleAssignment(
      user: user,
      role: role,
      assignedAt: userRole.assignedAt,
      isActive: userRole.isActive,
    );
  }).toList();
}
```

---

# Managing Many-to-Many

> **Adding, updating, and removing relationships**

## Add Relationship

```dart
// 👇 Assign role to user
Future<void> assignRoleToUser({
  required int userId,
  required int roleId,
}) async {
  await db.into(db.userRoles).insert(
    UserRolesCompanion.insert(
      userId: userId,
      roleId: roleId,
      // assignedAt: uses default (current time)
      // isActive: uses default (true)
    ),
  );
}

// 👇 Assign multiple roles
Future<void> assignRolesToUser({
  required int userId,
  required List<int> roleIds,
}) async {
  await db.transaction(() async {
    for (final roleId in roleIds) {
      await db.into(db.userRoles).insert(
        UserRolesCompanion.insert(
          userId: userId,
          roleId: roleId,
        ),
      );
    }
  });
}
```

## Remove Relationship

```dart
// 👇 Remove role from user
Future<void> removeRoleFromUser({
  required int userId,
  required int roleId,
}) async {
  await (db.delete(db.userRoles)
    ..where((ur) => 
      ur.userId.equals(userId) &
      ur.roleId.equals(roleId)
    ))
    .go();
}

// 👇 Soft delete (deactivate) role
Future<void> deactivateRoleFromUser({
  required int userId,
  required int roleId,
}) async {
  await (db.update(db.userRoles)
    ..where((ur) => 
      ur.userId.equals(userId) &
      ur.roleId.equals(roleId)
    ))
    .write(UserRolesCompanion(
      isActive: const Value(false),
    ));
}
```

---

# Real-World Example

> **Complete e-commerce many-to-many system**

```dart
// lib/database/tables/products.dart
import 'package:drift/drift.dart';

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get sku => text().unique()();
  RealColumn get price => real()();
  IntColumn get stock => integer()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

// lib/database/tables/categories.dart
class Categories extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().unique()();
  TextColumn get description => text().nullable()();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
}

// lib/database/tables/product_categories.dart (Junction)
class ProductCategories extends Table {
  @override
  Set<Column> get primaryKey => {productId, categoryId};
  
  IntColumn get productId => integer()
    .references(Products, #id, onDelete: KeyAction.cascade)
    .named('product_id')();
  
  IntColumn get categoryId => integer()
    .references(Categories, #id, onDelete: KeyAction.cascade)
    .named('category_id')();
  
  BoolColumn get isPrimary => boolean()
    .withDefault(const Constant(false))
    .named('is_primary')();
  
  @override
  List<Index> get indexes => [
    Index('idx_product_categories_product', 'product_id'),
    Index('idx_product_categories_category', 'category_id'),
  ];
}

// lib/database/tables/tags.dart
class Tags extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().unique()();
  TextColumn get color => text().nullable()();
}

// lib/database/tables/product_tags.dart (Junction)
class ProductTags extends Table {
  @override
  Set<Column> get primaryKey => {productId, tagId};
  
  IntColumn get productId => integer()
    .references(Products, #id, onDelete: KeyAction.cascade)
    .named('product_id')();
  
  IntColumn get tagId => integer()
    .references(Tags, #id, onDelete: KeyAction.cascade)
    .named('tag_id')();
  
  @override
  List<Index> get indexes => [
    Index('idx_product_tags_product', 'product_id'),
    Index('idx_product_tags_tag', 'tag_id'),
  ];
}

// lib/database/many_to_many_service.dart
import 'package:drift/drift.dart';

class ManyToManyService {
  final AppDatabase db;
  
  ManyToManyService(this.db);

  // ==================== PRODUCT-CATEGORY ====================
  
  // 👇 Get all categories for a product
  Future<List<Category>> getProductCategories(int productId) async {
    final query = db.select(db.categories).join([
      innerJoin(
        db.productCategories,
        db.productCategories.categoryId.equals(db.categories.id),
      )
    ]);
    
    query.where((pc) => db.productCategories.productId.equals(productId));
    query.where((c) => db.categories.isActive.equals(true));
    
    final results = await query.get();
    return results.map((row) => row.readTable(db.categories)).toList();
  }
  
  // 👇 Get all products in a category
  Future<List<Product>> getCategoryProducts(int categoryId) async {
    final query = db.select(db.products).join([
      innerJoin(
        db.productCategories,
        db.productCategories.productId.equals(db.products.id),
      )
    ]);
    
    query.where((pc) => db.productCategories.categoryId.equals(categoryId));
    query.where((p) => db.products.isActive.equals(true));
    
    final results = await query.get();
    return results.map((row) => row.readTable(db.products)).toList();
  }
  
  // 👇 Add product to category
  Future<void> addProductToCategory({
    required int productId,
    required int categoryId,
    bool isPrimary = false,
  }) async {
    await db.into(db.productCategories).insert(
      ProductCategoriesCompanion.insert(
        productId: productId,
        categoryId: categoryId,
        isPrimary: Value(isPrimary),
      ),
    );
  }
  
  // 👇 Remove product from category
  Future<void> removeProductFromCategory({
    required int productId,
    required int categoryId,
  }) async {
    await (db.delete(db.productCategories)
      ..where((pc) => 
        pc.productId.equals(productId) &
        pc.categoryId.equals(categoryId)
      ))
      .go();
  }
  
  // 👇 Get products with all categories
  Future<List<ProductWithCategories>> getProductsWithCategories() async {
    final query = db.select(db.products).join([
      leftJoin(
        db.productCategories,
        db.productCategories.productId.equals(db.products.id),
      ),
      leftJoin(
        db.categories,
        db.categories.id.equals(db.productCategories.categoryId),
      ),
    ]);
    
    query.where((p) => db.products.isActive.equals(true));
    query.orderBy([(p) => OrderingTerm.asc(db.products.name)]);
    
    final results = await query.get();
    
    // Group by product
    final productMap = <int, ProductWithCategories>{};
    
    for (final row in results) {
      final product = row.readTable(db.products);
      
      if (!productMap.containsKey(product.id)) {
        productMap[product.id] = ProductWithCategories(
          product: product,
          categories: [],
        );
      }
      
      final category = row.readTableOrNull(db.categories);
      if (category != null) {
        productMap[product.id]!.categories.add(category);
      }
    }
    
    return productMap.values.toList();
  }

  // ==================== PRODUCT-TAG ====================
  
  // 👇 Get all tags for a product
  Future<List<Tag>> getProductTags(int productId) async {
    final query = db.select(db.tags).join([
      innerJoin(
        db.productTags,
        db.productTags.tagId.equals(db.tags.id),
      )
    ]);
    
    query.where((pt) => db.productTags.productId.equals(productId));
    query.orderBy([(t) => OrderingTerm.asc(db.tags.name)]);
    
    final results = await query.get();
    return results.map((row) => row.readTable(db.tags)).toList();
  }
  
  // 👇 Get all products with a tag
  Future<List<Product>> getTagProducts(int tagId) async {
    final query = db.select(db.products).join([
      innerJoin(
        db.productTags,
        db.productTags.productId.equals(db.products.id),
      )
    ]);
    
    query.where((pt) => db.productTags.tagId.equals(tagId));
    query.where((p) => db.products.isActive.equals(true));
    
    final results = await query.get();
    return results.map((row) => row.readTable(db.products)).toList();
  }
  
  // 👇 Add tags to product (bulk)
  Future<void> addTagsToProduct({
    required int productId,
    required List<int> tagIds,
  }) async {
    await db.transaction(() async {
      for (final tagId in tagIds) {
        await db.into(db.productTags).insert(
          ProductTagsCompanion.insert(
            productId: productId,
            tagId: tagId,
          ),
        );
      }
    });
  }
  
  // 👇 Replace all tags for a product
  Future<void> replaceProductTags({
    required int productId,
    required List<int> tagIds,
  }) async {
    await db.transaction(() async {
      // Remove all existing tags
      await (db.delete(db.productTags)
        ..where((pt) => pt.productId.equals(productId)))
        .go();
      
      // Add new tags
      for (final tagId in tagIds) {
        await db.into(db.productTags).insert(
          ProductTagsCompanion.insert(
            productId: productId,
            tagId: tagId,
          ),
        );
      }
    });
  }
  
  // 👇 Get products with tags
  Future<List<ProductWithTags>> getProductsWithTags() async {
    final query = db.select(db.products).join([
      leftJoin(
        db.productTags,
        db.productTags.productId.equals(db.products.id),
      ),
      leftJoin(
        db.tags,
        db.tags.id.equals(db.productTags.tagId),
      ),
    ]);
    
    query.where((p) => db.products.isActive.equals(true));
    query.orderBy([(p) => OrderingTerm.asc(db.products.name)]);
    
    final results = await query.get();
    
    final productMap = <int, ProductWithTags>{};
    
    for (final row in results) {
      final product = row.readTable(db.products);
      
      if (!productMap.containsKey(product.id)) {
        productMap[product.id] = ProductWithTags(
          product: product,
          tags: [],
        );
      }
      
      final tag = row.readTableOrNull(db.tags);
      if (tag != null) {
        productMap[product.id]!.tags.add(tag);
      }
    }
    
    return productMap.values.toList();
  }

  // ==================== ADVANCED MANY-TO-MANY ====================
  
  // 👇 Get products with their categories and tags
  Future<List<ProductWithFullDetails>> getProductsWithFullDetails() async {
    final query = db.select(db.products).join([
      leftJoin(
        db.productCategories,
        db.productCategories.productId.equals(db.products.id),
      ),
      leftJoin(
        db.categories,
        db.categories.id.equals(db.productCategories.categoryId),
      ),
      leftJoin(
        db.productTags,
        db.productTags.productId.equals(db.products.id),
      ),
      leftJoin(
        db.tags,
        db.tags.id.equals(db.productTags.tagId),
      ),
    ]);
    
    query.where((p) => db.products.isActive.equals(true));
    query.orderBy([(p) => OrderingTerm.asc(db.products.name)]);
    
    final results = await query.get();
    
    final productMap = <int, ProductWithFullDetails>{};
    
    for (final row in results) {
      final product = row.readTable(db.products);
      
      if (!productMap.containsKey(product.id)) {
        productMap[product.id] = ProductWithFullDetails(
          product: product,
          categories: [],
          tags: [],
        );
      }
      
      final category = row.readTableOrNull(db.categories);
      if (category != null && !productMap[product.id]!.categories.contains(category)) {
        productMap[product.id]!.categories.add(category);
      }
      
      final tag = row.readTableOrNull(db.tags);
      if (tag != null && !productMap[product.id]!.tags.contains(tag)) {
        productMap[product.id]!.tags.add(tag);
      }
    }
    
    return productMap.values.toList();
  }
  
  // 👇 Get products by categories and tags
  Future<List<Product>> getProductsByFilters({
    List<int>? categoryIds,
    List<int>? tagIds,
  }) async {
    var query = db.select(db.products)
      ..where((p) => p.isActive.equals(true));
    
    if (categoryIds != null && categoryIds.isNotEmpty) {
      query = query.join([
        innerJoin(
          db.productCategories,
          db.productCategories.productId.equals(db.products.id),
        ),
      ]);
      query.where((pc) => db.productCategories.categoryId.isIn(categoryIds));
    }
    
    if (tagIds != null && tagIds.isNotEmpty) {
      query = query.join([
        innerJoin(
          db.productTags,
          db.productTags.productId.equals(db.products.id),
        ),
      ]);
      query.where((pt) => db.productTags.tagId.isIn(tagIds));
    }
    
    // Use custom SQL for complex filtering
    final results = await db.customSelect('''
      SELECT DISTINCT p.*
      FROM products p
      ${categoryIds != null && categoryIds.isNotEmpty ? 'INNER JOIN product_categories pc ON p.id = pc.product_id AND pc.category_id IN (${categoryIds.join(',')})' : ''}
      ${tagIds != null && tagIds.isNotEmpty ? 'INNER JOIN product_tags pt ON p.id = pt.product_id AND pt.tag_id IN (${tagIds.join(',')})' : ''}
      WHERE p.is_active = 1
    ''').get();
    
    return results.map((row) {
      return Product(
        id: row.data['id'] as int,
        name: row.data['name'] as String,
        sku: row.data['sku'] as String,
        price: row.data['price'] as double,
        stock: row.data['stock'] as int,
        isActive: (row.data['is_active'] as int) == 1,
        createdAt: DateTime.fromMillisecondsSinceEpoch(row.data['created_at'] as int),
      );
    }).toList();
  }
}

// ==================== DATA CLASSES ====================

class UserRoleAssignment {
  final User user;
  final Role role;
  final DateTime assignedAt;
  final bool isActive;
  
  UserRoleAssignment({
    required this.user,
    required this.role,
    required this.assignedAt,
    required this.isActive,
  });
}

class ProductWithCategories {
  final Product product;
  final List<Category> categories;
  
  ProductWithCategories({
    required this.product,
    required this.categories,
  });
}

class ProductWithTags {
  final Product product;
  final List<Tag> tags;
  
  ProductWithTags({
    required this.product,
    required this.tags,
  });
}

class ProductWithFullDetails {
  final Product product;
  final List<Category> categories;
  final List<Tag> tags;
  
  ProductWithFullDetails({
    required this.product,
    required this.categories,
    required this.tags,
  });
}
```

```dart
// lib/ui/pages/product_detail_page.dart
class ProductDetailPage extends StatefulWidget {
  final ManyToManyService service;
  final int productId;
  
  const ProductDetailPage({
    required this.service,
    required this.productId,
  });
  
  @override
  _ProductDetailPageState createState() => _ProductDetailPageState();
}

class _ProductDetailPageState extends State<ProductDetailPage> {
  ProductWithFullDetails? _product;
  bool _isLoading = true;
  String? _error;
  
  @override
  void initState() {
    super.initState();
    _loadProduct();
  }
  
  Future<void> _loadProduct() async {
    setState(() => _isLoading = true);
    
    try {
      final products = await widget.service.getProductsWithFullDetails();
      final product = products.firstWhere(
        (p) => p.product.id == widget.productId,
      );
      
      setState(() {
        _product = product;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Product Details')),
      body: _isLoading
          ? Center(child: CircularProgressIndicator())
          : _error != null
              ? _buildError()
              : _buildContent(),
    );
  }
  
  Widget _buildError() {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.error, size: 64, color: Colors.red),
          SizedBox(height: 16),
          Text('Error: $_error'),
          SizedBox(height: 16),
          ElevatedButton(
            onPressed: _loadProduct,
            child: Text('Retry'),
          ),
        ],
      ),
    );
  }
  
  Widget _buildContent() {
    final product = _product!;
    
    return Padding(
      padding: EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            product.product.name,
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 8),
          Text('SKU: ${product.product.sku}'),
          Text('Price: \$${product.product.price.toStringAsFixed(2)}'),
          Text('Stock: ${product.product.stock}'),
          SizedBox(height: 16),
          Text(
            'Categories:',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),
          Wrap(
            spacing: 8,
            children: product.categories.map((category) {
              return Chip(label: Text(category.name));
            }).toList(),
          ),
          SizedBox(height: 16),
          Text(
            'Tags:',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),
          Wrap(
            spacing: 8,
            children: product.tags.map((tag) {
              return Chip(
                label: Text(tag.name),
                backgroundColor: Colors.blue[100],
              );
            }).toList(),
          ),
        ],
      ),
    );
  }
}
```

---

# Best Practices

- **Use composite primary key** – For junction tables
- **Add foreign key constraints** – With cascade delete
- **Add indexes** – On foreign key columns
- **Use transactions** – For multiple relationship updates
- **Add metadata** – To junction tables when needed
- **Use distinct** – Avoid duplicate results
- **Use inner joins** – For existing relationships
- **Use left joins** – For optional relationships

---

# Common Mistakes

## Mistake 1: Missing composite primary key

Wrong:
```dart
// 🚫 Duplicate relationships possible
class UserRoles extends Table {
  IntColumn get userId => integer()();
  IntColumn get roleId => integer()();
}
```

Correct:
```dart
// ✅ Prevent duplicates
class UserRoles extends Table {
  @override
  Set<Column> get primaryKey => {userId, roleId};
  IntColumn get userId => integer()();
  IntColumn get roleId => integer()();
}
```

## Mistake 2: Not handling duplicates

Wrong:
```dart
// 🚫 Inserting duplicate relationship fails
await into(userRoles).insert(companion);
```

Correct:
```dart
// ✅ Check before insert or use onConflict
await into(userRoles).insert(
  companion,
  onConflict: DoNothing(), // Skip duplicates
);
```

## Mistake 3: Missing indexes

Wrong:
```dart
// 🚫 Slow queries on large tables
class UserRoles extends Table {
  IntColumn get userId => integer()();
  IntColumn get roleId => integer()();
}
```

Correct:
```dart
// ✅ Add indexes for performance
class UserRoles extends Table {
  IntColumn get userId => integer()();
  IntColumn get roleId => integer()();
  
  @override
  List<Index> get indexes => [
    Index('idx_user_roles_user', 'user_id'),
    Index('idx_user_roles_role', 'role_id'),
  ];
}
```

---

# Summary

| Concept | Description | Key Feature |
|---------|-------------|-------------|
| **Junction Table** | Connects two tables | Composite primary key |
| **Many-to-Many** | Multiple relationships | Foreign keys |
| **Metadata** | Relationship data | Additional columns |
| **Query** | Get related records | Joins with junction |

---

# Next Steps

Now you understand many-to-many, let's dive deeper:

- [One-to-Many](link) – One-to-many relationships
- [One-to-One](link) – One-to-one relationships
- [Mapping Joined Results](link) – Result mapping

---

# Did You Know?

- **Many-to-many is common** – In relational databases

- **Junction tables have composite keys** – Prevent duplicates

- **Indexes are critical** – For many-to-many performance

- **Metadata can be stored** – In junction tables

- **Cascade delete is useful** – Clean up relationships

- **Transactions are important** – For consistency

- **Distinct may be needed** – To avoid duplicates

- **Many-to-many is powerful** – For complex relationships

---
