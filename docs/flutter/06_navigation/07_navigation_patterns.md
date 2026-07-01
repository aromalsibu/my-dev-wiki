# Navigation Patterns

Understand common navigation patterns and best practices in Flutter.

---

# What is it?

Navigation patterns are proven strategies for managing screen transitions, user flows, and navigation behavior in Flutter applications. They cover everything from basic push/pop navigation to complex flows like authentication, onboarding, tab navigation, and deep linking. Understanding these patterns helps you design intuitive and maintainable navigation structures.

---

# Why does it exist?

Navigation patterns exist to:

- Provide consistent user experiences
- Handle common navigation scenarios
- Manage navigation state effectively
- Support different app architectures
- Improve code organization
- Enable complex navigation flows
- Follow platform conventions

---

# Authentication Flow

> **Managing login and protected routes.**

```dart
// Authentication flow pattern
class AuthFlowApp extends StatelessWidget {
  const AuthFlowApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => AuthProvider(),
      child: MaterialApp(
        title: 'Auth Flow',
        theme: ThemeData(primarySwatch: Colors.blue),
        home: const AuthWrapper(),
      ),
    );
  }
}

class AuthProvider extends ChangeNotifier {
  bool _isAuthenticated = false;
  bool _isLoading = true;

  bool get isAuthenticated => _isAuthenticated;
  bool get isLoading => _isLoading;

  AuthProvider() {
    _checkAuthStatus();
  }

  Future<void> _checkAuthStatus() async {
    await Future.delayed(const Duration(seconds: 1));
    _isLoading = false;
    notifyListeners();
  }

  Future<void> login(String email, String password) async {
    await Future.delayed(const Duration(seconds: 1));
    _isAuthenticated = true;
    notifyListeners();
  }

  Future<void> logout() async {
    await Future.delayed(const Duration(milliseconds: 500));
    _isAuthenticated = false;
    notifyListeners();
  }
}

class AuthWrapper extends StatelessWidget {
  const AuthWrapper({super.key});

  @override
  Widget build(BuildContext context) {
    final auth = Provider.of<AuthProvider>(context);

    if (auth.isLoading) {
      return const SplashScreen();
    }

    if (auth.isAuthenticated) {
      return const HomeScreen();
    }

    return const LoginScreen();
  }
}

class LoginScreen extends StatelessWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            context.read<AuthProvider>().login('test@email.com', 'password');
          },
          child: const Text('Login'),
        ),
      ),
    );
  }
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            context.read<AuthProvider>().logout();
          },
          child: const Text('Logout'),
        ),
      ),
    );
  }
}
```

What's happening here?
- AuthProvider manages auth state
- AuthWrapper decides initial screen
- Login screen sets authenticated
- Logout clears auth state

---

# Onboarding Flow

> **Guiding new users** through app introduction.

