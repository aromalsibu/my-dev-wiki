# Functions

Understand how to define and use functions in Dart.

---

# What is it?

Functions are reusable blocks of code that perform specific tasks. In Dart, functions are first-class objects, meaning they can be assigned to variables, passed as arguments, and returned from other functions. Functions help organize code, reduce duplication, and make programs more modular.

---

# Why does it exist?

Functions exist to:

- Encapsulate reusable logic
- Break complex problems into smaller pieces
- Reduce code duplication
- Make code more readable and maintainable
- Enable abstraction and modularity
- Support functional programming patterns

---

# Function Declaration

## Basic Function

```dart
// Function with return type and parameters
int add(int a, int b) {
  return a + b;
}

// Void function (no return value)
void greet(String name) {
  print('Hello, $name');
}

// Function without return type (returns dynamic)
addNumbers(a, b) {
  return a + b;
}
```

---

## Arrow Functions

```dart
// Single expression function
int multiply(int a, int b) => a * b;

// Multiple expressions need curly braces
int calculate(int a, int b) {
  int sum = a + b;
  return sum * 2;
}

// Arrow function with return type inference
greet(String name) => 'Hello, $name';

// Arrow function as void
printHello() => print('Hello');
```

---

# Parameters

## Required Positional Parameters

```dart
// Required parameters
void display(String name, int age) {
  print('$name is $age years old');
}

// Usage
display('Alice', 25); // name and age are required
```

---

## Optional Positional Parameters

```dart
// Optional positional parameters (in square brackets)
void display(String name, [int? age, String? city]) {
  print('Name: $name');
  if (age != null) print('Age: $age');
  if (city != null) print('City: $city');
}

// Usage
display('Alice');                    // Only name
display('Alice', 25);                // Name and age
display('Alice', 25, 'New York');    // All parameters

// With default values
void display(String name, [int age = 0, String city = 'Unknown']) {
  print('$name, $age, $city');
}
```

---

## Named Parameters

```dart
// Named parameters (in curly braces)
void display({String name = 'Guest', int age = 0}) {
  print('$name is $age years old');
}

// Usage
display(name: 'Alice', age: 25);
display(age: 25, name: 'Alice'); // Order doesn't matter
display(name: 'Alice');          // age uses default

// Required named parameters
void display({required String name, required int age}) {
  print('$name is $age years old');
}

// Required named parameters must be provided
display(name: 'Alice', age: 25); // Works
display(name: 'Alice');          // Error! age is required
```

---

## Mixed Parameters

```dart
// Positional + Named parameters
void display(String name, {required int age, String? city}) {
  print('$name, $age, $city');
}

// Usage
display('Alice', age: 25);
display('Alice', age: 25, city: 'NYC');

// Multiple named parameters
void configure({bool debug = false, int timeout = 30, bool cache = true}) {
  print('Debug: $debug, Timeout: $timeout, Cache: $cache');
}

// Usage
configure(debug: true);
configure(timeout: 60, cache: false);
```

---

# Return Types

## Return Values

```dart
// Returns int
int add(int a, int b) {
  return a + b;
}

// Returns String
String greet(String name) {
  return 'Hello, $name';
}

// Returns bool
bool isEven(int number) {
  return number % 2 == 0;
}

// Returns Future (async function)
Future<String> fetchData() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Data loaded';
}

// Returns Stream
Stream<int> countStream() async* {
  for (var i = 0; i < 5; i++) {
    yield i;
  }
}
```

---

## Void Return

```dart
// Void function (returns nothing)
void sayHello() {
  print('Hello');
}

// Void function with implicit return
void doSomething() {
  // Does something, no return statement needed
}

// Void arrow function
void printMessage(String msg) => print(msg);
```

---

## No Return (dynamic)

```dart
// No explicit return type (returns dynamic)
getValue() {
  return 42; // Could return anything
}

// Better: Use explicit types
int getValue() {
  return 42;
}
```

---

# Anonymous Functions

## Lambda Expressions

