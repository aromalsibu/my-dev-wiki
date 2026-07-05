# guard()

`AsyncValue.guard()` is a utility method that executes an asynchronous operation and automatically converts its result into either `AsyncData` or `AsyncError`.

---

## What is it?

`AsyncValue.guard()` is a helper that wraps an asynchronous function in a `try-catch` block for you.

Instead of manually writing error handling every time you perform an asynchronous operation, you can let Riverpod create the appropriate `AsyncValue`.

If the operation succeeds:

- An `AsyncData<T>` is returned.

If the operation throws an exception:

- An `AsyncError<T>` is returned.

This greatly simplifies asynchronous state management inside `AsyncNotifier`s.

---

## Why does it exist?

Without `guard()`, updating asynchronous state often looks like this:

```dart
state = const AsyncLoading();

try {
  final user = await repository.fetchUser();
  state = AsyncData(user);
} catch (error, stackTrace) {
  state = AsyncError(error, stackTrace);
}
```

This pattern is repetitive and easy to get wrong.

`AsyncValue.guard()` eliminates the boilerplate by automatically handling the `try-catch` logic.

---

## Syntax

```dart
state = await AsyncValue.guard(() async {
  return repository.fetchUser();
});
```

Explanation:

- The asynchronous function is executed.
- On success, Riverpod returns `AsyncData`.
- On failure, Riverpod returns `AsyncError`.
- No manual `try-catch` is required.

---

## How guard() Works

```text
Async Function
       │
       ▼
AsyncValue.guard()
       │
 ┌─────┴─────┐
 │           │
 ▼           ▼
Success    Exception
 │           │
 ▼           ▼
AsyncData AsyncError
```

The loading state is **not** created automatically. You typically set it before calling `guard()`.

---

## Mental Model

Think of `guard()` as a **protective wrapper**.

```text
Your Async Function
        │
        ▼
   guard()
        │
        ▼
Never throws
Always returns AsyncValue
```

Instead of throwing exceptions, it converts them into Riverpod state.

---

## Examples

### Basic Example

```dart
state = await AsyncValue.guard(() async {
  return repository.fetchUser();
});
```

Explanation:

- A successful request becomes `AsyncData<User>`.
- A failed request becomes `AsyncError<User>`.

---

### Inside an AsyncNotifier

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() {
    return repository.fetchUser();
  }

  Future<void> refreshUser() async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() async {
      return repository.fetchUser();
    });
  }
}
```

Explanation:

- `AsyncLoading` is set before the request starts.
- `guard()` updates the state to either `AsyncData` or `AsyncError`.

---

### Saving Data

```dart
Future<void> saveUser(User user) async {
  state = const AsyncLoading();

  state = await AsyncValue.guard(() async {
    return repository.saveUser(user);
  });
}
```

Explanation:

- Works for any asynchronous operation, not just fetching data.

---

### Real-World Example

```dart
Future<void> login(
  String email,
  String password,
) async {
  state = const AsyncLoading();

  state = await AsyncValue.guard(() async {
    return authRepository.login(
      email,
      password,
    );
  });
}
```

Explanation:

- Automatically converts login success or failure into the appropriate `AsyncValue`.

---

## What guard() Replaces

Without `guard()`:

```dart
try {
  final result = await repository.fetch();

  state = AsyncData(result);
} catch (error, stackTrace) {
  state = AsyncError(
    error,
    stackTrace,
  );
}
```

With `guard()`:

```dart
state = await AsyncValue.guard(() async {
  return repository.fetch();
});
```

Explanation:

- Cleaner.
- Less boilerplate.
- Easier to maintain.

---

## When to Use

Use `guard()` when:

- Updating an `AsyncNotifier`.
- Calling repositories.
- Making API requests.
- Reading databases.
- Saving data.
- Performing any asynchronous operation that may throw.

---

## When NOT to Use

Avoid `guard()` when:

- The operation is synchronous.
- You need custom exception handling before updating state.
- You need to recover from errors inside the operation.

In these cases, use a manual `try-catch`.

---

## Best Practices

- Set `AsyncLoading` before calling `guard()`.
- Use `guard()` for most asynchronous state updates.
- Keep the callback focused on the asynchronous operation.
- Handle errors in the UI using `when()` or `map()`.
- Avoid nesting multiple `guard()` calls.

---

## Common Mistakes

### 1. Forgetting Loading State

❌ Wrong

```dart
state = await AsyncValue.guard(() async {
  return repository.fetchUser();
});
```

Why it's wrong:

- Users won't see that the operation has started.

✔ Correct

```dart
state = const AsyncLoading();

state = await AsyncValue.guard(() async {
  return repository.fetchUser();
});
```

---

### 2. Using Manual Try-Catch Unnecessarily

❌ Wrong

```dart
try {
  final user = await repository.fetchUser();

  state = AsyncData(user);
} catch (error, stackTrace) {
  state = AsyncError(
    error,
    stackTrace,
  );
}
```

Why it's wrong:

- This duplicates functionality already provided by `guard()`.

✔ Correct

```dart
state = await AsyncValue.guard(() async {
  return repository.fetchUser();
});
```

---

### 3. Using guard() for Synchronous Code

❌ Wrong

```dart
state = await AsyncValue.guard(() async {
  return 5 + 10;
});
```

Why it's wrong:

- `guard()` is intended for asynchronous operations.

✔ Correct

Use it with `Future`-based work.

---

### 4. Catching Exceptions Inside guard()

❌ Wrong

```dart
state = await AsyncValue.guard(() async {
  try {
    return repository.fetchUser();
  } catch (_) {
    return User.empty();
  }
});
```

Why it's wrong:

- Errors are swallowed before `guard()` can convert them into `AsyncError`.

✔ Correct

Allow exceptions to propagate unless you intentionally want to recover from them.

---

## guard() vs Manual try-catch

| Feature | `guard()` | Manual `try-catch` |
|----------|-----------|--------------------|
| Creates `AsyncData` | ✅ | Manual |
| Creates `AsyncError` | ✅ | Manual |
| Handles stack trace | ✅ | Manual |
| Less boilerplate | ✅ | ❌ |
| Custom recovery logic | Limited | ✅ |

---

## Related APIs

- `AsyncValue`
- `AsyncData`
- `AsyncError`
- `AsyncLoading`
- `AsyncNotifier`
- `when()`
- `map()`

---

## Summary

`AsyncValue.guard()` is a convenience method that converts an asynchronous operation into an `AsyncValue` by automatically handling exceptions. It replaces repetitive `try-catch` blocks with concise, readable code and is especially useful inside `AsyncNotifier` methods. By combining `AsyncLoading` with `guard()`, you can manage asynchronous state transitions consistently with minimal boilerplate.