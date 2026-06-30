# AspectRatio

Understand how to maintain a specific aspect ratio for widgets.

---

# What is it?

AspectRatio is a widget that attempts to size its child to a specific aspect ratio. The aspect ratio is expressed as a ratio of width to height (e.g., 16/9 for widescreen, 1/1 for square). AspectRatio is useful for creating widgets that maintain their proportions regardless of the available space.

---

# Why does it exist?

AspectRatio exists to:

- Maintain consistent proportions
- Create square widgets
- Support video and image containers
- Build responsive designs
- Handle different screen sizes
- Preserve aspect ratios in layouts
- Create uniform grid items

---

# Basic AspectRatio

> **AspectRatio** maintains width-to-height proportions.

```dart
// Basic AspectRatio usage
class BasicAspectRatio extends StatelessWidget {
  const BasicAspectRatio({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Square (1:1)
        Container(
          width: 200,
          color: Colors.grey[200],
          child: const AspectRatio(
            aspectRatio: 1.0,
            child: ColoredBox(
              color: Colors.blue,
              child: Center(
                child: Text('1:1 Square'),
              ),
            ),
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. Widescreen (16:9)
        Container(
          width: 300,
          color: Colors.grey[200],
          child: const AspectRatio(
            aspectRatio: 16 / 9,
            child: ColoredBox(
              color: Colors.green,
              child: Center(
                child: Text('16:9 Widescreen'),
              ),
            ),
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 3. Portrait (3:4)
        Container(
          height: 200,
          color: Colors.grey[200],
          child: const AspectRatio(
            aspectRatio: 3 / 4,
            child: ColoredBox(
              color: Colors.red,
              child: Center(
                child: Text('3:4 Portrait'),
              ),
            ),
          ),
        ),
      ],
    );
  }
}

// AspectRatio properties:
// 1. aspectRatio - Width/Height ratio
// 2. child - The widget to size
```

What's happening here?
- aspectRatio controls proportions
- 1.0 = square
- 16/9 = widescreen
- 3/4 = portrait
- Child fills the space

---

# AspectRatio Calculations

> **How AspectRatio** determines size.

```dart
// AspectRatio calculations
class AspectRatioCalculations extends StatelessWidget {
  const AspectRatioCalculations({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Width constrained (height calculated)
        Container(
          width: 200,
          color: Colors.grey[200],
          child: const AspectRatio(
            aspectRatio: 2.0,
            child: ColoredBox(
              color: Colors.blue,
              child: Center(
                child: Text('Width 200, Height 100'),
              ),
            ),
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. Height constrained (width calculated)
        Container(
          height: 100,
          color: Colors.grey[200],
          child: const AspectRatio(
            aspectRatio: 2.0,
            child: ColoredBox(
              color: Colors.green,
              child: Center(
                child: Text('Height 100, Width 200'),
              ),
            ),
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 3. Both constrained (fits within)
        Container(
          width: 300,
          height: 200,
          color: Colors.grey[200],
          child: const AspectRatio(
            aspectRatio: 1.0,
            child: ColoredBox(
              color: Colors.red,
              child: Center(
                child: Text('Fits within 300x200'),
              ),
            ),
          ),
        ),
      ],
    );
  }
}

// Calculation rules:
// 1. If width is constrained, height = width / aspectRatio
// 2. If height is constrained, width = height * aspectRatio
// 3. If both constrained, child is centered within available space
// 4. If neither constrained, uses parent constraints
```

What's happening here?
- Width constrained → height calculated
- Height constrained → width calculated
- Both constrained → fits within
- Respects parent constraints

---

# AspectRatio with Images

> **AspectRatio** is perfect for image containers.

