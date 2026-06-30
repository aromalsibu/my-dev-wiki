# Streams

Understand how to work with asynchronous data streams in Dart.

---

# What is it?

A `Stream` is a sequence of asynchronous events that delivers values over time. Unlike a `Future` which delivers a single value, a `Stream` can deliver multiple values, errors, or a completion event. Streams are ideal for handling continuous data like user input, network responses, or sensor readings.

---

# Why does it exist?

Streams exist to:

- Handle sequences of asynchronous events
- Process data in chunks or over time
- Handle continuous data streams (websockets, user input)
- Enable reactive programming patterns
- Process large data sets efficiently
- Provide backpressure handling

---

# Creating Streams

## Basic Stream Creation

> **Creating a Stream** can be done using `Stream.fromIterable()`, `Stream.value()`, or `Stream.error()` for simple cases. For more complex streams, use `async*` generators.

```dart
// Stream from iterable (multiple values)
Stream<int> numberStream = Stream.fromIterable([1, 2, 3, 4, 5]);

// Stream with a single value
Stream<String> singleValue = Stream.value('Hello');

// Stream with error
Stream<int> errorStream = Stream.error('Something went wrong');

// Using the streams
void main() async {
  // Listen to number stream
  await for (var num in numberStream) {
    print('Number: $num');
  }
  
  // Listen to single value stream
  await for (var value in singleValue) {
    print('Value: $value');
  }
  
  // Handle error stream
  try {
    await for (var value in errorStream) {
      print(value);
    }
  } catch (e) {
    print('Error: $e');
  }
}

// Output:
// Number: 1
// Number: 2
// Number: 3
// Number: 4
// Number: 5
// Value: Hello
// Error: Something went wrong
```

> **What's happening here?**
> - `Stream.fromIterable()` creates a stream from a list
> - `Stream.value()` creates a stream with one value
> - `Stream.error()` creates a stream that emits an error
> - `await for` listens to stream events
> - Streams can handle errors with try-catch

---

## Using async* Generators

> **`async*` generators** are the most common way to create custom streams. They can emit multiple values over time using the `yield` keyword.

```dart
// Async generator that emits numbers
Stream<int> countDown(int start) async* {
  for (var i = start; i >= 0; i--) {
    await Future.delayed(Duration(seconds: 1));
    yield i; // Emit the current value
  }
}

// Async generator with error
Stream<int> countWithError(int max) async* {
  for (var i = 0; i <= max; i++) {
    if (i == 3) {
      throw Exception('Error at 3!');
    }
    await Future.delayed(Duration(milliseconds: 500));
    yield i;
  }
}

// Async generator with conditional emission
Stream<int> evenNumbers(int max) async* {
  for (var i = 0; i <= max; i++) {
    if (i % 2 == 0) {
      await Future.delayed(Duration(milliseconds: 500));
      yield i;
    }
  }
}

void main() async {
  print('Counting down:');
  await for (var value in countDown(3)) {
    print('Value: $value');
  }
  print('Done!');
  
  print('\nEven numbers:');
  await for (var value in evenNumbers(10)) {
    print('Even: $value');
  }
}

// Output:
// Counting down:
// (1 second later)
// Value: 3
// (1 second later)
// Value: 2
// (1 second later)
// Value: 1
// (1 second later)
// Value: 0
// Done!
//
// Even numbers:
// Even: 0
// Even: 2
// Even: 4
// Even: 6
// Even: 8
// Even: 10
```

> **Key insights:**
> - `async*` marks a function that returns a `Stream`
> - `yield` emits a value into the stream
> - `await` can be used inside `async*`
> - Values are emitted lazily as they're requested
> - Streams can contain delays between values

---

# Listening to Streams

## Using `await for`

> **`await for`** is the most common way to listen to a stream. It pauses the function until the next event arrives.

```dart
Stream<int> getNumbers() async* {
  for (var i = 1; i <= 5; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

void main() async {
  print('Starting...');
  
  try {
    await for (var value in getNumbers()) {
      print('Received: $value');
    }
    print('Stream complete!');
  } catch (e) {
    print('Error: $e');
  }
}

// Output:
// Starting...
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
```

