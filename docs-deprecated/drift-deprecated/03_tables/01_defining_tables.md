## Tables 

**Mastering table definitions in Drift**

---

# What is it?

**Tables** are the fundamental building blocks of your database schema. In Drift, you define tables as Dart classes that extend `Table`, with each property representing a column. Drift then generates type-safe code for querying, inserting, and updating these tables.

> **Think of Tables like "blueprints"** – they define the structure of your data, what fields exist, what types they have, and what constraints apply.

```dart
// A complete table definition
class Users extends Table {
  // Primary key with auto-increment
  IntColumn get id => integer().autoIncrement()();
  
  // Required text column with length validation
  TextColumn get name => text().withLength(min: 2, max: 50)();
  
  // Unique email column
  TextColumn get email => text().unique()();
  
  // Optional integer column
  IntColumn get age => integer().nullable()();
  
  // Boolean with default value
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  
  // DateTime with automatic timestamp
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  
  // Custom constraints
  @override
  List<String> get customConstraints => [
    'CHECK (age >= 0 AND age <= 150)',
    'CHECK (email LIKE "%@%")',
  ];
}
```

> **What's happening here?**
> - **Column types** – `IntColumn`, `TextColumn`, `BoolColumn`, `DateTimeColumn`
> - **Modifiers** – `autoIncrement()`, `nullable()`, `unique()`, `withDefault()`
> - **Validation** – `withLength(min: 2, max: 50)`
> - **Constraints** – Custom SQL CHECK constraints
> - **Defaults** – `Constant(true)`, `currentDateAndTime`

---

# Why does it exist?

- **Schema Definition** – Define your database structure in Dart
- **Type Safety** – Columns are strongly typed
- **Code Generation** – Drift generates all the boilerplate
- **Validation** – Enforce data integrity at the database level
- **Relationships** – Define foreign keys and relationships
- **Customization** – Full control over column behavior

---

# Column Types

> **All available column types in Drift**

## Basic Column Types

```dart
class ExampleTable extends Table {
  // 1️⃣ Integer (Dart int)
  IntColumn get id => integer()();
  
  // 2️⃣ Text (Dart String)
  TextColumn get name => text()();
  
  // 3️⃣ Boolean (Dart bool)
  BoolColumn get isActive => boolean()();
  
  // 4️⃣ DateTime (Dart DateTime)
  DateTimeColumn get createdAt => dateTime()();
  
  // 5️⃣ Real (Dart double)
  RealColumn get price => real()();
  
  // 6️⃣ Blob (Dart Uint8List)
  BlobColumn get imageData => blob()();
}
```

## Column Modifiers

```dart
class ModifiersTable extends Table {
  // ✅ Auto-increment (for primary keys)
  IntColumn get id => integer().autoIncrement()();
  
  // ✅ Nullable (optional field)
  TextColumn get description => text().nullable()();
  
  // ✅ Unique constraint
  TextColumn get email => text().unique()();
  
  // ✅ Default value
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  
  // ✅ Current date/time default
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  
  // ✅ Length validation (min/max)
  TextColumn get username => text().withLength(min: 3, max: 20)();
  
  // ✅ Custom name (different from Dart property name)
  TextColumn get userFullName => text().named('full_name')();
  
  // ✅ Custom constraints
  IntColumn get age => integer().customConstraint('CHECK (age >= 0)')();
}
```

> **Key insights:**
> - **`autoIncrement()`** – Automatically generates unique IDs
> - **`nullable()`** – Field can be null in the database
> - **`unique()`** – Ensures no duplicate values
> - **`withDefault()`** – Sets a default value
> - **`currentDateAndTime`** – Inserts current timestamp
> - **`withLength()`** – Validates string length
> - **`named()`** – Custom column name in SQL
> - **`customConstraint()`** – Raw SQL constraint

---

# Primary Keys

> **Different ways to define primary keys**

## Single Column Primary Key

```dart
class Users extends Table {
  // 👇 Simple auto-incrementing primary key
  IntColumn get id => integer().autoIncrement()();
  
  TextColumn get name => text()();
}

// Generated SQL:
// CREATE TABLE users (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   name TEXT NOT NULL
// )
```

## Custom Primary Key

```dart
class Products extends Table {
  // 👇 Custom primary key (not auto-increment)
  @override
  ColumnKey get primaryKey => const ColumnKey('product_code');
  
  TextColumn get productCode => text().named('product_code')();
  TextColumn get name => text()();
  RealColumn get price => real()();
}

// Generated SQL:
// CREATE TABLE products (
//   product_code TEXT PRIMARY KEY,
//   name TEXT NOT NULL,
//   price REAL NOT NULL
// )
```

