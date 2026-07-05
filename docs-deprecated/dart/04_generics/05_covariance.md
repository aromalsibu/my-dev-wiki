# Covariance

Understand how covariance works with generics in Dart.

---

# What is it?

Covariance is a type relationship that determines how generic types behave with respect to subtyping. In Dart, generics are invariant by default, meaning `List<Dog>` is not a subtype of `List<Animal>`. However, Dart supports covariance through the `covariant` keyword for certain situations.

---

# Why does it exist?

Covariance exists to:

- Understand type relationships in generics
- Enable type-safe method overriding
- Handle collections polymorphically
- Support flexible APIs
- Maintain type safety
- Prevent runtime errors

---

# Basic Covariance Concepts

## Understanding Covariance

```dart
// Class hierarchy
class Animal {
  void makeSound() => print('Animal sound');
}

class Dog extends Animal {
  @override
  void makeSound() => print('Woof!');
  
  void fetch() => print('Fetching...');
}

class Cat extends Animal {
  @override
  void makeSound() => print('Meow!');
  
  void purr() => print('Purring...');
}

// Variance example
void main() {
  // This works - Dog is a subtype of Animal
  Animal animal = Dog();
  
  // But generics are invariant
  List<Dog> dogs = [Dog(), Dog()];
  
  // This would NOT work - List<Dog> is not List<Animal>
  // List<Animal> animals = dogs; // Error!
  
  // Instead, you need to create a new list
  List<Animal> animals = dogs.map((dog) => dog as Animal).toList();
}
```

---

## Covariant Parameter

```dart
class AnimalProcessor {
  void process(Animal animal) {
    animal.makeSound();
  }
}

class DogProcessor extends AnimalProcessor {
  // Using covariant allows changing parameter type
  @override
  void process(covariant Dog animal) {
    animal.makeSound();
    animal.fetch();
  }
}

// Usage
AnimalProcessor processor = DogProcessor();
// This works because of covariant
processor.process(Dog()); // Woof! Fetching...
```

---

# Covariance in Collections

## Covariant Collections

```dart
class AnimalCollection {
  List<Animal> animals = [];
  
  void addAnimal(Animal animal) {
    animals.add(animal);
  }
  
  List<Animal> getAnimals() {
    return animals;
  }
}

class DogCollection extends AnimalCollection {
  @override
  void addAnimal(covariant Dog animal) {
    super.addAnimal(animal);
    print('Added dog: ${animal.runtimeType}');
  }
  
  @override
  List<Dog> getAnimals() {
    // This is covariant return type
    return animals.whereType<Dog>().toList();
  }
}

// Usage
var dogCollection = DogCollection();
dogCollection.addAnimal(Dog()); // Added dog: Dog
// dogCollection.addAnimal(Cat()); // Error! Cat is not covariant

var animals = dogCollection.getAnimals();
print(animals); // [Dog]
```

---

# Covariant Return Types

## Overriding with Covariant Return

```dart
class Factory {
  Animal create() {
    return Animal();
  }
  
  List<Animal> createList(int count) {
    return List.generate(count, (_) => create());
  }
}

class DogFactory extends Factory {
  @override
  Dog create() {
    // Covariant return type - Dog is subtype of Animal
    return Dog();
  }
  
  @override
  List<Dog> createList(int count) {
    // Covariant return type - List<Dog> is subtype of List<Animal>
    return List.generate(count, (_) => Dog());
  }
}

// Usage
Factory factory = DogFactory();
var animal = factory.create(); // Returns Dog but typed as Animal
var animals = factory.createList(3); // Returns List<Dog> but typed as List<Animal>

print(animal.runtimeType); // Dog
print(animals); // [Dog, Dog, Dog]
```

---

# Covariance and Type Safety

## Safe Covariance

```dart
class SafeCollection<T> {
  final List<T> _items = [];
  
  void add(T item) {
    _items.add(item);
  }
  
  T? find(bool Function(T) predicate) {
    try {
      return _items.firstWhere(predicate);
    } catch (_) {
      return null;
    }
  }
  
  List<T> getAll() {
    return List.unmodifiable(_items);
  }
}

// Subclass with covariant
class SafeDogCollection extends SafeCollection<Dog> {
  @override
  void add(covariant Animal animal) {
    if (animal is Dog) {
      super.add(animal);
    } else {
      throw ArgumentError('Only dogs allowed');
    }
  }
  
  List<Dog> getDogs() {
    return getAll();
  }
}

// Usage
var collection = SafeDogCollection();
collection.add(Dog()); // Works
// collection.add(Cat()); // Throws error
```

