# Base Classes

Understand how to use base classes to control inheritance in Dart.

---

# What is it?

Base classes are a feature introduced in Dart 3 that restrict how a class can be used. A class marked with the `base` modifier can only be extended (not implemented), and the base class itself cannot be instantiated outside its defining library.

---

# Why does it exist?

Base classes exist to:

- Control inheritance hierarchies
- Ensure proper implementation of methods
- Prevent misuse of interfaces
- Enforce design patterns
- Provide stable base classes
- Maintain encapsulation

---

# Basic Base Classes

## Creating a Base Class

```dart
// Base class - can only be extended, not implemented
base class Animal {
  String name;
  
  Animal(this.name);
  
  // Methods that subclasses should override
  void speak() {
    print('$name makes a sound');
  }
  
  void move() {
    print('$name moves');
  }
}

// Correct: Extending base class
class Dog extends Animal {
  String breed;
  
  Dog(String name, this.breed) : super(name);
  
  @override
  void speak() {
    print('$name barks');
  }
  
  @override
  void move() {
    print('$name runs');
  }
  
  void fetch() {
    print('$name fetches the ball');
  }
}

// Correct: Another extension
class Cat extends Animal {
  Cat(String name) : super(name);
  
  @override
  void speak() {
    print('$name meows');
  }
  
  @override
  void move() {
    print('$name jumps');
  }
}

// Usage
var dog = Dog('Rex', 'German Shepherd');
dog.speak(); // Rex barks
dog.move(); // Rex runs
dog.fetch(); // Rex fetches the ball

var cat = Cat('Whiskers');
cat.speak(); // Whiskers meows
cat.move(); // Whiskers jumps
```

---

## Base Class with Abstract Methods

```dart
// Base class with abstract methods
base class Database {
  // Abstract method (must be implemented)
  void connect();
  void disconnect();
  
  // Concrete method
  void query(String sql) {
    print('Executing query: $sql');
  }
  
  // Virtual method (can be overridden)
  void log(String message) {
    print('Database: $message');
  }
}

// Extending base class
class MySQLDatabase extends Database {
  String connectionString;
  
  MySQLDatabase(this.connectionString);
  
  @override
  void connect() {
    print('Connecting to MySQL: $connectionString');
  }
  
  @override
  void disconnect() {
    print('Disconnecting from MySQL');
  }
  
  @override
  void log(String message) {
    print('MySQL: $message');
  }
  
  // Additional method
  void backup() {
    print('Backing up MySQL database');
  }
}

// Usage
var db = MySQLDatabase('mysql://localhost:3306');
db.connect(); // Connecting to MySQL: mysql://localhost:3306
db.query('SELECT * FROM users'); // Executing query: SELECT * FROM users
db.log('Query executed'); // MySQL: Query executed
db.backup(); // Backing up MySQL database
```

---

# Base Classes vs Interfaces

## Comparison

```dart
// Base class (can be extended only)
base class Vehicle {
  void start() => print('Vehicle starting');
  void stop() => print('Vehicle stopping');
}

// Interface (can be implemented)
class Engine {
  void rev() => print('Engine revving');
  void stop() => print('Engine stopping');
}

// Extending base class and implementing interface
class Car extends Vehicle implements Engine {
  @override
  void start() {
    print('Car starting');
    rev(); // Using Engine interface
  }
  
  @override
  void stop() {
    print('Car stopping');
    super.stop(); // Using Vehicle's stop
  }
  
  @override
  void rev() {
    print('Car engine revving');
  }
}

// Usage
var car = Car();
car.start(); // Car starting, Car engine revving
car.stop(); // Car stopping, Vehicle stopping
car.rev(); // Car engine revving
```

---

# Base Class with Multiple Subclasses

## Hierarchy Control

