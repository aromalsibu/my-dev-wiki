# Caching

Understand how to cache images and data in Flutter applications for improved performance.

---

# What is it?

Caching is the process of storing data locally after it's been loaded, so that subsequent requests can be served faster. In Flutter, caching is especially important for images and network data to reduce bandwidth usage, improve loading times, and provide offline support. Flutter provides built-in caching mechanisms for images and network requests.

---

# Why does it exist?

Caching exists to:

- Reduce network bandwidth usage
- Improve loading performance
- Enable offline support
- Reduce app loading times
- Optimize user experience
- Reduce server load
- Support slow network conditions

---

# Image Caching

> **Understanding** image caching in Flutter.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Image caching example
class ImageCachingExample extends StatefulWidget {
  const ImageCachingExample({super.key});

  @override
  State<ImageCachingExample> createState() => _ImageCachingExampleState();
}

class _ImageCachingExampleState extends State<ImageCachingExample> {
  // 1. Image cache tracking
  int _cacheSize = 0;
  int _cacheSizeBytes = 0;

  @override
  void initState() {
    super.initState();
    _updateCacheInfo();
  }

  // 2. Get cache information
  void _updateCacheInfo() {
    final imageCache = PaintingBinding.instance.imageCache;
    setState(() {
      _cacheSize = imageCache.currentSize;
      _cacheSizeBytes = imageCache.currentSizeBytes;
    });
  }

