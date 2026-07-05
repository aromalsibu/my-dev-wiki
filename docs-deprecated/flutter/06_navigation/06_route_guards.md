# Route Guards

Understand how to protect routes with authentication and authorization checks.

---

# What is it?

Route guards are mechanisms that control access to specific routes based on certain conditions like authentication status, user roles, or other business logic. They act as gatekeepers that determine whether a user can navigate to a particular screen, redirecting them to appropriate destinations if they don't meet the requirements.

---

# Why does it exist?

Route guards exist to:

- Protect routes from unauthorized access
- Implement authentication requirements
- Enforce role-based access control
- Redirect users to appropriate screens
- Handle session validation
- Prevent route access in invalid states
- Implement authorization logic

---

# Authentication Guards

> **Protecting routes** with authentication checks.

```dart
// Authentication guard with GoRouter
import 'package:go_router/go_router.dart';
import 'package:provider/provider.dart';

// Auth state management
class AuthState extends ChangeNotifier {
  bool _isAuthenticated = false;
  String? _userToken;
  
  bool get isAuthenticated => _isAuthenticated;
  String? get token => _userToken;
  
  Future<void> login(String username, String password) async {
    // Simulate login
    await Future.delayed(const Duration(seconds: 1));
    _isAuthenticated = true;
    _userToken = 'fake-token-123';
    notifyListeners();
  }
  
  Future<void> logout() async {
    _isAuthenticated = false;
    _userToken = null;
    notifyListeners();
  }
}

class AuthGuardApp extends StatelessWidget {
  const AuthGuardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => AuthState(),
      child: MaterialApp.router(
        routerConfig: _authGuardRouter,
      ),
    );
  }
}

final _authGuardRouter = GoRouter(
  initialLocation: '/',
  // Global redirect for authentication
  redirect: (context, state) {
    final auth = Provider.of<AuthState>(context);
    final isAuthenticated = auth.isAuthenticated;
    final isLoginRoute = state.matchedLocation == '/login';
    
    // If not authenticated and not on login, go to login
    if (!isAuthenticated && !isLoginRoute) {
      return '/login';
    }
    
    // If authenticated and on login, go to home
    if (isAuthenticated && isLoginRoute) {
      return '/';
    }
    
    return null; // No redirect
  },
  routes: [
    // Public routes
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
    
    // Protected routes (requires auth)
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
      routes: [
        GoRoute(
          path: 'profile',
          builder: (context, state) => const ProfileScreen(),
        ),
        GoRoute(
          path: 'settings',
          builder: (context, state) => const SettingsScreen(),
        ),
        GoRoute(
          path: 'dashboard',
          builder: (context, state) => const DashboardScreen(),
        ),
      ],
    ),
  ],
  errorBuilder: (context, state) => const ErrorScreen(),
);
```

What's happening here?
- AuthState manages authentication
- Global redirect checks auth status
- Login route is public
- All other routes require auth
- Redirects to login if not authenticated

---

# Role-Based Guards

> **Controlling access** based on user roles.

```dart
// Role-based access control
class UserState extends ChangeNotifier {
  String _role = 'guest';
  
  String get role => _role;
  
  bool get isAdmin => _role == 'admin';
  bool get isManager => _role == 'manager';
  bool get isUser => _role == 'user';
  
  void setRole(String role) {
    _role = role;
    notifyListeners();
  }
}

class RoleGuardApp extends StatelessWidget {
  const RoleGuardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => UserState(),
      child: MaterialApp.router(
        routerConfig: _roleGuardRouter,
      ),
    );
  }
}

final _roleGuardRouter = GoRouter(
  initialLocation: '/',
  redirect: (context, state) {
    final user = Provider.of<UserState>(context);
    
    // Check if route requires specific role
    final location = state.matchedLocation;
    
    // Admin only routes
    if (location.startsWith('/admin') && !user.isAdmin) {
      return '/unauthorized';
    }
    
    // Manager routes
    if (location.startsWith('/manager') && !user.isManager && !user.isAdmin) {
      return '/unauthorized';
    }
    
    // User routes (any authenticated user)
    if (location.startsWith('/dashboard') && user.role == 'guest') {
      return '/login';
    }
    
    return null;
  },
  routes: [
    // Public
    GoRoute(
      path: '/',
      builder: (context, state) => const PublicHomeScreen(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
    GoRoute(
      path: '/unauthorized',
      builder: (context, state) => const UnauthorizedScreen(),
    ),
    
    // User routes
    GoRoute(
      path: '/dashboard',
      builder: (context, state) => const DashboardScreen(),
    ),
    
    // Manager routes
    GoRoute(
      path: '/manager/reports',
      builder: (context, state) => const ReportsScreen(),
    ),
    
    // Admin routes
    GoRoute(
      path: '/admin/users',
      builder: (context, state) => const UsersManagementScreen(),
    ),
    GoRoute(
      path: '/admin/settings',
      builder: (context, state) => const AdminSettingsScreen(),
    ),
  ],
);
```

