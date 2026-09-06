# URL-based Navigation

## Overview

**URL-based navigation** is the paradigm where every screen in your application is addressable by a **Uniform Resource Locator (URL)** —just like pages on the web. Instead of pushing and popping screens by widget reference, you navigate by changing the URL. GoRouter provides a **convenient, URL-based API for navigating between different screens** across all platforms—mobile, web, and desktop.

**Purpose:**
- Make every screen in your app addressable via a unique URL
- Enable deep linking from external sources (email links, push notifications, QR codes)
- Synchronize the browser URL with the app state on the web
- Provide a consistent navigation API across all platforms
- Allow sharing of app state via simple URLs

**Where It Fits:**
URL-based navigation is the **core mechanism** of GoRouter. Rather than navigating to widgets (`Navigator.push(context, DetailsScreen())`), you navigate to **locations** (`context.go('/details/42')`). The URL becomes the source of truth for what's displayed, and the router handles the mapping from URL to widget tree.

---

## Why It Exists

### The Problem: Widget-Based Navigation Is Brittle

With Navigator 1.0, navigation is **widget-centric**:

```dart
// You navigate to a widget, not a location
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => DetailsScreen(id: 42)),
);
```

This approach has fundamental problems:

| Problem | Description |
|---------|-------------|
| **No external entry** | You can't open a specific screen from outside the app (email link, notification) |
| **No shareable state** | You can't share a link to a specific screen with another user |
| **No web support** | The browser URL doesn't reflect the current screen |
| **No bookmarking** | Users can't bookmark a specific screen and return to it later |
| **No back-button sync** | The browser back button doesn't work correctly on the web |
| **State is implicit** | The navigation stack is a side effect, not a declarative description |

### The Previous Limitation: URLs Were Manual and Fragile

Before GoRouter, supporting URL-based navigation meant:
1. **Manually parsing** incoming URLs in `main()` or in `initState()`
2. **Manually constructing** the navigation stack with multiple `Navigator.push()` calls
3. **Manually synchronizing** the browser URL with the app state
4. **Handling edge cases** like back navigation, refresh, and state restoration

This was **error-prone, tedious, and rarely done correctly**.

### Why GoRouter Introduced This

GoRouter makes URL-based navigation **automatic and declarative**:

> "The go_router package uses the Flutter framework's Router API to provide a convenient, URL-based API for navigating between different screens. You can define URL patterns, navigate using a URL, handle deep links, and a number of other navigation-related scenarios."

Instead of manually parsing URLs and building stacks, you simply:
1. **Define the route tree** with URL patterns (`/product/:id`)
2. **Navigate using URLs** (`context.go('/product/42')`)
3. **Let GoRouter handle the rest**—parsing, matching, stack building, and URL sync

---

## Syntax / Basic Usage

### Navigating with URLs

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

// 1. Define routes with URL patterns
final GoRouter _router = GoRouter(
  routes: [
    GoRoute(
      path: '/',                    // URL: /
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/products',            // URL: /products
      builder: (context, state) => const ProductsScreen(),
      routes: [
        GoRoute(
          path: ':id',              // URL: /products/123
          builder: (context, state) {
            final id = state.pathParameters['id']!;
            return ProductDetailScreen(id: id);
          },
        ),
      ],
    ),
    GoRoute(
      path: '/profile',             // URL: /profile?tab=settings
      builder: (context, state) {
        final tab = state.uri.queryParameters['tab'] ?? 'overview';
        return ProfileScreen(tab: tab);
      },
    ),
  ],
);

