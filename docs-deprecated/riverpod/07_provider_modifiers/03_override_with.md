# .overrideWith

Replaces a provider's implementation with a custom implementation.

---

# What is it?

`.overrideWith` is a provider modifier used to replace the implementation of a provider.

Instead of using the original provider logic, Riverpod executes the new implementation supplied by the override. This allows you to change how a provider behaves without modifying its original definition.

Overrides are commonly used in testing, dependency injection, and application configuration.

---

# Why does it exist?

There are situations where the default provider implementation should not be used.

For example:

- A test should use fake data instead of making API calls.
- A development build should use mock services.
- A feature should use a different implementation based on the environment.

Without `.overrideWith`, you would need to modify the provider itself or introduce conditional logic.

`.overrideWith` keeps providers reusable by allowing their implementation to be swapped externally.

---

# Syntax

## Basic Usage

```dart
final greetingProvider = Provider<String>((ref) {
  return 'Hello, World!';
});

ProviderScope(
  overrides: [
    greetingProvider.overrideWith((ref) {
      return 'Hello, Riverpod!';
    }),
  ],
  child: const MyApp(),
);
```

Explanation:

- `overrideWith()` replaces the provider's implementation.
- Inside the `ProviderScope`, every read of `greetingProvider` uses the new implementation.
- Outside the scope, the original implementation remains unchanged.

---

## Overriding a FutureProvider

```dart
final userProvider = FutureProvider<User>((ref) async {
  return api.fetchUser();
});

ProviderScope(
  overrides: [
    userProvider.overrideWith((ref) async {
      return fakeUser;
    }),
  ],
  child: const MyApp(),
);
```

Explanation:

- The original API call is skipped.
- The override returns custom data instead.

---

## Overriding a NotifierProvider

```dart
final counterProvider =
    NotifierProvider<CounterNotifier, int>(
      CounterNotifier.new,
    );

ProviderScope(
  overrides: [
    counterProvider.overrideWith(CounterNotifier.new),
  ],
  child: const MyApp(),
);
```

Explanation:

- The original notifier is replaced.
- Riverpod creates the new notifier instead.

---

# Mental Model

Think of `.overrideWith` as replacing the provider's engine.

Without override

```text
Widget
   │
   ▼
Original Provider
   │
   ▼
Original Implementation
```

With override

```text
Widget
   │
   ▼
Provider
   │
   ▼
Override Implementation
```

The widget still reads the same provider, but the provider behaves differently.

---

# Examples

## Simple Example

```dart
final numberProvider = Provider<int>((ref) {
  return 10;
});

ProviderScope(
  overrides: [
    numberProvider.overrideWith((ref) => 99),
  ],
  child: const MyApp(),
);
```

Explanation:

- Inside the scope, `numberProvider` returns `99`.
- Outside the scope, it still returns `10`.

---

## Real-World Example

```dart
final apiProvider = Provider<ApiService>((ref) {
  return ApiService();
});

ProviderScope(
  overrides: [
    apiProvider.overrideWith((ref) {
      return FakeApiService();
    }),
  ],
  child: const MyApp(),
);
```

Explanation:

- Production uses the real API.
- Tests or development can use a fake implementation.

---

# When to Use

Use `.overrideWith` when:

- Writing unit tests
- Injecting mock services
- Replacing repositories
- Switching implementations
- Configuring environments
- Providing fake dependencies

---

# When NOT to Use

Avoid `.overrideWith` when:

- You only need to change a provider's value.
- The provider implementation should always remain the same.
- The change can be handled inside the provider itself.

For replacing only the returned value, consider `.overrideWithValue` instead.

---

# Best Practices

- Keep overrides close to the `ProviderScope` where they are needed.
- Use overrides primarily for testing and dependency injection.
- Prefer fake implementations over complex conditional logic.
- Override only the providers required for the current scope.
- Keep production providers free from testing code.

---

# Common Mistakes

## Modifying the Original Provider

**Wrong**

```dart
final apiProvider = Provider<ApiService>((ref) {
  if (kDebugMode) {
    return FakeApiService();
  }

  return ApiService();
});
```

This mixes production and testing logic.

**Correct**

```dart
ProviderScope(
  overrides: [
    apiProvider.overrideWith(
      (ref) => FakeApiService(),
    ),
  ],
  child: const MyApp(),
);
```

Keep the provider implementation clean and use overrides externally.

---

## Assuming the Override Is Global

**Wrong**

```dart
ProviderScope(
  overrides: [
    apiProvider.overrideWith(
      (ref) => FakeApiService(),
    ),
  ],
  child: const ScreenA(),
);
```

Expecting every screen in the application to use the override.

**Correct**

Overrides only affect the `ProviderScope` in which they are declared.

---

## Overriding More Providers Than Necessary

**Wrong**

```dart
overrides: [
  providerA.overrideWith(...),
  providerB.overrideWith(...),
  providerC.overrideWith(...),
]
```

Overriding unrelated providers makes tests harder to understand.

**Correct**

Override only the providers required for the current scenario.

---

# Related APIs

- `.overrideWithValue`
- `.family`
- `.autoDispose`
- `ProviderScope`
- `ProviderContainer`

---

# Summary

`.overrideWith` replaces a provider's implementation with a custom one. It is primarily used for testing, dependency injection, and environment-specific behavior, allowing providers to be reused without changing their original implementation.