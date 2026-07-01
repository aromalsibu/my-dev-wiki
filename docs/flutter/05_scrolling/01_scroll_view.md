# ScrollView

Understand the foundation of scrolling in Flutter.

---

# What is it?

ScrollView is the base widget for all scrollable content in Flutter. It provides the infrastructure for scrolling, including the viewport, scrollable widget, and physics. While you typically use specialized scrollable widgets like ListView, GridView, and CustomScrollView, understanding ScrollView helps you understand how scrolling works in Flutter.

---

# Why does it exist?

ScrollView exists to:

- Provide scrolling infrastructure
- Handle scrollable content
- Manage viewport and rendering
- Support different scroll behaviors
- Enable lazy loading of children
- Handle scroll physics and interactions
- Support custom scrolling configurations

---

# ScrollView Basics

> **ScrollView** is the foundation for scrolling widgets.

```dart
// Basic ScrollView usage
class BasicScrollView extends StatelessWidget {
  const BasicScrollView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ScrollView')),
      body: ScrollView(
        // Custom scroll view with children
        // Usually you'd use ListView or CustomScrollView
        slivers: [
          SliverList(
            delegate: SliverChildListDelegate(
              [
                for (int i = 0; i < 20; i++)
                  Container(
                    height: 50,
                    color: Colors.blue[100 * (i % 9 + 1)],
                    margin: const EdgeInsets.all(4),
                    child: Center(child: Text('Item $i')),
                  ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// ScrollView properties:
// 1. slivers - List of sliver widgets
// 2. scrollDirection - Axis.vertical or Axis.horizontal
// 3. reverse - Whether to reverse scroll direction
// 4. controller - ScrollController for controlling scroll
// 5. physics - ScrollPhysics for behavior
// 6. viewportBuilder - Builds the viewport
// 7. scrollBehavior - Custom scroll behavior
// 8. clipBehavior - Clipping behavior
// 9. keyboardDismissBehavior - Dismiss keyboard on scroll
```

What's happening here?
- ScrollView is the base for all scrolling
- Slivers are the building blocks
- Supports vertical and horizontal scrolling
- Customizable physics and controllers
- Usually use specialized widgets

---

# ScrollView vs Other Scrolling Widgets

> **Comparing ScrollView** with specialized scrollable widgets.

```dart
// Different scrolling widgets
class ScrollingWidgetsComparison extends StatelessWidget {
  const ScrollingWidgetsComparison({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. ListView - Most common
        Expanded(
          child: ListView(
            children: [
              for (int i = 0; i < 20; i++)
                Container(
                  height: 50,
                  color: Colors.blue[100 * (i % 9 + 1)],
                  child: Center(child: Text('ListView $i')),
                ),
            ],
          ),
        ),
        
        // 2. CustomScrollView - Sliver-based
        Expanded(
          child: CustomScrollView(
            slivers: [
              SliverList(
                delegate: SliverChildListDelegate(
                  [
                    for (int i = 0; i < 20; i++)
                      Container(
                        height: 50,
                        color: Colors.green[100 * (i % 9 + 1)],
                        child: Center(child: Text('Custom $i')),
                      ),
                  ],
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

// ScrollView hierarchy:
// ScrollView (abstract)
//   ├── BoxScrollView (abstract)
//   │   ├── ListView
//   │   └── GridView
//   └── CustomScrollView
//       └── (uses Slivers directly)

// When to use:
// ListView: Simple lists
// GridView: Grid layouts
// CustomScrollView: Complex scrollable layouts with mixed slivers
// ScrollView: Rarely used directly
```

What's happening here?
- ListView is most common for lists
- GridView for grid layouts
- CustomScrollView for complex layouts
- ScrollView is usually not used directly

---

# ScrollView Slivers

> **Slivers** are the building blocks of ScrollView.

