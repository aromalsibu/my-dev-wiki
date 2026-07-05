# Object Lifecycle

Understand the complete lifecycle of objects in Dart, from creation to destruction.

---

# What is it?

The object lifecycle refers to the sequence of events that occur from the moment an object is created until it is garbage collected. Understanding this lifecycle helps you manage resources, control initialization, and handle cleanup properly.

---

# Why does it exist?

Understanding object lifecycle exists to:

- Manage resources properly
- Prevent memory leaks
- Handle initialization correctly
- Clean up resources
- Understand garbage collection
- Write efficient, reliable code

---

# Object Creation

## The Creation Process

```dart
class Person {
  String name;
  int age;
  DateTime createdAt;
  
  // 1. Constructor runs
  Person(this.name, this.age)
      : createdAt = DateTime.now() {
    // 2. Constructor body executes
    print('Person created: $name, age $age');
    print('Created at: $createdAt');
  }
  
  // 3. Object is ready for use
  void sayHello() {
    print('Hello, I am $name');
  }
}

// 4. Usage
var person = Person('Alice', 25); // Created and initialized
person.sayHello(); // Ready to use
```

---

## Initialization Order

```dart
class Parent {
  // 1. Parent fields initialized
  final String parentField = _log('Parent field');
  
  static String _log(String msg) {
    print(msg);
    return msg;
  }
  
  Parent() {
    print('Parent constructor');
  }
}

class Child extends Parent {
  // 2. Child fields initialized
  final String childField = _log('Child field');
  
  Child() : super() {
    print('Child constructor');
  }
}

// Usage
var child = Child();
// Output:
// Parent field
// Parent constructor
// Child field
// Child constructor
```

---

# Initialization Sequence

## Complete Initialization Example

```dart
class Vehicle {
  final String brand;
  final String model;
  final int year;
  
  // 1. Superclass initialization
  Vehicle(this.brand, this.model, this.year) {
    print('Vehicle: $brand $model ($year)');
  }
}

class Car extends Vehicle {
  final String color;
  int _mileage = 0;
  final DateTime purchaseDate;
  
  // 2. Initializer list (runs before body)
  Car({
    required String brand,
    required String model,
    required int year,
    required this.color,
  })  : purchaseDate = DateTime.now(),
        super(brand, model, year) {
    // 3. Constructor body
    print('Car: $color $brand $model');
    print('Purchased: $purchaseDate');
  }
  
  // 4. Methods available
  void drive(int miles) {
    _mileage += miles;
    print('Drove $miles miles. Total: $_mileage');
  }
  
  int get mileage => _mileage;
}

// Usage
var car = Car(
  brand: 'Tesla',
  model: 'Model 3',
  year: 2024,
  color: 'Red',
);
car.drive(100);

// Output:
// Vehicle: Tesla Model 3 (2024)
// Car: Red Tesla Model 3
// Purchased: 2024-01-01 00:00:00.000
// Drove 100 miles. Total: 100
```

---

# Object Usage

## Active Lifecycle

```dart
class DataProcessor {
  final String name;
  bool _isInitialized = false;
  List<String> _data = [];
  
  DataProcessor(this.name) {
    print('DataProcessor $name created');
  }
  
  // Initialization method
  void initialize() {
    if (!_isInitialized) {
      _isInitialized = true;
      _loadData();
      print('DataProcessor $name initialized');
    }
  }
  
  void _loadData() {
    print('Loading data...');
    _data = ['item1', 'item2', 'item3'];
  }
  
  // Active usage
  void process() {
    if (!_isInitialized) {
      throw StateError('Processor not initialized');
    }
    print('Processing data: $_data');
  }
  
  void addItem(String item) {
    if (!_isInitialized) {
      throw StateError('Processor not initialized');
    }
    _data.add(item);
    print('Added: $item');
  }
  
  // Cleanup
  void dispose() {
    _data.clear();
    _isInitialized = false;
    print('DataProcessor $name disposed');
  }
  
  bool get isInitialized => _isInitialized;
}

// Usage
var processor = DataProcessor('Main'); // Created
processor.initialize(); // Initialized
processor.process(); // Active usage
processor.addItem('item4'); // Active usage
processor.dispose(); // Cleanup
```

---

# Object Destruction

## Garbage Collection

```dart
class Resource {
  final String id;
  
  Resource(this.id) {
    print('Resource $id created');
  }
  
  void use() {
    print('Resource $id in use');
  }
  
  // Optional cleanup method
  void dispose() {
    print('Resource $id disposed');
  }
  
  @override
  void finalize() {
    print('Resource $id garbage collected');
  }
}

// Usage
void createResource() {
  var resource = Resource('temp');
  resource.use();
  // resource goes out of scope here
  // Ready for garbage collection
}

void createAndDispose() {
  var resource = Resource('disposable');
  resource.use();
  resource.dispose(); // Explicit cleanup
}

// Usage
createResource();
createAndDispose();

// Force garbage collection (for testing)
import 'dart:developer';
var resource = Resource('test');
resource.use();
resource = null; // Remove reference
// GC will collect when convenient
```

---

# Resource Management

## Dispose Pattern

