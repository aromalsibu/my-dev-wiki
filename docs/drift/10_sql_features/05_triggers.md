# Triggers

**Triggers** are database objects that automatically execute SQL statements when specific events, such as inserts, updates, or deletes, occur on a table.

---

# What is it?

A trigger is like an automatic event listener inside SQLite.

Instead of writing Dart code to perform an action after every database change, you can define a trigger that SQLite executes automatically.

A trigger can run:

* Before an insert
* After an insert
* Before an update
* After an update
* Before a delete
* After a delete

Unlike your Dart code, triggers run entirely inside the database.

---

# Why does it exist?

Suppose every time an order is created, you also want to record it in an audit table.

Without a trigger, every place that inserts an order must also remember to insert an audit record.

```text
Create Order
      │
      ├── Insert Order
      └── Insert Audit Log
```

If one developer forgets the second step, your audit log becomes incomplete.

With a trigger:

```text
Insert Order
      │
      ▼
SQLite Trigger
      │
      ▼
Insert Audit Log
```

The database itself guarantees that the audit record is always created.

---

# Syntax

Triggers are created using SQL.

```dart
await customStatement('''
CREATE TRIGGER log_user_insert
AFTER INSERT ON users
BEGIN
  INSERT INTO audit_logs(action)
  VALUES ('User created');
END;
''');
```

Explanation:

* `AFTER INSERT` means the trigger runs after a row is inserted.
* SQLite automatically executes the SQL inside the `BEGIN...END` block.

---

Create a trigger that updates a timestamp.

```dart
await customStatement('''
CREATE TRIGGER update_modified_at
AFTER UPDATE ON users
BEGIN
  UPDATE users
  SET modified_at = CURRENT_TIMESTAMP
  WHERE id = NEW.id;
END;
''');
```

Explanation:

* `NEW` refers to the updated row.
* The trigger automatically updates the timestamp whenever the row changes.

---

# Mental Model

Think of a trigger as an automatic reaction.

```text
Database Event
      │
      ▼
Trigger
      │
      ▼
Execute SQL
```

For example:

```text
Insert User
      │
      ▼
Trigger Fires
      │
      ▼
Create Audit Log
```

Your application doesn't need to call the trigger—it happens automatically.

---

# Examples

## Simple Example

Log deleted users.

```dart
await customStatement('''
CREATE TRIGGER log_user_delete
AFTER DELETE ON users
BEGIN
  INSERT INTO audit_logs(action)
  VALUES ('User deleted');
END;
''');
```

Explanation:

* Every deleted user automatically creates an audit log entry.

---

## Real-World Example

Suppose you're building an inventory system.

Whenever a product is sold, you want to automatically record the inventory change.

```dart
await customStatement('''
CREATE TRIGGER inventory_history
AFTER UPDATE ON products
WHEN NEW.stock != OLD.stock
BEGIN
  INSERT INTO inventory_logs(
    product_id,
    old_stock,
    new_stock
  )
  VALUES(
    NEW.id,
    OLD.stock,
    NEW.stock
  );
END;
''');
```

Explanation:

* `OLD` contains the row before the update.
* `NEW` contains the updated row.
* The trigger runs only when the stock value changes.

---

# When to Use

Use triggers for:

* Audit logging
* Automatically updating timestamps
* Maintaining derived data
* Enforcing business rules inside the database
* Synchronizing related tables
* Recording change history

---

# When NOT to Use

Avoid triggers when:

* The logic belongs in your application's business layer.
* The behavior should be explicit and easy to trace.
* The operation involves external systems such as APIs, emails, or notifications.
* The logic is complex and difficult to express in SQL.

Remember that triggers can make database behavior less obvious because they execute automatically.

---

# Best Practices

* Keep triggers simple.
* Use descriptive trigger names.
* Document why a trigger exists.
* Avoid putting complex business logic in triggers.
* Test triggers thoroughly after schema changes.
* Prefer Drift APIs for normal application logic.

---

# Common Mistakes

## Putting Too Much Logic in a Trigger

Wrong:

```text
Insert Order
      │
      ▼
Trigger
      ├── Update Inventory
      ├── Send Email
      ├── Generate Invoice
      ├── Call API
      └── Update Analytics
```

Explanation:

* Triggers should focus on database operations.
* They cannot safely perform application-level tasks.

Correct:

```text
Insert Order
      │
      ▼
Trigger
      │
      ▼
Insert Audit Record
```

Explanation:

* Keep triggers limited to database responsibilities.

---

## Forgetting About Trigger Side Effects

Wrong assumption:

> "This insert only adds one row."

Explanation:

* An insert may also execute one or more triggers that modify other tables.

Correct understanding:

* Always consider whether a table has triggers when debugging unexpected database changes.

---

## Using Triggers Instead of Constraints

Wrong:

Using a trigger to prevent duplicate emails.

Explanation:

* SQLite already provides constraints like `UNIQUE` for this purpose.

Correct:

```dart
TextColumn get email => text().unique()();
```

Explanation:

* Use built-in constraints whenever possible.
* Reserve triggers for behavior that constraints cannot express.

---

# Related APIs

* Custom SQL
* Constraints
* Default Values
* Transactions
* Migrations

---

# Summary

Triggers allow SQLite to automatically execute SQL when specific database events occur. They're useful for tasks like audit logging, maintaining timestamps, and recording history without requiring application code. Because triggers run automatically inside the database, they should remain simple, predictable, and focused on database-related responsibilities.
