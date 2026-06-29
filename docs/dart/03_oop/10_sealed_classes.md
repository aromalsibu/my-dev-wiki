# Sealed Classes

Understand how to use sealed classes for exhaustive pattern matching in Dart.

---

# What is it?

Sealed classes are a feature introduced in Dart 3 that allow you to define a restricted hierarchy of classes. A sealed class can only be extended or implemented within the same file, enabling exhaustive checking in switch statements and pattern matching.

---

# Why does it exist?

Sealed classes exist to:

- Enable exhaustive pattern matching
- Create closed type hierarchies
- Enforce complete handling of all cases
- Improve type safety
- Support algebraic data types
- Prevent unwanted subclassing

---

# Basic Sealed Classes

## Simple Sealed Class

```dart
// Sealed class (can only be extended in this file)
sealed class Result<T> {
  // Can have fields and methods
  const Result();
  
  bool get isSuccess => this is Success<T>;
  bool get isError => this is Error<T>;
  bool get isLoading => this is Loading<T>;
}

class Success<T> extends Result<T> {
  final T data;
  
  const Success(this.data);
  
  @override
  String toString() => 'Success($data)';
}

class Error<T> extends Result<T> {
  final String message;
  
  const Error(this.message);
  
  @override
  String toString() => 'Error($message)';
}

class Loading<T> extends Result<T> {
  const Loading();
  
  @override
  String toString() => 'Loading...';
}

// Usage - Exhaustive pattern matching
String handleResult<T>(Result<T> result) {
  return switch (result) {
    Success(data: var data) => 'Success: $data',
    Error(message: var msg) => 'Error: $msg',
    Loading() => 'Loading...',
  };
}

// Usage
var success = Success('Data loaded');
var error = Error('Something went wrong');
var loading = Loading();

print(handleResult(success)); // Success: Data loaded
print(handleResult(error)); // Error: Something went wrong
print(handleResult(loading)); // Loading...
```

---

## Sealed Class with Data

```dart
sealed class ApiResponse {
  const ApiResponse();
}

class SuccessResponse extends ApiResponse {
  final Map<String, dynamic> data;
  final int statusCode;
  
  const SuccessResponse(this.data, this.statusCode);
}

class ErrorResponse extends ApiResponse {
  final String message;
  final int statusCode;
  
  const ErrorResponse(this.message, this.statusCode);
}

class LoadingResponse extends ApiResponse {
  const LoadingResponse();
}

class TimeoutResponse extends ApiResponse {
  const TimeoutResponse();
}

// Handling all cases
void processApiResponse(ApiResponse response) {
  switch (response) {
    case SuccessResponse(data: var data, statusCode: var code):
      print('Success ($code): $data');
    case ErrorResponse(message: var msg, statusCode: var code):
      print('Error ($code): $msg');
    case LoadingResponse():
      print('Loading...');
    case TimeoutResponse():
      print('Request timed out');
  }
}

// Usage
var success = SuccessResponse({'id': 1, 'name': 'Alice'}, 200);
var error = ErrorResponse('Not found', 404);
var loading = LoadingResponse();
var timeout = TimeoutResponse();

processApiResponse(success);
// Success (200): {id: 1, name: Alice}

processApiResponse(error);
// Error (404): Not found

processApiResponse(loading);
// Loading...

processApiResponse(timeout);
// Request timed out
```

---

# Sealed Classes with Methods

## Including Behavior

