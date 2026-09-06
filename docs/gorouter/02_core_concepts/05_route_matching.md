# Route Matching

## Overview

**Route matching** is the process by which GoRouter determines which route definition corresponds to a given URL. When the URL changes—whether through user navigation, a deep link, or the browser back button—GoRouter **parses the URL, matches it against the route tree, and builds the appropriate page stack** [0†L4-L7].

**Purpose:**
- Map every possible URL to a specific screen or set of screens
- Extract path and query parameters from the URL
- Build the correct navigation stack for the matched route
- Enable deep linking, web URL synchronization, and bookmarkable states

**Where It Fits:**
Route matching sits at the **heart** of GoRouter's declarative routing system. It is the mechanism that transforms a URL string—such as `/products/electronics/laptops?sort=price`—into a concrete widget tree. It operates automatically whenever navigation occurs, whether triggered by `context.go()`, a deep link, or a browser history event.

---

## Why It Exists

### The Problem: Manual URL Parsing Is Error-Prone

Before GoRouter, handling URLs required **manual, fragile code**:

```dart
// Typical manual URL parsing before GoRouter
void handleDeepLink(String url) {
  final uri = Uri.parse(url);
  final segments = uri.pathSegments; // ['products', 'electronics', 'laptops']
  
  if (segments.isNotEmpty) {
    switch (segments[0]) {
      case 'products':
        if (segments.length == 1) {
          Navigator.push(context, ProductsScreen());
        } else if (segments.length == 2) {
          Navigator.push(context, CategoryScreen(segments[1]));
        } else if (segments.length == 3) {
          Navigator.push(context, ProductListScreen(segments[1], segments[2]));
        }
        break;
      // ... and so on for every possible path
    }
  }
}
```

This approach has severe limitations:

| Problem | Description |
|---------|-------------|
| **Brittle** | Every path structure must be hard-coded; adding a new route requires updating the parser |
| **Error-Prone** | Off-by-one errors, missing segments, and incorrect type conversions are common |
| **No Hierarchy** | Manual parsing cannot easily represent nested navigation (parent-child relationships) |
| **No Parameter Extraction** | Path parameters like `/user/:id` require manual parsing and validation |
| **Duplicated Logic** | Deep linking, web navigation, and in-app navigation all require separate handling |
| **No Fallback** | Unknown URLs often crash the app or show a blank screen |

### The Previous Limitation: Navigator 2.0's Raw Matching Was Complex

Flutter's Navigator 2.0 provided the foundation for route matching through `RouteInformationParser`, but implementing it required:

1. Defining a custom data type to represent the parsed route
2. Implementing `RouteInformationParser.parseRouteInformation()` to parse URLs
3. Implementing `RouteInformationParser.restoreRouteInformation()` to serialize back
4. Manually extracting path segments and parameters
5. Handling edge cases like trailing slashes, case sensitivity, and malformed URLs

This was **hundreds of lines of boilerplate** for even simple apps.

### Why GoRouter Introduced This

GoRouter provides a **declarative, automatic route matching system**:

> "When the URL changes, it is matched against each route path. The path is matched in a case-insensitive way, but the case for parameters is preserved." [0†L18-L21]

Instead of writing manual parsing code, you simply:
1. **Define your route tree** with path patterns (`/products/:id`)
2. **Let GoRouter match** incoming URLs against these patterns
3. **Access extracted parameters** via `state.pathParameters` and `state.queryParameters`

---

## Syntax / Basic Usage

### Defining Routes for Matching

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

