# RenderObject

Understand the low-level rendering system in Flutter through RenderObject.

---

# What is it?

RenderObject is the fundamental class in Flutter's rendering pipeline that handles layout, painting, and hit testing. It represents a node in the render tree and is responsible for determining its size, position, and how to paint itself on screen. RenderObject is the lowest-level building block for custom rendering in Flutter.

---

# Why does it exist?

RenderObject exists to:

- Handle layout and painting
- Manage the render tree
- Perform hit testing
- Control rendering performance
- Enable custom rendering logic
- Provide low-level rendering control
- Handle compositing and layering

---

# Basic RenderObject

> **Understanding** RenderObject structure.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

/// Basic RenderObject example
class BasicRenderObjectExample extends StatelessWidget {
  const BasicRenderObjectExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('RenderObject'),
      ),
      body: Center(
        child: CustomRenderObjectWidget(),
      ),
    );
  }
}

/// 1. Widget that creates the RenderObject
/// This widget is the bridge between the widget tree and the render tree
class CustomRenderObjectWidget extends LeafRenderObjectWidget {
  const CustomRenderObjectWidget({super.key});

  @override
  RenderObject createRenderObject(BuildContext context) {
    // This method creates the RenderObject
    // Called when the widget is first inserted into the tree
    return CustomRenderObject();
  }

  @override
  void updateRenderObject(
    BuildContext context,
    CustomRenderObject renderObject,
  ) {
    // Called when the widget is updated
    // Update properties of the RenderObject here
  }
}

/// 2. Custom RenderObject
/// This is the actual rendering object that handles layout and painting
class CustomRenderObject extends RenderBox {
  // 1. Override performLayout to control sizing
  // This is called when the parent asks this render object to size itself
  @override
  void performLayout() {
    // Set the size of this render object
    // constraints is the BoxConstraints from the parent
    size = constraints.constrain(const Size(200, 200));
  }

  // 2. Override paint to control drawing
  // This is called when this render object needs to be painted
  @override
  void paint(PaintingContext context, Offset offset) {
    // Get the canvas from the painting context
    final canvas = context.canvas;

    // Create a paint object for styling
    final paint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    // Draw a rectangle at the offset position
    canvas.drawRect(
      offset & size, // Combine offset and size
      paint,
    );

    // Draw a border
    final borderPaint = Paint()
      ..color = Colors.black
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2;

    canvas.drawRect(
      offset & size,
      borderPaint,
    );

    // Draw text in the center
    final textPainter = TextPainter(
      text: const TextSpan(
        text: 'Custom RenderObject',
        style: TextStyle(
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
        offset.dx + (size.width - textPainter.width) / 2,
        offset.dy + (size.height - textPainter.height) / 2,
      ),
    );
  }

  // 3. Override hitTest to handle touch events
  // This determines if a touch point is within this render object
  @override
  bool hitTest(HitTestResult result, {required Offset position}) {
    // Check if the position is within the bounds
    if (size.contains(position)) {
      // Add this render object to the hit test result
      result.add(BoxHitTestEntry(this, position));
      return true;
    }
    return false;
  }

  // 4. Override handleEvent to respond to events
  // This handles events that hit this render object
  @override
  void handleEvent(PointerEvent event, BoxHitTestEntry entry) {
    if (event is PointerDownEvent) {
      // Handle tap
      print('Custom RenderObject tapped at: ${event.position}');
    }
  }
}
```

What's happening here?
- LeafRenderObjectWidget creates the RenderObject
- RenderBox is the base for box-shaped render objects
- performLayout sets the size
- paint draws the content
- hitTest handles touch detection
- handleEvent responds to events

---

# RenderObject with Children

> **Managing children** in RenderObject.

```dart
/// RenderObject with children
class RenderObjectWithChildrenExample extends StatelessWidget {
  const RenderObjectWithChildrenExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('RenderObject with Children'),
      ),
      body: Center(
        child: CustomMultiChildRenderObjectWidget(
          children: [
            Container(
              width: 60,
              height: 60,
              color: Colors.red,
              child: const Center(child: Text('1')),
            ),
            Container(
              width: 60,
              height: 60,
              color: Colors.green,
              child: const Center(child: Text('2')),
            ),
            Container(
              width: 60,
              height: 60,
              color: Colors.blue,
              child: const Center(child: Text('3')),
            ),
          ],
        ),
      ),
    );
  }
}

/// 1. Multi-child widget
class CustomMultiChildRenderObjectWidget extends MultiChildRenderObjectWidget {
  const CustomMultiChildRenderObjectWidget({
    super.key,
    required super.children,
  });

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CustomMultiChildRenderObject();
  }
}

