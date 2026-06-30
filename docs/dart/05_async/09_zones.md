# Zones

Understand how to manage and control code execution using Zones in Dart.

---

# What is it?

Zones are an execution context that allows you to intercept and control code execution. They provide a way to manage asynchronous operations, handle errors globally, and add cross-cutting concerns like logging and profiling without modifying existing code.

---

# Why does it exist?

Zones exist to:

- Intercept asynchronous operations
- Handle uncaught errors globally
- Add logging and monitoring
- Manage execution context (like HTTP headers)
- Enable debugging and profiling
- Control code execution flow

---

# Basic Zones

## Creating a Zone

> **A Zone** is created by forking the current zone. Each zone can have custom behavior specified through `ZoneSpecification`.

```dart
import 'dart:async';

void main() {
  print('Outside zone');
  
  // Create a zone with custom behavior
  Zone zone = Zone.current.fork(
    specification: ZoneSpecification(
      // Intercept print operations
      print: (Zone self, ZoneDelegate parent, Zone zone, String message) {
        parent.print(zone, '[ZONE] $message');
      },
    ),
  );
  
  // Run code inside the zone
  zone.run(() {
    print('Inside zone');
    print('Another message');
  });
  
  print('Back outside');
}

// Output:
// Outside zone
// [ZONE] Inside zone
// [ZONE] Another message
// Back outside
```

> **What's happening here?**
> - `Zone.current` gets the current zone
> - `fork()` creates a new zone with custom behavior
> - `ZoneSpecification` defines custom behavior
> - `print()` is intercepted and modified
> - `run()` executes code within the zone

---

## Zone with Multiple Interceptions

> **Zones can intercept** multiple operations like print, scheduleMicrotask, and timers.

```dart
import 'dart:async';

void main() {
  Zone zone = Zone.current.fork(
    specification: ZoneSpecification(
      // Intercept print
      print: (Zone self, ZoneDelegate parent, Zone zone, String message) {
        parent.print(zone, 'LOG: $message');
      },
      
      // Intercept microtasks
      scheduleMicrotask: (
        Zone self,
        ZoneDelegate parent,
        Zone zone,
        void Function() task,
      ) {
        parent.print(zone, 'Scheduling microtask');
        parent.scheduleMicrotask(zone, () {
          parent.print(zone, 'Running microtask');
          task();
        });
      },
      
      // Intercept timer creation
      createTimer: (
        Zone self,
        ZoneDelegate parent,
        Zone zone,
        Duration duration,
        void Function() f,
      ) {
        parent.print(zone, 'Creating timer: $duration');
        return parent.createTimer(zone, duration, () {
          parent.print(zone, 'Timer fired');
          f();
        });
      },
    ),
  );
  
  zone.run(() {
    print('Hello');
    scheduleMicrotask(() => print('Microtask'));
    Timer(Duration(seconds: 1), () => print('Timer'));
  });
}

// Output:
// LOG: Hello
// Scheduling microtask
// LOG: Running microtask
// LOG: Microtask
// Creating timer: 0:00:01.000000
// (1 second later)
// Timer fired
// LOG: Timer
```

> **Key insights:**
> - Multiple operations can be intercepted
> - Microtasks are intercepted and logged
> - Timers are intercepted with logging
> - The original behavior is preserved via `parent`
> - This is useful for debugging and monitoring

---

# Error Handling

## Zone Error Handling

> **Zones can handle** uncaught errors globally, preventing crashes and providing centralized error handling.

```dart
import 'dart:async';

void main() {
  Zone zone = Zone.current.fork(
    specification: ZoneSpecification(
      // Handle uncaught errors
      handleUncaughtError: (
        Zone self,
        ZoneDelegate parent,
        Zone zone,
        Object error,
        StackTrace stack,
      ) {
        parent.print(zone, 'Caught error: $error');
        parent.print(zone, 'Stack: $stack');
      },
      
      // Intercept print
      print: (Zone self, ZoneDelegate parent, Zone zone, String message) {
        parent.print(zone, '[ZONE] $message');
      },
    ),
  );
  
  // Run code that throws an error
  zone.run(() {
    print('Starting');
    throw Exception('Something went wrong!');
    print('This never executes');
  });
  
  print('App continues running');
}

// Output:
// [ZONE] Starting
// [ZONE] Caught error: Exception: Something went wrong!
// [ZONE] Stack: ...
// App continues running
```

> **What's happening here?**
> - `handleUncaughtError` catches any uncaught errors
> - The error doesn't crash the app
> - You can log, report, or handle the error
> - The app continues running
> - This is perfect for global error handling

---

## Custom Error Handling

> **Zones can implement** complex error handling strategies.