/// The route tree defines all possible URL patterns.
/// GoRouter matches incoming URLs against these patterns.
final GoRouter _router = GoRouter(
  routes: [
    // Exact match: URL must be exactly '/'
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    
    // Static path: URL must be exactly '/about'
    GoRoute(
      path: '/about',
      builder: (context, state) => const AboutScreen(),
    ),
    
    // Path parameter: matches '/products/anything'
    // Extracts 'id' from the URL
    GoRoute(
      path: '/products/:id',
      builder: (context, state) {
        // The router automatically extracts :id from the URL
        final id = state.pathParameters['id']!;
        return ProductScreen(id: id);
      },
    ),
    
    // Nested route: matches '/products/42/reviews'
    // Parent matches '/products/:id', child matches 'reviews'
    GoRoute(
      path: '/products/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductScreen(id: id);
      },
      routes: [
        GoRoute(
          path: 'reviews',  // Child path is relative
          builder: (context, state) {
            final id = state.pathParameters['id']!;
            return ReviewsScreen(productId: id);
          },
        ),
      ],
    ),
    
    // Query parameters: matched via state.uri.queryParameters
    GoRoute(
      path: '/search',
      builder: (context, state) {
        final query = state.uri.queryParameters['q'] ?? '';
        final page = int.tryParse(state.uri.queryParameters['page'] ?? '1') ?? 1;
        return SearchScreen(query: query, page: page);
      },
    ),
  ],
);

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,
      title: 'Route Matching Demo',
    );
  }
}
```

### Code Breakdown

| Line/Block | Explanation |
|------------|-------------|
| `GoRoute(path: '/', ...)` | Exact match—only the root URL matches this route |
| `GoRoute(path: '/about', ...)` | Static match—only `/about` matches this route |
| `GoRoute(path: '/products/:id', ...)` | **Path parameter**—matches `/products/anything`. The `:id` segment is a variable. |
| `state.pathParameters['id']` | Access the extracted path parameter value. The router automatically parses it from the URL. |
| `routes: [GoRoute(path: 'reviews', ...)]` | **Nested matching**—the child route matches `/products/:id/reviews`. The parent's `:id` is inherited. |
| `state.uri.queryParameters['q']` | Access query parameters (the part after `?` in the URL). |

### How Routes Are Matched

```
URL: /products/42/reviews?sort=newest

Step 1: GoRouter parses the URL
        ├── Path: /products/42/reviews
        └── Query: sort=newest

Step 2: GoRouter matches against the route tree
        ├── Matches GoRoute(path: '/products/:id')
        │   └── Extracts: id = "42"
        └── Matches child GoRoute(path: 'reviews')
            └── Inherits: id = "42"

Step 3: GoRouter extracts query parameters
        └── sort = "newest"

Step 4: GoRouter builds the page stack
        ├── ProductScreen(id: "42")
        └── ReviewsScreen(productId: "42", sort: "newest")
```

### Code Breakdown

| Step | Explanation |
|------|-------------|
| **Parse** | The URL is split into path (`/products/42/reviews`) and query (`sort=newest`) |
| **Match** | The path is matched against the route tree, segment by segment |
| **Extract** | Path parameters (`:id`) are extracted from the matched segments |
| **Build** | Each matched route's builder is called, producing the page stack |

---

## Mental Model

### The Route Tree as a Decision Tree

Think of the route tree as a **decision tree** that GoRouter traverses to find a match:

```
                          URL: /products/42/reviews
                                     │
                                     ▼
                    ┌─────────────────────────────────┐
                    │       Route Tree                │
                    │                                 │
                    │         [ / ]                   │
                    │      (HomeScreen)               │
                    │         │                       │
                    │    ┌────┴────┐                  │
                    │    │         │                  │
                    │ [ /about ] [ /products/:id ]    │
                    │ (About)    (ProductScreen)      │
                    │                 │               │
                    │            ┌────┴────┐          │
                    │            │         │          │
                    │         [ reviews ] [ /edit ]   │
                    │         (Reviews)   (Edit)      │
                    │                                 │
                    └─────────────────────────────────┘
                                     │
                                     ▼
                    ┌─────────────────────────────────┐
                    │       Matched Path              │
                    │  /products/:id → /products/42   │
                    │  /products/:id/reviews          │
                    │  → /products/42/reviews         │
                    └─────────────────────────────────┘
