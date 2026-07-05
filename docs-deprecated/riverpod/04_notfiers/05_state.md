# state

`state` is the internal value holder of a `Notifier`, `AsyncNotifier`, or `StreamNotifier` that represents the current emitted state of a Riverpod provider.

---

## What is it?

`state` is the **single source of truth** inside Riverpod notifiers.

It holds the current value being exposed to the UI through a provider, and whenever it changes, Riverpod automatically notifies all listeners and rebuilds dependent widgets.

Depending on the notifier type, `state` can be:

- A plain value (`Notifier<T>`)
- An `AsyncValue<T>` (`AsyncNotifier<T>`)
- A stream-driven state (`StreamNotifier<T>`)

It is not just a variable — it is a **reactive state container managed by Riverpod**.

---

## Why does it exist?

Traditional state management often requires manual update mechanisms like:

- `setState()`
- `notifyListeners()`
- External state stores

These approaches are error-prone and scattered across UI layers.

`state` exists to:

- Centralize state updates inside notifiers
- Ensure predictable updates
- Automatically trigger UI rebuilds
- Prevent direct UI mutation of state
- Maintain reactive consistency across the app

It is the core mechanism that connects business logic to UI updates.

---

## Syntax

### Notifier

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}
```

Explanation:

- `state` holds the current integer value.
- Updating `state` triggers rebuilds.

---

### AsyncNotifier

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchUser();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(
      repository.fetchUser,
    );
  }
}
```

Explanation:

- `state` is an `AsyncValue<User>`.
- It represents loading, data, or error.

---

### StreamNotifier

```dart
class CounterNotifier extends StreamNotifier<int> {
  @override
  Stream<int> build() async* {
    yield* Stream.periodic(
      const Duration(seconds: 1),
      (i) => i,
    );
  }
}
```

Explanation:

- `state` is implicitly updated from stream emissions.
- Each yield updates the state automatically.

---

## Mental Model

Think of `state` as the **live memory of a provider**.

```text
Notifier
   │
   ▼
state  ←── current value
   │
   ▼
Provider exposes state
   │
   ▼
UI rebuilds on change
```

Every update to `state` instantly propagates to the UI.

---

## Examples

### Counter

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}
```

Explanation:

- `state` holds the counter value.
- UI updates on each increment.

---

### Toggle State

```dart
class ThemeNotifier extends Notifier<bool> {
  @override
  bool build() => false;

  void toggle() {
    state = !state;
  }
}
```

Explanation:

- `state` represents theme mode.
- Changing it triggers rebuild.

---

### Async State Handling

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchUser();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(
      repository.fetchUser,
    );
  }
}
```

Explanation:

- `state` switches between loading/data/error.
- UI reacts automatically.

---

## When to Use

Use `state` when:

- Updating any notifier-based state
- Emitting new values to UI
- Handling async lifecycle states
- Managing stream emissions internally
- Encapsulating business logic updates

---

## When NOT to Use

Avoid using `state`:

- Outside a notifier class
- Directly in UI widgets
- Without proper immutable updates (for collections)
- For complex multi-step logic without methods

Instead, expose methods that modify `state`.

---

## Best Practices

- Always update state inside notifier methods
- Prefer immutable updates (especially for collections)
- Avoid exposing `state` directly for modification in UI
- Keep state transitions predictable
- Use `AsyncValue.guard()` for async updates

---

## Common Mistakes

### 1. Mutating State Directly (Collections)

❌ Wrong

```dart
state.add(item);
```

Why it's wrong:

- Mutation is not detected by Riverpod.
- UI may not rebuild.

✔ Correct

```dart
state = [...state, item];
```

---

### 2. Updating State Outside Notifier

❌ Wrong

```dart
state = 5;
```

(in UI or external scope)

Why it's wrong:

- `state` is only available inside notifiers.

✔ Correct

```dart
ref.read(counterProvider.notifier).increment();
```

---

### 3. Overwriting Async State Incorrectly

❌ Wrong

```dart
state = AsyncData(await repo.fetch());
```

Why it's wrong:

- Does not handle errors properly.

✔ Correct

```dart
state = await AsyncValue.guard(repo.fetch);
```

---

## Related APIs

- Notifier
- AsyncNotifier
- StreamNotifier
- NotifierProvider
- AsyncNotifierProvider
- StreamNotifierProvider
- AsyncValue

---

## Summary

`state` is the core reactive value inside Riverpod notifiers. It represents the current state of a provider and ensures that any change automatically updates the UI. It is the single source of truth for all notifier-based state management in Riverpod.