```dart
// Assign to variable
var add = (int a, int b) {
  return a + b;
};

// Arrow syntax
var multiply = (int a, int b) => a * b;

// Usage
var result = add(3, 5);  // 8
var product = multiply(4, 6); // 24
```

---

## Inline Functions

```dart
// Passing anonymous function
List<int> numbers = [1, 2, 3, 4, 5];

// With curly braces
numbers.forEach((number) {
  print(number);
});

// Arrow syntax
numbers.forEach((number) => print(number));

// With type annotation
numbers.forEach((int number) {
  print(number * 2);
});
```

---

## Functional Programming

```dart
// Map with anonymous function
var doubled = numbers.map((n) => n * 2).toList();

// Filter with anonymous function
var evens = numbers.where((n) => n % 2 == 0).toList();

// Reduce with anonymous function
var sum = numbers.reduce((a, b) => a + b);

// Fold with anonymous function
var product = numbers.fold(1, (a, b) => a * b);
```

---

# Function Types

## Type Annotations

```dart
// Function type as variable type
int Function(int, int) add = (a, b) => a + b;

// Function type as parameter type
void execute(int Function(int, int) operation, int a, int b) {
  print(operation(a, b));
}

// Function type as return type
int Function(int, int) getOperation(String type) {
  if (type == 'add') {
    return (a, b) => a + b;
  } else {
    return (a, b) => a - b;
  }
}
```

---

## Typedef

```dart
// Define function type
typedef IntOperation = int Function(int a, int b);

// Using typedef
IntOperation add = (a, b) => a + b;
IntOperation multiply = (a, b) => a * b;

// Typedef with generics
typedef Operation<T> = T Function(T a, T b);
Operation<int> addInts = (a, b) => a + b;
Operation<double> addDoubles = (a, b) => a + b;

// Complex typedefs
typedef AsyncProcessor<T> = Future<T> Function(T input);
typedef Callback<T> = void Function(T result, String? error);
```

---

# Closures

## Lexical Scope

```dart
// Closures capture variables from their scope
Function makeAdder(int addBy) {
  return (int value) => value + addBy; // Captures addBy
}

var add5 = makeAdder(5);
var result = add5(10); // 15

// Multiple closures with different captured values
var add10 = makeAdder(10);
print(add5(3));  // 8
print(add10(3)); // 13
```

---

## Stateful Closures

```dart
// Closure maintaining state
Function counter() {
  int count = 0;
  return () {
    count++;
    return count;
  };
}

var myCounter = counter();
print(myCounter()); // 1
print(myCounter()); // 2
print(myCounter()); // 3

// Multiple independent counters
var counter1 = counter();
var counter2 = counter();
print(counter1()); // 1
print(counter2()); // 1
print(counter1()); // 2
```

---

# Higher-Order Functions

## Functions as Parameters

```dart
// Function that takes another function
void processItems(List<int> items, int Function(int) processor) {
  for (var item in items) {
    print(processor(item));
  }
}

// Usage
processItems([1, 2, 3], (n) => n * 2);

// Using named parameters
void withTimeout(Future<void> Function() operation, {int seconds = 5}) {
  // Execute operation with timeout
}
```

---

## Functions as Return Values

```dart
// Function returning another function
Function(String) getGreeter(String greeting) {
  return (String name) => '$greeting, $name';
}

// Usage
var sayHello = getGreeter('Hello');
var sayHi = getGreeter('Hi');
print(sayHello('Alice')); // 'Hello, Alice'
print(sayHi('Bob'));      // 'Hi, Bob'
```

---

# Async Functions

## Async/Await

```dart
// Async function returns Future
Future<String> fetchData() async {
  await Future.delayed(Duration(seconds: 2));
  return 'Data loaded';
}

// Using async function
void main() async {
  print('Loading...');
  String data = await fetchData();
  print(data);
}

// Async with error handling
Future<void> fetchUser() async {
  try {
    var user = await api.getUser();
    print('User: $user');
  } catch (e) {
    print('Error: $e');
  }
}
```