```dart
sealed class Shape {
  const Shape();
  
  // Abstract methods
  double get area;
  double get perimeter;
  String get description;
  
  // Concrete method
  bool get isRegular => false;
  
  // Helper methods
  String get shortDescription => '$runtimeType: ${description}';
}

class Circle extends Shape {
  final double radius;
  
  const Circle(this.radius);
  
  @override
  double get area => math.pi * radius * radius;
  
  @override
  double get perimeter => 2 * math.pi * radius;
  
  @override
  String get description => 'Circle with radius $radius';
  
  @override
  bool get isRegular => true;
}

class Rectangle extends Shape {
  final double width;
  final double height;
  
  const Rectangle(this.width, this.height);
  
  @override
  double get area => width * height;
  
  @override
  double get perimeter => 2 * (width + height);
  
  @override
  String get description => 'Rectangle $width x $height';
  
  @override
  bool get isRegular => width == height;
}

class Square extends Shape {
  final double side;
  
  const Square(this.side);
  
  @override
  double get area => side * side;
  
  @override
  double get perimeter => 4 * side;
  
  @override
  String get description => 'Square with side $side';
  
  @override
  bool get isRegular => true;
}

// Usage
void processShape(Shape shape) {
  // Pattern matching with guards
  switch (shape) {
    case Circle(radius: var r) when r > 10:
      print('Large circle: $r');
    case Circle(radius: var r):
      print('Circle: $r');
    case Rectangle(width: var w, height: var h) when w == h:
      print('Rectangle (square): $w x $h');
    case Rectangle(width: var w, height: var h):
      print('Rectangle: $w x $h');
    case Square(side: var s):
      print('Square: $s');
  }
  
  // Using methods
  print('Area: ${shape.area}');
  print('Perimeter: ${shape.perimeter}');
  print('Regular: ${shape.isRegular}');
  print('---');
}

var shapes = [
  Circle(5),
  Rectangle(10, 20),
  Square(15),
  Circle(20),
];

for (var shape in shapes) {
  processShape(shape);
}
```

---

# Sealed Classes with Generics

## Generic Sealed Classes

```dart
sealed class Either<L, R> {
  const Either();
  
  bool get isLeft => this is Left<L, R>;
  bool get isRight => this is Right<L, R>;
  
  // Methods for handling either value
  T fold<T>(T Function(L) leftFn, T Function(R) rightFn) {
    return switch (this) {
      Left(value: var l) => leftFn(l),
      Right(value: var r) => rightFn(r),
    };
  }
  
  // Helper methods
  L? get leftOrNull => isLeft ? (this as Left<L, R>).value : null;
  R? get rightOrNull => isRight ? (this as Right<L, R>).value : null;
}

class Left<L, R> extends Either<L, R> {
  final L value;
  
  const Left(this.value);
  
  @override
  String toString() => 'Left($value)';
}

class Right<L, R> extends Either<L, R> {
  final R value;
  
  const Right(this.value);
  
  @override
  String toString() => 'Right($value)';
}

// Usage
Either<String, int> parseNumber(String input) {
  try {
    var value = int.parse(input);
    return Right(value);
  } catch (e) {
    return Left('Invalid number: $input');
  }
}

void processResult(Either<String, int> result) {
  // Using fold
  String message = result.fold(
    (left) => 'Error: $left',
    (right) => 'Success: $right',
  );
  print(message);
  
  // Pattern matching
  switch (result) {
    case Left(value: var error):
      print('Error: $error');
    case Right(value: var number):
      print('Number: $number');
  }
}

// Usage
var result1 = parseNumber('123');
var result2 = parseNumber('abc');

processResult(result1); // Success: 123, Number: 123
processResult(result2); // Error: Invalid number: abc, Error: Invalid number: abc

// Using helper methods
print(result1.rightOrNull); // 123
print(result2.rightOrNull); // null
print(result1.leftOrNull); // null
print(result2.leftOrNull); // Invalid number: abc
```

---

# Sealed Classes vs Enums

## Comparison

