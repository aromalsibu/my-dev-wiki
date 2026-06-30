# Future

Understand how to work with Future objects for asynchronous operations in Dart.

---

# What is it?

A `Future` represents a potential value or error that will be available at some time in the future. It's a placeholder for a value that hasn't been computed yet, allowing you to write non-blocking code that waits for operations like network requests, file I/O, or timers.

---

# Why does it exist?

Future exists to:

- Represent asynchronous operations
- Avoid blocking the main thread
- Handle long-running operations gracefully
- Manage success and error states
- Chain asynchronous operations
- Enable responsive applications

---

# Creating Futures

## Basic Future Creation

> **Creating a Future** can be done in several ways. The simplest is using `Future.value()` or `Future.error()`, but you can also use `Future.delayed()` for delayed execution.

```dart
// Future that completes immediately with a value
Future<String> immediateFuture = Future.value('Hello');

// Future that completes with an error
Future<int> errorFuture = Future.error('Something went wrong');

// Future with delayed completion
Future<String> delayedFuture = Future.delayed(
  Duration(seconds: 2),
  () => 'Data loaded',
);

// Using the Future
void main() {
  delayedFuture.then((value) {
    print('Received: $value');
  });
  
  print('Loading...');
}

// Output:
// Loading...
// (2 seconds later)
// Received: Data loaded
```

> **What's happening here?**
> - `Future.value()` creates an immediately completed Future
> - `Future.error()` creates a Future that completes with an error
> - `Future.delayed()` schedules a Future that completes after a delay
> - `then()` registers a callback for when the Future completes
> - The program continues running while waiting

---

## Factory Constructors

> **Future factory constructors** provide different ways to create Futures based on your needs.

```dart
// Future.value() - Immediately completes with a value
Future<String> future1 = Future.value('Hello');

// Future.error() - Immediately completes with an error
Future<int> future2 = Future.error('Network error');

// Future.delayed() - Completes after a delay
Future<String> future3 = Future.delayed(
  Duration(seconds: 1),
  () => 'Data loaded',
);

// Future.sync() - Executes synchronously then returns Future
Future<String> future4 = Future.sync(() {
  print('Running synchronously');
  return 'Result';
});

// Future.microtask() - Schedules a microtask
Future<int> future5 = Future.microtask(() {
  print('Running in microtask');
  return 42;
});

void main() {
  future4.then((value) => print(value));
  future5.then((value) => print(value));
}

// Output:
// Running synchronously
// Result
// Running in microtask
// 42
```

> **Key insights:**
> - `Future.sync()` executes immediately and returns a Future
> - `Future.microtask()` schedules work for the microtask queue
> - Microtasks run before normal tasks
> - Different constructors for different use cases
> - All return a Future that can be listened to

---

# Working with Futures

## Using `then()` and `catchError()`

> **Handling Future results** is done using `then()` for success and `catchError()` for errors. This is the callback-based approach.

```dart
Future<String> fetchData() {
  // Simulate network request
  return Future.delayed(
    Duration(seconds: 1),
    () => 'Data loaded successfully',
  );
}

Future<int> riskyOperation() {
  return Future.delayed(
    Duration(seconds: 1),
    () => throw Exception('Operation failed!'),
  );
}

void main() {
  // Successful operation
  fetchData()
    .then((value) {
      print('Success: $value');
    })
    .catchError((error) {
      print('Error: $error');
    });
  
  // Risky operation
  riskyOperation()
    .then((value) {
      print('Success: $value');
    })
    .catchError((error) {
      print('Caught: $error');
    });
}

// Output:
// Success: Data loaded successfully
// Caught: Exception: Operation failed!
```

> **What's happening here?**
> - `then()` is called when the Future succeeds
> - `catchError()` handles any errors
> - The chain continues regardless of success or failure
> - This is the callback-based way to handle Futures

---

## Chaining Futures

> **Chaining** allows you to perform sequential asynchronous operations. Each `then()` returns a new Future, enabling you to chain multiple operations.

