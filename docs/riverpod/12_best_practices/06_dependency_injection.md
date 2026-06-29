# Dependency Injection

Provide dependencies through Riverpod instead of creating them manually, making your application more modular, testable, and maintainable.

---

# What is it?

Dependency Injection (DI) is the practice of supplying an object with the dependencies it needs instead of allowing it to create those dependencies itself.

In Riverpod, providers act as a dependency injection container. Services, repositories, clients, and other shared objects are exposed through providers and consumed wherever they are needed.

Unlike traditional DI frameworks that rely on code generation or reflection, Riverpod uses providers to create and manage dependencies in a type-safe and compile-time friendly way.

---

# Why does it exist?

Without dependency injection, classes often create their own dependencies.

For example:

- A repository creates its own HTTP client.
- A notifier creates its own repository.
- A widget creates its own service.

This tightly couples classes together, making them difficult to:

- Test
- Replace
- Mock
- Reuse
- Maintain

Dependency injection separates object creation from object usage.

---

# Syntax

## Providing a dependency

```dart
final apiClientProvider = Provider<ApiClient>((ref) {
  return ApiClient();
});
```

Explanation:

- The provider owns the `ApiClient`.
- Every consumer receives the same instance unless overridden.

---

## Injecting a dependency into another provider

```dart
final userRepositoryProvider = Provider<UserRepository>((ref) {
  final apiClient = ref.watch(apiClientProvider);

  return UserRepository(apiClient);
});
```

Explanation:

- `UserRepository` depends on `ApiClient`.
- Riverpod resolves the dependency automatically.

---

## Using the injected dependency

```dart
final userProvider = FutureProvider<User>((ref) async {
  final repository = ref.watch(userRepositoryProvider);

  return repository.fetchUser();
});
```

Explanation:

- The provider doesn't know how the repository was created.
- It simply consumes the injected dependency.

---

# Mental Model

Think of providers as a dependency graph.

```text
ApiClientProvider
        │
        ▼
UserRepositoryProvider
        │
        ▼
UserProvider
        │
        ▼
ConsumerWidget
```

Each provider depends only on the providers directly below it, creating a clear and maintainable dependency chain.

---

# Examples

## Simple Example

```dart
final loggerProvider = Provider<Logger>((ref) {
  return Logger();
});

final analyticsProvider = Provider<Analytics>((ref) {
  final logger = ref.watch(loggerProvider);

  return Analytics(logger);
});
```

Explanation:

- `Analytics` receives its dependency through Riverpod.
- No object creates its own dependencies.

---

## Real-World Example

```dart
final dioProvider = Provider<Dio>((ref) {
  return Dio();
});

final authRepositoryProvider = Provider<AuthRepository>((ref) {
  final dio = ref.watch(dioProvider);

  return AuthRepository(dio);
});

final authProvider =
    AsyncNotifierProvider<AuthNotifier, User?>(
      AuthNotifier.new,
    );
```

Explanation:

- The HTTP client is shared.
- The repository depends on the HTTP client.
- The notifier depends on the repository.
- Each layer has a single responsibility.

---

# When to Use

Use dependency injection for:

- Repositories
- API clients
- Database clients
- Local storage
- Authentication services
- Analytics services
- Logging
- Shared business services
- Configuration objects

---

# When NOT to Use

Avoid dependency injection for:

- Temporary local variables
- Simple helper methods
- Immutable constants
- Values used only within a single function

Not every object needs to become a provider.

---

# Best Practices

- Inject dependencies through providers.
- Keep dependencies immutable whenever possible.
- Prefer constructor injection over global variables.
- Keep dependency chains simple.
- Place shared dependencies in the `core` module.
- Override dependencies during testing instead of modifying production code.

---

# Common Mistakes

## Creating dependencies manually

Wrong:

```dart
class UserRepository {
  final api = ApiClient();
}
```

Explanation:

- The repository controls how its dependency is created.
- Replacing or mocking the API client becomes difficult.

Correct:

```dart
class UserRepository {
  UserRepository(this.api);

  final ApiClient api;
}
```

Explanation:

- The dependency is supplied from outside.
- The repository is easier to test and reuse.

---

## Accessing global singletons

Wrong:

```dart
final api = ApiClient.instance;
```

Explanation:

- Global state creates hidden dependencies.
- Tests become more difficult to isolate.

Correct:

```dart
final api = ref.watch(apiClientProvider);
```

Explanation:

- Dependencies are explicit.
- Riverpod manages their lifecycle.

---

## Skipping providers for shared services

Wrong:

```dart
final repository = UserRepository(ApiClient());

await repository.fetchUser();
```

Explanation:

- Every caller creates a new repository and API client.
- Resources are duplicated unnecessarily.

Correct:

```dart
final repository = ref.watch(userRepositoryProvider);

await repository.fetchUser();
```

Explanation:

- The repository is shared and managed by Riverpod.
- Consumers don't need to know how it was created.

---

# Related APIs

- Provider
- ProviderScope
- ProviderContainer
- `ref.watch()`
- `ref.read()`
- `.overrideWith()`
- `.overrideWithValue()`

---

# Summary

Riverpod's providers form a powerful dependency injection system. Instead of creating objects manually, expose shared dependencies through providers and inject them where needed. This reduces coupling, improves testability, simplifies maintenance, and allows Riverpod to manage the lifecycle of your application's services and repositories.