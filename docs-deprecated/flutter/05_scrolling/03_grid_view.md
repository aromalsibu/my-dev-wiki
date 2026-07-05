# GridView

Understand how to create scrollable grid layouts in Flutter.

---

# What is it?

GridView is a scrollable widget that displays items in a 2D grid layout. It's perfect for displaying collections of items like photos, products, icons, or any content that benefits from a grid arrangement. GridView provides multiple constructors for different use cases and efficient lazy loading.

---

# Why does it exist?

GridView exists to:

- Display items in a grid pattern
- Create gallery and collection views
- Support responsive grid layouts
- Handle large datasets efficiently
- Enable custom grid configurations
- Create visually organized content
- Support both fixed and dynamic grid sizes

---

# GridView Constructors

> **Multiple constructors** for different grid needs.

```dart
// 1. GridView.count() - Fixed number of columns
class CountGridView extends StatelessWidget {
  const CountGridView({super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.count(
      crossAxisCount: 3, // 3 columns
      padding: const EdgeInsets.all(8),
      crossAxisSpacing: 8,
      mainAxisSpacing: 8,
      children: [
        for (int i = 0; i < 20; i++)
          Container(
            color: Colors.blue[100 * (i % 9 + 1)],
            child: Center(child: Text('${i + 1}')),
          ),
      ],
    );
  }
}

// 2. GridView.builder() - Efficient for large grids
class BuilderGridView extends StatelessWidget {
  const BuilderGridView({super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        crossAxisSpacing: 8,
        mainAxisSpacing: 8,
      ),
      itemCount: 50,
      itemBuilder: (context, index) {
        return Container(
          decoration: BoxDecoration(
            color: Colors.green[100 * (index % 9 + 1)],
            borderRadius: BorderRadius.circular(8),
          ),
          child: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(
                  Icons.star,
                  color: Colors.yellow[700],
                ),
                const SizedBox(height: 4),
                Text('Item ${index + 1}'),
              ],
            ),
          ),
        );
      },
    );
  }
}

// 3. GridView.extent() - Maximum cross-axis extent
class ExtentGridView extends StatelessWidget {
  const ExtentGridView({super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.extent(
      maxCrossAxisExtent: 150, // Max width per item
      padding: const EdgeInsets.all(8),
      crossAxisSpacing: 8,
      mainAxisSpacing: 8,
      children: [
        for (int i = 0; i < 20; i++)
          Container(
            color: Colors.red[100 * (i % 9 + 1)],
            child: Center(child: Text('${i + 1}')),
          ),
      ],
    );
  }
}

// 4. GridView.custom() - Custom grid delegate
class CustomGridView extends StatelessWidget {
  const CustomGridView({super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.custom(
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 3,
        crossAxisSpacing: 8,
        mainAxisSpacing: 8,
      ),
      childrenDelegate: SliverChildListDelegate(
        [
          for (int i = 0; i < 20; i++)
            Container(
              color: Colors.orange[100 * (i % 9 + 1)],
              child: Center(child: Text('${i + 1}')),
            ),
        ],
      ),
    );
  }
}
```

What's happening here?
- GridView.count: fixed number of columns
- GridView.builder: efficient for large grids
- GridView.extent: max width per item
- GridView.custom: custom child delegate

---

# GridView with Different Aspect Ratios

> **Custom aspect ratios** for grid items.

