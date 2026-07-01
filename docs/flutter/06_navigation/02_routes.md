# Routes

Understand how routes define navigation destinations in Flutter.

---

# What is it?

Routes are the individual screens or pages that Navigator manages in a Flutter application. A route represents a single screen, dialog, or page that can be displayed to the user. Routes define what content is shown when a user navigates to a specific destination in your app.

---

# Why does it exist?

Routes exist to:

- Define navigation destinations
- Manage screen transitions
- Handle back navigation
- Pass data between screens
- Enable deep linking
- Support named navigation
- Control navigation flow

---

# Route Basics

> **Routes** represent individual screens.

```dart
// Basic route usage
class RouteBasics extends StatelessWidget {
  const RouteBasics({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Route Basics')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. MaterialPageRoute - Standard route
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const StandardScreen(),
                  ),
                );
              },
              child: const Text('MaterialPageRoute'),
            ),
            
            // 2. Custom route with settings
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const CustomScreen(),
                    settings: const RouteSettings(
                      name: '/custom',
                      arguments: {'data': 'Hello'},
                    ),
                  ),
                );
              },
              child: const Text('Route with Settings'),
            ),
          ],
        ),
      ),
    );
  }
}

class StandardScreen extends StatelessWidget {
  const StandardScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Standard Screen')),
      body: const Center(
        child: Text('This is a standard screen'),
      ),
    );
  }
}

// Route settings properties:
// 1. name - Route name for navigation
// 2. arguments - Data passed to route
// 3. isInitialRoute - Whether this is the first route
```

What's happening here?
- MaterialPageRoute: standard screen
- RouteSettings: route metadata
- builder: builds the screen
- Navigator.push: shows the route

---

# Route Types

> **Different types** of routes in Flutter.

```dart
// Different route types
class RouteTypes extends StatelessWidget {
  const RouteTypes({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Route Types')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. MaterialPageRoute - Material Design
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const Screen('Material'),
                  ),
                );
              },
              child: const Text('MaterialPageRoute'),
            ),
            
            // 2. CupertinoPageRoute - iOS style
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  CupertinoPageRoute(
                    builder: (context) => const Screen('Cupertino'),
                  ),
                );
              },
              child: const Text('CupertinoPageRoute'),
            ),
            
            // 3. PageRouteBuilder - Custom route
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  PageRouteBuilder(
                    pageBuilder: (context, animation, secondaryAnimation) {
                      return const Screen('Custom');
                    },
                    transitionsBuilder: (context, animation, secondaryAnimation, child) {
                      return FadeTransition(
                        opacity: animation,
                        child: child,
                      );
                    },
                    transitionDuration: const Duration(milliseconds: 500),
                  ),
                );
              },
              child: const Text('Custom Route'),
            ),
          ],
        ),
      ),
    );
  }
}

class Screen extends StatelessWidget {
  const Screen(this.title, {super.key});

  final String title;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('$title Screen'),
      ),
      body: Center(
        child: Text('This is a $title route'),
      ),
    );
  }
}
```

What's happening here?
- MaterialPageRoute: Material Design transitions
- CupertinoPageRoute: iOS-style transitions
- PageRouteBuilder: custom transitions
- Each route type has different behavior

---

# Custom Route Transitions

> **Custom animations** for routes.

