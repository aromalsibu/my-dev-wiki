# Project Structure

Understand the standard layout and organization of Dart projects.

---

# What is it?

Project structure refers to the standard directory layout and file organization conventions used in Dart applications and packages. Following these conventions ensures consistency across projects and enables tools like `pub` to work correctly.

---

# Why does it exist?

A standardized project structure:

- Makes projects easier to navigate and understand
- Enables the package manager (`pub`) to find and manage files
- Separates concerns (public API vs implementation)
- Makes it easier to share and reuse code
- Provides a clear location for tests and documentation
- Follows community best practices

---

# Basic Project Structure

## Default layout

```text
my_project/
├── lib/
│   ├── src/
│   │   └── (private implementation)
│   └── my_project.dart
├── bin/
│   └── main.dart
├── test/
│   └── my_project_test.dart
├── pubspec.yaml
├── pubspec.lock
├── README.md
├── CHANGELOG.md
└── analysis_options.yaml
```

---

# Core Directories

## lib/

The `lib/` directory contains the main source code of your package or application.

```text
lib/
├── src/                # Private implementation
│   ├── models/
│   ├── services/
│   └── utils/
├── models.dart         # Public API exports
├── services.dart       # Public API exports
├── my_project.dart     # Main entry point
└── main.dart           # Application entry
```

---

### lib/src/

Private implementation that should not be imported directly by users.

```text
lib/src/
├── core/
├── models/
├── services/
└── utils/
```

Rules:

- Files in `src/` are considered internal
- Users should import only from `lib/` root
- Use `export` to expose public APIs

---

### lib/main.dart

The application entry point (for applications, not packages).

```dart
// lib/main.dart
import 'package:my_app/src/app.dart';

void main() {
  // Application bootstrap
  runApp(MyApp());
}
```

---

## bin/

The `bin/` directory contains executable scripts.

```text
bin/
├── main.dart           # Main executable
├── my_cli.dart         # CLI tool entry
└── scripts/
    └── helper.dart
```

---

### bin/main.dart

```dart
// bin/main.dart
import 'package:my_app/my_app.dart';

void main(List<String> args) {
  print('Hello from my_app!');
  // Parse args and run the CLI
}
```

---

## test/

The `test/` directory contains tests for your package.

```text
test/
├── unit/
│   ├── models_test.dart
│   └── services_test.dart
├── integration/
│   └── api_test.dart
├── helpers/
│   └── test_utils.dart
└── my_project_test.dart
```

---

### Example test file

```dart
// test/unit/models_test.dart
import 'package:test/test.dart';
import 'package:my_app/models.dart';

void main() {
  group('User', () {
    test('should create user with name', () {
      final user = User(name: 'Alice');
      expect(user.name, equals('Alice'));
    });
  });
}
```

---

## web/

For web applications with `build_runner`:

```text
web/
├── index.html
├── main.dart
├── styles.css
└── assets/
    ├── images/
    └── fonts/
```

---

## example/

For packages that provide examples:

```text
example/
├── lib/
│   └── example.dart
├── test/
│   └── example_test.dart
├── pubspec.yaml
└── README.md
```

---

# Core Files

## pubspec.yaml

The package manifest file.

```yaml
name: my_app
description: A Dart application
version: 1.0.0
repository: https://github.com/username/my_app

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  http: ^1.0.0
  path: ^1.8.0

dev_dependencies:
  test: ^1.24.0
  build_runner: ^2.4.0

flutter:
  uses-material-design: true
```

---

## pubspec.lock

Automatically generated lock file.

```yaml
# This file is generated automatically
# Do not edit manually
packages:
  http:
    dependency: "direct main"
    description:
      name: http
      sha256: "123456..."
    source: hosted
    version: "1.0.0"
```

---

## analysis_options.yaml

Dart analyzer configuration.

```yaml
include: package:lints/recommended.yaml

analyzer:
  strong-mode:
    implicit-casts: false
    implicit-dynamic: false
  exclude:
    - build/**
    - lib/**/*.g.dart

linter:
  rules:
    - avoid_print
    - prefer_const_constructors
    - prefer_final_fields
    - require_trailing_commas
```

