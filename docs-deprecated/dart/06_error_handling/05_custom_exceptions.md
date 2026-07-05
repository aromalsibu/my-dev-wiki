# Custom Exceptions

Understand how to create and use custom exception types in Dart.

---

# What is it?

Custom exceptions are user-defined exception classes that extend or implement the `Exception` class. They allow you to create domain-specific error types with rich context, enabling precise error handling and better debugging.

---

# Why does it exist?

Custom exceptions exist to:

- Create domain-specific error types
- Provide rich error context
- Enable precise error handling
- Improve code readability
- Add custom behavior to errors
- Simplify debugging with structured error data

---

# Basic Custom Exceptions

## Creating Custom Exceptions

> **Custom exceptions** are created by extending or implementing the `Exception` class. They can include additional fields and methods.

```dart
// Simple custom exception
class MyException implements Exception {
  final String message;
  
  MyException(this.message);
  
  @override
  String toString() => 'MyException: $message';
}

// Exception with multiple fields
class ValidationException implements Exception {
  final String field;
  final String message;
  final dynamic value;
  
  ValidationException({
    required this.field,
    required this.message,
    this.value,
  });
  
  @override
  String toString() {
    if (value != null) {
      return 'ValidationException: $field - $message (value: $value)';
    }
    return 'ValidationException: $field - $message';
  }
}

// Usage
void validateEmail(String email) {
  if (email.isEmpty) {
    throw MyException('Email cannot be empty');
  }
  if (!email.contains('@')) {
    throw ValidationException(
      field: 'email',
      message: 'Invalid email format',
      value: email,
    );
  }
}

void main() {
  try {
    validateEmail('invalid');
  } catch (e) {
    print('Caught: $e');
  }
}

// Output:
// Caught: ValidationException: email - Invalid email format (value: invalid)
```

> **What's happening here?**
> - Custom exceptions implement `Exception`
> - They can have custom fields and methods
> - `toString()` provides meaningful error messages
> - Each exception type can carry specific data

---

## Custom Exceptions with Methods

> **Custom exceptions** can include methods and behaviors.

```dart
class NetworkException implements Exception {
  final int statusCode;
  final String message;
  final String? url;
  final DateTime timestamp;
  
  NetworkException({
    required this.statusCode,
    required this.message,
    this.url,
  }) : timestamp = DateTime.now();
  
  bool get isClientError => statusCode >= 400 && statusCode < 500;
  bool get isServerError => statusCode >= 500 && statusCode < 600;
  bool get isNotFound => statusCode == 404;
  bool get isTimeout => statusCode == 408;
  
  String get userFriendlyMessage {
    if (isNotFound) return 'The requested resource was not found';
    if (isTimeout) return 'The request timed out. Please try again';
    if (isClientError) return 'An error occurred with your request';
    if (isServerError) return 'The server encountered an error';
    return 'A network error occurred';
  }
  
  @override
  String toString() {
    var base = 'NetworkException: $statusCode - $message';
    if (url != null) base += ' (URL: $url)';
    base += ' at $timestamp';
    return base;
  }
}

void fetchData(String url) {
  if (url.contains('404')) {
    throw NetworkException(
      statusCode: 404,
      message: 'Resource not found',
      url: url,
    );
  }
  if (url.contains('500')) {
    throw NetworkException(
      statusCode: 500,
      message: 'Internal server error',
      url: url,
    );
  }
  print('Data fetched from $url');
}

void main() {
  try {
    fetchData('https://api.example.com/404');
  } on NetworkException catch (e) {
    print('Error: ${e.userFriendlyMessage}');
    print('Status: ${e.statusCode}');
    print('Is client error: ${e.isClientError}');
    print('Timestamp: ${e.timestamp}');
  }
}

// Output:
// Error: The requested resource was not found
// Status: 404
// Is client error: true
// Timestamp: 2024-01-01 12:00:00.000
```

> **Key insights:**
> - Custom exceptions can have methods
> - They can provide user-friendly messages
> - You can add computed properties
> - Rich context helps with debugging
> - Different handling based on error type

---

# Custom Exception Hierarchy

## Creating Exception Hierarchies

> **You can create** a hierarchy of custom exceptions for different error types.

```dart
// Base exception
class AppException implements Exception {
  final String message;
  final StackTrace? stackTrace;
  
  AppException(this.message, [this.stackTrace]);
  
  @override
  String toString() => 'AppException: $message';
}

// Network-related exceptions
class NetworkException extends AppException {
  NetworkException(String message) : super(message);
}

class TimeoutException extends NetworkException {
  final int timeoutSeconds;
  
  TimeoutException(this.timeoutSeconds)
      : super('Connection timed out after ${timeoutSeconds}s');
}

class ConnectionException extends NetworkException {
  final String host;
  
  ConnectionException(this.host)
      : super('Cannot connect to $host');
}

// Data-related exceptions
class DataException extends AppException {
  final String data;
  
  DataException(String message, this.data) : super(message);
}

class ValidationException extends DataException {
  final String field;
  
  ValidationException(String message, this.field, String data)
      : super(message, data);
}

class ParsingException extends DataException {
  ParsingException(String message, String data)
      : super(message, data);
}

// Usage
void fetchUserData(String userId) {
  if (userId.isEmpty) {
    throw ValidationException(
      'User ID cannot be empty',
      'userId',
      userId,
    );
  }
  if (userId == 'timeout') {
    throw TimeoutException(30);
  }
  if (userId == 'error') {
    throw ConnectionException('api.example.com');
  }
  print('Fetched user: $userId');
}

void main() {
  try {
    fetchUserData('');
  } on ValidationException catch (e) {
    print('Validation error on ${e.field}: ${e.message}');
  } on TimeoutException catch (e) {
    print('Timeout after ${e.timeoutSeconds}s');
  } on ConnectionException catch (e) {
    print('Cannot connect to ${e.host}');
  } on AppException catch (e) {
    print('App error: ${e.message}');
  }
  
  try {
    fetchUserData('timeout');
  } on NetworkException catch (e) {
    print('Network error: ${e.message}');
  }
}
```

