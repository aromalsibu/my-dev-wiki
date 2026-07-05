# Mapping Joined Results

Mapping joined results is the process of converting the rows returned by a join into meaningful Dart objects.

---

# What is it?

When you execute a join in Drift, the result is **not** a generated data class.

Instead, Drift returns a list of `TypedResult` objects.

Each `TypedResult` contains the data from every table involved in the join. You extract the individual table rows using methods like `readTable()` and `readTableOrNull()`, then map them into the model your application needs.

---

# Why does it exist?

A join can return data from multiple tables.

For example, an order and its customer:

```text
Order
+ Customer
-------------
Order #101
Alice
```

Drift cannot automatically decide what object your application should create.

Should it return:

* An `Order`
* A `User`
* A custom `OrderWithUser`
* Something else

Instead, Drift gives you access to all joined tables, allowing you to map them into the structure that best fits your application.

---

# Syntax

## Reading Tables from a Joined Result

```dart
final query = select(orders).join([
  innerJoin(
    users,
    users.id.equalsExp(orders.userId),
  ),
]);

final rows = await query.get();

for (final row in rows) {
  final order = row.readTable(orders);
  final user = row.readTable(users);
}
```

Explanation:

* `get()` returns a `List<TypedResult>`.
* `readTable()` extracts the generated data class for a table.
* Each `TypedResult` contains data for all joined tables.

---

## Reading Optional Tables

```dart
final profile = row.readTableOrNull(userProfiles);
```

Explanation:

* `readTableOrNull()` is used when the table was joined with `leftOuterJoin()`.
* It returns `null` if no matching row exists.

---

# Mental Model

Think of a joined result as a container holding multiple objects.

```text
TypedResult
│
├── User
├── Order
└── Profile
```

Your job is to unpack the container and build the object your application needs.

---

# Examples

## Simple Example

Extract user and order data.

```dart
final rows = await select(orders).join([
  innerJoin(
    users,
    users.id.equalsExp(orders.userId),
  ),
]).get();

for (final row in rows) {
  final order = row.readTable(orders);
  final user = row.readTable(users);

  print('${user.name} placed order #${order.id}');
}
```

Explanation:

* `TypedResult` is unpacked into its individual table objects.
* Both generated data classes are available.

---

## Real-World Example

Suppose your UI needs an `OrderWithUser` model.

```dart
class OrderWithUser {
  final Order order;
  final User user;

  OrderWithUser({
    required this.order,
    required this.user,
  });
}
```

Now map the joined results into this model.

```dart
final query = select(orders).join([
  innerJoin(
    users,
    users.id.equalsExp(orders.userId),
  ),
]);

final rows = await query.get();

final ordersWithUsers = rows.map((row) {
  return OrderWithUser(
    order: row.readTable(orders),
    user: row.readTable(users),
  );
}).toList();
```

Explanation:

* `readTable()` extracts each generated data class.
* `map()` converts `TypedResult` into your application's domain model.
* The UI can now work with `OrderWithUser` instead of raw database results.

---

# When to Use

Use result mapping when you need to:

* Convert joined data into custom models.
* Combine data from multiple tables.
* Prepare data for the UI.
* Separate database models from business models.
* Simplify repository and DAO APIs.

---

# When NOT to Use

Avoid custom mapping when:

* You're querying a single table.
* The generated data class already matches your needs.
* No transformation is required.

---

# Best Practices

* Map joined results inside DAOs or repositories.
* Create dedicated models for joined data.
* Use `readTableOrNull()` for optional joins.
* Keep mapping logic separate from UI code.
* Return strongly typed models instead of `TypedResult` whenever possible.

---

# Common Mistakes

## Returning `TypedResult` directly

**Wrong**

```dart
Future<List<TypedResult>> getOrders() async {
  return query.get();
}
```

Explanation:

* `TypedResult` is a database-specific type.
* It exposes implementation details to the rest of the application.

**Correct**

Map the results into a custom model before returning them.

---

## Using `readTable()` after a left join

**Wrong**

```dart
final profile = row.readTable(userProfiles);
```

Explanation:

* The joined row may not exist.
* This can fail if the joined columns are `NULL`.

**Correct**

```dart
final profile = row.readTableOrNull(userProfiles);
```

Explanation:

* Safely returns `null` when there is no matching row.

---

## Mapping inside the UI

**Wrong**

```dart
// Widget builds TypedResult into UI models
```

Explanation:

* Database mapping logic belongs in the data layer.
* UI code becomes harder to maintain.

**Correct**

Perform the mapping inside the DAO or repository and expose clean models to the UI.

---

# Related APIs

* Inner Join
* Left Join
* Self Join
* Aliases
* DAOs
* Generated Data Classes

---

# Summary

Joined queries in Drift return `TypedResult` objects that contain data from every table involved in the query. By using `readTable()` and `readTableOrNull()`, you can extract the generated data classes and map them into custom models tailored to your application's needs. Performing this mapping in the data layer keeps your code clean, reusable, and easy to maintain.
