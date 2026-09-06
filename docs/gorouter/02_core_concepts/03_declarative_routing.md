# Declarative Routing

## Overview

**Declarative routing** is a paradigm where you define **what** your app's navigation should look like as a function of state, rather than issuing **imperative** commands (push, pop) to mutate a stack. In GoRouter, this means declaring your entire route tree upfront—a single, readable structure that maps URLs to screens—and letting the router handle the rest.

**Purpose:**
- Define all routes in one place, eliminating scattered navigation logic
- Use the URL as the single source of truth for navigation state
- Enable deep linking, web URL synchronization, and complex navigation patterns without manual stack management
- Make navigation predictable, testable, and maintainable

**Where It Fits:**
Declarative routing is the **core philosophy** that distinguishes GoRouter from Navigator 1.0. Instead of imperative `Navigator.push()` calls scattered across your widget tree, you define a route tree once—in a `router.dart` file or directly in `main.dart`—and the router takes over from there.

---

## Why It Exists

### The Problem: Imperative Navigation Doesn't Scale

Traditional Navigator 1.0 navigation is **imperative**—you issue commands like `Navigator.push(context, route)` and `Navigator.pop(context)`. While simple for linear flows, this approach breaks down as apps grow:

| Problem | Description |
|---------|-------------|
| **Scattered Logic** | Push/pop commands are sprinkled across dozens of widgets, making navigation hard to trace |
| **No Global View** | There's no single place to see all possible navigation paths |
| **Deep Linking is Manual** | Opening a deep link like `/user/123/post/456` requires manually constructing the entire stack—pushing user profile, then post—and handling edge cases |
| **Web URL Sync is Impossible** | The browser address bar doesn't reflect the current screen |
| **Authentication Logic is Duplicated** | Login checks must be repeated in every screen or every push |
| **Nested Navigation is Fragile** | Managing multiple Navigators for tabs is complex and error-prone |

### The Previous Limitation: Navigator 2.0 Was Too Complex

Flutter's Navigator 2.0 (the Router API) introduced **declarative** navigation to solve these problems. You describe the desired page stack via a `pages` list, and Flutter reconciles the difference.

However, the raw Navigator 2.0 API requires implementing:
- A custom `RouterDelegate` (manages the `Navigator`'s `pages` list)
- A custom `RouteInformationParser` (parses URLs into a custom type)
- Manual handling of back navigation, state restoration, and URL updates

This involves **hundreds of lines of boilerplate** and a steep learning curve.

### Why GoRouter Introduced This

GoRouter was created to **make declarative routing accessible** to every Flutter developer:

> "The purpose of the go_router package is to use declarative routes to reduce complexity, regardless of the platform you're targeting (mobile, web, desktop), handle deep and dynamic linking from Android, iOS and the web, along with a number of other navigation-related scenarios, while still providing an easy-to-use developer experience."

GoRouter abstracts away Navigator 2.0's complexity, giving you a simple, URL-based API that works across all platforms.

---

## Syntax / Basic Usage

Here's the simplest possible declarative route configuration:

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

/// The entire route tree for the app, defined declaratively in one place.
/// This is the "single source of truth" for all navigation.
final GoRouter _router = GoRouter(
  // The route tree: every screen is a GoRoute with a path and a builder
  routes: [
    // Root route — the entry point of the app
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    // A route with a path parameter — :id is extracted from the URL
    GoRoute(
      path: '/details/:id',
      builder: (context, state) {
        // The router automatically parses :id from the URL
        final id = state.pathParameters['id']!;
        return DetailsScreen(id: id);
      },
    ),
    // A route with a query parameter — accessed via state.uri.queryParameters
    GoRoute(
      path: '/profile',
      builder: (context, state) {
        final tab = state.uri.queryParameters['tab'] ?? 'overview';
        return ProfileScreen(initialTab: tab);
      },
    ),
  ],
);

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    // Use MaterialApp.router instead of MaterialApp
    return MaterialApp.router(
      routerConfig: _router,  // ← Inject the declarative route configuration
      title: 'Declarative Routing Demo',
    );
  }
}

