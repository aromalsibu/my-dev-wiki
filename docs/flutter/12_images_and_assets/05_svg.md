# SVG

Understand how to display and use SVG (Scalable Vector Graphics) in Flutter applications.

---

# What is it?

SVG (Scalable Vector Graphics) is an XML-based vector image format for two-dimensional graphics. SVG files are resolution-independent, meaning they scale perfectly to any size without losing quality. In Flutter, SVG support is provided through the flutter_svg package, which allows you to render SVG images and icons in your applications.

---

# Why does it exist?

SVG exists to:

- Display resolution-independent graphics
- Support vector icons and illustrations
- Enable scalable and responsive images
- Reduce app size compared to raster images
- Support animations and interactivity
- Maintain quality at any size
- Provide flexibility for design changes

---

# Setting Up SVG

> **Adding SVG support** to your project.

```yaml
# 1. Add flutter_svg to pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_svg: ^2.0.9

# 2. Declare SVG assets
flutter:
  assets:
    - assets/svgs/
    - assets/svgs/logo.svg
    - assets/svgs/icons/
    - assets/svgs/illustrations/
```

```dart
// 3. Import the package
import 'package:flutter/material.dart';
import 'package:flutter_svg/flutter_svg.dart';

/// Basic SVG example
class BasicSvgExample extends StatelessWidget {
  const BasicSvgExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('SVG Example'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 4. Load SVG from asset
            const Text(
              'SVG from Asset',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            SvgPicture.asset(
              'assets/svgs/logo.svg',
              width: 100,
              height: 100,
            ),
            const SizedBox(height: 24),
            
            // 5. Load SVG from network
            const Text(
              'SVG from Network',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            SvgPicture.network(
              'https://dev.w3.org/SVG/tools/svgweb/samples/svg-files/debian.svg',
              width: 100,
              height: 100,
            ),
            const SizedBox(height: 24),
            
            // 6. Load SVG from string
            const Text(
              'SVG from String',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            SvgPicture.string(
              '''<svg viewBox="0 0 100 100">
                <circle cx="50" cy="50" r="40" fill="blue" />
                <text x="50" y="55" text-anchor="middle" fill="white">
                  SVG
                </text>
              </svg>''',
              width: 100,
              height: 100,
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- flutter_svg package adds SVG support
- SvgPicture.asset loads from assets
- SvgPicture.network loads from URL
- SvgPicture.string loads from XML string

---

# SVG Properties

> **Customizing SVG** appearance.

```dart
/// SVG properties example
class SvgPropertiesExample extends StatelessWidget {
  const SvgPropertiesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('SVG Properties'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Basic SVG with size
            const Text(
              'Basic SVG',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            SvgPicture.asset(
              'assets/svgs/logo.svg',
              width: 100,
              height: 100,
            ),
            const SizedBox(height: 16),
            
            // 2. SVG with color
            const Text(
              'SVG with Color',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildColoredSvg(Colors.blue),
                _buildColoredSvg(Colors.red),
                _buildColoredSvg(Colors.green),
              ],
            ),
            const SizedBox(height: 16),
            
            // 3. SVG with fit
            const Text(
              'SVG with Fit',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Container(
              width: 200,
              height: 100,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey),
              ),
              child: SvgPicture.asset(
                'assets/svgs/logo.svg',
                fit: BoxFit.cover,
              ),
            ),
            const SizedBox(height: 16),
            
            // 4. SVG with alignment
            const Text(
              'SVG with Alignment',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Container(
              width: 200,
              height: 100,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey),
              ),
              child: SvgPicture.asset(
                'assets/svgs/logo.svg',
                alignment: Alignment.centerRight,
                fit: BoxFit.contain,
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildColoredSvg(Color color) {
    return SvgPicture.asset(
      'assets/svgs/logo.svg',
      colorFilter: ColorFilter.mode(
        color,
        BlendMode.srcIn,
      ),
      width: 50,
      height: 50,
    );
  }
}
```

What's happening here?
- colorFilter changes SVG color
- fit controls sizing
- alignment positions the SVG
- width/height set dimensions

---

# SVG Icons

> **Using SVG as** icons.

```dart
/// SVG icons example
class SvgIconsExample extends StatelessWidget {
  const SvgIconsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('SVG Icons'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. SVG as icon
            const Text(
              'SVG Icons',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildIcon('assets/svgs/icons/home.svg', 'Home'),
                _buildIcon('assets/svgs/icons/search.svg', 'Search'),
                _buildIcon('assets/svgs/icons/settings.svg', 'Settings'),
                _buildIcon('assets/svgs/icons/person.svg', 'Profile'),
              ],
            ),
            const SizedBox(height: 24),
            
            // 2. Colored icons
            const Text(
              'Colored Icons',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildColoredIcon('assets/svgs/icons/heart.svg', Colors.red),
                _buildColoredIcon('assets/svgs/icons/star.svg', Colors.yellow),
                _buildColoredIcon('assets/svgs/icons/check.svg', Colors.green),
              ],
            ),
            const SizedBox(height: 24),
            
