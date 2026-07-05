# Control Flow

Understand how to control the flow of execution in Dart programs.

---

# What is it?

Control flow refers to the order in which statements are executed in a program. Dart provides various control flow structures that allow you to make decisions, repeat operations, and handle different conditions based on your program's state.

---

# Why does it exist?

Control flow exists to:

- Make decisions based on conditions
- Execute code repeatedly (loops)
- Branch execution to different paths
- Handle exceptional cases
- Create complex program logic
- Respond to user input and program state

---

# Conditional Statements

## If Statement

```dart
// Basic if
int score = 85;

if (score >= 90) {
  print('Excellent!');
}

// If-else
if (score >= 90) {
  print('Excellent!');
} else {
  print('Good job!');
}

// If-else if-else
if (score >= 90) {
  print('A');
} else if (score >= 80) {
  print('B');
} else if (score >= 70) {
  print('C');
} else {
  print('Below C');
}
```

---

## If with Null Checks

```dart
// Null check
String? name = getUserName();

if (name != null) {
  print('Hello, $name');
} else {
  print('Hello, Guest');
}

// Type promotion with null check
String? maybeName = getData();
if (maybeName != null) {
  // maybeName is promoted to non-nullable String
  print('Length: ${maybeName.length}');
}
```

---

## If with Type Checks

```dart
void process(dynamic value) {
  if (value is String) {
    print('String: ${value.length}');
  } else if (value is int) {
    print('Integer: $value');
  } else if (value is List) {
    print('List: ${value.length} items');
  } else {
    print('Unknown type');
  }
}
```

---

# Switch Statements

## Basic Switch

```dart
String grade = 'B';

switch (grade) {
  case 'A':
    print('Excellent');
    break;
  case 'B':
    print('Good');
    break;
  case 'C':
    print('Average');
    break;
  case 'D':
    print('Below Average');
    break;
  case 'F':
    print('Fail');
    break;
  default:
    print('Invalid grade');
}
```

---

## Switch with Multiple Cases

```dart
String month = 'January';

switch (month) {
  case 'January':
  case 'February':
  case 'March':
    print('Q1 (First Quarter)');
    break;
  case 'April':
  case 'May':
  case 'June':
    print('Q2 (Second Quarter)');
    break;
  case 'July':
  case 'August':
  case 'September':
    print('Q3 (Third Quarter)');
    break;
  case 'October':
  case 'November':
  case 'December':
    print('Q4 (Fourth Quarter)');
    break;
  default:
    print('Invalid month');
}
```

---

## Switch with Patterns (Dart 3+)

```dart
// Switch with pattern matching
dynamic value = 42;

switch (value) {
  case 0:
    print('Zero');
    break;
  case int x when x > 0:
    print('Positive integer: $x');
    break;
  case int x when x < 0:
    print('Negative integer: $x');
    break;
  case String s:
    print('String: $s');
    break;
  case double d:
    print('Double: $d');
    break;
  case [int a, int b]:
    print('List with two ints: $a, $b');
    break;
  default:
    print('Unknown value');
}
```

---

## Switch Expression (Dart 3+)

```dart
// Switch as expression
String color = 'red';
String message = switch (color) {
  'red' => 'Stop',
  'yellow' => 'Caution',
  'green' => 'Go',
  _ => 'Unknown color',
};

print(message); // 'Stop'
```

---

# Loops

## For Loop

```dart
// Basic for loop
for (var i = 0; i < 5; i++) {
  print('Count: $i');
}

// For loop with collection
List<String> names = ['Alice', 'Bob', 'Charlie'];
for (var i = 0; i < names.length; i++) {
  print(names[i]);
}

// For loop with custom step
for (var i = 0; i < 10; i += 2) {
  print(i); // 0, 2, 4, 6, 8
}
```

---

## For-in Loop

```dart
// Iterating collections
List<String> names = ['Alice', 'Bob', 'Charlie'];
for (var name in names) {
  print(name);
}

// With type annotation
for (String name in names) {
  print(name.toUpperCase());
}

// Iterating maps
Map<String, int> scores = {'Alice': 95, 'Bob': 87};
for (var entry in scores.entries) {
  print('${entry.key}: ${entry.value}');
}
```

---

## For-each with Index

```dart
// Using forEach method
List<String> names = ['Alice', 'Bob', 'Charlie'];
names.forEach((name) {
  print(name);
});

// Using forEach with index
names.asMap().forEach((index, name) {
  print('$index: $name');
});

// Manual index tracking
var index = 0;
for (var name in names) {
  print('$index: $name');
  index++;
}
```

---

## While Loop

```dart
// While loop
var i = 0;
while (i < 5) {
  print(i);
  i++;
}

// While with condition
var count = 0;
while (count < 10) {
  if (count % 2 == 0) {
    print('Even: $count');
  }
  count++;
}

// While with break condition
var random = 0;
while (random != 7) {
  random = Random().nextInt(10);
  print('Rolled: $random');
}
```

---

## Do-While Loop

```dart
// Do-while (executes at least once)
var i = 0;
do {
  print(i);
  i++;
} while (i < 5);

// Do-while with validation
String? input;
do {
  print('Enter a number:');
  input = stdin.readLineSync();
} while (input == null || input.isEmpty);
```

---

# Break and Continue

## Break Statement

```dart
// Break from loop
for (var i = 0; i < 10; i++) {
  if (i == 5) {
    break; // Exits loop when i == 5
  }
  print(i); // 0, 1, 2, 3, 4
}

// Break with label
outerLoop:
for (var i = 0; i < 3; i++) {
  for (var j = 0; j < 3; j++) {
    if (i == 1 && j == 1) {
      break outerLoop; // Breaks both loops
    }
    print('$i, $j');
  }
}
```

