# ScrollPhysics

Understand how to control scroll behavior and physics in Flutter.

---

# What is it?

ScrollPhysics defines the physical behavior of scrollable widgets in Flutter. It controls how a scrollable view responds to user interactions like dragging, flinging, and overscrolling. Different physics objects provide different scrolling behaviors, from Android-style clamping to iOS-style bouncing.

---

# Why does it exist?

ScrollPhysics exists to:

- Define scroll behavior and animations
- Control overscroll effects
- Implement platform-specific scrolling
- Customize scroll interactions
- Handle drag and fling physics
- Create custom scroll behaviors
- Ensure consistent user experience

---

# ScrollPhysics Types

> **Different physics** for different behaviors.

```dart
// ScrollPhysics comparison
class ScrollPhysicsComparison extends StatelessWidget {
  const ScrollPhysicsComparison({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. BouncingScrollPhysics - iOS style
        Expanded(
          child: Container(
            color: Colors.grey[200],
            child: Column(
              children: [
                const Padding(
                  padding: EdgeInsets.all(8),
                  child: Text(
                    'Bouncing (iOS)',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                ),
                Expanded(
                  child: ListView.builder(
                    physics: const BouncingScrollPhysics(),
                    itemCount: 20,
                    itemBuilder: (context, index) {
                      return Container(
                        height: 50,
                        color: Colors.blue[100 * (index % 9 + 1)],
                        margin: const EdgeInsets.all(4),
                        child: Center(child: Text('Item $index')),
                      );
                    },
                  ),
                ),
              ],
            ),
          ),
        ),
        
        const SizedBox(height: 8),
        
        // 2. ClampingScrollPhysics - Android style
        Expanded(
          child: Container(
            color: Colors.grey[200],
            child: Column(
              children: [
                const Padding(
                  padding: EdgeInsets.all(8),
                  child: Text(
                    'Clamping (Android)',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                ),
                Expanded(
                  child: ListView.builder(
                    physics: const ClampingScrollPhysics(),
                    itemCount: 20,
                    itemBuilder: (context, index) {
                      return Container(
                        height: 50,
                        color: Colors.green[100 * (index % 9 + 1)],
                        margin: const EdgeInsets.all(4),
                        child: Center(child: Text('Item $index')),
                      );
                    },
                  ),
                ),
              ],
            ),
          ),
        ),
      ],
    );
  }
}

// ScrollPhysics options:
// 1. BouncingScrollPhysics - Bounces at edges (iOS-like)
// 2. ClampingScrollPhysics - Stops at edges (Android-like)
// 3. NeverScrollableScrollPhysics - Disables scrolling
// 4. AlwaysScrollableScrollPhysics - Always scrollable
// 5. RangeMaintainingScrollPhysics - Maintains range
// 6. FixedExtentScrollPhysics - Fixed extent scrolling
// 7. PageScrollPhysics - Page-by-page scrolling
```

What's happening here?
- BouncingScrollPhysics: iOS-style bounce
- ClampingScrollPhysics: Android-style clamp
- Different behaviors for different platforms
- Choose based on platform or preference

---

# Physics Comparison

> **Detailed comparison** of physics types.

