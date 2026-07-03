# Network Images

Understand how to load, display, and manage images from the internet in Flutter applications.

---

# What is it?

Network Images are images loaded from URLs over the internet. Flutter provides built-in support for loading images from web sources through the Image.network widget and the NetworkImage class. Network images are essential for displaying dynamic content like user avatars, product images, and media from external sources.

---

# Why does it exist?

Network Images exist to:

- Load images from the internet
- Display dynamic and remote content
- Support user-generated images
- Enable CDN and cloud storage integration
- Handle different image formats and sizes
- Implement image caching
- Support image loading states

---

# Basic Network Image

> **Loading images** from URLs.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic network image example
class BasicNetworkImageExample extends StatelessWidget {
  const BasicNetworkImageExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Network Images'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Simple network image
            // Directly load image from URL
            const Text(
              'Simple Network Image',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Image.network(
              'https://picsum.photos/200/200',
              width: 200,
              height: 200,
              fit: BoxFit.cover,
            ),
            const SizedBox(height: 16),
            
            // 2. Network image with loading indicator
            const Text(
              'With Loading Indicator',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            _buildLoadingImage(),
            const SizedBox(height: 16),
            
            // 3. Network image with error handling
            const Text(
              'With Error Handling',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            _buildErrorHandlingImage(),
          ],
        ),
      ),
    );
  }

  Widget _buildLoadingImage() {
    return Image.network(
      'https://picsum.photos/200/200',
      width: 200,
      height: 200,
      fit: BoxFit.cover,
      // 1. loadingBuilder shows loading progress
      loadingBuilder: (context, child, loadingProgress) {
        if (loadingProgress == null) {
          // Image is fully loaded
          return child;
        }
        // Image is still loading
        return Container(
          width: 200,
          height: 200,
          color: Colors.grey[200],
          child: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                CircularProgressIndicator(
                  value: loadingProgress.expectedTotalBytes != null
                      ? loadingProgress.cumulativeBytesLoaded /
                          loadingProgress.expectedTotalBytes!
                      : null,
                ),
                const SizedBox(height: 8),
                Text(
                  loadingProgress.expectedTotalBytes != null
                      ? '${(loadingProgress.cumulativeBytesLoaded /
                          loadingProgress.expectedTotalBytes! * 100).round()}%'
                      : 'Loading...',
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
    );
  }

  Widget _buildErrorHandlingImage() {
    return Image.network(
      'https://invalid-url-12345.com/image.jpg',
      width: 200,
      height: 200,
      fit: BoxFit.cover,
      // 2. errorBuilder handles loading errors
      errorBuilder: (context, error, stackTrace) {
        return Container(
          width: 200,
          height: 200,
          color: Colors.grey[200],
          child: const Column(
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
                style: TextStyle(
                  color: Colors.grey,
                  fontSize: 14,
                ),
              ),
            ],
          ),
        );
      },
    );
  }
}
```

What's happening here?
- Image.network loads from URL
- loadingBuilder shows progress
- errorBuilder handles failures
- Progress percentage can be shown

---

# NetworkImage Class

> **Using NetworkImage** for more control.

```dart
/// NetworkImage class example
class NetworkImageClassExample extends StatelessWidget {
  const NetworkImageClassExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('NetworkImage Class'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Using NetworkImage with Image
            // NetworkImage provides more control over image loading
            const Text(
              'Using NetworkImage',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Image(
              image: NetworkImage(
                'https://picsum.photos/200/200',
                // 2. Custom headers for authentication
                headers: {
                  'Authorization': 'Bearer your_token_here',
                  'Cache-Control': 'max-age=3600',
                },
                // 3. Scale factor for resolution
                scale: 1.0,
              ),
              width: 200,
              height: 200,
              fit: BoxFit.cover,
            ),
            const SizedBox(height: 16),
            
            // 4. Using NetworkImage with different scale
            const Text(
              'With Scale Factor',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                // Scale 1.0
                _buildNetworkImageWithScale(1.0, '1.0x'),
                // Scale 2.0
                _buildNetworkImageWithScale(2.0, '2.0x'),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildNetworkImageWithScale(double scale, String label) {
    return Column(
      children: [
        Image(
          image: NetworkImage(
            'https://picsum.photos/100/100',
            scale: scale,
          ),
          width: 100,
          height: 100,
          fit: BoxFit.cover,
        ),
        const SizedBox(height: 4),
        Text(
          label,
          style: TextStyle(
            color: Colors.grey[600],
            fontSize: 12,
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- NetworkImage provides more control
- headers for authentication
- scale for resolution control
- Used with Image widget

---

# Image Cache

> **Controlling image** caching.

```dart
/// Image caching example
class ImageCachingExample extends StatefulWidget {
  const ImageCachingExample({super.key});

  @override
  State<ImageCachingExample> createState() => _ImageCachingExampleState();
}

class _ImageCachingExampleState extends State<ImageCachingExample> {
  // 1. Image cache management
  final ImageCache _imageCache = ImageCache();
  int _cacheSize = 0;

  @override
  void initState() {
    super.initState();
    // 2. Configure image cache
    WidgetsFlutterBinding.ensureInitialized();
    
    // Set cache limits
    PaintingBinding.instance.imageCache.maximumSize = 200; // Maximum number of images
    PaintingBinding.instance.imageCache.maximumSizeBytes = 50 * 1024 * 1024; // 50 MB
    
    // 3. Track cache size
    _updateCacheSize();
  }

  void _updateCacheSize() {
    setState(() {
      _cacheSize = PaintingBinding.instance.imageCache.currentSize;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Image Cache'),
        actions: [
          // 4. Clear cache button
          IconButton(
            icon: const Icon(Icons.clear_all),
            onPressed: () {
              // Clear image cache
              PaintingBinding.instance.imageCache.clear();
              _updateCacheSize();
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(
                  content: Text('Cache cleared'),
                ),
              );
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 5. Cache info
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  const Text(
                    'Cache Size:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  Text(
                    '$_cacheSize images',
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 16),
            
            // 6. Cacheable images
            Expanded(
              child: GridView.builder(
                gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 3,
                  crossAxisSpacing: 8,
                  mainAxisSpacing: 8,
                ),
                itemCount: 9,
                itemBuilder: (context, index) {
                  return Image.network(
                    'https://picsum.photos/200/200?random=$index',
                    fit: BoxFit.cover,
                    // 7. Cache key to control caching
                    cacheKey: 'image_$index',
                  );
                },
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
- ImageCache manages cached images
- maximumSize controls cache count
- maximumSizeBytes controls cache size
- clear() clears the cache
- cacheKey identifies images for caching

---

# Real-World Examples

> **Common patterns** with network images.

```dart
/// 1. User avatar with caching
class UserAvatar extends StatelessWidget {
  const UserAvatar({
    super.key,
    required this.imageUrl,
    this.size = 50,
    this.placeholder,
  });

  final String imageUrl;
  final double size;
  final Widget? placeholder;

  @override
  Widget build(BuildContext context) {
    return CircleAvatar(
      radius: size / 2,
      backgroundColor: Colors.grey[200],
      child: ClipOval(
        child: FadeInImage(
          placeholder: placeholder ?? 
              Image.asset('assets/images/placeholder.png'),
          image: NetworkImage(imageUrl),
          width: size,
          height: size,
          fit: BoxFit.cover,
          // 1. Fade transition
          fadeInDuration: const Duration(milliseconds: 300),
          fadeOutDuration: const Duration(milliseconds: 100),
          // 2. Error handling
          imageErrorBuilder: (context, error, stackTrace) {
            return Container(
              color: Colors.grey[200],
              child: const Icon(
                Icons.person,
                color: Colors.grey,
              ),
            );
          },
        ),
      ),
    );
  }
}

/// 2. Image gallery with preloading
class ImageGallery extends StatefulWidget {
  const ImageGallery({super.key, required this.imageUrls});

  final List<String> imageUrls;

  @override
  State<ImageGallery> createState() => _ImageGalleryState();
}

class _ImageGalleryState extends State<ImageGallery> {
  final List<ImageProvider> _preloadedImages = [];
  bool _isPreloading = false;

  @override
  void initState() {
    super.initState();
    _preloadImages();
  }

  // 3. Preload images for performance
  Future<void> _preloadImages() async {
    setState(() {
      _isPreloading = true;
    });

    // Preload first 5 images
    final toPreload = widget.imageUrls.take(5);
    
    for (final url in toPreload) {
      final provider = NetworkImage(url);
      await provider.evict(); // Remove from cache
      await provider.obtainKey(ImageConfiguration.empty); // Load into cache
      _preloadedImages.add(provider);
    }

    setState(() {
      _isPreloading = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Image Gallery'),
        actions: [
          if (_isPreloading)
            const Padding(
              padding: EdgeInsets.all(8),
              child: SizedBox(
                width: 20,
                height: 20,
                child: CircularProgressIndicator(
                  strokeWidth: 2,
                  valueColor: AlwaysStoppedAnimation<Color>(Colors.white),
                ),
              ),
            ),
        ],
      ),
      body: GridView.builder(
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 3,
          crossAxisSpacing: 4,
          mainAxisSpacing: 4,
        ),
        itemCount: widget.imageUrls.length,
        itemBuilder: (context, index) {
          return Image.network(
            widget.imageUrls[index],
            fit: BoxFit.cover,
            // 4. Cache width for performance
            cacheWidth: 200,
            cacheHeight: 200,
          );
        },
      ),
    );
  }
}

/// 3. Progressive JPEG loader
class ProgressiveImageLoader extends StatelessWidget {
  const ProgressiveImageLoader({
    super.key,
    required this.imageUrl,
    required this.placeholder,
    this.width,
    this.height,
    this.fit,
  });

  final String imageUrl;
  final Widget placeholder;
  final double? width;
  final double? height;
  final BoxFit? fit;

  @override
  Widget build(BuildContext context) {
    return Image.network(
      imageUrl,
      width: width,
      height: height,
      fit: fit,
      loadingBuilder: (context, child, loadingProgress) {
        if (loadingProgress == null) {
          return child;
        }
        // 5. Show placeholder with progress
        return Stack(
          alignment: Alignment.center,
          children: [
            placeholder,
            if (loadingProgress.expectedTotalBytes != null)
              Container(
                padding: const EdgeInsets.all(8),
                decoration: BoxDecoration(
                  color: Colors.black.withOpacity(0.6),
                  borderRadius: BorderRadius.circular(16),
                ),
                child: Text(
                  '${(loadingProgress.cumulativeBytesLoaded /
                      loadingProgress.expectedTotalBytes! * 100).round()}%',
                  style: const TextStyle(
                    color: Colors.white,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              )
            else
              const CircularProgressIndicator(),
          ],
        );
      },
    );
  }
}
```

What's happening here?
- User avatar with caching and error handling
- Image gallery with preloading
- Progressive image loading
- Cache width optimization

---

# Best Practices

## Use Caching

```dart
// Good - Enable caching
Image.network(
  url,
  cacheWidth: 200, // Caches optimized version
  cacheHeight: 200,
)

// Bad - No caching
Image.network(url)
```

## Handle Loading States

```dart
// Good - Show loading indicator
Image.network(
  url,
  loadingBuilder: (context, child, progress) {
    if (progress == null) return child;
    return CircularProgressIndicator();
  },
)
```

## Handle Errors

```dart
// Good - Show error widget
Image.network(
  url,
  errorBuilder: (context, error, stackTrace) {
    return Icon(Icons.error);
  },
)
```

---

# Common Mistakes

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

## No Loading State

Wrong:
```dart
// No loading feedback
Image.network('https://example.com/image.jpg')
```

Correct:
```dart
// With loading feedback
Image.network(
  'https://example.com/image.jpg',
  loadingBuilder: (context, child, progress) {
    if (progress == null) return child;
    return CircularProgressIndicator();
  },
)
```

---

# Summary

Network Images load images from URLs with support for loading states, error handling, and caching. Use Image.network for simple cases, NetworkImage for more control, and loadingBuilder/errorBuilder for user feedback. Optimize with caching and proper image sizing.

---

# Next Steps

- [Caching](caching.md)
- [SVG](svg.md)
- [Asset Bundles](asset-bundles.md)

---

# Did You Know?

- Image.network loads from URLs
- loadingBuilder shows progress
- errorBuilder handles errors
- NetworkImage provides more control
- ImageCache manages caching
- cacheWidth/cacheHeight optimize memory
- Preloading improves performance
- Headers support authentication