# Image Widget

Understand how to display and manipulate images in Flutter applications.

---

# What is it?

The Image widget is a versatile widget for displaying images from various sources including assets, network, memory, and the device's file system. It provides extensive options for controlling how images are displayed, including fit, alignment, repeat, color blending, and caching. Image is one of the most commonly used widgets in Flutter applications.

---

# Why does it exist?

The Image widget exists to:

- Display images from multiple sources
- Control image rendering and appearance
- Handle different screen sizes and densities
- Support image caching and optimization
- Provide accessibility features
- Enable image manipulation (color, fit, repeat)
- Support various image formats

---

# Image Sources

> **Displaying images** from different sources.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Image sources example
class ImageSourcesExample extends StatelessWidget {
  const ImageSourcesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Image Sources'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Asset image
            // Loads image from the asset bundle
            const Text(
              'Asset Image',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Image.asset(
              'assets/images/logo.png',
              width: 100,
              height: 100,
            ),
            const SizedBox(height: 16),
            
            // 2. Network image
            // Loads image from a URL
            const Text(
              'Network Image',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Image.network(
              'https://picsum.photos/200',
              width: 100,
              height: 100,
            ),
            const SizedBox(height: 16),
            
            // 3. Memory image
            // Loads image from byte data
            const Text(
              'Memory Image',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            // Image.memory(bytes), // Requires Uint8List bytes
            Container(
              width: 100,
              height: 100,
              color: Colors.grey[300],
              child: const Center(
                child: Text(
                  'Memory Image\n(requires bytes)',
                  textAlign: TextAlign.center,
                  style: TextStyle(fontSize: 12),
                ),
              ),
            ),
            const SizedBox(height: 16),
            
            // 4. File image
            // Loads image from the file system
            const Text(
              'File Image',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Container(
              width: 100,
              height: 100,
              color: Colors.grey[300],
              child: const Center(
                child: Text(
                  'File Image\n(requires file path)',
                  textAlign: TextAlign.center,
                  style: TextStyle(fontSize: 12),
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
- Image.asset: Loads from assets
- Image.network: Loads from URL
- Image.memory: Loads from bytes
- Image.file: Loads from file system

---

# Image Fit

> **Controlling how** images fill their container.

```dart
/// Image fit examples
class ImageFitExample extends StatelessWidget {
  const ImageFitExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Image Fit'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. BoxFit.fill
            // Stretches to fill the container
            _buildFitDemo(
              'BoxFit.fill',
              BoxFit.fill,
              'Stretches image to fill container',
            ),
            
            // 2. BoxFit.contain
            // Scales to fit within the container
            _buildFitDemo(
              'BoxFit.contain',
              BoxFit.contain,
              'Scales to fit inside container',
            ),
            
            // 3. BoxFit.cover
            // Scales to cover the container
            _buildFitDemo(
              'BoxFit.cover',
              BoxFit.cover,
              'Scales to cover container (may crop)',
            ),
            
            // 4. BoxFit.fitWidth
            // Scales to fit the width
            _buildFitDemo(
              'BoxFit.fitWidth',
              BoxFit.fitWidth,
              'Scales to fit container width',
            ),
            
            // 5. BoxFit.fitHeight
            // Scales to fit the height
            _buildFitDemo(
              'BoxFit.fitHeight',
              BoxFit.fitHeight,
              'Scales to fit container height',
            ),
            
            // 6. BoxFit.none
            // Original size
            _buildFitDemo(
              'BoxFit.none',
              BoxFit.none,
              'Original image size',
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildFitDemo(String label, BoxFit fit, String description) {
    return Container(
      margin: const EdgeInsets.only(bottom: 16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            label,
            style: const TextStyle(fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 4),
          Text(
            description,
            style: TextStyle(
              fontSize: 12,
              color: Colors.grey[600],
            ),
          ),
          const SizedBox(height: 8),
          Container(
            width: 200,
            height: 100,
            decoration: BoxDecoration(
              border: Border.all(color: Colors.grey, width: 2),
            ),
            child: Image.network(
              'https://picsum.photos/400/300',
              fit: fit,
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- BoxFit.fill: Stretches to fill
- BoxFit.contain: Fits inside
- BoxFit.cover: Covers container
- BoxFit.fitWidth: Fits width
- BoxFit.fitHeight: Fits height
- BoxFit.none: Original size

---

# Image Properties

> **Customizing image** appearance.

```dart
/// Image properties example
class ImagePropertiesExample extends StatelessWidget {
  const ImagePropertiesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Image Properties'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Basic image with properties
            const Text(
              'Basic Properties',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Image.network(
              'https://picsum.photos/200',
              width: 150,
              height: 150,
              fit: BoxFit.cover,
              alignment: Alignment.center,
            ),
            const SizedBox(height: 16),
            
            // 2. Image with color blending
            const Text(
              'Color Blending',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildColorBlendDemo(Colors.blue.withOpacity(0.5)),
                _buildColorBlendDemo(Colors.red.withOpacity(0.5)),
                _buildColorBlendDemo(Colors.green.withOpacity(0.5)),
              ],
            ),
            const SizedBox(height: 16),
            
            // 3. Image with opacity
            const Text(
              'Opacity',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildOpacityDemo(1.0),
                _buildOpacityDemo(0.5),
                _buildOpacityDemo(0.2),
              ],
            ),
            const SizedBox(height: 16),
            
            // 4. Image with repeat
            const Text(
              'Image Repeat',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Container(
              width: 200,
              height: 100,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey),
              ),
              child: Image.network(
                'https://picsum.photos/50',
                repeat: ImageRepeat.repeat,
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildColorBlendDemo(Color color) {
    return Container(
      width: 80,
      height: 80,
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey),
      ),
      child: Image.network(
        'https://picsum.photos/80',
        color: color,
        colorBlendMode: BlendMode.multiply,
        fit: BoxFit.cover,
      ),
    );
  }

  Widget _buildOpacityDemo(double opacity) {
    return Container(
      width: 80,
      height: 80,
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey),
      ),
      child: Opacity(
        opacity: opacity,
        child: Image.network(
          'https://picsum.photos/80',
          fit: BoxFit.cover,
        ),
      ),
    );
  }
}
```

What's happening here?
- color: Tints the image
- colorBlendMode: Blends color with image
- opacity: Controls transparency
- repeat: Repeats the image

---

# Fade-In Images

> **Creating smooth** image loading experiences.

```dart
/// Fade-in images example
class FadeInImageExample extends StatelessWidget {
  const FadeInImageExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Fade-In Images'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. FadeInImage with asset placeholder
            const Text(
              'Asset Placeholder',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            FadeInImage.assetNetwork(
              placeholder: 'assets/images/placeholder.png',
              image: 'https://picsum.photos/300/200',
              width: 200,
              height: 150,
              fit: BoxFit.cover,
            ),
            const SizedBox(height: 16),
            
            // 2. FadeInImage with memory placeholder
            const Text(
              'Memory Placeholder',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            FadeInImage.memoryNetwork(
              placeholder: kTransparentImage,
              image: 'https://picsum.photos/300/200',
              width: 200,
              height: 150,
              fit: BoxFit.cover,
            ),
            const SizedBox(height: 16),
            
            // 3. Custom fade duration
            const Text(
              'Custom Fade Duration',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            FadeInImage.assetNetwork(
              placeholder: 'assets/images/placeholder.png',
              image: 'https://picsum.photos/300/200',
              width: 200,
              height: 150,
              fit: BoxFit.cover,
              fadeInDuration: const Duration(milliseconds: 1000),
              fadeOutDuration: const Duration(milliseconds: 500),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- FadeInImage fades in when loaded
- placeholder: Shows while loading
- fadeInDuration: Controls fade speed
- assetNetwork: Uses asset placeholder with network image

---

# Real-World Examples

> **Common patterns** with Image widget.

```dart
/// 1. Profile avatar with network image
class ProfileAvatar extends StatelessWidget {
  const ProfileAvatar({
    super.key,
    required this.imageUrl,
    required this.name,
    this.size = 80,
  });

  final String imageUrl;
  final String name;
  final double size;

  @override
  Widget build(BuildContext context) {
    return CircleAvatar(
      radius: size / 2,
      backgroundImage: NetworkImage(imageUrl),
      child: const Center(
        child: Text(
          'Error',
          style: TextStyle(color: Colors.white),
        ),
      ),
      onBackgroundImageError: (exception, stackTrace) {
        print('Image loading error: $exception');
      },
    );
  }
}

/// 2. Product image with loading indicator
class ProductImage extends StatefulWidget {
  const ProductImage({
    super.key,
    required this.imageUrl,
    this.width = double.infinity,
    this.height = 200,
  });

  final String imageUrl;
  final double width;
  final double height;

  @override
  State<ProductImage> createState() => _ProductImageState();
}

class _ProductImageState extends State<ProductImage> {
  bool _isLoading = true;
  bool _hasError = false;

  @override
  Widget build(BuildContext context) {
    return Container(
      width: widget.width,
      height: widget.height,
      color: Colors.grey[200],
      child: Stack(
        children: [
          // 3. Image with loading state
          Image.network(
            widget.imageUrl,
            width: widget.width,
            height: widget.height,
            fit: BoxFit.cover,
            loadingBuilder: (context, child, loadingProgress) {
              if (loadingProgress == null) {
                // Image loaded
                return child;
              }
              // Still loading
              return Container(
                color: Colors.grey[200],
                child: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const CircularProgressIndicator(),
                      const SizedBox(height: 8),
                      Text(
                        'Loading...',
                        style: TextStyle(
                          color: Colors.grey[600],
                          fontSize: 12,
                        ),
                      ),
                    ],
                  ),
                ),
              );
            },
            errorBuilder: (context, error, stackTrace) {
              // Error loading image
              return Container(
                color: Colors.grey[200],
                child: const Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(
                        Icons.broken_image,
                        color: Colors.grey,
                        size: 48,
                      ),
                      SizedBox(height: 8),
                      Text(
                        'Failed to load image',
                        style: TextStyle(color: Colors.grey),
                      ),
                    ],
                  ),
                ),
              );
            },
          ),
        ],
      ),
    );
  }
}

/// 4. Image with overlay
class ImageWithOverlay extends StatelessWidget {
  const ImageWithOverlay({
    super.key,
    required this.imageUrl,
    required this.title,
    required this.subtitle,
  });

  final String imageUrl;
  final String title;
  final String subtitle;

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        // Image
        Image.network(
          imageUrl,
          width: double.infinity,
          height: 200,
          fit: BoxFit.cover,
        ),
        // Gradient overlay
        Container(
          height: 200,
          decoration: BoxDecoration(
            gradient: LinearGradient(
              begin: Alignment.topCenter,
              end: Alignment.bottomCenter,
              colors: [
                Colors.transparent,
                Colors.black.withOpacity(0.7),
              ],
            ),
          ),
        ),
        // Text overlay
        Positioned(
          bottom: 16,
          left: 16,
          right: 16,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                title,
                style: const TextStyle(
                  color: Colors.white,
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 4),
              Text(
                subtitle,
                style: TextStyle(
                  color: Colors.white.withOpacity(0.8),
                  fontSize: 14,
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- Profile avatar with error handling
- Product image with loading indicator
- Custom loading and error builders
- Image with gradient overlay

---

# Best Practices

## Use Appropriate Image Source

```dart
// Good - Network image for web content
Image.network('https://example.com/image.jpg')

// Good - Asset image for app resources
Image.asset('assets/images/logo.png')

// Good - Memory image for generated content
Image.memory(bytes)
```

## Use Loading and Error Handling

```dart
// Good - Handle loading and errors
Image.network(
  url,
  loadingBuilder: (context, child, progress) {
    if (progress == null) return child;
    return CircularProgressIndicator();
  },
  errorBuilder: (context, error, stackTrace) {
    return Icon(Icons.error);
  },
)
```

## Optimize Image Size

```dart
// Good - Use cacheWidth/cacheHeight
Image.network(
  url,
  cacheWidth: 200, // Optimize memory usage
  cacheHeight: 200,
)
```

---

# Common Mistakes

## Not Using BoxFit

Wrong:
```dart
// Image overflows
Image.network('https://example.com/image.jpg')
```

Correct:
```dart
// Use BoxFit to control sizing
Image.network(
  'https://example.com/image.jpg',
  fit: BoxFit.cover,
)
```

## No Error Handling

Wrong:
```dart
// No error handling
Image.network('https://example.com/image.jpg')
```

Correct:
```dart
// With error handling
Image.network(
  'https://example.com/image.jpg',
  errorBuilder: (context, error, stackTrace) {
    return Icon(Icons.error);
  },
)
```

---

# Summary

The Image widget displays images from various sources with extensive customization. Use appropriate source types (asset, network, memory, file), control fit and alignment, handle loading and errors, and optimize with caching. Images are essential for creating visually rich applications.

---

# Next Steps

- [Network Images](network-images.md)
- [Caching](caching.md)
- [SVG](svg.md)

---

# Did You Know?

- Image can load from asset, network, memory, file
- BoxFit controls how image fills container
- FadeInImage provides smooth loading
- loadingBuilder shows loading progress
- errorBuilder handles loading errors
- color and colorBlendMode create effects
- cacheWidth/cacheHeight optimize memory
- Images can be repeated with ImageRepeat