```dart
Future<String> fetchUser() {
  return Future.delayed(
    Duration(seconds: 1),
    () => 'User: Alice',
  );
}

Future<String> fetchProfile(String user) {
  return Future.delayed(
    Duration(seconds: 1),
    () => '$user - Profile: Active',
  );
}

Future<String> fetchPosts(String profile) {
  return Future.delayed(
    Duration(seconds: 1),
    () => '$profile - Posts: 5',
  );
}

void main() {
  // Chaining Futures
  fetchUser()
    .then((user) {
      print('Fetched: $user');
      return fetchProfile(user);
    })
    .then((profile) {
      print('Fetched: $profile');
      return fetchPosts(profile);
    })
    .then((posts) {
      print('Fetched: $posts');
    })
    .catchError((error) {
      print('Error: $error');
    });
}

// Output:
// Fetched: User: Alice
// Fetched: User: Alice - Profile: Active
// Fetched: User: Alice - Profile: Active - Posts: 5
```

> **What's happening here?**
> - Each `then()` returns a new Future
> - Operations are sequential (one after another)
> - The next `then()` receives the previous result
> - A single `catchError()` handles all errors
> - The chain completes when all operations succeed

---

# Error Handling

## Handling Future Errors

> **Error handling in Futures** is crucial for robust applications. You can use `catchError()`, `onError`, or `try-catch` with `await`.

```dart
// Different error handling approaches
Future<int> divide(int a, int b) {
  return Future.sync(() {
    if (b == 0) throw Exception('Cannot divide by zero');
    return a ~/ b;
  });
}

void main() {
  // Approach 1: catchError
  divide(10, 0)
    .then((result) {
      print('Result: $result');
    })
    .catchError((error) {
      print('Error 1: $error');
    });
  
  // Approach 2: onError with type
  divide(10, 0)
    .then((result) {
      print('Result: $result');
    }, onError: (error) {
      print('Error 2: $error');
    });
  
  // Approach 3: async/await with try-catch
  Future<void> handleDivision() async {
    try {
      var result = await divide(10, 0);
      print('Result: $result');
    } catch (e) {
      print('Error 3: $e');
    }
  }
  
  handleDivision();
}

// Output:
// Error 1: Exception: Cannot divide by zero
// Error 2: Exception: Cannot divide by zero
// Error 3: Exception: Cannot divide by zero
```

> **Key insights:**
> - Multiple ways to handle errors
> - `catchError` is the standard approach
> - `onError` can be passed to `then()`
> - `try-catch` works with `await`
> - Choose the approach that fits your style

---

## Handling Specific Errors

```dart
class NetworkException implements Exception {
  final String message;
  NetworkException(this.message);
}

class ValidationException implements Exception {
  final String message;
  ValidationException(this.message);
}

Future<String> fetchData(bool shouldFail) {
  return Future.delayed(
    Duration(seconds: 1),
    () {
      if (shouldFail) {
        throw NetworkException('Connection refused');
      }
      return 'Data loaded';
    },
  );
}

void main() {
  // Handle specific error types
  fetchData(true)
    .then((data) {
      print('Success: $data');
    })
    .catchError((error) {
      if (error is NetworkException) {
        print('Network error: ${error.message}');
        // Try to retry
      } else if (error is ValidationException) {
        print('Validation error: ${error.message}');
      } else {
        print('Unknown error: $error');
      }
    });
}

// Output:
// Network error: Connection refused
```

> **What's happening here?**
> - Custom exceptions for different error types
> - `is` operator checks the error type
> - Different handling for different error types
> - You can attempt retry logic based on error type

---

# Future Utilities

## Common Future Methods

> **Dart provides** several useful Future utilities for common operations like running multiple Futures in parallel.

