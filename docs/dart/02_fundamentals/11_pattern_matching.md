# Pattern Matching

Understand Dart's powerful pattern matching capabilities for destructuring and matching data structures.

---

# What is it?

Pattern matching is a feature that allows you to test a value against a pattern and extract data from it. In Dart, pattern matching is integrated with switch statements, if statements, and variable declarations, enabling concise and expressive code.

---

# Why does it exist?

Pattern matching exists to:

- Extract data from complex structures
- Write more expressive and readable code
- Reduce boilerplate and manual casting
- Enable exhaustive checking
- Support functional programming patterns
- Handle multiple cases elegantly

---

# Switch Pattern Matching

## Basic Patterns

```dart
void describe(dynamic value) {
  switch (value) {
    case 0:
      print('Zero');
    case 42:
      print('The answer');
    case 'hello':
      print('Greeting');
    case int n:
      print('Integer: $n');
    case String s:
      print('String: $s');
    case bool b:
      print('Boolean: $b');
    default:
      print('Unknown');
  }
}
```

---

## Record Patterns

```dart
void processPoint(Object value) {
  switch (value) {
    case (0, 0):
      print('Origin');
    case (int x, 0):
      print('On X-axis: x = $x');
    case (0, int y):
      print('On Y-axis: y = $y');
    case (int x, int y) when x == y:
      print('On diagonal: ($x, $y)');
    case (int x, int y):
      print('Point: ($x, $y)');
    default:
      print('Invalid point');
  }
}
```

---

## List Patterns

```dart
void processList(Object value) {
  switch (value) {
    case []:
      print('Empty list');
    case [int x]:
      print('Single element: $x');
    case [int a, int b]:
      print('Two elements: $a, $b');
    case [int a, int b, int c] when a + b == c:
      print('Valid: $a + $b = $c');
    case [int first, ...int rest]:
      print('First: $first, Rest: $rest');
    default:
      print('Unknown list');
  }
}
```

---

## Map Patterns

```dart
void processMap(Object value) {
  switch (value) {
    case {'type': 'user', 'name': String name, 'age': int age}:
      print('User: $name, $age');
    case {'type': 'product', 'name': String name, 'price': int price}:
      print('Product: $name, \$$price');
    case {'error': String error}:
      print('Error: $error');
    case {}:
      print('Empty map');
    default:
      print('Unknown map');
  }
}
```

---

## Object Patterns

```dart
class Point {
  final int x, y;
  Point(this.x, this.y);
}

class Circle {
  final Point center;
  final int radius;
  Circle(this.center, this.radius);
}

void processShape(Object value) {
  switch (value) {
    case Point(x: 0, y: 0):
      print('Origin point');
    case Point(x: int x, y: int y) when x == y:
      print('Point on diagonal: ($x, $y)');
    case Point(x: int x, y: int y):
      print('Point: ($x, $y)');
    case Circle(center: Point(x: 0, y: 0), radius: int r):
      print('Circle at origin with radius $r');
    case Circle(center: Point(x: int cx, y: int cy), radius: int r):
      print('Circle at ($cx, $cy) with radius $r');
    default:
      print('Unknown shape');
  }
}
```

---

# Exhaustive Pattern Matching

## Sealed Classes

```dart
// Sealed class ensures exhaustive checking
sealed class Result<T> {
  const Result();
}

class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);
}

class Error<T> extends Result<T> {
  final String message;
  const Error(this.message);
}

class Loading<T> extends Result<T> {
  const Loading();
}

// Exhaustive switch (all cases handled)
String handleResult<T>(Result<T> result) {
  return switch (result) {
    Success(data: var data) => 'Success: $data',
    Error(message: var msg) => 'Error: $msg',
    Loading() => 'Loading...',
  };
}
```

---

## Pattern Matching with Exhaustiveness

```dart
// All enum cases must be handled
enum Status { loading, success, error }

String getStatusMessage(Status status) {
  return switch (status) {
    Status.loading => 'Loading...',
    Status.success => 'Success!',
    Status.error => 'Error occurred',
  };
}

// With data attached
sealed class ApiState {}

class ApiLoading extends ApiState {}
class ApiSuccess extends ApiState {
  final String data;
  ApiSuccess(this.data);
}
class ApiError extends ApiState {
  final String error;
  ApiError(this.error);
}

String handleApiState(ApiState state) {
  return switch (state) {
    ApiLoading() => 'Loading...',
    ApiSuccess(data: var data) => 'Data: $data',
    ApiError(error: var err) => 'Error: $err',
  };
}
```

---

# Pattern Matching with Guards

## Multiple Conditions

```dart
void processNumber(Object value) {
  switch (value) {
    case int n when n < 0:
      print('Negative: $n');
    case int n when n == 0:
      print('Zero');
    case int n when n > 0 && n <= 10:
      print('Small positive: $n');
    case int n when n > 10 && n <= 100:
      print('Medium positive: $n');
    case int n when n > 100:
      print('Large positive: $n');
    default:
      print('Not an integer');
  }
}
```

---

## Complex Guards

```dart
class User {
  final String name;
  final int age;
  final bool isActive;
  User(this.name, this.age, this.isActive);
}

void processUser(Object value) {
  switch (value) {
    case User(
      name: String name,
      age: int age,
      isActive: true
    ) when age >= 18:
      print('Active adult: $name');
    case User(
      name: String name,
      age: int age,
      isActive: true
    ) when age < 18:
      print('Active minor: $name');
    case User(
      name: String name,
      isActive: false
    ):
      print('Inactive user: $name');
    default:
      print('Not a user');
  }
}
```

