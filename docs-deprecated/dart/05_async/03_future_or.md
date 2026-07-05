# FutureOr

Understand how to work with FutureOr for flexible asynchronous APIs in Dart.

---

# What is it?

`FutureOr<T>` is a special type in Dart that can be either a `Future<T>` or a direct value of type `T`. It allows functions to return either a synchronous value or an asynchronous Future, giving callers flexibility in how they handle the result.

---

# Why does it exist?

`FutureOr` exists to:

- Allow functions to return sync or async values
- Provide flexibility in API design
- Enable performance optimizations
- Support both immediate and delayed results
- Simplify function signatures
- Handle cases where computation might be synchronous

---

# Basic FutureOr

## Understanding FutureOr

> **`FutureOr<T>`** is a union type that can hold either `T` or `Future<T>`. This is useful when a function doesn't know if it will return a value immediately or asynchronously.

```dart
import 'dart:async';

// Function that returns either a value or a Future
FutureOr<String> getData(bool fromCache) {
  if (fromCache) {
    // Returns a value directly (synchronous)
    return 'Cached data';
  } else {
    // Returns a Future (asynchronous)
    return Future.delayed(
      Duration(seconds: 2),
      () => 'Fresh data',
    );
  }
}

void main() {
  // Handle sync and async uniformly
  var result = getData(true);
  
  // Check if it's a Future
  if (result is Future<String>) {
    result.then((value) => print('Async: $value'));
  } else {
    print('Sync: $result');
  }
}

// Output:
// Sync: Cached data
```

> **What's happening here?**
> - `getData()` returns `FutureOr<String>`
> - `fromCache: true` returns a `String` directly
> - `fromCache: false` returns a `Future<String>`
> - Caller checks the type and handles accordingly
> - This provides flexibility in implementation

---

## Using FutureOr in Functions

> **Functions that use `FutureOr`** can return either type, making them more flexible for callers.

```dart
// Calculator that might be sync or async
FutureOr<int> calculate(bool useAsync) {
  if (useAsync) {
    return Future.delayed(
      Duration(seconds: 1),
      () => 42,
    );
  } else {
    return 42;
  }
}

// Function accepting FutureOr
void processData(FutureOr<String> data) {
  if (data is Future<String>) {
    data.then((value) => print('Async data: $value'));
  } else {
    print('Sync data: $data');
  }
}

void main() async {
  // Handle FutureOr results
  var result1 = calculate(false);
  print('Sync result: $result1'); // 42
  
  var result2 = calculate(true);
  if (result2 is Future<int>) {
    var value = await result2;
    print('Async result: $value'); // 42
  }
  
  // Pass FutureOr to functions
  processData('Hello'); // Sync data: Hello
  processData(Future.value('World')); // Async data: World
}
```

> **Key insights:**
> - Functions can return sync or async values
> - Callers can check the type and handle accordingly
> - `FutureOr` can be used in parameters and return types
> - This provides maximum flexibility

---

# Working with FutureOr

## Handling FutureOr Values

> **When working with `FutureOr`**, you typically need to handle both cases: immediate values and Future values.

```dart
// Method 1: Type checking
void handleValue(FutureOr<String> value) {
  if (value is Future<String>) {
    value.then((v) => print('Got: $v'));
  } else {
    print('Got: $value');
  }
}

// Method 2: Convert to Future (always async)
void handleValue2(FutureOr<String> value) {
  Future<String> future = Future.value(value);
  future.then((v) => print('Got: $v'));
}

// Method 3: Using await (works with both)
void handleValue3(FutureOr<String> value) async {
  // await works with both Future and value
  var result = await value;
  print('Got: $result');
}

void main() {
  // All methods work with sync and async
  handleValue('Hello'); // Got: Hello
  handleValue(Future.value('World')); // Got: World
  
  handleValue2('Hello'); // Got: Hello
  handleValue2(Future.value('World')); // Got: World
  
  handleValue3('Hello'); // Got: Hello
  handleValue3(Future.value('World')); // Got: World
}
```

> **What's happening here?**
> - Method 1: Type checking with `is`
> - Method 2: Always convert to `Future` using `Future.value()`
> - Method 3: Use `await` (works with both types)
> - All methods handle both sync and async values
> - Choose the approach that fits your needs

---

## Converting FutureOr to Future

> **Converting `FutureOr` to `Future`** makes it easier to handle uniformly in async contexts.

```dart
import 'dart:async';

// Helper to convert FutureOr to Future
Future<T> toFuture<T>(FutureOr<T> value) {
  return Future.value(value);
}

// Example function
FutureOr<String> getData(bool fromCache) {
  if (fromCache) {
    return 'Cached data';
  }
  return Future.delayed(Duration(seconds: 1), () => 'Fresh data');
}

void main() async {
  // Convert and await
  var data1 = await toFuture(getData(true));
  var data2 = await toFuture(getData(false));
  
  print(data1); // Cached data
  print(data2); // Fresh data
  
  // Or use await directly
  var data3 = await getData(true); // Works because await handles both
  var data4 = await getData(false);
  
  print(data3); // Cached data
  print(data4); // Fresh data
}
```

