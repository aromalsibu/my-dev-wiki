# Type Inference

Understand how Dart infers types and when to rely on it.

---

# What is it?

Type inference is Dart's ability to automatically determine the type of a variable, expression, or function return value based on its context and usage. This allows you to write concise code while maintaining type safety, as Dart will infer the appropriate type without requiring explicit annotations.

---

# Why does it exist?

Type inference exists to:

- Reduce boilerplate code (less typing)
- Make code cleaner and more readable
- Maintain type safety without explicit annotations
- Allow developers to focus on logic rather than type declarations
- Provide better tooling and IDE support
- Enable smart code completion and refactoring

---

# Variable Inference

## Basic Variable Inference

```dart
// Type inferred from initialization value
var name = 'Alice';        // String
var age = 25;              // int
var height = 1.75;         // double
var isActive = true;       // bool
var items = [1, 2, 3];     // List<int>
var scores = {'a': 1, 'b': 2}; // Map<String, int>
```

---

## Multiple Variables

```dart
// All variables infer their own type
var x = 10;                // int
var y = 3.14;              // double
var z = 'Hello';           // String

// Type can be used in operations
var result = x + y;        // double (int promoted to double)
var message = z + ' World'; // String
```

---

## Final and Const Inference

```dart
// final variables also use inference
final name = 'Alice';      // String
final age = 25;            // int
final currentTime = DateTime.now(); // DateTime

// const variables must be compile-time constants
const pi = 3.14159;        // double
const maxRetries = 3;       // int
```

---

# Collection Inference

## List Inference

```dart
// Inferred as List<int>
var numbers = [1, 2, 3, 4, 5];

// Inferred as List<String>
var names = ['Alice', 'Bob', 'Charlie'];

// Mixed types infer to common supertype
var mixed = [1, 'two', 3.0]; // List<Object> (common supertype)

// Empty lists need explicit type
var emptyList = <String>[];  // List<String>
var anotherEmpty = [];       // List<dynamic> (avoid!)
```

---

## Set Inference

```dart
// Inferred as Set<int>
var numbers = {1, 2, 3, 4, 5};

// Inferred as Set<String>
var names = {'Alice', 'Bob', 'Charlie'};

// Empty sets need explicit type
var emptySet = <String>{};   // Set<String>
var anotherEmpty = {};        // Map<dynamic, dynamic> (not a set!)
```

---

## Map Inference

```dart
// Inferred as Map<String, int>
var scores = {'Alice': 95, 'Bob': 87};

// Inferred as Map<int, String>
var grades = {1: 'A', 2: 'B', 3: 'C'};

// Mixed key types
var mixed = {'key': 1, 2: 'value'}; // Map<Object, Object>

// Empty maps need explicit type
var emptyMap = <String, int>{};    // Map<String, int>
```

---

# Function Return Type Inference

## Return Type Inference

```dart
// Return type inferred as int
add(a, b) => a + b;

// Return type inferred as String
greet(name) => 'Hello, $name';

// Return type inferred as Object (common supertype)
getData() => 42; // int is Object

// Better: Explicit return types for clarity
int add(int a, int b) => a + b;
String greet(String name) => 'Hello, $name';
```

---

## Multiple Return Paths

```dart
// Return type inferred as int
getValue(bool condition) {
  if (condition) {
    return 42;      // int
  } else {
    return 0;       // int
  }
}

// Return type inferred as Object (int and String)
getMixed(bool condition) {
  if (condition) {
    return 42;      // int
  } else {
    return 'zero';  // String
  }
}
```

---

# Lambda and Function Inference

## Anonymous Function Inference

```dart
// Parameter types inferred from context
var numbers = [1, 2, 3, 4, 5];
numbers.forEach((number) => print(number)); // number is int

// Return type inferred
var doubled = numbers.map((n) => n * 2); // n is int, returns int

// Function variable with inference
var add = (int a, int b) => a + b;      // (int, int) => int
var multiply = (a, b) => a * b;         // (int, int) => int
```

---

## Function Parameters

```dart
// Named parameters
void display({String name, int age}) {
  print('$name is $age years old');
}
// name inferred as String? (nullable)
// age inferred as int? (nullable)

// Required named parameters
void display({required String name, required int age}) {
  print('$name is $age years old');
}
// name and age are non-nullable
```

---

# Generic Type Inference

## Generic Functions

```dart
// Generic function without explicit type
T identity<T>(T value) {
  return value;
}

// Type inferred from arguments
var result = identity('Hello');    // T is String
var number = identity(42);          // T is int

// Explicit type annotation (optional)
var explicit = identity<String>('Hello');
```

---

## Generic Classes

```dart
class Box<T> {
  T value;
  Box(this.value);
}

// Type inferred from constructor argument
var stringBox = Box('Hello');    // Box<String>
var intBox = Box(42);            // Box<int>
var doubleBox = Box(3.14);       // Box<double>

// Explicit type (optional)
var explicitBox = Box<String>('Hello');
```

