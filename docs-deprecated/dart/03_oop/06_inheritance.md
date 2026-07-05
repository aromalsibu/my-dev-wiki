# Inheritance

Understand how to create class hierarchies through inheritance in Dart.

---

# What is it?

Inheritance is a mechanism in object-oriented programming that allows a class (subclass/child) to inherit properties and methods from another class (superclass/parent). It enables code reuse, establishes hierarchical relationships, and supports polymorphism.

---

# Why does it exist?

Inheritance exists to:

- Reuse code across related classes
- Establish "is-a" relationships
- Enable polymorphism
- Support hierarchical classification
- Reduce code duplication
- Create extensible systems

---

# Basic Inheritance

## Extending a Class

```dart
// Base class (Superclass)
class Animal {
  String name;
  int age;
  
  Animal(this.name, this.age);
  
  void speak() {
    print('$name makes a sound');
  }
  
  String get type => 'Animal';
  
  void move() {
    print('$name moves');
  }
}

// Derived class (Subclass)
class Dog extends Animal {
  String breed;
  
  Dog(String name, int age, this.breed) : super(name, age);
  
  // Override method
  @override
  void speak() {
    print('$name barks');
  }
  
  // Override getter
  @override
  String get type => 'Dog';
  
  // Additional method
  void fetch() {
    print('$name fetches the ball');
  }
}

// Usage
var dog = Dog('Rex', 3, 'German Shepherd');
dog.speak(); // Rex barks (overridden)
dog.move(); // Rex moves (inherited)
dog.fetch(); // Rex fetches the ball (own method)
print(dog.type); // Dog (overridden)
print(dog.name); // Rex (inherited)
```

---

## Calling Superclass Methods

```dart
class Vehicle {
  String brand;
  String model;
  int year;
  
  Vehicle(this.brand, this.model, this.year);
  
  void start() {
    print('$brand $model engine starting...');
  }
  
  void stop() {
    print('$brand $model stopping...');
  }
  
  String get info => '$brand $model ($year)';
}

class Car extends Vehicle {
  int doors;
  String fuelType;
  
  Car(String brand, String model, int year, this.doors, this.fuelType)
      : super(brand, model, year);
  
  @override
  void start() {
    super.start(); // Call parent method
    print('Car is ready to drive');
  }
  
  @override
  void stop() {
    print('Applying brakes...');
    super.stop(); // Call parent after
  }
  
  @override
  String get info {
    return '${super.info}, $doors doors, $fuelType';
  }
  
  void honk() {
    print('Beep beep!');
  }
}

// Usage
var car = Car('Toyota', 'Camry', 2020, 4, 'Gasoline');
car.start();
// Toyota Camry engine starting...
// Car is ready to drive

car.stop();
// Applying brakes...
// Toyota Camry stopping...

print(car.info); // Toyota Camry (2020), 4 doors, Gasoline
```

---

# Inheritance Hierarchy

## Multi-level Inheritance

```dart
// Base class
class Employee {
  String name;
  int id;
  double salary;
  
  Employee(this.name, this.id, this.salary);
  
  void work() {
    print('$name is working');
  }
  
  double calculateBonus() {
    return salary * 0.1; // 10% bonus
  }
  
  String get position => 'Employee';
}

// Intermediate class
class Manager extends Employee {
  List<Employee> team = [];
  
  Manager(String name, int id, double salary) : super(name, id, salary);
  
  void addEmployee(Employee employee) {
    team.add(employee);
  }
  
  @override
  void work() {
    print('$name is managing the team');
  }
  
  @override
  double calculateBonus() {
    return salary * 0.2; // 20% bonus
  }
  
  @override
  String get position => 'Manager';
}

// Derived class
class Director extends Manager {
  String department;
  
  Director(String name, int id, double salary, this.department)
      : super(name, id, salary);
  
  @override
  void work() {
    print('$name is directing the $department department');
  }
  
  @override
  double calculateBonus() {
    return salary * 0.3; // 30% bonus
  }
  
  @override
  String get position => 'Director';
  
  void approveBudget() {
    print('$name approved budget for $department');
  }
}

// Usage
var employee = Employee('Alice', 1, 50000);
var manager = Manager('Bob', 2, 80000);
var director = Director('Charlie', 3, 120000, 'Engineering');

print(employee.position); // Employee
print(manager.position); // Manager
print(director.position); // Director

print(employee.calculateBonus()); // 5000
print(manager.calculateBonus()); // 16000
print(director.calculateBonus()); // 36000

director.approveBudget(); // Charlie approved budget for Engineering
```

---

# The `super` Keyword

## Using Super

