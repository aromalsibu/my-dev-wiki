# AsyncNotifierProvider

`AsyncNotifierProvider` is a provider that exposes an `AsyncNotifier`, allowing you to manage asynchronous state and business logic in a single place.

---

## What is it?

`AsyncNotifierProvider` is the recommended provider for **mutable asynchronous state** in modern Riverpod.

Unlike `FutureProvider`, which is designed for one-time asynchronous data fetching, `AsyncNotifierProvider` allows you to:

- Load asynchronous data
- Modify that data
- Expose methods
- Handle loading and error states
- Perform CRUD operations
- Encapsulate business logic

It combines the capabilities of:

- `FutureProvider`
- `NotifierProvider`

into a single provider.

---

## Why does it exist?

Consider a Todo application.

Initially, you fetch the list:

```text
Fetch Todos
```

Later, you also need to:

- Add todos
- Edit todos
- Delete todos
- Refresh the list
- Retry failed requests

With `FutureProvider`, this logic becomes scattered across multiple providers and widgets.

`AsyncNotifierProvider` centralizes everything inside one notifier.

Benefits include:

- Centralized business logic
- Automatic loading and error handling
- Mutable asynchronous state
- Better maintainability
- Cleaner architecture

---

## Syntax

### Creating an AsyncNotifier

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchUser();
  }
}
```

Explanation:

- `build()` performs the initial asynchronous load.
- The returned value becomes `AsyncData`.
- Errors automatically become `AsyncError`.

---

### Creating an AsyncNotifierProvider

```dart
final userProvider =
    AsyncNotifierProvider<UserNotifier, User>(
      UserNotifier.new,
    );
```

Explanation:

- Exposes the notifier.
- Widgets watch an `AsyncValue<User>`.
- Methods are accessed through `.notifier`.

---

### Reading the State

```dart
final user = ref.watch(userProvider);
```

Explanation:

- Returns an `AsyncValue<User>`.
- Supports loading, data, and error states.

---

### Calling Methods

```dart
ref.read(userProvider.notifier).refresh();
```

Explanation:

- `.notifier` returns the `UserNotifier`.
- Methods can modify asynchronous state.

---

## Mental Model

Think of `AsyncNotifier` as an asynchronous controller.

```text
             Widget
                │
                ▼
     AsyncNotifierProvider
                │
                ▼
         AsyncNotifier
        ┌───────┴────────┐
        ▼                ▼
 Business Logic     AsyncValue
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      Loading         Data          Error
```

The notifier owns both the asynchronous state and the logic that changes it.

---

## Examples

### Loading a User

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchUser();
  }
}
```

Explanation:

- Loads the user when the provider is first created.
- The UI automatically receives loading, data, and error states.

---

### Refreshing Data

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

- Displays a loading state.
- `AsyncValue.guard()` converts the result into `AsyncData` or `AsyncError`.

---

### Adding a Todo

```dart
class TodoNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    return repository.fetchTodos();
  }

  Future<void> addTodo(Todo todo) async {
    await repository.add(todo);

    state = await AsyncValue.guard(
      repository.fetchTodos,
    );
  }
}
```

Explanation:

- Performs an asynchronous update.
- Reloads the latest list after the operation.

---

### Removing a Todo

```dart
Future<void> removeTodo(Todo todo) async {
  await repository.delete(todo);

  state = await AsyncValue.guard(
    repository.fetchTodos,
  );
}
```

Explanation:

- Deletes the todo.
- Reloads the updated data.

---

## State Flow

```text
Provider Created
        │
        ▼
build()
        │
        ▼
 AsyncLoading
        │
        ▼
 Future Completes
        │
        ▼
 AsyncData
        │
        ▼
User Action
        │
        ▼
 Method Executes
        │
        ▼
 AsyncLoading
        │
        ▼
 AsyncData / AsyncError
```

Every asynchronous operation updates the provider's `AsyncValue`.

---

## AsyncNotifierProvider vs FutureProvider

| Feature | FutureProvider | AsyncNotifierProvider |
|---------|----------------|-----------------------|
| Initial async loading | ✅ | ✅ |
| Mutable state | ❌ | ✅ |
| Business logic | ❌ | ✅ |
| Exposes methods | ❌ | ✅ |
| CRUD operations | ❌ | ✅ |
| AsyncValue support | ✅ | ✅ |

---

## When to Use

Use `AsyncNotifierProvider` for:

- Authentication
- Todo applications
- User profiles
- Shopping carts
- REST APIs
- CRUD operations
- Form submission
- Data synchronization
- Any mutable asynchronous state

It is the recommended provider for most asynchronous application state.

---

## When NOT to Use

Avoid `AsyncNotifierProvider` when:

- Only a single asynchronous value is required.
- No business logic exists.
- The data comes from a continuous stream.
- The value is synchronous.

Instead, use:

| Scenario | Recommended Provider |
|----------|----------------------|
| One-time async fetch | `FutureProvider` |
| Continuous updates | `StreamProvider` |
| Mutable synchronous state | `NotifierProvider` |
| Read-only synchronous value | `Provider` |

---

## Best Practices

- Keep all asynchronous business logic inside the notifier.
- Use `AsyncValue.guard()` to simplify error handling.
- Expose meaningful methods instead of updating `state` from widgets.
- Keep `build()` focused on the initial load.
- Return immutable models whenever possible.

---

## Common Mistakes

### 1. Using FutureProvider for Mutable Data

❌ Wrong

```dart
final todosProvider =
    FutureProvider<List<Todo>>((ref) async {
  return repository.fetchTodos();
});
```

Why it's wrong:

- `FutureProvider` cannot expose methods.
- Updating the list becomes difficult.

✔ Correct

```dart
final todosProvider =
    AsyncNotifierProvider<
      TodoNotifier,
      List<Todo>
    >(TodoNotifier.new);
```

---

### 2. Updating State from Widgets

❌ Wrong

```dart
ref.read(userProvider.notifier).state =
    const AsyncLoading();
```

Why it's wrong:

- Widgets should not manipulate notifier state directly.
- Business logic belongs inside the notifier.

✔ Correct

```dart
ref.read(userProvider.notifier).refresh();
```

Expose methods that update state internally.

---

### 3. Forgetting Error Handling

❌ Wrong

```dart
Future<void> refresh() async {
  state = AsyncData(
    await repository.fetchUser(),
  );
}
```

Why it's wrong:

- Exceptions are not converted into `AsyncError`.
- Errors may escape unexpectedly.

✔ Correct

```dart
Future<void> refresh() async {
  state = const AsyncLoading();

  state = await AsyncValue.guard(
    repository.fetchUser,
  );
}
```

Use `AsyncValue.guard()` to handle exceptions consistently.

---

## Recommendation

> **Use `AsyncNotifierProvider` for almost all mutable asynchronous state.**
>
> If your application loads data and later needs to create, update, delete, or refresh that data, `AsyncNotifierProvider` is the recommended choice over `FutureProvider`.

---

## Related APIs

- AsyncNotifier
- FutureProvider
- StreamNotifierProvider
- NotifierProvider
- AsyncValue
- AsyncValue.guard()
- ref.watch()
- ref.read()

---

## Summary

`AsyncNotifierProvider` is Riverpod's recommended provider for mutable asynchronous state. It combines asynchronous loading, state management, and business logic inside an `AsyncNotifier`, exposing an `AsyncValue` to consumers while providing methods for updates, CRUD operations, and error handling. For most asynchronous application state, it is the preferred provider type in modern Riverpod.