```dart
// Enum (simple values)
enum Status { pending, processing, completed, failed }

// Sealed class (with data)
sealed class TaskStatus {
  const TaskStatus();
}

class Pending extends TaskStatus {
  final DateTime createdAt;
  
  const Pending(this.createdAt);
}

class Processing extends TaskStatus {
  final double progress;
  
  const Processing(this.progress);
}

class Completed extends TaskStatus {
  final DateTime completedAt;
  
  const Completed(this.completedAt);
}

class Failed extends TaskStatus {
  final String error;
  final DateTime failedAt;
  
  const Failed(this.error, this.failedAt);
}

// Usage with enum
String getStatusMessage(Status status) {
  return switch (status) {
    Status.pending => 'Waiting...',
    Status.processing => 'Processing...',
    Status.completed => 'Done!',
    Status.failed => 'Failed!',
  };
}

// Usage with sealed class
String getTaskStatusMessage(TaskStatus status) {
  return switch (status) {
    Pending(createdAt: var time) => 'Created at $time',
    Processing(progress: var p) => 'Processing: ${(p * 100).toStringAsFixed(0)}%',
    Completed(completedAt: var time) => 'Completed at $time',
    Failed(error: var err, failedAt: var time) => 'Failed: $err at $time',
  };
}

// Usage
var pending = Pending(DateTime.now());
var processing = Processing(0.75);
var completed = Completed(DateTime.now());
var failed = Failed('Network error', DateTime.now());

print(getTaskStatusMessage(pending)); // Created at 2024-01-01 00:00:00.000
print(getTaskStatusMessage(processing)); // Processing: 75%
print(getTaskStatusMessage(completed)); // Completed at 2024-01-01 00:00:00.001
print(getTaskStatusMessage(failed)); // Failed: Network error at 2024-01-01 00:00:00.001
```

---

# Best Practices

## Use for State Management

```dart
// Good: State management with sealed classes
sealed class UIState {
  const UIState();
}

class InitialState extends UIState {
  const InitialState();
}

class LoadingState extends UIState {
  const LoadingState();
}

class LoadedState extends UIState {
  final List<String> items;
  const LoadedState(this.items);
}

class ErrorState extends UIState {
  final String message;
  const ErrorState(this.message);
}

class EmptyState extends UIState {
  const EmptyState();
}

void handleState(UIState state) {
  switch (state) {
    case InitialState():
      print('Initial state');
    case LoadingState():
      print('Loading...');
    case LoadedState(items: var items):
      print('Loaded ${items.length} items');
    case ErrorState(message: var msg):
      print('Error: $msg');
    case EmptyState():
      print('No items to display');
  }
}
```

---

## Use Exhaustive Pattern Matching

```dart
// Good: Exhaustive pattern matching
String processResult<T>(Result<T> result) {
  return switch (result) {
    Success(data: var data) => 'Success: $data',
    Error(message: var msg) => 'Error: $msg',
    Loading() => 'Loading...',
  };
}

// Bad: Non-exhaustive handling
String processResultBad<T>(Result<T> result) {
  if (result is Success) {
    return 'Success';
  }
  // Missing Error and Loading cases
}
```

---

# Common Mistakes

## Subclassing Outside File

Wrong:

```dart
// In another file
class CustomResult<T> extends Result<T> {
  // Error: Sealed class can only be extended in same file
}
```

Correct:

```dart
// Extend only in the same file where sealed class is defined
sealed class Result<T> {
  // All subclasses defined in same file
}

class Success<T> extends Result<T> { /* ... */ }
class Error<T> extends Result<T> { /* ... */ }
```

---

## Missing Cases in Switch

Wrong:

```dart
String processResult<T>(Result<T> result) {
  return switch (result) {
    Success(data: var data) => 'Success: $data',
    Error(message: var msg) => 'Error: $msg',
    // Missing Loading case - Warning/Error!
  };
}
```

Correct:

```dart
String processResult<T>(Result<T> result) {
  return switch (result) {
    Success(data: var data) => 'Success: $data',
    Error(message: var msg) => 'Error: $msg',
    Loading() => 'Loading...',
  };
}
```

---

# Summary

Sealed classes provide a powerful way to create closed type hierarchies with exhaustive pattern matching. They're ideal for state management, result types, and algebraic data types.

---

# Next Steps

Now that you understand sealed classes, continue to:

- [Base Classes](base-classes.md)
- [Interface Classes](interface-classes.md)
- [Final Classes](final-classes.md)
- [Factory Constructors](factory-constructors.md)

---

# Did You Know?

- Sealed classes were introduced in Dart 3
- They can only be extended in the same file
- Sealed classes enable exhaustive pattern matching
- They support generic types
- Sealed classes can have methods and fields
- They work great with switch expressions
- Sealed classes are more powerful than enums for complex cases