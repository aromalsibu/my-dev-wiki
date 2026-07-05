# FutureProvider → AsyncNotifier

Migrate from `FutureProvider` to `AsyncNotifier` when your asynchronous state needs to support mutations, refreshes, or business logic.

---

`FutureProvider` is ideal for read-only asynchronous data, while `AsyncNotifier` is designed for asynchronous state that can change over time.

---

# What is it?

Both `FutureProvider` and `AsyncNotifier` manage asynchronous state, but they serve different purposes.

- **FutureProvider** performs a single asynchronous computation and exposes its result.
- **AsyncNotifier** manages asynchronous state **and** allows that state to be modified after it has been loaded.

Think of `FutureProvider` as **read-only**, while `AsyncNotifier` is **read and write**.

---

# Why does it exist?

`FutureProvider` is excellent for loading data, but it has an important limitation:

- It cannot expose methods to modify its state.

As applications grow, asynchronous state often needs to:

- Refresh data
- Save changes
- Delete items
- Retry failed requests
- Optimistically update the UI

`AsyncNotifier` was introduced to support these use cases while keeping asynchronous state inside a single class.

---

# Syntax

## FutureProvider

```dart
final userProvider =
    FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- Loads data once.
- Exposes an `AsyncValue<User>`.
- Cannot mutate its own state.

---

## AsyncNotifier

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchUser();
  }
}

final userProvider =
    AsyncNotifierProvider<UserNotifier, User>(
      UserNotifier.new,
    );
```

Explanation:

- `build()` performs the initial asynchronous load.
- Additional methods can modify the state later.

---

## Updating state

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
```

Explanation:

- `refreshUser()` updates the provider after initialization.
- This is not possible with `FutureProvider`.

---

# Mental Model

FutureProvider:

```text
Request
   │
   ▼
Response
   │
   ▼
Finished
```

AsyncNotifier:

```text
Load
 │
 ▼
State
 │
 ├── Refresh
 ├── Update
 ├── Delete
 └── Retry
```

`AsyncNotifier` manages the entire lifecycle of asynchronous state instead of a single request.

---

# Examples

## Simple Example

### Before

```dart
final profileProvider =
    FutureProvider<Profile>((ref) async {
  return repository.loadProfile();
});
```

Explanation:

- The profile can only be loaded.
- It cannot be updated directly.

---

### After

```dart
class ProfileNotifier
    extends AsyncNotifier<Profile> {
  @override
  Future<Profile> build() async {
    return repository.loadProfile();
  }

  Future<void> reload() async {
    state = const AsyncLoading();

    state = await AsyncValue.guard(() async {
      return repository.loadProfile();
    });
  }
}
```

Explanation:

- The notifier can reload the profile whenever needed.
- State management stays in one place.

---

## Real-World Example

```dart
class TodoNotifier
    extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    return repository.fetchTodos();
  }

  Future<void> addTodo(Todo todo) async {
    await repository.add(todo);

    state = await AsyncValue.guard(() async {
      return repository.fetchTodos();
    });
  }
}
```

Explanation:

- Loading and updating todos are handled by the same notifier.
- The UI simply watches the provider.

---

# When to Use

Use `AsyncNotifier` when you need to:

- Refresh data
- Save changes
- Delete records
- Retry failed requests
- Perform optimistic updates
- Manage asynchronous business logic
- Expose methods to the UI

---

# When NOT to Use

Continue using `FutureProvider` when:

- Data is read-only.
- Only a single asynchronous request is required.
- No mutations are needed.
- The provider simply loads configuration or reference data.

Use the simplest provider that satisfies your requirements.

---

# Best Practices

- Prefer `FutureProvider` for read-only data.
- Use `AsyncNotifier` when state can change.
- Keep asynchronous business logic inside the notifier.
- Use `AsyncValue.guard()` for error handling.
- Expose meaningful methods instead of manipulating `state` from the UI.

---

# Common Mistakes

## Using FutureProvider for mutable state

Wrong:

```dart
final todosProvider =
    FutureProvider<List<Todo>>(...);
```

Explanation:

- There is no place to add, edit, or remove todos.

Correct:

```dart
final todosProvider =
    AsyncNotifierProvider<
        TodoNotifier,
        List<Todo>>(
      TodoNotifier.new,
    );
```

Explanation:

- The notifier owns both loading and mutations.

---

## Refreshing manually everywhere

Wrong:

```dart
ref.refresh(userProvider);
```

Explanation:

- The UI becomes responsible for state management.

Correct:

```dart
ref
    .read(userProvider.notifier)
    .refreshUser();
```

Explanation:

- Business logic stays inside the notifier.

---

## Putting repository logic in the UI

Wrong:

```dart
await repository.saveUser(user);
```

Explanation:

- The widget directly accesses the data layer.

Correct:

```dart
await ref
    .read(userProvider.notifier)
    .saveUser(user);
```

Explanation:

- The notifier coordinates repository operations.
- The UI remains focused on presentation.

---

# Related APIs

- FutureProvider
- AsyncNotifier
- AsyncNotifierProvider
- AsyncValue
- `AsyncValue.guard()`
- `ref.watch()`
- `ref.read()`

---

# Summary

`FutureProvider` is best for read-only asynchronous data, while `AsyncNotifier` is the preferred choice for asynchronous state that needs to change over time. When your provider needs methods such as refresh, save, delete, or retry, migrating to `AsyncNotifier` results in cleaner, more maintainable business logic and a better separation of concerns.