```dart
import 'dart:async';

class ErrorHandler {
  final List<Object> errors = [];
  
  void handleError(Object error, StackTrace stack) {
    errors.add(error);
    print('Error logged: $error');
  }
}

void main() {
  var handler = ErrorHandler();
  
  Zone zone = Zone.current.fork(
    specification: ZoneSpecification(
      handleUncaughtError: (
        Zone self,
        ZoneDelegate parent,
        Zone zone,
        Object error,
        StackTrace stack,
      ) {
        // Log the error
        handler.handleError(error, stack);
        
        // Re-throw in some cases
        if (error is Exception && error.toString().contains('fatal')) {
          // Let it crash for fatal errors
          parent.handleUncaughtError(zone, error, stack);
        }
      },
    ),
  );
  
  zone.run(() {
    // Non-fatal error
    throw Exception('Network timeout');
  });
  
  zone.run(() {
    // Fatal error
    throw Exception('fatal: Out of memory');
  });
  
  print('Total errors: ${handler.errors.length}');
}

// Output:
// Error logged: Exception: Network timeout
// Total errors: 1
// (Then fatal error crashes)
```

> **Key insights:**
> - You can implement custom error logging
> - Decide which errors to handle vs. propagate
> - Track errors across the application
> - This is great for error reporting services

---

# Zone Local Values

## Storing Context

> **Zones can store** local values that are accessible within the zone, useful for context like request IDs, user sessions, or configuration.

```dart
import 'dart:async';

void main() {
  // Create zone with local value
  Zone zone = Zone.current.fork(
    zoneValues: {
      'requestId': 'REQ-12345',
      'userId': 'USER-42',
      'isDebug': true,
    },
  );
  
  zone.run(() {
    // Access zone values
    var requestId = Zone.current['requestId'] as String;
    var userId = Zone.current['userId'] as String;
    var isDebug = Zone.current['isDebug'] as bool;
    
    print('Request: $requestId');
    print('User: $userId');
    print('Debug: $isDebug');
    
    // Nested zone inherits values
    var nestedZone = Zone.current.fork(
      zoneValues: {
        'requestId': 'REQ-67890', // Override
      },
    );
    
    nestedZone.run(() {
      var requestId = Zone.current['requestId'] as String;
      var userId = Zone.current['userId'] as String;
      
      print('Nested request: $requestId');
      print('Nested user: $userId');
    });
  });
}

// Output:
// Request: REQ-12345
// User: USER-42
// Debug: true
// Nested request: REQ-67890
// Nested user: USER-42
```

> **What's happening here?**
> - `zoneValues` stores context data
> - Values are accessible with `Zone.current['key']`
> - Nested zones inherit values
> - Values can be overridden in child zones
> - This is great for request contexts

---

# Real-World Examples

## HTTP Request Context

> **Zones are perfect** for managing HTTP request contexts with logging and error handling.

```dart
import 'dart:async';

class RequestContext {
  final String requestId;
  final String userId;
  final DateTime timestamp;
  
  RequestContext(this.requestId, this.userId, this.timestamp);
  
  @override
  String toString() => '$requestId (User: $userId)';
}

class ApiService {
  Future<void> handleRequest(String userId) async {
    // Create a zone with request context
    var zone = Zone.current.fork(
      zoneValues: {
        'context': RequestContext(
          'REQ-${DateTime.now().millisecondsSinceEpoch}',
          userId,
          DateTime.now(),
        ),
      },
      specification: ZoneSpecification(
        handleUncaughtError: (
          Zone self,
          ZoneDelegate parent,
          Zone zone,
          Object error,
          StackTrace stack,
        ) {
          // Get context for error logging
          var context = zone['context'] as RequestContext;
          print('Error in request ${context.requestId}: $error');
          print('User: ${context.userId}');
          // Don't rethrow - handle gracefully
        },
        print: (Zone self, ZoneDelegate parent, Zone zone, String message) {
          var context = zone['context'] as RequestContext?;
          if (context != null) {
            parent.print(zone, '[${context.requestId}] $message');
          } else {
            parent.print(zone, message);
          }
        },
      ),
    );
    
    // Process request in the zone
    await zone.run(() => _processRequest(userId));
  }
  
  Future<void> _processRequest(String userId) async {
    var context = Zone.current['context'] as RequestContext;
    print('Processing request: $context');
    
    // Simulate work
    await Future.delayed(Duration(seconds: 1));
    
    // Simulate potential error
    if (userId == 'error') {
      throw Exception('Invalid user');
    }
    
    print('Request completed successfully');
  }
}

void main() async {
  var api = ApiService();
  
  print('=== Successful request ===');
  await api.handleRequest('user123');
  
  print('\n=== Failed request ===');
  await api.handleRequest('error');
  
  print('\n=== Done ===');
}

// Output:
// === Successful request ===
// [REQ-1234567890] Processing request: REQ-1234567890 (User: user123)
// [REQ-1234567890] Request completed successfully
//
// === Failed request ===
// [REQ-1234567891] Processing request: REQ-1234567891 (User: error)
// Error in request REQ-1234567891: Exception: Invalid user
// User: error
//
// === Done ===
```

