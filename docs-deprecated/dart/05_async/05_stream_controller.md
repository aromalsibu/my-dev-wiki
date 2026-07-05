# StreamController

Understand how to create and manage streams manually using StreamController in Dart.

---

# What is it?

`StreamController` is a class that provides manual control over a stream. It allows you to create a stream and add events (data, errors, and completion) to it programmatically, giving you full control over the stream's lifecycle.

---

# Why does it exist?

`StreamController` exists to:

- Create streams manually
- Add events programmatically
- Control stream lifecycle
- Implement custom stream sources
- Handle complex stream patterns
- Create broadcast streams

---

# Basic StreamController

## Creating a StreamController

> **A StreamController** provides a stream that you can control. You add events to the controller, and they are delivered to listeners.

```dart
import 'dart:async';

void main() {
  // Create a StreamController
  StreamController<String> controller = StreamController<String>();
  
  // Listen to the stream
  controller.stream.listen(
    (data) => print('Received: $data'),
    onError: (error) => print('Error: $error'),
    onDone: () => print('Stream closed'),
  );
  
  // Add events
  controller.add('First message');
  controller.add('Second message');
  controller.add('Third message');
  
  // Add an error
  controller.addError('Something went wrong');
  
  // Close the stream
  controller.close();
}

// Output:
// Received: First message
// Received: Second message
// Received: Third message
// Error: Something went wrong
// Stream closed
```

> **What's happening here?**
> - `StreamController` creates a new stream
> - `add()` pushes data events into the stream
> - `addError()` pushes an error into the stream
> - `close()` ends the stream and calls `onDone`
> - Listeners receive events as they're added

---

## Single Subscription Controller

> **Single subscription controllers** are the default type. They support only one listener and are suitable for event sequences that are consumed once.

```dart
StreamController<int> controller = StreamController<int>();

// Add events before listening
controller.add(1);
controller.add(2);

// Listen (only one listener allowed)
controller.stream.listen(
  (data) => print('Data: $data'),
  onDone: () => print('Done'),
);

controller.add(3);
controller.add(4);
controller.close();

// Output:
// Data: 1
// Data: 2
// Data: 3
// Data: 4
// Done
```

> **Key insights:**
> - Default controller type (single subscription)
> - Only one listener is allowed
> - Events are buffered before listening
> - Perfect for one-time event sequences

---

# Broadcast Controllers

## Creating Broadcast Streams

> **Broadcast controllers** allow multiple listeners to receive events simultaneously. They're useful for sharing events across multiple parts of your application.

```dart
import 'dart:async';

void main() {
  // Broadcast StreamController
  StreamController<int> controller = StreamController<int>.broadcast();
  
  // Add initial events
  controller.add(1);
  controller.add(2);
  
  // First listener
  controller.stream.listen(
    (data) => print('Listener 1: $data'),
    onDone: () => print('Listener 1 done'),
  );
  
  // Second listener
  controller.stream.listen(
    (data) => print('Listener 2: $data'),
    onDone: () => print('Listener 2 done'),
  );
  
  // Add more events
  controller.add(3);
  controller.add(4);
  controller.close();
}

// Output:
// Listener 1: 3
// Listener 2: 3
// Listener 1: 4
// Listener 2: 4
// Listener 1 done
// Listener 2 done
```

> **What's happening here?**
> - `broadcast()` creates a broadcast controller
> - Multiple listeners can subscribe
> - Each listener receives all events
> - Events after subscription are received
> - Events before subscription are not received

---

## Broadcast vs Single Subscription

```dart
// Single subscription (default)
void singleSubscriptionExample() {
  var controller = StreamController<int>();
  
  controller.stream.listen((data) => print('1: $data'));
  
  // This would cause an error
  // controller.stream.listen((data) => print('2: $data')); // Error!
  
  controller.add(42);
  controller.close();
}

// Broadcast (multiple listeners)
void broadcastExample() {
  var controller = StreamController<int>.broadcast();
  
  controller.stream.listen((data) => print('Listener 1: $data'));
  controller.stream.listen((data) => print('Listener 2: $data'));
  
  controller.add(42); // Both listeners receive this
  controller.close();
}
```

> **Key insights:**
> - Single subscription = one listener only
> - Broadcast = multiple listeners allowed
> - Choose based on your use case
> - Broadcast is more common in UI applications

---

# Advanced Features

## StreamController with Transformer

> **StreamController** can be used with transformers to modify events before they reach listeners.

```dart
import 'dart:async';

void main() {
  // Create controller
  var controller = StreamController<int>();
  
  // Transform the stream
  var transformed = controller.stream.map((value) => value * 2);
  
  // Listen to transformed stream
  transformed.listen((data) => print('Transformed: $data'));
  
  // Add events
  controller.add(1); // -> Transformed: 2
  controller.add(2); // -> Transformed: 4
  controller.add(3); // -> Transformed: 6
  
  controller.close();
}
```

---

## StreamController with Backpressure

> **StreamController** provides backpressure handling through the `onListen`, `onPause`, `onResume` callbacks.

```dart
void backpressureExample() {
  var controller = StreamController<int>(
    onListen: () => print('Listener attached'),
    onPause: () => print('Listener paused'),
    onResume: () => print('Listener resumed'),
    onCancel: () => print('Listener cancelled'),
  );
  
  var subscription = controller.stream.listen(
    (data) => print('Data: $data'),
    onDone: () => print('Done'),
  );
  
  controller.add(1);
  controller.add(2);
  
  // Pause the subscription
  subscription.pause();
  print('Paused');
  
  controller.add(3); // Buffered, not delivered yet
  controller.add(4); // Buffered, not delivered yet
  
  // Resume the subscription
  subscription.resume();
  print('Resumed');
  
  controller.add(5);
  controller.close();
}

// Output:
// Listener attached
// Data: 1
// Data: 2
// Paused
// Resumed
// Data: 3
// Data: 4
// Data: 5
// Done
// Listener cancelled
```

