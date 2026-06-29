# Your First Provider

A provider is the fundamental building block of Riverpod. It exposes a piece of state, a value, or business logic that can be shared and consumed throughout your application.

---

## What is it?

A provider is an object that tells Riverpod **how to create a value**.

Instead of manually creating objects and passing them through constructors or widget trees, you define a provider once and let Riverpod manage its lifecycle.

Providers can expose:

- Simple values
- Services
- Repositories
- Configuration
- Computed values
- Application state
- Asynchronous data

A provider does **not** store state by itself (except stateful provider types like `NotifierProvider` or `StateProvider`). It simply defines how a value should be created.

---

## Why does it exist?

Without providers, shared objects often need to be:

- Passed through multiple widget constructors (prop drilling)
- Created as global variables
- Managed manually using dependency injection

For example:

```text
App
 ├── Home
 │    └── Profile
 │          └── Settings
```

If `Settings` needs a `UserRepository`, you would normally pass it through every widget above it.

Providers eliminate this problem.

Instead, any widget can directly access the provider whenever needed.

Benefits include:

- No prop drilling
- Automatic dependency management
- Lazy initialization
- Cached values
- Easy testing
- Automatic rebuilding when dependencies change

---

## Syntax

### Creating a Provider

```dart
final greetingProvider = Provider<String>((ref) {
  return 'Hello Riverpod!';
});
```

Explanation:

- `Provider<T>` creates a provider that exposes a value of type `T`.
- `ref` allows the provider to read other providers or use lifecycle APIs.
- The returned value is cached until the provider is invalidated.

---

### Reading a Provider

```dart
class HomePage extends ConsumerWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final greeting = ref.watch(greetingProvider);

    return Text(greeting);
  }
}
```

Explanation:

- `ConsumerWidget` provides access to `WidgetRef`.
- `ref.watch()` subscribes to the provider.
- The widget rebuilds whenever the provider's value changes.

---

### Provider Depending on Another Provider

```dart
final firstNameProvider = Provider((ref) => 'John');

final fullNameProvider = Provider((ref) {
  final firstName = ref.watch(firstNameProvider);

  return '$firstName Doe';
});
```

Explanation:

- Providers can depend on other providers.
- `ref.watch()` creates a dependency relationship.
- If `firstNameProvider` changes, `fullNameProvider` recomputes automatically.

---

## Mental Model

Think of providers as **factories managed by Riverpod**.

```text
             Provider
                 │
      "How do I create this value?"
                 │
                 ▼
          Riverpod Container
                 │
         Creates the value
                 │
        Caches the value
                 │
      Gives it to every consumer
```

You define **how** to create a value.

Riverpod decides:

- when to create it
- when to reuse it
- when to dispose it
- when to recreate it

---

## Examples

### Simple Example

```dart
final appNameProvider = Provider<String>((ref) {
  return 'My Application';
});
```

Explanation:

- Provides a constant string.
- Every consumer receives the same cached value.

---

### Repository Example

```dart
class UserRepository {
  Future<String> getUser() async {
    return 'John';
  }
}

final userRepositoryProvider = Provider<UserRepository>((ref) {
  return UserRepository();
});
```

Explanation:

- The provider creates a single `UserRepository`.
- Widgets and other providers can access the same instance.

---

### Computed Provider

```dart
final firstNameProvider = Provider((ref) => 'John');
final lastNameProvider = Provider((ref) => 'Doe');

final fullNameProvider = Provider((ref) {
  return '${ref.watch(firstNameProvider)} ${ref.watch(lastNameProvider)}';
});
```

Explanation:

- Combines values from multiple providers.
- Automatically updates whenever either dependency changes.

---

## When to Use

Use `Provider` when you need:

- Read-only values
- Dependency injection
- Services
- Repositories
- Configuration objects
- Utility classes
- Computed values
- Shared business logic without mutable state

---

## When NOT to Use

Do not use `Provider` when the value changes over time.

Instead use:

| Situation | Use |
|-----------|-----|
| Mutable state | `NotifierProvider` |
| Simple mutable value | `StateProvider` |
| Future | `FutureProvider` |
| Stream | `StreamProvider` |
| Async business logic | `AsyncNotifierProvider` |

---

## Best Practices

- Keep providers focused on one responsibility.
- Prefer immutable values.
- Compose providers instead of creating large providers.
- Use providers for dependency injection.
- Avoid putting UI logic inside providers.
- Let providers depend on other providers rather than manually creating dependencies.

---

## Common Mistakes

### 1. Using Provider for mutable state

❌ Wrong

```dart
final counterProvider = Provider((ref) => 0);
```

Why it's wrong:

- `Provider` is read-only.
- The value cannot be updated.

✔ Correct

```dart
final counterProvider = StateProvider((ref) => 0);
```

---

### 2. Creating dependencies manually

❌ Wrong

```dart
final apiProvider = Provider((ref) {
  final client = HttpClient();

  return ApiService(client);
});
```

If another provider also creates `HttpClient`, multiple unnecessary instances exist.

✔ Correct

```dart
final httpClientProvider = Provider((ref) {
  return HttpClient();
});

final apiProvider = Provider((ref) {
  final client = ref.watch(httpClientProvider);

  return ApiService(client);
});
```

Why it's better:

- Dependencies are reusable.
- Riverpod manages the dependency graph.
- Easier to override during testing.

---

### 3. Doing expensive work inside a Provider

❌ Wrong

```dart
final usersProvider = Provider((ref) {
  return fetchUsers();
});
```

Why it's wrong:

- `Provider` is intended for synchronous values.
- Asynchronous operations belong in dedicated async providers.

✔ Correct

```dart
final usersProvider = FutureProvider((ref) async {
  return fetchUsers();
});
```

---

## Related APIs

- Provider
- StateProvider
- FutureProvider
- StreamProvider
- NotifierProvider
- AsyncNotifierProvider
- WidgetRef
- ref.watch()
- ref.read()

---

## Summary

A `Provider` is the simplest Riverpod provider. It defines **how to create a value**, while Riverpod handles its creation, caching, dependency tracking, and lifecycle. Use it for read-only values, dependency injection, services, repositories, and computed data.