What's happening here?
- UserState manages user role
- Role-based redirect logic
- Admin, manager, user roles
- Unauthorized screen for access denied

---

# Route-Level Guards

> **Protecting individual** routes with guards.

```dart
// Route-level guards
class RouteGuardApp extends StatelessWidget {
  const RouteGuardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => AuthState(),
      child: MaterialApp.router(
        routerConfig: _routeGuardRouter,
      ),
    );
  }
}

final _routeGuardRouter = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
    // Route with guard
    GoRoute(
      path: '/profile',
      builder: (context, state) => const ProfileScreen(),
      // Route-level redirect
      redirect: (context, state) {
        final auth = Provider.of<AuthState>(context);
        if (!auth.isAuthenticated) {
          return '/login';
        }
        return null;
      },
    ),
    // Route with specific conditions
    GoRoute(
      path: '/settings',
      builder: (context, state) => const SettingsScreen(),
      redirect: (context, state) {
        final auth = Provider.of<AuthState>(context);
        final user = Provider.of<UserState>(context);
        
        if (!auth.isAuthenticated) {
          return '/login';
        }
        if (!user.isAdmin) {
          return '/unauthorized';
        }
        return null;
      },
    ),
  ],
);

// Custom guard for specific features
class FeatureGuard {
  static String? checkAccess(BuildContext context, String feature) {
    final auth = Provider.of<AuthState>(context);
    final user = Provider.of<UserState>(context);
    
    if (!auth.isAuthenticated) {
      return '/login';
    }
    
    // Feature-specific checks
    if (feature == 'admin' && !user.isAdmin) {
      return '/unauthorized';
    }
    
    if (feature == 'premium' && !user.isPremium) {
      return '/upgrade';
    }
    
    return null;
  }
}

// Using custom guard
class FeatureRoutes {
  static final routes = [
    GoRoute(
      path: '/admin',
      builder: (context, state) => const AdminPanelScreen(),
      redirect: (context, state) {
        return FeatureGuard.checkAccess(context, 'admin');
      },
    ),
    GoRoute(
      path: '/premium',
      builder: (context, state) => const PremiumContentScreen(),
      redirect: (context, state) {
        return FeatureGuard.checkAccess(context, 'premium');
      },
    ),
  ];
}
```

What's happening here?
- Route-level redirects
- Multiple condition checks
- Custom guard classes
- Feature-based access control

---

# Guard with Async Checks

> **Asynchronous** route guards.