---

## Performance Monitoring

> **Zones can be used** to monitor and measure performance of code execution.

```dart
import 'dart:async';

class PerformanceMonitor {
  static final Map<String, List<Duration>> _measurements = {};
  
  static void recordMeasurement(String name, Duration duration) {
    _measurements.putIfAbsent(name, () => []).add(duration);
    print('⏱ $name took ${duration.inMilliseconds}ms');
  }
  
  static void printStats() {
    print('\n=== Performance Stats ===');
    for (var entry in _measurements.entries) {
      var durations = entry.value;
      var avg = durations.reduce((a, b) => a + b) ~/ durations.length;
      print('${entry.key}:');
      print('  Count: ${durations.length}');
      print('  Average: ${avg.inMilliseconds}ms');
      print('  Min: ${durations.reduce((a, b) => a < b ? a : b).inMilliseconds}ms');
      print('  Max: ${durations.reduce((a, b) => a > b ? a : b).inMilliseconds}ms');
    }
  }
}

void main() {
  // Create zone with performance monitoring
  var zone = Zone.current.fork(
    specification: ZoneSpecification(
      // Monitor timers
      createTimer: (
        Zone self,
        ZoneDelegate parent,
        Zone zone,
        Duration duration,
        void Function() f,
      ) {
        var start = DateTime.now();
        return parent.createTimer(zone, duration, () {
          var end = DateTime.now();
          var elapsed = end.difference(start);
          PerformanceMonitor.recordMeasurement('Timer', elapsed);
          f();
        });
      },
      
      // Monitor microtasks
      scheduleMicrotask: (
        Zone self,
        ZoneDelegate parent,
        Zone zone,
        void Function() task,
      ) {
        var start = DateTime.now();
        parent.scheduleMicrotask(zone, () {
          var end = DateTime.now();
          var elapsed = end.difference(start);
          PerformanceMonitor.recordMeasurement('Microtask', elapsed);
          task();
        });
      },
    ),
  );
  
  zone.run(() {
    print('Starting performance tests...');
    
    // Several operations
    for (var i = 0; i < 5; i++) {
      Timer(Duration(milliseconds: 100 * (i + 1)), () {
        print('Timer $i done');
      });
    }
    
    scheduleMicrotask(() => print('Microtask 1'));
    scheduleMicrotask(() => print('Microtask 2'));
  });
  
  // Wait for all operations
  Future.delayed(Duration(seconds: 3), () {
    PerformanceMonitor.printStats();
  });
}
```

---

# Best Practices

## Use Zones for Cross-Cutting Concerns

```dart
// Good: Zones for logging
void setupLogging() {
  Zone.current.fork(
    specification: ZoneSpecification(
      print: (Zone self, ZoneDelegate parent, Zone zone, String msg) {
        parent.print(zone, '[${DateTime.now()}] $msg');
      },
    ),
  ).run(() {
    print('This will be logged with timestamp');
  });
}
```

---

## Use Zones for Error Boundaries

```dart
// Good: Error boundary with zone
void withErrorBoundary(Function fn) {
  Zone.current.fork(
    specification: ZoneSpecification(
      handleUncaughtError: (zone, parent, error, stack) {
        // Handle error gracefully
        print('Error captured: $error');
        // Report to error service
      },
    ),
  ).run(fn);
}
```

---

# Common Mistakes

## Modifying Zone in Existing Code

Wrong:
```dart
void someFunction() {
  print('Hello'); // Can't modify this without zone
}
```

Correct:
```dart
void someFunction() {
  Zone.current.fork(
    specification: ZoneSpecification(
      print: (zone, parent, msg) => parent.print(zone, 'MODIFIED: $msg'),
    ),
  ).run(() {
    print('Hello'); // This will be modified
  });
}
```

---

## Forgetting to Call Parent

Wrong:
```dart
Zone.current.fork(
  specification: ZoneSpecification(
    print: (zone, parent, msg) {
      // Forgot to call parent.print
      // The original print won't execute
    },
  ),
);
```

Correct:
```dart
Zone.current.fork(
  specification: ZoneSpecification(
    print: (zone, parent, msg) {
      // Call parent.print to execute original behavior
      parent.print(zone, msg);
      // Add custom behavior
    },
  ),
);
```

---

# Summary

Zones provide powerful execution context management, enabling error handling, logging, monitoring, and context propagation across asynchronous operations.

---

# Next Steps

Now that you understand zones, continue to:

- [Error Handling](error-handling.md)
- [Core Libraries](core-libraries.md)
- [Concurrency](concurrency.md)
- [Testing](testing.md)

---

# Did You Know?

- Zones were introduced in Dart 1.0
- Each isolate has its own zone hierarchy
- Zones can be nested
- Zone values are inherited
- Zones can intercept most async operations
- Zones are used in Flutter for error handling
- The root zone is created by the Dart runtime

---
