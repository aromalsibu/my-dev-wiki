# Records

Understand Dart's record types for returning multiple values and grouping related data.

---

# What is it?

Records are a lightweight, immutable data structure introduced in Dart 3 that allows you to group multiple values into a single object. Unlike classes, records don't require defining a formal type; they are created on the fly with positional or named fields.

---

# Why does it exist?

Records exist to:

- Return multiple values from a function
- Group related data without creating a class
- Provide lightweight data transfer objects
- Enable pattern matching and destructuring
- Reduce boilerplate code
- Create temporary data structures

---

# Record Syntax

## Positional Records

```dart
// Positional record with two fields
(int, String) person = (25, 'Alice');

// Access fields using $1, $2, etc.
int age = person.$1;   // 25
String name = person.$2; // 'Alice'

// Positional record with more fields
(int, String, bool) data = (42, 'Hello', true);
int number = data.$1;   // 42
String message = data.$2; // 'Hello'
bool flag = data.$3;    // true
```

---

## Named Records

```dart
// Named record with fields
({int age, String name}) person = (age: 25, name: 'Alice');

// Access fields by name
int age = person.age;   // 25
String name = person.name; // 'Alice'

// Named record with different order
var person2 = (name: 'Bob', age: 30);
String name2 = person2.name; // 'Bob'
```

---

## Mixed Records

```dart
// Mixed positional and named fields
(int, {String name}) person = (25, name: 'Alice');

// Access positional
int age = person.$1;   // 25

// Access named
String name = person.name; // 'Alice'

// All combinations
(String, int, {bool isActive, String? nickname}) data;
```

---

# Returning Multiple Values

## Basic Example

```dart
// Function returning multiple values
(String, int) getUserInfo() {
  return ('Alice', 25);
}

// Usage
void main() {
  var info = getUserInfo();
  print('Name: ${info.$1}'); // Name: Alice
  print('Age: ${info.$2}');  // Age: 25
}

// Destructuring
void main2() {
  var (name, age) = getUserInfo();
  print('Name: $name, Age: $age');
}
```

---

## Named Returns

```dart
// Returning named record
({String name, int age}) getUserInfo() {
  return (name: 'Alice', age: 25);
}

// Usage
void main() {
  var info = getUserInfo();
  print('Name: ${info.name}');
  print('Age: ${info.age}');
  
  // Destructuring with named fields
  var (:name, :age) = getUserInfo();
  print('Name: $name, Age: $age');
}
```

---

## Mixed Returns

```dart
// Mixed positional and named
(int, {String name, bool isActive}) getUserData() {
  return (25, name: 'Alice', isActive: true);
}

// Usage
void main() {
  var data = getUserData();
  int age = data.$1;
  String name = data.name;
  bool isActive = data.isActive;
}
```

---

# Destructuring

## Positional Destructuring

```dart
// Destructuring records
var (x, y) = (10, 20);
print('x: $x, y: $y'); // x: 10, y: 20

// Destructuring with types
(String name, int age) = ('Alice', 25);
print('$name is $age years old');

// Partial destructuring (ignore some fields)
var (_, age) = ('Alice', 25);
print('Age: $age');

// Nesting
var ((x1, y1), (x2, y2)) = ((1, 2), (3, 4));
print('($x1,$y1) - ($x2,$y2)');
```

---

## Named Destructuring

```dart
// Named record destructuring
var (:age, :name) = (name: 'Alice', age: 25);
print('Name: $name, Age: $age');

// With different variable names
var (:age as userAge, :name as userName) = (name: 'Alice', age: 25);
print('$userName is $userAge years old');

// Partial destructuring
var (:name, :) = (name: 'Alice', age: 25);
print('Name: $name');
```

---

# Pattern Matching with Records

## Switch with Records

```dart
void describe(Object value) {
  switch (value) {
    case (int x, int y):
      print('Point: ($x, $y)');
    case (String name, int age):
      print('Person: $name is $age years old');
    case (String name, _, bool active):
      print('User: $name active: $active');
    default:
      print('Unknown');
  }
}

// Usage
describe((3, 4)); // Point: (3, 4)
describe(('Alice', 25)); // Person: Alice is 25 years old
describe(('Bob', 30, true)); // User: Bob active: true
```

---

## Pattern Matching with Conditions

```dart
void processPoint(Object value) {
  switch (value) {
    case (int x, int y) when x > 0 && y > 0:
      print('First quadrant: ($x, $y)');
    case (int x, int y) when x < 0 && y > 0:
      print('Second quadrant: ($x, $y)');
    case (int x, int y) when x < 0 && y < 0:
      print('Third quadrant: ($x, $y)');
    case (int x, int y) when x > 0 && y < 0:
      print('Fourth quadrant: ($x, $y)');
    case (0, int y):
      print('On Y-axis: y = $y');
    case (int x, 0):
      print('On X-axis: x = $x');
    case (0, 0):
      print('Origin');
    default:
      print('Invalid point');
  }
}
```

