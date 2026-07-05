
# What is Dart?

Dart is a client-optimized programming language developed by Google.

---

# What is it?

Dart is a modern, object-oriented language designed for building web, mobile, and desktop applications. It is the primary language used with the Flutter framework for cross-platform app development.

Dart features:

- **Strong typing** with sound null safety
- **Garbage collection** for automatic memory management
- **Ahead-of-time (AOT)** compilation for fast performance
- **Just-in-time (JIT)** compilation for fast development cycles
- **Isolates** for concurrency
- **Async/await** for asynchronous programming

---

# Why does it exist?

Dart was created to address the challenges of building modern applications:

**Performance** - AOT compilation delivers native performance on mobile and desktop

**Developer experience** - JIT compilation with hot reload enables rapid development

**Productivity** - Clear syntax, powerful features, and excellent tooling

**Cross-platform** - Write once, run on iOS, Android, web, macOS, Windows, and Linux

**Interoperability** - Works with JavaScript, C, and other languages

---

# Installing Dart

## Windows

```powershell
# Using Chocolatey
choco install dart-sdk

# Or download from: https://dart.dev/get-dart
```

---

## macOS

```bash
# Using Homebrew
brew tap dart-lang/dart
brew install dart

# Or download from: https://dart.dev/get-dart
```

---

## Linux

```bash
# Using apt (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install dart

# Using snap (any distribution)
sudo snap install dart --classic

# Or download from: https://dart.dev/get-dart
```

---

## Verify Installation

```bash
# Check version
dart --version

# Run a simple program
dart run
```

---

# Running Dart Programs

## Create a file

```dart
// main.dart
void main() {
  print('Hello, World!');
}
```

---

## Run the program

```bash
dart run main.dart
```

Output:

```text
Hello, World!
```

---

## Different ways to run

```bash
# Run a file directly
dart run main.dart

# Run with package resolution
dart run bin/my_app.dart

# Run a specific file in a package
dart run my_package:bin/my_script.dart
```

---

# Project Structure

## Default project structure

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

## pubspec.yaml example

```yaml
name: my_app
description: A sample Dart application
version: 1.0.0

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  http: ^1.0.0

dev_dependencies:
  test: ^1.24.0
```

---

## Entry point

```dart
// bin/main.dart or lib/main.dart
void main() {
  // Your application starts here
  print('Application started');
}
```

---

# Language Tour

## Hello World

```dart
void main() {
  print('Hello, World!');
}
```

---

## Variables

```dart
// Explicit typing
String name = 'John';

// Type inference
var age = 25;

// Constant values
const pi = 3.14159;

// Final (runtime constant)
final currentTime = DateTime.now();
```

---

## Functions

```dart
int add(int a, int b) {
  return a + b;
}

// Arrow syntax for single expressions
int multiply(int a, int b) => a * b;

// Optional parameters
void greet(String name, {String? title}) {
  print('Hello, $title $name');
}
```

---

## Classes

```dart
class Person {
  String name;
  int age;
  
  Person(this.name, this.age);
  
  void sayHello() {
    print('Hello, I am $name');
  }
}
```

---

## Collections

```dart
// Lists (arrays)
var names = ['Alice', 'Bob', 'Charlie'];

// Sets (unique values)
var uniqueNumbers = {1, 2, 3};

// Maps (key-value pairs)
var person = {
  'name': 'John',
  'age': 30,
};
```

---

## Control Flow

```dart
// If-else
if (score > 90) {
  print('Excellent');
} else if (score > 70) {
  print('Good');
} else {
  print('Needs improvement');
}

// For loop
for (var i = 0; i < 5; i++) {
  print(i);
}

// While loop
while (condition) {
  // Do something
}

// Switch
switch (color) {
  case 'red':
    print('Stop');
    break;
  case 'green':
    print('Go');
    break;
  default:
    print('Unknown');
}
```

---

## Async/Await

```dart
Future<String> fetchData() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Data loaded';
}

void main() async {
  String result = await fetchData();
  print(result);
}
```

---

## Exception Handling

```dart
try {
  int result = 10 ~/ 0;
} on IntegerDivisionByZeroException {
  print('Cannot divide by zero');
} catch (e) {
  print('Error: $e');
} finally {
  print('Cleanup');
}
```

---

# Best Practices

- Use `var` or `final` when type is obvious
- Prefer `final` over `var` for variables that don't change
- Use `const` for compile-time constants
- Follow effective Dart guidelines
- Use `??` and `?.` operators for null safety
- Keep functions small and focused
- Use proper naming conventions

---

# Summary

Dart is a modern, versatile programming language that powers Flutter applications. It combines productivity, performance, and platform flexibility. With its strong type system, async support, and comprehensive standard library, Dart is an excellent choice for building applications across all platforms.

---

# Next Steps

- Explore the [fundamentals](fundamentals.md)
- Learn about [OOP](oop.md)
- Understand [asynchronous programming](async.md)
- Check out [testing](testing.md)
- Study [best practices](best-practices.md)

---

# Did You Know?

- Dart was first announced in 2011
- Flutter was first released in 2015
- Dart supports both JIT and AOT compilation
- Dart is used by major companies like Google, Alibaba, and eBay
- The language is continuously evolving with new features