# StateNotifier → Notifier

Migrate from `StateNotifier` to Riverpod's modern `Notifier` API for simpler, more integrated state management.

---

# What is it?

`Notifier` is the modern replacement for `StateNotifier` in Riverpod.

Both APIs are used to manage mutable application state, but `Notifier` is designed specifically for Riverpod and provides a cleaner, more consistent developer experience.

For new applications, **`Notifier` is the recommended choice**, while `StateNotifier` is considered a legacy API maintained for compatibility.

---

# Why does it exist?

`StateNotifier` originally came from the standalone **state_notifier** package before Riverpod introduced its own notifier system.

Although it works well, it has some limitations:

- State is initialized in the constructor.
- Access to `Ref` requires additional setup.
- Lifecycle is less integrated with Riverpod.
- More boilerplate is required.

`Notifier` solves these issues by:

- Initializing state with `build()`
- Providing `ref` automatically
- Integrating directly with Riverpod's lifecycle
- Reducing boilerplate
- Providing a consistent API with `AsyncNotifier` and `StreamNotifier`

---

# Syntax

## StateNotifier

```dart
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);

  void increment() {
    state++;
  }
}

final counterProvider =
    StateNotifierProvider<CounterNotifier, int>(
      (ref) => CounterNotifier(),
    );
```

Explanation:

- Initial state is provided through `super(0)`.
- `StateNotifierProvider` exposes the notifier.

---

## Notifier

```dart
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}

final counterProvider =
    NotifierProvider<CounterNotifier, int>(
      CounterNotifier.new,
    );
```

Explanation:

- `build()` replaces the constructor for state initialization.
- `NotifierProvider` is the modern provider type.
- `CounterNotifier.new` creates the notifier instance.

---

## Accessing dependencies

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

- `ref` is available directly inside `Notifier`.
- Dependencies can be watched during initialization.

---

# Mental Model

StateNotifier:

```text
Constructor
      │
      ▼
super(initialState)
      │
      ▼
Business Logic
```

Notifier:

```text
build()
    │
    ▼
Initial State
    │
    ▼
Business Logic
```

Think of `build()` as the entry point for creating a notifier's initial state.

---

# Examples

## Simple Example

### Before

```dart
class ThemeNotifier extends StateNotifier<ThemeMode> {
  ThemeNotifier() : super(ThemeMode.light);

  void toggle() {
    state = state == ThemeMode.light
        ? ThemeMode.dark
        : ThemeMode.light;
  }
}
```

Explanation:

- State starts in the constructor.

---

### After

```dart
class ThemeNotifier extends Notifier<ThemeMode> {
  @override
  ThemeMode build() => ThemeMode.light;

  void toggle() {
    state = state == ThemeMode.light
        ? ThemeMode.dark
        : ThemeMode.light;
  }
}
```

Explanation:

- Initial state is created in `build()`.
- Business logic remains unchanged.

---

## Real-World Example

```dart
class CartNotifier extends Notifier<Cart> {
  @override
  Cart build() {
    final repository = ref.watch(cartRepositoryProvider);

    return repository.loadCart();
  }

  void add(Product product) {
    state = state.add(product);
  }
}
```

Explanation:

- Dependencies are obtained directly from `ref`.
- State initialization and dependency resolution happen in one place.

---

# When to Use

Use `Notifier` when:

- Creating new Riverpod applications.
- Managing synchronous state.
- Replacing `StateNotifier`.
- Building business logic.
- Depending on other providers.

---

# When NOT to Use

Continue using `StateNotifier` only when:

- Maintaining legacy applications.
- Working with existing code that already uses it extensively.
- Migrating incrementally.

For new development, prefer `Notifier`.

---

# Best Practices

- Prefer `Notifier` for all new synchronous state.
- Move initialization logic into `build()`.
- Access dependencies through `ref`.
- Keep business logic inside the notifier.
- Keep state immutable whenever possible.

---

# Common Mistakes

## Initializing state in the constructor

Wrong:

```dart
class CounterNotifier extends Notifier<int> {
  CounterNotifier();
}
```

Explanation:

- `Notifier` does not use the constructor to initialize state.

Correct:

```dart
@override
int build() => 0;
```

Explanation:

- `build()` defines the initial state.

---

## Using `StateNotifierProvider` with `Notifier`

Wrong:

```dart
final counterProvider =
    StateNotifierProvider<CounterNotifier, int>(
      ...
    );
```

Explanation:

- `StateNotifierProvider` is only for `StateNotifier`.

Correct:

```dart
final counterProvider =
    NotifierProvider<CounterNotifier, int>(
      CounterNotifier.new,
    );
```

Explanation:

- Use the provider type that matches the notifier.

---

## Avoiding `ref`

Wrong:

```dart
final repository = UserRepository();
```

Explanation:

- Dependencies are created manually.

Correct:

```dart
final repository = ref.watch(
  userRepositoryProvider,
);
```

Explanation:

- Let Riverpod provide dependencies.

---

# Related APIs

- Notifier
- NotifierProvider
- AsyncNotifier
- AsyncNotifierProvider
- StateNotifier
- StateNotifierProvider
- `ref.watch()`

---

# Summary

`Notifier` is the modern replacement for `StateNotifier` in Riverpod. It initializes state with `build()`, provides direct access to `ref`, integrates cleanly with Riverpod's lifecycle, and reduces boilerplate. For new applications, `Notifier` should be the default choice for managing synchronous state.