# Mixins

Understand how to reuse code across multiple class hierarchies using mixins in Dart.

---

# What is it?

Mixins are a way to reuse code across multiple classes without using inheritance. They allow you to add functionality to classes that are not in the same inheritance hierarchy, providing a flexible and composable approach to code reuse.

---

# Why does it exist?

Mixins exist to:

- Reuse code across unrelated classes
- Compose behaviors from multiple sources
- Avoid deep inheritance hierarchies
- Add functionality without subclassing
- Create modular and reusable components
- Support multiple inheritance of behavior

---

# Basic Mixins

## Simple Mixin

```dart
// Define a mixin
mixin Flyable {
  void fly() {
    print('Flying...');
  }
}

mixin Swimmable {
  void swim() {
    print('Swimming...');
  }
}

mixin Walkable {
  void walk() {
    print('Walking...');
  }
}

// Use mixins with 'with'
class Duck extends Animal with Walkable, Swimmable, Flyable {
  Duck(String name) : super(name);
}

class Fish extends Animal with Swimmable {
  Fish(String name) : super(name);
}

class Dog extends Animal with Walkable {
  Dog(String name) : super(name);
}

// Usage
var duck = Duck('Donald');
duck.walk(); // Walking...
duck.swim(); // Swimming...
duck.fly(); // Flying...

var fish = Fish('Nemo');
fish.swim(); // Swimming...

var dog = Dog('Rex');
dog.walk(); // Walking...
```

---

## Mixin with State

```dart
mixin Counter {
  int _count = 0;
  
  int get count => _count;
  
  void increment() {
    _count++;
  }
  
  void decrement() {
    if (_count > 0) {
      _count--;
    }
  }
  
  void reset() {
    _count = 0;
  }
}

mixin Logger {
  void log(String message) {
    print('${DateTime.now()}: $message');
  }
}

class CounterApp with Counter, Logger {
  void run() {
    log('Starting counter');
    increment();
    increment();
    log('Count: $count');
    decrement();
    log('Count after decrement: $count');
  }
}

// Usage
var app = CounterApp();
app.run();
// Output:
// 2024-01-01 00:00:00.000: Starting counter
// 2024-01-01 00:00:00.000: Count: 2
// 2024-01-01 00:00:00.000: Count after decrement: 1
```

---

# Advanced Mixins

## Mixin with Constraints

```dart
// Mixin that requires the class to implement a specific interface
mixin ComparableMixin<T> on Comparable<T> {
  bool isGreater(T other) => compareTo(other) > 0;
  bool isLess(T other) => compareTo(other) < 0;
  bool isEqual(T other) => compareTo(other) == 0;
  bool isGreaterOrEqual(T other) => compareTo(other) >= 0;
  bool isLessOrEqual(T other) => compareTo(other) <= 0;
}

// Can only be used on classes that implement Comparable
class Person implements Comparable<Person> {
  final String name;
  final int age;
  
  Person(this.name, this.age);
  
  @override
  int compareTo(Person other) => age.compareTo(other.age);
}

class ExtendedPerson extends Person with ComparableMixin<Person> {
  ExtendedPerson(String name, int age) : super(name, age);
}

// Usage
var alice = ExtendedPerson('Alice', 25);
var bob = ExtendedPerson('Bob', 30);

print(alice.isLess(bob)); // true
print(alice.isGreater(bob)); // false
print(bob.isGreater(alice)); // true
```

---

## Mixin with Lifecycle Methods

```mixin
mixin LifecycleMixin {
  // Initialization
  void init() {
    print('Initializing...');
  }
  
  // Cleanup
  void dispose() {
    print('Disposing...');
  }
  
  // Lifecycle hooks
  void onStart() {
    print('Starting...');
  }
  
  void onStop() {
    print('Stopping...');
  }
  
  void onPause() {
    print('Pausing...');
  }
  
  void onResume() {
    print('Resuming...');
  }
}

class Application with LifecycleMixin {
  void run() {
    init();
    onStart();
    // Application running...
    onPause();
    onResume();
    onStop();
    dispose();
  }
}

// Usage
var app = Application();
app.run();
// Output:
// Initializing...
// Starting...
// Pausing...
// Resuming...
// Stopping...
// Disposing...
```

---

# Mixin vs Inheritance

## Comparison

```dart
// Inheritance approach
class Animal {
  void eat() => print('Eating');
  void sleep() => print('Sleeping');
}

class Bird extends Animal {
  void fly() => print('Flying');
}

class Fish extends Animal {
  void swim() => print('Swimming');
}

// Mixins approach
mixin Eater {
  void eat() => print('Eating');
}

mixin Sleeper {
  void sleep() => print('Sleeping');
}

mixin Flyer {
  void fly() => print('Flying');
}

mixin Swimmer {
  void swim() => print('Swimming');
}

class Bird with Eater, Sleeper, Flyer {}
class Fish with Eater, Sleeper, Swimmer {}
class Duck with Eater, Sleeper, Flyer, Swimmer {}

// More flexible: Duck can do both fly and swim!
```

---

# Real-World Example

## UI Components with Mixins