---

# Contextual Type Inference

## From Assignment Context

```dart
// Type inferred from target variable
List<String> names = ['Alice', 'Bob']; // List<String> inferred

// Type inferred from function parameter
void processNumbers(List<int> numbers) {
  // numbers is List<int>
}

processNumbers([1, 2, 3]); // List<int> inferred
```

---

## From Return Context

```dart
// Type inferred from return type
List<int> getNumbers() {
  return [1, 2, 3]; // List<int> inferred
}

Future<String> fetchData() async {
  return 'Data'; // Future<String> inferred
}
```

---

## From Collection Context

```dart
// Type inferred from generic context
List<int> numbers = [1, 2, 3]; // List<int>

Set<String> names = {'Alice', 'Bob'}; // Set<String>

Map<String, int> scores = {'Alice': 95}; // Map<String, int>
```

---

# Type Inference Limitations

## When Inference Fails

```dart
// Empty collections need explicit type
// var emptyList = [];          // List<dynamic> (bad)
var emptyList = <String>[];    // List<String> (good)

// Inference can be ambiguous
// var result = a > b ? 1 : 'two'; // Object (bad)
Object result = a > b ? 1 : 'two'; // Good: explicit type

// Complex type relationships
// var controller = StreamController(); // StreamController<dynamic> (bad)
var controller = StreamController<int>(); // Good
```

---

## Implicit Returns

```dart
// Returns null by default
getNull() {
  // No return statement -> returns null
}

// Better: Explicit return type
void getNull() {
  // Void return
}
```

---

# Best Practices

## Use `var` When Type is Obvious

```dart
// Good: Type is obvious
var name = 'Alice';            // String obvious
var numbers = [1, 2, 3];       // List<int> obvious
var config = {'host': 'localhost'}; // Map<String, String> obvious

// Bad: Type is not clear
var result = fetchData();      // What type is this?
// Better: Explicit type
Map<String, dynamic> result = fetchData();
```

---

## Use `final` for Unchanging Variables

```dart
// Good: Use final when value doesn't change
final name = 'Alice';
final age = 25;
final config = await loadConfig();

// Bad: Using var when final would be better
var name = 'Alice'; // Could be final
var age = 25;       // Could be final
```

---

## Explicit Type for Public APIs

```dart
// Good: Explicit return type for public APIs
class Calculator {
  int add(int a, int b) => a + b;
  int subtract(int a, int b) => a - b;
}

// Bad: Implicit return type for public APIs
class Calculator {
  add(a, b) => a + b;          // Return type unclear
}
```

---

# Common Mistakes

## Inferred Wrong Type

Wrong:

```dart
var number = '42'; // Inferred as String
var result = number + 10; // Error! String + int
```

Correct:

```dart
var number = 42; // Inferred as int
var result = number + 10; // Works
```

---

## Empty Collections

Wrong:

```dart
var emptyList = [];        // List<dynamic> (bad)
var emptySet = {};         // Map<dynamic, dynamic> (bad)
```

Correct:

```dart
var emptyList = <String>[];
var emptySet = <String>{};
var emptyMap = <String, int>{};
```

---

## Overusing Dynamic

Wrong:

```dart
dynamic data = 'Hello'; // Avoid dynamic when possible
```

Correct:

```dart
String data = 'Hello'; // Use specific types
// Or
Object data = 'Hello'; // Use Object if truly dynamic
```

---

# Type Inference vs Explicit Types

## When to Use Inference

```dart
// Local variables with obvious types
var name = 'Alice';
var count = 42;
var items = ['a', 'b', 'c'];

// Lambda expressions
items.forEach((item) => print(item));

// Generic instances
var box = Box('value');
```

---

## When to Use Explicit Types

```dart
// Public API surface
class Calculator {
  int add(int a, int b) => a + b;
}

// Complex types
Map<String, List<int>> data = processData();

// When type isn't obvious
Map<String, dynamic> config = loadConfig();

// For clarity and documentation
String getUserName() => fetchName();
```

---

# Summary

Type inference in Dart reduces boilerplate while maintaining type safety. Use `var` for local variables with obvious types, `final` for unchanging values, and explicit types for public APIs and complex scenarios. This balance makes code cleaner, safer, and more maintainable.

---

# Next Steps

Now that you understand type inference, continue to:

- [Operators](operators.md)
- [Control Flow](control-flow.md)
- [Functions](functions.md)
- [Collections](collections.md)

---

# Did You Know?

- Dart's type inference is context-sensitive and very smart
- Type inference works with generic types as well
- The analyzer can suggest where to add explicit types
- Type inference is performed at compile time, not runtime
- Inference works with both `var`, `final`, and `const`
- You can configure inference behavior with `analysis_options.yaml`
- Type inference has been greatly improved in Dart 3