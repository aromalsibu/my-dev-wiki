# AsyncNotifier

`AsyncNotifier<T>` is Riverpod’s base class for managing asynchronous state and business logic that is exposed through an `AsyncNotifierProvider`.

---

## What is it?

`AsyncNotifier` is used to manage **asynchronous state with mutable behavior**.

It allows you to:

- Fetch data asynchronously
- Modify or refresh that data
- Handle loading and error states automatically
- Expose methods that trigger async operations

The state is always wrapped in an `AsyncValue<T>`, which represents:

- Loading
- Data
- Error

Unlike `FutureProvider`, which runs once and stays static, `AsyncNotifier` supports **ongoing, controlled async state updates**.

---

## Why does it exist?

Real-world async data is rarely static.

For example:

- Fetching a user profile
- Updating user data
- Refreshing a list of items
- Handling login/logout
- Syncing remote data

With `FutureProvider`, you can fetch data, but:

- You cannot easily refresh with logic
- You cannot expose actions (add/update/delete)
- Business logic gets scattered across UI

`AsyncNotifier` solves this by centralizing:

- Async loading
- State transitions
- Business logic
- Side effects

It gives you full control over async workflows.

---

## Syntax

### Creating an AsyncNotifier

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return await repository.fetchUser();
  }
}
```

Explanation:

- `build()` runs automatically when provider is first used.
- It must return a `Future<T>`.
- The result becomes `AsyncData`.

---

### Connecting with AsyncNotifierProvider

```dart
final userProvider =
    AsyncNotifierProvider<UserNotifier, User>(
  UserNotifier.new,
);
```

Explanation:

- Exposes the notifier to Riverpod.
- State is always `AsyncValue<User>`.

---

### Reading State

```dart
final user = ref.watch(userProvider);
```

Explanation:

- Returns `AsyncValue<User>`.
- Includes loading, error, and data states.

---

### Calling Methods

```dart
ref.read(userProvider.notifier).refreshUser();
```

Explanation:

- Accesses the notifier.
- Triggers custom async logic.

---

## Mental Model

Think of `AsyncNotifier` as an **async controller with state awareness**.

```text
          Widget
             │
             ▼
  AsyncNotifierProvider
             │
             ▼
       AsyncNotifier
     ┌──────┴──────┐
     ▼             ▼
 async logic   AsyncValue
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
  Loading       Data       Error
```

It controls both:
- how data is fetched
- how state changes over time

---

## Examples

### Fetch User

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchUser();
  }
}
```

Explanation:

- Automatically fetches user data.
- UI reacts to loading/data/error states.

---

### Refresh Data

```dart
Future<void> refreshUser() async {
  state = const AsyncLoading();

  state = await AsyncValue.guard(
    repository.fetchUser,
  );
}
```

Explanation:

- Forces loading state.
- Safely handles errors using `AsyncValue.guard`.

---

### Todo List

```dart
class TodoNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    return repository.fetchTodos();
  }

  Future<void> addTodo(Todo todo) async {
    await repository.addTodo(todo);

    state = await AsyncValue.guard(
      repository.fetchTodos,
    );
  }
}
```

Explanation:

- Fetches todos initially.
- Allows adding todos and refreshing state.

---

### Authentication Flow

```dart
class AuthNotifier extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async {
    return repository.currentUser();
  }

  Future<void> login(String email, String password) async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() {
      return repository.login(email, password);
    });
  }

  Future<void> logout() async {
    await repository.logout();
    state = const AsyncData(null);
  }
}
```

Explanation:

- Handles login/logout flows.
- Maintains async state transitions.

---

## State Flow

```text
build() called
      │
      ▼
 AsyncLoading
      │
      ▼
 Future completes
      │
 ┌────┴────┐
 ▼         ▼
Data     Error
      │
      ▼
Method called
      │
      ▼
 AsyncLoading
      │
      ▼
 New AsyncData / AsyncError
```

Each async operation updates the `AsyncValue`.

---

## When to Use

Use `AsyncNotifier` when:

- You need mutable async state
- Data changes after initial fetch
- You need actions like add/update/delete
- You want centralized async logic
- You are building real app workflows (auth, CRUD, etc.)

---

## When NOT to Use

Avoid `AsyncNotifier` when:

- Data is static → use `FutureProvider`
- Data is read-only stream → use `StreamProvider`
- State is synchronous → use `Notifier`
- No business logic is needed

---

## Best Practices

- Always use `AsyncValue.guard()` for safe async handling
- Keep async logic inside notifier, not UI
- Return immutable models
- Use methods instead of direct state manipulation
- Keep `build()` simple and focused on initial load

---

## Common Mistakes

### 1. Using FutureProvider Instead of AsyncNotifier

❌ Wrong

```dart
final userProvider = FutureProvider((ref) async {
  return repository.fetchUser();
});
```

Why it's wrong:

- Cannot add methods like refresh or update.
- Business logic leaks into UI.

✔ Correct

```dart
final userProvider =
    AsyncNotifierProvider<UserNotifier, User>(
  UserNotifier.new,
);
```

---

### 2. Updating State Without AsyncValue.guard

❌ Wrong

```dart
state = AsyncData(await repository.fetchUser());
```

Why it's wrong:

- Does not handle exceptions safely.

✔ Correct

```dart
state = await AsyncValue.guard(
  repository.fetchUser,
);
```

---

### 3. Putting Async Logic in Widgets

❌ Wrong

```dart
onPressed: () async {
  await repository.fetchUser();
}
```

Why it's wrong:

- Breaks separation of concerns.
- No state tracking.

✔ Correct

```dart
ref.read(userProvider.notifier).refreshUser();
```

---

## Related APIs

- AsyncNotifierProvider
- FutureProvider
- StreamProvider
- AsyncValue
- AsyncValue.guard
- ref.watch()
- ref.read()

---

## Summary

`AsyncNotifier` is Riverpod’s core tool for managing asynchronous state with business logic. It replaces scattered async code with a centralized controller that handles loading, data, error states, and user-triggered async actions, making it the recommended choice for real-world async application workflows.