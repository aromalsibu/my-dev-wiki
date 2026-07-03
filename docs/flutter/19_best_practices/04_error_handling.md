


# Error Handling

Understand how to handle errors gracefully in Flutter applications.

---

# What is it?

Error Handling is the practice of anticipating, detecting, and responding to errors that occur during application execution. It involves catching exceptions, displaying user-friendly messages, logging errors for debugging, and recovering from failures gracefully.

---

# Why does it exist?

Error Handling exists to:

- Prevent app crashes
- Improve user experience
- Provide meaningful feedback
- Log errors for debugging
- Recover from failures
- Maintain app stability
- Handle edge cases

---

# Error Handling Basics

> **Catching** and handling errors.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// 1. Basic try-catch
Future<void> fetchData() async {
  try {
    // Code that might throw an error
    final response = await http.get(Uri.parse('https://api.example.com/data'));
    if (response.statusCode == 200) {
      // Process data
    } else {
      throw Exception('Failed to load data: ${response.statusCode}');
    }
  } catch (e) {
    // Handle error
    print('Error: $e');
    // Show user-friendly message
  }
}

/// 2. Specific exception handling
class NetworkException implements Exception {
  final String message;
  NetworkException(this.message);
  @override
  String toString() => 'NetworkException: $message';
}

class AuthException implements Exception {
  final String message;
  AuthException(this.message);
  @override
  String toString() => 'AuthException: $message';
}

Future<void> login(String email, String password) async {
  try {
    // Simulate network call
    await Future.delayed(const Duration(seconds: 1));
    
    if (email.isEmpty || password.isEmpty) {
      throw AuthException('Email and password are required');
    }
    
    if (email != 'test@example.com' || password != 'password') {
      throw AuthException('Invalid credentials');
    }
    
    // Success
  } on NetworkException catch (e) {
    // Handle network errors specifically
    print('Network error: ${e.message}');
  } on AuthException catch (e) {
    // Handle auth errors specifically
    print('Auth error: ${e.message}');
  } catch (e) {
    // Handle any other errors
    print('Unexpected error: $e');
  }
}
```

What's happening here?
- try-catch catches errors
- Specific exception types for different errors
- Multiple catch blocks for different types
- Graceful error recovery

---

# Error Handling in Widgets

> **Showing** errors in the UI.

```dart
/// 1. ErrorWidget
class ErrorWidgetExample extends StatelessWidget {
  const ErrorWidgetExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Error Handling'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. FutureBuilder with error handling
            FutureBuilder<String>(
              future: _fetchData(),
              builder: (context, snapshot) {
                if (snapshot.connectionState == ConnectionState.waiting) {
                  return const CircularProgressIndicator();
                }
                
                if (snapshot.hasError) {
                  return _buildErrorWidget(
                    'Failed to load data',
                    snapshot.error.toString(),
                    () => _fetchData(),
                  );
                }
                
                return Text(snapshot.data ?? 'No data');
              },
            ),
            const SizedBox(height: 16),
            
            // 2. Custom error widget
            _buildCustomErrorWidget(),
            
            const SizedBox(height: 16),
            
            // 3. Error boundary
            ErrorBoundary(
              child: _buildRiskyWidget(),
            ),
          ],
        ),
      ),
    );
  }

  Future<String> _fetchData() async {
    await Future.delayed(const Duration(seconds: 2));
    throw Exception('Network error');
  }

  Widget _buildErrorWidget(String title, String details, VoidCallback onRetry) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.red[50],
        borderRadius: BorderRadius.circular(8),
        border: Border.all(color: Colors.red[200]!),
      ),
      child: Column(
        children: [
          const Icon(Icons.error, color: Colors.red, size: 48),
          const SizedBox(height: 8),
          Text(
            title,
            style: const TextStyle(
              fontWeight: FontWeight.bold,
              fontSize: 16,
            ),
          ),
          const SizedBox(height: 4),
          Text(
            details,
            style: TextStyle(color: Colors.grey[600], fontSize: 14),
            textAlign: TextAlign.center,
          ),
          const SizedBox(height: 16),
          ElevatedButton(
            onPressed: onRetry,
            child: const Text('Retry'),
          ),
        ],
      ),
    );
  }

  Widget _buildCustomErrorWidget() {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.orange[50],
        borderRadius: BorderRadius.circular(8),
        border: Border.all(color: Colors.orange[200]!),
      ),
      child: Row(
        children: [
          const Icon(Icons.warning, color: Colors.orange),
          const SizedBox(width: 8),
          const Expanded(
            child: Text(
              'Something went wrong. Please try again.',
              style: TextStyle(fontSize: 14),
            ),
          ),
          TextButton(
            onPressed: () {},
            child: const Text('Retry'),
          ),
        ],
      ),
    );
  }

  Widget _buildRiskyWidget() {
    // Simulate a widget that might throw
    final random = DateTime.now().millisecondsSinceEpoch % 2 == 0;
    if (random) {
      throw Exception('Widget error');
    }
    return const Text('Safe widget');
  }
}

