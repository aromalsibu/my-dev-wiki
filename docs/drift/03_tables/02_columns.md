## Columns

**Mastering column definitions in Drift**

---

# What is it?

**Columns** are the individual fields within a database table. In Drift, columns are defined as properties in your Table classes, with each property representing a column in the database table. Drift provides a rich API for configuring column behavior, constraints, and types.

> **Think of Columns like the "slots" in a filing cabinet** – each slot holds a specific type of information, with rules about what can go in it.

```dart
class Users extends Table {
  // A simple column definition
  TextColumn get name => text()();
  
  // A column with multiple configurations
  TextColumn get email => text()
    .unique()
    .withLength(max: 100)
    .customConstraint('CHECK (email LIKE "%@%")')
    .named('user_email')();
}
```

> **What's happening here?**
> - **Column Type** – `text()`, `integer()`, `boolean()`, etc.
> - **Modifiers** – `.unique()`, `.nullable()`, `.withDefault()`
> - **Validations** – `.withLength()`, `.customConstraint()`
> - **Naming** – `.named()` for custom SQL column names

---

# Why does it exist?

- **Data Typing** – Ensure correct data types in the database
- **Data Integrity** – Enforce constraints at the database level
- **Validation** – Prevent invalid data from being stored
- **Code Generation** – Generate type-safe Dart code
- **Performance** – Indexes and constraints for query optimization
- **Documentation** – Self-documenting schema

---

# Column Types

> **All available column types and their Dart equivalents**

## Basic Column Types

```dart
class AllColumnTypes extends Table {
  // 1️⃣ INTEGER (Dart int)
  IntColumn get id => integer()();
  
  // 2️⃣ TEXT (Dart String)
  TextColumn get name => text()();
  
  // 3️⃣ REAL (Dart double)
  RealColumn get price => real()();
  
  // 4️⃣ BOOLEAN (Dart bool) - stored as INTEGER 0/1
  BoolColumn get isActive => boolean()();
  
  // 5️⃣ DATETIME (Dart DateTime)
  DateTimeColumn get createdAt => dateTime()();
  
  // 6️⃣ BLOB (Dart Uint8List)
  BlobColumn get imageData => blob()();
  
  // 7️⃣ NUMERIC (Dart num)
  NumericColumn get rating => numeric()();
}
```

## Nullability

```dart
class NullabilityExample extends Table {
  // 👇 NOT NULL (default)
  TextColumn get required => text()();
  
  // 👇 NULL allowed
  TextColumn get optional => text().nullable()();
  
  // 👇 NOT NULL with default
  TextColumn get requiredWithDefault => text()
    .withDefault(const Constant('default'))();
  
  // 👇 NULL allowed with default
  TextColumn get optionalWithDefault => text()
    .nullable()
    .withDefault(const Constant('default'))();
}
```

> **Key insights:**
> - **By default**, columns are NOT NULL
> - **`.nullable()`** allows NULL values
> - **`.withDefault()`** provides a default value
> - **NULL** is different from empty string or 0

---

# Column Modifiers

> **All modifiers available for columns**

## Basic Modifiers

```dart
class ModifierExample extends Table {
  // 1️⃣ Auto-increment (for primary keys)
  IntColumn get id => integer().autoIncrement()();
  
  // 2️⃣ Unique constraint
  TextColumn get email => text().unique()();
  
  // 3️⃣ Nullable (allows NULL)
  TextColumn get description => text().nullable()();
  
  // 4️⃣ Default value
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))();
  
  // 5️⃣ Current timestamp
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)();
  
  // 6️⃣ Custom column name
  TextColumn get userFullName => text().named('full_name')();
  
  // 7️⃣ Custom constraint
  IntColumn get age => integer()
    .customConstraint('CHECK (age >= 0)')();
  
  // 8️⃣ Length validation
  TextColumn get username => text()
    .withLength(min: 3, max: 20)();
}
```

## Combining Modifiers

```dart
class CombinedModifiers extends Table {
  // 👇 Multiple modifiers combined
  TextColumn get email => text()
    .unique()                    // Unique constraint
    .withLength(max: 100)        // Max length
    .nullable()                  // Can be NULL
    .named('user_email')();      // Custom SQL name
  
  // 👇 Multiple constraints
  IntColumn get age => integer()
    .nullable()
    .customConstraint('CHECK (age >= 0)')
    .customConstraint('CHECK (age <= 150)')();
  
  // 👇 Default with validation
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .customConstraint("CHECK (status IN ('active', 'inactive', 'pending'))")();
}
```

---

# Text Column Options

> **Special options for text columns**

```dart
class TextColumnExample extends Table {
  // 1️⃣ Length validation
  TextColumn get short => text().withLength(max: 10)();
  TextColumn get medium => text().withLength(min: 5, max: 50)();
  TextColumn get long => text().withLength(min: 10)();
  
  // 2️⃣ Case sensitivity (for LIKE queries)
  TextColumn get caseSensitive => text()
    .named('sensitive')();
  // Queries are case-sensitive by default
  
  // 3️⃣ Collation (sorting rules)
  TextColumn get caseInsensitive => text()
    .withCollation('NOCASE')(); // Case-insensitive sorting
  
  TextColumn get unicode => text()
    .withCollation('UNICODE')(); // Unicode sorting
  
  // 4️⃣ Combined with other modifiers
  TextColumn get username => text()
    .withLength(min: 3, max: 20)
    .unique()
    .withCollation('NOCASE')
    .named('user_name')();
}
```

