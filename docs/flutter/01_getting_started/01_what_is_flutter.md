

# What is Flutter?

Understand Flutter - Google's UI toolkit for building natively compiled applications.

---

# What is it?

Flutter is an open-source UI software development kit (SDK) created by Google. It allows developers to build beautiful, natively compiled applications for mobile (Android, iOS), web, desktop (Windows, macOS, Linux), and embedded devices from a single codebase.

Unlike other cross-platform solutions, Flutter doesn't use WebViews or OEM widgets. Instead, it provides its own rendering engine (Skia) to paint pixels directly to the screen.

---

# Why does it exist?

- Cross-platform efficiency - Write once, run everywhere
- Performance - Compiled to native ARM code, no JavaScript bridge
- Hot Reload - See changes instantly without losing state
- Beautiful UIs - Customizable, pixel-perfect designs
- Developer productivity - Large widget catalog, great tooling
- Community & ecosystem - Growing package ecosystem

---

# How it works

The Flutter Architecture: Flutter sits between your Dart code and the native platform.

```dart
// A simple Flutter app
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Hello Flutter'),
        ),
        body: const Center(
          child: Text('Welcome!'),
        ),
      ),
    );
  }
}
```

What's happening here?
- `main()` is the entry point
- `runApp()` mounts the widget tree
- `MaterialApp` provides Material Design
- `Scaffold` gives app structure
- Everything is a Widget

---

# Core Principles

1. Widgets are everything - UI built from widgets
2. Composition over inheritance - Build complex UIs from simple widgets
3. Declarative UI - Describe what UI should look like, Flutter handles rendering
4. Single codebase - Same code for all platforms
5. Performance first - Compiled to native code

---

# Platform Support

| Platform | Status |
|----------|--------|
| Android | Stable ✅ |
| iOS | Stable ✅ |
| Web | Stable ✅ |
| Windows | Stable ✅ |
| macOS | Stable ✅ |
| Linux | Stable ✅ |
| Embedded | In Development 🔄 |

---

# Real-World Example

Building a simple counter app:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const CounterApp());

class CounterApp extends StatefulWidget {
  const CounterApp({super.key});

  @override
  State<CounterApp> createState() => _CounterAppState();
}

class _CounterAppState extends State<CounterApp> {
  int _count = 0;

  void _increment() {
    setState(() {
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Counter')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Text('You have pushed the button:'),
              Text(
                '$_count',
                style: Theme.of(context).textTheme.headlineMedium,
              ),
            ],
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: _increment,
          child: const Icon(Icons.add),
        ),
      ),
    );
  }
}
```

Benefits of this approach:
- State is encapsulated
- UI rebuilds automatically
- Clear separation of concerns
- Hot Reload friendly

---

# Best Practices

- Use `const` widgets where possible for performance
- Split large widgets into smaller ones
- Follow Flutter's naming conventions
- Use `flutter create` for new projects
- Keep `main.dart` clean, separate concerns

---

# Common Mistakes

## Using `new` keyword

Wrong:
```dart
Widget child = new Text('Hello');
```

Correct:
```dart
Widget child = const Text('Hello');
```

## Heavy computation in `build()`

Wrong:
```dart
@override
Widget build(BuildContext context) {
  final data = expensiveOperation();
  return Text(data);
}
```

Correct:
```dart
final data = expensiveOperation();

@override
Widget build(BuildContext context) {
  return Text(data);
}
```

---

# Summary

Flutter is a UI toolkit that lets you build cross-platform apps from a single codebase using widgets. It's fast, productive, and backed by Google with a growing ecosystem.

---

# Next Steps

- [Installation](installation.md)
- [Project Structure](project-structure.md)
- [Running Your First App](first-app.md)

---

# Did You Know?

- Flutter was originally called "Sky" (internal codename)
- It uses the Skia graphics engine (also used in Chrome and Android)
- Flutter apps run at 120fps on capable devices
- Over 500,000 apps have been built with Flutter
- The first stable version was released in December 2018