/// 2. ErrorBoundary - catches errors in child widgets
class ErrorBoundary extends StatefulWidget {
  const ErrorBoundary({super.key, required this.child});

  final Widget child;

  @override
  State<ErrorBoundary> createState() => _ErrorBoundaryState();
}

class _ErrorBoundaryState extends State<ErrorBoundary> {
  dynamic _error;
  bool _hasError = false;

  @override
  Widget build(BuildContext context) {
    if (_hasError) {
      return Container(
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: Colors.red[50],
          borderRadius: BorderRadius.circular(8),
        ),
        child: Column(
          children: [
            const Icon(Icons.error, color: Colors.red),
            const SizedBox(height: 8),
            Text(
              'Error: ${_error.toString()}',
              style: const TextStyle(fontSize: 14),
            ),
            const SizedBox(height: 8),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _hasError = false;
                  _error = null;
                });
              },
              child: const Text('Reset'),
            ),
          ],
        ),
      );
    }

    try {
      return widget.child;
    } catch (e) {
      // Catch error and show fallback UI
      WidgetsBinding.instance.addPostFrameCallback((_) {
        setState(() {
          _hasError = true;
          _error = e;
        });
      });
      return const SizedBox.shrink();
    }
  }
}
```

What's happening here?
- FutureBuilder shows errors
- Custom error widgets
- Error boundary for widgets
- Retry functionality

---

# Error Handling in State Management

> **Handling errors** in providers and BLoC.

```dart
/// 1. Provider with error handling
class AuthProvider extends ChangeNotifier {
  User? _user;
  bool _isLoading = false;
  String? _error;

  User? get user => _user;
  bool get isLoading => _isLoading;
  String? get error => _error;
  bool get hasError => _error != null;

  Future<void> login(String email, String password) async {
    _setLoading(true);
    _clearError();

    try {
      // Simulate API call
      await Future.delayed(const Duration(seconds: 2));
      
      if (email.isEmpty || password.isEmpty) {
        throw AuthException('Email and password are required');
      }
      
      if (email != 'test@example.com' || password != 'password') {
        throw AuthException('Invalid credentials');
      }
      
      _user = User(
        id: '1',
        name: 'John Doe',
        email: email,
      );
    } on AuthException catch (e) {
      _setError(e.message);
    } on NetworkException catch (e) {
      _setError('Network error: ${e.message}');
    } catch (e) {
      _setError('Something went wrong. Please try again.');
    } finally {
      _setLoading(false);
    }
  }

  void _setLoading(bool loading) {
    _isLoading = loading;
    notifyListeners();
  }

  void _setError(String error) {
    _error = error;
    notifyListeners();
  }

  void _clearError() {
    _error = null;
    notifyListeners();
  }

  void clearError() {
    _clearError();
  }
}

