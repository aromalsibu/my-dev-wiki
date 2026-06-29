# Enums

Understand how to define and use enumerations in Dart.

---

# What is it?

Enums (enumerations) are a special type that represents a fixed set of constant values. They provide a way to define a type with a limited number of possible values, making code more readable, type-safe, and maintainable.

---

# Why does it exist?

Enums exist to:

- Define a fixed set of constant values
- Make code more readable and self-documenting
- Provide type safety for restricted values
- Enable exhaustive checking in switch statements
- Group related constants together
- Reduce magic numbers and strings

---

# Basic Enums

## Simple Enum

```dart
enum Status {
  pending,
  processing,
  completed,
  failed,
}

// Usage
Status currentStatus = Status.pending;

// Compare
if (currentStatus == Status.completed) {
  print('Done!');
}
```

---

## Enum Properties

```dart
enum Day {
  monday,
  tuesday,
  wednesday,
  thursday,
  friday,
  saturday,
  sunday,
}

void main() {
  // Access by name
  Day today = Day.monday;
  
  // Get string representation
  print(today.name); // 'monday'
  
  // Get all values
  for (var day in Day.values) {
    print(day.name);
  }
  
  // Get by index
  Day firstDay = Day.values[0]; // Day.monday
  Day lastDay = Day.values.last; // Day.sunday
  
  // Get index
  int index = Day.monday.index; // 0
  int wedIndex = Day.wednesday.index; // 2
}
```

---

## Enum with Switch

```dart
enum Status {
  pending,
  processing,
  completed,
  failed,
}

String getStatusMessage(Status status) {
  switch (status) {
    case Status.pending:
      return 'Waiting to start...';
    case Status.processing:
      return 'Processing...';
    case Status.completed:
      return 'Completed successfully!';
    case Status.failed:
      return 'Failed. Please try again.';
  }
}

// Usage
print(getStatusMessage(Status.pending)); // Waiting to start...
```

---

# Enhanced Enums (Dart 3+)

## Enum with Fields

```dart
enum Status {
  pending('Waiting...'),
  processing('In progress...'),
  completed('Done!'),
  failed('Error occurred');
  
  final String message;
  
  const Status(this.message);
  
  // Additional methods
  bool get isDone => this == Status.completed || this == Status.failed;
}

// Usage
Status current = Status.pending;
print(current.message); // Waiting...
print(current.isDone); // false
```

---

## Enum with Multiple Fields

```dart
enum HttpStatus {
  ok(200, 'OK'),
  created(201, 'Created'),
  badRequest(400, 'Bad Request'),
  unauthorized(401, 'Unauthorized'),
  notFound(404, 'Not Found'),
  internalServerError(500, 'Internal Server Error');
  
  final int code;
  final String message;
  
  const HttpStatus(this.code, this.message);
  
  bool get isSuccess => code >= 200 && code < 300;
  bool get isClientError => code >= 400 && code < 500;
  bool get isServerError => code >= 500 && code < 600;
}

// Usage
HttpStatus status = HttpStatus.ok;
print('${status.code}: ${status.message}'); // 200: OK
print(status.isSuccess); // true
```

---

## Enum with Methods

```dart
enum Color {
  red,
  green,
  blue,
  yellow,
  purple,
  orange;
  
  // Method on enum
  String get hexCode {
    switch (this) {
      case Color.red:
        return '#FF0000';
      case Color.green:
        return '#00FF00';
      case Color.blue:
        return '#0000FF';
      case Color.yellow:
        return '#FFFF00';
      case Color.purple:
        return '#800080';
      case Color.orange:
        return '#FFA500';
    }
  }
  
  // Method with logic
  bool isWarm() {
    switch (this) {
      case Color.red:
      case Color.yellow:
      case Color.orange:
        return true;
      default:
        return false;
    }
  }
}

// Usage
Color color = Color.red;
print(color.hexCode); // #FF0000
print(color.isWarm()); // true
```

---

# Enum with Generics

## Generic Enum

```dart
// Generic enum (using sealed classes)
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

// Usage as enum-like pattern
Result<String> result = Success('Data loaded');

switch (result) {
  case Success(data: var data):
    print('Success: $data');
  case Error(message: var msg):
    print('Error: $msg');
  case Loading():
    print('Loading...');
}
```

---

# Enum Extensions

## Extending Enums

```dart
enum Priority { low, medium, high }

extension PriorityExtension on Priority {
  // Add methods to enum
  int get value {
    switch (this) {
      case Priority.low:
        return 1;
      case Priority.medium:
        return 2;
      case Priority.high:
        return 3;
    }
  }
  
  String get label {
    switch (this) {
      case Priority.low:
        return 'Low Priority';
      case Priority.medium:
        return 'Medium Priority';
      case Priority.high:
        return 'High Priority';
    }
  }
  
  bool get isUrgent => this == Priority.high;
}

// Usage
Priority priority = Priority.medium;
print(priority.value); // 2
print(priority.label); // Medium Priority
print(priority.isUrgent); // false
```

---

# Enum with JSON

## Serialization

