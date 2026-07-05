# Slivers

Understand the building blocks of scrollable content in Flutter.

---

# What is it?

Slivers are the fundamental building blocks of scrollable areas in Flutter. They are pieces of a scrollable view that can be combined in various ways to create complex scrolling effects. Unlike regular widgets, slivers have special properties that allow them to be laid out lazily and efficiently in a scrolling viewport.

---

# Why does it exist?

Slivers exist to:

- Enable efficient lazy loading of scrollable content
- Create complex scrollable layouts
- Support custom scrolling effects
- Combine different scrollable elements
- Optimize performance for large lists
- Enable scrollable app bars and headers
- Create advanced scrolling experiences

---

# Understanding Slivers

> **Slivers are scrollable** building blocks.

```dart
// Basic sliver usage
class BasicSliverExample extends StatelessWidget {
  const BasicSliverExample({super.key});

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: [
        // Each sliver is a scrollable piece
        SliverToBoxAdapter(
          child: Container(
            height: 100,
            color: Colors.blue,
            child: const Center(child: Text('Regular Widget')),
          ),
        ),
        
        SliverList(
          delegate: SliverChildBuilderDelegate(
            (context, index) => Container(
              height: 50,
              color: Colors.blue[100 * (index % 9 + 1)],
              child: Center(child: Text('Item $index')),
            ),
            childCount: 20,
          ),
        ),
      ],
    );
  }
}

// Sliver characteristics:
// 1. Scrollable - Part of a scroll view
// 2. Lazy - Can build children on demand
// 3. Flexible - Can be combined freely
// 4. Efficient - Optimized for scrolling
// 5. Customizable - Various types available
```

What's happening here?
- Slivers are scrollable building blocks
- Combine multiple slivers in CustomScrollView
- Each sliver has specific behavior
- Lazy loading for performance

---

# Sliver Types

> **Common sliver types** and their uses.