```

The matching algorithm:
1. Starts at the **root** (`/`)
2. Compares the **next URL segment** against each child route's path
3. If a path parameter (`:id`) is found, it **matches any segment**
4. Continues until the entire URL is consumed or no match is found
5. If no match is found, the `errorBuilder` is called

### The "Wildcard vs. Exact Match" Analogy

Think of route matching like **file globbing** in a terminal:

| Pattern | Matches | Does Not Match |
|---------|---------|----------------|
| `/` | `/` | `/about`, `/products` |
| `/about` | `/about` | `/`, `/about/team` |
| `/products/:id` | `/products/42`, `/products/abc` | `/products`, `/products/42/reviews` |
| `/products/:id/reviews` | `/products/42/reviews` | `/products/42`, `/products/42/reviews/new` |

Path parameters (`:id`) act like **wildcards**—they match any value in that segment.

---

## Core Concepts

### 1. Exact Matching

A route with a **static path** (no parameters) matches only the **exact URL**:

```dart
GoRoute(path: '/about', ...)
// Matches: /about
// Does NOT match: /about/team, /aboutus, /About
```

### 2. Path Parameter Matching

A route with a **path parameter** (`:param`) matches any value in that segment:

```dart
GoRoute(path: '/users/:userId', ...)
// Matches: /users/123, /users/alice, /users/abc-123
// Extracts: userId = "123" (or "alice", or "abc-123")
```

Path parameters are **required**—the URL must include a value for that segment.

### 3. Nested Matching

Child routes are matched **after** their parent:

```dart
GoRoute(
  path: '/products/:id',
  routes: [
    GoRoute(path: 'reviews', ...),  // Matches /products/:id/reviews
    GoRoute(path: 'edit', ...),     // Matches /products/:id/edit
  ],
)
```

Child routes **inherit** the parent's path parameters.

### 4. Case-Insensitive Matching

Route paths are matched in a **case-insensitive** way, but the **case for parameters is preserved** [0†L16-L17][1†L5-L6]:

```dart
GoRoute(path: '/products/:id', ...)
// /products/42    → Matches, id = "42"
// /PRODUCTS/42    → Matches, id = "42"  (case-insensitive)
// /Products/42    → Matches, id = "42"
// /products/AbC   → Matches, id = "AbC"  (case preserved for parameter)
```

### 5. Matching Priority

If there are **multiple route matches**, the **first match in the list takes priority** [0†L20-L21][6†L26-L27][8†L25-L26]:

```dart
routes: [
  GoRoute(path: '/products/:id', ...),  // ← Matches /products/42
  GoRoute(path: '/products/featured', ...),  // ← Never matches!
]
```

Since `/products/:id` is listed first, it matches `/products/featured` (where `id = "featured"`). The more specific route `/products/featured` is never reached.

**Solution**: List more specific routes **first**:

```dart
routes: [
  GoRoute(path: '/products/featured', ...),  // ← More specific first
  GoRoute(path: '/products/:id', ...),       // ← Catch-all second
]
```

### 6. Wildcard Matching (`*`)

GoRouter supports **wildcard matching** using `*` [4†L5-L8][4†L10-L13]:

```dart
GoRoute(path: '/docs/*', ...)
// Matches: /docs/anything, /docs/any/path/here
// Does NOT match: /docs (must have at least one segment after /docs/)
```

The wildcard matches **any number of remaining segments**. It should be the **last segment** in the path.

---

## How It Works

### Step-by-Step: The Matching Algorithm

When a navigation occurs (via `context.go()`, deep link, or browser back):

```
1. URL received: /products/42/reviews?sort=newest
   │
   ▼
2. URL is parsed into:
   ├── Path segments: ["products", "42", "reviews"]
   └── Query parameters: {"sort": "newest"}
   │
   ▼
3. Matching starts at the root route list
   │   ├── For each top-level route in order:
   │   │   ├── Check if the route's path matches the first segment(s)
   │   │   ├── If path parameter (:id), match any value
   │   │   └── If match found, continue to child routes
   │   └── Continue until URL is fully consumed or no match found
   │
   ▼
4. Match found: /products/:id → /products/42
   │   └── Extract path parameter: id = "42"
   │
   ▼
5. Check child routes of the matched route
   │   ├── Match: reviews → /products/42/reviews
   │   └── Extract inherited parameter: id = "42"
   │
   ▼
6. Build the page stack:
   ├── ProductScreen(id: "42") (from parent route)
   └── ReviewsScreen(productId: "42", sort: "newest") (from child route)
   │
   ▼
