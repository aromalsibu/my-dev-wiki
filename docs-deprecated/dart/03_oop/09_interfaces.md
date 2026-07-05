# Interfaces

Understand how to define and implement interfaces in Dart.

---

# What is it?

Interfaces define a contract that classes must follow. In Dart, every class implicitly defines an interface, meaning you can implement any class as an interface. Interfaces specify which methods and properties a class must have, enabling polymorphism and loose coupling.

---

# Why does it exist?

Interfaces exist to:

- Define contracts for classes to implement
- Enable polymorphism without inheritance
- Support multiple interface implementation
- Create loose coupling between components
- Facilitate testing through mocking
- Design clean and maintainable APIs

---

# Basic Interfaces

## Implementing an Interface

```dart
// Interface (defined as a class)
class Flyable {
  void fly() {
    // No implementation needed, but can have default
  }
  
  String get wings => '';
}

// Class implementing the interface
class Bird implements Flyable {
  @override
  void fly() {
    print('Bird flying...');
  }
  
  @override
  String get wings => 'Feathered wings';
}

// Another implementation
class Airplane implements Flyable {
  @override
  void fly() {
    print('Airplane flying...');
  }
  
  @override
  String get wings => 'Metal wings';
}

// Usage
void makeItFly(Flyable flyable) {
  flyable.fly();
  print('Wings: ${flyable.wings}');
}

var bird = Bird();
var plane = Airplane();

makeItFly(bird); // Bird flying... Wings: Feathered wings
makeItFly(plane); // Airplane flying... Wings: Metal wings
```

---

## Interface with Multiple Methods

```dart
// Interface with multiple methods
class Drawable {
  void draw() {}
  void erase() {}
  void resize(double factor) {}
  String get description => '';
}

class Rectangle implements Drawable {
  double width;
  double height;
  
  Rectangle(this.width, this.height);
  
  @override
  void draw() {
    print('Drawing rectangle $width x $height');
  }
  
  @override
  void erase() {
    print('Erasing rectangle');
  }
  
  @override
  void resize(double factor) {
    width *= factor;
    height *= factor;
    print('Resized to $width x $height');
  }
  
  @override
  String get description => 'Rectangle($width x $height)';
}

class Circle implements Drawable {
  double radius;
  
  Circle(this.radius);
  
  @override
  void draw() {
    print('Drawing circle with radius $radius');
  }
  
  @override
  void erase() {
    print('Erasing circle');
  }
  
  @override
  void resize(double factor) {
    radius *= factor;
    print('Resized to radius $radius');
  }
  
  @override
  String get description => 'Circle(radius $radius)';
}

// Usage
void processDrawable(Drawable drawable) {
  print(drawable.description);
  drawable.draw();
  drawable.resize(2);
  drawable.erase();
  print('---');
}

var rect = Rectangle(10, 20);
var circle = Circle(5);

processDrawable(rect);
processDrawable(circle);
```

---

# Multiple Interfaces

## Implementing Multiple Interfaces

```dart
// Multiple interfaces
class JsonSerializable {
  Map<String, dynamic> toJson();
}

class Validatable {
  bool isValid();
}

class Renderable {
  void render();
}

// Class implementing multiple interfaces
class User implements JsonSerializable, Validatable, Renderable {
  String name;
  int age;
  String email;
  
  User(this.name, this.age, this.email);
  
  // From JsonSerializable
  @override
  Map<String, dynamic> toJson() {
    return {
      'name': name,
      'age': age,
      'email': email,
    };
  }
  
  // From Validatable
  @override
  bool isValid() {
    return name.isNotEmpty && age > 0 && email.contains('@');
  }
  
  // From Renderable
  @override
  void render() {
    print('Rendering user: $name ($age)');
  }
}

// Usage
var user = User('Alice', 25, 'alice@example.com');

// Use as JsonSerializable
Map<String, dynamic> json = user.toJson();
print(json); // {name: Alice, age: 25, email: alice@example.com}

// Use as Validatable
print(user.isValid()); // true

// Use as Renderable
user.render(); // Rendering user: Alice (25)

// Polymorphic usage
void processJson(JsonSerializable item) {
  print(item.toJson());
}

void validate(Validatable item) {
  print('Valid: ${item.isValid()}');
}

processJson(user); // {name: Alice, age: 25, email: alice@example.com}
validate(user); // Valid: true
```

---

# Interface Inheritance

## Extending Interfaces

```dart
// Base interface
class Animal {
  void eat() {}
  void sleep() {}
  String get name => '';
}

// Extended interface
class Mammal implements Animal {
  @override
  void eat() => print('Mammal eating');
  
  @override
  void sleep() => print('Mammal sleeping');
  
  @override
  String get name => 'Mammal';
  
  void nurse() => print('Nursing young');
}

// Extended interface with additional methods
class Bird implements Animal {
  @override
  void eat() => print('Bird eating');
  
  @override
  void sleep() => print('Bird sleeping');
  
  @override
  String get name => 'Bird';
  
  void fly() => print('Bird flying');
}

// Usage
void processAnimal(Animal animal) {
  print(animal.name);
  animal.eat();
  animal.sleep();
  
  // Check for specific interface
  if (animal is Mammal) {
    animal.nurse();
  }
  if (animal is Bird) {
    animal.fly();
  }
  print('---');
}

var mammal = Mammal();
var bird = Bird();

processAnimal(mammal);
// Mammal
// Mammal eating
// Mammal sleeping
// Nursing young
// ---

processAnimal(bird);
// Bird
// Bird eating
// Bird sleeping
// Bird flying
// ---
```