```dart
// Different sliver types
class SliverTypesExample extends StatelessWidget {
  const SliverTypesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: [
        // 1. SliverAppBar - Scrollable app bar
        SliverAppBar(
          title: const Text('Sliver Types'),
          expandedHeight: 150,
          flexibleSpace: const FlexibleSpaceBar(
            background: ColoredBox(
              color: Colors.blue,
              child: Center(
                child: Text(
                  'App Bar',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
          pinned: true,
        ),
        
        // 2. SliverToBoxAdapter - Regular widget
        const SliverToBoxAdapter(
          child: Padding(
            padding: EdgeInsets.all(16),
            child: Text(
              'Section 1: List',
              style: TextStyle(
                fontSize: 20,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
        ),
        
        // 3. SliverList - List of children
        SliverList(
          delegate: SliverChildBuilderDelegate(
            (context, index) => ListTile(
              title: Text('List Item $index'),
              leading: CircleAvatar(child: Text('${index + 1}')),
            ),
            childCount: 10,
          ),
        ),
        
        // 4. SliverToBoxAdapter - Section header
        const SliverToBoxAdapter(
          child: Padding(
            padding: EdgeInsets.all(16),
            child: Text(
              'Section 2: Grid',
              style: TextStyle(
                fontSize: 20,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
        ),
        
        // 5. SliverGrid - Grid of children
        SliverGrid(
          delegate: SliverChildBuilderDelegate(
            (context, index) => Container(
              color: Colors.green[100 * (index % 9 + 1)],
              child: Center(child: Text('Grid $index')),
            ),
            childCount: 12,
          ),
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 3,
            crossAxisSpacing: 4,
            mainAxisSpacing: 4,
          ),
        ),
        
        // 6. SliverPadding - Adds padding
        SliverPadding(
          padding: const EdgeInsets.all(16),
          sliver: SliverToBoxAdapter(
            child: Container(
              height: 50,
              color: Colors.orange,
              child: const Center(child: Text('Padded Widget')),
            ),
          ),
        ),
        
        // 7. SliverFillRemaining - Fill remaining space
        const SliverFillRemaining(
          child: Center(
            child: Text(
              'Fills remaining space',
              style: TextStyle(fontSize: 18),
            ),
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- SliverAppBar: scrollable app bar
- SliverToBoxAdapter: regular widget
- SliverList: scrollable list
- SliverGrid: scrollable grid
- SliverPadding: padding around sliver
- SliverFillRemaining: fills remaining space

---

# SliverList and Delegate

> **SliverList** uses delegates for efficient building.

```dart
// SliverList delegates
class SliverListDelegates extends StatelessWidget {
  const SliverListDelegates({super.key});

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: [
        const SliverAppBar(
          title: Text('SliverList Delegates'),
          pinned: true,
        ),
        
        // 1. SliverChildListDelegate - Fixed children
        SliverList(
          delegate: SliverChildListDelegate(
            [
              for (int i = 0; i < 5; i++)
                Container(
                  height: 50,
                  color: Colors.blue[100 * (i % 9 + 1)],
                  margin: const EdgeInsets.all(4),
                  child: Center(child: Text('List $i')),
                ),
            ],
          ),
        ),
        
        // 2. SliverChildBuilderDelegate - Lazy building
        SliverList(
          delegate: SliverChildBuilderDelegate(
            (context, index) => Container(
              height: 50,
              color: Colors.green[100 * (index % 9 + 1)],
              margin: const EdgeInsets.all(4),
              child: Center(child: Text('Builder $index')),
            ),
            childCount: 50,
            // Performance optimizations
            addRepaintBoundaries: true,
            addSemanticIndexes: true,
          ),
        ),
        
        // 3. SliverFixedExtentList - Fixed height
        SliverFixedExtentList(
          delegate: SliverChildBuilderDelegate(
            (context, index) => Container(
              color: Colors.orange[100 * (index % 9 + 1)],
              child: Center(child: Text('Fixed $index')),
            ),
            childCount: 10,
          ),
          itemExtent: 60, // All items are 60px tall
        ),
      ],
    );
  }
}

// Delegate comparison:
// SliverChildListDelegate: All children built at once
// SliverChildBuilderDelegate: Children built lazily
// SliverFixedExtentList: Optimized for fixed-height items
// Use builder delegate for large lists
```

What's happening here?
- SliverChildListDelegate: all children built
- SliverChildBuilderDelegate: lazy building
- SliverFixedExtentList: fixed height items
- Choose delegate based on needs

---

# SliverGrid and Delegate

> **SliverGrid** provides grid layouts in scroll views.

```dart
// SliverGrid delegates
class SliverGridDelegates extends StatelessWidget {
  const SliverGridDelegates({super.key});

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: [
        const SliverAppBar(
          title: Text('SliverGrid Delegates'),
          pinned: true,
        ),
        
        // 1. Fixed cross axis count
        SliverGrid(
          delegate: SliverChildBuilderDelegate(
            (context, index) => Container(
              color: Colors.blue[100 * (index % 9 + 1)],
              child: Center(child: Text('$index')),
            ),
            childCount: 20,
          ),
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 3,
            crossAxisSpacing: 4,
            mainAxisSpacing: 4,
          ),
        ),
        
