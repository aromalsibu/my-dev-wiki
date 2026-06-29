# Classes

Understand how to define and use classes in Dart's object-oriented programming model.

---

# What is it?

Classes are blueprints for creating objects in Dart. They encapsulate data (fields) and behavior (methods) into a single unit, supporting object-oriented programming principles like encapsulation, inheritance, and polymorphism.

---

# Why does it exist?

Classes exist to:

- Model real-world entities and concepts
- Organize code into logical units
- Encapsulate data and behavior
- Enable code reuse through inheritance
- Support polymorphism and interfaces
- Create complex systems with clear structure

---

# Basic Classes

## Simple Class

```dart
class Person {
  // Fields (data)
  String name;
  int age;
  
  // Constructor
  Person(this.name, this.age);
  
  // Methods (behavior)
  void sayHello() {
    print('Hello, I am $name');
  }
  
  String get greeting => 'Hello, $name';
}

// Usage
var person = Person('Alice', 25);
person.sayHello(); // Hello, I am Alice
print(person.greeting); // Hello, Alice
print(person.name); // Alice
print(person.age); // 25
```

---

## Class with Final Fields

```dart
class User {
  // Final fields (set once)
  final String id;
  final String name;
  final DateTime createdAt;
  
  // Constructor with initializer list
  User(this.id, this.name) : createdAt = DateTime.now();
  
  // No setters for final fields
  String get greeting => 'Hello, $name';
  
  @override
  String toString() => 'User($id, $name)';
}

// Usage
var user = User('123', 'Alice');
print(user.id); // 123
print(user.name); // Alice
print(user.createdAt); // Current timestamp
```

---

# Fields

## Instance Fields

```dart
class Product {
  // Public fields
  String name;
  double price;
  
  // Private field (starts with _)
  int _stock = 0;
  
  // Final field
  final String sku;
  
  Product(this.name, this.price, this.sku);
  
  // Getter for private field
  int get stock => _stock;
  
  // Setter with validation
  set stock(int value) {
    if (value >= 0) {
      _stock = value;
    }
  }
  
  // Method to update stock
  void addStock(int amount) {
    if (amount > 0) {
      _stock += amount;
    }
  }
}

// Usage
var product = Product('Laptop', 999.99, 'LAP123');
print(product.name); // Laptop
print(product.stock); // 0
product.stock = 10;
print(product.stock); // 10
product.addStock(5);
print(product.stock); // 15
```

---

## Static Fields

```dart
class AppConfig {
  // Static field (shared across all instances)
  static String appName = 'My App';
  static const String version = '1.0.0';
  static int _instanceCount = 0;
  
  // Static getter
  static int get instanceCount => _instanceCount;
  
  // Static method
  static void printConfig() {
    print('App: $appName, Version: $version');
  }
  
  // Constructor increments counter
  AppConfig() {
    _instanceCount++;
  }
}

// Usage (no instance needed)
print(AppConfig.appName); // My App
print(AppConfig.version); // 1.0.0
AppConfig.printConfig(); // App: My App, Version: 1.0.0

// Count instances
var config1 = AppConfig();
var config2 = AppConfig();
print(AppConfig.instanceCount); // 2
```

---

# Methods

## Instance Methods

```dart
class Calculator {
  // Instance methods
  int add(int a, int b) => a + b;
  int subtract(int a, int b) => a - b;
  int multiply(int a, int b) => a * b;
  double divide(int a, int b) {
    if (b == 0) {
      throw ArgumentError('Cannot divide by zero');
    }
    return a / b;
  }
  
  // Method with named parameters
  int calculate({
    required int a,
    required int b,
    String operation = 'add',
  }) {
    switch (operation) {
      case 'add':
        return add(a, b);
      case 'subtract':
        return subtract(a, b);
      case 'multiply':
        return multiply(a, b);
      default:
        throw ArgumentError('Unknown operation: $operation');
    }
  }
}

// Usage
var calc = Calculator();
print(calc.add(5, 3)); // 8
print(calc.calculate(a: 5, b: 3, operation: 'multiply')); // 15
```

---

## Static Methods

