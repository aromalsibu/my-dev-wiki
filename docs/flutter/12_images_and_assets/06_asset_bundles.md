# Asset Bundles

Understand how to work with asset bundles in Flutter applications.

---

# What is it?

Asset Bundles are the containers that hold all the resources (images, fonts, JSON files, etc.) that are packaged with a Flutter application. The Asset Bundle system provides a unified way to access these resources at runtime. Every Flutter app has a root bundle that contains all declared assets, and plugins can also have their own bundles.

---

# Why does it exist?

Asset Bundles exist to:

- Package resources with the application
- Provide a unified API for asset access
- Support platform-specific resources
- Enable resource loading at runtime
- Optimize asset delivery
- Support internationalization
- Allow plugin asset access

---

# Understanding Asset Bundles

> **Working with** the root asset bundle.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

/// Asset bundle example
class AssetBundleExample extends StatefulWidget {
  const AssetBundleExample({super.key});

  @override
  State<AssetBundleExample> createState() => _AssetBundleExampleState();
}

class _AssetBundleExampleState extends State<AssetBundleExample> {
  // 1. Track loaded assets
  String _imageInfo = '';
  String _jsonData = '';
  String _textFile = '';
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadAssets();
  }

  // 2. Load multiple assets
  Future<void> _loadAssets() async {
    // Get the root bundle
    final bundle = rootBundle;

    try {
      // 3. Load image info
      final imageData = await bundle.load('assets/images/logo.png');
      
      // 4. Load JSON data
      final jsonString = await bundle.loadString('assets/data/config.json');
      
      // 5. Load text file
      final textString = await bundle.loadString('assets/files/sample.txt');

      setState(() {
        _imageInfo = 'Image size: ${imageData.lengthInBytes} bytes';
        _jsonData = jsonString;
        _textFile = textString;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _imageInfo = 'Error loading assets: $e';
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Asset Bundles'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _loadAssets,
          ),
        ],
      ),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : Padding(
              padding: const EdgeInsets.all(16),
              child: ListView(
                children: [
                  // 6. Image info
                  _buildInfoCard('Image Info', _imageInfo),
                  
                  // 7. JSON data
                  _buildInfoCard('JSON Data', _jsonData),
                  
                  // 8. Text file
                  _buildInfoCard('Text File', _textFile),
                ],
              ),
            ),
    );
  }

  Widget _buildInfoCard(String title, String content) {
    return Card(
      margin: const EdgeInsets.only(bottom: 16),
      child: Padding(
        padding: const EdgeInsets.all(16),
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
            const SizedBox(height: 8),
            Text(
              content,
              style: const TextStyle(fontSize: 14),
              softWrap: true,
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- rootBundle provides access to assets
- load() loads raw data
- loadString() loads text content
- Assets are accessed by path

---

# Custom Asset Bundles

> **Creating and using** custom bundles.

```dart
/// Custom asset bundle example
class CustomAssetBundleExample extends StatefulWidget {
  const CustomAssetBundleExample({super.key});

  @override
  State<CustomAssetBundleExample> createState() => _CustomAssetBundleExampleState();
}

class _CustomAssetBundleExampleState extends State<CustomAssetBundleExample> {
  // 1. Custom bundle for plugin assets
  static const AssetBundle _pluginBundle = PlatformAssetBundle();

  // 2. Track loaded data
  String _pluginData = '';
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadPluginAssets();
  }

  // 3. Load from custom bundle
  Future<void> _loadPluginAssets() async {
    try {
      // Load from plugin bundle
      final data = await _pluginBundle.loadString(
        'packages/your_plugin/assets/data.json',
      );
      
      setState(() {
        _pluginData = data;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _pluginData = 'Error loading plugin assets: $e';
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Asset Bundles'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 4. Bundle info
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Bundle Information',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 16,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text('Root Bundle: ${rootBundle.runtimeType}'),
                    Text('Plugin Bundle: ${_pluginBundle.runtimeType}'),
                    Text('Platform: ${PlatformAssetBundle().runtimeType}'),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 5. Plugin data
            Expanded(
              child: Card(
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text(
                        'Plugin Asset Data',
                        style: TextStyle(
                          fontWeight: FontWeight.bold,
                          fontSize: 16,
                        ),
                      ),
                      const SizedBox(height: 8),
                      Expanded(
                        child: _isLoading
                            ? const Center(child: CircularProgressIndicator())
                            : SingleChildScrollView(
                                child: Text(
                                  _pluginData,
                                  style: const TextStyle(fontSize: 14),
                                ),
                              ),
                      ),
                    ],
                  ),
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
- PlatformAssetBundle creates custom bundles
- Plugin assets use packages/ prefix
- Multiple bundles can exist
- Each bundle can have different assets

---

# Asset Manifest

> **Understanding** asset structure.

```dart
/// Asset manifest example
class AssetManifestExample extends StatefulWidget {
  const AssetManifestExample({super.key});

  @override
  State<AssetManifestExample> createState() => _AssetManifestExampleState();
}

class _AssetManifestExampleState extends State<AssetManifestExample> {
  // 1. Track assets
  List<String> _assets = [];
  Map<String, dynamic> _assetSizes = {};
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadAssetManifest();
  }

  // 2. Load asset manifest
  Future<void> _loadAssetManifest() async {
    try {
      // 3. Load asset manifest
      final manifest = await rootBundle.loadString('AssetManifest.json');
      final List<dynamic> assets = json.decode(manifest);
      
      setState(() {
        _assets = assets.map((e) => e.toString()).toList();
        _isLoading = false;
      });

      // 4. Get sizes of assets
      for (final asset in _assets) {
        try {
          final data = await rootBundle.load(asset);
          _assetSizes[asset] = data.lengthInBytes;
        } catch (e) {
          _assetSizes[asset] = 'Error';
        }
      }

      setState(() {});
    } catch (e) {
      setState(() {
        _isLoading = false;
      });
      print('Error loading asset manifest: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Asset Manifest'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _loadAssetManifest,
          ),
        ],
      ),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : _assets.isEmpty
              ? const Center(child: Text('No assets found'))
              : ListView.builder(
                  itemCount: _assets.length,
                  itemBuilder: (context, index) {
                    final asset = _assets[index];
                    final size = _assetSizes[asset];
                    final sizeString = size is int
                        ? '${(size / 1024).toStringAsFixed(1)} KB'
                        : 'Unknown';

                    return Card(
                      margin: const EdgeInsets.symmetric(
                        horizontal: 16,
                        vertical: 4,
                      ),
                      child: ListTile(
                        leading: _getIconForAsset(asset),
                        title: Text(
                          asset,
                          style: const TextStyle(fontSize: 12),
                        ),
                        trailing: Text(
                          sizeString,
                          style: TextStyle(
                            color: Colors.grey[600],
                            fontSize: 12,
                          ),
                        ),
                      ),
                    );
                  },
                ),
    );
  }

  Icon _getIconForAsset(String asset) {
    if (asset.endsWith('.png') || asset.endsWith('.jpg')) {
      return const Icon(Icons.image, color: Colors.blue);
    } else if (asset.endsWith('.json')) {
      return const Icon(Icons.data_array, color: Colors.orange);
    } else if (asset.endsWith('.ttf') || asset.endsWith('.otf')) {
      return const Icon(Icons.text_format, color: Colors.purple);
    } else if (asset.endsWith('.svg')) {
      return const Icon(Icons.vector, color: Colors.green);
    } else {
      return const Icon(Icons.insert_drive_file, color: Colors.grey);
    }
  }
}
```

What's happening here?
- AssetManifest.json lists all assets
- Asset sizes can be checked
- Different icons for different asset types
- Asset discovery at runtime

---

# Real-World Examples

> **Common patterns** with asset bundles.

```dart
/// 1. Asset service
class AssetService {
  // Singleton
  static final AssetService _instance = AssetService._internal();
  factory AssetService() => _instance;
  AssetService._internal();

  // Cache
  final Map<String, dynamic> _cache = {};

  // Load JSON asset
  Future<Map<String, dynamic>> loadJson(String path) async {
    if (_cache.containsKey(path)) {
      return _cache[path] as Map<String, dynamic>;
    }

    try {
      final jsonString = await rootBundle.loadString(path);
      final data = json.decode(jsonString) as Map<String, dynamic>;
      _cache[path] = data;
      return data;
    } catch (e) {
      throw Exception('Failed to load JSON from $path: $e');
    }
  }

  // Load text asset
  Future<String> loadText(String path) async {
    if (_cache.containsKey(path)) {
      return _cache[path] as String;
    }

    try {
      final text = await rootBundle.loadString(path);
      _cache[path] = text;
      return text;
    } catch (e) {
      throw Exception('Failed to load text from $path: $e');
    }
  }

  // Load image asset
  Future<ByteData> loadImage(String path) async {
    if (_cache.containsKey(path)) {
      return _cache[path] as ByteData;
    }

    try {
      final data = await rootBundle.load(path);
      _cache[path] = data;
      return data;
    } catch (e) {
      throw Exception('Failed to load image from $path: $e');
    }
  }

  // Clear cache
  void clearCache() {
    _cache.clear();
  }
}

/// 2. Multi-bundle asset loader
class MultiBundleAssetLoader extends StatelessWidget {
  const MultiBundleAssetLoader({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Multi-Bundle Assets'),
      ),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Load from root bundle
            Text('Root Bundle:'),
            SizedBox(height: 8),
            _RootAssetWidget(),
            
            SizedBox(height: 24),
            
            // Load from plugin bundle
            Text('Plugin Bundle:'),
            SizedBox(height: 8),
            _PluginAssetWidget(),
          ],
        ),
      ),
    );
  }
}

