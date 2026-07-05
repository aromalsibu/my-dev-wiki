# FutureProvider

`FutureProvider` is a provider that exposes the result of an asynchronous operation as an `AsyncValue`, making it ideal for one-time asynchronous data fetching.

---

## What is it?

`FutureProvider` is designed for asynchronous operations that produce a **single value**.

Unlike `Provider`, which returns a value immediately, `FutureProvider` waits for a `Future` to complete before exposing its result.

The exposed value is wrapped in an `AsyncValue`, allowing Riverpod to automatically manage:

- Loading state
- Success state
- Error state

Common use cases include:

- API requests
- Database queries
- Reading local files
- Loading configuration
- Fetching user data

---

## Why does it exist?

Managing asynchronous state manually often involves multiple variables.

For example:

```dart
bool isLoading = true;
User? user;
Object? error;
```

Then you need to update these variables throughout the request lifecycle.

`FutureProvider` simplifies this by automatically handling:

- Request execution
- Loading state
- Success state
- Error state
- Caching
- Sharing results across multiple consumers

This reduces boilerplate and makes asynchronous code more declarative.

---

## Syntax

### Creating a FutureProvider

```dart
final userProvider = FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- `FutureProvider<T>` manages a `Future<T>`.
- The provider begins executing when first accessed.
- The result is exposed as an `AsyncValue<User>`.

---

### Reading the Provider

```dart
class UserPage extends ConsumerWidget {
  const UserPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);

    return user.when(
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text(error.toString()),
      data: (user) => Text(user.name),
    );
  }
}
```

Explanation:

- `ref.watch()` returns an `AsyncValue<User>`.
- `when()` handles loading, success, and error states.

---

### Refreshing the Data

```dart
ref.refresh(userProvider);
```

Explanation:

- Clears the cached value.
- Executes the future again immediately.
- Returns a new `AsyncValue`.

---

### Invalidating the Provider

```dart
ref.invalidate(userProvider);
```

Explanation:

- Removes the cached result.
- The future runs again the next time the provider is accessed.

---

## Mental Model

Think of `FutureProvider` as an asynchronous pipeline.

```text
        FutureProvider
              │
              ▼
      Execute Future
              │
              ▼
        AsyncValue
     ┌──────┼──────┐
     ▼      ▼      ▼
Loading   Data   Error
```

Riverpod automatically transitions between these states as the future progresses.

---

## Examples

### Fetching a User

```dart
final userProvider = FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- Fetches user data from a repository.
- Result is cached until invalidated or refreshed.

---

### Loading Configuration

```dart
final configProvider = FutureProvider<AppConfig>((ref) async {
  return AppConfig.load();
});
```

Explanation:

- Loads configuration asynchronously.
- Multiple consumers share the same cached result.

---

### Database Query

```dart
final productsProvider =
    FutureProvider<List<Product>>((ref) async {
  return database.getProducts();
});
```

Explanation:

- Retrieves products from a database.
- Widgets rebuild automatically when the future completes.

---

### HTTP Request

```dart
final weatherProvider =
    FutureProvider<Weather>((ref) async {
  return api.fetchWeather();
});
```

Explanation:

- Performs an HTTP request.
- Automatically exposes loading, data, and error states.

---

## AsyncValue States

A `FutureProvider` always returns an `AsyncValue`.

```text
Future Starts
      │
      ▼
 AsyncLoading
      │
      ▼
 Future Completes
      │
 ┌────┴────┐
 ▼         ▼
Data     Error
```

Possible states:

- `AsyncLoading`
- `AsyncData`
- `AsyncError`

These states are discussed in detail in the **AsyncValue** section.

---

## Caching Behavior

A `FutureProvider` executes only once while its value remains cached.

```text
Widget A
     │
     ▼
FutureProvider
     ▲
     │
Widget B
```

Both widgets share:

- The same request
- The same loading state
- The same result
- The same error

Riverpod does **not** execute multiple identical requests unless the provider is refreshed or invalidated.

---

## FutureProvider vs Provider

| Feature | Provider | FutureProvider |
|---------|----------|----------------|
| Synchronous value | ✅ | ❌ |
| Asynchronous value | ❌ | ✅ |
| Returns `AsyncValue` | ❌ | ✅ |
| Loading state | ❌ | ✅ |
| Error handling | ❌ | ✅ |
| Caching | ✅ | ✅ |

---

## When to Use

Use `FutureProvider` when:

- Fetching API data
- Reading from a database
- Loading configuration
- Reading files
- Executing one-time asynchronous operations
- Displaying asynchronous data without complex business logic

---

## When NOT to Use

Avoid `FutureProvider` when:

- The data needs to be modified.
- The request should support retry or mutation logic.
- Business logic is required.
- State changes after the initial load.

Instead, use:

| Scenario | Recommended Provider |
|----------|----------------------|
| Mutable async state | `AsyncNotifierProvider` |
| Continuous updates | `StreamProvider` |
| Simple synchronous value | `Provider` |

---

## Best Practices

- Keep asynchronous logic inside repositories or services.
- Return immutable models.
- Use `when()` to handle loading, data, and error states.
- Refresh or invalidate providers when data becomes stale.
- Use `AsyncNotifierProvider` when asynchronous state needs to change after loading.

---

## Common Mistakes

### 1. Returning a Future from Provider

❌ Wrong

```dart
final userProvider = Provider((ref) async {
  return repository.fetchUser();
});
```

Why it's wrong:

- `Provider` is intended for synchronous values.
- It does not manage asynchronous state.

✔ Correct

```dart
final userProvider = FutureProvider((ref) async {
  return repository.fetchUser();
});
```

---

### 2. Ignoring Loading and Error States

❌ Wrong

```dart
final user = ref.watch(userProvider);

Text(user.value!.name);
```

Why it's wrong:

- The future may still be loading.
- It may have failed.

✔ Correct

```dart
user.when(
  loading: () => const CircularProgressIndicator(),
  error: (error, stack) => Text(error.toString()),
  data: (user) => Text(user.name),
);
```

Handle all possible states.

---

### 3. Using FutureProvider for Mutable Data

❌ Wrong

```dart
final todosProvider =
    FutureProvider<List<Todo>>((ref) async {
  return repository.fetchTodos();
});

// Later...
// Add or remove todos directly
```

Why it's wrong:

- `FutureProvider` is intended for one-time asynchronous values.
- Managing updates becomes difficult.

✔ Correct

Use `AsyncNotifierProvider` when the data needs to support create, update, or delete operations.

---

## Recommendation

> **Use `FutureProvider` for read-only asynchronous data.**
>
> If the data needs to be modified after loading (such as adding, editing, or deleting items), prefer `AsyncNotifierProvider`.

---

## Related APIs

- AsyncValue
- AsyncNotifierProvider
- Provider
- StreamProvider
- ref.watch()
- ref.refresh()
- ref.invalidate()

---

## Summary

`FutureProvider` is Riverpod's solution for one-time asynchronous operations. It executes a `Future`, exposes the result as an `AsyncValue`, and automatically manages loading, success, error handling, caching, and shared requests. It is the ideal choice for read-only asynchronous data such as API calls, database queries, and configuration loading.