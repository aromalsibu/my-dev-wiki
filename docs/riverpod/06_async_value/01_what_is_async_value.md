# What is AsyncValue?

`AsyncValue<T>` is Riverpod's built-in type for representing the state of an asynchronous operation, such as loading data, returning successful results, or handling errors.

---

## What is it?

`AsyncValue<T>` is a sealed class that wraps the result of an asynchronous operation.

Instead of working directly with `Future` or `Stream`, Riverpod converts asynchronous states into an `AsyncValue`.

An `AsyncValue` can be in **one of three states**:

- **Loading** – The operation is still in progress.
- **Data** – The operation completed successfully.
- **Error** – The operation failed.

This gives your UI a single, type-safe way to handle every possible outcome of an asynchronous operation.

---

## Why does it exist?

Handling asynchronous operations manually often leads to repetitive code.

For example, using a `Future` directly usually requires:

- Loading indicators
- Error handling
- Success handling
- Try-catch blocks
- Null checks
- State variables

Without `AsyncValue`, you might end up managing multiple pieces of state:

```text
isLoading
error
data
```

`AsyncValue` combines all of these into a single object.

Instead of tracking multiple variables, you only need to work with one.

---

## Syntax

### AsyncValue from a FutureProvider

```dart
final userProvider = FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- `FutureProvider` automatically returns an `AsyncValue<User>`.
- Riverpod manages loading, success, and error states for you.

---

### Reading an AsyncValue

```dart
final user = ref.watch(userProvider);
```

Explanation:

- `user` is of type `AsyncValue<User>`.
- It may represent loading, data, or an error.

---

### Handling Every State

```dart
return user.when(
  loading: () => const CircularProgressIndicator(),
  data: (user) => Text(user.name),
  error: (error, stackTrace) {
    return Text(error.toString());
  },
);
```

Explanation:

- `loading` handles pending operations.
- `data` receives the successful value.
- `error` receives both the error and its stack trace.

---

## The Three States

### 1. Loading

```text
Request Started
      │
      ▼
 AsyncLoading
```

The operation has not completed yet.

Typical UI:

- Loading spinner
- Skeleton screen
- Progress indicator

---

### 2. Data

```text
Request Completed
       │
       ▼
 AsyncData
```

The operation completed successfully.

The wrapped value is available.

---

### 3. Error

```text
Request Failed
      │
      ▼
 AsyncError
```

The operation threw an exception.

Both the error and stack trace are preserved.

---

## Execution Flow

```text
Request Starts
      │
      ▼
AsyncLoading
      │
      ▼
Operation Completes
      │
 ┌────┴────┐
 │         │
Success   Failure
 │         │
 ▼         ▼
AsyncData AsyncError
```

Every asynchronous provider follows this lifecycle.

---

## Mental Model

Think of `AsyncValue` as a **traffic light** for asynchronous operations.

```text
          AsyncValue
               │
      ┌────────┼────────┐
      │        │        │
      ▼        ▼        ▼
 Loading     Data     Error
   🟡         🟢        🔴
```

Instead of asking:

- "Is it loading?"
- "Did it fail?"
- "Do I have data?"

You simply ask:

> "What state is this AsyncValue currently in?"

---

## Examples

### Simple FutureProvider

```dart
final userProvider = FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- Riverpod wraps the result in `AsyncValue<User>` automatically.

---

### Display User Data

```dart
final user = ref.watch(userProvider);

return user.when(
  loading: () => const CircularProgressIndicator(),
  data: (user) => Text(user.name),
  error: (error, stackTrace) {
    return Text('Failed to load user');
  },
);
```

Explanation:

- Every possible state is handled.
- No manual loading or error flags are required.

---

### StreamProvider Example

```dart
final messagesProvider =
    StreamProvider<List<Message>>((ref) {
  return repository.messages();
});
```

Explanation:

- Every stream update is wrapped in an `AsyncValue<List<Message>>`.
- The UI still handles loading, data, and error consistently.

---

### AsyncNotifier Example

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() {
    return repository.fetchUser();
  }
}
```

Explanation:

- `AsyncNotifier` also exposes its state as `AsyncValue<User>`.
- State transitions are managed automatically.

---

## When to Use

Use `AsyncValue` when:

- Fetching data from an API.
- Reading a database.
- Listening to streams.
- Performing asynchronous operations.
- Building loading and error UIs.

---

## When NOT to Use

Avoid `AsyncValue` when:

- Managing simple synchronous state.
- Storing counters, booleans, or form values.
- Working with values that don't involve asynchronous operations.

Instead, use:

- `Notifier`
- `StateProvider`
- `Provider`

---

## Best Practices

- Handle all three states in the UI.
- Prefer `when()` for exhaustive state handling.
- Display meaningful error messages.
- Avoid manual loading flags alongside `AsyncValue`.
- Let Riverpod manage asynchronous state whenever possible.

---

## Common Mistakes

### 1. Ignoring the Loading State

❌ Wrong

```dart
Text(user.value!.name);
```

Why it's wrong:

- The value may not be available yet.
- This can cause runtime exceptions.

✔ Correct

```dart
user.when(
  loading: () => const CircularProgressIndicator(),
  data: (user) => Text(user.name),
  error: (error, stackTrace) {
    return Text(error.toString());
  },
);
```

---

### 2. Using Try-Catch in the UI

❌ Wrong

```dart
try {
  final user = ref.watch(userProvider);
} catch (_) {}
```

Why it's wrong:

- `AsyncValue` already captures asynchronous errors.
- UI code should react to the `error` state instead.

✔ Correct

Handle errors using `when()` or `map()`.

---

### 3. Managing Loading Flags Manually

❌ Wrong

```dart
bool isLoading = true;
```

Why it's wrong:

- `AsyncValue` already tracks loading automatically.

✔ Correct

Use:

```dart
user.isLoading
```

or

```dart
user.when(...)
```

---

### 4. Assuming AsyncValue Contains Data

❌ Wrong

```dart
final user = ref.watch(userProvider);

print(user.value!.name);
```

Why it's wrong:

- The provider may still be loading or may have failed.

✔ Correct

Check the current state before accessing the value.

---

## Related APIs

- `AsyncLoading`
- `AsyncData`
- `AsyncError`
- `when()`
- `maybeWhen()`
- `map()`
- `maybeMap()`
- `guard()`
- `FutureProvider`
- `StreamProvider`
- `AsyncNotifier`

---

## Summary

`AsyncValue<T>` is Riverpod's unified representation of asynchronous state. It wraps loading, successful data, and errors into a single type, allowing you to build predictable, type-safe UIs without manually managing loading flags or error states. It is automatically used by `FutureProvider`, `StreamProvider`, and `AsyncNotifier`, making asynchronous programming simpler and more consistent throughout a Riverpod application.