```dart
class Shape {
  String color;
  
  Shape(this.color);
  
  void draw() {
    print('Drawing a $color shape');
  }
  
  double get area => 0;
}

class Circle extends Shape {
  double radius;
  
  Circle(String color, this.radius) : super(color);
  
  @override
  void draw() {
    super.draw(); // Call parent method
    print('Circle with radius $radius');
  }
  
  @override
  double get area => 3.14159 * radius * radius;
  
  // Using super in methods
  String getDescription() {
    return '${super.toString()} - Circle($radius)';
  }
}

class ColoredCircle extends Circle {
  String borderColor;
  
  ColoredCircle(String color, double radius, this.borderColor)
      : super(color, radius);
  
  @override
  void draw() {
    super.draw(); // Circle's draw
    print('Border color: $borderColor');
  }
  
  // Multi-level super
  String get fullDescription() {
    return '${super.getDescription()} with border $borderColor';
  }
}

// Usage
var circle = ColoredCircle('Red', 5, 'Black');
circle.draw();
// Drawing a Red shape
// Circle with radius 5
// Border color: Black

print(circle.fullDescription);
// Instance of 'ColoredCircle' - Circle(5.0) with border Black
```

---

# Method Overriding

## Overriding Methods

```dart
class Payment {
  double amount;
  String currency;
  
  Payment(this.amount, this.currency);
  
  void process() {
    print('Processing payment of $amount $currency');
  }
  
  bool validate() {
    return amount > 0;
  }
  
  String get status => 'Pending';
}

class CreditCardPayment extends Payment {
  String cardNumber;
  String expiryDate;
  
  CreditCardPayment(
    double amount,
    String currency,
    this.cardNumber,
    this.expiryDate,
  ) : super(amount, currency);
  
  @override
  void process() {
    if (!validate()) {
      throw Exception('Invalid payment');
    }
    print('Processing credit card payment...');
    print('Card: ****${cardNumber.substring(cardNumber.length - 4)}');
    print('Amount: $amount $currency');
    print('Payment successful!');
  }
  
  @override
  bool validate() {
    return super.validate() && 
           cardNumber.length >= 16 && 
           expiryDate.isNotEmpty;
  }
  
  @override
  String get status => 'Completed';
}

class PayPalPayment extends Payment {
  String email;
  
  PayPalPayment(double amount, String currency, this.email)
      : super(amount, currency);
  
  @override
  void process() {
    print('Processing PayPal payment for $email');
    print('Amount: $amount $currency');
    print('Payment successful!');
  }
  
  @override
  bool validate() {
    return super.validate() && email.contains('@');
  }
  
  @override
  String get status => 'Completed';
}

// Usage
void processPayment(Payment payment) {
  print('Validating payment...');
  if (payment.validate()) {
    print('Payment is valid');
    payment.process();
    print('Status: ${payment.status}');
  } else {
    print('Invalid payment');
  }
  print('---');
}

var creditCard = CreditCardPayment(100, 'USD', '1234567890123456', '12/25');
var paypal = PayPalPayment(50, 'EUR', 'user@example.com');

processPayment(creditCard);
processPayment(paypal);
```

---

# Inheritance and Constructors

## Constructor Chaining

```dart
class Product {
  final String id;
  final String name;
  final double price;
  
  Product(this.id, this.name, this.price);
  
  Product.basic(String name, double price)
      : id = DateTime.now().millisecondsSinceEpoch.toString(),
        name = name,
        price = price;
  
  Product.fromJson(Map<String, dynamic> json)
      : id = json['id'] as String,
        name = json['name'] as String,
        price = json['price'] as double;
  
  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'price': price,
  };
}

class DigitalProduct extends Product {
  final String downloadUrl;
  final double fileSize;
  
  DigitalProduct(
    String id,
    String name,
    double price,
    this.downloadUrl,
    this.fileSize,
  ) : super(id, name, price);
  
  DigitalProduct.basic(String name, double price, this.downloadUrl, this.fileSize)
      : super.basic(name, price);
  
  DigitalProduct.fromJson(Map<String, dynamic> json)
      : downloadUrl = json['downloadUrl'] as String,
        fileSize = json['fileSize'] as double,
        super.fromJson(json);
  
  @override
  Map<String, dynamic> toJson() => {
    ...super.toJson(),
    'downloadUrl': downloadUrl,
    'fileSize': fileSize,
  };
}

// Usage
var product1 = DigitalProduct('1', 'E-book', 9.99, 'https://...', 2.5);
var product2 = DigitalProduct.basic('Video Course', 49.99, 'https://...', 1024.0);
var product3 = DigitalProduct.fromJson({
  'id': '3',
  'name': 'Software',
  'price': 99.99,
  'downloadUrl': 'https://...',
  'fileSize': 512.0,
});

print(product1.name); // E-book
print(product2.downloadUrl); // https://...
print(product3.toJson());
```

