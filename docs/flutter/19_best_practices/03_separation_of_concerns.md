# Separation of Concerns

Understand how to separate different responsibilities in your Flutter applications for better maintainability and testability.

---

# What is it?

Separation of Concerns is a design principle that divides a computer program into distinct sections, each addressing a separate concern. In Flutter, this means separating the UI (presentation layer) from the business logic (domain layer) and data layer. This makes your code more modular, easier to test, and simpler to maintain.

---

# Why does it exist?

Separation of Concerns exists to:

- Improve code maintainability
- Make code easier to test
- Enable code reuse
- Reduce complexity
- Support team collaboration
- Allow independent development
- Simplify debugging

---

# Layered Architecture

> **Understanding** the different layers.

```
┌─────────────────────────────────────────────┐
│           Presentation Layer                 │
│  ┌─────────────────────────────────────────┐ │
│  │   Widgets    │   Screens    │  Pages    │ │
│  └─────────────────────────────────────────┘ │
│              ↓                               │
├─────────────────────────────────────────────┤
│           Domain Layer                       │
│  ┌─────────────────────────────────────────┐ │
│  │  Entities  │  Use Cases  │  Interfaces  │ │
│  └─────────────────────────────────────────┘ │
│              ↓                               │
├─────────────────────────────────────────────┤
│           Data Layer                         │
│  ┌─────────────────────────────────────────┐ │
│  │ Repositories │ Data Sources │   Models  │ │
│  └─────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

What's happening here?
- Presentation Layer: UI components
- Domain Layer: Business logic
- Data Layer: Data management
- Clean separation of responsibilities

---

# Presentation Layer

> **UI components** and state management.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

/// 1. Presentation Layer - Widget
/// This widget only handles UI and delegates logic to the provider
class LoginScreen extends StatelessWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Login'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Consumer<AuthProvider>(
          builder: (context, auth, _) {
            if (auth.isLoading) {
              return const Center(child: CircularProgressIndicator());
            }
            return Column(
              children: [
                // 1. UI components only
                const Text(
                  'Welcome Back!',
                  style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 32),
                // 2. Form widgets
                _buildEmailField(),
                const SizedBox(height: 16),
                _buildPasswordField(),
                const SizedBox(height: 24),
                // 3. Button with callback
                _buildLoginButton(auth),
              ],
            );
          },
        ),
      ),
    );
  }

  Widget _buildEmailField() {
    return TextField(
      key: const Key('email_field'),
      decoration: const InputDecoration(
        labelText: 'Email',
        border: OutlineInputBorder(),
      ),
      keyboardType: TextInputType.emailAddress,
      onChanged: (value) {
        // Only UI handling, logic delegated
      },
    );
  }

  Widget _buildPasswordField() {
    return TextField(
      key: const Key('password_field'),
      decoration: const InputDecoration(
        labelText: 'Password',
        border: OutlineInputBorder(),
      ),
      obscureText: true,
      onChanged: (value) {
        // Only UI handling, logic delegated
      },
    );
  }

  Widget _buildLoginButton(AuthProvider auth) {
    return SizedBox(
      width: double.infinity,
      child: ElevatedButton(
        key: const Key('login_button'),
        onPressed: () {
          // Delegate logic to provider
          auth.login();
        },
        child: const Text('Login'),
      ),
    );
  }
}
```

What's happening here?
- Widgets only handle UI
- Logic is delegated to providers
- No business logic in widgets
- Clean separation of concerns

---

# Domain Layer

> **Business logic** and entities.

```dart
/// 2. Domain Layer - Entities
/// These are pure business objects with no dependencies
class User {
  final String id;
  final String name;
  final String email;

  const User({
    required this.id,
    required this.name,
    required this.email,
  });

  // Factory methods for creating instances
  factory User.create(String name, String email) {
    return User(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      name: name,
      email: email,
    );
  }

  // Validation methods
  bool isValid() {
    return name.isNotEmpty && email.contains('@');
  }

  // Copy method for immutability
  User copyWith({
    String? id,
    String? name,
    String? email,
  }) {
    return User(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
    );
  }

  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is User && other.id == id;
  }

  @override
  int get hashCode => id.hashCode;
}

/// 3. Domain Layer - Use Cases
/// These contain the business logic
class LoginUseCase {
  final AuthRepository repository;

  LoginUseCase(this.repository);

  Future<User> execute(String email, String password) async {
    // 1. Validate input
    if (email.isEmpty || password.isEmpty) {
      throw Exception('Email and password are required');
    }

    // 2. Delegate to repository
    final user = await repository.login(email, password);

    // 3. Validate result
    if (!user.isValid()) {
      throw Exception('Invalid user data received');
    }

    return user;
  }
}

/// 4. Domain Layer - Repository Interface
/// This defines the contract for data operations
abstract class AuthRepository {
  Future<User> login(String email, String password);
  Future<void> logout();
  Future<User> getCurrentUser();
  bool isAuthenticated();
}
```

