# Generated Data Classes

Generated data classes represent rows in your database tables as strongly typed Dart objects.

---

# What is it?

When you define a table in Drift, it automatically generates a corresponding Dart data class.

Each instance of the generated data class represents a single row in the table.

For example, given a `Users` table, Drift generates a `User` data class.

Instead of working with raw database rows, your application works with typed Dart objects.

---

# Why does it exist?

Without generated data classes, query results would typically be returned as maps.

For example:

```text
{
  "id": 1,
  "name": "Alice",
  "age": 25
}
```

Accessing values by string keys is error-prone and lacks type safety.

Generated data classes provide:

* Strong typing.
* Compile-time checking.
* Better IDE support.
* Easier serialization.
* Cleaner application code.

---

# Syntax

## Defining a Table

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get name => text()();

  IntColumn get age => integer()();
}
```

Explanation:

* Drift uses this table definition to generate a `User` data class automatically.

---

## Using the Generated Data Class

```dart
final users = await select(usersTable).get();

for (final user in users) {
  print(user.name);
}
```

Explanation:

* `select(...).get()` returns a list of generated data class instances.
* Each property corresponds to a column in the table.

---

# Mental Model

Think of a generated data class as a Dart representation of a database row.

```text
Database Row
+----+-------+-----+
| 1  | Alice | 25  |
+----+-------+-----+
        │
        ▼
Generated User Object

User(
  id: 1,
  name: "Alice",
  age: 25,
)
```

Each row becomes a Dart object with typed fields.

---

# Examples

## Simple Example

Reading all products.

```dart
final products = await select(products).get();

for (final product in products) {
  print(product.name);
}
```

Explanation:

* Each item in `products` is a generated `Product` object.
* Columns are exposed as Dart properties.

---

## Real-World Example

Finding a specific task.

```dart
final task = await (select(tasks)
      ..where((t) => t.id.equals(1)))
    .getSingle();

print(task.title);
print(task.completed);
```

Explanation:

* `getSingle()` returns one generated `Task` object.
* The result can be used directly throughout your application.

---

# When to Use

Generated data classes are used whenever you:

* Read data from the database.
* Pass database records between layers.
* Display query results.
* Serialize database records.
* Work with individual rows.

They are the standard representation of database records in Drift.

---

# When NOT to Use

Don't use generated data classes for:

* Partial inserts.
* Partial updates.
* Optional update values.

For these operations, use companion classes instead.

---

# Best Practices

* Treat generated data classes as immutable models.
* Use them for reading data.
* Pass them between repositories and UI when appropriate.
* Prefer companions for inserts and updates.
* Avoid modifying generated code manually.

---

# Common Mistakes

## Using a data class for inserts

**Wrong**

```dart
final user = User(
  id: 1,
  name: 'Alice',
  age: 25,
);

await into(users).insert(user);
```

Explanation:

* Data classes represent complete rows.
* Inserts often require optional or omitted fields such as auto-incrementing IDs.

**Correct**

Use a companion class for insert operations.

---

## Editing generated files

**Wrong**

Modifying the generated `.g.dart` file.

Explanation:

* Generated code is recreated every time Drift runs code generation.
* Manual changes are lost.

**Correct**

Modify the table definition and regenerate the code.

---

## Confusing data classes with tables

**Wrong**

Assuming the generated `User` class defines the database schema.

Explanation:

* The table defines the schema.
* The generated data class represents a row of data.

**Correct**

Use the table for schema definitions and the generated data class for working with query results.

---

# Related APIs

* Companions
* Insertable
* Value<T>
* Defining Tables
* Select

---

# Summary

Generated data classes provide a strongly typed representation of rows in your database tables. Drift automatically generates these classes from your table definitions, allowing you to work with Dart objects instead of raw database rows. They are primarily used for reading and representing data, while companion classes handle inserts and updates.
