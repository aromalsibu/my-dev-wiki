# CustomScrollView

Understand how to create complex scrollable layouts with slivers.

---

# What is it?

CustomScrollView is a scrollable widget that allows you to combine multiple types of scrollable elements (slivers) into a single scrollable view. Unlike ListView or GridView, which only display one type of child, CustomScrollView lets you mix different scrollable elements like lists, grids, app bars, and custom widgets together.

---

# Why does it exist?

CustomScrollView exists to:

- Combine different scrollable elements
- Create complex scrollable layouts
- Implement scrollable app bars
- Mix lists and grids in one scroll view
- Create custom scrollable effects
- Build advanced UI patterns
- Enable smooth scrolling with mixed content

---

# Basic CustomScrollView

> **CustomScrollView** combines different slivers.

```dart
// Basic CustomScrollView
class BasicCustomScrollView extends StatelessWidget {
  const BasicCustomScrollView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // 1. App bar
          const SliverAppBar(
            title: Text('Custom Scroll View'),
            pinned: true,
          ),
          
          // 2. Text section
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'Welcome to CustomScrollView',
                style: TextStyle(
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
          
          // 3. List of items
          SliverList(
            delegate: SliverChildListDelegate(
              [
                for (int i = 0; i < 10; i++)
                  Container(
                    height: 60,
                    color: Colors.blue[100 * (i % 9 + 1)],
                    margin: const EdgeInsets.symmetric(
                      horizontal: 16,
                      vertical: 4,
                    ),
                    child: Center(child: Text('List Item $i')),
                  ),
              ],
            ),
          ),
          
          // 4. Grid section
          SliverGrid(
            delegate: SliverChildListDelegate(
              [
                for (int i = 0; i < 6; i++)
                  Container(
                    color: Colors.green[100 * (i % 9 + 1)],
                    child: Center(child: Text('Grid $i')),
                  ),
              ],
            ),
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 3,
              crossAxisSpacing: 4,
              mainAxisSpacing: 4,
            ),
          ),
        ],
      ),
    );
  }
}

// CustomScrollView properties:
// 1. slivers - List of sliver widgets
// 2. scrollDirection - Axis.vertical or Axis.horizontal
// 3. reverse - Reverse scroll direction
// 4. controller - ScrollController
// 5. physics - ScrollPhysics
// 6. primary - Whether to use primary scroll view
```

What's happening here?
- CustomScrollView combines multiple slivers
- Mixes lists, grids, app bars, and adapters
- All scroll together as one view
- Flexible and powerful

---

# Sliver Types

> **Different sliver types** for different purposes.

