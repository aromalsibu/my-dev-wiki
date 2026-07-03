# RenderBox

Understand how to create box-shaped render objects in Flutter's rendering system.

---

# What is it?

RenderBox is a subclass of RenderObject that represents a box-shaped render object in the render tree. It handles the layout, painting, and hit testing for rectangular widgets. RenderBox is the most commonly used render object type and forms the foundation for most Flutter widgets.

---

# Why does it exist?

RenderBox exists to:

- Handle box-shaped rendering
- Manage layout constraints
- Control sizing and positioning
- Perform painting of rectangular areas
- Handle hit testing for box shapes
- Provide parent data management
- Enable custom box rendering

---

# Basic RenderBox

> **Creating a custom** RenderBox.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

/// Basic RenderBox example
class BasicRenderBoxExample extends StatelessWidget {
  const BasicRenderBoxExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('RenderBox'),
      ),
      body: Center(
        child: CustomBoxWidget(),
      ),
    );
  }
}

/// 1. Widget that creates the RenderBox
class CustomBoxWidget extends LeafRenderObjectWidget {
  const CustomBoxWidget({super.key});

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CustomRenderBox();
  }
}

/// 2. Custom RenderBox
class CustomRenderBox extends RenderBox {
  // 1. Layout - Determine size
  // This is called when the parent asks this render box to size itself
  @override
  void performLayout() {
    // constraints contains min/max width and height from parent
    // Set the size of this render box
    // constrain ensures the size respects the parent's constraints
    size = constraints.constrain(const Size(200, 200));
  }

  // 2. Painting - Draw the content
  // This is called when this render box needs to be painted
  @override
  void paint(PaintingContext context, Offset offset) {
    // Get the canvas for drawing
    final canvas = context.canvas;

    // Create paint objects
    final fillPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    final borderPaint = Paint()
      ..color = Colors.black
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;

    // Draw rectangle
    final rect = offset & size; // Combine offset and size
    canvas.drawRect(rect, fillPaint);
    canvas.drawRect(rect, borderPaint);

    // Draw text
    final textPainter = TextPainter(
      text: const TextSpan(
        text: 'RenderBox',
        style: TextStyle(
          color: Colors.white,
          fontSize: 20,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );

    textPainter.layout();
    textPainter.paint(
      canvas,
      Offset(
        offset.dx + (size.width - textPainter.width) / 2,
        offset.dy + (size.height - textPainter.height) / 2,
      ),
    );
  }

  // 3. Hit testing - Detect touches
  // This determines if a touch point is within this render box
  @override
  bool hitTest(HitTestResult result, {required Offset position}) {
    // Check if the position is within the box's bounds
    if (size.contains(position)) {
      // Add this render box to the hit test result
      result.add(BoxHitTestEntry(this, position));
      return true;
    }
    return false;
  }

  // 4. Event handling - Respond to touches
  @override
  void handleEvent(PointerEvent event, BoxHitTestEntry entry) {
    if (event is PointerDownEvent) {
      print('RenderBox tapped at: ${event.position}');
    }
  }
}
```

What's happening here?
- performLayout sets the size
- constraints define min/max sizes
- paint draws the content
- hitTest checks if point is inside
- handleEvent responds to touches

---

# RenderBox with Children

> **Managing child** RenderBoxes.

```dart
/// RenderBox with children
class RenderBoxWithChildrenExample extends StatelessWidget {
  const RenderBoxWithChildrenExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('RenderBox with Children'),
      ),
      body: Center(
        child: CustomRowWidget(
          children: [
            Container(color: Colors.red, width: 50, height: 50),
            Container(color: Colors.green, width: 50, height: 50),
            Container(color: Colors.blue, width: 50, height: 50),
          ],
        ),
      ),
    );
  }
}

/// 1. Multi-child widget
class CustomRowWidget extends MultiChildRenderObjectWidget {
  const CustomRowWidget({
    super.key,
    required super.children,
  });

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CustomRowRenderBox();
  }
}

