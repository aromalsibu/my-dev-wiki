# Inner Join

An inner join combines rows from two or more tables based on a matching condition.

---

# What is it?

An inner join returns only the rows where the join condition matches in both tables.

It's the most commonly used type of join and is used to retrieve related data stored across multiple tables.

For example:

* Users and their orders.
* Posts and their authors.
* Students and their classes.

Rows without a matching relationship are excluded from the result.

---

# Why does it exist?

Relational databases avoid storing duplicate information.

Instead of storing a user's name inside every order, the order stores the user's ID.

```text
Users
+----+-------+
| ID | Name  |
+----+-------+
| 1  | Alice |
| 2  | Bob   |
+----+-------+

Orders
+----+---------+
| ID | User ID |
+----+---------+
|101 |    1    |
|102 |    1    |
|103 |    2    |
+----+---------+
```

An inner join combines these related rows into a single result.

---

# Syntax

## Joining Two Tables

```dart
final query = select(orders).join([
  innerJoin(
    users,
    users.id.equalsExp(orders.userId),
  ),
]);

final results = await query.get();
```

Explanation:

* `innerJoin()` joins the `users` table with `orders`.
* `equalsExp()` compares two database columns.
* Only matching rows are returned.

---

## Reading Joined Data

```dart
for (final row in results) {
  final order = row.readTable(orders);
  final user = row.readTable(users);

  print('${user.name} placed order ${order.id}');
}
```

Explanation:

* `readTable()` extracts the data class for each table.
* Each result contains data from both tables.

---

# Mental Model

Think of an inner join as finding matching puzzle pieces.

```text
Users                Orders

Alice (1)  ◄────────► User 1
Bob   (2)  ◄────────► User 2
Carol (3)

↓

Result

Alice + Order
Bob + Order
```

Carol isn't included because she has no matching order.

---

# Examples

## Simple Example

Retrieve every order with its customer.

```dart
final query = select(orders).join([
  innerJoin(
    users,
    users.id.equalsExp(orders.userId),
  ),
]);

final rows = await query.get();
```

Explanation:

* Each row contains both an order and its associated user.

---

## Real-World Example

Load comments with their authors.

```dart
final query = select(comments).join([
  innerJoin(
    users,
    users.id.equalsExp(comments.authorId),
  ),
]);

final rows = await query.get();
```

Explanation:

* Each comment is paired with the user who wrote it.
* Comments without valid authors are excluded.

---

# When to Use

Use an inner join when you need to:

* Combine related tables.
* Display parent-child relationships.
* Retrieve normalized data.
* Load associated records.
* Build relational queries.

---

# When NOT to Use

Avoid an inner join when:

* You need rows even if no related record exists.
* Missing relationships should still appear in the result.

In these cases, use a **left join** instead.

---

# Best Practices

* Join tables using primary and foreign keys.
* Index columns used in join conditions.
* Read only the tables you need.
* Keep join conditions simple.
* Use meaningful table aliases for self joins.

---

# Common Mistakes

## Joining on unrelated columns

**Wrong**

```dart
innerJoin(
  users,
  users.name.equalsExp(orders.userId),
)
```

Explanation:

* The columns represent different types of data.
* The join condition is incorrect.

**Correct**

```dart
innerJoin(
  users,
  users.id.equalsExp(orders.userId),
)
```

Explanation:

* The foreign key references the primary key.

---

## Expecting unmatched rows

**Wrong**

Assuming every user appears in the result.

Explanation:

* Inner joins only return rows with matching records in both tables.

**Correct**

Use a left join if unmatched rows should also be included.

---

## Forgetting to read joined tables

**Wrong**

Trying to access joined data directly from the query.

Explanation:

* Joined results are wrapped in `TypedResult`.

**Correct**

Use:

```dart
row.readTable(users);
row.readTable(orders);
```

to retrieve each table's data.

---

# Related APIs

* Left Join
* Cross Join
* Self Join
* Mapping Joined Results
* Foreign Keys

---

# Summary

An inner join combines related rows from multiple tables by matching a join condition, typically between a primary key and a foreign key. Only rows with matching records in both tables are returned, making inner joins the standard choice for retrieving related data in relational databases.
