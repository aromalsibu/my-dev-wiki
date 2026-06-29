# Generic Classes

Understand how to create and use generic classes in Dart.

---

# What is it?

Generic classes are classes that can work with different types while maintaining type safety. They allow you to write flexible, reusable code that works with any type, with compile-time type checking.

---

# Why does it exist?

Generic classes exist to:

- Write type-safe, reusable code
- Avoid code duplication
- Enable compile-time type checking
- Create flexible APIs
- Work with collections of any type
- Provide better IDE support

---

# Basic Generic Classes

## Simple Generic Class

```dart
// Generic class with type parameter T
class Box<T> {
  T value;
  
  Box(this.value);
  
  T getValue() => value;
  
  void setValue(T newValue) {
    value = newValue;
  }
  
  @override
  String toString() => 'Box($value)';
}

// Usage with different types
var stringBox = Box<String>('Hello');
var intBox = Box<int>(42);
var doubleBox = Box<double>(3.14);

print(stringBox.getValue()); // Hello
print(intBox.getValue()); // 42
print(doubleBox.getValue()); // 3.14

// Type inference
var inferredBox = Box('Hello'); // Box<String>
```

---

## Generic Class with Multiple Types

```dart
// Generic class with two type parameters
class Pair<K, V> {
  final K key;
  final V value;
  
  Pair(this.key, this.value);
  
  @override
  String toString() => 'Pair($key: $value)';
}

// Usage
var pair1 = Pair<String, int>('age', 25);
var pair2 = Pair<String, String>('name', 'Alice');
var pair3 = Pair<int, bool>(1, true);

print(pair1); // Pair(age: 25)
print(pair2); // Pair(name: Alice)
print(pair3); // Pair(1: true)

// Type inference
var inferredPair = Pair('score', 95); // Pair<String, int>
```

---

# Generic Classes with Constraints

## Type Bounds

```dart
// Generic with upper bound (must be Comparable)
class MaxFinder<T extends Comparable<T>> {
  T findMax(T a, T b) {
    return a.compareTo(b) > 0 ? a : b;
  }
  
  T findMaxInList(List<T> items) {
    if (items.isEmpty) {
      throw ArgumentError('List cannot be empty');
    }
    return items.reduce((a, b) => a.compareTo(b) > 0 ? a : b);
  }
}

// Usage
var intFinder = MaxFinder<int>();
print(intFinder.findMax(5, 3)); // 5
print(intFinder.findMaxInList([1, 5, 3, 9, 2])); // 9

var stringFinder = MaxFinder<String>();
print(stringFinder.findMax('apple', 'banana')); // banana

// Error: Cannot use non-Comparable type
// class NonComparable {}
// var badFinder = MaxFinder<NonComparable>(); // Error!
```

---

## Multiple Constraints

```dart
// Generic with multiple constraints
class Repository<T extends Object> {
  final Map<String, T> _data = {};
  
  void save(String id, T item) {
    _data[id] = item;
  }
  
  T? find(String id) {
    return _data[id];
  }
  
  List<T> findAll() {
    return _data.values.toList();
  }
  
  void delete(String id) {
    _data.remove(id);
  }
  
  bool exists(String id) {
    return _data.containsKey(id);
  }
}

// Usage with any Object type
class User {
  final String name;
  User(this.name);
  
  @override
  String toString() => 'User($name)';
}

class Product {
  final String name;
  final double price;
  Product(this.name, this.price);
  
  @override
  String toString() => 'Product($name, $price)';
}

var userRepo = Repository<User>();
userRepo.save('1', User('Alice'));
userRepo.save('2', User('Bob'));

var productRepo = Repository<Product>();
productRepo.save('P1', Product('Laptop', 999.99));
productRepo.save('P2', Product('Phone', 599.99));

print(userRepo.findAll()); // [User(Alice), User(Bob)]
print(productRepo.findAll()); // [Product(Laptop, 999.99), Product(Phone, 599.99)]
```

---

# Generic Collection Classes

## Custom Generic Collection

```dart
class Stack<T> {
  final List<T> _items = [];
  
  void push(T item) {
    _items.add(item);
  }
  
  T pop() {
    if (_items.isEmpty) {
      throw StateError('Stack is empty');
    }
    return _items.removeLast();
  }
  
  T peek() {
    if (_items.isEmpty) {
      throw StateError('Stack is empty');
    }
    return _items.last;
  }
  
  bool get isEmpty => _items.isEmpty;
  bool get isNotEmpty => _items.isNotEmpty;
  int get length => _items.length;
  
  void clear() {
    _items.clear();
  }
  
  @override
  String toString() => 'Stack($_items)';
}

// Usage
var intStack = Stack<int>();
intStack.push(1);
intStack.push(2);
intStack.push(3);
print(intStack); // Stack([1, 2, 3])
print(intStack.pop()); // 3
print(intStack.peek()); // 2
print(intStack.length); // 2

var stringStack = Stack<String>();
stringStack.push('Hello');
stringStack.push('World');
print(stringStack.pop()); // World
```

