# Mocking Dependencies

Replace external dependencies with fake or mock implementations to create fast, isolated, and reliable tests.

---

# What is it?

Mocking dependencies is the practice of replacing real services with controlled test implementations.

Instead of allowing providers to use real objects like:

- APIs
- Databases
- Local storage
- Authentication services
- Analytics

you provide fake or mock versions that return predictable results.

In Riverpod, this is typically done using **provider overrides**.

---

# Why does it exist?

External dependencies introduce problems during testing:

- Network requests are slow.
- APIs may be unavailable.
- Databases require setup.
- Tests become unpredictable.
- Results may change over time.

Mocking removes these variables, allowing tests to focus only on your application's business logic.

---

# Syntax

## Overriding a Provider with a Fake

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

- Replaces the real API with a fake implementation.
- Only this container uses the fake dependency.

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
- Useful when creating the dependency requires access to `ref`.

---

## Reading the Mocked Provider

```dart
final repository = container.read(
  repositoryProvider,
);
```

Explanation:

- Reads the overridden provider.
- The real implementation is never used.

---

# Mental Model

Think of provider overrides as swapping parts before the application starts.

Without mocking:

```text
Provider
   │
   ▼
Real Repository
   │
   ▼
Real API
```

With mocking:

```text
Provider
   │
   ▼
Fake Repository
   │
   ▼
Fake Data
```

Your provider doesn't know the dependency has been replaced.

---

# Examples

## Fake API Service

```dart
class FakeApiService implements ApiService {
  @override
  Future<User> getUser() async {
    return User(name: 'Alice');
  }
}
```

Explanation:

- Returns predictable data.
- No network request is performed.

---

## Using the Fake in a Test

```dart
test('loads fake user', () async {
  final container = ProviderContainer(
    overrides: [
      apiProvider.overrideWithValue(
        FakeApiService(),
      ),
    ],
  );

  final user = await container.read(
    userProvider.future,
  );

  expect(user.name, 'Alice');

  container.dispose();
});
```

Explanation:

- Uses the fake service.
- Produces deterministic test results.

---

## Mocking a Repository

```dart
class FakeUserRepository
    implements UserRepository {
  @override
  Future<List<User>> fetchUsers() async {
    return [
      User(name: 'John'),
      User(name: 'Jane'),
    ];
  }
}
```

Explanation:

- Simulates repository behavior.
- Makes tests independent of external systems.

---

# Fake vs Mock

| Fake | Mock |
|------|------|
| Has a simple working implementation | Behavior is configured during the test |
| Returns predictable data | Can verify method calls and interactions |
| Usually handwritten | Often created using a mocking library |
| Good for most provider tests | Useful when interaction verification is important |

---

# When to Use

Use mocked or fake dependencies when:

- Testing providers
- Testing notifiers
- Avoiding network requests
- Simulating failures
- Testing edge cases
- Isolating business logic

---

# When NOT to Use

Avoid mocking when:

- Writing integration tests
- Verifying real API behavior
- Testing the actual database implementation
- End-to-end application testing

Those scenarios should use real dependencies.

---

# Best Practices

- Prefer simple fake implementations whenever possible.
- Mock only external dependencies, not the code being tested.
- Keep fake implementations deterministic.
- Create fresh overrides for every test.
- Keep production code free from testing logic.

---

# Common Mistakes

## Mocking the Class Under Test

**Wrong**

Mocking the notifier or provider being tested.

```dart
final mockNotifier = MockNotifier();
```

This doesn't test your actual implementation.

**Correct**

Mock the notifier's dependencies instead.

---

## Using Real APIs

**Wrong**

```dart
await api.getUsers();
```

Tests become slow and unreliable.

**Correct**

Override the API provider with a fake implementation.

---

## Sharing Fake Objects Across Tests

**Wrong**

```dart
final fakeApi = FakeApiService();
```

Used by multiple tests.

Shared state can make tests influence each other.

**Correct**

Create a new fake instance inside each test.

---

# Related APIs

- `ProviderContainer`
- `ProviderScope`
- `.overrideWith`
- `.overrideWithValue`
- `NotifierProvider`
- `FutureProvider`

---

# Summary

Mocking dependencies allows Riverpod tests to replace external services with predictable implementations. By overriding providers with fake or mock objects, you can create fast, isolated, and reliable tests that focus entirely on your application's business logic without relying on external systems.