---

## Stream Functions

```dart
// Async generator (returns Stream)
Stream<int> countDown(int start) async* {
  for (var i = start; i >= 0; i--) {
    await Future.delayed(Duration(seconds: 1));
    yield i; // Emit value
  }
}

// Usage
void main() async {
  await for (var value in countDown(5)) {
    print(value);
  }
}
```

---

## Sync Generator

```dart
// Sync generator (returns Iterable)
Iterable<int> countUp(int end) sync* {
  for (var i = 0; i <= end; i++) {
    yield i;
  }
}

// Usage
for (var value in countUp(3)) {
  print(value); // 0, 1, 2, 3
}
```

---

# Best Practices

## Naming Conventions

```dart
// Use verb + noun for function names
void calculateTotal() { /* ... */ }
String getUserName() { /* ... */ }
bool isValidEmail(String email) { /* ... */ }

// Use descriptive names
// Bad: void process() { /* ... */ }
// Good: void processUserOrder() { /* ... */ }

// Use consistent naming
Future<User> fetchUser() async { /* ... */ }
Stream<String> getMessages() async* { /* ... */ }
Iterable<int> generateNumbers() sync* { /* ... */ }
```

---

## Single Responsibility

```dart
// Bad: Does too much
void processUserData(user) {
  validateUser(user);
  saveUser(user);
  sendWelcomeEmail(user);
  updateAnalytics(user);
}

// Good: Single responsibility
void processUserData(user) {
  validateUser(user);
  saveUser(user);
  sendWelcomeEmail(user);
  updateAnalytics(user);
}

// Even better: Separate concerns
class UserProcessor {
  final UserValidator validator;
  final UserRepository repository;
  final EmailService emailService;
  final AnalyticsService analytics;
  
  void processUser(User user) {
    validator.validate(user);
    repository.save(user);
    emailService.sendWelcome(user);
    analytics.trackUser(user);
  }
}
```

---

## Avoid Side Effects

```dart
// Bad: Function has side effects
int count = 0;
int addWithSideEffect(int x) {
  count++; // Modifies external state
  return x + 1;
}

// Good: Pure function
int add(int x) {
  return x + 1; // No side effects
}

// Better: Return new state
(int newCount, int result) processValue(int count, int x) {
  return (count + 1, x + 1);
}
```

---

# Common Mistakes

## Parameter Order

Wrong:

```dart
void display(String name, int age, [String? city]) {
  // ...
}
display('Alice', 25, 'NYC'); // Works
display('Alice', 'NYC', 25); // Wrong order
```

Correct:

```dart
void display(String name, int age, [String? city]) {
  // ...
}
display('Alice', 25, 'NYC'); // Correct order
display('Alice', age: 25, city: 'NYC'); // Named for clarity
```

---

## Missing Return

Wrong:

```dart
int add(int a, int b) {
  // Missing return
}
```

Correct:

```dart
int add(int a, int b) {
  return a + b;
}
```

---

## Async Return Type

Wrong:

```dart
String fetchData() async { // Wrong: returns Future<String>
  await Future.delayed(Duration(seconds: 1));
  return 'Data';
}
```

Correct:

```dart
Future<String> fetchData() async { // Correct
  await Future.delayed(Duration(seconds: 1));
  return 'Data';
}
```

---

# Summary

Functions are the building blocks of Dart programs. They encapsulate logic, promote code reuse, and enable abstraction. Understanding function types, parameters, and best practices helps you write cleaner, more maintainable code.

---

# Next Steps

Now that you understand functions, continue to:

- [Collections](collections.md)
- [Records](records.md)
- [Patterns](patterns.md)
- [Classes](classes.md)

---

# Did You Know?

- Functions are first-class objects in Dart
- Dart supports both positional and named parameters
- Arrow functions can only contain a single expression
- Typedefs can be used to create aliases for function types
- Closures capture variables from their lexical scope
- Dart supports both synchronous and asynchronous functions
- Functions can be passed as arguments to other functions