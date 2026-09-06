# Route Configuration

## Overview

**Route configuration** is the process of defining your app's entire navigation structure in a single, declarative `GoRouter` instance. This configuration serves as the **single source of truth** for all navigation in your app—mapping URLs to screens, defining nested hierarchies, and specifying how the router should behave under various conditions.

**Purpose:**
- Centralize all routing logic in one place
- Establish the relationship between URLs and widgets
- Configure router-wide behavior (logging, error handling, redirection)
- Enable deep linking and URL-based navigation

**Where It Fits:**
Route configuration is the **foundation** of your GoRouter setup. It lives at the root of your app, typically in `main.dart` or a dedicated `router.dart` file, and is passed to `MaterialApp.router` via the `routerConfig` parameter.

---

## Why It Exists

### The Problem: Scattered Navigation Logic

Before GoRouter, navigation logic was scattered across your app:
- **`MaterialApp.routes`** defined named routes in one place
- **`Navigator.push`** calls were scattered throughout widget trees
- **Deep linking** required manual URL parsing in multiple places
- **Authentication checks** were duplicated across screens
- **Nested navigation** (tabs, sub-navigators) required complex manual management

This scattering made routing **hard to reason about, test, and maintain**.

### The Previous Limitation: No Declarative, Centralized Configuration

Navigator 1.0 offered no way to define your entire navigation structure declaratively. Navigator 2.0's raw API required implementing `RouterDelegate` and `RouteInformationParser`—hundreds of lines of boilerplate that were difficult to get right.

### Why GoRouter Introduced This

GoRouter introduces a **declarative, centralized route configuration** where:

- **All routes are defined in one place**—the `routes` list in the `GoRouter` constructor
- **URLs map directly to screens**—no manual parsing required
- **Router-wide behavior** (logging, error handling, redirects) is configured once
- **Nested navigation** is expressed as parent-child relationships in the route tree
- **The configuration is just code**—easy to read, test, and modify

This centralization makes routing **predictable, maintainable, and debuggable**.

---

## Syntax / Basic Usage

Here's a complete route configuration with the most commonly used parameters:

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

/// The global router configuration for the app.
/// 
/// This is the single source of truth for all navigation.
/// It defines the route tree, initial location, error handling,
/// and debugging behavior.
final GoRouter _router = GoRouter(
  // REQUIRED: The route tree - must contain at least a root route '/'
  routes: <RouteBase>[
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/details/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return DetailsScreen(id: id);
      },
    ),
    GoRoute(
      path: '/settings',
      builder: (context, state) => const SettingsScreen(),
    ),
  ],
  
  // OPTIONAL: Where to start when the app launches
  // Defaults to '/' if not provided
  initialLocation: '/',
  
  // OPTIONAL: Enable verbose logging for debugging
  // Prints all navigation events, redirects, and matches to the console
  debugLogDiagnostics: true,
  
  // OPTIONAL: Global navigator key for accessing navigator state
  // Useful for navigation from outside the widget tree (e.g., from BLoC/Cubit)
  navigatorKey: _navigatorKey,
  
  // OPTIONAL: Custom error screen for unknown routes or exceptions
  errorBuilder: (context, state) => const ErrorScreen(),
  
  // OPTIONAL: Global redirect for authentication or other conditions
  // Runs before any route is matched
  redirect: (context, state) {
    final isLoggedIn = AuthService.isLoggedIn();
    final isAuthRoute = state.matchedLocation == '/login';
    
    if (!isLoggedIn && !isAuthRoute) {
      return '/login'; // Redirect to login if not authenticated
    }
    if (isLoggedIn && isAuthRoute) {
      return '/'; // Redirect to home if already logged in
    }
    return null; // Allow navigation to proceed
  },
  
  // OPTIONAL: Maximum consecutive redirects before throwing an error
  // Prevents infinite redirect loops
  redirectLimit: 5,
  
  // OPTIONAL: Refresh listenable for reactive redirects
  // Re-evaluates redirects when the listenable notifies
  refreshListenable: authNotifier,
);

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,
      title: 'My App',
    );
  }
}
```

### Code Breakdown

| Parameter | Type | Required? | Description |
|-----------|------|-----------|-------------|
| `routes` | `List<RouteBase>` | **Yes** | The route tree. Must contain at least one route, and the root route (`/`) must be defined. |
| `initialLocation` | `String?` | No | The path to navigate to when the app starts. Defaults to `'/'` if not provided. |
| `debugLogDiagnostics` | `bool` | No | When `true`, logs all navigation events, redirects, and route matches to the console. Defaults to `false`. |
| `navigatorKey` | `GlobalKey<NavigatorState>?` | No | A global key for accessing the navigator's state. Essential for navigation from outside the widget tree. |
| `errorBuilder` | `GoRouterWidgetBuilder?` | No | Builds a custom error widget when a route fails or an exception is thrown. |
| `redirect` | `GoRouterRedirect?` | No | A function that can intercept navigation and redirect to a different location. |
| `redirectLimit` | `int` | No | Maximum consecutive redirects before throwing an error. Defaults to `5` in recent versions. |
| `refreshListenable` | `Listenable?` | No | A listenable that triggers re-evaluation of redirects when it notifies. |
| `observers` | `List<NavigatorObserver>?` | No | List of navigator observers for monitoring navigation events. |
| `restorationScopeId` | `String?` | No | Enables state restoration for the router. |

---

## Mental Model

### The Router as a "Traffic Controller"

Think of route configuration as a **traffic control system** for your app:

```
┌──────────────────────────────────────────────────────────────────┐
│                    ROUTE CONFIGURATION                          │
│                    (The Traffic Controller)                     │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  routes: The Road Map                                     │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐               │ │
│  │  │   /      │  │ /details │  │ /settings│               │ │
│  │  │  Home    │  │  :id     │  │          │               │ │
│  │  └──────────┘  └──────────┘  └──────────┘               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  initialLocation: The Starting Point                      │ │
│  │  "/" → Everyone starts at Home                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  redirect: The Gatekeeper                                 │ │
│  │  "Are you authenticated? No → Send to /login"            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  errorBuilder: The Emergency Response                     │ │
│  │  "Route not found → Show 404 page"                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  debugLogDiagnostics: The Dashboard                       │ │
│  │  "Log every navigation event for monitoring"              │ │
│  └────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

