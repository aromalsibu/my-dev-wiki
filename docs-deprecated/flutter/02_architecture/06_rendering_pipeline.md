# Rendering Pipeline

Understand how Flutter renders pixels on the screen.

---

# What is it?

The rendering pipeline is the process by which Flutter takes your widget tree and converts it into actual pixels on the screen. It involves multiple stages including building, layout, painting, compositing, and rasterization. The pipeline runs every frame (typically 60 or 120 times per second) to keep the UI smooth and responsive.

---

# Why does it exist?

The rendering pipeline exists to:

- Convert widget configurations into visual output
- Maintain smooth 60fps or 120fps performance
- Handle complex animations and transitions
- Optimize rendering for different platforms
- Manage the frame budget efficiently
- Provide consistent rendering across devices

---

# The Rendering Pipeline Stages

> **The pipeline** consists of several sequential stages.

```
1. Build Phase
   ↓
2. Layout Phase
   ↓
3. Paint Phase
   ↓
4. Compositing Phase
   ↓
5. Rasterization Phase
   ↓
6. Display Phase
```

What's happening here?
- Each stage depends on the previous
- Stages run in order every frame
- Each stage has a specific purpose
- Performance is critical at each stage
- Total time must be under 16ms

---

# Stage 1: Build Phase

> **Build phase** creates and updates the widget tree.

```dart
// Build phase in action
class BuildPhaseWidget extends StatefulWidget {
  const BuildPhaseWidget({super.key});

  @override
  State<BuildPhaseWidget> createState() => _BuildPhaseWidgetState();
}

class _BuildPhaseWidgetState extends State<BuildPhaseWidget> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    // Build phase starts here
    // Creates widget configuration
    
    print('Build phase: Creating widgets');
    
    return Column(
      children: [
        Text('Counter: $_counter'),
        ElevatedButton(
          onPressed: () {
            setState(() {
              _counter++;
            });
          },
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

// What happens during build:
// 1. Widgets are created or updated
// 2. Widget tree is configured
// 3. Elements are updated
// 4. Dirty widgets are rebuilt
// 5. Only changed widgets rebuild

// Build phase timeline:
// 1. Schedule frame
// 2. Mark dirty widgets
// 3. Rebuild dirty widgets
// 4. Update element tree
// 5. Proceed to layout
```

What's happening here?
- Creates/updates widget configurations
- Only dirty widgets rebuild
- Builds the widget tree
- Updates the element tree
- Quick and efficient

---

# Stage 2: Layout Phase

> **Layout phase** computes sizes and positions.

```dart
// Layout phase in action
class LayoutPhaseWidget extends StatelessWidget {
  const LayoutPhaseWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 200,
      height: 100,
      color: Colors.blue,
      child: const Center(
        child: Text('Layout'),
      ),
    );
  }
}

// Behind the scenes layout:
// Container → RenderConstrainedBox
// Center → RenderPositionedBox
// Text → RenderParagraph

// Layout process:
// 1. Constraints flow down from parent
// 2. Child determines its size
// 3. Parent positions child
// 4. Parent sets its own size
// 5. Sizes flow back up the tree

// Layout constraints:
class LayoutExample extends StatefulWidget {
  const LayoutExample({super.key});

  @override
  State<LayoutExample> createState() => _LayoutExampleState();
}

class _LayoutExampleState extends State<LayoutExample> {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Unconstrained - child sizes itself
        Container(
          color: Colors.red,
          child: const Text('Unconstrained'),
        ),
        
        // 2. Constrained - parent forces size
        SizedBox(
          width: 200,
          height: 50,
          child: Container(
            color: Colors.green,
            child: const Text('Constrained'),
          ),
        ),
        
        // 3. Expanded - flexible layout
        Expanded(
          child: Container(
            color: Colors.blue,
            child: const Text('Expanded'),
          ),
        ),
      ],
    );
  }
}

// Layout timeline:
// 1. Parent passes constraints
// 2. Child performs layout
// 3. Child returns size
// 4. Parent positions child
// 5. Layout complete
```

