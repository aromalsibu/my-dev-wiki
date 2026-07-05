# Overriding Providers

Replace a provider's implementation or value during testing to isolate the code under test.

---

# What is it?

Overriding providers allows you to replace a provider with a different implementation or value within a specific `ProviderContainer` or `ProviderScope`.

Instead of using the real provider, Riverpod uses the overridden version.

This is one of Riverpod's most powerful testing features because it lets you test business logic without modifying production code.

---

# Why does it exist?

Applications often depend on external services such as:

- APIs
- Databases
- Authentication services
- Local storage
- Analytics

Using these real services during tests makes tests:

- Slow
- Difficult to reproduce
- Dependent on external systems

Provider overrides allow tests to replace these dependencies with fake or mock implementations, making tests fast, reliable, and isolated.

---

# Syntax

## Overriding in a ProviderContainer

```dart
final container = ProviderContainer(
  overrides: [
    apiProvider.overrideWithValue(
      FakeApiService(),
    ),
  ],
);
```

Explanation:

- The override applies only to this container.
- The original provider remains unchanged elsewhere.
- All reads of `apiProvider` use `FakeApiService`.

---

## Overriding in a ProviderScope

```dart
ProviderScope(
  overrides: [
    apiProvider.overrideWithValue(
      FakeApiService(),
    ),
  ],
  child: const MyApp(),
);
```

Explanation:

- Every widget inside this `ProviderScope` uses the fake implementation.
- Useful for widget tests and preview environments.

---

## Using overrideWith()

```dart
final container = ProviderContainer(
  overrides: [
    repositoryProvider.overrideWith(
      (ref) => FakeRepository(),
    ),
  ],
);
```

Explanation:

- Replaces the provider's implementation.
- The new implementation can access `ref`.
- Suitable when provider creation requires logic.

---

# Mental Model

Think of overrides as dependency replacement.

Without override:

```text
Widget/Test
      │
      ▼
UserRepository
      │
      ▼
Real API
```

With override:

```text
Widget/Test
      │
      ▼
UserRepository
      │
      ▼
Fake API
```

The widget or test still reads the same provider, but Riverpod supplies a different implementation.

---

# Examples

## Simple Value Override

```dart
final container = ProviderContainer(
  overrides: [
    counterProvider.overrideWithValue(100),
  ],
);

expect(
  container.read(counterProvider),
  100,
);
```

Explanation:

- The provider always returns `100`.
- No production logic is executed.

---

## Mocking an API

```dart
final container = ProviderContainer(
  overrides: [
    apiProvider.overrideWithValue(
      FakeApiService(),
    ),
  ],
);

final api = container.read(apiProvider);
```

Explanation:

- Tests use a fake API.
- No real network requests are made.

---

## Replacing a Repository

```dart
final container = ProviderContainer(
  overrides: [
    repositoryProvider.overrideWith(
      (ref) => FakeRepository(),
    ),
  ],
);
```

Explanation:

- Swaps the repository implementation.
- Keeps the rest of the application unchanged.

---

# When to Use

Use provider overrides when:

- Writing unit tests
- Writing widget tests
- Mocking dependencies
- Replacing APIs
- Injecting fake repositories
- Testing different application states

---

# When NOT to Use

Avoid provider overrides when:

- Building normal production applications
- The provider should always use its real implementation
- A simple provider is sufficient
- You are trying to change business logic permanently

Overrides are temporary and scoped.

---

# Best Practices

- Override only the providers required by the test.
- Create fresh overrides for every test.
- Use fake implementations instead of real services.
- Dispose containers after testing.
- Keep production providers free from testing code.

---

# Common Mistakes

## Overriding Too Many Providers

**Wrong**

```dart
overrides: [
  providerA.overrideWithValue(...),
  providerB.overrideWithValue(...),
  providerC.overrideWithValue(...),
]
```

Unnecessary overrides make tests harder to understand.

**Correct**

Override only the providers required for the current test.

---

## Sharing Overrides Between Tests

**Wrong**

```dart
final container = ProviderContainer(
  overrides: [...],
);
```

Reusing the same container across multiple tests.

**Correct**

Create a new `ProviderContainer` for every test.

---

## Modifying Production Providers

**Wrong**

```dart
final apiProvider = Provider<Api>((ref) {
  if (kDebugMode) {
    return FakeApi();
  }

  return RealApi();
});
```

Mixing test logic with production code.

**Correct**

Keep providers unchanged and override them during testing.

---

# Related APIs

- `ProviderContainer`
- `ProviderScope`
- `.overrideWith`
- `.overrideWithValue`
- `ProviderObserver`

---

# Summary

Provider overrides allow you to replace a provider's implementation or value within a specific `ProviderContainer` or `ProviderScope`. They are fundamental to Riverpod's testing strategy, enabling isolated, fast, and reliable tests without changing production code.