// 2. Navigate using URLs
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Navigate to a static URL
        ElevatedButton(
          onPressed: () => context.go('/products'),
          child: Text('View Products'),
        ),
        // Navigate with a path parameter
        ElevatedButton(
          onPressed: () => context.go('/products/42'),
          child: Text('View Product #42'),
        ),
        // Navigate with a query parameter
        ElevatedButton(
          onPressed: () => context.go('/profile?tab=settings'),
          child: Text('Profile Settings'),
        ),
        // Navigate using Uri class for complex URLs
        ElevatedButton(
          onPressed: () {
            final uri = Uri(
              path: '/products',
              queryParameters: {'category': 'electronics', 'sort': 'price'},
            );
            context.go(uri.toString());
          },
          child: Text('Filtered Products'),
        ),
      ],
    );
  }
}
```

### Code Breakdown

| Line/Block | Explanation |
|------------|-------------|
| `path: '/products/:id'` | Defines a URL pattern with a path parameter (`:id`). GoRouter matches this pattern against incoming URLs. |
| `state.pathParameters['id']` | Extracts the path parameter value from the URL. The parameter name must match the one defined in the path. |
| `state.uri.queryParameters['tab']` | Extracts query parameters from the URL (the part after `?`). |
| `context.go('/products/42')` | Navigates to a URL, replacing the entire navigation stack. |
| `Uri(path: ..., queryParameters: ...)` | Builds a URL with query parameters using Dart's standard `Uri` class. |

### The `go` vs `push` Distinction

| Method | Behavior | URL Effect |
|--------|----------|------------|
| **`context.go('/path')`** | Replaces the entire navigation stack to match the target URL | Updates the browser URL (on web) |
| **`context.push('/path')`** | Adds the target route on top of the existing stack | Does **not** update the browser URL (web) |

> **Critical**: `push` does **not** change the browser URL on the web. This is by design—`push` is for **imperative additions** within a flow, not for changing the URL location.

---

## Mental Model

### The URL as a "Coordinate System"

Think of URL-based navigation like a **coordinate system** for your app:

```
┌─────────────────────────────────────────────────────────────────┐
│                    APP COORDINATE SYSTEM                       │
│                                                                  │
│   /                              → Home (0,0)                   │
│   /products                      → Products list (1,0)          │
│   /products/42                   → Product #42 (1,1)            │
│   /products/42?view=detailed    → Product #42, detailed view   │
│   /profile                       → Profile (2,0)               │
│   /profile?tab=settings         → Profile, settings tab        │
│   /checkout                      → Checkout (3,0)              │
└─────────────────────────────────────────────────────────────────┘
```

Every screen is a **coordinate** in this system. Navigating is as simple as **changing coordinates**—the router handles the path.

### The "Web Browser" Analogy

URL-based navigation works **exactly like a web browser**:

| Browser | GoRouter |
|---------|----------|
| You type `example.com/products/42` in the address bar | You call `context.go('/products/42')` |
| The browser fetches and displays the page | GoRouter matches the route and builds the screen |
| The address bar shows the current location | The browser URL stays in sync (on web) |
| You can bookmark the URL | Users can share the URL (deep linking) |
| The back button navigates to the previous URL | The back button works automatically |

**The key insight**: URLs are **portable, shareable, and persistent**. They work the same way whether accessed from a browser, a mobile app, or a desktop application.

---

## Core Concepts

### 1. Path Parameters

Path parameters are **variables embedded in the URL path**, prefixed with `:`.

```
URL: /products/42
Pattern: /products/:id
Parameter: id = "42"
```

**Syntax**:
```dart
GoRoute(
  path: '/products/:id',  // ← :id is a path parameter
  builder: (context, state) {
    final id = state.pathParameters['id']!;  // ← Access the value
    return ProductScreen(id: id);
  },
)
```

Path parameters are **required**—the URL must include them to match the route.

### 2. Query Parameters

Query parameters are **key-value pairs after the `?`** in the URL.

```
URL: /profile?tab=settings&theme=dark
Query Parameters: tab="settings", theme="dark"
```

**Syntax**:
```dart
GoRoute(
  path: '/profile',
  builder: (context, state) {
    final tab = state.uri.queryParameters['tab'] ?? 'overview';
    final theme = state.uri.queryParameters['theme'] ?? 'light';
    return ProfileScreen(tab: tab, theme: theme);
  },
)
```

Query parameters are **optional**—they can be omitted.

### 3. URL Building

You can build URLs programmatically using Dart's `Uri` class:

```dart
// Simple URL
context.go('/products/42');

