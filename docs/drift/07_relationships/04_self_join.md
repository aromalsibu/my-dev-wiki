# Self Join

A self join joins a table to itself to retrieve related rows within the same table.

---

# What is it?

A self join is a join where the same table appears twice in a query.

Since both references point to the same table, one of them must be given an alias so SQLite can distinguish between them.

Self joins are commonly used for hierarchical or recursive relationships, such as:

* Employees and managers.
* Categories and parent categories.
* Comments and parent comments.
* Folders and parent folders.

---

# Why does it exist?

Consider an `employees` table.

```text
+----+---------+-----------+
| ID | Name    | Manager ID|
+----+---------+-----------+
| 1  | Alice   | NULL      |
| 2  | Bob     | 1         |
| 3  | Carol   | 2         |
+----+---------+-----------+
```

Managers are also employees.

Instead of creating a separate `managers` table, the `managerId` column references another row in the same table.

A self join allows both the employee and the manager to be retrieved in one query.

---

# Syntax

## Creating an Alias

```dart
final manager = alias(employees, 'manager');
```

Explanation:

* `alias()` creates a second reference to the `employees` table.
* SQLite treats the alias as a separate table within the query.

---

## Performing a Self Join

```dart
final manager = alias(employees, 'manager');

final query = select(employees).join([
  leftOuterJoin(
    manager,
    manager.id.equalsExp(employees.managerId),
  ),
]);

final results = await query.get();
```

Explanation:

* `employees` represents the employee.
* `manager` represents the employee's manager.
* A left join is used because top-level managers may not have their own manager.

---

## Reading the Results

```dart
for (final row in results) {
  final employee = row.readTable(employees);
  final managerData = row.readTableOrNull(manager);

  print(
    '${employee.name} -> ${managerData?.name ?? "No Manager"}',
  );
}
```

Explanation:

* `readTable()` reads the primary employee.
* `readTableOrNull()` safely reads the optional manager.

---

# Mental Model

Think of the same table playing two different roles.

```text
Employees Table

        ┌──────────────┐
        │ Employees    │
        └──────┬───────┘
               │
        ┌──────┴───────┐
        │              │
   Employee         Manager
        │              │
        └──────Join────┘
```

Although both references point to the same table, they represent different entities in the query.

---

# Examples

## Simple Example

Retrieve every category with its parent category.

```dart
final parent = alias(categories, 'parent');

final query = select(categories).join([
  leftOuterJoin(
    parent,
    parent.id.equalsExp(categories.parentId),
  ),
]);
```

Explanation:

* Each category is joined with its parent category.
* Root categories have no parent.

---

## Real-World Example

Display employees and their managers.

```dart
final manager = alias(employees, 'manager');

final query = select(employees).join([
  leftOuterJoin(
    manager,
    manager.id.equalsExp(employees.managerId),
  ),
]);

final rows = await query.get();
```

Explanation:

* Every employee is returned.
* Manager information is included when available.

---

# When to Use

Use a self join when working with:

* Organizational hierarchies.
* Nested categories.
* Tree structures.
* Parent-child relationships stored in one table.
* Recursive data models.

---

# When NOT to Use

Avoid a self join when:

* The relationship is between different tables.
* A normal inner or left join is sufficient.

Only use self joins when the same table must appear multiple times in a query.

---

# Best Practices

* Always create a descriptive alias.
* Use meaningful names like `manager`, `parent`, or `author`.
* Use left joins for optional parent relationships.
* Join on primary and foreign keys.
* Keep aliases consistent throughout the query.

---

# Common Mistakes

## Forgetting to create an alias

**Wrong**

```dart
select(employees).join([
  leftOuterJoin(
    employees,
    employees.id.equalsExp(employees.managerId),
  ),
]);
```

Explanation:

* SQLite cannot distinguish between the two references to the same table.

**Correct**

```dart
final manager = alias(employees, 'manager');
```

Explanation:

* The alias gives the second table reference its own identity.

---

## Reading the wrong table

**Wrong**

```dart
row.readTable(employees);
row.readTable(employees);
```

Explanation:

* Both calls read the same table reference.

**Correct**

```dart
row.readTable(employees);
row.readTableOrNull(manager);
```

Explanation:

* Read the original table and its alias separately.

---

## Using an inner join for optional parents

**Wrong**

```dart
innerJoin(
  manager,
  manager.id.equalsExp(employees.managerId),
)
```

Explanation:

* Employees without managers are excluded.

**Correct**

```dart
leftOuterJoin(
  manager,
  manager.id.equalsExp(employees.managerId),
)
```

Explanation:

* Every employee is returned, even if they have no manager.

---

# Related APIs

* Aliases
* Inner Join
* Left Join
* Foreign Keys
* Mapping Joined Results

---

# Summary

A self join allows a table to be joined with itself by creating an alias for one of its references. It is essential for querying hierarchical data such as managers, parent categories, and nested structures, where different rows in the same table are related to one another.