What's happening here?
- Computes sizes and positions
- Constraints flow down
- Sizes flow back up
- Every render object is laid out
- Positions are determined

---

# Stage 3: Paint Phase

> **Paint phase** draws the UI elements.

```dart
// Paint phase in action
class PaintPhaseWidget extends CustomPaint {
  const PaintPhaseWidget({super.key});

  @override
  void paint(Canvas canvas, Size size) {
    // Paint phase draws to canvas
    
    // 1. Draw background
    final Paint bgPaint = Paint()..color = Colors.blue;
    canvas.drawRect(Rect.fromLTWH(0, 0, size.width, size.height), bgPaint);
    
    // 2. Draw circle
    final Paint circlePaint = Paint()..color = Colors.red;
    canvas.drawCircle(Offset(size.width/2, size.height/2), 50, circlePaint);
    
    // 3. Draw text
    final TextPainter textPainter = TextPainter(
      text: const TextSpan(text: 'Paint Phase'),
      textDirection: TextDirection.ltr,
    );
    textPainter.layout();
    textPainter.paint(canvas, Offset(10, 10));
  }
}

// What happens during paint:
// 1. Paint methods are called
// 2. Canvas operations are recorded
// 3. Draw commands are generated
// 4. Display lists are created
// 5. Only dirty areas repaint

// Paint optimization:
class OptimizedPaintWidget extends StatelessWidget {
  const OptimizedPaintWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return RepaintBoundary(
      child: CustomPaint(
        painter: MyPainter(),
        child: const Text('Optimized paint'),
      ),
    );
  }
}

// Paint timeline:
// 1. Determine dirty regions
// 2. Paint dirty regions
// 3. Record paint commands
// 4. Create display lists
// 5. Send to compositor
```

What's happening here?
- Draws UI elements to canvas
- Records draw commands
- Only paints dirty regions
- Creates display lists
- Handles custom painting

---

# Stage 4: Compositing Phase

> **Compositing phase** combines layers together.

```dart
// Compositing in action
class CompositingWidget extends StatelessWidget {
  const CompositingWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        // 1. Background layer
        Container(
          color: Colors.blue,
          child: const Text('Background'),
        ),
        
        // 2. Middle layer
        Container(
          color: Colors.red.withOpacity(0.5),
          child: const Text('Middle'),
        ),
        
        // 3. Foreground layer
        Container(
          color: Colors.green.withOpacity(0.5),
          child: const Text('Foreground'),
        ),
      ],
    );
  }
}

// Layer types:
// 1. PictureLayer - Paint commands
// 2. TextureLayer - Video/GPU content
// 3. OffsetLayer - Positioned children
// 4. OpacityLayer - Opacity effects
// 5. ClipRectLayer - Clipping
// 6. TransformLayer - Transformations

// Compositing process:
// 1. Collect all layers
// 2. Sort layers by z-index
// 3. Apply layer effects
// 4. Combine layers together
// 5. Create composite image

// Custom compositing:
class CustomLayerWidget extends StatelessWidget {
  const CustomLayerWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Transform(
      transform: Matrix4.rotationZ(0.5),
      child: Container(
        color: Colors.blue,
        child: const Text('Transformed'),
      ),
    );
  }
}
```

What's happening here?
- Combines multiple layers
- Applies transformations
- Handles opacity and effects
- Creates final composite
- GPU-accelerated

---

# Stage 5: Rasterization Phase

> **Rasterization phase** converts vector graphics to pixels.

