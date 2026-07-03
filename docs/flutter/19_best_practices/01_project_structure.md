# Project Structure

Understand how to organize your Flutter project for scalability, maintainability, and team collaboration.

---

# What is it?

Project Structure refers to the organization of folders, files, and code in a Flutter application. A well-organized project structure makes your code easier to navigate, maintain, and scale. It follows established patterns and best practices for separating concerns, managing dependencies, and organizing features.

---

# Why does it exist?

Project Structure exists to:

- Organize code in a logical way
- Separate concerns and responsibilities
- Make code easier to navigate
- Support team collaboration
- Enable scalability and maintainability
- Follow best practices and patterns
- Simplify testing and debugging

---

# Basic Project Structure

> **Understanding** the default Flutter project structure.

```
my_app/
├── android/                 # Android platform-specific code
├── ios/                     # iOS platform-specific code
├── web/                     # Web platform-specific code
├── windows/                 # Windows platform-specific code
├── macos/                   # macOS platform-specific code
├── linux/                   # Linux platform-specific code
├── lib/                     # Main source code
│   ├── main.dart            # App entry point
│   ├── models/              # Data models
│   ├── screens/             # Screen widgets
│   ├── widgets/             # Reusable widgets
│   ├── services/            # Business logic and services
│   ├── utils/               # Utility functions
│   └── config/              # App configuration
├── test/                    # Unit and widget tests
│   ├── models/              # Model tests
│   ├── services/            # Service tests
│   └── widgets/             # Widget tests
├── assets/                  # Images, fonts, and other assets
│   ├── images/              # Image assets
│   ├── fonts/               # Font assets
│   └── data/                # JSON and other data files
├── pubspec.yaml             # Project dependencies and metadata
├── analysis_options.yaml    # Dart linter rules
├── .gitignore              # Git ignore rules
└── README.md               # Project documentation
```

What's happening here?
- Platform-specific code in separate folders
- lib/ contains all Dart source code
- test/ contains all tests
- assets/ contains static resources
- Configuration files at root level

---

# Feature-Based Structure

> **Organizing** by features.

```
lib/
├── main.dart
├── core/                         # Core utilities and services
│   ├── constants/                # App constants
│   │   ├── app_constants.dart
│   │   └── theme_constants.dart
│   ├── utils/                    # Utility functions
│   │   ├── validators.dart
│   │   └── formatters.dart
│   ├── services/                 # Core services
│   │   ├── api_service.dart
│   │   ├── auth_service.dart
│   │   └── storage_service.dart
│   └── widgets/                  # Core widgets
│       ├── app_scaffold.dart
│       └── loading_indicator.dart
├── features/                     # Feature modules
│   ├── auth/                     # Authentication feature
│   │   ├── models/
│   │   │   ├── user.dart
│   │   │   └── auth_state.dart
│   │   ├── screens/
│   │   │   ├── login_screen.dart
│   │   │   └── register_screen.dart
│   │   ├── widgets/
│   │   │   ├── login_form.dart
│   │   │   └── register_form.dart
│   │   ├── services/
│   │   │   └── auth_service.dart
│   │   └── providers/
│   │       └── auth_provider.dart
│   ├── home/                     # Home feature
│   │   ├── models/
│   │   ├── screens/
│   │   ├── widgets/
│   │   └── services/
│   └── profile/                  # Profile feature
│       ├── models/
│       ├── screens/
│       ├── widgets/
│       └── services/
└── shared/                       # Shared across features
    ├── widgets/                  # Shared widgets
    │   ├── buttons.dart
    │   └── cards.dart
    └── extensions/               # Dart extensions
        └── context_extensions.dart
```

What's happening here?
- core/ for app-wide utilities
- features/ for feature modules
- shared/ for shared widgets
- Each feature is self-contained
- Clear separation of concerns

---

# Clean Architecture Structure

> **Implementing** clean architecture.