```dart
// Sliver types overview
class SliverTypesOverview extends StatelessWidget {
  const SliverTypesOverview({super.key});

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
                  'Custom App Bar',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
          pinned: true,
        ),
        
        // 2. SliverList - List of children
        SliverList(
          delegate: SliverChildListDelegate(
            [
              for (int i = 0; i < 5; i++)
                ListTile(title: Text('List Item $i')),
            ],
          ),
        ),
        
        // 3. SliverGrid - Grid of children
        SliverGrid(
          delegate: SliverChildListDelegate(
            [
              for (int i = 0; i < 6; i++)
                Container(
                  color: Colors.blue[100 * (i % 9 + 1)],
                  child: Center(child: Text('Grid $i')),
                ),
            ],
          ),
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 3,
            crossAxisSpacing: 4,
            mainAxisSpacing: 4,
          ),
        ),
        
        // 4. SliverToBoxAdapter - Regular widget
        const SliverToBoxAdapter(
          child: Padding(
            padding: EdgeInsets.all(16),
            child: Text(
              'This is a regular widget in a sliver',
              style: TextStyle(fontSize: 16),
            ),
          ),
        ),
        
        // 5. SliverPadding - Padding around sliver
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
        
        // 6. SliverFillRemaining - Fill remaining space
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
- SliverList: list of children
- SliverGrid: grid of children
- SliverToBoxAdapter: regular widget
- SliverPadding: padding around sliver
- SliverFillRemaining: fill remaining space

---

# SliverAppBar

> **SliverAppBar** provides scrollable app bar functionality.

```dart
// SliverAppBar features
class SliverAppBarFeatures extends StatelessWidget {
  const SliverAppBarFeatures({super.key});

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: [
        // 1. Standard SliverAppBar
        SliverAppBar(
          title: const Text('Standard App Bar'),
          expandedHeight: 200,
          flexibleSpace: const FlexibleSpaceBar(
            background: ColoredBox(
              color: Colors.blue,
              child: Center(
                child: Text(
                  'Flexible Space',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
          // pinned - stays at top when scrolling
          pinned: true,
        ),
        
        // 2. Floating App Bar
        SliverAppBar(
          title: const Text('Floating App Bar'),
          expandedHeight: 150,
          flexibleSpace: const FlexibleSpaceBar(
            background: ColoredBox(
              color: Colors.green,
              child: Center(
                child: Text(
                  'Floating',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
          // floating - appears when scrolling up
          floating: true,
        ),
        
        // 3. Snapping App Bar
        SliverAppBar(
          title: const Text('Snapping App Bar'),
          expandedHeight: 150,
          flexibleSpace: const FlexibleSpaceBar(
            background: ColoredBox(
              color: Colors.red,
              child: Center(
                child: Text(
                  'Snapping',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
          // snap - snaps open when scrolling
          snap: true,
        ),
        
        // Content after app bar
        SliverList(
          delegate: SliverChildListDelegate(
            [
              for (int i = 0; i < 20; i++)
                Container(
                  height: 60,
                  color: Colors.blue[100 * (i % 9 + 1)],
                  margin: const EdgeInsets.all(4),
                  child: Center(child: Text('Item $i')),
                ),
            ],
          ),
        ),
      ],
    );
  }
}

// SliverAppBar properties:
// 1. title - App bar title
// 2. expandedHeight - Height when fully expanded
// 3. flexibleSpace - Space behind the app bar
// 4. pinned - Stays visible when scrolling
// 5. floating - Appears when scrolling up
// 6. snap - Snaps open when scrolling
// 7. collapsedHeight - Height when collapsed
// 8. stretch - Stretches when pulling down
```

What's happening here?
- Pinned: stays at top
- Floating: reappears on scroll up
- Snap: snaps to expanded state
- FlexibleSpace: background content

---

# SliverList and SliverGrid

> **Efficient slivers** for lists and grids.

```dart
// Efficient sliver delegates
class EfficientSlivers extends StatelessWidget {
  const EfficientSlivers({super.key});

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: [
        const SliverAppBar(
          title: Text('Efficient Slivers'),
          pinned: true,
        ),
        
        // 1. SliverList with builder delegate
        SliverList(
          delegate: SliverChildBuilderDelegate(
            (context, index) {
              return Container(
                height: 60,
                color: Colors.blue[100 * (index % 9 + 1)],
                margin: const EdgeInsets.all(4),
                child: Center(child: Text('List Item $index')),
              );
            },
            childCount: 50,
            // Performance optimizations
            addRepaintBoundaries: true,
            addSemanticIndexes: true,
          ),
        ),
        
        // 2. SliverGrid with builder delegate
        SliverGrid(
          delegate: SliverChildBuilderDelegate(
            (context, index) {
              return Container(
                color: Colors.green[100 * (index % 9 + 1)],
                child: Center(child: Text('Grid $index')),
              );
            },
            childCount: 30,
          ),
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 3,
            crossAxisSpacing: 4,
            mainAxisSpacing: 4,
          ),
        ),
        
        // 3. SliverFixedExtentList - Fixed height items
        SliverFixedExtentList(
          delegate: SliverChildBuilderDelegate(
            (context, index) {
              return Container(
                color: Colors.orange[100 * (index % 9 + 1)],
                child: Center(child: Text('Fixed $index')),
              );
            },
            childCount: 20,
          ),
          itemExtent: 50, // All items are 50px tall
        ),
      ],
    );
  }
}
```

What's happening here?
- SliverChildBuilderDelegate: lazy building
- SliverChildListDelegate: fixed children
- SliverFixedExtentList: fixed height items
- Performance optimizations

---

# Real-World Examples

> **Common patterns** using CustomScrollView.

```dart
// 1. Profile screen with header
class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // Profile header
          SliverAppBar(
            expandedHeight: 250,
            flexibleSpace: FlexibleSpaceBar(
              background: Container(
                decoration: BoxDecoration(
                  gradient: LinearGradient(
                    begin: Alignment.topLeft,
                    end: Alignment.bottomRight,
                    colors: [
                      Colors.blue[400]!,
                      Colors.purple[400]!,
                    ],
                  ),
                ),
                child: const Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    CircleAvatar(
                      radius: 50,
                      backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
                    ),
                    SizedBox(height: 16),
                    Text(
                      'John Doe',
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: 24,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    Text(
                      'Flutter Developer',
                      style: TextStyle(
                        color: Colors.white70,
                        fontSize: 16,
                      ),
                    ),
                  ],
                ),
              ),
            ),
            pinned: true,
          ),
          
          // Stats section
          SliverToBoxAdapter(
            child: Container(
              padding: const EdgeInsets.all(16),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  _buildStat('Posts', '150'),
                  _buildStat('Followers', '2.5K'),
                  _buildStat('Following', '350'),
                ],
              ),
            ),
          ),
          
          // Posts list
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                return Card(
                  margin: const EdgeInsets.symmetric(
                    horizontal: 16,
                    vertical: 8,
                  ),
                  child: Padding(
                    padding: const EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Row(
                          children: [
                            const CircleAvatar(
                              radius: 20,
                              backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
                            ),
                            const SizedBox(width: 12),
                            const Text(
                              'John Doe',
                              style: TextStyle(
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                            const Spacer(),
                            Text(
                              '2h ago',
                              style: TextStyle(
                                color: Colors.grey[600],
                                fontSize: 12,
                              ),
                            ),
                          ],
                        ),
                        const SizedBox(height: 12),
                        Text(
                          'This is post number ${index + 1}. '
                          'It contains some content about Flutter.',
                          style: const TextStyle(height: 1.5),
                        ),
                        const SizedBox(height: 12),
                        Row(
                          children: [
                            IconButton(
                              icon: const Icon(Icons.favorite_border),
                              onPressed: () {},
                            ),
                            const Text('25'),
                            const SizedBox(width: 16),
                            IconButton(
                              icon: const Icon(Icons.comment_outlined),
                              onPressed: () {},
                            ),
                            const Text('8'),
                          ],
                        ),
                      ],
                    ),
                  ),
                );
              },
              childCount: 20,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildStat(String label, String value) {
    return Column(
      children: [
        Text(
          value,
          style: const TextStyle(
            fontSize: 20,
            fontWeight: FontWeight.bold,
          ),
        ),
        Text(
          label,
          style: TextStyle(
            color: Colors.grey[600],
            fontSize: 14,
          ),
        ),
      ],
    );
  }
}

// 2. Store front
class StoreFront extends StatelessWidget {
  const StoreFront({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          const SliverAppBar(
            title: Text('Store'),
            pinned: true,
          ),
          
          // Banner
          SliverToBoxAdapter(
            child: Container(
              height: 150,
              margin: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue,
                borderRadius: BorderRadius.circular(12),
              ),
              child: const Center(
                child: Text(
                  'Summer Sale!\n50% OFF',
                  textAlign: TextAlign.center,
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            ),
          ),
          
          // Categories
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.symmetric(horizontal: 16),
              child: Text(
                'Categories',
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
                (context, index) {
                  final categories = ['Electronics', 'Clothing', 'Books', 'Home'];
                  return Container(
                    decoration: BoxDecoration(
                      color: Colors.blue[100 * (index % 9 + 1)],
                      borderRadius: BorderRadius.circular(8),
                    ),
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        Icon(
                          Icons.category,
                          color: Colors.blue[700],
                          size: 32,
                        ),
                        const SizedBox(height: 4),
                        Text(categories[index % categories.length]),
                      ],
                    ),
                  );
                },
                childCount: 8,
              ),
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 4,
                crossAxisSpacing: 8,
                mainAxisSpacing: 8,
              ),
            ),
          ),
          
