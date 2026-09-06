The next page is **Best Practices** (since we skipped the implementation-focused pages like CRUD Inside DAOs, Watching Data, etc.).

---

# Best Practices

Following a few simple practices when working with DAOs makes your Drift code easier to maintain, test, and scale as your application grows.

---

# What is it?

A DAO is more than just a place to put queries—it's an architectural boundary between your application and the database.

Well-designed DAOs:

* Have a single responsibility.
* Expose clean, reusable methods.
* Keep database logic in one place.
* Avoid leaking implementation details to the rest of the application.

Poorly designed DAOs quickly become difficult to understand and maintain.

---

# Why does it exist?

Without consistent practices, DAOs often become:

* Very large classes.
* A mix of unrelated queries.
* Filled with business logic.
* Difficult to test.
* Hard to reuse.

Following a consistent structure keeps the data layer predictable for everyone working on the project.

---

# Best Practices

## 1. Organize DAOs by Feature

Create a separate DAO for each feature or related group of tables.

**Good**

```text
UserDao
ProductDao
OrderDao
CartDao
```

**Avoid**

```text
AppDao
```

Feature-based DAOs are easier to navigate and maintain.

---

## 2. Keep a Single Responsibility

A DAO should manage one area of the database.

**Good**

```dart
class UserDao {
  getUsers()

  createUser()

  deleteUser()
}
```

Everything relates to users.

**Avoid**

```dart
class UserDao {
  getUsers()

  getProducts()

  createOrder()

  deleteCategory()
}
```

Mixing unrelated features makes the DAO harder to understand.

---

## 3. Keep Business Logic Outside the DAO

A DAO should perform database operations—not business workflows.

**Good**

```dart
Future<int> createOrder(
  OrdersCompanion order,
) {
  return into(orders).insert(order);
}
```

The DAO performs one database operation.

Business logic belongs elsewhere.

---

## 4. Give Methods Descriptive Names

Method names should clearly describe what they do.

**Good**

```dart
getUserById()

watchActiveUsers()

deleteUser()

updateProfile()
```

Avoid vague names.

```dart
get()

load()

fetch()

run()
```

Clear names improve readability.

---

## 5. Reuse Existing Queries

Instead of duplicating queries, reuse existing DAO methods whenever possible.

**Instead of**

```dart
Future<User?> findUser(int id) { ... }

Future<User?> loadUser(int id) { ... }
```

Prefer one reusable method.

```dart
Future<User?> getUserById(int id) { ... }
```

---

## 6. Keep Methods Small

Each DAO method should perform one operation.

**Good**

```dart
Future<int> createUser(...) { ... }

Future<bool> deleteUser(...) { ... }

Future<List<User>> getUsers() { ... }
```

Small methods are easier to test and maintain.

---

## 7. Share One Database Instance

Create one `AppDatabase` and inject it into all DAOs.

```dart
final database = AppDatabase();

final userDao = UserDao(database);

final orderDao = OrderDao(database);
```

Avoid creating a new database inside every DAO.

---

## 8. Prefer Drift's Query Builder

Use Drift's fluent API whenever possible.

```dart
return select(users).get();
```

Only use raw SQL when the query builder cannot express your query cleanly.

---

## 9. Return the Appropriate Type

Return the type that best matches the operation.

```dart
Future<List<User>>

Future<User?>

Future<int>

Future<bool>

Stream<List<User>>
```

Avoid returning overly generic types such as `dynamic` or `Object`.

---

## 10. Keep Related Queries Together

For example:

```text
UserDao
├── Reads
├── Writes
├── Streams
└── Helper methods
```

Keeping similar operations together makes the DAO easier to navigate.

---

# Mental Model

Think of a DAO as the **database API** for one feature.

```text
UI
 │
 ▼
Repository / Service
 │
 ▼
UserDao
 │
 ├── getUsers()
 ├── createUser()
 ├── deleteUser()
 └── watchUsers()
 │
 ▼
Database
```

Everything related to users goes through `UserDao`.

---

# Examples

## Simple Example

A focused DAO.

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {

  UserDao(AppDatabase db) : super(db);

  Future<List<User>> getUsers() {
    return select(users).get();
  }

  Future<int> createUser(
    UsersCompanion user,
  ) {
    return into(users).insert(user);
  }

  Future<bool> deleteUser(int id) {
    return (delete(users)
          ..where((u) => u.id.equals(id)))
        .go();
  }
}
```

This DAO is focused entirely on user-related operations.

---

## Real-World Example

A project organized with multiple focused DAOs.

```text
lib/
│
├── database/
│   ├── app_database.dart
│   │
│   ├── daos/
│   │   ├── user_dao.dart
│   │   ├── product_dao.dart
│   │   ├── order_dao.dart
│   │   ├── cart_dao.dart
│   │   └── category_dao.dart
│   │
│   └── tables/
│       ├── users.dart
│       ├── products.dart
│       ├── orders.dart
│       └── categories.dart
```

This structure keeps related files together, making it easier to find and maintain database code as the project grows.

---

# When to Use

Apply these practices in:

* Small projects that may grow over time.
* Medium-sized applications.
* Large production applications.
* Team-based development.

The earlier you adopt these practices, the easier your project will be to maintain.

---

# When NOT to Use

These practices are recommendations, not strict rules.

For quick experiments or tutorials, it's acceptable to simplify your DAOs. However, production applications benefit greatly from following these guidelines.

---

# Common Mistakes

## One DAO for Everything

**Wrong**

```text
AppDao
```

Contains hundreds of unrelated queries.

**Correct**

```text
UserDao

ProductDao

OrderDao

CartDao
```

Split DAOs by feature.

---

## Mixing Business Logic

**Wrong**

```dart
Future<void> checkout() async {
  // Validate payment

  // Call API

  // Send email

  // Insert order
}
```

A DAO should not manage application workflows.

**Correct**

```dart
Future<int> createOrder(
  OrdersCompanion order,
) {
  return into(orders).insert(order);
}
```

Keep the DAO focused on database operations.

---

## Creating Multiple Databases

**Wrong**

```dart
UserDao(AppDatabase());

ProductDao(AppDatabase());
```

Each DAO uses a different database instance.

**Correct**

```dart
final database = AppDatabase();

UserDao(database);

ProductDao(database);
```

Share one database throughout the application.

---

# Related APIs

* `DatabaseAccessor`
* `@DriftAccessor`
* `GeneratedDatabase`
* `Select`
* `Insert`
* `Update`
* `Delete`
* `Transactions`

---

# Summary

A well-designed DAO is small, focused, and responsible only for database operations. Organize DAOs by feature, keep methods descriptive, avoid mixing business logic with persistence, and share a single database instance across your application. Following these practices results in cleaner, more maintainable Drift code that scales well as your project grows.