/// 2. RenderObject with children management
class CustomMultiChildRenderObject extends RenderBox
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
  // This is where we position each child
  @override
  void performLayout() {
    // Get constraints from parent
    final BoxConstraints constraints = this.constraints;
    
    // Calculate layout
    final double spacing = 8.0;
    final double totalWidth = constraints.maxWidth;
    final double childWidth = (totalWidth - spacing * 2) / 3;
    final double childHeight = constraints.maxHeight - spacing * 2;
    
    // Layout each child
    RenderBox? child = firstChild;
    int index = 0;
    while (child != null) {
      // Set constraints for the child
      child.layout(
        BoxConstraints.tight(Size(childWidth, childHeight)),
        parentUsesSize: true,
      );
      
      // Position the child
      final BoxParentData childParentData = 
          child.parentData! as BoxParentData;
      childParentData.offset = Offset(
        index * (childWidth + spacing) + spacing,
        spacing,
      );
      
      child = childAfter(child);
      index++;
    }
    
    // Set own size
    size = constraints.constrain(Size(
      totalWidth,
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
    // Use default hit test for children
    return defaultHitTestChildren(result, position: position);
  }
}
```

What's happening here?
- MultiChildRenderObjectWidget manages multiple children
- ContainerRenderObjectMixin provides child management
- performLayout positions each child
- paint draws each child
- hitTestChildren handles child hit testing

---

# Custom RenderObject with Animation

> **Animated** RenderObject.

```dart
/// Animated RenderObject example
class AnimatedRenderObjectExample extends StatefulWidget {
  const AnimatedRenderObjectExample({super.key});

  @override
  State<AnimatedRenderObjectExample> createState() => _AnimatedRenderObjectExampleState();
}

class _AnimatedRenderObjectExampleState extends State<AnimatedRenderObjectExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    )..repeat(reverse: true);
    
    _animation = Tween<double>(begin: 0.0, end: 1.0).animate(_controller)
      ..addListener(() {
        // Trigger repaint when animation changes
        setState(() {});
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
        title: const Text('Animated RenderObject'),
      ),
      body: Center(
        child: AnimatedRenderObjectWidget(
          progress: _animation.value,
        ),
      ),
    );
  }
}

/// Animated widget
class AnimatedRenderObjectWidget extends LeafRenderObjectWidget {
  const AnimatedRenderObjectWidget({
    super.key,
    required this.progress,
  });

  final double progress;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return AnimatedRenderObject(progress: progress);
  }

  @override
  void updateRenderObject(
    BuildContext context,
    AnimatedRenderObject renderObject,
  ) {
    // Update the render object when progress changes
    renderObject.progress = progress;
  }
}

/// Animated RenderObject
class AnimatedRenderObject extends RenderBox {
  AnimatedRenderObject({required double progress}) : _progress = progress;

  double _progress;
  set progress(double value) {
    if (_progress == value) return;
    _progress = value;
    // Mark as needing paint to trigger redraw
    markNeedsPaint();
  }

  @override
  void performLayout() {
    // Set size based on constraints
    size = constraints.constrain(const Size(200, 200));
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    final canvas = context.canvas;
    final center = Offset(size.width / 2, size.height / 2) + offset;
    final radius = size.width / 2 - 20;

    // 1. Background circle
    final bgPaint = Paint()
      ..color = Colors.grey[300]!
      ..style = PaintingStyle.stroke
      ..strokeWidth = 10;

    canvas.drawCircle(center, radius, bgPaint);

    // 2. Progress arc
    final progressPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.stroke
      ..strokeWidth = 10
      ..strokeCap = StrokeCap.round;

    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -3.14159 / 2,
      2 * 3.14159 * _progress,
      false,
      progressPaint,
    );

    // 3. Animated circle on the arc
    final circlePaint = Paint()
      ..color = Colors.red
      ..style = PaintingStyle.fill;

    final angle = _progress * 2 * 3.14159 - 3.14159 / 2;
    final circleX = center.dx + radius * cos(angle);
    final circleY = center.dy + radius * sin(angle);

    canvas.drawCircle(
      Offset(circleX, circleY),
      12,
      circlePaint,
    );

