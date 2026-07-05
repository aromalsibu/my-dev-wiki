# Flow

Understand how to create custom multi-child layouts with Flow.

---

# What is it?

Flow is a layout widget that provides more control over multi-child layouts than Wrap. Unlike Wrap, which automatically positions children, Flow allows you to define custom positioning and painting logic for each child using a delegate. This makes it ideal for complex custom layouts, animations, and situations where you need precise control over child positioning.

---

# Why does it exist?

Flow exists to:

- Provide custom multi-child layout control
- Enable complex positioning algorithms
- Support custom painting and animations
- Create advanced UI patterns
- Optimize performance for dynamic layouts
- Give full control over child placement
- Handle custom flow arrangements

---

# Flow vs Wrap

> **Flow provides more control** than Wrap.

```dart
// Wrap - Automatic positioning
class WrapExample extends StatelessWidget {
  const WrapExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Wrap(
      spacing: 8,
      runSpacing: 8,
      children: [
        for (int i = 0; i < 20; i++)
          Container(
            width: 60,
            height: 40,
            color: Colors.blue[100 * (i % 9 + 1)],
            child: Center(child: Text('$i')),
          ),
      ],
    );
  }
}

// Flow - Custom positioning
class FlowExample extends StatelessWidget {
  const FlowExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Flow(
      delegate: CustomFlowDelegate(),
      children: [
        for (int i = 0; i < 20; i++)
          Container(
            width: 60,
            height: 40,
            color: Colors.blue[100 * (i % 9 + 1)],
            child: Center(child: Text('$i')),
          ),
      ],
    );
  }
}

class CustomFlowDelegate extends FlowDelegate {
  @override
  void paintChildren(FlowPaintingContext context) {
    // Custom positioning logic
    for (int i = 0; i < context.childCount; i++) {
      final x = (i % 4) * 70;
      final y = (i ~/ 4) * 50;
      context.paintChild(i, transform: Matrix4.translationValues(x, y, 0));
    }
  }

  @override
  bool shouldRepaint(covariant FlowDelegate oldDelegate) {
    return false;
  }
}
```

What's happening here?
- Wrap: automatic positioning
- Flow: custom positioning
- Flow gives full control
- Flow can optimize performance
- Flow supports custom painting

---

# FlowDelegate

> **FlowDelegate** controls positioning and painting.

```dart
// Custom FlowDelegate
class CustomFlowDelegate extends FlowDelegate {
  // 1. paintChildren - Main painting method
  @override
  void paintChildren(FlowPaintingContext context) {
    // Get container size
    final size = context.size;
    
    // Position each child
    for (int i = 0; i < context.childCount; i++) {
      // Get child size
      final childSize = context.getChildSize(i)!;
      
      // Calculate position
      final x = (i % 3) * (childSize.width + 10);
      final y = (i ~/ 3) * (childSize.height + 10);
      
      // Paint child with transformation
      context.paintChild(
        i,
        transform: Matrix4.translationValues(x, y, 0),
      );
    }
  }

  // 2. shouldRepaint - When to repaint
  @override
  bool shouldRepaint(covariant FlowDelegate oldDelegate) {
    // Return true when need to repaint
    return false;
  }

  // 3. getSize - Custom size (optional)
  @override
  Size getSize(BoxConstraints constraints) {
    // Return custom size
    return constraints.constrain(const Size(400, 400));
  }

  // 4. getConstraintsForChild - Child constraints (optional)
  @override
  BoxConstraints getConstraintsForChild(int i, BoxConstraints constraints) {
    // Custom constraints for each child
    return BoxConstraints.loose(const Size(60, 40));
  }
}

// Using the custom delegate
class FlowUsageExample extends StatelessWidget {
  const FlowUsageExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Flow(
      delegate: CustomFlowDelegate(),
      children: [
        for (int i = 0; i < 20; i++)
          Container(
            width: 60,
            height: 40,
            color: Colors.blue[100 * (i % 9 + 1)],
            child: Center(child: Text('$i')),
          ),
      ],
    );
  }
}
```

What's happening here?
- paintChildren: main positioning logic
- shouldRepaint: control repaints
- getSize: custom container size
- getConstraintsForChild: per-child constraints
- Matrix4 transformations for positioning

---

# Layout Patterns with Flow

> **Common layout patterns** using Flow.

