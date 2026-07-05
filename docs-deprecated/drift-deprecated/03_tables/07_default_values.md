## Default Values

**Setting automatic values for columns in Drift**

---

# What is it?

**Default Values** are automatic values that are assigned to a column when no explicit value is provided during insertion. Drift provides flexible ways to set default values, from static constants to dynamic expressions like current timestamps.

> **Think of Default Values like "automatic fill-in forms"** – if you don't fill in a field, it automatically gets a predefined value, saving you time and ensuring consistency.

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 👇 Static default value
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
  
  // 👇 Dynamic default (current timestamp)
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  
  // 👇 Default with SQL expression
  TextColumn get status => text()
    .withDefault(const Constant("'active'"))();
}
```

> **What's happening here?**
> - **Static defaults** – Constant values like `true`, `false`, `0`, `'default'`
> - **Dynamic defaults** – Functions like `currentDateAndTime`
> - **Expression defaults** – SQL expressions like `'active'`, `random()`
> - **Automatic assignment** – Applied when no value is provided

---

# Why does it exist?

- **Convenience** – Automatically fill common values
- **Consistency** – Ensure consistent default states
- **Data Integrity** – Prevent NULL values in NOT NULL columns
- **Time Tracking** – Automatically record creation times
- **Business Logic** – Set default statuses and states
- **Developer Experience** – Less code to write

---

# Types of Default Values

> **All the ways to set default values in Drift**

## Static Defaults (Constants)

```dart
class StaticDefaults extends Table {
  // 👇 Boolean default
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))();
  
  // 👇 Integer default
  IntColumn get score => integer()
    .withDefault(const Constant(0))();
  
  // 👇 Text default
  TextColumn get status => text()
    .withDefault(const Constant('pending'))();
  
  // 👇 Real default
  RealColumn get rating => real()
    .withDefault(const Constant(0.0))();
  
  // 👇 Nullable with default
  IntColumn get age => integer()
    .nullable()
    .withDefault(const Constant(18))();
}

// Generated SQL:
// CREATE TABLE static_defaults (
//   is_active INTEGER NOT NULL DEFAULT 1,
//   score INTEGER NOT NULL DEFAULT 0,
//   status TEXT NOT NULL DEFAULT 'pending',
//   rating REAL NOT NULL DEFAULT 0.0,
//   age INTEGER DEFAULT 18
// )
```

> **Key insights:**
> - **Constants** – Values that never change
> - **Type matching** – Default must match column type
> - **NOT NULL** – Default ensures column always has a value
> - **Nullable** – Default provides value even for nullable columns

---

## Dynamic Defaults (Functions)

```dart
class DynamicDefaults extends Table {
  // 👇 Current timestamp
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)();
  
  // 👇 Current date (without time)
  DateTimeColumn get createdDate => dateTime()
    .withDefault(currentDate)();
  
  // 👇 Custom timestamp
  DateTimeColumn get updatedAt => dateTime()
    .withDefault(currentDateAndTime)();
  
  // 👇 Random value (SQL expression)
  IntColumn get randomId => integer()
    .withDefault(const Constant('(ABS(RANDOM()) % 1000000)'))();
  
  // 👇 UUID (SQL expression)
  TextColumn get uuid => text()
    .withDefault(const Constant('(UUID())'))();
}

// Generated SQL:
// CREATE TABLE dynamic_defaults (
//   created_at INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   created_date INTEGER NOT NULL DEFAULT CURRENT_DATE,
//   updated_at INTEGER NOT NULL DEFAULT CURRENT_TIMESTAMP,
//   random_id INTEGER NOT NULL DEFAULT (ABS(RANDOM()) % 1000000),
//   uuid TEXT NOT NULL DEFAULT (UUID())
// )
```

> **Key insights:**
> - **`currentDateAndTime`** – SQLite's `CURRENT_TIMESTAMP`
> - **`currentDate`** – SQLite's `CURRENT_DATE`
> - **SQL expressions** – Use `Constant` with raw SQL
> - **Functions** – `RANDOM()`, `UUID()`, `ABS()`

---

## Combining Defaults with Other Modifiers

```dart
class CombinedDefaults extends Table {
  // 👇 Default with NOT NULL
  TextColumn get status => text()
    .withDefault(const Constant('active'))
    .customConstraint('CHECK (status IN ("active", "inactive"))')();
  
  // 👇 Default with UNIQUE
  TextColumn get username => text()
    .withDefault(const Constant('user_123'))
    .unique()();
  
  // 👇 Default with nullable
  IntColumn get age => integer()
    .nullable()
    .withDefault(const Constant(18))();
  
