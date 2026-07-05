# Testing Notifiers

Verify that `Notifier`, `AsyncNotifier`, and `StreamNotifier` correctly update and expose state.

---

# What is it?

Testing notifiers is the process of validating that a notifier updates its state correctly in response to method calls.

Unlike provider tests that verify a provider's output, notifier tests focus on:

- State transitions
- Business logic
- Method behavior
- Error handling

Since notifiers contain mutable state and application logic, they are one of the most important parts of a Riverpod application to test.

---

# Why does it exist?

Notifiers often contain business logic such as:

- Incrementing counters
- Logging users in
- Fetching remote data
- Updating application settings
- Managing lists

Bugs in these methods can affect the entire application.

Testing notifiers directly ensures that state changes are correct before the UI is involved.

---

# Syntax

## Testing a Notifier

```dart
test('increment increases counter', () {
  final container = ProviderContainer();

  final notifier = container.read(
    counterProvider.notifier,
  );

  notifier.increment();

  expect(
    container.read(counterProvider),
    1,
  );

  container.dispose();
});
```

Explanation:

- `.notifier` returns the notifier instance.
- Call the notifier method.
- Verify the updated provider state.

---

## Testing Multiple Updates

```dart
test('multiple increments', () {
  final container = ProviderContainer();

  final notifier = container.read(
    counterProvider.notifier,
  );

  notifier.increment();
  notifier.increment();

  expect(
    container.read(counterProvider),
    2,
  );

  container.dispose();
});
```

Explanation:

- Calls multiple notifier methods.
- Confirms state changes accumulate correctly.

---

## Testing an AsyncNotifier

```dart
test('loads users', () async {
  final container = ProviderContainer();

  final users = await container.read(
    usersProvider.future,
  );

  expect(
    users.isNotEmpty,
    true,
  );

  container.dispose();
});
```

Explanation:

- Waits for the asynchronous state to complete.
- Verifies the resulting data.

---

# Mental Model

A notifier test interacts directly with the state manager.

```text
Test
 │
 ▼
Notifier Method
 │
 ▼
State Changes
 │
 ▼
Assertions
```

Instead of testing the UI, you verify the state transitions themselves.

---

# Examples

## Counter Notifier

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}
```

```dart
test('counter increments', () {
  final container = ProviderContainer();

  final notifier = container.read(
    counterProvider.notifier,
  );

  notifier.increment();

  expect(
    container.read(counterProvider),
    1,
  );

  container.dispose();
});
```

Explanation:

- Tests the notifier method directly.
- Verifies the updated state.

---

## Todo Notifier

```dart
test('adds todo', () {
  final container = ProviderContainer();

  final notifier = container.read(
    todoProvider.notifier,
  );

  notifier.addTodo(
    Todo(title: 'Buy milk'),
  );

  expect(
    container.read(todoProvider).length,
    1,
  );

  container.dispose();
});
```

Explanation:

- Calls the business logic.
- Confirms the provider state reflects the change.

---

## AsyncNotifier Example

```dart
test('loads profile', () async {
  final container = ProviderContainer();

  await container.read(
    profileProvider.future,
  );

  final profile = container.read(profileProvider);

  expect(
    profile.hasValue,
    true,
  );

  container.dispose();
});
```

Explanation:

- Waits for the async operation to finish.
- Verifies the final provider state.

---

# When to Use

Test notifiers when they:

- Modify state
- Perform calculations
- Fetch data
- Validate user input
- Manage collections
- Execute business rules

---

# When NOT to Use

Avoid notifier tests when:

- Testing widget layouts
- Testing navigation
- Verifying UI appearance
- Testing Flutter framework behavior

Those belong in widget or integration tests.

---

# Best Practices

- Test one notifier method per test.
- Assert the final state after every operation.
- Override repositories and APIs with fake implementations.
- Keep notifier methods small and focused.
- Create a fresh `ProviderContainer` for every test.
- Dispose the container after testing.

---

# Common Mistakes

## Testing Through the UI

**Wrong**

Triggering a button press just to verify notifier logic.

**Correct**

Call the notifier method directly.

---

## Testing Multiple Behaviors Together

**Wrong**

```dart
notifier.login();
notifier.logout();
notifier.refresh();
```

Testing several behaviors in one test makes failures harder to diagnose.

**Correct**

Write separate tests for each method.

---

## Using Real Dependencies

**Wrong**

Allowing the notifier to call a real API during testing.

**Correct**

Override repositories or services with fake implementations before creating the container.

---

# Related APIs

- `Notifier`
- `AsyncNotifier`
- `StreamNotifier`
- `NotifierProvider`
- `AsyncNotifierProvider`
- `ProviderContainer`
- `.overrideWith`
- `.overrideWithValue`

---

# Summary

Testing notifiers focuses on verifying state transitions and business logic rather than UI behavior. By calling notifier methods directly, asserting the resulting state, and replacing external dependencies with fake implementations, you can create fast, reliable, and maintainable unit tests for your application's core logic.