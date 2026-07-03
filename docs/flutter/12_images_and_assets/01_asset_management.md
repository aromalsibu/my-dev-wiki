# Asset Management

Understand how to manage and use assets (images, fonts, files) in Flutter applications.

---

# What is it?

Asset Management in Flutter refers to the system of including, organizing, and accessing static resources like images, fonts, JSON files, and other data files in your application. Assets are bundled with your app and can be accessed at runtime. Flutter uses a declarative approach where you define assets in the pubspec.yaml file.

---

# Why does it exist?

Asset Management exists to:

- Bundle resources with your application
- Organize static files efficiently
- Support multiple resolutions and densities
- Enable internationalization
- Manage fonts and custom typography
- Provide a consistent way to access resources
- Optimize app size and performance

---

# Asset Declaration

> **Declaring assets** in pubspec.yaml.

```yaml
# pubspec.yaml

name: my_app
description: A Flutter app with assets

dependencies:
  flutter:
    sdk: flutter

flutter:
  # 1. Use material design
  uses-material-design: true

  # 2. Assets configuration
  assets:
    # Images
    - assets/images/
    - assets/images/logo.png
    - assets/images/profile.jpg
    
    # Icons
    - assets/icons/
    
    # JSON data
    - assets/data/config.json
    - assets/data/strings.json
    
    # Fonts
    - assets/fonts/
    
    # Other files
    - assets/files/sample.pdf
    - assets/files/data.csv

  # 3. Fonts configuration
  fonts:
    - family: CustomFont
      fonts:
        - asset: assets/fonts/CustomFont-Regular.ttf
        - asset: assets/fonts/CustomFont-Bold.ttf
          weight: 700
        - asset: assets/fonts/CustomFont-Italic.ttf
          style: italic
    
    - family: AnotherFont
      fonts:
        - asset: assets/fonts/AnotherFont-Regular.ttf
        - asset: assets/fonts/AnotherFont-Light.ttf
          weight: 300
```

What's happening here?
- assets: List of asset folders and files
- fonts: Custom font declarations
- uses-material-design: Enables Material icons
- Asset paths are relative to pubspec.yaml

---

# Asset Organization

> **Organizing assets** in your project.

```
my_app/
├── pubspec.yaml
├── lib/
│   └── main.dart
├── assets/
│   ├── images/
│   │   ├── logo.png
│   │   ├── profile.jpg
│   │   ├── 2.0x/           # For 2x resolution
│   │   │   └── logo.png
│   │   ├── 3.0x/           # For 3x resolution
│   │   │   └── logo.png
│   │   └── icons/
│   │       └── star.png
│   ├── fonts/
│   │   ├── CustomFont-Regular.ttf
│   │   └── CustomFont-Bold.ttf
│   ├── data/
│   │   ├── config.json
│   │   └── strings.json
│   └── files/
│       └── sample.pdf
└── assets/                  # Alternate location
    └── images/
        └── logo.png
```

---

# Loading Images

> **Loading images** from assets.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Asset image examples
class AssetImageExample extends StatelessWidget {
  const AssetImageExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Asset Images'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. AssetImage - Load image from assets
            // This is the most common way to load images
            const Text(
              'AssetImage',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Image.asset(
              'assets/images/logo.png',
              width: 100,
              height: 100,
            ),
            const SizedBox(height: 16),
            
            // 2. Image with box fit
            // BoxFit controls how the image fills the container
            const Text(
              'BoxFit.cover',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Container(
              width: 150,
              height: 100,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey),
              ),
              child: Image.asset(
                'assets/images/logo.png',
                fit: BoxFit.cover,
              ),
            ),
            const SizedBox(height: 16),
            
