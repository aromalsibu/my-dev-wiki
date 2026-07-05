# Null Safety

Understand Dart's sound null safety system and how it prevents null errors.

---

# What is it?

Null Safety is a feature in Dart that ensures variables cannot contain null values unless explicitly allowed. It provides sound type safety, meaning the compiler can guarantee that non-nullable variables will never be null at runtime. This eliminates null pointer exceptions, one of the most common sources of bugs in programming.

---

# Why does it exist?

Null Safety exists to:

- Eliminate null pointer exceptions (NullReferenceError)
- Make code more reliable and predictable
- Improve developer productivity by catching errors early
- Make APIs clearer about what values can be null
- Enable compiler optimizations
- Reduce debugging time spent on null errors

---

# Core Concepts

## Non-nullable Types

```dart
// All types are non-nullable by default
String name = 'Alice';     // Must have a value
int age = 25;             // Must have a value
bool isActive = true;      // Must have a value

// Cannot assign null
// String name = null;     // Error!
// int age = null;         // Error!
```

---

## Nullable Types

```dart
// Use '?' to make a type nullable
String? maybeName = 'Alice';
String? nullName = null;   // Can be null

int? maybeAge = 25;
int? nullAge = null;       // Can be null

// Can be assigned either a value or null
String? flexibleName = 'Bob';
flexibleName = null;       // Works!
```

---

## Type Promotion

```dart
// Dart promotes nullable types to non-nullable after null checks
String? maybeName = 'Alice';

if (maybeName != null) {
  // Here maybeName is promoted to String (non-nullable)
  print(maybeName.length); // Safe! No ! needed
}

// With ! (non-null assertion)
String definitelyName = maybeName!; // Assert not null
```

---

# Null-aware Operators

## Null-aware Access (?.)

```dart
String? maybeName = null;

// Access property only if not null
int? length = maybeName?.length; // null (safe)
String? upper = maybeName?.toUpperCase(); // null

// With actual value
String? name = 'Alice';
int? nameLength = name?.length; // 5
```

---

## Null Coalescing (??)

```dart
String? maybeName = null;

// Provide default value if null
String displayName = maybeName ?? 'Guest'; // 'Guest'

// Chain multiple fallbacks
String finalName = maybeName ?? nickname ?? 'Anonymous';
```

---

## Null Coalescing Assignment (??=)

```dart
String? maybeName = null;
String? nickname = null;

// Assign only if current value is null
maybeName ??= 'Alice'; // Assigns 'Alice'
nickname ??= 'Bob';    // Assigns 'Bob'
```

---

## Null-aware Cascade (?..)

```dart
class Person {
  String? name;
  int? age;
}

Person? person = Person()
  ?..name = 'Alice'
  ..age = 25;

// Equivalent to:
if (person != null) {
  person.name = 'Alice';
  person.age = 25;
}
```

---

# Working with Nullables

## Checking for Null

```dart
String? maybeName = 'Alice';

// Traditional null check
if (maybeName != null) {
  print('Name: $maybeName');
} else {
  print('No name provided');
}

// Using ?? operator
print('Name: ${maybeName ?? "Unknown"}');

// Using ! (only when absolutely sure)
print('Name: ${maybeName!}');
```

---

## Handling Null in Functions

```dart
// Function with nullable parameter
void greet(String? name) {
  // Safe handling
  String displayName = name ?? 'Guest';
  print('Hello, $displayName');
  
  // Or using conditional logic
  if (name != null) {
    print('Hello, $name (length: ${name.length})');
  }
}

// Call with null
greet(null); // Hello, Guest
greet('Alice'); // Hello, Alice (length: 5)
```

---

## Returning Null

```dart
// Function that can return null
String? findName(int id) {
  if (id == 1) {
    return 'Alice';
  }
  return null; // Return null for non-existent IDs
}

// Usage
String? name = findName(42);
if (name != null) {
  print('Found: $name');
} else {
  print('Not found');
}
```

---

# Initialization

## Late Initialization

```dart
// Declare without initial value
late String description;
late int userId;

// Initialize later
void initUser(int id) {
  userId = id;
  description = 'User #$id';
}

// Access after initialization
void displayUser() {
  print('User: $userId, Description: $description');
}

// Late final (assigned once)
late final String name;
name = 'Alice';
// name = 'Bob'; // Error! Cannot reassign final
```

---

## Required Parameters

```dart
// Required non-nullable parameters
void greet({required String name, required int age}) {
  print('$name is $age years old');
}

// Usage
greet(name: 'Alice', age: 25); // Works

// Required nullable parameters
void display({required String? nickname}) {
  print('Nickname: ${nickname ?? "None"}');
}

// Can pass null explicitly
display(nickname: null); // Works
```

