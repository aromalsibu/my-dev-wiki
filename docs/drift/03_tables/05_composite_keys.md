# Composite Keys

A composite key uses two or more columns together to uniquely identify a row.

---

# What is it?

A composite key is a primary key made up of multiple columns instead of a single column.

Individually, each column may contain duplicate values, but the combination of their values must be unique.

For example, consider a table that records which students are enrolled in which courses.

| studentId | courseId |
| --------- | -------- |
| 1         | 101      |
| 1         | 102      |
| 2         | 101      |

Neither `studentId` nor `courseId` is unique on its own, but the combination of both uniquely identifies each enrollment.

---

# Why does it exist?

Some data naturally requires multiple values to identify a record.

Using a single auto-incrementing ID in these situations may not accurately represent the relationship between the data.

Composite keys are commonly used for:

* Many-to-many relationships
* Junction tables
* Lookup tables
* Records identified by multiple attributes

They ensure that duplicate combinations cannot be inserted into the database.

---

# Syntax

## Defining a Composite Key

```dart
class Enrollments extends Table {
  IntColumn get studentId => integer()();

  IntColumn get courseId => integer()();

  @override
  Set<Column> get primaryKey => {
    studentId,
    courseId,
  };
}
```

Explanation:

* `primaryKey` returns a set of columns that together form the primary key.
* Each individual column can contain duplicate values.
* The combination of both columns must be unique.

---

# Mental Model

Think of a composite key as using multiple fields to identify something.

```text
Student ID + Course ID
        │
        ▼
Unique Enrollment
```

For example:

| Student | Course | Valid?      |
| ------- | ------ | ----------- |
| 1       | 101    | ✅           |
| 1       | 102    | ✅           |
| 2       | 101    | ✅           |
| 1       | 101    | ❌ Duplicate |

The database treats the pair `(1, 101)` as the unique identifier.

---

# Examples

## Simple Example

A table storing user permissions.

```dart
class UserPermissions extends Table {
  IntColumn get userId => integer()();

  TextColumn get permission => text()();

  @override
  Set<Column> get primaryKey => {
    userId,
    permission,
  };
}
```

Explanation:

* A user can have multiple permissions.
* Each permission can belong to multiple users.
* A user cannot have the same permission twice.

---

## Real-World Example

A shopping application storing items in an order.

```dart
class OrderItems extends Table {
  IntColumn get orderId => integer()();

  IntColumn get productId => integer()();

  IntColumn get quantity => integer()();

  @override
  Set<Column> get primaryKey => {
    orderId,
    productId,
  };
}
```

Explanation:

* An order can contain many products.
* A product can appear in many orders.
* The same product cannot appear twice in the same order.

---

# When to Use

Use composite keys when:

* Two or more columns together identify a record.
* Building many-to-many relationship tables.
* Preventing duplicate combinations of values.
* Modeling relationships without introducing an unnecessary ID column.

---

# When NOT to Use

Avoid composite keys when:

* A single column uniquely identifies the row.
* An auto-incrementing ID is sufficient.
* The combination of columns may change frequently.

For most tables, a single primary key is simpler and easier to work with.

---

# Best Practices

* Keep composite keys small.
* Use only columns that naturally identify the record.
* Avoid including mutable columns in a composite key.
* Prefer composite keys for junction tables and relationship tables.
* Don't add an auto-incrementing ID unless it serves a purpose.

---

# Common Mistakes

## Using only one column

**Wrong**

```dart
@override
Set<Column> get primaryKey => {studentId};
```

Explanation:

* A student can enroll in multiple courses.
* `studentId` alone does not uniquely identify a row.

**Correct**

```dart
@override
Set<Column> get primaryKey => {
  studentId,
  courseId,
};
```

Explanation:

* The combination uniquely identifies each enrollment.

---

## Adding an unnecessary ID column

**Wrong**

```dart
IntColumn get id => integer().autoIncrement()();

IntColumn get studentId => integer()();

IntColumn get courseId => integer()();
```

Explanation:

* If the combination of `studentId` and `courseId` already identifies each row, the additional ID may be unnecessary.

**Correct**

Use the natural composite key when it accurately represents the relationship.

---

## Including mutable columns

**Wrong**

```dart
@override
Set<Column> get primaryKey => {
  userId,
  username,
};
```

Explanation:

* If `username` changes, the primary key changes as well.

**Correct**

Use stable values that are unlikely to change.

---

# Related APIs

* Primary Keys
* Foreign Keys
* Many-to-Many
* Constraints
* Insert
* Replace

---

# Summary

A composite key uses multiple columns together as a table's primary key. It is useful when no single column uniquely identifies a record, especially in junction tables and many-to-many relationships. By using the combination of values as the unique identifier, Drift helps maintain data integrity without requiring an additional ID column.
