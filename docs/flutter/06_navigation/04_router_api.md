# Router API

Understand the modern routing system in Flutter using the Router API.

---

# What is it?

The Router API is a modern navigation system introduced in Flutter that provides more control and flexibility than the traditional Navigator 1.0 approach. It uses a declarative routing model where routes are defined in a configuration object and can be dynamically updated based on the app state, enabling features like deep linking, URL-based navigation, and more complex routing scenarios.

---

# Why does it exist?

The Router API exists to:

- Support declarative routing
- Enable URL-based navigation
- Handle deep linking elegantly
- Manage complex navigation flows
- Support web and desktop navigation
- Provide more control over routing
- Enable dynamic route updates

---

# Router vs Navigator

> **Comparing** Router API with traditional Navigator.

```dart
// Traditional Navigator (1.0)
class Navigator1Example extends StatelessWidget {
  const Navigator1Example({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Navigator 1.0',
      initialRoute: '/',
      routes: {
        '/': (context) => const HomeScreen(),
        '/detail': (context) => const DetailScreen(),
      },
    );
  }
}

// Modern Router API (2.0)
class RouterApiExample extends StatelessWidget {
  const RouterApiExample({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Router API',
      routerConfig: router,
    );
  }
}

// Router configuration
final router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/detail',
      builder: (context, state) => const DetailScreen(),
    ),
  ],
);

// Router properties:
// 1. initialLocation - Starting route
// 2. routes - List of route definitions
// 3. redirect - Redirect logic
// 4. errorBuilder - Error handling
```

What's happening here?
- Router API is declarative
- Navigator 1.0 is imperative
- Router supports URL-based navigation
- Router enables deep linking

---

# GoRouter Basics

> **GoRouter** is the most common Router API implementation.

```dart
// Basic GoRouter setup
import 'package:go_router/go_router.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'GoRouter Demo',
      routerConfig: _router,
    );
  }
}

// Define router
final _router = GoRouter(
  initialLocation: '/',
  routes: [
    // Home route
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    
    // Detail route
    GoRoute(
      path: '/detail',
      builder: (context, state) => const DetailScreen(),
    ),
    
    // Profile route with parameter
    GoRoute(
      path: '/profile/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return ProfileScreen(userId: int.parse(id));
      },
    ),
  ],
);

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Navigate using GoRouter
            ElevatedButton(
              onPressed: () {
                context.go('/detail');
              },
              child: const Text('Go to Detail'),
            ),
            ElevatedButton(
              onPressed: () {
                context.go('/profile/123');
              },
              child: const Text('Go to Profile'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- GoRouter defines routes declaratively
- context.go() navigates to routes
- Path parameters with :id syntax
- State contains route information

---

# Nested Routes

> **Nesting routes** with GoRouter.

```dart
// Nested routes with GoRouter
class NestedRoutesApp extends StatelessWidget {
  const NestedRoutesApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _nestedRouter,
    );
  }
}

final _nestedRouter = GoRouter(
  initialLocation: '/',
  routes: [
    // Root route with ShellBranch for nested navigation
    ShellRoute(
      builder: (context, state, child) {
        return ScaffoldWithNavBar(child: child);
      },
      routes: [
        // Home tab
        GoRoute(
          path: '/',
          builder: (context, state) => const HomeTab(),
        ),
        
        // Settings tab
        GoRoute(
          path: '/settings',
          builder: (context, state) => const SettingsTab(),
        ),
        
        // Profile tab with nested routes
        GoRoute(
          path: '/profile',
          builder: (context, state) => const ProfileTab(),
          routes: [
            GoRoute(
              path: 'edit',
              builder: (context, state) => const EditProfileScreen(),
            ),
            GoRoute(
              path: 'settings',
              builder: (context, state) => const ProfileSettingsScreen(),
            ),
          ],
        ),
      ],
    ),
  ],
);

// Scaffold with bottom navigation
class ScaffoldWithNavBar extends StatelessWidget {
  const ScaffoldWithNavBar({
    super.key,
    required this.child,
  });

  final Widget child;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.settings), label: 'Settings'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
        onTap: (index) {
          switch (index) {
            case 0:
              context.go('/');
              break;
            case 1:
              context.go('/settings');
              break;
            case 2:
              context.go('/profile');
              break;
          }
        },
      ),
    );
  }
}
```

What's happening here?
- ShellRoute provides persistent layout
- Nested routes under tabs
- context.go() updates navigation
- BottomNavigationBar integration

---

# Redirects and Guards

> **Redirecting routes** with GoRouter.

```dart
// Route guards and redirects
class AuthState extends ChangeNotifier {
  bool _isLoggedIn = false;
  
