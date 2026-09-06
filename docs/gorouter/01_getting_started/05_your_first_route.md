# Your First Route

## Overview

**Your first route** is the foundational building block of any GoRouter-powered app. A route in GoRouter—defined using the `GoRoute` class—represents a **mapping between a URL path and a screen widget**. Once defined, your app can navigate to that screen using the URL, whether by user interaction, deep linking, or browser address bar changes.

**Purpose:**
- Establish the entry point for your app's navigation system
- Replace the traditional `home:` and `routes:` parameters in `MaterialApp`
- Enable URL-based navigation from the very first screen

**Where It Fits:**
Your first route is almost always the **root route** (`/`), which defines the initial screen of your app. From this foundation, you build your entire route tree, adding more routes for each screen in your application.

---

## Why It Exists

### The Problem: The Old Way Lacks URLs

With Navigator 1.0, you would set:

```dart
MaterialApp(
  home: HomeScreen(),  // ← Root screen is hardcoded
  routes: {
    '/details': (context) => DetailsScreen(),
  },
)
```

The root screen is **static**—it's always the same widget. There's no URL representation of the home screen. If a user opens a deep link to `/details`, you can't easily preserve the fact that the root screen is still `HomeScreen`.

### The Previous Limitation: No Declarative Root

Before GoRouter, defining a root route declaratively was impossible. You either:
- Hardcoded `home:` (no URL sync)
- Implemented Navigator 2.0's `RouterDelegate` (complex boilerplate)

### Why GoRouter Introduced This

GoRouter makes the root route **declarative** and **URL-addressable**. The root route:
- Has a **path** (`/`)
- Is part of the **route tree** (the base of all navigation)
- Can be navigated to **explicitly** via `context.go('/')`
- Supports **redirects** (e.g., if not authenticated, redirect to login)

Your first route is the **anchor** for all subsequent navigation.

---

## Syntax / Basic Usage

The simplest possible GoRouter setup with one route:

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

// 1. Define the root route
final GoRouter _router = GoRouter(
  routes: [
    GoRoute(
      path: '/', // ← The root URL path
      builder: (context, state) => const HomeScreen(),
      // 'state' contains information about the current navigation
    ),
  ],
);

// 2. Use it in your app
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router, // ← Inject the router
      title: 'My First GoRouter App',
    );
  }
}

// 3. A simple home screen
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: const Center(
        child: Text('Welcome to GoRouter!'),
      ),
    );
  }
}

void main() => runApp(const MyApp());
```

### Code Breakdown

| Line/Block | Explanation |
|------------|-------------|
| `import 'package:go_router/go_router.dart';` | Imports the GoRouter package, making `GoRouter` and `GoRoute` available. |
| `final GoRouter _router = GoRouter(routes: [...])` | Creates the router instance. The `routes` list must contain at least the root route. |
| `GoRoute(path: '/', builder: ...)` | Defines a route that matches the **exact path** `/`. The `builder` receives `context` and `state` and returns a `Widget`. |
| `builder: (context, state) => const HomeScreen()` | The builder function is called when this route is matched. The `state` parameter (not used here) provides access to parameters, location, etc. |
| `MaterialApp.router(routerConfig: _router)` | Tells `MaterialApp` to use a `Router` instead of a `Navigator` with the default `home`/`routes`. This is the entry point for Navigator 2.0. |
| `void main() => runApp(const MyApp());` | Standard Flutter entry point. |

---

## Mental Model

### The Route as a Page in a Book

Think of each `GoRoute` as a **chapter** in a book:

```
┌─────────────────────────────────────────────────────────────┐
│                      Your App (The Book)                   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Chapter 1: Home Screen                           │   │
│  │  URL path: /                                      │   │
│  │  Builder: returns HomeScreen widget               │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Chapter 2: Details Screen                        │   │
│  │  URL path: /details                               │   │
│  │  Builder: returns DetailsScreen widget            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

The root route (`/`) is the **first chapter**—the page that always exists and is the default destination. When you navigate to `/details`, you're turning to a different chapter. The URL is the **bookmark** that tells you exactly where you are.

