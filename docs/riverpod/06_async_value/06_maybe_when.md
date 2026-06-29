# maybeWhen()

`maybeWhen()` is an `AsyncValue` method that lets you handle only the asynchronous states you care about while providing a fallback for all other states.

---

## What is it?

`maybeWhen()` is a more flexible version of `when()`.

Unlike `when()`, which requires callbacks for **all three states**, `maybeWhen()` allows you to specify callbacks for only the states you want to handle.

Any unhandled state is automatically passed to the `orElse` callback.

This makes `maybeWhen()` useful when you only need to customize one or two states.

---

## Why does it exist?

Sometimes you don't need to handle every possible state.

For example:

- Show a loading indicator only.
- Handle errors specially.
- Display cached data for all non-loading states.
- Perform logic only when data is available.

Using `when()` in these situations forces you to write unnecessary callbacks.

`maybeWhen()` keeps the code concise.

---

## Syntax

```dart
user.maybeWhen(
  data: (user) {
    return Text(user.name);
  },
  orElse: () {
    return const SizedBox();
  },
);
```

Explanation:

- The `data` callback runs only when the state is `AsyncData`.
- Every other state executes `orElse`.

---

## Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `loading` | ❌ | Handles the loading state |
| `data` | ❌ | Handles the data state |
| `error` | ❌ | Handles the error state |
| `orElse` | ✅ | Handles every unhandled state |

Unlike `when()`, only `orElse` is mandatory.

---

## Execution Flow

```text
          AsyncValue
               │
      ┌────────┼────────┐
      │        │        │
      ▼        ▼        ▼
 Loading     Data     Error
      │        │        │
      │        ▼        │
      │     data()      │
      │                 │
      └──────┬──────────┘
             ▼
         orElse()
```

Only the callbacks you provide are executed. Every other state falls back to `orElse`.

---

## Mental Model

Think of `maybeWhen()` as a **selective filter**.

```text
AsyncValue
     │
     ▼
Is it Data?
     │
 ┌───┴───┐
 │       │
Yes      No
 │       │
 ▼       ▼
data() orElse()
```

Instead of handling every state, you only focus on the one(s) you care about.

---

## Examples

### Handle Only Data

```dart
return user.maybeWhen(
  data: (user) {
    return Text(user.name);
  },
  orElse: () {
    return const CircularProgressIndicator();
  },
);
```

Explanation:

- Displays the user's name when data is available.
- All other states show a loading indicator.

---

### Handle Only Errors

```dart
return user.maybeWhen(
  error: (error, stackTrace) {
    return ErrorView(error);
  },
  orElse: () {
    return const SizedBox();
  },
);
```

Explanation:

- Only customizes the error state.
- Loading and data share the default UI.

---

### Handle Loading Only

```dart
return user.maybeWhen(
  loading: () {
    return const CircularProgressIndicator();
  },
  orElse: () {
    return const UserScreen();
  },
);
```

Explanation:

- Shows a spinner while loading.
- Uses the same UI for data and error.

---

### Real-World Example

```dart
final profile = ref.watch(profileProvider);

return profile.maybeWhen(
  error: (error, stackTrace) {
    return RetryScreen(
      message: 'Unable to load profile.',
    );
  },
  orElse: () {
    return const ProfileContent();
  },
);
```

Explanation:

- Only the error state receives special treatment.
- Every other state uses the default content.

---

## When to Use

Use `maybeWhen()` when:

- You only care about one or two states.
- Most states share the same UI.
- You want a default fallback.
- You're writing concise conditional UI.

---

## When NOT to Use

Avoid `maybeWhen()` when:

- Every state requires different behavior.
- You need exhaustive handling.
- You're building complete loading/data/error UIs.

In these cases, prefer `when()`.

---

## Best Practices

- Always provide a meaningful `orElse`.
- Use `when()` when handling all states.
- Keep callbacks lightweight.
- Avoid putting business logic inside callbacks.
- Prefer `maybeWhen()` only when it simplifies the code.

---

## Common Mistakes

### 1. Forgetting orElse

❌ Wrong

```dart
user.maybeWhen(
  data: (user) {
    return Text(user.name);
  },
);
```

Why it's wrong:

- `orElse` is required.
- Riverpod needs a fallback for unhandled states.

✔ Correct

```dart
user.maybeWhen(
  data: (user) {
    return Text(user.name);
  },
  orElse: () {
    return const SizedBox();
  },
);
```

---

### 2. Using maybeWhen() for Every State

❌ Wrong

```dart
user.maybeWhen(
  loading: LoadingView.new,
  data: UserView.new,
  error: ErrorView.new,
  orElse: SizedBox.new,
);
```

Why it's wrong:

- All states are already handled.
- `when()` is simpler and more expressive.

✔ Correct

```dart
user.when(
  loading: LoadingView.new,
  data: UserView.new,
  error: ErrorView.new,
);
```

---

### 3. Ignoring Important States

❌ Wrong

```dart
user.maybeWhen(
  data: UserView.new,
  orElse: SizedBox.new,
);
```

Why it's wrong:

- Errors are silently ignored.
- Users receive no feedback.

✔ Correct

Handle errors when they matter to the user.

---

### 4. Returning Different Types

❌ Wrong

```dart
data: (user) => Text(user.name),
orElse: () => 'Loading',
```

Why it's wrong:

- All callbacks must return the same type.

✔ Correct

Return a `Widget` (or another consistent type) from every callback.

---

## maybeWhen() vs when()

| Feature | `maybeWhen()` | `when()` |
|----------|---------------|-----------|
| Handles all states | ❌ | ✅ |
| `orElse` required | ✅ | ❌ |
| Optional callbacks | ✅ | ❌ |
| Exhaustive | ❌ | ✅ |
| Best for partial handling | ✅ | ❌ |
| Recommended for full UI | ❌ | ✅ |

---

## Related APIs

- `when()`
- `map()`
- `maybeMap()`
- `AsyncValue`
- `AsyncLoading`
- `AsyncData`
- `AsyncError`

---

## Summary

`maybeWhen()` allows you to handle only the asynchronous states you care about while providing a default `orElse` callback for every other state. It is ideal when one or two states need special handling, but for complete loading, data, and error UIs, `when()` remains the recommended and more expressive choice.