# Error Handling

Handle failures in a consistent, predictable way so your application remains stable and provides a good user experience.

---

# What is it?

Error handling is the process of detecting, reporting, and recovering from failures that occur while your application is running.

Common sources of errors include:

- Network failures
- API errors
- Database exceptions
- Invalid user input
- Permission issues
- Unexpected runtime exceptions

Riverpod does not prescribe a single error handling strategy, but it provides APIs such as `AsyncValue` and `AsyncValue.guard()` that make handling asynchronous errors much simpler.

---

# Why does it exist?

Applications should never assume that every operation succeeds.

Without proper error handling, an application may:

- Crash unexpectedly
- Show blank screens
- Display stale data
- Leave users without feedback
- Become difficult to debug

A consistent strategy makes failures predictable and easier to recover from.

---

# Syntax

## Returning an AsyncValue

```dart
final userProvider = FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- Any exception thrown becomes an `AsyncError`.
- Consumers can handle loading, success, and error states.

---

## Handling errors with `when()`

```dart
final user = ref.watch(userProvider);

return user.when(
  data: UserView.new,
  loading: () => const CircularProgressIndicator(),
  error: (error, stackTrace) {
    return Text(error.toString());
  },
);
```

Explanation:

- `when()` forces every asynchronous state to be handled.
- Users always receive appropriate feedback.

---

## Using `AsyncValue.guard()`

```dart
Future<void> loadUser() async {
  state = const AsyncLoading();

  state = await AsyncValue.guard(() async {
    return repository.fetchUser();
  });
}
```

Explanation:

- `AsyncValue.guard()` automatically converts exceptions into `AsyncError`.
- Eliminates repetitive `try-catch` blocks.

---

## Using try-catch when needed

```dart
try {
  await repository.saveUser(user);
} catch (e) {
  logger.log(e.toString());
}
```

Explanation:

- Use `try-catch` when additional recovery or logging is required.
- Don't catch exceptions only to ignore them.

---

# Mental Model

```text
Repository
      │
      ▼
Exception
      │
      ▼
Provider
      │
      ▼
AsyncValue
      │
      ▼
UI shows Loading / Data / Error
```

Errors flow upward through the provider instead of crashing the application.

---

# Examples

## Simple Example

```dart
final weather = ref.watch(weatherProvider);

return weather.when(
  data: WeatherCard.new,
  loading: () => const CircularProgressIndicator(),
  error: (e, _) => Text(e.toString()),
);
```

Explanation:

- Every state is handled explicitly.
- The UI never enters an undefined state.

---

## Real-World Example

```dart
class LoginNotifier extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async => null;

  Future<void> login(
    String email,
    String password,
  ) async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() async {
      return repository.login(email, password);
    });
  }
}
```

Explanation:

- Loading starts immediately.
- Any exception automatically becomes an `AsyncError`.
- Successful login becomes `AsyncData`.

---

# When to Use

Handle errors whenever you:

- Call APIs
- Read databases
- Access local storage
- Authenticate users
- Upload or download files
- Perform asynchronous operations
- Execute business logic that may fail

---

# When NOT to Use

Avoid:

- Swallowing exceptions silently.
- Catching every exception at every layer.
- Using exceptions for normal application flow.

Errors should either be handled or propagated to a layer that can handle them appropriately.

---

# Best Practices

- Prefer `AsyncValue` for asynchronous state.
- Use `AsyncValue.guard()` for async operations.
- Display meaningful error messages.
- Log unexpected exceptions.
- Keep error handling consistent across the project.
- Allow repositories to throw exceptions and let providers decide how to expose them.
- Recover gracefully whenever possible.

---

# Common Mistakes

## Ignoring errors

Wrong:

```dart
await repository.fetchUser();
```

Explanation:

- If an exception occurs, the UI has no way to respond.

Correct:

```dart
state = await AsyncValue.guard(() async {
  return repository.fetchUser();
});
```

Explanation:

- Errors become part of the provider state.
- The UI can display appropriate feedback.

---

## Empty catch blocks

Wrong:

```dart
try {
  await repository.login();
} catch (_) {}
```

Explanation:

- The exception is silently discarded.
- Debugging becomes much more difficult.

Correct:

```dart
try {
  await repository.login();
} catch (e, stackTrace) {
  logger.log(e.toString());

  rethrow;
}
```

Explanation:

- Log unexpected failures or convert them into application-specific errors.
- Don't hide failures.

---

## Showing raw exception messages

Wrong:

```dart
Text(error.toString())
```

Explanation:

- Raw exception messages are often technical and not user-friendly.

Correct:

```dart
Text('Unable to load your profile. Please try again.')
```

Explanation:

- Present clear, actionable messages to users.
- Log detailed errors separately for debugging.

---

# Related APIs

- AsyncValue
- `AsyncValue.guard()`
- FutureProvider
- AsyncNotifier
- `when()`
- `maybeWhen()`
- `map()`
- `maybeMap()`

---

# Summary

A good error handling strategy makes failures predictable instead of disruptive. Use `AsyncValue` to represent asynchronous state, `AsyncValue.guard()` to simplify exception handling, provide meaningful feedback to users, and log unexpected errors for troubleshooting. Consistent error handling results in more reliable and maintainable Riverpod applications.