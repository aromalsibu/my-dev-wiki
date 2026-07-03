# Multiple Databases

Manage more than one database within the same application.

---

# What is it?

A Drift application can contain multiple databases, each with its own tables, schema, and connection.

While most applications only need a single database, there are situations where separating data into multiple databases makes sense.

Each database is completely independent and manages its own:

* Tables
* Schema version
* Migrations
* Queries
* Transactions
* Database connection

---

# Why does it exist?

Keeping all data in one database isn't always the best approach.

Using multiple databases can help when:

* Different parts of an application are unrelated.
* Data should be isolated for security or organization.
* Separate modules evolve independently.
* Different databases require different storage strategies.

Drift supports multiple databases by allowing you to create multiple database classes.

---

# Syntax

## Defining Multiple Databases

```dart
@DriftDatabase(tables: [Users])
class UserDatabase extends _$UserDatabase {
  UserDatabase(super.executor);

  @override
  int get schemaVersion => 1;
}

@DriftDatabase(tables: [Products])
class ProductDatabase extends _$ProductDatabase {
  ProductDatabase(super.executor);

  @override
  int get schemaVersion => 1;
}
```

Explanation:

* `UserDatabase` manages only the `Users` table.
* `ProductDatabase` manages only the `Products` table.
* Each database has its own generated implementation and schema.

---

## Creating Database Instances

```dart
final userDb = UserDatabase(
  driftDatabase(name: 'users.db'),
);

final productDb = ProductDatabase(
  driftDatabase(name: 'products.db'),
);
```

Explanation:

* Each database uses its own SQLite file.
* Operations on one database do not affect the other.

---

# Mental Model

Think of each database as its own storage system.

```text
Application
     │
     ├──────────────┐
     ▼              ▼
UserDatabase   ProductDatabase
     │              │
 users.db      products.db
```

Each database has its own connection, tables, and migrations.

There is no direct relationship between them.

---

# Examples

## Simple Example

A chat application might separate user accounts from cached messages.

```text
UserDatabase
├── Users
└── Profiles

CacheDatabase
├── Messages
└── Images
```

Each database serves a different purpose.

---

## Real-World Example

A large enterprise application could have:

```text
AuthenticationDatabase
├── Users
├── Roles
└── Permissions

BusinessDatabase
├── Customers
├── Orders
└── Invoices

AnalyticsDatabase
├── Events
├── Metrics
└── Reports
```

Each module can evolve independently with its own schema version and migration history.

---

# When to Use

Use multiple databases when:

* Different modules have completely independent data.
* Separate migration histories are required.
* Different databases use different storage locations.
* Sensitive data should be isolated.
* You're building a modular application.

---

# When NOT to Use

Avoid multiple databases when:

* The data is related.
* Tables need joins or foreign keys.
* Most queries involve data from multiple tables.

In these cases, a single database with multiple tables is usually a better design.

---

# Best Practices

* Prefer a single database unless there's a clear reason to separate data.
* Group related tables in the same database.
* Keep each database focused on one domain.
* Use meaningful names for database files.
* Manage each database's migrations independently.

---

# Common Mistakes

## Splitting related tables across databases

**Wrong**

```text
UserDatabase
└── Users

OrderDatabase
└── Orders
```

Explanation:

* If `Orders` references `Users`, they can't use foreign keys or joins across databases.

**Correct**

```text
AppDatabase
├── Users
└── Orders
```

Explanation:

* Related tables belong in the same database.

---

## Creating unnecessary databases

**Wrong**

```text
UsersDatabase

TasksDatabase

SettingsDatabase

ProductsDatabase
```

Explanation:

* Multiple small databases increase complexity without providing benefits.

**Correct**

```text
AppDatabase
├── Users
├── Tasks
├── Settings
└── Products
```

Explanation:

* Most applications are easier to manage with a single database.

---

## Assuming databases share transactions

**Wrong**

Expecting a transaction in one database to automatically include operations in another database.

Explanation:

* Transactions are scoped to a single database connection.

**Correct**

Treat each database as an independent unit with its own transaction lifecycle.

---

# Related APIs

* GeneratedDatabase
* Database Connection
* Opening the Database
* Transactions
* Foreign Keys
* Inner Join

---

# Summary

Drift supports multiple databases by allowing you to define multiple database classes, each with its own tables, schema, and connection. While this is useful for isolating unrelated data or building modular applications, most projects are better served by a single database containing multiple related tables.