7. Update the Navigator with the new page stack
```

### Matching Rules Summary

| Rule | Description |
|------|-------------|
| **Top-down** | Routes are matched from the root down through nested children |
| **Order matters** | The first matching route in the list wins [0†L20-L21] |
| **Case-insensitive** | Paths are matched case-insensitively [1†L5-L6] |
| **Parameter preservation** | Parameter values preserve their original case [0†L16-L17] |
| **Segment-by-segment** | Each path segment is matched independently |
| **Wildcard (`*`)** | Matches any number of remaining segments [4†L5-L8] |
| **No partial matches** | The entire URL must be matched [1†L32-L34] |

---

## Internal Architecture / Under the Hood

### How GoRouter Implements Route Matching

GoRouter's matching engine is built on a **tree structure** [1†L9-L10][1†L21-L22]:

```
┌─────────────────────────────────────────────────────────────────┐
│                    ROUTE MATCHING ENGINE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Route Tree (Internal Representation)                     │ │
│  │                                                           │ │
│  │  RootNode                                                  │ │
│  │    ├── RouteNode(path: '/')                               │ │
│  │    │   └── builder: HomeScreen                            │ │
│  │    ├── RouteNode(path: '/products/:id')                  │ │
│  │    │   ├── builder: ProductScreen                        │ │
│  │    │   └── ChildNodes:                                   │ │
│  │    │       ├── RouteNode(path: 'reviews')               │ │
│  │    │       │   └── builder: ReviewsScreen               │ │
│  │    │       └── RouteNode(path: 'edit')                  │ │
│  │    │           └── builder: EditScreen                  │ │
│  │    └── RouteNode(path: '/about')                        │ │
│  │        └── builder: AboutScreen                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Matching Process                                          │ │
│  │                                                           │ │
│  │  1. Parse URL into segments                               │ │
│  │  2. Traverse tree depth-first                             │ │
│  │  3. Match segment against node's path pattern             │ │
│  │  4. Extract parameters when pattern has :param           │ │
│  │  5. Return matched nodes and extracted parameters        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Result: RouteMatchList                                    │ │
│  │                                                           │ │
│  │  [                                                         │ │
│  │    RouteMatch(route: /products/:id, params: {id: 42}),   │ │
│  │    RouteMatch(route: reviews, params: {id: 42}),         │ │
│  │  ]                                                         │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

The internal matching algorithm:
1. **Parses** the URL into path segments and query parameters
2. **Traverses** the route tree depth-first, matching each segment against each node's path pattern
3. **Extracts** path parameters when a pattern contains `:param`
4. **Returns** a `RouteMatchList` containing all matched routes and their parameters

---

## Lifecycle / Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                  ROUTE MATCHING LIFECYCLE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. NAVIGATION TRIGGERED                                        │
│     └── context.go('/products/42/reviews?sort=newest')         │
│                                                                  │
│  2. URL PARSING                                                 │
│     ├── Path: /products/42/reviews                             │
│     └── Query: sort=newest                                     │
│                                                                  │
│  3. ROUTE TREE TRAVERSAL                                        │
│     ├── Check route '/products/:id' → Match!                  │
│     │   └── Extract: id = "42"                                │
│     ├── Check child 'reviews' → Match!                        │
│     │   └── Inherit: id = "42"                                │
│     └── No more segments → Complete match                     │
│                                                                  │
│  4. PARAMETER EXTRACTION                                        │
│     ├── pathParameters: {id: "42"}                             │
│     └── queryParameters: {sort: "newest"}                     │
│                                                                  │
│  5. PAGE STACK BUILDING                                         │
│     ├── ProductScreen(id: "42")                                │
│     └── ReviewsScreen(productId: "42", sort: "newest")        │
│                                                                  │
│  6. NAVIGATOR UPDATE                                            │
│     └── Navigator.pages = new page stack                      │
│                                                                  │
│  7. UI RECONCILIATION                                           │
│     └── Flutter animates the transition                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Practical Examples

### Example 1: Basic Route Matching

```dart
final GoRouter router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: (_, __) => const HomeScreen()),
    GoRoute(path: '/about', builder: (_, __) => const AboutScreen()),
    GoRoute(path: '/contact', builder: (_, __) => const ContactScreen()),
  ],
);

// URL: / → Matches '/'
// URL: /about → Matches '/about'
// URL: /contact → Matches '/contact'
// URL: /unknown → No match → errorBuilder called
```

