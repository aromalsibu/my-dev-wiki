# build()

`build()` is the lifecycle method used inside Riverpod Notifiers to define and initialize the provider’s initial state.

---

## What is it?

`build()` is a method that runs when a provider is first created.

It is responsible for returning the **initial state** of a `Notifier`, `AsyncNotifier`, or `StreamNotifier`.

Depending on the notifier type, it can return:

- A synchronous value (`Notifier`)
- A `Future<T>` (`AsyncNotifier`)
- A `Stream<T>` (`StreamNotifier`)

It acts as the **entry point of the provider’s state lifecycle**.

---

## Why does it exist?

Before Riverpod introduced `build()`, initialization was handled using:

- Constructors (StateNotifier)
- Manual setup logic in widgets
- External initialization functions

These approaches made it harder to:

- Control lifecycle consistently
- Inject dependencies cleanly
- Centralize initialization logic
- Test state creation

`build()` solves this by:

- Providing a single lifecycle entry point
- Supporting dependency access via `ref`
- Making initialization declarative
- Standardizing all notifier types

---

## Syntax

### Notifier

```dart
class Counter extends Notifier<int> {
  @override
  int build() {
    return 0;
  }
}
```

Explanation:

- Returns the initial synchronous state.
- Runs once when provider is first used.

---

### AsyncNotifier

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchUser();
  }
}
```

Explanation:

- Returns a `Future`.
- Automatically handled as loading/data/error states.

---

### StreamNotifier

```dart
class CounterNotifier extends StreamNotifier<int> {
  @override
  Stream<int> build() async* {
    yield* Stream.periodic(
      const Duration(seconds: 1),
      (i) => i,
    );
  }
}
```

Explanation:

- Returns a stream of values.
- Emits continuously over time.

---

## Mental Model

Think of `build()` as the **starting point of a provider lifecycle**.

```text
Provider Created
       │
       ▼
    build()
       │
       ▼
Initial State Produced
       │
       ▼
UI Starts Listening
       │
       ▼
State Updates Over Time
```

It defines what the provider looks like at the moment it is created.

---

## Examples

### Simple Initialization

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;
}
```

Explanation:

- Initial state is `0`.

---

### Dependency Usage

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    final repo = ref.watch(userRepositoryProvider);
    return repo.fetchUser();
  }
}
```

Explanation:

- `ref.watch()` is available inside `build()`.
- Allows dependency injection.

---

### Stream Setup

```dart
class MessageNotifier extends StreamNotifier<Message> {
  @override
  Stream<Message> build() {
    final socket = ref.watch(socketProvider);
    return socket.messages();
  }
}
```

Explanation:

- Stream depends on external provider.
- Clean dependency integration.

---

## When `build()` Runs

`build()` runs:

- Once when provider is first created
- Again if dependencies change
- When provider is invalidated or refreshed

It is **not called on every state update**.

---

## When to Use

Use `build()` for:

- Initial state setup
- Fetching initial data
- Connecting dependencies
- Creating streams
- Preparing async resources

---

## When NOT to Use

Avoid using `build()` for:

- UI logic
- Event handlers
- State mutations after initialization
- Side effects unrelated to initialization

Those belong in notifier methods, not `build()`.

---

## Best Practices

- Keep `build()` focused on initialization only
- Avoid heavy side effects inside `build()`
- Use `ref.watch()` for reactive dependencies
- Prefer immutable return values
- Move business logic into methods, not `build()`

---

## Common Mistakes

### 1. Putting Business Logic in `build()`

❌ Wrong

```dart
Future<User> build() async {
  state = AsyncLoading(); // not allowed logic pattern
  return repository.fetchUser();
}
```

Why it's wrong:

- `build()` should only return initial state.
- Side effects should be in methods.

✔ Correct

```dart
Future<User> build() async {
  return repository.fetchUser();
}
```

---

### 2. Doing Heavy Work in `build()`

❌ Wrong

```dart
int build() {
  for (var i = 0; i < 1000000; i++) {
    // heavy computation
  }
  return 0;
}
```

Why it's wrong:

- Slows down provider initialization.
- Can block UI.

✔ Correct

Move heavy logic outside or cache results.

---

### 3. Ignoring Dependencies

❌ Wrong

```dart
Future<User> build() async {
  return repository.fetchUser();
}
```

Why it's wrong:

- No dependency tracking.
- Hard to test and override.

✔ Correct

```dart
Future<User> build() async {
  final repo = ref.watch(userRepositoryProvider);
  return repo.fetchUser();
}
```

---

## Related APIs

- Notifier
- AsyncNotifier
- StreamNotifier
- ref.watch()
- ref.read()
- Provider lifecycle

---

## Summary

`build()` is Riverpod’s initialization method for all Notifier types. It defines the initial state, sets up dependencies, and connects streams or async operations. It runs once per provider lifecycle and serves as the foundation for all state creation in Riverpod’s modern architecture.