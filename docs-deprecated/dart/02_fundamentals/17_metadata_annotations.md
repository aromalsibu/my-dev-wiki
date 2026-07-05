# Metadata (Annotations)

Understand how to use metadata to annotate Dart code with additional information.

---

# What is it?

Metadata (also called annotations) is a way to add extra information to Dart code elements like classes, methods, variables, and parameters. This information can be used by tools, frameworks, and at runtime to modify behavior or provide additional context.

---

# Why does it exist?

Metadata exists to:

- Annotate code with extra information
- Enable code generation
- Configure frameworks (like Flutter)
- Mark code for specific purposes (deprecation)
- Provide hints to tools and analyzers
- Drive testing frameworks

---

# Built-in Annotations

## @Deprecated and @deprecated

```dart
// Mark as deprecated with message
@Deprecated('Use newMethod() instead')
void oldMethod() {
  // Old implementation
}

// Shorthand deprecation
@deprecated
void legacyMethod() {
  // Legacy code
}

// With replacement
@Deprecated(
  'Will be removed in v2.0. Use calculateNew() instead',
)
void calculate() {
  // Old calculation
}

// Usage (will show warning)
void main() {
  oldMethod(); // Warning: Deprecated
  calculate(); // Warning: Deprecated
}
```

---

## @override

```dart
// Override annotation
class Animal {
  void speak() {
    print('Animal sound');
  }
}

class Dog extends Animal {
  @override
  void speak() {
    print('Woof!');
  }
  
  // Override with different signature (will cause warning)
  // @override
  // void speak(String sound) { // Warning: Not overriding anything
  //   print(sound);
  // }
}
```

---

## @required (meta package)

```dart
// Using @required from meta package
import 'package:meta/meta.dart';

class User {
  final String name;
  final int age;
  
  User({
    @required this.name, // Must be provided
    this.age = 0,
  });
}

// Usage
void main() {
  // var user = User(); // Error: name is required
  var user = User(name: 'Alice'); // OK
}
```

---

## @immutable

```dart
// Mark class as immutable
@immutable
class Person {
  final String name;
  final int age;
  
  const Person(this.name, this.age);
}

// Usage
void processPerson(Person person) {
  // person.name = 'Bob'; // Error: Can't modify immutable object
}
```

---

# Custom Annotations

## Creating Annotations

```dart
// Define custom annotation class
class Todo {
  final String task;
  final String? assignedTo;
  final DateTime? dueDate;
  
  const Todo(this.task, {this.assignedTo, this.dueDate});
}

// Usage of custom annotation
@Todo('Refactor this method', assignedTo: 'Alice')
void complexMethod() {
  // Complex implementation
}

@Todo('Fix bug in this class')
class BuggyClass {
  // Implementation
}
```

---

## Annotations with Named Parameters

```dart
// Annotation with multiple named parameters
class Route {
  final String path;
  final String? method;
  final List<String>? middlewares;
  
  const Route(this.path, {this.method, this.middlewares});
}

// Usage
@Route('/users', method: 'GET')
void getUsers() {
  // Get all users
}

@Route('/users', method: 'POST', middlewares: ['auth', 'validation'])
void createUser() {
  // Create user
}
```

---

# Annotations in Frameworks

## Flutter Annotations

```dart
// Flutter annotations
class MyWidget extends StatelessWidget {
  const MyWidget({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}

// @mustCallSuper
class ParentWidget extends StatefulWidget {
  @override
  _ParentWidgetState createState() => _ParentWidgetState();
}

class _ParentWidgetState extends State<ParentWidget> {
  @mustCallSuper
  void dispose() {
    // Subclasses must call super.dispose()
    super.dispose();
  }
}

// @protected
class BaseWidget extends StatefulWidget {
  @protected
  void protectedMethod() {
    // Only accessible in subclasses
  }
}
```

---

## JSON Serialization Annotations

```dart
// Using json_serializable package
import 'package:json_annotation/json_annotation.dart';

part 'user.g.dart';

@JsonSerializable()
class User {
  final String name;
  final int age;
  
  @JsonKey(name: 'email_address')
  final String? email;
  
  User(this.name, this.age, {this.email});
  
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}

// Custom JSON annotation
@JsonKey(ignore: true)
String get fullName => '$name ${email ?? ''}';
```

---

# Code Generation with Annotations

## Generator Annotation

```dart
// Custom code generator annotation
class Generate {
  final String? fileName;
  final bool overwrite;
  
  const Generate({this.fileName, this.overwrite = false});
}

@Generate(fileName: 'user.g.dart')
class User {
  final String name;
  final int age;
  
  User(this.name, this.age);
}
```

---

# Reading Annotations at Runtime

## Reflection with dart:mirrors

```dart
import 'dart:mirrors';

// Define annotation
class Todo {
  final String description;
  final DateTime date;
  
  const Todo(this.description, {DateTime? date}) : date = date ?? DateTime.now();
}

// Use annotation
@Todo('Implement this feature')
class Feature {
  void doSomething() {}
}

// Read annotations (not recommended in Flutter)
void readAnnotations() {
  var mirror = reflectClass(Feature);
  var annotations = mirror.metadata;
  
  for (var metadata in annotations) {
    if (metadata.reflectee is Todo) {
      var todo = metadata.reflectee as Todo;
      print('Todo: ${todo.description}');
    }
  }
}
```

---

# Best Practices

## Use Annotations for Documentation

```dart
// Good: Annotations for tracking
@Todo('Add error handling', assignedTo: 'Alice')
void unstableMethod() {
  // Implementation
}

// Good: Annotations for configuration
@Route('/api/users')
class UsersController {
  @Route('/:id', method: 'GET')
  void getUser(String id) {}
}
```

---

## Keep Annotations Simple

```dart
// Good: Simple annotation
class Author {
  final String name;
  final String? email;
  
  const Author(this.name, {this.email});
}

@Author('Alice', email: 'alice@example.com')
class MyClass {
  // Implementation
}

// Bad: Overly complex
class ComplexAnnotation {
  final Map<String, dynamic> config;
  final List<String> tags;
  final Map<String, List<int>> data;
  
  const ComplexAnnotation(this.config, this.tags, this.data);
}
```

---

# Common Mistakes

## Missing import

Wrong:

```dart
@required // Error: Undefined name 'required'
void process({String? name}) {}
```

Correct:

```dart
import 'package:meta/meta.dart';

@required
void process({required String? name}) {}
```

---

## Using Annotations at Runtime

Wrong:

```dart
// Trying to use annotations at runtime without mirrors
@deprecated
void oldMethod() {
  // Can't detect @deprecated at runtime
}
```

Correct:

```dart
// Use annotations for compile-time only
// Or use dart:mirrors (not in Flutter)
// Or use build_runner for code generation
```

---

# Summary

Metadata annotations provide a way to add extra information to code that can be used by tools, frameworks, and developers. They're essential for code generation, API documentation, and framework configuration.

---

# Next Steps

Now that you understand metadata, continue to:

- [Comments & Documentation](comments-documentation.md)
- [Classes](classes.md)
- [Constructors](constructors.md)
- [Fields](fields.md)

---

# Did You Know?

- Annotations are compile-time only (in Flutter)
- Dart doesn't support runtime reflection in Flutter
- The `meta` package provides common annotations
- Annotations can be used for code generation
- `@Deprecated` vs `@deprecated` (one has message support)
- Annotations can have constructors with parameters
- Frameworks like Flutter extensively use annotations