```dart
// Sliver types in ScrollView
class SliverTypesExample extends StatelessWidget {
  const SliverTypesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: [
        // 1. SliverAppBar - Scrollable app bar
        SliverAppBar(
          title: const Text('Sliver App Bar'),
          expandedHeight: 200,
          flexibleSpace: FlexibleSpaceBar(
            background: Container(
              color: Colors.blue,
              child: const Center(
                child: Text(
                  'Header',
                  style: TextStyle(color: Colors.white, fontSize: 24),
                ),
              ),
            ),
          ),
          floating: true,
          pinned: true,
        ),
        
        // 2. SliverList - List of children
        SliverList(
          delegate: SliverChildListDelegate(
            [
              for (int i = 0; i < 10; i++)
                Container(
                  height: 60,
                  color: Colors.blue[100 * (i % 9 + 1)],
                  margin: const EdgeInsets.all(4),
                  child: Center(child: Text('Item $i')),
                ),
            ],
          ),
        ),
        
        // 3. SliverGrid - Grid of children
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
        
        // 4. SliverFixedExtentList - Fixed height list
        SliverFixedExtentList(
          delegate: SliverChildListDelegate(
            [
              for (int i = 0; i < 5; i++)
                Container(
                  color: Colors.orange[100 * (i % 9 + 1)],
                  child: Center(child: Text('Fixed $i')),
                ),
            ],
          ),
          itemExtent: 60,
        ),
        
        // 5. SliverPadding - Adds padding
        const SliverPadding(
          padding: EdgeInsets.all(16),
          sliver: SliverToBoxAdapter(
            child: Text('Padded content'),
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
- SliverFixedExtentList: fixed height list
- SliverPadding: adds padding to slivers

---

# ScrollController

> **ScrollController** controls scrolling programmatically.

```dart
// ScrollController example
class ScrollControllerExample extends StatefulWidget {
  const ScrollControllerExample({super.key});

  @override
  State<ScrollControllerExample> createState() => _ScrollControllerExampleState();
}

class _ScrollControllerExampleState extends State<ScrollControllerExample> {
  final ScrollController _controller = ScrollController();
  double _scrollPosition = 0;