```dart
// Custom route transitions
class CustomTransitions extends StatelessWidget {
  const CustomTransitions({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Custom Transitions')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Fade transition
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  PageRouteBuilder(
                    pageBuilder: (context, animation, secondaryAnimation) {
                      return const TargetScreen('Fade');
                    },
                    transitionsBuilder: (context, animation, secondaryAnimation, child) {
                      return FadeTransition(
                        opacity: animation,
                        child: child,
                      );
                    },
                  ),
                );
              },
              child: const Text('Fade Transition'),
            ),
            
            // 2. Slide transition
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  PageRouteBuilder(
                    pageBuilder: (context, animation, secondaryAnimation) {
                      return const TargetScreen('Slide');
                    },
                    transitionsBuilder: (context, animation, secondaryAnimation, child) {
                      const begin = Offset(1.0, 0.0);
                      const end = Offset.zero;
                      final tween = Tween(begin: begin, end: end);
                      final offsetAnimation = animation.drive(tween);
                      return SlideTransition(
                        position: offsetAnimation,
                        child: child,
                      );
                    },
                  ),
                );
              },
              child: const Text('Slide Transition'),
            ),
            
            // 3. Scale transition
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  PageRouteBuilder(
                    pageBuilder: (context, animation, secondaryAnimation) {
                      return const TargetScreen('Scale');
                    },
                    transitionsBuilder: (context, animation, secondaryAnimation, child) {
                      const begin = 0.0;
                      const end = 1.0;
                      final tween = Tween(begin: begin, end: end);
                      final scaleAnimation = animation.drive(tween);
                      return ScaleTransition(
                        scale: scaleAnimation,
                        child: child,
                      );
                    },
                  ),
                );
              },
              child: const Text('Scale Transition'),
            ),
            
            // 4. Combined transition
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  PageRouteBuilder(
                    pageBuilder: (context, animation, secondaryAnimation) {
                      return const TargetScreen('Combined');
                    },
                    transitionsBuilder: (context, animation, secondaryAnimation, child) {
                      return FadeTransition(
                        opacity: animation,
                        child: SlideTransition(
                          position: Tween<Offset>(
                            begin: const Offset(0.0, 0.5),
                            end: Offset.zero,
                          ).animate(animation),
                          child: child,
                        ),
                      );
                    },
                  ),
                );
              },
              child: const Text('Combined Transition'),
            ),
          ],
        ),
      ),
    );
  }
}

class TargetScreen extends StatelessWidget {
  const TargetScreen(this.transition, {super.key});

  final String transition;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('$transition Screen'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('$transition transition'),
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
- FadeTransition: fades in
- SlideTransition: slides in
- ScaleTransition: scales in
- Combined transitions: multiple effects

---

# Route with Arguments

> **Passing data** through routes.

```dart
// Route with arguments
class RouteWithArguments extends StatelessWidget {
  const RouteWithArguments({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Route Arguments')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const ArgumentScreen(),
                    settings: const RouteSettings(
                      arguments: {
                        'name': 'John Doe',
                        'age': 25,
                        'email': 'john@example.com',
                      },
                    ),
                  ),
                );
              },
              child: const Text('Pass Arguments'),
            ),
          ],
        ),
      ),
    );
  }
}

class ArgumentScreen extends StatelessWidget {
  const ArgumentScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Extract arguments from settings
    final args = ModalRoute.of(context)?.settings.arguments as Map<String, dynamic>?;
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Arguments Screen'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text('Name: ${args?['name'] ?? 'N/A'}'),
            Text('Age: ${args?['age'] ?? 'N/A'}'),
            Text('Email: ${args?['email'] ?? 'N/A'}'),
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

// Type-safe arguments with object
class UserArguments {
  final String name;
  final int age;
  final String email;

  UserArguments({
    required this.name,
    required this.age,
    required this.email,
  });
}

class TypeSafeRoute extends StatelessWidget {
  const TypeSafeRoute({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Type-Safe Arguments')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.push(
              context,
              MaterialPageRoute(
                builder: (context) => TypeSafeScreen(
                  args: UserArguments(
                    name: 'Alice',
                    age: 30,
                    email: 'alice@example.com',
                  ),
                ),
              ),
            );
          },
          child: const Text('Pass Type-Safe Args'),
        ),
      ),
    );
  }
}

class TypeSafeScreen extends StatelessWidget {
  const TypeSafeScreen({super.key, required this.args});