// URL with query parameters
final uri = Uri(
  path: '/products',
  queryParameters: {
    'category': 'electronics',
    'sort': 'price',
    'page': '2',
  },
);
context.go(uri.toString());  // → /products?category=electronics&sort=price&page=2
```

### 4. Extra Data (Non-URL Data)

Sometimes you need to pass data that **shouldn't appear in the URL** (e.g., large objects, sensitive data):

```dart
// Pass extra data
context.go('/profile', extra: userObject);

// Retrieve extra data
GoRoute(
  path: '/profile',
  builder: (context, state) {
    final user = state.extra as User;  // ← Access the extra data
    return ProfileScreen(user: user);
  },
)
```

> **Security Note**: Extra data is **not** URL-encoded and won't appear in the browser address bar.

---

## How It Works

### Step-by-Step: URL-Based Navigation

When you call `context.go('/products/42?view=detailed')`:

```
1. User triggers navigation → context.go('/products/42?view=detailed')
   │
   ▼
2. GoRouter parses the URL
   │   ├── Path: /products/42
   │   └── Query: view=detailed
   │
   ▼
3. GoRouter matches the path against the route tree
   │   ├── Finds GoRoute(path: '/products')
   │   │   └── Finds child GoRoute(path: ':id')
   │   └── Extracts path parameter: id = "42"
   │
   ▼
4. GoRouter extracts query parameters
   │   └── view = "detailed"
   │
   ▼
5. GoRouter builds the page stack
   │   ├── ProductsScreen (from parent route)
   │   └── ProductDetailScreen(id: "42") (from child route)
   │       └── Passes view="detailed" via query parameter
   │
   ▼
6. GoRouter updates the Navigator's pages list
   │
   ▼
7. Flutter reconciles the difference (animates transition)
   │
   ▼
8. UI shows ProductDetailScreen with id=42, detailed view
   │
   ▼
