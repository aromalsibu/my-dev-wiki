# Common Performance Pitfalls

Understand the most common mistakes that lead to unnecessary rebuilds, excessive memory usage, and inefficient state management in Riverpod applications.

---

# What is it?

Performance pitfalls are patterns that make a Riverpod application slower or less efficient than necessary.

Most Riverpod applications perform well without special optimization. However, certain coding patterns can cause:

- Unnecessary widget rebuilds
- Repeated network requests
- Excessive provider recomputations
- Memory leaks
- Poor application responsiveness

Knowing these pitfalls helps you write scalable applications from the beginning.

---

# Why does it exist?

Riverpod is designed to update only the parts of an application that need to change.

Poor provider design or incorrect API usage can prevent Riverpod from taking advantage of its reactive architecture.

Understanding these common mistakes helps you:

- Reduce rebuilds
- Improve memory usage
- Avoid duplicate work
- Keep applications responsive
- Build maintainable state management

---

# Syntax

## Prefer watching only required state

```dart
final userName = ref.watch(
  userProvider.select((user) => user.name),
);
```

Explanation:

- `select()` listens only to the required property.
- The widget rebuilds only when `name` changes.

---

## Split large providers

```dart
final userProvider =
    NotifierProvider<UserNotifier, User>(
      UserNotifier.new,
    );

final cartProvider =
    NotifierProvider<CartNotifier, Cart>(
      CartNotifier.new,
    );
```

Explanation:

- Separate providers reduce unnecessary rebuilds.
- Each provider has a single responsibility.

---

## Dispose temporary providers

```dart
final searchProvider =
    FutureProvider.autoDispose<List<Product>>((ref) async {
  return repository.search();
});
```

Explanation:

- `.autoDispose` frees resources when the provider is no longer used.
- Temporary state does not remain in memory.

---

# Mental Model

Good provider architecture:

```text
Feature
   │
   ├── UserProvider
   ├── CartProvider
   ├── ThemeProvider
   └── SettingsProvider

Small updates
      │
      ▼
Small rebuilds
```

Poor provider architecture:

```text
AppProvider
    │
Everything depends on it
    │
Large rebuilds
```

Think of providers as independent building blocks rather than one central store.

---

# Examples

## Simple Example

```dart
final count = ref.watch(counterProvider);
```

Explanation:

- Only widgets using `counterProvider` rebuild when the counter changes.

---

## Real-World Example

```dart
class Dashboard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        UserHeader(),
        WeatherCard(),
        NewsFeed(),
        NotificationPanel(),
      ],
    );
  }
}
```

Explanation:

- Each child watches its own provider.
- Updating one section does not rebuild the others.

---

# When to Use

Review these practices when:

- Optimizing large applications.
- Investigating unnecessary rebuilds.
- Improving memory usage.
- Working with frequently changing state.
- Building feature-rich dashboards.
- Profiling application performance.

---

# When NOT to Use

Avoid premature optimization.

Riverpod is already efficient by default.

Measure performance before introducing complex optimizations, and prioritize readable code unless a bottleneck has been identified.

---

# Best Practices

- Keep providers focused on one responsibility.
- Build small, reusable widgets.
- Use `select()` when only part of the state is needed.
- Prefer immutable state models.
- Use `.autoDispose` for temporary state.
- Invalidate providers instead of recreating them manually.
- Cache expensive operations inside providers.
- Profile before optimizing.

---

# Common Mistakes

## Watching more state than necessary

Wrong:

```dart
final user = ref.watch(userProvider);

return Text(user.name);
```

Explanation:

- The widget rebuilds whenever any field of `User` changes.

Correct:

```dart
final name = ref.watch(
  userProvider.select((user) => user.name),
);

return Text(name);
```

Explanation:

- Only changes to `name` trigger a rebuild.

---

## Creating one massive provider

Wrong:

```dart
final appProvider =
    NotifierProvider<AppNotifier, AppState>(
      AppNotifier.new,
    );
```

Explanation:

- Unrelated state becomes tightly coupled.
- Small changes can rebuild large parts of the UI.

Correct:

```dart
final authProvider =
    NotifierProvider<AuthNotifier, AuthState>(
      AuthNotifier.new,
    );

final settingsProvider =
    NotifierProvider<SettingsNotifier, Settings>(
      SettingsNotifier.new,
    );
```

Explanation:

- Each provider owns a single concern.
- Updates remain localized.

---

## Refreshing providers unnecessarily

Wrong:

```dart
ref.refresh(productsProvider);
```

Explanation:

- Forces immediate recomputation.
- May trigger unnecessary API calls.

Correct:

```dart
final products = ref.watch(productsProvider);
```

Explanation:

- Reuse cached state until new data is actually required.

---

## Forgetting `.autoDispose`

Wrong:

```dart
final searchProvider =
    FutureProvider<List<Product>>((ref) async {
  return repository.search();
});
```

Explanation:

- Temporary search results remain alive longer than necessary.

Correct:

```dart
final searchProvider =
    FutureProvider.autoDispose<List<Product>>((ref) async {
  return repository.search();
});
```

Explanation:

- Temporary state is automatically cleaned up when unused.

---

## Performing side effects inside `build()`

Wrong:

```dart
Widget build(BuildContext context, WidgetRef ref) {
  ref.read(loggerProvider).log('Opened page');

  return const HomeView();
}
```

Explanation:

- `build()` can run many times.
- Side effects may execute repeatedly.

Correct:

```dart
ref.listen(userProvider, (previous, next) {
  logger.log('User updated');
});
```

Explanation:

- `ref.listen()` is intended for reacting to state changes without rebuilding the UI.

---

# Related APIs

- `ref.watch()`
- `ref.read()`
- `ref.listen()`
- `select()`
- `.autoDispose`
- `ref.invalidate()`
- `ref.refresh()`
- Provider
- NotifierProvider
- AsyncNotifierProvider

---

# Summary

Most Riverpod performance problems come from application architecture rather than the framework itself. By creating focused providers, minimizing rebuilds, using `select()` appropriately, leveraging automatic caching, disposing temporary state, and avoiding unnecessary recomputation, you can build Riverpod applications that remain responsive and scalable as they grow.