/// 2. RenderBox with children management
class CustomRowRenderBox extends RenderBox
    with ContainerRenderObjectMixin<RenderBox, BoxParentData>,
         RenderBoxContainerDefaultsMixin<RenderBox, BoxParentData> {
  
  // 1. Setup parent data for children
  // Each child needs its own parent data to store position
  @override
  void setupParentData(RenderObject child) {
    if (child.parentData is! BoxParentData) {
      child.parentData = BoxParentData();
    }
  }

  // 2. Layout children
  @override
  void performLayout() {
    // Get constraints from parent
    final BoxConstraints constraints = this.constraints;
    
    // Calculate layout
    final double spacing = 8.0;
    final int childCount = childCount;
    
    if (childCount == 0) {
      size = constraints.constrain(const Size(0, 0));
      return;
    }
    
    // Calculate child size
    final double totalSpacing = spacing * (childCount - 1);
    final double childWidth = (constraints.maxWidth - totalSpacing) / childCount;
    final double childHeight = constraints.maxHeight;
    
    // Layout each child
    RenderBox? child = firstChild;
    int index = 0;
    while (child != null) {
      // Layout child with tight constraints
      child.layout(
        BoxConstraints.tight(Size(childWidth, childHeight)),
        parentUsesSize: true,
      );
      
      // Position child
      final BoxParentData childParentData = 
          child.parentData! as BoxParentData;
      childParentData.offset = Offset(
        index * (childWidth + spacing),
        0,
      );
      
      child = childAfter(child);
      index++;
    }
    
    // Set own size
    size = constraints.constrain(Size(
      constraints.maxWidth,
      constraints.maxHeight,
    ));
  }

  // 3. Paint children
  @override
  void paint(PaintingContext context, Offset offset) {
    // Paint each child at its position
    RenderBox? child = firstChild;
    while (child != null) {
      final BoxParentData childParentData = 
          child.parentData! as BoxParentData;
      context.paintChild(child, offset + childParentData.offset);
      child = childAfter(child);
    }
  }

  // 4. Hit testing for children
  @override
  bool hitTestChildren(BoxHitTestResult result, {required Offset position}) {
    return defaultHitTestChildren(result, position: position);
  }
}
```

What's happening here?
- ContainerRenderObjectMixin manages children
- BoxParentData stores child position
- performLayout positions children
- paint draws each child
- hitTestChildren handles child hit testing

---

# RenderBox with Constraints

> **Understanding** layout constraints.

```dart
/// RenderBox constraints example
class ConstraintsExample extends StatelessWidget {
  const ConstraintsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('RenderBox Constraints'),
      ),
      body: Center(
        child: ConstraintBoxWidget(),
      ),
    );
  }
}

class ConstraintBoxWidget extends LeafRenderObjectWidget {
  const ConstraintBoxWidget({super.key});

  @override
  RenderObject createRenderObject(BuildContext context) {
    return ConstraintRenderBox();
  }
}

