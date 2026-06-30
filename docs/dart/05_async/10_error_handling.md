# Error Handling

Understand how to handle errors effectively in asynchronous and synchronous Dart code.

---

# What is it?

Error handling is the process of anticipating, detecting, and responding to errors that occur during program execution. In Dart, errors can be synchronous (thrown directly) or asynchronous (thrown in Futures, Streams, or Isolates). Proper error handling prevents crashes and ensures graceful degradation.

---

# Why does it exist?

Error handling exists to:

- Prevent application crashes
- Provide graceful failure recovery
- Log errors for debugging
- Display user-friendly error messages
- Maintain application stability
- Handle exceptional conditions properly

---

# Synchronous Error Handling

## Try-Catch-Finally

> **Synchronous error handling** uses `try-catch` blocks to catch and handle exceptions that occur during execution.

```dart
void main() {
  // Basic try-catch
  try {
    int result = 10 ~/ 0; // This throws an exception
    print('Result: $result');
  } catch (e) {
    print('Error caught: $e');
  }
  
  // Try-catch with specific exception type
  try {
    var value = int.parse('abc');
    print('Value: $value');
  } on FormatException catch (e) {
    print('Format error: $e');
  } catch (e) {
    print('Other error: $e');
  }
  
  // Try-catch-finally
  try {
    var file = File('data.txt');
    var content = file.readAsStringSync();
    print(content);
  } catch (e) {
    print('Error reading file: $e');
  } finally {
    print('Cleanup executed');
  }
}

// Output:
// Error caught: IntegerDivisionByZeroException
// Format error: FormatException: Invalid radix-10 number
// Error reading file: FileSystemException: Cannot open file
// Cleanup executed
```

> **What's happening here?**
> - `try` block contains code that might throw an error
> - `catch` block handles any error thrown
> - `on` catches specific exception types
> - `finally` always executes, regardless of success or failure
> - This is the standard synchronous error handling pattern

---

## Throwing Exceptions

> **You can throw** exceptions using the `throw` keyword, with either built-in or custom exception types.

```dart
// Custom exception classes
class ValidationException implements Exception {
  final String message;
  final String? field;
  
  ValidationException(this.message, {this.field});
  
  @override
  String toString() {
    if (field != null) {
      return 'ValidationException: $field - $message';
    }
    return 'ValidationException: $message';
  }
}

class NetworkException implements Exception {
  final int statusCode;
  final String message;
  
  NetworkException(this.statusCode, this.message);
  
  @override
  String toString() {
    return 'NetworkException: $statusCode - $message';
  }
}

// Function that throws exceptions
void validateEmail(String email) {
  if (email.isEmpty) {
    throw ValidationException('Email cannot be empty', field: 'email');
  }
  if (!email.contains('@')) {
    throw ValidationException('Invalid email format', field: 'email');
  }
}

void fetchData(String url) {
  if (url.isEmpty) {
    throw NetworkException(400, 'URL cannot be empty');
  }
  if (!url.startsWith('http')) {
    throw NetworkException(400, 'Invalid URL format');
  }
  // Simulate network error
  throw NetworkException(500, 'Server error');
}

void main() {
  // Handle custom exceptions
  try {
    validateEmail('invalid');
  } on ValidationException catch (e) {
    print('Validation error: ${e.field} - ${e.message}');
  }
  
  try {
    fetchData('');
  } on NetworkException catch (e) {
    print('Network error: ${e.statusCode} - ${e.message}');
  } catch (e) {
    print('Other error: $e');
  }
}

// Output:
// Validation error: email - Invalid email format
// Network error: 400 - URL cannot be empty
```

> **Key insights:**
> - Custom exceptions provide richer error information
> - `throw` throws an exception
> - `on` catches specific exception types
> - Use `rethrow` to rethrow the exception to the caller
> - Custom exceptions make error handling more precise

---

# Asynchronous Error Handling

## Future Error Handling

> **Future errors** can be handled with `catchError()` or using `try-catch` with `await`.

```dart
import 'dart:async';

// Function that returns a Future with potential error
Future<String> fetchData() async {
  await Future.delayed(Duration(seconds: 1));
  throw Exception('Network request failed');
}

void main() {
  // Method 1: catchError
  fetchData()
    .then((data) {
      print('Success: $data');
    })
    .catchError((error) {
      print('Error caught: $error');
    });
  
  // Method 2: onError with then
  fetchData()
    .then((data) {
      print('Success: $data');
    }, onError: (error) {
      print('Error caught in onError: $error');
    });
  
  // Method 3: try-catch with await
  Future<void> handleWithTryCatch() async {
    try {
      var data = await fetchData();
      print('Success: $data');
    } catch (e) {
      print('Error caught in try-catch: $e');
    } finally {
      print('Cleanup done');
    }
  }
  
  handleWithTryCatch();
}

// Output:
// Error caught: Exception: Network request failed
// Error caught in onError: Exception: Network request failed
// Error caught in try-catch: Exception: Network request failed
// Cleanup done
```

