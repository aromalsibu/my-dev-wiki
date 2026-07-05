# sync* (Sync Generators)

Understand how to create synchronous collections using sync* generators in Dart.

---

# What is it?

`sync*` is a keyword used to define synchronous generator functions that produce an `Iterable` of values. Unlike a regular function that returns a single value or an `async*` generator that produces a `Stream`, a `sync*` generator produces an `Iterable` that can be iterated over synchronously.

---

# Why does it exist?

`sync*` exists to:

- Create iterables with lazy evaluation
- Generate sequences of values efficiently
- Produce infinite or large sequences without memory overhead
- Simplify the creation of custom iterables
- Enable functional programming patterns
- Provide lazy computation on demand

---

# Basic sync* Syntax

## Creating a Sync Generator

> **A `sync*` function** returns an `Iterable` and uses `yield` to emit values synchronously. Values are generated lazily as they're requested.

```dart
// Basic sync generator
Iterable<int> countUp(int max) sync* {
  for (var i = 1; i <= max; i++) {
    yield i; // Emit the current value
  }
}

void main() {
  print('Counting up:');
  
  // Iterate over the iterable
  for (var value in countUp(5)) {
    print('Value: $value');
  }
  
  print('Done!');
}

// Output:
// Counting up:
// Value: 1
// Value: 2
// Value: 3
// Value: 4
// Value: 5
// Done!
```

> **What's happening here?**
> - `sync*` marks the function as a sync generator
> - It returns an `Iterable<int>`
> - `yield` emits a value into the iterable
> - Values are generated lazily as they're requested
> - The iterable can be used in for-in loops

---

## Sync Generator with Conditions

> **Sync generators** can include conditions and control flow to create complex sequences.

```dart
// Generator with conditions
Iterable<int> evenNumbers(int max) sync* {
  for (var i = 1; i <= max; i++) {
    if (i % 2 == 0) {
      yield i;
    }
  }
}

// Generator with early return
Iterable<int> limitedNumbers(int max) sync* {
  for (var i = 1; i <= max; i++) {
    if (i > 10) {
      return; // Stops the generator
    }
    yield i;
  }
}

// Generator with break
Iterable<int> numbersUntil(int max, int stop) sync* {
  for (var i = 1; i <= max; i++) {
    if (i == stop) {
      break; // Breaks the loop, generator ends
    }
    yield i;
  }
}

void main() {
  print('Even numbers:');
  for (var value in evenNumbers(10)) {
    print('Even: $value');
  }
  
  print('\nLimited numbers:');
  for (var value in limitedNumbers(15)) {
    print('Value: $value');
  }
  
  print('\nNumbers until stop:');
  for (var value in numbersUntil(10, 5)) {
    print('Value: $value');
  }
}

// Output:
// Even numbers: 2, 4, 6, 8, 10
// Limited numbers: 1-10 (stops at 10)
// Numbers until stop: 1-4 (stops at 5)
```

> **Key insights:**
> - Conditions work inside sync generators
> - `return` ends the generator early
> - `break` can exit loops
> - All control flow works naturally

---

# Advanced sync*

## Nested Sync Generators

> **You can yield** values from other iterables or generators using `yield*`.

```dart
// Helper generator
Iterable<int> generateNumbers(int start, int count) sync* {
  for (var i = 0; i < count; i++) {
    yield start + i;
  }
}

// Main generator using yield*
Iterable<int> combinedNumbers() sync* {
  // Emit values from first generator
  yield* generateNumbers(1, 3); // 1, 2, 3
  
  // Emit values from second generator
  yield* generateNumbers(10, 3); // 10, 11, 12
  
  // Emit a single value
  yield 100;
}

void main() {
  print('Combined numbers:');
  for (var value in combinedNumbers()) {
    print('Value: $value');
  }
}

// Output:
// Combined numbers:
// Value: 1
// Value: 2
// Value: 3
// Value: 10
// Value: 11
// Value: 12
// Value: 100
```

> **What's happening here?**
> - `yield*` delegates to another generator
> - It emits all values from the nested generator
> - You can combine multiple generators
> - This is useful for composing iterables

---

## Infinite Generators

> **Sync generators** can create infinite sequences using `while(true)` or recursion.

```dart
// Infinite generator
Iterable<int> infiniteNumbers() sync* {
  var i = 0;
  while (true) {
    yield i++;
  }
}

// Fibonacci sequence (infinite)
Iterable<int> fibonacciSequence() sync* {
  int a = 0, b = 1;
  while (true) {
    yield a;
    var temp = a;
    a = b;
    b = temp + b;
  }
}

// Recursive generator
Iterable<int> recursiveRange(int start, int end) sync* {
  if (start <= end) {
    yield start;
    yield* recursiveRange(start + 1, end);
  }
}

void main() {
  // Take only what you need
  print('First 5 infinite numbers:');
  var numbers = infiniteNumbers().take(5);
  for (var value in numbers) {
    print('Value: $value');
  }
  
  print('\nFirst 8 Fibonacci numbers:');
  var fib = fibonacciSequence().take(8);
  for (var value in fib) {
    print('Fibonacci: $value');
  }
  
  print('\nRecursive range:');
  for (var value in recursiveRange(1, 5)) {
    print('Value: $value');
  }
}

// Output:
// First 5 infinite numbers: 0, 1, 2, 3, 4
// First 8 Fibonacci numbers: 0, 1, 1, 2, 3, 5, 8, 13
// Recursive range: 1, 2, 3, 4, 5
```

> **Key insights:**
> - Infinite generators are possible with `while(true)`
> - Use `.take(n)` to limit the sequence
> - Recursive generators work with `yield*`
> - Generators are lazy - values are only computed when needed