class _RootAssetWidget extends StatelessWidget {
  const _RootAssetWidget();

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<String>(
      future: rootBundle.loadString('assets/data/config.json'),
      builder: (context, snapshot) {
        if (snapshot.hasData) {
          return Text(snapshot.data!);
        } else if (snapshot.hasError) {
          return Text('Error: ${snapshot.error}');
        }
        return const CircularProgressIndicator();
      },
    );
  }
}

class _PluginAssetWidget extends StatelessWidget {
  const _PluginAssetWidget();

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<String>(
      future: const PlatformAssetBundle().loadString(
        'packages/your_plugin/assets/data.json',
      ),
      builder: (context, snapshot) {
        if (snapshot.hasData) {
          return Text(snapshot.data!);
        } else if (snapshot.hasError) {
          return Text('Error: ${snapshot.error}');
        }
        return const CircularProgressIndicator();
      },
    );
  }
}
```

What's happening here?
- AssetService with caching
- Multi-bundle asset loading
- Plugin assets with packages/ prefix
- Root bundle for app assets

---

# Best Practices

## Use Cache for Performance

```dart
// Good - Cache loaded assets
class AssetCache {
  final Map<String, dynamic> _cache = {};
  
  Future<String> loadString(String path) async {
    if (_cache.containsKey(path)) {
      return _cache[path] as String;
    }
    final data = await rootBundle.loadString(path);
    _cache[path] = data;
    return data;
  }
}
```

## Handle Errors Gracefully

```dart
// Good - Error handling
try {
  final data = await rootBundle.loadString('assets/data.json');
} catch (e) {
  print('Error loading asset: $e');
  // Use fallback data
}
```

## Use Appropriate Bundle

```dart
// Good - Use right bundle
// App assets: rootBundle
// Plugin assets: PlatformAssetBundle()
// Test assets: TestAssetBundle()
```

---

# Common Mistakes

## Wrong Asset Path

Wrong:
```dart
// Missing assets/ prefix
rootBundle.loadString('data/config.json')
```

Correct:
```dart
// With assets/ prefix
rootBundle.loadString('assets/data/config.json')
```

## Not Catching Errors

Wrong:
```dart
// No error handling
final data = await rootBundle.loadString('assets/data.json');
```

Correct:
```dart
// With error handling
try {
  final data = await rootBundle.loadString('assets/data.json');
} catch (e) {
  // Handle error
}
```

---

# Summary

Asset Bundles provide access to packaged resources. Use rootBundle for app assets, PlatformAssetBundle for plugin assets, and implement caching for performance. Asset bundles support various file types and provide a unified API for resource access.

---

# Next Steps

- [Image Widget](image-widget.md)
- [Network Images](network-images.md)
- [SVG](svg.md)

---

# Did You Know?

- rootBundle is the main asset bundle
- PlatformAssetBundle creates custom bundles
- AssetManifest.json lists all assets
- loadString loads text content
- load loads binary data
- Plugin assets use packages/ prefix
- Asset bundles support caching
- Assets are optimized during build