```dart
class MathUtils {
  // Static methods (utility functions)
  static int sum(List<int> numbers) {
    return numbers.reduce((a, b) => a + b);
  }
  
  static int product(List<int> numbers) {
    return numbers.reduce((a, b) => a * b);
  }
  
  static double average(List<int> numbers) {
    if (numbers.isEmpty) return 0;
    return sum(numbers) / numbers.length;
  }
  
  static int max(List<int> numbers) {
    if (numbers.isEmpty) {
      throw ArgumentError('List cannot be empty');
    }
    return numbers.reduce((a, b) => a > b ? a : b);
  }
  
  static int min(List<int> numbers) {
    if (numbers.isEmpty) {
      throw ArgumentError('List cannot be empty');
    }
    return numbers.reduce((a, b) => a < b ? a : b);
  }
}

// Usage (no instance needed)
var numbers = [1, 2, 3, 4, 5];
print(MathUtils.sum(numbers)); // 15
print(MathUtils.average(numbers)); // 3.0
print(MathUtils.max(numbers)); // 5
print(MathUtils.min(numbers)); // 1
```

---

# Getters and Setters

## Custom Accessors

```dart
class Rectangle {
  double _width;
  double _height;
  
  Rectangle(this._width, this._height);
  
  // Getter
  double get width => _width;
  double get height => _height;
  
  // Setter with validation
  set width(double value) {
    if (value > 0) {
      _width = value;
    }
  }
  
  set height(double value) {
    if (value > 0) {
      _height = value;
    }
  }
  
  // Computed getters
  double get area => _width * _height;
  double get perimeter => 2 * (_width + _height);
  
  // Getter with calculation
  String get description {
    return 'Rectangle(${_width.toStringAsFixed(1)} x ${_height.toStringAsFixed(1)})';
  }
  
  // Setter with side effect
  set size(double value) {
    _width = value;
    _height = value;
  }
}

// Usage
var rect = Rectangle(10, 20);
print(rect.area); // 200
print(rect.perimeter); // 60
rect.width = 15;
rect.height = 25;
print(rect.area); // 375
rect.size = 10;
print(rect.description); // Rectangle(10.0 x 10.0)
```

---

# Constructors

## Constructor Types

```dart
class Point {
  final double x;
  final double y;
  
  // Generative constructor
  Point(this.x, this.y);
  
  // Named constructor
  Point.origin() : this(0, 0);
  
  // Named constructor with calculation
  Point.fromPolar(double radius, double angle)
      : this(radius * math.cos(angle), radius * math.sin(angle));
  
  // Factory constructor
  factory Point.fromJson(Map<String, double> json) {
    if (json.containsKey('x') && json.containsKey('y')) {
      return Point(json['x']!, json['y']!);
    }
    throw FormatException('Invalid JSON for Point');
  }
  
  // Redirecting constructor
  Point.zero() : this(0, 0);
  
  // Constant constructor
  const Point.constant(this.x, this.y);
  
  // Override toString
  @override
  String toString() => 'Point($x, $y)';
}

// Usage
var p1 = Point(3, 4);
var p2 = Point.origin();
var p3 = Point.zero();
var p4 = const Point.constant(1, 2);
var p5 = Point.fromPolar(5, math.pi / 4);
var p6 = Point.fromJson({'x': 10, 'y': 20});

print(p1); // Point(3.0, 4.0)
print(p2); // Point(0.0, 0.0)
print(p6); // Point(10.0, 20.0)
```

---

# Inheritance

## Extending Classes

