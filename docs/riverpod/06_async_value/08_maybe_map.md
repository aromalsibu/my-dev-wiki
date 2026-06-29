# maybeMap()

`maybeMap()` is an `AsyncValue` method that lets you handle only specific `AsyncValue` state objects while providing a fallback for all other states.

---

## What is it?

`maybeMap()` is the flexible version of `map()`.

Instead of requiring callbacks for every state, it lets you specify only the state objects you care about.

Any unhandled state is automatically passed to the `orElse` callback.

Unlike `maybeWhen()`, the callbacks receive the **entire state objects**, not just the wrapped values.

---

## Why does it exist?

Sometimes you only need to inspect one particular state object.

For example:

- Handle only errors.
- Access the stack trace.
- Inspect `AsyncData`.
- Check state-specific metadata.

Using `map()` in these situations forces you to write unnecessary callbacks for every state.

`maybeMap()` keeps the code concise while still giving access to the full state objects.

---

## Syntax

```dart
user.maybeMap(
  error: (error) {
    return ErrorView(error.error);
  },
  orElse: () {
    return const SizedBox();
  },
);
```

Explanation:

- The `error` callback receives the `AsyncError` object.
- Every other state executes `orElse`.

---

## Parameters

| Parameter | Required | Receives |
|-----------|----------|----------|
| `loading` | ❌ | `AsyncLoading<T>` |
| `data` | ❌ | `AsyncData<T>` |
| `error` | ❌ | `AsyncError<T>` |
| `orElse` | ✅ | No parameters |

Only `orElse` is required.

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
     │          │          ▼
     │          │      error()
     │          │
     └──────────┴──────────┐
                            ▼
                        orElse()
```

Only the callbacks you provide are executed.

---

## Mental Model

Think of `maybeMap()` as a **selective inspector**.

```text
AsyncValue
     │
     ▼
Is it AsyncError?
     │
 ┌───┴───┐
 │       │
Yes      No
 │       │
 ▼       ▼
error() orElse()
```

You inspect only the state objects that matter.

---

## Examples

### Handle Only Errors

```dart
return user.maybeMap(
  error: (error) {
    return ErrorView(error.error);
  },
  orElse: () {
    return const SizedBox();
  },
);
```

Explanation:

- Only customizes the error state.
- Loading and data use the fallback.

---

### Handle Only Data

```dart
return user.maybeMap(
  data: (data) {
    return UserView(data.value);
  },
  orElse: () {
    return const CircularProgressIndicator();
  },
);
```

Explanation:

- Receives the full `AsyncData` object.
- Accesses the value using `data.value`.

---

### Log Stack Traces

```dart
user.maybeMap(
  error: (error) {
    report(
      error.error,
      error.stackTrace,
    );

    return const ErrorScreen();
  },
  orElse: () => const SizedBox(),
);
```

Explanation:

- Uses the `AsyncError` object to access both the exception and stack trace.

---

### Real-World Example

```dart
final profile = ref.watch(profileProvider);

return profile.maybeMap(
  error: (error) {
    return RetryPage(
      message: 'Unable to load profile.',
    );
  },
  orElse: () {
    return const ProfileContent();
  },
);
```

Explanation:

- Only the error state has custom behavior.
- All other states use the default UI.

---

## maybeMap() vs maybeWhen()

The main difference is what the callbacks receive.

### maybeWhen()

```dart
data: (user) {
  print(user.name);
}
```

Explanation:

- Receives the wrapped value.

---

### maybeMap()

```dart
data: (data) {
  print(data.value.name);
}
```

Explanation:

- Receives the `AsyncData` object.

---

## When to Use

Use `maybeMap()` when:

- You need the `AsyncError` object.
- You need the stack trace.
- You need `AsyncData` metadata.
- Only one or two state objects require special handling.

---

## When NOT to Use

Avoid `maybeMap()` when:

- You only need the wrapped value.
- You want to handle all states.
- You're writing standard UI code.

Prefer:

- `when()`
- `maybeWhen()`

for most Flutter applications.

---

## Best Practices

- Always provide a meaningful `orElse`.
- Use `when()` for complete state handling.
- Use `map()` when every state object is needed.
- Use `maybeMap()` only when partial handling simplifies the code.
- Keep callbacks lightweight.

---

## Common Mistakes

### 1. Forgetting orElse

❌ Wrong

```dart
user.maybeMap(
  error: (error) {
    return ErrorView(error.error);
  },
);
```

Why it's wrong:

- `orElse` is mandatory.
- Riverpod needs a fallback.

✔ Correct

```dart
user.maybeMap(
  error: (error) {
    return ErrorView(error.error);
  },
  orElse: () {
    return const SizedBox();
  },
);
```

---

### 2. Using maybeMap() Instead of maybeWhen()

❌ Wrong

```dart
data: (data) {
  return Text(data.value.name);
}
```

When only the value is needed.

Why it's wrong:

- The extra wrapper adds unnecessary complexity.

✔ Correct

```dart
data: (user) {
  return Text(user.name);
}
```

using `maybeWhen()`.

---

### 3. Forgetting value

❌ Wrong

```dart
data: (data) {
  print(data.name);
}
```

Why it's wrong:

- `data` is an `AsyncData`.
- The wrapped value is stored in `data.value`.

✔ Correct

```dart
data: (data) {
  print(data.value.name);
}
```

---

### 4. Returning Different Types

❌ Wrong

```dart
error: (_) => const Text('Error'),
orElse: () => 'Loading',
```

Why it's wrong:

- All callbacks must return the same type.

✔ Correct

Return a consistent type from every callback.

---

## maybeMap() vs map()

| Feature | `maybeMap()` | `map()` |
|----------|--------------|----------|
| Handles all states | ❌ | ✅ |
| `orElse` required | ✅ | ❌ |
| Optional callbacks | ✅ | ❌ |
| Receives state objects | ✅ | ✅ |
| Best for partial handling | ✅ | ❌ |
| Best for exhaustive transformations | ❌ | ✅ |

---

## Related APIs

- `map()`
- `when()`
- `maybeWhen()`
- `AsyncValue`
- `AsyncLoading`
- `AsyncData`
- `AsyncError`

---

## Summary

`maybeMap()` lets you selectively handle specific `AsyncValue` state objects while providing a fallback through `orElse`. Unlike `maybeWhen()`, its callbacks receive the full `AsyncLoading`, `AsyncData`, or `AsyncError` objects, making it useful when you need access to metadata such as `value`, `error`, or `stackTrace`. For most UI code, `when()` and `maybeWhen()` are simpler choices, while `maybeMap()` is better suited for advanced state inspection.