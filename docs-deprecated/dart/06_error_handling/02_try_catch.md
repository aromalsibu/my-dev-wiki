# try-catch

Understand how to catch and handle exceptions using try-catch blocks in Dart.

---

# What is it?

`try-catch` is a fundamental error handling construct that allows you to catch and handle exceptions that occur during program execution. The `try` block contains code that might throw an exception, and the `catch` block handles the exception if one is thrown.

---

# Why does it exist?

`try-catch` exists to:

- Prevent program crashes from exceptions
- Handle errors gracefully
- Provide alternative execution paths
- Separate error handling from normal code
- Enable recovery from exceptional conditions
- Log errors for debugging

---

# Basic try-catch

## Simple try-catch

> **The `try-catch` block** allows you to attempt risky code and handle any exceptions that occur.

```dart
void main() {
  print('Starting program');
  
  try {
    // Risky code that might throw an exception
    int result = 10 ~/ 0;
    print('Result: $result'); // This won't execute
  } catch (e) {
    // Handle the exception
    print('Error caught: $e');
  }
  
  print('Program continues');
}

// Output:
// Starting program
// Error caught: IntegerDivisionByZeroException
// Program continues
```

> **What's happening here?**
> - `try` block contains code that might throw
> - When exception is thrown, execution jumps to `catch`
> - The rest of the `try` block is skipped
> - `catch` block receives the exception object
> - Program continues after the try-catch

---

## try-catch with Stack Trace

> **You can capture** the stack trace along with the exception for debugging purposes.

```dart
void riskyFunction() {
  throw Exception('Something went wrong!');
}

void main() {
  try {
    riskyFunction();
  } catch (e, stackTrace) {
    print('Error: $e');
    print('Stack trace:');
    print(stackTrace);
  }
}
```

---

# Catching Specific Exceptions

## Using `on` with try-catch

> **The `on` keyword** allows you to catch specific exception types, enabling different handling for different errors.

```dart
void processData(String input) {
  if (input.isEmpty) {
    throw FormatException('Input cannot be empty');
  }
  if (input == 'error') {
    throw Exception('Processing error');
  }
  int value = int.parse(input);
  print('Processed: $value');
}

void main() {
  // Example 1: Catch specific exception type
  try {
    processData('abc');
  } on FormatException catch (e) {
    print('Format error: $e');
  } catch (e) {
    print('Other error: $e');
  }
  
  // Example 2: Multiple specific catches
  try {
    processData('');
  } on FormatException catch (e) {
    print('Format exception: $e');
  } on Exception catch (e) {
    print('Generic exception: $e');
  } catch (e) {
    print('Unknown error: $e');
  }
  
  // Example 3: catch all with specific first
  try {
    processData('error');
  } on FormatException catch (e) {
    print('Format error: $e');
  } catch (e) {
    print('Caught any error: $e');
  }
}

// Output:
// Format error: FormatException: Input cannot be empty
// Format exception: FormatException: Input cannot be empty
// Caught any error: Exception: Processing error
```

> **What's happening here?**
> - `on` catches specific exception types
> - Multiple `on` blocks can be chained
> - `catch` catches any exception not caught by `on`
> - Order matters: specific types first, generic last
> - This enables different handling for different errors

---

# try-catch with Custom Exceptions

## Handling Custom Exceptions

> **Custom exceptions** can be caught and handled like built-in exceptions.

```dart
class NetworkException implements Exception {
  final int statusCode;
  final String message;
  
  NetworkException(this.statusCode, this.message);
  
  @override
  String toString() => 'NetworkException: $statusCode - $message';
}

class ValidationException implements Exception {
  final String field;
  final String message;
  
  ValidationException(this.field, this.message);
  
  @override
  String toString() => 'ValidationException: $field - $message';
}

void fetchData(String url) {
  if (url.isEmpty) {
    throw ValidationException('url', 'URL cannot be empty');
  }
  if (url == 'error') {
    throw NetworkException(500, 'Server error');
  }
  print('Fetching data from: $url');
}

void main() {
  // Handle custom exceptions
  try {
    fetchData('');
  } on ValidationException catch (e) {
    print('Validation error on ${e.field}: ${e.message}');
  } on NetworkException catch (e) {
    print('Network error: ${e.statusCode} - ${e.message}');
  } catch (e) {
    print('Other error: $e');
  }
  
  try {
    fetchData('error');
  } on ValidationException catch (e) {
    print('Validation error: $e');
  } on NetworkException catch (e) {
    print('Network error: ${e.statusCode} - ${e.message}');
  }
}

// Output:
// Validation error on url: URL cannot be empty
// Network error: 500 - Server error
```