```dart
mixin Draggable {
  void onDragStart() => print('Drag started');
  void onDragUpdate() => print('Drag updating');
  void onDragEnd() => print('Drag ended');
  
  void enableDragging() => print('Dragging enabled');
  void disableDragging() => print('Dragging disabled');
}

mixin Resizable {
  void onResizeStart() => print('Resize started');
  void onResizeUpdate() => print('Resizing');
  void onResizeEnd() => print('Resize ended');
  
  void enableResizing() => print('Resizing enabled');
  void disableResizing() => print('Resizing disabled');
}

mixin Clickable {
  void onTap() => print('Tapped');
  void onDoubleTap() => print('Double tapped');
  void onLongPress() => print('Long pressed');
}

mixin Hoverable {
  void onHoverEnter() => print('Hover entered');
  void onHoverExit() => print('Hover exited');
}

// Base class
class Widget {
  String id;
  double x, y;
  double width, height;
  
  Widget(this.id, {this.x = 0, this.y = 0, this.width = 100, this.height = 100});
  
  void render() => print('Rendering widget: $id');
}

// Different widget types with different capabilities
class Button extends Widget with Clickable {
  String text;
  
  Button(String id, this.text) : super(id);
  
  void click() {
    print('Button clicked: $text');
    onTap();
  }
}

class ImageWidget extends Widget with Draggable, Resizable {
  String url;
  
  ImageWidget(String id, this.url) : super(id);
}

class Panel extends Widget with Draggable, Resizable, Hoverable {
  List<Widget> children = [];
  
  Panel(String id) : super(id);
  
  void addChild(Widget child) => children.add(child);
}

class InteractivePanel extends Widget with Draggable, Resizable, Clickable, Hoverable {
  Panel(String id) : super(id);
}

// Usage
var button = Button('btn1', 'Click Me');
button.render(); // Rendering widget: btn1
button.click(); // Button clicked: Click Me, Tapped

var image = ImageWidget('img1', 'photo.jpg');
image.enableDragging(); // Dragging enabled
image.onDragStart(); // Drag started

var panel = Panel('panel1');
panel.onHoverEnter(); // Hover entered
panel.enableResizing(); // Resizing enabled
panel.onResizeStart(); // Resize started
```

---

# Best Practices

## Use Mixins for Reusable Behaviors

```dart
// Good: Reusable behaviors
mixin Loggable {
  void log(String message) {
    print('${DateTime.now()}: $message');
  }
}

mixin Validatable {
  bool isValid();
}

mixin Serializable {
  Map<String, dynamic> toJson();
}

// Bad: Mixin with specific domain logic
mixin UserManager {
  void deleteUser(User user) { /* ... */ }
  // This should be in a service class
}
```

---

## Keep Mixins Focused

```dart
// Good: Single responsibility
mixin Logger {
  void log(String message) { /* ... */ }
}

mixin ErrorHandler {
  void handleError(Exception error) { /* ... */ }
}

mixin PerformanceMonitor {
  void startTimer() { /* ... */ }
  void endTimer() { /* ... */ }
}

// Bad: Too many responsibilities
mixin Utils {
  void log(String message) { /* ... */ }
  void handleError(Exception error) { /* ... */ }
  void startTimer() { /* ... */ }
  void endTimer() { /* ... */ }
  // Too many unrelated behaviors
}
```

---

## Use `on` for Constraints

```dart
// Good: Constrained mixin
mixin ComparableMixin<T> on Comparable<T> {
  bool isGreater(T other) => compareTo(other) > 0;
}

// Bad: Unconstrained mixin
mixin ComparableMixin<T> {
  bool isGreater(T other) => compareTo(other) > 0; // Error: compareTo may not exist
}
```

---

# Common Mistakes

## Using Mixins Where Inheritance is Better

Wrong:

```dart
// Using mixin for IS-A relationship
mixin Vehicle {
  void startEngine() { /* ... */ }
}

class Car with Vehicle { /* ... */ }
// Should be inheritance: class Car extends Vehicle
```

Correct:

```dart
// Inheritance for IS-A relationship
class Vehicle { /* ... */ }
class Car extends Vehicle { /* ... */ }
```

---

## Not Using `on` Correctly

Wrong:

```dart
mixin Logger {
  void log(String message) {
    print('$error: $message'); // Error: error doesn't exist
  }
}
```

Correct:

```dart
mixin Logger on ExceptionHandler {
  void log(String message) {
    print('$error: $message'); // error comes from ExceptionHandler
  }
}

class ExceptionHandler {
  String error = 'Error';
}

class MyClass extends ExceptionHandler with Logger { /* ... */ }
```

---

# Summary

Mixins provide a powerful way to reuse code across unrelated classes. They offer more flexibility than inheritance and allow you to compose behaviors from multiple sources.

---

# Next Steps

Now that you understand mixins, continue to:

- [Abstract Classes](abstract-classes.md)
- [Interfaces](interfaces.md)
- [Sealed Classes](sealed-classes.md)
- [Factory Constructors](factory-constructors.md)

---

# Did You Know?

- Mixins were introduced in Dart 2.0
- You can use multiple mixins with `with`
- Mixins can have state (fields)
- Mixins can be constrained with `on`
- Mixins cannot have constructors
- Mixins can override methods
- Mixin order matters (rightmost overrides)