What's happening here?
- Entities are pure business objects
- Use cases contain business logic
- Repository interfaces define contracts
- No external dependencies

---

# Data Layer

> **Data management** and repositories.

```dart
/// 5. Data Layer - Models
/// These are data transfer objects
class UserModel {
  final String id;
  final String name;
  final String email;

  UserModel({
    required this.id,
    required this.name,
    required this.email,
  });

  // Convert from JSON
  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }

  // Convert to JSON
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
    };
  }

  // Convert to Domain Entity
  User toDomain() {
    return User(
      id: id,
      name: name,
      email: email,
    );
  }

  // Create from Domain Entity
  static UserModel fromDomain(User user) {
    return UserModel(
      id: user.id,
      name: user.name,
      email: user.email,
    );
  }
}

/// 6. Data Layer - Repository Implementation
class AuthRepositoryImpl implements AuthRepository {
  final ApiService apiService;
  final StorageService storageService;

  AuthRepositoryImpl({
    required this.apiService,
    required this.storageService,
  });

  @override
  Future<User> login(String email, String password) async {
    try {
      // 1. Call the API
      final response = await apiService.post('/auth/login', {
        'email': email,
        'password': password,
      });

      // 2. Parse response
      final userModel = UserModel.fromJson(response['user']);
      final user = userModel.toDomain();

      // 3. Store token
      await storageService.saveString('token', response['token']);
      await storageService.saveString('user', response['user'].toString());

      return user;
    } catch (e) {
      throw Exception('Login failed: $e');
    }
  }

  @override
  Future<void> logout() async {
    await apiService.post('/auth/logout', {});
    await storageService.remove('token');
    await storageService.remove('user');
  }

  @override
  Future<User> getCurrentUser() async {
    final userJson = storageService.getString('user');
    if (userJson == null) {
      throw Exception('No user logged in');
    }
    // Parse and return user
    return const User(
      id: '1',
      name: 'John Doe',
      email: 'john@example.com',
    );
  }

  @override
  bool isAuthenticated() {
    return storageService.getString('token') != null;
  }
}

/// 7. Data Layer - Data Source
class ApiService {
  final String baseUrl;
  final http.Client client;

  ApiService({required this.baseUrl, required this.client});

  Future<Map<String, dynamic>> post(String endpoint, Map<String, dynamic> data) async {
    // Network call implementation
    return {};
  }
}

class StorageService {
  final SharedPreferences _prefs;

  StorageService(this._prefs);

  Future<void> saveString(String key, String value) async {
    await _prefs.setString(key, value);
  }

  String? getString(String key) {
    return _prefs.getString(key);
  }

  Future<void> remove(String key) async {
    await _prefs.remove(key);
  }
}
```

What's happening here?
- Models handle data transformation
- Repositories implement data operations
- Data sources handle network/storage
- Clear separation of responsibilities

---

# Provider/Service Layer

> **Connecting** UI and business logic.

```dart
/// 8. Provider Layer - State Management
class AuthProvider extends ChangeNotifier {
  final LoginUseCase loginUseCase;
  final AuthRepository repository;

  User? _user;
  bool _isLoading = false;
  bool _isAuthenticated = false;

  AuthProvider({
    required this.loginUseCase,
    required this.repository,
  });

  User? get user => _user;
  bool get isLoading => _isLoading;
  bool get isAuthenticated => _isAuthenticated;

  Future<void> login(String email, String password) async {
    // 1. Set loading state
    _setLoading(true);

    try {
      // 2. Execute use case
      _user = await loginUseCase.execute(email, password);
      _isAuthenticated = true;
    } catch (e) {
      // 3. Handle error
      throw Exception('Login failed: $e');
    } finally {
      // 4. Clear loading state
      _setLoading(false);
    }
  }

  Future<void> logout() async {
    try {
      await repository.logout();
      _user = null;
      _isAuthenticated = false;
      notifyListeners();
    } catch (e) {
      throw Exception('Logout failed: $e');
    }
  }

  void _setLoading(bool loading) {
    _isLoading = loading;
    notifyListeners();
  }
}

/// 9. Dependency Injection Setup
class AppContainer {
  // Services
  static ApiService? _apiService;
  static StorageService? _storageService;

  // Repositories
  static AuthRepository? _authRepository;

  // Use Cases
  static LoginUseCase? _loginUseCase;

  // Providers
  static AuthProvider? _authProvider;

  static void initialize() {
    // Initialize services
    _apiService = ApiService(baseUrl: 'https://api.example.com');
    _storageService = StorageService();

    // Initialize repositories
    _authRepository = AuthRepositoryImpl(
      apiService: _apiService!,
      storageService: _storageService!,
    );

    // Initialize use cases
    _loginUseCase = LoginUseCase(_authRepository!);
  }

  static AuthProvider getAuthProvider() {
    _authProvider ??= AuthProvider(
      loginUseCase: _loginUseCase!,
      repository: _authRepository!,
    );
    return _authProvider!;
  }
}
```

What's happening here?
- Provider manages state
- Use cases are executed
- Repository is accessed
- Clear separation of UI and logic

