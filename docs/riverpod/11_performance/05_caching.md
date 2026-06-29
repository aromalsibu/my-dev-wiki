# Caching

Store and reuse provider results to avoid unnecessary work and improve application performance.

---

# What is it?

Caching is the practice of keeping previously computed or fetched data so it can be reused instead of being recreated every time it is requested.

Riverpod automatically caches provider state. Once a provider is created, its state is reused by every consumer until the provider is invalidated or disposed.

This built-in caching is one of Riverpod's core features and helps reduce unnecessary computations, network requests, and database queries.

---

# Why does it exist?

Many operations are expensive, such as:

- Making HTTP requests
- Reading from a database
- Performing complex calculations
- Loading configuration files
- Parsing large JSON responses

Without caching, these operations would execute every time a widget rebuilt or another consumer requested the provider.

Caching allows multiple consumers to share the same provider state, making applications faster and more efficient.

---

# Syntax

## Automatic provider caching

```dart
final userProvider = FutureProvider<User>((ref) async {
  return fetchUser();
});
```

Explanation:

- Riverpod executes the provider only once.
- All consumers share the cached result.
- The provider runs again only after invalidation or disposal.

---

## Reading the same provider from multiple widgets

```dart
final user = ref.watch(userProvider);
```

Explanation:

- Every widget receives the same cached provider state.
- Riverpod does not execute the provider separately for each consumer.

---

## Refreshing cached data

```dart
ref.invalidate(userProvider);
```

Explanation:

- Removes the cached state.
- The next read recreates the provider.

---

## Force an immediate refresh

```dart
ref.refresh(userProvider);
```

Explanation:

- Disposes the current state.
- Immediately creates a new provider instance.
- Returns the new provider value.

---

# Mental Model

Think of a provider as a shared cache.

```text
           UserProvider
                │
        Cached User Data
      ┌──────┼──────┐
      ▼      ▼      ▼
   Widget A Widget B Widget C
```

Instead of every widget loading its own data, they all use the same cached provider state.

---

# Examples

## Simple Example

```dart
final productsProvider =
    FutureProvider<List<Product>>((ref) async {
  return api.fetchProducts();
});
```

Explanation:

- Products are fetched only once.
- Every widget watches the same cached result.

---

## Real-World Example

```dart
class ProductPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final products = ref.watch(productsProvider);

    return products.when(
      data: ProductList.new,
      loading: CircularProgressIndicator.new,
      error: (err, stack) => Text(err.toString()),
    );
  }
}
```

Explanation:

- Multiple pages or widgets can watch `productsProvider`.
- Riverpod reuses the cached data instead of making additional API requests.

---

# When to Use

Caching is useful for:

- API responses
- Database queries
- User profiles
- Settings
- Configuration
- Expensive computations
- Shared application state
- Frequently accessed data

---

# When NOT to Use

Avoid long-lived caching for:

- Temporary form state
- Short-lived UI state
- Frequently changing values that should always be fresh
- One-time event data

For these cases, consider using `.autoDispose` or invalidating the provider when appropriate.

---

# Best Practices

- Let Riverpod handle caching automatically.
- Invalidate providers when the underlying data changes.
- Cache expensive operations instead of repeating them.
- Combine caching with `.autoDispose` when appropriate.
- Avoid manually storing duplicate cached data outside providers.

---

# Common Mistakes

## Fetching data repeatedly

Wrong:

```dart
Future<User> loadUser() async {
  return fetchUser();
}
```

Explanation:

- Every call performs a new network request.
- Results are not shared.

Correct:

```dart
final userProvider =
    FutureProvider<User>((ref) async {
  return fetchUser();
});
```

Explanation:

- Riverpod caches the result.
- Every consumer shares the same data.

---

## Refreshing instead of reusing

Wrong:

```dart
ref.refresh(userProvider);
```

Explanation:

- Refreshing recreates the provider immediately.
- Avoid refreshing when the cached data is still valid.

Correct:

```dart
final user = ref.watch(userProvider);
```

Explanation:

- Reuse the cached provider whenever possible.

---

## Forgetting to invalidate stale data

Wrong:

```dart
await repository.updateProfile(profile);
```

Explanation:

- The provider cache still contains outdated data.

Correct:

```dart
await repository.updateProfile(profile);

ref.invalidate(userProvider);
```

Explanation:

- Invalidating removes the stale cache.
- The next read fetches updated data.

---

# Related APIs

- FutureProvider
- StreamProvider
- AsyncNotifierProvider
- `ref.watch()`
- `ref.refresh()`
- `ref.invalidate()`
- `.autoDispose`
- `ref.keepAlive()`

---

# Summary

Caching is a built-in feature of Riverpod that allows providers to reuse previously created state instead of recreating it for every consumer. By sharing cached data across the application, Riverpod reduces unnecessary work, improves performance, and simplifies state management. Refresh or invalidate providers only when the cached data is no longer valid.