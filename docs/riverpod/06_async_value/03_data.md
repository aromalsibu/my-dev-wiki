# Data

`AsyncData<T>` represents the successful state of an `AsyncValue`, containing the value produced by a completed asynchronous operation.

---

## What is it?

`AsyncData<T>` is one of the three possible states of an `AsyncValue`.

It indicates that an asynchronous operation has completed successfully and the requested value is now available.

For example:

- An API request returned data.
- A database query completed successfully.
- A file was loaded.
- A stream emitted a new value.

Unlike `AsyncLoading` or `AsyncError`, `AsyncData` always contains a valid value of type `T`.

---

## Why does it exist?

After an asynchronous operation finishes successfully, your application needs a safe and consistent way to access the result.

Instead of manually checking:

- Is the request finished?
- Did an error occur?
- Is the value null?

Riverpod wraps the successful result inside `AsyncData`, making it easy to build the UI.

This provides:

- Type safety
- Predictable state handling
- Consistent APIs across all asynchronous providers

---

## How Data Works

When an asynchronous operation succeeds, Riverpod transitions from `AsyncLoading` to `AsyncData`.

```text
Request Starts
      │
      ▼
AsyncLoading
      │
      ▼
Request Succeeds
      │
      ▼
 AsyncData
```

The value is now available to the application.

---

## Syntax

### FutureProvider

```dart
final userProvider = FutureProvider<User>((ref) async {
  return repository.fetchUser();
});
```

Explanation:

- Once `fetchUser()` completes successfully, Riverpod creates an `AsyncData<User>`.

---

### Reading Data with when()

```dart
final user = ref.watch(userProvider);

return user.when(
  loading: () => const CircularProgressIndicator(),
  data: (user) => Text(user.name),
  error: (error, stackTrace) {
    return Text(error.toString());
  },
);
```

Explanation:

- The `data` callback receives the successful value.
- No additional casting or null checks are required.

---

### Accessing value

```dart
if (user.hasValue) {
  print(user.value);
}
```

Explanation:

- `hasValue` indicates whether data is available.
- `value` returns the wrapped data.

---

## Data Lifecycle

```text
AsyncLoading
      │
      ▼
AsyncData
      │
      ▼
UI Displays Data
```

The provider remains in the data state until another refresh or update occurs.

---

## Mental Model

Think of `AsyncData` as a **delivered package**.

```text
Order Placed
      │
      ▼
Shipping
      │
      ▼
Delivered
```

Once the package arrives:

- The waiting is over.
- No error occurred.
- The requested item is available.

`AsyncData` represents that delivered result.

---

## Examples

### Display User Information

```dart
user.when(
  loading: () => const CircularProgressIndicator(),
  data: (user) {
    return Text(user.name);
  },
  error: (error, stackTrace) {
    return Text(error.toString());
  },
);
```

Explanation:

- Displays the user's name after a successful request.

---

### Display Product List

```dart
products.when(
  loading: LoadingView.new,
  data: ProductGrid.new,
  error: ErrorView.new,
);
```

Explanation:

- Builds the product grid only when products are available.

---

### Using hasValue

```dart
if (user.hasValue) {
  return Text(user.value!.name);
}
```

Explanation:

- Safely checks whether data exists before accessing it.
- Useful for simple conditional rendering.

---

### Real-World Example

```dart
final profile = ref.watch(profileProvider);

return profile.when(
  loading: ProfileSkeleton.new,
  data: ProfileView.new,
  error: ErrorScreen.new,
);
```

Explanation:

- The profile screen is shown only after successful loading.

---

## When to Use

Use the `data` state when:

- Displaying API responses.
- Showing database records.
- Rendering stream values.
- Building successful asynchronous UIs.
- Working with completed async operations.

---

## When NOT to Use

Avoid accessing data directly when:

- The provider is still loading.
- The provider has failed.
- The value hasn't been verified.

Instead:

- Use `when()`.
- Check `hasValue`.
- Handle errors appropriately.

---

## Best Practices

- Prefer `when()` for handling data.
- Treat `AsyncData` as immutable.
- Display meaningful content after successful loading.
- Keep UI logic separate from business logic.
- Avoid force-unwrapping values without checking state.

---

## Common Mistakes

### 1. Force-Unwrapping value

❌ Wrong

```dart
print(user.value!.name);
```

Why it's wrong:

- The provider may still be loading or in an error state.

✔ Correct

```dart
user.when(
  loading: () => const CircularProgressIndicator(),
  data: (user) => Text(user.name),
  error: (error, stackTrace) {
    return Text(error.toString());
  },
);
```

---

### 2. Ignoring Error State

❌ Wrong

```dart
if (user.hasValue) {
  return Text(user.value!.name);
}

return const SizedBox();
```

Why it's wrong:

- Errors are silently ignored.
- Users receive no feedback.

✔ Correct

Always handle loading, data, and error states.

---

### 3. Using Data Without Checking

❌ Wrong

```dart
final user = ref.watch(userProvider);

Text(user.value!.name);
```

Why it's wrong:

- `value` may be unavailable.

✔ Correct

Use `when()` or verify `hasValue`.

---

### 4. Storing AsyncData Separately

❌ Wrong

```dart
AsyncData<User> currentUser;
```

Why it's wrong:

- Applications should typically work with `AsyncValue<User>`.
- Riverpod manages state transitions automatically.

✔ Correct

Use:

```dart
AsyncValue<User>
```

and handle all possible states.

---

## AsyncData vs AsyncLoading vs AsyncError

| State | Meaning | Contains Value |
|--------|---------|----------------|
| `AsyncLoading` | Waiting for completion | ❌ |
| `AsyncData` | Operation succeeded | ✅ |
| `AsyncError` | Operation failed | ❌ |

---

## Related APIs

- `AsyncValue`
- `AsyncLoading`
- `AsyncError`
- `when()`
- `maybeWhen()`
- `map()`
- `FutureProvider`
- `StreamProvider`
- `AsyncNotifier`

---

## Summary

`AsyncData<T>` represents the successful completion of an asynchronous operation. It safely wraps the resulting value and provides a consistent way to build UI after data has been loaded. By using `AsyncData` through APIs like `when()`, Riverpod applications can display asynchronous results in a predictable, type-safe manner while keeping loading and error handling separate.