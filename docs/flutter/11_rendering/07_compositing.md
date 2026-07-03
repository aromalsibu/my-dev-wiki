# Compositing

Understand how Flutter combines layers to create the final image displayed on screen.

---

# What is it?

Compositing is the process of combining multiple visual layers into a single final image. In Flutter, compositing takes all the layers generated during the rendering pipeline and combines them in the correct order, applying effects like opacity, transformations, and blending modes to create the final output that appears on screen.

---

# Why does it exist?

Compositing exists to:

- Combine multiple visual layers
- Apply visual effects and transformations
- Handle z-ordering and layering
- Optimize rendering performance
- Enable complex visual effects
- Support animations and transitions
- Create the final screen image

---

# Basic Compositing

> **Understanding** how compositing works.

```dart
// Import required packages
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

/// Basic compositing example
class BasicCompositingExample extends StatelessWidget {
  const BasicCompositingExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Compositing'),
      ),
      body: Center(
        child: Container(
          width: 300,
          height: 300,
          color: Colors.grey[200],
          child: Stack(
            children: [
              // 1. Layer 1: Background
              Container(
                color: Colors.blue,
                width: double.infinity,
                height: double.infinity,
              ),
              
              // 2. Layer 2: Semi-transparent overlay
              Positioned(
                left: 50,
                top: 50,
                child: Container(
                  width: 100,
                  height: 100,
                  color: Colors.red.withOpacity(0.7),
                ),
              ),
              
              // 3. Layer 3: Transformed overlay
              Positioned(
                left: 120,
                top: 120,
                child: Transform.rotate(
                  angle: 0.3,
                  child: Container(
                    width: 80,
                    height: 80,
                    color: Colors.green,
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Layers are composed in order
- Later layers appear on top
- Opacity and transformations are applied
- The final image combines all layers

---

# Compositing with Paint

> **Using Paint** for compositing effects.

```dart
/// Compositing with Paint
class CompositingPaintExample extends StatelessWidget {
  const CompositingPaintExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Compositing with Paint'),
      ),
      body: Center(
        child: CustomPaint(
          size: const Size(300, 300),
          painter: CompositingPainter(),
        ),
      ),
    );
  }
}

class CompositingPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 1. Save the canvas state
    canvas.save();

    // 2. Draw background
    final bgPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    canvas.drawRect(
      Rect.fromLTWH(0, 0, size.width, size.height),
      bgPaint,
    );

    // 3. Draw a circle with opacity
    final circlePaint = Paint()
      ..color = Colors.red.withOpacity(0.5)
      ..style = PaintingStyle.fill;

    canvas.drawCircle(
      const Offset(100, 100),
      80,
      circlePaint,
    );

    // 4. Draw another circle with blend mode
    final blendPaint = Paint()
      ..color = Colors.green.withOpacity(0.5)
      ..style = PaintingStyle.fill
      ..blendMode = BlendMode.multiply; // Blending effect

    canvas.drawCircle(
      const Offset(200, 100),
      80,
      blendPaint,
    );

    // 5. Draw a rectangle with shadow
    final shadowPaint = Paint()
      ..color = Colors.black.withOpacity(0.2)
      ..maskFilter = const MaskFilter.blur(BlurStyle.normal, 10);

    canvas.drawRect(
      const Rect.fromLTWH(50, 180, 200, 80),
      shadowPaint,
    );

    final rectPaint = Paint()
      ..color = Colors.orange
      ..style = PaintingStyle.fill;

    canvas.drawRect(
      const Rect.fromLTWH(50, 170, 200, 80),
      rectPaint,
    );

    // 6. Restore canvas state
    canvas.restore();
  }

  @override
  bool shouldRepaint(CompositingPainter oldDelegate) => false;
}
```

What's happening here?
- Blend modes combine colors
- Opacity creates transparency
- Shadows create depth
- Each drawing operation is a layer

---

# Compositing Modes

> **Different blend** modes in Flutter.

```dart
/// Blend modes example
class BlendModesExample extends StatelessWidget {
  const BlendModesExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Blend Modes'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            _buildBlendModeDemo('Normal', BlendMode.srcOver),
            _buildBlendModeDemo('Multiply', BlendMode.multiply),
            _buildBlendModeDemo('Screen', BlendMode.screen),
            _buildBlendModeDemo('Overlay', BlendMode.overlay),
            _buildBlendModeDemo('Darken', BlendMode.darken),
            _buildBlendModeDemo('Lighten', BlendMode.lighten),
          ],
        ),
      ),
    );
  }

  Widget _buildBlendModeDemo(String label, BlendMode blendMode) {
    return Card(
      margin: const EdgeInsets.symmetric(vertical: 8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text(label, style: const TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Container(
              width: 200,
              height: 100,
              color: Colors.grey[200],
              child: CustomPaint(
                painter: BlendModePainter(blendMode: blendMode),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class BlendModePainter extends CustomPainter {
  BlendModePainter({required this.blendMode});

  final BlendMode blendMode;

  @override
  void paint(Canvas canvas, Size size) {
    // 1. Draw base rectangle
    final basePaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    canvas.drawRect(
      Rect.fromLTWH(10, 10, 80, 80),
      basePaint,
    );

    // 2. Draw overlay with blend mode
    final blendPaint = Paint()
      ..color = Colors.red
      ..style = PaintingStyle.fill
      ..blendMode = blendMode;

    canvas.drawRect(
      Rect.fromLTWH(40, 40, 80, 80),
      blendPaint,
    );
  }

  @override
  bool shouldRepaint(BlendModePainter oldDelegate) => false;
}
```

What's happening here?
- srcOver: Default compositing
- multiply: Darkens both colors
- screen: Lightens both colors
- overlay: Combines multiply and screen
- darken: Uses darker color
- lighten: Uses lighter color

---

# Compositing with Opacity

> **Controlling transparency** in compositing.

```dart
/// Opacity compositing example
class OpacityCompositingExample extends StatefulWidget {
  const OpacityCompositingExample({super.key});

  @override
  State<OpacityCompositingExample> createState() => _OpacityCompositingExampleState();
}

class _OpacityCompositingExampleState extends State<OpacityCompositingExample> {
  double _opacity = 0.5;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Opacity Compositing'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Compositing with opacity
            Container(
              width: 200,
              height: 200,
              color: Colors.grey[200],
              child: Stack(
                children: [
                  // Background
                  Container(
                    color: Colors.blue,
                    width: double.infinity,
                    height: double.infinity,
                  ),
                  // Opacity layer
                  Positioned(
                    left: 50,
                    top: 50,
                    child: Opacity(
                      opacity: _opacity,
                      child: Container(
                        width: 100,
                        height: 100,
                        color: Colors.red,
                      ),
                    ),
                  ),
                ],
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 2. Opacity control
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  Text(
                    'Opacity: ${(_opacity * 100).round()}%',
                    style: const TextStyle(fontWeight: FontWeight.bold),
                  ),
                  Slider(
                    value: _opacity,
                    min: 0.0,
                    max: 1.0,
                    onChanged: (value) {
                      setState(() {
                        _opacity = value;
                      });
                    },
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Opacity controls transparency
- Values range from 0.0 to 1.0
- Real-time opacity adjustment
- Compositing with opacity layer

---

# Real-World Examples

> **Common patterns** with compositing.

```dart
/// 1. Custom compositing widget
class CompositingWidget extends LeafRenderObjectWidget {
  const CompositingWidget({
    super.key,
    required this.opacity,
    required this.child,
  });

  final double opacity;
  final Widget child;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CompositingRenderObject(opacity: opacity);
  }

  @override
  void updateRenderObject(
    BuildContext context,
    CompositingRenderObject renderObject,
  ) {
    renderObject.opacity = opacity;
  }
}

class CompositingRenderObject extends RenderProxyBox {
  CompositingRenderObject({required double opacity})
      : _opacity = opacity;

  double _opacity;
  set opacity(double value) {
    if (_opacity == value) return;
    _opacity = value;
    markNeedsPaint();
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    if (_opacity < 1.0) {
      // Create opacity layer
      context.pushOpacity(
        offset,
        (_opacity * 255).round(),
        super.paint,
        oldLayer: null,
      );
    } else {
      super.paint(context, offset);
    }
  }
}

/// 2. Compositing with clipping
class ClippingCompositingExample extends StatelessWidget {
  const ClippingCompositingExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Clipping Compositing'),
      ),
      body: Center(
        child: Container(
          width: 300,
          height: 300,
          color: Colors.grey[200],
          child: Stack(
            children: [
              // Background
              Container(
                color: Colors.blue,
                width: double.infinity,
                height: double.infinity,
              ),
              // Clipped image
              Positioned(
                left: 50,
                top: 50,
                child: ClipRRect(
                  borderRadius: BorderRadius.circular(20),
                  child: Container(
                    width: 100,
                    height: 100,
                    color: Colors.red,
                    child: const Center(
                      child: Text(
                        'Clipped',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                ),
              ),
              // Clipped with shape
              Positioned(
                left: 160,
                top: 50,
                child: ClipOval(
                  child: Container(
                    width: 100,
                    height: 100,
                    color: Colors.green,
                    child: const Center(
                      child: Text(
                        'Oval',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Custom compositing widget
- Opacity layer with RenderObject
- Clipping in compositing
- Shape clipping (oval, rounded)

---

# Best Practices

## Use Compositing for Effects

```dart
// Good - Compositing for visual effects
Stack(
  children: [
    BackgroundWidget(),
    OpacityLayer(),
    TransformLayer(),
    ClipLayer(),
  ],
)
```

## Use Appropriate Blend Modes

```dart
// Good - Right blend mode for the job
multiply -> Darkening
screen -> Lightening
overlay -> Contrast enhancement
```

## Optimize Compositing

```dart
// Good - Efficient compositing
// Use RepaintBoundary for complex compositing
RepaintBoundary(
  child: ComplexCompositingWidget(),
)
```

---

# Common Mistakes

## Overusing Blend Modes

Wrong:
```dart
// Too many blend modes
Paint()
  ..blendMode = BlendMode.multiply
  ..blendMode = BlendMode.screen
```

Correct:
```dart
// Single blend mode per paint
Paint()
  ..blendMode = BlendMode.multiply
```

## Not Using save/restore

Wrong:
```dart
// State not restored
canvas.save();
// Transformations applied
// Missing restore
```

Correct:
```dart
// Always restore
canvas.save();
// Transformations
canvas.restore();
```

---

# Summary

Compositing combines multiple visual layers into the final image. Use opacity, blend modes, and transformations for visual effects. Optimize compositing with appropriate layer types and RepaintBoundary. Compositing is essential for creating rich, engaging visual experiences.

---

# Next Steps

- [Layers](layers.md)
- [RenderObject](renderobject.md)
- [RenderBox](renderbox.md)

---

# Did You Know?

- Compositing combines visual layers
- Blend modes control color interaction
- Opacity creates transparency
- Transformations apply position/rotation
- Clipping limits visible area
- save/restore manages layers
- Compositing is GPU-accelerated
- RepaintBoundary optimizes compositing