// ====== SCREEN WIDGETS ======

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
            // Navigate using URLs — declarative, not imperative
            ElevatedButton(
              onPressed: () {
                // 'go' replaces the entire stack with the target route
                context.go('/details/42');
              },
              child: const Text('Go to Details (id=42)'),
            ),
            ElevatedButton(
              onPressed: () {
                // 'push' adds the route on top of the current stack
                context.push('/details/99');
              },
              child: const Text('Push Details (id=99)'),
            ),
            ElevatedButton(
              onPressed: () {
                // Navigate with a query parameter
                context.go('/profile?tab=settings');
              },
              child: const Text('Go to Profile (settings tab)'),
            ),
          ],
        ),
      ),
    );
  }
}

class DetailsScreen extends StatelessWidget {
  final String id;

  const DetailsScreen({super.key, required this.id});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Details #$id')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Showing details for item #$id'),
            ElevatedButton(
              onPressed: () {
                // Pop back to the previous screen
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

class ProfileScreen extends StatelessWidget {
  final String initialTab;

  const ProfileScreen({super.key, required this.initialTab});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Profile')),
      body: Center(
        child: Text('Showing tab: $initialTab'),
      ),
    );
  }
}
```

### Code Breakdown

| Line/Block | Explanation |
|------------|-------------|
| `GoRouter(routes: [...])` | The entire route configuration is defined in one place—the `routes` list |
| `GoRoute(path: '/', builder: ...)` | A single route definition. The `path` is the URL pattern; the `builder` returns the widget |
| `path: '/details/:id'` | A path parameter (`:id`) that GoRouter automatically extracts from the URL |
| `state.pathParameters['id']` | Access the extracted path parameter. The router handles parsing automatically |
| `state.uri.queryParameters['tab']` | Access query parameters from the URL (e.g., `?tab=settings`) |
| `MaterialApp.router(routerConfig: _router)` | The integration point—the app now uses the declarative router instead of imperative Navigator |
| `context.go('/details/42')` | **Declarative navigation**: set the URL to `/details/42`. The router matches this against the route tree and updates the UI |
| `context.push('/details/99')` | **Imperative addition**: push a new route on top of the existing stack, preserving the current state |
| `context.pop()` | Pop the current route—works the same as Navigator 1.0 |

### The Critical Distinction: `go` vs `push`

| Method | Behavior | Mental Model |
|--------|----------|--------------|
| **`context.go('/path')`** | Rebuilds the entire navigation stack to match the target URL. Ancestor routes are automatically included | **Declarative** — "The URL is now `/path`" |
| **`context.push('/path')`** | Adds the target route on top of the **existing** stack, preserving the current state | **Imperative** — "Add this screen on top" |

**Example**: If you're on `/detail` and call `context.go('/modal')`, you'll end up with `[Home, Modal]`—the detail is discarded because the URL no longer includes it. If you call `context.push('/modal')`, you'll get `[Home, Detail, Modal]`. The `go` method embodies **declarative** thinking—the URL is the source of truth.

---

## Mental Model

### The "Map" vs. "Tour Guide" Analogy

**Imperative Navigation (Navigator 1.0)** is like a **tour guide** who physically leads you through a building:

```
You: "Take me to the next room."
Guide: [Opens door, walks you through]
You: "Now take me back."
Guide: [Turns around, walks you back]
```

You issue **commands** ("push," "pop"), and the guide executes them. You have no map—you only know where you are relative to where you've been.

**Declarative Navigation (GoRouter)** is like a **map** of the entire building:

```
You: [Point to a location on the map] "I want to be here."
Map: [Shows you the path, figures out the route]
```

You declare **where you want to be** (the URL), and the system figures out how to get there. You always have a complete view of all possible destinations.

### The Route Tree as a File System

Declarative routing treats your app's navigation like a **file system**:

```
/                           ← Root directory (must exist)
├── /login                  ← File: login screen
├── /home                   ← Directory: home section
│   ├── /home/feed          ← File: feed screen
│   └── /home/profile       ← File: profile screen
├── /products               ← Directory: products section
│   └── /products/:id       ← File: product detail (dynamic)
└── /settings               ← File: settings screen
```

Each route is a **location** in this file system. The URL is the **path** to that location. This hierarchical structure is the foundation for nested routes and deep linking.

### The "Single Source of Truth" Concept

```
┌─────────────────────────────────────────────────────────────────┐
│                    APP STATE (The URL)                         │
│                 /products/123?view=detailed                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   GoRouter (The Router)                        │
│  1. Parse URL: path = /products/123, query = view=detailed    │
│  2. Match against route tree: /products/:id                    │
│  3. Extract parameters: id=123, view=detailed                 │
│  4. Build page stack: [Home, Products, ProductDetail]         │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   FLUTTER UI                                   │
│  [Home] → [Products] → [ProductDetail (id=123, detailed)]     │
└─────────────────────────────────────────────────────────────────┘
```

The URL is the **single source of truth**. Change the URL, and the UI updates automatically. This is the essence of declarative routing.

---

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Declarative Configuration** | The entire route tree is defined in one place, typically a `GoRouter` instance with a `routes` list |
| **URL as Source of Truth** | Navigation state is represented by the current URL. Changing the URL changes the UI |
| **Route Tree** | A hierarchical structure of `GoRoute` objects that mirrors the app's navigation paths |
| **Path Parameters** | Variables in the URL path (e.g., `/user/:id`) that are extracted and passed to the builder |
| **Query Parameters** | Key-value pairs after `?` in the URL (e.g., `?tab=settings`) that provide additional context |
| **State-Driven** | The UI is a function of the URL state—when state changes, the UI reconciles |
| **Declarative Navigation** | Use `context.go()` to set the URL; the router handles the stack |
| **Imperative Addition** | Use `context.push()` to add a screen imperatively on top of the existing stack |
| **Centralized Logic** | Authentication, error handling, and redirects are configured once in the router |

---

## How It Works

### Step-by-Step: App Initialization

```
1. App starts → MaterialApp.router builds
   │
   ▼
2. GoRouter reads the routes list and builds an internal route tree
   │
   ▼
3. GoRouter checks initialLocation (defaults to '/')
   │
   ▼
4. GoRouter matches '/' against the route tree
   │   └── Finds GoRoute(path: '/') → calls its builder
   │
   ▼
5. The Navigator displays the HomeScreen as the initial page
   │
   ▼
6. (On web) The browser URL is set to '/'
```

### Step-by-Step: Declarative Navigation (`context.go`)

When you call `context.go('/details/42')`:

```
1. User taps button → context.go('/details/42')
   │
   ▼
2. GoRouter parses the URL: path = '/details/42'
   │
   ▼
3. GoRouter matches against the route tree:
   │   └── Finds GoRoute(path: '/details/:id')
   │       └── Extracts parameter: id = '42'
   │
   ▼
4. GoRouter builds the page stack from the matched routes:
   │   └── [HomeScreen, DetailsScreen(id: '42')]
   │
   ▼
5. GoRouter updates the Navigator's pages list
   │
   ▼
6. Flutter reconciles the difference (animates transition)
   │
   ▼
7. UI shows DetailsScreen with id=42
   │
   ▼
8. (On web) The browser URL updates to '/details/42'
```

The key insight: **you never manually manipulate the stack**. You declare the desired URL, and GoRouter builds the stack for you.

### Step-by-Step: Imperative Addition (`context.push`)

When you call `context.push('/modal')` from the DetailsScreen:

```
1. User taps button → context.push('/modal')
   │
   ▼
2. GoRouter parses the URL: path = '/modal'
   │
   ▼
3. GoRouter matches against the route tree:
   │   └── Finds GoRoute(path: '/modal')
   │
   ▼
4. GoRouter adds the new route on top of the EXISTING stack:
   │   └── [HomeScreen, DetailsScreen, ModalScreen]
   │
   ▼
5. GoRouter updates the Navigator's pages list
   │
   ▼
6. Flutter animates the modal on top
   │
   ▼
7. (On web) The URL does NOT change—push does not update the browser URL
```

> **Note**: Starting from GoRouter 8.0, `push` no longer changes the browser URL on web—the URL remains the same as the parent route.

---

## Internal Architecture / Under the Hood

Declarative routing in GoRouter works by **wrapping Navigator 2.0's complex APIs**:

| Navigator 2.0 Component | What GoRouter Does |
|--------------------------|-------------------|
| **`RouterDelegate`** | GoRouter provides a custom delegate that manages the `Navigator`'s `pages` list based on the current URL |
| **`RouteInformationParser`** | GoRouter parses URLs into `GoRouterState` objects automatically |
| **`BackButtonDispatcher`** | GoRouter handles Android's system back button by updating the URL state |
| **`Navigator`** | GoRouter updates the `pages` list declaratively—you never touch it directly |

When you define `routes` in GoRouter, it:
1. **Builds an internal trie (tree) structure** for efficient route matching
2. **Registers each route's path** with its builder
3. **Sets up URL parsing** to extract path and query parameters
4. **Connects everything** to the Navigator 2.0 `Router` widget

All of this happens **behind the scenes**.

---

## Lifecycle / Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                     DECLARATIVE ROUTING LIFECYCLE              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. DEFINE                                                      │
│     └── Create GoRouter with routes list                        │
│                                                                  │
│  2. INTEGRATE                                                   │
│     └── Pass to MaterialApp.router(routerConfig: _router)      │
│                                                                  │
│  3. NAVIGATE (DECLARATIVE)                                      │
│     └── context.go('/path') → URL changes → UI updates         │
│                                                                  │
│  4. NAVIGATE (IMPERATIVE ADDITION)                              │
│     └── context.push('/path') → adds screen on top             │
│                                                                  │
│  5. POP                                                         │
│     └── context.pop() → removes top screen                     │
│                                                                  │
│  6. REDIRECT                                                    │
│     └── redirect function intercepts and changes destination   │
│                                                                  │
│  7. ERROR                                                       │
│     └── errorBuilder shows fallback for unknown routes          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Practical Examples

### Example 1: Basic Declarative vs Imperative Comparison

```dart
// ===== IMPERATIVE (Navigator 1.0) =====
// Navigation logic is scattered and imperative
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        // Imperative command: "Push this screen"
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (_) => DetailsScreen(id: '42'),
          ),
        );
      },
      child: Text('Go to Details'),
    );
  }
}
// Problem: The navigation logic is inside the widget.
// There's no single place to see all possible routes.
// Deep linking requires manual stack construction.

