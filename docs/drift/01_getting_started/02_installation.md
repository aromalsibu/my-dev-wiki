# Installation

Install Drift and its required packages to enable type-safe SQLite database development in Dart and Flutter.

---

# What is it?

Installing Drift involves adding the necessary packages to your project so you can define database tables, generate database code, and connect to SQLite.

Unlike many libraries, Drift relies on code generation, so you'll install both runtime and development dependencies.

The exact packages depend on your platform and how you want to store your database.

---

# Why does it exist?

Drift is divided into multiple packages to keep your application lightweight and flexible.

Each package has a specific responsibility:

* **drift** provides the core ORM and query APIs.
* **drift_flutter** (or another database executor) provides a platform-specific database connection.
* **drift_dev** generates the database code.
* **build_runner** runs the code generator.

Separating these responsibilities allows Drift to support multiple platforms and database backends.

---

# Syntax

## Step 1: Add Dependencies

Add Drift to your project.

```yaml
dependencies:
  drift: ^latest
  drift_flutter: ^latest

dev_dependencies:
  drift_dev: ^latest
  build_runner: ^latest
```

Explanation:

* `drift` contains the core APIs for defining tables and writing queries.
* `drift_flutter` provides a SQLite database connection optimized for Flutter.
* `drift_dev` generates database-related code.
* `build_runner` executes the code generation process.

---

## Step 2: Install Packages

Run:

```bash
flutter pub get
```

Explanation:

* Downloads and installs all project dependencies.

---

## Step 3: Generate Code

Run:

```bash
dart run build_runner build
```

Explanation:

* Scans your Drift annotations.
* Generates database classes, data classes, companions, and query helpers.

During development, you can automatically regenerate code whenever files change.

```bash
dart run build_runner watch
```

Explanation:

* Watches your project for changes.
* Regenerates code automatically after edits.

---

# Mental Model

Think of the installation as assembling a complete toolkit.

```
drift
   │
   ├── Database APIs
   ├── Query Builder
   ├── Tables
   └── Streams

drift_flutter
        │
        ▼
 SQLite Connection

drift_dev
        │
        ▼
 Code Generator

build_runner
        │
        ▼
 Generates *.g.dart files
```

Each package contributes a different piece of the Drift development experience.

---

# Examples

## Simple Example

After installing the packages, you're ready to create your first database.

```dart
@DriftDatabase(tables: [])
class AppDatabase extends _$AppDatabase {}
```

Explanation:

* `@DriftDatabase` marks the class for code generation.
* `_$AppDatabase` will be generated after running `build_runner`.

---

## Real-World Example

A typical Flutter project includes:

```
lib/
├── database/
│   ├── app_database.dart
│   ├── tables/
│   ├── daos/
│   └── app_database.g.dart
```

After installation, every time you add a table or DAO, running the generator updates the generated code automatically.

---

# When to Use

Install Drift when:

* Building a Flutter app with SQLite.
* You need relational data storage.
* You want compile-time type safety.
* Your application requires reactive database queries.
* You plan to use migrations and structured database access.

---

# When NOT to Use

Drift may not be necessary if:

* Your app only stores simple key-value data.
* You're using another database solution like a remote backend without local persistence.
* You don't need a relational database.

---

# Best Practices

* Use compatible package versions.
* Keep generated files under version control unless your team's workflow differs.
* Use `build_runner watch` during active development.
* Never edit generated files manually.
* Update dependencies regularly to stay compatible with the latest Drift features.

---

# Common Mistakes

## Forgetting the code generator

**Wrong**

Installing only:

```yaml
dependencies:
  drift: ^latest
```

Explanation:

* Drift's generated classes won't be created without `drift_dev` and `build_runner`.

**Correct**

Install both runtime and development dependencies.

---

## Forgetting to run code generation

**Wrong**

Creating annotated classes but never running:

```bash
dart run build_runner build
```

Explanation:

* Generated classes such as `_$AppDatabase` won't exist, causing compilation errors.

**Correct**

Run the code generator whenever you add or modify annotated Drift classes.

---

## Editing generated files

**Wrong**

Making changes inside:

```
app_database.g.dart
```

Explanation:

* Generated files are overwritten every time code generation runs.

**Correct**

Only edit your source files and regenerate the code.

---

# Related APIs

* Setting Up a Database
* Code Generation
* GeneratedDatabase
* Defining Tables
* Database Connection

---

# Summary

Installing Drift involves adding the core library, a database executor, and the code generation tools. Once installed and configured, Drift generates type-safe database APIs, allowing you to work with SQLite using clean, maintainable Dart code.