class ConstraintRenderBox extends RenderBox {
  @override
  void performLayout() {
    // 1. Get constraints from parent
    final BoxConstraints constraints = this.constraints;
    
    // 2. Check constraint types
    final bool isTight = constraints.isTight;
    final bool isBounded = constraints.hasBoundedWidth && 
                           constraints.hasBoundedHeight;
    final bool isUnbounded = !constraints.hasBoundedWidth || 
                             !constraints.hasBoundedHeight;
    
    // 3. Calculate size based on constraints
    // Tight constraints: min == max
    // Loose constraints: min < max
    // Unbounded: max is infinite
    
    // Get the maximum available size
    final double maxWidth = constraints.maxWidth;
    final double maxHeight = constraints.maxHeight;
    
    // Calculate desired size
    final double desiredWidth = maxWidth.isFinite ? maxWidth * 0.8 : 200;
    final double desiredHeight = maxHeight.isFinite ? maxHeight * 0.8 : 100;
    
    // Constrain the desired size
    size = constraints.constrain(Size(desiredWidth, desiredHeight));
    
    // 4. Print constraint information
    print('Constraints: $constraints');
    print('Size: $size');
    print('Tight: $isTight, Bounded: $isBounded, Unbounded: $isUnbounded');
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    final canvas = context.canvas;
    
    // Paint background
    final bgPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;
    
    canvas.drawRect(offset & size, bgPaint);
    
    // Paint constraint info
    final textPainter = TextPainter(
      text: TextSpan(
        text: 'Width: ${size.width.toStringAsFixed(0)}\n'
              'Height: ${size.height.toStringAsFixed(0)}',
        style: const TextStyle(
          color: Colors.white,
          fontSize: 16,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    
    textPainter.layout();
    textPainter.paint(
      canvas,
      Offset(
        offset.dx + 10,
        offset.dy + 10,
      ),
    );
  }
}
```

What's happening here?
- constraints define min/max sizes
- isTight checks if min == max
- constrain ensures valid size
- Unbounded constraints have infinite max

---

# Real-World Examples

> **Common patterns** with RenderBox.

```dart
/// 1. Custom circular RenderBox
class CircularRenderBoxWidget extends LeafRenderObjectWidget {
  const CircularRenderBoxWidget({
    super.key,
    required this.radius,
    required this.color,
  });

  final double radius;
  final Color color;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CircularRenderBox(radius: radius, color: color);
  }

  @override
  void updateRenderObject(
    BuildContext context,
    CircularRenderBox renderObject,
  ) {
    renderObject
      ..radius = radius
      ..color = color;
  }
}

class CircularRenderBox extends RenderBox {
  CircularRenderBox({
    required double radius,
    required Color color,
  }) : _radius = radius,
       _color = color;

  double _radius;
  set radius(double value) {
    if (_radius == value) return;
    _radius = value;
    markNeedsLayout();
  }

  Color _color;
  set color(Color value) {
    if (_color == value) return;
    _color = value;
    markNeedsPaint();
  }

  @override
  void performLayout() {
    final double size = _radius * 2;
    this.size = constraints.constrain(Size(size, size));
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    final canvas = context.canvas;
    final center = offset + Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2;

    // Shadow
    final shadowPaint = Paint()
      ..color = Colors.black.withOpacity(0.2)
      ..maskFilter = const MaskFilter.blur(BlurStyle.normal, 10);

    canvas.drawCircle(center, radius, shadowPaint);

    // Circle
    final fillPaint = Paint()
      ..color = _color
      ..style = PaintingStyle.fill;

    canvas.drawCircle(center, radius, fillPaint);

    // Border
    final borderPaint = Paint()
      ..color = Colors.black
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;

    canvas.drawCircle(center, radius, borderPaint);

    // Text
    final textPainter = TextPainter(
      text: TextSpan(
        text: '${_radius.toStringAsFixed(0)}px',
        style: const TextStyle(
          color: Colors.white,
          fontSize: 16,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );

    textPainter.layout();
    textPainter.paint(
      canvas,
      Offset(
        center.dx - textPainter.width / 2,
        center.dy - textPainter.height / 2,
      ),
    );
  }
}

/// 2. Custom gradient box
class GradientBoxWidget extends LeafRenderObjectWidget {
  const GradientBoxWidget({
    super.key,
    required this.colors,
    this.borderRadius = 8,
  });

  final List<Color> colors;
  final double borderRadius;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return GradientBoxRenderObject(
      colors: colors,
      borderRadius: borderRadius,
    );
  }

  @override
  void updateRenderObject(
    BuildContext context,
    GradientBoxRenderObject renderObject,
  ) {
    renderObject
      ..colors = colors
      ..borderRadius = borderRadius;
  }
}

class GradientBoxRenderObject extends RenderBox {
  GradientBoxRenderObject({
    required List<Color> colors,
    required double borderRadius,
  }) : _colors = colors,
       _borderRadius = borderRadius;

  List<Color> _colors;
  set colors(List<Color> value) {
    if (_colors == value) return;
    _colors = value;
    markNeedsPaint();
  }

  double _borderRadius;
  set borderRadius(double value) {
    if (_borderRadius == value) return;
    _borderRadius = value;
    markNeedsLayout();
  }

  @override
  void performLayout() {
    size = constraints.constrain(const Size(200, 100));
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    final canvas = context.canvas;
    final rect = offset & size;

    // Create gradient
    final gradient = LinearGradient(
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
      colors: _colors,
    );

    final paint = Paint()
      ..shader = gradient.createShader(rect)
      ..style = PaintingStyle.fill;

    // Draw rounded rectangle
    final rrect = RRect.fromRectAndRadius(
      rect,
      Radius.circular(_borderRadius),
    );

    canvas.drawRRect(rrect, paint);

    // Border
    final borderPaint = Paint()
      ..color = Colors.black
      ..style = PaintingStyle.stroke
      ..strokeWidth = 1;

    canvas.drawRRect(rrect, borderPaint);
  }
}
```

What's happening here?
- Custom circular render box
- Custom gradient render box
- markNeedsLayout for size changes
- markNeedsPaint for appearance changes

---

# Best Practices

## Use Appropriate Layout Methods

```dart
// Good - Proper layout
@override
void performLayout() {
  size = constraints.constrain(Size(width, height));
}

// Bad - Ignoring constraints
@override
void performLayout() {
  size = Size(100, 100); // May violate constraints
}
```

## Use markNeedsLayout/Paint Correctly

```dart
// Good - Trigger correct updates
set radius(double value) {
  if (_radius == value) return;
  _radius = value;
  markNeedsLayout(); // Size changed
}

set color(Color value) {
  if (_color == value) return;
  _color = value;
  markNeedsPaint(); // Only appearance changed
}
```

## Handle Unbounded Constraints

```dart
// Good - Handle infinite constraints
@override
void performLayout() {
  final width = constraints.hasBoundedWidth 
      ? constraints.maxWidth 
      : 200; // Default width
  final height = constraints.hasBoundedHeight 
      ? constraints.maxHeight 
      : 100; // Default height
  size = constraints.constrain(Size(width, height));
}
```

---

# Common Mistakes

## Not Calling super.performLayout

Wrong:
```dart
@override
void performLayout() {
  // Missing super call
}
```

Correct:
```dart
@override
void performLayout() {
  super.performLayout();
  // Your layout logic
}
```

## Not Handling Constraints

Wrong:
```dart
@override
void performLayout() {
  size = Size(200, 200); // May violate constraints
}
```

Correct:
```dart
@override
void performLayout() {
  size = constraints.constrain(Size(200, 200));
}
```

---

# Summary

RenderBox provides box-shaped rendering with layout, painting, and hit testing. Use performLayout for sizing, paint for drawing, and handle constraints properly. ContainerRenderObjectMixin helps manage children. RenderBox is the foundation for most Flutter widgets.

---

# Next Steps

- [Layers](layers.md)
- [Compositing](compositing.md)
- [RenderObject](renderobject.md)

---

# Did You Know?

- RenderBox is the most common RenderObject
- constraints define min/max sizes
- constrain ensures valid size
- BoxParentData stores child position
- markNeedsLayout triggers relayout
- markNeedsPaint triggers redraw
- ContainerRenderObjectMixin manages children
- RenderBox enables custom rendering