> **Key insights:**
> - **`.withLength(min, max)`** – Validate string length
> - **`.withCollation()`** – Define sorting rules
> - **NOCASE** – Case-insensitive comparisons
> - **UNICODE** – Proper Unicode sorting

---

# Integer Column Options

> **Special options for integer columns**

```dart
class IntegerColumnExample extends Table {
  // 1️⃣ Auto-increment (for primary keys)
  IntColumn get id => integer().autoIncrement()();
  
  // 2️⃣ Unsigned (positive only)
  IntColumn get positiveNumber => integer()
    .customConstraint('CHECK (value >= 0)')();
  
  // 3️⃣ Range validation
  IntColumn get score => integer()
    .customConstraint('CHECK (score >= 0 AND score <= 100)')();
  
  // 4️⃣ Default with expression
  IntColumn get randomId => integer()
    .withDefault(const Constant('(ABS(RANDOM()) % 1000000)'))();
  
  // 5️⃣ Combined
  IntColumn get age => integer()
    .nullable()
    .customConstraint('CHECK (age >= 0 AND age <= 150)')
    .withDefault(const Constant(18))();
}
```

> **Key insights:**
> - **`.autoIncrement()`** – Automatically generates unique IDs
> - **`CHECK` constraints** – Validate numeric ranges
> - **Default expressions** – Use SQL expressions for defaults

---

# DateTime Column Options

> **Special options for datetime columns**

```dart
class DateTimeColumnExample extends Table {
  // 1️⃣ Auto timestamp (current time)
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)();
  
  // 2️⃣ Auto timestamp with custom format
  DateTimeColumn get updatedAt => dateTime()
    .withDefault(currentDateAndTime)();
  
  // 3️⃣ Default to specific date
  DateTimeColumn get startDate => dateTime()
    .withDefault(const Constant("'2024-01-01'"))();
  
  // 4️⃣ Nullable date
  DateTimeColumn get endDate => dateTime()
    .nullable()();
  
  // 5️⃣ Date only (no time)
  DateTimeColumn get birthDate => dateTime()
    .nullable()
    .customConstraint('CHECK (birth_date IS NOT NULL)')();
  
  // 6️⃣ Combined
  DateTimeColumn get lastLogin => dateTime()
    .nullable()
    .withDefault(currentDateAndTime)
    .named('last_login')();
}
```

> **Key insights:**
> - **`currentDateAndTime`** – Inserts current timestamp
> - **SQLite stores dates as integers** – Unix timestamps
> - **Drift handles conversion** – Between DateTime and SQLite
> - **Check constraints** – Validate date ranges

---

# Boolean Column Options

> **Special options for boolean columns**

```dart
class BooleanColumnExample extends Table {
  // 1️⃣ Simple boolean
  BoolColumn get isActive => boolean()();
  
  // 2️⃣ Default true
  BoolColumn get isVerified => boolean()
    .withDefault(const Constant(true))();
  
  // 3️⃣ Default false
  BoolColumn get isDeleted => boolean()
    .withDefault(const Constant(false))();
  
  // 4️⃣ Nullable boolean
  BoolColumn get isApproved => boolean()
    .nullable()();
  
  // 5️⃣ Combined
  BoolColumn get isPublished => boolean()
    .withDefault(const Constant(false))
    .named('published')
    .customConstraint('CHECK (published IN (0, 1))')();
}
```

> **Key insights:**
> - **Stored as INTEGER** – 0 = false, 1 = true
> - **`.withDefault()`** – Set default boolean value
> - **Check constraints** – Ensure valid values
> - **Nullable boolean** – Three-state logic (true, false, null)

---

# Real-World Example

> **Complete user table with all column types**

