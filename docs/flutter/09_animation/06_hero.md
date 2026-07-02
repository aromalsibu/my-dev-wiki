# Hero

Understand how to create smooth transitions between screens using Hero animations in Flutter.

---

# What is it?

Hero is a widget that creates seamless, shared-element transitions between two screens. When you navigate from one screen to another, a Hero widget "flies" from its position on the first screen to its position on the second screen, creating a smooth, visually appealing transition. This is commonly used for images, avatars, and other visual elements that appear on both screens.

---

# Why does it exist?

Hero exists to:

- Create smooth, engaging screen transitions
- Guide the user's attention between screens
- Provide visual continuity in navigation
- Enhance user experience with polished animations
- Support shared-element transitions
- Make navigation feel fluid and natural

---

# Basic Hero

> **Creating a simple** Hero transition.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic Hero example
class BasicHeroExample extends StatelessWidget {
  const BasicHeroExample({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const HeroHomeScreen(),
    );
  }
}

/// First screen with Hero widget
class HeroHomeScreen extends StatelessWidget {
  const HeroHomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Hero Example'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Hero widget with a tag
            // The tag identifies this Hero and connects it to the Hero on the detail screen
            Hero(
              // Tag must be unique and match on both screens
              tag: 'avatar_hero',
              child: GestureDetector(
                onTap: () {
                  // Navigate to detail screen
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => const HeroDetailScreen(),
                    ),
                  );
                },
                child: Container(
                  width: 100,
                  height: 100,
                  decoration: BoxDecoration(
                    color: Colors.blue,
                    shape: BoxShape.circle,
                    boxShadow: [
                      BoxShadow(
                        color: Colors.black.withOpacity(0.2),
                        blurRadius: 10,
                        spreadRadius: 2,
                      ),
                    ],
                  ),
                  child: const Center(
                    child: Icon(
                      Icons.person,
                      color: Colors.white,
                      size: 50,
                    ),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 16),
            
            // 2. Text indicating tap action
            const Text(
              'Tap the avatar to see the Hero animation',
              style: TextStyle(color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}

/// Second screen with matching Hero
class HeroDetailScreen extends StatelessWidget {
  const HeroDetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Hero Detail'),
        leading: IconButton(
          icon: const Icon(Icons.arrow_back),
          onPressed: () {
            Navigator.pop(context);
          },
        ),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 3. Hero widget with matching tag
            // The tag must be identical to the one on the first screen
            Hero(
              tag: 'avatar_hero',
              child: Container(
                width: 200,
                height: 200,
                decoration: BoxDecoration(
                  color: Colors.blue,
                  shape: BoxShape.circle,
                  boxShadow: [
                    BoxShadow(
                      color: Colors.black.withOpacity(0.3),
                      blurRadius: 20,
                      spreadRadius: 5,
                    ),
                  ],
                ),
                child: const Center(
                  child: Icon(
                    Icons.person,
                    color: Colors.white,
                    size: 100,
                  ),
                ),
              ),
            ),
            const SizedBox(height: 16),
            
            // 4. Additional content
            const Text(
              'This is the detail screen',
              style: TextStyle(fontSize: 18),
            ),
            const Text(
              'The avatar flew here!',
              style: TextStyle(color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Hero widget wraps the element to animate
- tag identifies and pairs Heroes
- Hero flies from source to destination
- Container size and shape can change

---

# Hero with Images

> **Creating Hero transitions** with images.

```dart
/// Hero with images example
class HeroImageExample extends StatelessWidget {
  const HeroImageExample({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const ImageGalleryScreen(),
    );
  }
}

/// Gallery screen with image grid
class ImageGalleryScreen extends StatelessWidget {
  const ImageGalleryScreen({super.key});

  // Sample image data
  final List<Map<String, dynamic>> _images = const [
    {
      'id': 1,
      'title': 'Mountain View',
      'color': Colors.blue,
    },
    {
      'id': 2,
      'title': 'Sunset Beach',
      'color': Colors.orange,
    },
    {
      'id': 3,
      'title': 'Forest Trail',
      'color': Colors.green,
    },
    {
      'id': 4,
      'title': 'City Lights',
      'color': Colors.purple,
    },
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Image Gallery'),
      ),
      body: GridView.builder(
        padding: const EdgeInsets.all(8),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 8,
          mainAxisSpacing: 8,
        ),
        itemCount: _images.length,
        itemBuilder: (context, index) {
          final image = _images[index];
          
          return GestureDetector(
            onTap: () {
              // Navigate to detail with the image data
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => ImageDetailScreen(
                    imageData: image,
                  ),
                ),
              );
            },
            child: Hero(
              // 1. Unique tag for each image
              tag: 'image_${image['id']}',
              child: Container(
                decoration: BoxDecoration(
                  color: image['color'],
                  borderRadius: BorderRadius.circular(8),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.black.withOpacity(0.2),
                      blurRadius: 8,
                    ),
                  ],
                ),
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(
                      Icons.image,
                      color: Colors.white,
                      size: 50,
                    ),
                    const SizedBox(height: 8),
                    Text(
                      image['title'],
                      style: const TextStyle(
                        color: Colors.white,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ],
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}

/// Detail screen for image
class ImageDetailScreen extends StatelessWidget {
  const ImageDetailScreen({
    super.key,
    required this.imageData,
  });

  final Map<String, dynamic> imageData;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(imageData['title']),
        leading: IconButton(
          icon: const Icon(Icons.arrow_back),
          onPressed: () {
            Navigator.pop(context);
          },
        ),
      ),
      body: Center(
        child: Hero(
          // 2. Matching tag for the image
          tag: 'image_${imageData['id']}',
          child: Container(
            width: 300,
            height: 300,
            decoration: BoxDecoration(
              color: imageData['color'],
              borderRadius: BorderRadius.circular(16),
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withOpacity(0.3),
                  blurRadius: 20,
                  spreadRadius: 5,
                ),
              ],
            ),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(
                  Icons.image,
                  color: Colors.white,
                  size: 80,
                ),
                const SizedBox(height: 16),
                Text(
                  imageData['title'],
                  style: const TextStyle(
                    color: Colors.white,
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                Text(
                  'Image ${imageData['id']}',
                  style: const TextStyle(
                    color: Colors.white70,
                    fontSize: 16,
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Unique tags per image
- Grid to detail transition
- Image flies to new position
- Size and shape change smoothly

---

# Hero with Custom Child

> **Using different child** widgets on each screen.

```dart
/// Hero with custom child example
class CustomHeroExample extends StatelessWidget {
  const CustomHeroExample({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const CustomHeroHomeScreen(),
    );
  }
}

class CustomHeroHomeScreen extends StatelessWidget {
  const CustomHeroHomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Hero'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Hero with custom child (different on each screen)
            Hero(
              tag: 'custom_hero',
              child: GestureDetector(
                onTap: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => const CustomHeroDetailScreen(),
                    ),
                  );
                },
                child: Container(
                  width: 120,
                  height: 120,
                  decoration: BoxDecoration(
                    gradient: const LinearGradient(
                      colors: [Colors.blue, Colors.purple],
                    ),
                    shape: BoxShape.circle,
                    boxShadow: [
                      BoxShadow(
                        color: Colors.blue.withOpacity(0.3),
                        blurRadius: 20,
                        spreadRadius: 5,
                      ),
                    ],
                  ),
                  child: const Center(
                    child: Text(
                      'Tap Me',
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: 16,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 16),
            const Text(
              'The hero child changes shape',
              style: TextStyle(color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}

class CustomHeroDetailScreen extends StatelessWidget {
  const CustomHeroDetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Detail'),
        leading: IconButton(
          icon: const Icon(Icons.arrow_back),
          onPressed: () {
            Navigator.pop(context);
          },
        ),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 2. Hero with different child (rectangular)
            Hero(
              tag: 'custom_hero',
              child: Container(
                width: 250,
                height: 150,
                decoration: BoxDecoration(
                  gradient: const LinearGradient(
                    colors: [Colors.purple, Colors.pink],
                  ),
                  borderRadius: BorderRadius.circular(16),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.purple.withOpacity(0.3),
                      blurRadius: 30,
                      spreadRadius: 10,
                    ),
                  ],
                ),
                child: const Center(
                  child: Text(
                    'I Changed Shape!',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 16),
            const Text(
              'The hero morphs from circle to rectangle',
              style: TextStyle(color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Different child widgets on each screen
- Shape changes from circle to rectangle
- Color gradient changes
- Hero morphs seamlessly

---

# Real-World Examples

> **Common patterns** with Hero.

```dart
/// 1. Profile card with Hero
class ProfileCard extends StatelessWidget {
  const ProfileCard({
    super.key,
    required this.name,
    required this.email,
    required this.avatarColor,
  });

  final String name;
  final String email;
  final Color avatarColor;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: ListTile(
        onTap: () {
          Navigator.push(
            context,
            MaterialPageRoute(
              builder: (context) => ProfileDetailScreen(
                name: name,
                email: email,
                avatarColor: avatarColor,
              ),
            ),
          );
        },
        leading: Hero(
          tag: 'profile_${name.hashCode}',
          child: CircleAvatar(
            backgroundColor: avatarColor,
            child: Text(
              name[0].toUpperCase(),
              style: const TextStyle(color: Colors.white),
            ),
          ),
        ),
        title: Text(name),
        subtitle: Text(email),
        trailing: const Icon(Icons.arrow_forward_ios),
      ),
    );
  }
}

class ProfileDetailScreen extends StatelessWidget {
  const ProfileDetailScreen({
    super.key,
    required this.name,
    required this.email,
    required this.avatarColor,
  });

  final String name;
  final String email;
  final Color avatarColor;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(name),
        leading: IconButton(
          icon: const Icon(Icons.arrow_back),
          onPressed: () {
            Navigator.pop(context);
          },
        ),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Hero(
              tag: 'profile_${name.hashCode}',
              child: CircleAvatar(
                backgroundColor: avatarColor,
                radius: 50,
                child: Text(
                  name[0].toUpperCase(),
                  style: const TextStyle(
                    color: Colors.white,
                    fontSize: 40,
                  ),
                ),
              ),
            ),
            const SizedBox(height: 16),
            Text(
              name,
              style: const TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              email,
              style: const TextStyle(
                fontSize: 16,
                color: Colors.grey,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// 2. Product card with Hero
class ProductCard extends StatelessWidget {
  const ProductCard({
    super.key,
    required this.productId,
    required this.title,
    required this.price,
    required this.color,
  });

  final int productId;
  final String title;
  final double price;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => ProductDetailScreen(
              productId: productId,
              title: title,
              price: price,
              color: color,
            ),
          ),
        );
      },
      child: Card(
        margin: const EdgeInsets.all(8),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Hero(
              tag: 'product_$productId',
              child: Container(
                height: 150,
                width: double.infinity,
                color: color,
                child: const Center(
                  child: Icon(
                    Icons.shopping_bag,
                    color: Colors.white,
                    size: 50,
                  ),
                ),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(12),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    title,
                    style: const TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                  ),
                  Text(
                    '\$${price.toStringAsFixed(2)}',
                    style: const TextStyle(
                      color: Colors.blue,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class ProductDetailScreen extends StatelessWidget {
  const ProductDetailScreen({
    super.key,
    required this.productId,
    required this.title,
    required this.price,
    required this.color,
  });

  final int productId;
  final String title;
  final double price;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(title),
      ),
      body: Column(
        children: [
          Hero(
            tag: 'product_$productId',
            child: Container(
              height: 300,
              width: double.infinity,
              color: color,
              child: const Center(
                child: Icon(
                  Icons.shopping_bag,
                  color: Colors.white,
                  size: 100,
                ),
              ),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  title,
                  style: const TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                Text(
                  '\$${price.toStringAsFixed(2)}',
                  style: const TextStyle(
                    fontSize: 20,
                    color: Colors.blue,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 16),
                const Text(
                  'Product description goes here. This is a detailed description of the product.',
                  style: TextStyle(fontSize: 16),
                ),
                const SizedBox(height: 24),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: () {
                      Navigator.pop(context);
                    },
                    child: const Text('Add to Cart'),
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
```

What's happening here?
- Profile card with avatar hero
- Product card with image hero
- Different sizes on each screen
- Smooth transitions

---

# Best Practices

## Use Unique Tags

```dart
// Good - Unique tags
Hero(
  tag: 'user_${user.id}',
  child: ...
)

// Bad - Duplicate tags
Hero(
  tag: 'avatar', // Same tag used multiple times
  child: ...
)
```

## Keep Hero Content Consistent

```dart
// Good - Similar content
Hero(
  tag: 'image',
  child: Image.asset('image.jpg'),
)

// Bad - Completely different content
Hero(
  tag: 'image',
  child: Text('Different'), // Won't look good
)
```

## Use Appropriate Widgets

```dart
// Good - Widgets that look good when animated
Container, Image, Icon, CircleAvatar

// Bad - Complex widgets
ListView, GridView, Form
```

---

# Common Mistakes

## Duplicate Tags

Wrong:
```dart
// Multiple Heroes with same tag
Hero(tag: 'same', child: ...)
Hero(tag: 'same', child: ...) // Error
```

Correct:
```dart
// Unique tags
Hero(tag: 'hero_1', child: ...)
Hero(tag: 'hero_2', child: ...)
```

## Missing Tag on One Screen

Wrong:
```dart
// Hero without tag on detail screen
// Error: No Hero widget with matching tag
```

Correct:
```dart
// Hero on both screens with matching tag
Hero(tag: 'my_hero', child: ...)
```

---

# Summary

Hero creates smooth transitions between screens using shared elements. Use Hero with unique tags to animate widgets between screens. Heroes can change size, shape, and color during transition. Heroes are perfect for images, avatars, and product cards.

---

# Next Steps

- [AnimatedBuilder](animatedbuilder.md)
- [CustomPainter Animation](custompainter-animation.md)
- [Physics Animations](physics-animations.md)

---

# Did You Know?

- Hero transitions are called "shared element transitions"
- Tags must be unique and match exactly
- Heroes can change size and shape
- Heroes work with any widget
- Heroes are commonly used for images
- Hero animations are built-in
- Heroes can be customized
- Heroes improve navigation experience