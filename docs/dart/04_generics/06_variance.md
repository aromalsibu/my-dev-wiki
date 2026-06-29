# Variance

Understand how variance (covariance and contravariance) works with generics in Dart.

---

# What is it?

Variance describes how subtyping between generic types relates to subtyping between their type arguments. In Dart, generics are invariant by default, but covariance and contravariance can be specified using the `covariant` keyword.

---

# Why does it exist?

Variance exists to:

- Understand type relationships in generics
- Enable type-safe method overriding
- Handle collections polymorphically
- Support flexible APIs
- Maintain type safety
- Prevent runtime errors

---

# Covariance

## What is Covariance?

```dart
// Covariance: If A is subtype of B, then Generic<A> is subtype of Generic<B>
// This means you can use Generic<Dog> where Generic<Animal> is expected

class Animal {
  void makeSound() => print('Animal sound');
}

class Dog extends Animal {
  @override
  void makeSound() => print('Woof!');
  void fetch() => print('Fetching...');
}

// Covariance demonstration
void processAnimals(List<Animal> animals) {
  for (var animal in animals) {
    animal.makeSound();
  }
}

void main() {
  // This works with covariance
  List<Dog> dogs = [Dog(), Dog()];
  processAnimals(dogs); // Covariance allows this
}
```

---

## Covariance with Collections

```dart
class AnimalShelter {
  List<Animal> animals = [];
  
  void addAnimal(Animal animal) {
    animals.add(animal);
  }
  
  List<Animal> getAnimals() => animals;
}

class DogShelter extends AnimalShelter {
  @override
  void addAnimal(covariant Dog animal) {
    super.addAnimal(animal);
    print('Added dog: ${animal.runtimeType}');
  }
  
  @override
  List<Dog> getAnimals() {
    // Covariant return type
    return animals.whereType<Dog>().toList();
  }
}

// Usage
var shelter = DogShelter();
shelter.addAnimal(Dog()); // Added dog: Dog
// shelter.addAnimal(Cat()); // Error! Cat is not covariant

var animals = shelter.getAnimals(); // Returns List<Dog>
```

---

## Covariant Parameters

```dart
class Box<T> {
  final T value;
  Box(this.value);
  
  void process(T value) {
    print('Processing: $value');
  }
}

class StringBox extends Box<String> {
  StringBox(String value) : super(value);
  
  @override
  void process(covariant Object value) {
    // Covariant allows broader type
    if (value is String) {
      print('Processing string: $value');
    } else {
      print('Processing object: $value');
    }
  }
}

// Usage
var box = StringBox('Hello');
box.process('World'); // Processing string: World
box.process(42); // Processing object: 42
```

---

# Contravariance

## What is Contravariance?

```dart
// Contravariance: If A is subtype of B, then Generic<B> is subtype of Generic<A>
// This means you can use Generic<Animal> where Generic<Dog> is expected

abstract class Comparator<T> {
  int compare(T a, T b);
}

// Covariant comparators
class AnimalComparator implements Comparator<Animal> {
  @override
  int compare(Animal a, Animal b) {
    return a.runtimeType.toString().compareTo(b.runtimeType.toString());
  }
}

class DogComparator extends AnimalComparator {
  @override
  int compare(covariant Dog a, covariant Dog b) {
    // More specific comparison
    return a.toString().compareTo(b.toString());
  }
}

// Usage
void compareDogs(Comparator<Dog> comparator, Dog a, Dog b) {
  var result = comparator.compare(a, b);
  print('Comparison result: $result');
}

void main() {
  // This works due to contravariance
  var comparator = AnimalComparator();
  compareDogs(comparator, Dog(), Dog()); // Works!
}
```

---

## Contravariant Example

```dart
abstract class Event {}
class ClickEvent extends Event {}
class HoverEvent extends Event {}

abstract class EventHandler<T extends Event> {
  void handle(T event);
}

// Contravariant event handler
class ClickHandler extends EventHandler<ClickEvent> {
  @override
  void handle(covariant ClickEvent event) {
    print('Handling click: $event');
  }
}

class EventProcessor {
  void processEvent(EventHandler<Event> handler, Event event) {
    handler.handle(event);
  }
}

// Usage
void main() {
  var processor = EventProcessor();
  var clickHandler = ClickHandler();
  
  // This works due to covariance
  processor.processEvent(clickHandler, ClickEvent());
  
  // But not with a different event type
  // processor.processEvent(clickHandler, HoverEvent()); // Error!
}
```