```dart
// Base class
class Animal {
  String name;
  
  Animal(this.name);
  
  void speak() {
    print('$name makes a sound');
  }
  
  String get type => 'Animal';
}

// Derived class
class Dog extends Animal {
  String breed;
  
  Dog(String name, this.breed) : super(name);
  
  @override
  void speak() {
    print('$name barks');
  }
  
  @override
  String get type => 'Dog';
  
  // Additional method
  void fetch() {
    print('$name fetches the ball');
  }
}

// Another derived class
class Cat extends Animal {
  Cat(String name) : super(name);
  
  @override
  void speak() {
    print('$name meows');
  }
  
  @override
  String get type => 'Cat';
  
  void purr() {
    print('$name purrs');
  }
}

// Usage
var dog = Dog('Rex', 'German Shepherd');
dog.speak(); // Rex barks
dog.fetch(); // Rex fetches the ball
print(dog.type); // Dog

var cat = Cat('Whiskers');
cat.speak(); // Whiskers meows
cat.purr(); // Whiskers purrs
print(cat.type); // Cat

// Polymorphism
void processAnimal(Animal animal) {
  print('Processing ${animal.name} (${animal.type})');
  animal.speak();
}

processAnimal(dog); // Processing Rex (Dog)
processAnimal(cat); // Processing Whiskers (Cat)
```

---

# Abstract Classes

## Abstract Base Classes

```dart
// Abstract class (cannot be instantiated)
abstract class Shape {
  // Abstract method (must be implemented)
  double get area;
  
  // Abstract method
  double get perimeter;
  
  // Concrete method
  String get description => 'Shape with area $area and perimeter $perimeter';
}

class Circle extends Shape {
  final double radius;
  
  Circle(this.radius);
  
  @override
  double get area => math.pi * radius * radius;
  
  @override
  double get perimeter => 2 * math.pi * radius;
  
  @override
  String get description => 'Circle with radius $radius';
}

class Rectangle extends Shape {
  final double width;
  final double height;
  
  Rectangle(this.width, this.height);
  
  @override
  double get area => width * height;
  
  @override
  double get perimeter => 2 * (width + height);
  
  @override
  String get description => 'Rectangle $width x $height';
}

// Usage
var circle = Circle(5);
var rectangle = Rectangle(10, 20);

print(circle.area); // 78.54
print(circle.perimeter); // 31.42
print(circle.description); // Circle with radius 5

print(rectangle.area); // 200
print(rectangle.perimeter); // 60
print(rectangle.description); // Rectangle 10 x 20
```

---

# Best Practices

## Use Private Fields

```dart
// Good: Encapsulation with private fields
class BankAccount {
  String _accountNumber;
  double _balance = 0;
  
  BankAccount(this._accountNumber);
  
  double get balance => _balance;
  
  void deposit(double amount) {
    if (amount > 0) {
      _balance += amount;
    }
  }
  
  bool withdraw(double amount) {
    if (amount > 0 && amount <= _balance) {
      _balance -= amount;
      return true;
    }
    return false;
  }
}

// Bad: Exposed fields
class BankAccountBad {
  String accountNumber;
  double balance = 0;
  // Anyone can modify directly!
}
```

---

## Use Final When Possible

```dart
// Good: Immutable where possible
class ImmutableUser {
  final String id;
  final String name;
  final DateTime createdAt;
  
  const ImmutableUser(this.id, this.name, this.createdAt);
}

// Bad: Mutable when not needed
class MutableUser {
  String id;
  String name;
  DateTime createdAt;
  
  MutableUser(this.id, this.name, this.createdAt);
}
```

---

# Common Mistakes

## Forgetting to Initialize

Wrong:

```dart
class Person {
  String name; // Error! Must be initialized
  int age; // Error! Must be initialized
}
```

Correct:

```dart
class Person {
  String name;
  int age;
  
  Person(this.name, this.age); // Initialized in constructor
}
```

---

## Missing Constructor

Wrong:

```dart
class Person {
  String name = ''; // Initialized
  int age = 0; // Initialized
}
```

Correct:

```dart
class Person {
  String name = ''; // With default value
  int age = 0;
}
```

---

# Summary

Classes are the foundation of object-oriented programming in Dart. They encapsulate data and behavior, support inheritance and abstraction, and enable organized, reusable code.

---

# Next Steps

Now that you understand classes, continue to:

- [Constructors](constructors.md)
- [Fields](fields.md)
- [Methods](methods.md)
- [Inheritance](inheritance.md)

---

# Did You Know?

- Dart supports both single inheritance and multiple mixins
- Classes can have both instance and static members
- Abstract classes cannot be instantiated
- All classes inherit from `Object`
- Dart uses `@override` for method overriding
- Classes can have factory constructors
- Dart supports class-level constants with `static const`