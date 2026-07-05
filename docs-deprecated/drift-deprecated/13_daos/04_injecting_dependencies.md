## Injecting Dependencies

**Managing DAO dependencies with dependency injection in Drift**

---

# What is it?

**Injecting Dependencies** is the practice of providing DAOs and other dependencies to your application components rather than having them create their own instances. This promotes loose coupling, testability, and maintainability. In Drift, you typically inject DAOs into services, repositories, or UI components using a dependency injection (DI) framework or manual constructor injection.

> **Think of Dependency Injection like "ordering supplies for a restaurant"** – instead of each chef growing their own vegetables (creating dependencies), a central supplier (DI container) delivers everything they need to the kitchen.

```dart
// 👇 Without Dependency Injection (tight coupling)
class UserService {
  // ❌ Creates its own dependencies
  final _db = AppDatabase();
  final _userDao = UserDao(_db);
  
  Future<User> getUser(int id) => _userDao.getUserById(id);
}

// 👇 With Dependency Injection (loose coupling)
class UserService {
  // ✅ Dependencies are injected
  final UserDao _userDao;
  
  UserService(this._userDao); // 👈 Constructor injection
  
  Future<User> getUser(int id) => _userDao.getUserById(id);
}

// 👇 Creating the service with injected dependencies
final db = AppDatabase();
final userDao = UserDao(db);
final userService = UserService(userDao);
```

> **What's happening here?**
> - **Loose coupling** – Services don't create their own dependencies
> - **Testability** – Easy to mock dependencies for testing
> - **Flexibility** – Easy to swap implementations
> - **Centralized creation** – Dependencies created in one place

---

# Why does it exist?

- **Loose Coupling** – Components don't create their own dependencies
- **Testability** – Easy to mock dependencies for unit tests
- **Maintainability** – Easy to change implementations
- **Reusability** – Components can be used in different contexts
- **Clarity** – Explicit dependencies, easy to see what a component needs
- **Centralized Configuration** – Dependencies configured in one place

---

# Dependency Injection Patterns

> **Different DI approaches for DAOs**

## 1. Constructor Injection (Manual)

```dart
// 👇 Manual constructor injection
class UserService {
  final UserDao _userDao;
  final OrderDao _orderDao;
  final ProductDao _productDao;
  
  // 👇 Dependencies injected through constructor
  UserService({
    required UserDao userDao,
    required OrderDao orderDao,
    required ProductDao productDao,
  })  : _userDao = userDao,
        _orderDao = orderDao,
        _productDao = productDao;
  
  Future<UserWithOrders> getUserWithOrders(int userId) async {
    final user = await _userDao.getUserById(userId);
    final orders = await _orderDao.getOrdersByUser(userId);
    return UserWithOrders(user: user, orders: orders);
  }
}

// Usage
final db = AppDatabase();
final userService = UserService(
  userDao: UserDao(db),
  orderDao: OrderDao(db),
  productDao: ProductDao(db),
);
```

---

## 2. Service Locator Pattern

```dart
// 👇 Simple service locator
class ServiceLocator {
  static final ServiceLocator _instance = ServiceLocator._internal();
  factory ServiceLocator() => _instance;
  ServiceLocator._internal();
  
  final Map<Type, Object> _services = {};
  
  void register<T>(T service) {
    _services[T] = service;
  }
  
  T get<T>() {
    return _services[T] as T;
  }
}

// Setup
void setupServices() {
  final db = AppDatabase();
  ServiceLocator().register<UserDao>(UserDao(db));
  ServiceLocator().register<OrderDao>(OrderDao(db));
  ServiceLocator().register<ProductDao>(ProductDao(db));
}

// Usage
class UserService {
  final UserDao _userDao = ServiceLocator().get<UserDao>();
  
  Future<User> getUser(int id) => _userDao.getUserById(id);
}
```

---

## 3. GetIt (Popular DI Framework)

```dart
// 👇 Using GetIt for DI
import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

// Setup
void setupDependencies() {
  // Register database
  getIt.registerLazySingleton(() => AppDatabase());
  
  // Register DAOs
  getIt.registerLazySingleton(() => UserDao(getIt()));
  getIt.registerLazySingleton(() => OrderDao(getIt()));
  getIt.registerLazySingleton(() => ProductDao(getIt()));
  
  // Register services
  getIt.registerLazySingleton(() => UserService(getIt()));
  getIt.registerLazySingleton(() => OrderService(getIt()));
}

// Usage
class UserService {
  final UserDao _userDao;
  
  UserService(this._userDao);
  
  Future<User> getUser(int id) => _userDao.getUserById(id);
}

// Getting dependencies
final userService = getIt<UserService>();
final user = await userService.getUser(1);
```

