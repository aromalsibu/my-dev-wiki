# AsyncValue APIs

A quick reference for the most commonly used APIs on `AsyncValue<T>`.

---

# What is it?

`AsyncValue<T>` is Riverpod's representation of asynchronous state.

Instead of exposing only the final result, it represents the entire lifecycle of an asynchronous operation:

- Loading
- Success
- Error

This makes it easy to build robust UIs that correctly handle every state.

---

# Why does it exist?

Without `AsyncValue`, developers typically need multiple variables such as:

- `isLoading`
- `data`
- `error`

`AsyncValue` combines all of these into a single type, making asynchronous state easier to manage and harder to misuse.

---

# AsyncValue States

| State | Description |
|--------|-------------|
| `AsyncLoading` | The operation is in progress |
| `AsyncData<T>` | Data was loaded successfully |
| `AsyncError<T>` | The operation failed |

---

# Common APIs

| API | Purpose |
|------|---------|
| `when()` | Handle loading, data, and error |
| `maybeWhen()` | Handle selected states |
| `map()` | Transform each state object |
| `maybeMap()` | Transform selected state objects |
| `guard()` | Convert exceptions into `AsyncError` |
| `value` | Read the data if available |
| `hasValue` | Check whether data exists |
| `hasError` | Check whether an error exists |
| `isLoading` | Check whether loading is in progress |
| `error` | Read the current error |
| `stackTrace` | Read the captured stack trace |

---

# Build UI

## `when()`

```dart
return user.when(
  data: UserView.new,
  loading: LoadingView.new,
  error: ErrorView.new,
);
```

Explanation:

- Handles every possible asynchronous state.
- The preferred API for building UI.

---

## `maybeWhen()`

```dart
return user.maybeWhen(
  data: UserView.new,
  orElse: LoadingView.new,
);
```

Explanation:

- Handles only selected states.
- Falls back to `orElse()` for all others.

---

# Transform States

## `map()`

```dart
final widget = user.map(
  data: (_) => const UserView(),
  loading: (_) => const LoadingView(),
  error: (_) => const ErrorView(),
);
```

Explanation:

- Maps each concrete `AsyncValue` subtype.
- Useful when you need access to the state object itself.

---

## `maybeMap()`

```dart
final widget = user.maybeMap(
  error: (_) => const ErrorView(),
  orElse: () => const LoadingView(),
);
```

Explanation:

- Handles only selected state objects.
- Uses `orElse()` for the remaining cases.

---

# Error Handling

## `AsyncValue.guard()`

```dart
state = await AsyncValue.guard(() async {
  return repository.fetchUser();
});
```

Explanation:

- Executes asynchronous code.
- Converts thrown exceptions into `AsyncError`.
- Eliminates repetitive `try`/`catch` blocks.

---

# Read State

## `value`

```dart
final userData = user.value;
```

Explanation:

- Returns the loaded value if available.
- Returns `null` while loading or after an error.

---

## `hasValue`

```dart
if (user.hasValue) {
  // Data is available
}
```

Explanation:

- Indicates whether the asynchronous operation completed successfully.

---

## `hasError`

```dart
if (user.hasError) {
  // Handle error
}
```

Explanation:

- Indicates whether the provider is currently in an error state.

---

## `isLoading`

```dart
if (user.isLoading) {
  // Show loading UI
}
```

Explanation:

- Indicates whether the operation is still running.

---

## `error`

```dart
final exception = user.error;
```

Explanation:

- Returns the captured exception, if one exists.

---

## `stackTrace`

```dart
final trace = user.stackTrace;
```

Explanation:

- Returns the stack trace associated with the error.

---

# Mental Model

```text
             AsyncValue
                  │
      ┌───────────┼───────────┐
      │           │           │
      ▼           ▼           ▼
 Loading       Data        Error
      │           │           │
      └───────────┼───────────┘
                  ▼
               when()
```

`AsyncValue` always represents one of these three states.

---

# Choosing the Right API

| Situation | API |
|-----------|-----|
| Build complete async UI | `when()` |
| Handle only selected states | `maybeWhen()` |
| Transform state objects | `map()` |
| Transform selected states | `maybeMap()` |
| Simplify async error handling | `guard()` |
| Read loaded value | `value` |
| Check for data | `hasValue` |
| Check for errors | `hasError` |
| Check loading | `isLoading` |
| Access exception | `error` |
| Access stack trace | `stackTrace` |

---

# Best Practices

- Prefer `when()` when building UI.
- Use `guard()` instead of repetitive `try`/`catch` blocks.
- Handle loading and error states explicitly.
- Avoid using `.value` without checking the state first.
- Keep asynchronous business logic inside `AsyncNotifier`.

---

# Related APIs

- `AsyncValue`
- `AsyncData`
- `AsyncLoading`
- `AsyncError`
- `AsyncNotifier`
- `FutureProvider`
- `StreamProvider`

---

# Summary

`AsyncValue` provides a unified way to represent asynchronous state in Riverpod. Its most commonly used APIs are `when()` for building UI, `guard()` for error handling, and helper properties such as `hasValue`, `hasError`, and `isLoading` for inspecting the current state. Mastering these APIs makes asynchronous code cleaner, safer, and easier to maintain.