  bool get isLoggedIn => _isLoggedIn;
  
  void login() {
    _isLoggedIn = true;
    notifyListeners();
  }
  
  void logout() {
    _isLoggedIn = false;
    notifyListeners();
  }
}

class GuardedRoutesApp extends StatelessWidget {
  const GuardedRoutesApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => AuthState(),
      child: MaterialApp.router(
        routerConfig: _guardedRouter,
      ),
    );
  }
}

final _guardedRouter = GoRouter(
  initialLocation: '/',
  redirect: (context, state) {
    // Check auth state
    final auth = Provider.of<AuthState>(context);
    final isLoggedIn = auth.isLoggedIn;
    final isLoginRoute = state.matchedLocation == '/login';
    
    // Redirect to login if not authenticated
    if (!isLoggedIn && !isLoginRoute) {
      return '/login';
    }
    
    // Redirect to home if already logged in
    if (isLoggedIn && isLoginRoute) {
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
    
    // Protected routes
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
      ],
    ),
  ],
  errorBuilder: (context, state) => const ErrorScreen(),
);

class LoginScreen extends StatelessWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // Login and navigate to home
            context.read<AuthState>().login();
            context.go('/');
          },
          child: const Text('Login'),
        ),
      ),
    );
  }
}
```

What's happening here?
- redirect controls navigation flow
- Auth state determines routes
- Login redirects to home
- Protected routes require auth

---

# Passing Data with GoRouter

> **Passing data** between routes.

```dart
// Passing data with GoRouter
class DataRoutingApp extends StatelessWidget {
  const DataRoutingApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _dataRouter,
    );
  }
}

final _dataRouter = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const DataHomeScreen(),
    ),
    GoRoute(
      path: '/detail',
      builder: (context, state) {
        // Extract data from state
        final extra = state.extra as Map<String, dynamic>?;
        return DetailScreen(
          title: extra?['title'] ?? 'Default',
          count: extra?['count'] ?? 0,
        );
      },
    ),
    GoRoute(
      path: '/user/:id',
      builder: (context, state) {
        // Extract path parameter
        final id = state.pathParameters['id'] ?? '0';
        // Extract query parameters
        final name = state.uri.queryParameters['name'] ?? '';
        return UserScreen(
          id: int.parse(id),
          name: name,
        );
      },
    ),
  ],
);

class DataHomeScreen extends StatelessWidget {
  const DataHomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Data Routing')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Pass data via extra
            ElevatedButton(
              onPressed: () {
                context.push(
                  '/detail',
                  extra: {
                    'title': 'Hello World',
                    'count': 42,
                  },
                );
              },
              child: const Text('Go to Detail (Extra)'),
            ),
            
            // Pass data via path and query
            ElevatedButton(
              onPressed: () {
                context.push('/user/123?name=John');
              },
              child: const Text('Go to User (Path/Query)'),
            ),
          ],
        ),
      ),
    );
  }
}

class DetailScreen extends StatelessWidget {
  const DetailScreen({
    super.key,
    required this.title,
    required this.count,
  });

