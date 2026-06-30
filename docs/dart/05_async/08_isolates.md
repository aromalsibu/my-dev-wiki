# Isolates

Understand how to achieve concurrency using Isolates in Dart.

---

# What is it?

Isolates are Dart's model for concurrent programming. Each isolate runs in its own memory space with its own event loop, communicating only through message passing. Unlike threads that share memory, isolates are completely independent, making them safer and more predictable.

---

# Why does it exist?

Isolates exist to:

- Achieve true parallelism (multiple CPU cores)
- Prevent shared memory issues (race conditions)
- Run expensive computations without blocking UI
- Handle multiple tasks simultaneously
- Improve application performance
- Create responsive applications

---

# Basic Isolates

## What is an Isolate?

> **An Isolate** is like a lightweight process that runs independently. Each isolate has its own memory, event loop, and can only communicate via messages.

```dart
import 'dart:isolate';

// Function that runs in the isolate
void isolateFunction(SendPort sendPort) {
  // Send a message back to the main isolate
  sendPort.send('Hello from isolate!');
}

void main() async {
  // Create a ReceivePort to receive messages
  ReceivePort receivePort = ReceivePort();
  
  // Spawn a new isolate
  Isolate isolate = await Isolate.spawn(
    isolateFunction,
    receivePort.sendPort,
  );
  
  // Listen for messages
  receivePort.listen((message) {
    print('Received: $message');
    // Clean up
    receivePort.close();
    isolate.kill();
  });
  
  print('Main isolate continuing...');
}

// Output:
// Main isolate continuing...
// Received: Hello from isolate!
```

> **What's happening here?**
> - `Isolate.spawn()` creates a new isolate
> - The isolate runs `isolateFunction` in its own memory space
> - `SendPort` is used to send messages back to the main isolate
> - `ReceivePort` listens for incoming messages
> - Isolates communicate via message passing

---

## Two-Way Communication

> **Isolates can communicate** in both directions by exchanging SendPorts.

```dart
import 'dart:isolate';

// Worker isolate function
void workerIsolate(SendPort mainSendPort) {
  // Create a receive port for this isolate
  ReceivePort receivePort = ReceivePort();
  
  // Send our receive port back to main
  mainSendPort.send(receivePort.sendPort);
  
  // Listen for messages from main
  receivePort.listen((message) {
    if (message is String) {
      print('Worker received: $message');
      
      // Process the message
      String result = 'Processed: ${message.toUpperCase()}';
      
      // Send result back to main
      mainSendPort.send(result);
    }
  });
}

void main() async {
  ReceivePort mainReceivePort = ReceivePort();
  
  // Spawn the worker isolate
  Isolate isolate = await Isolate.spawn(
    workerIsolate,
    mainReceivePort.sendPort,
  );
  
  // Get the worker's send port
  SendPort? workerSendPort;
  await for (var message in mainReceivePort) {
    if (message is SendPort) {
      workerSendPort = message;
      break;
    }
  }
  
  // Send messages to the worker
  workerSendPort!.send('Hello');
  workerSendPort!.send('World');
  
  // Listen for responses
  int received = 0;
  mainReceivePort.listen((message) {
    if (message is String) {
      print('Main received: $message');
      received++;
      if (received == 2) {
        mainReceivePort.close();
        isolate.kill();
      }
    }
  });
}

// Output:
// Worker received: Hello
// Worker received: World
// Main received: Processed: HELLO
// Main received: Processed: WORLD
```

> **Key insights:**
> - Both isolates exchange SendPorts
> - Each isolate has its own ReceivePort
> - Messages are sent asynchronously
> - Communication is one-way (send and receive)
> - This enables bidirectional communication

---

# Isolate Communication

## Passing Complex Data

> **Isolates can pass** complex data structures, but they must be sendable (primitive types, lists, maps, etc.).