### Example 2: Path Parameters

```dart
GoRoute(
  path: '/users/:userId/posts/:postId',
  builder: (context, state) {
    final userId = state.pathParameters['userId']!;
    final postId = state.pathParameters['postId']!;
    return PostScreen(userId: userId, postId: postId);
  },
)

// URL: /users/123/posts/456
// → userId = "123", postId = "456"

// URL: /users/alice/posts/hello-world
// → userId = "alice", postId = "hello-world"
```

### Example 3: Query Parameters

```dart
GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.uri.queryParameters['q'] ?? '';
    final category = state.uri.queryParameters['category'];
    final page = int.tryParse(state.uri.queryParameters['page'] ?? '1') ?? 1;
    return SearchScreen(query: query, category: category, page: page);
  },
)

// URL: /search?q=flutter&category=tutorials&page=2
// → query = "flutter", category = "tutorials", page = 2

// URL: /search?q=dart
// → query = "dart", category = null, page = 1 (defaults)
```

### Example 4: Wildcard Matching

```dart
GoRoute(
  path: '/docs/*',
  builder: (context, state) {
    // The wildcard matches any number of remaining segments
    // Access the full remaining path via state.uri.path
    final fullPath = state.uri.path;  // e.g., "/docs/getting-started/installation"
    return DocsScreen(path: fullPath);
  },
)

// URL: /docs/getting-started → Matches
// URL: /docs/getting-started/installation → Matches
// URL: /docs → Does NOT match (needs at least one segment after /docs/)
```

### Example 5: Matching Priority

```dart
// WRONG ORDER: Catch-all matches before specific routes
final router = GoRouter(
  routes: [
    GoRoute(path: '/:category', ...),        // ← Matches ANY single segment
    GoRoute(path: '/products', ...),         // ← Never matches!
    GoRoute(path: '/products/featured', ...),// ← Never matches!
  ],
);
// /products → Matches '/:category' with category="products"
// /products/featured → No match (':category' only matches one segment)

// CORRECT ORDER: Specific routes first
final router = GoRouter(
  routes: [
    GoRoute(path: '/products/featured', ...),// ← Specific first
    GoRoute(path: '/products', ...),         // ← Then less specific
    GoRoute(path: '/:category', ...),        // ← Catch-all last
  ],
);
// /products/featured → Matches '/products/featured'
// /products → Matches '/products'
// /anything → Matches '/:category' with category="anything"
```

### Example 6: Checking if a Route Exists

You can check if a route exists using `context.matchingRoute()` [5†L8-L10]:

```dart
// Check if a route exists before navigating
final route = context.matchingRoute('/products/42');
if (route != null) {
  context.go('/products/42');  // Route exists, navigate
} else {
  // Route doesn't exist, handle gracefully
  ScaffoldMessenger.of(context).showSnackBar(
    const SnackBar(content: Text('Page not found')),
  );
}
```

---

## Common Use Cases

1. **Deep Linking**: Automatically matching incoming URLs to the correct screen
2. **Web URL Synchronization**: Keeping the browser URL in sync with the app state
3. **Parameter Extraction**: Parsing IDs, slugs, and other data from the URL
4. **Nested Navigation**: Building page stacks from nested route matches
5. **404 Handling**: Displaying a not-found screen when no route matches
6. **Dynamic Routing**: Using path parameters to handle variable URLs

---

## Best Practices

### 1. Order Routes from Most Specific to Least Specific

**Why**: GoRouter matches routes in the order they appear in the list [0†L20-L21]. A catch-all route will match everything if placed first.

**How**: List static routes first, then routes with path parameters, then wildcards.

```dart
// GOOD: Specific to general
routes: [
  GoRoute(path: '/products/featured', ...),  // Most specific
  GoRoute(path: '/products/:id', ...),       // Then parameterized
  GoRoute(path: '/:category', ...),          // Least specific (catch-all)
]

// BAD: General to specific
routes: [
  GoRoute(path: '/:category', ...),          // Matches EVERYTHING first
  GoRoute(path: '/products/featured', ...),  // Never reached
]
```

