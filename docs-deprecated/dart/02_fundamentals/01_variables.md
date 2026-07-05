# Variables

Understand how to declare and use variables in Dart.

---

# What is it?

Variables are named storage locations in memory that hold values. In Dart, variables can store various types of data including numbers, strings, booleans, lists, maps, and custom objects.

---

# Why does it exist?

Variables exist to:

- Store and manipulate data in programs
- Reference values by meaningful names
- Enable dynamic behavior in applications
- Track state throughout program execution
- Pass data between functions and classes

---

# Variable Declaration

## Explicit Typing

```dart
// Declare with explicit type
String name = 'Alice';
int age = 25;
double height = 1.75;
bool isActive = true;
List<String> hobbies = ['reading', 'swimming'];
Map<String, int> scores = {'math': 95, 'science': 90};
```

---

## Type Inference with `var`

```dart
// Dart infers the type
var name = 'Alice';      // String
var age = 25;           // int
var height = 1.75;      // double
var isActive = true;    // bool
var hobbies = ['reading', 'swimming']; // List<String>
var scores = {'math': 95, 'science': 90}; // Map<String, int>
```

---

## Type Inference with `final`

```dart
// Runtime constants (assigned once)
final name = 'Alice';
final age = 25;
final currentTime = DateTime.now(); // Set at runtime

// Cannot reassign
// name = 'Bob'; // Error!
```

---

## Compile-time Constants with `const`

```dart
// Compile-time constants
const pi = 3.14159;
const appName = 'MyApp';
const maxRetries = 3;

// Must be known at compile time
const currentTime = DateTime.now(); // Error! Not compile-time constant
```

---

# Null Safety

## Non-nullable Variables

```dart
// By default, variables cannot be null
String name = 'Alice'; // Must have a value
int age = 25;          // Must have a value
```

---

## Nullable Variables

```dart
// Use '?' to allow null values
String? maybeName = null;
int? maybeAge = null;

// Can be assigned later
String? nickname;
nickname = 'Ally';
```

---

## Late Initialization

```dart
// Declare without initial value
late String description;
late int userId;

// Initialize later
void initializeUser() {
  description = 'Administrator';
  userId = 42;
}

// Access after initialization
void printUser() {
  print('$description: $userId');
}
```

---

# Variable Types

## Numbers

```dart
// int - Whole numbers
int count = 10;
int negative = -5;
int hex = 0xFF;          // Hexadecimal
int binary = 0b1010;     // Binary

// double - Floating point numbers
double price = 19.99;
double decimal = 3.14159;
double scientific = 1.5e12;

// num - Can be either int or double
num number = 42;
number = 3.14; // Can change type
```

---

## Strings

```dart
// Single quotes
String single = 'Hello';

// Double quotes
String double = "World";

// String interpolation
String greeting = 'Hello, $name';
String message = '${name.toUpperCase()} says hello';

// Multi-line strings
String multiline = '''
This is a
multi-line
string
''';

// Raw strings (ignore escape sequences)
String raw = r'C:\Users\name\file.txt';
```

---

## Booleans

```dart
// Only true or false
bool isActive = true;
bool isComplete = false;

// Cannot be null by default
bool? maybeFlag = null; // Nullable boolean
```

---

## Lists

```dart
// Generic list with type
List<String> names = ['Alice', 'Bob', 'Charlie'];

// Type inference
var names = ['Alice', 'Bob', 'Charlie'];

// Empty list
List<int> empty = [];
var empty = <String>[];

// Fixed-length list
var fixed = List<int>.filled(3, 0);
fixed[0] = 1;
// fixed.add(4); // Error! Cannot add to fixed-length list

// List operations
names.add('David');
names.insert(1, 'Eve');
names.remove('Bob');
names.removeAt(0);
```

---

## Sets

```dart
// Generic set with type
Set<String> uniqueNames = {'Alice', 'Bob', 'Charlie'};

// Type inference
var uniqueNames = {'Alice', 'Bob', 'Charlie'};

// Empty set (not a map!)
Set<int> empty = {};
var empty = <int>{};

// Set operations
uniqueNames.add('David');
uniqueNames.add('Bob'); // Won't add duplicate
uniqueNames.remove('Alice');
```

---

## Maps

```dart
// Generic map with types
Map<String, int> scores = {
  'Alice': 95,
  'Bob': 87,
  'Charlie': 92,
};

// Type inference
var scores = {
  'Alice': 95,
  'Bob': 87,
  'Charlie': 92,
};

// Empty map
Map<String, int> empty = {};
var empty = <String, int>{};

// Map operations
scores['David'] = 88;
scores.remove('Bob');
var aliceScore = scores['Alice']; // 95
```

