# Generic Methods

Understand how to define and use generic methods in Dart classes and interfaces.

---

# What is it?

Generic methods are methods that can work with different types while maintaining type safety. Unlike generic classes that are parameterized at the class level, generic methods introduce type parameters at the method level, allowing for more flexible and reusable code.

---

# Why does it exist?

Generic methods exist to:

- Write type-safe, reusable methods
- Avoid code duplication
- Enable compile-time type checking
- Create flexible APIs
- Work with any type
- Provide better IDE support

---

# Generic Methods in Classes

## Basic Generic Methods

```dart
class CollectionUtils {
  // Generic method in non-generic class
  T identity<T>(T value) {
    return value;
  }
  
  List<R> mapList<T, R>(List<T> items, R Function(T) mapper) {
    return items.map(mapper).toList();
  }
  
  T? findFirst<T>(List<T> items, bool Function(T) predicate) {
    for (var item in items) {
      if (predicate(item)) {
        return item;
      }
    }
    return null;
  }
}

// Usage
var utils = CollectionUtils();
var numbers = [1, 2, 3, 4, 5];

// Identity
print(utils.identity('Hello')); // Hello
print(utils.identity(42)); // 42

// Map
var doubled = utils.mapList(numbers, (n) => n * 2);
print(doubled); // [2, 4, 6, 8, 10]

// Find
var firstEven = utils.findFirst(numbers, (n) => n % 2 == 0);
print(firstEven); // 2
```

---

## Generic Methods in Generic Classes

```dart
class Repository<T> {
  final Map<String, T> _data = {};
  
  // Generic method with different type
  R process<R>(T item, R Function(T) processor) {
    return processor(item);
  }
  
  // Generic method returning different type
  List<R> mapTo<R>(R Function(T) mapper) {
    return _data.values.map(mapper).toList();
  }
  
  // Generic method with type parameter
  U? findAndTransform<U>(String id, U Function(T) transformer) {
    var item = _data[id];
    if (item != null) {
      return transformer(item);
    }
    return null;
  }
  
  // Method with generic constraint
  U findMax<U extends Comparable<U>>(U Function(T) extractor) {
    if (_data.isEmpty) {
      throw StateError('Repository is empty');
    }
    return _data.values
        .map(extractor)
        .reduce((a, b) => a.compareTo(b) > 0 ? a : b);
  }
}

// Usage
class User {
  final String name;
  final int age;
  
  User(this.name, this.age);
}

var repo = Repository<User>();
repo.save('1', User('Alice', 25));
repo.save('2', User('Bob', 30));
repo.save('3', User('Charlie', 20));

// Process method
var info = repo.process(
  User('Alice', 25),
  (user) => '${user.name} is ${user.age}',
);
print(info); // Alice is 25

// Map to method
var names = repo.mapTo((user) => user.name);
print(names); // [Alice, Bob, Charlie]

// Find and transform
var age = repo.findAndTransform('2', (user) => user.age);
print(age); // 30

// Find max with constraint
var maxAge = repo.findMax((user) => user.age);
print(maxAge); // 30
```

---

# Generic Methods with Constraints

## Bounded Generic Methods

```dart
class Utilities {
  // Generic method with upper bound
  T maxValue<T extends Comparable<T>>(T a, T b) {
    return a.compareTo(b) > 0 ? a : b;
  }
  
  // Generic method with multiple constraints
  T process<T extends Object & Comparable<T>>(List<T> items) {
    if (items.isEmpty) {
      throw ArgumentError('List cannot be empty');
    }
    return items.reduce((a, b) => a.compareTo(b) > 0 ? a : b);
  }
  
  // Generic method with type bound
  List<T> sortList<T extends Comparable<T>>(List<T> items) {
    var sorted = List<T>.from(items);
    sorted.sort();
    return sorted;
  }
}

// Usage
var utils = Utilities();
print(utils.maxValue(5, 3)); // 5
print(utils.maxValue('apple', 'banana')); // banana

var numbers = [3, 1, 4, 1, 5, 9, 2];
print(utils.sortList(numbers)); // [1, 1, 2, 3, 4, 5, 9]
print(utils.process(numbers)); // 9

// Error: Cannot use non-Comparable
// class NonComparable {}
// utils.maxValue(NonComparable(), NonComparable()); // Error!
```

---

# Generic Methods in Interfaces

## Interface with Generic Methods

```dart
// Interface with generic methods
abstract class DataTransformer {
  // Generic method in interface
  T transform<T>(T input);
  
  // Generic method with constraints
  R process<T, R>(T input, R Function(T) processor);
  
  // Generic method with different return type
  List<R> transformList<T, R>(List<T> items, R Function(T) mapper);
}

class StringTransformer implements DataTransformer {
  @override
  T transform<T>(T input) {
    // Transform based on type
    if (input is String) {
      return (input as String).toUpperCase() as T;
    } else if (input is int) {
      return (input * 2) as T;
    }
    return input;
  }
  
  @override
  R process<T, R>(T input, R Function(T) processor) {
    return processor(input);
  }
  
  @override
  List<R> transformList<T, R>(List<T> items, R Function(T) mapper) {
    return items.map(mapper).toList();
  }
}

// Usage
var transformer = StringTransformer();

// Transform method
print(transformer.transform<String>('hello')); // HELLO
print(transformer.transform<int>(5)); // 10

// Process method
var result = transformer.process<String, int>(
  '123',
  (s) => int.parse(s),
);
print(result); // 123

// Transform list
var numbers = [1, 2, 3];
var doubled = transformer.transformList<int, int>(
  numbers,
  (n) => n * 2,
);
print(doubled); // [2, 4, 6]
```