Each parameter in your configuration plays a specific role in guiding users through your app.

### The Route Tree as a Filesystem

The `routes` list defines a **tree structure**—like a filesystem where each route is a file or directory:

```
/                           ← Root (must exist)
├── /login                  ← Auth route (parallel to root)
├── /home                   ← Home route
│   ├── /home/feed          ← Child route (nested)
│   └── /home/profile       ← Child route (nested)
├── /settings               ← Settings route
└── /product/:id            ← Dynamic route with parameter
```

Each path represents a **location** in your app. The router matches the current URL against this tree to determine what to display.

---

## Core Concepts

### 1. The Route Tree (`routes`)

The `routes` parameter is the **heart** of your configuration—a list of `RouteBase` objects (typically `GoRoute` instances) that define your app's navigation structure.

**Key rules:**
- The list **must not be empty**
- It **must contain a route matching `/`** (the root route)
- Routes are matched in the order they appear in the list

### 2. Initial Location (`initialLocation`)

The `initialLocation` parameter determines **where the app starts** when it launches.

- Defaults to `'/'` if not provided
- Can be any valid path defined in your route tree
- Useful for starting on a specific screen (e.g., `/login` if not authenticated)

### 3. Debug Logging (`debugLogDiagnostics`)

When `debugLogDiagnostics: true`, GoRouter prints detailed logs to the console:

- **Route matching**: Which routes were considered and which matched
- **Redirects**: When and where redirects occurred
- **Navigation events**: Every `go`, `push`, and `pop` operation
- **Errors**: Any exceptions during navigation

This is **invaluable** for debugging routing issues.

### 4. Error Handling (`errorBuilder` / `errorPageBuilder` / `onException`)

GoRouter provides three ways to handle errors:

| Parameter | When It's Called | What It Returns |
|-----------|------------------|-----------------|
| `onException` | An exception is thrown during navigation | Side effect (logging, analytics) |
| `errorPageBuilder` | An exception is thrown and `onException` is not provided | A `Page` object (with transitions) |
| `errorBuilder` | An exception is thrown and neither of the above are provided | A `Widget` (simpler) |

If none are provided, GoRouter builds a **default error screen**.

### 5. Redirects (`redirect`)

