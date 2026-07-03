# Image Optimization

Understand how to optimize images for better performance in Flutter applications.

---

# What is it?

Image Optimization is the practice of reducing image file sizes, loading times, and memory usage while maintaining acceptable visual quality. In Flutter, image optimization involves using appropriate image formats, compression, caching strategies, and loading techniques to improve app performance and user experience.

---

# Why does it exist?

Image Optimization exists to:

- Reduce app size and download time
- Improve loading performance
- Reduce memory usage
- Prevent out-of-memory errors
- Optimize network usage
- Improve user experience
- Support slow network conditions

---

# Image Formats

> **Choosing the right** image format.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Image formats comparison
class ImageFormatsExample extends StatelessWidget {
  const ImageFormatsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Image Formats'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. PNG - Lossless, good for icons, transparent backgrounds
            const Text(
              'PNG - Lossless (Icons, Transparent)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Image.asset(
              'assets/images/logo.png',
              width: 100,
              height: 100,
            ),
            const SizedBox(height: 16),
            
            // 2. JPEG - Lossy, good for photographs
            const Text(
              'JPEG - Lossy (Photographs)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Image.network(
              'https://picsum.photos/200/200?random=1',
              width: 100,
              height: 100,
              fit: BoxFit.cover,
            ),
            const SizedBox(height: 16),
            
            // 3. WebP - Modern format, better compression
            const Text(
              'WebP - Modern (Better Compression)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Image.network(
              'https://www.gstatic.com/webp/gallery/1.webp',
              width: 100,
              height: 100,
              fit: BoxFit.cover,
            ),
            const SizedBox(height: 16),
            