> **What's happening here?**
> - `await for` waits for each event
> - The loop continues until the stream closes
> - Each event is processed one at a time
> - Errors can be caught with try-catch
> - The stream completes when no more events are emitted

---

## Using `listen()`

> **The `listen()` method** provides a more flexible way to listen to streams with callbacks for data, errors, and completion.

```dart
Stream<int> getNumbers() async* {
  for (var i = 1; i <= 5; i++) {
    await Future.delayed(Duration(milliseconds: 500));
    if (i == 3) {
      throw Exception('Error at 3!');
    }
    yield i;
  }
}

void main() {
  print('Starting...');
  
  // Listen to the stream
  getNumbers().listen(
    (data) => print('Data: $data'),
    onError: (error) => print('Error: $error'),
    onDone: () => print('Done!'),
    cancelOnError: false, // Continue after errors
  );
  
  print('Listening...');
}

// Output:
// Starting...
// Listening...
// Data: 1
// Data: 2
// Error: Exception: Error at 3!
// Data: 4
// Data: 5
// Done!
```

> **Key insights:**
> - `listen()` registers callbacks for events
> - `onData` handles data events
> - `onError` handles errors
> - `onDone` is called when stream completes
> - `cancelOnError` controls whether to stop on error
> - This is the foundation of stream processing

---

# Transforming Streams

## Common Stream Transformations

> **Streams can be transformed** using methods like `map()`, `where()`, `take()`, and `skip()`.

```dart
Stream<int> getNumbers() async* {
  for (var i = 1; i <= 10; i++) {
    await Future.delayed(Duration(milliseconds: 300));
    yield i;
  }
}

void main() async {
  var stream = getNumbers();
  
  print('Original stream:');
  await for (var value in stream) {
    print(value);
  }
  
  print('\nMap (double):');
  await for (var value in getNumbers().map((n) => n * 2)) {
    print(value);
  }
  
  print('\nWhere (even):');
  await for (var value in getNumbers().where((n) => n % 2 == 0)) {
    print(value);
  }
  
  print('\nTake (first 3):');
  await for (var value in getNumbers().take(3)) {
    print(value);
  }
  
  print('\nSkip (first 5):');
  await for (var value in getNumbers().skip(5)) {
    print(value);
  }
  
  print('\nCombined transformations:');
  await for (var value in getNumbers()
      .where((n) => n % 2 == 0)
      .map((n) => n * 2)
      .take(3)) {
    print(value);
  }
}

// Output shows transformed values
// Combined: 4, 8, 12 (even numbers doubled, first 3)
```

> **What's happening here?**
> - `map()` transforms each value
> - `where()` filters values
> - `take()` limits to a number of values
> - `skip()` skips a number of values
> - Transformations can be chained together
> - Each transformation returns a new stream

---

## Reducing Streams

> **Streams can be reduced** to a single value using `reduce()`, `fold()`, and `toList()`.

```dart
Stream<int> getNumbers() async* {
  for (var i = 1; i <= 5; i++) {
    await Future.delayed(Duration(milliseconds: 300));
    yield i;
  }
}

void main() async {
  // toList - collects all values
  var list = await getNumbers().toList();
  print('List: $list'); // [1, 2, 3, 4, 5]
  
  // reduce - combines values
  var sum = await getNumbers().reduce((a, b) => a + b);
  print('Sum: $sum'); // 15
  
  // fold - combines with initial value
  var product = await getNumbers().fold(1, (a, b) => a * b);
  print('Product: $product'); // 120
  
  // first - gets first value
  var first = await getNumbers().first;
  print('First: $first'); // 1
  
  // last - gets last value
  var last = await getNumbers().last;
  print('Last: $last'); // 5
  
  // contains - checks if value exists
  var hasThree = await getNumbers().contains(3);
  print('Has 3: $hasThree'); // true
  
  // any - checks if any value matches
  var hasEven = await getNumbers().any((n) => n % 2 == 0);
  print('Has even: $hasEven'); // true
  
  // every - checks if all values match
  var allPositive = await getNumbers().every((n) => n > 0);
  print('All positive: $allPositive'); // true
}
```

> **Key insights:**
> - `toList()` collects all values into a list
> - `reduce()` combines values into a single value
> - `fold()` combines values with an initial value
> - `first` and `last` get the first/last value
> - `contains()` checks for a value
> - `any()` and `every()` check conditions

