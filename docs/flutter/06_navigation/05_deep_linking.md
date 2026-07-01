# Deep Linking

Understand how to handle deep linking in Flutter applications.

---

# What is it?

Deep linking is the ability to launch a Flutter app and navigate to a specific screen or state using a URL. It allows users to access specific content directly, such as a product page, profile, or specific section of your app. Deep links can be triggered from web links, push notifications, QR codes, or other apps.

---

# Why does it exist?

Deep linking exists to:

- Direct users to specific content
- Improve user engagement
- Enable sharing of app content
- Support web-to-app transitions
- Handle push notification navigation
- Implement universal links
- Create seamless user experiences

---

# Deep Linking Basics

> **Deep linking** navigates to specific content.

```dart
// Basic deep linking setup
import 'package:go_router/go_router.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Deep Linking Demo',
      routerConfig: _router,
    );
  }
}

final _router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    // Product deep link: /product/123
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return ProductScreen(productId: int.parse(id));
      },
    ),
    // User profile deep link: /profile/456
    GoRoute(
      path: '/profile/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return ProfileScreen(userId: int.parse(id));
      },
    ),
    // Post deep link: /post/789
    GoRoute(
      path: '/post/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return PostScreen(postId: int.parse(id));
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
            const Text('Deep Linking Example'),
            const SizedBox(height: 16),
            // Simulate deep link navigation
            ElevatedButton(
              onPressed: () {
                context.push('/product/123');
              },
              child: const Text('Open Product 123'),
            ),
            ElevatedButton(
              onPressed: () {
                context.push('/profile/456');
              },
              child: const Text('Open Profile 456'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- GoRouter handles deep links
- Path parameters extract IDs
- Each screen receives specific data
- Direct navigation to content

---

# Platform Configuration

> **Configuring platforms** for deep linking.

```dart
// Android configuration (AndroidManifest.xml)
/*
<activity android:name=".MainActivity">
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="myapp" />
    <data android:host="example.com" />
    <data android:pathPrefix="/product" />
    <data android:pathPrefix="/profile" />
  </intent-filter>
</activity>
*/

// iOS configuration (Info.plist)
/*
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>myapp</string>
    </array>
  </dict>
</array>
<key>FlutterDeepLinkingEnabled</key>
<true/>
*/

// Web configuration (Web manifest)
/*
{
  "name": "My App",
  "short_name": "MyApp",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#ffffff",
  "background_color": "#ffffff",
  "scope": "/",
  "icons": [...]
}
*/
```

What's happening here?
- Android: Intent filters
- iOS: URL schemes
- Web: Manifest configuration
- Each platform requires specific setup

---

# Handling Deep Links

> **Processing** deep link data.

```dart
// Deep link handling with GoRouter
class DeepLinkHandler extends StatelessWidget {
  const DeepLinkHandler({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _deepLinkRouter,
    );
  }
}

final _deepLinkRouter = GoRouter(
  initialLocation: '/',
  // Parse deep link when app starts
  redirect: (context, state) {
    // Check for deep link data
    final uri = state.uri;
    if (uri.pathSegments.isNotEmpty) {
      // Handle specific deep links
      switch (uri.pathSegments.first) {
        case 'product':
          if (uri.pathSegments.length > 1) {
            return '/product/${uri.pathSegments[1]}';
          }
          break;
        case 'profile':
          if (uri.pathSegments.length > 1) {
            return '/profile/${uri.pathSegments[1]}';
          }
          break;
        case 'post':
          if (uri.pathSegments.length > 1) {
            return '/post/${uri.pathSegments[1]}';
          }
          break;
      }
    }
    return null;
  },
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return DeepLinkContentScreen(
          type: 'Product',
          id: int.parse(id),
        );
      },
    ),
    GoRoute(
      path: '/profile/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return DeepLinkContentScreen(
          type: 'Profile',
          id: int.parse(id),
        );
      },
    ),
    GoRoute(
      path: '/post/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return DeepLinkContentScreen(
          type: 'Post',
          id: int.parse(id),
        );
      },
    ),
  ],
);

