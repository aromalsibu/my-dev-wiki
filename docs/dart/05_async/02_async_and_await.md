# async & await

Understand how to write asynchronous code that looks synchronous using async and await in Dart.

---

# What is it?

`async` and `await` are keywords that make working with Futures easier and more readable. `async` marks a function as asynchronous, allowing it to use `await`. `await` pauses the execution of the function until a Future completes, without blocking the main thread.

---

# Why does it exist?

async & await exist to:

- Write asynchronous code that looks synchronous
- Eliminate callback nesting ("callback hell")
- Make code more readable and maintainable
- Simplify error handling with try-catch
- Reduce boilerplate code
- Make async code easier to reason about

---

# Basic async/await

## Using async and await

> **The `async` keyword** marks a function as asynchronous. Inside an `async` function, you can use `await` to wait for Futures to complete. The function automatically returns a `Future` of the return type.

```dart
// Function that returns a Future<String>
Future<String> fetchData() async {
  // Simulate network delay
  await Future.delayed(Duration(seconds: 2));
  return 'Data loaded';
}

// Using the async function
void main() async {
  print('Loading...');
  
  // Wait for the Future to complete
  String result = await fetchData();
  print('Result: $result');
  
  print('Done!');
}

// Output:
// Loading...
// (2 seconds later)
// Result: Data loaded
// Done!
```

> **What's happening here?**
> - `async` marks `fetchData()` as asynchronous
> - `await` pauses execution until `Future.delayed()` completes
> - The function returns a `Future<String>` automatically
> - In `main()`, `await fetchData()` waits for the result
> - The program looks sequential but runs asynchronously

---

## async Function Return Types

> **When you mark a function with `async`**, Dart automatically wraps the return value in a `Future`. The return type must be a `Future` of the actual return type.

```dart
// Returns Future<String>
Future<String> getGreeting() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Hello, World!';
}

// Returns Future<int>
Future<int> getNumber() async {
  await Future.delayed(Duration(seconds: 1));
  return 42;
}

// Returns Future<void> (no return value)
Future<void> doSomething() async {
  await Future.delayed(Duration(seconds: 1));
  print('Something done!');
}

// Returns Future<List<String>>
Future<List<String>> getNames() async {
  await Future.delayed(Duration(seconds: 1));
  return ['Alice', 'Bob', 'Charlie'];
}

// Usage
void main() async {
  var greeting = await getGreeting();
  var number = await getNumber();
  await doSomething();
  var names = await getNames();
  
  print(greeting); // Hello, World!
  print(number); // 42
  print(names); // [Alice, Bob, Charlie]
}
```

> **Key insights:**
> - `async` automatically wraps return in `Future`
> - Return type must match what's actually returned
> - `Future<void>` is for functions with no return value
> - The return type can be any type wrapped in Future

---

# Error Handling with async/await

## Try-Catch with async/await

> **Error handling** with `async/await` uses the same `try-catch` syntax as synchronous code. This makes error handling much more natural.

```dart
// Function that throws an error
Future<String> riskyOperation() async {
  await Future.delayed(Duration(seconds: 1));
  throw Exception('Operation failed!');
}

// Function that might succeed or fail
Future<int> fetchNumber(bool shouldFail) async {
  await Future.delayed(Duration(seconds: 1));
  if (shouldFail) {
    throw FormatException('Invalid format');
  }
  return 42;
}

void main() async {
  // Standard try-catch
  try {
    var result = await riskyOperation();
    print('Success: $result');
  } catch (e) {
    print('Error: $e');
  }
  
  // Try-catch with specific error types
  try {
    var number = await fetchNumber(true);
    print('Number: $number');
  } on FormatException catch (e) {
    print('Format error: $e');
  } catch (e) {
    print('Other error: $e');
  } finally {
    print('Always executes');
  }
}

// Output:
// Error: Exception: Operation failed!
// Format error: Invalid format
// Always executes
```

> **What's happening here?**
> - `try-catch` works naturally with `await`
> - `on` can catch specific exception types
> - `finally` always executes, success or failure
> - Error handling is clean and readable

---

## Multiple Async Operations

> **You can perform** multiple asynchronous operations sequentially or in parallel using `async/await`.

```dart
// Sequential operations
Future<String> fetchUser() async {
  await Future.delayed(Duration(seconds: 1));
  return 'User: Alice';
}

Future<String> fetchProfile(String user) async {
  await Future.delayed(Duration(seconds: 1));
  return '$user - Profile: Active';
}

Future<String> fetchPosts(String profile) async {
  await Future.delayed(Duration(seconds: 1));
  return '$profile - Posts: 5';
}

// Sequential execution
Future<void> sequentialExample() async {
  print('Sequential:');
  
  var user = await fetchUser();
  print(user);
  
  var profile = await fetchProfile(user);
  print(profile);
  
  var posts = await fetchPosts(profile);
  print(posts);
}

// Parallel execution with Future.wait
Future<void> parallelExample() async {
  print('Parallel:');
  
  var results = await Future.wait([
    fetchUser(),
    fetchProfile('User: Alice'),
    fetchPosts('User: Alice - Profile: Active'),
  ]);
  
  print(results);
}

// Parallel with individual await
Future<void> parallelAwaitExample() async {
  print('Parallel with individual awaits:');
  
  var future1 = fetchUser();
  var future2 = fetchProfile('User: Alice');
  var future3 = fetchPosts('User: Alice - Profile: Active');
  
  var user = await future1;
  var profile = await future2;
  var posts = await future3;
  
  print('$user, $profile, $posts');
}
```

> **What's happening here?**
> - Sequential operations run one after another
> - Parallel operations run simultaneously
> - `Future.wait()` runs multiple Futures in parallel
> - Individual `await` on pre-started Futures
> - Choose based on your needs

