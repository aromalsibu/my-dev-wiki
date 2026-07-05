# throw

Understand how to throw exceptions in Dart.

---

# What is it?

The `throw` keyword is used to explicitly raise (throw) an exception in Dart. When an exception is thrown, the normal flow of execution is interrupted, and the runtime looks for the nearest `catch` block to handle it.

---

# Why does it exist?

`throw` exists to:

- Signal exceptional conditions
- Indicate invalid operations
- Validate inputs and states
- Control program flow
- Provide error context
- Enable structured error handling

---

# Basic throw

## Throwing Exceptions

> **The `throw` keyword** raises an exception that can be caught and handled.

```dart
void validateAge(int age) {
  if (age < 0) {
    throw Exception('Age cannot be negative');
  }
  if (age < 18) {
    throw Exception('Must be at least 18 years old');
  }
  print('Age is valid: $age');
}

void main() {
  try {
    validateAge(-5);
  } catch (e) {
    print('Error: $e');
  }
  
  try {
    validateAge(15);
  } catch (e) {
    print('Error: $e');
  }
  
  // This works fine
  validateAge(25);
}

// Output:
// Error: Exception: Age cannot be negative
// Error: Exception: Must be at least 18 years old
// Age is valid: 25
```

> **What's happening here?**
> - `throw` creates and raises an exception
> - Program flow jumps to the nearest `catch`
> - Different conditions throw different messages
> - Valid inputs don't throw exceptions

---

# Throwing Different Types

## Built-in Exception Types

> **Dart provides** various built-in exception types for common scenarios.

```dart
void main() {
  // Throw FormatException
  try {
    throw FormatException('Invalid number format');
  } catch (e) {
    print('FormatException: $e');
  }
  
  // Throw ArgumentError
  try {
    throw ArgumentError('Invalid argument provided');
  } catch (e) {
    print('ArgumentError: $e');
  }
  
  // Throw RangeError
  try {
    throw RangeError('Value out of range');
  } catch (e) {
    print('RangeError: $e');
  }
  
  // Throw StateError
  try {
    throw StateError('Invalid state');
  } catch (e) {
    print('StateError: $e');
  }
  
  // Throw TypeError
  try {
    throw TypeError();
  } catch (e) {
    print('TypeError: $e');
  }
}
```

---

# Throwing with Stack Trace

## Capturing Stack Traces

> **You can provide** a stack trace when throwing exceptions.

```dart
void functionA() {
  functionB();
}

void functionB() {
  functionC();
}

void functionC() {
  // Throw with stack trace
  throw Exception('Error in functionC');
}

void main() {
  try {
    functionA();
  } catch (e, stackTrace) {
    print('Error: $e');
    print('Stack trace:');
    print(stackTrace);
  }
}
```

---

# Conditional Throws

## Throwing Based on Conditions

> **You can use** conditionals to decide when to throw exceptions.

```dart
class BankAccount {
  double _balance = 0;
  
  void deposit(double amount) {
    if (amount <= 0) {
      throw ArgumentError('Deposit amount must be positive');
    }
    _balance += amount;
    print('Deposited: \$${amount.toStringAsFixed(2)}');
  }
  
  void withdraw(double amount) {
    if (amount <= 0) {
      throw ArgumentError('Withdrawal amount must be positive');
    }
    if (amount > _balance) {
      throw Exception('Insufficient funds');
    }
    _balance -= amount;
    print('Withdrew: \$${amount.toStringAsFixed(2)}');
  }
  
  void transfer(BankAccount to, double amount) {
    if (to == this) {
      throw Exception('Cannot transfer to self');
    }
    withdraw(amount);
    to.deposit(amount);
    print('Transferred: \$${amount.toStringAsFixed(2)}');
  }
  
  double get balance => _balance;
}

void main() {
  var account = BankAccount();
  
  try {
    account.deposit(-50);
  } catch (e) {
    print('Error: $e');
  }
  
  try {
    account.withdraw(100);
  } catch (e) {
    print('Error: $e');
  }
  
  // Valid operations
  account.deposit(100);
  account.withdraw(50);
  print('Balance: \$${account.balance.toStringAsFixed(2)}');
}
```

