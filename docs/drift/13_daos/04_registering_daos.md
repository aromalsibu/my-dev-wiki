
---

# Registering DAOs

Registering a DAO makes it part of your database, allowing Drift to generate and expose it through your `AppDatabase`.

---

# What is it?

After defining a DAO, you must register it with your database using the `daos` parameter of `@DriftDatabase`.

Until a DAO is registered, Drift doesn't know that it belongs to your database.

Registering a DAO enables Drift to:

* Generate a getter for the DAO.
* Connect the DAO to the database.
* Inject the database instance automatically.
* Make the DAO accessible from `AppDatabase`.

Without registration, your DAO cannot be used through the database.

---

# Why does it exist?

Imagine you create a `UserDao`.

```dart
@DriftAccessor(
  tables: [Users],
)
class UserDao extends DatabaseAccessor<AppDatabase>
    with _$UserDaoMixin {
  UserDao(AppDatabase db) : super(db);
}
```

At this point, Drift knows how the DAO works.

However, the database still doesn't know that this DAO exists.

By registering it:

```dart
@DriftDatabase(
  tables: [Users],
  daos: [UserDao],
)
```

Drift connects the DAO to your database and generates everything needed to access it.

---

# Syntax

## Registering a Single DAO

```dart
import 'package:drift/drift.dart';

part 'database.g.dart';

@DriftDatabase(
  tables: [Users],
  daos: [UserDao],
)
class AppDatabase extends _$AppDatabase {
  AppDatabase(QueryExecutor e) : super(e);

  @override
  int get schemaVersion => 1;
}
```

Explanation:

* `daos` lists every DAO belonging to the database.
* Drift generates the code required to instantiate the DAO.
* The DAO becomes available through the database.

---

## Registering Multiple DAOs

Large applications typically have several DAOs.

```dart
@DriftDatabase(
  tables: [
    Users,
    Products,
    Orders,
    OrderItems,
  ],
  daos: [
    UserDao,
    ProductDao,
    OrderDao,
  ],
)
class AppDatabase extends _$AppDatabase {
  AppDatabase(QueryExecutor e) : super(e);

  @override
  int get schemaVersion => 1;
}
```

Explanation:

* Every DAO is registered once.
* Drift generates accessors for each DAO.
* All DAOs share the same database instance.

---

## Generate the Code

After registering a DAO, regenerate Drift's code.

```bash
dart run build_runner build
```

Or during development:

```bash
dart run build_runner watch
```

Explanation:

* Drift updates the generated database code.
* New DAO getters are generated.
* Your application can now access the registered DAO.

---

# Mental Model

Think of `@DriftAccessor` and `@DriftDatabase` as two separate registration steps.

```text
             Create DAO
                  │
                  ▼
         @DriftAccessor
                  │
     Defines what the DAO
        can access
                  │
                  ▼
        @DriftDatabase
      daos: [UserDao]
                  │
      Registers the DAO
      with the database
                  │
                  ▼
      database.userDao
```

One configures the DAO.

The other makes it part of the database.

---

# Examples

## Simple Example

Registering a single DAO.

```dart
@DriftDatabase(
  tables: [Users],
  daos: [UserDao],
)
class AppDatabase extends _$AppDatabase {
  AppDatabase(QueryExecutor e) : super(e);

  @override
  int get schemaVersion => 1;
}
```

Explanation:

* `UserDao` is now part of the database.
* Drift generates everything required to use it.

---

## Real-World Example

An e-commerce application with multiple DAOs.

```dart
@DriftDatabase(
  tables: [
    Users,
    Products,
    Orders,
    OrderItems,
    Categories,
  ],
  daos: [
    UserDao,
    ProductDao,
    OrderDao,
    CategoryDao,
  ],
)
class AppDatabase extends _$AppDatabase {
  AppDatabase(QueryExecutor e) : super(e);

  @override
  int get schemaVersion => 1;
}
```

Now the database can expose every registered DAO.

```dart
final database = AppDatabase(connection);

await database.userDao.getUsers();

await database.productDao.getProducts();

await database.orderDao.getOrders();
```

Explanation:

* Drift generates a getter for every registered DAO.
* Each DAO automatically receives the same database instance.
* No manual DAO construction is required.

---

# What Happens Behind the Scenes?

When Drift generates your database, it creates a getter for every registered DAO.

Conceptually, it generates something similar to this:

```dart
late final UserDao userDao = UserDao(this);

late final ProductDao productDao = ProductDao(this);

late final OrderDao orderDao = OrderDao(this);
```

Explanation:

* `this` refers to the current `AppDatabase`.
* Each DAO automatically receives the database instance.
* Drift generates this code for you.

The actual generated implementation may differ, but this illustrates the idea.

---

# When to Use

Register every DAO that belongs to your database.

Whenever you create a new DAO:

1. Define it.
2. Register it.
3. Regenerate the code.

---

# When NOT to Use

Do not register:

* Repository classes.
* Service classes.
* Widgets.
* Utility classes.

Only classes annotated with `@DriftAccessor` should appear in the `daos` list.

---

# Best Practices

* Register every DAO exactly once.
* Keep the `daos` list organized.
* Regenerate code after adding or removing DAOs.
* Use one `AppDatabase` to manage all DAOs.
* Group related database functionality into separate DAOs.

---

# Common Mistakes

## Forgetting to Register the DAO

**Wrong**

```dart
@DriftDatabase(
  tables: [Users],
)
class AppDatabase extends _$AppDatabase {}
```

Why it's wrong:

The database doesn't know about `UserDao`.

**Correct**

```dart
@DriftDatabase(
  tables: [Users],
  daos: [UserDao],
)
class AppDatabase extends _$AppDatabase {}
```

---

## Forgetting Code Generation

**Wrong**

```text
1. Add UserDao
2. Run the app
```

Why it's wrong:

The generated database won't contain the new DAO.

**Correct**

```text
1. Add UserDao
2. Register UserDao
3. Run build_runner
4. Run the app
```

---

## Registering a Non-DAO Class

**Wrong**

```dart
@DriftDatabase(
  daos: [
    UserRepository,
  ],
)
```

Why it's wrong:

Only DAO classes belong in the `daos` list.

**Correct**

```dart
@DriftDatabase(
  daos: [
    UserDao,
  ],
)
```

---

# Related APIs

* `@DriftDatabase`
* `@DriftAccessor`
* `DatabaseAccessor`
* Generated DAO Mixins
* Code Generation

---

# Summary

Registering a DAO connects it to your database. By adding the DAO to the `daos` parameter of `@DriftDatabase`, Drift generates the necessary code to instantiate and expose it through `AppDatabase`. Defining a DAO and registering it are two separate steps—both are required before the DAO can be used effectively.
