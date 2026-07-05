# Layout Process

Understand how Flutter performs layout and positions widgets on screen.

---

# What is it?

The layout process is the mechanism by which Flutter determines the size and position of every widget on screen. It involves passing constraints down the widget tree, computing sizes, and positioning children. The layout process happens every frame and is critical for creating responsive, well-designed UIs.

---

# Why does it exist?

The layout process exists to:

- Determine the size of every widget
- Position widgets correctly on screen
- Handle different screen sizes and orientations
- Optimize rendering performance
- Support responsive and adaptive designs
- Manage constraints and sizing rules
- Enable complex layout patterns

---

# The Layout Flow

> **Layout follows** a specific flow in Flutter.

```
1. Parent passes constraints to child
   ↓
2. Child processes constraints
   ↓
3. Child determines its size
   ↓
4. Child reports size back to parent
   ↓
5. Parent positions child
   ↓
6. Parent sets its own size
```

What's happening here?
- Constraints flow down the tree
- Sizes flow back up the tree
- Positions are set by parents
- Each widget participates in layout
- Layout is recursive

---

# Constraints in Layout

> **Constraints control** how widgets can size themselves.

```dart
// Layout with constraints
class ConstraintsLayoutExample extends StatelessWidget {
  const ConstraintsLayoutExample({super.key});

  @override
  Widget build(BuildContext context) {
    // Parent container gives constraints
    return Container(
      width: 300,  // Tight constraint: must be 300 wide
      height: 200, // Tight constraint: must be 200 tall
      color: Colors.grey[200],
      child: Center(
        // Center receives constraints from parent
        // Center tells child it can be any size
        child: Container(
          // Child receives loose constraints
          // Can size itself based on content
          color: Colors.blue,
          child: const Text(
            'Hello World',
            style: TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }
}

// Layout process:
// 1. Container (300x200) passes constraints to Center
// 2. Center passes loose constraints to inner Container
// 3. Inner Container sizes itself based on text
// 4. Inner Container reports size (approx 100x20)
// 5. Center positions child in center
// 6. Container sets size to 300x200
```

What's happening here?
- Parent gives constraints to child
- Child computes size based on constraints
- Child returns size to parent
- Parent positions child
- Parent sets its own size

---

# Layout Phases

> **Layout happens in phases** for each render object.

```dart
// Layout phases example
class LayoutPhasesExample extends StatefulWidget {
  const LayoutPhasesExample({super.key});

  @override
  State<LayoutPhasesExample> createState() => _LayoutPhasesExampleState();
}

class _LayoutPhasesExampleState extends State<LayoutPhasesExample> {
  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // Phase 1: Get constraints from parent
        print('Phase 1 - Constraints: $constraints');
        
        // Phase 2: Layout children
        // Phase 3: Set own size
        // Phase 4: Position children
        
        return Container(
          width: constraints.maxWidth * 0.5,
          height: constraints.maxHeight * 0.5,
          color: Colors.blue,
          child: const Center(
            child: Text('Layout Example'),
          ),
        );
      },
    );
  }
}

// RenderObject layout phases:
// 1. setupParentData() - Initialize parent data
// 2. performLayout() - Main layout logic
// 3. layout() - Called on children
// 4. setParentData() - Set child position
// 5. markNeedsLayout() - Mark for relayout
```

What's happening here?
- LayoutBuilder reveals constraints
- Children are laid out recursively
- Sizes are computed bottom-up
- Positions are set top-down
- Layout can be triggered by changes

---

# The performLayout Method

> **performLayout** is where layout logic happens.

```dart
// Custom layout with performLayout
class CustomLayoutWidget extends LeafRenderObjectWidget {
  const CustomLayoutWidget({super.key});

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CustomRenderObject();
  }
}

class CustomRenderObject extends RenderBox {
  @override
  void performLayout() {
    // 1. Get constraints from parent
    final BoxConstraints constraints = this.constraints;
    
    // 2. Calculate size based on constraints
    // min width = 100, max width = constraints.maxWidth
    final double width = constraints.constrainWidth(200);
    final double height = constraints.constrainHeight(100);
    
    // 3. Set own size
    size = Size(width, height);
    
    // 4. Layout children if any
    // For single child:
    if (child != null) {
      // Pass constraints to child
      child!.layout(
        BoxConstraints.loose(size),
        parentUsesSize: true,
      );
      
      // Position child
      final BoxParentData childParentData = 
          child!.parentData! as BoxParentData;
      childParentData.offset = Offset.zero;
    }
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    // Paint after layout
    final Paint paint = Paint()..color = Colors.blue;
    context.canvas.drawRect(
      offset & size,
      paint,
    );
  }
}

// Using custom layout
class CustomLayoutExample extends StatelessWidget {
  const CustomLayoutExample({super.key});

  @override
  Widget build(BuildContext context) {
    return const SizedBox(
      width: 200,
      height: 100,
      child: CustomLayoutWidget(),
    );
  }
}
```