The `redirect` parameter allows you to **intercept navigation** and change the destination:

- Called **before** any route is matched
- Returns a `String?`—the new location to navigate to, or `null` to proceed
- Useful for authentication guards, feature flags, and conditional navigation

### 6. Redirect Limit (`redirectLimit`)

Prevents **infinite redirect loops** by limiting consecutive redirects.

- Defaults to `5` in recent versions (was `20` in older versions)
- If the limit is exceeded, GoRouter throws an exception
- Lower this in tests to catch redirect loops early

### 7. Navigator Key (`navigatorKey`)

A `GlobalKey<NavigatorState>` that provides **direct access to the navigator**:

- Essential for navigation **from outside the widget tree** (e.g., from BLoC, Cubit, or service classes)
- Enables programmatic navigation without a `BuildContext`
- Must be defined **before** the `GoRouter` instance

---

## How It Works

### Step-by-Step: App Initialization

```
1. App starts
   │
   ▼
2. GoRouter instance is created with your configuration
   │
   ▼
3. MaterialApp.router reads routerConfig
   │
   ▼
4. GoRouter checks initialLocation (or defaults to '/')
   │
   ▼
5. GoRouter runs the redirect function (if provided)
   │   ├── If redirect returns a String → navigate there instead
   │   └── If redirect returns null → proceed with original location
   │
   ▼
6. GoRouter matches the location against the route tree
   │
   ▼
7. GoRouter builds the matched route(s) using their builders
   │
   ▼
8. The Navigator displays the resulting page stack
   │
   ▼
9. If debugLogDiagnostics is true, logs the entire process
```

### Step-by-Step: Navigation

When you call `context.go('/details/42')`:

```
1. Navigation request received
   │
   ▼
2. redirect function is called
   │   ├── Returns String → redirect and repeat
   │   └── Returns null → proceed
   │
   ▼
3. Route tree is searched for a match
   │   ├── Matches GoRoute(path: '/details/:id')
   │   └── Extracts path parameter: id = '42'
   │
   ▼
4. The route's builder is called with the state
   │
   ▼
5. The new page is added to the navigator's stack
   │
   ▼
6. If debugLogDiagnostics is true, logs the navigation
```

---

## Internal Architecture / Under the Hood

When you create a `GoRouter` instance, it internally:

1. **Builds a `RouterDelegate`**: A custom delegate that manages the `Navigator`'s page stack
2. **Builds a `RouteInformationParser`**: Parses incoming URLs into `GoRouterState` objects
3. **Sets up a `BackButtonDispatcher`**: Handles Android's system back button
4. **Registers the `navigatorKey`** (if provided) for global access
5. **Sets up logging** based on `debugLogDiagnostics`
6. **Configures error handling** based on `onException`/`errorPageBuilder`/`errorBuilder`

All of this happens **behind the scenes**—you just provide the configuration.

---

## Practical Examples

### Example 1: Production-Ready Configuration

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

// 1. Define a global navigator key for outside-widget navigation
final GlobalKey<NavigatorState> _navigatorKey = GlobalKey<NavigatorState>();