---

# Advanced Variable Concepts

## Getters and Setters

```dart
class Person {
  String _name;
  
  Person(this._name);
  
  // Getter
  String get name => _name;
  
  // Setter with validation
  set name(String value) {
    if (value.isNotEmpty) {
      _name = value;
    }
  }
}
```

---

## Variables in Classes

```dart
class User {
  // Instance variables
  String name;
  int age;
  final String id;       // Must be initialized in constructor
  late String email;     // Late initialized
  
  // Static variables (class level)
  static int totalUsers = 0;
  
  // Constants (compile-time)
  static const int maxAge = 150;
  
  User(this.name, this.age, this.id) {
    totalUsers++;
  }
  
  // Instance method
  void display() {
    print('$name, $age years old');
  }
}
```

---

## Scope

```dart
class Example {
  // Class scope
  var classVariable = 'class';
  
  void method() {
    // Method scope
    var methodVariable = 'method';
    
    if (true) {
      // Block scope
      var blockVariable = 'block';
      print(classVariable);    // Accessible
      print(methodVariable);   // Accessible
    }
    
    // print(blockVariable);   // Error! Not accessible outside block
  }
}
```

---

# Type Conversion

## Explicit Casting

```dart
// Number conversions
int toDouble = 42;
double doubleValue = toDouble.toDouble();

double toInt = 42.7;
int intValue = toInt.toInt(); // 42 (truncates)

// String conversions
String numberString = '123';
int parsedInt = int.parse(numberString);
double parsedDouble = double.parse('3.14');

// To string
String intString = 42.toString();
String doubleString = 3.14159.toStringAsFixed(2); // '3.14'
```

---

# Common Operators

## Assignment

```dart
// Basic assignment
var x = 5;

// Compound assignment
x += 3; // x = x + 3
x -= 2; // x = x - 2
x *= 2; // x = x * 2
x /= 2; // x = x / 2
x %= 3; // x = x % 3
```

---

## Increment/Decrement

```dart
var count = 0;

// Prefix (increment before use)
print(++count); // 1

// Postfix (increment after use)
print(count++); // 1
print(count);   // 2
```

---

## Null-aware Operators

```dart
String? maybeName = null;

// Null-aware access
String? upperName = maybeName?.toUpperCase(); // null

// Null-aware assignment
maybeName ??= 'Guest'; // Assign if null

// Null coalescing
String displayName = maybeName ?? 'Anonymous';
```

---

# Best Practices

## Naming Conventions

```dart
// Use camelCase for variables
String firstName = 'Alice';
int userAge = 25;

// Use UPPER_SNAKE_CASE for constants
const MAX_RETRIES = 3;
const DEFAULT_NAME = 'Guest';

// Use _ prefix for private variables
String _internalState = 'private';
```

---

## Variable Usage Guidelines

```dart
// Prefer final for variables that don't change
final name = 'Alice';
final age = 25;

// Use var when type is obvious from initialization
var items = ['a', 'b', 'c']; // Type is clear

// Explicitly type when type isn't obvious
Map<String, dynamic> response = fetchData();

// Avoid dynamic unless absolutely necessary
dynamic value = 'hello'; // Prefer specific types
```

---

# Common Mistakes

## Null Safety Violation

Wrong:

```dart
String name; // Non-nullable variable not initialized
print(name); // Error!
```

Correct:

```dart
String name = 'Alice';
print(name);

// Or
String? name;
print(name); // Prints null (but handle it)
```

---

## Type Mismatch

Wrong:

```dart
int count = '5'; // Error! Can't assign String to int
```

Correct:

```dart
int count = int.parse('5');
```

---

## Reassigning Final Variables

Wrong:

```dart
final name = 'Alice';
name = 'Bob'; // Error! Can't reassign final
```

Correct:

```dart
var name = 'Alice';
name = 'Bob'; // Works
```

---

# Summary

Variables are fundamental to programming in Dart. Understanding how to declare, initialize, and use variables effectively is essential for writing clear, efficient, and safe code.

---

# Next Steps

Now that you understand variables, continue to:

- [Null Safety](null-safety.md)
- [Type System](type-system.md)
- [Operators](operators.md)
- [Control Flow](control-flow.md)

---

# Did You Know?

- Dart's type inference is very smart and can infer complex types
- The Dart VM optimizes variables based on their usage patterns
- Dart's null safety helps prevent billions of dollars in bugs
- The language supports both strong and weak typing through inference
- Dart variables are references to objects, not primitive values