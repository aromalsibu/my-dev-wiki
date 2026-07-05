# Type System

Understand Dart's sound type system and how types work in the language.

---

# What is it?

Dart's type system is a statically-typed, sound type system that ensures type safety at compile time. It combines static typing with type inference, allowing developers to write code that is both type-safe and concise.

---

# Why does it exist?

The type system exists to:

- Catch type errors at compile time
- Make code more readable and self-documenting
- Enable better tooling (autocomplete, refactoring)
- Improve performance through optimizations
- Provide safety guarantees for developers
- Make APIs clearer and more predictable

---

# Type Categories

## Primitive Types

```dart
// Numbers
int age = 25;              // Whole numbers
double price = 19.99;       // Floating point numbers
num anyNumber = 42;        // Can be int or double

// Strings
String name = 'Alice';     // Sequence of characters

// Booleans
bool isActive = true;      // true or false

// Null
Null nothing = null;       // The null value
```

---

## Collection Types

```dart
// Lists (ordered collections)
List<String> names = ['Alice', 'Bob'];
var numbers = [1, 2, 3];    // List<int>

// Sets (unique elements)
Set<String> uniqueNames = {'Alice', 'Bob'};
var uniqueNumbers = {1, 2, 3}; // Set<int>

// Maps (key-value pairs)
Map<String, int> scores = {'Alice': 95};
var scoreMap = {'Alice': 95};   // Map<String, int>
```

---

## Function Types

```dart
// Function type
typedef IntOperation = int Function(int a, int b);

// Function variable
int Function(int, int) add = (a, b) => a + b;

// Nullable function type
void Function(String)? callback;
```

---

## Record Types

```dart
// Positional record type
(int, String) person = (25, 'Alice');

// Named record type
({int age, String name}) person = (age: 25, name: 'Alice');

// Mixed record type
(int, {String name}) person = (25, name: 'Alice');
```

---

# Type Safety

## Sound Typing

```dart
// Type checking at compile time
String name = 'Alice';
// name = 42; // Error! int cannot be assigned to String

// Type inference preserves safety
var count = 10;     // Inferred as int
// count = 'ten';   // Error! String cannot be assigned to int

// Generic type safety
List<int> numbers = [1, 2, 3];
// numbers.add('4'); // Error! String cannot be added to List<int>
```

---

## Type Checks

```dart
// Runtime type checking
void processValue(dynamic value) {
  if (value is String) {
    print('String: ${value.length}');
  } else if (value is int) {
    print('Integer: $value');
  } else {
    print('Unknown type');
  }
}
```

---

## Type Casting

```dart
// Safe casting with 'as'
dynamic value = 'Hello';
String str = value as String; // Works

// Unsafe casting (throws error if type mismatch)
dynamic number = 42;
// String str2 = number as String; // Throws error!

// Better: Check before casting
if (number is String) {
  String str3 = number as String;
}
```

---

# Type Inference

## Local Variable Inference

```dart
// Type inferred from initialization
var name = 'Alice';          // String
var age = 25;               // int
var height = 1.75;          // double
var isActive = true;        // bool
var list = [1, 2, 3];       // List<int>
var map = {'key': 'value'}; // Map<String, String>
```

---

## Collection Inference

```dart
// Empty collections need explicit type
var emptyList = <String>[];     // List<String>
var emptySet = <int>{};         // Set<int>
var emptyMap = <String, int>{}; // Map<String, int>

// Mixed types inferred as common supertype
var mixed = [1, 'two', 3.0];    // List<Object> (common supertype)
```

---

## Return Type Inference

```dart
// Return type inferred from body
add(a, b) => a + b;  // Return type inferred as int or double

// Better to be explicit
int add(int a, int b) => a + b;
```

---

# Type Hierarchy

## The Type Hierarchy

```text
Object
├── Null
├── bool
├── num
│   ├── int
│   └── double
├── String
├── List
├── Set
├── Map
├── Function
├── Record
├── Symbol
├── Future
├── Stream
└── ... (custom classes)
```

---

## Top Types

```dart
// Object - top type (everything is an Object)
Object value = 'Hello';
value = 42;
value = true;
value = null; // Works! (Null is a subtype of Object)

// Null - only contains null
Null nothing = null;

// void - no value at all (used for return types)
void doNothing() {
  // No return value
}

// dynamic - disables type checking
dynamic anyValue = 'Hello';
anyValue = 42;
anyValue = null;
anyValue.undefinedProperty; // No compile-time error
```

---

## Bottom Type

```dart
// 'never' - bottom type (Dart 3+)
// Represents a value that never occurs
// Used for exhaustive switch statements

String describeNever() {
  // Returns 'never' when a function throws or never returns
  throw Exception('Never returns');
}
```

---

# Generic Types

## Basic Generics

```dart
// Generic class
class Box<T> {
  T value;
  Box(this.value);
  
  T getValue() => value;
}

// Usage with different types
var stringBox = Box<String>('Hello');
var intBox = Box<int>(42);
```

---

## Generic Functions