```
lib/
├── main.dart
├── data/                         # Data layer
│   ├── datasources/              # Data sources
│   │   ├── local/
│   │   │   └── local_datasource.dart
│   │   └── remote/
│   │       └── remote_datasource.dart
│   ├── models/                   # Data models
│   │   ├── user_model.dart
│   │   └── todo_model.dart
│   └── repositories/             # Repository implementations
│       ├── auth_repository_impl.dart
│       └── todo_repository_impl.dart
├── domain/                       # Domain layer
│   ├── entities/                 # Business entities
│   │   ├── user.dart
│   │   └── todo.dart
│   ├── repositories/             # Repository interfaces
│   │   ├── auth_repository.dart
│   │   └── todo_repository.dart
│   └── usecases/                 # Use cases (business logic)
│       ├── auth/
│       │   ├── login_usecase.dart
│       │   └── register_usecase.dart
│       └── todo/
│           ├── get_todos_usecase.dart
│           └── add_todo_usecase.dart
└── presentation/                 # Presentation layer
    ├── blocs/                    # BLoC state management
    │   ├── auth/
    │   │   ├── auth_bloc.dart
    │   │   └── auth_event.dart
    │   └── todo/
    │       ├── todo_bloc.dart
    │       └── todo_event.dart
    ├── pages/                    # Pages/Screens
    │   ├── auth/
    │   │   ├── login_page.dart
    │   │   └── register_page.dart
    │   └── home/
    │       └── home_page.dart
    └── widgets/                  # Presentation widgets
        ├── common/
        └── auth/
```

What's happening here?
- data/ handles data sources and repositories
- domain/ contains business logic
- presentation/ handles UI
- Clear layer separation
- Dependency inversion

---

# Example Implementation

> **Implementing** the structure.

```dart
/// 1. lib/main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'features/auth/providers/auth_provider.dart';
import 'features/home/screens/home_screen.dart';
import 'features/auth/screens/login_screen.dart';
import 'core/services/storage_service.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthProvider()),
      ],
      child: MaterialApp(
        title: 'My App',
        theme: ThemeData(primarySwatch: Colors.blue),
        home: Consumer<AuthProvider>(
          builder: (context, auth, _) {
            if (auth.isAuthenticated) {
              return const HomeScreen();
            }
            return const LoginScreen();
          },
        ),
        routes: {
          '/home': (context) => const HomeScreen(),
          '/login': (context) => const LoginScreen(),
        },
      ),
    );
  }
}

/// 2. lib/core/constants/app_constants.dart
class AppConstants {
  static const String appName = 'My App';
  static const String apiBaseUrl = 'https://api.example.com';
  static const String tokenKey = 'auth_token';
}

/// 3. lib/core/services/api_service.dart
import 'package:http/http.dart' as http;
import 'dart:convert';

class ApiService {
  final String baseUrl;
  final http.Client client;

  ApiService({
    required this.baseUrl,
    required this.client,
  });

  Future<Map<String, dynamic>> get(String endpoint) async {
    final response = await client.get(
      Uri.parse('$baseUrl/$endpoint'),
    );
    return _handleResponse(response);
  }

  Future<Map<String, dynamic>> post(
    String endpoint,
    Map<String, dynamic> data,
  ) async {
    final response = await client.post(
      Uri.parse('$baseUrl/$endpoint'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode(data),
    );
    return _handleResponse(response);
  }

  Map<String, dynamic> _handleResponse(http.Response response) {
    if (response.statusCode >= 200 && response.statusCode < 300) {
      return json.decode(response.body);
    } else {
      throw Exception('API error: ${response.statusCode}');
    }
  }
}

/// 4. lib/features/auth/models/user.dart
class User {
  final String id;
  final String name;
  final String email;

  User({
    required this.id,
    required this.name,
    required this.email,
  });

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
    };
  }
}

/// 5. lib/features/auth/providers/auth_provider.dart
import 'package:flutter/material.dart';
import '../models/user.dart';

class AuthProvider extends ChangeNotifier {
  User? _user;
  bool _isAuthenticated = false;

  User? get user => _user;
  bool get isAuthenticated => _isAuthenticated;

  Future<void> login(String email, String password) async {
    try {
      // Simulate API call
      await Future.delayed(const Duration(seconds: 1));
      _user = User(
        id: '1',
        name: 'John Doe',
        email: email,
      );
      _isAuthenticated = true;
      notifyListeners();
    } catch (e) {
      throw Exception('Login failed: $e');
    }
  }

  Future<void> logout() async {
    _user = null;
    _isAuthenticated = false;
    notifyListeners();
  }
}

/// 6. lib/features/auth/screens/login_screen.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/auth_provider.dart';
import '../widgets/login_form.dart';

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
            if (auth.isAuthenticated) {
              WidgetsBinding.instance.addPostFrameCallback((_) {
                Navigator.pushReplacementNamed(context, '/home');
              });
            }
            return LoginForm(
              onLogin: (email, password) async {
                try {
                  await auth.login(email, password);
                } catch (e) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Error: $e')),
                  );
                }
              },
            );
          },
        ),
      ),
    );
  }
}

/// 7. lib/features/auth/widgets/login_form.dart
import 'package:flutter/material.dart';

class LoginForm extends StatefulWidget {
  const LoginForm({
    super.key,
    required this.onLogin,
  });

  final Future<void> Function(String email, String password) onLogin;

  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isLoading = false;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  Future<void> _submit() async {
    if (_formKey.currentState!.validate()) {
      setState(() => _isLoading = true);
      try {
        await widget.onLogin(
          _emailController.text,
          _passwordController.text,
        );
      } finally {
        if (mounted) {
          setState(() => _isLoading = false);
        }
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            controller: _emailController,
            decoration: const InputDecoration(
              labelText: 'Email',
              border: OutlineInputBorder(),
            ),
            keyboardType: TextInputType.emailAddress,
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Email is required';
              }
              if (!value.contains('@')) {
                return 'Invalid email';
              }
              return null;
            },
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _passwordController,
            decoration: const InputDecoration(
              labelText: 'Password',
              border: OutlineInputBorder(),
            ),
            obscureText: true,
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Password is required';
              }
              return null;
            },
          ),
          const SizedBox(height: 24),
          SizedBox(
            width: double.infinity,
            child: _isLoading
                ? const Center(child: CircularProgressIndicator())
                : ElevatedButton(
                    onPressed: _submit,
                    child: const Text('Login'),
                  ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- main.dart sets up the app
- core/ contains app-wide utilities
- features/ contains feature modules
- Each feature has its own structure
- Providers manage state

---

# Best Practices

## Keep Features Self-Contained

```dart
// Good - Self-contained feature
features/auth/
  models/
  screens/
  widgets/
  services/
  providers/

