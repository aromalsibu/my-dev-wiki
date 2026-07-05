# Testing Providers

Verify that providers return the expected values and behave correctly under different conditions.

---

# What is it?

Testing providers is the process of validating that a Riverpod provider produces the correct state or value.

Unlike widget tests, provider tests focus only on business logic. They do not require a Flutter widget tree, making them:

- Fast
- Isolated
- Easy to maintain

Provider tests typically use a `ProviderContainer` to create an isolated Riverpod environment.

---

# Why does it exist?

Providers often contain important application logic, such as:

- Fetching data
- Transforming state
- Combining dependencies
- Caching results

If this logic is only tested through the UI, tests become slower and harder to debug.

Testing providers directly allows you to verify business logic independently from the presentation layer.

---

# Syntax

## Basic Provider Test

```dart
test('counter starts at 0', () {
  final container = ProviderContainer();

  expect(
    container.read(counterProvider),
    0,
  );

  container.dispose();
});
```

Explanation:

- Creates an isolated provider environment.
- `read()` returns the provider's current value.
- Dispose the container after the test completes.

---

## Testing with a Fake Dependency

```dart
test('returns fake user', () {
  final container = ProviderContainer(
    overrides: [
      userRepositoryProvider.overrideWithValue(
        FakeUserRepository(),
      ),
    ],
  );

  final user = container.read(userProvider);

  expect(user.name, 'John');

  container.dispose();
});
```

Explanation:

- Replaces the real repository with a fake implementation.
- Tests only the provider's logic.
- No external services are used.

---

## Testing a FutureProvider

```dart
test('loads user', () async {
  final container = ProviderContainer();

  final user = await container.read(
    userProvider.future,
  );

  expect(user.name, 'Alice');

  container.dispose();
});
```

Explanation:

- `.future` waits for the provider to complete.
- Useful for testing asynchronous providers.

---

# Mental Model

Think of a provider test as interacting directly with Riverpod's engine.

```text
Test
 │
 ▼
ProviderContainer
 │
 ▼
Provider
 │
 ▼
Result
```

No widgets are involved, making provider tests fast and focused.

---

# Examples

## Testing a Provider

```dart
final greetingProvider = Provider<String>(
  (ref) => 'Hello',
);
```

```dart
test('returns greeting', () {
  final container = ProviderContainer();

  expect(
    container.read(greetingProvider),
    'Hello',
  );

  container.dispose();
});
```

Explanation:

- Reads the provider directly.
- Verifies the expected value.

---

## Testing a Computed Provider

```dart
final firstNameProvider = Provider((ref) => 'John');

final greetingProvider = Provider(
  (ref) => 'Hello ${ref.watch(firstNameProvider)}',
);
```

```dart
test('creates greeting', () {
  final container = ProviderContainer();

  expect(
    container.read(greetingProvider),
    'Hello John',
  );

  container.dispose();
});
```

Explanation:

- Confirms provider dependencies work correctly.
- Ensures computed values are correct.

---

## Testing an Async Provider

```dart
test('loads products', () async {
  final container = ProviderContainer();

  final products = await container.read(
    productsProvider.future,
  );

  expect(products.isNotEmpty, true);

  container.dispose();
});
```

Explanation:

- Waits for asynchronous work to finish.
- Verifies the loaded data.

---

# When to Use

Test providers when they:

- Contain business logic
- Combine multiple providers
- Fetch data
- Transform values
- Cache results
- Expose computed state

---

# When NOT to Use

Avoid provider tests when:

- You're testing widget layout
- You're verifying navigation
- You're testing animations
- The behavior belongs in a widget test instead

Choose the test type that matches what you're verifying.

---

# Best Practices

- Create a new `ProviderContainer` for every test.
- Dispose the container after each test.
- Override external dependencies with fake implementations.
- Test one behavior per test case.
- Keep tests deterministic and independent.

---

# Common Mistakes

## Sharing a ProviderContainer

**Wrong**

```dart
final container = ProviderContainer();

test(...);

test(...);
```

Tests may affect one another by sharing state.

**Correct**

```dart
test('...', () {
  final container = ProviderContainer();

  // Test logic

  container.dispose();
});
```

Create a fresh container for each test.

---

## Testing Real APIs

**Wrong**

```dart
final user = await container.read(userProvider.future);
```

When `userProvider` makes a real network request.

**Correct**

Override the API or repository with a fake implementation before running the test.

---

## Forgetting to Dispose

**Wrong**

```dart
final container = ProviderContainer();
```

Leaving the container alive after the test.

**Correct**

```dart
final container = ProviderContainer();

try {
  // Test logic
} finally {
  container.dispose();
}
```

Always clean up manually created containers.

---

# Related APIs

- `ProviderContainer`
- `Provider`
- `FutureProvider`
- `NotifierProvider`
- `.overrideWith`
- `.overrideWithValue`

---

# Summary

Testing providers allows you to verify Riverpod business logic without involving the Flutter UI. By creating an isolated `ProviderContainer`, overriding dependencies when necessary, and asserting provider values, you can build fast, reliable, and maintainable unit tests.