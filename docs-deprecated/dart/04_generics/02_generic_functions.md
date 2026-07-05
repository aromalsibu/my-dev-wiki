# Generic Functions

Understand how to create and use generic functions in Dart.

---

# What is it?

Generic functions are functions that can work with different types while maintaining type safety. They allow you to write flexible, reusable functions that work with any type, with compile-time type checking.

---

# Why does it exist?

Generic functions exist to:

- Write type-safe, reusable functions
- Avoid code duplication
- Enable compile-time type checking
- Create flexible APIs
- Work with any type
- Provide better IDE support

---

# Basic Generic Functions

## Simple Generic Function

```dart
// Generic function with type parameter T
T identity<T>(T value) {
  return value;
}

// Usage
var stringResult = identity<String>('Hello');
var intResult = identity<int>(42);
var boolResult = identity<bool>(true);

print(stringResult); // Hello
print(intResult); // 42
print(boolResult); // true

// Type inference
var inferredResult = identity('World'); // Returns String
```

---

## Generic Function with Multiple Types

```dart
// Generic function with two type parameters
R transform<T, R>(T input, R Function(T) transformer) {
  return transformer(input);
}

// Usage
var stringToInt = transform<String, int>(
  '123',
  (s) => int.parse(s),
);

var intToString = transform<int, String>(
  42,
  (n) => n.toString(),
);

print(stringToInt); // 123
print(intToString); // 42

// Type inference
var inferred = transform(
  'Hello',
  (s) => s.length,
); // Returns int
```

---

# Generic Functions with Constraints

## Type Bounds

```dart
// Generic with upper bound (must be Comparable)
T maxValue<T extends Comparable<T>>(T a, T b) {
  return a.compareTo(b) > 0 ? a : b;
}

T findMax<T extends Comparable<T>>(List<T> items) {
  if (items.isEmpty) {
    throw ArgumentError('List cannot be empty');
  }
  return items.reduce((a, b) => a.compareTo(b) > 0 ? a : b);
}

// Usage
print(maxValue(5, 3)); // 5
print(maxValue('apple', 'banana')); // banana

print(findMax([1, 5, 3, 9, 2])); // 9
print(findMax(['apple', 'banana', 'cherry'])); // cherry

// Error: Cannot use non-Comparable type
// class NonComparable {}
// maxValue(NonComparable(), NonComparable()); // Error!
```

---

## Multiple Constraints

```dart
// Generic with multiple constraints
class Validatable {
  bool isValid();
}

class Serializeable {
  Map<String, dynamic> toJson();
}

// Function with constraints
R process<T extends Validatable & Serializeable, R>(
  T item,
  R Function(T) processor,
) {
  if (!item.isValid()) {
    throw ArgumentError('Invalid item');
  }
  return processor(item);
}

// Usage
class User implements Validatable, Serializeable {
  final String name;
  final int age;
  
  User(this.name, this.age);
  
  @override
  bool isValid() => name.isNotEmpty && age > 0;
  
  @override
  Map<String, dynamic> toJson() => {
    'name': name,
    'age': age,
  };
}

var user = User('Alice', 25);
var json = process<User, Map<String, dynamic>>(
  user,
  (u) => u.toJson(),
);

print(json); // {name: Alice, age: 25}
```

---

# Generic Functions in Collections

## Collection Processing

```dart
// Generic collection processing functions
List<R> mapCollection<T, R>(
  List<T> items,
  R Function(T) mapper,
) {
  return items.map(mapper).toList();
}

List<T> filterCollection<T>(
  List<T> items,
  bool Function(T) predicate,
) {
  return items.where(predicate).toList();
}

T? findFirst<T>(
  List<T> items,
  bool Function(T) predicate,
) {
  for (var item in items) {
    if (predicate(item)) {
      return item;
    }
  }
  return null;
}

// Usage
var numbers = [1, 2, 3, 4, 5, 6];

var doubled = mapCollection(numbers, (n) => n * 2);
print(doubled); // [2, 4, 6, 8, 10, 12]

var evens = filterCollection(numbers, (n) => n % 2 == 0);
print(evens); // [2, 4, 6]

var firstEven = findFirst(numbers, (n) => n % 2 == 0);
print(firstEven); // 2
```

---

## Generic Reducer

```dart
// Generic reduce function
R reduce<T, R>(
  List<T> items,
  R initialValue,
  R Function(R accumulator, T item) reducer,
) {
  var result = initialValue;
  for (var item in items) {
    result = reducer(result, item);
  }
  return result;
}

// Usage
var numbers = [1, 2, 3, 4, 5];

var sum = reduce<int, int>(
  numbers,
  0,
  (acc, n) => acc + n,
);
print(sum); // 15

var product = reduce<int, int>(
  numbers,
  1,
  (acc, n) => acc * n,
);
print(product); // 120

var concatenated = reduce<String, String>(
  ['Hello', ' ', 'World'],
  '',
  (acc, s) => acc + s,
);
print(concatenated); // Hello World
```