---

# Polymorphism

## Using Polymorphism

```dart
abstract class Employee {
  String name;
  
  Employee(this.name);
  
  double calculateSalary();
  String get role;
}

class Developer extends Employee {
  double baseSalary;
  int projectsCompleted;
  
  Developer(String name, this.baseSalary, this.projectsCompleted)
      : super(name);
  
  @override
  double calculateSalary() {
    return baseSalary + (projectsCompleted * 500);
  }
  
  @override
  String get role => 'Developer';
}

class Designer extends Employee {
  double baseSalary;
  int designsCreated;
  
  Designer(String name, this.baseSalary, this.designsCreated)
      : super(name);
  
  @override
  double calculateSalary() {
    return baseSalary + (designsCreated * 300);
  }
  
  @override
  String get role => 'Designer';
}

class Manager extends Employee {
  double baseSalary;
  List<Employee> team;
  
  Manager(String name, this.baseSalary, this.team) : super(name);
  
  @override
  double calculateSalary() {
    return baseSalary + (team.length * 1000);
  }
  
  @override
  String get role => 'Manager';
}

// Polymorphic usage
void printEmployeeInfo(Employee employee) {
  print('${employee.name} (${employee.role})');
  print('Salary: \$${employee.calculateSalary().toStringAsFixed(2)}');
  print('---');
}

void processEmployees(List<Employee> employees) {
  double totalSalary = 0;
  
  for (var employee in employees) {
    printEmployeeInfo(employee);
    totalSalary += employee.calculateSalary();
  }
  
  print('Total salary: \$${totalSalary.toStringAsFixed(2)}');
}

// Usage
var employees = [
  Developer('Alice', 50000, 10),
  Designer('Bob', 45000, 8),
  Manager('Charlie', 60000, []),
];

processEmployees(employees);
```

---

# Best Practices

## Use Inheritance for "Is-A" Relationships

```dart
// Good: Car IS-A Vehicle
class Vehicle { /* ... */ }
class Car extends Vehicle { /* ... */ }

// Good: Dog IS-A Animal
class Animal { /* ... */ }
class Dog extends Animal { /* ... */ }

// Bad: Car HAS-A Engine (use composition instead)
class Car extends Engine { /* ... */ } // Wrong!

// Better: Composition
class Car {
  Engine engine; // HAS-A relationship
}
```

---

## Keep Inheritance Shallow

```dart
// Bad: Deep inheritance tree
// Animal -> Mammal -> Canine -> Dog -> GermanShepherd

// Good: Shallow inheritance
// Animal -> Dog (with breed as property)
class Animal { /* ... */ }
class Dog extends Animal { 
  String breed; // Breed is property, not subclass
}
```

---

## Use @override Annotation

```dart
// Good: With @override
class Dog extends Animal {
  @override
  void speak() {
    print('Woof!');
  }
}

// Bad: Missing @override
class Cat extends Animal {
  void speak() { // Should use @override
    print('Meow!');
  }
}
```

---

# Common Mistakes

## Constructor Without Super

Wrong:

```dart
class Animal {
  String name;
  Animal(this.name);
}

class Dog extends Animal {
  String breed;
  Dog(this.breed) : super(''); // Must call super
}
```

Correct:

```dart
class Dog extends Animal {
  String breed;
  Dog(String name, this.breed) : super(name);
}
```

---

## Confusing IS-A with HAS-A

Wrong:

```dart
// Car IS-A Engine? No! Car HAS-A Engine
class Car extends Engine { /* ... */ }
```

Correct:

```dart
// Car HAS-A Engine
class Car {
  final Engine engine;
  Car(this.engine);
}
```

---

# Summary

Inheritance is a fundamental OOP concept that enables code reuse and hierarchical relationships. Use it wisely for IS-A relationships, and prefer composition for HAS-A relationships.

---

# Next Steps

Now that you understand inheritance, continue to:

- [Mixins](mixins.md)
- [Abstract Classes](abstract-classes.md)
- [Interfaces](interfaces.md)
- [Sealed Classes](sealed-classes.md)

---

# Did You Know?

- Dart supports single inheritance only (one superclass)
- All classes inherit from Object
- Use `super` to call parent methods
- `@override` is optional but recommended
- Inheritance is transitive (grandchild inherits from grandparent)
- Constructors are not inherited
- You can override methods and getters/setters