// Bad - Scattered code
lib/models/auth.dart
lib/screens/login.dart
lib/widgets/login_form.dart
lib/services/auth_service.dart
```

## Use Consistent Naming

```dart
// Good - Consistent naming
auth_screen.dart
auth_provider.dart
auth_service.dart

// Bad - Inconsistent naming
auth_screen.dart
authProvider.dart
AuthService.dart
```

## Separate Concerns

```dart
// Good - Clear separation
lib/data/     # Data layer
lib/domain/   # Domain layer
lib/presentation/ # Presentation layer

// Bad - Mixed concerns
lib/models/   # Data and domain mixed
lib/widgets/  # Presentation and business logic mixed
```

---

# Common Mistakes

## Everything in lib/

Wrong:
```
lib/
├── main.dart
├── auth.dart
├── home.dart
├── profile.dart
├── user.dart
├── api.dart
├── utils.dart
└── widgets.dart
```

Correct:
```
lib/
├── main.dart
├── features/
│   ├── auth/
│   └── home/
├── core/
│   ├── services/
│   └── utils/
└── shared/
    └── widgets/
```

## Mixed Responsibilities

Wrong:
```dart
// Widget contains business logic
class LoginScreen extends StatelessWidget {
  void login() {
    // API call and business logic
  }
}
```

Correct:
```dart
// Widget only handles UI
class LoginScreen extends StatelessWidget {
  final AuthProvider auth;
  // UI only
}

// Business logic in providers/services
class AuthProvider {
  Future<void> login() {
    // Business logic
  }
}
```

---

# Summary

A well-organized project structure is essential for scalable, maintainable Flutter applications. Use feature-based organization, separate concerns, and follow consistent naming conventions. Choose a structure that fits your app's complexity and team's workflow.

---

# Next Steps

- [Folder Organization](folder-organization.md)
- [Separation of Concerns](separation-of-concerns.md)
- [Architecture Patterns](architecture-patterns.md)

---

# Did You Know?

- Flutter supports multiple project structures
- Feature-based organization scales well
- Clean architecture separates layers
- Consistent naming improves readability
- Project structure affects maintainability
- Choose structure based on app size
- Team collaboration benefits from structure
- Structure can evolve with the app