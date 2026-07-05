# Defining DAOs

A **DAO is defined by creating a class that extends `DatabaseAccessor` and annotating it with `@DriftAccessor`.**

---

# What is it?

Defining a DAO means creating a class that Drift recognizes as a **Data Access Object**. This class becomes the home for all database operations related to one or more tables.

Unlike a regular Dart class, a Drift DAO must follow a specific structure so that Drift can generate the code needed to access your tables.

A typical DAO consists of:

* The `@DriftAccessor` annotation.
* A class extending `DatabaseAccessor`.
* A generated mixin.
* A constructor that receives the database instance.

Once defined, you can start adding query methods such as inserts, updates, deletes, selects, and streams.

---

# Why does it exist?

Drift needs to know:

* Which tables the DAO can access.
* Which database it belongs to.
* What helper code should be generated.

Instead of manually wiring everything together, Drift generates much of the boilerplate for you.

Without this structure, every query would need to reference tables manually, making code more verbose and error-prone.

---

# Syntax

A minimal DAO looks like this.

```dart
import 'package:drift/drift.dart';

import 'database.dart';
import 'tables/users.dart';

part 'user_dao.g.dart';

@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

Explanation:

* `part 'user_dao.g.dart';` includes the generated code for this DAO.
* `@DriftAccessor` registers the tables accessible from this DAO.
* `DatabaseAccessor<AppDatabase>` connects the DAO to your database.
* `_$UserDaoMixin` provides getters such as `users`.
* `super(db)` stores the database instance inside the accessor.

Although this DAO doesn't contain any methods yet, it is fully configured and ready for queries.

---

# Mental Model

Think of a DAO definition as **registering a department inside your database**.

```text
AppDatabase
      │
      ├─────────────┐
      │             │
      ▼             ▼
  UserDao      ProductDao
      │             │
      ▼             ▼
 Users Table   Products Table
```

Each DAO declares which tables it is responsible for.

When Drift generates code, it gives each DAO direct access only to the tables it registered.

---

# Examples

## Simple Example

A DAO responsible for the `Users` table.

```dart
import 'package:drift/drift.dart';

part 'user_dao.g.dart';

@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

Explanation:

* This DAO can access the `Users` table.
* Query methods will be added inside this class.
* Drift generates the `users` table getter automatically.

---

## Real-World Example

Suppose an e-commerce application has several tables.

```dart
import 'package:drift/drift.dart';

part 'order_dao.g.dart';

@DriftDatabase(
  tables: [Users, Orders, OrderItems, Products],
  daos: [UserDao, OrderDao],
)
class AppDatabase extends _$AppDatabase {
  AppDatabase(QueryExecutor e) : super(e);

  @override
  int get schemaVersion => 1;
}

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

  // Order-related queries will be added here.
}
```

Explanation:

* This DAO can work with three related tables.
* Since orders often require products and order items, grouping them in one DAO makes sense.
* Drift generates getters for `orders`, `orderItems`, and `products`.

---

After creating a new DAO, generate the required code by running:

```bash
dart run build_runner build
```

Or, during development:

```bash
dart run build_runner watch
```

Explanation:

- `build` generates the DAO's `.g.dart` file once.
- `watch` automatically regenerates code whenever your Drift files change.
- Without running code generation, the generated mixin (such as `_$UserDaoMixin`) will not exist, resulting in compilation errors.

# When to Use

Define a DAO when:

* A feature has multiple database operations.
* Several queries belong together.
* Multiple screens need the same database logic.
* You want to organize database code by feature.

---

# When NOT to Use

You don't need a new DAO for:

* Every individual table.
* Every single query.
* Tiny demo applications with only one or two queries.

It's common for one DAO to manage multiple related tables.

---

# Best Practices

* Group related tables in the same DAO.
* Name DAOs after the feature they manage.
* Keep DAOs focused on one responsibility.
* Add only related tables to `@DriftAccessor`.
* Create separate DAOs for unrelated features.

---

# Common Mistakes

## Forgetting the generated part file

**Wrong**

```dart
import 'package:drift/drift.dart';

@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

Why it's wrong:

Without the `part` directive, Drift cannot include the generated code, causing compilation errors.

**Correct**

```dart
import 'package:drift/drift.dart';

part 'user_dao.g.dart';

@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

The generated file becomes part of the library.

---

## Forgetting to extend `DatabaseAccessor`

**Wrong**

```dart
@DriftAccessor(tables: [Users])
class UserDao {}
```

Why it's wrong:

The DAO won't have access to the database or Drift helper methods.

**Correct**

```dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

Extending `DatabaseAccessor` connects the DAO to the database.

---

## Registering unrelated tables

**Wrong**

```dart
@DriftAccessor(
  tables: [
    Users,
    Orders,
    Products,
    Messages,
    Notifications,
    Settings,
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

Only register the tables the DAO actually manages.

---

# Related APIs

* `@DriftAccessor`
* `DatabaseAccessor`
* `Generated DAO Mixins`
* `GeneratedDatabase`
* `Tables`

---

# Summary

Defining a DAO involves creating a class that extends `DatabaseAccessor`, annotating it with `@DriftAccessor`, and including the generated mixin. This structure allows Drift to generate helper code, provide access to registered tables, and organize database operations into reusable, feature-focused classes.
