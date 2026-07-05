# ref.listen()

`ref.listen()` registers a listener for a provider and executes a callback whenever the provider's state changes without rebuilding the caller.

---

## What is it?

`ref.listen()` is used to **react to provider state changes by performing side effects**.

Unlike `ref.watch()`, it does **not** rebuild the widget or provider.

Instead, it invokes a callback whenever the provider's value changes, giving you access to both the previous and current values.

Typical side effects include:

- Showing a `SnackBar`
- Displaying a dialog
- Navigating to another screen
- Logging state changes
- Triggering analytics events

Think of `ref.listen()` as **"watch without rebuilding."**

---

## Why does it exist?

Not every state change should rebuild the UI.

Sometimes you only want to **respond** to a change.

For example:

- Navigate after a successful login.
- Show an error message when an API call fails.
- Log analytics events.
- Display a success toast.

Using `ref.watch()` for these scenarios is incorrect because rebuilding can cause the side effect to run multiple times.

`ref.listen()` separates **state observation** from **UI rendering**, making side effects predictable and safe.

---

## Syntax

### Listening to a Provider

```dart
ref.listen(counterProvider, (previous, next) {
  print('Counter changed: $next');
});
```

Explanation:

- Registers a listener.
- Executes the callback whenever the provider changes.
- Does not rebuild the caller.

---

### Listening to an Async Provider

```dart
ref.listen(userProvider, (previous, next) {
  next.whenOrNull(
    error: (error, stackTrace) {
      print(error);
    },
  );
});
```

Explanation:

- Listens to `AsyncValue<User>`.
- Executes only when an error occurs.

---

### Listening Inside a ConsumerWidget

```dart
class LoginPage extends ConsumerWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    ref.listen(authProvider, (previous, next) {
      if (next.isLoggedIn) {
        Navigator.pushReplacementNamed(context, '/home');
      }
    });

    return const LoginView();
  }
}
```

Explanation:

- UI is built normally.
- Navigation occurs only when authentication changes.

---

## Return Value

`ref.listen()` returns `void`.

It registers a listener with Riverpod, but it does **not** return the provider's value.

Instead, the callback receives two parameters:

```dart
(previous, next)
```

| Parameter | Description |
|-----------|-------------|
| `previous` | Previous provider value |
| `next` | Current provider value |

These values allow you to compare state transitions.

---

## Execution Flow

```text
Provider changes
        │
        ▼
Riverpod detects update
        │
        ▼
Listener callback executes
        │
        ▼
Side effect runs
        │
        ▼
No widget rebuild occurs
```

Unlike `watch()`, dependency tracking is not used for rebuilding.

---

## Mental Model

Think of `ref.listen()` as a **notification system**.

```text
Provider
    │
State Changes
    │
    ▼
 ref.listen()
    │
    ▼
 Callback Runs
    │
    ├───────────────┐
    ▼               ▼
Show Snackbar   Navigate
Log Event       Show Dialog
```

The UI remains unchanged unless you explicitly change it.

---

## Examples

### Simple Example

```dart
ref.listen(counterProvider, (previous, next) {
  print('Counter: $next');
});
```

Explanation:

- Logs every counter update.
- No rebuild occurs.

---

### Show a SnackBar

```dart
ref.listen(loginProvider, (previous, next) {
  if (next.hasError) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(next.error.toString()),
      ),
    );
  }
});
```

Explanation:

- Shows an error message.
- UI rebuild is unnecessary.

---

### Navigate After Login

```dart
ref.listen(authProvider, (previous, next) {
  if (next.isLoggedIn) {
    Navigator.pushReplacementNamed(
      context,
      '/home',
    );
  }
});
```

Explanation:

- Navigation is triggered only when login succeeds.
- Keeps navigation logic separate from UI rendering.

---

### Analytics

```dart
ref.listen(cartProvider, (previous, next) {
  analytics.logCartUpdated(next.items.length);
});
```

Explanation:

- Sends analytics events.
- No widget rebuild.

---

## When to Use

Use `ref.listen()` when:

- Showing a `SnackBar`
- Displaying dialogs
- Navigating between screens
- Logging state changes
- Sending analytics events
- Triggering side effects based on state

---

## When NOT to Use

Avoid `ref.listen()` when:

- Displaying data in the UI
- Building reactive widgets
- Returning computed state

For those cases, use:

- `ref.watch()`

---

## Best Practices

- Use `listen()` only for side effects.
- Keep callback logic short and focused.
- Compare `previous` and `next` when appropriate.
- Avoid modifying unrelated state inside listeners.
- Separate rendering (`watch`) from reactions (`listen`).

---

## Common Mistakes

### 1. Using `watch()` for Navigation

❌ Wrong

```dart
final auth = ref.watch(authProvider);

if (auth.isLoggedIn) {
  Navigator.pushNamed(context, '/home');
}
```

Why it's wrong:

- Every rebuild can trigger navigation again.

✔ Correct

```dart
ref.listen(authProvider, (previous, next) {
  if (next.isLoggedIn) {
    Navigator.pushNamed(context, '/home');
  }
});
```

---

### 2. Using `listen()` to Build UI

❌ Wrong

```dart
ref.listen(counterProvider, (previous, next) {
  counter = next;
});
```

Why it's wrong:

- `listen()` is not meant for rendering.
- The widget won't rebuild automatically.

✔ Correct

```dart
final counter = ref.watch(counterProvider);
```

---

### 3. Ignoring Previous State

❌ Wrong

```dart
ref.listen(counterProvider, (_, next) {
  print(next);
});
```

Why it's wrong:

- Sometimes you only want to react to specific transitions.

✔ Correct

```dart
ref.listen(counterProvider, (previous, next) {
  if (previous != next) {
    print(next);
  }
});
```

---

### 4. Putting Heavy Logic in the Listener

❌ Wrong

```dart
ref.listen(userProvider, (previous, next) {
  // Large business logic here
});
```

Why it's wrong:

- Makes listeners difficult to maintain.
- Business logic belongs in notifiers or services.

✔ Correct

Keep listeners focused on UI reactions and side effects.

---

## ref.listen() vs ref.watch()

| Feature | `ref.listen()` | `ref.watch()` |
|----------|----------------|---------------|
| Reads current value | ❌ | ✅ |
| Callback on changes | ✅ | ❌ |
| Registers rebuild dependency | ❌ | ✅ |
| Performs side effects | ✅ | ❌ |
| Rebuilds UI | ❌ | ✅ |

---

## Related APIs

- `ref.watch()`
- `ref.read()`
- `ref.listenManual()`
- `WidgetRef`
- `ConsumerWidget`
- `ConsumerStatefulWidget`

---

## Summary

`ref.listen()` allows you to observe provider state changes and execute side effects without rebuilding the UI. It is the preferred API for navigation, snackbars, dialogs, logging, and analytics, ensuring that rendering and side effects remain cleanly separated in Riverpod applications.