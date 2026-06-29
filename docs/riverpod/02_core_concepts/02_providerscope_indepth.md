# ProviderScope

`ProviderScope` is the Flutter widget that creates and exposes a `ProviderContainer` to the widget tree.

> **Note:** Although `ProviderScope` was briefly introduced in the **Getting Started** section, this page covers it in detail from an architectural perspective.

---

## What is it?

`ProviderScope` is the root widget that enables Riverpod in a Flutter application.

It is responsible for:

- Creating a `ProviderContainer`
- Exposing that container to descendant widgets
- Managing provider lifecycles
- Applying provider overrides
- Registering provider observers

Every widget that uses Riverpod must be inside a `ProviderScope`.

Unlike `ProviderContainer`, which is a Dart object, `ProviderScope` is a Flutter widget.

---

## Why does it exist?

Flutter widgets need a way to access Riverpod's provider container.

Instead of manually passing a `ProviderContainer` throughout the widget tree, Riverpod provides `ProviderScope`.

It acts as the bridge between:

- Flutter's widget tree
- Riverpod's provider system

This provides several benefits:

- Easy provider access anywhere in the widget tree
- Automatic lifecycle management
- Dependency injection
- Provider overrides
- State isolation
- Better testing support

Without `ProviderScope`, widgets cannot access providers using `WidgetRef`.

---

## Syntax

### Basic Usage

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
- All descendant widgets can access providers.
- This is the standard setup for every Riverpod application.

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

- `observers` monitor provider events.
- Useful for debugging, analytics, and logging.

---

### Overriding Providers

```dart
ProviderScope(
  overrides: [
    apiProvider.overrideWithValue(
      MockApiService(),
    ),
  ],
  child: const MyApp(),
)
```

Explanation:

- `overrides` replace providers within this scope.
- Child widgets use the overridden implementation.

---

### Nested ProviderScopes

```dart
ProviderScope(
  child: HomePage(
    child: ProviderScope(
      overrides: [
        themeProvider.overrideWithValue(
          DarkTheme(),
        ),
      ],
      child: SettingsPage(),
    ),
  ),
)
```

Explanation:

- Creates a second provider container.
- Only widgets inside the nested scope see the overridden provider.

---

## Mental Model

Think of `ProviderScope` as the **root of a provider tree**.

```text
                ProviderScope
                      │
          Creates ProviderContainer
                      │
      ┌───────────────┼───────────────┐
      │               │               │
 Provider A      Provider B      Provider C
      │               │               │
      └───────────────┼───────────────┘
                      │
                Flutter Widgets
```

Every widget accesses providers through the nearest `ProviderScope`.

---

## Examples

### Entire Application

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

- One `ProviderScope` serves the entire application.
- This is the recommended architecture for most apps.

---

### Feature-specific Override

```dart
ProviderScope(
  overrides: [
    apiProvider.overrideWithValue(
      FakeApiService(),
    ),
  ],
  child: const AdminScreen(),
)
```

Explanation:

- Only `AdminScreen` and its descendants use `FakeApiService`.
- The rest of the application continues using the original provider.

---

### Widget Testing

```dart
await tester.pumpWidget(
  ProviderScope(
    overrides: [
      repositoryProvider.overrideWithValue(
        MockRepository(),
      ),
    ],
    child: const MyApp(),
  ),
);
```

Explanation:

- Tests use mocked dependencies without modifying production code.
- Each test has its own isolated provider scope.

---

## When to Use

Use `ProviderScope`:

- Once at the root of every Flutter application.
- To override providers.
- During widget testing.
- To isolate state between different parts of the app.
- To register provider observers.

---

## When NOT to Use

Avoid:

- Wrapping every page with a new `ProviderScope`.
- Creating nested scopes without a reason.
- Using `ProviderScope` for local widget state.

One root `ProviderScope` is sufficient for most applications.

---

## Best Practices

- Create a single root `ProviderScope`.
- Use nested scopes only for overrides or state isolation.
- Register observers only when needed.
- Keep overrides close to the feature or test that requires them.
- Let `ProviderScope` manage the `ProviderContainer`; don't create one manually in widgets.

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

- No `ProviderContainer` exists.
- Providers cannot be accessed.

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

### 2. Creating Unnecessary Nested Scopes

❌ Wrong

```dart
ProviderScope(
  child: ProviderScope(
    child: HomePage(),
  ),
)
```

Why it's wrong:

- Creates multiple provider containers.
- Adds unnecessary complexity.

✔ Correct

```dart
ProviderScope(
  child: HomePage(),
)
```

---

### 3. Using ProviderScope Instead of Overrides

❌ Wrong

```dart
ProviderScope(
  child: LoginScreen(),
)

ProviderScope(
  child: Dashboard(),
)
```

Why it's wrong:

- Creates isolated state unintentionally.
- Shared providers are duplicated.

✔ Correct

```dart
ProviderScope(
  overrides: [
    authProvider.overrideWithValue(
      FakeAuthService(),
    ),
  ],
  child: LoginScreen(),
)
```

Create a new scope only when you need different provider behavior.

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

`ProviderScope` is the Flutter entry point for Riverpod. It creates and exposes a `ProviderContainer`, allowing widgets to access providers, while also managing provider lifecycles, overrides, observers, and scoped state. Most applications require only one root `ProviderScope`, with additional scopes used only for testing, overrides, or state isolation.