```dart
enum UserRole {
  admin,
  user,
  guest,
  moderator;
  
  // Convert to string for JSON
  String toJson() => name;
  
  // Parse from string
  static UserRole fromJson(String json) {
    return UserRole.values.firstWhere(
      (role) => role.name == json,
      orElse: () => UserRole.guest,
    );
  }
}

// Usage
class User {
  final String name;
  final UserRole role;
  
  User(this.name, this.role);
  
  // JSON serialization
  Map<String, dynamic> toJson() => {
    'name': name,
    'role': role.toJson(),
  };
  
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      json['name'] as String,
      UserRole.fromJson(json['role'] as String),
    );
  }
}
```

---

# Enum with Mapping

## Converting Enums

```dart
enum Planet {
  mercury,
  venus,
  earth,
  mars,
  jupiter,
  saturn,
  uranus,
  neptune;
  
  // Mapping enum to data
  static const Map<Planet, String> _names = {
    Planet.mercury: 'Mercury',
    Planet.venus: 'Venus',
    Planet.earth: 'Earth',
    Planet.mars: 'Mars',
    Planet.jupiter: 'Jupiter',
    Planet.saturn: 'Saturn',
    Planet.uranus: 'Uranus',
    Planet.neptune: 'Neptune',
  };
  
  static const Map<Planet, int> _distanceFromSun = {
    Planet.mercury: 58,
    Planet.venus: 108,
    Planet.earth: 150,
    Planet.mars: 228,
    Planet.jupiter: 778,
    Planet.saturn: 1433,
    Planet.uranus: 2877,
    Planet.neptune: 4504,
  };
  
  String get displayName => _names[this]!;
  int get distanceFromSun => _distanceFromSun[this]!;
}

// Usage
Planet earth = Planet.earth;
print(earth.displayName); // Earth
print(earth.distanceFromSun); // 150 (million km)
```

---

# Best Practices

## Use Enums for Fixed Sets

```dart
// Good: Fixed set of values
enum DayOfWeek {
  monday,
  tuesday,
  wednesday,
  thursday,
  friday,
  saturday,
  sunday,
}

// Bad: Using strings
const String monday = 'monday';
const String tuesday = 'tuesday';
// No type safety, no exhaustiveness checking
```

---

## Prefer Enums Over Booleans

```dart
// Bad: Ambiguous boolean
bool isComplete = false;
bool hasError = false;

// Good: Clear enum
enum TaskStatus {
  pending,
  inProgress,
  completed,
  failed,
}
TaskStatus status = TaskStatus.pending;

// Even better with data
enum PaymentStatus {
  pending('Waiting for payment'),
  processing('Processing payment'),
  completed('Payment received'),
  failed('Payment failed');
  
  final String message;
  const PaymentStatus(this.message);
}
```

---

## Use Enhanced Enums for Data

```dart
// Enhanced enum with data
enum HttpMethod {
  get('GET', false),
  post('POST', true),
  put('PUT', true),
  delete('DELETE', false),
  patch('PATCH', true);
  
  final String method;
  final bool hasBody;
  
  const HttpMethod(this.method, this.hasBody);
}

// Usage
void makeRequest(HttpMethod method) {
  if (method.hasBody) {
    print('${method.method} with body');
  } else {
    print('${method.method} without body');
  }
}
```

---

# Common Mistakes

## Using Magic Strings

Wrong:

```dart
void processStatus(String status) {
  if (status == 'pending') {
    // ...
  } else if (status == 'completed') {
    // ...
  }
}
```

Correct:

```dart
enum Status { pending, completed }

void processStatus(Status status) {
  switch (status) {
    case Status.pending:
      // ...
    case Status.completed:
      // ...
  }
}
```

---

## Non-exhaustive Switch

Wrong:

```dart
enum Color { red, green, blue }

String getHex(Color color) {
  switch (color) {
    case Color.red:
      return '#FF0000';
    case Color.green:
      return '#00FF00';
    // Missing blue case!
  }
}
```

Correct:

```dart
String getHex(Color color) {
  switch (color) {
    case Color.red:
      return '#FF0000';
    case Color.green:
      return '#00FF00';
    case Color.blue:
      return '#0000FF';
  }
}
```

---

## Enums with Non-constant Values

Wrong:

```dart
enum Status {
  pending('Waiting...'), // Values must be constant
  completed('Done!'),
}
```

Correct:

```dart
enum Status {
  pending('Waiting...'),
  completed('Done!');
  
  final String message;
  const Status(this.message); // Constructor must be const
}
```

---

# Summary

Enums provide a type-safe way to define fixed sets of values. Enhanced enums (Dart 3+) allow fields, methods, and additional data, making them more powerful and flexible.

---

# Next Steps

Now that you understand enums, continue to:

- [Extensions](extensions.md)
- [Extension Types](extension-types.md)
- [Classes](classes.md)
- [Sealed Classes](sealed-classes.md)

---

# Did You Know?

- Enums were introduced in Dart 2.0
- Enhanced enums were added in Dart 3.0
- Enums can have methods and fields (Dart 3+)
- `Enum.values` gives you all enum values
- The `.name` property gives you the string name
- Enums work great with switch statements
- Enum constructors must be `const`