# FutureProvider vs AsyncNotifier

Choose between `FutureProvider` and `AsyncNotifier` based on whether your asynchronous data is read-only or needs to be modified after it is loaded.

---

`FutureProvider` is designed for one-time asynchronous computations, while `AsyncNotifier` manages asynchronous state throughout its lifecycle.

---

# What is it?

Both providers expose asynchronous state as an `AsyncValue<T>`, but they have different responsibilities.

- **FutureProvider** loads data asynchronously and exposes the result.
- **AsyncNotifier** loads data and also provides methods to update, refresh, and manage that data.

A simple way to think about them is:

- **FutureProvider = Read**
- **AsyncNotifier = Read + Write**

---

# Why does it exist?

Many asynchronous operations only need to fetch data once.

Examples include:

- App configuration
- Current weather
- Application version
- Remote feature flags

Other operations need ongoing state management:

- User authentication
- Shopping carts
- Todo lists
- User profiles
- Orders

Using `FutureProvider` for mutable state often results in business logic being moved into the UI, which makes the application harder to maintain.

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

- Performs a single asynchronous computation.
- Exposes an `AsyncValue<User>`.
- Does not expose methods to modify state.

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

- `build()` loads the initial state.
- Additional methods manage future state changes.

---

# Mental Model

```text
FutureProvider
──────────────

Load
 │
 ▼
Data

Done
```

```text
AsyncNotifier
─────────────

Load
 │
 ▼
Data
 │
 ├── Refresh
 ├── Save
 ├── Delete
 ├── Retry
 └── Update
```

`FutureProvider` completes after loading, while `AsyncNotifier` continues managing the state.

---

# Examples

## Simple Example

### FutureProvider

```dart
final appVersionProvider =
    FutureProvider<String>((ref) async {
  return repository.fetchVersion();
});
```

Explanation:

- The application version is read-only.
- No mutations are required.

---

### AsyncNotifier

```dart
class ProfileNotifier
    extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    return repository.fetchProfile();
  }

  Future<void> updateName(String name) async {
    state = await AsyncValue.guard(() async {
      return repository.updateName(name);
    });
  }
}
```

Explanation:

- The profile can be updated after it is loaded.
- Business logic remains inside the notifier.

---

## Real-World Example

### FutureProvider

```dart
final countriesProvider =
    FutureProvider<List<Country>>((ref) async {
  return repository.fetchCountries();
});
```

Explanation:

- The country list rarely changes during runtime.
- A read-only provider is sufficient.

---

### AsyncNotifier

```dart
class CartNotifier
    extends AsyncNotifier<Cart> {
  @override
  Future<Cart> build() async {
    return repository.loadCart();
  }

  Future<void> add(Product product) async {
    await repository.add(product);

    state = await AsyncValue.guard(() async {
      return repository.loadCart();
    });
  }
}
```

Explanation:

- The cart changes frequently.
- The notifier manages all cart operations.

---

# Feature Comparison

| Feature | FutureProvider | AsyncNotifier |
|----------|----------------|---------------|
| Async loading | ✅ | ✅ |
| Returns `AsyncValue` | ✅ | ✅ |
| Business logic | ❌ | ✅ |
| Mutable state | ❌ | ✅ |
| Refresh methods | ❌ | ✅ |
| Save/Delete operations | ❌ | ✅ |
| Best for read-only data | ✅ | ⚠️ Possible but unnecessary |
| Best for changing data | ❌ | ✅ |

---

# When to Use

Use **FutureProvider** when:

- Loading configuration
- Fetching static data
- Reading remote settings
- Displaying read-only information
- No updates are required

Use **AsyncNotifier** when:

- Users can edit data
- State needs refreshing
- CRUD operations are required
- Multiple asynchronous actions modify state
- Business logic is involved

---

# When NOT to Use

Avoid **FutureProvider** when:

- State changes after loading.
- Methods such as save, refresh, or delete are needed.

Avoid **AsyncNotifier** when:

- A simple one-time asynchronous request is all that's required.

Choose the simplest provider that satisfies your requirements.

---

# Best Practices

- Start with `FutureProvider` for read-only data.
- Migrate to `AsyncNotifier` when mutations are introduced.
- Keep asynchronous business logic inside the notifier.
- Use `AsyncValue.guard()` for error handling.
- Avoid placing asynchronous update logic in widgets.

---

# Common Mistakes

## Using FutureProvider for editable data

Wrong:

```dart
final profileProvider =
    FutureProvider<User>(...);
```

Explanation:

- There is no place to update the profile.
- Widgets often end up containing business logic.

Correct:

```dart
final profileProvider =
    AsyncNotifierProvider<
        ProfileNotifier,
        User>(
      ProfileNotifier.new,
    );
```

Explanation:

- The notifier owns loading and updates.

---

## Refreshing from the UI

Wrong:

```dart
ref.refresh(userProvider);
```

Explanation:

- The widget becomes responsible for state management.

Correct:

```dart
ref
    .read(userProvider.notifier)
    .refreshUser();
```

Explanation:

- Refresh logic stays inside the notifier.

---

## Putting repository calls in widgets

Wrong:

```dart
await repository.saveProfile(user);
```

Explanation:

- The UI becomes tightly coupled to the data layer.

Correct:

```dart
await ref
    .read(profileProvider.notifier)
    .saveProfile(user);
```

Explanation:

- The notifier coordinates repository operations.
- Widgets remain focused on presentation.

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

`FutureProvider` is best for read-only asynchronous data that is loaded once, while `AsyncNotifier` is designed for asynchronous state that changes over time. If your feature needs refresh, save, delete, retry, or other state-changing operations, `AsyncNotifier` is the recommended choice.