---

## Generic Queue

```dart
class Queue<T> {
  final List<T> _items = [];
  
  void enqueue(T item) {
    _items.add(item);
  }
  
  T dequeue() {
    if (_items.isEmpty) {
      throw StateError('Queue is empty');
    }
    return _items.removeAt(0);
  }
  
  T peek() {
    if (_items.isEmpty) {
      throw StateError('Queue is empty');
    }
    return _items.first;
  }
  
  bool get isEmpty => _items.isEmpty;
  bool get isNotEmpty => _items.isNotEmpty;
  int get length => _items.length;
  
  @override
  String toString() => 'Queue($_items)';
}

// Usage
var queue = Queue<String>();
queue.enqueue('First');
queue.enqueue('Second');
queue.enqueue('Third');

print(queue.dequeue()); // First
print(queue.peek()); // Second
print(queue.length); // 2
```

---

# Generic Class with Methods

## Generic Methods in Generic Class

```dart
class CollectionUtils<T> {
  final List<T> _items = [];
  
  // Generic method with same type
  void add(T item) {
    _items.add(item);
  }
  
  void addAll(Iterable<T> items) {
    _items.addAll(items);
  }
  
  // Generic method with different type
  R map<R>(R Function(T) mapper) {
    return mapper(_items.first);
  }
  
  // Generic method returning new collection
  List<R> mapToList<R>(R Function(T) mapper) {
    return _items.map(mapper).toList();
  }
  
  // Method with type parameter
  List<U> filter<U extends T>(bool Function(T) predicate) {
    return _items.where(predicate).cast<U>().toList();
  }
  
  @override
  String toString() => 'CollectionUtils($_items)';
}

// Usage
var utils = CollectionUtils<String>();
utils.addAll(['apple', 'banana', 'cherry']);

var lengths = utils.mapToList((item) => item.length);
print(lengths); // [5, 6, 6]

var filtered = utils.filter((item) => item.startsWith('b'));
print(filtered); // [banana]
```

---

# Best Practices

## Use Descriptive Type Parameter Names

```dart
// Good: Descriptive names
class Repository<Entity> {
  // Entity is descriptive
}

class Box<Content> {
  // Content describes what's inside
}

// Good: Common conventions
class Result<T> {
  // T for generic type
}

class Pair<K, V> {
  // K for key, V for value
}
```

---

## Use Constraints When Needed

```dart
// Good: Constraint for specific behavior
class Sorter<T extends Comparable<T>> {
  List<T> sort(List<T> items) {
    var sorted = List<T>.from(items);
    sorted.sort();
    return sorted;
  }
}

// Bad: No constraint when needed
class BadSorter<T> {
  List<T> sort(List<T> items) {
    // Can't call compareTo on T
    // items.sort(); // Error!
    return items;
  }
}
```

---

# Common Mistakes

## Using Dynamic Instead of Generic

Wrong:

```dart
class Box {
  dynamic value; // Loses type safety
  Box(this.value);
}

var box = Box('Hello');
var value = box.value; // Type is dynamic
```

Correct:

```dart
class Box<T> {
  T value; // Type safe
  Box(this.value);
}

var box = Box<String>('Hello');
var value = box.value; // Type is String
```

---

## Wrong Type Arguments

Wrong:

```dart
class Box<T> {
  T value;
  Box(this.value);
}

var box = Box<int>('Hello'); // Error! String is not int
```

Correct:

```dart
class Box<T> {
  T value;
  Box(this.value);
}

var box = Box<String>('Hello'); // OK
```

---

# Summary

Generic classes provide type-safe, reusable code. They allow you to write flexible classes that work with any type while maintaining compile-time type checking.

---

# Next Steps

Now that you understand generic classes, continue to:

- [Generic Functions](generic-functions.md)
- [Type Bounds](type-bounds.md)
- [Generic Methods](generic-methods.md)
- [Covariance](covariance.md)

---

# Did You Know?

- Generic classes were introduced in Dart 2.0
- Type parameters are resolved at compile time
- Generics help catch errors early
- Dart uses type erasure (generic types are not retained at runtime)
- You can have multiple type parameters
- Generics work with collections and classes
- Type inference reduces boilerplate for generics