---

## Continue Statement

```dart
// Skip iteration
for (var i = 0; i < 5; i++) {
  if (i % 2 == 0) {
    continue; // Skip even numbers
  }
  print(i); // 1, 3
}

// Continue with label
outerLoop:
for (var i = 0; i < 3; i++) {
  for (var j = 0; j < 3; j++) {
    if (i == 1 && j == 1) {
      continue outerLoop; // Continue outer loop
    }
    print('$i, $j');
  }
}
```

---

# Exception Handling

## Try-Catch-Finally

```dart
try {
  // Code that might throw an exception
  int result = 10 ~/ 0;
} on IntegerDivisionByZeroException {
  // Handle specific exception
  print('Cannot divide by zero');
} catch (e) {
  // Handle any exception
  print('Error: $e');
} finally {
  // Always executes
  print('Cleanup completed');
}
```

---

## Catching Specific Exceptions

```dart
try {
  var data = jsonDecode(input);
  // Process data
} on FormatException catch (e) {
  print('Invalid JSON format: $e');
} on TypeError catch (e) {
  print('Type mismatch: $e');
} catch (e, stacktrace) {
  print('Unknown error: $e');
  print('Stack trace: $stacktrace');
} finally {
  print('Always executed');
}
```

---

## Throwing Exceptions

```dart
// Throw an exception
void validateAge(int age) {
  if (age < 0) {
    throw ArgumentError('Age cannot be negative');
  }
  if (age < 18) {
    throw Exception('Must be at least 18 years old');
  }
}

// Custom exception
class ValidationException implements Exception {
  final String message;
  ValidationException(this.message);
  
  @override
  String toString() => 'ValidationException: $message';
}

void validateEmail(String email) {
  if (!email.contains('@')) {
    throw ValidationException('Invalid email address');
  }
}
```

---

# Assertions

## Assert Statement

```dart
// Assertions (only in debug mode)
void divide(int a, int b) {
  assert(b != 0, 'Cannot divide by zero');
  return a / b;
}

// Assert with conditions
String? name = getUserName();
assert(name != null, 'Name cannot be null');

// Assert with complex condition
List<int> numbers = [1, 2, 3];
assert(numbers.isNotEmpty, 'List cannot be empty');
```

---

# Patterns in Control Flow

## If with Pattern Matching

```dart
// Using patterns in if statements
var pair = (1, 2);

if (pair case (int x, int y)) {
  print('Coordinates: $x, $y');
}

// Destructuring in if
Map<String, dynamic> data = {'name': 'Alice', 'age': 25};
if (data case {'name': String name, 'age': int age}) {
  print('$name is $age years old');
}
```

---

## Switch with Guards

```dart
// Pattern matching with guards
void describe(dynamic value) {
  switch (value) {
    case int x when x > 0:
      print('Positive integer: $x');
    case int x when x < 0:
      print('Negative integer: $x');
    case int x:
      print('Zero');
    case String s when s.isNotEmpty:
      print('Non-empty string: $s');
    case String s:
      print('Empty string');
    default:
      print('Unknown type');
  }
}
```

---

# Best Practices

## Avoid Deep Nesting

```dart
// Bad: Deep nesting
if (condition1) {
  if (condition2) {
    if (condition3) {
      // Do something
    }
  }
}

// Good: Early returns
if (!condition1) return;
if (!condition2) return;
if (!condition3) return;
// Do something
```

---

## Use Meaningful Conditions

```dart
// Bad: Magic numbers
if (status == 1) {
  // ...
}

// Good: Named constants
const Status active = 1;
if (status == active) {
  // ...
}

// Better: Enum
enum Status { inactive, active, pending }
if (status == Status.active) {
  // ...
}
```

---

## Prefer Switch for Multiple Conditions

```dart
// Bad: Many if-else if
if (value == 'A') {
  // ...
} else if (value == 'B') {
  // ...
} else if (value == 'C') {
  // ...
}

// Good: Switch statement
switch (value) {
  case 'A':
    // ...
    break;
  case 'B':
    // ...
    break;
  case 'C':
    // ...
    break;
}
```

---

# Common Mistakes

## Infinite Loops

Wrong:

```dart
var i = 0;
while (i < 10) {
  print(i);
  // Missing i++ -> infinite loop
}
```

Correct:

```dart
var i = 0;
while (i < 10) {
  print(i);
  i++; // Increment
}
```

---

## Break in Switch

Wrong:

```dart
switch (value) {
  case 'A':
    print('A');
  case 'B':
    print('B');
  // Missing break causes fall-through
}
```

Correct:

```dart
switch (value) {
  case 'A':
    print('A');
    break; // Prevents fall-through
  case 'B':
    print('B');
    break;
}
```

---

## Null Check Missing

Wrong:

```dart
String? name = getUserName();
if (name.isNotEmpty) { // Error! name might be null
  // ...
}
```

Correct:

```dart
String? name = getUserName();
if (name != null && name.isNotEmpty) {
  // ...
}
// Or
if (name?.isNotEmpty ?? false) {
  // ...
}
```

---

# Summary

Control flow structures are essential for creating dynamic, responsive programs. Understanding when and how to use conditionals, loops, switch statements, and exception handling helps you write clear, efficient, and robust code.

---

# Next Steps

Now that you understand control flow, continue to:

- [Functions](functions.md)
- [Collections](collections.md)
- [Records](records.md)
- [Patterns](patterns.md)

---

# Did You Know?

- Dart supports pattern matching in switch statements (Dart 3+)
- The `break` statement can be used with labels
- Assertions are only active in debug mode
- Dart has no `goto` statement
- The `continue` statement can jump to a labeled loop
- For loops in Dart support the `in` keyword for iteration