```dart
import 'dart:isolate';

// Data classes
class Task {
  final int id;
  final String name;
  final List<int> numbers;
  
  Task(this.id, this.name, this.numbers);
  
  // Convert to map for sending
  Map<String, dynamic> toMap() => {
    'id': id,
    'name': name,
    'numbers': numbers,
  };
  
  // Create from map
  factory Task.fromMap(Map<String, dynamic> map) {
    return Task(
      map['id'] as int,
      map['name'] as String,
      (map['numbers'] as List).cast<int>(),
    );
  }
}

class Result {
  final int taskId;
  final int sum;
  final int count;
  
  Result(this.taskId, this.sum, this.count);
  
  Map<String, dynamic> toMap() => {
    'taskId': taskId,
    'sum': sum,
    'count': count,
  };
  
  factory Result.fromMap(Map<String, dynamic> map) {
    return Result(
      map['taskId'] as int,
      map['sum'] as int,
      map['count'] as int,
    );
  }
}

// Worker isolate
void workerIsolate(SendPort mainSendPort) {
  ReceivePort receivePort = ReceivePort();
  mainSendPort.send(receivePort.sendPort);
  
  receivePort.listen((message) {
    if (message is Map<String, dynamic>) {
      // Parse the task
      var task = Task.fromMap(message);
      print('Worker processing task: ${task.name}');
      
      // Compute result
      int sum = task.numbers.reduce((a, b) => a + b);
      int count = task.numbers.length;
      
      // Send result back
      var result = Result(task.id, sum, count);
      mainSendPort.send(result.toMap());
    }
  });
}

void main() async {
  ReceivePort mainReceivePort = ReceivePort();
  Isolate isolate = await Isolate.spawn(
    workerIsolate,
    mainReceivePort.sendPort,
  );
  
  // Get worker's send port
  SendPort? workerSendPort;
  await for (var message in mainReceivePort) {
    if (message is SendPort) {
      workerSendPort = message;
      break;
    }
  }
  
  // Create tasks
  var tasks = [
    Task(1, 'Sum 1-10', [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]),
    Task(2, 'Sum evens', [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]),
    Task(3, 'Sum odds', [1, 3, 5, 7, 9, 11, 13, 15, 17, 19]),
  ];
  
  // Send tasks to worker
  for (var task in tasks) {
    workerSendPort!.send(task.toMap());
  }
  
  // Collect results
  int received = 0;
  mainReceivePort.listen((message) {
    if (message is Map<String, dynamic>) {
      var result = Result.fromMap(message);
      print('Task ${result.taskId}: Sum = ${result.sum}, Count = ${result.count}');
      received++;
      if (received == tasks.length) {
        mainReceivePort.close();
        isolate.kill();
      }
    }
  });
}

// Output:
// Worker processing task: Sum 1-10
// Worker processing task: Sum evens
// Worker processing task: Sum odds
// Task 1: Sum = 55, Count = 10
// Task 2: Sum = 110, Count = 10
// Task 3: Sum = 100, Count = 10
```

> **What's happening here?**
> - Complex data is serialized to maps
> - Tasks are sent to the worker isolate
> - Worker processes each task
> - Results are sent back to main
> - This is how you pass data between isolates

---

# Isolate Lifecycle

## Managing Isolates

> **Isolates have** a lifecycle that includes creation, running, and termination.

```dart
import 'dart:isolate';

void workerFunction(SendPort sendPort) {
  print('Worker: Starting');
  
  // Do some work
  for (var i = 1; i <= 5; i++) {
    print('Worker: Working... $i');
    // Simulate work
  }
  
  print('Worker: Finished');
  sendPort.send('Done');
}

void main() async {
  ReceivePort receivePort = ReceivePort();
  
  // Spawn isolate
  print('Main: Creating isolate');
  Isolate isolate = await Isolate.spawn(
    workerFunction,
    receivePort.sendPort,
  );
  
  print('Main: Isolate created');
  
  // Handle completion
  receivePort.listen((message) {
    print('Main: Received: $message');
    // Clean up
    receivePort.close();
    isolate.kill(priority: Isolate.immediate);
    print('Main: Isolate killed');
  });
}

// Output:
// Main: Creating isolate
// Main: Isolate created
// Worker: Starting
// Worker: Working... 1
// Worker: Working... 2
// Worker: Working... 3
// Worker: Working... 4
// Worker: Working... 5
// Worker: Finished
// Main: Received: Done
// Main: Isolate killed
```

> **Key insights:**
> - Isolates are created with `spawn()`
> - They run asynchronously
> - They can be killed with `kill()`
> - Use `priority: Isolate.immediate` for immediate termination
> - Always clean up resources

---

# Real-World Examples

## Parallel Processing

> **Isolates are perfect** for CPU-intensive operations that would block the UI.