// ===== DECLARATIVE (GoRouter) =====
// All routes are defined in one place
final router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: (_, __) => const HomeScreen()),
    GoRoute(
      path: '/details/:id',
      builder: (_, state) => DetailsScreen(id: state.pathParameters['id']!),
    ),
  ],
);

// Navigation is declarative: "The URL is now /details/42"
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () => context.go('/details/42'),  // ← Declarative
      child: Text('Go to Details'),
    );
  }
}
// Benefit: All routes are centralized.
// Deep linking works automatically: /details/42 opens the right screen.
// The URL is the source of truth.
```

### Example 2: Nested Routes (Declarative Hierarchy)

```dart
final router = GoRouter(
  routes: [
    // Root route with children—nested navigation
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
      routes: [
        // Child routes are displayed on top of the parent
        GoRoute(
          path: 'products',  // → /products
          builder: (context, state) => const ProductsScreen(),
          routes: [
            // Grandchild route with a path parameter
            GoRoute(
              path: ':id',  // → /products/:id
              builder: (context, state) {
                final id = state.pathParameters['id']!;
                return ProductDetailScreen(id: id);
              },
            ),
          ],
        ),
        GoRoute(
          path: 'settings',  // → /settings
          builder: (context, state) => const SettingsScreen(),
        ),
      ],
    ),
  ],
);

