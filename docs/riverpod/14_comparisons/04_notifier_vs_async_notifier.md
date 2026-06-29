# Notifier vs AsyncNotifier

Choose between `Notifier` and `AsyncNotifier` based on whether your state is synchronous or asynchronous.

---

`Notifier` manages synchronous state, while `AsyncNotifier` manages asynchronous state represented by `AsyncValue<T>`.

---

# What is it?

Both `Notifier` and `AsyncNotifier` are Riverpod's modern APIs for managing application state.

They share the same architecture and programming model but differ in how state is initialized and updated.

- **Notifier** manages synchronous state.
- **AsyncNotifier** manages asynchronous state.

As a rule of thumb:

- **Notifier = Immediate state**
- **AsyncNotifier = Future-based state**

---

# Why does it exist?

Applications contain different kinds of state.

Some state is available immediately:

- Counter values
- Theme mode
- Selected tab
- Shopping cart quantity
- UI preferences

Other state requires asynchronous work:

- Network requests
- Database queries
- Authentication
- File operations
- Local storage

Riverpod provides two specialized APIs so each type of state can be managed naturally.

---

# Syntax

## Notifier

```dart
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}

final counterProvider =
    NotifierProvider<CounterNotifier, int>(
      CounterNotifier.new,
    );
```

Explanation:

- `build()` returns the initial state immediately.
- State updates are synchronous.

---

## AsyncNotifier

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchUser();
  }

  Future<void> refreshUser() async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() async {
      return repository.fetchUser();
    });
  }
}

final userProvider =
    AsyncNotifierProvider<UserNotifier, User>(
      UserNotifier.new,
    );
```

Explanation:

- `build()` returns a `Future`.
- State is exposed as an `AsyncValue<User>`.
- Methods typically perform asynchronous work.

---

# Mental Model

```text
Notifier
────────

build()
   │
   ▼
State
   │
   ▼
Immediate updates
```

```text
AsyncNotifier
─────────────

build()
   │
   ▼
Loading
   │
   ▼
Data / Error
   │
   ▼
Async updates
```

`Notifier` updates immediately, while `AsyncNotifier` moves through loading, data, and error states.

---

# Examples

## Simple Example

### Notifier

```dart
class ThemeNotifier
    extends Notifier<ThemeMode> {
  @override
  ThemeMode build() => ThemeMode.light;

  void toggle() {
    state = state == ThemeMode.light
        ? ThemeMode.dark
        : ThemeMode.light;
  }
}
```

Explanation:

- Theme changes happen instantly.
- No asynchronous work is involved.

---

### AsyncNotifier

```dart
class ProfileNotifier
    extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchProfile();
  }
}
```

Explanation:

- The profile must be loaded from an external source.
- Consumers receive an `AsyncValue<User>`.

---

## Real-World Example

### Notifier

```dart
class CartNotifier
    extends Notifier<Cart> {
  @override
  Cart build() => Cart.empty();

  void add(Product product) {
    state = state.add(product);
  }
}
```

Explanation:

- Cart updates are immediate.
- No asynchronous operations are required.

---

### AsyncNotifier

```dart
class OrderNotifier
    extends AsyncNotifier<List<Order>> {
  @override
  Future<List<Order>> build() async {
    return repository.fetchOrders();
  }

  Future<void> cancelOrder(
    String id,
  ) async {
    await repository.cancel(id);

    state = await AsyncValue.guard(() async {
      return repository.fetchOrders();
    });
  }
}
```

Explanation:

- Orders come from a remote API.
- State updates depend on asynchronous operations.

---

# Feature Comparison

| Feature | Notifier | AsyncNotifier |
|----------|----------|----------------|
| Synchronous state | ✅ | ❌ |
| Asynchronous state | ❌ | ✅ |
| Returns `AsyncValue` | ❌ | ✅ |
| Loading state | ❌ | ✅ |
| Error state | ❌ | ✅ |
| Immediate updates | ✅ | ⚠️ After async work |
| Best for local state | ✅ | ❌ |
| Best for API-backed state | ❌ | ✅ |

---

# When to Use

Use **Notifier** when:

- Managing counters
- Theme mode
- Form state
- Local preferences
- Selection state
- Shopping cart quantities

Use **AsyncNotifier** when:

- Calling APIs
- Authentication
- Database operations
- Loading files
- CRUD operations
- Refreshable asynchronous state

---

# When NOT to Use

Avoid **Notifier** when:

- Initial state depends on asynchronous work.

Avoid **AsyncNotifier** when:

- State is entirely synchronous.
- No loading or error states are needed.

Choose the simplest provider that matches the nature of your state.

---

# Best Practices

- Use `Notifier` for synchronous business logic.
- Use `AsyncNotifier` for asynchronous business logic.
- Keep widgets free of business logic.
- Use `AsyncValue.guard()` in `AsyncNotifier` methods.
- Prefer immutable state models in both cases.

---

# Common Mistakes

## Using Notifier for API calls

Wrong:

```dart
class UserNotifier extends Notifier<User> {
  @override
  User build() {
    return repository.fetchUser();
  }
}
```

Explanation:

- `fetchUser()` is asynchronous and cannot be returned directly from `build()`.

Correct:

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

- `AsyncNotifier` is designed for asynchronous initialization.

---

## Using AsyncNotifier for simple counters

Wrong:

```dart
class CounterNotifier
    extends AsyncNotifier<int> {
  @override
  Future<int> build() async => 0;
}
```

Explanation:

- The state is synchronous.
- Introducing asynchronous behavior adds unnecessary complexity.

Correct:

```dart
class CounterNotifier
    extends Notifier<int> {
  @override
  int build() => 0;
}
```

Explanation:

- `Notifier` is simpler and more appropriate.

---

## Ignoring loading and error states

Wrong:

```dart
final user = ref.watch(userProvider).value;
```

Explanation:

- Loading and error cases are ignored.

Correct:

```dart
final user = ref.watch(userProvider);

return user.when(
  data: UserView.new,
  loading: LoadingView.new,
  error: ErrorView.new,
);
```

Explanation:

- Handle every asynchronous state explicitly.

---

# Related APIs

- Notifier
- AsyncNotifier
- NotifierProvider
- AsyncNotifierProvider
- AsyncValue
- FutureProvider

---

# Summary

`Notifier` is the preferred choice for synchronous state that is available immediately, while `AsyncNotifier` is designed for asynchronous state that involves loading, success, and error states. Choose `Notifier` for local state and `AsyncNotifier` whenever your state depends on asynchronous operations such as network requests or database access.