```dart
// Future.wait - Wait for multiple Futures
Future<int> slowAdd(int a, int b) {
  return Future.delayed(Duration(seconds: 1), () => a + b);
}

Future<int> slowMultiply(int a, int b) {
  return Future.delayed(Duration(seconds: 1), () => a * b);
}

void main() async {
  // Wait for all Futures to complete
  List<int> results = await Future.wait([
    slowAdd(5, 3),
    slowMultiply(5, 3),
  ]);
  
  print(results); // [8, 15]
  
  // Future.any - First to complete wins
  var firstResult = await Future.any([
    slowAdd(1, 2),
    slowAdd(3, 4),
    Future.delayed(Duration(milliseconds: 500), () => 100),
  ]);
  
  print('First result: $firstResult'); // 100 (fastest)
  
  // Future.forEach - Process items sequentially
  await Future.forEach([1, 2, 3], (item) async {
    await Future.delayed(Duration(seconds: 1));
    print('Processed: $item');
  });
}

// Output:
// [8, 15]
// First result: 100
// (1 second delay)
// Processed: 1
// (1 second delay)
// Processed: 2
// (1 second delay)
// Processed: 3
```

> **Key insights:**
> - `Future.wait()` runs Futures in parallel
> - `Future.any()` returns the first completed Future
> - `Future.forEach()` processes items sequentially
> - These utilities make common patterns easier

---

# Best Practices

## Async/Await Over Raw Futures

```dart
// Bad: Raw Futures with nested callbacks
void badExample() {
  fetchUser()
    .then((user) {
      fetchProfile(user)
        .then((profile) {
          fetchPosts(profile)
            .then((posts) {
              print('Posts: $posts');
            });
        });
    })
    .catchError((error) {
      print('Error: $error');
    });
}

// Good: Clean async/await
void goodExample() async {
  try {
    var user = await fetchUser();
    var profile = await fetchProfile(user);
    var posts = await fetchPosts(profile);
    print('Posts: $posts');
  } catch (e) {
    print('Error: $e');
  }
}
```

> **Why async/await is better:**
> - No callback nesting
> - Easier to read and maintain
> - Natural error handling with try-catch
> - Looks like synchronous code
> - Easier to debug

---

## Always Handle Errors

```dart
// Bad: Unhandled errors
void badExample() {
  riskyOperation()
    .then((value) => print(value));
  // Error will crash the app!
}

// Good: Always handle errors
void goodExample() {
  riskyOperation()
    .then((value) => print(value))
    .catchError((error) {
      print('Handled: $error');
    });
}

// Good: With async/await
void goodExample2() async {
  try {
    var result = await riskyOperation();
    print(result);
  } catch (e) {
    print('Handled: $e');
  }
}
```

---

# Common Mistakes

## Forgetting to Await

Wrong:
```dart
void main() {
  var data = fetchData(); // This is a Future, not the data
  print(data); // Prints: Instance of 'Future<String>'
}
```

Correct:
```dart
void main() async {
  var data = await fetchData(); // Waits for the result
  print(data); // Prints: Data loaded
}
```

---

## Not Returning in then()

Wrong:
```dart
void main() {
  fetchUser()
    .then((user) {
      fetchProfile(user); // Missing return!
    })
    .then((profile) {
      print(profile); // Prints: null
    });
}
```

Correct:
```dart
void main() {
  fetchUser()
    .then((user) {
      return fetchProfile(user); // Returns the Future
    })
    .then((profile) {
      print(profile); // Prints the profile
    });
}
```

---

## Swallowing Errors

Wrong:
```dart
void main() {
  riskyOperation()
    .then((value) => print(value))
    .catchError((error) {
      // Silent error - bad practice!
    });
}
```

Correct:
```dart
void main() {
  riskyOperation()
    .then((value) => print(value))
    .catchError((error) {
      print('Error: $error'); // Handle appropriately
      // Or log it
    });
}
```

---

# Summary

Futures represent asynchronous operations that complete at some point in the future. They enable non-blocking code, handle success and error states, and can be chained for sequential operations.

---

# Next Steps

Now that you understand Futures, continue to:

- [async & await](async-await.md)
- [FutureOr](future-or.md)
- [Streams](streams.md)
- [Error Handling](error-handling.md)

---

# Did You Know?

- Futures are used extensively in Dart and Flutter
- Each Future runs on the event loop
- Microtasks run before normal tasks
- Futures can be completed multiple ways
- `Future.wait()` runs operations in parallel
- Unhandled errors in Futures can crash your app
- The event loop processes Futures one at a time

---
