

# Installation

**Add Drift to your Dart or Flutter project**

---

# What is it?

**Installation** is the process of adding Drift to your project and setting up all the necessary dependencies, code generators, and configuration files to start building type-safe databases.

> **Drift requires multiple packages** – the core library, SQLite drivers, and code generation tools. Getting this right is crucial for a smooth development experience.

```yaml
# pubspec.yaml - The complete Drift setup

dependencies:
  drift: ^2.14.0                    # Core Drift library
  sqlite3_flutter_libs: ^0.5.0      # SQLite for Flutter
  path_provider: ^2.1.0             # Find database directory
  path: ^1.9.0                      # Path manipulation

dev_dependencies:
  drift_dev: ^2.14.0                # Code generator
  build_runner: ^2.4.0              # Run code generation
```

> **What's happening here?**
> - **drift**: The main package with all the ORM features
> - **sqlite3_flutter_libs**: Bundles SQLite with your Flutter app
> - **drift_dev**: Development tool that generates Dart code from your table definitions
> - **build_runner**: The engine that runs the code generation

---

# Why does it exist?

- **Modular Architecture** – Different packages for different needs (core, drivers, generators)
- **Platform-Specific Drivers** – SQLite works differently on mobile, desktop, and web
- **Development vs Production** – Code generation tools are only needed during development
- **Version Management** – Keeping core and generator versions in sync prevents issues
- **Minimal Production Footprint** – dev_dependencies aren't bundled with your app

---

# Step 1: Add Dependencies

> **The complete pubspec.yaml setup for a Flutter project**

```yaml
name: my_app
description: A Drift-powered Flutter app

environment:
  sdk: ">=3.0.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  
  # Core Drift
  drift: ^2.14.0
  
  # SQLite driver for Flutter
  sqlite3_flutter_libs: ^0.5.0
  
  # File system helpers
  path_provider: ^2.1.0
  path: ^1.9.0
  
  # For web support (optional)
  # sqlite3: ^2.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  
  # Drift code generator
  drift_dev: ^2.14.0
  
  # Build system for code generation
  build_runner: ^2.4.0
  
  # For better linting (optional)
  flutter_lints: ^3.0.0
```

> **Key insights:**
> - **Version matching**: Keep `drift` and `drift_dev` versions in sync
> - **Flutter-specific**: Use `sqlite3_flutter_libs` for Flutter, not `sqlite3`
> - **Web support**: Add `sqlite3` and `sqlite3_wasm` for web builds
> - **Desktop support**: Most desktop platforms use the same Flutter libs

---

# Step 2: Platform-Specific Setup

> **Different platforms need different SQLite drivers**

## Flutter (Mobile & Desktop)
```dart
// No additional setup needed - sqlite3_flutter_libs handles it
import 'package:sqlite3_flutter_libs/sqlite3_flutter_libs.dart';

Future<Database> openDatabase() async {
  final db = await databaseFactoryIo.openDatabase(
    await getDatabasesPath() + '/app.db',
  );
  return db;
}
```

## Web
```dart
// Web requires wasm support
import 'package:sqlite3/sqlite3.dart';
import 'package:sqlite3/wasm.dart';

Future<Database> openDatabase() async {
  // Use sqlite3_wasm for web
  final db = await databaseFactoryWeb.openDatabase(
    '/app.db',
  );
  return db;
}
```

## Pure Dart (CLI)
```dart
// Use sqlite3 package directly
import 'package:sqlite3/sqlite3.dart';

Future<Database> openDatabase() async {
  final db = await databaseFactoryIo.openDatabase(
    'app.db',
  );
  return db;
}
```

> **What's happening here?**
> - **Flutter**: Uses the bundled native SQLite library
> - **Web**: Uses WebAssembly SQLite (sqlite3.wasm)
> - **CLI**: Uses system SQLite via dart:ffi
> - **Same API**: All platforms use the same Drift API

---

# Step 3: Configure Build Runner

> **Setting up the code generation system**

```dart
// Create a file: database.dart

import 'package:drift/drift.dart';

// 1. Define your tables
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

// 2. Define your database
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  
  @override
  int get schemaVersion => 1;
}

// 3. Database connection
LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final db = await databaseFactoryIo.openDatabase(
      await getDatabasesPath() + '/app.db',
    );
    return NativeDatabase(db);
  });
}
```

> **Key insights:**
> - **@DriftDatabase annotation** – Tells Drift which tables to generate
> - **_$AppDatabase** – The generated class that extends your database
> - **LazyDatabase** – Opens the connection only when first used
> - **NativeDatabase** – Wraps the native SQLite connection

---

# Step 4: Run Code Generation

> **Generate the Drift code**

```bash
# One-time build
flutter pub run build_runner build

# Watch mode (recommended)
flutter pub run build_runner watch

# Delete old generated files first
flutter pub run build_runner build --delete-conflicting-outputs

# For web-specific builds
flutter pub run build_runner build --define "drift_dev:builder=web"
```

