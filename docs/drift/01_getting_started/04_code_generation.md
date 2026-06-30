
# Code Generation

**Understanding Drift's automatic code generation system**

---

# What is it?

**Code Generation** in Drift is the process where the `drift_dev` package analyzes your table definitions and database classes, then automatically generates all the boilerplate code needed for type-safe database operations.

> **Think of code generation as your "personal coding assistant"** – you write the high-level definitions, and Drift generates the hundreds of lines of boring, repetitive code you'd otherwise have to write yourself.

```dart
// 👇 You write THIS (100 lines)
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  IntColumn get age => integer()();
}

@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  @override
  int get schemaVersion => 1;
}

// 👇 Drift generates THIS (1000+ lines!)
// database.g.dart
// - Data classes (User)
// - Companion classes (UsersCompanion)
// - Table manager (users)
// - Query methods
// - Stream methods
// - Insert/Update/Delete helpers
// - And much more!
```

> **What's happening here?**
> - **Input** – Your table definitions with `@DriftDatabase` annotation
> - **Processing** – `build_runner` runs `drift_dev` generator
> - **Output** – `*.g.dart` files with all database code
> - **Result** – Type-safe, efficient, boilerplate-free development

---

# Why does it exist?

- **Boilerplate Elimination** – Automatically generates data classes, companions, and table managers
- **Type Safety** – Generated code is fully typed, catching errors at compile-time
- **Performance** – Generated code is optimized and doesn't use reflection
- **Maintainability** – Change your schema in one place, everything regenerates
- **Developer Experience** – Autocomplete, refactoring, and navigation work perfectly
- **Compile-Time Validation** – SQL errors are caught before runtime

---

# How It Works

> **The code generation pipeline**

```dart
// 1. Your source code
// lib/database/tables/users.dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get email => text().unique()();
  DateTimeColumn get createdAt => dateTime()();
}

// 2. Your database class
// lib/database/database.dart
import 'tables/users.dart';
part 'database.g.dart';  // 👈 Important!

@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  
  @override
  int get schemaVersion => 1;
}

// 3. Run build_runner
// flutter pub run build_runner build

// 4. Generated code is created
// lib/database/database.g.dart

// 5. Your code now has access to:
// - User class (data class)
// - UsersCompanion (for inserts/updates)
// - users getter (table reference)
// - select(users) for queries
// - into(users) for inserts
// - update(users) for updates
// - delete(users) for deletions
```

> **Key insights:**
> - **`part 'database.g.dart'`** – Must be included to use generated code
> - **`@DriftDatabase`** – Tells Drift which tables to generate
> - **`_$AppDatabase`** – The generated class your database extends
> - **One file per database** – All tables in one database share one generated file

---

# What Gets Generated?

> **Everything Drift creates for you**

```dart
// lib/database/database.g.dart (simplified view)

// 1. ✅ Data Classes
@UseRowClass
class User {
  final int id;
  final String name;
  final String email;
  final DateTime createdAt;
  
  User({required this.id, required this.name, required this.email, required this.createdAt});
  
  Map<String, Object?> toJson() => {...};
}

// 2. ✅ Companion Classes (for inserts/updates)
class UsersCompanion extends UpdateCompanion<User> {
  final Value<int> id;
  final Value<String> name;
  final Value<String> email;
  final Value<DateTime> createdAt;
  
  const UsersCompanion({
    this.id = const Value.absent(),
    this.name = const Value.absent(),
    this.email = const Value.absent(),
    this.createdAt = const Value.absent(),
  });
  
  UsersCompanion.insert({
    this.id = const Value.absent(),
    required String name,
    required String email,
    DateTime? createdAt,
  }) : ...
}

// 3. ✅ Table Manager
class $UsersTable extends TableManager {
  final AppDatabase _db;
  
  $UsersTable(this._db);
  
  Future<List<User>> get() => ...;
  Stream<List<User>> watch() => ...;
  Future<User> getSingle() => ...;
  // ... many more methods
}

// 4. ✅ Database extension
extension DatabaseExtensions on AppDatabase {
  $UsersTable get users => $UsersTable(this);
}

// 5. ✅ Query building helpers
class UserQuery extends Query<User> {
  // Where clauses, order by, limits, etc.
}
```

