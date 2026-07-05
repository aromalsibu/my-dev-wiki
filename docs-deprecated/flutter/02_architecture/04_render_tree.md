# Render Tree

Understand the render tree and how Flutter paints UI elements on screen.

---

# What is it?

The render tree is a tree of RenderObject objects that handle the actual layout, painting, and hit testing of your Flutter application. While the widget tree defines what to display and the element tree manages state, the render tree is responsible for how to display it on screen.

---

# Why does it exist?

The render tree exists to:

- Handle layout calculations and constraints
- Perform actual painting to the canvas
- Manage hit testing for user interactions
- Optimize rendering performance
- Handle compositing and layers
- Manage repaint boundaries
- Coordinate with the rendering pipeline

---

# Understanding the Render Tree

> **The render tree** mirrors the widget tree but with render objects.

```dart
// Widget Tree (Configuration)
Container
  └── Center
        └── Text('Hello')

// Element Tree (State)
ContainerElement
  └── CenterElement
        └── TextElement

// Render Tree (Painting)
RenderConstrainedBox
  └── RenderPositionedBox
        └── RenderParagraph('Hello')

// Relationship:
// Widget → creates → Element → creates → RenderObject
// Each widget type creates a specific render object
// Render objects handle layout and painting
```

What's happening here?
- Render tree handles actual rendering
- Each render object knows its size and position
- Render objects paint to the canvas
- Tree mirrors the widget structure
- Render objects are mutable

---

# Types of Render Objects

> **Different render objects** handle different layout and painting needs.

```dart
// 1. RenderBox - Most common render object
class CustomRenderBox extends RenderBox {
  @override
  void performLayout() {
    // Handle layout
    size = constraints.constrain(Size(100, 100));
  }
  
  @override
  void paint(PaintingContext context, Offset offset) {
    // Handle painting
    context.canvas.drawRect(
      offset & size,
      Paint()..color = Colors.blue,
    );
  }
}

// 2. RenderParagraph - Renders text
// Used by Text widget
RenderParagraph(
  text: TextSpan(text: 'Hello'),
  textDirection: TextDirection.ltr,
)

// 3. RenderImage - Renders images
// Used by Image widget
RenderImage(
  image: imageProvider,
  width: 100,
  height: 100,
)

// 4. RenderFlex - Handles flex layouts
// Used by Row and Column
RenderFlex(
  direction: Axis.vertical,
  mainAxisAlignment: MainAxisAlignment.center,
  children: renderChildren,
)

// 5. RenderPositionedBox - Centers child
// Used by Center widget
RenderPositionedBox(
  alignment: Alignment.center,
  child: renderChild,
)

// 6. RenderStack - Overlays children
// Used by Stack widget
RenderStack(
  alignment: Alignment.topLeft,
  children: renderChildren,
)

// 7. RenderConstrainedBox - Applies constraints
// Used by Container with constraints
RenderConstrainedBox(
  additionalConstraints: BoxConstraints.tight(Size(100, 100)),
  child: renderChild,
)
```

What's happening here?
- RenderBox is the base for most render objects
- Different render objects handle different widgets
- Each render object has specific properties
- Render objects form a tree structure
- Each handles its own layout and painting

---

# Render Tree Building

> **Render objects are created** from elements during the build process.

```dart
// Building the render tree
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Widget tree is created
    return Container(
      color: Colors.blue,
      child: const Center(
        child: Text('Hello'),
      ),
    );
  }
}

// Behind the scenes:
// 1. Container widget creates RenderConstrainedBox
// 2. Center widget creates RenderPositionedBox
// 3. Text widget creates RenderParagraph

// Render object creation process
class MyCustomWidget extends LeafRenderObjectWidget {
  const MyCustomWidget({super.key});

  @override
  RenderObject createRenderObject(BuildContext context) {
    // Called when element is mounted
    // Creates the render object
    return CustomRenderBox();
  }

  @override
  void updateRenderObject(
    BuildContext context,
    CustomRenderBox renderObject,
  ) {
    // Called when widget updates
    // Updates render object properties
  }
}

// Render tree after build:
RenderConstrainedBox
  └── RenderPositionedBox
        └── RenderParagraph('Hello')
```

