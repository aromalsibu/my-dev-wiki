# when()

`when()` is an `AsyncValue` method that requires you to handle every possible asynchronous state by providing callbacks for loading, data, and error.

---

## What is it?

`when()` is the primary way to work with an `AsyncValue`.

Instead of checking whether the value is loading, successful, or failed using multiple `if` statements, `when()` lets you describe what should happen for each state in one place.

An `AsyncValue` can only be in one of three states:

- `AsyncLoading`
- `AsyncData`
- `AsyncError`

`when()` requires you to handle all three, making your code exhaustive and predictable.

---

## Why does it exist?

Without `when()`, you would typically write code like this:

```dart
if (user.isLoading) {
  return const CircularProgressIndicator();
}

if (user.hasError) {
  return Text(user.error.toString());
}

return Text(user.value!.name);
```

While this works, it has several drawbacks:

- More boilerplate
- Easier to forget a state
- Less readable
- More opportunities for runtime errors

`when()` provides a cleaner and safer alternative.

---

## Syntax

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

- `loading` is called while the operation is in progress.
- `data` receives the successful value.
- `error` receives the exception and stack trace.
- Exactly one callback is executed.

---

## Parameters

| Parameter | Description |
|-----------|-------------|
| `loading` | Builds the UI while loading |
| `data` | Receives the successful value |
| `error` | Receives the exception and stack trace |

All three callbacks are required.

---

## Execution Flow

```text
        AsyncValue
             │
     ┌───────┼────────┐
     │       │        │
     ▼       ▼        ▼
Loading    Data     Error
     │       │        │
     ▼       ▼        ▼
loading() data() error()
```

Only one callback runs based on the current state.

---

## Mental Model

Think of `when()` as a **three-way switch**.

```text
            AsyncValue
                 │
      ┌──────────┼──────────┐
      │          │          │
      ▼          ▼          ▼
 Loading      Success      Failure
      │          │          │
      ▼          ▼          ▼
 loading()   data()     error()
```

Instead of asking multiple questions about the state, you let Riverpod choose the correct callback.

---

## Examples

### Basic Example

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
    return Text('Failed to load user');
  },
);
```

Explanation:

- Handles every possible state in one expression.
- The UI automatically updates as the state changes.

---

### Display a List

```dart
return products.when(
  loading: ProductGridSkeleton.new,
  data: ProductGrid.new,
  error: ErrorView.new,
);
```

Explanation:

- Constructor tear-offs keep the code concise.
- Riverpod selects the appropriate callback automatically.

---

### Custom Error UI

```dart
return user.when(
  loading: LoadingScreen.new,
  data: UserView.new,
  error: (error, stackTrace) {
    return RetryScreen(
      message: 'Unable to load user.',
    );
  },
);
```

Explanation:

- Allows customized handling for failures.
- Makes it easy to provide retry actions.

---

### Real-World Example

```dart
final profile = ref.watch(profileProvider);

return profile.when(
  loading: ProfileSkeleton.new,
  data: ProfilePage.new,
  error: (error, stackTrace) {
    return ErrorPage(
      message: 'Unable to load profile.',
    );
  },
);
```

Explanation:

- Each state has its own dedicated UI.
- Results in clean, maintainable widget code.

---

## Return Type

`when()` returns whatever your callbacks return.

For example:

```dart
Widget widget = user.when(...);
```

or

```dart
String message = asyncValue.when(...);
```

All callbacks must return the same type.

---

## When to Use

Use `when()` when:

- Building Flutter widgets.
- Handling loading, success, and error states.
- Displaying asynchronous data.
- You want exhaustive state handling.

It is the recommended way to consume `AsyncValue` in most UI code.

---

## When NOT to Use

Avoid `when()` when:

- You only care about one state.
- You want default behavior for other states.
- You're transforming states rather than consuming them.

In those cases, consider:

- `maybeWhen()`
- `map()`
- `maybeMap()`

---

## Best Practices

- Prefer `when()` for UI rendering.
- Always provide meaningful error UIs.
- Keep callbacks small and focused.
- Use constructor tear-offs when possible.
- Avoid putting business logic inside callbacks.

---

## Common Mistakes

### 1. Ignoring One of the States

❌ Wrong

```dart
if (user.hasValue) {
  return Text(user.value!.name);
}
```

Why it's wrong:

- Loading and error states are ignored.

✔ Correct

```dart
user.when(
  loading: LoadingView.new,
  data: UserView.new,
  error: ErrorView.new,
);
```

---

### 2. Force-Unwrapping value

❌ Wrong

```dart
Text(user.value!.name);
```

Why it's wrong:

- The value may not exist yet.

✔ Correct

Use the `data` callback to access the value safely.

---

### 3. Performing Heavy Business Logic

❌ Wrong

```dart
data: (user) {
  calculateLargeReport(user);
  return UserPage(user);
}
```

Why it's wrong:

- `when()` may execute multiple times as widgets rebuild.
- Expensive work should live in providers, not the UI.

✔ Correct

Keep callbacks focused on building widgets.

---

### 4. Returning Different Types

❌ Wrong

```dart
loading: () => const CircularProgressIndicator(),
data: (user) => user.name,
error: (_, __) => const Text('Error'),
```

Why it's wrong:

- The callbacks return different types.
- `when()` requires a single consistent return type.

✔ Correct

Ensure all callbacks return the same type.

---

## when() vs Manual if Statements

| Feature | `when()` | Manual `if` |
|----------|----------|-------------|
| Handles all states | ✅ | Optional |
| Exhaustive | ✅ | ❌ |
| Cleaner syntax | ✅ | ❌ |
| Type-safe | ✅ | Less safe |
| Recommended for UI | ✅ | ❌ |

---

## Related APIs

- `AsyncValue`
- `maybeWhen()`
- `map()`
- `maybeMap()`
- `AsyncLoading`
- `AsyncData`
- `AsyncError`

---

## Summary

`when()` is the primary way to consume an `AsyncValue`. It requires you to handle the loading, data, and error states explicitly, resulting in clean, exhaustive, and type-safe code. For most Flutter UIs using Riverpod, `when()` is the recommended approach because it keeps asynchronous state handling centralized, readable, and easy to maintain.