### The Route Tree as a File System

Another way to think about routes is like a **file system**:

```
/                    ← Root directory (HomeScreen)
├── /about           ← File "about"
├── /products         ← Directory "products"
│   ├── /products/123 ← File "123" inside products
│   └── /products/456
└── /profile          ← File "profile"
```

Your first route (`/`) is the **root directory**. Every other route is a file or subdirectory inside it. This hierarchical structure is the foundation for nested routes and deep linking.

---

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Root Route** | The route with `path: '/'`. It's the entry point when the app starts or when you navigate to the root URL. |
| **Path** | A string that defines the URL pattern, e.g., `/`, `/details`, `/user/:id`. Must start with `/`. |
| **Builder** | A function that returns the `Widget` to display for this route. Receives `BuildContext` and `GoRouterState`. |
| **GoRouterState** | An object containing current navigation information: `location`, `pathParameters`, `queryParameters`, `matchedRoutes`, etc. |
| **MaterialApp.router** | The constructor that replaces the standard `MaterialApp` and accepts a `routerConfig`. |

---

## How It Works

### Step-by-Step: App Startup

```
1. App starts
   │
   ▼
2. MaterialApp.router builds
   │
   ▼
3. Flutter reads routerConfig → GoRouter instance
   │
   ▼
4. GoRouter checks initialLocation (defaults to '/')
   │
   ▼
5. GoRouter matches '/' against routes list
   │
   ▼
6. Finds GoRoute(path: '/') → calls builder
   │
   ▼
7. HomeScreen is displayed as the initial page
   │
   ▼
8. URL is synchronized (on web, the browser shows '/')
```

### Step-by-Step: Navigation to Root

When you call `context.go('/')` from anywhere:

```
1. User taps button → calls context.go('/')
   │
   ▼
2. GoRouter parses the URL '/'
   │
   ▼
3. Matches the root route
   │
   ▼
4. Builds the HomeScreen using the route's builder
   │
   ▼
5. Replaces the entire page stack with just HomeScreen
   │
   ▼
6. Updates the browser URL (on web) to '/'
   │
   ▼
7. User sees HomeScreen
```

> **Key Insight**: The root route is always available and can be navigated to explicitly. This is useful for "go home" buttons or resetting the navigation stack.

---

## Practical Examples

### Example 1: Basic Root Route with State

This example shows how you might use the `state` parameter to conditionally modify the root screen:

```dart
GoRoute(
  path: '/',
  builder: (context, state) {
    // Check if the user is authenticated (simplified)
    final isAuthenticated = state.extra != null && state.extra == 'authenticated';
    return HomeScreen(showWelcome: isAuthenticated);
  },
)
```

### Example 2: Root Route with Query Parameters

You can read query parameters from the root route URL (e.g., `/?utm_source=email`):

```dart
GoRoute(
  path: '/',
  builder: (context, state) {
    final utmSource = state.uri.queryParameters['utm_source'];
    return HomeScreen(utmSource: utmSource);
  },
)
```

### Example 3: Root Route with Navigation to Other Screens

The root route can include buttons that navigate elsewhere:

```dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // Navigate to a details screen (assuming it's defined)
            context.go('/details');
          },
          child: const Text('Go to Details'),
        ),
      ),
    );
  }
}
```

> **Note**: For this to work, you must also define the `'/details'` route in your `routes` list.

---

## Common Use Cases

1. **App Entry Point**: The root route is the first screen users see when the app launches.
2. **Landing Page**: The root route can be a marketing or onboarding screen.
3. **Dashboard**: For authenticated apps, the root route often becomes the main dashboard.
4. **Deep Link Fallback**: If a deep link doesn't match any route, you can redirect to the root route.

---

## Best Practices

1. **Always define a root route (`/`)**: Without it, your app has no entry point.
2. **Keep the root route simple**: It should be the starting point for navigation, not a complex screen with heavy logic.
3. **Use const constructors**: For stateless widgets like `HomeScreen`, use `const` to enable widget caching and improve performance.
4. **Avoid side effects in the builder**: The builder should be pure—it should only return a widget based on `state`. Don't perform network calls or navigation inside the builder.
5. **Consider redirects for authentication**: If your app requires login, use a `redirect` on the root route (or globally) instead of building a different widget conditionally.