```dart
// Async route guards
class AsyncGuardApp extends StatelessWidget {
  const AsyncGuardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => AsyncAuthState(),
      child: MaterialApp.router(
        routerConfig: _asyncGuardRouter,
      ),
    );
  }
}

class AsyncAuthState extends ChangeNotifier {
  bool _isLoading = true;
  bool _isAuthenticated = false;
  
  bool get isLoading => _isLoading;
  bool get isAuthenticated => _isAuthenticated;
  
  AsyncAuthState() {
    _checkAuth();
  }
  
  Future<void> _checkAuth() async {
    // Simulate async auth check
    await Future.delayed(const Duration(seconds: 2));
    _isAuthenticated = true;
    _isLoading = false;
    notifyListeners();
  }
  
  Future<void> login() async {
    await Future.delayed(const Duration(seconds: 1));
    _isAuthenticated = true;
    notifyListeners();
  }
}

final _asyncGuardRouter = GoRouter(
  initialLocation: '/',
  redirect: (context, state) {
    final auth = Provider.of<AsyncAuthState>(context);
    
    // Show loading while checking auth
    if (auth.isLoading) {
      return '/splash';
    }
    
    // Redirect based on auth status
    if (!auth.isAuthenticated) {
      return '/login';
    }
    
    return null;
  },
  routes: [
    GoRoute(
      path: '/splash',
      builder: (context, state) => const SplashScreen(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
  ],
);

// Custom AsyncGuard
class AsyncGuard {
  static Future<String?> check(
    BuildContext context,
    String route,
  ) async {
    final auth = Provider.of<AsyncAuthState>(context);
    
    // Simulate async validation
    await Future.delayed(const Duration(milliseconds: 500));
    
    if (!auth.isAuthenticated) {
      return '/login';
    }
    
    return null;
  }
}
```

What's happening here?
- Async auth state checking
- Loading screen during auth check
- Async guard class
- Simulated network delays

---

# Real-World Examples

> **Common patterns** with route guards.

```dart
// 1. Complete auth guard implementation
class AuthGuard {
  static String? check(BuildContext context, String route) {
    final auth = Provider.of<AuthState>(context);
    final user = Provider.of<UserState>(context);
    
    // Check authentication
    if (!auth.isAuthenticated) {
      return '/login';
    }
    
    // Check session timeout
    if (auth.isSessionExpired) {
      return '/session-expired';
    }
    
    // Check account status
    if (!user.isVerified) {
      return '/verify-email';
    }
    
    // Check subscription status
    if (route.startsWith('/premium') && !user.isSubscribed) {
      return '/pricing';
    }
    
    return null;
  }
}

// 2. Role-based guard with redirects
class RoleGuard {
  static String? check(
    BuildContext context,
    String route,
    List<String> allowedRoles,
  ) {
    final user = Provider.of<UserState>(context);
    
    if (route == '/admin') {
      if (user.isAdmin) return null;
      if (user.isManager) return '/manager/dashboard';
      return '/unauthorized';
    }
    
    if (!allowedRoles.contains(user.role)) {
      return '/unauthorized';
    }
    
    return null;
  }
}

// 3. Multi-factor guard
class MfaGuard {
  static String? check(BuildContext context, String route) {
    final auth = Provider.of<AuthState>(context);
    final mfa = Provider.of<MfaState>(context);
    
    if (!auth.isAuthenticated) {
      return '/login';
    }
    
    if (auth.requiresMfa && !mfa.isVerified) {
      return '/mfa-verify';
    }
    
    return null;
  }
}

// 4. Registration flow guard
class RegistrationFlowGuard {
  static String? check(BuildContext context, String route) {
    final flow = Provider.of<RegistrationState>(context);
    
    if (route == '/register/step1' && flow.step > 0) {
      return '/register/step${flow.step + 1}';
    }
    
    if (route == '/register/step2' && flow.step < 1) {
      return '/register/step1';
    }
    
    if (route == '/register/complete' && flow.step < 2) {
      return '/register/step${flow.step + 1}';
    }
    
    return null;
  }
}

// Router with all guards
final _fullGuardRouter = GoRouter(
  initialLocation: '/',
  routes: [
    // Public routes
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
    GoRoute(
      path: '/verify-email',
      builder: (context, state) => const VerifyEmailScreen(),
    ),
    GoRoute(
      path: '/mfa-verify',
      builder: (context, state) => const MfaVerifyScreen(),
    ),
    
    // Registration flow
    GoRoute(
      path: '/register/step1',
      builder: (context, state) => const RegisterStep1Screen(),
      redirect: (context, state) => RegistrationFlowGuard.check(context, state.matchedLocation),
    ),
    GoRoute(
      path: '/register/step2',
      builder: (context, state) => const RegisterStep2Screen(),
      redirect: (context, state) => RegistrationFlowGuard.check(context, state.matchedLocation),
    ),
    GoRoute(
      path: '/register/complete',
      builder: (context, state) => const RegisterCompleteScreen(),
      redirect: (context, state) => RegistrationFlowGuard.check(context, state.matchedLocation),
    ),
    
    // Protected routes
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
      redirect: (context, state) => AuthGuard.check(context, state.matchedLocation),
    ),
    GoRoute(
      path: '/profile',
      builder: (context, state) => const ProfileScreen(),
      redirect: (context, state) => AuthGuard.check(context, state.matchedLocation),
    ),
    GoRoute(
      path: '/admin',
      builder: (context, state) => const AdminScreen(),
      redirect: (context, state) => RoleGuard.check(context, state.matchedLocation, ['admin']),
    ),
    GoRoute(
      path: '/premium',
      builder: (context, state) => const PremiumScreen(),
      redirect: (context, state) => AuthGuard.check(context, state.matchedLocation),
    ),
  ],
);
```