---

# Advanced Patterns

## Conditional Async Operations

> **You can use** `async/await` with conditional logic, loops, and other control flow structures.

```dart
// Conditional async operations
Future<String> fetchData(bool fromCache) async {
  if (fromCache) {
    // Simulate cache hit
    await Future.delayed(Duration(milliseconds: 100));
    return 'Cached data';
  } else {
    // Simulate network request
    await Future.delayed(Duration(seconds: 2));
    return 'Fresh data';
  }
}

// Looping with async
Future<List<String>> fetchAllUsers(List<int> ids) async {
  List<String> users = [];
  
  for (var id in ids) {
    await Future.delayed(Duration(milliseconds: 500));
    users.add('User $id');
  }
  
  return users;
}

// Retry logic
Future<String> fetchWithRetry(int maxRetries) async {
  for (var attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      await Future.delayed(Duration(seconds: 1));
      if (attempt < 3) {
        throw Exception('Temporary failure');
      }
      return 'Success on attempt $attempt';
    } catch (e) {
      print('Attempt $attempt failed: $e');
      if (attempt == maxRetries) {
        rethrow;
      }
    }
  }
  return 'Unreachable';
}

void main() async {
  // Conditional
  var data = await fetchData(false);
  print(data); // Fresh data
  
  // Loop
  var users = await fetchAllUsers([1, 2, 3]);
  print(users); // [User 1, User 2, User 3]
  
  // Retry
  try {
    var result = await fetchWithRetry(3);
    print(result); // Success on attempt 3
  } catch (e) {
    print('All retries failed');
  }
}
```

> **Key insights:**
> - `async/await` works with if/else, loops, etc.
> - Retry logic is straightforward with loops
> - Conditional async operations are natural
> - Complex flows are easier with async/await

---

## Streams with async/await

> **While `async/await`** is primarily for Futures, you can use it with Streams using `await for`.

```dart
// Stream that emits numbers
Stream<int> countStream(int max) async* {
  for (var i = 1; i <= max; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

// Processing Stream with await for
Future<void> processStream() async {
  print('Processing stream:');
  
  await for (var value in countStream(5)) {
    print('Received: $value');
  }
  
  print('Stream complete!');
}

// Combining Future and Stream
Future<int> sumStream(Stream<int> stream) async {
  int sum = 0;
  await for (var value in stream) {
    sum += value;
  }
  return sum;
}

void main() async {
  await processStream();
  
  var total = await sumStream(countStream(5));
  print('Sum: $total'); // 15
}

// Output:
// Processing stream:
// (1 second later)
// Received: 1
// (1 second later)
// Received: 2
// (1 second later)
// Received: 3
// (1 second later)
// Received: 4
// (1 second later)
// Received: 5
// Stream complete!
// Sum: 15
```

> **What's happening here?**
> - `await for` loops over Stream events
> - Each event pauses the function until received
> - Streams can be processed sequentially
> - Combine with Futures for complex logic

---

# Best Practices

## Use async/await for Readability

```dart
// Bad: Callback hell with raw Futures
void badExample() {
  fetchUser()
    .then((user) {
      fetchProfile(user)
        .then((profile) {
          fetchPosts(profile)
            .then((posts) {
              print('Posts: $posts');
            })
            .catchError((error) {
              print('Error: $error');
            });
        })
        .catchError((error) {
          print('Error: $error');
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

---

## Don't Use await in Parallel

```dart
// Bad: Sequential when it could be parallel
void badExample() async {
  var user = await fetchUser();
  var profile = await fetchProfile(); // Could be parallel
  var posts = await fetchPosts(); // Could be parallel
}

// Good: Parallel when possible
void goodExample() async {
  // Start all Futures at once
  var userFuture = fetchUser();
  var profileFuture = fetchProfile();
  var postsFuture = fetchPosts();
  
  // Then await all results
  var user = await userFuture;
  var profile = await profileFuture;
  var posts = await postsFuture;
}

// Even better: Future.wait
void bestExample() async {
  var results = await Future.wait([
    fetchUser(),
    fetchProfile(),
    fetchPosts(),
  ]);
  
  var user = results[0];
  var profile = results[1];
  var posts = results[2];
}
```

---

# Common Mistakes

## Forgetting to Mark Function async

Wrong:
```dart
void main() {
  var data = await fetchData(); // Error: await only in async
  print(data);
}
```

Correct:
```dart
void main() async {
  var data = await fetchData();
  print(data);
}
```

---

## Returning Future in async Function

Wrong:
```dart
Future<String> fetchData() async {
  return Future.value('Data'); // Wrapping extra Future
}
```

Correct:
```dart
Future<String> fetchData() async {
  return 'Data'; // Automatically wrapped in Future
}
```

---

## Not Using try-catch in Async

Wrong:
```dart
void main() async {
  var result = await riskyOperation(); // Unhandled error
  print(result);
}
```

Correct:
```dart
void main() async {
  try {
    var result = await riskyOperation();
    print(result);
  } catch (e) {
    print('Error: $e');
  }
}
```

---

# Summary

`async` and `await` make asynchronous code look and behave like synchronous code. They eliminate callback nesting, simplify error handling, and make code more readable and maintainable.

---

# Next Steps

Now that you understand async & await, continue to:

- [FutureOr](future-or.md)
- [Streams](streams.md)
- [StreamController](stream-controller.md)
- [Error Handling](error-handling.md)

---

# Did You Know?

- `async` functions always return a Future
- `await` can only be used inside `async` functions
- The event loop processes microtasks before normal tasks
- `async`/`await` is syntactic sugar over Future
- You can use `try-catch` with `await` naturally
- `Future.wait()` runs operations in parallel
- The Dart VM optimizes async/await for performance

---

