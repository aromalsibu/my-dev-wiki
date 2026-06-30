# async* (Async Generators)

Understand how to create asynchronous streams using async* generators in Dart.

---

# What is it?

`async*` is a keyword used to define asynchronous generator functions that produce a `Stream` of values over time. Unlike a regular function that returns a single value, an async* generator can emit multiple values using the `yield` keyword, with each value being emitted asynchronously.

---

# Why does it exist?

`async*` exists to:

- Create streams that emit multiple values over time
- Generate values asynchronously (with delays, network calls, etc.)
- Handle sequences of data efficiently
- Enable lazy evaluation of stream elements
- Simplify stream creation with natural syntax
- Combine async operations with stream generation

---

# Basic async* Syntax

## Creating an Async Generator

> **An `async*` function** returns a `Stream` and uses `yield` to emit values. It can use `await` to pause and wait for async operations.

```dart
// Basic async generator
Stream<int> countUp(int max) async* {
  for (var i = 1; i <= max; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i; // Emit the current value
  }
}

void main() async {
  print('Counting up:');
  
  // Listen to the stream
  await for (var value in countUp(5)) {
    print('Value: $value');
  }
  
  print('Done!');
}

// Output:
// Counting up:
// (1 second later)
// Value: 1
// (1 second later)
// Value: 2
// (1 second later)
// Value: 3
// (1 second later)
// Value: 4
// (1 second later)
// Value: 5
// Done!
```

> **What's happening here?**
> - `async*` marks the function as an async generator
> - It returns a `Stream<int>`
> - `yield` emits a value into the stream
> - `await` pauses the generator for async operations
> - The stream emits values one at a time

---

## Async Generator with Conditions

> **Async generators** can include conditions and control flow just like regular functions.

```dart
// Generator with conditions
Stream<int> evenNumbers(int max) async* {
  for (var i = 1; i <= max; i++) {
    if (i % 2 == 0) {
      await Future.delayed(Duration(milliseconds: 500));
      yield i;
    }
  }
}

// Generator with early return
Stream<int> limitedNumbers(int max) async* {
  for (var i = 1; i <= max; i++) {
    if (i > 10) {
      return; // Stops the generator
    }
    await Future.delayed(Duration(milliseconds: 300));
    yield i;
  }
}

// Generator with break
Stream<int> numbersUntil(int max, int stop) async* {
  for (var i = 1; i <= max; i++) {
    if (i == stop) {
      break; // Breaks the loop, generator ends
    }
    await Future.delayed(Duration(milliseconds: 300));
    yield i;
  }
}

void main() async {
  print('Even numbers:');
  await for (var value in evenNumbers(10)) {
    print('Even: $value');
  }
  
  print('\nLimited numbers:');
  await for (var value in limitedNumbers(15)) {
    print('Value: $value');
  }
  
  print('\nNumbers until stop:');
  await for (var value in numbersUntil(10, 5)) {
    print('Value: $value');
  }
}

// Output:
// Even numbers: 2, 4, 6, 8, 10
// Limited numbers: 1-10 (stops at 10)
// Numbers until stop: 1-4 (stops at 5)
```

> **Key insights:**
> - Conditions work inside async generators
> - `return` ends the generator early
> - `break` can exit loops
> - All control flow works naturally

---

# Advanced async*

## Nested async Generators

> **You can yield** values from other streams or async generators using `yield*`.

```dart
// Helper generator
Stream<int> generateNumbers(int start, int count) async* {
  for (var i = 0; i < count; i++) {
    await Future.delayed(Duration(milliseconds: 200));
    yield start + i;
  }
}

// Main generator using yield*
Stream<int> combinedNumbers() async* {
  // Emit values from first generator
  yield* generateNumbers(1, 3); // 1, 2, 3
  
  // Add a delay between groups
  await Future.delayed(Duration(seconds: 1));
  
  // Emit values from second generator
  yield* generateNumbers(10, 3); // 10, 11, 12
  
  // Emit a single value
  await Future.delayed(Duration(milliseconds: 500));
  yield 100;
}

void main() async {
  print('Combined numbers:');
  await for (var value in combinedNumbers()) {
    print('Value: $value');
  }
}

// Output:
// Combined numbers:
// Value: 1
// Value: 2
// Value: 3
// (1 second delay)
// Value: 10
// Value: 11
// Value: 12
// Value: 100
```

> **What's happening here?**
> - `yield*` delegates to another generator
> - It emits all values from the nested generator
> - You can combine multiple generators
> - You can add delays between groups
> - This is useful for composing streams

---

## Error Handling in async*

> **Async generators** can handle errors using try-catch blocks, just like regular async functions.

```dart
Stream<int> safeGenerator(int max) async* {
  for (var i = 1; i <= max; i++) {
    try {
      if (i == 3) {
        throw Exception('Error at 3!');
      }
      await Future.delayed(Duration(milliseconds: 300));
      yield i;
    } catch (e) {
      // Handle the error
      print('Caught: $e');
      // Continue or break
      continue; // Skip this value
    }
  }
}

Stream<int> generatorWithError(int max) async* {
  for (var i = 1; i <= max; i++) {
    if (i == 3) {
      throw Exception('Error at 3!'); // Throws to the listener
    }
    await Future.delayed(Duration(milliseconds: 300));
    yield i;
  }
}

void main() async {
  print('Safe generator:');
  await for (var value in safeGenerator(5)) {
    print('Value: $value');
  }
  
  print('\nGenerator with error:');
  try {
    await for (var value in generatorWithError(5)) {
      print('Value: $value');
    }
  } catch (e) {
    print('Caught in main: $e');
  }
}

// Output:
// Safe generator:
// Value: 1
// Value: 2
// Caught: Exception: Error at 3!
// Value: 4
// Value: 5
//
// Generator with error:
// Value: 1
// Value: 2
// Caught in main: Exception: Error at 3!
```

