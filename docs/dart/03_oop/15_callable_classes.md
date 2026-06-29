# Callable Classes

Understand how to make classes callable like functions in Dart.

---

# What is it?

Callable classes are classes that define a `call()` method, allowing instances to be invoked as if they were functions. This feature enables objects to behave like functions, making them useful for creating function-like objects with state.

---

# Why does it exist?

Callable classes exist to:

- Create function-like objects with state
- Implement closures with custom behavior
- Enable object-oriented functional programming
- Create reusable function factories
- Provide flexible APIs
- Simplify code that uses functions

---

# Basic Callable Classes

## Simple Callable Class

```dart
class Greeter {
  final String greeting;
  
  Greeter(this.greeting);
  
  // The call method makes the class callable
  String call(String name) {
    return '$greeting, $name!';
  }
}

// Usage
var greeter = Greeter('Hello');
print(greeter('Alice')); // Hello, Alice!
print(greeter('Bob')); // Hello, Bob!

var formalGreeter = Greeter('Good morning');
print(formalGreeter('Dr. Smith')); // Good morning, Dr. Smith!
```

---

## Callable Class with State

```dart
class Counter {
  int _count = 0;
  
  // Call method increments and returns count
  int call() {
    _count++;
    return _count;
  }
  
  // Additional methods
  int get count => _count;
  void reset() => _count = 0;
}

// Usage
var counter = Counter();
print(counter()); // 1
print(counter()); // 2
print(counter()); // 3
print(counter.count); // 3
counter.reset();
print(counter()); // 1
```

---

# Callable Classes with Multiple Parameters

## Complex Call Methods

```dart
class Calculator {
  // Call with multiple parameters
  double call(double a, double b, {String operation = 'add'}) {
    switch (operation) {
      case 'add':
        return a + b;
      case 'subtract':
        return a - b;
      case 'multiply':
        return a * b;
      case 'divide':
        if (b == 0) throw ArgumentError('Cannot divide by zero');
        return a / b;
      default:
        throw ArgumentError('Unknown operation: $operation');
    }
  }
}

// Usage
var calc = Calculator();
print(calc(5, 3)); // 8.0
print(calc(5, 3, operation: 'subtract')); // 2.0
print(calc(5, 3, operation: 'multiply')); // 15.0
print(calc(10, 2, operation: 'divide')); // 5.0
```

---

## Call with Named Parameters

```dart
class QueryBuilder {
  String _base = '';
  List<String> _conditions = [];
  
  QueryBuilder(String base) : _base = base;
  
  // Call method builds query
  String call({
    List<String>? select,
    String? where,
    String? orderBy,
    int? limit,
  }) {
    var query = _base;
    
    if (select != null && select.isNotEmpty) {
      query = query.replaceFirst('*', select.join(', '));
    }
    
    if (where != null) {
      query += ' WHERE $where';
    }
    
    if (orderBy != null) {
      query += ' ORDER BY $orderBy';
    }
    
    if (limit != null) {
      query += ' LIMIT $limit';
    }
    
    return query;
  }
  
  // Add where condition
  QueryBuilder where(String condition) {
    _conditions.add(condition);
    return this;
  }
}

// Usage
var query = QueryBuilder('SELECT * FROM users');
print(query());
// SELECT * FROM users

print(query(
  select: ['id', 'name'],
  where: 'age > 18',
  orderBy: 'name ASC',
  limit: 10,
));
// SELECT id, name FROM users WHERE age > 18 ORDER BY name ASC LIMIT 10
```

---

# Callable Classes as Function Factories

## Creating Function-Like Objects

```dart
class Multiplier {
  final int factor;
  
  Multiplier(this.factor);
  
  // Call method multiplies by factor
  int call(int value) {
    return value * factor;
  }
}

class Adder {
  final int addend;
  
  Adder(this.addend);
  
  // Call method adds addend
  int call(int value) {
    return value + addend;
  }
}

class FunctionComposer {
  final List<Function> _functions = [];
  
  // Add functions to compose
  void add(Function fn) {
    _functions.add(fn);
  }
  
  // Call method composes all functions
  dynamic call(dynamic input) {
    dynamic result = input;
    for (var fn in _functions) {
      result = fn(result);
    }
    return result;
  }
}

// Usage
var times2 = Multiplier(2);
var add5 = Adder(5);

// Create composer
var composer = FunctionComposer()
  ..add(times2)
  ..add(add5)
  ..add(times2);

print(composer(3)); // 3 * 2 + 5 * 2 = 22
print(composer(10)); // 10 * 2 + 5 * 2 = 30
```

---

# Callable Classes for Validation

## Validator as Callable

