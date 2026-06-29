# Notifier

`Notifier<T>` is Riverpod’s base class for managing synchronous state and encapsulating business logic that is exposed through a `NotifierProvider`.

---

## What is it?

`Notifier` is a class used in Riverpod to manage **mutable synchronous state** in a structured and testable way.

It holds a piece of state (`T`) and exposes methods that modify that state. Whenever the state changes, Riverpod automatically notifies all listeners and rebuilds the UI.

Unlike `StateProvider`, which is minimal and direct, `Notifier` is designed for **real application logic**, where state changes involve rules, validation, or multiple operations.

---

## Why does it exist?

As applications grow, state updates stop being simple assignments.

For example:

- Adding validation before updating state
- Combining multiple updates into one action
- Reusing logic across widgets
- Injecting dependencies
- Keeping UI free from business logic

Before `Notifier`, developers often used:

- `StateProvider` (too simple for complex logic)
- `StateNotifier` (older API with more boilerplate)
- Logic inside widgets (unmaintainable)

`Notifier` was introduced to provide:

- Clean separation of concerns
- Built-in access to `ref`
- Simple lifecycle (`build()`)
- Less boilerplate than `StateNotifier`
- Better scalability for app-level state

---

## Syntax

### Creating a Notifier

```dart
class Counter extends Notifier<int> {
  @override
  int build() {
    return 0;
  }

  void increment() {
    state++;
  }
}
```

Explanation:

- `Notifier<int>` defines the type of state managed.
- `build()` returns the initial state.
- `state` holds the current value.
- Updating `state` triggers UI updates automatically.

---

### Connecting with NotifierProvider

```dart
final counterProvider =
    NotifierProvider<Counter, int>(
  Counter.new,
);
```

Explanation:

- Connects the `Counter` class to Riverpod.
- Exposes `int` as the provider’s state.
- `Counter.new` is lazily instantiated by Riverpod.

---

### Reading State

```dart
final count = ref.watch(counterProvider);
```

Explanation:

- Returns the current state (`int`).
- Widget rebuilds when state changes.

---

### Calling Methods

```dart
ref.read(counterProvider.notifier).increment();
```

Explanation:

- `.notifier` gives access to the `Notifier` instance.
- Methods encapsulate all state-changing logic.

---

## Mental Model

Think of `Notifier` as a **state controller with rules**.

```text
        Widget
           │
           ▼
   NotifierProvider
           │
           ▼
        Notifier
     ┌─────┴─────┐
     ▼           ▼
  state     business logic
     │
     ▼
UI rebuilds
```

Instead of directly changing values, you call methods that enforce logic.

---

## Examples

### Counter

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}
```

Explanation:

- Simple state increment logic.
- UI automatically updates.

---

### Theme Toggle

```dart
class ThemeNotifier extends Notifier<bool> {
  @override
  bool build() => false;

  void toggle() {
    state = !state;
  }
}
```

Explanation:

- Encapsulates toggle logic inside the notifier.
- UI never directly modifies state.

---

### Cart Management

```dart
class Cart extends Notifier<List<String>> {
  @override
  List<String> build() => [];

  void addItem(String item) {
    state = [...state, item];
  }

  void removeItem(String item) {
    state = state.where((e) => e != item).toList();
  }
}
```

Explanation:

- Immutable updates ensure correct rebuilds.
- Business logic is centralized.

---

## When to Use

Use `Notifier` when:

- You need mutable synchronous state
- State has business rules or validation
- Multiple methods update the same state
- You want clean separation from UI
- You want scalable app architecture

---

## When NOT to Use

Avoid `Notifier` when:

- State is read-only → use `Provider`
- State is extremely simple → use `StateProvider`
- State is asynchronous → use `AsyncNotifier`
- State is streaming → use `StreamNotifier`

---

## Best Practices

- Keep state immutable (always replace objects)
- Expose meaningful methods instead of modifying `state` directly in UI
- Keep `build()` minimal and dependency-focused
- Avoid putting UI logic inside Notifier
- Split large notifiers into smaller ones when needed

---

## Common Mistakes

### 1. Mutating Collections Directly

❌ Wrong

```dart
state.add(item);
```

Why it's wrong:

- Riverpod cannot detect internal mutation.
- UI may not rebuild.

✔ Correct

```dart
state = [...state, item];
```

---

### 2. Putting Logic in Widgets

❌ Wrong

```dart
onPressed: () {
  if (count < 10) {
    ref.read(counterProvider.notifier).state++;
  }
}
```

Why it's wrong:

- Business rules belong in the Notifier.

✔ Correct

```dart
void increment() {
  if (state < 10) state++;
}
```

---

### 3. Overusing Notifier for Simple Values

❌ Wrong

```dart
class Flag extends Notifier<bool> {
  @override
  bool build() => false;
}
```

Why it's wrong:

- Too much boilerplate for a simple flag.

✔ Correct

```dart
final flagProvider = StateProvider((ref) => false);
```

---

## Related APIs

- NotifierProvider
- AsyncNotifier
- StreamNotifier
- state
- ref
- build()

---

## Summary

`Notifier` is Riverpod’s modern API for managing synchronous state with built-in business logic support. It replaces older patterns like `StateNotifier` and reduces boilerplate while providing a clean, scalable architecture for application state management.