            // 4. SVG - Vector, scalable
            const Text(
              'SVG - Vector (Scalable)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            // SVG needs flutter_svg package
            Container(
              width: 100,
              height: 100,
              color: Colors.grey[200],
              child: const Center(
                child: Text('SVG support\nneeds flutter_svg'),
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
- PNG: Lossless, transparent, icons
- JPEG: Lossy, photos, smaller size
- WebP: Modern, better compression
- SVG: Vector, scalable

---

# Image Compression

> **Compressing images** for better performance.

```dart
/// Image compression example
class ImageCompressionExample extends StatefulWidget {
  const ImageCompressionExample({super.key});

  @override
  State<ImageCompressionExample> createState() => _ImageCompressionExampleState();
}

class _ImageCompressionExampleState extends State<ImageCompressionExample> {
  // 1. Track compression info
  String _compressionInfo = 'Original size: Unknown';

  @override
  void initState() {
    super.initState();
    _checkImageSize();
  }

  // 2. Check image size
  void _checkImageSize() async {
    // In a real app, you'd check the actual file size
    setState(() {
      _compressionInfo = 'Original: 2.5 MB\n'
                          'Compressed: 0.3 MB\n'
                          'Reduction: 88%';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Image Compression'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Original image
            const Text(
              'Original Image (2.5 MB)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Image.network(
              'https://picsum.photos/400/300?random=1',
              width: 200,
              height: 150,
              fit: BoxFit.cover,
            ),
            const SizedBox(height: 16),
            
            // 2. Compressed image
            const Text(
              'Compressed Image (0.3 MB)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Image.network(
              'https://picsum.photos/200/150?random=1',
              width: 200,
              height: 150,
              fit: BoxFit.cover,
            ),
            const SizedBox(height: 16),
            
            // 3. Compression info
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.green[50],
                borderRadius: BorderRadius.circular(8),
                border: Border.all(color: Colors.green[200]!),
              ),
              child: Column(
                children: [
                  const Text(
                    'Compression Results',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Text(_compressionInfo),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 4. Tips
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'Compression Tips:',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                  ),
                  SizedBox(height: 8),
                  Text('• Use JPEG for photographs'),
                  Text('• Use PNG for icons with transparency'),
                  Text('• Use WebP for modern browsers'),
                  Text('• Compress before adding to assets'),
                  Text('• Use appropriate resolution for screen'),
                ],
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
- Original vs compressed images
- Significant size reduction
- Quality vs size trade-off
- Compression tips

---

# Responsive Images

> **Loading appropriate** image sizes.

```dart
/// Responsive images example
class ResponsiveImagesExample extends StatelessWidget {
  const ResponsiveImagesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Responsive Images'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Using MediaQuery for responsive sizing
            const Text(
              'Responsive Image Sizing',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            _buildResponsiveImage(),
            
            const SizedBox(height: 24),
            
            // 2. Different sizes for different devices
            const Text(
              'Device-Aware Image Loading',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            _buildDeviceAwareImage(),
            
            const SizedBox(height: 24),
            
            // 3. Aspect ratio maintained
            const Text(
              'Maintaining Aspect Ratio',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            _buildAspectRatioImage(),
          ],
        ),
      ),
    );
  }

  Widget _buildResponsiveImage() {
    // Get screen width for responsive sizing
    final screenWidth = MediaQuery.of(context).size.width;
    final imageWidth = screenWidth * 0.8;
    final imageHeight = imageWidth * 0.75;

    return Container(
      width: imageWidth,
      height: imageHeight,
      decoration: BoxDecoration(
        color: Colors.grey[200],
        borderRadius: BorderRadius.circular(8),
      ),
      child: Image.network(
        'https://picsum.photos/${imageWidth.toInt()}/${imageHeight.toInt()}',
        fit: BoxFit.cover,
        // Cache optimized version
        cacheWidth: imageWidth.toInt(),
        cacheHeight: imageHeight.toInt(),
      ),
    );
  }

  Widget _buildDeviceAwareImage() {
    // Different sizes based on device pixel ratio
    final pixelRatio = MediaQuery.of(context).devicePixelRatio;
    final baseWidth = 200;
    final imageWidth = (baseWidth * pixelRatio).toInt();

    return Container(
      width: 200,
      height: 150,
      decoration: BoxDecoration(
        color: Colors.grey[200],
        borderRadius: BorderRadius.circular(8),
      ),
      child: Image.network(
        'https://picsum.photos/$imageWidth/${(imageWidth * 0.75).toInt()}',
        fit: BoxFit.cover,
        cacheWidth: imageWidth,
        cacheHeight: (imageWidth * 0.75).toInt(),
      ),
    );
  }

  Widget _buildAspectRatioImage() {
    return AspectRatio(
      aspectRatio: 16 / 9,
      child: Image.network(
        'https://picsum.photos/400/225',
        fit: BoxFit.cover,
        cacheWidth: 400,
        cacheHeight: 225,
      ),
    );
  }
}
```

What's happening here?
- Images adapt to screen size
- Device pixel ratio aware
- Aspect ratio maintained
- Cache optimization

---

# Memory Optimization

> **Reducing image** memory usage.

```dart
/// Memory optimization example

class MemoryOptimizationExample extends StatefulWidget {
  const MemoryOptimizationExample({super.key});

  @override
  State<MemoryOptimizationExample> createState() =>
      _MemoryOptimizationExampleState();
}

class _MemoryOptimizationExampleState extends State<MemoryOptimizationExample> {
  String _memoryUsage = '0.0 MB';
  int _imageCount = 0;

  @override
  void initState() {
    super.initState();
    // Schedule a check right after the first frame renders
    WidgetsBinding.instance.addPostFrameCallback((_) => _checkMemoryUsage());
  }

  void _checkMemoryUsage() {
    final cache = PaintingBinding.instance.imageCache;
    final cacheSize = cache.currentSize;
    final cacheSizeBytes = cache.currentSizeBytes;

    setState(() {
      _imageCount = cacheSize;
      _memoryUsage = '${(cacheSizeBytes / 1024 / 1024).toStringAsFixed(2)} MB';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Memory Optimization'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _checkMemoryUsage,
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Memory display card
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceAround,
                children: [
                  Column(
                    children: [
                      const Text(
                        'Cache Size',
                        style: TextStyle(fontWeight: FontWeight.bold),
                      ),
                      Text(
                        '$_imageCount images',
                        style: const TextStyle(fontSize: 20),
                      ),
                    ],
                  ),
                  Column(
                    children: [
                      const Text(
                        'Memory Used',
                        style: TextStyle(fontWeight: FontWeight.bold),
                      ),
                      Text(_memoryUsage, style: const TextStyle(fontSize: 20)),
                    ],
                  ),
                ],
              ),
            ),
            const SizedBox(height: 24),

            const Text(
              'Optimized Image Loading',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),

            // 2. The optimized image frame
            Container(
              width: 200,
              height: 150,
              color: Colors.grey[200],
              child: Image.network(
                // Added a timestamp to break the browser/CDN cache so it downloads fresh
                'https://picsum.photos/1200/800?cb=${DateTime.now().millisecondsSinceEpoch}',
                width: 200,
                height: 150,
                fit: BoxFit.cover,
                cacheWidth:
                    200, // Truncates image matrix calculation down to layout dimensions
                cacheHeight: 150,
                frameBuilder: (context, child, frame, wasSynchronouslyLoaded) {
                  if (frame != null) {
                    // CRITICAL: Trigger layout state recalculation when the image actually finishes rendering
                    WidgetsBinding.instance.addPostFrameCallback(
                      (_) => _checkMemoryUsage(),
                    );
                  }
                  return child;
                },
              ),
            ),
            const SizedBox(height: 24),

            // 3. Destructive state modification action
            ElevatedButton(
              onPressed: () {
                PaintingBinding.instance.imageCache.clear();
                PaintingBinding.instance.imageCache
                    .clearLiveImages(); // Clears running engine references
                _checkMemoryUsage();
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(
                    content: Text('Image cache cleared completely'),
                  ),
                );
              },
              style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
              child: const Text(
                'Clear Image Cache',
                style: TextStyle(color: Colors.white),
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
- Monitor cache size
- Optimize cache usage
- Clear cache when needed
- Memory-efficient loading

---

# Real-World Examples

> **Common patterns** with image optimization.

```dart
/// 1. Progressive image loading
class ProgressiveImageLoader extends StatelessWidget {
  const ProgressiveImageLoader({
    super.key,
    required this.imageUrl,
    this.width,
    this.height,
    this.fit,
  });

  final String imageUrl;
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
      // 1. Load optimized version
      cacheWidth: width != null ? width!.toInt() : null,
      cacheHeight: height != null ? height!.toInt() : null,
      // 2. Show loading indicator
      loadingBuilder: (context, child, loadingProgress) {
        if (loadingProgress == null) return child;
        return Container(
          width: width,
          height: height,
          color: Colors.grey[200],
          child: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const CircularProgressIndicator(),
                const SizedBox(height: 8),
                if (loadingProgress.expectedTotalBytes != null)
                  Text(
                    '${(loadingProgress.cumulativeBytesLoaded /
                        loadingProgress.expectedTotalBytes! * 100).round()}%',
                    style: const TextStyle(fontSize: 12, color: Colors.grey),
                  ),
              ],
            ),
          ),
        );
      },
      // 3. Error handling
      errorBuilder: (context, error, stackTrace) {
        return Container(
          width: width,
          height: height,
          color: Colors.grey[200],
          child: const Icon(Icons.broken_image),
        );
      },
    );
  }
}

/// 2. Lazy image loading
class LazyImageLoader extends StatefulWidget {
  const LazyImageLoader({
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
  State<LazyImageLoader> createState() => _LazyImageLoaderState();
}

class _LazyImageLoaderState extends State<LazyImageLoader> {
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadImage();
  }

  void _loadImage() {
    // Preload image
    final provider = NetworkImage(widget.imageUrl);
    provider.obtainKey(ImageConfiguration.empty).then((_) {
      if (mounted) {
        setState(() {
          _isLoading = false;
        });
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return widget.placeholder;
    }

    return Image.network(
      widget.imageUrl,
      width: widget.width,
      height: widget.height,
      fit: widget.fit,
      cacheWidth: widget.width != null ? widget.width!.toInt() : null,
      cacheHeight: widget.height != null ? widget.height!.toInt() : null,
    );
  }
}
```

What's happening here?
- Progressive loading with indicator
- Lazy loading with placeholder
- Cache optimization
- Error handling

---

# Best Practices

## Use Appropriate Formats

```dart
// Good - Use JPEG for photos
Image.network('photo.jpg')

// Good - Use PNG for icons
Image.asset('icon.png')

// Good - Use WebP for modern apps
Image.network('image.webp')
```

## Optimize Image Size

```dart
// Good - Load appropriate size
Image.network(
  url,
  cacheWidth: 200,
  cacheHeight: 200,
)

// Bad - Loading full size
Image.network(url)
```

## Use Caching

```dart
// Good - Cache optimized images
Image.network(
  url,
  cacheWidth: 200,
  cacheHeight: 200,
)
```

---

# Common Mistakes

## Loading Full-Size Images

Wrong:
```dart
// Loads full size image
Image.network('https://example.com/huge-image.jpg')
```

Correct:
```dart
// Loads optimized size
Image.network(
  'https://example.com/huge-image.jpg',
  cacheWidth: 200,
  cacheHeight: 200,
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

Image Optimization improves app performance by reducing file sizes, memory usage, and loading times. Use appropriate formats, compress images, load responsive sizes, and implement caching. Optimized images result in faster loading and better user experience.

---

# Next Steps

- [DevTools](devtools.md)
- [Profiling](profiling.md)
- [Performance Best Practices](performance-best-practices.md)

---

# Did You Know?

- JPEG is best for photographs
- PNG is best for icons with transparency
- WebP offers better compression
- cacheWidth/cacheHeight optimize memory
- Image cache reduces network usage
- Progressive loading improves UX
- Lazy loading saves bandwidth
- Proper optimization reduces app size