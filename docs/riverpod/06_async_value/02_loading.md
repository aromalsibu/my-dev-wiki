# Loading

`AsyncLoading<T>` represents the loading state of an `AsyncValue`, indicating that an asynchronous operation is currently in progress and no result is available yet.

---

## What is it?

`AsyncLoading` is one of the three possible states of an `AsyncValue`.

It indicates that Riverpod is still waiting for an asynchronous operation to complete.

For example:

- An API request is being sent.
- A database query is running.
- A file is being loaded.
- A stream is connecting.

During this state:

- There is no successful data yet.
- No error has occurred.
- The operation is still in progress.

---

## Why does it exist?

Without a loading state, users wouldn't know whether the application is:

- Working correctly
- Frozen
- Waiting for a response

`AsyncLoading` allows your UI to provide immediate feedback while asynchronous work is being performed.

Instead of showing a blank screen, you can display:

- Progress indicators
- Skeleton loaders
- Shimmer effects
- Placeholder content

This greatly improves the user experience.

---

## How Loading Works

Whenever an asynchronous provider starts executing, Riverpod first creates an `AsyncLoading` state.

```text
Provider Starts
       │
       ▼
 AsyncLoading
       │
       ▼
Operation Completes
       │
 ┌─────┴─────┐
 │           │
 ▼           ▼
Data       Error
```

Loading always comes before either success or failure.

---

## Syntax

### FutureProvider

```dart
final userProvider = FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- While `fetchUser()` is executing, the provider is in the `AsyncLoading` state.

---

### Display Loading UI

```dart
final user = ref.watch(userProvider);

return user.when(
  loading: () {
    return const CircularProgressIndicator();
  },
  data: (user) {
    return Text(user.name);
  },
  error: (error, stackTrace) {
    return Text(error.toString());
  },
);
```

Explanation:

- The `loading` callback is executed while waiting for the result.
- Once complete, Riverpod automatically switches to `data` or `error`.

---

### Checking isLoading

```dart
if (user.isLoading) {
  return const CircularProgressIndicator();
}
```

Explanation:

- `isLoading` returns `true` when the provider is currently loading.
- Useful for simple conditional logic.

---

## Loading Lifecycle

```text
Async Operation Starts
          │
          ▼
 AsyncLoading
          │
          ▼
Operation Finishes
      │         │
      ▼         ▼
 AsyncData  AsyncError
```

Loading is always a temporary state.

---

## Mental Model

Think of `AsyncLoading` as a **restaurant order in progress**.

```text
Order Placed
      │
      ▼
Preparing Food
      │
      ▼
Ready
```

While the food is being prepared:

- You don't have the meal yet.
- Nothing has gone wrong.
- You simply need to wait.

`AsyncLoading` represents that waiting period.

---

## Examples

### Loading Indicator

```dart
user.when(
  loading: () {
    return const CircularProgressIndicator();
  },
  data: (user) {
    return Text(user.name);
  },
  error: (error, stackTrace) {
    return Text(error.toString());
  },
);
```

Explanation:

- Displays a spinner while data is loading.

---

### Skeleton Screen

```dart
user.when(
  loading: () {
    return const UserSkeleton();
  },
  data: (user) {
    return UserCard(user);
  },
  error: (error, stackTrace) {
    return ErrorView(error);
  },
);
```

Explanation:

- Displays placeholder content until the real data arrives.

---

### Loading Overlay

```dart
if (user.isLoading) {
  return const Center(
    child: CircularProgressIndicator(),
  );
}
```

Explanation:

- Uses the `isLoading` property instead of `when()`.

---

### Real-World Example

```dart
final products = ref.watch(productsProvider);

return products.when(
  loading: () {
    return const ProductGridSkeleton();
  },
  data: ProductGrid.new,
  error: ErrorScreen.new,
);
```

Explanation:

- Users immediately see placeholder content while products are loading.

---

## When to Use

Show a loading UI when:

- Fetching API data.
- Loading local database records.
- Reading files.
- Waiting for authentication.
- Performing long-running asynchronous operations.

---

## When NOT to Use

Avoid displaying loading indicators for:

- Synchronous state.
- Very short operations that complete instantly.
- Every tiny asynchronous task (too many spinners can hurt UX).

Consider subtle loading indicators or optimistic UI where appropriate.

---

## Best Practices

- Always provide feedback during long operations.
- Use skeleton screens for lists and complex layouts.
- Keep loading UIs lightweight.
- Avoid blocking the entire interface unnecessarily.
- Design loading indicators that match the surrounding UI.

---

## Common Mistakes

### 1. Ignoring Loading State

❌ Wrong

```dart
Text(user.value!.name);
```

Why it's wrong:

- The value may not exist yet.
- This can throw an exception.

✔ Correct

```dart
user.when(
  loading: () {
    return const CircularProgressIndicator();
  },
  data: (user) {
    return Text(user.name);
  },
  error: (error, stackTrace) {
    return Text(error.toString());
  },
);
```

---

### 2. Showing a Blank Screen

❌ Wrong

```dart
if (user.isLoading) {
  return Container();
}
```

Why it's wrong:

- Users receive no indication that work is happening.

✔ Correct

Display a loading indicator or skeleton UI.

---

### 3. Using Manual Loading Flags

❌ Wrong

```dart
bool isLoading = true;
```

Why it's wrong:

- `AsyncValue` already tracks loading automatically.

✔ Correct

```dart
if (user.isLoading) {
  ...
}
```

---

### 4. Treating Loading as an Error

❌ Wrong

```dart
if (user.value == null) {
  return const Text('Error');
}
```

Why it's wrong:

- `null` may simply mean the operation is still loading.

✔ Correct

Handle loading and errors separately using `when()` or `map()`.

---

## AsyncLoading vs AsyncData vs AsyncError

| State | Meaning | Data Available | Error Available |
|--------|---------|----------------|-----------------|
| `AsyncLoading` | Operation in progress | ❌ | ❌ |
| `AsyncData` | Operation succeeded | ✅ | ❌ |
| `AsyncError` | Operation failed | ❌ | ✅ |

---

## Related APIs

- `AsyncValue`
- `AsyncData`
- `AsyncError`
- `when()`
- `maybeWhen()`
- `FutureProvider`
- `StreamProvider`
- `AsyncNotifier`

---

## Summary

`AsyncLoading` represents the period during which an asynchronous operation is still running. It allows your UI to provide meaningful feedback—such as progress indicators or skeleton screens—while waiting for data. By handling the loading state explicitly, applications become more responsive, predictable, and user-friendly.