// Navigating to /products/123 builds the stack:
// [HomeScreen, ProductsScreen, ProductDetailScreen(id: '123')]
// The URL structure mirrors the UI structure
```

### Example 3: Authentication Guard (Declarative Logic)

```dart
final router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: (_, __) => const HomeScreen()),
    GoRoute(path: '/login', builder: (_, __) => const LoginScreen()),
    GoRoute(path: '/profile', builder: (_, __) => const ProfileScreen()),
  ],
  // Centralized authentication logic—runs on every navigation
  redirect: (context, state) {
    final isLoggedIn = AuthService.isLoggedIn();
    final isLoginRoute = state.matchedLocation == '/login';
    
    // Not logged in and trying to access protected route → go to login
    if (!isLoggedIn && !isLoginRoute) {
      return '/login';  // ← Declarative redirect
    }
    // Logged in and on login route → go to home
    if (isLoggedIn && isLoginRoute) {
      return '/';  // ← Declarative redirect
    }
    // Otherwise, allow navigation
    return null;
  },
);
// Benefit: Authentication logic is centralized, not scattered across screens
```

---

## Common Use Cases

1. **Deep linking**: Opening `/user/123/post/456` automatically builds the correct stack
2. **Web URL synchronization**: The browser address bar stays in sync with the app state
3. **Authentication flows**: Centralized `redirect` guards all protected routes
4. **Nested navigation**: Tabs with independent navigation stacks via `ShellRoute`
5. **Feature flags**: Conditional routing based on user permissions or app version
6. **Onboarding flows**: Declarative redirects based on user progress

---

## Best Practices

### 1. Define All Routes in One Place

**Why**: Centralization makes the route tree easy to find, modify, and reason about.

**How**: Create a dedicated `router.dart` file with the complete `GoRouter` configuration.

### 2. Use `go` for Declarative Navigation, `push` for Imperative Additions

**Why**: `go` embodies declarative thinking—the URL is the source of truth. `push` is for flows where the user must return to the exact previous screen.

**How**:
- Use `context.go()` for top-level navigation (tabs, home, logout)
- Use `context.push()` for drill-down flows (detail views, modals)

### 3. Keep Builders Thin

**Why**: The builder's job is to read parameters and pass them to the screen—nothing more.

**How**: Extract widget construction into the screen itself.

```dart
// GOOD: Builder only reads parameters
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = state.pathParameters['id']!;
    return ProductScreen(id: id);  // ← Screen handles its own UI
  },
)