```dart
// 1. Grid layout with Flow
class GridFlowDelegate extends FlowDelegate {
  final int crossAxisCount;
  final double spacing;
  final double runSpacing;

  GridFlowDelegate({
    this.crossAxisCount = 3,
    this.spacing = 8,
    this.runSpacing = 8,
  });

  @override
  void paintChildren(FlowPaintingContext context) {
    final childWidth = (context.size.width - (crossAxisCount - 1) * spacing) / crossAxisCount;
    
    for (int i = 0; i < context.childCount; i++) {
      final row = i ~/ crossAxisCount;
      final col = i % crossAxisCount;
      
      final x = col * (childWidth + spacing);
      final y = row * (60 + runSpacing);
      
      context.paintChild(
        i,
        transform: Matrix4.translationValues(x, y, 0),
        size: Size(childWidth, 50),
      );
    }
  }

  @override
  Size getSize(BoxConstraints constraints) {
    // Calculate total height
    final childCount = constraints.maxWidth > 0 ? 20 : 0;
    final rows = (childCount / crossAxisCount).ceil();
    final height = rows * (50 + runSpacing) - runSpacing;
    return constraints.constrain(Size(constraints.maxWidth, height));
  }

  @override
  bool shouldRepaint(covariant GridFlowDelegate oldDelegate) {
    return crossAxisCount != oldDelegate.crossAxisCount ||
           spacing != oldDelegate.spacing ||
           runSpacing != oldDelegate.runSpacing;
  }
}

// 2. Masonry layout (Pinterest-style)
class MasonryFlowDelegate extends FlowDelegate {
  final List<double> columnHeights;
  final int columnCount;

  MasonryFlowDelegate({
    this.columnCount = 2,
    List<double>? columnHeights,
  }) : columnHeights = columnHeights ?? List.filled(2, 0);

  @override
  void paintChildren(FlowPaintingContext context) {
    // Reset column heights
    for (int i = 0; i < columnHeights.length; i++) {
      columnHeights[i] = 0;
    }

    for (int i = 0; i < context.childCount; i++) {
      // Find shortest column
      int shortestColumn = 0;
      double minHeight = columnHeights[0];
      for (int j = 1; j < columnHeights.length; j++) {
        if (columnHeights[j] < minHeight) {
          minHeight = columnHeights[j];
          shortestColumn = j;
        }
      }

      // Get child size
      final childSize = context.getChildSize(i)!;
      final x = shortestColumn * (childSize.width + 8);
      final y = columnHeights[shortestColumn];

      // Paint child
      context.paintChild(
        i,
        transform: Matrix4.translationValues(x, y, 0),
      );

      // Update column height
      columnHeights[shortestColumn] += childSize.height + 8;
    }
  }

  @override
  Size getSize(BoxConstraints constraints) {
    final maxHeight = columnHeights.reduce((a, b) => a > b ? a : b);
    return constraints.constrain(Size(constraints.maxWidth, maxHeight));
  }

  @override
  bool shouldRepaint(covariant MasonryFlowDelegate oldDelegate) {
    return columnCount != oldDelegate.columnCount;
  }
}

// 3. Circular layout with Flow
class CircularFlowDelegate extends FlowDelegate {
  final double radius;
  final double startAngle;

  CircularFlowDelegate({
    this.radius = 100,
    this.startAngle = 0,
  });

  @override
  void paintChildren(FlowPaintingContext context) {
    final centerX = context.size.width / 2;
    final centerY = context.size.height / 2;

    for (int i = 0; i < context.childCount; i++) {
      final angle = startAngle + (i / context.childCount) * 2 * 3.14159;
      
      final x = centerX + radius * cos(angle) - 25;
      final y = centerY + radius * sin(angle) - 25;

      context.paintChild(
        i,
        transform: Matrix4.translationValues(x, y, 0),
      );
    }
  }

  @override
  Size getSize(BoxConstraints constraints) {
    return constraints.constrain(const Size(250, 250));
  }

  @override
  bool shouldRepaint(covariant CircularFlowDelegate oldDelegate) {
    return radius != oldDelegate.radius || 
           startAngle != oldDelegate.startAngle;
  }
}
```

What's happening here?
- Grid layout with custom spacing
- Masonry layout (Pinterest-style)
- Circular layout with trigonometry
- Custom positioning algorithms
- Flexible delegate patterns

---

# Animated Flow

> **Flow can be animated** for dynamic layouts.

```dart
// Animated Flow
class AnimatedFlowExample extends StatefulWidget {
  const AnimatedFlowExample({super.key});

  @override
  State<AnimatedFlowExample> createState() => _AnimatedFlowExampleState();
}

class _AnimatedFlowExampleState extends State<AnimatedFlowExample> {
  int _childCount = 5;
  double _spacing = 10;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _childCount = (_childCount + 1).clamp(1, 20);
                });
              },
              child: const Text('Add'),
            ),
            const SizedBox(width: 10),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _childCount = (_childCount - 1).clamp(1, 20);
                });
              },
              child: const Text('Remove'),
            ),
            const SizedBox(width: 10),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _spacing = _spacing == 10 ? 30 : 10;
                });
              },
              child: const Text('Toggle Spacing'),
            ),
          ],
        ),
        const SizedBox(height: 20),
        Container(
          height: 300,
          color: Colors.grey[200],
          child: Flow(
            delegate: AnimatedFlowDelegate(
              childCount: _childCount,
              spacing: _spacing,
            ),
            children: List.generate(
              _childCount,
              (index) => Container(
                width: 50,
                height: 50,
                color: Colors.blue[100 * ((index % 9) + 1)],
                child: Center(
                  child: Text(
                    '${index + 1}',
                    style: const TextStyle(
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ),
            ),
          ),
        ),
      ],
    );
  }
}

class AnimatedFlowDelegate extends FlowDelegate {
  final int childCount;
  final double spacing;

  AnimatedFlowDelegate({
    required this.childCount,
    required this.spacing,
  });

  @override
  void paintChildren(FlowPaintingContext context) {
    final cols = (context.size.width / (60 + spacing)).floor().clamp(1, childCount);
    final rows = (childCount / cols).ceil();

    for (int i = 0; i < context.childCount; i++) {
      final row = i ~/ cols;
      final col = i % cols;

      final x = col * (60 + spacing);
      final y = row * (60 + spacing);

      context.paintChild(
        i,
        transform: Matrix4.translationValues(x, y, 0),
      );
    }
  }

  @override
  Size getSize(BoxConstraints constraints) {
    final cols = (constraints.maxWidth / (60 + spacing)).floor().clamp(1, childCount);
    final rows = (childCount / cols).ceil();
    final height = rows * (60 + spacing) - spacing;
    return constraints.constrain(Size(constraints.maxWidth, height));
  }

  @override
  bool shouldRepaint(covariant AnimatedFlowDelegate oldDelegate) {
    return childCount != oldDelegate.childCount ||
           spacing != oldDelegate.spacing;
  }
}
```