```dart
// Generic function
T identity<T>(T value) {
  return value;
}

// Usage
var stringResult = identity<String>('Hello');
var intResult = identity<int>(42);

// Type inference
var inferred = identity('Hello'); // T inferred as String
```

---

## Type Bounds

```dart
// Restrict generic types
class NumberBox<T extends num> {
  T value;
  NumberBox(this.value);
}

// Only num and its subtypes allowed
var intBox = NumberBox<int>(42);    // Works
var doubleBox = NumberBox<double>(3.14); // Works
// var stringBox = NumberBox<String>('Hello'); // Error!
```

---

# Type Modifiers

## Sealed Types

```dart
// Sealed class (Dart 3+)
sealed class Result<T> {
  // Can only be extended in the same file
}

class Success<T> extends Result<T> {
  final T data;
  Success(this.data);
}

class Error<T> extends Result<T> {
  final String message;
  Error(this.message);
}

// Exhaustive pattern matching
void handleResult(Result<int> result) {
  switch (result) {
    case Success(data: var data):
      print('Success: $data');
    case Error(message: var msg):
      print('Error: $msg');
  }
}
```

---

## Final Classes

```dart
// Final class (cannot be extended or implemented)
final class Singleton {
  static final Singleton _instance = Singleton._internal();
  factory Singleton() => _instance;
  Singleton._internal();
}

// class SubClass extends Singleton {} // Error!
```

---

## Base Classes

```dart
// Base class (can only be extended, not implemented)
base class Animal {
  void speak() => print('Sound');
}

// Can extend
class Dog extends Animal {
  @override
  void speak() => print('Bark');
}

// Cannot implement
// class Cat implements Animal {} // Error!
```

---

# Type Relationships

## Subtyping

```dart
// Subtype relationships
class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}

// Dog and Cat are subtypes of Animal
Animal animal = Dog();
Animal animal2 = Cat();

// List<Dog> is NOT a subtype of List<Animal> (invariance)
// List<Animal> animals = List<Dog>(); // Error!
```

---

## Covariance

```dart
// Covariant parameter (rarely needed)
class Animal {
  void eat(covariant Animal food) {
    print('Eating $food');
  }
}

class Dog extends Animal {
  @override
  void eat(covariant Dog food) {
    print('Dog eating $food');
  }
}
```

---

# Type Operators

## Type Checks

```dart
// 'is' - type check
Object value = 'Hello';
if (value is String) {
  print('String with length ${value.length}');
}

// 'is!' - negative type check
if (value is! int) {
  print('Not an integer');
}
```

---

## Type Casting

```dart
// 'as' - type cast
dynamic value = 'Hello';
String str = value as String; // Safe cast

// 'as' with nullable types
String? maybeString = 'Hello';
String str2 = maybeString as String; // Assert not null
```

---

# Best Practices

## Prefer Explicit Types

```dart
// Better: Explicit return type
int calculateSum(List<int> numbers) {
  return numbers.reduce((a, b) => a + b);
}

// Worse: Implicit return type
calculateSum(numbers) => numbers.reduce((a, b) => a + b);
```

---

## Use Type Inference Wisely

```dart
// Good: Type is obvious
var names = ['Alice', 'Bob']; // Clear List<String>

// Good: Type is clear from context
final user = await fetchUser(); // Return type is clear

// Better: Explicit type when not obvious
Map<String, dynamic> response = await api.fetchData();
```

---

## Avoid Dynamic

```dart
// Avoid using dynamic
// dynamic value = getData(); // Avoid

// Prefer Object or specific types
Object value = getData(); // Or specific type
```

---

# Common Mistakes

## Type Mismatch

Wrong:

```dart
List<int> numbers = [1, 2, 3];
String result = numbers[0]; // Error! int can't be String
```

Correct:

```dart
List<int> numbers = [1, 2, 3];
int result = numbers[0];
```

---

## Generic Invariance

Wrong:

```dart
void processList(List<Object> items) {
  // ...
}

var strings = ['a', 'b', 'c'];
processList(strings); // Error! List<String> not List<Object>
```

Correct:

```dart
void processList<T>(List<T> items) {
  // ...
}

var strings = ['a', 'b', 'c'];
processList(strings); // Works
```

---

## Unnecessary Casting

Wrong:

```dart
var name = 'Alice' as String; // Unnecessary
```

Correct:

```dart
var name = 'Alice'; // Type inferred
```

---

# Summary

Dart's type system provides safety, clarity, and performance benefits. Understanding types helps you write better code and leverage Dart's full potential. With features like type inference, generics, and sound typing, Dart offers a powerful and flexible type system.

---

# Next Steps

Now that you understand the type system, continue to:

- [Type Inference](type-inference.md)
- [Operators](operators.md)
- [Control Flow](control-flow.md)
- [Functions](functions.md)

---

# Did You Know?

- Dart's type system is fully sound (the compiler guarantees type safety)
- The type system enables whole-program optimizations
- Dart's type inference is context-sensitive
- You can create your own generic types with constraints
- Type annotations are optional, but recommended
- The analyzer uses type information for code quality checks
- Runtime type checks are not affected by type erasure