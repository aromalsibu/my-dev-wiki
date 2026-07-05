# Named Routes

Understand how to organize navigation with named routes in Flutter.

---

# What is it?

Named Routes are a way to define and navigate to screens using string identifiers instead of building routes directly. They provide a centralized way to manage navigation, making your code more organized, maintainable, and easier to handle deep linking and complex navigation flows.

---

# Why does it exist?

Named Routes exist to:

- Centralize route definitions
- Simplify navigation code
- Support deep linking
- Enable dynamic route generation
- Organize app navigation
- Handle unknown routes
- Pass data with navigation

---

# Defining Named Routes

> **Named routes** are defined in MaterialApp.

```dart
// Defining named routes
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Named Routes',
      // 1. Define initial route
      initialRoute: '/',
      
      // 2. Define routes map
      routes: {
        '/': (context) => const HomeScreen(),
        '/profile': (context) => const ProfileScreen(),
        '/settings': (context) => const SettingsScreen(),
        '/about': (context) => const AboutScreen(),
      },
      
      // 3. Handle unknown routes
      onUnknownRoute: (settings) {
        return MaterialPageRoute(
          builder: (context) => const NotFoundScreen(),
        );
      },
    );
  }
}

// Using named routes
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
            // Navigate using pushNamed
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/profile');
              },
              child: const Text('Go to Profile'),
            ),
            
            // Navigate and replace
            ElevatedButton(
              onPressed: () {
                Navigator.pushReplacementNamed(context, '/settings');
              },
              child: const Text('Replace with Settings'),
            ),
            
            // Navigate and remove all
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamedAndRemoveUntil(
                  context,
                  '/about',
                  (route) => false,
                );
              },
              child: const Text('Go to About (Clear Stack)'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- routes map defines named routes
- initialRoute sets starting screen
- pushNamed navigates to named route
- pushReplacementNamed replaces current
- pushNamedAndRemoveUntil clears stack

---

# Named Routes with Arguments

> **Passing data** to named routes.

```dart
// Named routes with arguments
class NamedRoutesWithArgs extends StatelessWidget {
  const NamedRoutesWithArgs({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Named Routes with Args',
      initialRoute: '/',
      routes: {
        '/': (context) => const NamedHome(),
        '/user': (context) => const UserDetailScreen(),
        '/product': (context) => const ProductDetailScreen(),
      },
      // Handle routes with arguments
      onGenerateRoute: (settings) {
        if (settings.name == '/user') {
          final args = settings.arguments as Map<String, dynamic>?;
          return MaterialPageRoute(
            builder: (context) => UserDetailScreen(
              userId: args?['id'] ?? 0,
              userName: args?['name'] ?? 'Unknown',
            ),
          );
        }
        if (settings.name == '/product') {
          final args = settings.arguments as ProductArgs?;
          return MaterialPageRoute(
            builder: (context) => ProductDetailScreen(
              product: args!,
            ),
          );
        }
        return null;
      },
    );
  }
}

class NamedHome extends StatelessWidget {
  const NamedHome({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Pass arguments as map
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(
                  context,
                  '/user',
                  arguments: {
                    'id': 123,
                    'name': 'John Doe',
                  },
                );
              },
              child: const Text('Go to User (Map)'),
            ),
            
            // Pass arguments as object
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(
                  context,
                  '/product',
                  arguments: ProductArgs(
                    id: 456,
                    name: 'Laptop',
                    price: 999.99,
                  ),
                );
              },
              child: const Text('Go to Product (Object)'),
            ),
          ],
        ),
      ),
    );
  }
}

class UserDetailScreen extends StatelessWidget {
  const UserDetailScreen({
    super.key,
    required this.userId,
    required this.userName,
  });

  final int userId;
  final String userName;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('User $userId')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text('ID: $userId'),
            Text('Name: $userName'),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context);
              },
              child: const Text('Go Back'),
            ),
          ],
        ),
      ),
    );
  }
}

// Type-safe arguments object
class ProductArgs {
  final int id;
  final String name;
  final double price;

  ProductArgs({
    required this.id,
    required this.name,
    required this.price,
  });
}

class ProductDetailScreen extends StatelessWidget {
  const ProductDetailScreen({
    super.key,
    required this.product,
  });

  final ProductArgs product;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Product ${product.id}')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text('ID: ${product.id}'),
            Text('Name: ${product.name}'),
            Text('Price: \$${product.price}'),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context);
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
- arguments parameter passes data
- onGenerateRoute extracts arguments
- Type-safe arguments objects
- Map for dynamic arguments

---

# onGenerateRoute

> **Advanced route generation** with onGenerateRoute.