## Composite Primary Keys

```dart
class OrderItems extends Table {
  IntColumn get orderId => integer()();
  IntColumn get productId => integer()();
  IntColumn get quantity => integer()();
  
  // 👇 Composite primary key (multiple columns)
  @override
  Set<Column> get primaryKey => {orderId, productId};
}

// Generated SQL:
// CREATE TABLE order_items (
//   order_id INTEGER NOT NULL,
//   product_id INTEGER NOT NULL,
//   quantity INTEGER NOT NULL,
//   PRIMARY KEY (order_id, product_id)
// )
```

---

# Constraints

> **Enforcing data integrity with constraints**

## Foreign Key Constraints

```dart
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  
  // 👇 Foreign key to Users table
  IntColumn get userId => integer().references(
    Users, // Target table
    #id,   // Target column
    onDelete: KeyAction.cascade, // Delete posts when user is deleted
    onUpdate: KeyAction.cascade, // Update posts when user ID changes
  )();
}

// Generated SQL:
// CREATE TABLE posts (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   title TEXT NOT NULL,
//   content TEXT NOT NULL,
//   user_id INTEGER NOT NULL REFERENCES users(id)
//     ON DELETE CASCADE ON UPDATE CASCADE
// )
```

## Custom Constraints

```dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  RealColumn get price => real()();
  IntColumn get stock => integer()();
  TextColumn get status => text()();
  
  // 👇 Custom CHECK constraints
  @override
  List<String> get customConstraints => [
    'CHECK (price >= 0)',
    'CHECK (stock >= 0)',
    "CHECK (status IN ('active', 'inactive', 'discontinued'))",
    'CHECK (LENGTH(name) >= 2)',
  ];
}

// Generated SQL:
// CREATE TABLE products (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   name TEXT NOT NULL,
//   price REAL NOT NULL,
//   stock INTEGER NOT NULL,
//   status TEXT NOT NULL,
//   CHECK (price >= 0),
//   CHECK (stock >= 0),
//   CHECK (status IN ('active', 'inactive', 'discontinued')),
//   CHECK (LENGTH(name) >= 2)
// )
```

---

# Default Values

> **Setting default values for columns**

```dart
class DefaultValuesTable extends Table {
  // 👇 Constant default
  BoolColumn get isPublished => boolean().withDefault(const Constant(false))();
  
  // 👇 Current timestamp
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  
  // 👇 Default with expression
  TextColumn get uniqueId => text().withDefault(
    const Constant('UUID()') // SQL expression
  )();
  
  // 👇 Default for optional fields
  IntColumn get priority => integer()
    .nullable()
    .withDefault(const Constant(1))();
  
  // 👇 Complex default using SQL
  TextColumn get fullName => text().withDefault(
    const Constant("(first_name || ' ' || last_name)")
  )();
}

// Generated SQL:
// CREATE TABLE default_values (
//   is_published INTEGER NOT NULL DEFAULT 0,
//   created_at INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   unique_id TEXT NOT NULL DEFAULT UUID(),
//   priority INTEGER DEFAULT 1,
//   full_name TEXT NOT NULL DEFAULT (first_name || ' ' || last_name)
// )
```