> **What's happening here?**
> - `catchError()` handles Future errors
> - `onError` in `then()` handles errors
> - `try-catch` with `await` works naturally
> - `finally` always executes
> - Multiple ways to handle Future errors

---

## Handling Specific Future Errors

> **You can handle** specific error types in Future chains.

```dart
import 'dart:async';

class NetworkError implements Exception {
  final int statusCode;
  NetworkError(this.statusCode);
  
  @override
  String toString() => 'NetworkError: $statusCode';
}

class ValidationError implements Exception {
  final String message;
  ValidationError(this.message);
  
  @override
  String toString() => 'ValidationError: $message';
}

Future<String> riskyOperation() async {
  await Future.delayed(Duration(seconds: 1));
  throw NetworkError(404);
}

Future<String> anotherRiskyOp() async {
  await Future.delayed(Duration(seconds: 1));
  throw ValidationError('Invalid input');
}

void main() {
  // Handle specific error types in Future chain
  riskyOperation()
    .then((data) {
      print('Success: $data');
    })
    .catchError((error) {
      if (error is NetworkError) {
        print('Network error: ${error.statusCode}');
        // Retry logic
      } else if (error is ValidationError) {
        print('Validation error: ${error.message}');
      } else {
        print('Unknown error: $error');
      }
    });
  
  // With onError type checking
  anotherRiskyOp()
    .then((data) {
      print('Success: $data');
    }, onError: (error) {
      if (error is ValidationError) {
        print('Caught validation error: ${error.message}');
      } else {
        print('Caught other error: $error');
      }
    });
}

// Output:
// Network error: 404
// Caught validation error: Invalid input
```

> **Key insights:**
> - `is` operator checks error types
> - Different handling for different error types
> - You can implement retry logic for network errors
> - Specific error types enable precise handling

---

# Stream Error Handling

## Handling Stream Errors

> **Stream errors** are handled with the `onError` parameter in `listen()` or with `try-catch` in `await for`.

```dart
import 'dart:async';

// Stream that emits values with potential error
Stream<int> getNumbers() async* {
  for (var i = 1; i <= 5; i++) {
    if (i == 3) {
      throw Exception('Error at 3!');
    }
    await Future.delayed(Duration(milliseconds: 500));
    yield i;
  }
}

void main() {
  // Method 1: listen with onError
  print('Method 1: listen with onError');
  getNumbers().listen(
    (data) => print('Data: $data'),
    onError: (error) => print('Error: $error'),
    onDone: () => print('Done'),
  );
  
  // Method 2: try-catch with await for
  Future<void> processStream() async {
    print('\nMethod 2: try-catch with await for');
    try {
      await for (var value in getNumbers()) {
        print('Value: $value');
      }
    } catch (e) {
      print('Error caught: $e');
    }
  }
  
  processStream();
}

// Output:
// Method 1: listen with onError
// Data: 1
// Data: 2
// Error: Exception: Error at 3!
// Data: 4
// Data: 5
// Done
//
// Method 2: try-catch with await for
// Value: 1
// Value: 2
// Error caught: Exception: Error at 3!
```

> **What's happening here?**
> - `onError` in `listen()` handles stream errors
> - `try-catch` works with `await for`
> - Error doesn't stop the stream (by default)
> - The stream continues after the error
> - `cancelOnError` can stop the stream on error

---

## Stream with Error Handling Options

> **Streams provide** options for error handling like `cancelOnError`.

```dart
import 'dart:async';

Stream<int> errorProneStream() async* {
  for (var i = 1; i <= 5; i++) {
    if (i == 3) {
      throw Exception('Error at 3');
    }
    await Future.delayed(Duration(milliseconds: 300));
    yield i;
  }
}

void main() {
  // With cancelOnError: false (default)
  print('=== cancelOnError: false ===');
  errorProneStream().listen(
    (data) => print('Data: $data'),
    onError: (error) => print('Error: $error'),
    onDone: () => print('Done'),
    cancelOnError: false, // Continue after error
  );
  
  Future.delayed(Duration(seconds: 3), () {
    print('\n=== cancelOnError: true ===');
    errorProneStream().listen(
      (data) => print('Data: $data'),
      onError: (error) => print('Error: $error'),
      onDone: () => print('Done'),
      cancelOnError: true, // Stop on first error
    );
  });
  
  Future.delayed(Duration(seconds: 6), () {
    print('\n=== Custom error handler with retry ===');
    int retryCount = 0;
    errorProneStream().listen(
      (data) => print('Data: $data'),
      onError: (error) {
        retryCount++;
        print('Error: $error (retry $retryCount)');
        if (retryCount < 3) {
          print('Retrying...');
        }
      },
      onDone: () => print('Done'),
    );
  });
}
```