---

## 4. Provider (Flutter)

```dart
// 👇 Using Provider in Flutter
import 'package:provider/provider.dart';

// Setup
void main() {
  runApp(
    MultiProvider(
      providers: [
        Provider(create: (_) => AppDatabase()),
        ProxyProvider<AppDatabase, UserDao>(
          update: (_, db, __) => UserDao(db),
        ),
        ProxyProvider<UserDao, UserService>(
          update: (_, userDao, __) => UserService(userDao),
        ),
      ],
      child: MyApp(),
    ),
  );
}

// Usage in widget
class UserProfileWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final userService = Provider.of<UserService>(context);
    
    return FutureBuilder(
      future: userService.getUser(1),
      builder: (context, snapshot) {
        // ... build UI
      },
    );
  }
}
```

---

# Real-World Example

> **Complete e-commerce dependency injection system**

```dart
// lib/database/database_module.dart
import 'package:drift/drift.dart';
import 'package:get_it/get_it.dart';
import '../services/user_service.dart';
import '../services/order_service.dart';
import '../services/product_service.dart';
import '../services/analytics_service.dart';

class DatabaseModule {
  static final getIt = GetIt.instance;
  
  static void setup() {
    // ==================== DATABASE ====================
    getIt.registerLazySingleton<AppDatabase>(
      () => AppDatabase(),
    );
    
    // ==================== DAOS ====================
    getIt.registerLazySingleton<UserDao>(
      () => UserDao(getIt<AppDatabase>()),
    );
    
    getIt.registerLazySingleton<OrderDao>(
      () => OrderDao(getIt<AppDatabase>()),
    );
    
    getIt.registerLazySingleton<ProductDao>(
      () => ProductDao(getIt<AppDatabase>()),
    );
    
    getIt.registerLazySingleton<AnalyticsDao>(
      () => AnalyticsDao(getIt<AppDatabase>()),
    );
    
    // ==================== SERVICES ====================
    getIt.registerLazySingleton<UserService>(
      () => UserService(
        userDao: getIt<UserDao>(),
        orderDao: getIt<OrderDao>(),
      ),
    );
    
    getIt.registerLazySingleton<OrderService>(
      () => OrderService(
        orderDao: getIt<OrderDao>(),
        productDao: getIt<ProductDao>(),
        userDao: getIt<UserDao>(),
      ),
    );
    
    getIt.registerLazySingleton<ProductService>(
      () => ProductService(
        productDao: getIt<ProductDao>(),
      ),
    );
    
    getIt.registerLazySingleton<AnalyticsService>(
      () => AnalyticsService(
        analyticsDao: getIt<AnalyticsDao>(),
        userDao: getIt<UserDao>(),
        orderDao: getIt<OrderDao>(),
      ),
    );
  }
}

// lib/services/user_service.dart
class UserService {
  final UserDao _userDao;
  final OrderDao _orderDao;
  
  UserService({
    required UserDao userDao,
    required OrderDao orderDao,
  })  : _userDao = userDao,
        _orderDao = orderDao;
  
  Future<User> createUser({
    required String username,
    required String email,
    required String password,
  }) async {
    return await _userDao.createUser(
      username: username,
      email: email,
      password: password,
    );
  }
  
  Future<User?> getUserById(int id) async {
    return await _userDao.getUserById(id);
  }
  
  Future<UserWithOrders> getUserWithOrders(int userId) async {
    final user = await _userDao.getUserById(userId);
    if (user == null) {
      throw Exception('User not found');
    }
    
    final orders = await _orderDao.getOrdersByUser(userId);
    return UserWithOrders(
      user: user,
      orders: orders,
    );
  }
  
  Future<List<User>> getActiveUsers() async {
    return await _userDao.getActiveUsers();
  }
  
  Future<void> activateUser(int userId) async {
    await _userDao.activateUser(userId);
  }
  
  Future<void> deactivateUser(int userId) async {
    await _userDao.deactivateUser(userId);
  }
}

// lib/services/order_service.dart
class OrderService {
  final OrderDao _orderDao;
  final ProductDao _productDao;
  final UserDao _userDao;
  
  OrderService({
    required OrderDao orderDao,
    required ProductDao productDao,
    required UserDao userDao,
  })  : _orderDao = orderDao,
        _productDao = productDao,
        _userDao = userDao;
  
  Future<Order> createOrder({
    required int userId,
    required List<CartItem> items,
  }) async {
    // Validate user exists
    final user = await _userDao.getUserById(userId);
    if (user == null) {
      throw Exception('User not found');
    }
    
    // Validate products and stock
    for (final item in items) {
      final product = await _productDao.getProductById(item.productId);
      if (product == null) {
        throw Exception('Product not found: ${item.productId}');
      }
      if (product.stock < item.quantity) {
        throw Exception('Insufficient stock for ${product.name}');
      }
    }
    
    return await _orderDao.createOrder(
      userId: userId,
      items: items,
    );
  }
  
  Future<Order?> getOrderById(int id) async {
    return await _orderDao.getOrderById(id);
  }
  
  Future<List<Order>> getUserOrders(int userId) async {
    return await _orderDao.getOrdersByUser(userId);
  }
  
  Future<void> cancelOrder(int orderId) async {
    await _orderDao.cancelOrder(orderId);
  }
}

// lib/main.dart
void main() {
  // Setup dependencies
  DatabaseModule.setup();
  
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'E-Commerce App',
      home: HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  HomePage({super.key});

  final userService = DatabaseModule.getIt<UserService>();
  final orderService = DatabaseModule.getIt<OrderService>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: FutureBuilder(
        future: userService.getActiveUsers(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return Center(child: CircularProgressIndicator());
          }
          
          final users = snapshot.data!;
          return ListView.builder(
            itemCount: users.length,
            itemBuilder: (context, index) {
              final user = users[index];
              return ListTile(
                title: Text(user.username),
                subtitle: Text(user.email),
                trailing: Icon(
                  user.isActive ? Icons.check_circle : Icons.cancel,
                  color: user.isActive ? Colors.green : Colors.red,
                ),
              );
            },
          );
        },
      ),
    );
  }
}
```