---

## Destructuring in Loop

```dart
// List of records
List<(String, int)> users = [
  ('Alice', 25),
  ('Bob', 30),
  ('Charlie', 35),
];

// Destructuring in loop
for (var (name, age) in users) {
  print('$name is $age years old');
}

// With map entries
Map<String, int> scores = {'Alice': 95, 'Bob': 87};
for (var MapEntry(:key, :value) in scores.entries) {
  print('$key scored $value');
}
```

---

# Records vs Classes

## When to Use Records

```dart
// Records: For temporary data
(String, int) getPerson() => ('Alice', 25);

// For simple data grouping
var point = (x: 10, y: 20);

// For return values from functions
var (success, data) = await apiCall();
```

---

## When to Use Classes

```dart
// Classes: For complex behavior
class User {
  final String name;
  final int age;
  
  User(this.name, this.age);
  
  void sayHello() => print('Hello, $name');
}

// Classes: For validation and logic
class Email {
  final String value;
  
  Email(this.value) {
    if (!value.contains('@')) {
      throw ArgumentError('Invalid email');
    }
  }
}
```

---

## Converting Records to Classes

```dart
// Record
var user = (name: 'Alice', age: 25, email: 'alice@example.com');

// Convert to class
class User {
  final String name;
  final int age;
  final String email;
  
  User({required this.name, required this.age, required this.email});
}

var userClass = User(
  name: user.name,
  age: user.age,
  email: user.email,
);
```

---

# Type Aliases for Records

## Typedef with Records

```dart
// Type alias for record
typedef Person = (String name, int age);
typedef Point = ({int x, int y});
typedef UserData = (String name, {int age, bool active});

// Usage
Person getPerson() => ('Alice', 25);
Point getPoint() => (x: 10, y: 20);
UserData getData() => ('Bob', age: 30, active: true);

// Using aliases
void processPerson(Person person) {
  print('${person.$1} is ${person.$2} years old');
}

void processPoint(Point point) {
  print('Point: (${point.x}, ${point.y})');
}
```

---

# Best Practices

## Keep Records Simple

```dart
// Good: Simple record for return values
(String, int) getUserInfo() {
  return ('Alice', 25);
}

// Good: Simple named record
({String name, int age}) getUserInfo() {
  return (name: 'Alice', age: 25);
}

// Bad: Overly complex record
(int, String, bool, {String? email, List<String>? tags, Map<String, dynamic>? metadata}) 
    getComplexData() {
  return (42, 'Hello', true);
}
// Use a class instead
```

---

## Use Records for Temporary Data

```dart
// Good: Temporary data in a function
void processUsers() {
  var user = fetchUser();
  var (name, age) = (user.name, user.age);
  // Process name and age
}

// Good: Returning multiple values
(bool success, String message) validateEmail(String email) {
  if (email.isEmpty) {
    return (false, 'Email is empty');
  }
  if (!email.contains('@')) {
    return (false, 'Invalid email format');
  }
  return (true, 'Valid email');
}
```

---

## Use Meaningful Names

```dart
// Good: Clear named fields
({String firstName, String lastName, int age}) person;

// Good: Clear typedef
typedef User = ({String name, int age, String email});

// Bad: Unclear positional fields
(String, int, String) data; // What does each field mean?
```

---

# Common Mistakes

## Accessing Wrong Field

Wrong:

```dart
var person = (name: 'Alice', age: 25);
String name = person.$1; // Error! $1 doesn't exist for named records
```

Correct:

```dart
var person = (name: 'Alice', age: 25);
String name = person.name;
```

---

## Mutating Records

Wrong:

```dart
var person = (name: 'Alice', age: 25);
person.name = 'Bob'; // Error! Records are immutable
```

Correct:

```dart
var person = (name: 'Alice', age: 25);
var updatedPerson = (name: 'Bob', age: person.age);
```

---

## Mixing Positional and Named Incorrectly

Wrong:

```dart
var person = (name: 'Alice', 25); // Error! Can't mix like this
```

Correct:

```dart
var person = (25, name: 'Alice'); // Positional first, then named
```

---

# Summary

Records provide a lightweight way to group and return multiple values without creating formal classes. They support both positional and named fields, enable destructuring and pattern matching, and are ideal for temporary data and function returns.

---

# Next Steps

Now that you understand records, continue to:

- [Patterns](patterns.md)
- [Pattern Matching](pattern-matching.md)
- [Destructuring](destructuring.md)
- [Classes](classes.md)

---

# Did You Know?

- Records were introduced in Dart 3
- Records are immutable
- Records support both positional and named fields
- Records can be destructured
- Records work with pattern matching
- Records are a value type (not reference type)
- Records have structural equality (based on their values)