/// 2. Using provider with error handling
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
                // Error display
                if (auth.hasError)
                  _buildErrorDisplay(auth.error!),
                
                const SizedBox(height: 16),
                
                // Form fields
                const TextField(
                  decoration: InputDecoration(
                    labelText: 'Email',
                    border: OutlineInputBorder(),
                  ),
                ),
                const SizedBox(height: 16),
                const TextField(
                  decoration: InputDecoration(
                    labelText: 'Password',
                    border: OutlineInputBorder(),
                  ),
                  obscureText: true,
                ),
                const SizedBox(height: 24),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: () {
                      // In a real app, use controllers for input
                      auth.login('test@example.com', 'password');
                    },
                    child: const Text('Login'),
                  ),
                ),
              ],
            );
          },
        ),
      ),
    );
  }

  Widget _buildErrorDisplay(String error) {
    return Container(
      padding: const EdgeInsets.all(12),
      decoration: BoxDecoration(
        color: Colors.red[50],
        borderRadius: BorderRadius.circular(8),
        border: Border.all(color: Colors.red[200]!),
      ),
      child: Row(
        children: [
          const Icon(Icons.error_outline, color: Colors.red),
          const SizedBox(width: 8),
          Expanded(
            child: Text(
              error,
              style: const TextStyle(color: Colors.red),
            ),
          ),
          IconButton(
            icon: const Icon(Icons.close, size: 16),
            onPressed: () {
              context.read<AuthProvider>().clearError();
            },
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Error state in provider
- Error display in UI
- Error clearing
- Different error types

---

# Error Handling with BLoC

> **Handling errors** in BLoC pattern.

```dart
/// 1. BLoC with error handling
abstract class LoginEvent {}

class LoginSubmitted extends LoginEvent {
  final String email;
  final String password;
  LoginSubmitted({required this.email, required this.password});
}

abstract class LoginState {}

class LoginInitial extends LoginState {}

class LoginLoading extends LoginState {}

class LoginSuccess extends LoginState {
  final User user;
  LoginSuccess(this.user);
}

class LoginFailure extends LoginState {
  final String error;
  LoginFailure(this.error);
}

class LoginBloc extends Bloc<LoginEvent, LoginState> {
  final AuthRepository repository;

  LoginBloc({required this.repository}) : super(LoginInitial()) {
    on<LoginSubmitted>(_onLoginSubmitted);
  }

  Future<void> _onLoginSubmitted(
    LoginSubmitted event,
    Emitter<LoginState> emit,
  ) async {
    emit(LoginLoading());

    try {
      final user = await repository.login(event.email, event.password);
      emit(LoginSuccess(user));
    } on AuthException catch (e) {
      emit(LoginFailure(e.message));
    } on NetworkException catch (e) {
      emit(LoginFailure('Network error: ${e.message}'));
    } catch (e) {
      emit(LoginFailure('Something went wrong. Please try again.'));
    }
  }
}

/// 2. Using BLoC with error handling
class LoginBlocScreen extends StatelessWidget {
  const LoginBlocScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => LoginBloc(
        repository: AuthRepository(),
      ),
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Login BLoC'),
        ),
        body: Padding(
          padding: const EdgeInsets.all(16),
          child: BlocConsumer<LoginBloc, LoginState>(
            listener: (context, state) {
              if (state is LoginFailure) {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text(state.error),
                    backgroundColor: Colors.red,
                  ),
                );
              }
              if (state is LoginSuccess) {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(
                    content: Text('Login successful!'),
                    backgroundColor: Colors.green,
                  ),
                );
              }
            },
            builder: (context, state) {
              if (state is LoginLoading) {
                return const Center(child: CircularProgressIndicator());
              }

              return Column(
                children: [
                  const TextField(
                    decoration: InputDecoration(
                      labelText: 'Email',
                      border: OutlineInputBorder(),
                    ),
                  ),
                  const SizedBox(height: 16),
                  const TextField(
                    decoration: InputDecoration(
                      labelText: 'Password',
                      border: OutlineInputBorder(),
                    ),
                    obscureText: true,
                  ),
                  const SizedBox(height: 24),
                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton(
                      onPressed: () {
                        context.read<LoginBloc>().add(
                          LoginSubmitted(
                            email: 'test@example.com',
                            password: 'password',
                          ),
                        );
                      },
                      child: const Text('Login'),
                    ),
                  ),
                ],
              );
            },
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Error states in BLoC
- Error handling in events
- UI error display
- Snackbar errors

---

# Global Error Handling

> **Catching** errors globally.

```dart
/// 1. Global error handler
void main() {
  // Setup global error handling
  WidgetsFlutterBinding.ensureInitialized();
  
  // Catch Flutter framework errors
  FlutterError.onError = (FlutterErrorDetails details) {
    // Log error
    print('Flutter Error: ${details.exception}');
    print('Stack trace: ${details.stack}');
    
    // Report to crash reporting service
    // reportError(details.exception, details.stack);
    
    // Show error in debug mode
    if (kDebugMode) {
      FlutterError.dumpErrorToConsole(details);
    }
  };
  
  // Catch platform errors
  PlatformDispatcher.instance.onError = (error, stack) {
    print('Platform Error: $error');
    print('Stack: $stack');
    
    // Report to crash reporting service
    // reportError(error, stack);
    
    return true; // Prevent crash
  };
  
  runApp(const MyApp());
}

/// 2. Error reporting service
class ErrorReportingService {
  static final ErrorReportingService _instance = ErrorReportingService._internal();
  factory ErrorReportingService() => _instance;
  ErrorReportingService._internal();

  void reportError(dynamic error, StackTrace? stack) {
    // Log to console
    print('Error: $error');
    if (stack != null) {
      print('Stack: $stack');
    }
    
    // Send to backend
    _sendToBackend(error, stack);
    
    // Store locally
    _storeLocally(error, stack);
  }

  Future<void> _sendToBackend(dynamic error, StackTrace? stack) async {
    try {
      // Send error report to server
      await http.post(
        Uri.parse('https://api.example.com/error'),
        body: {
          'error': error.toString(),
          'stack': stack?.toString() ?? '',
          'timestamp': DateTime.now().toIso8601String(),
        },
      );
    } catch (e) {
      // Don't throw if error reporting fails
      print('Failed to send error: $e');
    }
  }

  void _storeLocally(dynamic error, StackTrace? stack) {
    // Store error locally for later reporting
    // Implementation depends on your storage solution
  }
}

/// 3. Custom error widget
class CustomErrorWidget extends StatelessWidget {
  const CustomErrorWidget({super.key, required this.error});

  final dynamic error;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(32),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(
                Icons.error_outline,
                size: 64,
                color: Colors.red,
              ),
              const SizedBox(height: 16),
              const Text(
                'Something went wrong',
                style: TextStyle(
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 8),
              Text(
                error.toString(),
                textAlign: TextAlign.center,
                style: const TextStyle(fontSize: 14, color: Colors.grey),
              ),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () {
                  // Restart app
                  // Implementation depends on your navigation
                },
                child: const Text('Restart App'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Global error handler for Flutter errors
- Platform error handler
- Error reporting service
- Custom error widget

---

# Best Practices

## Always Handle Errors

```dart
// Good - Error handling
try {
  await fetchData();
} catch (e) {
  // Handle error
  showError(e);
}

// Bad - No error handling
await fetchData();
```

## Show User-Friendly Messages

```dart
// Good - User-friendly message
SnackBar(content: Text('Failed to load data. Please try again.'))

// Bad - Technical error message
SnackBar(content: Text('Exception: NetworkError at line 42'))
```

## Log Errors

```dart
// Good - Log errors
try {
  // Code
} catch (e, stack) {
  print('Error: $e');
  print('Stack: $stack');
  // Report to crash reporting service
}

// Bad - Silent failure
try {
  // Code
} catch (e) {
  // Do nothing
}
```

---

# Common Mistakes

## Not Handling Async Errors

Wrong:
```dart
// No error handling
Future<void> fetchData() async {
  final data = await api.getData();
}
```

Correct:
```dart
// With error handling
Future<void> fetchData() async {
  try {
    final data = await api.getData();
  } catch (e) {
    // Handle error
  }
}
```

## Showing Technical Errors to Users

Wrong:
```dart
// Shows stack trace to user
SnackBar(content: Text('$e'))
```

Correct:
```dart
// Shows user-friendly message
SnackBar(content: Text('Something went wrong. Please try again.'))
```

---

# Summary

Error Handling is essential for building robust Flutter applications. Use try-catch for async operations, show user-friendly messages, log errors for debugging, and handle errors globally. Proper error handling prevents crashes and improves user experience.

---

# Next Steps

- [Responsive Design](responsive-design.md)
- [Adaptive Design](adaptive-design.md)
- [Architecture Patterns](architecture-patterns.md)

---

# Did You Know?

- Flutter has global error handlers
- Specific exceptions improve debugging
- Error boundaries catch widget errors
- User-friendly messages improve UX
- Error logging aids debugging
- Crash reporting services help fix issues
- Proper error handling prevents crashes
- Error handling is a best practice