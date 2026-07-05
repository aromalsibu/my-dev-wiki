# Methods

Understand how to define and use methods in Dart classes.

---

# What is it?

Methods are functions that belong to a class and define the behavior of objects. They can access and modify the object's state (fields) and perform operations. Methods are essential for implementing the functionality of classes in object-oriented programming.

---

# Why does it exist?

Methods exist to:

- Define object behavior
- Encapsulate operations on data
- Provide interfaces for interacting with objects
- Implement business logic
- Organize code into reusable units
- Enable polymorphism

---

# Instance Methods

## Basic Methods

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
  
  // Method with side effects
  void printResult(int a, int b, String operation) {
    var result = switch (operation) {
      'add' => add(a, b),
      'subtract' => subtract(a, b),
      'multiply' => multiply(a, b),
      _ => throw ArgumentError('Unknown operation'),
    };
    print('$a $operation $b = $result');
  }
}

// Usage
var calc = Calculator();
print(calc.add(5, 3)); // 8
calc.printResult(5, 3, 'multiply'); // 5 multiply 3 = 15
```

---

## Methods Accessing Fields

```dart
class Person {
  String name;
  int age;
  List<String> hobbies;
  
  Person(this.name, this.age, this.hobbies);
  
  // Method accessing instance fields
  String get introduction {
    return 'Hi, I am $name, $age years old';
  }
  
  void addHobby(String hobby) {
    hobbies.add(hobby);
  }
  
  bool hasHobby(String hobby) {
    return hobbies.contains(hobby);
  }
  
  void celebrateBirthday() {
    age++;
    print('Happy birthday $name! You are now $age');
  }
  
  String getHobbiesList() {
    return hobbies.isEmpty 
        ? 'No hobbies yet' 
        : 'Hobbies: ${hobbies.join(', ')}';
  }
}

// Usage
var person = Person('Alice', 25, ['reading', 'swimming']);
print(person.introduction); // Hi, I am Alice, 25 years old
person.addHobby('coding');
print(person.getHobbiesList()); // Hobbies: reading, swimming, coding
person.celebrateBirthday(); // Happy birthday Alice! You are now 26
```

---

# Static Methods

## Utility Methods

```dart
class MathUtils {
  // Static methods (can be called without instance)
  static int add(int a, int b) => a + b;
  static int subtract(int a, int b) => a - b;
  static int multiply(int a, int b) => a * b;
  static double divide(int a, int b) => a / b;
  
  static int sum(List<int> numbers) {
    return numbers.reduce((a, b) => a + b);
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
  
  static int factorial(int n) {
    if (n < 0) throw ArgumentError('Cannot factorial negative');
    return n <= 1 ? 1 : n * factorial(n - 1);
  }
}

// Usage (no instance needed)
var numbers = [1, 2, 3, 4, 5];
print(MathUtils.sum(numbers)); // 15
print(MathUtils.average(numbers)); // 3.0
print(MathUtils.max(numbers)); // 5
print(MathUtils.factorial(5)); // 120
```

---

# Private Methods

## Internal Helper Methods

```dart
class UserValidator {
  // Public methods
  bool validateUser(String email, String password) {
    if (!_validateEmail(email)) {
      return false;
    }
    if (!_validatePassword(password)) {
      return false;
    }
    return true;
  }
  
  // Private methods (starts with _)
  bool _validateEmail(String email) {
    return email.contains('@') && email.contains('.');
  }
  
  bool _validatePassword(String password) {
    if (password.length < 8) {
      return false;
    }
    if (!_hasUpperCase(password)) {
      return false;
    }
    if (!_hasLowerCase(password)) {
      return false;
    }
    if (!_hasDigit(password)) {
      return false;
    }
    return true;
  }
  
  bool _hasUpperCase(String password) {
    return RegExp(r'[A-Z]').hasMatch(password);
  }
  
  bool _hasLowerCase(String password) {
    return RegExp(r'[a-z]').hasMatch(password);
  }
  
  bool _hasDigit(String password) {
    return RegExp(r'\d').hasMatch(password);
  }
}

// Usage
var validator = UserValidator();
print(validator.validateUser('user@example.com', 'Password123')); // true
print(validator.validateUser('invalid', 'short')); // false

// Can't access private methods
// validator._validateEmail('test'); // Error: Private method
```

---

# Method Overriding

## Inherited Methods

```dart
class Animal {
  String name;
  
  Animal(this.name);
  
  // Method to override
  void speak() {
    print('$name makes a sound');
  }
  
  String get type => 'Animal';
  
  void move() {
    print('$name moves');
  }
}

class Dog extends Animal {
  String breed;
  
  Dog(String name, this.breed) : super(name);
  
  // Override method
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
  
  // Call parent method
  void introduce() {
    super.move(); // Calls parent method
    print('I am a $breed dog named $name');
  }
}

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
dog.introduce(); // Rex moves \n I am a German Shepherd dog named Rex

var cat = Cat('Whiskers');
cat.speak(); // Whiskers meows
cat.purr(); // Whiskers purrs
```

---

# Polymorphic Methods

## Using Methods Polymorphically

```dart
abstract class Shape {
  // Abstract method (must be overridden)
  double get area;
  double get perimeter;
  