```dart
class EmailValidator {
  // Call method validates email
  bool call(String email) {
    if (email.isEmpty) return false;
    return email.contains('@') && email.contains('.');
  }
}

class PasswordValidator {
  final int minLength;
  final bool requireUppercase;
  final bool requireLowercase;
  final bool requireDigit;
  
  PasswordValidator({
    this.minLength = 8,
    this.requireUppercase = true,
    this.requireLowercase = true,
    this.requireDigit = true,
  });
  
  // Call method validates password
  bool call(String password) {
    if (password.length < minLength) return false;
    
    if (requireUppercase && !RegExp(r'[A-Z]').hasMatch(password)) {
      return false;
    }
    
    if (requireLowercase && !RegExp(r'[a-z]').hasMatch(password)) {
      return false;
    }
    
    if (requireDigit && !RegExp(r'\d').hasMatch(password)) {
      return false;
    }
    
    return true;
  }
}

class ValidatorChain {
  final List<bool Function(dynamic)> _validators = [];
  
  void add(bool Function(dynamic) validator) {
    _validators.add(validator);
  }
  
  // Call method runs all validators
  bool call(dynamic value) {
    for (var validator in _validators) {
      if (!validator(value)) {
        return false;
      }
    }
    return true;
  }
}

// Usage
var emailValidator = EmailValidator();
print(emailValidator('test@example.com')); // true
print(emailValidator('invalid')); // false

var passwordValidator = PasswordValidator();
print(passwordValidator('Password123')); // true
print(passwordValidator('pass')); // false

var chain = ValidatorChain()
  ..add(emailValidator)
  ..add((value) => value.isNotEmpty);

print(chain('test@example.com')); // true
print(chain('')); // false
```

---

# Callable Classes in Collections

## Filter as Callable

```dart
class Filter<T> {
  final bool Function(T) _predicate;
  
  Filter(this._predicate);
  
  // Call method filters collection
  List<T> call(Iterable<T> items) {
    return items.where(_predicate).toList();
  }
}

class Transform<T, R> {
  final R Function(T) _mapper;
  
  Transform(this._mapper);
  
  // Call method transforms collection
  List<R> call(Iterable<T> items) {
    return items.map(_mapper).toList();
  }
}

class Pipeline {
  final List<Function> _operations = [];
  
  void add(Function operation) {
    _operations.add(operation);
  }
  
  // Call method runs operations
  List<dynamic> call(List<dynamic> input) {
    var result = input;
    for (var op in _operations) {
      if (op is Function) {
        result = op(result);
      }
    }
    return result;
  }
}

// Usage
var numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

var even = Filter<int>((n) => n % 2 == 0);
var doubled = Transform<int, int>((n) => n * 2);

print(even(numbers)); // [2, 4, 6, 8, 10]
print(doubled(numbers)); // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

var pipeline = Pipeline()
  ..add(even)
  ..add(doubled);

print(pipeline(numbers)); // [4, 8, 12, 16, 20]
```

---

# Best Practices

## Use Callable for Stateful Functions

```dart
// Good: Callable with state
class Counter {
  int _count = 0;
  
  int call() => ++_count;
  
  int get count => _count;
  void reset() => _count = 0;
}

// Bad: Using global state
int _globalCount = 0;
int increment() => ++_globalCount;
```

---

## Keep Call Methods Simple

```dart
// Good: Simple call method
class Calculator {
  double call(double a, double b, {String op = 'add'}) {
    switch (op) {
      case 'add': return a + b;
      case 'sub': return a - b;
      default: throw ArgumentError('Unknown op');
    }
  }
}

// Bad: Too much logic in call
class BadCalculator {
  double call(double a, double b, {String op = 'add'}) {
    // Complex validation
    // Logging
    // Complex calculations
    // Side effects
    // Too much happening here
  }
}
```

---

# Common Mistakes

## Overusing Callable

Wrong:

```dart
class MathUtils {
  double call(double a, double b) => a + b;
  // Should just be a static method
}
```

Correct:

```dart
class MathUtils {
  static double add(double a, double b) => a + b;
}
```

---

## Call Method Too Complex

Wrong:

```dart
class ComplexProcessor {
  dynamic call(dynamic input) {
    // Handles multiple types
    if (input is String) { /* ... */ }
    if (input is int) { /* ... */ }
    if (input is List) { /* ... */ }
    // Too complex
  }
}
```

Correct:

```dart
class StringProcessor {
  String call(String input) {
    // Handles only strings
  }
}

class IntProcessor {
  int call(int input) {
    // Handles only ints
  }
}
```

---

# Summary

Callable classes provide a powerful way to create function-like objects with state and behavior. They're useful for creating reusable components, validators, and function factories.

---

# Next Steps

Now that you understand callable classes, continue to:

- [Object Lifecycle](object-lifecycle.md)
- [Generic Classes](generic-classes.md)
- [Generic Functions](generic-functions.md)
- [Type Bounds](type-bounds.md)

---

# Did You Know?

- Callable classes use the `call()` method
- They can be invoked like functions
- Call methods can have multiple parameters
- They can maintain internal state
- Callable classes support named parameters
- They're useful for function factories
- Callable classes can be used in collections