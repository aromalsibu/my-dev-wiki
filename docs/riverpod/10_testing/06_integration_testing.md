# Integration Testing

Verify that multiple providers, services, and widgets work together as a complete application.

---

# What is it?

Integration testing validates that different parts of your application function correctly when combined.

Unlike unit tests, which test individual providers or notifiers in isolation, integration tests verify interactions between:

- Providers
- Notifiers
- Repositories
- Services
- Flutter widgets
- Navigation
- User interactions

The goal is to ensure the application behaves correctly as a whole.

---

# Why does it exist?

Passing unit tests does not guarantee that different parts of the application work together.

For example:

- A provider may return the correct data.
- A notifier may update state correctly.
- A widget may display state correctly.

But when combined, unexpected issues can still occur.

Integration tests help verify the complete flow from user interaction to UI update.

---

# Syntax

## Basic Integration Test

```dart
testWidgets('user can log in', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      child: const MyApp(),
    ),
  );

  await tester.enterText(
    find.byType(TextField),
    'alice@example.com',
  );

  await tester.tap(
    find.text('Login'),
  );

  await tester.pumpAndSettle();

  expect(
    find.text('Welcome Alice'),
    findsOneWidget,
  );
});
```

Explanation:

- `ProviderScope` initializes Riverpod.
- `pumpWidget()` builds the application.
- User interactions are simulated with the test framework.
- Assertions verify the final UI state.

---

## Using Provider Overrides

```dart
await tester.pumpWidget(
  ProviderScope(
    overrides: [
      apiProvider.overrideWithValue(
        FakeApiService(),
      ),
    ],
    child: const MyApp(),
  ),
);
```

Explanation:

- Replaces real dependencies during the test.
- Prevents real network requests.
- Produces predictable results.

---

## Verifying UI Updates

```dart
await tester.tap(
  find.text('Increment'),
);

await tester.pump();

expect(
  find.text('1'),
  findsOneWidget,
);
```

Explanation:

- Simulates user interaction.
- Rebuilds the widget tree.
- Confirms the provider updated the UI correctly.

---

# Mental Model

Integration tests validate the complete application flow.

```text
User Action
      │
      ▼
Widget
      │
      ▼
Provider
      │
      ▼
Repository
      │
      ▼
Service
      │
      ▼
Updated UI
```

Instead of testing each layer individually, the entire chain is tested together.

---

# Examples

## Counter Application

```dart
testWidgets('counter increments', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      child: const MyApp(),
    ),
  );

  await tester.tap(
    find.byIcon(Icons.add),
  );

  await tester.pump();

  expect(
    find.text('1'),
    findsOneWidget,
  );
});
```

Explanation:

- Simulates pressing the increment button.
- Verifies the notifier updated the provider.
- Confirms the UI reflects the new state.

---

## Login Flow

```dart
testWidgets('login succeeds', (tester) async {
  await tester.enterText(
    find.byType(TextField),
    'alice@example.com',
  );

  await tester.tap(
    find.text('Login'),
  );

  await tester.pumpAndSettle();

  expect(
    find.text('Dashboard'),
    findsOneWidget,
  );
});
```

Explanation:

- Tests the complete authentication flow.
- Verifies navigation and state updates.

---

## Testing with Fake Dependencies

```dart
await tester.pumpWidget(
  ProviderScope(
    overrides: [
      userRepositoryProvider.overrideWithValue(
        FakeUserRepository(),
      ),
    ],
    child: const MyApp(),
  ),
);
```

Explanation:

- Uses predictable test data.
- Keeps integration tests fast and reliable.

---

# When to Use

Use integration tests when you need to verify:

- Complete user flows
- Navigation
- Authentication
- Provider interactions
- State updates across multiple screens
- Widget and provider integration

---

# When NOT to Use

Avoid integration tests when:

- Testing a single provider
- Testing one notifier method
- Verifying isolated business logic
- Testing individual widgets

Those scenarios are better suited for unit or widget tests.

---

# Best Practices

- Test real user workflows.
- Keep integration tests focused on one scenario.
- Override external dependencies with fake implementations when appropriate.
- Write unit tests for business logic and use integration tests to verify end-to-end behavior.
- Avoid relying on external APIs or databases.

---

# Common Mistakes

## Testing Everything in One Test

**Wrong**

One test that covers login, profile updates, settings, logout, and navigation.

Large tests are difficult to maintain and diagnose when failures occur.

**Correct**

Write separate integration tests for each user workflow.

---

## Depending on Real Services

**Wrong**

Using production APIs during testing.

Tests become slow, unreliable, and dependent on network availability.

**Correct**

Override providers with fake implementations.

---

## Replacing Unit Tests with Integration Tests

**Wrong**

Testing all business logic only through integration tests.

This makes failures harder to isolate.

**Correct**

Use a testing pyramid:

- Many unit tests
- Some widget tests
- Fewer integration tests

Each level validates a different part of the application.

---

# Related APIs

- `ProviderScope`
- `ProviderContainer`
- `.overrideWith`
- `.overrideWithValue`
- `ConsumerWidget`
- `NotifierProvider`

---

# Summary

Integration testing verifies that Riverpod providers, notifiers, services, and Flutter widgets work together as a complete application. By simulating real user interactions and validating end-to-end workflows, integration tests provide confidence that the entire system behaves correctly under real-world conditions.