  // 3. Clear cache
  void _clearCache() {
    PaintingBinding.instance.imageCache.clear();
    _updateCacheInfo();
    
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(
        content: Text('Cache cleared'),
        duration: Duration(seconds: 1),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Image Caching'),
        actions: [
          IconButton(
            icon: const Icon(Icons.clear_all),
            onPressed: _clearCache,
          ),
        ],
      ),
      body: Column(
        children: [
          // 4. Cache info display
          Container(
            padding: const EdgeInsets.all(16),
            color: Colors.blue[50],
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                Column(
                  children: [
                    const Text('Images Cached'),
                    Text(
                      '$_cacheSize',
                      style: const TextStyle(
                        fontSize: 20,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ],
                ),
                Column(
                  children: [
                    const Text('Cache Size'),
                    Text(
                      '${(_cacheSizeBytes / 1024 / 1024).toStringAsFixed(1)} MB',
                      style: const TextStyle(
                        fontSize: 20,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ],
                ),
              ],
            ),
          ),
          
          // 5. Images that will be cached
          Expanded(
            child: GridView.builder(
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 3,
                crossAxisSpacing: 4,
                mainAxisSpacing: 4,
              ),
              itemCount: 30,
              itemBuilder: (context, index) {
                return Image.network(
                  'https://picsum.photos/200/200?random=$index',
                  fit: BoxFit.cover,
                  // 6. Cache key for identification
                  cacheKey: 'image_$index',
                  // 7. Optimize cache size
                  cacheWidth: 200,
                  cacheHeight: 200,
                  // 8. Loading builder with caching info
                  loadingBuilder: (context, child, loadingProgress) {
                    if (loadingProgress == null) {
                      // Image loaded and cached
                      return Stack(
                        children: [
                          child,
                          Positioned(
                            bottom: 4,
                            right: 4,
                            child: Container(
                              padding: const EdgeInsets.symmetric(
                                horizontal: 4,
                                vertical: 2,
                              ),
                              color: Colors.black54,
                              child: const Text(
                                'Cached',
                                style: TextStyle(
                                  color: Colors.white,
                                  fontSize: 10,
                                ),
                              ),
                            ),
                          ),
                        ],
                      );
                    }
                    // Loading
                    return Container(
                      color: Colors.grey[200],
                      child: const Center(
                        child: SizedBox(
                          width: 20,
                          height: 20,
                          child: CircularProgressIndicator(
                            strokeWidth: 2,
                          ),
                        ),
                      ),
                    );
                  },
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- PaintingBinding.instance.imageCache manages cache
- currentSize: Number of cached images
- currentSizeBytes: Cache memory usage
- clear(): Clears the cache
- cacheKey: Identifies images for caching

---

# Custom Cache Control

> **Managing cache** behavior.

```dart
/// Custom cache management
class CustomCacheControl extends StatefulWidget {
  const CustomCacheControl({super.key});

  @override
  State<CustomCacheControl> createState() => _CustomCacheControlState();
}

class _CustomCacheControlState extends State<CustomCacheControl> {
  // 1. Cache configuration
  int _maxCacheSize = 100;
  int _maxCacheSizeBytes = 50 * 1024 * 1024; // 50 MB
  
  // 2. Track cache stats
  int _cacheHits = 0;
  int _cacheMisses = 0;

  @override
  void initState() {
    super.initState();
    _applyCacheConfig();
  }

  // 3. Apply cache configuration
  void _applyCacheConfig() {
    final imageCache = PaintingBinding.instance.imageCache;
    imageCache.maximumSize = _maxCacheSize;
    imageCache.maximumSizeBytes = _maxCacheSizeBytes;
  }

  // 4. Update cache stats
  void _updateCacheStats() {
    // In a real app, you'd implement actual cache tracking
    setState(() {
      _cacheHits = 10; // Placeholder
      _cacheMisses = 5; // Placeholder
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Cache Control'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 5. Cache configuration controls
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'Cache Configuration',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 18,
                      ),
                    ),
                    const SizedBox(height: 16),
                    
                    // Max images
                    Row(
                      children: [
                        const Text('Max Images:'),
                        Expanded(
                          child: Slider(
                            value: _maxCacheSize.toDouble(),
                            min: 20,
                            max: 300,
                            divisions: 14,
                            onChanged: (value) {
                              setState(() {
                                _maxCacheSize = value.round();
                              });
                              _applyCacheConfig();
                            },
                          ),
                        ),
                        Text('$_maxCacheSize'),
                      ],
                    ),
                    
                    // Cache size
                    Row(
                      children: [
                        const Text('Cache Size (MB):'),
                        Expanded(
                          child: Slider(
                            value: _maxCacheSizeBytes / (1024 * 1024),
                            min: 10,
                            max: 100,
                            divisions: 9,
                            onChanged: (value) {
                              setState(() {
                                _maxCacheSizeBytes = (value * 1024 * 1024).round();
                              });
                              _applyCacheConfig();
                            },
                          ),
                        ),
                        Text('${_maxCacheSizeBytes ~/ (1024 * 1024)} MB'),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 6. Cache statistics
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'Cache Statistics',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 18,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceAround,
                      children: [
                        _buildStat('Hits', _cacheHits),
                        _buildStat('Misses', _cacheMisses),
                        _buildStat(
                          'Hit Rate',
                          _cacheHits + _cacheMisses > 0
                              ? '${(_cacheHits / (_cacheHits + _cacheMisses) * 100).round()}%'
                              : '0%',
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 7. Cache actions
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: () {
                      PaintingBinding.instance.imageCache.clear();
                      _updateCacheStats();
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          content: Text('Cache cleared'),
                        ),
                      );
                    },
                    child: const Text('Clear Cache'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _updateCacheStats,
                    child: const Text('Refresh Stats'),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildStat(String label, dynamic value) {
    return Column(
      children: [
        Text(
          value.toString(),
          style: const TextStyle(
            fontSize: 20,
            fontWeight: FontWeight.bold,
          ),
        ),
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
- maximumSize: Max cached images
- maximumSizeBytes: Max cache memory
- Configure cache limits dynamically
- Track cache hits and misses

---

# Data Caching

> **Caching JSON** and other data.

```dart
/// Data caching example
class DataCachingExample extends StatefulWidget {
  const DataCachingExample({super.key});

  @override
  State<DataCachingExample> createState() => _DataCachingExampleState();
}

class _DataCachingExampleState extends State<DataCachingExample> {
  // 1. Simple in-memory cache
  final Map<String, dynamic> _cache = {};
  String _status = 'Ready';
  String _lastLoaded = 'None';

  // 2. Load data with caching
  Future<void> _loadData(String key) async {
    setState(() {
      _status = 'Loading...';
    });

    // Check cache first
    if (_cache.containsKey(key)) {
      setState(() {
        _status = 'Loaded from cache';
        _lastLoaded = '$key (cached)';
        _displayData(_cache[key] as Map<String, dynamic>);
      });
      return;
    }

    try {
      // Simulate network request
      await Future.delayed(const Duration(seconds: 2));
      
      // Simulated data
      final data = {
        'id': key,
        'timestamp': DateTime.now().toString(),
        'value': 'Data for $key',
        'random': (100 * DateTime.now().millisecondsSinceEpoch % 100).round(),
      };
      
      // Cache the data
      _cache[key] = data;
      
      setState(() {
        _status = 'Loaded from network';
        _lastLoaded = '$key (network)';
        _displayData(data);
      });
    } catch (e) {
      setState(() {
        _status = 'Error: $e';
      });
    }
  }

  // 3. Display data
  void _displayData(Map<String, dynamic> data) {
    // In a real app, you'd display this in the UI
    print('Data loaded: $data');
  }

  // 4. Clear cache
  void _clearCache() {
    _cache.clear();
    setState(() {
      _status = 'Cache cleared';
      _lastLoaded = 'None';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Data Caching'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 5. Cache info
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('Cache Size:', style: TextStyle(fontWeight: FontWeight.bold)),
                      Text('${_cache.length} items'),
                    ],
                  ),
                  const SizedBox(height: 4),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('Status:', style: TextStyle(fontWeight: FontWeight.bold)),
                      Text(_status),
                    ],
                  ),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('Last Loaded:', style: TextStyle(fontWeight: FontWeight.bold)),
                      Text(_lastLoaded),
                    ],
                  ),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 6. Load buttons
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                _buildLoadButton('Data 1', 'key1'),
                _buildLoadButton('Data 2', 'key2'),
                _buildLoadButton('Data 3', 'key3'),
                _buildLoadButton('Data 1 (again)', 'key1'),
              ],
            ),
            
            const SizedBox(height: 16),
            
            // 7. Cache actions
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _clearCache,
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.red,
                    ),
                    child: const Text('Clear Cache'),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildLoadButton(String label, String key) {
    return ElevatedButton(
      onPressed: () => _loadData(key),
      child: Text(label),
    );
  }
}
```

What's happening here?
- Simple in-memory cache
- Check cache before network request
- Data loaded from cache or network
- Cache status tracking

---

# Real-World Examples

> **Common patterns** with caching.

```dart
/// 1. Cached network image widget
class CachedNetworkImageWidget extends StatelessWidget {
  const CachedNetworkImageWidget({
    super.key,
    required this.url,
    this.width,
    this.height,
    this.fit,
  });

  final String url;
  final double? width;
  final double? height;
  final BoxFit? fit;

  @override
  Widget build(BuildContext context) {
    return Image.network(
      url,
      width: width,
      height: height,
      fit: fit,
      // Use cacheKey for better caching
      cacheKey: url,
      // Optimize cache size
      cacheWidth: width != null ? width!.round() : null,
      cacheHeight: height != null ? height!.round() : null,
      loadingBuilder: (context, child, loadingProgress) {
        if (loadingProgress == null) return child;
        return Container(
          width: width,
          height: height,
          color: Colors.grey[200],
          child: const Center(
            child: CircularProgressIndicator(),
          ),
        );
      },
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

/// 2. Cache service
class CacheService {
  // Singleton pattern
  static final CacheService _instance = CacheService._internal();
  factory CacheService() => _instance;
  CacheService._internal();

  // In-memory cache
  final Map<String, dynamic> _memoryCache = {};
  final Map<String, DateTime> _cacheExpiry = {};
  
  // Cache duration
  Duration _defaultDuration = const Duration(hours: 1);

  // Store data in cache
  void set(String key, dynamic value, {Duration? duration}) {
    _memoryCache[key] = value;
    _cacheExpiry[key] = DateTime.now().add(duration ?? _defaultDuration);
  }

  // Get data from cache
  dynamic get(String key) {
    if (!_memoryCache.containsKey(key)) {
      return null;
    }
    
    // Check expiry
    if (DateTime.now().isAfter(_cacheExpiry[key]!)) {
      _memoryCache.remove(key);
      _cacheExpiry.remove(key);
      return null;
    }
    
    return _memoryCache[key];
  }

  // Remove from cache
  void remove(String key) {
    _memoryCache.remove(key);
    _cacheExpiry.remove(key);
  }

  // Clear all cache
  void clear() {
    _memoryCache.clear();
    _cacheExpiry.clear();
  }

  // Get cache size
  int get size => _memoryCache.length;
}

/// 3. Using cache service
class CacheServiceExample extends StatefulWidget {
  const CacheServiceExample({super.key});

  @override
  State<CacheServiceExample> createState() => _CacheServiceExampleState();
}

class _CacheServiceExampleState extends State<CacheServiceExample> {
  final CacheService _cache = CacheService();
  String _displayData = 'No data';
  String _source = '';

  @override
  void initState() {
    super.initState();
    _loadData();
  }

  Future<void> _loadData() async {
    // Try cache first
    final cachedData = _cache.get('user_data');
    if (cachedData != null) {
      setState(() {
        _displayData = cachedData.toString();
        _source = 'Cache';
      });
      return;
    }

    // Load from network
    try {
      await Future.delayed(const Duration(seconds: 1));
      final data = {
        'name': 'John Doe',
        'age': 30,
        'timestamp': DateTime.now().toString(),
      };
      
      // Store in cache with 30-second expiry
      _cache.set('user_data', data, duration: const Duration(seconds: 30));
      
      setState(() {
        _displayData = data.toString();
        _source = 'Network';
      });
    } catch (e) {
      setState(() {
        _displayData = 'Error: $e';
        _source = 'Error';
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Cache Service'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _loadData,
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    Text(
                      'Source: $_source',
                      style: const TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    Text(_displayData),
                    const SizedBox(height: 8),
                    Text(
                      'Cache Size: ${_cache.size} items',
                      style: TextStyle(
                        color: Colors.grey[600],
                        fontSize: 12,
                      ),
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                _cache.clear();
                setState(() {
                  _displayData = 'Cache cleared';
                  _source = 'Cleared';
                });
              },
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.red,
              ),
              child: const Text('Clear Cache'),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- CachedNetworkImageWidget with cache optimization
- CacheService for data caching
- Cache with expiry times
- Singleton cache pattern

---

# Best Practices

## Use Appropriate Cache Size

```dart
// Good - Set reasonable cache limits
PaintingBinding.instance.imageCache.maximumSize = 200;
PaintingBinding.instance.imageCache.maximumSizeBytes = 50 * 1024 * 1024;
```

## Implement Cache Expiry

```dart
// Good - Cache with expiry
class CacheItem {
  final dynamic data;
  final DateTime expiry;
  
  bool get isExpired => DateTime.now().isAfter(expiry);
}
```

## Use Cache Keys

```dart
// Good - Use cache keys
Image.network(
  url,
  cacheKey: '${url}_${width}_${height}',
)
```

---

# Common Mistakes

## No Cache Management

Wrong:
```dart
// No cache limits
// Cache can grow indefinitely
```

Correct:
```dart
// Set cache limits
PaintingBinding.instance.imageCache.maximumSize = 200;
```

## Not Checking Cache

Wrong:
```dart
// Always loads from network
Future loadData() {
  return fetchFromNetwork();
}
```

Correct:
```dart
// Check cache first
Future loadData() {
  if (cache.has(key)) return cache.get(key);
  return fetchFromNetwork();
}
```

---

# Summary

Caching improves performance by storing frequently used data locally. Use image caching for images, in-memory caching for data, and implement cache expiry for freshness. Optimize cache size and use appropriate caching strategies for your application.

---

# Next Steps

- [SVG](svg.md)
- [Asset Bundles](asset-bundles.md)
- [Image Widget](image-widget.md)

---

# Did You Know?

- PaintingBinding.instance.imageCache manages images
- maximumSize controls cache image count
- maximumSizeBytes controls cache memory
- cacheKey identifies images
- In-memory caching speeds up data access
- Cache expiry prevents stale data
- Singleton pattern for cache service
- Cache improves app performance