```dart
// Onboarding flow pattern
class OnboardingFlowApp extends StatelessWidget {
  const OnboardingFlowApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => OnboardingProvider(),
      child: MaterialApp(
        title: 'Onboarding Flow',
        theme: ThemeData(primarySwatch: Colors.blue),
        home: const OnboardingWrapper(),
      ),
    );
  }
}

class OnboardingProvider extends ChangeNotifier {
  bool _hasCompletedOnboarding = false;

  bool get hasCompletedOnboarding => _hasCompletedOnboarding;

  void completeOnboarding() {
    _hasCompletedOnboarding = true;
    notifyListeners();
  }

  void resetOnboarding() {
    _hasCompletedOnboarding = false;
    notifyListeners();
  }
}

class OnboardingWrapper extends StatelessWidget {
  const OnboardingWrapper({super.key});

  @override
  Widget build(BuildContext context) {
    final onboarding = Provider.of<OnboardingProvider>(context);

    if (onboarding.hasCompletedOnboarding) {
      return const HomeScreen();
    }

    return const OnboardingScreen();
  }
}

class OnboardingScreen extends StatefulWidget {
  const OnboardingScreen({super.key});

  @override
  State<OnboardingScreen> createState() => _OnboardingScreenState();
}

class _OnboardingScreenState extends State<OnboardingScreen> {
  final PageController _controller = PageController();
  int _currentPage = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: PageView(
        controller: _controller,
        onPageChanged: (index) {
          setState(() {
            _currentPage = index;
          });
        },
        children: [
          _buildPage('Welcome', 'Get started with our amazing app'),
          _buildPage('Features', 'Explore all the features we offer'),
          _buildPage('Ready', 'You\'re ready to begin!'),
        ],
      ),
      bottomSheet: Container(
        padding: const EdgeInsets.all(16),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            TextButton(
              onPressed: () {
                context.read<OnboardingProvider>().completeOnboarding();
              },
              child: const Text('Skip'),
            ),
            Row(
              children: List.generate(3, (index) {
                return Container(
                  width: 8,
                  height: 8,
                  margin: const EdgeInsets.symmetric(horizontal: 4),
                  decoration: BoxDecoration(
                    shape: BoxShape.circle,
                    color: _currentPage == index ? Colors.blue : Colors.grey,
                  ),
                );
              }),
            ),
            ElevatedButton(
              onPressed: () {
                if (_currentPage == 2) {
                  context.read<OnboardingProvider>().completeOnboarding();
                } else {
                  _controller.nextPage(
                    duration: const Duration(milliseconds: 300),
                    curve: Curves.easeInOut,
                  );
                }
              },
              child: Text(_currentPage == 2 ? 'Get Started' : 'Next'),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildPage(String title, String subtitle) {
    return Container(
      padding: const EdgeInsets.all(32),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.star, size: 80),
          const SizedBox(height: 32),
          Text(
            title,
            style: const TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 16),
          Text(
            subtitle,
            textAlign: TextAlign.center,
            style: const TextStyle(fontSize: 18),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- OnboardingProvider tracks completion
- OnboardingWrapper decides screen
- PageView for onboarding pages
- Skip or complete onboarding

---

# Tab Navigation

> **Bottom navigation and tabs.**

```dart
// Tab navigation pattern
class TabNavigationApp extends StatefulWidget {
  const TabNavigationApp({super.key});

  @override
  State<TabNavigationApp> createState() => _TabNavigationAppState();
}

class _TabNavigationAppState extends State<TabNavigationApp> {
  int _currentIndex = 0;

  final List<Widget> _screens = [
    const HomeTab(),
    const SearchTab(),
    const ProfileTab(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _screens[_currentIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) {
          setState(() {
            _currentIndex = index;
          });
        },
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'Home',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.search),
            label: 'Search',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person),
            label: 'Profile',
          ),
        ],
      ),
    );
  }
}

// Tab navigation with Navigator (preserves state)
class TabWithNavigatorApp extends StatelessWidget {
  const TabWithNavigatorApp({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          bottom: const TabBar(
            tabs: [
              Tab(icon: Icon(Icons.home), text: 'Home'),
              Tab(icon: Icon(Icons.search), text: 'Search'),
              Tab(icon: Icon(Icons.person), text: 'Profile'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            HomeTab(),
            SearchTab(),
            ProfileTab(),
          ],
        ),
      ),
    );
  }
}