class DeepLinkContentScreen extends StatelessWidget {
  const DeepLinkContentScreen({
    super.key,
    required this.type,
    required this.id,
  });

  final String type;
  final int id;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('$type Details'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              '$type ID: $id',
              style: const TextStyle(fontSize: 24),
            ),
            const SizedBox(height: 16),
            const Text('This content was opened via deep link'),
            const SizedBox(height: 16),
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
- redirect parses deep link
- Path segments determine destination
- Parameters extracted from path
- Content screen displays data

---

# Deep Link with Arguments

> **Passing complex data** through deep links.

```dart
// Deep links with arguments
class DeepLinkWithArgs extends StatelessWidget {
  const DeepLinkWithArgs({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _argsRouter,
    );
  }
}

final _argsRouter = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const ArgsHomeScreen(),
    ),
    // Multiple path parameters
    GoRoute(
      path: '/product/:id/:name',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        final name = state.pathParameters['name'] ?? 'Unknown';
        return ProductArgsScreen(
          id: int.parse(id),
          name: name,
        );
      },
    ),
    // Query parameters
    GoRoute(
      path: '/search',
      builder: (context, state) {
        final query = state.uri.queryParameters['q'] ?? '';
        final category = state.uri.queryParameters['category'] ?? '';
        return SearchArgsScreen(
          query: query,
          category: category,
        );
      },
    ),
    // Extra data
    GoRoute(
      path: '/checkout',
      builder: (context, state) {
        final data = state.extra as Map<String, dynamic>?;
        return CheckoutArgsScreen(
          items: data?['items'] ?? [],
          total: data?['total'] ?? 0.0,
        );
      },
    ),
  ],
);

class ArgsHomeScreen extends StatelessWidget {
  const ArgsHomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Deep Link Args')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Path params
            ElevatedButton(
              onPressed: () {
                context.push('/product/123/Laptop');
              },
              child: const Text('Product (Path Params)'),
            ),
            // Query params
            ElevatedButton(
              onPressed: () {
                context.push('/search?q=phone&category=electronics');
              },
              child: const Text('Search (Query Params)'),
            ),
            // Extra data
            ElevatedButton(
              onPressed: () {
                context.push(
                  '/checkout',
                  extra: {
                    'items': ['Item 1', 'Item 2'],
                    'total': 99.99,
                  },
                );
              },
              child: const Text('Checkout (Extra)'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Path params: /product/:id/:name
- Query params: ?q=search&category=...
- Extra: complex data objects
- Multiple ways to pass data

---

# Real-World Examples

> **Common deep linking patterns.**

```dart
// 1. E-commerce product sharing
class ProductDeepLinkApp extends StatelessWidget {
  const ProductDeepLinkApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _productRouter,
    );
  }
}

final _productRouter = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const EcommerceHomeScreen(),
    ),
    // Product detail with deep link
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return ProductDeepScreen(productId: int.parse(id));
      },
    ),
    // Category with filters
    GoRoute(
      path: '/category/:name',
      builder: (context, state) {
        final name = state.pathParameters['name'] ?? '';
        final sort = state.uri.queryParameters['sort'] ?? 'popular';
        return CategoryDeepScreen(
          categoryName: name,
          sortBy: sort,
        );
      },
    ),
  ],
);

class ProductDeepScreen extends StatelessWidget {
  const ProductDeepScreen({
    super.key,
    required this.productId,
  });

  final int productId;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Product $productId'),
        actions: [
          IconButton(
            icon: const Icon(Icons.share),
            onPressed: () {
              // Share deep link
              // final url = 'https://example.com/product/$productId';
            },
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.shopping_bag, size: 64),
            const SizedBox(height: 16),
            Text(
              'Product $productId',
              style: const TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              '\$${19.99 + productId * 10}',
              style: const TextStyle(
                fontSize: 20,
                color: Colors.blue,
              ),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                context.pop();
              },
              child: const Text('Back to Home'),
            ),
          ],
        ),
      ),
    );
  }
}

