# Provider

`Provider` is the simplest Riverpod provider. It exposes an immutable, read-only value or object and automatically manages its creation, caching, and lifecycle.

---

## What is it?

`Provider` is used to expose values that **do not change over time**.

It is ideal for:

- Services
- Repositories
- Configuration
- Dependency injection
- Utility classes
- Computed values
- Immutable objects

Unlike stateful providers such as `StateProvider` or `NotifierProvider`, a `Provider` does **not** manage mutable state.

Instead, it defines **how a value should be created**, and Riverpod handles the rest.

---

## Why does it exist?

Many applications contain objects that should be created once and shared throughout the application.

Examples include:

- API clients
- Authentication services
- Repositories
- Database instances
- Configuration values

Without `Provider`, these objects are often:

- Global variables
- Singleton classes
- Passed through constructors
- Managed manually

`Provider` offers a cleaner solution by:

- Lazily creating objects
- Caching instances
- Managing lifecycles
- Tracking dependencies
- Supporting overrides during testing

---

## Syntax

### Creating a Provider

```dart
final greetingProvider = Provider<String>((ref) {
  return 'Hello Riverpod';
});
```

Explanation:

- `Provider<T>` exposes a value of type `T`.
- `ref` provides access to other providers and lifecycle APIs.
- The returned value is cached automatically.

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

- `ref.watch()` subscribes to the provider.
- The widget rebuilds if the provider is recomputed.

---

### Dependency Injection

```dart
final apiProvider = Provider<ApiService>((ref) {
  return ApiService();
});

final repositoryProvider = Provider<UserRepository>((ref) {
  return UserRepository(
    ref.watch(apiProvider),
  );
});
```

Explanation:

- `repositoryProvider` depends on `apiProvider`.
- Riverpod automatically tracks this dependency.

---

## Mental Model

Think of a `Provider` as a **factory**.

```text
          Provider
              │
              ▼
    Creates an Object
              │
              ▼
Riverpod Caches It
              │
      Shared Everywhere
```

The provider does **not** hold mutable state.

It simply defines how to create an object.

---

## Examples

### Configuration Provider

```dart
final apiUrlProvider = Provider<String>((ref) {
  return 'https://api.example.com';
});
```

Explanation:

- Exposes a constant configuration value.
- The value is shared across the application.

---

### Service Provider

```dart
final authServiceProvider = Provider<AuthService>((ref) {
  return AuthService();
});
```

Explanation:

- Creates a single `AuthService`.
- All consumers use the same cached instance.

---

### Repository Provider

```dart
final apiProvider = Provider<ApiService>((ref) {
  return ApiService();
});

final userRepositoryProvider = Provider<UserRepository>((ref) {
  return UserRepository(
    ref.watch(apiProvider),
  );
});
```

Explanation:

- Demonstrates dependency injection.
- Riverpod manages the dependency graph automatically.

---

### Computed Provider

```dart
final firstNameProvider = Provider((ref) => 'John');
final lastNameProvider = Provider((ref) => 'Doe');

final fullNameProvider = Provider((ref) {
  final firstName = ref.watch(firstNameProvider);
  final lastName = ref.watch(lastNameProvider);

  return '$firstName $lastName';
});
```

Explanation:

- Computes a value from other providers.
- Automatically recomputes when dependencies change.

---

## When to Use

Use `Provider` for:

- Dependency injection
- Services
- Repositories
- API clients
- Database instances
- Configuration
- Utility classes
- Immutable objects
- Computed values

---

## When NOT to Use

Do not use `Provider` when the value changes over time.

Instead, use the appropriate provider type:

| Scenario | Recommended Provider |
|----------|----------------------|
| Mutable value | `StateProvider` |
| Business logic with mutable state | `NotifierProvider` |
| Future | `FutureProvider` |
| Stream | `StreamProvider` |
| Asynchronous state | `AsyncNotifierProvider` |

---

## Best Practices

- Keep providers focused on a single responsibility.
- Prefer immutable objects.
- Compose providers using `ref.watch()`.
- Use providers for dependency injection instead of singletons.
- Avoid business logic that mutates state inside a `Provider`.

---

## Common Mistakes

### 1. Using Provider for Mutable State

❌ Wrong

```dart
final counterProvider = Provider((ref) => 0);
```

Why it's wrong:

- `Provider` exposes a read-only value.
- It cannot update or notify listeners.

✔ Correct

```dart
final counterProvider = StateProvider((ref) => 0);
```

---

### 2. Creating Dependencies Manually

❌ Wrong

```dart
final repositoryProvider = Provider((ref) {
  return UserRepository(
    ApiService(),
  );
});
```

Why it's wrong:

- Dependencies cannot be shared or overridden.
- Makes testing more difficult.

✔ Correct

```dart
final apiProvider = Provider((ref) {
  return ApiService();
});

final repositoryProvider = Provider((ref) {
  return UserRepository(
    ref.watch(apiProvider),
  );
});
```

---

### 3. Performing Asynchronous Work

❌ Wrong

```dart
final usersProvider = Provider((ref) async {
  return repository.fetchUsers();
});
```

Why it's wrong:

- `Provider` is intended for synchronous values.
- Asynchronous operations should use async provider types.

✔ Correct

```dart
final usersProvider = FutureProvider((ref) async {
  return repository.fetchUsers();
});
```

---

## Related APIs

- StateProvider
- FutureProvider
- StreamProvider
- NotifierProvider
- AsyncNotifierProvider
- ProviderContainer
- ref.watch()
- ref.read()

---

## Summary

`Provider` is Riverpod's simplest provider type. It creates and exposes immutable values, services, repositories, and computed objects while Riverpod manages caching, dependency tracking, and lifecycle. It is the preferred choice for dependency injection and read-only data, but it should not be used for mutable state.