// Nested tab navigation (tabs inside tabs)
class NestedTabNavigation extends StatelessWidget {
  const NestedTabNavigation({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Nested Tabs')),
      body: DefaultTabController(
        length: 2,
        child: Column(
          children: [
            const TabBar(
              tabs: [
                Tab(text: 'Posts'),
                Tab(text: 'Videos'),
              ],
            ),
            Expanded(
              child: TabBarView(
                children: [
                  _buildPostTab(),
                  _buildVideoTab(),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildPostTab() {
    return DefaultTabController(
      length: 3,
      child: Column(
        children: [
          const TabBar(
            tabs: [
              Tab(text: 'All'),
              Tab(text: 'Recent'),
              Tab(text: 'Popular'),
            ],
          ),
          Expanded(
            child: TabBarView(
              children: [
                ListView.builder(
                  itemCount: 10,
                  itemBuilder: (context, index) {
                    return ListTile(title: Text('All Post $index'));
                  },
                ),
                ListView.builder(
                  itemCount: 10,
                  itemBuilder: (context, index) {
                    return ListTile(title: Text('Recent Post $index'));
                  },
                ),
                ListView.builder(
                  itemCount: 10,
                  itemBuilder: (context, index) {
                    return ListTile(title: Text('Popular Post $index'));
                  },
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildVideoTab() {
    return ListView.builder(
      itemCount: 20,
      itemBuilder: (context, index) {
        return ListTile(
          leading: const Icon(Icons.video_library),
          title: Text('Video $index'),
        );
      },
    );
  }
}
```

What's happening here?
- BottomNavigationBar for main tabs
- TabBar with TabBarView for tabs
- DefaultTabController manages state
- Nested tabs for complex layouts

---

# Modal Navigation

> **Dialogs, bottom sheets, and overlays.**

```dart
// Modal navigation patterns
class ModalNavigation extends StatelessWidget {
  const ModalNavigation({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Modal Navigation')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Alert Dialog
            ElevatedButton(
              onPressed: () {
                showDialog(
                  context: context,
                  builder: (context) => AlertDialog(
                    title: const Text('Alert Dialog'),
                    content: const Text('This is an alert dialog'),
                    actions: [
                      TextButton(
                        onPressed: () => Navigator.pop(context),
                        child: const Text('Cancel'),
                      ),
                      ElevatedButton(
                        onPressed: () => Navigator.pop(context),
                        child: const Text('Confirm'),
                      ),
                    ],
                  ),
                );
              },
              child: const Text('Alert Dialog'),
            ),
            
            // 2. Custom Dialog
            ElevatedButton(
              onPressed: () {
                showDialog(
                  context: context,
                  builder: (context) => Dialog(
                    child: Container(
                      padding: const EdgeInsets.all(16),
                      child: Column(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          const Icon(Icons.star, size: 64),
                          const SizedBox(height: 16),
                          const Text('Custom Dialog'),
                          const SizedBox(height: 16),
                          ElevatedButton(
                            onPressed: () => Navigator.pop(context),
                            child: const Text('Close'),
                          ),
                        ],
                      ),
                    ),
                  ),
                );
              },
              child: const Text('Custom Dialog'),
            ),
            
            // 3. Bottom Sheet
            ElevatedButton(
              onPressed: () {
                showModalBottomSheet(
                  context: context,
                  shape: const RoundedRectangleBorder(
                    borderRadius: BorderRadius.vertical(
                      top: Radius.circular(16),
                    ),
                  ),
                  builder: (context) => Container(
                    padding: const EdgeInsets.all(16),
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        const SizedBox(height: 8),
                        Container(
                          width: 40,
                          height: 4,
                          decoration: BoxDecoration(
                            color: Colors.grey[300],
                            borderRadius: BorderRadius.circular(2),
                          ),
                        ),
                        const SizedBox(height: 16),
                        ListTile(
                          leading: const Icon(Icons.share),
                          title: const Text('Share'),
                          onTap: () => Navigator.pop(context),
                        ),
                        ListTile(
                          leading: const Icon(Icons.copy),
                          title: const Text('Copy'),
                          onTap: () => Navigator.pop(context),
                        ),
                        ListTile(
                          leading: const Icon(Icons.delete),
                          title: const Text('Delete'),
                          onTap: () => Navigator.pop(context),
                        ),
                      ],
                    ),
                  ),
                );
              },
              child: const Text('Bottom Sheet'),
            ),
            
            // 4. Full-screen dialog
            ElevatedButton(
              onPressed: () {
                showDialog(
                  context: context,
                  barrierDismissible: false,
                  builder: (context) => const FullScreenDialog(),
                );
              },
              child: const Text('Full Screen Dialog'),
            ),
          ],
        ),
      ),
    );
  }
}

class FullScreenDialog extends StatelessWidget {
  const FullScreenDialog({super.key});

  @override
  Widget build(BuildContext context) {
    return Dialog(
      insetPadding: EdgeInsets.zero,
      child: Container(
        color: Colors.white,
        child: Column(
          children: [
            AppBar(
              title: const Text('Full Screen Dialog'),
              leading: IconButton(
                icon: const Icon(Icons.close),
                onPressed: () => Navigator.pop(context),
              ),
            ),
            Expanded(
              child: Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    const Text('Full Screen Content'),
                    const SizedBox(height: 16),
                    ElevatedButton(
                      onPressed: () => Navigator.pop(context),
                      child: const Text('Close'),
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- AlertDialog for confirmations
- Custom Dialog for content
- ModalBottomSheet for options
- Full screen dialog for complex content

---

# Real-World Examples

> **Common navigation patterns in real apps.**

```dart
// 1. E-commerce navigation flow
class EcommerceNavigation extends StatefulWidget {
  const EcommerceNavigation({super.key});

  @override
  State<EcommerceNavigation> createState() => _EcommerceNavigationState();
}

class _EcommerceNavigationState extends State<EcommerceNavigation> {
  int _currentIndex = 0;

  final List<Widget> _tabs = [
    const HomeTab(),
    const SearchTab(),
    const CartTab(),
    const ProfileTab(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _tabs[_currentIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) {
          setState(() {
            _currentIndex = index;
          });
        },
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(
            icon: Icon(Icons.shopping_cart),
            label: 'Cart',
          ),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}

// 2. Form wizard navigation
class FormWizard extends StatefulWidget {
  const FormWizard({super.key});

  @override
  State<FormWizard> createState() => _FormWizardState();
}

class _FormWizardState extends State<FormWizard> {
  int _currentStep = 0;

  final List<Widget> _steps = [
    const Step1Screen(),
    const Step2Screen(),
    const Step3Screen(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Form Wizard'),
      ),
      body: Column(
        children: [
          LinearProgressIndicator(
            value: (_currentStep + 1) / _steps.length,
            backgroundColor: Colors.grey[200],
          ),
          Expanded(
            child: _steps[_currentStep],
          ),
          Container(
            padding: const EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                if (_currentStep > 0)
                  ElevatedButton(
                    onPressed: () {
                      setState(() {
                        _currentStep--;
                      });
                    },
                    child: const Text('Back'),
                  )
                else
                  const SizedBox.shrink(),
                ElevatedButton(
                  onPressed: () {
                    if (_currentStep == _steps.length - 1) {
                      // Submit form
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(content: Text('Form submitted!')),
                      );
                      Navigator.pop(context);
                    } else {
                      setState(() {
                        _currentStep++;
                      });
                    }
                  },
                  child: Text(
                    _currentStep == _steps.length - 1 ? 'Submit' : 'Next',
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// 3. Master-detail navigation (tablet)
class MasterDetailNavigation extends StatelessWidget {
  const MasterDetailNavigation({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        if (constraints.maxWidth > 600) {
          // Tablet: Master-Detail layout
          return const Row(
            children: [
              Expanded(
                flex: 1,
                child: MasterList(),
              ),
              VerticalDivider(),
              Expanded(
                flex: 2,
                child: DetailView(),
              ),
            ],
          );
        } else {
          // Phone: Navigation stack
          return const MasterList();
        }
      },
    );
  }
}

class MasterList extends StatelessWidget {
  const MasterList({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Items')),
      body: ListView.builder(
        itemCount: 20,
        itemBuilder: (context, index) {
          return ListTile(
            title: Text('Item $index'),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => DetailView(),
                ),
              );
            },
          );
        },
      ),
    );
  }
}

class DetailView extends StatelessWidget {
  const DetailView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Detail')),
      body: const Center(child: Text('Item Details')),
    );
  }
}
```

What's happening here?
- E-commerce bottom navigation
- Form wizard with steps
- Master-detail for tablets
- Responsive navigation patterns

---

# Best Practices

## Manage Navigation State

```dart
// Good - Centralized navigation
class NavigationService {
  final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();

  void push(Widget screen) {
    navigatorKey.currentState?.push(
      MaterialPageRoute(builder: (_) => screen),
    );
  }

  void pop() {
    navigatorKey.currentState?.pop();
  }
}
```

## Use Named Routes for Complex Apps

```dart
// Good - Named routes
MaterialApp(
  routes: {
    '/': (context) => const HomeScreen(),
    '/profile': (context) => const ProfileScreen(),
    '/settings': (context) => const SettingsScreen(),
  },
)
```

## Handle Back Navigation

```dart
// Good - Custom back navigation
WillPopScope(
  onWillPop: () async {
    if (hasUnsavedChanges) {
      return await showDialog(
        context: context,
        builder: (context) => AlertDialog(
          title: const Text('Discard changes?'),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context, false),
              child: const Text('No'),
            ),
            ElevatedButton(
              onPressed: () => Navigator.pop(context, true),
              child: const Text('Yes'),
            ),
          ],
        ),
      ) ?? false;
    }
    return true;
  },
  child: const Screen(),
)
```

---

# Summary

Navigation patterns provide proven strategies for managing app navigation. Use authentication flows for protected routes, onboarding for new users, tab navigation for main sections, and modals for temporary content. Choose patterns based on your app's needs and complexity.

---

# Next Steps

- [Local State](../state-management/local-state.md)
- [InheritedWidget](../state-management/inheritedwidget.md)
- [Provider](../state-management/provider.md)

---

# Did You Know?

- Authentication flow uses wrappers
- Onboarding uses PageView
- Tab navigation preserves state
- Modal dialogs are transient
- Form wizards use step navigation
- Master-detail adapts to screen size
- Navigation patterns improve UX
- Patterns can be combined