        // 2. Max cross axis extent
        SliverGrid(
          delegate: SliverChildBuilderDelegate(
            (context, index) => Container(
              color: Colors.green[100 * (index % 9 + 1)],
              child: Center(child: Text('$index')),
            ),
            childCount: 20,
          ),
          gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
            maxCrossAxisExtent: 100,
            crossAxisSpacing: 4,
            mainAxisSpacing: 4,
          ),
        ),
        
        // 3. Custom aspect ratio
        SliverGrid(
          delegate: SliverChildBuilderDelegate(
            (context, index) => Container(
              color: Colors.orange[100 * (index % 9 + 1)],
              child: Center(child: Text('$index')),
            ),
            childCount: 20,
          ),
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 2,
            crossAxisSpacing: 4,
            mainAxisSpacing: 4,
            childAspectRatio: 1.5, // Wider than tall
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- Fixed cross axis count: fixed number of columns
- Max cross axis extent: adaptive columns
- Custom aspect ratio: control item proportions
- Efficient lazy loading

---

# Real-World Examples

> **Common patterns** using slivers.

```dart
// 1. Weather app
class WeatherApp extends StatelessWidget {
  const WeatherApp({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // Weather header
          SliverAppBar(
            expandedHeight: 200,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              background: Container(
                decoration: const BoxDecoration(
                  gradient: LinearGradient(
                    begin: Alignment.topLeft,
                    end: Alignment.bottomRight,
                    colors: [Colors.blue, Colors.purple],
                  ),
                ),
                child: const Padding(
                  padding: EdgeInsets.all(16),
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text(
                        '72°F',
                        style: TextStyle(
                          color: Colors.white,
                          fontSize: 64,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      Text(
                        'Sunny',
                        style: TextStyle(
                          color: Colors.white70,
                          fontSize: 24,
                        ),
                      ),
                      Text(
                        'New York, NY',
                        style: TextStyle(
                          color: Colors.white70,
                          fontSize: 16,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          ),
          
          // Hourly forecast
          SliverToBoxAdapter(
            child: Container(
              padding: const EdgeInsets.all(16),
              child: const Text(
                'Hourly Forecast',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
          
          SliverToBoxAdapter(
            child: SizedBox(
              height: 100,
              child: ListView.builder(
                scrollDirection: Axis.horizontal,
                itemCount: 24,
                itemBuilder: (context, index) {
                  return Container(
                    width: 60,
                    margin: const EdgeInsets.symmetric(horizontal: 4),
                    child: Column(
                      children: [
                        Text('${index}:00'),
                        const SizedBox(height: 4),
                        Icon(
                          index % 2 == 0 ? Icons.wb_sunny : Icons.cloud,
                          color: Colors.orange,
                        ),
                        const SizedBox(height: 4),
                        Text('${65 + index}°F'),
                      ],
                    ),
                  );
                },
              ),
            ),
          ),
          
          // Daily forecast
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                '7-Day Forecast',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
          
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                final days = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];
                return ListTile(
                  leading: Text(days[index % days.length]),
                  title: Row(
                    children: [
                      Icon(
                        index % 2 == 0 ? Icons.wb_sunny : Icons.cloud,
                        color: Colors.orange,
                      ),
                      const SizedBox(width: 8),
                      Text('${65 + index}°F'),
                    ],
                  ),
                  trailing: Text('${70 + index}°F'),
                );
              },
              childCount: 7,
            ),
          ),
        ],
      ),
    );
  }
}

// 2. E-commerce product page
class ProductPage extends StatelessWidget {
  const ProductPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // Product image
          SliverAppBar(
            expandedHeight: 300,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              background: Container(
                color: Colors.blue[100],
                child: const Center(
                  child: Icon(
                    Icons.image,
                    size: 100,
                    color: Colors.white,
                  ),
                ),
              ),
            ),
            actions: [
              IconButton(
                icon: const Icon(Icons.favorite_border),
                onPressed: () {},
              ),
              IconButton(
                icon: const Icon(Icons.share),
                onPressed: () {},
              ),
            ],
          ),
          
          // Product info
          SliverToBoxAdapter(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    'Product Name',
                    style: TextStyle(
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Row(
                    children: [
                      const Text(
                        '\$99.99',
                        style: TextStyle(
                          fontSize: 20,
                          color: Colors.blue,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(width: 8),
                      Text(
                        '\$149.99',
                        style: TextStyle(
                          fontSize: 16,
                          color: Colors.grey[600],
                          decoration: TextDecoration.lineThrough,
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 16),
                  const Text(
                    'Product description goes here. '
                    'This describes the product in detail.',
                    style: TextStyle(
                      fontSize: 16,
                      height: 1.5,
                    ),
                  ),
                ],
              ),
            ),
          ),
          
          // Related products
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'Related Products',
                style: TextStyle(
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
          
          SliverPadding(
            padding: const EdgeInsets.all(8),
            sliver: SliverGrid(
              delegate: SliverChildBuilderDelegate(
                (context, index) => Card(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Expanded(
                        child: Container(
                          width: double.infinity,
                          color: Colors.blue[100 * (index % 9 + 1)],
                          child: const Center(
                            child: Icon(
                              Icons.shopping_bag,
                              color: Colors.white,
                              size: 30,
                            ),
                          ),
                        ),
                      ),
                      Padding(
                        padding: const EdgeInsets.all(8),
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              'Product ${index + 1}',
                              maxLines: 1,
                              overflow: TextOverflow.ellipsis,
                            ),
                            Text(
                              '\$${19.99 + index * 10}',
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
                childCount: 10,
              ),
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 2,
                crossAxisSpacing: 8,
                mainAxisSpacing: 8,
                childAspectRatio: 0.7,
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
- Weather app with header and forecasts
- E-commerce product page with images
- Mixed sliver types for complex layouts
- Real-world scrolling patterns

---

# Best Practices

## Use Lazy Delegates

```dart
// Good - Lazy loading
SliverList(
  delegate: SliverChildBuilderDelegate(
    (context, index) => Text('Item $index'),
    childCount: 1000,
  ),
)

// Bad - All children built at once
SliverList(
  delegate: SliverChildListDelegate(
    List.generate(1000, (index) => Text('Item $index')),
  ),
)
```

## Use Appropriate Sliver Types

```dart
// Good - Appropriate sliver for the job
SliverAppBar(...),  // For app bars
SliverList(...),    // For lists
SliverGrid(...),    // For grids
SliverToBoxAdapter(...), // For regular widgets
```

## Optimize Performance

```dart
// Good - Performance optimizations
SliverList(
  delegate: SliverChildBuilderDelegate(
    (context, index) => Container(...),
    childCount: 1000,
    addRepaintBoundaries: true,
    addSemanticIndexes: true,
  ),
)
```

---

# Common Mistakes

## Not Using Builder Delegate

Wrong:
```dart
// Inefficient for large data
SliverList(
  delegate: SliverChildListDelegate(
    List.generate(1000, (index) => Container()),
  ),
)
```

Correct:
```dart
// Efficient lazy loading
SliverList(
  delegate: SliverChildBuilderDelegate(
    (context, index) => Container(),
    childCount: 1000,
  ),
)
```

## Mixing Sliver and Non-Sliver

Wrong:
```dart
// Error - cannot mix directly
CustomScrollView(
  slivers: [
    Container(), // Error
    ListView(), // Error
  ],
)
```

Correct:
```dart
// Wrap in SliverToBoxAdapter
CustomScrollView(
  slivers: [
    SliverToBoxAdapter(child: Container()),
    SliverToBoxAdapter(child: ListView()),
  ],
)
```

---

# Summary

Slivers are the building blocks of scrollable views in Flutter. They enable efficient lazy loading, complex scrolling effects, and mixed content types. Use different sliver types for different purposes: SliverList for lists, SliverGrid for grids, SliverAppBar for headers, and SliverToBoxAdapter for regular widgets.

---

# Next Steps

- [ScrollController](scrollcontroller.md)
- [ScrollPhysics](scrollphysics.md)
- [Nested Scrolling](nestedscrolling.md)

---

# Did You Know?

- Slivers enable lazy loading
- SliverList and SliverGrid support large datasets
- SliverAppBar creates scrollable app bars
- SliverToBoxAdapter wraps regular widgets
- SliverPadding adds padding to slivers
- SliverFillRemaining fills remaining space
- Slivers can be combined freely
- Slivers are optimized for performance