---

# Collections and Null Safety

## Lists with Nullable Types

```dart
// List of nullable strings
List<String?> items = ['Apple', null, 'Banana'];

// List of non-nullable strings
List<String> nonNullItems = ['Apple', 'Banana'];
// Cannot add null: nonNullItems.add(null); // Error!

// Filtering out nulls
var validItems = items.where((item) => item != null).toList();
```

---

## Maps with Nullable Values

```dart
// Map with nullable values
Map<String, int?> scores = {
  'Alice': 95,
  'Bob': null,  // Bob hasn't taken the test yet
  'Charlie': 87,
};

// Safe access
int? aliceScore = scores['Alice'];
int? bobScore = scores['Bob']; // null

// Provide default
int displayScore = scores['Alice'] ?? 0;
```

---

## Working with Iterables

```dart
List<String?> names = ['Alice', null, 'Bob', 'Charlie'];

// Filter out nulls
var nonNullNames = names.whereType<String>().toList();
// ['Alice', 'Bob', 'Charlie']

// Map with null safety
var upperNames = names.map((name) => name?.toUpperCase()).toList();
// ['ALICE', null, 'BOB', 'CHARLIE']
```

---

# Async and Null Safety

## Future and Null

```dart
// Future that can return null
Future<String?> fetchName() async {
  await Future.delayed(Duration(seconds: 1));
  return 'Alice'; // Or return null;
}

// Handling in async code
void main() async {
  String? name = await fetchName();
  print('Name: ${name ?? "Unknown"}');
}
```

---

## Stream and Null Safety

```dart
// Stream that emits nullable values
Stream<String?> getNameStream() async* {
  yield 'Alice';
  yield null;
  yield 'Bob';
}

// Handling nullable values
void main() async {
  await for (String? name in getNameStream()) {
    if (name != null) {
      print('Name: $name');
    } else {
      print('Name is null');
    }
  }
}
```

---

# Best Practices

## Prefer Non-nullable Types

```dart
// Better: Non-nullable with default
class User {
  String name = 'Guest';
  int age = 0;
}

// Worse: Nullable with default handling
class User {
  String? name;
  int? age;
}
```

---

## Use Nullable Types Intentionally

```dart
// Clear intention that value might be missing
class ApiResponse {
  final String? error;  // Null means no error
  final Data? data;     // Null when error occurred
  
  ApiResponse({this.error, this.data});
}
```

---

## Avoid Using ! When Possible

```dart
// Bad: Asserting without checking
String? maybeName = getUserName();
String name = maybeName!; // Risky!

// Good: Check before asserting
if (maybeName != null) {
  String name = maybeName;
}

// Better: Use null-aware operators
String name = maybeName ?? 'Guest';
```

---

# Common Mistakes

## Null Assertion on Maybe Null

Wrong:

```dart
String? maybeName = null;
String name = maybeName!; // Runtime error!
```

Correct:

```dart
String? maybeName = null;
String name = maybeName ?? 'Guest';
```

---

## Forgetting Null Check

Wrong:

```dart
String? name = getUserName();
print(name.length); // Error! Might be null
```

Correct:

```dart
String? name = getUserName();
if (name != null) {
  print(name.length);
}
```

---

## Using Nullable Where Not Needed

Wrong:

```dart
class Person {
  String? name = 'Alice'; // Unnecessarily nullable
}
```

Correct:

```dart
class Person {
  String name = 'Alice'; // Non-nullable
}
```

---

# Migration Guide

## From Non-null Safe to Null Safe

```dart
// Before null safety
class User {
  String name;
  int age;
  
  User(this.name, this.age);
}

// After null safety
class User {
  String name;  // Must be initialized
  int age;
  
  User(this.name, this.age);
}

// If values might be missing
class User {
  String? name;
  int? age;
  
  User({this.name, this.age});
}
```

---

# Summary

Null Safety in Dart provides compile-time guarantees that variables won't be null unless explicitly allowed. This eliminates null pointer exceptions and makes code more reliable, predictable, and easier to maintain.

---

# Next Steps

Now that you understand null safety, continue to:

- [Type System](type-system.md)
- [Type Inference](type-inference.md)
- [Operators](operators.md)
- [Control Flow](control-flow.md)

---

# Did You Know?

- Dart's null safety is sound - the compiler guarantees it
- Billions of dollars are lost annually to null reference errors
- Kotlin's null safety inspired Dart's implementation
- Java's null safety is optional and not sound
- TypeScript's null safety is not sound (can be bypassed)
- Dart was the first major language to implement sound null safety
- The migration to null safety was the biggest change in Dart's history