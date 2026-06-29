# ref.onCancel()

`ref.onCancel()` registers a callback that executes when the last listener is removed from an `.autoDispose` provider, before the provider is actually disposed.

---

## What is it?

`ref.onCancel()` is a lifecycle API that lets you know when **a provider is no longer being listened to**.

This is **not** the same as disposal.

When the last widget or provider stops watching an `.autoDispose` provider:

1. `ref.onCancel()` is called.
2. Riverpod waits to see if a new listener appears.
3. If no listener returns, the provider is disposed.
4. If a listener returns before disposal, `ref.onResume()` is called instead.

This makes `onCancel()` useful for pausing work rather than destroying resources.

---

## Why does it exist?

Sometimes you don't want to destroy a provider immediately when it loses its last listener.

For example:

- Pause a WebSocket connection.
- Pause polling.
- Suspend expensive background work.
- Log lifecycle events.
- Delay cleanup until disposal is certain.

Without `onCancel()`, you'd only know when the provider is completely disposed using `onDispose()`.

`onCancel()` provides an intermediate lifecycle event.

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

It is **not available** in regular providers because they are never automatically disposed when listeners disappear.

---

## Syntax

### Registering an onCancel Callback

```dart
ref.onCancel(() {
  print('No more listeners');
});
```

Explanation:

- Executes when the last listener is removed.
- The provider is not yet disposed.

---

### Pause Polling

```dart
final timer = Timer.periodic(
  const Duration(seconds: 5),
  (_) => fetchData(),
);

ref.onCancel(() {
  timer.cancel();
});
```

Explanation:

- Stops polling when nobody is watching.
- Conserves CPU and network usage.

---

### Pause a WebSocket

```dart
ref.onCancel(() {
  socket.pause();
});
```

Explanation:

- Pauses incoming messages.
- Keeps the provider alive until disposal.

---

## Return Value

`ref.onCancel()` returns `void`.

It simply registers a lifecycle callback that executes when the last listener disappears.

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
ref.onCancel()
        │
        ▼
Waiting...
        │
   ┌────┴────┐
   │         │
New Listener  No Listener
Returns       Returns
   │             │
   ▼             ▼
onResume()   onDispose()
```

Notice that `onCancel()` happens **before** disposal.

---

## Mental Model

Think of `onCancel()` as **putting the provider on standby**.

```text
Watching
    │
    ▼
Last Listener Leaves
    │
    ▼
onCancel()
    │
    ▼
Standby Mode
    │
    ├──────────────┐
    ▼              ▼
Listener Returns  Disposed
     │              │
     ▼              ▼
 onResume()    onDispose()
```

The provider is still alive—it just has no active listeners.

---

## Examples

### Log Lifecycle Events

```dart
ref.onCancel(() {
  debugPrint('Provider has no listeners.');
});
```

Explanation:

- Useful for debugging provider lifecycles.

---

### Pause Background Polling

```dart
final timer = Timer.periodic(
  const Duration(seconds: 10),
  (_) => fetchUpdates(),
);

ref.onCancel(() {
  timer.cancel();
});
```

Explanation:

- Stops unnecessary work.
- Saves battery and network resources.

---

### Pause a Stream

```dart
ref.onCancel(() {
  subscription.pause();
});
```

Explanation:

- Temporarily pauses the subscription.
- Can later be resumed with `onResume()`.

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

- No listeners → pause receiving messages.
- Listeners return → resume.
- Provider disposed → cancel permanently.

---

## When to Use

Use `ref.onCancel()` when:

- Pausing polling.
- Pausing stream subscriptions.
- Suspending background work.
- Tracking lifecycle events.
- Conserving resources while waiting for listeners to return.

---

## When NOT to Use

Avoid `ref.onCancel()` when:

- Releasing resources permanently.
- Closing sockets or controllers.
- Performing cleanup that should happen only once.

Use `ref.onDispose()` for permanent cleanup.

---

## Best Practices

- Use `onCancel()` for temporary suspension.
- Pair `onCancel()` with `onResume()` whenever possible.
- Keep callbacks lightweight.
- Don't confuse cancellation with disposal.
- Reserve `onDispose()` for final cleanup.

---

## Common Mistakes

### 1. Closing Resources in onCancel()

❌ Wrong

```dart
ref.onCancel(() {
  socket.close();
});
```

Why it's wrong:

- The provider may become active again.
- Closing the socket requires creating a new connection later.

✔ Correct

```dart
ref.onCancel(() {
  socket.pause();
});
```

Then:

```dart
ref.onResume(() {
  socket.resume();
});
```

---

### 2. Assuming onCancel() Means Disposal

❌ Wrong

```dart
ref.onCancel(() {
  print('Provider disposed');
});
```

Why it's wrong:

- The provider still exists.
- Disposal hasn't happened yet.

✔ Correct

Use `ref.onDispose()` for final cleanup.

---

### 3. Forgetting to Handle Resume

❌ Wrong

```dart
ref.onCancel(() {
  subscription.pause();
});
```

Why it's wrong:

- If listeners return, the subscription stays paused.

✔ Correct

```dart
ref.onResume(() {
  subscription.resume();
});
```

---

### 4. Using onCancel() in Regular Providers

❌ Wrong

```dart
final provider = Provider((ref) {
  ref.onCancel(() {});
});
```

Why it's wrong:

- Regular providers don't support listener cancellation.
- This API is only available for `.autoDispose` providers.

✔ Correct

Use an `.autoDispose` provider if cancellation behavior is needed.

---

## ref.onCancel() vs ref.onDispose()

| Feature | `ref.onCancel()` | `ref.onDispose()` |
|----------|------------------|-------------------|
| Triggered when last listener leaves | ✅ | ❌ |
| Triggered when provider is destroyed | ❌ | ✅ |
| Provider still alive afterward | ✅ | ❌ |
| Best for pausing work | ✅ | ❌ |
| Best for permanent cleanup | ❌ | ✅ |

---

## Related APIs

- `ref.onResume()`
- `ref.onDispose()`
- `ref.keepAlive()`
- `.autoDispose`
- `Ref Lifecycle`

---

## Summary

`ref.onCancel()` is called when the last listener is removed from an `.autoDispose` provider, before the provider is disposed. It is designed for temporarily pausing work—such as polling, stream subscriptions, or background tasks—while giving the provider a chance to resume if a new listener appears. For permanent cleanup, use `ref.onDispose()` instead.