What's happening here?
- performLayout is the main layout method
- Constraints are received from parent
- Size is computed and set
- Children are laid out recursively
- Children are positioned

---

# Layout with Children

> **Multi-child layout** requires laying out each child.

```dart
// Layout with multiple children
class MultiChildLayoutWidget extends MultiChildRenderObjectWidget {
  const MultiChildLayoutWidget({
    super.key,
    required super.children,
  });

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CustomMultiChildRenderObject();
  }
}

class CustomMultiChildRenderObject extends RenderBox
    with ContainerRenderObjectMixin<RenderBox, BoxParentData>,
         RenderBoxContainerDefaultsMixin<RenderBox, BoxParentData> {
  
  @override
  void setupParentData(RenderObject child) {
    if (child.parentData is! BoxParentData) {
      child.parentData = BoxParentData();
    }
  }

  @override
  void performLayout() {
    // 1. Get constraints
    final BoxConstraints constraints = this.constraints;
    
    // 2. Layout each child
    double maxWidth = 0;
    double totalHeight = 0;
    
    RenderBox? child = firstChild;
    while (child != null) {
      // Layout child with constraints
      child.layout(
        BoxConstraints.loose(constraints),
        parentUsesSize: true,
      );
      
      // Track size
      maxWidth = max(maxWidth, child.size.width);
      totalHeight += child.size.height;
      
      // Position child (will be set after size is known)
      final BoxParentData childParentData = 
          child.parentData! as BoxParentData;
      childParentData.offset = Offset(0, totalHeight - child.size.height);
      
      child = childAfter(child);
    }
    
    // 3. Set own size
    size = constraints.constrain(Size(
      maxWidth,
      totalHeight,
    ));
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    // Paint children
    RenderBox? child = firstChild;
    while (child != null) {
      final BoxParentData childParentData = 
          child.parentData! as BoxParentData;
      context.paintChild(child, offset + childParentData.offset);
      child = childAfter(child);
    }
  }

  @override
  bool hitTestChildren(BoxHitTestResult result, {required Offset position}) {
    return defaultHitTestChildren(result, position: position);
  }
}

// Using multi-child layout
class MultiChildExample extends StatelessWidget {
  const MultiChildExample({super.key});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: 200,
      height: 150,
      child: CustomMultiChildLayoutWidget(
        children: [
          Container(color: Colors.red, height: 50),
          Container(color: Colors.green, height: 50),
          Container(color: Colors.blue, height: 50),
        ],
      ),
    );
  }
}
```

What's happening here?
- Each child is laid out individually
- Child sizes are tracked
- Children are positioned
- Own size is computed from children
- All children must be laid out

---

# Layout Constraints Types

> **Different constraint types** affect layout behavior.

```dart
// Constraint types in layout
class ConstraintTypesExample extends StatelessWidget {
  const ConstraintTypesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Tight constraints (fixed size)
        Container(
          width: 100,
          height: 100,
          color: Colors.red,
          child: const Text('Tight'),
        ),
        
        // 2. Loose constraints (flexible size)
        Container(
          constraints: const BoxConstraints(
            minWidth: 50,
            maxWidth: 200,
            minHeight: 50,
            maxHeight: 100,
          ),
          color: Colors.green,
          child: const Text('Loose'),
        ),
        
        // 3. Unbounded constraints
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              // Unbounded width constraint
              // (Row has unbounded width)
              Container(
                color: Colors.blue,
                child: const Text('Unbounded'),
              ),
            ],
          ),
        ),
        
        // 4. Aspect ratio constraints
        AspectRatio(
          aspectRatio: 16 / 9,
          child: Container(
            color: Colors.orange,
            child: const Text('Aspect Ratio'),
          ),
        ),
      ],
    );
  }
}

// Constraint properties:
// 1. minWidth - Minimum allowed width
// 2. maxWidth - Maximum allowed width
// 3. minHeight - Minimum allowed height
// 4. maxHeight - Maximum allowed height
// 5. isTight - min == max
// 6. isNormalized - min <= max
// 7. hasBoundedWidth - maxWidth != double.infinity
// 8. hasBoundedHeight - maxHeight != double.infinity
```

What's happening here?
- Tight constraints force exact size
- Loose constraints allow size range
- Unbounded constraints allow infinite size
- Aspect ratio constraints maintain ratio
- Constraint types affect layout behavior

---

# Layout with Constraints

> **Constraining children** during layout.

