# Folder Organization

Understand how to organize folders and files in Flutter applications for better maintainability and scalability.

---

# What is it?

Folder Organization refers to the logical grouping of files and directories in your Flutter project. It's the practical implementation of your chosen project structure, determining where different types of files (widgets, models, services, etc.) should be placed. Good folder organization makes your codebase easier to navigate, understand, and maintain.

---

# Why does it exist?

Folder Organization exists to:

- Make code easier to find and navigate
- Group related files together
- Separate different concerns
- Support team collaboration
- Enable scalable development
- Improve code maintainability
- Follow industry best practices

---

# Type-Based Organization

> **Organizing** by file type.

```
lib/
├── main.dart
├── models/                     # Data models
│   ├── user.dart
│   ├── product.dart
│   └── order.dart
├── views/                      # UI screens
│   ├── home_screen.dart
│   ├── profile_screen.dart
│   └── settings_screen.dart
├── widgets/                    # Reusable widgets
│   ├── buttons.dart
│   ├── cards.dart
│   └── inputs.dart
├── services/                   # Business logic
│   ├── api_service.dart
│   ├── auth_service.dart
│   └── storage_service.dart
├── providers/                  # State management
│   ├── auth_provider.dart
│   └── theme_provider.dart
├── utils/                      # Utility functions
│   ├── validators.dart
│   ├── formatters.dart
│   └── helpers.dart
├── constants/                  # Constants
│   ├── app_constants.dart
│   └── theme_constants.dart
└── extensions/                 # Dart extensions
    ├── context_extensions.dart
    └── string_extensions.dart
```

What's happening here?
- Files grouped by type
- Clear separation of concerns
- Easy to find related files
- Simple and straightforward

---

# Feature-Based Organization

> **Organizing** by features.

```
lib/
├── main.dart
├── core/                       # Core functionality
│   ├── constants/
│   ├── utils/
│   ├── services/
│   └── widgets/
├── features/                   # Feature modules
│   ├── auth/                   # Authentication feature
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
│   ├── home/                   # Home feature
│   │   ├── models/
│   │   ├── screens/
│   │   ├── widgets/
│   │   └── services/
│   └── profile/                # Profile feature
│       ├── models/
│       ├── screens/
│       ├── widgets/
│       └── services/
└── shared/                     # Shared across features
    ├── widgets/
    │   ├── buttons.dart
    │   └── cards.dart
    ├── extensions/
    │   └── context_extensions.dart
    └── constants/
        └── app_constants.dart
```

What's happening here?
- Features are self-contained
- Easy to add/remove features
- Clear feature boundaries
- Scalable for large projects

---

# Clean Architecture Organization

> **Organizing** by layers.

```
lib/
├── main.dart
├── data/                       # Data layer
│   ├── datasources/            # Data sources
│   │   ├── local/
│   │   │   └── local_datasource.dart
│   │   └── remote/
│   │       └── remote_datasource.dart
│   ├── models/                 # Data models
│   │   ├── user_model.dart
│   │   └── todo_model.dart
│   └── repositories/           # Repository implementations
│       ├── auth_repository_impl.dart
│       └── todo_repository_impl.dart
├── domain/                     # Domain layer
│   ├── entities/               # Business entities
│   │   ├── user.dart
│   │   └── todo.dart
│   ├── repositories/           # Repository interfaces
│   │   ├── auth_repository.dart
│   │   └── todo_repository.dart
│   └── usecases/               # Use cases
│       ├── auth/
│       │   ├── login_usecase.dart
│       │   └── register_usecase.dart
│       └── todo/
│           ├── get_todos_usecase.dart
│           └── add_todo_usecase.dart
└── presentation/               # Presentation layer
    ├── blocs/                  # State management
    │   ├── auth/
    │   │   ├── auth_bloc.dart
    │   │   └── auth_event.dart
    │   └── todo/
    │       ├── todo_bloc.dart
    │       └── todo_event.dart
    ├── pages/                  # Screens
    │   ├── auth/
    │   │   ├── login_page.dart
    │   │   └── register_page.dart
    │   └── home/
    │       └── home_page.dart
    └── widgets/                # UI widgets
        ├── common/
        └── auth/
```

What's happening here?
- Layers are clearly separated
- Dependency direction is clear
- Business logic is isolated
- High testability

---

# Example Implementation

> **Implementing** folder organization.

