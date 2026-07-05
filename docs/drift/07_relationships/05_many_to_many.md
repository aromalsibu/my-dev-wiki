# Many-to-Many

A many-to-many relationship allows multiple rows in one table to be associated with multiple rows in another table.

---

# What is it?

In a many-to-many relationship:

* One record can relate to many records in another table.
* Those related records can also relate back to many records in the first table.

Since relational databases cannot represent this relationship directly, an intermediate **junction table** (also called a **bridge table**) is used.

---

# Why does it exist?

Consider students and courses.

A student can enroll in many courses.

A course can have many students.

```text
Students

Alice
Bob

Courses

Math
Science

Relationships

Alice → Math
Alice → Science
Bob   → Science
```

Neither table can store this relationship alone.

A third table records each association.

---

# Syntax

## Defining the Junction Table

```dart
class StudentCourses extends Table {
  IntColumn get studentId =>
      integer().references(Students, #id)();

  IntColumn get courseId =>
      integer().references(Courses, #id)();

  @override
  Set<Column> get primaryKey => {
        studentId,
        courseId,
      };
}
```

Explanation:

* `studentId` references the `Students` table.
* `courseId` references the `Courses` table.
* A composite primary key prevents duplicate relationships.

---

## Querying Related Data

```dart
final query = select(studentCourses).join([
  innerJoin(
    students,
    students.id.equalsExp(studentCourses.studentId),
  ),
  innerJoin(
    courses,
    courses.id.equalsExp(studentCourses.courseId),
  ),
]);

final results = await query.get();
```

Explanation:

* The junction table connects the two tables.
* Each result contains one student-course relationship.

---

# Mental Model

Think of the junction table as a connector.

```text
Students
    │
    │
    ▼
StudentCourses
    ▲
    │
    │
Courses
```

The junction table doesn't usually contain business data—it stores relationships.

---

# Examples

## Simple Example

A user can belong to multiple groups.

```text
Users

Alice
Bob

Groups

Admin
Editor

UserGroups

Alice → Admin
Alice → Editor
Bob   → Editor
```

The `UserGroups` table records every membership.

---

## Real-World Example

Products and tags.

```text
Products

Laptop
Phone

Tags

Electronics
Portable

ProductTags

Laptop → Electronics
Laptop → Portable
Phone   → Electronics
```

A product can have many tags, and each tag can belong to many products.

---

# When to Use

Use a many-to-many relationship when:

* Users belong to multiple groups.
* Students enroll in multiple courses.
* Products have multiple tags.
* Movies have multiple actors.
* Books have multiple authors.

---

# When NOT to Use

Avoid many-to-many relationships when:

* Each child belongs to only one parent.
* The relationship is naturally one-to-many or one-to-one.

Introducing a junction table unnecessarily increases complexity.

---

# Best Practices

* Always use a dedicated junction table.
* Add foreign keys to both related tables.
* Use a composite primary key to prevent duplicate relationships.
* Index foreign key columns for faster joins.
* Store additional relationship data (such as enrollment date) in the junction table when needed.

---

# Common Mistakes

## Trying to store multiple IDs in one column

**Wrong**

```text
studentIds = "1,2,5"
```

Explanation:

* Relational databases are designed to store one value per column.
* Searching and joining become difficult.

**Correct**

Create a junction table with one row per relationship.

---

## Forgetting a composite primary key

**Wrong**

Allowing duplicate rows like:

```text
Alice → Math
Alice → Math
```

Explanation:

* Duplicate relationships usually have no meaning.

**Correct**

Use both foreign keys as the primary key.

---

## Skipping the junction table

**Wrong**

Adding a `courseIds` column to the `Students` table.

Explanation:

* This breaks normalization and makes queries difficult.

**Correct**

Use a separate table to represent the relationship.

---

# Related APIs

* One-to-Many
* One-to-One
* Foreign Keys
* Inner Join
* Mapping Joined Results

---

# Summary

A many-to-many relationship allows records from two tables to be associated with multiple records in each other. In Drift, this is implemented using a junction table containing foreign keys to both tables, enabling efficient querying while keeping the database normalized and easy to maintain.