---

# Stream Types

## Single Subscription vs Broadcast Streams

> **Dart has two types of streams:** Single subscription (default) and Broadcast streams. Single subscription streams can only be listened to once.

```dart
// Single subscription stream (default)
Stream<int> singleStream() async* {
  for (var i = 1; i <= 3; i++) {
    await Future.delayed(Duration(milliseconds: 500));
    yield i;
  }
}

// Broadcast stream (can have multiple listeners)
Stream<int> broadcastStream() {
  var controller = StreamController<int>.broadcast();
  
  // Add data periodically
  var count = 0;
  Timer.periodic(Duration(seconds: 1), (timer) {
    count++;
    controller.add(count);
    if (count == 3) {
      timer.cancel();
      controller.close();
    }
  });
  
  return controller.stream;
}

void main() {
  // Single subscription - can only listen once
  var single = singleStream();
  single.listen((data) => print('Listener 1: $data'));
  // single.listen((data) => print('Listener 2')); // Error!
  
  // Broadcast - can have multiple listeners
  var broadcast = broadcastStream();
  broadcast.listen((data) => print('Broadcast 1: $data'));
  broadcast.listen((data) => print('Broadcast 2: $data'));
}

// Output:
// Single stream: 1, 2, 3
// Broadcast: 1, 2, 3 (both listeners receive all values)
```

> **What's happening here?**
> - Single subscription: one listener only
> - Broadcast: multiple listeners can subscribe
> - Single subscription is for one-time events
> - Broadcast is for sharing events with multiple listeners
> - Use `StreamController.broadcast()` for broadcast

---

# Best Practices

## Use Streams for Continuous Data

```dart
// Good: Use stream for continuous data
class UserService {
  final StreamController<User> _userController = StreamController.broadcast();
  
  // Stream of user updates
  Stream<User> get userUpdates => _userController.stream;
  
  void updateUser(User user) {
    _userController.add(user);
  }
}

// Bad: Using Future for continuous data
class BadUserService {
  User _currentUser;
  
  Future<User> getUser() async => _currentUser;
  void updateUser(User user) { _currentUser = user; }
}
```

---

## Always Cancel Subscriptions

```dart
// Good: Cancel subscriptions
class DataProcessor {
  StreamSubscription? _subscription;
  
  void startListening(Stream<String> stream) {
    _subscription = stream.listen((data) {
      print('Processing: $data');
    });
  }
  
  void dispose() {
    _subscription?.cancel(); // Always cancel!
    _subscription = null;
  }
}

// Bad: Not canceling
class BadProcessor {
  void startListening(Stream<String> stream) {
    stream.listen((data) {
      print('Processing: $data');
    }); // Subscription leaks!
  }
}
```

---

# Common Mistakes

## Not Handling Errors

Wrong:
```dart
void main() {
  getNumbers().listen((data) {
    print(data); // Errors will crash the app
  });
}
```

Correct:
```dart
void main() {
  getNumbers().listen(
    (data) => print(data),
    onError: (error) => print('Error: $error'),
  );
}
```

---

## Multiple Listeners on Single Subscription

Wrong:
```dart
Stream<int> stream = countDown(5);
stream.listen((v) => print('1: $v'));
stream.listen((v) => print('2: $v')); // Error!
```

Correct:
```dart
Stream<int> stream = countDown(5).asBroadcastStream();
stream.listen((v) => print('1: $v'));
stream.listen((v) => print('2: $v')); // Works!
```

---

# Summary

Streams provide a powerful way to handle sequences of asynchronous events. They support transformations, filtering, aggregation, and can be either single subscription or broadcast streams.

---

# Next Steps

Now that you understand streams, continue to:

- [StreamController](stream-controller.md)
- [async*](async-star.md)
- [sync*](sync-star.md)
- [Isolates](isolates.md)

---

# Did You Know?

- Streams are used extensively in Flutter for UI events
- `StreamBuilder` is a common Flutter widget
- Streams can be paused and resumed
- Broadcast streams can have multiple listeners
- `await for` is syntactic sugar over `listen()`
- Streams support backpressure handling
- Stream transformations are lazy (computed on demand)

---

