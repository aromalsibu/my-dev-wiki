

---

# Installation

A setup guide for adding Riverpod to a Flutter project.

---

## What is it?

Installation is the process of adding Riverpod dependencies to your Flutter project so you can start using providers, ref, and Riverpod’s state management system.

It includes:

* Adding required packages
* Setting up `ProviderScope`
* (Optional) enabling code generation

---

## Why does it exist?

Riverpod is not built into Flutter, so it must be explicitly added.

It exists to:

* Enable compile-safe state management in your project
* Register Riverpod as a dependency in your app
* Prepare the app for provider-based architecture

Without installation:

* Providers won’t work
* `ref` cannot be accessed
* State graph cannot be initialized

---

## Syntax

### 1. Add dependency

```yaml
dependencies:
  flutter_riverpod: ^2.5.0
```

Explanation:

* `flutter_riverpod` is the main package for Flutter integration
* Provides widgets like `ConsumerWidget` and `ProviderScope`

---

### 2. Wrap your app

```dart
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

Explanation:

* `ProviderScope` initializes Riverpod container
* It must wrap the entire app
* Acts as the root of the provider graph

---

### 3. (Optional) Code generation setup

```yaml
dependencies:
  riverpod_annotation: ^2.3.0

dev_dependencies:
  riverpod_generator: ^2.3.0
  build_runner: ^2.4.0
```

Explanation:

* `riverpod_annotation` enables annotations like `@riverpod`
* `riverpod_generator` auto-generates provider code
* `build_runner` runs the generator

---

## Mental Model

Think of installation as **turning on the Riverpod engine**:

```text
Flutter App
    │
    ▼
ProviderScope (Engine ON)
    │
    ▼
Providers become active
```

Without `ProviderScope`, Riverpod has no global state container.

---

## Examples

### Minimal setup

```dart
void main() {
  runApp(
    ProviderScope(
      child: MaterialApp(
        home: HomePage(),
      ),
    ),
  );
}
```

Explanation:

* `ProviderScope` wraps entire app
* `MaterialApp` runs inside Riverpod context

---

### With debugging support

```dart
void main() {
  runApp(
    ProviderScope(
      observers: [
        // ProviderObserver for logging
      ],
      child: MyApp(),
    ),
  );
}
```

Explanation:

* `observers` allow tracking provider changes
* Useful for debugging state updates

---

## When to Use

Always install Riverpod when:

* Starting a new Flutter project using Riverpod
* Migrating from Provider / Bloc / GetX
* Setting up scalable architecture

---

## When NOT to Use

Do NOT install Riverpod when:

* You are not using Flutter
* Your app is extremely simple (no shared state)
* You only need local UI state

---

## Best Practices

* Always wrap app with `ProviderScope` at root
* Install `flutter_riverpod`, not just `riverpod`
* Add code generation only if your project needs scalability
* Keep dependency versions aligned with Flutter SDK

---

## Common Mistakes

### 1. Forgetting ProviderScope

❌ Wrong:

```dart
void main() {
  runApp(MyApp());
}
```

Why it's wrong:

* Riverpod cannot initialize
* Providers will throw runtime errors

✔ Correct:

```dart
void main() {
  runApp(ProviderScope(child: MyApp()));
}
```

---

### 2. Using wrong package

❌ Wrong:

```yaml
dependencies:
  riverpod: ^2.0.0
```

Why it's wrong:

* Missing Flutter integration
* No widgets like `ConsumerWidget`

✔ Correct:

```yaml
dependencies:
  flutter_riverpod: ^2.5.0
```

---

## Related APIs

* ProviderScope
* flutter_riverpod
* riverpod_annotation
* riverpod_generator
* build_runner

---

## Summary

Installation sets up Riverpod in your Flutter project by adding dependencies and wrapping your app with `ProviderScope`, enabling the reactive provider system to function.