> **Key insights:**
> - **`Constant()`** – Static value
> - **`currentDateAndTime`** – Current timestamp (SQLite's `CURRENT_TIMESTAMP`)
> - **Expressions** – SQL expressions as defaults
> - **Nullable defaults** – Default for optional columns

---

# Generated Columns

> **Columns that are computed from other columns**

```dart
class GeneratedColumns extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get firstName => text()();
  TextColumn get lastName => text()();
  
  // 👇 Generated column (computed from other columns)
  TextColumn get fullName => text().generatedAs(
    // 👇 SQL expression
    const Constant("first_name || ' ' || last_name"),
    // 👇 Optional: store or virtual
    stored: true, // Stored in database (vs virtual)
  )();
  
  // 👇 Generated with CASE statement
  TextColumn get nameLength => text().generatedAs(
    const Constant("CASE WHEN LENGTH(first_name) > 5 THEN 'long' ELSE 'short' END"),
    stored: true,
  )();
  
  // 👇 Generated numeric column
  IntColumn get nameLengthInt => integer().generatedAs(
    const Constant("LENGTH(first_name) + LENGTH(last_name)"),
    stored: true,
  )();
}

// Generated SQL:
// CREATE TABLE generated_columns (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   first_name TEXT NOT NULL,
//   last_name TEXT NOT NULL,
//   full_name TEXT GENERATED ALWAYS AS (first_name || ' ' || last_name) STORED,
//   name_length TEXT GENERATED ALWAYS AS (CASE WHEN LENGTH(first_name) > 5 THEN 'long' ELSE 'short' END) STORED,
//   name_length_int INTEGER GENERATED ALWAYS AS (LENGTH(first_name) + LENGTH(last_name)) STORED
// )
```

---

# Custom Types

> **Using custom types with TypeConverter**

```dart
import 'dart:convert';

// 👇 Custom TypeConverter for JSON
class JsonConverter extends TypeConverter<Map<String, dynamic>, String> {
  const JsonConverter();
  
  @override
  Map<String, dynamic> fromSql(String fromDb) {
    return jsonDecode(fromDb) as Map<String, dynamic>;
  }
  
  @override
  String toSql(Map<String, dynamic> value) {
    return jsonEncode(value);
  }
}

// 👇 Custom TypeConverter for List<String>
class StringListConverter extends TypeConverter<List<String>, String> {
  const StringListConverter();
  
  @override
  List<String> fromSql(String fromDb) {
    return fromDb.split(',').where((s) => s.isNotEmpty).toList();
  }
  
  @override
  String toSql(List<String> value) {
    return value.join(',');
  }
}

// 👇 Custom TypeConverter for Enum
enum Status { active, inactive, pending }

class StatusConverter extends TypeConverter<Status, String> {
  const StatusConverter();
  
  @override
  Status fromSql(String fromDb) {
    return Status.values.firstWhere(
      (s) => s.name == fromDb,
      orElse: () => Status.pending,
    );
  }
  
  @override
  String toSql(Status value) {
    return value.name;
  }
}

// 👇 Using custom types in a table
class CustomTypesTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // JSON data
  TextColumn get metadata => text().map(const JsonConverter())();
  
  // List of strings
  TextColumn get tags => text().map(const StringListConverter())();
  
  // Enum
  TextColumn get status => text().map(const StatusConverter())();
  
  // 👇 Custom converter for any type
  TextColumn get customData => text().map(const CustomTypeConverter())();
}

// Generated SQL:
// CREATE TABLE custom_types (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   metadata TEXT NOT NULL,  -- JSON stored as TEXT
//   tags TEXT NOT NULL,      -- List stored as comma-separated TEXT
//   status TEXT NOT NULL,    -- Enum stored as TEXT
//   custom_data TEXT NOT NULL
// )
```

---

# Table Inheritance

> **Reusing table definitions with inheritance**

```dart
// 👇 Base table (not a real table, just shared columns)
abstract class BaseTable extends Table {
  IntColumn get id => integer().autoIncrement()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().nullable()();
  BoolColumn get isDeleted => boolean().withDefault(const Constant(false))();
}

// 👇 Users table inherits base columns
class Users extends BaseTable {
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  IntColumn get age => integer().nullable()();
}

// 👇 Posts table inherits base columns
class Posts extends BaseTable {
  TextColumn get title => text()();
  TextColumn get content => text()();
  IntColumn get userId => integer().references(Users, #id)();
}

// 👇 Products table inherits base columns
class Products extends BaseTable {
  TextColumn get name => text()();
  RealColumn get price => real()();
  IntColumn get stock => integer()();
}

// Generated SQL for Users:
// CREATE TABLE users (
//   id INTEGER PRIMARY KEY AUTOINCREMENT,
//   created_at INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   updated_at INTEGER,
//   is_deleted INTEGER NOT NULL DEFAULT 0,
//   name TEXT NOT NULL,
//   email TEXT NOT NULL UNIQUE,
//   age INTEGER
// )
```

---

# Complete Table Example

> **Production-ready table with all features**

```dart
// lib/database/tables/users.dart
import 'package:drift/drift.dart';

class Users extends Table {
  // 1️⃣ Primary Key
  IntColumn get id => integer().autoIncrement()();
  
  // 2️⃣ Required fields with validation
  TextColumn get username => text()
    .withLength(min: 3, max: 30)
    .unique()
    .customConstraint('CHECK (LENGTH(username) >= 3)')();
  
  TextColumn get email => text()
    .unique()
    .customConstraint('CHECK (email LIKE "%@%")')();
  
  // 3️⃣ Optional field
  TextColumn get fullName => text().nullable()();
  
  // 4️⃣ Number fields
  IntColumn get age => integer()
    .nullable()
    .customConstraint('CHECK (age >= 0 AND age <= 150)')();
  
  RealColumn get balance => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (balance >= 0)')();
  
  // 5️⃣ Boolean with default
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))();
  
  BoolColumn get isVerified => boolean()
    .withDefault(const Constant(false))();
  
  // 6️⃣ DateTime fields
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)();
  
  DateTimeColumn get updatedAt => dateTime()
    .nullable()();
  
  DateTimeColumn get lastLogin => dateTime()
    .nullable()();
  
  // 7️⃣ JSON data (custom type)
  TextColumn get preferences => text()
    .nullable()
    .map(const JsonConverter())();
  
  // 8️⃣ Custom constraints
  @override
  List<String> get customConstraints => [
    'CHECK (LENGTH(email) > 0)',
    'CHECK (age >= 0 OR age IS NULL)',
  ];
  
  // 9️⃣ Indexes
  @override
  List<Index> get indexes => [
    Index('idx_users_username', 'username'),
    Index('idx_users_email', 'email'),
    Index('idx_users_is_active', 'is_active'),
    Index('idx_users_created_at', 'created_at'),
  ];
}

// lib/database/tables/posts.dart
import 'package:drift/drift.dart';
import 'users.dart';

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get userId => integer().references(
    Users, 
    #id,
    onDelete: KeyAction.cascade,
  )();
  
  TextColumn get title => text()
    .withLength(min: 1, max: 200)
    .customConstraint('CHECK (LENGTH(title) > 0)')();
  
  TextColumn get content => text()
    .withLength(min: 1)
    .customConstraint('CHECK (LENGTH(content) > 0)')();
  
  TextColumn get excerpt => text()
    .nullable()
    .withLength(max: 500)();
  
  BoolColumn get isPublished => boolean()
    .withDefault(const Constant(false))();
  
  DateTimeColumn get publishedAt => dateTime()
    .nullable()();
  
  IntColumn get views => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (views >= 0)')();
  
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)();
  
  DateTimeColumn get updatedAt => dateTime()
    .nullable()();
  
  // JSON tags
  TextColumn get tags => text()
    .nullable()
    .map(const StringListConverter())();
  
  @override
  List<Index> get indexes => [
    Index('idx_posts_user_id', 'user_id'),
    Index('idx_posts_is_published', 'is_published'),
    Index('idx_posts_created_at', 'created_at'),
    Index('idx_posts_published_at', 'published_at'),
  ];
}
```

---

# Best Practices

- **Use meaningful column names** – Clear and descriptive
- **Add constraints** – Validate data at the database level
- **Use indexes** – For frequently queried columns
- **Set appropriate defaults** – Reduce null checks
- **Use nullable for optional fields** – Clear intent
- **Use custom constraints** – Complex validation rules
- **Use TypeConverter** – For custom data types
- **Use inheritance** – DRY for common columns
- **Add foreign keys** – Maintain referential integrity
- **Use CHECK constraints** – Prevent invalid data

---

# Common Mistakes

## Mistake 1: Forgetting NOT NULL constraints

Wrong:
```dart
// 🚫 Should be NOT NULL
TextColumn get name => text()();
```

Correct:
```dart
// ✅ Explicit NOT NULL
TextColumn get name => text()();
// Drift makes text() NOT NULL by default
```

## Mistake 2: Not using nullable for optional fields

Wrong:
```dart
// 🚫 Can't be null, but should be optional
TextColumn get description => text()();
```

Correct:
```dart
// ✅ Optional field
TextColumn get description => text().nullable()();
```

## Mistake 3: Not handling custom constraints

Wrong:
```dart
// 🚫 Invalid data can be inserted
IntColumn get age => integer()();
```

Correct:
```dart
// ✅ Validate data
IntColumn get age => integer()
  .customConstraint('CHECK (age >= 0 AND age <= 150)')();
```

---

# Summary

| Feature | Purpose | Best Practice |
|---------|---------|---------------|
| **Column Types** | Define data types | Use appropriate type |
| **Primary Keys** | Unique identification | Use auto-increment |
| **Constraints** | Data integrity | Use CHECK constraints |
| **Foreign Keys** | Relationships | Use cascade delete |
| **Default Values** | Automatic values | Use sensible defaults |
| **Indexes** | Performance | Add for frequent queries |
| **Generated Columns** | Computed values | Use for derived data |
| **Custom Types** | Complex data | Use TypeConverter |

---

# Next Steps

Now you understand tables, let's dive deeper:

- [Columns](link) – Advanced column configuration
- [Primary Keys](link) – Complete primary key guide
- [Constraints](link) – Advanced constraints

---

# Did You Know?

- **Drift supports over 10 column types** – Including custom types

- **Indexes can speed up queries by 100x** – On large tables

- **CHECK constraints are enforced at the database level** – Even if you insert via raw SQL

- **Foreign keys with cascade** – Automatically delete related records

- **Generated columns are computed by SQLite** – No Dart code needed

- **TypeConverters are reusable** – You can share them across tables

- **Table inheritance reduces duplication** – Common columns in base tables

---