```dart
// AspectRatio with images
class AspectRatioImages extends StatelessWidget {
  const AspectRatioImages({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Square image
        const AspectRatio(
          aspectRatio: 1.0,
          child: ColoredBox(
            color: Colors.blue,
            child: Center(
              child: Icon(
                Icons.image,
                size: 50,
                color: Colors.white,
              ),
            ),
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. 16:9 image container
        const AspectRatio(
          aspectRatio: 16 / 9,
          child: ColoredBox(
            color: Colors.green,
            child: Center(
              child: Text(
                '16:9 Image',
                style: TextStyle(color: Colors.white),
              ),
            ),
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 3. 4:3 image container
        Container(
          width: 300,
          color: Colors.grey[200],
          child: const AspectRatio(
            aspectRatio: 4 / 3,
            child: ColoredBox(
              color: Colors.red,
              child: Center(
                child: Text(
                  '4:3 Image',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- Maintains image proportions
- Square, widescreen, standard
- Responsive to container size
- Perfect for image grids

---

# AspectRatio in Grids

> **AspectRatio** creates uniform grid items.

```dart
// AspectRatio in grids
class AspectRatioGrid extends StatelessWidget {
  const AspectRatioGrid({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Grid with square items
        SizedBox(
          height: 200,
          child: GridView.count(
            crossAxisCount: 3,
            padding: const EdgeInsets.all(8),
            crossAxisSpacing: 8,
            mainAxisSpacing: 8,
            children: [
              for (int i = 0; i < 6; i++)
                const AspectRatio(
                  aspectRatio: 1.0,
                  child: ColoredBox(
                    color: Colors.blue,
                    child: Center(
                      child: Text(
                        'Square',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                ),
            ],
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. Grid with rectangular items
        SizedBox(
          height: 150,
          child: GridView.count(
            crossAxisCount: 2,
            padding: const EdgeInsets.all(8),
            crossAxisSpacing: 8,
            mainAxisSpacing: 8,
            children: [
              for (int i = 0; i < 4; i++)
                const AspectRatio(
                  aspectRatio: 2 / 1,
                  child: ColoredBox(
                    color: Colors.green,
                    child: Center(
                      child: Text(
                        '2:1',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
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
- Uniform grid items
- Consistent proportions
- Responsive grid layout
- Clean and organized

---

# Real-World Examples

> **Common patterns** using AspectRatio.

```dart
// 1. Video player container
class VideoPlayerContainer extends StatelessWidget {
  const VideoPlayerContainer({super.key});

  @override
  Widget build(BuildContext context) {
    return AspectRatio(
      aspectRatio: 16 / 9,
      child: Container(
        color: Colors.black,
        child: const Center(
          child: Icon(
            Icons.play_circle_outline,
            size: 64,
            color: Colors.white,
          ),
        ),
      ),
    );
  }
}

// 2. Square profile picture
class SquareProfilePicture extends StatelessWidget {
  const SquareProfilePicture({super.key});

  @override
  Widget build(BuildContext context) {
    return AspectRatio(
      aspectRatio: 1.0,
      child: Container(
        decoration: BoxDecoration(
          shape: BoxShape.circle,
          color: Colors.blue,
          image: const DecorationImage(
            image: NetworkImage('https://example.com/avatar.jpg'),
            fit: BoxFit.cover,
          ),
        ),
      ),
    );
  }
}

// 3. Product card with image
class ProductCard extends StatelessWidget {
  const ProductCard({super.key});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Product image with 1:1 ratio
          const AspectRatio(
            aspectRatio: 1.0,
            child: ColoredBox(
              color: Colors.blue,
              child: Center(
                child: Icon(
                  Icons.inventory,
                  size: 64,
                  color: Colors.white,
                ),
              ),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(12),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text(
                  'Product Name',
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    fontSize: 16,
                  ),
                ),
                const SizedBox(height: 4),
                Text(
                  '\$99.99',
                  style: TextStyle(
                    color: Colors.grey[600],
                    fontSize: 14,
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

// 4. Hero image with overlay
class HeroImage extends StatelessWidget {
  const HeroImage({super.key});

  @override
  Widget build(BuildContext context) {
    return AspectRatio(
      aspectRatio: 16 / 9,
      child: Stack(
        children: [
          // Background image
          Container(
            decoration: const BoxDecoration(
              image: DecorationImage(
                image: NetworkImage('https://example.com/hero.jpg'),
                fit: BoxFit.cover,
              ),
            ),
          ),
          // Gradient overlay
          Container(
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
          const Padding(
            padding: EdgeInsets.all(16),
            child: Align(
              alignment: Alignment.bottomLeft,
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                mainAxisSize: MainAxisSize.min,
                children: [
                  Text(
                    'Hero Title',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  Text(
                    'Hero subtitle goes here',
                    style: TextStyle(
                      color: Colors.white70,
                      fontSize: 16,
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Video player container
- Square profile picture
- Product card with image
- Hero image with overlay

---

# Best Practices

## Use for Consistent Proportions

```dart
// Good - Maintains aspect ratio
@override
Widget build(BuildContext context) {
  return AspectRatio(
    aspectRatio: 16 / 9,
    child: Image.network('https://example.com/image.jpg'),
  );
}
```

## Use with Grids

```dart
// Good - Uniform grid items
@override
Widget build(BuildContext context) {
  return GridView.builder(
    gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: 3,
      childAspectRatio: 1.0, // Square items
    ),
    itemBuilder: (context, index) {
      return AspectRatio(
        aspectRatio: 1.0,
        child: Container(color: Colors.blue),
      );
    },
  );
}
```

## Use for Media Containers

```dart
// Good - Video container
@override
Widget build(BuildContext context) {
  return AspectRatio(
    aspectRatio: 16 / 9,
    child: Container(
      color: Colors.black,
      child: const VideoPlayer(),
    ),
  );
}
```

---

# Common Mistakes

## Very Small Aspect Ratio

Wrong:
```dart
// Aspect ratio too small
AspectRatio(
  aspectRatio: 0.1, // Very wide
  child: Container(color: Colors.blue),
)
```

Correct:
```dart
// Reasonable aspect ratio
AspectRatio(
  aspectRatio: 16 / 9,
  child: Container(color: Colors.blue),
)
```

## Unbounded Constraints

Wrong:
```dart
// No width or height constraints
AspectRatio(
  aspectRatio: 1.0,
  child: Container(color: Colors.blue),
)
```

Correct:
```dart
// With constraints
Container(
  width: 200,
  child: AspectRatio(
    aspectRatio: 1.0,
    child: Container(color: Colors.blue),
  ),
)
```

---

# Summary

AspectRatio maintains width-to-height proportions for widgets. Use it for images, videos, profile pictures, and grid items. AspectRatio ensures consistent proportions across different screen sizes and orientations.

---

# Next Steps

- [FractionallySizedBox](fractionallysizedbox.md)
- [Intrinsic Widgets](intrinsic-widgets.md)
- [Custom Layouts](custom-layouts.md)

---

# Did You Know?

- AspectRatio 1.0 creates a square
- AspectRatio calculates missing dimension
- AspectRatio respects parent constraints
- AspectRatio is used in many media widgets
- AspectRatio works with any child
- AspectRatio helps maintain proportions
- AspectRatio is responsive by default
- AspectRatio can be used in GridView