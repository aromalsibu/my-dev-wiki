# ProviderContainer

`ProviderContainer` is the core object that stores, creates, caches, and manages all providers in Riverpod.

---

## What is it?

`ProviderContainer` is the **engine behind Riverpod**.

Whenever a provider is read for the first time, the container:

- Creates the provider
- Stores its state
- Tracks its dependencies
- Reuses the value when needed
- Disposes it when appropriate

In Flutter applications, you rarely create a `ProviderContainer` yourself because `ProviderScope` creates one automatically.

Outside Flutter—such as in tests, command-line applications, or pure Dart—you create it manually.

---

## Why does it exist?

Imagine every provider created its own state independently.

```text
Widget A → Counter = 5

Widget B → Counter = 0

Widget C → Counter = 10
```

Each widget would have a different instance, leading to inconsistent state.

Instead, Riverpod stores provider instances inside a single `ProviderContainer`.

This allows:

- Shared provider instances
- Automatic caching
- Dependency tracking
- State invalidation
- Lifecycle management
- Provider overrides
- Easy testing

Every provider belongs to exactly one `ProviderContainer`.

---

## Syntax

### Creating a ProviderContainer

```dart
final container = ProviderContainer();
```

Explanation:

- Creates an independent Riverpod container.
- Mostly used in tests or non-Flutter applications.

---

### Reading a Provider

```dart
final greetingProvider = Provider((ref) => 'Hello');

final container = ProviderContainer();

final greeting = container.read(greetingProvider);
```

Explanation:

- `container.read()` retrieves the provider's value.
- The provider is created the first time it is read.
- Future reads return the cached value.

---

### Disposing a Container

```dart
final container = ProviderContainer();

// Use providers...

container.dispose();
```

Explanation:

- Releases all provider resources.
- Calls disposal callbacks for active providers.
- Prevents memory leaks.

---

### Overriding Providers

```dart
final container = ProviderContainer(
  overrides: [
    apiProvider.overrideWithValue(MockApiService()),
  ],
);
```

Explanation:

- Replaces a provider with a different implementation.
- Common in testing.

---

## Mental Model

Think of `ProviderContainer` as a **warehouse**.

```text
                 ProviderContainer
                        │
        ┌───────────────┼───────────────┐
        │               │               │
    Provider A      Provider B      Provider C
        │               │               │
     Cached          Cached          Cached
        │               │               │
        └───────────────┼───────────────┘
                        │
                   Consumers
```

Providers don't store themselves.

The container stores:

- Their values
- Their dependencies
- Their lifecycle
- Their cache

---

## Examples

### Reading a Provider

```dart
final appNameProvider = Provider((ref) => 'Riverpod');

void main() {
  final container = ProviderContainer();

  print(container.read(appNameProvider));
}
```

Explanation:

- The provider is lazily created.
- The container caches the value for future reads.

---

### Using ProviderContainer in Tests

```dart
void main() {
  test('Counter starts at zero', () {
    final container = ProviderContainer();

    expect(container.read(counterProvider), 0);

    container.dispose();
  });
}
```

Explanation:

- Each test gets its own isolated container.
- Tests do not share provider state.

---

### Using Overrides

```dart
final container = ProviderContainer(
  overrides: [
    userRepositoryProvider.overrideWithValue(
      MockUserRepository(),
    ),
  ],
);
```

Explanation:

- The mock implementation replaces the real provider.
- Useful for unit and integration testing.

---

## When to Use

Use `ProviderContainer` when:

- Writing unit tests
- Building command-line applications
- Using Riverpod outside Flutter
- Creating isolated provider environments
- Manually controlling provider lifecycles

In Flutter applications, `ProviderScope` usually handles this automatically.

---

## When NOT to Use

Do not manually create a `ProviderContainer`:

- Inside Flutter widgets
- Instead of `ProviderScope`
- For sharing state across the widget tree

Use `ProviderScope` in Flutter applications.

---

## Best Practices

- Let `ProviderScope` manage the container in Flutter apps.
- Dispose manually created containers when finished.
- Create a new container for each test.
- Use provider overrides instead of modifying production code.
- Avoid sharing one container across unrelated tests.

---

## Common Mistakes

### 1. Forgetting to Dispose

❌ Wrong

```dart
final container = ProviderContainer();
```

Why it's wrong:

- Resources remain allocated.
- Disposal callbacks are never called.

✔ Correct

```dart
final container = ProviderContainer();

// Use providers...

container.dispose();
```

---

### 2. Creating Multiple Containers Accidentally

❌ Wrong

```dart
final container1 = ProviderContainer();
final container2 = ProviderContainer();
```

Why it's wrong:

- Each container has its own provider instances.
- State is not shared between containers.

✔ Correct

Use one container unless isolation is intentional.

---

### 3. Using ProviderContainer Inside Widgets

❌ Wrong

```dart
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final container = ProviderContainer();

    final value = container.read(counterProvider);

    return Text('$value');
  }
}
```

Why it's wrong:

- Bypasses the app's `ProviderScope`.
- Creates a new container every rebuild.
- Breaks shared application state.

✔ Correct

```dart
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(counterProvider);

    return Text('$value');
  }
}
```

---

## Related APIs

- ProviderScope
- Provider
- WidgetRef
- ProviderObserver
- overrideWith()
- overrideWithValue()
- ref.read()
- ref.watch()

---

## Summary

`ProviderContainer` is the core runtime of Riverpod. It creates, caches, manages, and disposes providers while tracking their dependencies. In Flutter, it is created automatically by `ProviderScope`; in tests and pure Dart applications, it is created and managed manually.