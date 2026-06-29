# ref.listenManual()

`ref.listenManual()` registers a manual listener for a provider and returns a subscription that can be explicitly closed when it is no longer needed.

---

## What is it?

`ref.listenManual()` is an alternative to `ref.listen()` that gives you **manual control over the listener's lifecycle**.

Instead of Riverpod automatically managing the listener, `ref.listenManual()` returns a `ProviderSubscription<T>`.

This subscription can later be:

- Closed manually
- Stored for later use
- Recreated when needed

It is primarily used in **`ConsumerStatefulWidget`** when the listener needs to be started or stopped manually.

---

## Why does it exist?

`ref.listen()` is lifecycle-aware.

It is automatically registered and disposed by Riverpod.

However, some situations require more control:

- Start listening only after initialization.
- Stop listening before the widget is disposed.
- Replace one listener with another.
- Manage listeners dynamically.

`ref.listenManual()` exists for these advanced scenarios.

---

## Syntax

### Creating a Manual Listener

```dart
late final ProviderSubscription<int> subscription;

@override
void initState() {
  super.initState();

  subscription = ref.listenManual(
    counterProvider,
    (previous, next) {
      print(next);
    },
  );
}
```

Explanation:

- Registers a listener manually.
- Returns a `ProviderSubscription<int>`.
- The subscription is stored for later use.

---

### Closing the Listener

```dart
@override
void dispose() {
  subscription.close();
  super.dispose();
}
```

Explanation:

- Stops listening to the provider.
- Releases resources immediately.

---

## Return Value

`ref.listenManual()` returns a `ProviderSubscription<T>`.

This object allows you to manually manage the listener.

Common methods:

| Method | Description |
|---------|-------------|
| `close()` | Stops listening to the provider |

Unlike `ref.listen()`, which returns `void`, this method gives you direct control over the subscription.

---

## Execution Flow

```text
Widget created
      │
      ▼
ref.listenManual()
      │
      ▼
Subscription returned
      │
      ▼
Provider changes
      │
      ▼
Callback executes
      │
      ▼
subscription.close()
      │
      ▼
Listener removed
```

The listener remains active until it is explicitly closed.

---

## Mental Model

Think of `ref.listenManual()` as **subscribing to a newsletter**.

```text
Provider
    │
    ▼
Manual Subscription
    │
    ▼
Receive Updates
    │
    ▼
subscription.close()
    │
    ▼
No More Updates
```

Unlike `ref.listen()`, **you decide when the subscription ends**.

---

## Examples

### Simple Example

```dart
late final ProviderSubscription<int> subscription;

@override
void initState() {
  super.initState();

  subscription = ref.listenManual(
    counterProvider,
    (previous, next) {
      print(next);
    },
  );
}

@override
void dispose() {
  subscription.close();
  super.dispose();
}
```

Explanation:

- Starts listening in `initState()`.
- Stops listening in `dispose()`.

---

### Login Navigation

```dart
late final ProviderSubscription<AuthState> authSubscription;

@override
void initState() {
  super.initState();

  authSubscription = ref.listenManual(
    authProvider,
    (previous, next) {
      if (next.isLoggedIn) {
        Navigator.pushReplacementNamed(
          context,
          '/home',
        );
      }
    },
  );
}
```

Explanation:

- Navigation listener begins during initialization.
- Automatically stops when closed.

---

### Analytics Listener

```dart
subscription = ref.listenManual(
  cartProvider,
  (previous, next) {
    analytics.logCartUpdated(next.items.length);
  },
);
```

Explanation:

- Tracks cart changes.
- Easy to disable when analytics are no longer needed.

---

## When to Use

Use `ref.listenManual()` when:

- Working inside `ConsumerStatefulWidget`
- Registering listeners in `initState()`
- You need manual lifecycle control
- A listener must be stopped before widget disposal
- Managing dynamic subscriptions

---

## When NOT to Use

Avoid `ref.listenManual()` when:

- You're inside `build()`
- Automatic lifecycle management is sufficient
- You only need simple side effects

In these cases, prefer:

- `ref.listen()`

---

## Best Practices

- Store the returned subscription.
- Always call `close()` when finished.
- Register listeners in `initState()` when appropriate.
- Keep listener callbacks focused on side effects.
- Prefer `ref.listen()` unless manual control is required.

---

## Common Mistakes

### 1. Forgetting to Close the Subscription

❌ Wrong

```dart
subscription = ref.listenManual(...);
```

Why it's wrong:

- The listener continues receiving updates.
- Can lead to memory leaks or unexpected behavior.

✔ Correct

```dart
@override
void dispose() {
  subscription.close();
  super.dispose();
}
```

---

### 2. Using listenManual() Instead of listen()

❌ Wrong

```dart
ref.listenManual(...);
```

Inside a `ConsumerWidget`.

Why it's wrong:

- Adds unnecessary complexity.
- Riverpod can manage the lifecycle automatically.

✔ Correct

```dart
ref.listen(...);
```

---

### 3. Creating Multiple Subscriptions

❌ Wrong

```dart
@override
Widget build(BuildContext context) {
  ref.listenManual(...);
}
```

Why it's wrong:

- A new subscription is created on every rebuild.
- Can result in duplicate listeners.

✔ Correct

Create the subscription once in `initState()`.

---

### 4. Using listenManual() for UI Rendering

❌ Wrong

```dart
subscription = ref.listenManual(
  counterProvider,
  (_, next) {
    counter = next;
  },
);
```

Why it's wrong:

- Listeners are for side effects.
- They don't automatically rebuild the UI.

✔ Correct

Use:

```dart
final counter = ref.watch(counterProvider);
```

---

## ref.listen() vs ref.listenManual()

| Feature | `ref.listen()` | `ref.listenManual()` |
|----------|----------------|----------------------|
| Automatic lifecycle | ✅ | ❌ |
| Manual subscription | ❌ | ✅ |
| Returns subscription | ❌ | ✅ |
| Returns `void` | ✅ | ❌ |
| Best for `build()` | ✅ | ❌ |
| Best for `initState()` | ❌ | ✅ |

---

## Related APIs

- `ref.listen()`
- `ref.watch()`
- `ref.read()`
- `ProviderSubscription`
- `ConsumerStatefulWidget`

---

## Summary

`ref.listenManual()` provides manual control over provider listeners by returning a `ProviderSubscription`. It is intended for advanced scenarios where the listener's lifecycle must be explicitly managed, such as registering listeners in `initState()` and closing them in `dispose()`. For most side effects, `ref.listen()` remains the preferred and simpler choice.