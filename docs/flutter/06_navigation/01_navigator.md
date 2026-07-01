# Navigator

Understand how to manage navigation between screens in Flutter.

---

# What is it?

Navigator is the widget that manages a stack of routes (pages) in Flutter. It works like a stack data structure where you push new screens onto the stack and pop them off when going back. Navigator provides methods like push, pop, and pushReplacement to control navigation flow and manage the navigation history.

---

# Why does it exist?

Navigator exists to:

- Manage navigation between screens
- Handle back navigation
- Maintain navigation history
- Support deep linking
- Enable passing data between screens
- Control navigation animations
- Manage app flow and user journeys

---

# Basic Navigation

> **Navigator** manages a stack of routes.

```dart
// Basic navigation example
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
            ElevatedButton(
              // Navigate to new screen
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const DetailScreen(),
                  ),
                );
              },
              child: const Text('Go to Detail'),
            ),
            ElevatedButton(
              // Navigate and replace
              onPressed: () {
                Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const ReplacementScreen(),
                  ),
                );
              },
              child: const Text('Replace Current'),
            ),
          ],
        ),
      ),
    );
  }
}

class DetailScreen extends StatelessWidget {
  const DetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Detail'),
        // Back button automatically added
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'Detail Screen',
              style: TextStyle(fontSize: 24),
            ),
            ElevatedButton(
              // Go back
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

// Navigator methods:
// 1. push() - Navigate to new screen
// 2. pop() - Return to previous screen
// 3. pushReplacement() - Replace current screen
// 4. pushAndRemoveUntil() - Navigate and remove previous screens
// 5. maybePop() - Pop if possible
// 6. canPop() - Check if can pop
```

What's happening here?
- push: adds new screen to stack
- pop: removes current screen
- pushReplacement: replaces current screen
- Stack manages navigation history

---

# Passing Data Between Screens

> **Passing data** with navigation.

```dart
// Passing data between screens
class User {
  final String name;
  final int age;
  final String email;

  User({
    required this.name,
    required this.age,
    required this.email,
  });
}

class DataHomeScreen extends StatelessWidget {
  const DataHomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Data Home')),
      body: Center(
        child: ElevatedButton(
          onPressed: () async {
            // Pass data to detail screen
            final result = await Navigator.push(
              context,
              MaterialPageRoute(
                builder: (context) => DataDetailScreen(
                  user: User(
                    name: 'John Doe',
                    age: 25,
                    email: 'john@example.com',
                  ),
                ),
              ),
            );
            
            // Handle returned data
            if (result != null) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text('Returned: $result')),
              );
            }
          },
          child: const Text('Open Detail'),
        ),
      ),
    );
  }
}

class DataDetailScreen extends StatelessWidget {
  const DataDetailScreen({super.key, required this.user});

  final User user;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('User Detail'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text('Name: ${user.name}'),
            Text('Age: ${user.age}'),
            Text('Email: ${user.email}'),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                // Return data to previous screen
                Navigator.pop(context, 'Updated successfully');
              },
              child: const Text('Save and Return'),
            ),
            ElevatedButton(
              onPressed: () {
                // Return with no data
                Navigator.pop(context);
              },
              child: const Text('Cancel'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Pass data in constructor
- Use async/await for results
- pop() returns data to previous screen
- Handle returned data with await

---

# Named Routes

> **Named routes** for organized navigation.

```dart
// Named routes configuration
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Named Routes',
      // Define routes
      initialRoute: '/',
      routes: {
        '/': (context) => const NamedHomeScreen(),
        '/detail': (context) => const NamedDetailScreen(),
        '/settings': (context) => const NamedSettingsScreen(),
        '/profile': (context) => const NamedProfileScreen(),
      },
      // Handle unknown routes
      onUnknownRoute: (settings) {
        return MaterialPageRoute(
          builder: (context) => const NotFoundScreen(),
        );
      },
    );
  }
}

class NamedHomeScreen extends StatelessWidget {
  const NamedHomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Navigate using named route
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/detail');
              },
              child: const Text('Go to Detail'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/settings');
              },
              child: const Text('Go to Settings'),
            ),
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/profile');
              },
              child: const Text('Go to Profile'),
            ),
          ],
        ),
      ),
    );
  }
}

class NamedDetailScreen extends StatelessWidget {
  const NamedDetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Detail'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.pop(context);
          },
          child: const Text('Go Back'),
        ),
      ),
    );
  }
}
```

What's happening here?
- routes map defines named routes
- pushNamed() uses route name
- initialRoute sets starting screen
- onUnknownRoute handles invalid routes

---

# Named Routes with Arguments

> **Passing arguments** to named routes.

```dart
// Named routes with arguments
class ArgHomeScreen extends StatelessWidget {
  const ArgHomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Arguments Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                // Pass arguments with named route
                Navigator.pushNamed(
                  context,
                  '/arg-detail',
                  arguments: {
                    'id': 123,
                    'name': 'Product Name',
                    'price': 99.99,
                  },
                );
              },
              child: const Text('Go with Arguments'),
            ),
          ],
        ),
      ),
    );
  }
}

