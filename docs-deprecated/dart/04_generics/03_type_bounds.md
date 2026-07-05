# Type Bounds

Understand how to restrict generic types using type bounds in Dart.

---

# What is it?

Type bounds are constraints that limit which types can be used as type arguments in generics. They ensure that only types with specific characteristics (like implementing an interface or extending a class) can be used with a generic class or function.

---

# Why does it exist?

Type bounds exist to:

- Enforce type safety
- Enable specific operations on generic types
- Restrict generic types to valid subtypes
- Ensure types have required properties
- Prevent runtime errors
- Document type requirements

---

# Upper Bounds

## Basic Upper Bound

```dart
// Type parameter T must be a subtype of num
class NumberBox<T extends num> {
  T value;
  
  NumberBox(this.value);
  
  // Can use num methods
  T add(T other) {
    return (value + other) as T;
  }
  
  T multiply(T other) {
    return (value * other) as T;
  }
}

// Usage - Works with int and double
var intBox = NumberBox<int>(5);
var doubleBox = NumberBox<double>(3.14);

print(intBox.add(3)); // 8
print(doubleBox.multiply(2)); // 6.28

// Error: String is not a subtype of num
// var stringBox = NumberBox<String>('Hello'); // Error!
```

---

## Upper Bound with Comparable

```dart
// Type must implement Comparable
class Sorter<T extends Comparable<T>> {
  List<T> sort(List<T> items) {
    var sorted = List<T>.from(items);
    sorted.sort();
    return sorted;
  }
  
  T findMax(List<T> items) {
    if (items.isEmpty) {
      throw ArgumentError('List cannot be empty');
    }
    return items.reduce((a, b) => a.compareTo(b) > 0 ? a : b);
  }
  
  T findMin(List<T> items) {
    if (items.isEmpty) {
      throw ArgumentError('List cannot be empty');
    }
    return items.reduce((a, b) => a.compareTo(b) < 0 ? a : b);
  }
}

// Usage
var intSorter = Sorter<int>();
var numbers = [3, 1, 4, 1, 5, 9, 2];
print(intSorter.sort(numbers)); // [1, 1, 2, 3, 4, 5, 9]
print(intSorter.findMax(numbers)); // 9
print(intSorter.findMin(numbers)); // 1

var stringSorter = Sorter<String>();
var words = ['apple', 'banana', 'cherry'];
print(stringSorter.sort(words)); // [apple, banana, cherry]

// Error: NonComparable doesn't implement Comparable
// class NonComparable {}
// var badSorter = Sorter<NonComparable>(); // Error!
```

---

# Multiple Bounds

## Combining Constraints

```dart
// Multiple bounds using interfaces
class Identifiable {
  String get id;
}

class Serializable {
  Map<String, dynamic> toJson();
}

class Validatable {
  bool isValid();
}

// Type must implement all three
class EntityManager<T extends Object 
    & Identifiable 
    & Serializable 
    & Validatable> {
  final Map<String, T> _entities = {};
  
  void save(T entity) {
    if (!entity.isValid()) {
      throw ArgumentError('Entity is invalid');
    }
    _entities[entity.id] = entity;
  }
  
  T? find(String id) {
    return _entities[id];
  }
  
  List<Map<String, dynamic>> exportAll() {
    return _entities.values.map((e) => e.toJson()).toList();
  }
  
  List<T> findAll() {
    return _entities.values.toList();
  }
}

// Implementation class
class User implements Identifiable, Serializable, Validatable {
  final String name;
  final int age;
  
  User(this.name, this.age);
  
  @override
  String get id => name.toLowerCase().replaceAll(' ', '_');
  
  @override
  bool isValid() => name.isNotEmpty && age > 0 && age < 150;
  
  @override
  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'age': age,
  };
}

// Usage
var manager = EntityManager<User>();
var user = User('Alice', 25);
manager.save(user);

var found = manager.find('alice');
print(found?.toJson()); // {id: alice, name: Alice, age: 25}

var exported = manager.exportAll();
print(exported); // [{id: alice, name: Alice, age: 25}]
```

---

# Lower Bounds

## Using Lower Bounds

```dart
// Lower bound with Null
class Optional<T extends Object?> {
  final T? _value;
  
  Optional(this._value);
  
  T? get value => _value;
  
  bool get hasValue => _value != null;
  
  T orElse(T defaultValue) {
    return _value ?? defaultValue;
  }
  
  Optional<T> map<R>(R Function(T) mapper) {
    if (hasValue) {
      return Optional(mapper(_value as T));
    }
    return Optional(null);
  }
}

// Usage - Works with any type
var stringOptional = Optional<String>('Hello');
var nullOptional = Optional<String>(null);

print(stringOptional.orElse('World')); // Hello
print(nullOptional.orElse('World')); // World

var numberOptional = Optional<int>(42);
print(numberOptional.orElse(0)); // 42

var optionalWithNull = Optional<int?>(null);
print(optionalWithNull.orElse(0)); // 0
```

