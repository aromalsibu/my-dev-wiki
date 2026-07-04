# Foreign Keys

A foreign key creates a relationship between tables by referencing the primary key of another table.

---

# What is it?

A foreign key is a column whose value refers to a row in another table.

It establishes a relationship between two tables and ensures that the referenced row exists.

For example, consider two tables:

**Users**

| id | name  |
| -- | ----- |
| 1  | Alice |
| 2  | Bob   |

**Posts**

| id | userId | title          |
| -- | ------ | -------------- |
| 1  | 1      | Hello World    |
| 2  | 2      | Drift Tutorial |

The `userId` column in the `Posts` table is a foreign key because it references the `id` column in the `Users` table.

---

# Why does it exist?

Without foreign keys, related data can become inconsistent.

For example:

* A post could reference a user that doesn't exist.
* An order could reference a deleted customer.
* A comment could belong to a non-existent post.

Foreign keys help maintain **referential integrity** by ensuring relationships between tables remain valid.

---

# Syntax

## Defining a Foreign Key

```dart
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();

  IntColumn get userId =>
      integer().references(Users, #id)();

  TextColumn get title => text()();
}
```

Explanation:

* `references()` creates a foreign key relationship.
* `Users` is the table being referenced.
* `#id` refers to the primary key column of the `Users` table.
* Every `userId` value must exist in the `Users` table.

---

# Mental Model

Think of a foreign key as a reference or link.

```text
Users
+----+--------+
| ID | Name   |
+----+--------+
| 1  | Alice  |
| 2  | Bob    |
+----+--------+

        ▲
        │
        │ userId
        │
Posts
+----+--------+----------------+
| ID | userId | Title          |
+----+--------+----------------+
| 1  |   1    | Hello World    |
| 2  |   2    | Drift Tutorial |
+----+--------+----------------+
```

Instead of storing the user's entire information in every post, the post stores only the user's ID.

---

# Examples

## Simple Example

A comments table.

```dart
class Comments extends Table {
  IntColumn get id => integer().autoIncrement()();

  IntColumn get postId =>
      integer().references(Posts, #id)();

  TextColumn get content => text()();
}
```

Explanation:

* Every comment belongs to a post.
* `postId` references the `Posts` table.

---

## Real-World Example

An orders table.

```dart
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();

  IntColumn get customerId =>
      integer().references(Customers, #id)();

  DateTimeColumn get orderDate => dateTime()();
}
```

Explanation:

* Every order belongs to a customer.
* SQLite prevents orders from referencing customers that don't exist.

---

# When to Use

Use foreign keys when:

* One table depends on another.
* Modeling one-to-one relationships.
* Modeling one-to-many relationships.
* Modeling many-to-many relationships through junction tables.
* Maintaining referential integrity.

---

# When NOT to Use

Avoid foreign keys when:

* Tables are completely unrelated.
* The referenced data is stored in another database.
* There is no logical relationship between the tables.

Don't create foreign keys simply because two tables have similar columns.

---

# Best Practices

* Reference primary keys whenever possible.
* Use foreign keys to model real relationships.
* Keep related tables in the same database.
* Choose meaningful column names such as `userId` or `productId`.
* Avoid storing duplicate data when a relationship can be used instead.

---

# Common Mistakes

## Storing duplicate information

**Wrong**

```dart
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get authorName => text()();
}
```

Explanation:

* The author's name is duplicated in every post.
* Updating the user's name becomes difficult.

**Correct**

```dart
IntColumn get userId =>
    integer().references(Users, #id)();
```

Explanation:

* Store a reference to the user instead of duplicating user data.

---

## Missing foreign key relationships

**Wrong**

```dart
IntColumn get userId => integer()();
```

Explanation:

* SQLite doesn't know that `userId` should reference the `Users` table.

**Correct**

```dart
IntColumn get userId =>
    integer().references(Users, #id)();
```

Explanation:

* The relationship is enforced by the database.

---

## Referencing unrelated tables

**Wrong**

```text
Orders
   │
   ▼
Products
```

When the relationship doesn't actually exist.

Explanation:

* Foreign keys should represent real relationships in your data model.

**Correct**

Create foreign keys only when one record genuinely depends on another.

---

# Related APIs

* Primary Keys
* Composite Keys
* Inner Join
* Left Join
* One-to-One
* One-to-Many
* Many-to-Many

---

# Summary

Foreign keys establish relationships between tables by referencing the primary key of another table. They help maintain referential integrity, prevent invalid references, and reduce data duplication. By using foreign keys, you can model real-world relationships while keeping your database consistent and reliable.