What's happening here?
- Render objects are created during build
- Each widget creates a specific render object
- Render objects form a tree mirroring widgets
- Render objects are updated when widgets change
- The render tree is separate from widget tree

---

# Layout Process

> **Render objects perform layout** using constraints passed down the tree.

```dart
// Layout process flow
class CustomLayoutWidget extends RenderObjectWidget {
  const CustomLayoutWidget({super.key});

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CustomRenderObject();
  }
}

class CustomRenderObject extends RenderBox {
  RenderBox? _child;

  @override
  void setupParentData(RenderObject child) {
    if (child.parentData is! BoxParentData) {
      child.parentData = BoxParentData();
    }
  }

  @override
  void performLayout() {
    // 1. Get constraints from parent
    final BoxConstraints constraints = this.constraints;
    
    // 2. Calculate available space
    final double maxWidth = constraints.maxWidth;
    final double maxHeight = constraints.maxHeight;
    
    // 3. Layout child if exists
    if (_child != null) {
      // 4. Pass constraints to child
      _child!.layout(
        constraints,
        parentUsesSize: true,
      );
      
      // 5. Position child
      final BoxParentData childParentData = 
          _child!.parentData! as BoxParentData;
      childParentData.offset = Offset.zero;
      
      // 6. Set own size based on child
      size = _child!.size;
    } else {
      // 7. Set size if no child
      size = constraints.constrain(Size(100, 100));
    }
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    // Paint child
    if (_child != null) {
      context.paintChild(_child!, offset);
    }
  }
}

// Layout constraints flow:
// 1. Parent passes constraints to child
// 2. Child computes its size
// 3. Child returns its size to parent
// 4. Parent positions child
// 5. Parent sets its own size
```

What's happening here?
- Layout is top-down (constraints down)
- Size is bottom-up (sizes up)
- Parent controls child's constraints
- Child chooses its own size
- Parent positions the child
- Both layout and painting happen

---

# Painting Process

> **Render objects paint** themselves and their children.

```dart
// Painting process
class CustomPainterWidget extends LeafRenderObjectWidget {
  const CustomPainterWidget({super.key, required this.painter});

  final CustomPainter painter;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return _RenderCustomPainter(painter: painter);
  }

  @override
  void updateRenderObject(
    BuildContext context,
    _RenderCustomPainter renderObject,
  ) {
    renderObject.painter = painter;
  }
}

class _RenderCustomPainter extends RenderBox {
  _RenderCustomPainter({required CustomPainter painter})
    : _painter = painter;

  CustomPainter _painter;

  set painter(CustomPainter value) {
    if (_painter == value) return;
    _painter = value;
    markNeedsPaint(); // Trigger repaint
  }

  @override
  void performLayout() {
    // Set size based on constraints
    size = constraints.constrain(Size(200, 200));
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    // 1. Create canvas
    final Canvas canvas = context.canvas;
    
    // 2. Save canvas state
    canvas.save();
    
    // 3. Translate to offset
    canvas.translate(offset.dx, offset.dy);
    
    // 4. Paint custom content
    _painter.paint(canvas, size);
    
    // 5. Restore canvas state
    canvas.restore();
  }

  @override
  void dispose() {
    _painter.dispose();
    super.dispose();
  }
}

// Painting layers:
// 1. Canvas - drawing surface
// 2. Paint - style and color
// 3. Path - shape to draw
// 4. Offset - position on canvas
```

What's happening here?
- paint() is called every frame
- Canvas provides drawing methods
- Save/restore handles transformations
- markNeedsPaint triggers repaint
- Painting is bottom-up (children first)

---

# Hit Testing

> **Render objects handle** user interactions through hit testing.

