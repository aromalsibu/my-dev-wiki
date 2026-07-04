# Primary Keys

A primary key uniquely identifies each row in a table.

---

# What is it?

A primary key is a column, or a combination of columns, whose value uniquely identifies every row in a table.

Every table should have a primary key so that each record can be uniquely identified.

For example, in a `Users` table:

| id | name    |
| -- | ------- |
| 1  | Alice   |
| 2  | Bob     |
| 3  | Charlie |

The `id` column is the primary key because each value is unique and identifies exactly one user.

In Drift, the most common way to define a primary key is by using `autoIncrement()`.

---

# Why does it exist?

Without a primary key, it becomes difficult to uniquely identify a row.

For example, if two users have the same name:

| id | name |
| -- | ---- |
| 1  | John |
| 2  | John |

Updating or deleting a specific user based only on their name would be unreliable.

A primary key ensures every row has a unique identity, making queries, updates, relationships, and indexing more efficient.

---

# Syntax

## Auto-Incrementing Primary Key

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();
}
```

Explanation:

* `autoIncrement()` creates an integer primary key.
* SQLite automatically assigns a unique value for each new row.
* The `id` column becomes the table's primary key.

---

## Custom Primary Key

If you don't use `autoIncrement()`, you can define the primary key manually.

```dart
class Users extends Table {
  TextColumn get email => text()();

  TextColumn get name => text()();

  @override
  Set<Column> get primaryKey => {email};
}
```

Explanation:

* `primaryKey` specifies which column uniquely identifies each row.
* Here, `email` acts as the primary key instead of an auto-generated ID.

---

# Mental Model

Think of a primary key as a unique ID card.

```text
Users

ID   Name
------------
1    Alice
2    Bob
3    Charlie
```

Even if two people have the same name, their ID numbers are different.

The database uses the primary key to distinguish one row from another.

---

# Examples

## Simple Example

A products table.

```dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();
}
```

Explanation:

* Every product receives a unique ID automatically.
* The ID can be used to retrieve, update, or delete a specific product.

---

## Real-World Example

A country table using ISO country codes.

```dart
class Countries extends Table {
  TextColumn get code => text()();

  TextColumn get name => text()();

  @override
  Set<Column> get primaryKey => {code};
}
```

Explanation:

* Country codes such as `US`, `IN`, and `JP` are already unique.
* There's no need for an additional auto-incrementing ID.

---

# When to Use

Use a primary key for:

* Every database table.
* Identifying individual records.
* Creating relationships between tables.
* Updating or deleting specific rows.
* Ensuring every record has a unique identity.

---

# When NOT to Use

Avoid using a column as a primary key if:

* Its value may change over time.
* Duplicate values are possible.
* It doesn't uniquely identify a row.

For example, names, phone numbers, and addresses are often poor primary keys because they can change or may not be unique.

---

# Best Practices

* Give every table a primary key.
* Prefer `autoIncrement()` for most tables.
* Use natural keys only when they're guaranteed to be unique and stable.
* Keep primary keys simple whenever possible.
* Avoid changing primary key values after records are created.

---

# Common Mistakes

## Forgetting a primary key

**Wrong**

```dart
class Users extends Table {
  TextColumn get name => text()();
}
```

Explanation:

* The table has no unique way to identify its rows.

**Correct**

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();
}
```

Explanation:

* Every row now has a unique identifier.

---

## Using non-unique values

**Wrong**

```dart
@override
Set<Column> get primaryKey => {name};
```

Explanation:

* Multiple users can have the same name.

**Correct**

```dart
@override
Set<Column> get primaryKey => {email};
```

Explanation:

* Use a column that is guaranteed to be unique.

---

## Using mutable data as a primary key

**Wrong**

Using a username as the primary key.

Explanation:

* Usernames may change over time, affecting relationships and references.

**Correct**

Use an immutable ID or another stable unique value.

---

# Related APIs

* Composite Keys
* Foreign Keys
* Constraints
* Columns
* Generated Data Classes
* Insert

---

# Summary

A primary key uniquely identifies every row in a table. In Drift, this is commonly achieved with `autoIncrement()`, though any unique and stable column can be used as a custom primary key. Choosing an appropriate primary key is essential for maintaining data integrity and building relationships between tables.
