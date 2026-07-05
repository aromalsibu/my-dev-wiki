# ProviderContainer

An isolated container that stores and manages providers outside the Flutter widget tree.

---

# What is it?

`ProviderContainer` is the core object responsible for creating, storing, and managing providers.

While `ProviderScope` is used inside Flutter applications, it internally owns a `ProviderContainer`.

You typically interact with `ProviderContainer` when:

- Writing unit tests
- Running providers outside Flutter
- Creating isolated provider environments
- Overriding providers for testing

In most Flutter applications, you rarely create a `ProviderContainer` manually because `ProviderScope` does it for you.

---

# Why does it exist?

Providers need somewhere to live.

Every provider instance, its state, dependencies, and lifecycle are managed by a container.

Without `ProviderContainer`, Riverpod would have nowhere to:

- Store provider state
- Resolve dependencies
- Cache providers
- Dispose unused providers
- Apply overrides

`ProviderScope` provides this automatically in Flutter, while `ProviderContainer` allows the same functionality outside the widget tree.

---

# Syntax

## Creating a Container

```dart
final container = ProviderContainer();
```

Explanation:

- Creates an isolated provider environment.
- No Flutter widgets are required.

---

## Reading a Provider

```dart
final container = ProviderContainer();

final value = container.read(counterProvider);
```

Explanation:

- `read()` returns the current provider value.
- The provider is created automatically if needed.

---

## Listening to a Provider

```dart
final container = ProviderContainer();

final subscription = container.listen(
  counterProvider,
  (previous, next) {
    print(next);
  },
);
```

Explanation:

- Observes provider changes.
- Unlike `read()`, the listener continues receiving updates.
- Remember to close the subscription when finished.

---

## Disposing the Container

```dart
final container = ProviderContainer();

// Use providers...

container.dispose();
```

Explanation:

- Disposes every provider managed by the container.
- Always dispose containers you create manually.

---

# Mental Model

Think of `ProviderContainer` as the **engine** that powers Riverpod.

```text
Flutter App
      │
      ▼
ProviderScope
      │
      ▼
ProviderContainer
      │
      ▼
Providers
```

Without Flutter:

```text
ProviderContainer
      │
      ▼
Providers
```

Every provider belongs to exactly one container.

---

# Examples

## Simple Example

```dart
final container = ProviderContainer();

final count = container.read(counterProvider);

print(count);
```

Explanation:

- Reads a provider without a widget.
- Useful for command-line apps or tests.

---

## Unit Test Example

```dart
test('Counter starts at zero', () {
  final container = ProviderContainer();

  expect(
    container.read(counterProvider),
    0,
  );

  container.dispose();
});
```

Explanation:

- Creates a fresh provider environment for each test.
- Tests remain isolated from one another.

---

## Using Overrides

```dart
final container = ProviderContainer(
  overrides: [
    apiProvider.overrideWithValue(
      FakeApi(),
    ),
  ],
);
```

Explanation:

- Replaces dependencies only inside this container.
- Ideal for testing and mocking.

---

# When to Use

Use `ProviderContainer` when:

- Writing unit tests
- Running providers outside Flutter
- Creating isolated provider environments
- Testing business logic
- Applying provider overrides

---

# When NOT to Use

Avoid manually creating `ProviderContainer` when:

- Building a normal Flutter application
- You already have a `ProviderScope`
- Widgets can access providers through `WidgetRef`

In Flutter, `ProviderScope` should usually manage the container for you.

---

# Best Practices

- Create a new container for each test.
- Always dispose manually created containers.
- Use overrides to inject fake dependencies.
- Keep containers isolated between tests.
- Prefer `ProviderScope` inside Flutter applications.

---

# Common Mistakes

## Forgetting to Dispose

**Wrong**

```dart
final container = ProviderContainer();
```

Never disposing the container can leave providers alive longer than necessary.

**Correct**

```dart
final container = ProviderContainer();

try {
  // Test code
} finally {
  container.dispose();
}
```

Always dispose manually created containers.

---

## Sharing One Container Across Tests

**Wrong**

```dart
final container = ProviderContainer();

test(...);

test(...);
```

State may leak between tests.

**Correct**

```dart
test('...', () {
  final container = ProviderContainer();

  // Test

  container.dispose();
});
```

Each test should have its own container.

---

## Creating a Container Inside Widgets

**Wrong**

```dart
final container = ProviderContainer();
```

Inside a Flutter widget.

**Correct**

Use `WidgetRef` or `ProviderScope` instead.

---

# Related APIs

- `ProviderScope`
- `WidgetRef`
- `ProviderObserver`
- `.overrideWith`
- `.overrideWithValue`
- `ConsumerWidget`

---

# Summary

`ProviderContainer` is the core object that manages providers, their state, and their lifecycle. It powers Riverpod behind the scenes and is most commonly used for testing, dependency injection, and running providers outside the Flutter widget tree. In Flutter applications, `ProviderScope` usually manages the container automatically.