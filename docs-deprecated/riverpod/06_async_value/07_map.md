# map()

`map()` is an `AsyncValue` method that transforms each concrete `AsyncValue` state (`AsyncLoading`, `AsyncData`, or `AsyncError`) into another value by giving you access to the state objects themselves.

---

## What is it?

`map()` is similar to `when()`, but instead of giving you the wrapped data or error, it gives you the actual `AsyncValue` state objects.

For example:

- `AsyncLoading<T>`
- `AsyncData<T>`
- `AsyncError<T>`

This is useful when you need information about the state object itself, rather than just the contained value.

---

## Why does it exist?

Most of the time, `when()` is enough because you only care about:

- Loading
- Data
- Error

However, there are situations where you need access to the actual state object.

For example:

- Reading `value`
- Inspecting `stackTrace`
- Checking metadata
- Transforming an `AsyncValue` into another object

`map()` provides direct access to these state objects.

---

## Syntax

```dart
user.map(
  loading: (loading) {
    return const CircularProgressIndicator();
  },
  data: (data) {
    return Text(data.value.name);
  },
  error: (error) {
    return Text(error.error.toString());
  },
);
```

Explanation:

- `loading` receives an `AsyncLoading<User>`.
- `data` receives an `AsyncData<User>`.
- `error` receives an `AsyncError<User>`.
- You can access properties directly from each state object.

---

## Parameters

| Callback | Receives |
|----------|----------|
| `loading` | `AsyncLoading<T>` |
| `data` | `AsyncData<T>` |
| `error` | `AsyncError<T>` |

All callbacks are required.

---

## Execution Flow

```text
           AsyncValue
                │
     ┌──────────┼──────────┐
     │          │          │
     ▼          ▼          ▼
Loading      Data       Error
     │          │          │
     ▼          ▼          ▼
loading()  data()    error()
```

Exactly one callback is executed based on the current state.

---

## Mental Model

Think of `map()` as working with the **wrapper**, not just its contents.

```text
when()
   │
   ▼
Gets the value

map()
   │
   ▼
Gets the entire AsyncValue object
```

This provides more information and greater flexibility.

---

## Examples

### Basic Example

```dart
return user.map(
  loading: (_) {
    return const CircularProgressIndicator();
  },
  data: (data) {
    return Text(data.value.name);
  },
  error: (error) {
    return Text(error.error.toString());
  },
);
```

Explanation:

- `data.value` contains the successful result.
- `error.error` contains the exception.

---

### Access Stack Trace

```dart
user.map(
  loading: (_) => const SizedBox(),
  data: (_) => const SizedBox(),
  error: (error) {
    reportError(
      error.error,
      error.stackTrace,
    );

    return const ErrorScreen();
  },
);
```

Explanation:

- The full `AsyncError` object provides both the exception and its stack trace.

---

### Logging State Types

```dart
user.map(
  loading: (_) {
    debugPrint('Loading');
    return const SizedBox();
  },
  data: (_) {
    debugPrint('Success');
    return const SizedBox();
  },
  error: (_) {
    debugPrint('Failure');
    return const SizedBox();
  },
);
```

Explanation:

- Useful for debugging or analytics.

---

### Real-World Example

```dart
return profile.map(
  loading: (_) => const ProfileSkeleton(),
  data: (data) => ProfilePage(data.value),
  error: (error) {
    return ErrorPage(
      message: error.error.toString(),
    );
  },
);
```

Explanation:

- Uses the state objects directly.
- Accesses the wrapped value through `data.value`.

---

## when() vs map()

The biggest difference is what each callback receives.

### when()

```dart
data: (user) {
  print(user.name);
}
```

Explanation:

- Receives the **wrapped value**.

---

### map()

```dart
data: (data) {
  print(data.value.name);
}
```

Explanation:

- Receives the **AsyncData object**.

---

## When to Use

Use `map()` when:

- You need the `AsyncData` object.
- You need the `AsyncError` object.
- You need the stack trace.
- You want metadata from the state.
- You're transforming `AsyncValue` objects.

---

## When NOT to Use

Avoid `map()` when:

- You only need the successful value.
- You're simply building loading/data/error UIs.

In most UI code, prefer:

```dart
when()
```

It is shorter and easier to read.

---

## Best Practices

- Prefer `when()` for UI rendering.
- Use `map()` when you need access to state objects.
- Keep callbacks concise.
- Handle all three states.
- Avoid business logic inside callbacks.

---

## Common Mistakes

### 1. Using map() Instead of when()

❌ Wrong

```dart
user.map(
  data: (data) {
    return Text(data.value.name);
  },
  ...
);
```

Why it's wrong:

- If you only need the value, `when()` is simpler.

✔ Correct

```dart
user.when(
  data: (user) {
    return Text(user.name);
  },
  ...
);
```

---

### 2. Forgetting value

❌ Wrong

```dart
data: (data) {
  return Text(data.name);
}
```

Why it's wrong:

- `data` is an `AsyncData`.
- The value is stored in `data.value`.

✔ Correct

```dart
data: (data) {
  return Text(data.value.name);
}
```

---

### 3. Ignoring Stack Trace

❌ Wrong

```dart
error: (error) {
  log(error.error);
}
```

Why it's wrong:

- Valuable debugging information is lost.

✔ Correct

```dart
error: (error) {
  log(
    error.error,
    error.stackTrace,
  );
}
```

---

### 4. Returning Different Types

❌ Wrong

```dart
loading: (_) => const CircularProgressIndicator(),
data: (_) => 'Done',
error: (_) => const Text('Error'),
```

Why it's wrong:

- Every callback must return the same type.

✔ Correct

Return a consistent type from every callback.

---

## map() vs when()

| Feature | `map()` | `when()` |
|----------|----------|----------|
| Receives wrapped value | ❌ | ✅ |
| Receives state object | ✅ | ❌ |
| Access stack trace directly | ✅ | ❌ |
| Access `AsyncData` metadata | ✅ | ❌ |
| Best for normal UI | ❌ | ✅ |
| Best for advanced state inspection | ✅ | ❌ |

---

## Related APIs

- `when()`
- `maybeWhen()`
- `maybeMap()`
- `AsyncValue`
- `AsyncLoading`
- `AsyncData`
- `AsyncError`

---

## Summary

`map()` transforms an `AsyncValue` by providing access to its concrete state objects (`AsyncLoading`, `AsyncData`, and `AsyncError`). Unlike `when()`, which exposes only the wrapped value, `map()` gives you the full state object, making it useful for advanced scenarios such as accessing stack traces, metadata, or performing state-specific transformations. For most UI code, however, `when()` remains the simpler and preferred choice.