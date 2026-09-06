# Organizing Queries

Organizing queries means grouping related database operations into well-structured, feature-focused DAOs to keep your code maintainable and easy to navigate.

---

# What is it?

As your application grows, so does the number of database queries. Instead of placing dozens of queries in a single DAO—or worse, scattering them throughout your application—you should organize them logically.

A well-organized DAO contains queries that belong to the same feature or domain.

For example:

* `UserDao` contains user-related queries.
* `ProductDao` contains product-related queries.
* `OrderDao` contains order-related queries.

Each DAO acts as the single source of truth for interacting with its part of the database.

---

# Why does it exist?

Imagine an application with hundreds of database queries.

If every query is placed inside one DAO:

```text
AppDao
├── getUsers()
├── getProducts()
├── getOrders()
├── getCategories()
├── getMessages()
├── getReviews()
├── getCart()
├── getPayments()
├── updateProfile()
├── deleteProduct()
├── ...
```

Finding or updating a query becomes increasingly difficult.

Instead, organize queries by feature:

```text
UserDao
├── getUsers()
├── getUserById()
├── createUser()
└── deleteUser()

ProductDao
├── getProducts()
├── searchProducts()
└── updateProduct()

OrderDao
├── createOrder()
├── watchOrders()
└── cancelOrder()
```

Each DAO becomes smaller, easier to understand, and simpler to maintain.

---

# Syntax

## Group Related Queries

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);

  Future<List<User>> getAllUsers() {
    return select(users).get();
  }

  Future<User?> getUserById(int id) {
    return (select(users)
          ..where((u) => u.id.equals(id)))
        .getSingleOrNull();
  }

  Future<int> createUser(UsersCompanion user) {
    return into(users).insert(user);
  }

  Future<bool> deleteUser(int id) {
    return (delete(users)
          ..where((u) => u.id.equals(id)))
        .go();
  }
}
```

Explanation:

* Every method is related to the `Users` table.
* The DAO has a single responsibility.
* Related queries are easy to locate.

---

## Separate Features into Different DAOs

```dart
@DriftAccessor(tables: [Products])
class ProductDao extends DatabaseAccessor<AppDatabase>
    with _$ProductDaoMixin {
  ProductDao(AppDatabase db) : super(db);

  Future<List<Product>> getProducts() {
    return select(products).get();
  }

  Future<int> addProduct(ProductsCompanion product) {
    return into(products).insert(product);
  }
}
```

Explanation:

* Product-related queries are separated from user queries.
* Each DAO focuses on one feature.
* This improves readability as the project grows.

---

# Mental Model

Think of each DAO as a department in a company.

```text
                AppDatabase
                     │
     ┌───────────────┼───────────────┐
     │               │               │
     ▼               ▼               ▼
 UserDao       ProductDao      OrderDao
     │               │               │
     ▼               ▼               ▼
 User Queries  Product Queries  Order Queries
```

Instead of one giant DAO handling everything, each DAO manages a specific area of the application.

---

# Examples

## Simple Example

A user feature with all related queries in one DAO.

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);

  Future<List<User>> getUsers() {
    return select(users).get();
  }

  Stream<List<User>> watchUsers() {
    return select(users).watch();
  }

  Future<int> addUser(UsersCompanion user) {
    return into(users).insert(user);
  }

  Future<bool> deleteUser(int id) {
    return (delete(users)
          ..where((u) => u.id.equals(id)))
        .go();
  }
}
```

Explanation:

* Every method performs a user-related operation.
* The DAO has a clear and focused responsibility.

---

## Real-World Example

An e-commerce application organized by feature.

```dart
// user_dao.dart

@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);

  Future<List<User>> getUsers() => select(users).get();

  Future<int> createUser(UsersCompanion user) =>
      into(users).insert(user);
}

// product_dao.dart

@DriftAccessor(tables: [Products])
class ProductDao extends DatabaseAccessor<AppDatabase>
    with _$ProductDaoMixin {
  ProductDao(AppDatabase db) : super(db);

  Future<List<Product>> getProducts() =>
      select(products).get();

  Future<int> addProduct(ProductsCompanion product) =>
      into(products).insert(product);
}

// order_dao.dart

@DriftAccessor(
  tables: [
    Orders,
    OrderItems,
  ],
)
class OrderDao extends DatabaseAccessor<AppDatabase>
    with _$OrderDaoMixin {
  OrderDao(AppDatabase db) : super(db);

  Future<List<Order>> getOrders() =>
      select(orders).get();

  Stream<List<Order>> watchOrders() =>
      select(orders).watch();
}
```

Explanation:

* Each DAO owns a specific feature.
* Queries are grouped by responsibility instead of by operation.
* Developers know exactly where to look when modifying database code.

---

# When to Use

Organize queries when:

* Your project contains multiple features.
* A DAO starts growing beyond a few methods.
* Queries are reused throughout the application.
* Multiple developers work on the same project.

Feature-based organization scales much better than one large DAO.

---

# When NOT to Use

You don't need extensive organization when:

* You're learning Drift with a simple example.
* Your application has only a handful of queries.
* A single DAO is sufficient for the entire project.

Avoid creating unnecessary DAOs for very small projects.

---

# Best Practices

* Organize DAOs by feature, not by query type.
* Keep each DAO focused on one responsibility.
* Use descriptive method names.
* Keep methods short and focused.
* Reuse query methods instead of duplicating them.
* Place related helper methods close to the queries that use them.
* Keep business logic outside the DAO whenever possible.

---

# Common Mistakes

## Creating one giant DAO

**Wrong**

```text
AppDao
├── Users
├── Products
├── Orders
├── Categories
├── Reviews
├── Payments
├── Cart
├── Notifications
├── Messages
└── Settings
```

Why it's wrong:

A large DAO becomes difficult to navigate, test, and maintain.

**Correct**

```text
UserDao
ProductDao
OrderDao
CartDao
ReviewDao
```

Each DAO has a clear responsibility.

---

## Organizing by CRUD operation

**Wrong**

```text
InsertDao
UpdateDao
DeleteDao
SelectDao
```

Why it's wrong:

Queries for the same feature become scattered across multiple files.

**Correct**

```text
UserDao
ProductDao
OrderDao
```

Keep all operations for a feature together.

---

## Mixing unrelated features

**Wrong**

```dart
class UserDao {
  getUsers()

  getProducts()

  createOrder()

  deleteCategory()
}
```

Why it's wrong:

The DAO loses its focus and becomes harder to understand.

**Correct**

```dart
class UserDao {
  getUsers()

  createUser()

  deleteUser()
}
```

Each DAO should manage only one feature or a closely related group of tables.

---

# Related APIs

* `@DriftAccessor`
* `DatabaseAccessor`
* `Select`
* `Insert`
* `Update`
* `Delete`
* `Transactions`

---

# Summary

Organizing queries is about structuring your DAOs around **features**, not individual operations. Grouping related database logic into focused DAOs keeps your code easier to navigate, reduces duplication, and makes applications more maintainable as they grow. A well-organized DAO should have a single responsibility and contain all the queries needed for that part of the application.
