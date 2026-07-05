# Running Dart Programs

Execute Dart code through various methods and understand the execution process.

---

# What is it?

Running Dart programs refers to the process of executing Dart code using the Dart VM or compiled binaries. Dart can be run in multiple ways, including:

- Direct execution with the Dart VM (JIT compilation)
- Compiled to native code (AOT compilation)
- Compiled to JavaScript for web deployment
- Using the Dart REPL for experimentation

---

# Why does it exist?

Running Dart programs enables you to:

- Test and debug your code during development
- Deploy production applications
- Execute command-line tools and scripts
- Build and run web applications
- Create native mobile and desktop applications with Flutter

---

# Basic Execution

## Running a single file

```dart
// hello.dart
void main() {
  print('Hello, World!');
}
```

Run with:

```bash
dart run hello.dart
```

Output:

```text
Hello, World!
```

---

## Running a file without extension

```bash
dart run hello
```

---

## Running with arguments

```dart
// greet.dart
void main(List<String> args) {
  if (args.isEmpty) {
    print('Please provide a name');
  } else {
    print('Hello, ${args[0]}!');
  }
}
```

```bash
dart run greet.dart Alice
```

Output:

```text
Hello, Alice!
```

---

# Running Projects

## Running a project with pubspec.yaml

```bash
# From the project root
dart run

# Run a specific entry point
dart run bin/my_app.dart

# Run a specific package executable
dart run my_package:bin/my_script.dart
```

---

## Example project structure

```text
my_app/
├── pubspec.yaml
├── bin/
│   └── main.dart
├── lib/
│   └── src/
│       └── my_lib.dart
└── test/
    └── my_test.dart
```

Running:

```bash
cd my_app
dart run bin/main.dart
```

---

# Running with Pub

## List available executables

```bash
dart pub global list
```

---

## Run global packages

```bash
# Activate a package
dart pub global activate my_cli_tool

# Run the executable
dart pub global run my_cli_tool
```

---

## Run local packages

```bash
# In the package directory
dart run

# Or specify the entry point
dart run bin/my_tool.dart
```

---

# Development vs Production

## Development (JIT)

```bash
# Uses the Dart VM with JIT compilation
# Faster startup for development
dart run main.dart
```

---

## Production (AOT)

```bash
# Compile to native code
dart compile exe main.dart -o my_app

# Run the compiled binary
./my_app
```

---

# Different Execution Methods

## Dart VM

```bash
# Run directly with the VM
dart main.dart
```

---

## Compiled executable

```bash
# Compile to AOT snapshot
dart compile aot-snapshot main.dart

# Run the snapshot
dartaotruntime main.aot
```

---

## JavaScript compilation

```bash
# Compile to JavaScript
dart compile js main.dart -o main.js

# Run in browser or Node.js
node main.js
```

---

## Kernel snapshot

```bash
# Create a kernel snapshot
dart compile kernel main.dart -o main.dill

# Run the snapshot
dart run main.dill
```

---

# REPL Mode

## Start the REPL

```bash
dart
```

Or:

```bash
dart -i
```

---

## Using the REPL

```dart
>>> var name = 'Dart';
>>> print('Hello, $name');
Hello, Dart
>>> void greet(String name) => print('Hi, $name');
>>> greet('World');
Hi, World
>>> ^D
```

---

# Debugging

## Run with debugging

```bash
# Enable debugging
dart run --enable-vm-service main.dart

# With observatory
dart run --observe main.dart

# Wait for debugger
dart run --pause-isolates-on-start main.dart
```

---

## Debugging with VS Code

1. Open the project in VS Code
2. Set breakpoints in your code
3. Press F5 or click "Run and Debug"
4. Use the debug console to inspect variables

---

## Debugging with Dart DevTools

```bash
# Run with observatory
dart run --observe main.dart

# Open the URL shown in the console
# Usually: http://localhost:8181/
```

---

# Hot Reload

## With Flutter development

```bash
# Flutter apps support hot reload
flutter run

# Press 'r' for hot reload
# Press 'R' for hot restart
```

---

# Command Line Options

## Common flags

```bash
# Show help
dart run --help

# Enable verbose output
dart run --verbose main.dart

# Set custom working directory
dart run --directory=./src main.dart

# Enable checked mode
dart run --checked main.dart

# Define a global variable
dart run --define=MY_VAR=value main.dart
```

---

## Performance flags

```bash
# Enable Dart VM flags
dart run --enable-asserts main.dart

# Disable type checking (for performance)
dart run --no-verify main.dart

# Set memory limit
dart run --memory-limit=256MB main.dart
```

---

# Environment Variables

## Setting environment variables

```bash
# Unix-like systems
DART_ENV=production dart run main.dart

# Windows
set DART_ENV=production && dart run main.dart
```

---

## Reading environment variables

```dart
void main() {
  final env = String.fromEnvironment('DART_ENV', defaultValue: 'development');
  print('Running in $env mode');
}
```

---

# Common Use Cases

## Script execution

```bash
# Run a simple script
dart run my_script.dart

# Pass arguments
dart run my_script.dart --input=data.json --output=result.json
```

---

## Server applications

```bash
# Run a Dart HTTP server
dart run bin/server.dart

# Run with environment variables
PORT=8080 dart run bin/server.dart
```

---

## CLI tools

```bash
# Run a command-line tool
dart run bin/my_cli.dart --help

# Install globally
dart pub global activate my_cli
my_cli command --flag
```

---

# Best Practices

- Use `dart run` instead of `dart` for package-aware execution
- Use `dart compile exe` for production deployments
- Add proper error handling for command-line arguments
- Use `exitCode` to indicate success or failure
- Include a `bin/` directory for executable scripts

---

# Summary

Running Dart programs is flexible and supports multiple execution modes. Choose the appropriate method based on your use case:

- `dart run` for development and scripts
- `dart compile exe` for production applications
- `dart compile js` for web deployment
- `dart` REPL for experimentation

---

# Next Steps

Now that you can run Dart programs, continue to:

- [Project Structure](project-structure.md)
- [Language Tour](language-tour.md)
- [Variables](variables.md)

---

# Did You Know?

- Dart's JIT compiler is faster than many other JIT-compiled languages
- AOT-compiled Dart apps can start in milliseconds
- The Dart VM can run multiple isolates concurrently
- Dart code can be compiled to WebAssembly (Wasm)
- The VM includes a built-in profiler for performance analysis