```dart
// Hit testing process
class InteractiveRenderObject extends RenderBox {
  bool _isHovering = false;
  bool _isPressed = false;

  @override
  bool hitTest(HitTestResult result, {required Offset position}) {
    // 1. Check if point is within bounds
    if (!size.contains(position)) {
      return false; // Miss
    }
    
    // 2. Add to hit test result
    result.add(BoxHitTestEntry(this, position));
    
    // 3. Return true for hit
    return true;
  }

  @override
  void handleEvent(PointerEvent event, BoxHitTestEntry entry) {
    // 4. Handle the event
    if (event is PointerDownEvent) {
      _isPressed = true;
      markNeedsPaint();
    }
    
    if (event is PointerUpEvent) {
      _isPressed = false;
      markNeedsPaint();
    }
    
    if (event is PointerHoverEvent) {
      _isHovering = true;
      markNeedsPaint();
    }
    
    if (event is PointerExitEvent) {
      _isHovering = false;
      markNeedsPaint();
    }
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    final Paint paint = Paint()
      ..color = _isPressed ? Colors.green : 
                _isHovering ? Colors.blue : Colors.grey;
    
    context.canvas.drawRect(
      offset & size,
      paint,
    );
  }
}

// Hit test flow:
// 1. Pointer event arrives
// 2. Hit test starts at root
// 3. Traverses tree top-down
// 4. Finds hit render objects
// 5. Event dispatched to objects
// 6. Objects handle the event
```

What's happening here?
- hitTest checks if point hits object
- Hit test results track hits
- handleEvent processes interactions
- Renders visual feedback
- Hit test order determines interaction

---

# Repaint Boundaries

> **Repaint boundaries** optimize painting performance.

```dart
// Repaint boundaries
class OptimizedWidget extends StatelessWidget {
  const OptimizedWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Repaint boundary for animation
        RepaintBoundary(
          child: AnimatedWidget(),
        ),
        
        // 2. Repaint boundary for video
        RepaintBoundary(
          child: VideoWidget(),
        ),
        
        // 3. Repaint boundary for static content
        const RepaintBoundary(
          child: StaticContent(),
        ),
      ],
    );
  }
}

// Custom repaint boundary
class CustomRepaintBoundary extends SingleChildRenderObjectWidget {
  const CustomRepaintBoundary({super.key, super.child});

  @override
  RenderObject createRenderObject(BuildContext context) {
    return RenderRepaintBoundary();
  }

  @override
  void updateRenderObject(
    BuildContext context,
    RenderRepaintBoundary renderObject,
  ) {
    // Configure boundary
  }
}

// When to use repaint boundaries:
// 1. Animated widgets
// 2. Video or image rendering
// 3. Complex custom paintings
// 4. Widgets that update frequently
// 5. Isolate expensive paints

// When NOT to use repaint boundaries:
// 1. Simple static widgets
// 2. Widgets that don't repaint
// 3. Very small widgets
// 4. Overuse can hurt performance
```

What's happening here?
- Repaint boundaries isolate repaint areas
- Only dirty areas are repainted
- Reduces painting overhead
- Improves performance
- Use for frequently updating widgets

---

# Compositing

> **Compositing combines** layers into the final image.

```dart
// Compositing process
class CompositeWidget extends StatelessWidget {
  const CompositeWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        // 1. Background layer
        Container(color: Colors.blue),
        
        // 2. Text layer
        const Text('Hello'),
        
        // 3. Image layer
        Image.network('https://example.com/image.jpg'),
        
        // 4. Overlay layer
        Container(
          color: Colors.black.withOpacity(0.5),
          child: const Text('Overlay'),
        ),
      ],
    );
  }
}

// Layer types:
// 1. PictureLayer - Contains draw calls
// 2. TextureLayer - Contains video/GPU content
// 3. OffsetLayer - Positioned child layer
// 4. OpacityLayer - Applied opacity
// 5. ClipRectLayer - Clips content

// Compositing flow:
// 1. Each render object paints to layers
// 2. Layers are collected
// 3. Layers are composited in order
// 4. Final image is produced
// 5. Display on screen
```

What's happening here?
- Each render object creates layers
- Layers are composited together
- Compositing creates final image
- Order determines layering
- GPU handles compositing

