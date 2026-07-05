# Views

**Views** are virtual tables that store a query instead of data, allowing you to reuse complex SQL queries as if they were regular tables.

---

# What is it?

A view is a named SQL query.

Unlike a table, a view doesn't store its own data. Instead, whenever you query a view, SQLite executes the underlying query and returns the result.

You can think of a view as a saved query that behaves like a table.

For example, instead of repeatedly writing a query to join `users` and `profiles`, you can define a view once and query it whenever needed.

---

# Why does it exist?

As applications grow, some queries become:

* Long
* Complex
* Reused in many places

Copying the same SQL everywhere makes your code harder to maintain.

Views solve this by moving the query into a reusable database object.

Instead of writing:

```sql
SELECT ...
FROM users
JOIN profiles ...
WHERE ...
```

multiple times, you can simply query:

```sql
SELECT * FROM user_profiles
```

This improves readability, maintainability, and code reuse.

---

# Syntax

Define a view.

```dart
class UserProfiles extends View {
  Users get users => attachedDatabase.users;
  Profiles get profiles => attachedDatabase.profiles;

  @override
  Query as() => select([
        users.id,
        users.name,
        profiles.bio,
      ]).from(users).join([
        innerJoin(
          profiles,
          profiles.userId.equalsExp(users.id),
        ),
      ]);
}
```

Explanation:

* `View` is the base class for defining database views.
* `as()` returns the query that defines the view.
* Drift generates code so the view can be queried like a table.

---

Query the view.

```dart
final result = await select(userProfiles).get();
```

Explanation:

* The generated view behaves like a regular table.
* Drift executes the underlying query automatically.

---

Watch the view reactively.

```dart
final stream = select(userProfiles).watch();
```

Explanation:

* Since the view depends on other tables, updates to those tables automatically refresh the stream.

---

# Mental Model

Think of a view as a reusable window into your database.

```text
Users Table
      │
      ▼
Profiles Table
      │
      ▼
     View
      │
      ▼
Application
```

The view doesn't contain its own data.

Instead, every time you query it:

```text
Query View
     │
     ▼
Run Underlying Query
     │
     ▼
Return Results
```

It's always based on the latest data in the underlying tables.

---

# Examples

## Simple Example

Suppose you frequently need a user's name and email.

```dart
class UserSummary extends View {
  Users get users => attachedDatabase.users;

  @override
  Query as() => select([
        users.id,
        users.name,
        users.email,
      ]).from(users);
}
```

Explanation:

* The view exposes only the columns needed by the application.
* Any query can now use `userSummary` instead of selecting these columns repeatedly.

---

## Real-World Example

Imagine an e-commerce application where every order screen needs the customer's name.

```dart
class OrderDetails extends View {
  Orders get orders => attachedDatabase.orders;
  Customers get customers => attachedDatabase.customers;

  @override
  Query as() => select([
        orders.id,
        orders.total,
        customers.name,
      ]).from(orders).join([
        innerJoin(
          customers,
          customers.id.equalsExp(orders.customerId),
        ),
      ]);
}
```

Query the view:

```dart
final orders = await select(orderDetails).get();
```

Explanation:

* The join logic is defined once.
* Every part of the application can reuse the same view.
* If the query changes, it only needs to be updated in one place.

---

# When to Use

Use views when:

* A query is reused in multiple places.
* Joining the same tables repeatedly.
* Creating reusable reports.
* Exposing only specific columns.
* Simplifying complex SQL queries.
* Creating read-only representations of data.

---

# When NOT to Use

Avoid views when:

* The query is used only once.
* A simple table query is sufficient.
* The logic is highly dynamic and changes frequently.
* The query depends on runtime parameters.

Views are best for static, reusable query definitions.

---

# Best Practices

* Keep views focused on a single purpose.
* Give views descriptive names.
* Reuse views instead of duplicating join logic.
* Keep view definitions readable.
* Use views for frequently accessed reporting data.

---

# Common Mistakes

## Using a View for Every Query

Wrong:

Creating a separate view for every simple query.

Explanation:

* Simple queries are often clearer using the query builder directly.

Correct:

* Create views only for queries that are reused or sufficiently complex.

---

## Treating a View Like a Table

Wrong assumption:

> "The view stores its own data."

Explanation:

* A view stores only the query definition.
* Data always comes from the underlying tables.

Correct understanding:

* Querying a view executes its underlying query against the current database state.

---

## Duplicating Complex Joins

Wrong:

```dart
// Screen A
join(users, profiles)

// Screen B
join(users, profiles)

// Screen C
join(users, profiles)
```

Explanation:

* The same join logic is repeated throughout the application.

Correct:

```dart
select(userProfiles).get();
```

Explanation:

* The join is defined once in the view and reused everywhere.

---

# Related APIs

* `View`
* `select()`
* `watch()`
* `innerJoin()`
* Custom SQL
* Mapping Joined Results

---

# Summary

Views let you encapsulate reusable queries behind a table-like interface. Instead of storing data, a view stores a query definition that SQLite executes whenever it's accessed. They're ideal for simplifying complex joins, reducing duplicated query logic, and creating reusable, read-only representations of your data.
