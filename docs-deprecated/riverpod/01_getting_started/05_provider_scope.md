# ProviderScope

`ProviderScope` is the root container that enables Riverpod in a Flutter application and manages the lifecycle of all providers.

---

## What is it?

`ProviderScope` is a Flutter widget that creates a **ProviderContainer** and makes it available to all descendant widgets.

Every provider in your application lives inside a `ProviderScope`.

Without it, Riverpod has nowhere to store, cache, or manage providers.

Think of `ProviderScope` as the **entry point** of Riverpod.

---

## Why does it exist?

Riverpod needs a place to:

- Store provider instances
- Cache provider values
- Track provider dependencies
- Manage provider lifecycles
- Handle provider overrides
- Dispose unused providers

Instead of using global variables, Riverpod keeps all providers inside a `ProviderContainer`, which is created by `ProviderScope`.

This design provides:

- Better testability
- Scoped state
- Dependency overrides
- Automatic lifecycle management

Without `ProviderScope`, attempting to read a provider results in a runtime error because no provider container exists.

---

## Syntax

### Basic Usage

Wrap your entire application with `ProviderScope`.

```dart
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

Explanation:

- `ProviderScope` creates the root `ProviderContainer`.
- All widgets inside it can access providers.
- This is the recommended setup for most applications.

---

### Using ProviderScope with MaterialApp

```dart
void main() {
  runApp(
    ProviderScope(
      child: MaterialApp(
        home: HomePage(),
      ),
    ),
  );
}
```

Explanation:

- `MaterialApp` becomes a descendant of `ProviderScope`.
- Every screen in the app can access Riverpod providers.

---

### Adding Provider Observers

```dart
void main() {
  runApp(
    ProviderScope(
      observers: [
        MyProviderObserver(),
      ],
      child: MyApp(),
    ),
  );
}
```

Explanation:

- `observers` monitor provider creation, updates, and disposal.
- Useful for debugging and logging.

---

### Overriding Providers

```dart
runApp(
  ProviderScope(
    overrides: [
      apiProvider.overrideWithValue(MockApiService()),
    ],
    child: MyApp(),
  ),
);
```

Explanation:

- `overrides` replace providers with custom implementations.
- Commonly used for testing and dependency injection.

---

## Mental Model

Think of `ProviderScope` as the **home** for all providers.

```text
                ProviderScope
                      │
          Creates ProviderContainer
                      │
      ┌───────────────┼───────────────┐
      │               │               │
  Provider A      Provider B     Provider C
      │               │               │
      └───────────────┼───────────────┘
                      │
                    Widgets
```

Every provider is stored inside the `ProviderContainer` owned by `ProviderScope`.

Widgets don't own providers—they simply access them.

---

## Examples

### Basic Application

```dart
void main() {
  runApp(
    ProviderScope(
      child: const MyApp(),
    ),
  );
}
```

Explanation:

- This is the standard setup for every Riverpod application.
- One root `ProviderScope` is usually sufficient.

---

### Nested ProviderScope

```dart
ProviderScope(
  child: HomePage(
    child: ProviderScope(
      child: SettingsPage(),
    ),
  ),
)
```

Explanation:

- Creates a new provider container.
- Providers inside the nested scope are independent of the parent scope.
- Useful for isolating state in specific parts of an application.

---

### Overriding a Provider

```dart
final apiProvider = Provider<ApiService>((ref) {
  return ApiService();
});

ProviderScope(
  overrides: [
    apiProvider.overrideWithValue(MockApiService()),
  ],
  child: const MyApp(),
)
```

Explanation:

- Widgets receive `MockApiService` instead of `ApiService`.
- Frequently used in tests and development environments.

---

## When to Use

Use `ProviderScope`:

- At the root of every Riverpod application.
- When overriding providers.
- When writing widget tests.
- When creating isolated provider scopes.
- When adding provider observers.

---

## When NOT to Use

Do not:

- Wrap every widget with `ProviderScope`.
- Create multiple root scopes unnecessarily.
- Use `ProviderScope` for local widget state.

In most applications, a single root `ProviderScope` is enough.

---

## Best Practices

- Wrap the entire app with one root `ProviderScope`.
- Use nested scopes only when state isolation is required.
- Keep provider overrides close to where they're needed.
- Register observers only when necessary for debugging or monitoring.
- Avoid unnecessary nested scopes, as they create separate provider containers.

---

## Common Mistakes

### 1. Forgetting ProviderScope

❌ Wrong

```dart
void main() {
  runApp(MyApp());
}
```

Why it's wrong:

- Riverpod cannot create a provider container.
- Reading any provider will throw an exception.

✔ Correct

```dart
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

---

### 2. Wrapping Every Screen

❌ Wrong

```dart
ProviderScope(
  child: HomePage(),
)

ProviderScope(
  child: SettingsPage(),
)
```

Why it's wrong:

- Creates multiple independent provider containers.
- Shared state is no longer shared.

✔ Correct

```dart
ProviderScope(
  child: MaterialApp(
    home: HomePage(),
  ),
)
```

Use a single root scope unless isolation is intentional.

---

### 3. Using Nested ProviderScope Unnecessarily

❌ Wrong

```dart
ProviderScope(
  child: Scaffold(
    body: ProviderScope(
      child: SettingsWidget(),
    ),
  ),
)
```

Why it's wrong:

- Creates a second provider container without any benefit.
- Can lead to duplicate provider instances and unexpected behavior.

✔ Correct

Use nested scopes only when you need:

- Provider overrides
- State isolation
- Independent provider lifecycles

---

## Related APIs

- ProviderContainer
- Provider
- ConsumerWidget
- WidgetRef
- ProviderObserver
- overrideWith()
- overrideWithValue()

---

## Summary

`ProviderScope` is the foundation of every Riverpod application. It creates the `ProviderContainer` that stores and manages providers, enabling dependency tracking, caching, lifecycle management, and provider overrides. In most applications, a single root `ProviderScope` wrapping the entire app is all that's needed.