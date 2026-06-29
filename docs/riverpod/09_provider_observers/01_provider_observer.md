# ProviderObserver

Observes provider lifecycle events and state changes across a Riverpod application.

---

# What is it?

`ProviderObserver` is a Riverpod class that allows you to observe what happens inside the provider system.

Instead of affecting provider behavior, it listens for events such as:

- Provider creation
- State updates
- Provider disposal
- Provider failures

A `ProviderObserver` is attached to a `ProviderScope` and receives callbacks whenever providers change.

It is primarily used for logging, debugging, analytics, and monitoring.

---

# Why does it exist?

As applications grow, it becomes difficult to understand:

- Which providers are being created?
- Why did a provider rebuild?
- Which provider changed?
- When was a provider disposed?

Without a centralized observer, you would have to add logging inside every provider.

`ProviderObserver` solves this by providing a single place to observe provider activity throughout the application.

---

# Syntax

## Creating an Observer

```dart
class AppProviderObserver extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderObserverContext context,
    Object? previousValue,
    Object? newValue,
  ) {
    print(
      '${context.provider.name ?? context.provider.runtimeType}: '
      '$previousValue -> $newValue',
    );
  }
}
```

Explanation:

- Extend `ProviderObserver`.
- Override the lifecycle methods you need.
- `didUpdateProvider()` is called whenever a provider's value changes.

---

## Registering an Observer

```dart
void main() {
  runApp(
    ProviderScope(
      observers: [
        AppProviderObserver(),
      ],
      child: const MyApp(),
    ),
  );
}
```

Explanation:

- Observers are registered through `ProviderScope`.
- Every provider inside the scope is monitored.

---

## Observing Provider Disposal

```dart
class AppProviderObserver extends ProviderObserver {
  @override
  void didDisposeProvider(
    ProviderObserverContext context,
  ) {
    print(
      'Disposed: ${context.provider.name ?? context.provider.runtimeType}',
    );
  }
}
```

Explanation:

- Called when a provider is destroyed.
- Useful for verifying `autoDispose` behavior.

---

# Mental Model

Think of `ProviderObserver` as a **security camera** for your provider graph.

```text
          ProviderScope
                │
                ▼
      ┌─────────────────┐
      │ ProviderObserver│
      └─────────────────┘
         ▲     ▲      ▲
         │     │      │
     Create  Update Dispose
```

It watches provider events without changing how providers work.

---

# Examples

## Simple Logging

```dart
class Logger extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderObserverContext context,
    Object? previousValue,
    Object? newValue,
  ) {
    print(newValue);
  }
}
```

Explanation:

- Prints every new provider value.
- Useful during development.

---

## Real-World Example

```dart
class AnalyticsObserver extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderObserverContext context,
    Object? previousValue,
    Object? newValue,
  ) {
    analytics.logEvent(
      name: 'provider_updated',
      parameters: {
        'provider':
            context.provider.name ??
            context.provider.runtimeType.toString(),
      },
    );
  }
}
```

Explanation:

- Sends provider activity to an analytics service.
- Useful for debugging production issues.

---

# When to Use

Use `ProviderObserver` when:

- Debugging provider behavior
- Logging state changes
- Monitoring provider lifecycle
- Verifying `autoDispose`
- Tracking provider updates during development
- Integrating analytics or diagnostics

---

# When NOT to Use

Avoid `ProviderObserver` when:

- Implementing business logic
- Modifying provider state
- Triggering UI updates
- Replacing provider lifecycle methods

Observers should only observe, not control application behavior.

---

# Best Practices

- Keep observer logic lightweight.
- Use it primarily for development and diagnostics.
- Log only useful information.
- Avoid expensive work inside observer callbacks.
- Consider disabling verbose logging in production.

---

# Common Mistakes

## Modifying Application State

**Wrong**

```dart
didUpdateProvider(...) {
  ref.read(counterProvider.notifier).increment();
}
```

An observer should never modify providers.

**Correct**

```dart
didUpdateProvider(...) {
  print(newValue);
}
```

Use observers only for monitoring.

---

## Logging Every Detail in Production

**Wrong**

```dart
print(previousValue);
print(newValue);
```

Excessive logging can affect performance and clutter logs.

**Correct**

Log only important events or disable verbose logging in release builds.

---

## Expecting ProviderObserver to Rebuild Widgets

**Wrong**

Using an observer to update the UI.

**Correct**

Use `ref.watch()` for UI updates and `ProviderObserver` for diagnostics only.

---

# Related APIs

- `ProviderScope`
- `ProviderContainer`
- `ref.watch()`
- `ref.listen()`
- `.autoDispose`

---

# Summary

`ProviderObserver` provides a centralized way to monitor provider lifecycle events and state changes across an application. It is an essential tool for debugging, logging, and diagnostics, allowing you to observe provider behavior without affecting how providers work.