What's happening here?
- Dynamic child count
- Animated spacing changes
- Automatic layout updates
- Responsive to changes

---

# Flow Performance

> **Flow optimizes** performance for complex layouts.

```dart
// Performance optimization with Flow
class OptimizedFlowExample extends StatelessWidget {
  const OptimizedFlowExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Flow(
      delegate: OptimizedFlowDelegate(),
      children: [
        for (int i = 0; i < 100; i++)
          Container(
            width: 50,
            height: 50,
            color: Colors.blue[100 * (i % 9 + 1)],
            child: Center(child: Text('$i')),
          ),
      ],
    );
  }
}

class OptimizedFlowDelegate extends FlowDelegate {
  @override
  void paintChildren(FlowPaintingContext context) {
    // Cache calculations
    final childSize = context.getChildSize(0)!;
    final spacing = 8.0;
    final cols = (context.size.width / (childSize.width + spacing)).floor();

    for (int i = 0; i < context.childCount; i++) {
      final row = i ~/ cols;
      final col = i % cols;

      final x = col * (childSize.width + spacing);
      final y = row * (childSize.height + spacing);

      // Use pre-calculated transforms
      context.paintChild(
        i,
        transform: Matrix4.translationValues(x, y, 0),
      );
    }
  }

  @override
  bool shouldRepaint(covariant FlowDelegate oldDelegate) {
    // Only repaint when needed
    return false;
  }
}

// Performance tips:
// 1. Cache child sizes
// 2. Pre-calculate positions
// 3. Minimize shouldRepaint
// 4. Use const children when possible
// 5. Limit child count
// 6. Use RepaintBoundary if needed
```

What's happening here?
- Cache calculations for performance
- Minimize repaints
- Pre-calculate positions
- Efficient for many children

---

# Best Practices

## Use Flow for Custom Layouts

```dart
// Good - Custom layout with Flow
@override
Widget build(BuildContext context) {
  return Flow(
    delegate: MyCustomDelegate(),
    children: customChildren,
  );
}
```

## Cache Calculations

```dart
// Good - Cache calculations
@override
void paintChildren(FlowPaintingContext context) {
  if (_cachedPositions == null) {
    _cachedPositions = _calculatePositions(context);
  }
  // Use cached positions
}

// Bad - Recalculate every time
@override
void paintChildren(FlowPaintingContext context) {
  for (int i = 0; i < context.childCount; i++) {
    // Calculate positions each time
  }
}
```

## Minimize Repaints

```dart
// Good - Only repaint when needed
@override
bool shouldRepaint(covariant FlowDelegate oldDelegate) {
  return false; // Or check specific conditions
}
```

---

# Common Mistakes

## Recalculating Every Frame

Wrong:
```dart
@override
void paintChildren(FlowPaintingContext context) {
  for (int i = 0; i < context.childCount; i++) {
    // Heavy calculation every frame
    final pos = calculateComplexPosition(i);
  }
}
```

Correct:
```dart
@override
void paintChildren(FlowPaintingContext context) {
  // Cache calculations
  if (_positions == null) {
    _positions = calculateAllPositions(context);
  }
  // Use cached positions
}
```

## Not Using getSize

Wrong:
```dart
@override
void paintChildren(FlowPaintingContext context) {
  // Assuming size without calculating
}
```

Correct:
```dart
@override
Size getSize(BoxConstraints constraints) {
  return constraints.constrain(Size(400, 400));
}
```

---

# Summary

Flow provides custom multi-child layout control through delegates. Use Flow when you need precise positioning, custom algorithms, or complex layouts. Flow offers better performance than Wrap for large numbers of children and supports animations and dynamic content.

---

# Next Steps

- [Align](align.md)
- [Center](center.md)
- [SizedBox](sizedbox.md)

---

# Did You Know?

- Flow gives full control over positioning
- FlowDelegate defines custom layout
- Flow can animate positions
- Flow is more performant than Wrap for many children
- getSize controls container size
- shouldRepaint controls repaint behavior
- Matrix4 transforms position children
- Flow supports custom painting