> **What's happening here?**
> - **Data classes** – Typed objects representing your table rows
> - **Companion classes** – For inserts/updates with `Value<T>` for optional fields
> - **Table managers** – CRUD operations for each table
> - **Extensions** – Adds `users` getter to your database
> - **Query builders** – Fluent API for building type-safe queries

---

# Running Code Generation

> **Different ways to generate code**

## Option 1: One-time Build
```bash
# Generate code once
flutter pub run build_runner build

# With clean (deletes previous generated files)
flutter pub run build_runner build --delete-conflicting-outputs
```

## Option 2: Watch Mode (Recommended)
```bash
# Continuously watch and regenerate on changes
flutter pub run build_runner watch

# Watch with clean
flutter pub run build_runner watch --delete-conflicting-outputs
```

## Option 3: Hot Restart
```bash
# Generate then run app
flutter pub run build_runner build && flutter run
```

## Option 4: Web-Specific
```bash
# Generate for web platform
flutter pub run build_runner build --define "drift_dev:builder=web"
```

> **Key insights:**
> - **Watch mode** – Saves time during development
> - **Delete conflicting outputs** – Fixes generator conflicts
> - **Web builds** – May need specific configuration
> - **Build runs on all Dart files** – Scans for `@DriftDatabase`

---

# Managing Generated Files

> **What to do with the generated code**

## `.gitignore` Setup
```gitignore
# .gitignore
*.g.dart
*.freezed.dart
*.part.dart
*.mocks.dart

# Drift specific
**/database.g.dart
**/*.g.dart
```

## File Structure
```
lib/
├── database/
│   ├── tables/
│   │   ├── users.dart          # ✍️ You write
│   │   └── posts.dart          # ✍️ You write
│   ├── database.dart           # ✍️ You write
│   ├── database.g.dart         # 🤖 Generated (DON'T EDIT)
│   └── database.sql            # 📝 Optional: Raw SQL
├── models/
│   └── user.dart               # ✍️ Custom models (optional)
```

> **What's happening here?**
> - **Never edit** `*.g.dart` files
> - **Always commit** your source `.dart` files
> - **Never commit** generated files (add to `.gitignore`)
> - **Regenerate** on every clone or branch switch

---

# Customizing Generation

> **Advanced configuration options**

## Option 1: Custom Data Classes
```dart
// Instead of generated data class, use your own
@UseRowClass(Column.custom)
class User {
  final int id;
  final String name;
  final String email;
  final DateTime createdAt;
  
  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.createdAt,
  });
  
  // Custom mapping from SQL row
  static User fromRow(Row row) {
    return User(
      id: row.read('id'),
      name: row.read('name'),
      email: row.read('email'),
      createdAt: row.read('createdAt'),
    );
  }
}
```

## Option 2: SQL Queries in Files
```dart
// lib/database/queries.sql
-- Users with email
SELECT * FROM users WHERE email LIKE '%@gmail.com';

// In database.dart
@DriftDatabase(
  tables: [Users],
  queries: {
    'gmailUsers': 'SELECT * FROM users WHERE email LIKE ?',
  },
)
class AppDatabase extends _$AppDatabase {
  // Generated method: Future<List<User>> gmailUsers(String pattern)
}
```

## Option 3: DAO Generation
```dart
// lib/database/daos/user_dao.dart
@DriftAccessor(tables: [Users])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);
  
  @Query('SELECT * FROM users WHERE age > :age')
  Future<List<User>> getUsersOlderThan(int age);
  
  @Query('SELECT * FROM users WHERE age > :age')
  Stream<List<User>> watchUsersOlderThan(int age);
}

// Generated: _$UserDaoMixin with all query implementations
```

## Option 4: Custom Type Converters
```dart
// Define converter
class StringListConverter extends TypeConverter<List<String>, String> {
  const StringListConverter();
  
  @override
  List<String> fromSql(String fromDb) {
    return fromDb.split(',');
  }
  
  @override
  String toSql(List<String> value) {
    return value.join(',');
  }
}

// Use in table
class Users extends Table {
  TextColumn get tags => text().map(const StringListConverter())();
}

// Generated code handles conversion automatically
```

---

# Real-World Example

> **Complete project with code generation**