```dart
// Constraining children in layout
class ConstraintChildLayout extends StatelessWidget {
  const ConstraintChildLayout({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 200,
      color: Colors.grey[200],
      child: Column(
        children: [
          // 1. Child with tight constraints
          Container(
            width: 200,
            height: 50,
            color: Colors.red,
            child: const Center(
              child: Text('Tight 200x50'),
            ),
          ),
          
          // 2. Child with loose constraints
          ConstrainedBox(
            constraints: const BoxConstraints(
              minWidth: 50,
              maxWidth: 200,
              minHeight: 30,
              maxHeight: 60,
            ),
            child: Container(
              color: Colors.green,
              child: const Center(
                child: Text('Loose 50-200x30-60'),
              ),
            ),
          ),
          
          // 3. Child with no constraints (uses parent)
          Container(
            color: Colors.blue,
            child: const Center(
              child: Text('Uses parent constraints'),
            ),
          ),
        ],
      ),
    );
  }
}

// Constraining children in custom layout:
class CustomConstrainedLayout extends RenderBox {
  RenderBox? _child;

  @override
  void performLayout() {
    // Get constraints from parent
    final BoxConstraints constraints = this.constraints;
    
    if (_child != null) {
      // Pass modified constraints to child
      _child!.layout(
        BoxConstraints(
          minWidth: 50,
          maxWidth: constraints.maxWidth * 0.8,
          minHeight: 30,
          maxHeight: constraints.maxHeight * 0.8,
        ),
        parentUsesSize: true,
      );
      
      // Position child
      final BoxParentData childParentData = 
          _child!.parentData! as BoxParentData;
      childParentData.offset = Offset.zero;
      
      // Set own size to match child
      size = _child!.size;
    } else {
      size = constraints.constrain(Size(100, 50));
    }
  }
}
```

What's happening here?
- Parents can modify constraints for children
- Tight constraints force specific sizes
- Loose constraints allow flexibility
- Children can be constrained differently
- Parent controls child sizing

---

# Layout Helpers

> **Helper widgets** for common layout patterns.

```dart
// Layout helper widgets
class LayoutHelpersExample extends StatelessWidget {
  const LayoutHelpersExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. LayoutBuilder - Get constraints
        LayoutBuilder(
          builder: (context, constraints) {
            return Container(
              width: constraints.maxWidth * 0.8,
              height: 50,
              color: Colors.blue,
              child: Center(
                child: Text('Width: ${constraints.maxWidth}'),
              ),
            );
          },
        ),
        
        // 2. FractionallySizedBox - Fraction of parent
        FractionallySizedBox(
          widthFactor: 0.5,
          heightFactor: 0.5,
          child: Container(
            color: Colors.red,
            child: const Center(child: Text('50%')),
          ),
        ),
        
        // 3. AspectRatio - Maintain aspect ratio
        AspectRatio(
          aspectRatio: 2,
          child: Container(
            color: Colors.green,
            child: const Center(child: Text('2:1 Ratio')),
          ),
        ),
        
        // 4. ConstrainedBox - Custom constraints
        ConstrainedBox(
          constraints: const BoxConstraints(
            minWidth: 100,
            maxWidth: 200,
          ),
          child: Container(
            color: Colors.orange,
            child: const Center(child: Text('100-200 Width')),
          ),
        ),
        
        // 5. UnconstrainedBox - Remove constraints
        UnconstrainedBox(
          child: Container(
            width: 300,
            height: 50,
            color: Colors.purple,
            child: const Center(child: Text('Unconstrained')),
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- LayoutBuilder reveals constraints
- FractionallySizedBox sizes by fraction
- AspectRatio maintains aspect ratio
- ConstrainedBox custom constraints
- UnconstrainedBox removes constraints

---

# Layout Performance

> **Optimizing layout** for performance.

```dart
// Layout performance considerations
class LayoutPerformanceExample extends StatelessWidget {
  const LayoutPerformanceExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Use const widgets
        const Text('Constant text'), // No rebuild
        
        // 2. Use repaint boundaries
        RepaintBoundary(
          child: AnimatedContainer(
            duration: const Duration(seconds: 1),
            width: 100,
            height: 100,
            color: Colors.blue,
          ),
        ),
        
        // 3. Avoid deep widget trees
        // Prefer shallow trees for performance
        
        // 4. Use ListView.builder for long lists
        // Not all children built at once
        
        // 5. Use keys for stable identities
      ],
    );
  }
}

