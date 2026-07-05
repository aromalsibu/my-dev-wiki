# StateNotifierProvider (Legacy)

> **Legacy Provider**
>
> `StateNotifierProvider` exists primarily for compatibility with applications built on Riverpod 1.x and 2.x. For new Riverpod applications, prefer `NotifierProvider` or `AsyncNotifierProvider`.

---

## What is it?

`StateNotifierProvider` exposes a `StateNotifier`, which manages immutable state and business logic.

Before the introduction of `Notifier`, `StateNotifier` was the recommended way to manage complex application state in Riverpod.

Although it remains fully supported, it has largely been replaced by the newer `Notifier` API.

---

## Why does it exist?

Early versions of Riverpod encouraged developers to write state like this:

```dart
class Counter extends StateNotifier<int> {
  Counter() : super(0);

  void increment() {
    state++;
  }
}
```

This approach was a significant improvement over `ChangeNotifier` because it promoted:

- Immutable state
- Predictable updates
- Better testing
- Separation of business logic

Later, Riverpod introduced `Notifier`, which offers the same benefits with a simpler API and built-in dependency access.

---

## Syntax

### Creating a StateNotifier

```dart
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);

  void increment() {
    state++;
  }

  void decrement() {
    state--;
  }
}
```

Explanation:

- `super(0)` sets the initial state.
- `state` stores the current value.
- Updating `state` rebuilds listeners automatically.

---

### Creating a StateNotifierProvider

```dart
final counterProvider =
    StateNotifierProvider<
      CounterNotifier,
      int
    >((ref) => CounterNotifier());
```

Explanation:

- Exposes the notifier.
- Consumers watch the state (`int`), not the notifier.

---

### Reading the State

```dart
final count = ref.watch(counterProvider);
```

Explanation:

- Returns the current state.
- Widgets rebuild when `state` changes.

---

### Calling Methods

```dart
ref.read(counterProvider.notifier).increment();
```

Explanation:

- `.notifier` returns the `StateNotifier`.
- Methods encapsulate state changes.

---

## Mental Model

`StateNotifierProvider` separates state from UI by placing business logic inside a notifier.

```text
            Widget
               │
               ▼
StateNotifierProvider
               │
               ▼
      StateNotifier
        ┌──────┴──────┐
        ▼             ▼
 Business Logic     State
               │
               ▼
        Widget Rebuild
```

This architecture is nearly identical to `NotifierProvider`, but with a different API.

---

## Examples

### Counter

```dart
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);

  void increment() {
    state++;
  }
}
```

Explanation:

- Encapsulates counter logic.
- Updates trigger automatic rebuilds.

---

### Shopping Cart

```dart
class CartNotifier
    extends StateNotifier<List<Product>> {
  CartNotifier() : super([]);

  void add(Product product) {
    state = [...state, product];
  }

  void remove(Product product) {
    state = state.where((p) => p != product).toList();
  }
}
```

Explanation:

- Uses immutable list updates.
- Business logic stays inside the notifier.

---

### Authentication

```dart
class AuthNotifier
    extends StateNotifier<AuthState> {
  AuthNotifier() : super(AuthState.loggedOut());

  void login(User user) {
    state = AuthState.loggedIn(user);
  }

  void logout() {
    state = AuthState.loggedOut();
  }
}
```

Explanation:

- Encapsulates authentication logic.
- UI reacts to state changes automatically.

---

## How It Differs from Notifier

### StateNotifier

```dart
class Counter extends StateNotifier<int> {
  Counter() : super(0);
}
```

The initial state is passed to the constructor.

---

### Notifier

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;
}
```

The initial state is provided by `build()`.

---

### Dependency Access

With `StateNotifier`, dependencies are usually passed through the constructor.

```dart
class UserNotifier extends StateNotifier<User> {
  UserNotifier(this.repository) : super(User.empty());

  final UserRepository repository;
}
```

---

With `Notifier`, dependencies are available directly.

```dart
class UserNotifier extends Notifier<User> {
  @override
  User build() {
    final repository = ref.watch(userRepositoryProvider);
    return repository.currentUser();
  }
}
```

No constructor injection is required.

---

## StateNotifierProvider vs NotifierProvider

| Feature | StateNotifierProvider | NotifierProvider |
|---------|------------------------|------------------|
| Recommended for new apps | ❌ | ✅ |
| Initial state | Constructor | `build()` |
| Access to `ref` | Constructor injection | Built-in |
| Less boilerplate | ❌ | ✅ |
| Business logic | ✅ | ✅ |
| Immutable state | ✅ | ✅ |

---

## When to Use

Use `StateNotifierProvider` only when:

- Maintaining an older Riverpod project
- Gradually migrating to `Notifier`
- Working with existing `StateNotifier` libraries
- Avoiding a full rewrite during migration

---

## When NOT to Use

Avoid it in new applications.

Instead, use:

| Scenario | Recommended Provider |
|----------|----------------------|
| Mutable synchronous state | `NotifierProvider` |
| Mutable asynchronous state | `AsyncNotifierProvider` |
| Read-only value | `Provider` |
| Simple UI state | `StateProvider` |

---

## Best Practices

- Prefer `NotifierProvider` for new development.
- Keep existing `StateNotifier` implementations stable until migration.
- Continue using immutable state updates.
- Move constructor dependencies into `build()` when migrating.
- Migrate incrementally rather than rewriting entire modules.

---

## Common Mistakes

### 1. Mutating Collections

❌ Wrong

```dart
state.add(product);
```

Why it's wrong:

- The list instance does not change.
- Consumers may not rebuild correctly.

✔ Correct

```dart
state = [...state, product];
```

Always replace collections with a new immutable instance.

---

### 2. Creating New StateNotifier Classes in New Projects

❌ Wrong

```dart
class Counter extends StateNotifier<int> {
  Counter() : super(0);
}
```

Why it's wrong:

- Riverpod's newer APIs are simpler.
- You'll write more boilerplate than necessary.

✔ Correct

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;
}
```

---

### 3. Passing Too Many Constructor Dependencies

❌ Wrong

```dart
CounterNotifier(
  api,
  repository,
  logger,
  analytics,
  cache,
);
```

Why it's wrong:

- Constructor injection becomes verbose.
- Managing dependencies becomes harder.

✔ Correct

Use `ref.watch()` inside a `Notifier` to access dependencies directly.

---

## Migration Example

### Before

```dart
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);

  void increment() {
    state++;
  }
}

final counterProvider =
    StateNotifierProvider<
      CounterNotifier,
      int
    >((ref) => CounterNotifier());
```

---

### After

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}

final counterProvider =
    NotifierProvider<Counter, int>(
      Counter.new,
    );
```

Explanation:

- `build()` replaces the constructor's initial state.
- Dependencies can be accessed using `ref`.
- The provider declaration is simpler.

---

## Recommendation

> **Use `StateNotifierProvider` only for maintaining or migrating existing codebases.**
>
> For new applications, `NotifierProvider` offers the same capabilities with less boilerplate, direct dependency access, and a more modern API.

---

## Related APIs

- NotifierProvider
- AsyncNotifierProvider
- ChangeNotifierProvider
- StateNotifier
- Notifier

---

## Summary

`StateNotifierProvider` is a legacy Riverpod provider that exposes a `StateNotifier` for managing immutable synchronous state and business logic. While it remains fully supported, it has been superseded by `NotifierProvider`, which provides a simpler API, built-in dependency access, and reduced boilerplate. Use `StateNotifierProvider` primarily for maintaining or migrating older Riverpod applications.