```dart
import 'dart:isolate';

// CPU-intensive task: Fibonacci calculation
int fibonacci(int n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Worker isolate for Fibonacci
void fibWorker(SendPort sendPort) {
  ReceivePort receivePort = ReceivePort();
  sendPort.send(receivePort.sendPort);
  
  receivePort.listen((message) {
    if (message is int) {
      print('Worker: Calculating Fibonacci of $message');
      int result = fibonacci(message);
      sendPort.send(result);
    }
  });
}

void main() async {
  ReceivePort mainPort = ReceivePort();
  Isolate isolate = await Isolate.spawn(fibWorker, mainPort.sendPort);
  
  // Get worker's send port
  SendPort? workerPort;
  await for (var message in mainPort) {
    if (message is SendPort) {
      workerPort = message;
      break;
    }
  }
  
  // Calculate Fibonacci numbers in parallel
  List<int> numbers = [35, 36, 37, 38, 39, 40];
  Map<int, Future<int>> futures = {};
  
  // Send each number to the worker
  for (var n in numbers) {
    var completer = Completer<int>();
    futures[n] = completer.future;
    
    // Store completer for when result arrives
    workerPort!.send(n);
  }
  
  // Collect results
  int received = 0;
  mainPort.listen((message) {
    if (message is int) {
      received++;
      print('Result: $message');
      if (received == numbers.length) {
        mainPort.close();
        isolate.kill();
      }
    }
  });
}
```

---

## Load Balancer

> **Multiple isolates** can be used to create a load balancer for parallel processing.

```dart
import 'dart:isolate';

class LoadBalancer {
  final List<SendPort> _workers = [];
  final List<bool> _isBusy = [];
  final List<Completer> _completers = [];
  
  LoadBalancer(int workerCount) {
    _initWorkers(workerCount);
  }
  
  Future<void> _initWorkers(int count) async {
    for (var i = 0; i < count; i++) {
      final completer = Completer<SendPort>();
      
      // Spawn worker
      Isolate.spawn(_workerFunction, completer);
      
      // Wait for worker to be ready
      final workerPort = await completer.future;
      _workers.add(workerPort);
      _isBusy.add(false);
    }
  }
  
  static void _workerFunction(SendPort mainPort) {
    ReceivePort workerPort = ReceivePort();
    mainPort.send(workerPort.sendPort);
    
    workerPort.listen((message) {
      // Process the work
      // Send result back
    });
  }
  
  Future<dynamic> execute(dynamic task) async {
    // Find free worker
    int index = _isBusy.indexOf(false);
    if (index == -1) {
      // Wait for a worker to become free
      // Implementation for waiting
    }
    
    _isBusy[index] = true;
    final completer = Completer();
    _completers.add(completer);
    
    // Send task to worker
    _workers[index].send(task);
    
    return completer.future;
  }
}
```

---

# Best Practices

## Use Isolates for CPU-Intensive Work

```dart
// Good: Use isolate for heavy computation
void heavyComputation() {
  Isolate.spawn(computeTask, sendPort);
}

// Bad: Block the main isolate with heavy computation
void heavyComputationBlocking() {
  // This will block the UI
  for (var i = 0; i < 1000000000; i++) {
    // Heavy computation
  }
}
```

---

## Always Clean Up

```dart
// Good: Clean up resources
void useIsolate() async {
  var receivePort = ReceivePort();
  var isolate = await Isolate.spawn(worker, receivePort.sendPort);
  
  // Use the isolate...
  
  // Clean up
  receivePort.close();
  isolate.kill();
}

// Bad: Leaking resources
void badUseIsolate() async {
  var receivePort = ReceivePort();
  var isolate = await Isolate.spawn(worker, receivePort.sendPort);
  // Never cleaned up!
}
```

---

# Common Mistakes

## Forgetting to Close ReceivePort

Wrong:
```dart
void main() async {
  var receivePort = ReceivePort();
  await Isolate.spawn(worker, receivePort.sendPort);
  // ReceivePort never closed
}
```

Correct:
```dart
void main() async {
  var receivePort = ReceivePort();
  var isolate = await Isolate.spawn(worker, receivePort.sendPort);
  // Use the isolate...
  receivePort.close();
  isolate.kill();
}
```

---

## Passing Non-Sendable Data

Wrong:
```dart
void worker(SendPort sendPort) {
  // Can't pass functions or class instances
  sendPort.send(() => print('Hello')); // Error!
}
```

Correct:
```dart
void worker(SendPort sendPort) {
  // Pass only sendable types
  sendPort.send('Hello'); // Works
  sendPort.send(42); // Works
  sendPort.send({'key': 'value'}); // Works
}
```

---

# Summary

Isolates provide safe, concurrent execution in Dart. They run independently with their own memory and event loops, communicating only through message passing.

---

# Next Steps

Now that you understand isolates, continue to:

- [Zones](zones.md)
- [Error Handling](error-handling.md)
- [Core Libraries](core-libraries.md)
- [Concurrency](concurrency.md)

---

# Did You Know?

- Isolates don't share memory
- Each isolate has its own event loop
- Isolates can use multiple CPU cores
- Spawning isolates has some overhead
- Isolates communicate via message passing
- You can pause, resume, and kill isolates
- Isolates are used in Flutter for background tasks

---