            // 3. Icon with size
            const Text(
              'Different Sizes',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildSizedIcon('assets/svgs/icons/camera.svg', 24),
                _buildSizedIcon('assets/svgs/icons/camera.svg', 48),
                _buildSizedIcon('assets/svgs/icons/camera.svg', 72),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildIcon(String assetPath, String label) {
    return Column(
      children: [
        SvgPicture.asset(
          assetPath,
          width: 40,
          height: 40,
        ),
        const SizedBox(height: 4),
        Text(
          label,
          style: const TextStyle(fontSize: 12),
        ),
      ],
    );
  }

  Widget _buildColoredIcon(String assetPath, Color color) {
    return SvgPicture.asset(
      assetPath,
      colorFilter: ColorFilter.mode(
        color,
        BlendMode.srcIn,
      ),
      width: 40,
      height: 40,
    );
  }

  Widget _buildSizedIcon(String assetPath, double size) {
    return SvgPicture.asset(
      assetPath,
      width: size,
      height: size,
    );
  }
}
```

What's happening here?
- SVGs as scalable icons
- Color filtering for customization
- Size variations
- Icon labels

---

# Real-World Examples

> **Common patterns** with SVG.

```dart
/// 1. SVG button
class SvgButton extends StatelessWidget {
  const SvgButton({
    super.key,
    required this.iconPath,
    required this.label,
    required this.onPressed,
    this.color = Colors.blue,
  });

  final String iconPath;
  final String label;
  final VoidCallback onPressed;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return ElevatedButton.icon(
      onPressed: onPressed,
      icon: SvgPicture.asset(
        iconPath,
        width: 20,
        height: 20,
        colorFilter: ColorFilter.mode(
          Colors.white,
          BlendMode.srcIn,
        ),
      ),
      label: Text(label),
      style: ElevatedButton.styleFrom(
        backgroundColor: color,
      ),
    );
  }
}

/// 2. SVG loader
class SvgLoader extends StatelessWidget {
  const SvgLoader({
    super.key,
    required this.assetPath,
    this.size = 50,
    this.color,
  });

  final String assetPath;
  final double size;
  final Color? color;

  @override
  Widget build(BuildContext context) {
    return SvgPicture.asset(
      assetPath,
      width: size,
      height: size,
      colorFilter: color != null
          ? ColorFilter.mode(
              color!,
              BlendMode.srcIn,
            )
          : null,
    );
  }
}

/// 3. SVG with placeholder
class SvgWithPlaceholder extends StatelessWidget {
  const SvgWithPlaceholder({
    super.key,
    required this.assetPath,
    this.width,
    this.height,
    this.fit,
  });

  final String assetPath;
  final double? width;
  final double? height;
  final BoxFit? fit;

  @override
  Widget build(BuildContext context) {
    return SvgPicture.asset(
      assetPath,
      width: width,
      height: height,
      fit: fit ?? BoxFit.contain,
      placeholderBuilder: (context) {
        return Container(
          width: width,
          height: height,
          color: Colors.grey[200],
          child: const Center(
            child: CircularProgressIndicator(),
          ),
        );
      },
    );
  }
}

/// 4. SVG illustration
class SvgIllustration extends StatelessWidget {
  const SvgIllustration({
    super.key,
    required this.assetPath,
    required this.title,
    required this.description,
  });

  final String assetPath;
  final String title;
  final String description;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      child: Column(
        children: [
          SvgPicture.asset(
            assetPath,
            width: 200,
            height: 200,
          ),
          const SizedBox(height: 16),
          Text(
            title,
            style: const TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          const SizedBox(height: 8),
          Text(
            description,
            textAlign: TextAlign.center,
            style: TextStyle(
              color: Colors.grey[600],
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- SVG in buttons
- SVG loader widget
- SVG with placeholder
- SVG illustrations with text

---

# Best Practices

## Optimize SVG Files

```dart
// Good - Optimized SVG
// Use tools to compress SVG files
// Remove unnecessary metadata

// Bad - Unoptimized SVG
// Large file sizes
// Contains unnecessary elements
```

## Use Appropriate Sizing

```dart
// Good - Set dimensions
SvgPicture.asset(
  'assets/svgs/icon.svg',
  width: 24,
  height: 24,
)

// Bad - No dimensions
SvgPicture.asset('assets/svgs/icon.svg')
```

## Cache SVG Assets

```dart
// Good - Cache SVG for performance
SvgPicture.asset(
  'assets/svgs/logo.svg',
  // Caching is handled automatically by flutter_svg
)
```

---

# Common Mistakes

## Wrong Asset Path

Wrong:
```dart
// Invalid path
SvgPicture.asset('images/logo.svg')
```

Correct:
```dart
// Full asset path
SvgPicture.asset('assets/svgs/logo.svg')
```

## Missing Color Filter

Wrong:
```dart
// Can't change color
SvgPicture.asset('assets/svgs/icon.svg')
```

Correct:
```dart
// With color filter
SvgPicture.asset(
  'assets/svgs/icon.svg',
  colorFilter: ColorFilter.mode(
    Colors.blue,
    BlendMode.srcIn,
  ),
)
```

---

# Summary

SVG provides resolution-independent vector graphics in Flutter. Use flutter_svg to display SVG images, customize colors with colorFilter, and optimize performance with proper sizing. SVGs are perfect for icons, illustrations, and scalable graphics.

---

# Next Steps

- [Asset Bundles](asset-bundles.md)
- [Image Widget](image-widget.md)
- [Network Images](network-images.md)

---

# Did You Know?

- SVG is resolution-independent
- flutter_svg adds SVG support
- colorFilter changes SVG colors
- SvgPicture.asset loads from assets
- SvgPicture.network loads from URLs
- SvgPicture.string loads from XML
- SVGs are great for icons
- SVGs scale without quality loss