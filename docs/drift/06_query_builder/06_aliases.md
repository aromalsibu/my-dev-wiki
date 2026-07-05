# Aliases

Aliases let you refer to the same table multiple times within a query.

---

# What is it?

An alias is an alternative name for a table used during a query.

In Drift, aliases are created with the `alias()` function.

They are primarily used when:

* Joining a table with itself.
* Using the same table multiple times in one query.
* Making complex queries easier to read.

---

# Why does it exist?

Imagine an `employees` table where each employee has a `managerId` that references another employee.

Both the employee and the manager are stored in the same table.

Without aliases, SQLite wouldn't know which instance of the table you're referring to.

Aliases give each instance its own identity.

---

# Syntax

## Creating an Alias

```dart
final manager = alias(employees, 'manager');
```

Explanation:

* `alias()` creates another reference to the `employees` table.
* `'manager'` is the SQL alias name used in the generated query.

---

## Using an Alias in a Join

```dart
final manager = alias(employees, 'manager');

final query = select(employees).join([
  leftOuterJoin(
    manager,
    manager.id.equalsExp(employees.managerId),
  ),
]);
```

Explanation:

* `employees` represents the employee.
* `manager` represents the manager.
* `equalsExp()` compares two database columns.

---

# Mental Model

Think of aliases as giving two people with the same name different nicknames.

```text
Employees Table

Alice
Bob
Carol

        │
        ▼

Employee      Manager
    │             │
    └────Same Table────┘
```

Although both references point to the same table, SQLite treats them as separate participants in the query.

---

# Examples

## Simple Example

Create an alias.

```dart
final parent = alias(categories, 'parent');
```

Explanation:

* `parent` is another reference to the `categories` table.
* It can be joined independently of the original table.

---

## Real-World Example

Retrieve employees and their managers.

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

* Each result contains an employee and the corresponding manager.
* Without the alias, the self-join wouldn't be possible.

---

# When to Use

Use aliases when you need to:

* Perform self joins.
* Reference the same table multiple times.
* Simplify complex joins.
* Improve query readability.

---

# When NOT to Use

Avoid aliases when:

* A table appears only once in the query.
* The query is simple enough without additional names.

Using unnecessary aliases can make straightforward queries harder to read.

---

# Best Practices

* Use meaningful alias names such as `manager`, `parent`, or `author`.
* Keep alias names short but descriptive.
* Create aliases only when required.
* Use aliases consistently throughout the query.
* Combine aliases with joins for self-referencing relationships.

---

# Common Mistakes

## Joining a table to itself without an alias

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

Use the alias in the join so each table reference is unique.

---

## Choosing unclear alias names

**Wrong**

```dart
final t1 = alias(employees, 't1');
```

Explanation:

* Generic names make complex queries difficult to understand.

**Correct**

```dart
final manager = alias(employees, 'manager');
```

Explanation:

* Descriptive names make the query easier to read and maintain.

---

## Creating aliases unnecessarily

**Wrong**

Creating an alias for a table that is only used once.

Explanation:

* This adds unnecessary complexity.

**Correct**

Use the original table directly unless multiple references are required.

---

# Related APIs

* Select
* Inner Join
* Left Join
* Self Join
* Expressions

---

# Summary

Aliases provide an alternate name for a table within a query, allowing the same table to be referenced multiple times. They are essential for self joins and other complex queries where a table plays more than one role, making SQL generation both possible and easier to understand.