---

# Throwing in Async Functions

## Async throw

> **You can throw** exceptions in async functions just like sync functions.

```dart
import 'dart:async';

Future<String> fetchData(String url) async {
  if (url.isEmpty) {
    throw ArgumentError('URL cannot be empty');
  }
  
  if (!url.startsWith('http')) {
    throw FormatException('Invalid URL format');
  }
  
  await Future.delayed(Duration(seconds: 1));
  
  // Simulate network error
  if (url == 'error') {
    throw Exception('Network error');
  }
  
  return 'Data from $url';
}

void main() async {
  try {
    var data = await fetchData('');
  } catch (e) {
    print('Error 1: $e');
  }
  
  try {
    var data = await fetchData('error');
  } catch (e) {
    print('Error 2: $e');
  }
  
  try {
    var data = await fetchData('https://example.com');
    print('Data: $data');
  } catch (e) {
    print('Error 3: $e');
  }
}

// Output:
// Error 1: ArgumentError: URL cannot be empty
// Error 2: Exception: Network error
// Data: Data from https://example.com
```

> **Key insights:**
> - `throw` works in async functions
> - The exception is caught by `try-catch`
> - The function returns a `Future` that completes with an error

---

# Throwing in Streams

## Error in Streams

> **Streams can throw** errors using `throw` in `async*` generators.

```dart
import 'dart:async';

Stream<int> numberStream(int max) async* {
  for (var i = 1; i <= max; i++) {
    if (i == 3) {
      throw Exception('Error at value 3');
    }
    await Future.delayed(Duration(milliseconds: 500));
    yield i;
  }
}

void main() async {
  try {
    await for (var value in numberStream(5)) {
      print('Value: $value');
    }
  } catch (e) {
    print('Stream error: $e');
  }
  
  // With listen
  numberStream(5).listen(
    (value) => print('Value: $value'),
    onError: (error) => print('Error: $error'),
  );
}

// Output:
// Value: 1
// Value: 2
// Stream error: Exception: Error at value 3
// Value: 1
// Value: 2
// Error: Exception: Error at value 3
// Value: 4
// Value: 5
```

---

# Best Practices

## Throw Descriptive Exceptions

```dart
// Good: Descriptive exception
throw ArgumentError('Username cannot be empty');

// Better: Custom exception with context
throw ValidationException(
  field: 'username',
  message: 'Username cannot be empty',
  value: '',
);

// Bad: Generic exception
throw Exception('Error');
```

---

## Use Specific Exception Types

```dart
// Good: Specific type
throw ArgumentError('Invalid argument');

// Good: Custom type
throw NetworkException(statusCode: 404, message: 'Not found');

// Bad: Generic
throw Exception('Error occurred');
```

---

# Common Mistakes

## Throwing Non-Exception Objects

Wrong:
```dart
throw 'Error message'; // Throwing a string
```

Correct:
```dart
throw Exception('Error message'); // Throwing an Exception
```

---

## Throwing Inside Finally

Wrong:
```dart
try {
  riskyOp();
} finally {
  throw Exception('Error in finally'); // May mask original error
}
```

Correct:
```dart
try {
  riskyOp();
} catch (e) {
  // Handle original error
  print('Original error: $e');
} finally {
  // Cleanup only, don't throw
  cleanup();
}
```

---

# Summary

`throw` is used to raise exceptions, signaling that something unexpected happened. Use specific exception types and provide descriptive messages.

---

# Next Steps

Now that you understand throw, continue to:

- [Custom Exceptions](custom-exceptions.md)
- [Assertions](assertions.md)
- [Libraries](libraries.md)
- [Imports](imports.md)

---

# Did You Know?

- `throw` works in both sync and async code
- You can throw any object that implements `Exception`
- `throw` can be used without `catch`
- `rethrow` re-throws a caught exception
- `throw` inside `finally` masks original errors
- Use `on` to catch specific exception types
- Stack traces show where the error occurred

---
