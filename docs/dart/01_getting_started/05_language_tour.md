# Language Tour

A comprehensive overview of Dart's syntax and core features.

---

# What is it?

The Language Tour is a guided introduction to Dart's syntax, features, and programming concepts. It covers everything from basic syntax to advanced language features, helping you understand how to write Dart code effectively.

---

# Why does it exist?

The Language Tour exists to:

- Provide a complete overview of Dart's capabilities
- Help developers learn Dart quickly and efficiently
- Serve as a reference for Dart's syntax and features
- Demonstrate idiomatic Dart code patterns
- Bridge the gap between beginner and advanced concepts

---

# Basic Syntax

## Hello World

```dart
// The entry point of every Dart program
void main() {
  print('Hello, World!');
}
```

---

## Comments

```dart
// Single line comment

/*
   Multi-line comment
   spanning multiple lines
*/

/// Documentation comment for classes and functions
/// Use this to document your API
```

---

## Semicolons

```dart
// Every statement ends with a semicolon
var name = 'Dart';
print(name);

// Except for curly braces and function bodies
void greet() {
  print('Hello');
}
```

---

# Variables

## Variable Declaration

```dart
// Explicit type
String name = 'Alice';

// Type inference with var
var age = 25;

// Type inference with final (runtime constant)
final height = 1.75;

// Compile-time constant
const pi = 3.14159;

// Late initialization
late String description;
description = 'Initialized later';
```

---

## Variable Types

```dart
// Numbers
int count = 10;
double price = 19.99;
num number = 42; // Can be int or double

// Strings
String greeting = 'Hello';
String multiline = '''
This is a
multi-line string
''';

// Booleans
bool isActive = true;

// Lists
List<String> names = ['Alice', 'Bob'];
var numbers = [1, 2, 3];

// Maps
Map<String, int> scores = {'Alice': 95, 'Bob': 87};
var scoreMap = {'Alice': 95};

// Sets
Set<String> uniqueNames = {'Alice', 'Bob', 'Alice'};
```

---

## Null Safety

```dart
// Non-nullable (default)
String name = 'Alice'; // Cannot be null

// Nullable
String? maybeName = null;

// Null-aware operators
String displayName = maybeName ?? 'Guest'; // Fallback
String? upperName = maybeName?.toUpperCase(); // Null-safe access

// Non-null assertion (use with caution)
String definitelyName = maybeName!;
```

---

# Functions

## Function Declaration

```dart
// Basic function
int add(int a, int b) {
  return a + b;
}

// Arrow syntax (single expression)
int multiply(int a, int b) => a * b;

// Void return
void greet(String name) {
  print('Hello, $name');
}
```

---

## Parameters

```dart
// Required positional parameters
void display(String name, int age) {
  print('$name is $age years old');
}

// Optional positional parameters
void display(String name, [int? age]) {
  if (age != null) {
    print('$name is $age years old');
  } else {
    print(name);
  }
}

// Named parameters
void display({required String name, int? age}) {
  print('$name ${age != null ? 'is $age' : ''}');
}

// Default parameter values
void greet(String name, {String greeting = 'Hello'}) {
  print('$greeting, $name');
}
```

---

## Anonymous Functions

```dart
// Assign to variable
var multiply = (int a, int b) => a * b;

// Inline function
var numbers = [1, 2, 3];
numbers.forEach((number) {
  print(number);
});

// Arrow syntax for single expression
numbers.forEach((number) => print(number));
```

---

# Control Flow

## If-Else

```dart
if (score > 90) {
  print('Excellent');
} else if (score > 70) {
  print('Good');
} else {
  print('Needs improvement');
}
```

---

## Switch-Case

```dart
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
  default:
    print('Unknown grade');
}

// Switch with patterns (Dart 3+)
switch (value) {
  case 0:
    print('Zero');
    break;
  case 1:
  case 2:
    print('Small number');
    break;
  case int x when x > 10:
    print('Large number: $x');
    break;
}
```

---

## Loops

```dart
// For loop
for (var i = 0; i < 5; i++) {
  print(i);
}

// For-in loop
for (var name in names) {
  print(name);
}

// While loop
var i = 0;
while (i < 5) {
  print(i);
  i++;
}

// Do-while loop
var j = 0;
do {
  print(j);
  j++;
} while (j < 5);
```

