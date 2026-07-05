# State Management Patterns

Common architectural patterns for organizing and managing state in Riverpod applications.

---

# What is it?

State management patterns are proven ways of structuring your application's state, business logic, and providers.

Riverpod is intentionally flexible—it doesn't force a particular architecture. Instead, it provides the building blocks that work well with patterns such as feature-first architecture, repository pattern, clean architecture, and unidirectional data flow.

Choosing a consistent pattern makes applications easier to understand, test, and scale.

---

# Why does it exist?

Without a consistent pattern, applications often suffer from:

- Business logic inside widgets
- Duplicate code
- Difficult testing
- Tight coupling
- Poor maintainability

A well-defined pattern keeps responsibilities separated and makes the codebase predictable.

---

# Syntax

## Keep business logic inside a Notifier

```dart
final counterProvider =
    NotifierProvider<CounterNotifier, int>(
      CounterNotifier.new,
    );

class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() {
    state++;
  }
}
```

Explanation:

- The widget interacts with the provider.
- Business logic remains inside the `Notifier`.

---

## Keep repositories separate

```dart
final userRepositoryProvider =
    Provider<UserRepository>((ref) {
  return UserRepository();
});

final userProvider =
    FutureProvider<User>((ref) async {
  final repository = ref.watch(userRepositoryProvider);

  return repository.fetchUser();
});
```

Explanation:

- The repository handles data access.
- The provider coordinates state.
- UI never talks directly to the repository.

---

## Compose providers

```dart
final fullNameProvider = Provider<String>((ref) {
  final user = ref.watch(userProvider);

  return '${user.firstName} ${user.lastName}';
});
```

Explanation:

- Derived providers build on existing providers.
- Avoid duplicating state.

---

# Mental Model

```text
UI
 │
 ▼
Provider
 │
 ▼
Notifier
 │
 ▼
Repository
 │
 ▼
API / Database
```

Each layer has one responsibility:

- UI displays state.
- Providers expose state.
- Notifiers manage business logic.
- Repositories access data sources.

---

# Examples

## Simple Example

```dart
class TodoNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];

  void add(Todo todo) {
    state = [...state, todo];
  }
}
```

Explanation:

- State changes are centralized inside the notifier.
- Widgets remain focused on presentation.

---

## Real-World Example

```dart
final authRepositoryProvider =
    Provider<AuthRepository>((ref) {
  return AuthRepository();
});

final authProvider =
    AsyncNotifierProvider<
        AuthNotifier,
        User?>(
      AuthNotifier.new,
    );
```

Explanation:

- Authentication logic lives in the notifier.
- Data access stays in the repository.
- UI simply observes the provider.

---

# When to Use

Use these patterns when:

- Building medium or large applications.
- Working with multiple developers.
- Following Clean Architecture.
- Writing testable business logic.
- Separating UI from data access.

---

# When NOT to Use

Small prototypes or learning projects may not need multiple architectural layers.

Avoid introducing repositories or abstractions that don't provide clear value.

Choose a level of complexity appropriate for the size of the application.

---

# Best Practices

- Keep widgets free of business logic.
- Let notifiers own mutable state.
- Use repositories for data access.
- Compose providers instead of duplicating state.
- Keep providers focused on one responsibility.
- Prefer immutable state models.
- Organize code by feature.

---

# Common Mistakes

## Putting business logic inside widgets

Wrong:

```dart
ElevatedButton(
  onPressed: () async {
    final user = await repository.fetchUser();
    // Update UI
  },
)
```

Explanation:

- The widget becomes responsible for both UI and business logic.
- Testing becomes more difficult.

Correct:

```dart
ElevatedButton(
  onPressed: () {
    ref.read(userProvider.notifier).loadUser();
  },
)
```

Explanation:

- The widget delegates the work to the notifier.
- Business logic remains centralized.

---

## Accessing repositories directly from the UI

Wrong:

```dart
final repository = UserRepository();

await repository.fetchUser();
```

Explanation:

- The UI becomes tightly coupled to the data layer.

Correct:

```dart
final user = ref.watch(userProvider);
```

Explanation:

- The provider manages state and data access.
- The UI only consumes state.

---

## Duplicating state across providers

Wrong:

```dart
final profileProvider = Provider<User>(...);

final accountProvider = Provider<User>(...);
```

Explanation:

- Multiple providers own the same data.
- Keeping them synchronized becomes difficult.

Correct:

```dart
final profileProvider = Provider<Profile>((ref) {
  final user = ref.watch(userProvider);

  return user.profile;
});
```

Explanation:

- One provider owns the state.
- Other providers derive values from it.

---

# Related APIs

- Notifier
- AsyncNotifier
- Provider
- NotifierProvider
- AsyncNotifierProvider
- FutureProvider
- `ref.watch()`
- `ref.read()`

---

# Summary

A good state management pattern separates presentation, business logic, and data access into distinct layers. Let widgets display state, notifiers manage state changes, providers expose state, and repositories handle external data sources. Following consistent patterns results in applications that are easier to test, maintain, and scale.