---

# Pattern Matching in Expressions

## Switch Expressions

```dart
// Switch as expression
String describe(int value) {
  return switch (value) {
    0 => 'Zero',
    1 => 'One',
    2 => 'Two',
    int n when n > 0 && n < 10 => 'Single digit: $n',
    int n when n >= 10 && n < 100 => 'Double digit: $n',
    _ => 'Other',
  };
}

// With records
String getPointType((int, int) point) {
  return switch (point) {
    (0, 0) => 'Origin',
    (int x, 0) => 'X-axis: $x',
    (0, int y) => 'Y-axis: $y',
    (int x, int y) when x == y => 'Diagonal: ($x, $y)',
    (int x, int y) => 'Point: ($x, $y)',
  };
}
```

---

## If Statement with Pattern

```dart
// Pattern matching in if
void processData(Object data) {
  if (data case (String name, int age)) {
    print('Name: $name, Age: $age');
  } else if (data case [String first, String second]) {
    print('Two strings: $first, $second');
  } else if (data case {'type': 'user', 'name': String name}) {
    print('User: $name');
  }
}
```

---

# Pattern Matching with Null Safety

## Handling Null

```dart
// Null-aware pattern matching
void processValue(Object? value) {
  switch (value) {
    case null:
      print('Null value');
    case String s:
      print('String: $s');
    case int n:
      print('Integer: $n');
    default:
      print('Other');
  }
}

// With null check
String? getName(Object? value) {
  if (value case String name) {
    return name;
  }
  return null;
}
```

---

# Pattern Matching with Collections

## Nested Patterns

```dart
// Complex nested pattern matching
void processComplex(Object value) {
  switch (value) {
    case [
      String first,
      [int x, int y],
      {'name': String name, 'age': int age}
    ]:
      print('First: $first, Point: ($x,$y), User: $name, $age');
    case [[int x1, int y1], [int x2, int y2]]:
      print('Line from ($x1,$y1) to ($x2,$y2)');
    default:
      print('Unknown structure');
  }
}
```

---

## Collection Patterns

```dart
// Pattern matching with collections
void processCollection(Object value) {
  switch (value) {
    case [int a, int b, int c]:
      print('Three integers: $a, $b, $c');
    case [int a, ...int rest] when rest.length > 2:
      print('First: $a, Rest: $rest');
    case {'name': String name, 'scores': List<int> scores}:
      print('$name scores: $scores');
    default:
      print('Unknown collection');
  }
}
```

---

# Best Practices

## Use Exhaustive Matching

```dart
// Good: Exhaustive matching with sealed classes
sealed class State {}
class Loading extends State {}
class Success extends State { final String data; Success(this.data); }
class Error extends State { final String error; Error(this.error); }

String handleState(State state) {
  return switch (state) {
    Loading() => 'Loading...',
    Success(data: var data) => 'Success: $data',
    Error(error: var err) => 'Error: $err',
  };
}

// Bad: Non-exhaustive matching
String handleStateBad(State state) {
  switch (state) {
    case Loading():
      return 'Loading...';
    case Success(data: var data):
      return 'Success: $data';
    // Missing Error case!
  }
}
```

---

## Use Guards Wisely

```dart
// Good: Clear guard conditions
case int n when n > 0 && n % 2 == 0:
  print('Even positive: $n');

// Bad: Overly complex guards
case int n when (n > 0 && n % 2 == 0) || (n < 0 && n % 2 != 0):
  print('Complex condition');

// Better: Simplify or extract logic
bool isValid(int n) => (n > 0 && n % 2 == 0) || (n < 0 && n % 2 != 0);
case int n when isValid(n):
  print('Valid number');
```

---

# Common Mistakes

## Missing Default Case

Wrong:

```dart
switch (value) {
  case 0:
    print('Zero');
  case 1:
    print('One');
  // No default case
}
```

Correct:

```dart
switch (value) {
  case 0:
    print('Zero');
    break;
  case 1:
    print('One');
    break;
  default:
    print('Other');
}
```

---

## Incorrect Guard Syntax

Wrong:

```dart
case int n if n > 0: // Error! Use 'when', not 'if'
  print('Positive');
```

Correct:

```dart
case int n when n > 0:
  print('Positive');
```

---

## Pattern Not Exhaustive

Wrong:

```dart
sealed class Result {}
class Success extends Result {}
class Error extends Result {}

String handle(Result result) {
  return switch (result) {
    Success() => 'Success',
    // Missing Error case - warning/error
  };
}
```

Correct:

```dart
String handle(Result result) {
  return switch (result) {
    Success() => 'Success',
    Error() => 'Error',
  };
}
```

---

# Summary

Pattern matching provides powerful ways to destructure and match against complex data structures. Combined with switch expressions and guards, it enables concise, expressive, and safe code.

---

# Next Steps

Now that you understand pattern matching, continue to:

- [Destructuring](destructuring.md)
- [Enums](enums.md)
- [Classes](classes.md)
- [Sealed Classes](sealed-classes.md)

---

# Did You Know?

- Pattern matching was introduced in Dart 3
- Sealed classes enable exhaustive pattern matching
- Guards use `when` for conditional matching
- Patterns can be nested arbitrarily deep
- Switch expressions return values directly
- The compiler can check exhaustiveness with sealed types