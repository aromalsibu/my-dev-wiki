# Error

`AsyncError<T>` represents the failed state of an `AsyncValue`, containing the exception and stack trace produced by an unsuccessful asynchronous operation.

---

## What is it?

`AsyncError<T>` is one of the three possible states of an `AsyncValue`.

It indicates that an asynchronous operation failed before producing a successful result.

Common reasons include:

- Network failures
- API errors
- Database exceptions
- File system errors
- Parsing failures
- Unexpected exceptions

Unlike throwing exceptions directly into the UI, Riverpod captures the failure inside an `AsyncError`, allowing your application to handle errors declaratively.

---

## Why does it exist?

Without `AsyncError`, every asynchronous operation would require manual error handling using `try-catch` blocks.

For example:

```dart
try {
  final user = await repository.fetchUser();
} catch (e) {
  // Handle error
}
```

In a UI, this quickly becomes repetitive.

`AsyncError` centralizes error handling into the `AsyncValue` state, allowing the UI to react consistently to failures without surrounding every asynchronous operation with `try-catch`.

---

## How Error Works

When an asynchronous operation throws an exception, Riverpod transitions from `AsyncLoading` to `AsyncError`.

```text
Request Starts
      │
      ▼
AsyncLoading
      │
      ▼
Exception Thrown
      │
      ▼
 AsyncError
```

The error and its stack trace are preserved for debugging and reporting.

---

## Syntax

### FutureProvider

```dart
final userProvider = FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- If `fetchUser()` throws an exception, Riverpod automatically creates an `AsyncError<User>`.

---

### Handling Errors with when()

```dart
final user = ref.watch(userProvider);

return user.when(
  loading: () => const CircularProgressIndicator(),
  data: (user) => Text(user.name),
  error: (error, stackTrace) {
    return Text(error.toString());
  },
);
```

Explanation:

- The `error` callback receives both the exception and its stack trace.
- This ensures all failures are handled in one place.

---

### Checking hasError

```dart
if (user.hasError) {
  print(user.error);
}
```

Explanation:

- `hasError` indicates whether the provider is currently in the error state.
- `error` returns the captured exception.

---

## Error Lifecycle

```text
AsyncLoading
      │
      ▼
Exception
      │
      ▼
 AsyncError
```

The provider remains in the error state until it is refreshed, invalidated, or rebuilt.

---

## Mental Model

Think of `AsyncError` as a **delivery failure**.

```text
Order Placed
      │
      ▼
Shipping
      │
      ▼
Delivery Failed
```

The order didn't arrive successfully, but you receive information explaining what went wrong.

Similarly, `AsyncError` preserves the failure details instead of crashing the application.

---

## Examples

### Display an Error Message

```dart
user.when(
  loading: () => const CircularProgressIndicator(),
  data: (user) => Text(user.name),
  error: (error, stackTrace) {
    return Text('Failed to load user');
  },
);
```

Explanation:

- Displays a user-friendly error message instead of crashing.

---

### Log the Exception

```dart
user.when(
  loading: () => const CircularProgressIndicator(),
  data: (_) => const SizedBox(),
  error: (error, stackTrace) {
    debugPrint(error.toString());
    return const ErrorScreen();
  },
);
```

Explanation:

- Logs the error for debugging.
- Shows an error screen to the user.

---

### Using hasError

```dart
if (user.hasError) {
  return Text(user.error.toString());
}
```

Explanation:

- Useful for simple conditional rendering.
- `when()` is usually preferred for exhaustive handling.

---

### Real-World Example

```dart
final products = ref.watch(productsProvider);

return products.when(
  loading: ProductGridSkeleton.new,
  data: ProductGrid.new,
  error: (error, stackTrace) {
    return RetryView(
      message: 'Unable to load products.',
    );
  },
);
```

Explanation:

- Displays a retry UI when the request fails.
- Improves the user experience by allowing recovery.

---

## When to Use

Handle the error state when:

- Calling REST APIs.
- Reading databases.
- Working with streams.
- Loading local files.
- Performing any asynchronous operation that may fail.

---

## When NOT to Use

Avoid using the error state for:

- Business validation (e.g., invalid form input).
- Simple synchronous conditions.
- Normal application flow.

Reserve `AsyncError` for genuine asynchronous failures.

---

## Best Practices

- Always handle the error state.
- Display user-friendly error messages.
- Log detailed exceptions for debugging.
- Preserve the stack trace when reporting errors.
- Provide retry mechanisms where appropriate.

---

## Common Mistakes

### 1. Ignoring Errors

❌ Wrong

```dart
user.when(
  loading: LoadingView.new,
  data: UserView.new,
  error: (_, __) => const SizedBox(),
);
```

Why it's wrong:

- Users receive no indication that something went wrong.

✔ Correct

Display an informative error message or retry UI.

---

### 2. Showing Raw Exceptions to Users

❌ Wrong

```dart
Text(error.toString());
```

Why it's wrong:

- Exception messages are often technical.
- They may confuse users or expose implementation details.

✔ Correct

```dart
Text('Something went wrong. Please try again.');
```

Log the original exception separately for debugging.

---

### 3. Using Try-Catch in the UI

❌ Wrong

```dart
try {
  final user = ref.watch(userProvider);
} catch (_) {}
```

Why it's wrong:

- `AsyncValue` already captures asynchronous exceptions.
- UI should react to the `error` state instead.

✔ Correct

Use the `error` callback in `when()`.

---

### 4. Forgetting the Stack Trace

❌ Wrong

```dart
error: (error, _) {
  report(error);
}
```

Why it's wrong:

- Stack traces help identify the source of failures.

✔ Correct

```dart
error: (error, stackTrace) {
  report(error, stackTrace);
}
```

---

## AsyncError vs AsyncLoading vs AsyncData

| State | Meaning | Contains Error |
|--------|---------|----------------|
| `AsyncLoading` | Operation in progress | ❌ |
| `AsyncData` | Operation succeeded | ❌ |
| `AsyncError` | Operation failed | ✅ |

---

## Related APIs

- `AsyncValue`
- `AsyncLoading`
- `AsyncData`
- `when()`
- `maybeWhen()`
- `map()`
- `guard()`
- `FutureProvider`
- `StreamProvider`
- `AsyncNotifier`

---

## Summary

`AsyncError<T>` represents a failed asynchronous operation. Instead of throwing exceptions into the UI, Riverpod captures the error and its stack trace inside an `AsyncValue`, allowing applications to handle failures consistently. By responding to the error state with meaningful messages, logging, and retry options, you can build more resilient and user-friendly applications.