// Layout performance tips:
// 1. Minimize layout rebuilds
// 2. Use const widgets when possible
// 3. Avoid unnecessary parent rebuilds
// 4. Use RepaintBoundary for animations
// 5. Keep widget trees shallow
// 6. Use efficient layouts (Row/Column over Stack)
// 7. Avoid complex layout calculations
```

What's happening here?
- const widgets reduce rebuilds
- RepaintBoundary isolates repaint
- Shallow trees are faster
- ListView.builder is efficient
- Keys improve performance

---

# Debugging Layout

> **Debugging layout issues** in Flutter.

```dart
// Layout debugging tools
class LayoutDebuggingExample extends StatelessWidget {
  const LayoutDebuggingExample({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Enable debug painting
    debugPaintSizeEnabled = true;
    
    // 2. Use LayoutBuilder for debugging
    return LayoutBuilder(
      builder: (context, constraints) {
        print('Constraints: $constraints');
        print('Max Width: ${constraints.maxWidth}');
        print('Max Height: ${constraints.maxHeight}');
        
        return Container(
          constraints: BoxConstraints(
            maxWidth: constraints.maxWidth,
            maxHeight: constraints.maxHeight,
          ),
          color: Colors.blue,
          child: const Text('Debug Layout'),
        );
      },
    );
  }
}

// Common layout errors:
// 1. "BoxConstraints forces an infinite width"
// 2. "RenderFlex children have non-zero flex"
// 3. "RenderViewport does not support returning intrinsic dimensions"
// 4. "Vertical viewport was given unbounded height"
// 5. "RenderBox was not laid out"

// Debugging tips:
// 1. Check constraints with LayoutBuilder
// 2. Use debugPaintSizeEnabled
// 3. Check for overflow errors
// 4. Use Flutter Inspector
// 5. Check widget hierarchy
// 6. Look for infinite constraints
```

What's happening here?
- debugPaintSizeEnabled shows layout bounds
- LayoutBuilder reveals constraints
- Print constraints for debugging
- Common errors have specific causes
- Inspector helps visualize layout

---

# Best Practices

## Understand Constraint Flow

```dart
// Good - Understanding constraints
@override
Widget build(BuildContext context) {
  return Container(
    child: Center(
      child: Container(
        child: const Text('Hello'),
      ),
    ),
  );
}
// Constraints flow: Container → Center → Container → Text
// Sizes flow up: Text → Container → Center → Container
```

## Use Proper Layout Widgets

```dart
// Good - Using appropriate widgets
@override
Widget build(BuildContext context) {
  return Padding(
    padding: const EdgeInsets.all(8),
    child: Row(
      children: [
        Expanded(child: Child1()),
        Expanded(child: Child2()),
      ],
    ),
  );
}

// Bad - Using wrong widgets
@override
Widget build(BuildContext context) {
  return Container(
    padding: const EdgeInsets.all(8),
    child: Row(
      children: [
        Container(width: 100, child: Child1()),
        Container(width: 100, child: Child2()),
      ],
    ),
  );
}
```

## Handle Overflow

```dart
// Good - Preventing overflow
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      Expanded(
        child: Text('Long text that might overflow'),
      ),
      const Icon(Icons.star),
    ],
  );
}

// Bad - Potential overflow
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      Text('Long text that might overflow'),
      const Icon(Icons.star),
    ],
  );
}
```

---

# Common Mistakes

## Infinite Constraints

Wrong:
```dart
// Error: Unbounded height
Column(
  children: [
    Expanded(
      child: Container(
        // Column with unbounded height
        child: Row(children: []),
      ),
    ),
  ],
)
```

Correct:
```dart
// Use ConstrainedBox or limit height
Column(
  children: [
    Expanded(
      child: Container(
        height: 100, // Gives height constraint
        child: Row(children: []),
      ),
    ),
  ],
)
```

## Not Checking Constraints

Wrong:
```dart
// Assuming constraints
@override
Widget build(BuildContext context) {
  return Container(
    width: 300, // May not fit
    height: 200,
  );
}
```

Correct:
```dart
// Check constraints
@override
Widget build(BuildContext context) {
  return LayoutBuilder(
    builder: (context, constraints) {
      return Container(
        width: min(300, constraints.maxWidth),
        height: min(200, constraints.maxHeight),
      );
    },
  );
}
```

---

# Summary

The layout process determines widget sizes and positions through constraints flowing down and sizes flowing up. Understanding constraints, layout phases, and common patterns helps build efficient, responsive layouts. Use layout helpers, debug tools, and best practices for maintainable code.

---

# Next Steps

- [Constraints](constraints.md)
- [Box Model](box-model.md)
- [Flex Layout](flex-layout.md)

---

# Did You Know?

- Layout happens every frame
- Constraints flow down, sizes flow up
- LayoutBuilder reveals constraints
- Unbounded constraints cause errors
- performLayout is the main layout method
- Children are laid out recursively
- Layout can be optimized with const widgets
- The layout tree mirrors the widget tree