  // 👇 Default with foreign key
  IntColumn get categoryId => integer()
    .references(Categories, #id)
    .withDefault(const Constant(1))();
}

// Generated SQL:
// CREATE TABLE combined_defaults (
//   status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ("active", "inactive")),
//   username TEXT NOT NULL DEFAULT 'user_123' UNIQUE,
//   age INTEGER DEFAULT 18,
//   category_id INTEGER NOT NULL DEFAULT 1 REFERENCES categories(id)
// )
```

---

# Companion Classes and Defaults

> **How defaults work with Companion classes**

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))();
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)();
}

// Generated Companion class (simplified)
class UsersCompanion extends UpdateCompanion<User> {
  final Value<int> id;
  final Value<String> name;
  final Value<bool> isActive;
  final Value<DateTime> createdAt;
  
  const UsersCompanion({
    this.id = const Value.absent(),
    this.name = const Value.absent(),
    this.isActive = const Value.absent(),
    this.createdAt = const Value.absent(),
  });
  
  // 👇 Insert companion uses defaults
  UsersCompanion.insert({
    this.id = const Value.absent(),
    required String name,
    this.isActive = const Value.absent(), // Defaults to true
    this.createdAt = const Value.absent(), // Defaults to current timestamp
  }) : super.insert();
}

// Usage:
// 1️⃣ Uses defaults
await into(users).insert(
  UsersCompanion.insert(
    name: 'John',
    // isActive will default to true
    // createdAt will default to current time
  ),
);

// 2️⃣ Override defaults
await into(users).insert(
  UsersCompanion.insert(
    name: 'John',
    isActive: Value(false), // Override default
    createdAt: Value(DateTime(2024, 1, 1)), // Override default
  ),
);
```

---

# Common Default Patterns

> **Typical default value use cases**

## Pattern 1: Active/Inactive Status

```dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 👇 Active by default
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))
    .named('is_active')();
  
  // 👇 Pending by default
  TextColumn get status => text()
    .withDefault(const Constant('pending'))
    .customConstraint("CHECK (status IN ('pending', 'approved', 'rejected'))")();
}
```

## Pattern 2: Timestamps

```dart
class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  
  // 👇 Creation time (never changes)
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
  
  // 👇 Update time (changes on update)
  DateTimeColumn get updatedAt => dateTime()
    .nullable()
    .named('updated_at')();
}
```

## Pattern 3: Counters and Scores

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 👇 Start with zero
  IntColumn get score => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (score >= 0)')();
  
  // 👇 Start with default level
  IntColumn get level => integer()
    .withDefault(const Constant(1))
    .customConstraint('CHECK (level >= 1)')();
  
  // 👇 Rating starts neutral
  RealColumn get rating => real()
    .withDefault(const Constant(0.0))
    .customConstraint('CHECK (rating >= 0 AND rating <= 5)')();
}
```

## Pattern 4: Configuration Values

```dart
class Settings extends Table {
  TextColumn get key => text().unique()();
  TextColumn get value => text()();
  
  // 👇 Default for boolean settings
  BoolColumn get isEnabled => boolean()
    .withDefault(const Constant(true))
    .named('is_enabled')();
  
  // 👇 Default for text settings
  TextColumn get theme => text()
    .withDefault(const Constant('light'))
    .named('theme')();
  
  // 👇 Default for numeric settings
  IntColumn get pageSize => integer()
    .withDefault(const Constant(20))
    .named('page_size')
    .customConstraint('CHECK (page_size >= 5 AND page_size <= 100)')();
}
```

---

# Default Values with SQL Expressions

> **Using SQL expressions for defaults**

```dart
class ExpressionDefaults extends Table {
  // 👇 UUID generation
  TextColumn get uuid => text()
    .withDefault(const Constant('(LOWER(HEX(RANDOMBLOB(16))))'))()
    .unique()();
  
  // 👇 Random number
  IntColumn get randomNumber => integer()
    .withDefault(const Constant('(ABS(RANDOM()) % 1000)'))();
  
  // 👇 Current date (no time)
  TextColumn get date => text()
    .withDefault(const Constant('DATE("now")'))();
  
  // 👇 Current year
  IntColumn get year => integer()
    .withDefault(const Constant('STRFTIME("%Y", "now")'))();
  
  // 👇 Combination of values
  TextColumn get shortId => text()
    .withDefault(const Constant('(UPPER(SUBSTR(UUID(), 1, 8)))'))();
}