```dart
// Rasterization process
class RasterizationExample extends StatelessWidget {
  const RasterizationExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      // These will be rasterized
      width: 100,
      height: 100,
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(10),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.3),
            blurRadius: 10,
            offset: const Offset(5, 5),
          ),
        ],
      ),
      child: const Text('Rasterized'),
    );
  }
}

// Rasterization steps:
// 1. Vector graphics to pixels
// 2. Apply anti-aliasing
// 3. Apply filters and effects
// 4. Render to frame buffer
// 5. Prepare for display

// Performance considerations:
// 1. Rasterization is GPU-intensive
// 2. Complex shapes cost more
// 3. Shadows and effects are expensive
// 4. Large areas cost more
// 5. Use RepaintBoundary to isolate

// Rasterization optimization:
class OptimizedRasterization extends StatelessWidget {
  const OptimizedRasterization({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Simplify shapes
    return Container(
      color: Colors.blue, // Simple shape - cheap
    );
    
    // 2. Avoid unnecessary effects
    // Use flat designs instead of heavy shadows
    
    // 3. Use raster cache for static content
    // Wrap in RepaintBoundary
  }
}
```

What's happening here?
- Vector to pixel conversion
- GPU handles rasterization
- Anti-aliasing applied
- Effects and filters rendered
- Final pixel buffer created

---

# Stage 6: Display Phase

> **Display phase** shows the final image on screen.

```dart
// Display process
class DisplayExample extends StatelessWidget {
  const DisplayExample({super.key});

  @override
  Widget build(BuildContext context) {
    // After rasterization:
    // 1. Frame buffer is ready
    // 2. Screen refreshes (vsync)
    // 3. Frame is displayed
    // 4. User sees the update
    
    return const Text('Displayed');
  }
}

// Display timeline:
// 1. Frame buffer complete
// 2. Wait for vsync
// 3. Swap buffers
// 4. Display on screen
// 5. Ready for next frame

// Frame timing:
// 60fps = 16.6ms per frame
// 120fps = 8.3ms per frame
// All stages must fit in frame budget
// Or frame drops occur
```

What's happening here?
- Frame buffer displayed
- Vsync synchronizes display
- Buffer swapping occurs
- User sees the update
- Ready for next frame

---

# Frame Scheduling

> **Frames are scheduled** by the engine.

```dart
// Frame scheduling
class FrameScheduler extends StatefulWidget {
  const FrameScheduler({super.key});

  @override
  State<FrameScheduler> createState() => _FrameSchedulerState();
}

class _FrameSchedulerState extends State<FrameScheduler>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
  }

  @override
  Widget build(BuildContext context) {
    // Frame is scheduled by AnimationController
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        // Called every frame
        return Transform.rotate(
          angle: _controller.value * 2 * 3.14,
          child: const Icon(Icons.refresh),
        );
      },
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}

// Frame scheduling triggers:
// 1. setState()
// 2. Animation controllers
// 3. WidgetsBinding.instance.scheduleFrame()
// 4. Native platform events
// 5. Timer callbacks

// Manual frame scheduling:
void _scheduleFrame() {
  WidgetsBinding.instance.scheduleFrame();
}
```

What's happening here?
- Frames are scheduled as needed
- Animations trigger frames
- State changes trigger frames
- Frames are batched together
- Vsync controls frame timing

---

# Performance Optimization

> **Optimizing the pipeline** for performance.

```dart
// Performance optimization techniques
class OptimizedPipeline extends StatelessWidget {
  const OptimizedPipeline({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Use RepaintBoundary
        RepaintBoundary(
          child: AnimatedWidget(), // Isolated repaint
        ),
        
        // 2. Use const widgets
        const Text('Static content'), // No rebuild
        
        // 3. Cache complex widgets
        const CachedWidget(
          child: ComplexWidget(),
        ),
        
        // 4. Use ListView.builder
        // Not all children built at once
        
        // 5. Use keys for lists
        // Efficient list updates
      ],
    );
  }
}

// Performance profiling:
class ProfilePipeline extends StatelessWidget {
  const ProfilePipeline({super.key});

  @override
  Widget build(BuildContext context) {
    // Use Flutter DevTools:
    // 1. Performance tab
    // 2. Timeline recording
    // 3. Frame analysis
    // 4. Identify bottlenecks
    // 5. Optimize slow frames
    
    return const Text('Profile me');
  }
}

// Common performance issues:
// 1. Build too slow
// 2. Layout too complex
// 3. Paint too expensive
// 4. Compositing too heavy
// 5. Rasterization too slow
// 6. Frame budget exceeded
```