// 2. Social media post sharing
class SocialDeepLinkApp extends StatelessWidget {
  const SocialDeepLinkApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _socialRouter,
    );
  }
}

final _socialRouter = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const SocialHomeScreen(),
    ),
    // User profile deep link
    GoRoute(
      path: '/user/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return UserProfileDeepScreen(userId: int.parse(id));
      },
    ),
    // Post deep link
    GoRoute(
      path: '/post/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return PostDeepScreen(postId: int.parse(id));
      },
    ),
    // Notification deep link
    GoRoute(
      path: '/notification/:id',
      builder: (context, state) {
        final id = state.pathParameters['id'] ?? '0';
        return NotificationDeepScreen(notificationId: int.parse(id));
      },
    ),
  ],
);

class PostDeepScreen extends StatelessWidget {
  const PostDeepScreen({
    super.key,
    required this.postId,
  });

  final int postId;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Post'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.article, size: 64),
            const SizedBox(height: 16),
            Text(
              'Post #$postId',
              style: const TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            const Text(
              'Lorem ipsum dolor sit amet, consectetur adipiscing elit.',
              textAlign: TextAlign.center,
            ),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                IconButton(
                  icon: const Icon(Icons.favorite_border),
                  onPressed: () {},
                ),
                const Text('25'),
                const SizedBox(width: 16),
                IconButton(
                  icon: const Icon(Icons.comment_outlined),
                  onPressed: () {},
                ),
                const Text('8'),
                const SizedBox(width: 16),
                IconButton(
                  icon: const Icon(Icons.share),
                  onPressed: () {},
                ),
              ],
            ),
            ElevatedButton(
              onPressed: () {
                context.pop();
              },
              child: const Text('Back'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Product sharing with deep link
- Social media post sharing
- Notification deep linking
- Share button generates deep link

---

# Best Practices

## Use GoRouter for Deep Links

```dart
// Good - GoRouter handles deep links
GoRouter(
  routes: [
    GoRoute(
      path: '/product/:id',
      builder: (context, state) => ProductScreen(productId: state.pathParameters['id']),
    ),
  ],
)
```

## Handle Missing Data

```dart
// Good - Handle missing parameters
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = state.pathParameters['id'];
    if (id == null) {
      return const NotFoundScreen();
    }
    return ProductScreen(productId: int.parse(id));
  },
)
```

## Support Multiple Link Formats

```dart
// Good - Multiple link formats
GoRoute(
  path: '/product/:id',
  builder: (context, state) => ProductScreen(...),
),
GoRoute(
  path: '/p/:id',
  builder: (context, state) => ProductScreen(...),
),
```

---

# Common Mistakes

## Not Testing Deep Links

Wrong:
```dart
// Not testing deep links
// Build and test with different URLs
```

Correct:
```dart
// Test deep links
// Android: adb shell am start -a android.intent.action.VIEW -d "myapp://product/123"
// iOS: xcrun simctl openurl booted "myapp://product/123"
// Web: http://localhost:8080/#/product/123
```

## Missing Platform Configuration

Wrong:
```dart
// Only implementing app-side
// Forgot platform-specific configuration
```

Correct:
```dart
// Configure all platforms
// Android: AndroidManifest.xml
// iOS: Info.plist
// Web: manifest.json
```

---

# Summary

Deep linking allows users to navigate directly to specific content in your app. Use GoRouter for handling deep links, configure each platform appropriately, and handle all edge cases like missing data. Deep links improve user engagement and enable content sharing.

---

# Next Steps

- [Route Guards](route-guards.md)
- [Navigation Patterns](navigation-patterns.md)
- [State Management](../state-management/local-state.md)

---

# Did You Know?

- Deep links can be triggered from web, notifications, QR codes
- GoRouter handles deep links automatically
- Path parameters extract data from URLs
- Query parameters provide additional context
- Platform-specific configuration is required
- Deep links improve user engagement
- Deep links enable content sharing
- Deep links can carry complex data