    // 4. Percentage text
    final textPainter = TextPainter(
      text: TextSpan(
        text: '${(_progress * 100).round()}%',
        style: TextStyle(
          color: Colors.blue,
          fontSize: 24 + _progress * 10,
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

  @override
  bool hitTest(HitTestResult result, {required Offset position}) {
    // Simple hit test - check if within bounds
    if (size.contains(position)) {
      result.add(BoxHitTestEntry(this, position));
      return true;
    }
    return false;
  }
}
```

What's happening here?
- Animation drives progress value
- updateRenderObject updates RenderObject
- markNeedsPaint triggers redraw
- Animated arc and circle
- Dynamic text size

---

# Real-World Examples

> **Common patterns** with RenderObject.

```dart
/// 1. Custom progress bar RenderObject
class CustomProgressBar extends LeafRenderObjectWidget {
  const CustomProgressBar({
    super.key,
    required this.progress,
    this.color = Colors.blue,
    this.height = 20,
  });

  final double progress;
  final Color color;
  final double height;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return ProgressBarRenderObject(
      progress: progress.clamp(0.0, 1.0),
      color: color,
      height: height,
    );
  }

  @override
  void updateRenderObject(
    BuildContext context,
    ProgressBarRenderObject renderObject,
  ) {
    renderObject
      ..progress = progress.clamp(0.0, 1.0)
      ..color = color
      ..height = height;
  }
}

class ProgressBarRenderObject extends RenderBox {
  ProgressBarRenderObject({
    required double progress,
    required Color color,
    required double height,
  }) : _progress = progress,
       _color = color,
       _height = height;

  double _progress;
  set progress(double value) {
    if (_progress == value) return;
    _progress = value;
    markNeedsPaint();
  }

  Color _color;
  set color(Color value) {
    if (_color == value) return;
    _color = value;
    markNeedsPaint();
  }

  double _height;
  set height(double value) {
    if (_height == value) return;
    _height = value;
    markNeedsLayout();
  }

  @override
  void performLayout() {
    // Set height from property, width from constraints
    size = constraints.constrain(Size(
      constraints.maxWidth,
      _height,
    ));
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    final canvas = context.canvas;
    
    // Background
    final bgPaint = Paint()
      ..color = Colors.grey[200]!
      ..style = PaintingStyle.fill;

    canvas.drawRRect(
      RRect.fromRectAndRadius(
        offset & size,
        const Radius.circular(10),
      ),
      bgPaint,
    );

    // Progress
    final progressPaint = Paint()
      ..shader = LinearGradient(
        colors: [_color, _color.withOpacity(0.7)],
      ).createShader(offset & size)
      ..style = PaintingStyle.fill;

    final progressWidth = size.width * _progress;
    if (progressWidth > 0) {
      canvas.drawRRect(
        RRect.fromRectAndRadius(
          Rect.fromLTWH(offset.dx, offset.dy, progressWidth, size.height),
          const Radius.circular(10),
        ),
        progressPaint,
      );
    }

    // Text
    final text = '${(_progress * 100).round()}%';
    final textPainter = TextPainter(
      text: TextSpan(
        text: text,
        style: TextStyle(
          color: _progress > 0.5 ? Colors.white : Colors.black,
          fontSize: 12,
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
}
```

What's happening here?
- Custom progress bar RenderObject
- markNeedsPaint for redraw
- markNeedsLayout for size changes
- Efficient rendering

---

# Best Practices

## Use markNeedsPaint Properly

```dart
// Good - Trigger repaint when needed
set progress(double value) {
  if (_progress == value) return;
  _progress = value;
  markNeedsPaint(); // Only repaint
}

// Bad - Trigger layout when only paint needed
set progress(double value) {
  if (_progress == value) return;
  _progress = value;
  markNeedsLayout(); // Unnecessary layout
}
```

## Use markNeedsLayout for Size Changes

```dart
// Good - Trigger layout for size changes
set height(double value) {
  if (_height == value) return;
  _height = value;
  markNeedsLayout(); // Need to relayout
}

// Bad - Only paint for size changes
set height(double value) {
  if (_height == value) return;
  _height = value;
  markNeedsPaint(); // Size changed but not relaid out
}
```

## Efficient Painting

```dart
// Good - Efficient painting
@override
void paint(PaintingContext context, Offset offset) {
  // Use cached values
  final paint = _cachedPaint;
  // Draw efficiently
}

// Bad - Creating objects in paint
@override
void paint(PaintingContext context, Offset offset) {
  final paint = Paint(); // Created every frame
}
```

---

# Common Mistakes

## Not Calling super.performLayout

Wrong:
```dart
@override
void performLayout() {
  // Missing super.performLayout()
  size = constraints.constrain(...);
}
```

Correct:
```dart
@override
void performLayout() {
  // Call super first
  super.performLayout();
  size = constraints.constrain(...);
}
```

## Not Marking Needs Layout

Wrong:
```dart
set size(double value) {
  _size = value;
  // Missing markNeedsLayout()
}
```

Correct:
```dart
set size(double value) {
  if (_size == value) return;
  _size = value;
  markNeedsLayout(); // Need to relayout
}
```

---

# Summary

RenderObject provides low-level rendering control in Flutter. Use RenderBox for box-shaped render objects, handle children with ContainerRenderObjectMixin, and manage performance with markNeedsPaint and markNeedsLayout. RenderObject is essential for custom rendering.

---

# Next Steps

- [RenderBox](renderbox.md)
- [Layers](layers.md)
- [Compositing](compositing.md)

---

# Did You Know?

- RenderObject is the lowest rendering primitive
- performLayout controls sizing
- paint controls drawing
- hitTest handles touch detection
- markNeedsPaint triggers redraw
- markNeedsLayout triggers relayout
- RenderBox is the most common RenderObject
- RenderObject enables custom rendering