---

# Collections

## Lists

```dart
// Creating lists
List<String> names = ['Alice', 'Bob', 'Charlie'];
var numbers = [1, 2, 3];

// Empty list
List<String> empty = [];

// Fixed-length list
var fixed = List<int>.filled(3, 0);

// List operations
names.add('David');
names.insert(1, 'Eve');
names.remove('Bob');
names.removeAt(0);
names.clear();

// Accessing elements
String first = names[0];
String last = names[names.length - 1];
```

---

## Sets

```dart
// Creating sets
Set<String> uniqueNames = {'Alice', 'Bob', 'Charlie'};
var numbers = {1, 2, 3};

// Empty set
Set<int> empty = {};

// Set operations
uniqueNames.add('David');
uniqueNames.remove('Bob');
bool contains = uniqueNames.contains('Alice');

// Set operations
var set1 = {1, 2, 3};
var set2 = {2, 3, 4};
var union = set1.union(set2); // {1, 2, 3, 4}
var intersection = set1.intersection(set2); // {2, 3}
var difference = set1.difference(set2); // {1}
```

---

## Maps

```dart
// Creating maps
Map<String, int> scores = {
  'Alice': 95,
  'Bob': 87,
  'Charlie': 92,
};
var scoreMap = {'Alice': 95, 'Bob': 87};

// Map operations
scores['David'] = 88;
scores.remove('Bob');
bool contains = scores.containsKey('Alice');

// Iterating maps
for (var entry in scores.entries) {
  print('${entry.key}: ${entry.value}');
}
```

---

# Classes

## Basic Class

```dart
class Person {
  String name;
  int age;
  
  // Constructor
  Person(this.name, this.age);
  
  // Methods
  void sayHello() {
    print('Hello, I am $name');
  }
  
  // Getters
  bool get isAdult => age >= 18;
  
  // Setters
  set age(int value) {
    if (value >= 0) {
      _age = value;
    }
  }
}

// Usage
var person = Person('Alice', 25);
person.sayHello();
print(person.isAdult);
```

---

## Constructor Types

```dart
class Point {
  double x, y;
  
  // Generative constructor
  Point(this.x, this.y);
  
  // Named constructor
  Point.origin() : this(0, 0);
  
  // Factory constructor
  factory Point.fromJson(Map<String, double> json) {
    return Point(json['x']!, json['y']!);
  }
  
  // Redirecting constructor
  Point.zero() : this(0, 0);
}

// Usage
var p1 = Point(3, 4);
var p2 = Point.origin();
var p3 = Point.fromJson({'x': 1.0, 'y': 2.0});
```

---

## Inheritance

```dart
// Base class
class Animal {
  String name;
  
  Animal(this.name);
  
  void speak() {
    print('$name makes a sound');
  }
}

// Derived class
class Dog extends Animal {
  Dog(String name) : super(name);
  
  @override
  void speak() {
    print('$name barks');
  }
}

// Usage
var dog = Dog('Rex');
dog.speak(); // Rex barks
```

---

## Mixins

```dart
mixin Flyable {
  void fly() {
    print('Flying');
  }
}

mixin Swimmable {
  void swim() {
    print('Swimming');
  }
}

class Duck extends Animal with Flyable, Swimmable {
  Duck(String name) : super(name);
}

// Usage
var duck = Duck('Donald');
duck.fly(); // Flying
duck.swim(); // Swimming
```

---

# Records

## Record Types

```dart
// Positional record
(int, String) person = (25, 'Alice');
var age = person.$1;
var name = person.$2;

// Named record
({int age, String name}) person = (age: 25, name: 'Alice');
var age = person.age;
var name = person.name;

// Record with both
(int, {String name}) person = (25, name: 'Alice');
```

---

## Record Usage

```dart
// Function returning multiple values
(String, int) getPerson() {
  return ('Alice', 25);
}

// Destructuring
var (name, age) = getPerson();
print('$name is $age years old');

// Pattern matching
switch (getPerson()) {
  case ('Alice', int age):
    print('Alice is $age');
  case (String name, 25):
    print('$name is 25');
}
```

---

# Patterns

## Pattern Matching