What's happening here?
- RepaintBoundary isolates painting
- const widgets reduce builds
- Cache complex widgets
- Use efficient widgets
- Profile and optimize

---

# Pipeline in Different Modes

> **Pipeline behavior** changes in different modes.

```dart
// Debug mode
class DebugPipeline extends StatelessWidget {
  const DebugPipeline({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Full pipeline
    // 2. Debug checks enabled
    // 3. Slower performance
    // 4. Detailed error messages
    // 5. Hot reload available
    return const Text('Debug mode');
  }
}

// Profile mode
class ProfilePipeline extends StatelessWidget {
  const ProfilePipeline({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Full pipeline
    // 2. Performance tracking
    // 3. Tracing enabled
    // 4. Optimized for profiling
    // 5. Best for performance testing
    return const Text('Profile mode');
  }
}

// Release mode
class ReleasePipeline extends StatelessWidget {
  const ReleasePipeline({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Optimized pipeline
    // 2. Debug checks removed
    // 3. Fastest performance
    // 4. No hot reload
    // 5. AOT compilation
    return const Text('Release mode');
  }
}
```

What's happening here?
- Debug mode for development
- Profile mode for performance
- Release mode for production
- Each mode optimizes differently
- Choose based on purpose

---

# Best Practices

## Understand Frame Budget

```dart
// Frame budget tracking
class FrameBudgetWidget extends StatelessWidget {
  const FrameBudgetWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // 60fps = 16.6ms per frame
    // 120fps = 8.3ms per frame
    
    // Keep within budget:
    // Build: < 8ms
    // Layout: < 4ms
    // Paint: < 4ms
    // Composite: < 2ms
    // Total: < 16.6ms
    
    return const Text('Frame budget');
  }
}
```

## Use Repaint Boundaries

```dart
// Good - Isolate expensive paints
@override
Widget build(BuildContext context) {
  return RepaintBoundary(
    child: ExpensivePaintingWidget(),
  );
}

// Bad - No isolation
@override
Widget build(BuildContext context) {
  return ExpensivePaintingWidget(); // Repaints with parent
}
```

## Avoid Overdraw

```dart
// Good - Minimize overdraw
Container(
  color: Colors.blue,
  child: const Text('Hello'),
)

// Bad - Multiple overlapping layers
Stack(
  children: [
    Container(color: Colors.blue),
    Container(color: Colors.red.withOpacity(0.5)),
    Container(color: Colors.green.withOpacity(0.5)),
  ],
)
```

---

# Common Mistakes

## Too Many Widgets

Wrong:
```dart
// 1000+ widgets in a single view
Column(
  children: List.generate(1000, (index) {
    return Text('Item $index');
  }),
)
```

Correct:
```dart
// Use ListView for large lists
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) {
    return Text('Item $index');
  },
)
```

## Expensive Operations

Wrong:
```dart
// Expensive paint operation
@override
void paint(Canvas canvas, Size size) {
  for (int i = 0; i < 1000; i++) {
    // Many draw operations
    canvas.drawCircle(Offset(i.toDouble(), 100), 10, paint);
  }
}
```

Correct:
```dart
// Batch operations or cache
// Use RepaintBoundary
// Simplify painting
```

---

# Summary

The rendering pipeline converts widgets to pixels through build, layout, paint, compositing, rasterization, and display stages. Understanding each stage helps optimize performance. Keep operations efficient, use repaint boundaries, and stay within frame budget.

---

# Next Steps

- [Frame Scheduling](frame-scheduling.md)
- [Constraints](constraints.md)
- [Layout Process](layout-process.md)

---

# Did You Know?

- Flutter targets 60fps by default
- Some devices support 120fps
- The pipeline runs per frame
- Repaint boundaries isolate paint
- Rasterization is GPU-accelerated
- Compositing happens on the GPU
- The entire pipeline is in Dart and C++
- Jank occurs when frames exceed budget