// BAD: Builder contains UI logic
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = state.pathParameters['id']!;
    return Scaffold(  // ← UI logic in the builder
      appBar: AppBar(title: Text('Product $id')),
      body: Center(child: Text('Details')),
    );
  },
)
```

### 4. Nest Routes to Reflect Hierarchy

**Why**: The URL structure should mirror the UI structure.

**How**: If a product detail is a child of products, nest it under `/products`.

```dart
// GOOD: Nested hierarchy
GoRoute(
  path: '/products',
  builder: (_, __) => ProductsScreen(),
  routes: [
    GoRoute(
      path: ':id',  // → /products/:id
      builder: (_, state) => ProductDetailScreen(id: state.pathParameters['id']!),
    ),
  ],
)

// BAD: Flat structure
GoRoute(path: '/products', builder: ...),
GoRoute(path: '/product/:id', builder: ...),  // ← Not nested, URL doesn't reflect hierarchy
```

### 5. Use Declarative Redirects for Authentication

**Why**: Authentication logic belongs in the router, not scattered across screens.

**How**: Use the `redirect` parameter on the `GoRouter` instance.

---

## Common Mistakes

### ❌ Mistake 1: Using `Navigator.push` with GoRouter

```dart
// WRONG: Mixing imperative and declarative
ElevatedButton(
  onPressed: () {
    Navigator.push(  // ← This bypasses GoRouter entirely!
      context,
      MaterialPageRoute(builder: (_) => DetailsScreen()),
    );
  },
  child: Text('Go to Details'),
)
```

**Why it's wrong**: `Navigator.push` bypasses GoRouter's declarative system. The URL won't update, deep linking breaks, and the router loses track of the stack.

**Correct**:
```dart
ElevatedButton(
  onPressed: () => context.go('/details/42'),  // ← Use GoRouter's API
  child: Text('Go to Details'),
)
```

### ❌ Mistake 2: Using `home` or `routes` with `MaterialApp.router`

```dart
// WRONG: Mixing old and new APIs
MaterialApp.router(
  routerConfig: router,
  home: const HomeScreen(),  // ← Ignored or causes errors
  routes: { '/details': ... },  // ← Ignored
)
```

**Why it's wrong**: `MaterialApp.router` ignores `home` and `routes`. All routes must be defined in the `GoRouter` configuration.

**Correct**:
```dart
MaterialApp.router(
  routerConfig: router,  // ← All routes are inside the router
)
```

### ❌ Mistake 3: Using `context.go` Inside a Builder

```dart
// WRONG: Navigation inside builder
GoRoute(
  path: '/',
  builder: (context, state) {
    if (!AuthService.isLoggedIn()) {
      context.go('/login');  // ← Causes infinite loop!
    }
    return const HomeScreen();
  },
)
```

**Why it's wrong**: The builder is called during the render phase. Performing navigation inside it causes infinite loops or inconsistent state.

**Correct**: Use `redirect` for conditional navigation.

### ❌ Mistake 4: Forgetting to Handle the Root Route

```dart
// WRONG: No root route
final router = GoRouter(
  routes: [
    GoRoute(path: '/login', builder: ...),
    GoRoute(path: '/home', builder: ...),
  ],
);
// Throws: 'The routes list must contain a route matching "/"'
```

**Why it's wrong**: GoRouter requires a root route (`/`) as the entry point.

**Correct**:
```dart
final router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: ...),  // ← Root route first
    GoRoute(path: '/login', builder: ...),
    GoRoute(path: '/home', builder: ...),
  ],
);
```

---

## Performance Considerations

- **Route matching is O(n)** where n is the number of routes. For most apps (<100 routes), this is negligible.
- **`redirect` runs on every navigation**—keep it lightweight. Avoid expensive operations like network calls.
- **Lazy loading**: Consider `go_router_deferred` for large apps to defer route loading.
- **Builder functions should be cheap**—they're called on every navigation to that route.
- **Use `const` where possible** for stateless screens to enable widget caching.

---

## Security Considerations

- **Don't log sensitive information** in `debugLogDiagnostics`—it prints all navigation events including query parameters.
- **Use `redirect` for authentication**—never rely on client-side checks alone. Always validate on the server.
- **Sanitize path parameters** before using them—users can inject malicious strings via deep links.
- **Query parameters are visible in the URL**—don't pass sensitive data (tokens, passwords) in query parameters.

---

## Debugging Tips

1. **Enable `debugLogDiagnostics: true`** to see every navigation event, route match, and parameter extraction
2. **Check the `redirectLimit`**—if you're hitting it, you likely have a redirect loop
3. **Use `GoRouter.of(context).routerDelegate.currentConfiguration`** to debug the current route configuration
4. **Check browser developer tools** (on web) for navigation errors
5. **Add a `NavigatorObserver`** to log navigation events programmatically

---

## When to Use

- **Any app with more than a few screens**—declarative routing scales better than imperative
- **Apps that need deep linking**—declarative routing makes deep linking automatic
- **Flutter Web apps**—URL synchronization is built-in
- **Apps with authentication**—centralized redirects are cleaner than scattered checks
- **Apps with nested navigation** (tabs, drawers)—`ShellRoute` handles this elegantly
- **Teams that value maintainability**—centralized routing is easier to reason about

> **Most Flutter apps in 2026 should use declarative routing via GoRouter.**

---

## When Not to Use

- **Tiny apps with 1-2 screens**—the overhead of GoRouter isn't justified
- **Prototypes or demos**—Navigator 1.0 is faster to set up
- **Apps that need to stay dependency-light**—GoRouter adds a dependency
- **Deeply custom navigation requirements** that GoRouter doesn't support (rare)

---

## Related Concepts

- `GoRouter`
- `GoRoute`
- `ShellRoute`
- `Redirect`
- `GoRouterState`
- `Navigator 2.0`
- `RouterDelegate`
- `RouteInformationParser`

---

## Did You Know?

- GoRouter's declarative routing is built on **Navigator 2.0**, but you never have to touch `RouterDelegate` or `RouteInformationParser` directly
- The `go` method **rebuilds the entire stack** to match the target URL—this is the essence of declarative routing
- Declarative routing makes your app **testable**—you can test navigation by checking the URL state
- The route tree is just a **list of `RouteBase` objects**—you can build it dynamically at runtime if needed
- GoRouter's declarative approach is inspired by **web frameworks** like React Router and Express

---

## Summary

- **Declarative routing** means defining **what** the navigation should look like, not issuing **imperative** commands
- **The URL is the single source of truth** for navigation state
- **All routes are defined in one place**—the `routes` list in the `GoRouter` constructor
- **`context.go()`** is declarative—it sets the URL and rebuilds the stack
- **`context.push()`** is imperative—it adds a screen on top of the existing stack
- **Declarative routing makes deep linking, web support, and authentication guards trivial**
- **It's the recommended approach for most Flutter apps in 2026**
- **The route tree should mirror your UI hierarchy**—nest routes to reflect relationships
- **Keep builders thin**—extract UI logic into the screen widgets

---

