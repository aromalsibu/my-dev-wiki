# One-to-Many

A one-to-many relationship allows one row in a table to be related to multiple rows in another table.

---

# What is it?

A one-to-many relationship is the most common type of relationship in relational databases.

In this relationship:

* One **parent** row can have many **child** rows.
* Each child row belongs to exactly one parent.

This is implemented using a **foreign key** in the child table.

---

# Why does it exist?

Consider users and orders.

A user can place many orders.

However, each order belongs to only one user.

```text
Users

Alice
Bob

Orders

Order 1 → Alice
Order 2 → Alice
Order 3 → Bob
```

Instead of storing multiple order IDs inside the user record, each order stores the ID of its owner.

This keeps the database normalized and easy to query.

---

# Syntax

## Defining the Relationship

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
}

class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();

  IntColumn get userId =>
      integer().references(Users, #id)();
}
```

Explanation:

* `userId` is a foreign key.
* Every order references one user.
* A user may be referenced by many orders.

---

## Querying Related Data

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

* Each order is joined with its owner.
* The relationship is established through the foreign key.

---

# Mental Model

Think of a tree.

```text
User

Alice
 │
 ├── Order 1
 ├── Order 2
 └── Order 3
```

One parent branches into many children.

---

# Examples

## Simple Example

A category contains many products.

```text
Category

Electronics
      │
      ├── Laptop
      ├── Phone
      └── Tablet
```

Each product belongs to exactly one category.

---

## Real-World Example

A blog author writes many posts.

```text
Author

Alice
 │
 ├── Flutter Basics
 ├── Learning Drift
 └── Riverpod Guide
```

Every post has one author, but an author can have many posts.

---

# When to Use

Use a one-to-many relationship for:

* Users and orders.
* Categories and products.
* Authors and posts.
* Customers and invoices.
* Projects and tasks.

---

# When NOT to Use

Avoid one-to-many relationships when:

* Each record should have only one matching record (use one-to-one).
* Both sides can have multiple relationships (use many-to-many).

---

# Best Practices

* Store the foreign key in the child table.
* Index foreign key columns.
* Enforce referential integrity with foreign key constraints.
* Use joins to retrieve related data.
* Consider cascade rules when deleting parent records.

---

# Common Mistakes

## Storing multiple child IDs in the parent

**Wrong**

```text
User

orderIds = "12,18,24"
```

Explanation:

* A single column should not contain multiple values.
* This breaks database normalization.

**Correct**

Each order stores a single `userId`.

---

## Using duplicate data instead of a foreign key

**Wrong**

```text
Orders

User Name = Alice
```

Explanation:

* If Alice changes her name, every order must be updated.
* This introduces redundant data.

**Correct**

Store the user's ID and join the tables when needed.

---

## Forgetting foreign key constraints

**Wrong**

Creating a `userId` column without referencing the `Users` table.

Explanation:

* Invalid user IDs can be inserted.
* Data integrity is not enforced.

**Correct**

Use `references(Users, #id)` to define the relationship.

---

# Related APIs

* Foreign Keys
* Inner Join
* Left Join
* Many-to-Many
* One-to-One

---

# Summary

A one-to-many relationship allows a single parent record to be associated with multiple child records. In Drift, this is implemented by storing a foreign key in the child table, making it easy to model common relationships such as users and orders, authors and posts, or categories and products while keeping the database normalized and efficient.
