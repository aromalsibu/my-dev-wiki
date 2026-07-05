# Dependency Graph

A dependency graph is the network of relationships between providers, showing how they depend on and react to one another.

---

## What is it?

One of Riverpod's core features is its ability to automatically track dependencies between providers.

Whenever one provider reads another provider using `ref.watch()`, Riverpod records that relationship.

For example:

```text
userProvider
      │
      ▼
profileProvider
      │
      ▼
HomeScreen
```

Here:

- `profileProvider` depends on `userProvider`.
- `HomeScreen` depends on `profileProvider`.

Together, these relationships form the **dependency graph**.

Riverpod uses this graph to know exactly which providers and widgets need updating when data changes.

---

## Why does it exist?

Imagine manually updating every dependent object whenever state changes.

```text
User changes
      │
      ▼
Update Profile
      │
      ▼
Update Orders
      │
      ▼
Update Dashboard
      │
      ▼
Update Settings
```

This quickly becomes difficult to maintain.

Instead, Riverpod automatically builds a dependency graph.

Benefits include:

- Automatic updates
- No manual dependency tracking
- Efficient recomputation
- Minimal widget rebuilds
- Better scalability
- Predictable data flow

---

## Syntax

Riverpod builds the dependency graph automatically through `ref.watch()`.

### A Simple Dependency

```dart
final firstNameProvider = Provider((ref) => 'John');

final fullNameProvider = Provider((ref) {
  final firstName = ref.watch(firstNameProvider);

  return '$firstName Doe';
});
```

Explanation:

- `fullNameProvider` depends on `firstNameProvider`.
- `ref.watch()` creates the dependency.
- If `firstNameProvider` changes, `fullNameProvider` is recomputed.

---

### Multiple Dependencies

```dart
final firstNameProvider = Provider((ref) => 'John');
final lastNameProvider = Provider((ref) => 'Doe');

final fullNameProvider = Provider((ref) {
  final firstName = ref.watch(firstNameProvider);
  final lastName = ref.watch(lastNameProvider);

  return '$firstName $lastName';
});
```

Explanation:

- `fullNameProvider` depends on two providers.
- Changes to either provider trigger recomputation.

---

### Chained Dependencies

```dart
final userProvider = Provider((ref) => User());

final profileProvider = Provider((ref) {
  return Profile(ref.watch(userProvider));
});

final dashboardProvider = Provider((ref) {
  return Dashboard(ref.watch(profileProvider));
});
```

Explanation:

- Each provider depends on the previous one.
- Riverpod updates only the affected providers.

---

## Mental Model

Think of providers as nodes in a graph.

```text
           userProvider
                 │
        ┌────────┴────────┐
        ▼                 ▼
profileProvider     orderProvider
        │                 │
        └────────┬────────┘
                 ▼
          dashboardProvider
                 │
                 ▼
             HomeScreen
```

When `userProvider` changes:

1. `profileProvider` updates.
2. `orderProvider` updates.
3. `dashboardProvider` updates.
4. Only widgets watching `dashboardProvider` rebuild.

Riverpod automatically follows the graph.

---

## How Riverpod Tracks Dependencies

Whenever a provider executes:

1. Riverpod starts recording dependencies.
2. Every `ref.watch()` call is registered.
3. The provider becomes a child of those dependencies.
4. Future updates use this graph to trigger recomputations.

Example:

```dart
final greetingProvider = Provider((ref) {
  final language = ref.watch(languageProvider);

  return language == 'en'
      ? 'Hello'
      : 'Hola';
});
```

Riverpod records:

```text
languageProvider
        │
        ▼
greetingProvider
```

Now any change to `languageProvider` automatically updates `greetingProvider`.

---

## Examples

### Computed Provider

```dart
final priceProvider = Provider((ref) => 100);
final taxProvider = Provider((ref) => 18);

final totalProvider = Provider((ref) {
  final price = ref.watch(priceProvider);
  final tax = ref.watch(taxProvider);

  return price + tax;
});
```

