# NotifierProvider

`NotifierProvider` is a provider that exposes a `Notifier`, allowing you to manage synchronous state and encapsulate business logic in a single place.

---

## What is it?

`NotifierProvider` is the recommended provider for managing **mutable synchronous state** in modern Riverpod.

Unlike `StateProvider`, which is designed for simple values, `NotifierProvider` lets you:

- Store state
- Update state
- Encapsulate business logic
- Expose methods
- React to dependencies

A `NotifierProvider` consists of two parts:

1. A **Notifier**, which contains the state and business logic.
2. A **NotifierProvider**, which exposes that notifier to the rest of the application.

---

## Why does it exist?

As applications grow, state updates become more complex.

For example, a counter might eventually need:

- Validation
- Logging
- Analytics
- Multiple update methods
- Dependency injection

Placing this logic inside widgets quickly becomes difficult to maintain.

`NotifierProvider` solves this by moving state and business logic into a dedicated class.

Benefits include:

- Better code organization
- Testable business logic
- Immutable state updates
- Separation of concerns
- Automatic UI updates

It replaces most use cases for `StateProvider` and the legacy `StateNotifierProvider`.

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

  void decrement() {
    state--;
  }
}
```

Explanation:

- `build()` returns the initial state.
- `state` holds the current value.
- Updating `state` notifies all listeners.

---

### Creating a NotifierProvider

```dart
final counterProvider =
    NotifierProvider<Counter, int>(
      Counter.new,
    );
```

Explanation:

- `NotifierProvider<Notifier, State>` exposes the notifier.
- `Counter.new` creates the notifier lazily.
- Consumers watch the state, not the notifier itself.

---

### Reading the State

```dart
final count = ref.watch(counterProvider);
```

Explanation:

- Returns the current state.
- Rebuilds whenever `state` changes.

---

### Calling Methods

```dart
ref.read(counterProvider.notifier).increment();
```

Explanation:

- `.notifier` returns the `Counter` instance.
- Methods update the state internally.

---

## Mental Model

Think of a `Notifier` as a controller.

```text
          Widget
             │
             ▼
     NotifierProvider
             │
             ▼
         Notifier
      ┌──────┴──────┐
      ▼             ▼
   Business       State
    Logic
```

Widgets interact with the provider.

The notifier manages all state changes.

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

- Encapsulates counter logic.
- Widgets only call `increment()`.

---

### Toggle Theme

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

- Manages theme state.
- Exposes a clear API through `toggle()`.

---

### Shopping Cart

```dart
class CartNotifier extends Notifier<List<Product>> {
  @override
  List<Product> build() => [];

  void add(Product product) {
    state = [...state, product];
  }

  void remove(Product product) {
    state = state.where((p) => p != product).toList();
  }
}
```

Explanation:

- Updates state immutably.
- Business logic stays inside the notifier.

---

### Using Dependencies

```dart
class UserNotifier extends Notifier<User> {
  @override
  User build() {
    final repository = ref.watch(userRepositoryProvider);

    return repository.currentUser();
  }
}
```

Explanation:

- `ref` is available inside every notifier.
- Dependencies are accessed with `ref.watch()`.

---

## State Flow

```text
Button Press
      │
      ▼
increment()
      │
      ▼
state++
      │
      ▼
NotifierProvider
      │
      ▼
Widget Rebuild
```

Only widgets watching the provider rebuild.

---

## NotifierProvider vs StateProvider

| Feature | StateProvider | NotifierProvider |
|---------|---------------|------------------|
| Mutable state | ✅ | ✅ |
| Business logic | ❌ | ✅ |
| Multiple methods | ❌ | ✅ |
| Dependency injection | ❌ | ✅ |
| Complex state | ❌ | ✅ |
| Recommended for app state | Limited | ✅ |

---

## When to Use

Use `NotifierProvider` for:

- Counters with logic
- Shopping carts
- Authentication state
- Form state
- Filters
- User preferences
- Domain models
- Business logic
- Any mutable synchronous state

It is the recommended choice for most synchronous state in Riverpod.

---

## When NOT to Use

Avoid `NotifierProvider` when:

- The value is read-only.
- The operation is asynchronous.
- The state comes from a stream.
- Only a simple boolean or integer is needed without business logic.

Instead, use:

| Scenario | Recommended Provider |
|----------|----------------------|
| Read-only value | `Provider` |
| Simple mutable value | `StateProvider` |
| Future | `FutureProvider` |
| Async state | `AsyncNotifierProvider` |
| Stream | `StreamProvider` |

---

## Best Practices

- Keep business logic inside the notifier.
- Keep widgets free of state manipulation.
- Update immutable state instead of mutating objects.
- Expose meaningful methods rather than modifying `state` externally.
- Keep `build()` focused on initialization.

---

## Common Mistakes

### 1. Updating State from Widgets

❌ Wrong

```dart
ref.read(counterProvider.notifier).state++;
```

Why it's wrong:

- Widgets should not manipulate internal state directly.
- It bypasses your business logic.

✔ Correct

```dart
ref.read(counterProvider.notifier).increment();
```

Expose methods that control state changes.

---

### 2. Mutating Collections

❌ Wrong

```dart
state.add(product);
```

Why it's wrong:

- The list instance does not change.
- Riverpod may not detect the update.

✔ Correct

```dart
state = [...state, product];
```

Replace the collection with a new immutable instance.

---

### 3. Putting Business Logic in Widgets

❌ Wrong

```dart
onPressed: () {
  if (count < 10) {
    ref.read(counterProvider.notifier).state++;
  }
}
```

Why it's wrong:

- Business rules belong inside the notifier.
- Logic becomes scattered across the UI.

✔ Correct

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    if (state < 10) {
      state++;
    }
  }
}
```

Widgets simply call `increment()`.

---

## Recommendation

> **Use `NotifierProvider` for almost all synchronous application state.**
>
> If your state requires business logic, validation, multiple operations, or dependency injection, `NotifierProvider` is the preferred choice over `StateProvider`.

---

## Related APIs

- Notifier
- AsyncNotifierProvider
- StreamNotifierProvider
- StateProvider
- Provider
- ref.watch()
- ref.read()
- state

---

## Summary

`NotifierProvider` is Riverpod's recommended provider for synchronous mutable state. It combines state management and business logic inside a `Notifier`, enabling clean architecture, immutable updates, dependency injection, and automatic UI rebuilding. For most non-asynchronous application state, it is the preferred provider type in modern Riverpod.