# GestureDetector

Understand how to detect and handle user gestures in Flutter.

---

# What is it?

GestureDetector is a widget that detects various user gestures like taps, double taps, long presses, drags, and swipes. It provides a comprehensive way to handle user interactions by wrapping any widget and listening for specific gesture events. GestureDetector is one of the most fundamental and widely used widgets for handling user input.

---

# Why does it exist?

GestureDetector exists to:

- Detect user interactions with widgets
- Handle tap, double tap, and long press events
- Track drag and swipe gestures
- Support custom gesture handling
- Enable interactive UI elements
- Provide feedback for user actions
- Handle multi-touch interactions

---

# Basic GestureDetector

> **Handling basic** tap gestures.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic GestureDetector example
class BasicGestureDetectorExample extends StatefulWidget {
  const BasicGestureDetectorExample({super.key});

  @override
  State<BasicGestureDetectorExample> createState() => _BasicGestureDetectorExampleState();
}

class _BasicGestureDetectorExampleState extends State<BasicGestureDetectorExample> {
  // 1. State variables for tracking gestures
  Color _containerColor = Colors.blue;
  String _lastGesture = 'No gesture detected';
  int _tapCount = 0;
  int _doubleTapCount = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('GestureDetector'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 2. GestureDetector wrapping a Container
            GestureDetector(
              // 3. Tap gesture
              onTap: () {
                // Called when the user taps the widget
                setState(() {
                  _containerColor = Colors.green;
                  _lastGesture = 'Tap detected!';
                  _tapCount++;
                });
              },
              
              // 4. Double tap gesture
              onDoubleTap: () {
                // Called when the user double taps
                setState(() {
                  _containerColor = Colors.red;
                  _lastGesture = 'Double Tap detected!';
                  _doubleTapCount++;
                });
              },
              
              // 5. Long press gesture
              onLongPress: () {
                // Called when the user long presses
                setState(() {
                  _containerColor = Colors.purple;
                  _lastGesture = 'Long Press detected!';
                });
              },
              
              // 6. Tap down and up
              onTapDown: (details) {
                // Called when the user presses down
                print('Tap down at: ${details.globalPosition}');
              },
              
              onTapUp: (details) {
                // Called when the user releases
                print('Tap up at: ${details.globalPosition}');
              },
              
              onTapCancel: () {
                // Called when the tap is cancelled
                print('Tap cancelled');
              },
              
              // 7. The child widget
              child: Container(
                width: double.infinity,
                height: 200,
                color: _containerColor,
                child: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const Text(
                        'Tap Me!',
                        style: TextStyle(
                          color: Colors.white,
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 8),
                      Text(
                        _lastGesture,
                        style: const TextStyle(
                          color: Colors.white70,
                          fontSize: 16,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
            
            const SizedBox(height: 24),
            
            // 8. Gesture statistics
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceAround,
                children: [
                  Column(
                    children: [
                      const Text('Taps', style: TextStyle(fontWeight: FontWeight.bold)),
                      Text('$_tapCount'),
                    ],
                  ),
                  Column(
                    children: [
                      const Text('Double Taps', style: TextStyle(fontWeight: FontWeight.bold)),
                      Text('$_doubleTapCount'),
                    ],
                  ),
                  Column(
                    children: [
                      const Text('Status', style: TextStyle(fontWeight: FontWeight.bold)),
                      Text(_containerColor == Colors.blue ? 'Ready' : 'Gesture'),
                    ],
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
- onTap: Detects single tap
- onDoubleTap: Detects double tap
- onLongPress: Detects long press
- onTapDown/Up: Tracks press state
- Color and text change on gestures

---

# Drag Gestures

> **Handling drag** and swipe gestures.

```dart
/// Drag gestures example
class DragGestureExample extends StatefulWidget {
  const DragGestureExample({super.key});

  @override
  State<DragGestureExample> createState() => _DragGestureExampleState();
}

class _DragGestureExampleState extends State<DragGestureExample> {
  // 1. Position tracking
  Offset _position = const Offset(100, 100);
  Color _dragColor = Colors.blue;
  bool _isDragging = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Drag Gestures'),
      ),
      body: Stack(
        children: [
          // 2. Drag target
          Positioned(
            left: _position.dx,
            top: _position.dy,
            child: GestureDetector(
              // 3. Drag start
              onPanStart: (details) {
                setState(() {
                  _isDragging = true;
                  _dragColor = Colors.green;
                });
              },
              
              // 4. Drag update
              onPanUpdate: (details) {
                setState(() {
                  // Update position based on drag delta
                  _position += details.delta;
                  
                  // Clamp position to screen bounds
                  final screenSize = MediaQuery.of(context).size;
                  _position = Offset(
                    _position.dx.clamp(0, screenSize.width - 80),
                    _position.dy.clamp(0, screenSize.height - 80 - 56), // AppBar offset
                  );
                });
              },
              
              // 5. Drag end
              onPanEnd: (details) {
                setState(() {
                  _isDragging = false;
                  _dragColor = Colors.blue;
                });
                
                // 6. Physics-based fling effect
                final velocity = details.velocity;
                print('Velocity: ${velocity.pixelsPerSecond}');
                // In a real app, you'd use animation controller for fling
              },
              
              // 7. Drag cancel
              onPanCancel: () {
                setState(() {
                  _isDragging = false;
                  _dragColor = Colors.blue;
                });
              },
              
              child: Container(
                width: 80,
                height: 80,
                decoration: BoxDecoration(
                  color: _dragColor,
                  borderRadius: BorderRadius.circular(8),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.black.withOpacity(0.2),
                      blurRadius: 10,
                      spreadRadius: 2,
                      offset: const Offset(0, 4),
                    ),
                  ],
                ),
                child: Center(
                  child: Text(
                    _isDragging ? 'Drag' : 'Drag Me',
                    style: const TextStyle(color: Colors.white),
                  ),
                ),
              ),
            ),
          ),
          
          // 8. Position display
          Positioned(
            bottom: 20,
            left: 0,
            right: 0,
            child: Container(
              padding: const EdgeInsets.all(16),
              color: Colors.black54,
              child: Column(
                children: [
                  Text(
                    'Position: (${_position.dx.toStringAsFixed(0)}, ${_position.dy.toStringAsFixed(0)})',
                    style: const TextStyle(color: Colors.white),
                  ),
                  Text(
                    _isDragging ? 'Dragging...' : 'Ready',
                    style: const TextStyle(color: Colors.white),
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
- onPanStart: Begins drag
- onPanUpdate: Tracks drag movement
- onPanEnd: Ends drag
- Position updates in real-time
- Velocity for fling effects

---

# Multi-Touch Gestures

> **Handling multiple** touch points.

```dart
/// Multi-touch gesture example
class MultiTouchExample extends StatefulWidget {
  const MultiTouchExample({super.key});

  @override
  State<MultiTouchExample> createState() => _MultiTouchExampleState();
}

class _MultiTouchExampleState extends State<MultiTouchExample> {
  // 1. Track multiple touch points
  final Map<int, Offset> _touchPoints = {};
  int _touchCount = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Multi-Touch'),
        actions: [
          Padding(
            padding: const EdgeInsets.all(8),
            child: Text(
              'Touches: $_touchCount',
              style: const TextStyle(fontSize: 16),
            ),
          ),
        ],
      ),
      body: GestureDetector(
        // 2. Scale gesture (pinch)
        onScaleStart: (details) {
          print('Scale start: ${details.pointerCount} pointers');
          setState(() {
            _touchCount = details.pointerCount;
          });
        },
        
        // 3. Scale update (pinch/zoom)
        onScaleUpdate: (details) {
          setState(() {
            _touchCount = details.pointerCount;
            
            // Track each touch point
            if (details.pointerCount > 1) {
              print('Scale factor: ${details.scale}');
              print('Focal point: ${details.focalPoint}');
              print('Horizontal scale: ${details.horizontalScale}');
              print('Vertical scale: ${details.verticalScale}');
            }
          });
        },
        
        // 4. Scale end
        onScaleEnd: (details) {
          setState(() {
            _touchCount = 0;
          });
          print('Scale ended');
        },
        
        child: Container(
          color: Colors.grey[200],
          child: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // 5. Display touch info
                Text(
                  'Touch Points: $_touchCount',
                  style: const TextStyle(fontSize: 24),
                ),
                const SizedBox(height: 16),
                
                // 6. Touch visualization
                if (_touchCount > 1)
                  const Text(
                    'Pinch detected!',
                    style: TextStyle(
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                      color: Colors.blue,
                    ),
                  ),
                
                const SizedBox(height: 16),
                
                // 7. Instruction text
                Container(
                  padding: const EdgeInsets.all(16),
                  decoration: BoxDecoration(
                    color: Colors.white,
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: const Text(
                    'Use two fingers to pinch/zoom\nMulti-touch gestures detected',
                    textAlign: TextAlign.center,
                    style: TextStyle(fontSize: 14),
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- onScaleStart: Detects multi-touch
- onScaleUpdate: Tracks pinch/zoom
- onScaleEnd: Handles gesture end
- Touch count tracking

---

# GestureDetector with Custom Painting

> **Interactive custom** painting with gestures.

```dart
/// Drawing app with gestures
class DrawingApp extends StatefulWidget {
  const DrawingApp({super.key});

  @override
  State<DrawingApp> createState() => _DrawingAppState();
}

class _DrawingAppState extends State<DrawingApp> {
  // 1. Store drawing points
  List<Offset> _points = [];
  Color _selectedColor = Colors.black;
  double _strokeWidth = 4;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Drawing App'),
        actions: [
          // 2. Clear button
          IconButton(
            icon: const Icon(Icons.clear),
            onPressed: () {
              setState(() {
                _points.clear();
              });
            },
          ),
        ],
      ),
      body: GestureDetector(
        // 3. Start drawing
        onPanStart: (details) {
          setState(() {
            _points.add(details.localPosition);
          });
        },
        
        // 4. Continue drawing
        onPanUpdate: (details) {
          setState(() {
            _points.add(details.localPosition);
          });
        },
        
        // 5. End drawing
        onPanEnd: (details) {
          print('Drawing ended');
        },
        
        // 6. Custom paint widget
        child: CustomPaint(
          painter: DrawingPainter(
            points: _points,
            color: _selectedColor,
            strokeWidth: _strokeWidth,
          ),
          size: const Size.infinite,
        ),
      ),
      bottomNavigationBar: BottomAppBar(
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              // 7. Color selection
              _buildColorButton(Colors.black),
              _buildColorButton(Colors.red),
              _buildColorButton(Colors.green),
              _buildColorButton(Colors.blue),
              _buildColorButton(Colors.yellow),
              
              // 8. Stroke width
              SizedBox(
                width: 100,
                child: Slider(
                  value: _strokeWidth,
                  min: 1,
                  max: 20,
                  onChanged: (value) {
                    setState(() {
                      _strokeWidth = value;
                    });
                  },
                ),
              ),
              Text('${_strokeWidth.round()}px'),
            ],
          ),
        ),
      ),
    );
  }

  // Helper to build color buttons
  Widget _buildColorButton(Color color) {
    return GestureDetector(
      onTap: () {
        setState(() {
          _selectedColor = color;
        });
      },
      child: Container(
        width: 30,
        height: 30,
        decoration: BoxDecoration(
          color: color,
          shape: BoxShape.circle,
          border: Border.all(
            color: _selectedColor == color ? Colors.black : Colors.transparent,
            width: 3,
          ),
        ),
      ),
    );
  }
}

/// Custom painter for drawing
class DrawingPainter extends CustomPainter {
  DrawingPainter({
    required this.points,
    required this.color,
    required this.strokeWidth,
  });

  final List<Offset> points;
  final Color color;
  final double strokeWidth;

  @override
  void paint(Canvas canvas, Size size) {
    if (points.isEmpty) return;

    // 9. Create paint
    final paint = Paint()
      ..color = color
      ..strokeWidth = strokeWidth
      ..strokeCap = StrokeCap.round
      ..strokeJoin = StrokeJoin.round
      ..style = PaintingStyle.stroke;

    // 10. Draw the path
    final path = Path();
    path.moveTo(points.first.dx, points.first.dy);

    for (int i = 1; i < points.length; i++) {
      path.lineTo(points[i].dx, points[i].dy);
    }

    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(DrawingPainter oldDelegate) {
    return oldDelegate.points != points ||
           oldDelegate.color != color ||
           oldDelegate.strokeWidth != strokeWidth;
  }
}
```

What's happening here?
- Drawing with GestureDetector
- onPanStart/Update for drawing
- CustomPaint for rendering
- Interactive color and width controls

---

# Real-World Examples

> **Common patterns** with GestureDetector.

```dart
/// 1. Swipe to dismiss
class SwipeToDismiss extends StatefulWidget {
  const SwipeToDismiss({
    super.key,
    required this.child,
    required this.onDismissed,
  });

  final Widget child;
  final VoidCallback onDismissed;

  @override
  State<SwipeToDismiss> createState() => _SwipeToDismissState();
}

class _SwipeToDismissState extends State<SwipeToDismiss> {
  double _dragOffset = 0;
  bool _isDragging = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onHorizontalDragStart: (_) {
        setState(() {
          _isDragging = true;
        });
      },
      onHorizontalDragUpdate: (details) {
        setState(() {
          _dragOffset += details.delta.dx;
          _dragOffset = _dragOffset.clamp(0, 200.0);
        });
      },
      onHorizontalDragEnd: (details) {
        if (_dragOffset > 100) {
          widget.onDismissed();
        } else {
          setState(() {
            _dragOffset = 0;
            _isDragging = false;
          });
        }
      },
      child: Container(
        transform: Matrix4.translationValues(_dragOffset, 0, 0),
        child: Row(
          children: [
            Expanded(child: widget.child),
            if (_isDragging)
              Container(
                width: 60,
                color: Colors.red,
                child: const Icon(
                  Icons.delete,
                  color: Colors.white,
                ),
              ),
          ],
        ),
      ),
    );
  }
}

/// 2. Long press menu
class LongPressMenu extends StatefulWidget {
  const LongPressMenu({super.key, required this.child});

  final Widget child;

  @override
  State<LongPressMenu> createState() => _LongPressMenuState();
}

class _LongPressMenuState extends State<LongPressMenu> {
  bool _showMenu = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onLongPress: () {
        setState(() {
          _showMenu = !_showMenu;
        });
      },
      child: Column(
        children: [
          widget.child,
          if (_showMenu)
            Container(
              padding: const EdgeInsets.all(8),
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(8),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.2),
                    blurRadius: 8,
                  ),
                ],
              ),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  _buildMenuItem(Icons.copy, 'Copy'),
                  _buildMenuItem(Icons.share, 'Share'),
                  _buildMenuItem(Icons.delete, 'Delete'),
                ],
              ),
            ),
        ],
      ),
    );
  }

  Widget _buildMenuItem(IconData icon, String label) {
    return GestureDetector(
      onTap: () {
        setState(() {
          _showMenu = false;
        });
        print('$label pressed');
      },
      child: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
        child: Column(
          children: [
            Icon(icon),
            Text(
              label,
              style: const TextStyle(fontSize: 10),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Swipe to dismiss with animation
- Long press context menu
- Interactive UI elements
- Real-world gesture patterns

---

# Best Practices

## Combine Gestures Appropriately

```dart
// Good - Clear gesture hierarchy
GestureDetector(
  onTap: () => handleTap(),
  onDoubleTap: () => handleDoubleTap(),
  onLongPress: () => handleLongPress(),
)

// Bad - Conflicting gestures
GestureDetector(
  onTap: () => handleTap(),
  onLongPress: () => handleLongPress(),
  onPanStart: () => handlePan(), // Conflicts with tap
)
```

## Provide Visual Feedback

```dart
// Good - Feedback on interaction
GestureDetector(
  onTapDown: (_) => setState(() => _isPressed = true),
  onTapUp: (_) => setState(() => _isPressed = false),
  child: Container(
    opacity: _isPressed ? 0.7 : 1.0,
    // ...
  ),
)
```

## Handle Gesture Conflicts

```dart
// Good - Using GestureDetector with child
GestureDetector(
  behavior: HitTestBehavior.opaque,
  onTap: () => handleParentTap(),
  child: GestureDetector(
    onTap: () => handleChildTap(),
  ),
)
```

---

# Common Mistakes

## Ignoring Gesture Conflicts

Wrong:
```dart
// Both gestures conflict
GestureDetector(
  onTap: () => navigate(),
  child: ListView(
    children: [...],
  ),
)
```

Correct:
```dart
// Use GestureDetector on individual items
ListView.builder(
  itemBuilder: (context, index) {
    return GestureDetector(
      onTap: () => navigate(),
      child: ListTile(...),
    );
  },
)
```

## Not Providing Feedback

Wrong:
```dart
// No visual feedback
GestureDetector(
  onTap: () => submit(),
  child: Container(...),
)
```

Correct:
```dart
// With InkWell for Material feedback
InkWell(
  onTap: () => submit(),
  child: Container(...),
)
```

---

# Summary

GestureDetector provides comprehensive gesture detection for Flutter apps. Handle taps, double taps, long presses, drags, and multi-touch gestures. Combine with visual feedback and handle gesture conflicts appropriately. GestureDetector is essential for creating interactive applications.

---

# Next Steps

- [InkWell](inkwell.md)
- [Pointer Events](pointer-events.md)
- [Drag & Drop](drag-drop.md)

---

# Did You Know?

- GestureDetector can detect multiple gestures
- onTapDown/Up track press state
- Pan gestures handle drag interactions
- Scale gestures handle pinch/zoom
- Multi-touch is supported
- Gestures can be combined
- Visual feedback improves UX
- GestureDetector works with any widget