# @DriftAccessor

`@DriftAccessor` is an annotation that tells Drift which database objects a DAO can access, allowing Drift to generate the code required for that DAO.

---

# What is it?

`@DriftAccessor` is placed above a DAO class to declare the tables and views that the DAO will work with.

When Drift generates code, it reads this annotation to determine:

* Which tables should be available inside the DAO.
* Which views should be available inside the DAO.
* Which helper code needs to be generated.

Without this annotation, Drift doesn't know what your DAO should have access to.

For example:

```dart
@DriftAccessor(
  tables: [Users],
)
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

Here, `UserDao` is allowed to work with the `Users` table.

---

# Why does it exist?

Drift generates much of a DAO's boilerplate automatically. To do that, it needs to know which database objects belong to the DAO.

Instead of manually creating references to every table:

```text
UserDao
├── Users users
├── Orders orders
└── Products products
```

You simply declare them:

```dart
@DriftAccessor(
  tables: [
    Users,
    Orders,
    Products,
  ],
)
```

Drift generates the required code for you.

This keeps your DAO clean, type-safe, and easy to maintain.

---

# What Can You Register?

A DAO can register:

* Tables
* Views

Most DAOs only register tables.

If your DAO needs to query a database view, you can register that view as well.

---

# Syntax

## Registering a Single Table

```dart
import 'package:drift/drift.dart';

part 'user_dao.g.dart';

@DriftAccessor(
  tables: [Users],
)
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

Explanation:

* `tables` lists every table the DAO needs.
* Drift generates a getter for the `Users` table.
* The generated mixin makes the table available inside the DAO.

---

## Registering Multiple Tables

Some features naturally work with multiple tables.

```dart
@DriftAccessor(
  tables: [
    Orders,
    OrderItems,
    Products,
  ],
)
class OrderDao extends DatabaseAccessor<AppDatabase>
    with _$OrderDaoMixin {
  OrderDao(AppDatabase db) : super(db);
}
```

Explanation:

* The DAO can directly access all three tables.
* This is common when writing joins or managing related data.

---

## Registering Views

A DAO can also access database views.

```dart
@DriftAccessor(
  tables: [Users],
  views: [ActiveUsersView],
)
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

Explanation:

* `views` registers Drift views.
* Views become available just like tables.
* This is useful for reusable SQL queries.

---

## Generate the Code

After creating or modifying a DAO, regenerate Drift's generated files.

```bash
dart run build_runner build
```

Or during development:

```bash
dart run build_runner watch
```

Explanation:

* `build` generates the `.g.dart` file once.
* `watch` automatically regenerates files whenever your Drift code changes.
* Without code generation, the generated mixin won't exist.

---

# Mental Model

Think of `@DriftAccessor` as a **permission list** for a DAO.

```text
               @DriftAccessor
                     │
     ┌───────────────┴───────────────┐
     │                               │
 Tables: [Users]          Views: [ActiveUsersView]
     │                               │
     └───────────────┬───────────────┘
                     │
                     ▼
          Drift Code Generator
                     │
                     ▼
             _$UserDaoMixin
                     │
                     ▼
        users      activeUsersView
```

The annotation doesn't perform any database operations.

It simply tells Drift what should be generated for the DAO.

---

# Examples

## Simple Example

A DAO that only works with users.

```dart
@DriftAccessor(
  tables: [Users],
)
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);

  Future<List<User>> getUsers() {
    return select(users).get();
  }
}
```

Explanation:

* The DAO registers the `Users` table.
* Drift generates the `users` getter.
* The query can use `users` directly.

---

## Real-World Example

An order management feature often needs multiple related tables.

```dart
@DriftAccessor(
  tables: [
    Orders,
    OrderItems,
    Products,
    Customers,
  ],
)
class OrderDao extends DatabaseAccessor<AppDatabase>
    with _$OrderDaoMixin {
  OrderDao(AppDatabase db) : super(db);

  Future<List<Order>> getOrders() {
    return select(orders).get();
  }

  Future<int> createOrder(
    OrdersCompanion order,
  ) {
    return into(orders).insert(order);
  }

  Stream<List<Order>> watchOrders() {
    return select(orders).watch();
  }
}
```

Explanation:

* One DAO manages several related tables.
* Drift generates getters for each registered table.
* All order-related database operations remain in one place.

---

# @DriftAccessor vs Registering a DAO

It's important not to confuse `@DriftAccessor` with registering a DAO in your database.

`@DriftAccessor` defines **what a DAO can access**.

```dart
@DriftAccessor(
  tables: [Users],
)
class UserDao ...
```

Registering a DAO tells the database **which DAOs belong to it**.

```dart
@DriftDatabase(
  tables: [Users],
  daos: [UserDao],
)
class AppDatabase extends _$AppDatabase {
  ...
}
```

Both steps are required.

* `@DriftAccessor` configures the DAO.
* `@DriftDatabase(daos: [...])` registers the DAO with the database.

The next page covers DAO registration in detail.

---

# When to Use

Use `@DriftAccessor` whenever you create a DAO.

Every Drift DAO should have this annotation.

---

# When NOT to Use

Do not use `@DriftAccessor` on:

* Your database class.
* Repository classes.
* Service classes.
* Widgets.
* Regular Dart classes.

It is intended only for DAOs.

---

# Best Practices

* Register only the tables the DAO actually uses.
* Group related tables in the same DAO.
* Avoid registering unrelated tables.
* Regenerate code after modifying the annotation.
* Keep each DAO focused on a single feature.

---

# Common Mistakes

## Forgetting the Annotation

**Wrong**

```dart
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

Why it's wrong:

Drift doesn't know which tables belong to the DAO, so it cannot generate the required code.

**Correct**

```dart
@DriftAccessor(
  tables: [Users],
)
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

---

## Forgetting to Run Code Generation

**Wrong**

```text
1. Create DAO
2. Run the application
```

Why it's wrong:

The generated mixin won't exist, resulting in compilation errors.

**Correct**

```text
1. Create DAO
2. Run build_runner
3. Run the application
```

---

## Registering Unrelated Tables

**Wrong**

```dart
@DriftAccessor(
  tables: [
    Users,
    Orders,
    Products,
    Messages,
    Notifications,
  ],
)
```

Why it's wrong:

The DAO becomes responsible for unrelated parts of the application.

**Correct**

```dart
@DriftAccessor(
  tables: [
    Orders,
    OrderItems,
    Products,
  ],
)
```

Keep the DAO focused on one feature.

---

## Confusing `@DriftAccessor` with DAO Registration

**Wrong assumption**

> "Adding `@DriftAccessor` automatically makes my DAO available in the database."

Why it's wrong:

`@DriftAccessor` only defines what the DAO can access.

You must also register the DAO using the `daos` parameter in `@DriftDatabase`.

---

# Related APIs

* `DatabaseAccessor`
* `@DriftDatabase`
* Generated DAO Mixins
* Tables
* Views
* Code Generation

---

# Summary

`@DriftAccessor` is the annotation that defines which tables and views a DAO can access. Drift reads this information during code generation to create the helper code that powers your DAO. It configures the DAO itself but **does not register the DAO with the database**—that is done separately using the `daos` parameter in `@DriftDatabase`.
