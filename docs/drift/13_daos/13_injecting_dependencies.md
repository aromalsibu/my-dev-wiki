
---

# Injecting Dependencies

Dependency injection is the practice of providing a DAO with the objects it needs—such as the database instance—instead of creating them internally.

---

# What is it?

Every DAO needs access to an instance of your database (`AppDatabase`) to execute queries.

Instead of creating a new database inside the DAO, the database is **injected** through the constructor.

```dart
UserDao(AppDatabase db) : super(db);
```

This allows the same database instance to be shared throughout your application.

Dependency injection also makes your code easier to test, easier to maintain, and more flexible.

---

# Why does it exist?

Imagine every DAO created its own database.

```dart
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {

  UserDao() : super(AppDatabase());
}
```

Now every `UserDao` creates a completely new database connection.

This causes several problems:

* Multiple unnecessary database instances.
* More memory usage.
* Difficult testing.
* Harder lifecycle management.
* Data may not be shared between DAOs as expected.

Instead, one database instance should be shared.

```text
                AppDatabase
                     │
      ┌──────────────┴──────────────┐
      ▼                             ▼
   UserDao                     ProductDao
      │                             │
      ▼                             ▼
 User Queries                 Product Queries
```

Every DAO communicates with the same database.

---

# Syntax

## Constructor Injection

The most common way to inject dependencies is through the constructor.

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {

  UserDao(AppDatabase db) : super(db);
}
```

Explanation:

* The constructor receives an existing `AppDatabase`.
* `super(db)` passes the database to `DatabaseAccessor`.
* The DAO never creates its own database.

---

## Creating the Database

```dart
final database = AppDatabase();

final userDao = UserDao(database);
```

Explanation:

* One database instance is created.
* The same instance is passed to the DAO.
* The DAO is now ready to execute queries.

---

## Sharing the Database Between Multiple DAOs

```dart
final database = AppDatabase();

final userDao = UserDao(database);
final productDao = ProductDao(database);
final orderDao = OrderDao(database);
```

Explanation:

* Every DAO shares the same database.
* All DAOs operate on the same SQLite connection.
* This is the recommended approach.

---

# Mental Model

Think of the database as a shared resource.

```text
                 AppDatabase
                      │
      ┌───────────────┼───────────────┐
      │               │               │
      ▼               ▼               ▼
  UserDao       ProductDao      OrderDao
```

The DAOs don't own the database.

They simply receive access to it.

---

# Examples

## Simple Example

A DAO receiving the database through its constructor.

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {

  UserDao(AppDatabase db) : super(db);

  Future<List<User>> getUsers() {
    return select(users).get();
  }
}
```

Explanation:

* The DAO depends on an existing database.
* It doesn't create or manage the database itself.

---

## Real-World Example

Suppose an application has several DAOs.

```dart
final database = AppDatabase();

final userDao = UserDao(database);
final productDao = ProductDao(database);
final orderDao = OrderDao(database);

await userDao.createUser(
  UsersCompanion.insert(name: 'John'),
);

final products = await productDao.getProducts();

final orders = await orderDao.getOrders();
```

Explanation:

* A single `AppDatabase` instance is created.
* Every DAO shares that instance.
* Changes made through one DAO are immediately visible to the others because they're using the same database.

---

# When to Use

Inject dependencies when:

* Creating any DAO.
* Sharing one database across your application.
* Writing testable code.
* Using dependency injection frameworks like Provider, Riverpod, GetIt, or injectable.

In practice, every Drift application should inject the database instead of creating it inside a DAO.

---

# When NOT to Use

Avoid creating the database inside the DAO.

For example:

```dart
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {

  UserDao() : super(AppDatabase());
}
```

This tightly couples the DAO to a specific database instance and makes testing much more difficult.

---

# Best Practices

* Create a single `AppDatabase` instance.
* Pass the database through the constructor.
* Share the same database across all DAOs.
* Let a dependency injection framework manage object creation in larger applications.
* Keep DAOs responsible only for database operations.

---

# Common Mistakes

## Creating the database inside the DAO

**Wrong**

```dart
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {

  UserDao() : super(AppDatabase());
}
```

Why it's wrong:

Every DAO creates its own database instance, leading to unnecessary resource usage and making testing difficult.

**Correct**

```dart
final database = AppDatabase();

final userDao = UserDao(database);
```

The database is created once and shared.

---

## Creating multiple database instances

**Wrong**

```dart
final userDao = UserDao(AppDatabase());

final productDao = ProductDao(AppDatabase());

final orderDao = OrderDao(AppDatabase());
```

Why it's wrong:

Each DAO uses a different database instance.

**Correct**

```dart
final database = AppDatabase();

final userDao = UserDao(database);
final productDao = ProductDao(database);
final orderDao = OrderDao(database);
```

All DAOs share the same database.

---

## Letting the DAO manage the database lifecycle

**Wrong**

```dart
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {

  UserDao(AppDatabase db) : super(db);

  Future<void> closeDatabase() async {
    await attachedDatabase.close();
  }
}
```

Why it's wrong:

The DAO should not decide when the database is opened or closed.

**Correct**

Manage the database lifecycle from a higher level, such as your application's startup and shutdown logic.

---

# Related APIs

* `DatabaseAccessor`
* `GeneratedDatabase`
* `AppDatabase`
* `attachedDatabase`
* `Provider`
* `Riverpod`
* `GetIt`

---

# Summary

Dependency injection allows a DAO to receive an existing database instance instead of creating one itself. Sharing a single `AppDatabase` across all DAOs improves performance, simplifies testing, and keeps responsibilities clearly separated. In production Drift applications, DAOs should always receive their dependencies through constructor injection.
