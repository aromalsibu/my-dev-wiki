# Pointer Events

Understand how to handle low-level pointer events in Flutter.

---

# What is it?

Pointer Events are the lowest level of input handling in Flutter. They represent raw touch, mouse, or stylus events that occur when a user interacts with the screen. Pointer events include information about the pointer's position, pressure, velocity, and button states. They provide fine-grained control over user input.

---

# Why does it exist?

Pointer Events exist to:

- Handle raw input events
- Support custom gesture recognition
- Detect touch pressure and velocity
- Handle mouse and stylus input
- Support advanced interaction patterns
- Enable drawing and painting apps
- Provide low-level input control

---

# Basic Pointer Events

> **Handling raw** pointer events.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic pointer events example
class BasicPointerEventsExample extends StatefulWidget {
  const BasicPointerEventsExample({super.key});

  @override
  State<BasicPointerEventsExample> createState() => _BasicPointerEventsExampleState();
}

class _BasicPointerEventsExampleState extends State<BasicPointerEventsExample> {
  // 1. Track pointer state
  bool _isPointerDown = false;
  bool _isPointerHover = false;
  Offset _pointerPosition = Offset.zero;
  String _eventType = 'None';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Pointer Events'),
      ),
      body: Center(
        child: Listener(
          // 2. Pointer down event
          onPointerDown: (event) {
            setState(() {
              _isPointerDown = true;
              _pointerPosition = event.position;
              _eventType = 'Down';
              print('Pointer Down: ${event.position}');
              print('Device: ${event.kind}');
              print('Pressure: ${event.pressure}');
              print('Buttons: ${event.buttons}');
            });
          },
          
          // 3. Pointer up event
          onPointerUp: (event) {
            setState(() {
              _isPointerDown = false;
              _eventType = 'Up';
              print('Pointer Up: ${event.position}');
            });
          },
          
          // 4. Pointer move event
          onPointerMove: (event) {
            setState(() {
              _pointerPosition = event.position;
              _eventType = 'Move';
              
              // Check if the pointer is pressed (dragging)
              if (_isPointerDown) {
                _eventType = 'Drag';
              }
            });
          },
          
          // 5. Pointer hover event (mouse only)
          onPointerHover: (event) {
            setState(() {
              _isPointerHover = true;
              _pointerPosition = event.position;
              _eventType = 'Hover';
            });
          },
          
          // 6. Pointer cancel event
          onPointerCancel: (event) {
            setState(() {
              _isPointerDown = false;
              _eventType = 'Cancel';
            });
          },
          
          // 7. Pointer signal event (scroll wheel)
          onPointerSignal: (event) {
            if (event is PointerScrollEvent) {
              setState(() {
                _eventType = 'Scroll: ${event.scrollDelta}';
              });
            }
          },
          
          // 8. Child widget
          child: Container(
            width: 300,
            height: 300,
            decoration: BoxDecoration(
              color: _isPointerDown 
                  ? Colors.blue 
                  : _isPointerHover 
                      ? Colors.green 
                      : Colors.grey,
              borderRadius: BorderRadius.circular(16),
              boxShadow: [
                if (_isPointerDown)
                  BoxShadow(
                    color: Colors.blue.withOpacity(0.3),
                    blurRadius: 20,
                    spreadRadius: 5,
                  ),
              ],
            ),
            child: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  // 9. Event type display
                  Text(
                    'Event: $_eventType',
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  
                  // 10. Position display
                  Text(
                    'Position: (${_pointerPosition.dx.toStringAsFixed(0)}, '
                    '${_pointerPosition.dy.toStringAsFixed(0)})',
                    style: const TextStyle(
                      color: Colors.white70,
                      fontSize: 16,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    _isPointerDown ? '⬇️ Pressed' : '🖱️ Released',
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
      ),
    );
  }
}
```

What's happening here?
- onPointerDown: Touch/mouse press
- onPointerUp: Touch/mouse release
- onPointerMove: Movement tracking
- onPointerHover: Mouse hover
- onPointerCancel: Interrupted events
- onPointerSignal: Scroll wheel events

---

# Pointer Event Details

> **Accessing detailed** pointer information.

```dart
/// Pointer event details example
class PointerDetailsExample extends StatefulWidget {
  const PointerDetailsExample({super.key});

  @override
  State<PointerDetailsExample> createState() => _PointerDetailsExampleState();
}

class _PointerDetailsExampleState extends State<PointerDetailsExample> {
  // 1. Track detailed pointer information
  String _eventInfo = 'Move your pointer over the area';
  double _pressure = 0.0;
  double _force = 0.0;
  int _buttons = 0;
  String _kind = 'None';
  double _velocity = 0.0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Pointer Details'),
      ),
      body: Listener(
        // 2. Pointer down with details
        onPointerDown: (event) {
          setState(() {
            _eventInfo = 'Pointer Down';
            _pressure = event.pressure;
            _force = event.pressure;
            _buttons = event.buttons;
            _kind = event.kind.toString();
            _velocity = 0;
          });
          
          // 3. Access detailed event properties
          print('Pointer ID: ${event.pointer}');
          print('Device Kind: ${event.kind}');
          print('Pressure: ${event.pressure}');
          print('Buttons: ${event.buttons}');
          print('Position: ${event.position}');
          print('Delta: ${event.delta}');
        },
        
        // 4. Pointer move with velocity
        onPointerMove: (event) {
          setState(() {
            _eventInfo = 'Pointer Move';
            _pressure = event.pressure;
            _buttons = event.buttons;
            _velocity = _calculateVelocity(event);
          });
        },
        
        // 5. Pointer up
        onPointerUp: (event) {
          setState(() {
            _eventInfo = 'Pointer Up';
            _pressure = 0;
            _velocity = 0;
          });
        },
        
        // 6. Pointer hover
        onPointerHover: (event) {
          setState(() {
            _eventInfo = 'Pointer Hover';
            _buttons = 0;
            _velocity = 0;
          });
        },
        
        // 7. Display pointer information
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            children: [
              // 8. Interactive area
              Expanded(
                child: Container(
                  decoration: BoxDecoration(
                    color: Colors.grey[200],
                    borderRadius: BorderRadius.circular(16),
                  ),
                  child: Center(
                    child: Text(
                      _eventInfo,
                      style: const TextStyle(
                        fontSize: 24,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                ),
              ),
              
              // 9. Detailed info display
              Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(16),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.black.withOpacity(0.1),
                      blurRadius: 8,
                    ),
                  ],
                ),
                child: Column(
                  children: [
                    _buildInfoRow('Event', _eventInfo),
                    _buildInfoRow('Device Kind', _kind),
                    _buildInfoRow('Pressure', _pressure.toStringAsFixed(2)),
                    _buildInfoRow('Buttons', '0x${_buttons.toRadixString(16)}'),
                    _buildInfoRow('Velocity', '${_velocity.toStringAsFixed(2)} px/s'),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  // Helper to build info rows
  Widget _buildInfoRow(String label, String value) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(
            label,
            style: const TextStyle(fontWeight: FontWeight.bold),
          ),
          Text(value),
        ],
      ),
    );
  }

  // Calculate velocity from pointer event
  double _calculateVelocity(PointerMoveEvent event) {
    // Calculate velocity based on delta
    final delta = event.delta;
    final dt = event.timeStamp.inMilliseconds / 1000.0;
    if (dt == 0) return 0;
    return delta.distance / dt;
  }
}
```

What's happening here?
- pressure: Touch pressure
- buttons: Button state
- kind: Device type
- delta: Movement delta
- velocity: Movement speed

---

# Mouse Events

> **Handling mouse** specific events.

```dart
/// Mouse events example
class MouseEventsExample extends StatefulWidget {
  const MouseEventsExample({super.key});

  @override
  State<MouseEventsExample> createState() => _MouseEventsExampleState();
}

class _MouseEventsExampleState extends State<MouseEventsExample> {
  // 1. Track mouse state
  bool _isHovering = false;
  bool _isPrimaryDown = false;
  bool _isSecondaryDown = false;
  bool _isMiddleDown = false;
  Offset _position = Offset.zero;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Mouse Events'),
      ),
      body: Center(
        child: MouseRegion(
          // 2. Enter event
          onEnter: (event) {
            setState(() {
              _isHovering = true;
            });
          },
          
          // 3. Exit event
          onExit: (event) {
            setState(() {
              _isHovering = false;
              _isPrimaryDown = false;
              _isSecondaryDown = false;
              _isMiddleDown = false;
            });
          },
          
          // 4. Hover event
          onHover: (event) {
            setState(() {
              _position = event.position;
            });
          },
          
          // 5. Mouse down event
          onMouseDown: (event) {
            setState(() {
              // Check which button was pressed
              if (event.buttons == kPrimaryButton) {
                _isPrimaryDown = true;
              } else if (event.buttons == kSecondaryButton) {
                _isSecondaryDown = true;
              } else if (event.buttons == kMiddleButton) {
                _isMiddleDown = true;
              }
            });
          },
          
          // 6. Mouse up event
          onMouseUp: (event) {
            setState(() {
              _isPrimaryDown = false;
              _isSecondaryDown = false;
              _isMiddleDown = false;
            });
          },
          
          // 7. Child with visual feedback
          child: Container(
            width: 300,
            height: 300,
            decoration: BoxDecoration(
              color: _getBackgroundColor(),
              borderRadius: BorderRadius.circular(16),
              boxShadow: [
                if (_isHovering)
                  BoxShadow(
                    color: Colors.blue.withOpacity(0.3),
                    blurRadius: 20,
                    spreadRadius: 5,
                  ),
              ],
            ),
            child: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  // 8. Status text
                  Text(
                    _getStatusText(),
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 16),
                  
                  // 9. Position display
                  Text(
                    'Position: (${_position.dx.toStringAsFixed(0)}, '
                    '${_position.dy.toStringAsFixed(0)})',
                    style: const TextStyle(
                      color: Colors.white70,
                      fontSize: 16,
                    ),
                  ),
                  const SizedBox(height: 8),
                  
                  // 10. Mouse buttons status
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      _buildButtonIndicator('Left', _isPrimaryDown),
                      const SizedBox(width: 16),
                      _buildButtonIndicator('Right', _isSecondaryDown),
                      const SizedBox(width: 16),
                      _buildButtonIndicator('Middle', _isMiddleDown),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }

  // Helper to get background color
  Color _getBackgroundColor() {
    if (_isPrimaryDown) return Colors.blue;
    if (_isSecondaryDown) return Colors.red;
    if (_isMiddleDown) return Colors.green;
    if (_isHovering) return Colors.purple;
    return Colors.grey;
  }

  // Helper to get status text
  String _getStatusText() {
    if (_isPrimaryDown) return 'Left Click';
    if (_isSecondaryDown) return 'Right Click';
    if (_isMiddleDown) return 'Middle Click';
    if (_isHovering) return 'Hovering';
    return 'Move Mouse Here';
  }

  // Helper to build button indicator
  Widget _buildButtonIndicator(String label, bool isPressed) {
    return Column(
      children: [
        Container(
          width: 20,
          height: 20,
          decoration: BoxDecoration(
            color: isPressed ? Colors.white : Colors.white38,
            shape: BoxShape.circle,
          ),
        ),
        const SizedBox(height: 4),
        Text(
          label,
          style: const TextStyle(
            color: Colors.white70,
            fontSize: 12,
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- onEnter/Exit: Hover state
- onMouseDown/Up: Button press
- Button detection: Primary, secondary, middle
- Visual feedback for mouse state

---

# Pointer Event with Canvas

> **Drawing with** pointer events.

```dart
/// Pointer events with canvas drawing
class CanvasDrawingExample extends StatefulWidget {
  const CanvasDrawingExample({super.key});

  @override
  State<CanvasDrawingExample> createState() => _CanvasDrawingExampleState();
}

class _CanvasDrawingExampleState extends State<CanvasDrawingExample> {
  // 1. Track drawing state
  List<List<Offset>> _strokes = [];
  List<Offset>? _currentStroke;
  bool _isDrawing = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Canvas Drawing'),
        actions: [
          // 2. Clear button
          IconButton(
            icon: const Icon(Icons.clear),
            onPressed: () {
              setState(() {
                _strokes.clear();
                _currentStroke = null;
              });
            },
          ),
        ],
      ),
      body: Listener(
        // 3. Pointer down - start drawing
        onPointerDown: (event) {
          setState(() {
            _isDrawing = true;
            _currentStroke = [event.localPosition];
          });
        },
        
        // 4. Pointer move - continue drawing
        onPointerMove: (event) {
          if (_isDrawing && _currentStroke != null) {
            setState(() {
              _currentStroke!.add(event.localPosition);
            });
          }
        },
        
        // 5. Pointer up - finish stroke
        onPointerUp: (event) {
          if (_currentStroke != null && _currentStroke!.isNotEmpty) {
            setState(() {
              _strokes.add(_currentStroke!);
              _currentStroke = null;
              _isDrawing = false;
            });
          }
        },
        
        // 6. Pointer cancel - cancel drawing
        onPointerCancel: (event) {
          setState(() {
            _currentStroke = null;
            _isDrawing = false;
          });
        },
        
        // 7. Custom paint for drawing
        child: Container(
          color: Colors.white,
          child: CustomPaint(
            painter: DrawingPainter(
              strokes: _strokes,
              currentStroke: _currentStroke,
            ),
            size: const Size.infinite,
          ),
        ),
      ),
    );
  }
}

/// Custom painter for drawing
class DrawingPainter extends CustomPainter {
  DrawingPainter({
    required this.strokes,
    required this.currentStroke,
  });

  final List<List<Offset>> strokes;
  final List<Offset>? currentStroke;

  @override
  void paint(Canvas canvas, Size size) {
    // 8. Paint for completed strokes
    final paint = Paint()
      ..color = Colors.black
      ..strokeWidth = 4
      ..strokeCap = StrokeCap.round
      ..strokeJoin = StrokeJoin.round
      ..style = PaintingStyle.stroke;

    // 9. Draw completed strokes
    for (final stroke in strokes) {
      if (stroke.length > 1) {
        final path = Path();
        path.moveTo(stroke.first.dx, stroke.first.dy);
        for (int i = 1; i < stroke.length; i++) {
          path.lineTo(stroke[i].dx, stroke[i].dy);
        }
        canvas.drawPath(path, paint);
      }
    }

    // 10. Draw current stroke
    if (currentStroke != null && currentStroke!.length > 1) {
      final paint2 = Paint()
        ..color = Colors.blue
        ..strokeWidth = 4
        ..strokeCap = StrokeCap.round
        ..strokeJoin = StrokeJoin.round
        ..style = PaintingStyle.stroke;

      final path = Path();
      path.moveTo(currentStroke!.first.dx, currentStroke!.first.dy);
      for (int i = 1; i < currentStroke!.length; i++) {
        path.lineTo(currentStroke![i].dx, currentStroke![i].dy);
      }
      canvas.drawPath(path, paint2);
    }
  }

  @override
  bool shouldRepaint(DrawingPainter oldDelegate) {
    return oldDelegate.strokes != strokes ||
           oldDelegate.currentStroke != currentStroke;
  }
}
```

What's happening here?
- Pointer events for drawing
- onPointerDown: Start stroke
- onPointerMove: Continue stroke
- onPointerUp: End stroke
- CustomPaint for rendering

---

# Best Practices

## Handle Multiple Pointers

```dart
// Good - Track multiple pointers
Map<int, Offset> _pointers = {};

onPointerDown: (event) {
  _pointers[event.pointer] = event.position;
}

onPointerUp: (event) {
  _pointers.remove(event.pointer);
}
```

## Use Appropriate Hit Testing

```dart
// Good - Control hit testing
Listener(
  behavior: HitTestBehavior.opaque,
  onPointerDown: (event) {
    // Always receives events
  },
)
```

## Handle Cancellation

```dart
// Good - Handle pointer cancel
onPointerCancel: (event) {
  // Clean up state
  _resetState();
}
```

---

# Common Mistakes

## Ignoring Pointer Cancel

Wrong:
```dart
// No cancellation handling
Listener(
  onPointerDown: (event) => startInteraction(),
  onPointerUp: (event) => endInteraction(),
  // Missing onPointerCancel
)
```

Correct:
```dart
// Handle cancellation
Listener(
  onPointerDown: (event) => startInteraction(),
  onPointerUp: (event) => endInteraction(),
  onPointerCancel: (event) => cancelInteraction(),
)
```

## Not Checking Button State

Wrong:
```dart
// Can't distinguish between left/right click
onPointerDown: (event) {
  // Same behavior for all buttons
}
```

Correct:
```dart
// Check buttons
onPointerDown: (event) {
  if (event.buttons == kPrimaryButton) {
    // Left click
  } else if (event.buttons == kSecondaryButton) {
    // Right click
  }
}
```

---

# Summary

Pointer Events provide low-level input handling with detailed information about touch, mouse, and stylus interactions. Use Listener for raw pointer events, MouseRegion for mouse-specific events, and access detailed properties like pressure, velocity, and button states.

---

# Next Steps

- [Drag & Drop](drag-drop.md)
- [Hit Testing](hit-testing.md)
- [InkWell](inkwell.md)

---

# Did You Know?

- Pointer events are the lowest level of input
- onPointerDown/Up track touch state
- MouseRegion handles mouse-specific events
- pressure provides touch sensitivity
- buttons identifies mouse buttons
- delta tracks movement
- velocity calculates speed
- Pointer events support stylus input