// Generated SQL:
// CREATE TABLE expression_defaults (
//   uuid TEXT NOT NULL DEFAULT (LOWER(HEX(RANDOMBLOB(16)))) UNIQUE,
//   random_number INTEGER NOT NULL DEFAULT (ABS(RANDOM()) % 1000),
//   date TEXT NOT NULL DEFAULT DATE("now"),
//   year INTEGER NOT NULL DEFAULT STRFTIME("%Y", "now"),
//   short_id TEXT NOT NULL DEFAULT (UPPER(SUBSTR(UUID(), 1, 8)))
// )
```

---

# Real-World Example

> **Complete application with default values**

```dart
// lib/database/tables/users.dart
import 'package:drift/drift.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get username => text().unique()();
  TextColumn get email => text().unique()();
  TextColumn get fullName => text().nullable().named('full_name')();
  TextColumn get passwordHash => text().named('password_hash')();
  
  // 👇 Boolean defaults
  BoolColumn get isActive => boolean()
    .withDefault(const Constant(true))
    .named('is_active')();
  
  BoolColumn get isVerified => boolean()
    .withDefault(const Constant(false))
    .named('is_verified')();
  
  BoolColumn get isAdmin => boolean()
    .withDefault(const Constant(false))
    .named('is_admin')();
  
  // 👇 Numeric defaults
  IntColumn get loginCount => integer()
    .withDefault(const Constant(0))
    .named('login_count')();
  
  IntColumn get rank => integer()
    .withDefault(const Constant(1))
    .named('rank')();
  
  // 👇 Text defaults
  TextColumn get status => text()
    .withDefault(const Constant('active'))
    .customConstraint("CHECK (status IN ('active', 'suspended', 'banned'))")();
  
  TextColumn get language => text()
    .withDefault(const Constant('en'))
    .withLength(max: 2)();
  
  // 👇 Timestamp defaults
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
  
  DateTimeColumn get updatedAt => dateTime()
    .nullable()
    .named('updated_at')();
  
  DateTimeColumn get lastLogin => dateTime()
    .nullable()
    .named('last_login')();
  
  DateTimeColumn get birthDate => dateTime()
    .nullable()
    .named('birth_date')();
  
  // 👇 Indexes
  @override
  List<Index> get indexes => [
    Index('idx_users_email', 'email'),
    Index('idx_users_status', 'status'),
    Index('idx_users_created_at', 'created_at'),
  ];
}

// lib/database/tables/posts.dart
import 'package:drift/drift.dart';
import 'users.dart';

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get userId => integer()
    .references(Users, #id)
    .named('user_id')();
  TextColumn get title => text()();
  TextColumn get content => text()();
  
  // 👇 Default statuses
  TextColumn get status => text()
    .withDefault(const Constant('draft'))
    .customConstraint("CHECK (status IN ('draft', 'published', 'archived'))")();
  
  // 👇 Default counts
  IntColumn get views => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (views >= 0)')();
  
  IntColumn get likes => integer()
    .withDefault(const Constant(0))
    .customConstraint('CHECK (likes >= 0)')();
  
  // 👇 Default flags
  BoolColumn get isPublished => boolean()
    .withDefault(const Constant(false))
    .named('is_published')();
  
  BoolColumn get isFeatured => boolean()
    .withDefault(const Constant(false))
    .named('is_featured')();
  
  BoolColumn get isDeleted => boolean()
    .withDefault(const Constant(false))
    .named('is_deleted')();
  
  // 👇 Timestamps
  DateTimeColumn get createdAt => dateTime()
    .withDefault(currentDateAndTime)
    .named('created_at')();
  
  DateTimeColumn get updatedAt => dateTime()
    .nullable()
    .named('updated_at')();
  
  DateTimeColumn get publishedAt => dateTime()
    .nullable()
    .named('published_at')();
  
  // 👇 Indexes
  @override
  List<Index> get indexes => [
    Index('idx_posts_user', 'user_id'),
    Index('idx_posts_status', 'status'),
    Index('idx_posts_published', 'is_published'),
    Index('idx_posts_created', 'created_at'),
  ];
}
```

```dart
// lib/database/database.dart - Usage
import 'package:drift/drift.dart';
import 'tables/users.dart';
import 'tables/posts.dart';