> **Key insights:**
> - Custom exceptions are caught with `on`
> - Each custom exception can have its own handler
> - You can access custom properties in the catch block
> - This enables precise error handling

---

# try-catch in Async Code

## Handling Async Exceptions

> **`try-catch` works** with `async/await` just like synchronous code.

```dart
import 'dart:async';

Future<String> fetchUserData() async {
  await Future.delayed(Duration(seconds: 1));
  throw FormatException('Invalid user data format');
}

Future<int> calculateSomething() async {
  await Future.delayed(Duration(seconds: 1));
  return 42;
}

void main() async {
  // Example 1: try-catch with async/await
  try {
    var data = await fetchUserData();
    print('User data: $data');
  } on FormatException catch (e) {
    print('Format error: $e');
  } catch (e) {
    print('Other error: $e');
  }
  
  // Example 2: try-catch with multiple awaits
  try {
    var data = await fetchUserData();
    var result = await calculateSomething();
    print('Data: $data, Result: $result');
  } catch (e) {
    print('Error in async operation: $e');
  }
  
  // Example 3: with finally (covered in next topic)
  try {
    var data = await fetchUserData();
    print(data);
  } catch (e) {
    print('Error: $e');
  }
}
```

---

# try-catch in Streams

## Handling Stream Errors

> **`try-catch` can be used** with `await for` to handle stream errors.

```dart
import 'dart:async';

Stream<int> errorProneStream() async* {
  for (var i = 1; i <= 5; i++) {
    if (i == 3) {
      throw Exception('Error at value 3');
    }
    await Future.delayed(Duration(milliseconds: 500));
    yield i;
  }
}

void main() async {
  print('Processing stream:');
  
  // try-catch with await for
  try {
    await for (var value in errorProneStream()) {
      print('Value: $value');
    }
  } catch (e) {
    print('Stream error caught: $e');
  }
  
  // Alternative: listen with onError
  errorProneStream().listen(
    (value) => print('Value: $value'),
    onError: (error) => print('Error: $error'),
  );
}

// Output:
// Processing stream:
// Value: 1
// Value: 2
// Stream error caught: Exception: Error at value 3
// Value: 1
// Value: 2
// Error: Exception: Error at value 3
// Value: 4
// Value: 5
```

---

# try-catch with Finally

> **`finally`** is covered in detail in the next topic. Briefly, it executes regardless of whether an exception was caught.

```dart
void main() {
  try {
    int.parse('abc');
  } catch (e) {
    print('Error: $e');
  } finally {
    print('This always executes');
  }
}
```

---

# Best Practices

## Catch Specific Exceptions First

```dart
// Good: Specific then generic
try {
  riskyOperation();
} on FormatException catch (e) {
  // Handle format errors
} on Exception catch (e) {
  // Handle other exceptions
} catch (e) {
  // Handle everything else
}

// Bad: Generic first (specific won't be reached)
try {
  riskyOperation();
} catch (e) {
  // Handles everything
} on FormatException catch (e) {
  // Never reached!
}
```

---

## Don't Catch Everything

```dart
// Good: Catch specific exceptions
try {
  processFile();
} on FileNotFoundException catch (e) {
  print('File not found: $e');
} on PermissionException catch (e) {
  print('Permission denied: $e');
}

// Bad: Catching everything
try {
  processFile();
} catch (e) {
  // Catches all errors, including programming bugs
}
```

---

# Common Mistakes

## Empty Catch Block

Wrong:
```dart
try {
  riskyOperation();
} catch (e) {
  // Empty catch - silently swallows errors
}
```

Correct:
```dart
try {
  riskyOperation();
} catch (e) {
  print('Error: $e');
  // Log, notify, or handle
}
```

---

## Catching Errors as Exceptions

Wrong:
```dart
try {
  var list = [1, 2, 3];
  list[10]; // Throws RangeError (not Exception)
} on Exception catch (e) {
  // Won't catch RangeError
  print('Not caught: $e');
}
```

Correct:
```dart
try {
  var list = [1, 2, 3];
  list[10];
} catch (e) {
  // Catches all errors
  print('Caught: $e');
}
```

---

# Summary

`try-catch` is essential for robust error handling. Use `on` for specific exception types, `catch` for generic handling, and always provide meaningful error responses.

---

# Next Steps

Now that you understand try-catch, continue to:

- [finally](finally.md)
- [throw](throw.md)
- [Custom Exceptions](custom-exceptions.md)
- [Assertions](assertions.md)

---

# Did You Know?

- `on` catches specific exception types
- `catch` catches all exceptions
- `catch (e, stackTrace)` captures stack trace
- `rethrow` re-throws caught exceptions
- `try-catch` works with async/await
- Order of `on` blocks matters
- You can have multiple `on` blocks

---