Explanation:

- `totalProvider` depends on both providers.
- Changes to either value recompute the total.

---

### Repository Dependency

```dart
final apiProvider = Provider((ref) {
  return ApiService();
});

final repositoryProvider = Provider((ref) {
  return UserRepository(
    ref.watch(apiProvider),
  );
});
```

Explanation:

- The repository depends on the API service.
- If the API provider changes (for example, through an override), the repository is recreated.

---

### UI Dependency

```dart
class HomePage extends ConsumerWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final profile = ref.watch(profileProvider);

    return Text(profile.name);
  }
}
```

Explanation:

- The widget depends on `profileProvider`.
- It rebuilds only when `profileProvider` changes.

---

## Dependency Graph vs Widget Tree

A common misconception is that providers follow the widget tree.

They do **not**.

### Widget Tree

```text
MaterialApp
    │
    ▼
HomePage
    │
    ▼
ProfilePage
```

This represents the UI hierarchy.

---

### Dependency Graph

```text
userProvider
      │
      ▼
profileProvider
      │
      ▼
dashboardProvider
```

This represents data relationships.

The two structures are independent.

---

## When to Use

The dependency graph is built automatically whenever you:

- Use `ref.watch()`
- Compose providers
- Build computed values
- Share repositories
- Combine multiple pieces of state

You don't need to create or manage it manually.

---

## When NOT to Use

Avoid using `ref.read()` when you need a reactive dependency.

Example:

❌

```dart
final fullNameProvider = Provider((ref) {
  final firstName = ref.read(firstNameProvider);

  return '$firstName Doe';
});
```

Why it's wrong:

- `ref.read()` does not create a dependency.
- `fullNameProvider` will not update if `firstNameProvider` changes.

✔ Correct

```dart
final fullNameProvider = Provider((ref) {
  final firstName = ref.watch(firstNameProvider);

  return '$firstName Doe';
});
```

---

## Best Practices

- Use `ref.watch()` for reactive dependencies.
- Keep dependency chains simple and focused.
- Compose providers instead of creating large providers.
- Avoid circular dependencies.
- Prefer many small providers over one large provider.

---

## Common Mistakes

### 1. Using `ref.read()` Instead of `ref.watch()`

❌ Wrong

```dart
final totalProvider = Provider((ref) {
  final price = ref.read(priceProvider);

  return price + 10;
});
```

Why it's wrong:

- Changes to `priceProvider` are ignored.

✔ Correct

```dart
final totalProvider = Provider((ref) {
  final price = ref.watch(priceProvider);

  return price + 10;
});
```

---

### 2. Circular Dependencies

❌ Wrong

```dart
final providerA = Provider((ref) {
  return ref.watch(providerB);
});

final providerB = Provider((ref) {
  return ref.watch(providerA);
});
```

Why it's wrong:

- Creates an infinite dependency cycle.
- Riverpod throws an error.

✔ Correct

Restructure providers so dependencies flow in one direction.

---

### 3. One Giant Provider

❌ Wrong

```text
AppProvider
 ├── Authentication
 ├── Users
 ├── Orders
 ├── Products
 ├── Settings
```

Why it's wrong:

- Hard to maintain.
- Causes unnecessary recomputation.

✔ Correct

```text
authProvider

userProvider

orderProvider

settingsProvider
```

Compose them where needed.

---

## Related APIs

- Provider
- ref.watch()
- ref.read()
- ref.listen()
- ProviderContainer
- State Invalidation
- Caching
- select()

---

## Summary

Riverpod automatically builds a dependency graph whenever providers use `ref.watch()`. This graph allows Riverpod to track relationships between providers, efficiently recompute only affected providers, and rebuild only the widgets that depend on updated data. By leveraging the dependency graph, Riverpod delivers predictable, reactive, and highly optimized state management.