---

# Abstract Class vs Interface

## Comparison

```dart
// Abstract class (can have implementation)
abstract class Database {
  // Abstract method
  Future<void> connect();
  
  // Concrete method
  Future<void> disconnect() async {
    print('Disconnecting...');
  }
}

// Interface (implicit, no implementation)
class Repository {
  Future<void> save(String data);
  Future<String> load(String id);
}

// Class extending abstract class and implementing interface
class MySQLDatabase extends Database implements Repository {
  @override
  Future<void> connect() async {
    print('Connecting to MySQL...');
  }
  
  @override
  Future<void> save(String data) async {
    print('Saving to MySQL: $data');
  }
  
  @override
  Future<String> load(String id) async {
    print('Loading from MySQL: $id');
    return 'Data for $id';
  }
  
  // Can override concrete method from abstract class
  @override
  Future<void> disconnect() async {
    print('Disconnecting from MySQL...');
  }
}

// Usage
var db = MySQLDatabase();
await db.connect(); // Connecting to MySQL...
await db.save('user_data'); // Saving to MySQL: user_data
await db.disconnect(); // Disconnecting from MySQL...
```

---

# Interface with Default Implementation

## Providing Default Behavior

```dart
// Interface with default implementation
class Logger {
  // Default implementation
  void log(String message) {
    print('${DateTime.now()}: $message');
  }
  
  void logError(String message) {
    log('ERROR: $message');
  }
  
  void logWarning(String message) {
    log('WARNING: $message');
  }
}

// Class using default implementation
class ConsoleLogger implements Logger {
  // Uses default implementation from Logger
  // Can override if needed
}

// Class with custom implementation
class FileLogger implements Logger {
  @override
  void log(String message) {
    // Write to file instead of console
    print('Writing to file: $message');
  }
  
  @override
  void logError(String message) {
    log('ERROR: $message');
    // Additional error handling
  }
}

// Usage
var consoleLogger = ConsoleLogger();
consoleLogger.log('App started'); // 2024-01-01 00:00:00.000: App started

var fileLogger = FileLogger();
fileLogger.log('App started'); // Writing to file: App started
```

---

# Best Practices

## Use Interfaces for Loose Coupling

```dart
// Good: Using interface for dependency injection
class UserService {
  final Repository _repository;
  
  UserService(this._repository);
  
  Future<void> saveUser(User user) async {
    await _repository.save(user.toJson());
  }
}

// Any class implementing Repository can be used
class MySQLRepository implements Repository {
  @override
  Future<void> save(Map<String, dynamic> data) async {
    print('Saving to MySQL: $data');
  }
}

class InMemoryRepository implements Repository {
  @override
  Future<void> save(Map<String, dynamic> data) async {
    print('Saving in memory: $data');
  }
}

var service = UserService(MySQLRepository());
await service.saveUser(user);
```

---

## Keep Interfaces Focused

```dart
// Good: Focused interfaces
class Logger {
  void log(String message) {}
}

class Configurable {
  void configure(Map<String, dynamic> config) {}
}

class Database {
  Future<void> query(String sql) {}
}

// Bad: Monolithic interface
class Everything {
  void log(String message) {}
  void configure(Map<String, dynamic> config) {}
  Future<void> query(String sql) {}
  void render() {}
  void validate() {}
  // Too many responsibilities
}
```

---

# Common Mistakes

## Missing @override Annotations

Wrong:

```dart
class Dog implements Animal {
  void speak() {
    print('Woof!');
  }
  // Missing @override
}
```

Correct:

```dart
class Dog implements Animal {
  @override
  void speak() {
    print('Woof!');
  }
}
```

---

## Incomplete Implementation

Wrong:

```dart
class User implements JsonSerializable, Validatable {
  String name;
  
  User(this.name);
  
  @override
  Map<String, dynamic> toJson() {
    return {'name': name};
  }
  // Missing isValid() from Validatable
}
```

Correct:

```dart
class User implements JsonSerializable, Validatable {
  String name;
  
  User(this.name);
  
  @override
  Map<String, dynamic> toJson() {
    return {'name': name};
  }
  
  @override
  bool isValid() {
    return name.isNotEmpty;
  }
}
```

---

# Summary

Interfaces provide contracts that enable polymorphism and loose coupling. In Dart, every class implicitly defines an interface, allowing you to implement multiple interfaces and create flexible, maintainable code.

---

# Next Steps

Now that you understand interfaces, continue to:

- [Sealed Classes](sealed-classes.md)
- [Base Classes](base-classes.md)
- [Interface Classes](interface-classes.md)
- [Factory Constructors](factory-constructors.md)

---

# Did You Know?

- Every class in Dart is also an interface
- A class can implement multiple interfaces
- Interfaces can have default implementations
- `@override` is recommended for clarity
- Interface methods must be implemented
- Abstract classes can implement interfaces
- Dart doesn't have a separate `interface` keyword