          // Products
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.symmetric(horizontal: 16),
              child: Text(
                'Featured Products',
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
                (context, index) {
                  return Card(
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
                                size: 40,
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
                                style: const TextStyle(
                                  fontWeight: FontWeight.bold,
                                ),
                              ),
                              Text(
                                '\$${(index + 1) * 19.99}',
                                style: const TextStyle(
                                  color: Colors.blue,
                                ),
                              ),
                            ],
                          ),
                        ),
                      ],
                    ),
                  );
                },
                childCount: 20,
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
- Profile screen with header and posts
- Store front with banners, categories, products
- Complex scrollable layouts
- Mixed content types

---

# Best Practices

## Use Efficient Delegates

```dart
// Good - Builder delegate
SliverList(
  delegate: SliverChildBuilderDelegate(
    (context, index) => Text('Item $index'),
    childCount: 1000,
  ),
)

// Bad - List delegate for large data
SliverList(
  delegate: SliverChildListDelegate(
    List.generate(1000, (index) => Text('Item $index')),
  ),
)
```

## Use SliverPadding

```dart
// Good - SliverPadding for slivers
SliverPadding(
  padding: const EdgeInsets.all(16),
  sliver: SliverList(...),
)

// Bad - Padding inside sliver
SliverList(
  delegate: SliverChildBuilderDelegate(
    (context, index) => Padding(
      padding: const EdgeInsets.all(16),
      child: Container(...),
    ),
  ),
)
```