What's happening here?
- Complete auth guard implementation
- Role-based guard with redirects
- Multi-factor authentication guard
- Registration flow guard
- Multiple guards combined

---

# Best Practices

## Centralize Guard Logic

```dart
// Good - Centralized guards
class AppGuards {
  static String? authCheck(BuildContext context, String route) {
    final auth = Provider.of<AuthState>(context);
    if (!auth.isAuthenticated) return '/login';
    return null;
  }
  
  static String? adminCheck(BuildContext context, String route) {
    final user = Provider.of<UserState>(context);
    if (!user.isAdmin) return '/unauthorized';
    return null;
  }
}

// Bad - Scattered guard logic
GoRoute(
  path: '/profile',
  redirect: (context, state) {
    final auth = Provider.of<AuthState>(context);
    if (!auth.isAuthenticated) return '/login';
    return null;
  },
)
```

## Use Redirects for Invalid States

```dart
// Good - Redirect to appropriate screens
redirect: (context, state) {
  if (!auth.isAuthenticated) return '/login';
  if (!user.isVerified) return '/verify-email';
  return null;
}

// Bad - Letting users access invalid states
redirect: (context, state) {
  if (!auth.isAuthenticated) return null; // Shows protected screen
  return null;
}
```

## Provide Feedback for Guard Failures

```dart
// Good - Show why access was denied
redirect: (context, state) {
  if (!user.isAdmin) {
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Admin access required')),
    );
    return '/unauthorized';
  }
  return null;
}
```

---

# Common Mistakes

## Not Handling Guard Failures

Wrong:
```dart
// Guard fails silently
redirect: (context, state) {
  if (!auth.isAuthenticated) {
    // User stays on protected route
    return null;
  }
  return null;
}
```

Correct:
```dart
// Redirect on failure
redirect: (context, state) {
  if (!auth.isAuthenticated) {
    return '/login';
  }
  return null;
}
```

## Overcomplicating Guards

Wrong:
```dart
// Too complex
redirect: (context, state) {
  final auth = Provider.of<AuthState>(context);
  final user = Provider.of<UserState>(context);
  final settings = Provider.of<SettingsState>(context);
  // Too many conditions
  if (auth.isAuthenticated && user.isVerified && settings.featureEnabled) {
    return null;
  }
  // Complex logic
}
```

Correct:
```dart
// Simplified guards
redirect: (context, state) {
  return AuthGuard.check(context, state.matchedLocation);
}
```

---

# Summary

Route guards protect routes from unauthorized access using authentication, authorization, and business logic checks. Use GoRouter's redirect functionality to implement guards, manage authentication state, and provide appropriate feedback to users when access is denied.

---

# Next Steps

- [Navigation Patterns](navigation-patterns.md)
- [State Management](../state-management/local-state.md)
- [Provider](../state-management/provider.md)

---

# Did You Know?

- Guards can be global or route-specific
- Authentication guards protect routes
- Role-based guards control permissions
- Async guards handle network checks
- Guards can redirect to login or error screens
- Multiple guards can be combined
- Guards provide user feedback
- Guards improve app security