---

# Generic Functions as Parameters

## Higher-Order Generic Functions

```dart
// Function that takes a generic function
void processItems<T>(
  List<T> items,
  void Function(T) processor,
) {
  for (var item in items) {
    processor(item);
  }
}

// Function that returns a generic function
T Function(T) createTransformer<T>(
  String Function(T) description,
  T Function(T) transformer,
) {
  return (T value) {
    print('Transforming: ${description(value)}');
    return transformer(value);
  };
}

// Usage
var names = ['Alice', 'Bob', 'Charlie'];

// Passing generic function
processItems<String>(
  names,
  (name) => print('Hello, $name'),
);

// Creating transformer
var doubleTransformer = createTransformer<int>(
  (n) => n.toString(),
  (n) => n * 2,
);

print(doubleTransformer(5)); // Transforming: 5, returns 10
```

---

# Generic Async Functions

## Generic Future and Stream

```dart
// Generic async function
Future<T> fetchData<T>(
  String url,
  T Function(Map<String, dynamic>) parser,
) async {
  // Simulate network request
  await Future.delayed(Duration(seconds: 1));
  
  // Mock response
  var response = {'data': 'value'};
  return parser(response);
}

// Generic stream function
Stream<T> streamData<T>(
  int count,
  T Function(int) generator,
) async* {
  for (var i = 0; i < count; i++) {
    await Future.delayed(Duration(milliseconds: 500));
    yield generator(i);
  }
}

// Usage
Future<void> main() async {
  // Fetch string
  var stringData = await fetchData<String>(
    'api/data',
    (json) => json['data'] as String,
  );
  print(stringData); // value
  
  // Fetch int
  var intData = await fetchData<int>(
    'api/data',
    (json) => int.parse(json['data'] as String),
  );
  print(intData); // 0 (or error if value is not int)
  
  // Stream
  await for (var value in streamData<int>(
    5,
    (i) => i * i,
  )) {
    print(value); // 0, 1, 4, 9, 16
  }
}
```

---

# Generic Functions in Dart SDK

## Built-in Generic Functions

```dart
// Dart SDK has many generic functions
void main() {
  // List methods
  var numbers = [1, 2, 3, 4, 5];
  var doubled = numbers.map((n) => n * 2).toList();
  
  // Map methods
  var map = {'a': 1, 'b': 2};
  var transformed = map.map((key, value) => MapEntry(key, value * 2));
  
  // Future methods
  Future<int> future = Future.value(42);
  
  // Stream methods
  var stream = Stream.fromIterable([1, 2, 3]);
  
  // Type conversion
  var stringList = ['1', '2', '3'];
  var intList = stringList.map(int.parse).toList();
  
  // Generic functions in dart:core
  var list = List<String>.filled(3, '');
  var set = Set<int>.from([1, 2, 3]);
  var map2 = Map<String, int>.from({'a': 1, 'b': 2});
}
```

---

# Best Practices

## Use Generic Functions for Reusable Logic

```dart
// Good: Reusable generic function
T defaultIfNull<T>(T? value, T defaultValue) {
  return value ?? defaultValue;
}

// Usage
String? maybeName = null;
String name = defaultIfNull(maybeName, 'Guest');

int? maybeNumber = 5;
int number = defaultIfNull(maybeNumber, 0);
```

---

## Use Type Inference When Possible

```dart
// Good: Type inference
var max = maxValue(5, 3); // T inferred as int

// Good: Explicit when needed
var intMax = maxValue<int>(5, 3);
var stringMax = maxValue<String>('apple', 'banana');

// Bad: Unnecessary explicit
var unnecessary = maxValue<int>(5, 3); // Type could be inferred
```

---

# Common Mistakes

## Using Dynamic Instead of Generic

Wrong:

```dart
List<dynamic> process(List<dynamic> items) {
  return items.map((item) => item.toString()).toList();
  // Loses type safety
}
```

Correct:

```dart
List<String> process<T>(List<T> items) {
  return items.map((item) => item.toString()).toList();
}
```

---

## Forgetting Constraints

Wrong:

```dart
T maxValue<T>(T a, T b) {
  return a > b ? a : b; // Error: > not defined for T
}
```

Correct:

```dart
T maxValue<T extends Comparable<T>>(T a, T b) {
  return a.compareTo(b) > 0 ? a : b;
}
```

---

# Summary

Generic functions provide type-safe, reusable logic. They allow you to write functions that work with any type while maintaining compile-time type checking.

---

# Next Steps

Now that you understand generic functions, continue to:

- [Type Bounds](type-bounds.md)
- [Generic Methods](generic-methods.md)
- [Covariance](covariance.md)
- [Variance](variance.md)

---

# Did You Know?

- Generic functions were introduced in Dart 2.0
- Type parameters are resolved at compile time
- Generic functions help catch errors early
- Dart uses type inference for generics
- You can have multiple type parameters
- Generic functions work with collections
- Type bounds restrict what types can be used