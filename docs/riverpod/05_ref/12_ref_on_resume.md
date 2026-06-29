# ref.onResume()

`ref.onResume()` registers a callback that executes when a previously cancelled `.autoDispose` provider gains a new listener before being disposed.

---

## What is it?

`ref.onResume()` is a lifecycle API that is called when an `.autoDispose` provider becomes **active again** after entering the cancelled state.

It always works together with `ref.onCancel()`.

The typical lifecycle is:

```text
Provider Active
      │
      ▼
Last Listener Removed
      │
      ▼
onCancel()
      │
      ▼
New Listener Added
      │
      ▼
onResume()
```

If the provider is disposed instead of being resumed, `onResume()` is **never called**.

---

## Why does it exist?

Some resources should be **paused** when nobody is listening and **resumed** when listeners return.

For example:

- Resume a paused WebSocket.
- Restart polling.
- Resume a paused stream.
- Continue background synchronization.

Without `onResume()`, you would need to recreate these resources every time, which can be expensive.

`ref.onResume()` allows providers to temporarily suspend work instead of destroying and recreating it.

---

## Availability

✅ **Only available in `.autoDispose` providers.**

It can be used inside:

- `Provider.autoDispose`
- `FutureProvider.autoDispose`
- `StreamProvider.autoDispose`
- `NotifierProvider.autoDispose`
- `AsyncNotifierProvider.autoDispose`
- `StreamNotifierProvider.autoDispose`

It is **not available** in regular providers.

---

## Syntax

### Registering an onResume Callback

```dart
ref.onResume(() {
  print('Provider resumed');
});
```

Explanation:

- Executes when a new listener starts watching the provider again.
- Only runs after `onCancel()`.

---

### Resume a Stream Subscription

```dart
ref.onResume(() {
  subscription.resume();
});
```

Explanation:

- Continues receiving stream events.
- Complements `subscription.pause()` in `onCancel()`.

---

### Resume Polling

```dart
ref.onResume(() {
  startPolling();
});
```

Explanation:

- Restarts background polling.
- Avoids unnecessary work while no listeners exist.

---

## Return Value

`ref.onResume()` returns `void`.

It simply registers a lifecycle callback that executes when the provider becomes active again.

Multiple callbacks may be registered.

---

## Execution Flow

```text
Provider Active
        │
        ▼
Last Listener Removed
        │
        ▼
onCancel()
        │
        ▼
Provider Waiting
        │
   ┌────┴────┐
   │         │
Listener     No Listener
Returns      Returns
   │             │
   ▼             ▼
onResume()   onDispose()
```

`onResume()` is only called if the provider survives long enough to receive another listener.

---

## Mental Model

Think of `onResume()` as **waking the provider up**.

```text
Provider Working
        │
        ▼
No Listeners
        │
        ▼
onCancel()
        │
        ▼
Sleep Mode
        │
        ▼
Listener Returns
        │
        ▼
onResume()
        │
        ▼
Continue Working
```

The provider doesn't restart—it simply continues from where it paused.

---

## Examples

### Resume a Subscription

```dart
ref.onResume(() {
  subscription.resume();
});
```

Explanation:

- Continues listening to stream events.
- Pairs naturally with `subscription.pause()`.

---

### Restart Polling

```dart
ref.onResume(() {
  timer = startPolling();
});
```

Explanation:

- Restarts periodic updates.
- Polling only occurs while listeners exist.

---

### Resume a WebSocket

```dart
ref.onResume(() {
  socket.resume();
});
```

Explanation:

- Continues receiving messages.
- Avoids reconnecting unnecessarily.

---

### Real-World Example

```dart
final chatProvider =
    StreamProvider.autoDispose<Message>((ref) {
  final subscription = chatStream.listen(print);

  ref.onCancel(() {
    subscription.pause();
  });

  ref.onResume(() {
    subscription.resume();
  });

  ref.onDispose(() {
    subscription.cancel();
  });

  return chatStream;
});
```

Explanation:

- Active listeners → receive messages.
- No listeners → pause subscription.
- Listeners return → resume subscription.
- Provider disposed → cancel subscription permanently.

---

## When to Use

Use `ref.onResume()` when:

- Resuming paused stream subscriptions.
- Restarting polling.
- Resuming WebSocket communication.
- Continuing background synchronization.
- Re-enabling expensive work only when needed.

---

## When NOT to Use

Avoid `ref.onResume()` when:

- Initializing resources.
- Cleaning up resources.
- Updating provider state.
- Performing work unrelated to provider lifecycle.

For cleanup, use `ref.onDispose()`.

For initialization, use `build()`.

---

## Best Practices

- Pair `onResume()` with `onCancel()`.
- Keep callbacks lightweight.
- Resume existing resources instead of recreating them.
- Use `onDispose()` for permanent cleanup.
- Avoid performing expensive initialization inside `onResume()`.

---

## Common Mistakes

### 1. Using onResume() Without onCancel()

❌ Wrong

```dart
ref.onResume(() {
  subscription.resume();
});
```

Why it's wrong:

- Nothing was paused.
- The callback has no meaningful purpose.

✔ Correct

```dart
ref.onCancel(() {
  subscription.pause();
});

ref.onResume(() {
  subscription.resume();
});
```

---

### 2. Recreating Resources Instead of Resuming

❌ Wrong

```dart
ref.onResume(() {
  socket = createSocket();
});
```

Why it's wrong:

- Creates unnecessary new connections.
- Existing resources should simply resume.

✔ Correct

```dart
ref.onResume(() {
  socket.resume();
});
```

---

### 3. Expecting onResume() After Disposal

❌ Wrong

Assuming:

```text
onDispose()
      ↓
onResume()
```

Why it's wrong:

- Once disposed, the provider is destroyed.
- A new provider instance is created instead.

✔ Correct

`onResume()` only occurs between `onCancel()` and `onDispose()`.

---

### 4. Using onResume() in Regular Providers

❌ Wrong

```dart
final provider = Provider((ref) {
  ref.onResume(() {});
});
```

Why it's wrong:

- Regular providers don't enter the cancelled state.
- `onResume()` is only available for `.autoDispose` providers.

✔ Correct

Use an `.autoDispose` provider when lifecycle callbacks are required.

---

## Lifecycle Timeline

```text
Provider Created
        │
        ▼
Listening
        │
        ▼
onCancel()
        │
        ▼
Waiting
   ┌────┴────┐
   │         │
Resume     Dispose
   │         │
   ▼         ▼
onResume() onDispose()
```

---

## Related APIs

- `ref.onCancel()`
- `ref.onDispose()`
- `ref.keepAlive()`
- `.autoDispose`
- `Ref Lifecycle`

---

## Summary

`ref.onResume()` is called when an `.autoDispose` provider receives a new listener after previously entering the cancelled state. It is intended for resuming paused work—such as stream subscriptions, polling, or WebSocket communication—and is typically paired with `ref.onCancel()`. If the provider is disposed instead of resumed, `onResume()` is never executed.