### 2. Use Path Parameters for Resource Identification

**Why**: Path parameters identify **which** resource is being accessed. They are part of the URL structure.

**How**: Use `:paramName` for IDs, slugs, or other identifiers.

```dart
// GOOD: Path parameter for ID
path: '/products/:productId'

// BAD: ID in query parameter
path: '/products'  // URL: /products?id=42
```

### 3. Use Query Parameters for Filtering and Options

**Why**: Query parameters are for **optional** data that modifies the view.

**How**: Use query parameters for sorting, filtering, pagination, and UI state.

```dart
// GOOD: Query parameters for options
path: '/products'
// URL: /products?category=electronics&sort=price&page=2

// BAD: Path parameters for every option
path: '/products/:category/:sort/:page'  // Too many path parameters
```

### 4. Validate Extracted Parameters

**Why**: Path and query parameters come from external sources (URLs, deep links) and could be malformed or malicious.

**How**: Validate and sanitize parameters before using them.

```dart
GoRoute(
  path: '/products/:id',
  builder: (context, state) {
    final id = state.pathParameters['id'];
    // Validate the ID format
    if (id == null || !RegExp(r'^[a-zA-Z0-9-]+$').hasMatch(id)) {
      return const NotFoundScreen();
    }
    return ProductScreen(id: id);
  },
)
```

### 5. Handle No-Match Scenarios

**Why**: Users may navigate to URLs that don't exist. A graceful error screen improves UX.

**How**: Always provide an `errorBuilder` in your `GoRouter` configuration.

```dart
final router = GoRouter(
  routes: [...],
  errorBuilder: (context, state) => const NotFoundScreen(),
);
```

---

## Common Mistakes

### ❌ Mistake 1: Incorrect Route Order

```dart
// WRONG: Catch-all before specific routes
routes: [
  GoRoute(path: '/:id', ...),           // Matches ANY single segment
  GoRoute(path: '/products', ...),      // Never matches!
  GoRoute(path: '/about', ...),         // Never matches!
]
```

**Why it's wrong**: The catch-all route `/:id` matches **any** single-segment URL, including `/products` and `/about`. More specific routes are never reached.

**Correct**:
```dart
routes: [
  GoRoute(path: '/products', ...),  // Specific first
  GoRoute(path: '/about', ...),     // Specific first
  GoRoute(path: '/:id', ...),       // Catch-all last
]
```

### ❌ Mistake 2: Forgetting the Root Route

```dart
// WRONG: No route matches '/'
routes: [
  GoRoute(path: '/products', ...),
  GoRoute(path: '/about', ...),
]
// Navigating to '/' results in no match → errorBuilder called
```

**Why it's wrong**: The app's entry point is `/`. Without a root route, the app has no default screen.

**Correct**:
```dart
routes: [
  GoRoute(path: '/', ...),        // Root route first
  GoRoute(path: '/products', ...),
  GoRoute(path: '/about', ...),
]
```

### ❌ Mistake 3: Assuming Parameters Always Exist

```dart
// WRONG: Using ! without checking
GoRoute(
  path: '/user/:id',
  builder: (context, state) {
    final id = state.pathParameters['id']!;  // ← Crashes if null!
    return UserScreen(id: id);
  },
)
```

**Why it's wrong**: If the URL is malformed (e.g., `/user/` without an ID), the app crashes.

**Correct**:
```dart
GoRoute(
  path: '/user/:id',
  builder: (context, state) {
    final id = state.pathParameters['id'];
    if (id == null) {
      return const NotFoundScreen();  // Handle gracefully
    }
    return UserScreen(id: id);
  },
)
```

### ❌ Mistake 4: Using `push` When You Mean `go`

```dart
// WRONG: Using push for top-level navigation
context.push('/products/42');  // URL does NOT update on web
```

**Why it's wrong**: `push` does not update the browser URL on the web. Users can't bookmark or share the product screen.

**Correct**:
```dart
context.go('/products/42');  // URL updates on web
```

### ❌ Mistake 5: Overlapping Route Patterns

