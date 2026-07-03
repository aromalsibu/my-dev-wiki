# Hit Testing

Understand how Flutter determines which widget receives touch events through hit testing.

---

# What is it?

Hit testing is the process Flutter uses to determine which widget should respond to a user's touch or click event. When a user taps the screen, Flutter performs hit testing from the top of the widget tree down to the bottom, checking which widgets contain the tap point. The hit test results determine which widget receives the event and how it propagates through the widget tree.

---

# Why does it exist?

Hit testing exists to:

- Determine which widget receives touch events
- Handle overlapping widgets correctly
- Enable gesture detection
- Support complex UI interactions
- Control event propagation
- Implement gesture recognition
- Provide accurate touch handling

---

# Basic Hit Testing

> **Understanding** how hit testing works.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic hit testing example
class BasicHitTestingExample extends StatefulWidget {
  const BasicHitTestingExample({super.key});

  @override
  State<BasicHitTestingExample> createState() => _BasicHitTestingExampleState();
}

class _BasicHitTestingExampleState extends State<BasicHitTestingExample> {
  // 1. Track which widget was tapped
  String _lastTapped = 'None';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Hit Testing'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. Parent widget
            // GestureDetector on the parent
            GestureDetector(
              onTap: () {
                setState(() {
                  _lastTapped = 'Parent (Container)';
                });
              },
              child: Container(
                width: double.infinity,
                height: 300,
                color: Colors.grey[200],
                child: Stack(
                  children: [
                    // 3. Child widgets inside the parent
                    // These are positioned on top of the parent
                    Positioned(
                      left: 50,
                      top: 50,
                      child: GestureDetector(
                        onTap: () {
                          setState(() {
                            _lastTapped = 'Child 1 (Red)';
                          });
                        },
                        child: Container(
                          width: 100,
                          height: 100,
                          color: Colors.red,
                          child: const Center(
                            child: Text(
                              'Tap Me',
                              style: TextStyle(color: Colors.white),
                            ),
                          ),
                        ),
                      ),
                    ),
                    Positioned(
                      right: 50,
                      bottom: 50,
                      child: GestureDetector(
                        onTap: () {
                          setState(() {
                            _lastTapped = 'Child 2 (Blue)';
                          });
                        },
                        child: Container(
                          width: 100,
                          height: 100,
                          color: Colors.blue,
                          child: const Center(
                            child: Text(
                              'Tap Me',
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
            
            const SizedBox(height: 24),
            
            // 4. Result display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Text(
                    'Last tapped: ',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  Text(
                    _lastTapped,
                    style: const TextStyle(fontSize: 16),
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
- Hit testing starts from top to bottom
- Child widgets can intercept events
- Tapping a child only triggers the child
- Tapping outside children triggers parent

---

# HitTestBehavior

> **Controlling hit test** behavior.

```dart
/// HitTestBehavior example
class HitTestBehaviorExample extends StatefulWidget {
  const HitTestBehaviorExample({super.key});

  @override
  State<HitTestBehaviorExample> createState() => _HitTestBehaviorExampleState();
}

class _HitTestBehaviorExampleState extends State<HitTestBehaviorExample> {
  // 1. Track tap results
  String _tapResult = 'Tap the circles to see behavior';
  
  // 2. Track which circle was tapped
  String _lastTapped = 'None';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('HitTestBehavior'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 3. Explanation of behaviors
            Container(
              padding: const EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'HitTestBehavior Types:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 4),
                  Text('• deferToChild: Only responds to child hits'),
                  Text('• opaque: Prevents hits to widgets behind'),
                  Text('• translucent: Allows hits to widgets behind'),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 4. Example with different behaviors
            Container(
              width: double.infinity,
              height: 300,
              color: Colors.grey[200],
              child: Stack(
                children: [
                  // 5. Background (deferToChild)
                  Positioned.fill(
                    child: GestureDetector(
                      behavior: HitTestBehavior.deferToChild,
                      onTap: () {
                        setState(() {
                          _tapResult = 'Background tapped (deferToChild)';
                          _lastTapped = 'Background';
                        });
                      },
                      child: Container(
                        color: Colors.grey[200],
                        child: const Center(
                          child: Text(
                            'Background (deferToChild)',
                            style: TextStyle(
                              color: Colors.grey,
                              fontSize: 16,
                            ),
                          ),
                        ),
                      ),
                    ),
                  ),
                  
                  // 6. Opaque circle
                  Positioned(
                    left: 50,
                    top: 50,
                    child: GestureDetector(
                      // opaque: Prevents taps from passing through
                      behavior: HitTestBehavior.opaque,
                      onTap: () {
                        setState(() {
                          _tapResult = 'Opaque circle tapped';
                          _lastTapped = 'Opaque';
                        });
                      },
                      child: Container(
                        width: 80,
                        height: 80,
                        decoration: const BoxDecoration(
                          color: Colors.red,
                          shape: BoxShape.circle,
                        ),
                        child: const Center(
                          child: Text(
                            'Opaque',
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 12,
                            ),
                          ),
                        ),
                      ),
                    ),
                  ),
                  
                  // 7. Translucent circle (overlapping)
                  Positioned(
                    left: 100,
                    top: 100,
                    child: GestureDetector(
                      // translucent: Allows taps to pass through
                      behavior: HitTestBehavior.translucent,
                      onTap: () {
                        setState(() {
                          _tapResult = 'Translucent circle tapped';
                          _lastTapped = 'Translucent';
                        });
                      },
                      child: Container(
                        width: 80,
                        height: 80,
                        decoration: const BoxDecoration(
                          color: Colors.blue,
                          shape: BoxShape.circle,
                        ),
                        child: const Center(
                          child: Text(
                            'Translucent',
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 12,
                            ),
                          ),
                        ),
                      ),
                    ),
                  ),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 8. Result display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  Text(
                    _tapResult,
                    style: const TextStyle(fontSize: 16),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    'Last tapped: $_lastTapped',
                    style: const TextStyle(
                      color: Colors.grey,
                      fontSize: 14,
                    ),
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
- deferToChild: Only child events
- opaque: Blocks events behind
- translucent: Passes events through
- Each behavior affects hit testing

---

# Custom Hit Testing

> **Customizing hit test** behavior.

```dart
/// Custom hit testing example
class CustomHitTestingExample extends StatefulWidget {
  const CustomHitTestingExample({super.key});

  @override
  State<CustomHitTestingExample> createState() => _CustomHitTestingExampleState();
}

class _CustomHitTestingExampleState extends State<CustomHitTestingExample> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Custom Hit Testing'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Custom hit test widget
            const CustomHitTestWidget(),
            
            const SizedBox(height: 24),
            
            // 2. Explanation
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Text(
                'Only the blue areas respond to taps.\n'
                'The transparent areas let clicks pass through.',
                textAlign: TextAlign.center,
                style: TextStyle(fontSize: 14),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// Custom hit test widget
class CustomHitTestWidget extends StatefulWidget {
  const CustomHitTestWidget({super.key});

  @override
  State<CustomHitTestWidget> createState() => _CustomHitTestWidgetState();
}

class _CustomHitTestWidgetState extends State<CustomHitTestWidget> {
  String _hitResult = 'Tap the shapes';

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        setState(() {
          _hitResult = 'Tapped the container';
        });
      },
      child: Container(
        width: 250,
        height: 250,
        color: Colors.grey[200],
        child: Stack(
          children: [
            // 3. Custom painter with hit testing
            // This widget handles hit testing for custom shapes
            CustomPaint(
              painter: CustomHitTestPainter(),
              size: const Size(250, 250),
              child: GestureDetector(
                onTapDown: (details) {
                  setState(() {
                    _hitResult = 'Tapped a custom shape';
                  });
                },
              ),
            ),
            
            // 4. Display result
            Positioned(
              bottom: 10,
              left: 0,
              right: 0,
              child: Center(
                child: Container(
                  padding: const EdgeInsets.all(8),
                  color: Colors.white,
                  child: Text(
                    _hitResult,
                    style: const TextStyle(
                      fontWeight: FontWeight.bold,
                      fontSize: 14,
                    ),
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// Custom painter with hit testing
class CustomHitTestPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // 5. Draw custom shapes
    final paint1 = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;
    
    final paint2 = Paint()
      ..color = Colors.red
      ..style = PaintingStyle.fill;
    
    // Rectangle
    canvas.drawRect(
      const Rect.fromLTWH(30, 30, 80, 80),
      paint1,
    );
    
    // Circle
    canvas.drawCircle(
      const Offset(160, 70),
      40,
      paint2,
    );
  }

  @override
  bool shouldRepaint(CustomHitTestPainter oldDelegate) => false;
}
```

What's happening here?
- Custom hit testing for shapes
- Only colored areas respond to taps
- Transparent areas let events pass
- Custom painter with hit test

---

# Hit Testing with Overlapping Widgets

> **Handling overlapping** widgets.

```dart
/// Overlapping widgets hit testing
class OverlappingWidgetsExample extends StatefulWidget {
  const OverlappingWidgetsExample({super.key});

  @override
  State<OverlappingWidgetsExample> createState() => _OverlappingWidgetsExampleState();
}

class _OverlappingWidgetsExampleState extends State<OverlappingWidgetsExample> {
  // 1. Track which widget was tapped
  String _tappedWidget = 'None';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Overlapping Widgets'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. Stack of overlapping widgets
            Container(
              width: 300,
              height: 300,
              color: Colors.grey[200],
              child: Stack(
                children: [
                  // 3. Bottom widget
                  Positioned.fill(
                    child: GestureDetector(
                      onTap: () {
                        setState(() {
                          _tappedWidget = 'Bottom (Grey)';
                        });
                      },
                      child: Container(
                        color: Colors.grey[300],
                        child: const Center(
                          child: Text(
                            'Bottom Widget',
                            style: TextStyle(fontSize: 18),
                          ),
                        ),
                      ),
                    ),
                  ),
                  
                  // 4. Middle widget (partially overlapping)
                  Positioned(
                    left: 50,
                    top: 50,
                    child: GestureDetector(
                      onTap: () {
                        setState(() {
                          _tappedWidget = 'Middle (Blue)';
                        });
                      },
                      child: Container(
                        width: 200,
                        height: 150,
                        color: Colors.blue.withOpacity(0.7),
                        child: const Center(
                          child: Text(
                            'Middle Widget',
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 16,
                            ),
                          ),
                        ),
                      ),
                    ),
                  ),
                  
                  // 5. Top widget (smaller, on top)
                  Positioned(
                    left: 120,
                    top: 120,
                    child: GestureDetector(
                      onTap: () {
                        setState(() {
                          _tappedWidget = 'Top (Red)';
                        });
                      },
                      child: Container(
                        width: 100,
                        height: 100,
                        color: Colors.red.withOpacity(0.8),
                        child: const Center(
                          child: Text(
                            'Top Widget',
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 16,
                            ),
                          ),
                        ),
                      ),
                    ),
                  ),
                ],
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 6. Result display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Text(
                    'Tapped: ',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  Text(
                    _tappedWidget,
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ],
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 7. Explanation
            Container(
              padding: const EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Text(
                'Top widgets receive taps first.\n'
                'If top widget doesn't handle the tap, it goes to the next layer.',
                textAlign: TextAlign.center,
                style: TextStyle(fontSize: 14),
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
- Top widgets get hit first
- Hit testing goes from top to bottom
- Overlapping widgets can intercept events
- Order determines hit test priority

---

# Real-World Examples

> **Common patterns** with hit testing.

```dart
/// 1. Interactive custom shape
class InteractiveShape extends StatefulWidget {
  const InteractiveShape({super.key});

  @override
  State<InteractiveShape> createState() => _InteractiveShapeState();
}

class _InteractiveShapeState extends State<InteractiveShape> {
  Color _shapeColor = Colors.blue;
  String _status = 'Tap the shape';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Interactive Shape'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 1. Custom shape with hit testing
            GestureDetector(
              onTap: () {
                setState(() {
                  _shapeColor = _shapeColor == Colors.blue 
                      ? Colors.red 
                      : Colors.blue;
                  _status = 'Shape tapped! Color changed';
                });
              },
              child: CustomPaint(
                painter: ShapePainter(color: _shapeColor),
                size: const Size(200, 200),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // 2. Status text
            Text(
              _status,
              style: const TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

/// Shape painter
class ShapePainter extends CustomPainter {
  ShapePainter({required this.color});

  final Color color;

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = color
      ..style = PaintingStyle.fill;

    // Draw a custom shape
    final path = Path()
      ..moveTo(size.width / 2, 0)
      ..lineTo(size.width, size.height * 0.75)
      ..lineTo(size.width * 0.25, size.height * 0.75)
      ..close();

    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(ShapePainter oldDelegate) {
    return oldDelegate.color != color;
  }
}

/// 2. Protected area (ignoring touches)
class ProtectedAreaExample extends StatelessWidget {
  const ProtectedAreaExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Protected Area'),
      ),
      body: Stack(
        children: [
          // 3. Background with tap detection
          GestureDetector(
            onTap: () {
              print('Background tapped');
            },
            child: Container(
              color: Colors.grey[200],
              child: const Center(
                child: Text(
                  'Tap anywhere (background)',
                  style: TextStyle(fontSize: 18),
                ),
              ),
            ),
          ),
          
          // 4. Protected area (ignores taps)
          Positioned(
            left: 50,
            top: 100,
            child: IgnorePointer(
              ignoring: true, // This makes the widget ignore all touches
              child: Container(
                width: 200,
                height: 150,
                decoration: BoxDecoration(
                  color: Colors.blue.withOpacity(0.5),
                  borderRadius: BorderRadius.circular(16),
                  border: Border.all(
                    color: Colors.blue,
                    width: 2,
                  ),
                ),
                child: const Center(
                  child: Text(
                    'Protected Area\n(No touches allowed)',
                    textAlign: TextAlign.center,
                    style: TextStyle(
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
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
- Interactive custom shapes
- Protected area with IgnorePointer
- Custom hit test handling
- Real-world hit test patterns

---

# Best Practices

## Use Appropriate HitTestBehavior

```dart
// Good - Appropriate behavior
GestureDetector(
  behavior: HitTestBehavior.opaque,
  onTap: () {},
  child: ...,
)
```

## Use IgnorePointer for Protection

```dart
// Good - Protect widgets from touches
IgnorePointer(
  ignoring: true,
  child: Container(...),
)
```

## Handle Custom Hit Testing

```dart
// Good - Custom hit test logic
class CustomWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Listener(
      onPointerDown: (event) {
        // Custom hit test logic
      },
      child: ...,
    );
  }
}
```

---

# Common Mistakes

## Wrong HitTestBehavior

Wrong:
```dart
// Taps pass through to background
GestureDetector(
  // Missing behavior
  onTap: () {},
  child: Container(...),
)
```

Correct:
```dart
// Block taps from passing through
GestureDetector(
  behavior: HitTestBehavior.opaque,
  onTap: () {},
  child: Container(...),
)
```

## Not Using IgnorePointer

Wrong:
```dart
// Widget that shouldn't receive taps
Container(
  onTap: () {}, // Still receives taps
)
```

Correct:
```dart
// Widget that ignores taps
IgnorePointer(
  ignoring: true,
  child: Container(...),
)
```

---

# Summary

Hit testing determines which widget receives touch events. HitTestBehavior controls event propagation, widgets on top receive events first, and overlapping widgets can intercept events. Use IgnorePointer to protect widgets from touches and custom hit testing for advanced scenarios.

---

# Next Steps

- [InkWell](inkwell.md)
- [Pointer Events](pointer-events.md)
- [Drag & Drop](drag-drop.md)

---

# Did You Know?

- Hit testing goes from top to bottom
- HitTestBehavior controls event propagation
- opaque blocks events to widgets behind
- translucent allows events to pass through
- IgnorePointer ignores all touches
- Widgets on top get hit first
- Hit testing is essential for gesture detection
- Custom hit testing is possible