---

# Variance in Practice

## Real-World Examples

```dart
// 1. Generic Repository
abstract class Repository<T> {
  void save(T entity);
  T? find(String id);
  List<T> findAll();
}

class UserRepository implements Repository<User> {
  @override
  void save(covariant Object entity) {
    // Covariant parameter
    if (entity is User) {
      print('Saving user: ${entity.name}');
    } else {
      throw ArgumentError('Expected User entity');
    }
  }
  
  @override
  User? find(String id) {
    print('Finding user: $id');
    return null;
  }
  
  @override
  List<User> findAll() {
    print('Finding all users');
    return [];
  }
}

// 2. Generic Builder
abstract class Builder<T> {
  T build();
}

class UserBuilder implements Builder<User> {
  final String name;
  final int age;
  
  UserBuilder(this.name, this.age);
  
  @override
  User build() {
    return User(name, age);
  }
}

// 3. Generic Handler
abstract class Handler<T> {
  void handle(T event);
}

class LogHandler extends Handler<Event> {
  @override
  void handle(covariant Event event) {
    print('Logging: ${event.runtimeType}');
  }
}
```

---

# Variance with Lists

## List Variance Example

```dart
void processList(List<Object> items) {
  for (var item in items) {
    print('Processing: $item');
  }
}

void processStringList(List<String> items) {
  for (var item in items) {
    print('String: $item');
  }
}

// Usage
void main() {
  // Covariance - List<Cat> can be used as List<Animal>
  List<Cat> cats = [Cat(), Cat()];
  processList(cats); // Works!
  
  // But cannot add different types
  // cats.add(Dog()); // Error: Type mismatch
  
  // List variance with modification
  List<Animal> animals = [Dog(), Cat()];
  // animals.add('string'); // Error: Type mismatch
}
```

---

# Variance Best Practices

## Use Covariant for Parameter Override

```dart
// Good: Covariant parameter
class AnimalProcessor {
  void process(Animal animal) {
    animal.makeSound();
  }
}

class DogProcessor extends AnimalProcessor {
  @override
  void process(covariant Dog animal) {
    animal.makeSound();
    animal.fetch();
  }
}

// Bad: Incorrect covariance
class BadProcessor extends AnimalProcessor {
  @override
  void process(covariant String animal) {
    // String is not a subtype of Animal!
  }
}
```

---

## Use Covariant Return Types

```dart
// Good: Covariant return type
class Factory {
  Animal create() => Animal();
}

class DogFactory extends Factory {
  @override
  Dog create() => Dog(); // Covariant return
}

// Good: Multiple levels
class AnimalCreator {
  Animal createAnimal() => Animal();
}

class MammalCreator extends AnimalCreator {
  @override
  Mammal createAnimal() => Mammal();
}

class DogCreator extends MammalCreator {
  @override
  Dog createAnimal() => Dog();
}
```

---

# Common Mistakes

## Incorrect Covariant Usage

Wrong:

```dart
class Container<T> {
  void add(covariant T item) {
    // Overuse of covariant
  }
}
```

Correct:

```dart
class Container<T> {
  void add(T item) {
    // Regular generic is fine
  }
}
```

---

## Covariance with Wrong Type

Wrong:

```dart
class AnimalProcessor {
  void process(Animal animal) {}
}

class DogProcessor extends AnimalProcessor {
  @override
  void process(covariant String animal) {
    // String is not a subtype of Animal
  }
}
```

Correct:

```dart
class AnimalProcessor {
  void process(Animal animal) {}
}

class DogProcessor extends AnimalProcessor {
  @override
  void process(covariant Dog animal) {
    // Dog is a subtype of Animal
  }
}
```

---

# Summary

Variance allows flexible type relationships in generics. Covariance allows subtypes to be used where supertypes are expected, while contravariance allows supertypes to be used where subtypes are expected.

---

# Next Steps

Now that you understand variance, continue to:

- [Async Programming](async-programming.md)
- [Future](future.md)
- [async & await](async-await.md)
- [FutureOr](future-or.md)

---

# Did You Know?

- Dart generics are invariant by default
- Covariance is specified with the `covariant` keyword
- Covariance is useful for method overriding
- Contravariance is less common in Dart
- Covariance helps maintain type safety
- Collections are invariant by default
- Covariance enables polymorphic behavior with generics