---

# Real-World Example

## Complete Custom Exception System

> **A complete exception system** for a real-world application.

```dart
// Base exception with formatting
abstract class AppException implements Exception {
  final String code;
  final String message;
  final String? details;
  final DateTime timestamp;
  
  AppException({
    required this.code,
    required this.message,
    this.details,
  }) : timestamp = DateTime.now();
  
  String get formattedMessage {
    var base = '[$code] $message';
    if (details != null) base += '\nDetails: $details';
    base += '\nTimestamp: $timestamp';
    return base;
  }
  
  @override
  String toString() => formattedMessage;
}

// Authentication exceptions
class AuthException extends AppException {
  AuthException({required String code, required String message, String? details})
      : super(code: 'AUTH_$code', message: message, details: details);
}

class InvalidCredentialsException extends AuthException {
  InvalidCredentialsException({String? details})
      : super(
          code: 'INVALID_CREDENTIALS',
          message: 'Invalid username or password',
          details: details,
        );
}

class SessionExpiredException extends AuthException {
  SessionExpiredException()
      : super(
          code: 'SESSION_EXPIRED',
          message: 'Your session has expired. Please log in again.',
        );
}

// Data exceptions
class DataException extends AppException {
  DataException({required String code, required String message, String? details})
      : super(code: 'DATA_$code', message: message, details: details);
}

class NotFoundException extends DataException {
  final String resource;
  final String id;
  
  NotFoundException(this.resource, this.id)
      : super(
          code: 'NOT_FOUND',
          message: '$resource not found',
          details: 'Resource: $resource, ID: $id',
        );
}

class ConflictException extends DataException {
  final String resource;
  final String field;
  final String value;
  
  ConflictException(this.resource, this.field, this.value)
      : super(
          code: 'CONFLICT',
          message: '$resource already exists',
          details: 'Field: $field, Value: $value',
        );
}

// Service layer
class UserService {
  final Map<String, String> _users = {
    'alice': 'alice@example.com',
    'bob': 'bob@example.com',
  };
  
  String getUser(String username, String token) {
    // Authentication
    if (token.isEmpty) {
      throw SessionExpiredException();
    }
    if (token != 'valid_token') {
      throw InvalidCredentialsException(details: 'Invalid token provided');
    }
    
    // Data retrieval
    if (!_users.containsKey(username)) {
      throw NotFoundException('User', username);
    }
    
    return _users[username]!;
  }
  
  void createUser(String username, String email) {
    if (_users.containsKey(username)) {
      throw ConflictException('User', 'username', username);
    }
    _users[username] = email;
  }
}

void main() {
  var service = UserService();
  
  // Test: Session expired
  try {
    var user = service.getUser('alice', '');
  } on AuthException catch (e) {
    print('Auth error: ${e.message}');
  }
  
  // Test: Invalid credentials
  try {
    var user = service.getUser('alice', 'invalid');
  } on InvalidCredentialsException catch (e) {
    print('Invalid credentials: ${e.details}');
  }
  
  // Test: User not found
  try {
    var user = service.getUser('charlie', 'valid_token');
  } on NotFoundException catch (e) {
    print('Not found: ${e.resource} ${e.id}');
  }
  
  // Test: User conflict
  try {
    service.createUser('alice', 'alice@example.com');
  } on ConflictException catch (e) {
    print('Conflict: ${e.field} ${e.value}');
  }
}

// Output:
// Auth error: Your session has expired. Please log in again.
// Invalid credentials: Invalid token provided
// Not found: User charlie
// Conflict: username alice
```

---

# Best Practices

## Use Custom Exceptions for Specific Errors

```dart
// Good: Specific custom exceptions
class NetworkError extends Exception {
  final int statusCode;
  NetworkError(this.statusCode);
}

// Bad: Generic exception
throw Exception('Network error');
```

---

## Provide Rich Context

```dart
// Good: Rich context in exception
class DataNotFoundException extends Exception {
  final String resource;
  final String id;
  final String? additionalInfo;
  
  DataNotFoundException(this.resource, this.id, {this.additionalInfo});
}

// Bad: No context
throw Exception('Data not found');
```

---

# Common Mistakes

## Not Implementing toString()

Wrong:
```dart
class CustomException implements Exception {}
```

Correct:
```dart
class CustomException implements Exception {
  final String message;
  CustomException(this.message);
  
  @override
  String toString() => 'CustomException: $message';
}
```

---

## Throwing Generic Exceptions

Wrong:
```dart
throw Exception('Error occurred'); // Too generic
```

Correct:
```dart
throw ValidationException(
  field: 'email',
  message: 'Invalid email format',
);
```

---

# Summary

Custom exceptions provide rich error context and enable precise error handling. Use them to create domain-specific error types with meaningful messages and data.

---

# Next Steps

Now that you understand custom exceptions, continue to:

- [Assertions](assertions.md)
- [Libraries](libraries.md)
- [Imports](imports.md)
- [Exports](exports.md)

---

# Did You Know?

- Custom exceptions can have methods
- Exception hierarchies enable grouped handling
- `toString()` should be overridden
- Custom exceptions can carry context data
- Use `on` to catch specific custom exceptions
- Rich exceptions improve debugging
- Custom exceptions make code self-documenting

---
