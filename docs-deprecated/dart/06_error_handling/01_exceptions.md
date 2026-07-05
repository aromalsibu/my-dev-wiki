# Exceptions

Understand what exceptions are and how they work in Dart.

---

# What is it?

Exceptions are events that occur during program execution that disrupt the normal flow of instructions. In Dart, exceptions are objects that represent errors or unexpected conditions. They can be thrown (raised) and caught (handled) to prevent program crashes and enable graceful error recovery.

---

# Why does it exist?

Exceptions exist to:

- Signal that something unexpected happened
- Prevent silent failures
- Enable structured error handling
- Separate error-handling code from normal code
- Provide debugging information (stack traces)
- Support graceful error recovery

---

# Exception Hierarchy

## The Exception Class Hierarchy

> **Dart has** a clear hierarchy of exception types. All exceptions implement the `Exception` interface, with `Error` and `Exception` as the two main branches.

```dart
// Exception hierarchy
// Object
// ├── Error (programmatic errors)
// │   ├── TypeError
// │   ├── AssertionError
// │   ├── RangeError
// │   ├── ArgumentError
// │   ├── UnsupportedError
// │   └── NoSuchMethodError
// │
// └── Exception (runtime exceptions)
//     ├── FormatException
//     ├── IntegerDivisionByZeroException
//     ├── TimeoutException
//     └── ...

void main() {
  // Error types - programmatic errors
  // These are usually bugs that should be fixed
  try {
    var list = [1, 2, 3];
    var value = list[10]; // Throws RangeError
  } catch (e) {
    print('Error: $e (${e.runtimeType})');
  }
  
  // Exception types - runtime exceptions
  // These can happen during normal operation
  try {
    int.parse('abc'); // Throws FormatException
  } catch (e) {
    print('Exception: $e (${e.runtimeType})');
  }
  
  // Both can be caught and handled
  try {
    throw Exception('Something went wrong');
  } catch (e) {
    print('Caught: $e');
  }
}

// Output:
// Error: RangeError (index): Invalid value: Not in range 0..2, inclusive: 10 (RangeError)
// Exception: FormatException: Invalid radix-10 number (at character 1)
// Caught: Exception: Something went wrong
```

> **What's happening here?**
> - `Error` types indicate programmatic errors (bugs)
> - `Exception` types indicate runtime conditions
> - Both can be caught and handled
> - The hierarchy helps with specific error handling
> - `Error` types are usually not meant to be caught

---

## Built-in Exception Types

> **Dart provides** several built-in exception types for common error scenarios.

```dart
void main() {
  // 1. FormatException - invalid format
  try {
    int.parse('123abc');
  } on FormatException catch (e) {
    print('FormatException: $e');
  }
  
  // 2. IntegerDivisionByZeroException - division by zero
  try {
    int result = 10 ~/ 0;
  } on IntegerDivisionByZeroException catch (e) {
    print('DivisionByZeroException: $e');
  }
  
  // 3. RangeError - index out of range
  try {
    var list = [1, 2, 3];
    var value = list[5];
  } on RangeError catch (e) {
    print('RangeError: $e');
  }
  
  // 4. ArgumentError - invalid argument
  try {
    void process(String? value) {
      if (value == null) {
        throw ArgumentError('Value cannot be null');
      }
    }
    process(null);
  } on ArgumentError catch (e) {
    print('ArgumentError: $e');
  }
  
  // 5. NoSuchMethodError - calling non-existent method
  try {
    var obj = Object();
    obj.nonExistentMethod(); // Throws NoSuchMethodError
  } on NoSuchMethodError catch (e) {
    print('NoSuchMethodError: $e');
  }
  
  // 6. TimeoutException - operation timed out
  try {
    throw TimeoutException('Operation timed out', Duration(seconds: 5));
  } on TimeoutException catch (e) {
    print('TimeoutException: $e');
  }
}
```

---

# Throwing Exceptions

## Using the `throw` Keyword

> **The `throw` keyword** is used to raise (throw) an exception. You can throw any object that implements `Exception`.

```dart
void validateAge(int age) {
  if (age < 0) {
    throw Exception('Age cannot be negative');
  }
  if (age < 18) {
    throw Exception('Must be at least 18 years old');
  }
}

void validateEmail(String email) {
  if (email.isEmpty) {
    throw FormatException('Email cannot be empty');
  }
  if (!email.contains('@')) {
    throw FormatException('Invalid email format');
  }
}

void main() {
  // Validate age
  try {
    validateAge(-5);
  } catch (e) {
    print('Age validation error: $e');
  }
  
  // Validate email
  try {
    validateEmail('invalid');
  } on FormatException catch (e) {
    print('Email validation error: $e');
  }
  
  // Throwing different types
  try {
    // Throw an Error (programmatic error)
    throw ArgumentError('Invalid argument provided');
  } on ArgumentError catch (e) {
    print('ArgumentError: $e');
  }
  
  // Throw a generic Exception
  try {
    throw Exception('Something unexpected happened');
  } catch (e) {
    print('Generic Exception: $e');
  }
}

// Output:
// Age validation error: Exception: Age cannot be negative
// Email validation error: FormatException: Invalid email format
// ArgumentError: Invalid argument: Invalid argument provided
// Generic Exception: Exception: Something unexpected happened
```