```dart
class DatabaseConnection implements Disposable {
  bool _isConnected = false;
  
  void connect() {
    if (!_isConnected) {
      print('Connecting to database...');
      _isConnected = true;
    }
  }
  
  void query(String sql) {
    if (!_isConnected) {
      throw StateError('Not connected');
    }
    print('Executing: $sql');
  }
  
  @override
  void dispose() {
    if (_isConnected) {
      print('Disconnecting from database...');
      _isConnected = false;
    }
  }
}

class Disposable {
  void dispose();
}

// Usage with try-finally
void useDatabase() {
  var db = DatabaseConnection();
  try {
    db.connect();
    db.query('SELECT * FROM users');
  } finally {
    db.dispose(); // Always called
  }
}

// Usage with late final
class DatabaseManager {
  late final DatabaseConnection _db;
  
  DatabaseManager() {
    _db = DatabaseConnection();
    _db.connect();
  }
  
  void executeQuery(String sql) {
    _db.query(sql);
  }
  
  void dispose() {
    _db.dispose();
  }
}
```

---

# Object Lifecycle Patterns

## Builder Pattern with Lifecycle

```dart
class QueryBuilder {
  List<String> _select = [];
  String? _from;
  List<String> _where = [];
  List<String> _orderBy = [];
  int? _limit;
  
  QueryBuilder select(List<String> fields) {
    _select = fields;
    return this;
  }
  
  QueryBuilder from(String table) {
    _from = table;
    return this;
  }
  
  QueryBuilder where(String condition) {
    _where.add(condition);
    return this;
  }
  
  QueryBuilder orderBy(String field) {
    _orderBy.add(field);
    return this;
  }
  
  QueryBuilder limit(int value) {
    _limit = value;
    return this;
  }
  
  // Build method completes the lifecycle
  String build() {
    if (_from == null) {
      throw StateError('Table not specified');
    }
    
    var query = 'SELECT ${_select.isEmpty ? '*' : _select.join(', ')}';
    query += ' FROM $_from';
    
    if (_where.isNotEmpty) {
      query += ' WHERE ${_where.join(' AND ')}';
    }
    
    if (_orderBy.isNotEmpty) {
      query += ' ORDER BY ${_orderBy.join(', ')}';
    }
    
    if (_limit != null) {
      query += ' LIMIT $_limit';
    }
    
    return query;
  }
}

// Usage
var query = QueryBuilder()
  .select(['id', 'name'])
  .from('users')
  .where('age > 18')
  .where('active = true')
  .orderBy('name')
  .limit(10)
  .build();

print(query);
// SELECT id, name FROM users WHERE age > 18 AND active = true ORDER BY name LIMIT 10
```

---

# Best Practices

## Always Dispose Resources

```dart
// Good: Dispose in finally
void processFile() {
  File file = File('data.txt');
  try {
    // Use file
  } finally {
    file.dispose(); // Always called
  }
}

// Good: Use try-with-resources pattern
void processResource() {
  var resource = Resource();
  try {
    resource.use();
  } finally {
    resource.dispose();
  }
}
```

---

## Avoid Memory Leaks

```dart
// Bad: Event listener not removed
class MemoryLeak {
  void setup() {
    button.onClick.listen((_) {
      // This listener holds reference
    });
  }
}

// Good: Remove listeners
class NoMemoryLeak {
  StreamSubscription? _subscription;
  
  void setup() {
    _subscription = button.onClick.listen((_) {
      // Handle click
    });
  }
  
  void dispose() {
    _subscription?.cancel();
    _subscription = null;
  }
}
```

---

# Common Mistakes

## Using Object After Dispose

Wrong:

```dart
class Service {
  void dispose() {
    // Cleanup
  }
  
  void doWork() {
    // Called after dispose
  }
}

var service = Service();
service.dispose();
service.doWork(); // Error: Service already disposed
```

Correct:

```dart
class Service {
  bool _isDisposed = false;
  
  void dispose() {
    _isDisposed = true;
  }
  
  void doWork() {
    if (_isDisposed) {
      throw StateError('Service already disposed');
    }
    // Do work
  }
}
```

---

## Forgetting Finalizer

Wrong:

```dart
class ResourceManager {
  StreamSubscription? _subscription;
  
  void setup() {
    _subscription = stream.listen((_) {});
    // No cleanup
  }
}
```

Correct:

```dart
class ResourceManager {
  StreamSubscription? _subscription;
  
  void setup() {
    _subscription = stream.listen((_) {});
  }
  
  void dispose() {
    _subscription?.cancel();
    _subscription = null;
  }
}
```

---

# Summary

Understanding the object lifecycle is crucial for proper resource management, avoiding memory leaks, and writing robust applications. Remember to initialize properly, manage resources, and clean up when done.

---

# Next Steps

Now that you understand object lifecycle, continue to:

- [Generic Classes](generic-classes.md)
- [Generic Functions](generic-functions.md)
- [Type Bounds](type-bounds.md)
- [Generic Methods](generic-methods.md)

---

# Did You Know?

- Dart uses garbage collection for memory management
- Objects are created with `new` or factory constructors
- The `dispose` pattern is used for cleanup
- Objects can have a finalizer (rarely needed)
- Streams and subscriptions should be cancelled
- Classes can have initialization order
- Object lifecycle is important in resource-constrained environments