```dart
// WRONG: Overlapping patterns
routes: [
  GoRoute(path: '/products/:category', ...),  // Matches /products/electronics
  GoRoute(path: '/products/featured', ...),   // Never matches (overlaps)
]
```

**Why it's wrong**: The first route `/:category` matches `featured` as a category, so the specific `/products/featured` route is never reached.

**Correct**:
```dart
routes: [
  GoRoute(path: '/products/featured', ...),   // Specific first
  GoRoute(path: '/products/:category', ...),  // Catch-all second
]
```

---

## Performance Considerations

- **Route matching is O(d × n)** where `d` is the depth of the route tree and `n` is the number of routes at each level. For typical apps (<100 routes), this is negligible.
- **Path parameter extraction** is fast—it's just string manipulation.
- **Avoid expensive operations in builders**: Builders are called on every navigation. Keep them lightweight.
- **Wildcard routes (`*`)** are slightly slower than static routes because they must match any number of segments.

---

## Security Considerations

- **Never trust URL parameters**: Validate and sanitize all path and query parameters.
- **Don't log sensitive data**: Query parameters may contain sensitive information (tokens, user IDs). Avoid logging full URLs.
- **Use HTTPS for deep links**: On mobile, configure deep links with HTTPS to prevent man-in-the-middle attacks.
- **Avoid passing sensitive data in URLs**: Use `extra` for sensitive data that shouldn't appear in the URL.

---

## Debugging Tips

1. **Enable `debugLogDiagnostics: true`** to see route matching in the console:
   ```dart
   final router = GoRouter(
     debugLogDiagnostics: true,  // ← Logs all route matching
     routes: [...],
   );
   ```

2. **Check the matched route** using `context.matchingRoute()`:
   ```dart
   final route = context.matchingRoute('/products/42');
   print('Matched route: ${route?.path}');
   ```

3. **Print the current state**:
   ```dart
   final state = GoRouter.of(context).routerDelegate.currentConfiguration;
   print('Current location: ${state.uri}');
   print('Path parameters: ${state.pathParameters}');
   ```

4. **Test deep links** using `flutter run` with a URL:
   ```bash
   flutter run -d chrome --web-port 8080
   # Then navigate to http://localhost:8080/#/product/42
   ```

5. **Check for overlapping routes**: If a route is never matched, check if an earlier route with a path parameter or wildcard is matching it instead.

---

## When to Use

- **Every GoRouter-powered app** uses route matching—it's automatic and unavoidable
- **When defining routes**—understanding matching helps you design correct route trees
- **When debugging navigation**—knowing how matching works helps identify why a route isn't being matched
- **When implementing deep linking**—route matching is what makes deep links work

---

## When Not to Use

- Route matching is **automatic**—you don't manually invoke it
- Don't try to **bypass** route matching by using `Navigator.push`—it breaks URL synchronization

---

## Related Concepts

- `GoRoute`
- `GoRouterState`
- `pathParameters`
- `queryParameters`
- `errorBuilder`
- `redirect`
- `ShellRoute`
- `RouteMatchList`

---

## Did You Know?

- GoRouter's route matching is **case-insensitive** for the path, but **case-preserving** for parameters [0†L16-L17][1†L5-L6].
- The matching algorithm uses a **tree structure**, not a flat list [1†L9-L10][1†L21-L22].
- Wildcards (`*`) match **any number** of remaining segments [4†L5-L8][4†L10-L13].
- You can check if a route exists using `context.matchingRoute()` [5†L8-L10].
- The `RouteMatchList` is itself a **tree structure**, not just a flat list [1†L21-L22].
- In GoRouter 5.0+, redirects are called from **top-most route to last**, not from last to first [2†L17-L19].

---

## Summary

- **Route matching** is the process of mapping a URL to a route definition
- **Paths are matched case-insensitively**, but parameter values preserve case
- **Order matters**—the first matching route in the list wins
- **Path parameters** (`:id`) match any value in that segment
- **Query parameters** are accessed via `state.uri.queryParameters`
- **Wildcards** (`*`) match any number of remaining segments
- **Nested routes** are matched after their parent
- **Always validate** extracted parameters from external sources
- **Enable `debugLogDiagnostics`** to debug matching issues
- **List specific routes before catch-all routes** to ensure correct matching

---