> **Key insights:**
> - Errors can be caught inside the generator
> - Errors can be propagated to listeners
> - You can choose to continue or stop on error
> - Use try-catch for graceful error handling
> - Uncaught errors go to the listener

---

# Real-World Examples

## Data Generator

> **Async generators** are perfect for generating sequences of data.

```dart
import 'dart:math';

// Generate random numbers
Stream<int> randomNumbers(int count, int seed) async* {
  var random = Random(seed);
  for (var i = 0; i < count; i++) {
    await Future.delayed(Duration(milliseconds: 100));
    yield random.nextInt(100);
  }
}

// Generate Fibonacci sequence
Stream<int> fibonacci(int count) async* {
  int a = 0, b = 1;
  for (var i = 0; i < count; i++) {
    await Future.delayed(Duration(milliseconds: 200));
    yield a;
    var temp = a;
    a = b;
    b = temp + b;
  }
}

// Generate prime numbers (sieve algorithm)
Stream<int> primes(int max) async* {
  var isPrime = List<bool>.filled(max + 1, true);
  isPrime[0] = isPrime[1] = false;
  
  for (var i = 2; i <= max; i++) {
    if (isPrime[i]) {
      await Future.delayed(Duration(milliseconds: 100));
      yield i;
      for (var j = i * i; j <= max; j += i) {
        isPrime[j] = false;
      }
    }
  }
}

void main() async {
  print('Random numbers:');
  await for (var value in randomNumbers(5, 42)) {
    print('Random: $value');
  }
  
  print('\nFibonacci:');
  await for (var value in fibonacci(8)) {
    print('Fibonacci: $value');
  }
  
  print('\nPrimes up to 30:');
  await for (var value in primes(30)) {
    print('Prime: $value');
  }
}
```

---

## Network Pagination

> **Async generators** can handle paginated data from APIs.

```dart
class ApiClient {
  // Simulate API call with pagination
  Future<List<String>> fetchPage(int page) async {
    await Future.delayed(Duration(seconds: 1));
    
    // Simulate empty page (end of data)
    if (page > 3) return [];
    
    // Generate page data
    return List.generate(3, (index) => 'Item ${page * 3 + index + 1}');
  }
  
  // Async generator for paginated data
  Stream<String> getAllItems() async* {
    var page = 0;
    bool hasMore = true;
    
    while (hasMore) {
      // Fetch the page
      var items = await fetchPage(page);
      
      // Check if we're done
      if (items.isEmpty) {
        hasMore = false;
        break;
      }
      
      // Emit each item
      for (var item in items) {
        yield item;
      }
      
      page++;
    }
  }
}

void main() async {
  var client = ApiClient();
  
  print('Fetching all items:');
  await for (var item in client.getAllItems()) {
    print('Received: $item');
  }
  print('All items fetched!');
}

// Output:
// Fetching all items:
// (1 second delay)
// Received: Item 1
// Received: Item 2
// Received: Item 3
// (1 second delay)
// Received: Item 4
// Received: Item 5
// Received: Item 6
// (1 second delay)
// Received: Item 7
// Received: Item 8
// Received: Item 9
// All items fetched!
```

> **What's happening here?**
> - Fetches data page by page
> - Emits each item as it's fetched
> - Handles pagination automatically
> - Stops when no more data
> - Perfect for large data sets

---

# Best Practices

## Use yield* for Delegation

```dart
// Good: Delegation with yield*
Stream<int> mergedStream() async* {
  yield* stream1();
  yield* stream2();
}

// Bad: Manual iteration
Stream<int> badMerged() async* {
  await for (var value in stream1()) {
    yield value;
  }
  await for (var value in stream2()) {
    yield value;
  }
}
```

---

## Handle Errors Gracefully

```dart
// Good: Error handling in generator
Stream<int> safeGenerator() async* {
  try {
    for (var i = 0; i < 10; i++) {
      yield i;
    }
  } catch (e) {
    print('Generator error: $e');
    // Optionally yield a fallback value
    yield -1;
  }
}
```

---

# Common Mistakes

## Forgetting async*

Wrong:
```dart
Stream<int> wrongGenerator() {
  // Missing async*
  for (var i = 0; i < 5; i++) {
    yield i; // Error: yield can't be used here
  }
}
```

Correct:
```dart
Stream<int> correctGenerator() async* {
  for (var i = 0; i < 5; i++) {
    yield i;
  }
}
```

---

## Using return Instead of yield

Wrong:
```dart
Stream<int> wrongGenerator() async* {
  return 42; // Error: Can't return value from async generator
}
```

Correct:
```dart
Stream<int> correctGenerator() async* {
  yield 42; // Emit the value
}
```

---

# Summary

`async*` generators provide a natural way to create streams that emit values over time. They support async operations, error handling, and composition through `yield*`.

---

# Next Steps

Now that you understand async*, continue to:

- [sync*](sync-star.md)
- [Isolates](isolates.md)
- [Zones](zones.md)
- [Error Handling](error-handling.md)

---

# Did You Know?

- `async*` was introduced in Dart 1.9
- `yield` pauses the generator
- `yield*` delegates to another generator
- Async generators are lazy (values produced on demand)
- They work well with `await for` loops
- You can use try-catch inside async generators
- The stream closes when the generator completes

---

**Next up, bro! `sync*` (Sync Generators)!** 🚀