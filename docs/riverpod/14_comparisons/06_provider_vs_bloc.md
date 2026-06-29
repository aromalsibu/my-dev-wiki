# Provider vs Bloc

Compare Riverpod and Bloc to choose the state management solution that best fits your application's architecture and team preferences.

---

Riverpod and Bloc are both mature Flutter state management solutions, but they encourage different approaches to organizing application state.

---

# What is it?

Both Riverpod and Bloc separate business logic from the UI, but they use different programming models.

**Riverpod** is a provider-based dependency injection and state management framework.

**Bloc** is an event-driven state management library based on the Business Logic Component (BLoC) pattern.

Both support scalable applications, but they differ in how state changes are represented.

---

# Why does it exist?

Flutter offers multiple state management solutions because applications have different requirements.

Some teams prefer:

- Simple APIs
- Flexible architecture
- Minimal boilerplate

Others prefer:

- Strict architectural rules
- Event-driven workflows
- Highly structured state transitions

Riverpod and Bloc address these different preferences.

---

# Syntax

## Riverpod

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

- State changes are performed through methods.
- Widgets interact with providers using `ref.watch()` and `ref.read()`.

---

## Bloc

```dart
class IncrementPressed extends CounterEvent {}

class CounterBloc
    extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<IncrementPressed>((event, emit) {
      emit(state + 1);
    });
  }
}
```

Explanation:

- UI dispatches events.
- The Bloc reacts to events and emits new states.

---

# Mental Model

Riverpod:

```text
Widget
   │
   ▼
Notifier
   │
   ▼
State
```

Bloc:

```text
Widget
   │
   ▼
Event
   │
   ▼
Bloc
   │
   ▼
State
```

Riverpod calls methods directly, while Bloc routes all changes through events.

---

# Examples

## Simple Example

### Riverpod

```dart
ref.read(counterProvider.notifier).increment();
```

Explanation:

- The widget calls a method on the notifier.

---

### Bloc

```dart
context.read<CounterBloc>()
    .add(IncrementPressed());
```

Explanation:

- The widget dispatches an event.
- The Bloc decides how to handle it.

---

## Real-World Example

### Riverpod

```dart
class AuthNotifier
    extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async {
    return repository.currentUser();
  }

  Future<void> login(
    String email,
    String password,
  ) async {
    state = await AsyncValue.guard(() async {
      return repository.login(email, password);
    });
  }
}
```

Explanation:

- Business logic lives inside the notifier.
- State changes happen through methods.

---

### Bloc

```text
LoginButton
      │
      ▼
LoginRequested Event
      │
      ▼
AuthenticationBloc
      │
      ▼
Loading
      │
      ▼
Authenticated
```

Explanation:

- Every state transition is triggered by an event.
- The event-driven workflow is explicit.

---

# Feature Comparison

| Feature | Riverpod | Bloc |
|----------|----------|------|
| Dependency injection | ✅ Built in | ❌ External solution often used |
| Event-based architecture | ❌ | ✅ |
| Business logic classes | ✅ Notifier | ✅ Bloc/Cubit |
| Async state support | ✅ AsyncNotifier | ✅ Bloc |
| Compile-time safety | ✅ | ✅ |
| Boilerplate | Low | Higher |
| Learning curve | Moderate | Moderate to High |
| Flutter dependency | ❌ Can run in pure Dart | ❌ Can run in pure Dart |

---

# When to Use

Choose **Riverpod** when you want:

- Flexible architecture
- Built-in dependency injection
- Less boilerplate
- Modern provider-based APIs
- Easy testing
- Simple state composition

Choose **Bloc** when you want:

- Event-driven architecture
- Explicit state transitions
- Strict architectural patterns
- Teams already experienced with Bloc
- Extensive event logging

---

# When NOT to Use

Riverpod may not be the best fit if your team relies heavily on event sourcing or already has a mature Bloc-based architecture.

Bloc may not be the best fit for smaller projects where its additional boilerplate provides little benefit.

The best choice often depends on your team's experience and the application's architectural requirements.

---

# Best Practices

### Riverpod

- Keep business logic in notifiers.
- Compose providers instead of nesting logic.
- Use dependency injection through providers.

### Bloc

- Keep events focused.
- Emit immutable states.
- Avoid putting business logic inside widgets.
- Keep event handlers small.

---

# Common Mistakes

## Choosing Bloc for very simple state

Wrong:

```text
Button
   │
Event
   │
Bloc
   │
State
```

Explanation:

- The event layer may be unnecessary for simple counters or toggles.

Correct:

```text
Button
   │
Notifier
   │
State
```

Explanation:

- Riverpod's notifier API is often simpler for straightforward state changes.

---

## Putting business logic in widgets

Wrong:

```dart
onPressed: () async {
  await repository.login();
}
```

Explanation:

- The widget becomes responsible for business logic.

Correct:

```dart
ref
    .read(authProvider.notifier)
    .login(email, password);
```

Explanation:

- Riverpod centralizes business logic inside the notifier.

---

## Overusing events

Wrong:

```text
IncrementPressed
IncrementCompleted
IncrementFinished
```

Explanation:

- Extremely granular events can increase complexity without providing additional value.

Correct:

```text
IncrementPressed
```

Explanation:

- Keep events meaningful and focused.

---

# Related APIs

## Riverpod

- Provider
- Notifier
- AsyncNotifier
- ProviderScope

## Bloc

- Bloc
- Cubit
- BlocProvider
- BlocBuilder
- BlocListener

---

# Summary

Riverpod and Bloc are both excellent choices for Flutter state management. Riverpod emphasizes providers, dependency injection, and direct method calls with minimal boilerplate, while Bloc focuses on explicit event-driven state transitions and structured workflows. Riverpod is often preferred for its flexibility and simplicity, whereas Bloc is a strong fit for teams that favor strict architectural patterns and event-based design.