> **Key insights:**
> - `Future.value()` handles both `Future` and values
> - `await` works directly on `FutureOr`
> - Converting to `Future` makes code more uniform
> - Choose the approach that's most readable

---

# Real-World Example

## Cache with FutureOr

> **A practical use of `FutureOr`** is in caching systems where data might be immediately available (cache hit) or need to be fetched asynchronously (cache miss).

```dart
class Cache<T> {
  final Map<String, T> _cache = {};
  final Duration _ttl;
  final Map<String, DateTime> _expiry = {};
  
  Cache({Duration ttl = const Duration(minutes: 5)}) : _ttl = ttl;
  
  // Get from cache - returns FutureOr
  FutureOr<T?> get(String key) {
    // Check if key exists and is not expired
    if (_cache.containsKey(key)) {
      var expiry = _expiry[key]!;
      if (DateTime.now().isBefore(expiry)) {
        // Cache hit - return value synchronously
        return _cache[key];
      } else {
        // Expired - remove from cache
        _cache.remove(key);
        _expiry.remove(key);
      }
    }
    
    // Cache miss - return Future for async fetch
    return _fetchAndCache(key);
  }
  
  // Simulate async fetch
  Future<T?> _fetchAndCache(String key) async {
    // Simulate network request
    await Future.delayed(Duration(seconds: 1));
    
    // Simulate data not found
    if (key == 'error') {
      return null;
    }
    
    // Simulate fetched data
    final value = 'Data for $key' as T;
    _cache[key] = value;
    _expiry[key] = DateTime.now().add(_ttl);
    return value;
  }
  
  void clear() {
    _cache.clear();
    _expiry.clear();
  }
}

void main() async {
  var cache = Cache<String>();
  
  // First request - cache miss (async)
  print('Fetching user1...');
  var result1 = await cache.get('user1');
  print('Result: $result1');
  
  // Second request - cache hit (sync)
  print('\nFetching user1 again...');
  var result2 = await cache.get('user1');
  print('Result: $result2');
  
  // Third request - cache miss (async)
  print('\nFetching user2...');
  var result3 = await cache.get('user2');
  print('Result: $result3');
}

// Output:
// Fetching user1...
// (1 second delay)
// Result: Data for user1
//
// Fetching user1 again...
// Result: Data for user1
//
// Fetching user2...
// (1 second delay)
// Result: Data for user2
```

> **What's happening here?**
> - Cache hit returns value synchronously (performance optimized)
> - Cache miss returns Future for async fetch
> - Caller uses `await` uniformly (works with both)
> - This pattern is used in real applications
> - `FutureOr` enables performance optimization

---

# Best Practices

## Use FutureOr for Optional Async

```dart
// Good: Use FutureOr when function might be sync or async
FutureOr<int> getCachedValue(String key) {
  if (_cache.containsKey(key)) {
    return _cache[key]; // Sync
  } else {
    return _fetchValue(key); // Async
  }
}

// Bad: Always async even when not needed
Future<int> getCachedValueBad(String key) async {
  if (_cache.containsKey(key)) {
    return _cache[key]; // Still async, unnecessary overhead
  } else {
    return await _fetchValue(key);
  }
}
```

---

## Document FutureOr Behavior

```dart
// Good: Document that it can be sync or async
/// Returns the user data.
///
/// The return type is [FutureOr<User>], meaning it can be:
/// - A [User] directly (if the data is already available)
/// - A [Future<User>] if the data needs to be fetched
///
/// Example:
/// ```dart
/// var user = await getUser('123'); // Works for both cases
/// ```
FutureOr<User> getUser(String id) {
  // Implementation
}
```

---

# Common Mistakes

## Assuming It's Always a Future

Wrong:
```dart
void process(FutureOr<String> data) {
  data.then((value) => print(value)); // Error: Not always a Future
}
```

Correct:
```dart
void process(FutureOr<String> data) {
  if (data is Future) {
    data.then((value) => print(value));
  } else {
    print(data);
  }
}
```

---

## Returning Future when Value is Available

Wrong:
```dart
FutureOr<int> getValue() {
  if (_cached) {
    return Future.value(42); // Unnecessary Future wrapping
  }
  return fetchValue();
}
```

Correct:
```dart
FutureOr<int> getValue() {
  if (_cached) {
    return 42; // Return value directly
  }
  return fetchValue();
}
```

---

# Summary

`FutureOr` provides flexibility by allowing functions to return either a value or a Future. It's useful for caching, optional async operations, and optimizing performance while maintaining a clean API.

---

# Next Steps

Now that you understand FutureOr, continue to:

- [Streams](streams.md)
- [StreamController](stream-controller.md)
- [async*](async-star.md)
- [sync*](sync-star.md)

---

# Did You Know?

- `FutureOr` is a union type (sum type)
- `await` works directly on `FutureOr`
- `Future.value()` converts any type to Future
- `FutureOr` is useful for caching patterns
- It can improve performance by avoiding async overhead
- `FutureOr` was introduced in Dart 2.0
- Type checking (`is`) works with `FutureOr`

---
