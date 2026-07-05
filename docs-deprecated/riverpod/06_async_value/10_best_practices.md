# Best Practices

Best practices for `AsyncValue` help you build asynchronous applications that are predictable, maintainable, and user-friendly by handling loading, data, and error states consistently.

---

## What are AsyncValue Best Practices?

`AsyncValue` is designed to eliminate many common problems when working with asynchronous code.

However, using it effectively requires following a few recommended practices.

Good practices help you:

- Write less boilerplate.
- Avoid runtime errors.
- Improve user experience.
- Make code easier to maintain.
- Handle asynchronous states consistently across your application.

---

## Why do they exist?

As applications grow, asynchronous operations become more common.

Without consistent patterns, you may encounter:

- Forgotten loading states.
- Unhandled exceptions.
- Duplicate error handling.
- Inconsistent UI.
- Difficult-to-maintain code.

Following best practices ensures your application behaves predictably.

---

## Best Practices

### 1. Prefer `when()` for UI

Use `when()` whenever you need to display loading, success, and error states.

```dart
return user.when(
  loading: LoadingView.new,
  data: UserView.new,
  error: ErrorView.new,
);
```

Explanation:

- Handles every state explicitly.
- Produces clean and readable UI code.

---

### 2. Use `guard()` Inside AsyncNotifier

Instead of writing manual `try-catch` blocks, use `AsyncValue.guard()`.

```dart
state = const AsyncLoading();

state = await AsyncValue.guard(() async {
  return repository.fetchUser();
});
```

Explanation:

- Reduces boilerplate.
- Automatically creates `AsyncData` or `AsyncError`.

---

### 3. Always Handle Errors

Never ignore the error state.

```dart
error: (error, stackTrace) {
  return const ErrorScreen();
},
```

Explanation:

- Users should always know when something went wrong.
- Avoid blank screens or silent failures.

---

### 4. Show Meaningful Loading UI

Instead of empty widgets, provide useful loading feedback.

```dart
loading: () {
  return const CircularProgressIndicator();
},
```

Explanation:

- Improves perceived performance.
- Reassures users that work is in progress.

For lists and complex layouts, consider skeleton screens instead of simple spinners.

---

### 5. Keep Business Logic Out of Widgets

Perform asynchronous work inside providers or notifiers.

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() {
    return repository.fetchUser();
  }
}
```

Explanation:

- Widgets should focus on displaying state.
- Providers should manage state and business logic.

---

### 6. Prefer User-Friendly Error Messages

Avoid exposing raw exceptions.

❌ Instead of

```dart
Text(error.toString());
```

✔ Prefer

```dart
Text(
  'Something went wrong. Please try again.',
);
```

Explanation:

- Technical exception messages are often confusing.
- Log detailed errors separately for debugging.

---

### 7. Use Constructor Tear-Offs When Appropriate

```dart
return user.when(
  loading: LoadingView.new,
  data: UserView.new,
  error: ErrorView.new,
);
```

Explanation:

- Keeps code concise.
- Improves readability.

---

### 8. Keep Callbacks Lightweight

Callbacks should primarily build UI.

```dart
data: (user) {
  return UserView(user);
},
```

Explanation:

- Avoid expensive calculations.
- Widgets may rebuild frequently.

---

### 9. Refresh Instead of Recreating Providers

Use Riverpod's refresh APIs when new data is required.

```dart
ref.refresh(userProvider);
```

Explanation:

- Keeps provider lifecycle predictable.
- Avoids unnecessary provider recreation.

---

### 10. Let Riverpod Manage State

Avoid creating duplicate loading or error variables.

❌ Wrong

```dart
bool isLoading = false;

String? error;
```

✔ Correct

```dart
AsyncValue<User>
```

Explanation:

- `AsyncValue` already represents loading, success, and failure.

---

## Recommended Flow

```text
User Action
      │
      ▼
AsyncLoading
      │
      ▼
AsyncValue.guard()
      │
 ┌────┴─────┐
 │          │
 ▼          ▼
Data      Error
      │
      ▼
UI uses when()
```

This is the standard pattern for most Riverpod applications.

---

## Common Mistakes

### 1. Ignoring Loading

❌ Wrong

```dart
Text(user.value!.name);
```

Why it's wrong:

- The value may not be available yet.

✔ Correct

Use `when()` to handle loading first.

---

### 2. Ignoring Errors

❌ Wrong

```dart
if (user.hasValue) {
  return UserView(user.value!);
}

return const SizedBox();
```

Why it's wrong:

- Errors are silently discarded.

✔ Correct

Always provide an error callback.

---

### 3. Using Try-Catch Everywhere

❌ Wrong

```dart
try {
  final user = await repository.fetchUser();
} catch (_) {}
```

Why it's wrong:

- Repetitive and harder to maintain.

✔ Correct

Use `AsyncValue.guard()` for most asynchronous state updates.

---

### 4. Performing Business Logic in when()

❌ Wrong

```dart
data: (user) {
  generateLargeReport(user);

  return UserView(user);
}
```

Why it's wrong:

- `when()` may execute multiple times due to widget rebuilds.
- Heavy work belongs in providers or notifiers.

✔ Correct

Keep callbacks focused on rendering the UI.

---

### 5. Showing Technical Error Messages

❌ Wrong

```dart
Text(error.toString());
```

Why it's wrong:

- Users may not understand exception details.

✔ Correct

Display friendly messages and log the original error separately.

---

## Checklist

Before shipping an asynchronous feature, verify that:

- ✅ Every `AsyncValue` handles loading.
- ✅ Every `AsyncValue` handles errors.
- ✅ `when()` is used for UI rendering where appropriate.
- ✅ `AsyncValue.guard()` is used inside `AsyncNotifier`s.
- ✅ Loading indicators are meaningful.
- ✅ Error messages are user-friendly.
- ✅ Business logic stays inside providers or notifiers.
- ✅ Callbacks remain lightweight.
- ✅ Duplicate loading/error flags are avoided.
- ✅ State transitions remain predictable.

---

## Related APIs

- `AsyncValue`
- `AsyncLoading`
- `AsyncData`
- `AsyncError`
- `when()`
- `maybeWhen()`
- `map()`
- `maybeMap()`
- `guard()`
- `AsyncNotifier`

---

## Summary

Using `AsyncValue` effectively is about handling asynchronous state consistently. Prefer `when()` for UI rendering, `AsyncValue.guard()` for asynchronous operations inside notifiers, always provide meaningful loading and error states, and keep business logic inside providers rather than widgets. Following these practices results in cleaner, safer, and more maintainable Riverpod applications.