```dart
// 1. lib/main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'features/auth/providers/auth_provider.dart';
import 'features/auth/screens/login_screen.dart';
import 'features/home/screens/home_screen.dart';
import 'core/services/storage_service.dart';
import 'core/constants/app_constants.dart';

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
        title: AppConstants.appName,
        theme: ThemeData(primarySwatch: Colors.blue),
        home: Consumer<AuthProvider>(
          builder: (context, auth, _) {
            if (auth.isAuthenticated) {
              return const HomeScreen();
            }
            return const LoginScreen();
          },
        ),
      ),
    );
  }
}

// 2. lib/core/constants/app_constants.dart
class AppConstants {
  static const String appName = 'My App';
  static const String apiBaseUrl = 'https://api.example.com';
  static const String tokenKey = 'auth_token';
}

// 3. lib/core/services/storage_service.dart
import 'package:shared_preferences/shared_preferences.dart';

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

  Future<void> clear() async {
    await _prefs.clear();
  }
}

// 4. lib/features/auth/models/user.dart
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

// 5. lib/features/auth/providers/auth_provider.dart
import 'package:flutter/material.dart';
import '../models/user.dart';

class AuthProvider extends ChangeNotifier {
  User? _user;
  bool _isAuthenticated = false;

  User? get user => _user;
  bool get isAuthenticated => _isAuthenticated;

  Future<void> login(String email, String password) async {
    // Simulate login
    await Future.delayed(const Duration(seconds: 1));
    _user = User(
      id: '1',
      name: 'John Doe',
      email: email,
    );
    _isAuthenticated = true;
    notifyListeners();
  }

  Future<void> logout() async {
    _user = null;
    _isAuthenticated = false;
    notifyListeners();
  }
}

// 6. lib/features/auth/widgets/login_form.dart
import 'package:flutter/material.dart';

class LoginForm extends StatefulWidget {
  const LoginForm({super.key, required this.onLogin});

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

// 7. lib/features/auth/screens/login_screen.dart
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

// 8. lib/shared/widgets/custom_button.dart
import 'package:flutter/material.dart';

class CustomButton extends StatelessWidget {
  const CustomButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.isPrimary = true,
  });

  final String label;
  final VoidCallback onPressed;
  final bool isPrimary;

  @override
  Widget build(BuildContext context) {
    return Container(
      width: double.infinity,
      height: 50,
      child: ElevatedButton(
        onPressed: onPressed,
        style: ElevatedButton.styleFrom(
          backgroundColor: isPrimary ? Colors.blue : Colors.grey[300],
          foregroundColor: isPrimary ? Colors.white : Colors.black,
        ),
        child: Text(label),
      ),
    );
  }
}
```

What's happening here?
- core/ for app-wide functionality
- features/ for feature modules
- shared/ for shared widgets
- Clear import paths
- Organized by responsibility

---

# Best Practices

## Be Consistent

```dart
// Good - Consistent naming
features/
  auth/
    auth_screen.dart
    auth_provider.dart
    auth_service.dart

// Bad - Inconsistent naming
features/
  auth/
    auth_screen.dart
    authProvider.dart
    AuthService.dart
```

## Use Index Files

```dart
// Good - Index file for exports
// lib/features/auth/index.dart
export 'models/user.dart';
export 'providers/auth_provider.dart';
export 'screens/login_screen.dart';
export 'widgets/login_form.dart';

// Usage
import 'package:my_app/features/auth/index.dart';
```

## Keep Folders Shallow

```dart
// Good - Shallow nesting
features/
  auth/
    models/
    screens/
    widgets/

// Bad - Deep nesting
features/
  auth/
    ui/
      screens/
        auth/
          login/
            login_screen.dart
```

---

# Common Mistakes

## Too Many Levels

Wrong:
```
lib/
├── features/
│   └── auth/
│       └── presentation/
│           └── screens/
│               └── auth_screen/
│                   └── widgets/
│                       └── login/
│                           └── login_button.dart
```

Correct:
```
lib/
├── features/
│   └── auth/
│       ├── screens/
│       │   └── auth_screen.dart
│       └── widgets/
│           └── login_button.dart
```

## Mixed Concerns

Wrong:
```
lib/
├── models/
│   ├── auth/
│   └── home/
├── screens/
│   ├── auth/
│   └── home/
└── services/
    ├── auth/
    └── home/
```

Correct:
```
lib/
└── features/
    ├── auth/
    │   ├── models/
    │   ├── screens/
    │   └── services/
    └── home/
        ├── models/
        ├── screens/
        └── services/
```

---

# Summary

Folder Organization is the practical implementation of your project structure. Choose an organization strategy (type-based, feature-based, or layer-based) that fits your project. Be consistent, use clear naming, and keep folders organized. Good folder organization makes your codebase easier to navigate and maintain.

---

# Next Steps

- [Separation of Concerns](separation-of-concerns.md)
- [Architecture Patterns](architecture-patterns.md)
- [Project Structure](project-structure.md)

---

# Did You Know?

- Feature-based organization scales well
- Type-based organization is simpler
- Clean architecture separates layers
- Index files simplify imports
- Consistent naming improves navigation
- Folder organization affects productivity
- Structure can evolve with the project
- Team consensus is important