> **What's happening here?**
> - `throw` raises an exception
> - Different exception types can be thrown
> - `throw` can be used anywhere, not just in functions
> - The exception propagates up until caught
> - Uncaught exceptions crash the program

---

# Rethrowing Exceptions

## Using `rethrow` to Propagate

> **The `rethrow` keyword** allows you to catch an exception and then rethrow it to be handled by the caller.

```dart
class DataService {
  Future<String> fetchData() async {
    try {
      // Simulate network request
      throw Exception('Network error occurred');
    } catch (e) {
      print('DataService: Caught error, adding context');
      // Add context and rethrow
      rethrow; // Propagates the original error
    }
  }
}

void main() async {
  var service = DataService();
  
  try {
    await service.fetchData();
  } catch (e) {
    print('Main: Caught error: $e');
    // Handle the error at the top level
  }
}

// Output:
// DataService: Caught error, adding context
// Main: Caught error: Exception: Network error occurred
```

> **Key insights:**
> - `rethrow` propagates the original exception
> - You can add context before rethrowing
> - The original stack trace is preserved
> - Useful for adding logging or metadata
> - The caller still handles the error

---

# Exception vs Error

## Understanding the Difference

> **In Dart**, `Exception` and `Error` serve different purposes. `Error` is for programmatic errors (bugs), while `Exception` is for runtime conditions.

```dart
void main() {
  // Exception - runtime conditions (should be handled)
  try {
    throw Exception('Network connection lost');
  } on Exception catch (e) {
    print('Handling Exception: $e');
    // Retry logic
  }
  
  // Error - programmatic errors (should be fixed, not caught)
  try {
    throw TypeError();
  } on Error catch (e) {
    // Usually don't catch Error
    print('Caught Error: $e - but this should be fixed');
    // You can catch it, but better to fix the code
  }
  
  // Best practice: Use Exception for expected errors
  try {
    if (someCondition) {
      throw Exception('Expected error scenario');
    }
  } catch (e) {
    // Handle expected errors
  }
}
```

---

# Stack Traces

## Understanding Stack Traces

> **Stack traces** provide information about where an exception occurred, including the call stack.

```dart
void functionA() {
  functionB();
}

void functionB() {
  functionC();
}

void functionC() {
  throw Exception('Error in functionC');
}

void main() {
  try {
    functionA();
  } catch (e, stackTrace) {
    print('Error: $e');
    print('Stack trace:');
    print(stackTrace);
    print('\n--- Formatted ---');
    // Print formatted stack trace
    print(stackTrace.toString());
  }
}
```

---

# Best Practices

## Use Specific Exception Types

```dart
// Good: Specific exceptions
class NetworkException implements Exception {
  final int statusCode;
  NetworkException(this.statusCode);
}

class TimeoutException implements Exception {
  final int timeoutSeconds;
  TimeoutException(this.timeoutSeconds);
}

// Bad: Generic exceptions
throw Exception('Network error'); // Too generic
```

---

## Provide Context in Exceptions

```dart
// Good: Rich context
throw NetworkException(
  404,
  message: 'User not found',
  url: '/api/users/123',
);

// Bad: No context
throw Exception('Error');
```

---

# Common Mistakes

## Catching Everything

Wrong:
```dart
try {
  riskyOperation();
} catch (e) {
  // Catches everything, including Errors
  // May hide bugs
}
```

Correct:
```dart
try {
  riskyOperation();
} on Exception catch (e) {
  // Only catches Exceptions
  // Errors will still crash
}
```

---

## Swallowing Exceptions

Wrong:
```dart
try {
  riskyOperation();
} catch (e) {
  // Silent fail - bad!
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

# Summary

Exceptions are Dart's way of handling errors. Use built-in exception types, create custom exceptions for specific scenarios, and always handle exceptions appropriately.

---

# Next Steps

Now that you understand exceptions, continue to:

- [try-catch](try-catch.md)
- [finally](finally.md)
- [throw](throw.md)
- [Custom Exceptions](custom-exceptions.md)
- [Assertions](assertions.md)

---

# Did You Know?

- Exceptions can be any object that implements `Exception`
- `Error` types are for programmatic errors
- `rethrow` preserves the original stack trace
- Uncaught exceptions crash the program
- Stack traces show where the error occurred
- Custom exceptions provide better error context
- `on` can catch specific exception types

---