---

# Covariance in Practice

## Real-World Examples

```dart
// 1. Covariance in Event System
class Event {}
class UserEvent extends Event {
  final String userId;
  UserEvent(this.userId);
}
class ProductEvent extends Event {
  final String productId;
  ProductEvent(this.productId);
}

class EventHandler {
  void handle(Event event) {
    print('Handling event: ${event.runtimeType}');
  }
}

class UserEventHandler extends EventHandler {
  @override
  void handle(covariant UserEvent event) {
    print('Handling user event for: ${event.userId}');
  }
}

// 2. Covariance in Comparators
abstract class Comparator<T> {
  int compare(T a, T b);
}

class AnimalComparator implements Comparator<Animal> {
  @override
  int compare(Animal a, Animal b) {
    return a.runtimeType.toString().compareTo(b.runtimeType.toString());
  }
}

class DogComparator extends AnimalComparator {
  @override
  int compare(covariant Dog a, covariant Dog b) {
    return a.fetch().toString().compareTo(b.fetch().toString());
  }
}
```

---

# Covariance vs Contravariance

## Understanding Variance

```dart
// Class hierarchy
class Fruit {}
class Apple extends Fruit {}
class Orange extends Fruit {}

// Covariance example
class Box<T> {
  final T value;
  Box(this.value);
}

void processBox(Box<Fruit> box) {
  print('Processing fruit: ${box.value}');
}

// This would NOT work without covariance
// Box<Apple> appleBox = Box(Apple());
// processBox(appleBox); // Error!

// But with covariant parameters it can work
class CovariantBoxProcessor {
  void process(covariant Box<Fruit> box) {
    print('Processing: ${box.value}');
  }
}

// Usage
void main() {
  // Covariance allows this
  var processor = CovariantBoxProcessor();
  processor.process(Box(Apple())); // Processing: Apple
  processor.process(Box(Orange())); // Processing: Orange
}
```

---

# Best Practices

## Use Covariant Judiciously

```dart
// Good: Covariant when needed
class Base {
  void process(covariant Object input) {
    print('Processing: $input');
  }
}

class StringProcessor extends Base {
  @override
  void process(covariant String input) {
    print('Processing string: $input');
  }
}

// Bad: Overuse of covariant
class BadBase {
  void process(covariant dynamic input) {
    // Using covariant too broadly
  }
}
```

---

## Maintain Type Safety

```dart
// Good: Type-safe covariance
class SafeContainer<T> {
  void add(covariant T item) {
    if (item is T) {
      print('Adding: $item');
    } else {
      throw TypeError();
    }
  }
}

// Bad: Unsafe covariance
class UnsafeContainer<T> {
  void add(covariant Object item) {
    // No type checking
    print('Adding: $item');
  }
}
```

---

# Common Mistakes

## Incorrect Covariant Usage

Wrong:

```dart
class Parent {
  void process(covariant String value) { // Overly restrictive
    print('Processing: $value');
  }
}
```

Correct:

```dart
class Parent {
  void process(covariant Object value) {
    if (value is String) {
      print('Processing string: $value');
    } else {
      print('Processing: $value');
    }
  }
}
```

---

## Unnecessary Covariance

Wrong:

```dart
class Container<T> {
  void add(covariant T item) {
    // No need for covariant here
    _items.add(item);
  }
}
```

Correct:

```dart
class Container<T> {
  void add(T item) {
    // Regular generic parameter is fine
    _items.add(item);
  }
}
```

---

# Summary

Covariance allows flexible type relationships in generics while maintaining type safety. Use it judiciously when you need to override methods with more specific types.

---

# Next Steps

Now that you understand covariance, continue to:

- [Variance](variance.md)
- [Async Programming](async-programming.md)
- [Future](future.md)
- [async & await](async-await.md)

---

# Did You Know?

- Dart generics are invariant by default
- Covariance is specified with the `covariant` keyword
- Covariance is useful for method overriding
- Covariance helps maintain type safety
- Covariance works with parameters and return types
- Collections are invariant by default
- Covariance enables polymorphic behavior with generics