  // Concrete method
  String get description => 'Shape with area $area and perimeter $perimeter';
  
  // Overridable method
  void scale(double factor);
}

class Circle extends Shape {
  double radius;
  
  Circle(this.radius);
  
  @override
  double get area => math.pi * radius * radius;
  
  @override
  double get perimeter => 2 * math.pi * radius;
  
  @override
  void scale(double factor) {
    radius *= factor;
  }
  
  @override
  String get description => 'Circle with radius $radius';
}

class Rectangle extends Shape {
  double width;
  double height;
  
  Rectangle(this.width, this.height);
  
  @override
  double get area => width * height;
  
  @override
  double get perimeter => 2 * (width + height);
  
  @override
  void scale(double factor) {
    width *= factor;
    height *= factor;
  }
  
  @override
  String get description => 'Rectangle $width x $height';
}

// Polymorphic usage
void processShapes(List<Shape> shapes) {
  for (var shape in shapes) {
    print(shape.description);
    shape.scale(2);
    print('After scaling: ${shape.description}');
    print('---');
  }
}

// Usage
var shapes = [
  Circle(5),
  Rectangle(10, 20),
];

processShapes(shapes);
// Circle with radius 5
// After scaling: Circle with radius 10
// Rectangle 10 x 20
// After scaling: Rectangle 20 x 40
```

---

# Method Chaining

## Returning this

```dart
class Builder {
  String _name = '';
  int _age = 0;
  String _email = '';
  bool _isActive = false;
  
  // Each method returns this for chaining
  Builder setName(String name) {
    _name = name;
    return this;
  }
  
  Builder setAge(int age) {
    _age = age;
    return this;
  }
  
  Builder setEmail(String email) {
    _email = email;
    return this;
  }
  
  Builder setActive(bool active) {
    _isActive = active;
    return this;
  }
  
  // Build method
  User build() {
    return User(
      name: _name,
      age: _age,
      email: _email,
      isActive: _isActive,
    );
  }
}

class User {
  final String name;
  final int age;
  final String email;
  final bool isActive;
  
  User({
    required this.name,
    required this.age,
    required this.email,
    required this.isActive,
  });
  
  @override
  String toString() => 'User($name, $age, $email, $isActive)';
}

// Usage - Method chaining
var user = Builder()
    .setName('Alice')
    .setAge(25)
    .setEmail('alice@example.com')
    .setActive(true)
    .build();

print(user); // User(Alice, 25, alice@example.com, true)
```

---

# Best Practices

## Keep Methods Focused

```dart
// Good: Single responsibility
class UserService {
  void validateUser(User user) { /* ... */ }
  void saveUser(User user) { /* ... */ }
  void sendWelcomeEmail(User user) { /* ... */ }
}

// Bad: Too many responsibilities
class BadUserService {
  void processUser(User user) {
    validateUser(user);
    saveUser(user);
    sendWelcomeEmail(user);
    updateAnalytics(user);
  }
  // All methods in one class
}
```

---

## Use Descriptive Names

```dart
// Good: Descriptive names
class Calculator {
  int add(int a, int b) => a + b;
  int subtract(int a, int b) => a - b;
  double divide(int a, int b) => a / b;
}

// Bad: Vague names
class BadCalc {
  int calc1(int a, int b) => a + b;
  int calc2(int a, int b) => a - b;
}
```

---

## Use @override Annotation

```dart
// Good: Using @override
class Dog extends Animal {
  @override
  void speak() {
    print('Woof!');
  }
}

// Bad: Missing @override
class Cat extends Animal {
  void speak() {
    print('Meow!');
  }
}
```

---

# Common Mistakes

## Static Method Accessing Instance

Wrong:

```dart
class Person {
  String name;
  
  static void setName(String name) {
    // this.name = name; // Error: Can't use this in static method
  }
}
```

Correct:

```dart
class Person {
  String name;
  
  void setName(String name) {
    this.name = name; // Instance method
  }
}
```

---

## Method Name Conflicts

Wrong:

```dart
class Animal {
  void speak() {}
}

class Dog extends Animal {
  void speak() {} // Missing @override
}
```

Correct:

```dart
class Animal {
  void speak() {}
}

class Dog extends Animal {
  @override
  void speak() {}
}
```

---

# Summary

Methods define the behavior of objects. Understanding instance methods, static methods, private methods, method overriding, and polymorphic usage is essential for effective object-oriented programming.

---

# Next Steps

Now that you understand methods, continue to:

- [Getters & Setters](getters-setters.md)
- [Inheritance](inheritance.md)
- [Mixins](mixins.md)
- [Abstract Classes](abstract-classes.md)

---

# Did You Know?

- Methods are functions that belong to a class
- Static methods can be called without an instance
- Private methods start with `_`
- @override annotates overridden methods
- Methods can return `this` for chaining
- Abstract methods have no implementation
- Method overloading is not supported in Dart (use different names)