// 2. Define the route configuration
final GoRouter router = GoRouter(
  // REQUIRED: The route tree
  routes: <RouteBase>[
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
      routes: [
        GoRoute(
          path: 'profile',
          builder: (context, state) => const ProfileScreen(),
        ),
      ],
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductScreen(id: id);
      },
    ),
  ],
  
  // Start at the home screen
  initialLocation: '/',
  
  // Enable verbose logging in development only
  debugLogDiagnostics: kDebugMode,
  
  // Provide the navigator key for global navigation
  navigatorKey: _navigatorKey,
  
  // Custom error screen
  errorBuilder: (context, state) => const NotFoundScreen(),
  
  // Authentication guard
  redirect: (context, state) {
    final isLoggedIn = AuthService.isLoggedIn();
    final isLoginRoute = state.matchedLocation == '/login';
    
    // Not logged in and trying to access protected route → go to login
    if (!isLoggedIn && !isLoginRoute) {
      return '/login';
    }
    // Logged in and on login route → go to home
    if (isLoggedIn && isLoginRoute) {
      return '/';
    }
    // Otherwise, allow navigation
    return null;
  },
  
  // Prevent infinite redirect loops
  redirectLimit: 5,
  
  // Re-evaluate redirects when authentication state changes
  refreshListenable: AuthService.authNotifier,
  
  // Add navigator observers for analytics
  observers: [
    AnalyticsObserver(),
  ],
);
```

### Code Breakdown

| Element | Purpose |
|---------|---------|
| `GlobalKey<NavigatorState>` | Enables navigation from anywhere (e.g., from a BLoC or service) |
| `kDebugMode` | Only enables verbose logging in development, not production |
| `errorBuilder` | Shows a custom 404 page for unknown routes |
| `redirect` | Authentication guard—redirects unauthenticated users to login |
| `refreshListenable` | Re-runs the redirect when auth state changes (e.g., after login/logout) |
| `observers` | Adds analytics tracking for every navigation event |

### Example 2: Configuration with Nested Routes

```dart
final GoRouter router = GoRouter(
  routes: <RouteBase>[
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
      routes: [
        // Child routes are displayed on top of the parent
        GoRoute(
          path: 'details',
          builder: (context, state) => const DetailsScreen(),
        ),
        GoRoute(
          path: 'settings',
          builder: (context, state) => const SettingsScreen(),
        ),
      ],
    ),
  ],
);
```

Accessing `/details` displays `DetailsScreen` **on top of** `HomeScreen`—just like `Navigator.push`.

---

## Common Use Cases

1. **Authentication flows**: Use `redirect` to guard protected routes
2. **Deep linking**: Define routes with path parameters (`/product/:id`)
3. **Tab navigation**: Use `ShellRoute` for persistent bottom navigation bars
4. **Feature toggles**: Use `redirect` to conditionally enable/disable routes
5. **Analytics**: Use `observers` to track navigation events
6. **Error monitoring**: Use `onException` to log errors to a service

---

## Best Practices

### 1. Centralize Your Route Configuration

**Why**: Keeping all routes in one place makes them easier to find, modify, and reason about.

**How**: Define your `GoRouter` in a dedicated file (e.g., `router.dart`) and export it.

```dart
// router.dart
final GoRouter router = GoRouter(
  routes: [
    // All routes defined here
  ],
);
```

### 2. Use `debugLogDiagnostics` in Development Only

**Why**: Verbose logging can clutter production logs and expose internal details.

**How**: Use `kDebugMode` to enable only in development.

```dart
debugLogDiagnostics: kDebugMode,
```

### 3. Always Define a Root Route (`/`)

**Why**: GoRouter requires a root route for initialization.

**How**: Always include `GoRoute(path: '/', builder: ...)` in your `routes` list.

### 4. Set a Reasonable `redirectLimit`

**Why**: Prevents infinite redirect loops from crashing your app.

**How**: Keep the default (`5`) or lower it to `3` for stricter detection.

### 5. Use `refreshListenable` for Reactive Redirects

**Why**: Ensures redirects are re-evaluated when dependencies (like auth state) change.

**How**: Pass a `Listenable` (e.g., a `ChangeNotifier`) that notifies when state changes.

### 6. Provide a Custom Error Builder

**Why**: The default error screen is generic and user-unfriendly.

**How**: Always provide a custom `errorBuilder` or `errorPageBuilder` for a polished user experience.

---

## Common Mistakes

### ❌ Mistake 1: Forgetting the Root Route

```dart
// WRONG: No route matches '/'
final router = GoRouter(
  routes: [
    GoRoute(path: '/login', builder: ...),
    GoRoute(path: '/home', builder: ...),
  ],
);
// Throws: 'The routes list must contain a route matching "/"'
```

**Why it's wrong**: GoRouter needs a root route (`/`) as the entry point.

**Correct**:
```dart
final router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: ...), // Root route first
    GoRoute(path: '/login', builder: ...),
    GoRoute(path: '/home', builder: ...),
  ],
);
```

### ❌ Mistake 2: Using `home:` or `routes:` with `MaterialApp.router`

```dart
// WRONG: Mixing old and new APIs
MaterialApp.router(
  routerConfig: router,
  home: const HomeScreen(), // ← Ignored or causes errors
  routes: { '/details': ... }, // ← Ignored
)
```

**Why it's wrong**: `MaterialApp.router` only uses `routerConfig`. All routes must be defined in the `GoRouter` configuration.

**Correct**:
```dart
MaterialApp.router(
  routerConfig: router, // All routes are inside the router
)
```

### ❌ Mistake 3: Performing Navigation Inside `redirect`

```dart
// WRONG: Navigation inside redirect
redirect: (context, state) {
  if (!isLoggedIn) {
    context.go('/login'); // ← Causes infinite loop
    return null;
  }
  return null;
}
```

**Why it's wrong**: The `redirect` function should **return** a new location, not perform navigation itself. Using `context.go` inside `redirect` creates an infinite loop.

**Correct**:
```dart
redirect: (context, state) {
  if (!isLoggedIn) {
    return '/login'; // ← Return the new location
  }
  return null; // ← Allow navigation to proceed
}
```

### ❌ Mistake 4: Not Handling the Default Case in `redirect`

```dart
// WRONG: No default return
redirect: (context, state) {
  if (!isLoggedIn) {
    return '/login';
  }
  // Missing return for logged-in case → returns null implicitly
}
```

**Why it's wrong**: If the condition is false, the function returns `null` implicitly, which is correct. However, being explicit is clearer.

**Correct**:
```dart
redirect: (context, state) {
  if (!isLoggedIn) {
    return '/login';
  }
  return null; // Explicitly allow navigation
}
```

---

## Performance Considerations

- **Route matching is O(n)** where n is the number of routes. For most apps (<100 routes), this is negligible.
- **`redirect` is called on every navigation**—keep it lightweight. Avoid expensive operations like network calls.
- **`refreshListenable` triggers re-evaluation** of redirects—ensure it doesn't notify too frequently.
- **Lazy loading**: Consider `go_router_deferred` for large apps to defer route loading.

---

## Security Considerations

- **Don't log sensitive information** in `debugLogDiagnostics`—it prints all navigation events including query parameters.
- **Use `redirect` for authentication**—never rely on client-side checks alone. Always validate on the server.
- **Sanitize path parameters** before using them—users can inject malicious strings via deep links.

---

## Debugging Tips

1. **Enable `debugLogDiagnostics: true`** to see every navigation event
2. **Check the `redirectLimit`**—if you're hitting it, you likely have a redirect loop
3. **Use `GoRouter.of(context).routerDelegate.currentConfiguration`** to debug the current route configuration
4. **Check browser developer tools** (on web) for navigation errors
5. **Add a `NavigatorObserver`** to log navigation events programmatically

---

## When to Use

- **Every GoRouter-powered app needs route configuration**—this is non-negotiable.
- **Use `redirect`** for authentication, feature flags, and conditional navigation.
- **Use `debugLogDiagnostics`** during development and debugging.
- **Use `errorBuilder`** for a polished error experience.
- **Use `navigatorKey`** when you need to navigate from outside the widget tree.

---

## When Not to Use

- **Don't overcomplicate your configuration**—if your app has 2-3 simple screens, a minimal configuration is fine.
- **Don't use `redirect` for logic that belongs in the UI**—use it for routing concerns only.
- **Don't use `navigatorKey`** if you can use `context`-based navigation—it adds complexity.

---

## Related Concepts

- `GoRoute`
- `ShellRoute`
- `GoRouterState`
- `redirect`
- `errorBuilder`
- `navigatorKey`

---

## Did You Know?

- The `routes` parameter accepts **any `RouteBase` subclass**, including `GoRoute`, `ShellRoute`, and `RedirectRoute`.
- GoRouter's `redirect` runs **before** route-level redirects—global redirects take precedence.
- The `redirectLimit` was increased from `5` to `20` in some versions, then reverted to `5` for safety.
- You can use `GoRouter.routingConfig` for **dynamic routing**—routes that change at runtime.

---

## Summary

- **Route configuration** is the process of defining your app's navigation structure in a `GoRouter` instance.
- **The `routes` parameter** is required and must contain a root route (`/`).
- **`initialLocation`** sets the starting point (defaults to `/`).
- **`debugLogDiagnostics`** enables verbose logging for debugging.
- **`errorBuilder`** provides a custom error screen for unknown routes or exceptions.
- **`redirect`** intercepts navigation for authentication and conditional routing.
- **`navigatorKey`** enables navigation from outside the widget tree.
- **Configuration is centralized**—all routing logic lives in one place.
- **Always define a root route** and provide a custom error builder for production apps.

---