```dart
// Detailed physics comparison
class PhysicsDetailComparison extends StatelessWidget {
  const PhysicsDetailComparison({super.key});

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      child: Column(
        children: [
          // 1. BouncingScrollPhysics
          _buildPhysicsDemo(
            'BouncingScrollPhysics',
            'Bounces when reaching edges. iOS style.',
            const BouncingScrollPhysics(),
            Colors.blue,
          ),
          
          const SizedBox(height: 16),
          
          // 2. ClampingScrollPhysics
          _buildPhysicsDemo(
            'ClampingScrollPhysics',
            'Stops at edges with no bounce. Android style.',
            const ClampingScrollPhysics(),
            Colors.green,
          ),
          
          const SizedBox(height: 16),
          
          // 3. NeverScrollableScrollPhysics
          _buildPhysicsDemo(
            'NeverScrollableScrollPhysics',
            'Scrolling is completely disabled.',
            const NeverScrollableScrollPhysics(),
            Colors.red,
          ),
          
          const SizedBox(height: 16),
          
          // 4. AlwaysScrollableScrollPhysics
          _buildPhysicsDemo(
            'AlwaysScrollableScrollPhysics',
            'Always allows scrolling even with insufficient content.',
            const AlwaysScrollableScrollPhysics(),
            Colors.orange,
          ),
          
          const SizedBox(height: 16),
          
          // 5. PageScrollPhysics
          _buildPhysicsDemo(
            'PageScrollPhysics',
            'Scrolls page by page. Used with PageView.',
            const PageScrollPhysics(),
            Colors.purple,
          ),
        ],
      ),
    );
  }

  Widget _buildPhysicsDemo(
    String title,
    String description,
    ScrollPhysics physics,
    Color color,
  ) {
    return Container(
      height: 150,
      decoration: BoxDecoration(
        color: Colors.grey[100],
        borderRadius: BorderRadius.circular(8),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Padding(
            padding: const EdgeInsets.all(8),
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
                Text(
                  description,
                  style: TextStyle(
                    color: Colors.grey[600],
                    fontSize: 12,
                  ),
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView.builder(
              physics: physics,
              scrollDirection: Axis.horizontal,
              itemCount: 10,
              itemBuilder: (context, index) {
                return Container(
                  width: 80,
                  margin: const EdgeInsets.all(4),
                  color: color.withOpacity(0.3 + (index % 5) * 0.1),
                  child: Center(child: Text('$index')),
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
- Different physics for different use cases
- Each has unique behavior
- Visual demonstration of each type
- Choose based on requirements

---

# Custom ScrollPhysics

> **Creating custom** scroll physics.

```dart
// Custom ScrollPhysics
class CustomScrollPhysics extends ScrollPhysics {
  final double friction;
  final double springStrength;

  const CustomScrollPhysics({
    this.friction = 0.5,
    this.springStrength = 100.0,
    super.parent,
  });

  @override
  CustomScrollPhysics applyTo(ScrollPhysics? ancestor) {
    return CustomScrollPhysics(
      friction: friction,
      springStrength: springStrength,
      parent: ancestor,
    );
  }

  @override
  double applyPhysicsToUserOffset(ScrollMetrics position, double offset) {
    // Apply custom friction to scroll
    return offset * (1 - friction);
  }

  @override
  double applyBoundaryConditions(ScrollMetrics position, double value) {
    // Custom boundary conditions with spring effect
    if (value < position.minScrollExtent) {
      final double overscroll = position.minScrollExtent - value;
      return -overscroll / springStrength;
    }
    if (value > position.maxScrollExtent) {
      final double overscroll = value - position.maxScrollExtent;
      return overscroll / springStrength;
    }
    return 0.0;
  }

  @override
  double carriedMomentum(double existingMomentum) {
    // Custom momentum decay
    return existingMomentum * 0.9;
  }
}

