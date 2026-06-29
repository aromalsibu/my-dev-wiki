# Provider Package → Riverpod

Migrate from the Flutter `provider` package to Riverpod by replacing `BuildContext`-based dependency lookup with provider-based dependency management.

---

# What is it?

This migration replaces the traditional `provider` package with Riverpod.

Although both libraries solve similar problems, they have different architectures:

- **Provider package** relies on `BuildContext` to access state.
- **Riverpod** removes the dependency on `BuildContext` and exposes state through `Ref` and providers.

Riverpod was designed to solve many limitations of the original `provider` package while keeping a familiar programming model.

---

# Why does it exist?

The `provider` package works well for many applications, but it has several limitations:

- Dependencies on `BuildContext`
- Providers must exist above the widget in the tree
- Runtime errors when a provider cannot be found
- More difficult testing
- Less flexible provider composition

Riverpod addresses these issues by:

- Removing `BuildContext` from provider access
- Providing compile-time safety
- Improving testing
- Supporting provider composition
- Managing provider lifecycles automatically

---

# Syntax

## Provider package

```dart
final counter = context.watch<Counter>();

Text('${counter.count}');
```

Explanation:

- `context.watch()` subscribes the widget to changes.
- The provider must exist above this widget in the widget tree.

---

## Riverpod

```dart
final counter = ref.watch(counterProvider);

Text('$counter');
```

Explanation:

- `ref.watch()` subscribes to the provider.
- No `BuildContext` lookup is required.

---

## Reading without listening

Provider package:

```dart
final counter = context.read<Counter>();
```

Explanation:

- Reads the current value without rebuilding.

Riverpod:

```dart
final counter = ref.read(counterProvider);
```

Explanation:

- Reads the provider without creating a subscription.

---

## Application setup

Provider package:

```dart
runApp(
  MultiProvider(
    providers: [
      ChangeNotifierProvider(
        create: (_) => Counter(),
      ),
    ],
    child: const MyApp(),
  ),
);
```

Explanation:

- Providers are registered using `MultiProvider`.

---

Riverpod:

```dart
runApp(
  const ProviderScope(
    child: MyApp(),
  ),
);
```

Explanation:

- `ProviderScope` is the root container for all Riverpod providers.
- Individual providers do not need to be registered here.

---

# Mental Model

Provider package:

```text
Widget
   │
BuildContext
   │
Provider Tree
   │
State
```

Riverpod:

```text
Widget
   │
WidgetRef
   │
Provider
   │
State
```

Riverpod separates state access from the widget tree, making providers easier to reuse and test.

---

# Examples

## Simple Example

Provider package:

```dart
final theme = context.watch<ThemeNotifier>();
```

Explanation:

- The widget rebuilds when the notifier changes.

Riverpod:

```dart
final theme = ref.watch(themeProvider);
```

Explanation:

- The widget watches a provider instead of a type in the widget tree.

---

## Real-World Example

Provider package:

```dart
class UserRepository {
  UserRepository(this.api);

  final ApiClient api;
}
```

```dart
Provider(
  create: (_) => ApiClient(),
),
Provider(
  create: (context) {
    return UserRepository(
      context.read<ApiClient>(),
    );
  },
)
```

Explanation:

- Dependencies are resolved using `BuildContext`.

Riverpod:

```dart
final apiProvider = Provider<ApiClient>((ref) {
  return ApiClient();
});

final repositoryProvider =
    Provider<UserRepository>((ref) {
  final api = ref.watch(apiProvider);

  return UserRepository(api);
});
```

Explanation:

- Providers depend on other providers directly.
- Dependencies are explicit and easier to test.

---

# When to Use

Migrate when you want:

- Better testability
- Compile-time safety
- Cleaner dependency injection
- Improved provider composition
- Better lifecycle management
- Modern Riverpod APIs like `Notifier` and `AsyncNotifier`

---

# When NOT to Use

Migration may not be necessary if:

- The application is small and stable.
- There are no plans for future development.
- Migration effort outweighs the benefits.

For actively maintained applications, Riverpod is generally the recommended choice.

---

# Best Practices

- Migrate feature by feature.
- Replace `context.watch()` with `ref.watch()`.
- Replace `context.read()` with `ref.read()`.
- Replace `ChangeNotifier` with `Notifier` where appropriate.
- Keep providers small and focused.
- Add tests during migration.

---

# Common Mistakes

## Trying to use `BuildContext`

Wrong:

```dart
context.watch<User>();
```

Explanation:

- Riverpod does not use `BuildContext` to access providers.

Correct:

```dart
ref.watch(userProvider);
```

Explanation:

- `WidgetRef` is the primary way to access providers in widgets.

---

## Keeping `ChangeNotifier` everywhere

Wrong:

```dart
ChangeNotifierProvider<UserNotifier>(
  create: (_) => UserNotifier(),
);
```

Explanation:

- This keeps the old Provider architecture.

Correct:

```dart
final userProvider =
    NotifierProvider<UserNotifier, User>(
      UserNotifier.new,
    );
```

Explanation:

- Prefer Riverpod's native `Notifier` APIs for new code.

---

## Registering every provider in one place

Wrong:

```dart
MultiProvider(
  providers: [
    ...
  ],
)
```

Explanation:

- Riverpod providers are declared where they belong.
- They do not require centralized registration.

Correct:

```dart
const ProviderScope(
  child: MyApp(),
);
```

Explanation:

- `ProviderScope` manages all providers automatically.

---

# Related APIs

- ProviderScope
- Provider
- Notifier
- NotifierProvider
- AsyncNotifier
- AsyncNotifierProvider
- WidgetRef
- `ref.watch()`
- `ref.read()`

---

# Summary

Migrating from the `provider` package to Riverpod primarily involves replacing `BuildContext`-based state access with `Ref`-based provider access. Riverpod offers better testability, stronger type safety, improved dependency injection, and a more flexible architecture, making it a solid choice for modern Flutter applications.