```dart
// Using onGenerateRoute for dynamic routing
class DynamicRoutesApp extends StatelessWidget {
  const DynamicRoutesApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Dynamic Routes',
      initialRoute: '/',
      onGenerateRoute: (settings) {
        // Parse route name
        final uri = Uri.parse(settings.name ?? '');
        final pathSegments = uri.pathSegments;
        
        // Handle dynamic routes
        if (pathSegments.isNotEmpty) {
          // /user/123
          if (pathSegments[0] == 'user' && pathSegments.length == 2) {
            final id = int.tryParse(pathSegments[1]) ?? 0;
            return MaterialPageRoute(
              builder: (context) => DynamicUserScreen(userId: id),
              settings: settings,
            );
          }
          
          // /product/123/detail
          if (pathSegments[0] == 'product' && pathSegments.length == 3) {
            final id = int.tryParse(pathSegments[1]) ?? 0;
            final action = pathSegments[2];
            return MaterialPageRoute(
              builder: (context) => DynamicProductScreen(
                productId: id,
                action: action,
              ),
              settings: settings,
            );
          }
        }
        
        // Static routes
        switch (settings.name) {
          case '/':
            return MaterialPageRoute(
              builder: (context) => const DynamicHomeScreen(),
            );
          case '/about':
            return MaterialPageRoute(
              builder: (context) => const AboutScreen(),
            );
        }
        
        // Unknown route
        return MaterialPageRoute(
          builder: (context) => const NotFoundScreen(),
        );
      },
      onUnknownRoute: (settings) {
        return MaterialPageRoute(
          builder: (context) => const NotFoundScreen(),
        );
      },
    );
  }
}

class DynamicHomeScreen extends StatelessWidget {
  const DynamicHomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Dynamic Routes')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                // Navigate to /user/123
                Navigator.pushNamed(context, '/user/123');
              },
              child: const Text('Go to User 123'),
            ),
            ElevatedButton(
              onPressed: () {
                // Navigate to /product/456/detail
                Navigator.pushNamed(context, '/product/456/detail');
              },
              child: const Text('Go to Product 456 Detail'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/about');
              },
              child: const Text('About'),
            ),
          ],
        ),
      ),
    );
  }
}

class DynamicUserScreen extends StatelessWidget {
  const DynamicUserScreen({super.key, required this.userId});

  final int userId;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('User $userId')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('User ID: $userId'),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context);
              },
              child: const Text('Go Back'),
            ),
          ],
        ),
      ),
    );
  }
}

class DynamicProductScreen extends StatelessWidget {
  const DynamicProductScreen({
    super.key,
    required this.productId,
    required this.action,
  });

  final int productId;
  final String action;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Product $productId')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Product ID: $productId'),
            Text('Action: $action'),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context);
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
- Parse URI for dynamic paths
- Extract parameters from path
- Handle complex routing patterns
- Support deep linking

---

# Navigation with Named Routes

> **Advanced navigation** using named routes.

```dart
// Advanced navigation with named routes
class AdvancedNamedNavigation extends StatelessWidget {
  const AdvancedNamedNavigation({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Advanced Navigation')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Push named route
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/next');
              },
              child: const Text('Push Named'),
            ),
            