```dart
// lib/database/tables/users.dart
import 'package:drift/drift.dart';
import 'package:uuid/uuid.dart';

class Users extends Table {
  // 1️⃣ Primary Key
  IntColumn get id => integer().autoIncrement()();
  
  // 2️⃣ Basic text fields
  TextColumn get username => text()
    .withLength(min: 3, max: 30)
    .unique()
    .customConstraint('CHECK (LENGTH(username) >= 3)')
    .named('user_name')();
  
  TextColumn get email => text()
    .withLength(max: 100)
    .unique()
    .customConstraint('CHECK (email LIKE "%@%")')();
  
  TextColumn get fullName => text()
    .withLength(max: 100)
    .nullable()
    .named('full_name')();
  
  // 3️⃣ Optional text
  TextColumn get bio => text()
    .withLength(max: 500)
    .nullable()();
  
  TextColumn get website => text()
    .withLength(max: 100)
    .nullable()
    .customConstraint('CHECK (website IS NULL OR website LIKE "http%")')();
  
  // 4️⃣ Numeric columns
  IntColumn get age => integer()
    .nullable()
    .customConstraint('CHECK (age >= 0 AND age <= 150)')();
  
  RealColumn get rating => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (rating >= 0 AND rating <= 5)')();
  
  // 5️⃣ Boolean columns
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))();
  
  BoolColumn get isVerified => boolean()
    .withDefault(const Constant(false))();
  
  BoolColumn get isPremium => boolean()
    .withDefault(const Constant(false))
    .named('premium')();
  
  BoolColumn get isDeleted => boolean()
    .withDefault(const Constant(false))
    .named('deleted')();
  
  // 6️⃣ DateTime columns
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
  
  DateTimeColumn get updatedAt => dateTime()
    .nullable()
    .withDefault(currentDateAndTime)
    .named('updated_at')();
  
  DateTimeColumn get lastLogin => dateTime()
    .nullable()
    .named('last_login')();
  
  DateTimeColumn get birthDate => dateTime()
    .nullable()
    .named('birth_date')();
  
  // 7️⃣ Custom constraints
  @override
  List<String> get customConstraints => [
    'CHECK (email != "")',
    'CHECK (username != "")',
    'CHECK (age IS NULL OR (age >= 0 AND age <= 150))',
  ];
  
  // 8️⃣ Indexes
  @override
  List<Index> get indexes => [
    Index('idx_users_username', 'user_name'),
    Index('idx_users_email', 'email'),
    Index('idx_users_is_active', 'active'),
    Index('idx_users_is_verified', 'verified'),
    Index('idx_users_created_at', 'created_at'),
    Index('idx_users_last_login', 'last_login'),
  ];
}

// Usage in queries
class AppDatabase extends _$AppDatabase {
  // Insert with all columns
  Future<int> createUser({
    required String username,
    required String email,
    String? fullName,
    String? bio,
    int? age,
  }) async {
    return await into(users).insert(
      UsersCompanion.insert(
        username: username,
        email: email,
        fullName: Value(fullName),
        bio: Value(bio),
        age: Value(age),
        // Default values will be used for other columns
      ),
    );
  }

  // Update with selective columns
  Future<void> updateUser(int id, {
    String? username,
    String? email,
    String? fullName,
    String? bio,
    int? age,
    bool? isActive,
  }) async {
    final companion = UsersCompanion(
      username: Value(username),
      email: Value(email),
      fullName: Value(fullName),
      bio: Value(bio),
      age: Value(age),
      isActive: Value(isActive),
    );
    
    await (update(users)..where((u) => u.id.equals(id)))
      .write(companion);
  }
}
```

---

# Best Practices

- **Use appropriate column types** – Match data to correct type
- **Add NOT NULL constraints** – For required fields (default)
- **Use `.nullable()`** – For optional fields
- **Add UNIQUE constraints** – For unique identifiers like email
- **Add CHECK constraints** – For data validation
- **Set sensible defaults** – Reduce null checks
- **Use `.withLength()`** – Prevent oversized strings
- **Add indexes** – For frequently queried columns
- **Use `.named()`** – For custom SQL column names
- **Document constraints** – Add comments for complex rules

---

# Common Mistakes

## Mistake 1: Not adding length constraints

Wrong:
```dart
// 🚫 No length limit
TextColumn get name => text()();
```

Correct:
```dart
// ✅ Limit text length
TextColumn get name => text().withLength(max: 50)();
```

## Mistake 2: Forgetting to use `.nullable()`

Wrong:
```dart
// 🚫 Should be nullable
TextColumn get description => text()();
```

Correct:
```dart
// ✅ Optional field
TextColumn get description => text().nullable()();
```

## Mistake 3: Not adding CHECK constraints

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

| Column Type | Dart Type | Default | Key Modifiers |
|-------------|-----------|---------|---------------|
| `integer()` | `int` | 0 | `autoIncrement()`, `nullable()` |
| `text()` | `String` | '' | `withLength()`, `unique()` |
| `boolean()` | `bool` | false | `withDefault()` |
| `dateTime()` | `DateTime` | null | `withDefault(currentDateAndTime)` |
| `real()` | `double` | 0.0 | `nullable()` |
| `blob()` | `Uint8List` | null | `nullable()` |
| `numeric()` | `num` | 0 | `nullable()` |

---

# Next Steps

Now you understand columns, let's dive deeper:

- [Constraints](link) – Advanced constraints
- [Primary Keys](link) – Complete primary key guide
- [Foreign Keys](link) – Relationships between tables

---

# Did You Know?

- **SQLite doesn't have a BOOLEAN type** – Drift stores booleans as INTEGER 0/1

- **Dates are stored as INTEGER** – Unix timestamps in SQLite

- **TEXT columns can store up to 1GB** – When not limited by `withLength()`

- **INTEGER primary keys are faster** – For lookups than TEXT keys

- **CHECK constraints are evaluated** – Before INSERT and UPDATE operations

- **Column names are case-insensitive** – In SQLite

- **Default values can be expressions** – Like `CURRENT_TIMESTAMP`

- **Indexes can be multi-column** – For complex query optimization

---