> **What's happening here?**
> - **build_runner watch** – Automatically regenerates code when files change
> - **Generated files** – Look for `*.g.dart` next to your source files
> - **Delete conflicting outputs** – Fixes issues when generators conflict
> - **Web builds** – Requires different configuration for web platform

---

# Step 5: Verify Installation

> **Test that everything works**

```dart
// test_database.dart

import 'package:drift/drift.dart';
import 'database.dart';

void main() async {
  // 1. Initialize database
  final db = AppDatabase();
  
  // 2. Insert a test user
  await db.into(db.users).insert(
    UsersCompanion(
      name: Value('Test User'),
    ),
  );
  
  // 3. Query users
  final users = await db.select(db.users).get();
  print('Users: $users'); // Should print the user data
  
  // 4. Clean up
  await db.close();
}
```

> **Expected output:**
> ```
> Users: [User(id: 1, name: Test User)]
> ```

---

# Real-World Example

> **Complete project setup with folder structure**

```
my_app/
├── lib/
│   ├── database/
│   │   ├── database.dart          # Main database class
│   │   ├── tables/
│   │   │   ├── users.dart         # Users table definition
│   │   │   └── posts.dart         # Posts table definition
│   │   └── database.g.dart        # GENERATED - don't edit
│   ├── models/
│   │   ├── user.dart              # Custom business logic
│   │   └── post.dart
│   └── main.dart
├── test/
│   └── database_test.dart
├── pubspec.yaml
└── analysis_options.yaml          # Optional: Lint rules
```

```dart
// lib/database/tables/users.dart
import 'package:drift/drift.dart';

class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  DateTimeColumn get createdAt => dateTime()();
}

// lib/database/database.dart
import 'package:drift/drift.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;
import 'package:sqlite3_flutter_libs/sqlite3_flutter_libs.dart';

import 'tables/users.dart';

part 'database.g.dart';  // Important: Include generated part

@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;
}

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = p.join(dbFolder.path, 'app.db');
    
    // Copy bundled database if needed (optional)
    // await copyDatabase(file);
    
    final db = await databaseFactoryIo.openDatabase(file);
    return NativeDatabase(db);
  });
}
```

> **Benefits of this approach:**
> - **Clear separation** – Tables, database, and models in separate files
> - **Scalable** – Easy to add more tables and queries
> - **Testable** – Database can be swapped for an in-memory version
> - **Maintainable** – Each file has a single responsibility

---

# Best Practices

- **Keep drift and drift_dev versions in sync** – Mismatched versions cause cryptic errors
- **Use `watch` mode** – Saves time during development by regenerating automatically
- **Add `*.g.dart` to `.gitignore`** – Generated files shouldn't be committed
- **Use `part` directive** – Include generated code with `part 'database.g.dart'`
- **Don't manually edit generated files** – They're overwritten every build
- **Separate table definitions** – Put each table in its own file for better organization
- **Use `LazyDatabase`** – Delay database initialization until needed

---

# Common Mistakes

## Mistake 1: Mismatched versions

Wrong:
```yaml
dependencies:
  drift: ^2.14.0
dev_dependencies:
  drift_dev: ^2.13.0  # Different version!
```

Correct:
```yaml
dependencies:
  drift: ^2.14.0
dev_dependencies:
  drift_dev: ^2.14.0  # Same version!
```

## Mistake 2: Forgetting the `part` directive

Wrong:
```dart
// database.dart
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  // ...
}
// ERROR: _$AppDatabase not found
```

Correct:
```dart
// database.dart
part 'database.g.dart';  // Must include this!

@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  // ...
}
```

## Mistake 3: Not running code generation

Wrong:
```bash
flutter run  # Compilation errors!
```

Correct:
```bash
flutter pub run build_runner build  # First generate
flutter run  # Now it works
```

---

# Summary

| Step | Command/File | Purpose |
|------|-------------|---------|
| **1** | `pubspec.yaml` | Add dependencies |
| **2** | `database.dart` | Define tables and database |
| **3** | `build_runner watch` | Generate code automatically |
| **4** | `flutter run` | Run your app |
| **5** | `.gitignore` | Exclude generated files |

---

# Next Steps

Now that Drift is installed, let's set up your first database:

- [Setting Up a Database](link) – Create your first Drift database
- [Code Generation](link) – Deep dive into build_runner
- [Your First Query](link) – Write your first type-safe query

---

# Did You Know?

- **Drift supports both SQLite and PostgreSQL** – though SQLite is the default

- **The code generator can produce over 10,000 lines of code** – from just a few table definitions

- **Build runner watch mode uses hot-reload** – It regenerates code faster than you can switch to your IDE

- **Drift's generator is written in Dart** – Unlike many ORMs that use external languages

- **The `sqlite3_flutter_libs` package is ~2MB** – It bundles a full SQLite engine with your app

---