```dart
// Variable pattern
var (x, y) = (1, 2);

// List pattern
var [a, b, c] = [1, 2, 3];

// Map pattern
var {'name': name, 'age': age} = {'name': 'Alice', 'age': 25};

// Switch with patterns
switch (value) {
  case 0:
    print('Zero');
  case int x when x > 0:
    print('Positive: $x');
  case String s:
    print('String: $s');
}
```

---

## Destructuring

```dart
// Record destructuring
var (x, y) = (1, 2);

// Class destructuring (with pattern)
class Point {
  final int x, y;
  Point(this.x, this.y);
}

var Point(x: x, y: y) = Point(1, 2);

// Map destructuring
var {'name': name, 'age': age} = {'name': 'Alice', 'age': 25};
```

---

# Asynchronous Programming

## Future

```dart
// Creating a Future
Future<String> fetchData() {
  return Future.delayed(
    Duration(seconds: 1),
    () => 'Data loaded',
  );
}

// Using Future
void main() {
  fetchData().then((data) {
    print(data);
  });
}
```

---

## Async/Await

```dart
// Async function
Future<String> fetchData() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Data loaded';
}

// Using await
void main() async {
  String data = await fetchData();
  print(data);
}

// Error handling
try {
  String data = await fetchData();
  print(data);
} catch (e) {
  print('Error: $e');
}
```

---

## Streams

```dart
// Creating a Stream
Stream<int> countStream() async* {
  for (var i = 1; i <= 5; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i;
  }
}

// Using a Stream
void main() async {
  await for (var value in countStream()) {
    print(value);
  }
}
```

---

# Exceptions

## Try-Catch

```dart
try {
  int result = 10 ~/ 0;
} on IntegerDivisionByZeroException {
  print('Cannot divide by zero');
} catch (e) {
  print('Error: $e');
} finally {
  print('Cleanup');
}
```

---

## Throwing Exceptions

```dart
void validateAge(int age) {
  if (age < 0) {
    throw ArgumentError('Age cannot be negative');
  }
  if (age < 18) {
    throw Exception('Must be at least 18');
  }
}

// Usage
try {
  validateAge(-1);
} catch (e) {
  print('Validation failed: $e');
}
```

---

# Generics

## Generic Classes

```dart
class Box<T> {
  T value;
  
  Box(this.value);
  
  T getValue() => value;
}

// Usage
Box<String> stringBox = Box('Hello');
Box<int> intBox = Box(42);
```

---

## Generic Functions

```dart
// Generic function
T identity<T>(T value) {
  return value;
}

// Usage
var stringResult = identity('Hello');
var intResult = identity(42);

// Type bounds
T max<T extends Comparable<T>>(T a, T b) {
  return a.compareTo(b) > 0 ? a : b;
}
```

---

# Common Operators

## Equality and Comparison

```dart
// Equality
var equals = a == b;
var notEquals = a != b;

// Comparison
var greater = a > b;
var greaterOrEqual = a >= b;
var less = a < b;
var lessOrEqual = a <= b;

// Null-aware operators
var value = maybeValue ?? defaultValue; // Null-aware default
var result = object?.property; // Null-aware access
```

---

## Type Operators

```dart
// Type check
bool isString = value is String;

// Type cast
String stringValue = value as String;

// Value assignment
var value ??= defaultValue; // Assign if null
```

---

## Cascade Notation

```dart
var list = [1, 2, 3]
  ..add(4)
  ..add(5)
  ..remove(1);

// Equivalent to:
var list = [1, 2, 3];
list.add(4);
list.add(5);
list.remove(1);
```

---

# Summary

Dart provides a comprehensive set of features for modern application development. From its clean syntax and strong typing to advanced features like async/await and pattern matching, Dart is designed to be productive, performant, and enjoyable to use.

---

# Next Steps

Now that you've seen the language tour, explore specific topics:

- [Variables](variables.md)
- [Null Safety](null-safety.md)
- [Functions](functions.md)
- [Classes](classes.md)
- [Asynchronous Programming](async.md)

---

# Did You Know?

- Dart is compiled to native code for mobile and desktop
- The language supports both JIT and AOT compilation
- Dart's type system is sound and null-safe
- Dart 3 introduced records and patterns
- The language is continuously evolving with community input