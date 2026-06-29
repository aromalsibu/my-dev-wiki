# Common APIs

A quick reference for the Riverpod APIs you'll use most frequently in day-to-day development.

---

# What is it?

Riverpod exposes a small set of APIs for creating providers, reading state, updating state, and handling asynchronous values.

This page serves as a quick lookup rather than a detailed explanation.

---

# Why does it exist?

As projects grow, remembering every API becomes difficult.

A cheat sheet lets you quickly find:

- Provider creation APIs
- State access APIs
- State update APIs
- Consumer APIs
- Common helper methods

Use this page as a reference while developing.

---

# Provider Creation APIs

| API | Purpose |
|------|---------|
| `Provider` | Immutable values and dependency injection |
| `StateProvider` | Simple mutable state |
| `NotifierProvider` | Complex synchronous state |
| `FutureProvider` | Read-only asynchronous state |
| `AsyncNotifierProvider` | Editable asynchronous state |
| `StreamProvider` | Read-only streams |
| `StreamNotifierProvider` | Managed streams |

---

# Consumer APIs

| API | Purpose |
|------|---------|
| `Consumer` | Watches providers in part of a widget tree |
| `ConsumerWidget` | Stateless widget with a `WidgetRef` |
| `ConsumerStatefulWidget` | Stateful widget with a `WidgetRef` |
| `WidgetRef` | Access providers inside widgets |

---

# Reading Providers

## Watch a provider

```dart
final count = ref.watch(counterProvider);
```

Explanation:

- Rebuilds when the provider's value changes.

---

## Read a provider

```dart
final notifier = ref.read(counterProvider.notifier);
```

Explanation:

- Reads the current value without subscribing.
- Commonly used inside callbacks.

---

## Listen to changes

```dart
ref.listen(userProvider, (previous, next) {
  // React to changes
});
```

Explanation:

- Executes a callback whenever the provider changes.
- Does not rebuild the widget.

---

# Updating State

## StateProvider

```dart
ref.read(counterProvider.notifier).state++;
```

Explanation:

- Updates the provider's state directly.

---

## Notifier

```dart
ref
    .read(counterProvider.notifier)
    .increment();
```

Explanation:

- Calls a method that owns the business logic.

---

## AsyncNotifier

```dart
await ref
    .read(userProvider.notifier)
    .refreshUser();
```

Explanation:

- Executes an asynchronous state update.

---

# AsyncValue

## Handle loading, data, and error

```dart
return user.when(
  data: UserView.new,
  loading: LoadingView.new,
  error: ErrorView.new,
);
```

Explanation:

- Handles all possible asynchronous states.

---

## Guard asynchronous operations

```dart
state = await AsyncValue.guard(() async {
  return repository.fetchUser();
});
```

Explanation:

- Converts exceptions into `AsyncError`.

---

# Refreshing Providers

## Refresh immediately

```dart
ref.refresh(userProvider);
```

Explanation:

- Recreates the provider immediately.
- Returns the new value.

---

## Invalidate

```dart
ref.invalidate(userProvider);
```

Explanation:

- Marks the provider as stale.
- Rebuilds on the next read.

---

# Provider Lifecycle

## Keep auto-disposed providers alive

```dart
final link = ref.keepAlive();
```

Explanation:

- Prevents an auto-disposed provider from being disposed.

---

## Dispose callback

```dart
ref.onDispose(() {
  controller.dispose();
});
```

Explanation:

- Runs cleanup logic before disposal.

---

# Watching Multiple Providers

```dart
final user = ref.watch(userProvider);
final settings = ref.watch(settingsProvider);
```

Explanation:

- A widget can watch any number of providers.

---

# Accessing Provider State

## Read current value

```dart
final value = ref.read(counterProvider);
```

Explanation:

- Reads the current state once.

---

## Subscribe to updates

```dart
final value = ref.watch(counterProvider);
```

Explanation:

- Rebuilds whenever the value changes.

---

# Provider Modifiers

| Modifier | Purpose |
|----------|---------|
| `.family` | Parameterized providers |
| `.autoDispose` | Dispose when unused |
| `.select()` | Listen to part of an object |
| `.overrideWith()` | Override provider implementation |
| `.overrideWithValue()` | Override with a fixed value |

---

# Testing

## Create a container

```dart
final container = ProviderContainer();
```

Explanation:

- Creates an isolated environment for providers.

---

## Read a provider

```dart
container.read(counterProvider);
```

Explanation:

- Reads provider values during tests.

---

# Most Frequently Used APIs

| API | Typical Usage |
|------|---------------|
| `ref.watch()` | Observe provider changes |
| `ref.read()` | Read once or call notifier methods |
| `ref.listen()` | React to changes without rebuilding |
| `ref.refresh()` | Immediately recreate a provider |
| `ref.invalidate()` | Mark a provider for recomputation |
| `AsyncValue.when()` | Build UI for async state |
| `AsyncValue.guard()` | Simplify async error handling |
| `Notifier.state` | Read or update synchronous state |
| `AsyncNotifier.state` | Read or update asynchronous state |

---

# Related APIs

- Provider
- StateProvider
- Notifier
- AsyncNotifier
- StreamNotifier
- WidgetRef
- AsyncValue

---

# Summary

The APIs you'll use most often are `ref.watch()` for observing state, `ref.read()` for one-time access, `ref.listen()` for side effects, `ref.refresh()` and `ref.invalidate()` for recomputation, and `AsyncValue.when()` for handling asynchronous UI. Mastering these core APIs covers the majority of everyday Riverpod development.