```dart
base class Shape {
  void draw() {
    print('Drawing shape');
  }
  
  double get area => 0;
  double get perimeter => 0;
  
  // Protected method (by convention with _)
  void _log(String message) {
    print('Shape: $message');
  }
}

// Level 1 subclasses
class Circle extends Shape {
  final double radius;
  
  Circle(this.radius);
  
  @override
  void draw() {
    _log('Drawing circle'); // Can access protected method
    print('Circle with radius $radius');
  }
  
  @override
  double get area => math.pi * radius * radius;
  
  @override
  double get perimeter => 2 * math.pi * radius;
}

class Rectangle extends Shape {
  final double width;
  final double height;
  
  Rectangle(this.width, this.height);
  
  @override
  void draw() {
    _log('Drawing rectangle');
    print('Rectangle $width x $height');
  }
  
  @override
  double get area => width * height;
  
  @override
  double get perimeter => 2 * (width + height);
}

// Level 2 subclasses
class ColoredCircle extends Circle {
  final String color;
  
  ColoredCircle(double radius, this.color) : super(radius);
  
  @override
  void draw() {
    super.draw();
    print('Color: $color');
  }
}

// Usage
var shapes = [
  Circle(5),
  Rectangle(10, 20),
  ColoredCircle(7, 'red'),
];

for (var shape in shapes) {
  shape.draw();
  print('Area: ${shape.area}');
  print('Perimeter: ${shape.perimeter}');
  print('---');
}
```

---

# Best Practices

## Use Base for Core Classes

```dart
// Good: Core classes as base
base class Component {
  String id;
  
  Component(this.id);
  
  void render() {
    print('Rendering component: $id');
  }
  
  void dispose() {
    print('Disposing component: $id');
  }
}

class Button extends Component {
  String text;
  
  Button(String id, this.text) : super(id);
  
  @override
  void render() {
    print('Rendering button: $text');
  }
  
  void click() {
    print('Button clicked: $text');
  }
}

class Input extends Component {
  String value;
  
  Input(String id, this.value) : super(id);
  
  @override
  void render() {
    print('Rendering input: $value');
  }
  
  void setValue(String newValue) {
    value = newValue;
    render();
  }
}
```

---

## Combine with Abstract Methods

```dart
// Good: Base with abstract methods
base class DataSource {
  // Abstract methods that must be implemented
  Future<void> connect();
  Future<void> disconnect();
  Future<List<Map<String, dynamic>>> fetch(String query);
  
  // Concrete method with common behavior
  Future<void> executeTransaction(List<String> queries) async {
    await connect();
    try {
      for (var query in queries) {
        await fetch(query);
      }
    } finally {
      await disconnect();
    }
  }
  
  // Virtual method that can be overridden
  void log(String message) {
    print('DataSource: $message');
  }
}

class APIDataSource extends DataSource {
  final String baseUrl;
  
  APIDataSource(this.baseUrl);
  
  @override
  Future<void> connect() async {
    print('Connecting to API: $baseUrl');
  }
  
  @override
  Future<void> disconnect() async {
    print('Disconnecting from API');
  }
  
  @override
  Future<List<Map<String, dynamic>>> fetch(String query) async {
    print('Fetching from API: $query');
    return [];
  }
  
  @override
  void log(String message) {
    print('API: $message');
  }
}
```

---

# Common Mistakes

## Implementing Base Class

Wrong:

```dart
base class Animal {
  void speak() => print('Animal sound');
}

class Bird implements Animal {
  @override
  void speak() => print('Bird chirps');
  // Error: Cannot implement base class
}
```

Correct:

```dart
base class Animal {
  void speak() => print('Animal sound');
}

class Bird extends Animal {
  @override
  void speak() => print('Bird chirps');
}
```

---

## Instantiating Base Class

Wrong:

```dart
base class AbstractBase {
  void doSomething() => print('Doing something');
}

var base = AbstractBase(); // Can instantiate base class
// But shouldn't if it's meant to be abstract
```

Correct:

```dart
abstract base class AbstractBase {
  void doSomething(); // Abstract method
}

// Must extend
class Concrete extends AbstractBase {
  @override
  void doSomething() => print('Doing something');
}
```

---

# Summary

Base classes provide controlled inheritance by allowing only extension (not implementation). They're ideal for creating stable hierarchies and enforcing design patterns.

---

# Next Steps

Now that you understand base classes, continue to:

- [Interface Classes](interface-classes.md)
- [Final Classes](final-classes.md)
- [Factory Constructors](factory-constructors.md)
- [Callable Classes](callable-classes.md)

---

# Did You Know?

- Base classes were introduced in Dart 3
- Base classes can only be extended, not implemented
- Base classes can have abstract methods
- Base classes can be instantiated unless abstract
- The `base` modifier enforces proper inheritance
- Base classes are useful for framework design
- Dart encourages composition over inheritance