class ArgDetailScreen extends StatelessWidget {
  const ArgDetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Extract arguments
    final args = ModalRoute.of(context)?.settings.arguments as Map<String, dynamic>;
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Detail with Args'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text('ID: ${args['id']}'),
            Text('Name: ${args['name']}'),
            Text('Price: \$${args['price']}'),
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

// Updating routes to handle arguments
class ArgApp extends StatelessWidget {
  const ArgApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Arguments Demo',
      initialRoute: '/',
      routes: {
        '/': (context) => const ArgHomeScreen(),
        '/arg-detail': (context) => const ArgDetailScreen(),
      },
      onGenerateRoute: (settings) {
        // Handle routes with arguments
        if (settings.name == '/arg-detail') {
          return MaterialPageRoute(
            builder: (context) => const ArgDetailScreen(),
            settings: settings,
          );
        }
        return null;
      },
    );
  }
}
```

What's happening here?
- arguments parameter passes data
- ModalRoute.of(context) extracts arguments
- onGenerateRoute for advanced handling
- Type-safe argument extraction

---

# Navigation Patterns

> **Common navigation patterns.**

```dart
// 1. Replace and clear stack (login flow)
class LoginFlow extends StatelessWidget {
  const LoginFlow({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                // Remove all previous routes and go to home
                Navigator.pushAndRemoveUntil(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const HomeScreen(),
                  ),
                  (route) => false, // Remove all routes
                );
              },
              child: const Text('Login'),
            ),
          ],
        ),
      ),
    );
  }
}

// 2. Dialog navigation
class DialogNavigation extends StatelessWidget {
  const DialogNavigation({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Dialog Demo')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            showDialog(
              context: context,
              barrierDismissible: true,
              builder: (context) {
                return AlertDialog(
                  title: const Text('Dialog Title'),
                  content: const Text('Dialog content goes here.'),
                  actions: [
                    TextButton(
                      onPressed: () {
                        Navigator.pop(context);
                      },
                      child: const Text('Cancel'),
                    ),
                    ElevatedButton(
                      onPressed: () {
                        Navigator.pop(context, 'Confirmed');
                      },
                      child: const Text('Confirm'),
                    ),
                  ],
                );
              },
            );
          },
          child: const Text('Show Dialog'),
        ),
      ),
    );
  }
}

// 3. Bottom sheet navigation
class BottomSheetNavigation extends StatelessWidget {
  const BottomSheetNavigation({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Bottom Sheet')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            showModalBottomSheet(
              context: context,
              builder: (context) {
                return Container(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      ListTile(
                        leading: const Icon(Icons.share),
                        title: const Text('Share'),
                        onTap: () {
                          Navigator.pop(context);
                        },
                      ),
                      ListTile(
                        leading: const Icon(Icons.copy),
                        title: const Text('Copy'),
                        onTap: () {
                          Navigator.pop(context);
                        },
                      ),
                    ],
                  ),
                );
              },
            );
          },
          child: const Text('Show Bottom Sheet'),
        ),
      ),
    );
  }
}

// 4. Deep linking navigation
class DeepLinkNavigation extends StatelessWidget {
  const DeepLinkNavigation({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Deep Link')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                // Navigate to specific screen with data
                Navigator.pushNamed(
                  context,
                  '/product/123',
                );
              },
              child: const Text('Open Product 123'),
            ),
          ],
        ),
      ),
    );
  }
}

// Advanced routing with parameters
class AdvancedApp extends StatelessWidget {
  const AdvancedApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Advanced Navigation',
      initialRoute: '/',
      onGenerateRoute: (settings) {
        // Parse route patterns like /product/123
        final uri = Uri.parse(settings.name ?? '');
        
        if (uri.pathSegments.length == 2 && uri.pathSegments[0] == 'product') {
          final id = uri.pathSegments[1];
          return MaterialPageRoute(
            builder: (context) => ProductDetailScreen(productId: id),
            settings: settings,
          );
        }
        
        // Default routes
        switch (settings.name) {
          case '/':
            return MaterialPageRoute(
              builder: (context) => const HomeScreen(),
            );
          case '/settings':
            return MaterialPageRoute(
              builder: (context) => const NamedSettingsScreen(),
            );
        }
        
        return null;
      },
    );
  }
}

class ProductDetailScreen extends StatelessWidget {
  const ProductDetailScreen({super.key, required this.productId});

  final String productId;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Product $productId'),
      ),
      body: Center(
        child: Text('Product ID: $productId'),
      ),
    );
  }
}
```

What's happening here?
- pushAndRemoveUntil: clear stack
- showDialog: modal dialogs
- showModalBottomSheet: bottom sheets
- Deep linking with route patterns
- onGenerateRoute for advanced routing

---

# Best Practices

## Use Named Routes for Complex Apps

```dart
// Good - Named routes for organization
MaterialApp(
  routes: {
    '/': (context) => const HomeScreen(),
    '/profile': (context) => const ProfileScreen(),
    '/settings': (context) => const SettingsScreen(),
  },
)
```

## Handle Return Data

```dart
// Good - Handle returned data
final result = await Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const DetailScreen()),
);

if (result != null) {
  // Handle result
}
```

## Use PushAndRemoveUntil for Login

```dart
// Good - Clear stack on login
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (context) => const HomeScreen()),
  (route) => false,
)
```

---

# Common Mistakes

## Not Handling Back Button

Wrong:
```dart
// User can go back to login after login
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const HomeScreen()),
)
```

Correct:
```dart
// Remove login from stack
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (context) => const HomeScreen()),
)
```

## Not Using await

Wrong:
```dart
// Missing await
Navigator.push(context, MaterialPageRoute(...));
// Data might not be handled
```

Correct:
```dart
// Use await for results
final result = await Navigator.push(...);
```

---

# Summary

Navigator manages screen navigation using a stack-based approach. Use push/pop for basic navigation, named routes for organization, and pass data between screens using constructor parameters or arguments. Handle navigation patterns like login flows, dialogs, and deep linking appropriately.

---

# Next Steps

- [Routes](routes.md)
- [Named Routes](named-routes.md)
- [Router API](router-api.md)

---

# Did You Know?

- Navigator works like a stack
- push adds screens, pop removes them
- Named routes help organize navigation
- Arguments pass data between screens
- pushAndRemoveUntil clears the stack
- showDialog displays modal dialogs
- showModalBottomSheet shows bottom sheets
- onGenerateRoute handles advanced routing