---

# Real-World Examples

## Data Processing Pipeline

> **Sync generators** are excellent for building data processing pipelines.

```dart
// Generate a sequence of numbers
Iterable<int> generateNumbers(int count) sync* {
  for (var i = 0; i < count; i++) {
    yield i;
  }
}

// Filter even numbers
Iterable<int> filterEven(Iterable<int> source) sync* {
  for (var value in source) {
    if (value % 2 == 0) {
      yield value;
    }
  }
}

// Transform numbers (double them)
Iterable<int> transformDouble(Iterable<int> source) sync* {
  for (var value in source) {
    yield value * 2;
  }
}

// Limit the sequence
Iterable<int> limitSequence(Iterable<int> source, int limit) sync* {
  var count = 0;
  for (var value in source) {
    if (count >= limit) break;
    yield value;
    count++;
  }
}

void main() {
  // Build a pipeline
  var pipeline = generateNumbers(20)
      .transform(filterEven)
      .transform(transformDouble)
      .transform((source) => limitSequence(source, 3));
  
  print('Pipeline result:');
  for (var value in pipeline) {
    print('Value: $value');
  }
}

// Output:
// Pipeline result:
// Value: 0
// Value: 4
// Value: 8
```

> **What's happening here?**
> - Each transformation returns an iterable
> - Data flows through the pipeline
> - Values are processed lazily
> - Memory efficient for large sequences

---

## Custom Collection Operations

> **Sync generators** can implement custom collection operations.

```dart
class CollectionUtils {
  // Custom map operation
  static Iterable<R> map<T, R>(Iterable<T> source, R Function(T) mapper) sync* {
    for (var item in source) {
      yield mapper(item);
    }
  }
  
  // Custom filter operation
  static Iterable<T> filter<T>(Iterable<T> source, bool Function(T) predicate) sync* {
    for (var item in source) {
      if (predicate(item)) {
        yield item;
      }
    }
  }
  
  // Custom reduce to list
  static List<T> reduceToList<T>(Iterable<T> source, int max) {
    var result = <T>[];
    var count = 0;
    for (var item in source) {
      if (count >= max) break;
      result.add(item);
      count++;
    }
    return result;
  }
  
  // Custom take operation
  static Iterable<T> take<T>(Iterable<T> source, int count) sync* {
    var taken = 0;
    for (var item in source) {
      if (taken >= count) break;
      yield item;
      taken++;
    }
  }
  
  // Custom distinct operation
  static Iterable<T> distinct<T>(Iterable<T> source) sync* {
    var seen = <T>{};
    for (var item in source) {
      if (!seen.contains(item)) {
        seen.add(item);
        yield item;
      }
    }
  }
}

void main() {
  var numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
  
  // Using custom operations
  var doubled = CollectionUtils.map(numbers, (n) => n * 2);
  var evens = CollectionUtils.filter(numbers, (n) => n % 2 == 0);
  var first3 = CollectionUtils.take(numbers, 3);
  var unique = CollectionUtils.distinct([1, 2, 2, 3, 3, 4]);
  
  print('Doubled: ${doubled.toList()}');
  print('Evens: ${evens.toList()}');
  print('First 3: ${first3.toList()}');
  print('Distinct: ${unique.toList()}');
}

// Output:
// Doubled: [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
// Evens: [2, 4, 6, 8, 10]
// First 3: [1, 2, 3]
// Distinct: [1, 2, 3, 4]
```

---

# Best Practices

## Use sync* for Lazy Evaluation

```dart
// Good: Lazy evaluation with sync*
Iterable<int> getLargeNumbers() sync* {
  for (var i = 0; i < 1000000; i++) {
    yield i; // Only generates as needed
  }
}

// Bad: Eager evaluation with List
List<int> getLargeList() {
  return List.generate(1000000, (i) => i); // Generates all at once
}
```

---

## Use yield* for Delegation

```dart
// Good: Delegation with yield*
Iterable<int> mergedIterable() sync* {
  yield* iterable1();
  yield* iterable2();
}

// Bad: Manual iteration
Iterable<int> badMerged() sync* {
  for (var value in iterable1()) {
    yield value;
  }
  for (var value in iterable2()) {
    yield value;
  }
}
```

---

# Common Mistakes

## Forgetting sync*

Wrong:
```dart
Iterable<int> wrongGenerator() {
  // Missing sync*
  for (var i = 0; i < 5; i++) {
    yield i; // Error: yield can't be used here
  }
}
```

Correct:
```dart
Iterable<int> correctGenerator() sync* {
  for (var i = 0; i < 5; i++) {
    yield i;
  }
}
```

---

## Using await in sync*

Wrong:
```dart
Iterable<int> wrongGenerator() sync* {
  await Future.delayed(Duration(seconds: 1)); // Error: Can't use await in sync*
  yield 42;
}
```

Correct:
```dart
Stream<int> correctGenerator() async* {
  await Future.delayed(Duration(seconds: 1)); // Use async* for async
  yield 42;
}
```

---

# Summary

`sync*` generators provide a powerful way to create lazy sequences of values. They're memory-efficient, support infinite sequences, and work perfectly for data processing pipelines.

---

# Next Steps

Now that you understand sync*, continue to:

- [Isolates](isolates.md)
- [Zones](zones.md)
- [Error Handling](error-handling.md)
- [Core Libraries](core-libraries.md)

---

# Did You Know?

- `sync*` was introduced in Dart 1.9
- Values are computed lazily (on demand)
- `yield` pauses the generator
- `yield*` delegates to another generator
- Sync generators can be infinite
- They're more efficient than creating full lists
- Use `.take()` to limit infinite sequences

---