9. (On web) The browser URL updates to /products/42?view=detailed
```

### URL Matching Priority

Routes are matched **in the order they appear** in the `routes` list:

```dart
routes: [
  GoRoute(path: '/products', ...),        // ← Matches /products exactly
  GoRoute(path: '/:category', ...),       // ← Matches /anything (catch-all)
]
```

If a URL matches multiple routes, the **first match** wins.

---

## Internal Architecture / Under the Hood

### How GoRouter Maintains URL State

GoRouter acts as a **bridge** between the Flutter framework and the browser (or platform):

```
┌─────────────────────────────────────────────────────────────────┐
│                    PLATFORM LAYER                              │
│  ┌─────────────────┐    ┌─────────────────┐                   │
│  │  Browser URL    │    │  Deep Link URI  │                   │
│  │  (Web)          │    │  (Mobile)       │                   │
│  └────────┬────────┘    └────────┬────────┘                   │
│           │                      │                             │
│           └──────────┬───────────┘                             │
│                      ▼                                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           RouteInformationParser                        │   │
│  │  (Parses URLs/URIs into GoRouterState)                 │   │
│  └─────────────────────────────────────────────────────────┘   │
│                      │                                         │
│                      ▼                                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           GoRouter (The Router)                         │   │
│  │  • Matches URL against route tree                      │   │
│  │  • Extracts path and query parameters                  │   │
│  │  • Builds the page stack                               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                      │                                         │
│                      ▼                                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           RouterDelegate                                │   │
│  │  (Manages the Navigator's pages list)                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                      │                                         │
│                      ▼                                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           Navigator                                     │   │
│  │  (Displays the page stack)                             │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

GoRouter **synchronizes the state** between the platform (browser URL or deep link) and the app's navigation stack. When the URL changes externally (e.g., user clicks a link, presses the back button), GoRouter updates the app state. When the app navigates internally, GoRouter updates the URL.

---

## Lifecycle / Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                  URL-BASED NAVIGATION LIFECYCLE                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. NAVIGATION TRIGGERED                                        │
│     └── context.go('/product/42')                              │
│                                                                  │
│  2. URL PARSING                                                 │
│     └── Extract path: /product/42                              │
│         Extract query: (none)                                  │
│                                                                  │
│  3. ROUTE MATCHING                                              │
│     └── Match against route tree: /product/:id → id=42        │
│                                                                  │
│  4. PARAMETER EXTRACTION                                        │
│     └── pathParameters: {id: "42"}                             │
│         queryParameters: {}                                    │
│                                                                  │
│  5. PAGE STACK BUILDING                                         │
│     └── [HomeScreen, ProductScreen, ProductDetail(id=42)]     │
│                                                                  │
│  6. NAVIGATOR UPDATE                                            │
│     └── Navigator.pages = new page list                        │
│                                                                  │
│  7. UI RECONCILIATION                                           │
│     └── Flutter animates transition                            │
│                                                                  │
│  8. URL SYNC (WEB ONLY)                                         │
│     └── window.location.href = '/product/42'                   │
│                                                                  │
│  9. STATE PERSISTENCE (OPTIONAL)                                │
│     └── Save current URL for state restoration                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Practical Examples

### Example 1: Basic URL Navigation

```dart
// Define routes with URL patterns
final GoRouter router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/about',
      builder: (context, state) => const AboutScreen(),
    ),
  ],
);

// Navigate using URLs
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () => context.go('/about'),  // ← Navigate to /about
      child: Text('About'),
    );
  }
}
```

### Example 2: Path Parameters

```dart
// Define a route with a path parameter
GoRoute(
  path: '/user/:userId',
  builder: (context, state) {
    final userId = state.pathParameters['userId']!;
    return UserProfileScreen(userId: userId);
  },
)

// Navigate with a path parameter
context.go('/user/123');  // → userId = "123"
```

### Example 3: Query Parameters

```dart
// Define a route that reads query parameters
GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.uri.queryParameters['q'] ?? '';
    final limit = int.tryParse(state.uri.queryParameters['limit'] ?? '10') ?? 10;
    return SearchResultsScreen(query: query, limit: limit);
  },
)

// Navigate with query parameters
context.go('/search?q=flutter&limit=20');
// → query = "flutter", limit = 20
```

### Example 4: Building URLs with `Uri`

```dart
// Build a URL with multiple query parameters
final uri = Uri(
  path: '/search',
  queryParameters: {
    'q': 'flutter routing',
    'category': 'tutorials',
    'sort': 'relevance',
    'page': '2',
  },
);
context.go(uri.toString());
// → /search?q=flutter+routing&category=tutorials&sort=relevance&page=2
```

### Example 5: Passing Extra Data (Not in URL)

```dart
// Pass a complex object as extra data
final user = User(id: '123', name: 'Alice', email: 'alice@example.com');
context.go('/profile', extra: user);

// Retrieve the extra data in the route
GoRoute(
  path: '/profile',
  builder: (context, state) {
    final user = state.extra as User;  // ← Type cast to User
    return ProfileScreen(user: user);
  },
)
```

### Example 6: Deep Linking from External Source

```dart
// The app receives a deep link: myapp://product/42
// GoRouter automatically matches the route and displays ProductScreen

GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = state.pathParameters['id']!;
    return ProductScreen(id: id);
  },
)
// No additional code needed—deep linking works automatically!
```

---

## Common Use Cases

1. **Web Navigation**: Users expect the browser URL to reflect the current screen. GoRouter provides this automatically.
2. **Deep Linking**: Open the app from an external link (email, push notification, QR code) and navigate to the correct screen.
3. **Shareable State**: Users can share a link to a specific screen with others (e.g., `myapp.com/product/42`).
4. **Bookmarking**: Users can bookmark a specific screen and return to it later.
5. **State Restoration**: The app can restore its navigation state by reading the initial URL.
6. **Analytics**: Track which screens users visit by logging URL changes.

---

## Best Practices

### 1. Design URLs to Be Human-Readable

**Why**: URLs are visible to users (especially on web). Meaningful URLs improve UX and SEO.

**How**: Use descriptive paths, not cryptic identifiers.

```dart
// GOOD: Human-readable
path: '/products/electronics/laptops'

// BAD: Cryptic
path: '/p/el/lap'
```

### 2. Use Path Parameters for Resource Identification

**Why**: Path parameters identify **which** resource is being accessed. They are part of the URL structure.

**How**: Use `:paramName` in the path for IDs, slugs, or other identifiers.

```dart
// GOOD: Path parameter for resource ID
path: '/product/:productId'

// BAD: Using query parameter for resource ID
path: '/product'
// URL: /product?id=42  ← ID should be in the path
```

### 3. Use Query Parameters for Filtering and Options

**Why**: Query parameters are for **optional** data that modifies the view, not the resource itself.

**How**: Use query parameters for sorting, filtering, pagination, and UI state.

```dart
// GOOD: Query parameters for options
path: '/products'
// URL: /products?category=electronics&sort=price&page=2

// BAD: Path parameters for every option
path: '/products/:category/:sort/:page'  // ← Too many path parameters
```

### 4. Keep URLs Stable

**Why**: Changing URLs breaks deep links, bookmarks, and external references.

**How**: Design URLs carefully and avoid changing them. If you must change them, implement redirects.

### 5. Use `go` for Navigation, `push` for Drill-Down

**Why**: `go` changes the URL and represents a **new location**. `push` is for **temporary** screens within a flow.

**How**:
- Use `context.go()` for top-level navigation (tabs, home, profile)
- Use `context.push()` for drill-down (detail views, modals)

### 6. Validate and Sanitize Parameters

**Why**: Path and query parameters come from external sources (URLs, deep links). They could be malicious or malformed.

**How**: Validate and sanitize parameters before using them.

```dart
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    // Validate the ID before using it
    final id = state.pathParameters['id'];
    if (id == null || !RegExp(r'^[a-zA-Z0-9-]+$').hasMatch(id)) {
      return const NotFoundScreen();  // ← Handle invalid input
    }
    return ProductScreen(id: id);
  },
)
```

---

## Common Mistakes

### ❌ Mistake 1: Using `Navigator.push` Instead of `context.go`

```dart
// WRONG: Bypasses URL-based navigation
ElevatedButton(
  onPressed: () {
    Navigator.push(  // ← This doesn't update the URL!
      context,
      MaterialPageRoute(builder: (_) => DetailsScreen()),
    );
  },
  child: Text('Details'),
)
```

**Why it's wrong**: `Navigator.push` bypasses GoRouter entirely. The URL doesn't update, deep linking breaks, and the router loses track of the stack.

**Correct**:
```dart
ElevatedButton(
  onPressed: () => context.go('/details/42'),  // ← URL updates
  child: Text('Details'),
)
```

### ❌ Mistake 2: Using `push` When You Mean `go`

```dart
// WRONG: Using push for top-level navigation
context.push('/profile');  // ← URL does NOT update on web
```

**Why it's wrong**: `push` does not update the browser URL on the web. Users can't bookmark or share the profile screen.

**Correct**:
```dart
context.go('/profile');  // ← URL updates on web
```

### ❌ Mistake 3: Hard-Coding URLs Everywhere

```dart
// WRONG: Hard-coded URLs scattered everywhere
context.go('/products/42');
context.go('/products/99');
context.go('/products/123');
```

**Why it's wrong**: If the URL pattern changes, you must update it everywhere.

**Correct**: Define route constants:
```dart
// routes.dart
class Routes {
  static const String home = '/';
  static const String products = '/products';
  static String product(String id) => '/products/$id';
}

// Usage
context.go(Routes.product('42'));
```

### ❌ Mistake 4: Not Handling Missing Parameters

```dart
// WRONG: Assuming parameters always exist
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = state.pathParameters['id']!;  // ← Crashes if null!
    return ProductScreen(id: id);
  },
)
```

**Why it's wrong**: If the URL is malformed (e.g., `/product/` without an ID), the app crashes.

**Correct**:
```dart
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = state.pathParameters['id'];
    if (id == null) {
      return const NotFoundScreen();  // ← Handle gracefully
    }
    return ProductScreen(id: id);
  },
)
```

### ❌ Mistake 5: Using Query Parameters for Required Data

```dart
// WRONG: Required data in query parameters
path: '/product'
// URL: /product?id=42

// If the ID is missing, the screen can't function properly.
```

**Why it's wrong**: Query parameters are optional. Required data should be in the path.

**Correct**:
```dart
path: '/product/:id'  // ← Required data in the path
// URL: /product/42
```

---

## Performance Considerations

- **URL parsing is fast**: GoRouter's route matching is O(n) where n is the number of routes. For typical apps (<100 routes), this is negligible.
- **Avoid expensive operations in builders**: Builders are called on every navigation. Keep them lightweight.
- **Use `const` where possible**: For stateless screens, use `const` to enable widget caching.
- **Consider lazy loading**: For large apps, consider `go_router_deferred` to defer route loading.

---

## Security Considerations

- **Never trust URL parameters**: Validate and sanitize all path and query parameters.
- **Don't log sensitive data**: Query parameters may contain sensitive information (tokens, user IDs). Avoid logging full URLs.
- **Use HTTPS for deep links**: On mobile, configure deep links with HTTPS to prevent man-in-the-middle attacks.
- **Avoid passing sensitive data in URLs**: Use `extra` for sensitive data that shouldn't appear in the URL.

---

## Debugging Tips

1. **Enable `debugLogDiagnostics: true`** to see URL parsing and route matching in the console.
2. **Check the browser URL**: On web, verify that the URL updates correctly.
3. **Use `state.uri.toString()`** to debug the current URL:
   ```dart
   print('Current URL: ${state.uri.toString()}');
   ```
4. **Test deep links** using `flutter run` with a URL:
   ```bash
   flutter run -d chrome --web-port 8080
   # Then navigate to http://localhost:8080/#/product/42
   ```
5. **Use the `Uri` class** to debug URL building:
   ```dart
   final uri = Uri(path: '/products', queryParameters: {'q': 'test'});
   print(uri.toString());  // → /products?q=test
   ```

---

## When to Use

- **Every app with more than a few screens**: URL-based navigation scales better than widget-based navigation.
- **Flutter Web apps**: URL synchronization is essential.
- **Apps with deep linking**: URL-based navigation makes deep linking automatic.
- **Apps that need shareable state**: Users can share URLs to specific screens.
- **Apps that need state restoration**: The URL can be used to restore navigation state.

> **Most Flutter apps in 2026 should use URL-based navigation via GoRouter.**

---

## When Not to Use

- **Tiny apps with 1-2 screens**: The overhead of URL-based navigation isn't justified.
- **Apps that don't need deep linking or web support**: Navigator 1.0 may be simpler.
- **Apps with extremely custom navigation requirements**: GoRouter may not support all edge cases.

---

## Related Concepts

- `GoRouterState`
- `pathParameters`
- `queryParameters`
- `Uri`
- `extra` parameter
- `go` vs `push`
- Deep linking
- URL synchronization

---

## Did You Know?

- GoRouter matches paths in a **case-insensitive** way, but preserves the case for parameters.
- The `go` method is shorthand for `GoRouter.of(context).go()`.
- You can use the `Link` widget from the `url_launcher` package to create real HTML links on the web.
- Pages pushed with `Navigator.push` are **not deep-linkable** and will be replaced if a parent `go` occurs.
- GoRouter's URL-based navigation is inspired by **web frameworks** like React Router and Express.

---

## Summary

- **URL-based navigation** means every screen is addressable by a URL
- **Path parameters** (`/product/:id`) identify resources; they are **required**
- **Query parameters** (`?tab=settings`) modify the view; they are **optional**
- **`context.go()`** replaces the stack and updates the URL (declarative)
- **`context.push()`** adds a screen and does NOT update the URL (imperative)
- **Deep linking** works automatically—no additional code required
- **URL sync** on web is automatic—the browser URL stays in sync
- **Use meaningful, stable URLs** for shareability and SEO
- **Validate parameters** from external sources (URLs, deep links)
- **URL-based navigation is the recommended approach** for most Flutter apps

---
