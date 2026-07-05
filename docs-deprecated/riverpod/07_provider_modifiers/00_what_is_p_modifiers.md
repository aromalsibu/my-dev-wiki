# Provider Modifiers (Section Summary)

Provider modifiers allow you to change how a provider behaves without changing its core logic.

They are used to control lifecycle, caching, parameterization, overrides, and performance optimizations.

---

# What are Provider Modifiers?

Provider modifiers are extensions applied to a provider that alter its behavior.

Instead of creating new provider types for every use case, Riverpod uses modifiers to extend existing providers in a composable way.

They let you:

- Control lifecycle (`autoDispose`, `keepAlive`)
- Pass parameters (`family`)
- Optimize rebuilds (`select`)
- Override behavior (`overrideWith`, `overrideWithValue`)

---

# Why do they exist?

Without modifiers, Riverpod would need separate provider classes for every combination of behavior.

For example:

- A provider that is auto-disposable
- A provider that takes parameters
- A provider that supports overrides

This would lead to an explosion of provider types.

Modifiers solve this by allowing **composition instead of duplication**.

---

# Syntax Overview

```dart
final provider = SomeProvider.autoDispose.family((ref, param) {
  return value;
});
```

Explanation:

- Modifiers are chained onto providers.
- Each modifier changes a specific aspect of behavior.
- Order generally does not matter for most cases, but should remain consistent for readability.

---

# Mental Model

Think of modifiers as **filters applied to a provider**:

```text
Base Provider
     │
     ├── .family      → adds parameters
     ├── .autoDispose → adds lifecycle control
     ├── .select       → adds fine-grained listening
     ├── override      → replaces implementation
     └── keepAlive     → prevents disposal
```

Each modifier adjusts how Riverpod manages the provider internally.

---

# List of Provider Modifiers

## .family

Creates parameterized providers.

- Used for dynamic inputs (IDs, queries, filters)

---

## .autoDispose

Automatically disposes providers when unused.

- Used for temporary or screen-based state

---

## keepAlive

Prevents disposal even if unused.

- Used for caching or persistent state

---

## .select

Subscribes to only part of a provider's state.

- Used for performance optimization

---

## .overrideWith

Replaces provider implementation with custom logic.

- Used for testing and dependency injection

---

## .overrideWithValue

Replaces provider with a fixed value.

- Used for simple mocks and static overrides

---

# When to Use Provider Modifiers

Use modifiers when:

- You need lifecycle control
- You need parameterized providers
- You want to optimize rebuilds
- You need to replace implementations for testing
- You want to cache or preserve state

---

# When NOT to Overuse Them

Avoid modifiers when:

- The base provider already solves the problem
- You are adding complexity without benefit
- Multiple modifiers make code harder to read
- Behavior can be handled inside the provider itself

---

# Best Practices

- Prefer simple providers first, then add modifiers only if needed
- Combine `.family` + `.autoDispose` for typical API-based screens
- Use `.select` only for performance-sensitive widgets
- Keep overrides limited to test or environment setup
- Avoid mixing too many modifiers unnecessarily

---

# Common Mistakes

## Overusing Modifiers

**Wrong**

```dart
final provider =
    Provider.autoDispose.family.overrideWithValue(...);
```

Too many modifiers make behavior unclear.

**Correct**

Use only required modifiers:

```dart
final provider = Provider.family((ref, id) {
  return fetch(id);
});
```

---

## Misunderstanding keepAlive + autoDispose

**Wrong**

Assuming `.autoDispose` always disposes the provider.

**Correct**

If `keepAlive` is used, disposal is prevented.

---

## Using override in production logic

**Wrong**

Using `.overrideWith` inside business logic.

**Correct**

Overrides should be external (tests, scopes, environments).

---

# Related APIs

- `Provider`
- `FutureProvider`
- `StreamProvider`
- `NotifierProvider`
- `AsyncNotifierProvider`
- `ProviderScope`
- `ref.keepAlive()`

---

# Summary

Provider modifiers are composable tools that extend Riverpod providers with lifecycle control, parameterization, optimization, and override capabilities. They prevent provider duplication and enable flexible state management patterns while keeping the core provider model simple and reusable.