            // 2. Push named and remove all
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamedAndRemoveUntil(
                  context,
                  '/home',
                  (route) => false,
                );
              },
              child: const Text('Push Named and Remove All'),
            ),
            
            // 3. Push named with predicate
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamedAndRemoveUntil(
                  context,
                  '/profile',
                  (route) => route.settings.name == '/home',
                );
              },
              child: const Text('Push Named (Keep Home)'),
            ),
            
            // 4. Pop until named route
            ElevatedButton(
              onPressed: () {
                Navigator.popUntil(
                  context,
                  (route) => route.settings.name == '/home',
                );
              },
              child: const Text('Pop Until Home'),
            ),
            
            // 5. Check if can pop
            ElevatedButton(
              onPressed: () {
                if (Navigator.canPop(context)) {
                  Navigator.pop(context);
                }
              },
              child: const Text('Pop if Possible'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- pushNamedAndRemoveUntil: clear stack
- popUntil: pop to specific route
- canPop: check if can pop
- Route settings for identification

---

# Real-World Examples

> **Common patterns** with named routes.

```dart
// 1. Route constants and organization
class AppRoutes {
  static const String home = '/';
  static const String login = '/login';
  static const String register = '/register';
  static const String forgotPassword = '/forgot-password';
  static const String profile = '/profile';
  static const String settings = '/settings';
  static const String product = '/product';
  static const String cart = '/cart';
  static const String checkout = '/checkout';
  static const String orderConfirmation = '/order-confirmation';
  
  // Dynamic routes
  static String productDetail(int id) => '/product/$id';
  static String userProfile(int id) => '/user/$id';
  static String orderDetails(int id) => '/order/$id';
}

// 2. App with organized routes
class OrganizedApp extends StatelessWidget {
  const OrganizedApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Organized Routes',
      initialRoute: AppRoutes.home,
      onGenerateRoute: (settings) {
        switch (settings.name) {
          case AppRoutes.home:
            return MaterialPageRoute(builder: (_) => const HomeScreen());
          case AppRoutes.login:
            return MaterialPageRoute(builder: (_) => const LoginScreen());
          case AppRoutes.register:
            return MaterialPageRoute(builder: (_) => const RegisterScreen());
          case AppRoutes.profile:
            return MaterialPageRoute(builder: (_) => const ProfileScreen());
          case AppRoutes.settings:
            return MaterialPageRoute(builder: (_) => const SettingsScreen());
          case AppRoutes.cart:
            return MaterialPageRoute(builder: (_) => const CartScreen());
          case AppRoutes.checkout:
            return MaterialPageRoute(builder: (_) => const CheckoutScreen());
        }
        
        // Dynamic routes
        final uri = Uri.parse(settings.name ?? '');
        if (uri.pathSegments.length == 2) {
          if (uri.pathSegments[0] == 'product') {
            final id = int.tryParse(uri.pathSegments[1]) ?? 0;
            return MaterialPageRoute(
              builder: (_) => ProductDetailScreen(productId: id),
            );
          }
          if (uri.pathSegments[0] == 'user') {
            final id = int.tryParse(uri.pathSegments[1]) ?? 0;
            return MaterialPageRoute(
              builder: (_) => UserProfileScreen(userId: id),
            );
          }
        }
        
        return MaterialPageRoute(builder: (_) => const NotFoundScreen());
      },
    );
  }
}

// 3. Using route constants
class RouteUsageExample extends StatelessWidget {
  const RouteUsageExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Route Usage')),
      body: ListView(
        children: [
          ListTile(
            title: const Text('Home'),
            onTap: () {
              Navigator.pushNamed(context, AppRoutes.home);
            },
          ),
          ListTile(
            title: const Text('Profile'),
            onTap: () {
              Navigator.pushNamed(context, AppRoutes.profile);
            },
          ),
          ListTile(
            title: const Text('Settings'),
            onTap: () {
              Navigator.pushNamed(context, AppRoutes.settings);
            },
          ),
          ListTile(
            title: const Text('Product 123'),
            onTap: () {
              Navigator.pushNamed(
                context,
                AppRoutes.productDetail(123),
              );
            },
          ),
          ListTile(
            title: const Text('User 456'),
            onTap: () {
              Navigator.pushNamed(
                context,
                AppRoutes.userProfile(456),
              );
            },
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Route constants for organization
- Dynamic route generation
- Centralized route management
- Type-safe navigation

---

# Best Practices

## Use Route Constants

```dart
// Good - Centralized route names
class Routes {
  static const String home = '/';
  static const String profile = '/profile';
  static const String settings = '/settings';
}

// Bad - Hardcoded strings
Navigator.pushNamed(context, '/profile'); // Hardcoded
```

## Organize Route Definitions

```dart
// Good - Centralized route generation
MaterialApp(
  onGenerateRoute: RouteGenerator.generateRoute,
)

class RouteGenerator {
  static Route<dynamic> generateRoute(RouteSettings settings) {
    // All route logic here
  }
}
```

## Use Type-Safe Navigation

```dart
// Good - Type-safe navigation
class NavigationService {
  void navigateToUser(int userId) {
    Navigator.pushNamed(
      context,
      Routes.user,
      arguments: UserArgs(id: userId),
    );
  }
}

// Bad - Unstructured navigation
Navigator.pushNamed(
  context,
  '/user',
  arguments: userId, // Unclear what's expected
)
```

---

# Common Mistakes

## Duplicate Route Names

Wrong:
```dart
// Duplicate definitions
routes: {
  '/profile': (context) => const ProfileScreen(),
  '/profile': (context) => const AnotherScreen(), // Duplicate
}
```

Correct:
```dart
// Unique routes
routes: {
  '/profile': (context) => const ProfileScreen(),
  '/settings': (context) => const SettingsScreen(),
}
```

## Missing Route Handler

Wrong:
```dart
// Route not defined
Navigator.pushNamed(context, '/missing'); // Error
```

Correct:
```dart
// Handle unknown routes
onUnknownRoute: (settings) {
  return MaterialPageRoute(
    builder: (context) => const NotFoundScreen(),
  );
},
```

---

# Summary

Named routes provide centralized navigation management using string identifiers. Define routes in MaterialApp, pass arguments with navigation, and use onGenerateRoute for dynamic routing. Organize route names in constants for maintainability.

---

# Next Steps

- [Router API](router-api.md)
- [Deep Linking](deep-linking.md)
- [Route Guards](route-guards.md)

---

# Did You Know?

- Named routes are defined in MaterialApp
- pushNamed navigates to named routes
- arguments pass data to routes
- onGenerateRoute handles dynamic routes
- Route constants improve maintainability
- onUnknownRoute handles missing routes
- Named routes support deep linking
- Routes can be nested for complex flows