---

## Common Mistakes

### ❌ Mistake 1: Forgetting the Root Route

```dart
// WRONG: No root route defined
final router = GoRouter(
  routes: [
    GoRoute(path: '/details', builder: ...), // Only details
  ],
);
// This will throw an error because the app has no entry point.
```

**Why it's wrong**: GoRouter requires at least one route, and the first route is usually the root. Without `/`, the app has no initial page.

**Correct**:
```dart
final router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: ...), // Root first
    GoRoute(path: '/details', builder: ...),
  ],
);
```

### ❌ Mistake 2: Using `home:` with `MaterialApp.router`

```dart
// WRONG: Mixing old and new routing
MaterialApp.router(
  routerConfig: _router,
  home: const HomeScreen(), // ← This will be ignored or cause errors
)
```

**Why it's wrong**: `MaterialApp.router` doesn't use `home` or `routes`. You must define all routes in the `GoRouter` configuration.

**Correct**:
```dart
MaterialApp.router(
  routerConfig: _router, // All routes are inside the router
)
```

### ❌ Mistake 3: Complex Logic Inside the Builder

```dart
// WRONG: Navigation inside builder
GoRoute(
  path: '/',
  builder: (context, state) {
    if (someCondition) {
      context.go('/login'); // ← This will cause infinite loops
    }
    return const HomeScreen();
  },
)
```

**Why it's wrong**: The builder is called during the render phase. Performing navigation inside it can cause infinite loops or inconsistent state.

**Correct**: Use `redirect` for conditional navigation, or perform navigation from event handlers (e.g., button presses).

---

## Performance Considerations

- **Root route builder is called once per navigation to `/`**: The build cost is minimal.
- **Use `const` where possible**: If your root widget is stateless, mark it as `const` to allow Flutter to reuse it.
- **Lazy-loading**: The root route is always loaded at app startup—keep it lightweight.

---

## Debugging Tips

- **Enable logging**: Set `debugLogDiagnostics: true` on your `GoRouter` to see route matching in the console:
  ```dart
  final router = GoRouter(
    debugLogDiagnostics: true, // ← Logs all navigation events
    routes: [...],
  );
  ```
- **Check URL parsing**: If your root route isn't matching, ensure the path is exactly `/` (no trailing slash issues).
- **Verify routerConfig**: Make sure you pass `routerConfig: _router` and not `router: _router` (common mistake).

---

## When to Use

- **Every app needs a root route**: This is non-negotiable.
- **When you want to navigate "home"**: Use `context.go('/')` to clear the stack and return to the root.
- **When deep linking to the root**: A URL like `myapp://` or `https://myapp.com/` will resolve to the root route.

---

## When Not to Use

- You don't need a separate root route if your app has no navigation at all (single screen). But even then, you still define it as a `GoRoute`—it's required.

---

## Related Concepts

- `GoRouterState`
- `initialLocation`
- `redirect`
- `shellRoute`
- `nested routes`

---

## Did You Know?

- The root route's path **must** be `'/'`—it cannot be named differently. This is because GoRouter uses the URL as the source of truth, and `/` is the standard root path across all web and routing systems.
- You can have **multiple root-level routes** in the `routes` list, not just one. For example, `/` and `/login` can both be top-level routes (parallel pages).
- The `builder` of the root route receives a `GoRouterState` where `state.matchedRoutes` contains at least one element: the root route itself.

---

## Summary

- **Your first route is the root route (`/`)** —the entry point of your app.
- **Define it** with `GoRoute(path: '/', builder: ...)` inside the `routes` list of `GoRouter`.
- **Use `MaterialApp.router`** with `routerConfig` to inject the router.
- **The builder** returns the widget for that route and receives `state` for context.
- **Always define a root route** to avoid initialization errors.
- **Keep the builder pure**—use `redirect` for conditional logic.
- **Enable `debugLogDiagnostics`** to debug routing issues.

---