---

# Generic Methods with Type Inference

## Leveraging Inference

```dart
class Processor {
  // Methods where types can be inferred
  R processValue<T, R>(T value, R Function(T) processor) {
    return processor(value);
  }
  
  // Multiple parameters with different types
  R combine<T, U, R>(
    T first,
    U second,
    R Function(T, U) combiner,
  ) {
    return combiner(first, second);
  }
  
  // Nested generic types
  List<R> processNested<T, R>(
    List<List<T>> nested,
    R Function(List<T>) processor,
  ) {
    return nested.map(processor).toList();
  }
}

// Usage
var processor = Processor();

// Type inference works
var length = processor.processValue(
  'Hello',
  (s) => s.length,
);
print(length); // 5

// Multiple types
var combined = processor.combine<String, int, String>(
  'Age: ',
  25,
  (s, n) => '$s$n',
);
print(combined); // Age: 25

// Nested processing
var matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];
var sums = processor.processNested(
  matrix,
  (row) => row.reduce((a, b) => a + b),
);
print(sums); // [6, 15, 24]
```

---

# Generic Methods in Practice

## Real-World Examples

```dart
// 1. JSON serialization helper
class JsonHelper {
  static T fromJson<T>(Map<String, dynamic> json, T Function(Map<String, dynamic>) parser) {
    return parser(json);
  }
  
  static List<T> fromJsonList<T>(
    List<Map<String, dynamic>> jsonList,
    T Function(Map<String, dynamic>) parser,
  ) {
    return jsonList.map(parser).toList();
  }
  
  static Map<String, dynamic> toJson<T>(
    T object,
    Map<String, dynamic> Function(T) serializer,
  ) {
    return serializer(object);
  }
}

// 2. Validation helper
class Validator {
  static bool validate<T>(
    T value,
    bool Function(T) rule,
  ) {
    return rule(value);
  }
  
  static List<T> filterValid<T>(
    List<T> items,
    bool Function(T) rule,
  ) {
    return items.where(rule).toList();
  }
  
  static bool validateAll<T>(
    List<T> items,
    bool Function(T) rule,
  ) {
    return items.every(rule);
  }
}

// 3. Cache helper
class Cache<T> {
  final Map<String, T> _cache = {};
  final Duration _ttl;
  
  Cache({Duration ttl = const Duration(minutes: 5)}) : _ttl = ttl;
  
  void put(String key, T value) {
    _cache[key] = value;
  }
  
  T? get(String key) {
    return _cache[key];
  }
  
  // Generic method for transforming cache
  U getOrCompute<U>(
    String key,
    U Function(T?) compute,
  ) {
    var cached = _cache[key] as T?;
    var result = compute(cached);
    if (result != null && result is T) {
      _cache[key] = result;
    }
    return result;
  }
}

// Usage
// 1. JSON
var json = {'name': 'Alice', 'age': 25};
var user = JsonHelper.fromJson(json, (j) => User(j['name'] as String, j['age'] as int));

// 2. Validation
var isValid = Validator.validate('test@example.com', (s) => s.contains('@'));
var filtered = Validator.filterValid([1, 2, 3, 4, 5], (n) => n % 2 == 0);

// 3. Cache
var cache = Cache<String>();
cache.put('key', 'value');
var cachedValue = cache.get('key');
```

---

# Generic Method Best Practices

## Type Safety

```dart
// Good: Type-safe generic method
List<R> safeTransform<T, R>(
  List<T> items,
  R Function(T) transformer,
) {
  return items.map(transformer).toList();
}

// Bad: Unnecessary dynamic
List<dynamic> unsafeTransform(List items, Function transformer) {
  return items.map(transformer).toList(); // Loses type safety
}
```

---

## Appropriate Use

```dart
// Good: Useful generic method
class StringUtils {
  static T parse<T>(String value, T Function(String) parser) {
    return parser(value);
  }
}

// Bad: Overly generic
class Utils {
  static T identity<T>(T value) {
    return value;
  }
}
```

---

# Common Mistakes

## Missing Type Information

Wrong:

```dart
void processItems(List items) {
  // No type information
  for (var item in items) {
    print(item);
  }
}
```

Correct:

```dart
void processItems<T>(List<T> items) {
  // Type information preserved
  for (var item in items) {
    print(item);
  }
}
```

---

## Unnecessary Generic Methods

Wrong:

```dart
class Utils {
  T wrap<T>(T value) {
    return value; // Unnecessary - just returns the value
  }
}
```

Correct:

```dart
class Utils {
  // Only generic when actually needed
  List<R> transform<T, R>(List<T> items, R Function(T) transformer) {
    return items.map(transformer).toList();
  }
}
```

---

# Summary

Generic methods provide type-safe, reusable functionality at the method level. They allow for flexible APIs while maintaining compile-time type safety.

---

# Next Steps

Now that you understand generic methods, continue to:

- [Covariance](covariance.md)
- [Variance](variance.md)
- [Async Programming](async-programming.md)
- [Future](future.md)

---

# Did You Know?

- Generic methods were introduced in Dart 2.0
- Type parameters are resolved at compile time
- Methods can have different type parameters than their class
- Generic methods enable functional programming patterns
- Type inference works with generic methods
- Generic methods can have constraints
- Generic methods are used throughout the Dart SDK