// Using custom physics
class CustomPhysicsExample extends StatelessWidget {
  const CustomPhysicsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Physics'),
      ),
      body: ListView.builder(
        physics: const CustomScrollPhysics(
          friction: 0.3,
          springStrength: 50.0,
        ),
        itemCount: 30,
        itemBuilder: (context, index) {
          return Container(
            height: 60,
            color: Colors.blue[100 * (index % 9 + 1)],
            margin: const EdgeInsets.all(4),
            child: Center(
              child: Text(
                'Custom Physics Item $index',
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

What's happening here?
- Custom physics with friction control
- Spring effect on boundaries
- Custom momentum handling
- Extend ScrollPhysics for custom behavior

---

# ScrollPhysics with Different Widgets

> **Physics in different** scrollable widgets.

```dart
// Physics in various widgets
class PhysicsInWidgets extends StatelessWidget {
  const PhysicsInWidgets({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Physics in Widgets'),
      ),
      body: Column(
        children: [
          // 1. ListView with custom physics
          Expanded(
            child: Container(
              color: Colors.grey[200],
              child: Column(
                children: [
                  const Padding(
                    padding: EdgeInsets.all(8),
                    child: Text('ListView with Bouncing Physics'),
                  ),
                  Expanded(
                    child: ListView.builder(
                      physics: const BouncingScrollPhysics(),
                      itemCount: 15,
                      itemBuilder: (context, index) {
                        return Container(
                          height: 50,
                          color: Colors.blue[100 * (index % 9 + 1)],
                          margin: const EdgeInsets.all(4),
                          child: Center(child: Text('Item $index')),
                        );
                      },
                    ),
                  ),
                ],
              ),
            ),
          ),
          
          const SizedBox(height: 8),
          
          // 2. GridView with clamping physics
          Expanded(
            child: Container(
              color: Colors.grey[200],
              child: Column(
                children: [
                  const Padding(
                    padding: EdgeInsets.all(8),
                    child: Text('GridView with Clamping Physics'),
                  ),
                  Expanded(
                    child: GridView.builder(
                      physics: const ClampingScrollPhysics(),
                      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                        crossAxisCount: 3,
                        crossAxisSpacing: 4,
                        mainAxisSpacing: 4,
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
              ),
            ),
          ),
          
          const SizedBox(height: 8),
          
          // 3. PageView with page physics
          Expanded(
            child: Container(
              color: Colors.grey[200],
              child: Column(
                children: [
                  const Padding(
                    padding: EdgeInsets.all(8),
                    child: Text('PageView with Page Physics'),
                  ),
                  Expanded(
                    child: PageView(
                      physics: const PageScrollPhysics(),
                      children: [
                        for (int i = 0; i < 3; i++)
                          Container(
                            color: Colors.orange[100 * (i + 1)],
                            child: Center(
                              child: Text(
                                'Page ${i + 1}',
                                style: const TextStyle(
                                  fontSize: 24,
                                  fontWeight: FontWeight.bold,
                                ),
                              ),
                            ),
                          ),
                      ],
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
- Physics in ListView, GridView, PageView
- Different widgets use different physics
- Consistent behavior across widgets
- Physics is customizable per widget

---

# Real-World Examples

> **Common patterns** using ScrollPhysics.

```dart
// 1. iOS-style scroll view
class IOSScrollView extends StatelessWidget {
  const IOSScrollView({super.key});

  @override
  Widget build(BuildContext context) {
    return CupertinoPageScaffold(
      navigationBar: const CupertinoNavigationBar(
        middle: Text('iOS Style Scroll'),
      ),
      child: SafeArea(
        child: ListView.builder(
          // iOS-style bouncing
          physics: const BouncingScrollPhysics(),
          itemCount: 30,
          itemBuilder: (context, index) {
            return Container(
              height: 60,
              color: Colors.blue[100 * (index % 9 + 1)],
              margin: const EdgeInsets.symmetric(
                horizontal: 16,
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
    );
  }
}

// 2. Android-style scroll view
class AndroidScrollView extends StatelessWidget {
  const AndroidScrollView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Android Style Scroll'),
      ),
      body: ListView.builder(
        // Android-style clamping
        physics: const ClampingScrollPhysics(),
        itemCount: 30,
        itemBuilder: (context, index) {
          return Container(
            height: 60,
            color: Colors.green[100 * (index % 9 + 1)],
            margin: const EdgeInsets.symmetric(
              horizontal: 16,
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
    );
  }
}

// 3. Disabled scrolling when content fits
class AdaptiveScrollView extends StatelessWidget {
  const AdaptiveScrollView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Adaptive Scroll'),
      ),
      body: LayoutBuilder(
        builder: (context, constraints) {
          // Calculate if content fits in viewport
          final contentHeight = 10 * 60 + 10 * 8; // 10 items * 60px + spacing
          final fitsInViewport = contentHeight <= constraints.maxHeight;
          
          return ListView.builder(
            // Disable scrolling if content fits
            physics: fitsInViewport 
                ? const NeverScrollableScrollPhysics()
                : const ClampingScrollPhysics(),
            itemCount: fitsInViewport ? 5 : 30,
            itemBuilder: (context, index) {
              return Container(
                height: 60,
                color: Colors.blue[100 * (index % 9 + 1)],
                margin: const EdgeInsets.symmetric(
                  horizontal: 16,
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
          );
        },
      ),
    );
  }
}

// 4. Sticky header with physics
class StickyHeaderScrollView extends StatelessWidget {
  const StickyHeaderScrollView({super.key});

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      physics: const BouncingScrollPhysics(),
      slivers: [
        SliverAppBar(
          title: const Text('Sticky Header'),
          expandedHeight: 150,
          flexibleSpace: const FlexibleSpaceBar(
            background: ColoredBox(
              color: Colors.blue,
              child: Center(
                child: Text(
                  'Pull to bounce',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
          pinned: true,
        ),
        SliverList(
          delegate: SliverChildBuilderDelegate(
            (context, index) => Container(
              height: 60,
              color: Colors.blue[100 * (index % 9 + 1)],
              margin: const EdgeInsets.all(4),
              child: Center(child: Text('Item $index')),
            ),
            childCount: 20,
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- iOS-style bouncing scroll
- Android-style clamping scroll
- Adaptive scrolling based on content
- Sticky header with physics

---

# Best Practices

## Use Platform-Specific Physics

```dart
// Good - Platform-specific physics
@override
Widget build(BuildContext context) {
  return ListView.builder(
    physics: Platform.isIOS 
        ? const BouncingScrollPhysics()
        : const ClampingScrollPhysics(),
    itemCount: 20,
    itemBuilder: (context, index) => Container(...),
  );
}
```

## Use Appropriate Physics for Widget

```dart
// Good - PageView with page physics
PageView(
  physics: const PageScrollPhysics(),
  children: pages,
)

// Good - ListView with appropriate physics
ListView.builder(
  physics: const ClampingScrollPhysics(),
  itemCount: items.length,
  itemBuilder: (context, index) => ...,
)
```

## Disable Scrolling When Needed

```dart
// Good - Disable scrolling
ListView.builder(
  physics: const NeverScrollableScrollPhysics(),
  itemCount: 5,
  itemBuilder: (context, index) => Container(...),
)
```

---

# Common Mistakes

## Wrong Physics for Platform

Wrong:
```dart
// iOS app with Android physics
ListView.builder(
  physics: const ClampingScrollPhysics(), // Wrong for iOS
  itemCount: 20,
  itemBuilder: (context, index) => Container(...),
)
```

Correct:
```dart
// Platform-aware physics
ListView.builder(
  physics: Platform.isIOS 
      ? const BouncingScrollPhysics()
      : const ClampingScrollPhysics(),
  itemCount: 20,
  itemBuilder: (context, index) => Container(...),
)
```

## Physics Not Applied

Wrong:
```dart
// Physics ignored in some widgets
PageView(
  physics: const ClampingScrollPhysics(), // Not recommended for PageView
  children: pages,
)
```

Correct:
```dart
// Appropriate physics for PageView
PageView(
  physics: const PageScrollPhysics(),
  children: pages,
)
```

---

# Summary

ScrollPhysics controls the physical behavior of scrollable widgets. Use BouncingScrollPhysics for iOS-style bouncing, ClampingScrollPhysics for Android-style clamping, and other physics types for specific behaviors. Custom physics can be created by extending ScrollPhysics.

---

# Next Steps

- [Nested Scrolling](nestedscrolling.md)
- [Infinite Scrolling](infinitescrolling.md)
- [ScrollController](scrollcontroller.md)

---

# Did You Know?

- BouncingScrollPhysics is iOS-like
- ClampingScrollPhysics is Android-like
- PageScrollPhysics is for PageView
- NeverScrollableScrollPhysics disables scrolling
- AlwaysScrollableScrollPhysics forces scrolling
- Physics can be customized
- Physics affects user experience
- Physics can be platform-aware