## Combine Slivers Strategically

```dart
// Good - Strategic combination
CustomScrollView(
  slivers: [
    SliverAppBar(...), // Header
    SliverToBoxAdapter(...), // Static content
    SliverList(...), // Scrollable list
    SliverGrid(...), // Grid
    SliverFillRemaining(...), // Footer
  ],
)
```

---

# Common Mistakes

## Mixing Slivers and Non-Slivers

Wrong:
```dart
// Can't mix directly
CustomScrollView(
  slivers: [
    Container(), // Error - must be Sliver
    ListView(), // Error - must be Sliver
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

## Not Using Builder for Large Data

Wrong:
```dart
// All items built at once
SliverList(
  delegate: SliverChildListDelegate(
    List.generate(1000, (index) => Text('Item $index')),
  ),
)
```

Correct:
```dart
// Lazy loading
SliverList(
  delegate: SliverChildBuilderDelegate(
    (context, index) => Text('Item $index'),
    childCount: 1000,
  ),
)
```

---

# Summary

CustomScrollView combines different slivers into a single scrollable view. Use it for complex layouts with mixed content types like app bars, lists, grids, and custom widgets. Slivers provide lazy loading and efficient scrolling for large datasets.

---

# Next Steps

- [Slivers](slivers.md)
- [ScrollController](scrollcontroller.md)
- [ScrollPhysics](scrollphysics.md)

---

# Did You Know?

- CustomScrollView combines multiple slivers
- SliverAppBar provides scrollable app bars
- SliverList and SliverGrid support lazy loading
- SliverToBoxAdapter wraps regular widgets
- SliverPadding adds padding to slivers
- SliverFillRemaining fills remaining space
- CustomScrollView is powerful for complex layouts
- Slivers are the building blocks of scrollable content