  final String title;
  final int count;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Detail')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Title: $title'),
            Text('Count: $count'),
            ElevatedButton(
              onPressed: () {
                context.pop();
              },
              child: const Text('Go Back'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- extra: pass arbitrary data
- Path parameters: /user/:id
- Query parameters: ?name=John
- state.uri for URL parsing

---

# Real-World Examples

> **Common patterns** with Router API.

```dart
// 1. E-commerce app routing
class EcommerceRouter {
  static final router = GoRouter(
    initialLocation: '/home',
    routes: [
      // Main shell
      ShellRoute(
        builder: (context, state, child) {
          return ScaffoldWithNavBar(
            currentIndex: _getCurrentIndex(state),
            child: child,
          );
        },
        routes: [
          // Home tab
          GoRoute(
            path: '/home',
            builder: (context, state) => const HomeScreen(),
          ),
          // Search tab
          GoRoute(
            path: '/search',
            builder: (context, state) => const SearchScreen(),
          ),
          // Cart tab
          GoRoute(
            path: '/cart',
            builder: (context, state) => const CartScreen(),
          ),
          // Profile tab
          GoRoute(
            path: '/profile',
            builder: (context, state) => const ProfileScreen(),
          ),
        ],
      ),
      
      // Product details (full screen)
      GoRoute(
        path: '/product/:id',
        builder: (context, state) {
          final id = state.pathParameters['id'] ?? '0';
          return ProductDetailScreen(productId: int.parse(id));
        },
      ),
      
      // Checkout flow
      GoRoute(
        path: '/checkout',
        builder: (context, state) => const CheckoutScreen(),
        routes: [
          GoRoute(
            path: 'shipping',
            builder: (context, state) => const ShippingScreen(),
          ),
          GoRoute(
            path: 'payment',
            builder: (context, state) => const PaymentScreen(),
          ),
          GoRoute(
            path: 'confirmation',
            builder: (context, state) => const OrderConfirmationScreen(),
          ),
        ],
      ),
    ],
    errorBuilder: (context, state) => const NotFoundScreen(),
  );

  static int _getCurrentIndex(GoRouterState state) {
    final location = state.matchedLocation;
    if (location.contains('/home')) return 0;
    if (location.contains('/search')) return 1;
    if (location.contains('/cart')) return 2;
    if (location.contains('/profile')) return 3;
    return 0;
  }
}

// 2. Deep linking support
class DeepLinkingApp extends StatelessWidget {
  const DeepLinkingApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _deepLinkRouter,
    );
  }
}

final _deepLinkRouter = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return ProductDetailScreen(productId: int.parse(id));
      },
    ),
    GoRoute(
      path: '/user/:userId/post/:postId',
      builder: (context, state) {
        final userId = state.pathParameters['userId'] ?? '0';
        final postId = state.pathParameters['postId'] ?? '0';
        return PostScreen(
          userId: int.parse(userId),
          postId: int.parse(postId),
        );
      },
    ),
  ],
  // Handle web deep links
  urlPathStrategy: UrlPathStrategy.path,
);
```

What's happening here?
- ShellRoute with bottom nav
- Nested checkout flow
- Deep linking support
- Path parameter extraction

---

# Best Practices

## Use ShellRoute for Persistent Layout

```dart
// Good - ShellRoute for navigation
ShellRoute(
  builder: (context, state, child) {
    return PersistentLayout(child: child);
  },
  routes: [...],
)

// Bad - Redundant layouts
GoRoute(
  path: '/',
  builder: (context, state) => const PersistentLayout(child: HomeScreen()),
)
```

## Use Redirects for Auth

```dart
// Good - Auth redirect
redirect: (context, state) {
  final auth = Provider.of<AuthState>(context);
  if (!auth.isLoggedIn) return '/login';
  return null;
}
```

## Use Extra for Complex Data

```dart
// Good - Extra for complex data
context.push('/detail', extra: userData);

// Bad - Encoding in path
context.push('/detail/${userData.id}/${userData.name}');
```

---

# Common Mistakes

## Not Handling Errors

Wrong:
```dart
// No error handling
GoRouter(
  routes: [...],
  // No errorBuilder
)
```

Correct:
```dart
// With error handling
GoRouter(
  routes: [...],
  errorBuilder: (context, state) => const ErrorScreen(),
)
```

## Missing Redirects

Wrong:
```dart
// No auth redirect
GoRouter(
  routes: [
    GoRoute(path: '/login', ...),
    GoRoute(path: '/profile', ...), // Can access without auth
  ],
)
```

Correct:
```dart
// With auth redirect
GoRouter(
  redirect: (context, state) {
    if (!auth.isLoggedIn) return '/login';
    return null;
  },
  routes: [...],
)
```

---

# Summary

The Router API provides a modern, declarative approach to navigation in Flutter. GoRouter is the most popular implementation, offering features like nested routes, redirects, path parameters, and deep linking support. Use it for complex navigation flows and web applications.

---

# Next Steps

- [Deep Linking](deep-linking.md)
- [Route Guards](route-guards.md)
- [Navigation Patterns](navigation-patterns.md)

---

# Did You Know?

- Router API is declarative
- GoRouter supports nested routes
- Redirects control navigation flow
- ShellRoute provides persistent layouts
- Path parameters use :id syntax
- Extra data can be passed with navigation
- Deep linking is built-in
- Router API is the future of Flutter navigation