```dart
// lib/database/tables/users.dart
import 'package:drift/drift.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().withLength(min: 2, max: 50)();
  TextColumn get email => text().unique()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  BoolColumn get isActive => boolean().withDefault(const Constant(true))();
}

// lib/database/tables/posts.dart
import 'package:drift/drift.dart';
import 'users.dart';

class Posts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
  IntColumn get userId => integer().references(Users, #id)();
  DateTimeColumn get publishedAt => dateTime().nullable()();
}

// lib/database/daos/user_dao.dart
import 'package:drift/drift.dart';
import '../database.dart';

@DriftAccessor(tables: [Users, Posts])
class UserDao extends DatabaseAccessor<AppDatabase> with _$UserDaoMixin {
  UserDao(super.db);
  
  // Generated automatically!
  // Future<User> getUser(int id)
  // Stream<User> watchUser(int id)
  // Future<List<User>> getUsers()
  // Stream<List<User>> watchUsers()
  
  // Custom query with join
  @Query('''
    SELECT users.*, COUNT(posts.id) as postCount 
    FROM users 
    LEFT JOIN posts ON users.id = posts.user_id 
    GROUP BY users.id
  ''')
  Future<List<UserWithPostCount>> getUsersWithPostCount();
}

// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables/users.dart';
import 'tables/posts.dart';
import 'daos/user_dao.dart';

part 'database.g.dart';

@DriftDatabase(
  tables: [Users, Posts],
  daos: [UserDao],
)
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor]) : super(executor ?? _openConnection());

  @override
  int get schemaVersion => 1;
  
  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'my_app');
  }
  
  // DAO accessor
  UserDao get userDao => UserDao(this);
}

// lib/main.dart - Usage
void main() async {
  final db = AppDatabase();
  
  // ✅ All methods are type-safe and auto-generated!
  
  // Insert user
  final userId = await db.into(db.users).insert(
    UsersCompanion.insert(
      name: 'John Doe',
      email: 'john@example.com',
    ),
  );
  
  // Query users
  final users = await db.select(db.users).get();
  print('All users: $users');
  
  // Watch users (reactive!)
  db.select(db.users).watch().listen((users) {
    print('Users updated: ${users.length}');
  });
  
  // Use DAO
  final activeUsers = await db.userDao.watchUsers().first;
  
  // Custom query
  final usersWithPosts = await db.userDao.getUsersWithPostCount();
  print('Users with post counts: $usersWithPosts');
}
```

```dart
// Generated output (simplified)
// lib/database/database.g.dart

// 1. Data classes
class User {
  final int id;
  final String name;
  final String email;
  final DateTime createdAt;
  final bool isActive;
  
  User({...});
}

class Post {
  final int id;
  final String title;
  final String content;
  final int userId;
  final DateTime? publishedAt;
  
  Post({...});
}

class UserWithPostCount {
  final User user;
  final int postCount;
  
  UserWithPostCount({...});
}

// 2. Companion classes
class UsersCompanion extends UpdateCompanion<User> {
  final Value<int> id;
  final Value<String> name;
  final Value<String> email;
  final Value<DateTime> createdAt;
  final Value<bool> isActive;
  
  const UsersCompanion({
    this.id = const Value.absent(),
    this.name = const Value.absent(),
    this.email = const Value.absent(),
    this.createdAt = const Value.absent(),
    this.isActive = const Value.absent(),
  });
  
  UsersCompanion.insert({
    this.id = const Value.absent(),
    required String name,
    required String email,
    this.createdAt = const Value.absent(),
    this.isActive = const Value.absent(),
  }) : super.insert();
}

// 3. Database extensions
extension DatabaseExtensions on AppDatabase {
  $UsersTable get users => $UsersTable(this);
  $PostsTable get posts => $PostsTable(this);
}

// 4. DAO mixin
mixin _$UserDaoMixin on DatabaseAccessor<AppDatabase> {
  // Implementations of all query methods
  Future<User> getUser(int id) async {
    return await (select(users)..where((u) => u.id.equals(id))).getSingle();
  }
  
  Stream<User> watchUser(int id) {
    return (select(users)..where((u) => u.id.equals(id))).watchSingle();
  }
  
  // ... and more
}
```

> **Benefits of this approach:**
> - **Zero boilerplate** – All code is generated
> - **Type-safe everywhere** – Everything is strongly typed
> - **Reactive by default** – Streams work out of the box
> - **DAO pattern** – Organize queries logically
> - **Custom queries** – Mix generated and custom SQL
> - **Maintainable** – Change schema, regenerate, everything updates

---

# Best Practices