            // 3. Image with BoxFit.contain
            const Text(
              'BoxFit.contain',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Container(
              width: 150,
              height: 100,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey),
              ),
              child: Image.asset(
                'assets/images/logo.png',
                fit: BoxFit.contain,
              ),
            ),
            const SizedBox(height: 16),
            
            // 4. Image with BoxFit.fill
            const Text(
              'BoxFit.fill',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Container(
              width: 150,
              height: 100,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey),
              ),
              child: Image.asset(
                'assets/images/logo.png',
                fit: BoxFit.fill,
              ),
            ),
            const SizedBox(height: 16),
            
            // 5. Image with placeholder
            const Text(
              'Image with Placeholder',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            FadeInImage.assetNetwork(
              placeholder: 'assets/images/placeholder.png',
              image: 'https://example.com/image.jpg',
              width: 100,
              height: 100,
              fit: BoxFit.cover,
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Image.asset loads from assets
- fit controls how image fills space
- BoxFit.cover: Fills container, may crop
- BoxFit.contain: Fits inside container
- FadeInImage shows placeholder while loading

---

# Resolution-Aware Images

> **Supporting multiple** screen densities.

```dart
/// Resolution-aware images example
class ResolutionAwareImagesExample extends StatelessWidget {
  const ResolutionAwareImagesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Resolution-Aware Images'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Automatic resolution selection
            // Flutter automatically picks the right resolution
            // based on the device's pixel ratio
            const Text(
              'Automatic Resolution Selection',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Image.asset(
              'assets/images/logo.png',
              width: 100,
              height: 100,
            ),
            
            const SizedBox(height: 24),
            
            // 2. Manual resolution selection
            const Text(
              'Manual Resolution Selection',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Image.asset(
              'assets/images/logo.png',
              width: 100,
              height: 100,
              // You can also use AssetImage directly
            ),
            
            const SizedBox(height: 24),
            
            // 3. Display current device pixel ratio
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  const Text(
                    'Device Information',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    'Pixel Ratio: ${MediaQuery.of(context).devicePixelRatio}',
                  ),
                  Text(
                    'Screen Size: ${MediaQuery.of(context).size}',
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
```

What's happening here?
- Flutter automatically selects the right image based on device pixel ratio
- Images in 2.0x and 3.0x folders are used for high-density screens
- MediaQuery provides device information

---

# Loading JSON Data

> **Loading and parsing** JSON assets.

```dart
/// JSON loading example
class JsonLoadingExample extends StatefulWidget {
  const JsonLoadingExample({super.key});

  @override
  State<JsonLoadingExample> createState() => _JsonLoadingExampleState();
}

class _JsonLoadingExampleState extends State<JsonLoadingExample> {
  // 1. State variables
  Map<String, dynamic>? _data;
  bool _isLoading = true;
  String _error = '';

  @override
  void initState() {
    super.initState();
    _loadData();
  }

  // 2. Load JSON from assets
  Future<void> _loadData() async {
    setState(() {
      _isLoading = true;
      _error = '';
    });

    try {
      // Load the JSON file from assets
      final String jsonString = await rootBundle.loadString(
        'assets/data/config.json',
      );
      
      // Parse the JSON string
      final Map<String, dynamic> jsonData = json.decode(jsonString);
      
      setState(() {
        _data = jsonData;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('JSON Loading'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _loadData,
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: _isLoading
            ? const Center(child: CircularProgressIndicator())
            : _error.isNotEmpty
                ? Center(
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        const Icon(Icons.error, color: Colors.red, size: 48),
                        const SizedBox(height: 16),
                        Text('Error: $_error'),
                        const SizedBox(height: 16),
                        ElevatedButton(
                          onPressed: _loadData,
                          child: const Text('Retry'),
                        ),
                      ],
                    ),
                  )
                : _data != null
                    ? _buildDataDisplay()
                    : const Center(child: Text('No data')),
      ),
    );
  }

  Widget _buildDataDisplay() {
    return ListView(
      children: [
        // Display parsed JSON data
        Card(
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text(
                  'App Config:',
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    fontSize: 18,
                  ),
                ),
                const SizedBox(height: 8),
                Text('Title: ${_data!['title'] ?? 'N/A'}'),
                Text('Version: ${_data!['version'] ?? 'N/A'}'),
                Text('Theme: ${_data!['theme'] ?? 'N/A'}'),
                if (_data!['features'] != null) ...[
                  const SizedBox(height: 8),
                  const Text(
                    'Features:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  ...(_data!['features'] as List<dynamic>).map(
                    (feature) => Text('• $feature'),
                  ),
                ],
              ],
            ),
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- rootBundle.loadString loads JSON
- json.decode parses JSON
- Data is displayed in UI
- Error handling and loading states

---

# Custom Fonts

> **Using custom** fonts in Flutter.

```dart
/// Custom fonts example
class CustomFontsExample extends StatelessWidget {
  const CustomFontsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Fonts'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Using custom font by family name
            // The font family is defined in pubspec.yaml
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'CustomFont - Regular',
                      style: TextStyle(
                        fontFamily: 'CustomFont',
                        fontSize: 20,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      'CustomFont - Bold',
                      style: TextStyle(
                        fontFamily: 'CustomFont',
                        fontSize: 20,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      'CustomFont - Italic',
                      style: TextStyle(
                        fontFamily: 'CustomFont',
                        fontSize: 20,
                        fontStyle: FontStyle.italic,
                      ),
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 2. Using multiple fonts
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'AnotherFont - Regular',
                      style: TextStyle(
                        fontFamily: 'AnotherFont',
                        fontSize: 20,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      'AnotherFont - Light',
                      style: TextStyle(
                        fontFamily: 'AnotherFont',
                        fontSize: 20,
                        fontWeight: FontWeight.w300,
                      ),
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 3. Global font theme
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text(
                      'Using Theme Font',
                      style: TextStyle(
                        fontSize: 20,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      'This uses the theme-defined font',
                      style: Theme.of(context).textTheme.bodyLarge,
                    ),
                  ],
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
- fontFamily specifies the font
- fontWeight controls weight
- fontStyle controls italic
- Theme uses custom fonts

---

# Asset Bundles

> **Working with** asset bundles.

```dart
/// Asset bundle example
class AssetBundleExample extends StatefulWidget {
  const AssetBundleExample({super.key});

  @override
  State<AssetBundleExample> createState() => _AssetBundleExampleState();
}

class _AssetBundleExampleState extends State<AssetBundleExample> {
  String _fileContent = '';
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadFile();
  }

  // 1. Load from asset bundle
  Future<void> _loadFile() async {
    try {
      // Load a text file from assets
      final content = await rootBundle.loadString(
        'assets/files/sample.txt',
      );
      
      setState(() {
        _fileContent = content;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _fileContent = 'Error loading file: $e';
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Asset Bundle'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Display loaded file content
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'File Content:',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 16,
                      ),
                    ),
                    const SizedBox(height: 8),
                    if (_isLoading)
                      const Center(child: CircularProgressIndicator())
                    else
                      Text(
                        _fileContent,
                        style: const TextStyle(fontSize: 14),
                      ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 2. List all assets
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Available Assets:',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 16,
                      ),
                    ),
                    const SizedBox(height: 8),
                    _buildAssetList(),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildAssetList() {
    // List of common asset types
    final assets = [
      'assets/images/logo.png',
      'assets/images/profile.jpg',
      'assets/data/config.json',
      'assets/fonts/CustomFont-Regular.ttf',
    ];

    return Column(
      children: assets.map((asset) {
        return ListTile(
          leading: const Icon(Icons.insert_drive_file),
          title: Text(
            asset,
            style: const TextStyle(fontSize: 12),
          ),
          dense: true,
        );
      }).toList(),
    );
  }
}
```

What's happening here?
- rootBundle loads assets
- loadString loads text files
- Asset bundles bundle all assets
- Files can be loaded at runtime

---

# Best Practices

## Organize Assets Properly

```dart
// Good - Organized structure
assets/
  images/
    logo.png
    icons/
      home.png
  fonts/
    CustomFont.ttf
  data/
    config.json

// Bad - Disorganized
assets/
  image1.png
  font1.ttf
  data.json
  image2.png
```

## Use Resolution-Aware Images

```dart
// Good - Resolution-aware
assets/images/
  logo.png
  2.0x/
    logo.png
  3.0x/
    logo.png

// Bad - Single resolution
assets/images/
  logo.png
```

## Load Assets Efficiently

```dart
// Good - Cache loaded data
class DataService {
  Map<String, dynamic>? _cachedData;
  
  Future<Map<String, dynamic>> loadData() async {
    if (_cachedData != null) return _cachedData!;
    final jsonString = await rootBundle.loadString('assets/data.json');
    _cachedData = json.decode(jsonString);
    return _cachedData!;
  }
}
```

---

# Common Mistakes

## Wrong Asset Path

Wrong:
```dart
// Path doesn't exist
Image.asset('images/logo.png')
```

Correct:
```dart
// Full path from assets folder
Image.asset('assets/images/logo.png')
```

## Missing Asset Declaration

Wrong:
```dart
// Asset not declared in pubspec.yaml
Image.asset('assets/images/logo.png')
```

Correct:
```dart
// Declared in pubspec.yaml
flutter:
  assets:
    - assets/images/logo.png
```

---

# Summary

Asset Management handles static resources in Flutter apps. Declare assets in pubspec.yaml, organize them in folders, and load them at runtime. Use resolution-aware images for multiple screen densities, load JSON data with rootBundle, and use custom fonts with proper declarations.

---

# Next Steps

- [Image Widget](image-widget.md)
- [Network Images](network-images.md)
- [Caching](caching.md)

---

# Did You Know?

- Assets are bundled with the app
- Flutter automatically selects the right image resolution
- rootBundle loads asset data
- pubspec.yaml declares all assets
- Fonts must be declared in pubspec.yaml
- Assets can be loaded at runtime
- Asset paths are case-sensitive
- Assets are optimized during build