---

## README.md

Project documentation.

```markdown
# My App

A Dart application that does something great.

## Getting Started

1. Install dependencies
2. Run the app

## Usage

```bash
dart run bin/main.dart
```

## License

MIT
```

---

## CHANGELOG.md

Version history.

```markdown
# Changelog

## 1.0.0
- Initial release
- Basic functionality

## 0.1.0
- First beta version
- Core features
```

---

## .gitignore

Version control exclusions.

```text
# Dart
.dart_tool/
build/
pubspec.lock
pubspec.lock.json

# IDE
.idea/
.vscode/
*.iml

# OS
.DS_Store
Thumbs.db

# Logs
*.log
```

---

# Advanced Structure

## Large application

```text
my_app/
├── lib/
│   ├── core/
│   │   ├── di/           # Dependency injection
│   │   ├── config/       # Configuration
│   │   └── constants/    # Constants
│   ├── features/
│   │   ├── auth/
│   │   │   ├── models/
│   │   │   ├── services/
│   │   │   ├── widgets/
│   │   │   └── auth.dart
│   │   ├── home/
│   │   │   ├── models/
│   │   │   ├── services/
│   │   │   ├── widgets/
│   │   │   └── home.dart
│   │   └── settings/
│   │       └── ...
│   ├── shared/
│   │   ├── widgets/
│   │   ├── utils/
│   │   └── extensions/
│   ├── app.dart
│   └── main.dart
├── assets/
│   ├── images/
│   ├── fonts/
│   └── translations/
├── scripts/
│   ├── build.dart
│   └── deploy.sh
├── tools/
│   └── gen_code.dart
├── docs/
│   └── api/
├── pubspec.yaml
└── analysis_options.yaml
```

---

# Package Types

## Application package

For runnable applications:

```text
my_app/
├── lib/
│   └── src/
├── bin/
│   └── main.dart
├── pubspec.yaml
└── ...
```

---

## Library package

For reusable libraries:

```text
my_package/
├── lib/
│   ├── my_package.dart
│   └── src/
├── pubspec.yaml
└── ...
```

---

## Flutter package

For Flutter plugins or packages:

```text
my_flutter_package/
├── lib/
├── test/
├── example/              # Flutter example app
├── pubspec.yaml
└── README.md
```

---

# Best Practices

- Keep `lib/` organized by feature or layer
- Expose only public APIs from `lib/` root
- Keep `src/` for private implementation
- Always include a `pubspec.yaml`
- Document your package with README.md
- Write tests in `test/`
- Use `analysis_options.yaml` for code quality
- Follow the [Effective Dart](https://dart.dev/guides/language/effective-dart) style guide

---

# Common Mistakes

## Importing from lib/src

Wrong:

```dart
import 'package:my_app/src/models/user.dart';
```

Correct:

```dart
import 'package:my_app/models.dart';
```

---

## Missing pubspec dependencies

Wrong:

```yaml
dependencies:
  # Forgetting to include a dependency
```

Correct:

```yaml
dependencies:
  http: ^1.0.0
```

---

## Not ignoring build output

Wrong:

```text
# Not in .gitignore
build/
```

Correct:

```text
# In .gitignore
build/
.dart_tool/
```

---

# Summary

Following the standard Dart project structure makes your code easier to maintain, share, and understand. Whether you're building a library, application, or Flutter app, these conventions help you and your team work more effectively.

---

# Next Steps

Now that you understand project structure, continue to:

- [Language Tour](language-tour.md)
- [Variables](variables.md)
- [Null Safety](null-safety.md)

---

# Did You Know?

- The `pub` tool was originally called "pub" because it's short for "package"
- The `lib/` and `bin/` separation is inspired by Ruby's `lib/` and `bin/` conventions
- Dart projects can include multiple entry points in `bin/`
- The `test/` directory can have any structure, but the convention is to mirror `lib/`
- You can use `dart pub get` to download dependencies listed in `pubspec.yaml`