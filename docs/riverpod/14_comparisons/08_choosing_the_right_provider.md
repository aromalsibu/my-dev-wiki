# Choosing the Right Riverpod Provider

Select the provider type based on the kind of state you need to manage, not on personal preference.

---

Choosing the correct provider keeps your application simpler, easier to test, and easier to maintain. Riverpod provides specialized providers for different use cases, so the best choice is usually the simplest provider that satisfies your requirements.

---

# What is it?

Riverpod offers multiple provider types because different kinds of state have different requirements.

For example:

- A constant value does not need mutable state.
- A counter does not need asynchronous loading.
- A network request does not need synchronous state.
- A live chat stream does not need a `Future`.

Each provider is optimized for a specific job.

---

# Why does it exist?

Using the wrong provider often leads to:

- Unnecessary complexity
- Extra boilerplate
- Business logic inside widgets
- Difficult testing
- Poor maintainability

Choosing the right provider from the beginning keeps your codebase clean and scalable.

---

# Decision Guide

```text
Do you need state?

        │
        ├── No
        │      │
        │      ▼
        │   Provider
        │
        └── Yes
               │
               ▼
      Is it synchronous?
               │
        ┌──────┴──────┐
        │             │
       Yes            No
        │             │
        ▼             ▼
Simple?         Read-only?
        │             │
   ┌────┴────┐    ┌───┴────┐
   │         │    │        │
 Yes        No   Yes      No
   │         │    │        │
   ▼         ▼    ▼        ▼
State   Notifier Future Async
Provider         Provider Notifier
```

Explanation:

- Start with the simplest provider.
- Move to more powerful providers only when your requirements grow.

---

# Provider Selection

## Provider

Use when:

- Exposing constants
- Dependency injection
- Repositories
- Services
- Configuration
- Derived values

Example:

```dart
final repositoryProvider =
    Provider<UserRepository>((ref) {
  return UserRepository();
});
```

Explanation:

- Creates an immutable dependency.
- Consumers cannot modify it.

---

## StateProvider

Use when:

- Counter
- Checkbox
- Selected tab
- Search query
- Temporary UI state

Example:

```dart
final tabProvider =
    StateProvider<int>((ref) => 0);
```

Explanation:

- Simple mutable state.
- No business logic required.

---

## NotifierProvider

Use when:

- Validation
- Business logic
- Multiple state updates
- Complex synchronous state

Example:

```dart
class CounterNotifier
    extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}
```

Explanation:

- Encapsulates state and related operations.

---

## FutureProvider

Use when:

- Loading configuration
- Read-only API calls
- Initial application data
- Static remote resources

Example:

```dart
final configProvider =
    FutureProvider<Config>((ref) async {
  return repository.loadConfig();
});
```

Explanation:

- Performs a one-time asynchronous operation.

---

## AsyncNotifierProvider

Use when:

- CRUD operations
- Authentication
- User profile management
- Refreshable API data
- Editable asynchronous state

Example:

```dart
class UserNotifier
    extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchUser();
  }
}
```

Explanation:

- Handles asynchronous loading and future updates.

---

## StreamProvider

Use when:

- Firebase streams
- Firestore snapshots
- WebSockets
- Sensor data
- Read-only live updates

Example:

```dart
final authProvider =
    StreamProvider<User?>((ref) {
  return repository.authChanges();
});
```

Explanation:

- Listens to an existing stream.

---

## StreamNotifierProvider

Use when:

- Chat applications
- Live dashboards
- Stream filtering
- Stream management
- Interactive real-time features

Example:

```dart
class ChatNotifier
    extends StreamNotifier<List<Message>> {
  @override
  Stream<List<Message>> build() {
    return repository.messages();
  }
}
```

Explanation:

- Owns both the stream and related business logic.

---

# Mental Model

```text
Immutable value
        │
        ▼
Provider

──────────────

Simple mutable value
        │
        ▼
StateProvider

──────────────

Complex synchronous state
        │
        ▼
Notifier

──────────────

Read-only Future
        │
        ▼
FutureProvider

──────────────

Editable async state
        │
        ▼
AsyncNotifier

──────────────

Read-only Stream
        │
        ▼
StreamProvider

──────────────

Managed Stream
        │
        ▼
StreamNotifier
```

---

# Real-World Mapping

| Scenario | Recommended Provider |
|----------|----------------------|
| App configuration | `Provider` |
| Repository injection | `Provider` |
| Theme mode | `StateProvider` |
| Selected tab | `StateProvider` |
| Shopping cart | `NotifierProvider` |
| Form validation | `NotifierProvider` |
| Login | `AsyncNotifierProvider` |
| User profile | `AsyncNotifierProvider` |
| REST API (read-only) | `FutureProvider` |
| Firebase Auth | `StreamProvider` |
| Firestore collection | `StreamProvider` |
| Chat room | `StreamNotifierProvider` |

---

# When to Use

Ask yourself these questions:

1. Is the value immutable?
2. Does it change?
3. Is it asynchronous?
4. Does it require business logic?
5. Does it come from a stream?
6. Will it grow in complexity?

The answers usually point to the correct provider.

---

# When NOT to Use

Avoid choosing providers based on familiarity alone.

For example:

- Don't use `Notifier` for a constant value.
- Don't use `AsyncNotifier` for a simple counter.
- Don't use `StateProvider` for complex business logic.
- Don't use `FutureProvider` if the state needs updates after loading.

More powerful providers are not always better.

---

# Best Practices

- Start with the simplest provider.
- Upgrade providers only when requirements change.
- Keep business logic inside notifiers.
- Use immutable state models.
- Prefer feature-based provider organization.
- Avoid premature complexity.

---

# Common Mistakes

## Using Notifier everywhere

Wrong:

```text
Notifier

Theme
Counter
Config
API URL
```

Explanation:

- Simple values don't benefit from complex providers.

Correct:

```text
Provider

Config
API URL

StateProvider

Theme

Notifier

Business Logic
```

Explanation:

- Match the provider to the problem.

---

## Using StateProvider for complex state

Wrong:

```dart
ref.read(userProvider.notifier).state = updatedUser;
```

Explanation:

- Validation and business logic end up in widgets.

Correct:

```dart
ref
    .read(userProvider.notifier)
    .updateProfile(updatedUser);
```

Explanation:

- The notifier owns state transitions.

---

## Starting with the most powerful provider

Wrong:

```text
Every feature

↓

AsyncNotifier
```

Explanation:

- Many features only need a simple `Provider` or `StateProvider`.

Correct:

```text
Start Simple

↓

Upgrade When Needed
```

Explanation:

- Complexity should grow with your application's requirements.

---

# Related APIs

- Provider
- StateProvider
- NotifierProvider
- FutureProvider
- AsyncNotifierProvider
- StreamProvider
- StreamNotifierProvider

---

# Summary

Choose the provider that matches the nature of your state. Use `Provider` for immutable values, `StateProvider` for simple mutable state, `Notifier` for complex synchronous logic, `FutureProvider` for read-only asynchronous data, `AsyncNotifier` for editable asynchronous state, `StreamProvider` for passive streams, and `StreamNotifier` for managed real-time data. Start with the simplest provider and scale up only as your application's needs evolve.