---

# Dependency Injection Best Practices

- **Use constructor injection** – Preferred for clarity
- **Register DAOs as singletons** – One instance per app
- **Use DI frameworks** – GetIt, Provider, Riverpod
- **Register services** – In a central location
- **Keep dependencies explicit** – Clear what each class needs
- **Test with mocks** – Easy to substitute dependencies
- **Use interfaces** – For easier mocking (optional)
- **Avoid service locator** – Prefer constructor injection

---

# Dependency Injection Checklist

| Practice | Description | Priority |
|----------|-------------|----------|
| **Constructor Injection** | Preferred approach | High |
| **Singleton DAOs** | One instance | High |
| **DI Framework** | Use GetIt, Provider | High |
| **Central Registration** | Single setup location | High |
| **Explicit Dependencies** | Clear constructor | High |
| **Testing** | Use mocks | High |
| **Lazy Loading** | Register lazily | Medium |

---

# Common Mistakes

## Mistake 1: Creating dependencies inside classes

Wrong:
```dart
// 🚫 Hard-coded dependency
class UserService {
  final UserDao _userDao = UserDao(AppDatabase());
}
```

Correct:
```dart
// ✅ Injected dependency
class UserService {
  final UserDao _userDao;
  UserService(this._userDao);
}
```

## Mistake 2: Using service locator everywhere

Wrong:
```dart
// 🚫 Service locator in every class
class UserService {
  final UserDao _userDao = getIt<UserDao>();
}
```

Correct:
```dart
// ✅ Constructor injection
class UserService {
  final UserDao _userDao;
  UserService(this._userDao);
}
```

## Mistake 3: Not registering dependencies

Wrong:
```dart
// 🚫 Trying to get unregistered dependency
final userService = getIt<UserService>(); // Error!
```

Correct:
```dart
// ✅ Register before using
getIt.registerLazySingleton(() => UserService(getIt()));
final userService = getIt<UserService>();
```

---

# Summary

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **Constructor Injection** | Pass dependencies via constructor | Most common |
| **Service Locator** | Global registry | Simple apps |
| **GetIt** | Popular DI framework | Large apps |
| **Provider** | Flutter DI | Flutter apps |

---

# Next Steps

Now you understand injecting dependencies, let's dive deeper:

- [Best Practices](link) – DAO best practices
- [Performance](link) – Performance optimization
- [Testing](link) – Testing DAOs and services

---

# Did You Know?

- **DI promotes loose coupling** – Easier to maintain

- **DI improves testability** – Easy to mock dependencies

- **DI is a design pattern** – Not a framework

- **DI frameworks are common** – GetIt, Provider, Riverpod

- **Constructor injection is preferred** – Clear and explicit

- **DI centralizes configuration** – One place to set up

- **DI makes dependencies explicit** – Easy to see what's needed

- **DI is essential** – For large applications

---