  @override
  void initState() {
    super.initState();
    _controller.addListener(() {
      setState(() {
        _scrollPosition = _controller.position.pixels;
      });
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ScrollController'),
        actions: [
          TextButton(
            onPressed: () {
              setState(() {});
            },
            child: Text('Position: ${_scrollPosition.toInt()}'),
          ),
        ],
      ),
      body: Column(
        children: [
          // Scroll controls
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ElevatedButton(
                onPressed: () {
                  _controller.animateTo(
                    0,
                    duration: const Duration(milliseconds: 500),
                    curve: Curves.easeInOut,
                  );
                },
                child: const Text('Top'),
              ),
              const SizedBox(width: 8),
              ElevatedButton(
                onPressed: () {
                  _controller.animateTo(
                    _controller.position.maxScrollExtent,
                    duration: const Duration(milliseconds: 500),
                    curve: Curves.easeInOut,
                  );
                },
                child: const Text('Bottom'),
              ),
              const SizedBox(width: 8),
              ElevatedButton(
                onPressed: () {
                  _controller.jumpTo(_controller.position.maxScrollExtent / 2);
                },
                child: const Text('Middle'),
              ),
            ],
          ),
          
          const SizedBox(height: 8),
          
          // Scrollable content
          Expanded(
            child: ListView.builder(
              controller: _controller,
              itemCount: 50,
              itemBuilder: (context, index) {
                return Container(
                  height: 60,
                  color: Colors.blue[100 * ((index + 1) % 9 + 1)],
                  margin: const EdgeInsets.symmetric(
                    horizontal: 8,
                    vertical: 4,
                  ),
                  child: Center(
                    child: Text(
                      'Item $index',
                      style: const TextStyle(
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

// ScrollController properties:
// 1. position - Current scroll position
// 2. offset - Current scroll offset
// 3. hasClients - Whether controller is attached
// 4. hasListeners - Whether listeners are attached

// ScrollController methods:
// 1. animateTo() - Animated scroll to position
// 2. jumpTo() - Jump to position
// 3. addListener() - Add scroll listener
// 4. removeListener() - Remove scroll listener
// 5. dispose() - Clean up controller
```

What's happening here?
- Controller tracks scroll position
- Animate to specific positions
- Jump to positions instantly
- Listen to scroll events
- Control scrolling programmatically

---

# ScrollPhysics

> **ScrollPhysics** controls scroll behavior and physics.

```dart
// ScrollPhysics examples
class ScrollPhysicsExample extends StatelessWidget {
  const ScrollPhysicsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. BouncingScrollPhysics - iOS style
        Expanded(
          child: ListView.builder(
            physics: const BouncingScrollPhysics(),
            itemCount: 30,
            itemBuilder: (context, index) {
              return Container(
                height: 50,
                color: Colors.red[100 * (index % 9 + 1)],
                margin: const EdgeInsets.all(4),
                child: Center(child: Text('Bouncing $index')),
              );
            },
          ),
        ),
        
        // 2. ClampingScrollPhysics - Android style
        Expanded(
          child: ListView.builder(
            physics: const ClampingScrollPhysics(),
            itemCount: 30,
            itemBuilder: (context, index) {
              return Container(
                height: 50,
                color: Colors.green[100 * (index % 9 + 1)],
                margin: const EdgeInsets.all(4),
                child: Center(child: Text('Clamping $index')),
              );
            },
          ),
        ),
        
        // 3. NeverScrollableScrollPhysics - Disable scrolling
        Expanded(
          child: ListView.builder(
            physics: const NeverScrollableScrollPhysics(),
            itemCount: 10,
            itemBuilder: (context, index) {
              return Container(
                height: 50,
                color: Colors.blue[100 * (index % 9 + 1)],
                margin: const EdgeInsets.all(4),
                child: Center(child: Text('Not Scrollable $index')),
              );
            },
          ),
        ),
        
        // 4. AlwaysScrollableScrollPhysics - Always scrollable
        Expanded(
          child: ListView.builder(
            physics: const AlwaysScrollableScrollPhysics(),
            itemCount: 5,
            itemBuilder: (context, index) {
              return Container(
                height: 50,
                color: Colors.orange[100 * (index % 9 + 1)],
                margin: const EdgeInsets.all(4),
                child: Center(child: Text('Always Scrollable $index')),
              );
            },
          ),
        ),
      ],
    );
  }
}

// ScrollPhysics options:
// 1. BouncingScrollPhysics - Bounces at edge (iOS)
// 2. ClampingScrollPhysics - Clamps at edge (Android)
// 3. NeverScrollableScrollPhysics - Disables scrolling
// 4. AlwaysScrollableScrollPhysics - Always scrollable
// 5. RangeMaintainingScrollPhysics - Maintains range
// 6. Custom physics - Extend ScrollPhysics
```

What's happening here?
- BouncingScrollPhysics: iOS-style bounce
- ClampingScrollPhysics: Android-style clamp
- NeverScrollableScrollPhysics: no scrolling
- AlwaysScrollableScrollPhysics: always scrollable

---

# ScrollBehavior

> **ScrollBehavior** customizes scroll appearance.

```dart
// Custom scroll behavior
class CustomScrollBehavior extends ScrollBehavior {
  @override
  Widget buildViewportChrome(
    BuildContext context,
    Widget child,
    AxisDirection axisDirection,
  ) {
    // Custom scrollbar
    return GlowingOverscrollIndicator(
      child: child,
      axisDirection: axisDirection,
      color: Colors.blue,
    );
  }
}

class ScrollBehaviorExample extends StatelessWidget {
  const ScrollBehaviorExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ScrollBehavior')),
      body: ScrollConfiguration(
        behavior: CustomScrollBehavior(),
        child: ListView.builder(
          itemCount: 50,
          itemBuilder: (context, index) {
            return Container(
              height: 60,
              color: Colors.blue[100 * (index % 9 + 1)],
              margin: const EdgeInsets.all(4),
              child: Center(
                child: Text(
                  'Item $index',
                  style: const TextStyle(
                    color: Colors.white,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

What's happening here?
- Custom scroll behavior
- Custom scrollbar appearance
- Overscroll indicator customization
- Platform-specific behavior

---

# Real-World Examples

> **Common patterns** using scrollable widgets.

```dart
// 1. Infinite scrolling with pagination
class InfiniteScrollExample extends StatefulWidget {
  const InfiniteScrollExample({super.key});

  @override
  State<InfiniteScrollExample> createState() => _InfiniteScrollExampleState();
}

class _InfiniteScrollExampleState extends State<InfiniteScrollExample> {
  List<int> _items = [];
  bool _isLoading = false;
  final ScrollController _controller = ScrollController();

  @override
  void initState() {
    super.initState();
    _loadMore();
    _controller.addListener(() {
      if (_controller.position.pixels >= _controller.position.maxScrollExtent - 200) {
        _loadMore();
      }
    });
  }

  Future<void> _loadMore() async {
    if (_isLoading) return;
    setState(() => _isLoading = true);
    
    await Future.delayed(const Duration(seconds: 1));
    final start = _items.length;
    _items.addAll(List.generate(20, (i) => start + i));
    
    setState(() => _isLoading = false);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Infinite Scroll')),
      body: ListView.builder(
        controller: _controller,
        itemCount: _items.length + (_isLoading ? 1 : 0),
        itemBuilder: (context, index) {
          if (index == _items.length) {
            return const Center(child: CircularProgressIndicator());
          }
          return Container(
            height: 60,
            color: Colors.blue[100 * (_items[index] % 9 + 1)],
            margin: const EdgeInsets.all(4),
            child: Center(
              child: Text(
                'Item ${_items[index]}',
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
            ),
          );
        },
      ),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}

// 2. Horizontal scroll
class HorizontalScrollExample extends StatelessWidget {
  const HorizontalScrollExample({super.key});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: 120,
      child: ListView.builder(
        scrollDirection: Axis.horizontal,
        itemCount: 20,
        itemBuilder: (context, index) {
          return Container(
            width: 100,
            margin: const EdgeInsets.all(8),
            decoration: BoxDecoration(
              color: Colors.blue[100 * (index % 9 + 1)],
              borderRadius: BorderRadius.circular(8),
            ),
            child: Center(
              child: Text(
                'Item $index',
                style: const TextStyle(
                  fontWeight: FontWeight.bold,
                  color: Colors.white,
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}

// 3. Scroll to top button
class ScrollToTopExample extends StatefulWidget {
  const ScrollToTopExample({super.key});

  @override
  State<ScrollToTopExample> createState() => _ScrollToTopExampleState();
}

class _ScrollToTopExampleState extends State<ScrollToTopExample> {
  final ScrollController _controller = ScrollController();
  bool _showButton = false;

  @override
  void initState() {
    super.initState();
    _controller.addListener(() {
      setState(() {
        _showButton = _controller.position.pixels > 100;
      });
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _scrollToTop() {
    _controller.animateTo(
      0,
      duration: const Duration(milliseconds: 500),
      curve: Curves.easeInOut,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Scroll to Top')),
      body: Stack(
        children: [
          ListView.builder(
            controller: _controller,
            itemCount: 50,
            itemBuilder: (context, index) {
              return Container(
                height: 60,
                color: Colors.blue[100 * (index % 9 + 1)],
                margin: const EdgeInsets.all(4),
                child: Center(
                  child: Text(
                    'Item $index',
                    style: const TextStyle(
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              );
            },
          ),
          if (_showButton)
            Positioned(
              bottom: 20,
              right: 20,
              child: FloatingActionButton(
                onPressed: _scrollToTop,
                child: const Icon(Icons.arrow_upward),
              ),
            ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Infinite scrolling with pagination
- Horizontal scroll view
- Scroll to top with floating button

---

# Best Practices

## Use Appropriate Widget

```dart
// Good - ListView for simple lists
@override
Widget build(BuildContext context) {
  return ListView.builder(
    itemCount: items.length,
    itemBuilder: (context, index) {
      return Text(items[index]);
    },
  );
}

// Bad - Using CustomScrollView for simple list
@override
Widget build(BuildContext context) {
  return CustomScrollView(
    slivers: [
      SliverList(
        delegate: SliverChildBuilderDelegate(
          (context, index) => Text(items[index]),
          childCount: items.length,
        ),
      ),
    ],
  );
}
```

## Dispose ScrollController

```dart
// Good - Dispose controller
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}

// Bad - Not disposing
@override
void dispose() {
  super.dispose();
}
```

## Use Lazy Loading

```dart
// Good - Lazy loading
@override
Widget build(BuildContext context) {
  return ListView.builder(
    itemCount: 1000,
    itemBuilder: (context, index) {
      return Text('Item $index');
    },
  );
}

// Bad - All items built at once
@override
Widget build(BuildContext context) {
  return ListView(
    children: List.generate(1000, (index) {
      return Text('Item $index');
    }),
  );
}
```

---

# Common Mistakes

## Not Using Builder

Wrong:
```dart
// Inefficient for large lists
ListView(
  children: [
    for (int i = 0; i < 1000; i++)
      Text('Item $i'),
  ],
)
```

Correct:
```dart
// Efficient for large lists
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) {
    return Text('Item $index');
  },
)
```

## Not Disposing Controller

Wrong:
```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final ScrollController _controller = ScrollController();
  // No dispose method
}
```

Correct:
```dart
class _MyWidgetState extends State<MyWidget> {
  final ScrollController _controller = ScrollController();
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

---

# Summary

ScrollView is the foundation for all scrollable content in Flutter. It provides scrolling infrastructure and supports various scroll behaviors through slivers, controllers, and physics. Use specialized widgets like ListView, GridView, and CustomScrollView for most use cases.

---

# Next Steps

- [ListView](listview.md)
- [GridView](gridview.md)
- [CustomScrollView](customscrollview.md)

---

# Did You Know?

- ScrollView is the base for all scrolling
- Slivers are the building blocks of ScrollView
- ListView and GridView extend ScrollView
- CustomScrollView uses slivers directly
- ScrollController controls scroll position
- ScrollPhysics controls scroll behavior
- ScrollBehavior customizes appearance
- Lazy loading improves performance