---

# Debugging Render Tree

> **Tools for inspecting** the render tree.

```dart
// Debug render tree
import 'package:flutter/rendering.dart';

class DebugRenderWidget extends StatelessWidget {
  const DebugRenderWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // Enable debug painting
    debugPaintSizeEnabled = true;
    
    // Print render tree
    debugDumpRenderTree();
    
    // Print layer tree
    debugDumpLayerTree();
    
    // Print paint tree
    debugDumpPaintTree();
    
    // Print semantics tree
    debugDumpSemanticsTree();
    
    return const Text('Debug');
  }
}

// Render tree debugging tips:
// 1. Check layout constraints
// 2. Verify sizes and positions
// 3. Inspect render object properties
// 4. View paint order
// 5. Check repaint boundaries
// 6. Monitor performance

// Flutter Inspector
// 1. Select widget
// 2. View render object
// 3. Check layout properties
// 4. View repaint boundaries
// 5. Highlight oversized widgets
```

What's happening here?
- debugPaintSizeEnabled shows layout bounds
- debugDumpRenderTree shows render hierarchy
- Flutter Inspector visualizes render tree
- Helps debug layout issues
- Helps performance debugging

---

# Best Practices

## Use Repaint Boundaries Wisely

```dart
// Good - Repaint boundary for animation
@override
Widget build(BuildContext context) {
  return RepaintBoundary(
    child: AnimatedBuilder(
      animation: controller,
      builder: (context, child) {
        return Transform.rotate(
          angle: controller.value,
          child: child,
        );
      },
      child: const Icon(Icons.refresh),
    ),
  );
}

// Bad - Unnecessary repaint boundary
@override
Widget build(BuildContext context) {
  return RepaintBoundary(
    child: const Text('Hello'), // Static text doesn't need it
  );
}
```

## Avoid Complex Layouts

```dart
// Good - Simple layout
@override
void performLayout() {
  size = constraints.constrain(Size(100, 100));
}

// Bad - Complex layout calculations
@override
void performLayout() {
  // Heavy computations in layout
  final double width = calculateWidth();
  final double height = calculateHeight();
  // Complex positioning
  // Many nested layouts
}
```

## Dispose Resources

```dart
// Good - Clean up resources
class CustomRenderObject extends RenderBox {
  final AnimationController controller;
  
  CustomRenderObject(this.controller) {
    controller.addListener(markNeedsPaint);
  }
  
  @override
  void dispose() {
    controller.removeListener(markNeedsPaint);
    super.dispose();
  }
}
```

---

# Common Mistakes

## Not Using Repaint Boundaries

Wrong:
```dart
// Animating widget with no repaint boundary
@override
Widget build(BuildContext context) {
  return AnimatedWidget(
    animation: controller,
    child: HeavyWidget(), // Rebuilds every frame
  );
}
```

Correct:
```dart
// Wrap with repaint boundary
@override
Widget build(BuildContext context) {
  return RepaintBoundary(
    child: AnimatedWidget(
      animation: controller,
      child: const HeavyWidget(),
    ),
  );
}
```

## Layout Constraints Too Tight

Wrong:
```dart
// Constraints too tight causing overflow
@override
void performLayout() {
  size = constraints.smallest; // Too small
}
```

Correct:
```dart
// Use constraints properly
@override
void performLayout() {
  size = constraints.constrain(Size(100, 100));
}
```

---

# Summary

The render tree handles layout, painting, and hit testing in Flutter. Render objects perform layout using constraints, paint themselves and their children, and handle user interactions. Understanding the render tree helps you build efficient, performant applications.

---

# Next Steps

- [Build Process](build-process.md)
- [Rendering Pipeline](rendering-pipeline.md)
- [Frame Scheduling](frame-scheduling.md)

---

# Did You Know?

- Render objects are mutable
- The render tree is separate from the widget tree
- Repaint boundaries optimize performance
- Layout is top-down, painting is bottom-up
- Render objects handle hit testing
- Compositing combines layers
- The render tree is rebuilt when needed