  final UserArguments args;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Type-Safe Screen'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text('Name: ${args.name}'),
            Text('Age: ${args.age}'),
            Text('Email: ${args.email}'),
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
- RouteSettings.arguments: pass data
- ModalRoute.of(): extract arguments
- Type-safe objects for arguments
- Constructor parameters for type safety

---

# Real-World Examples

> **Common patterns** using routes.

```dart
// 1. Login flow with routes
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
                // Replace login with home
                Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const HomeScreen(),
                  ),
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

// 2. Onboarding flow with pages
class OnboardingFlow extends StatefulWidget {
  const OnboardingFlow({super.key});

  @override
  State<OnboardingFlow> createState() => _OnboardingFlowState();
}

class _OnboardingFlowState extends State<OnboardingFlow> {
  int _currentPage = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: PageView(
        onPageChanged: (index) {
          setState(() {
            _currentPage = index;
          });
        },
        children: [
          _buildOnboardingPage('Welcome', 'Get started with our app'),
          _buildOnboardingPage('Features', 'Explore amazing features'),
          _buildOnboardingPage('Ready', 'Start using the app'),
        ],
      ),
      bottomNavigationBar: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          ...List.generate(3, (index) {
            return Container(
              width: 8,
              height: 8,
              margin: const EdgeInsets.all(4),
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                color: _currentPage == index ? Colors.blue : Colors.grey,
              ),
            );
          }),
          if (_currentPage == 2)
            ElevatedButton(
              onPressed: () {
                Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const HomeScreen(),
                  ),
                );
              },
              child: const Text('Get Started'),
            ),
        ],
      ),
    );
  }

  Widget _buildOnboardingPage(String title, String subtitle) {
    return Container(
      padding: const EdgeInsets.all(32),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.star, size: 80),
          const SizedBox(height: 32),
          Text(
            title,
            style: const TextStyle(
              fontSize: 32,
              fontWeight: FontWeight.bold,
            ),
          ),
          const SizedBox(height: 16),
          Text(
            subtitle,
            style: const TextStyle(fontSize: 18),
          ),
        ],
      ),
    );
  }
}

// 3. Profile edit flow
class ProfileEditFlow extends StatelessWidget {
  const ProfileEditFlow({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Profile'),
      ),
      body: ListView(
        children: [
          ListTile(
            title: const Text('Edit Profile'),
            trailing: const Icon(Icons.arrow_forward),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => const EditProfileScreen(),
                ),
              );
            },
          ),
          ListTile(
            title: const Text('Change Password'),
            trailing: const Icon(Icons.arrow_forward),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => const ChangePasswordScreen(),
                ),
              );
            },
          ),
          ListTile(
            title: const Text('Settings'),
            trailing: const Icon(Icons.arrow_forward),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => const SettingsScreen(),
                ),
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
- Login flow with route replacement
- Onboarding with PageView
- Profile edit flow with nested routes
- Real-world navigation patterns

---

# Best Practices

## Use Type-Safe Arguments

```dart
// Good - Type-safe arguments
class User {
  final String name;
  final int age;
  
  User({required this.name, required this.age});
}

Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => UserScreen(user: user),
  ),
);

// Bad - Using dynamic arguments
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => UserScreen(
      name: 'John',
      age: 25,
    ),
  ),
);
```

## Define Route Names in Constants

```dart
// Good - Route constants
class AppRoutes {
  static const String home = '/';
  static const String profile = '/profile';
  static const String settings = '/settings';
  static const String detail = '/detail';
}

// Usage
Navigator.pushNamed(context, AppRoutes.profile);
```

## Use RouteSettings for Navigation

```dart
// Good - Using RouteSettings
MaterialPageRoute(
  builder: (context) => const DetailScreen(),
  settings: const RouteSettings(
    name: '/detail',
    arguments: {'id': 123},
  ),
)
```

---

# Common Mistakes

## Not Using Settings for Arguments

Wrong:
```dart
// Arguments not accessible via settings
MaterialPageRoute(
  builder: (context) => DetailScreen(
    data: someData,
  ),
)
```

Correct:
```dart
// Arguments in settings
MaterialPageRoute(
  builder: (context) => const DetailScreen(),
  settings: RouteSettings(arguments: someData),
)
```

## Overcomplicating Routes

Wrong:
```dart
// Too complex for simple route
PageRouteBuilder(
  pageBuilder: (...) => SimpleScreen(),
  transitionDuration: ...,
  transitionsBuilder: ...,
  // Overkill for simple screen
)
```

Correct:
```dart
// Simple route
MaterialPageRoute(
  builder: (context) => const SimpleScreen(),
)
```

---

# Summary

Routes represent individual screens in Flutter applications. Use MaterialPageRoute for standard Material transitions, CupertinoPageRoute for iOS style, and PageRouteBuilder for custom transitions. Pass data through RouteSettings arguments or constructor parameters.

---

# Next Steps

- [Named Routes](named-routes.md)
- [Router API](router-api.md)
- [Deep Linking](deep-linking.md)

---

# Did You Know?

- Routes are managed by Navigator
- MaterialPageRoute provides Material transitions
- CupertinoPageRoute provides iOS transitions
- PageRouteBuilder enables custom transitions
- RouteSettings holds route metadata
- ModalRoute extracts route information
- Routes can have custom animations
- Routes can be nested for complex flows