---

# Recursive Type Bounds

## Self-Referential Bounds

```dart
// Recursive type bound
abstract class Comparable<T> {
  int compareTo(T other);
}

class Person implements Comparable<Person> {
  final String name;
  final int age;
  
  Person(this.name, this.age);
  
  @override
  int compareTo(Person other) {
    return age.compareTo(other.age);
  }
  
  @override
  String toString() => 'Person($name, $age)';
}

class Sorter<T extends Comparable<T>> {
  List<T> sort(List<T> items) {
    var sorted = List<T>.from(items);
    sorted.sort((a, b) => a.compareTo(b));
    return sorted;
  }
}

// Usage
var people = [
  Person('Alice', 25),
  Person('Bob', 30),
  Person('Charlie', 20),
];

var sorter = Sorter<Person>();
var sorted = sorter.sort(people);
print(sorted); // [Person(Charlie, 20), Person(Alice, 25), Person(Bob, 30)]
```

---

# Common Type Bounds

## Built-in Type Bounds

```dart
// Common bounds in Dart
void examples() {
  // 1. Must be Comparable
  void sort<T extends Comparable<T>>(List<T> items) {}
  
  // 2. Must be numeric
  class NumberContainer<T extends num> {}
  
  // 3. Must be non-nullable
  class NonNullContainer<T extends Object> {}
  
  // 4. Must implement multiple interfaces
  class MultiConstraint<T extends Object & Identifiable & Serializable> {}
  
  // 5. Must be a List
  class ListWrapper<T extends List> {}
  
  // 6. Must be a Map
  class MapWrapper<K, V, T extends Map<K, V>> {}
  
  // 7. Must be a Future
  class FutureWrapper<T extends Future> {}
}

// Example usage
// 1. Sorting
void sortNumbers(List<num> numbers) {
  numbers.sort(); // Works because num is Comparable
}

// 2. Numeric container
var intContainer = NumberContainer<int>(5);
var doubleContainer = NumberContainer<double>(3.14);

// 3. Non-nullable
var nonNull = NonNullContainer<String>('Hello');
// var nullContainer = NonNullContainer<String?>(null); // Error!

// 4. Multi-constraint
class UserManager extends EntityManager<User> {
  // Works because User implements all required interfaces
}
```

---

# Best Practices

## Use Upper Bounds for Safe Operations

```dart
// Good: Bound ensures safe operations
class Calculator<T extends num> {
  T add(T a, T b) {
    return (a + b) as T;
  }
}

// Bad: No bound, unsafe
class BadCalculator<T> {
  T add(T a, T b) {
    // Cannot safely add T types
    return a; // Doesn't actually add!
  }
}
```

---

## Use Multiple Bounds for Complex Requirements

```dart
// Good: Clear requirements
class DataProcessor<T extends Object & Serializable & Validatable> {
  void process(T data) {
    if (!data.isValid()) {
      throw ArgumentError('Invalid data');
    }
    var json = data.toJson();
    print('Processing: $json');
  }
}

// Bad: Implicit requirements
class BadProcessor<T> {
  void process(T data) {
    // No guarantee that data has isValid() or toJson()
  }
}
```

---

# Common Mistakes

## Too Restrictive Bounds

Wrong:

```dart
class Container<T extends String> {
  // Restricts to String only - defeats purpose of generics
  T value;
  Container(this.value);
}
```

Correct:

```dart
class Container<T> {
  // Works with any type
  T value;
  Container(this.value);
}
```

---

## Missing Necessary Bounds

Wrong:

```dart
class Sorter<T> {
  List<T> sort(List<T> items) {
    // Cannot sort without Comparable
    items.sort(); // Error!
    return items;
  }
}
```

Correct:

```dart
class Sorter<T extends Comparable<T>> {
  List<T> sort(List<T> items) {
    var sorted = List<T>.from(items);
    sorted.sort(); // Works because T is Comparable
    return sorted;
  }
}
```

---

# Summary

Type bounds provide powerful constraints for generics, ensuring type safety and enabling specific operations on generic types.

---

# Next Steps

Now that you understand type bounds, continue to:

- [Generic Methods](generic-methods.md)
- [Covariance](covariance.md)
- [Variance](variance.md)
- [Async Programming](async-programming.md)

---

# Did You Know?

- Type bounds restrict which types can be used
- Upper bounds are most common (`extends`)
- Multiple bounds use `&` operator
- Lower bounds use `super` (less common)
- Bounds can be recursive
- Type bounds enable generic operations
- Bounds are resolved at compile time