---

# Real-World Examples

> **Common patterns** for separation of concerns.

```dart
/// 1. Clean Architecture with BLoC
class LoginBloc extends Bloc<LoginEvent, LoginState> {
  final LoginUseCase loginUseCase;

  LoginBloc(this.loginUseCase) : super(LoginInitial()) {
    on<LoginSubmitted>(_onLoginSubmitted);
  }

  Future<void> _onLoginSubmitted(
    LoginSubmitted event,
    Emitter<LoginState> emit,
  ) async {
    emit(LoginLoading());
    try {
      final user = await loginUseCase.execute(
        event.email,
        event.password,
      );
      emit(LoginSuccess(user));
    } catch (e) {
      emit(LoginFailure(e.toString()));
    }
  }
}

/// 2. Repository Pattern
class TodoRepository implements ITodoRepository {
  final TodoLocalDataSource localDataSource;
  final TodoRemoteDataSource remoteDataSource;

  TodoRepository({
    required this.localDataSource,
    required this.remoteDataSource,
  });

  @override
  Future<List<Todo>> getTodos() async {
    try {
      // Try remote first
      final todos = await remoteDataSource.getTodos();
      // Cache locally
      await localDataSource.cacheTodos(todos);
      return todos;
    } catch (e) {
      // Fallback to local
      return await localDataSource.getTodos();
    }
  }
}

/// 3. Service Layer
class NotificationService {
  final FlutterLocalNotificationsPlugin _plugin;

  NotificationService(this._plugin);

  Future<void> showNotification(String title, String body) async {
    const androidDetails = AndroidNotificationDetails(
      'channel_id',
      'channel_name',
      importance: Importance.max,
    );

    const iosDetails = DarwinNotificationDetails();

    const details = NotificationDetails(
      android: androidDetails,
      iOS: iosDetails,
    );

    await _plugin.show(0, title, body, details);
  }
}
```

What's happening here?
- BLoC pattern for state management
- Repository pattern for data
- Service layer for platform features
- Clean separation of concerns

---

# Benefits

> **Advantages** of separation of concerns.

```
┌─────────────────────────────────────────────┐
│           Benefits                          │
├─────────────────────────────────────────────┤
│ • Testability      • Maintainability       │
│ • Reusability      • Scalability           │
│ • Readability      • Team Collaboration    │
│ • Flexibility      • Independent Layers    │
└─────────────────────────────────────────────┘
```

What's happening here?
- Easier to test each layer independently
- Code is more maintainable
- Components can be reused
- Team members can work independently

---

# Best Practices

## Keep UI Separate

```dart
// Good - UI only
class MyWidget extends StatelessWidget {
  final String data;
  final VoidCallback onTap;
  
  const MyWidget({required this.data, required this.onTap});
  
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: Text(data),
    );
  }
}

// Bad - UI with logic
class MyWidget extends StatelessWidget {
  const MyWidget();
  
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        // Business logic in UI
        repository.save(data);
      },
      child: Text(data),
    );
  }
}
```

## Use Dependency Injection

```dart
// Good - DI
class MyWidget extends StatelessWidget {
  final AuthProvider auth;
  
  const MyWidget({required this.auth});
}

// Bad - Direct instantiation
class MyWidget extends StatelessWidget {
  const MyWidget();
  
  @override
  Widget build(BuildContext context) {
    final auth = AuthProvider(); // Not testable
  }
}
```

## Separate Data Models

```dart
// Good - Separate models
// Data layer model
class UserModel { ... }

// Domain layer entity
class User { ... }

// Bad - One model for everything
class User {
  // Used in UI, data, and domain
}
```

---

# Common Mistakes

## UI with Business Logic

Wrong:
```dart
// UI handles business logic
class LoginScreen extends StatelessWidget {
  void login() {
    // API calls and business logic
  }
}
```

Correct:
```dart
// UI delegates to provider
class LoginScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final auth = Provider.of<AuthProvider>(context);
    return ElevatedButton(
      onPressed: auth.login,
      child: const Text('Login'),
    );
  }
}
```

## Tight Coupling

Wrong:
```dart
// Direct dependency on implementation
class MyWidget {
  final ApiService api; // Tight coupling
}
```

Correct:
```dart
// Dependency on abstraction
class MyWidget {
  final IApiService api; // Loose coupling
}
```

---

# Summary

Separation of Concerns divides your application into distinct layers: presentation, domain, and data. This makes your code more maintainable, testable, and scalable. Keep UI separate from logic, use dependency injection, and separate data models from domain entities.

---

# Next Steps

- [Architecture Patterns](architecture-patterns.md)
- [Project Structure](project-structure.md)
- [Folder Organization](folder-organization.md)

---

# Did You Know?

- Separation of Concerns improves testability
- Clean Architecture uses three layers
- Dependency Injection enables loose coupling
- UI should not contain business logic
- Models and entities serve different purposes
- Use cases contain business logic
- Repositories handle data operations
- Providers manage state