---

# Error Handling in Isolates

## Isolate Error Handling

> **Isolate errors** can be handled through `ReceivePort` and `SendPort` communication.

```dart
import 'dart:isolate';

void workerIsolate(SendPort sendPort) {
  ReceivePort receivePort = ReceivePort();
  sendPort.send(receivePort.sendPort);
  
  receivePort.listen((message) {
    try {
      if (message is int) {
        if (message == 0) {
          throw Exception('Cannot process zero');
        }
        int result = 100 ~/ message;
        sendPort.send({'type': 'result', 'value': result});
      }
    } catch (e) {
      // Send error back to main
      sendPort.send({'type': 'error', 'error': e.toString()});
    }
  });
}

void main() async {
  ReceivePort mainPort = ReceivePort();
  Isolate isolate = await Isolate.spawn(workerIsolate, mainPort.sendPort);
  
  // Get worker's send port
  SendPort? workerPort;
  await for (var message in mainPort) {
    if (message is SendPort) {
      workerPort = message;
      break;
    }
  }
  
  // Send tasks
  List<int> tasks = [10, 5, 0, 2];
  for (var task in tasks) {
    workerPort!.send(task);
  }
  
  // Listen for results
  int received = 0;
  mainPort.listen((message) {
    if (message is Map<String, dynamic>) {
      if (message['type'] == 'result') {
        print('Result: ${message['value']}');
      } else if (message['type'] == 'error') {
        print('Error: ${message['error']}');
      }
      received++;
      if (received == tasks.length) {
        mainPort.close();
        isolate.kill();
      }
    }
  });
}

// Output:
// Result: 10
// Result: 20
// Error: Exception: Cannot process zero
// Result: 50
```

---

# Best Practices

## Always Handle Errors

```dart
// Good: Always handle errors
void fetchData() async {
  try {
    var data = await apiCall();
    displayData(data);
  } catch (e) {
    displayError(e.toString());
    logError(e);
  }
}

// Bad: Swallowing errors
void badFetchData() async {
  try {
    var data = await apiCall();
    displayData(data);
  } catch (e) {
    // Silent fail - bad practice!
  }
}
```

---

## Use Specific Exception Types

```dart
// Good: Specific exception handling
try {
  processData();
} on NetworkException catch (e) {
  // Handle network errors
} on ValidationException catch (e) {
  // Handle validation errors
} catch (e) {
  // Handle unknown errors
}

// Bad: Catching everything
try {
  processData();
} catch (e) {
  // All errors handled the same way
}
```

---

# Common Mistakes

## Swallowing Exceptions

Wrong:
```dart
try {
  riskyOperation();
} catch (e) {
  // No error handling
}
```

Correct:
```dart
try {
  riskyOperation();
} catch (e) {
  print('Error: $e');
  // Log the error
  // Notify the user
}
```

---

## Not Handling Async Errors

Wrong:
```dart
Future<void> asyncOperation() {
  return Future.delayed(Duration(seconds: 1), () {
    throw Exception('Error');
  });
}

void main() {
  asyncOperation(); // Unhandled error!
}
```

Correct:
```dart
void main() async {
  try {
    await asyncOperation();
  } catch (e) {
    print('Error: $e');
  }
}
```

---

# Summary

Error handling is essential for robust Dart applications. Use try-catch for synchronous errors, catchError or try-catch with async/await for Future errors, and onError or try-catch for Stream errors.

---

# Next Steps

Now that you understand error handling, you've completed the Asynchronous Programming section! Continue to:

- [Core Libraries](core-libraries.md)
- [Concurrency](concurrency.md)
- [Testing](testing.md)
- [Performance](performance.md)

---

# Did You Know?

- Uncaught errors can crash your app
- The `finally` block always executes
- `rethrow` propagates errors up
- Custom exceptions provide better context
- Async errors require async handling
- Zone errors can be handled globally
- Error handling is essential for production apps