```dart
// Different aspect ratios
class AspectRatioGridView extends StatelessWidget {
  const AspectRatioGridView({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Square items (1:1)
        Expanded(
          child: GridView.count(
            crossAxisCount: 3,
            padding: const EdgeInsets.all(8),
            crossAxisSpacing: 8,
            mainAxisSpacing: 8,
            childAspectRatio: 1.0, // Square
            children: [
              for (int i = 0; i < 6; i++)
                Container(
                  color: Colors.blue[100 * (i % 9 + 1)],
                  child: const Center(child: Text('1:1')),
                ),
            ],
          ),
        ),
        
        // 2. Wide items (2:1)
        Expanded(
          child: GridView.count(
            crossAxisCount: 2,
            padding: const EdgeInsets.all(8),
            crossAxisSpacing: 8,
            mainAxisSpacing: 8,
            childAspectRatio: 2.0, // Wide
            children: [
              for (int i = 0; i < 4; i++)
                Container(
                  color: Colors.green[100 * (i % 9 + 1)],
                  child: const Center(child: Text('2:1')),
                ),
            ],
          ),
        ),
        
        // 3. Tall items (1:2)
        Expanded(
          child: GridView.count(
            crossAxisCount: 2,
            padding: const EdgeInsets.all(8),
            crossAxisSpacing: 8,
            mainAxisSpacing: 8,
            childAspectRatio: 0.5, // Tall
            children: [
              for (int i = 0; i < 4; i++)
                Container(
                  color: Colors.red[100 * (i % 9 + 1)],
                  child: const Center(child: Text('1:2')),
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
- childAspectRatio controls item proportions
- 1.0 = square
- >1.0 = wider
- <1.0 = taller

---

# GridView with Different Grid Delegate

> **Custom grid delegates** for advanced layouts.

```dart
// Different grid delegates
class GridDelegateExamples extends StatelessWidget {
  const GridDelegateExamples({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Fixed cross axis count
        Expanded(
          child: GridView.builder(
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 3,
              crossAxisSpacing: 8,
              mainAxisSpacing: 8,
            ),
            itemCount: 12,
            itemBuilder: (context, index) {
              return Container(
                color: Colors.blue[100 * (index % 9 + 1)],
                child: Center(child: Text('$index')),
              );
            },
          ),
        ),
        
        // 2. Max cross axis extent
        Expanded(
          child: GridView.builder(
            gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
              maxCrossAxisExtent: 100,
              crossAxisSpacing: 8,
              mainAxisSpacing: 8,
            ),
            itemCount: 12,
            itemBuilder: (context, index) {
              return Container(
                color: Colors.green[100 * (index % 9 + 1)],
                child: Center(child: Text('$index')),
              );
            },
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- SliverGridDelegateWithFixedCrossAxisCount: fixed columns
- SliverGridDelegateWithMaxCrossAxisExtent: adaptive columns
- Custom delegates for different needs

---

# Real-World Examples

> **Common patterns** using GridView.

```dart
// 1. Photo gallery
class PhotoGallery extends StatelessWidget {
  const PhotoGallery({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Photo Gallery')),
      body: GridView.builder(
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 3,
          crossAxisSpacing: 2,
          mainAxisSpacing: 2,
        ),
        itemCount: 30,
        itemBuilder: (context, index) {
          return Container(
            color: Colors.blue[100 * (index % 9 + 1)],
            child: Stack(
              fit: StackFit.expand,
              children: [
                // Image placeholder
                const Center(
                  child: Icon(
                    Icons.image,
                    color: Colors.white,
                    size: 30,
                  ),
                ),
                // Photo count badge
                if (index == 5)
                  Positioned(
                    bottom: 4,
                    right: 4,
                    child: Container(
                      padding: const EdgeInsets.symmetric(
                        horizontal: 6,
                        vertical: 2,
                      ),
                      decoration: const BoxDecoration(
                        color: Colors.black54,
                        borderRadius: BorderRadius.all(Radius.circular(4)),
                      ),
                      child: const Text(
                        '+10',
                        style: TextStyle(color: Colors.white, fontSize: 12),
                      ),
                    ),
                  ),
              ],
            ),
          );
        },
      ),
    );
  }
}

// 2. Product grid
class ProductGrid extends StatelessWidget {
  const ProductGrid({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Products')),
      body: GridView.builder(
        padding: const EdgeInsets.all(8),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 8,
          mainAxisSpacing: 8,
          childAspectRatio: 0.7,
        ),
        itemCount: 20,
        itemBuilder: (context, index) {
          return Card(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // Product image
                Expanded(
                  child: Container(
                    width: double.infinity,
                    color: Colors.blue[100 * (index % 9 + 1)],
                    child: const Center(
                      child: Icon(
                        Icons.shopping_bag,
                        color: Colors.white,
                        size: 40,
                      ),
                    ),
                  ),
                ),
                // Product info
                Padding(
                  padding: const EdgeInsets.all(8),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        'Product ${index + 1}',
                        style: const TextStyle(
                          fontWeight: FontWeight.bold,
                          fontSize: 14,
                        ),
                        maxLines: 1,
                        overflow: TextOverflow.ellipsis,
                      ),
                      const SizedBox(height: 4),
                      Row(
                        children: [
                          Text(
                            '\$${(index + 1) * 19.99}',
                            style: const TextStyle(
                              color: Colors.blue,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          const SizedBox(width: 8),
                          Text(
                            '\$${(index + 1) * 29.99}',
                            style: TextStyle(
                              color: Colors.grey[600],
                              fontSize: 12,
                              decoration: TextDecoration.lineThrough,
                            ),
                          ),
                        ],
                      ),
                      const SizedBox(height: 4),
                      Row(
                        children: [
                          const Icon(
                            Icons.star,
                            color: Colors.yellow,
                            size: 16,
                          ),
                          const Text(
                            '4.5',
                            style: TextStyle(fontSize: 12),
                          ),
                          const SizedBox(width: 4),
                          Text(
                            '(${(index + 1) * 100})',
                            style: TextStyle(
                              color: Colors.grey[600],
                              fontSize: 12,
                            ),
                          ),
                        ],
                      ),
                    ],
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}

// 3. Icon grid
class IconGrid extends StatelessWidget {
  const IconGrid({super.key});

  final List<IconData> icons = const [
    Icons.home,
    Icons.search,
    Icons.settings,
    Icons.person,
    Icons.star,
    Icons.favorite,
    Icons.phone,
    Icons.email,
    Icons.camera,
    Icons.video,
    Icons.music,
    Icons.game,
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Icons')),
      body: GridView.builder(
        padding: const EdgeInsets.all(8),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 4,
          crossAxisSpacing: 8,
          mainAxisSpacing: 8,
        ),
        itemCount: icons.length,
        itemBuilder: (context, index) {
          return Container(
            decoration: BoxDecoration(
              color: Colors.blue[50],
              borderRadius: BorderRadius.circular(8),
              border: Border.all(color: Colors.blue[200]!),
            ),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(
                  icons[index],
                  size: 32,
                  color: Colors.blue[700],
                ),
                const SizedBox(height: 4),
                Text(
                  'Icon ${index + 1}',
                  style: TextStyle(
                    fontSize: 10,
                    color: Colors.grey[600],
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}

// 4. Responsive grid
class ResponsiveGrid extends StatelessWidget {
  const ResponsiveGrid({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // Responsive crossAxisCount based on width
        int crossAxisCount;
        if (constraints.maxWidth > 800) {
          crossAxisCount = 4;
        } else if (constraints.maxWidth > 600) {
          crossAxisCount = 3;
        } else {
          crossAxisCount = 2;
        }

        return GridView.builder(
          padding: const EdgeInsets.all(8),
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: crossAxisCount,
            crossAxisSpacing: 8,
            mainAxisSpacing: 8,
          ),
          itemCount: 20,
          itemBuilder: (context, index) {
            return Container(
              color: Colors.blue[100 * (index % 9 + 1)],
              child: Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(
                      Icons.star,
                      color: Colors.yellow[700],
                      size: 30,
                    ),
                    Text('Item ${index + 1}'),
                    Text(
                      '${crossAxisCount} cols',
                      style: const TextStyle(fontSize: 10),
                    ),
                  ],
                ),
              ),
            );
          },
        );
      },
    );
  }
}
```

What's happening here?
- Photo gallery with badges
- Product grid with pricing
- Icon grid with styling
- Responsive grid with varying columns

---

# Best Practices

## Use Builder for Large Grids

```dart
// Good - Efficient for large grids
@override
Widget build(BuildContext context) {
  return GridView.builder(
    gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
      crossAxisCount: 3,
    ),
    itemCount: 1000,
    itemBuilder: (context, index) {
      return Container(color: Colors.blue);
    },
  );
}

// Bad - Inefficient for large grids
@override
Widget build(BuildContext context) {
  return GridView.count(
    crossAxisCount: 3,
    children: List.generate(1000, (index) {
      return Container(color: Colors.blue);
    }),
  );
}
```

## Use Appropriate Aspect Ratio

```dart
// Good - Appropriate for content
@override
Widget build(BuildContext context) {
  return GridView.count(
    crossAxisCount: 2,
    childAspectRatio: 0.8, // Good for product cards
    children: products,
  );
}
```

## Use Keys for Dynamic Grids

```dart
// Good - With keys
@override
Widget build(BuildContext context) {
  return GridView.builder(
    itemCount: items.length,
    itemBuilder: (context, index) {
      return Container(
        key: ValueKey(items[index].id),
        child: Text(items[index].name),
      );
    },
  );
}
```

---

# Common Mistakes

## Wrong Aspect Ratio

Wrong:
```dart
// Items too tall or too wide
GridView.count(
  crossAxisCount: 3,
  childAspectRatio: 0.3, // Too tall
  children: items,
)
```

Correct:
```dart
// Reasonable aspect ratio
GridView.count(
  crossAxisCount: 3,
  childAspectRatio: 1.0, // Square
  children: items,
)
```

## Not Using Builder for Large Data

Wrong:
```dart
// All items built at once
GridView.count(
  crossAxisCount: 3,
  children: List.generate(1000, (index) {
    return Container(color: Colors.blue);
  }),
)
```

Correct:
```dart
// Lazy loading
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 3,
  ),
  itemCount: 1000,
  itemBuilder: (context, index) {
    return Container(color: Colors.blue);
  },
)
```

---

# Summary

GridView displays items in a 2D grid layout with multiple constructors for different needs. Use GridView.count for fixed columns, GridView.builder for large datasets, and GridView.extent for adaptive columns. GridView is perfect for galleries, product displays, and any grid-based content.

---

# Next Steps

- [CustomScrollView](customscrollview.md)
- [Slivers](slivers.md)
- [ScrollController](scrollcontroller.md)

---

# Did You Know?

- GridView.builder builds items lazily
- childAspectRatio controls item proportions
- GridView can scroll horizontally with scrollDirection
- GridView.extent adapts to screen width
- GridView supports custom grid delegates
- Keys improve grid performance
- GridView is used for photo galleries
- GridView can be nested with other widgets