> **What's happening here?**
> - `onListen` fires when listener attaches
> - `onPause` fires when subscription pauses
> - `onResume` fires when subscription resumes
> - Events are buffered during pause
> - `onCancel` fires when listener cancels

---

# Real-World Examples

## UI State Management

> **StreamController** is commonly used for state management in UI applications.

```dart
class CounterState {
  final int count;
  CounterState(this.count);
}

class CounterBloc {
  final StreamController<CounterState> _stateController =
      StreamController<CounterState>();
  final StreamController<void> _incrementController =
      StreamController<void>();
  
  int _count = 0;
  
  CounterBloc() {
    // Listen to increment events
    _incrementController.stream.listen((_) {
      _count++;
      _stateController.add(CounterState(_count));
    });
  }
  
  // Stream of states
  Stream<CounterState> get state => _stateController.stream;
  
  // Sink for increment events
  Sink<void> get increment => _incrementController.sink;
  
  void dispose() {
    _stateController.close();
    _incrementController.close();
  }
}

void main() async {
  var bloc = CounterBloc();
  
  // Listen to state changes
  bloc.state.listen((state) {
    print('Count: ${state.count}');
  });
  
  // Trigger increments
  bloc.increment.add(null); // Count: 1
  bloc.increment.add(null); // Count: 2
  bloc.increment.add(null); // Count: 3
  
  bloc.dispose();
}
```

> **Key insights:**
> - BLoC pattern uses StreamController
> - Separate streams for events and states
> - Sink for adding events
> - Stream for receiving states
> - Clean disposal of controllers

---

## Event Bus

> **StreamController** is great for implementing event bus patterns.

```dart
class EventBus {
  static final EventBus _instance = EventBus._internal();
  factory EventBus() => _instance;
  EventBus._internal();
  
  final Map<Type, StreamController> _controllers = {};
  
  // Register event type
  StreamController<T> _getController<T>() {
    return _controllers.putIfAbsent(
      T,
      () => StreamController<T>.broadcast(),
    ) as StreamController<T>;
  }
  
  // Emit an event
  void emit<T>(T event) {
    _getController<T>().add(event);
  }
  
  // Listen to events
  Stream<T> on<T>() {
    return _getController<T>().stream as Stream<T>;
  }
  
  // Dispose all controllers
  void dispose() {
    for (var controller in _controllers.values) {
      controller.close();
    }
    _controllers.clear();
  }
}

// Event classes
class UserLoggedInEvent {
  final String userId;
  UserLoggedInEvent(this.userId);
}

class UserLoggedOutEvent {
  final String userId;
  UserLoggedOutEvent(this.userId);
}

void main() {
  var eventBus = EventBus();
  
  // Listen for events
  eventBus.on<UserLoggedInEvent>().listen((event) {
    print('User logged in: ${event.userId}');
  });
  
  eventBus.on<UserLoggedOutEvent>().listen((event) {
    print('User logged out: ${event.userId}');
  });
  
  // Emit events
  eventBus.emit(UserLoggedInEvent('user123'));
  eventBus.emit(UserLoggedOutEvent('user123'));
  
  eventBus.dispose();
}

// Output:
// User logged in: user123
// User logged out: user123
```

---

# Best Practices

## Always Close Controllers

```dart
// Good: Close controller when done
class DataService {
  final StreamController<String> _controller = StreamController();
  
  Stream<String> get data => _controller.stream;
  
  void dispose() {
    _controller.close(); // Always close!
  }
}

// Bad: Not closing
class BadDataService {
  final StreamController<String> _controller = StreamController();
  
  Stream<String> get data => _controller.stream;
  // Memory leak!
}
```

---

## Use Sink for Input

```dart
// Good: Use Sink for adding events
class CounterBloc {
  final StreamController<int> _controller = StreamController();
  
  // Sink for adding events
  Sink<int> get add => _controller.sink;
  
  // Stream for listening
  Stream<int> get stream => _controller.stream;
}

// Bad: Exposing controller directly
class BadBloc {
  final StreamController<int> controller = StreamController();
  // Exposed too much
}
```

---

# Common Mistakes

## Closing Controller While Adding

Wrong:
```dart
var controller = StreamController();
controller.close();
controller.add('data'); // Error! Controller closed
```

Correct:
```dart
var controller = StreamController();
controller.add('data');
controller.close(); // Close after adding
```

---

## Memory Leaks

Wrong:
```dart
class MyWidget {
  final StreamController controller = StreamController();
  
  // Controller never closed!
}
```

Correct:
```dart
class MyWidget {
  StreamController controller = StreamController();
  
  void dispose() {
    controller.close();
  }
}
```

---

# Summary

`StreamController` provides manual control over streams, allowing you to add events, handle errors, manage listeners, and control the stream lifecycle.

---

# Next Steps

Now that you understand StreamController, continue to:

- [async*](async-star.md)
- [sync*](sync-star.md)
- [Isolates](isolates.md)
- [Zones](zones.md)

---

# Did You Know?

- StreamController can be paused and resumed
- Broadcast controllers support multiple listeners
- Controllers should always be closed
- `sink` is used for adding events
- `stream` is used for listening
- Controllers support backpressure handling
- StreamController is used in BLoC pattern

---

**Next up, bro! `async*` (Async Generators)!** 🚀