@DriftDatabase(tables: [Users, Posts])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;

  // 👇 Using defaults
  Future<int> createUser({
    required String username,
    required String email,
    required String passwordHash,
    String? fullName,
  }) async {
    // Most defaults will be applied automatically
    return await into(users).insert(
      UsersCompanion.insert(
        username: username,
        email: email,
        passwordHash: passwordHash,
        fullName: Value(fullName),
        // isActive: defaults to true
        // isVerified: defaults to false
        // isAdmin: defaults to false
        // loginCount: defaults to 0
        // rank: defaults to 1
        // status: defaults to 'active'
        // language: defaults to 'en'
        // createdAt: defaults to current time
      ),
    );
  }

  // 👇 Overriding defaults
  Future<int> createVerifiedUser({
    required String username,
    required String email,
    required String passwordHash,
  }) async {
    return await into(users).insert(
      UsersCompanion.insert(
        username: username,
        email: email,
        passwordHash: passwordHash,
        isVerified: Value(true), // 👈 Override default
        status: Value('active'),
      ),
    );
  }

  // 👇 Creating posts with defaults
  Future<int> createPost({
    required int userId,
    required String title,
    required String content,
    bool publish = false,
  }) async {
    return await into(posts).insert(
      PostsCompanion.insert(
        userId: userId,
        title: title,
        content: content,
        status: Value(publish ? 'published' : 'draft'),
        isPublished: Value(publish),
        publishedAt: publish ? Value(DateTime.now()) : Value.absent(),
        // views: defaults to 0
        // likes: defaults to 0
        // isFeatured: defaults to false
        // isDeleted: defaults to false
        // createdAt: defaults to current time
      ),
    );
  }

  // 👇 Using default values in queries
  Future<List<User>> getDefaultUsers() async {
    // Users with default isActive = true
    return await (select(users)
      ..where((u) => u.isActive.equals(true)))
      .get();
  }

  // 👇 Checking default values
  Future<Map<String, dynamic>> getUserDefaults() async {
    // Insert a user with all defaults
    final id = await createUser(
      username: 'default_test',
      email: 'test@example.com',
      passwordHash: 'hash',
    );
    
    final user = await (select(users)
      ..where((u) => u.id.equals(id)))
      .getSingle();
    
    return {
      'id': user.id,
      'username': user.username,
      'email': user.email,
      'isActive': user.isActive,
      'isVerified': user.isVerified,
      'isAdmin': user.isAdmin,
      'loginCount': user.loginCount,
      'rank': user.rank,
      'status': user.status,
      'language': user.language,
      'createdAt': user.createdAt,
    };
  }
}
```

---

# Best Practices

- **Use sensible defaults** – Values that make sense for most cases
- **Use `currentDateAndTime`** – For creation timestamps
- **Use boolean defaults** – `true` or `false` based on common state
- **Set default status** – `'active'`, `'pending'`, `'draft'`
- **Use default counters** – Start at 0 or 1
- **Consider nullable columns** – Defaults can still apply
- **Use `Value.absent()`** – To use default in Companions
- **Document defaults** – Explain why they're set
- **Test defaults** – Verify they work as expected

---

# Common Mistakes

## Mistake 1: Wrong default type

Wrong:
```dart
// 🚫 Type mismatch (String vs Int)
IntColumn get age => integer()
  .withDefault(const Constant('18'))();
```

Correct:
```dart
// ✅ Correct type
IntColumn get age => integer()
  .withDefault(const Constant(18))();
```

## Mistake 2: Not using `Value.absent()`

Wrong:
```dart
// 🚫 Always overrides default
UsersCompanion(
  isActive: Value(true), // Doesn't use default
)
```

Correct:
```dart
// ✅ Uses default when absent
UsersCompanion(
  isActive: const Value.absent(), // Uses default
)
```

## Mistake 3: Using default with auto-increment

Wrong:
```dart
// 🚫 Auto-increment doesn't work with default
IntColumn get id => integer()
  .autoIncrement()
  .withDefault(const Constant(1))();
```

Correct:
```dart
// ✅ No default for auto-increment
IntColumn get id => integer().autoIncrement()();
```

---

# Summary

| Default Type | Example | Use Case |
|--------------|---------|----------|
| **Boolean** | `withDefault(const Constant(true))` | Active states |
| **Integer** | `withDefault(const Constant(0))` | Counters |
| **Text** | `withDefault(const Constant('pending'))` | Statuses |
| **Timestamp** | `withDefault(currentDateAndTime)` | Creation time |
| **SQL Expression** | `withDefault(const Constant('(UUID())'))` | UUIDs, random values |

---

# Next Steps

Now you understand default values, let's dive deeper:

- [Generated Columns](link) – Computed columns
- [Custom Types](link) – Custom type converters
- [Table Inheritance](link) – Reusing table definitions

---

# Did You Know?

- **Defaults are applied at the database level** – Not in Dart

- **`currentDateAndTime` uses SQLite's `CURRENT_TIMESTAMP`** – UTC time

- **Default expressions can use SQL functions** – `RANDOM()`, `UUID()`, `STRFTIME()`

- **Companion `insert` uses defaults** – When values are `Value.absent()`

- **Defaults can reference other columns** – Using SQL expressions

- **DEFAULT NULL is allowed** – For nullable columns

- **Defaults are ignored** – When you explicitly provide a value

- **Default values can be changed** – With ALTER TABLE migrations

---