- **Always run `build_runner watch`** – Saves time during development
- **Add `*.g.dart` to `.gitignore`** – Don't commit generated files
- **Use `part` directive correctly** – Include `part 'database.g.dart'`
- **Regenerate after schema changes** – Always rebuild after table changes
- **Use DAOs for complex queries** – Better organization
- **Customize data classes** – Use `@UseRowClass` if needed
- **Write SQL in .sql files** – Better for complex queries
- **Check generation logs** – Look for errors or warnings
- **Use build_runner with `--delete-conflicting-outputs`** – Fixes conflicts
- **Keep generator version in sync** – Match `drift` and `drift_dev` versions

---

# Common Mistakes

## Mistake 1: Forgetting `part` directive

Wrong:
```dart
// 🚫 Error: _$AppDatabase not found
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  // ...
}
```

Correct:
```dart
// ✅ Include part directive
part 'database.g.dart';

@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  // ...
}
```

## Mistake 2: Not running build_runner

Wrong:
```bash
# Just run the app
flutter run  # 🚫 Compilation errors!
```

Correct:
```bash
# Generate first
flutter pub run build_runner build
# Then run
flutter run
```

## Mistake 3: Mismatched versions

Wrong:
```yaml
dependencies:
  drift: ^2.14.0
dev_dependencies:
  drift_dev: ^2.13.0  # 🚫 Different version!
```

Correct:
```yaml
dependencies:
  drift: ^2.14.0
dev_dependencies:
  drift_dev: ^2.14.0  # ✅ Same version!
```

## Mistake 4: Editing generated files

Wrong:
```dart
// database.g.dart
// ✍️ Manually adding code here
class User {
  // Custom fields added manually
}
// 🚫 Will be overwritten on next build!
```

Correct:
```dart
// database.dart
// ✍️ Add custom code in source files
class MyCustomUser extends User {
  // Custom code here - safe from overwrites
}
```

## Mistake 5: Forgetting to import generated code

Wrong:
```dart
// 🚫 Can't use User class because it's generated
void main() {
  final user = User(id: 1, name: 'John'); // Error
}
```

Correct:
```dart
import 'database.dart'; // ✅ Imports generated code too

void main() {
  final user = User(id: 1, name: 'John'); // Works!
}
```

---

# Troubleshooting

## Issue: `_$AppDatabase` not found
```bash
# Solution: Run build_runner
flutter pub run build_runner build
```

## Issue: Conflicting outputs
```bash
# Solution: Delete and rebuild
flutter pub run build_runner build --delete-conflicting-outputs
```

## Issue: Generation taking too long
```bash
# Solution: Use watch mode
flutter pub run build_runner watch
```

## Issue: Web generation errors
```bash
# Solution: Specify web builder
flutter pub run build_runner build --define "drift_dev:builder=web"
```

## Issue: Version conflicts
```yaml
# Solution: Pin versions in pubspec.yaml
dependencies:
  drift: 2.14.0
dev_dependencies:
  drift_dev: 2.14.0
```

---

# Summary

| Concept | Purpose | Best Practice |
|---------|---------|---------------|
| **Code Generation** | Create type-safe database code | Run `build_runner watch` |
| **Data Classes** | Typed objects for rows | Use generated classes |
| **Companions** | For inserts/updates | Use `Companion.insert()` |
| **Table Managers** | CRUD operations | Access via `db.tableName` |
| **DAO** | Organize queries | Use `@DriftAccessor` |
| **Generated Files** | `*.g.dart` | Never edit, commit to .gitignore |
| **Build Runner** | Code generation engine | Always run before build |

---

# Next Steps

Now you understand code generation, let's write your first query:

- [Your First Query](link) – Write and execute your first type-safe query
- [Tables Deep Dive](link) – Advanced table definitions
- [CRUD Operations](link) – Complete CRUD guide

---

# Did You Know?

- **Drift generates over 1000 lines** of code from a single table definition

- **The generated code is fully optimized** – no reflection, all code is static

- **`build_runner watch` can regenerate in under 100ms** for small projects

- **Drift supports incremental generation** – only regenerates changed files

- **You can write raw SQL** and Drift will generate type-safe methods for it

- **The generated `$Table` classes** contain over 50 methods each

- **Drift's code generator is written in Dart** – no external tools needed

- **You can customize the generated code** using `@UseRowClass` and other annotations

