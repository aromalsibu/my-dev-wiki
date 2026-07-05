# Custom Layouts

Understand how to create custom layout widgets in Flutter.

---

# What is it?

Custom Layouts are widgets that implement your own layout logic when the built-in layout widgets (Row, Column, Stack, etc.) don't meet your needs. By creating custom layout widgets, you gain complete control over how children are sized and positioned, enabling complex and specialized layout patterns.

---

# Why does it exist?

Custom Layouts exist to:

- Implement custom layout algorithms
- Create specialized layout patterns
- Handle complex positioning logic
- Build custom UI components
- Support unique design requirements
- Optimize layout performance
- Enable creative UI designs

---

# MultiChildRenderObjectWidget

> **The foundation** for custom multi-child layouts.

```dart
// 1. Create a custom widget class
class CustomFlowWidget extends MultiChildRenderObjectWidget {
  const CustomFlowWidget({
    super.key,
    required super.children,
    this.spacing = 8,
  });

  final double spacing;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return CustomFlowRenderObject(spacing: spacing);
  }

  @override
  void updateRenderObject(
    BuildContext context,
    CustomFlowRenderObject renderObject,
  ) {
    renderObject.spacing = spacing;
  }
}

// 2. Create the render object
class CustomFlowRenderObject extends RenderBox
    with ContainerRenderObjectMixin<RenderBox, BoxParentData>,
         RenderBoxContainerDefaultsMixin<RenderBox, BoxParentData> {
  
  CustomFlowRenderObject({required double spacing}) : _spacing = spacing;

  double _spacing;
  set spacing(double value) {
    if (_spacing == value) return;
    _spacing = value;
    markNeedsLayout();
  }

  @override
  void setupParentData(RenderObject child) {
    if (child.parentData is! BoxParentData) {
      child.parentData = BoxParentData();
    }
  }

  @override
  void performLayout() {
    // Layout logic here
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    // Paint children
  }
}

// 3. Using the custom widget
class CustomLayoutExample extends StatelessWidget {
  const CustomLayoutExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 200,
      color: Colors.grey[200],
      child: CustomFlowWidget(
        spacing: 8,
        children: [
          Container(width: 50, height: 50, color: Colors.red),
          Container(width: 60, height: 40, color: Colors.green),
          Container(width: 40, height: 60, color: Colors.blue),
          Container(width: 70, height: 30, color: Colors.orange),
          Container(width: 30, height: 70, color: Colors.purple),
        ],
      ),
    );
  }
}
```

What's happening here?
- Custom widget extends MultiChildRenderObjectWidget
- RenderObject handles layout and painting
- Children are managed with mixins
- Full control over layout logic

---

# Custom Layout Example: Masonry Flow

> **Creating a masonry/pinterest-style layout.**

```dart
// 1. Masonry widget
class MasonryFlow extends MultiChildRenderObjectWidget {
  const MasonryFlow({
    super.key,
    required super.children,
    this.columnCount = 2,
    this.spacing = 8,
  });

  final int columnCount;
  final double spacing;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return MasonryRenderObject(
      columnCount: columnCount,
      spacing: spacing,
    );
  }

  @override
  void updateRenderObject(
    BuildContext context,
    MasonryRenderObject renderObject,
  ) {
    renderObject
      ..columnCount = columnCount
      ..spacing = spacing;
  }
}

// 2. Masonry render object
class MasonryRenderObject extends RenderBox
    with ContainerRenderObjectMixin<RenderBox, BoxParentData>,
         RenderBoxContainerDefaultsMixin<RenderBox, BoxParentData> {
  
  MasonryRenderObject({
    required int columnCount,
    required double spacing,
  }) : _columnCount = columnCount,
       _spacing = spacing;

  int _columnCount;
  set columnCount(int value) {
    if (_columnCount == value) return;
    _columnCount = value;
    markNeedsLayout();
  }

  double _spacing;
  set spacing(double value) {
    if (_spacing == value) return;
    _spacing = value;
    markNeedsLayout();
  }

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
    
    // 2. Calculate column widths
    final totalSpacing = (_columnCount - 1) * _spacing;
    final columnWidth = (constraints.maxWidth - totalSpacing) / _columnCount;
    
    // 3. Track column heights
    List<double> columnHeights = List.filled(_columnCount, 0.0);
    
    // 4. Layout each child
    RenderBox? child = firstChild;
    while (child != null) {
      // Find shortest column
      int shortestColumn = 0;
      double minHeight = columnHeights[0];
      for (int i = 1; i < columnHeights.length; i++) {
        if (columnHeights[i] < minHeight) {
          minHeight = columnHeights[i];
          shortestColumn = i;
        }
      }
      
      // Layout child
      child.layout(
        BoxConstraints(
          minWidth: columnWidth,
          maxWidth: columnWidth,
          minHeight: 0,
          maxHeight: constraints.maxHeight,
        ),
        parentUsesSize: true,
      );
      
      // Position child
      final BoxParentData childParentData = 
          child.parentData! as BoxParentData;
      childParentData.offset = Offset(
        shortestColumn * (columnWidth + _spacing),
        columnHeights[shortestColumn],
      );
      
      // Update column height
      columnHeights[shortestColumn] += child.size.height + _spacing;
      
      child = childAfter(child);
    }
    
    // 5. Set own size
    final maxHeight = columnHeights.reduce((a, b) => a > b ? a : b);
    size = constraints.constrain(Size(
      constraints.maxWidth,
      maxHeight,
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

// 3. Using masonry layout
class MasonryExample extends StatelessWidget {
  const MasonryExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 400,
      color: Colors.grey[200],
      child: MasonryFlow(
        columnCount: 3,
        spacing: 8,
        children: [
          _buildMasonryItem(100, Colors.red),
          _buildMasonryItem(150, Colors.green),
          _buildMasonryItem(80, Colors.blue),
          _buildMasonryItem(120, Colors.orange),
          _buildMasonryItem(200, Colors.purple),
          _buildMasonryItem(90, Colors.pink),
          _buildMasonryItem(160, Colors.teal),
          _buildMasonryItem(110, Colors.indigo),
        ],
      ),
    );
  }

  Widget _buildMasonryItem(double height, Color color) {
    return Container(
      height: height,
      color: color,
      child: Center(
        child: Text(
          '${height.toInt()}px',
          style: const TextStyle(color: Colors.white),
        ),
      ),
    );
  }
}
```

What's happening here?
- Masonry layout with variable heights
- Items placed in shortest column
- Custom layout algorithm
- Full control over positioning

---

# Custom Layout Example: Radial Layout

> **Creating a radial/circular layout.**

```dart
// 1. Radial widget
class RadialLayout extends MultiChildRenderObjectWidget {
  const RadialLayout({
    super.key,
    required super.children,
    this.radius = 100,
    this.startAngle = 0,
  });

  final double radius;
  final double startAngle;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return RadialRenderObject(
      radius: radius,
      startAngle: startAngle,
    );
  }

  @override
  void updateRenderObject(
    BuildContext context,
    RadialRenderObject renderObject,
  ) {
    renderObject
      ..radius = radius
      ..startAngle = startAngle;
  }
}

// 2. Radial render object
class RadialRenderObject extends RenderBox
    with ContainerRenderObjectMixin<RenderBox, BoxParentData>,
         RenderBoxContainerDefaultsMixin<RenderBox, BoxParentData> {
  
  RadialRenderObject({
    required double radius,
    required double startAngle,
  }) : _radius = radius,
       _startAngle = startAngle;

  double _radius;
  set radius(double value) {
    if (_radius == value) return;
    _radius = value;
    markNeedsLayout();
  }

  double _startAngle;
  set startAngle(double value) {
    if (_startAngle == value) return;
    _startAngle = value;
    markNeedsLayout();
  }

  @override
  void setupParentData(RenderObject child) {
    if (child.parentData is! BoxParentData) {
      child.parentData = BoxParentData();
    }
  }

  @override
  void performLayout() {
    final BoxConstraints constraints = this.constraints;
    
    // Calculate center
    final centerX = constraints.maxWidth / 2;
    final centerY = constraints.maxHeight / 2;
    
    // Layout each child
    RenderBox? child = firstChild;
    int index = 0;
    while (child != null) {
      // Layout child with loose constraints
      child.layout(
        BoxConstraints.loose(constraints),
        parentUsesSize: true,
      );
      
      // Calculate position on circle
      final angle = _startAngle + (index / childCount) * 2 * 3.14159;
      final x = centerX + _radius * cos(angle) - child.size.width / 2;
      final y = centerY + _radius * sin(angle) - child.size.height / 2;
      
      // Position child
      final BoxParentData childParentData = 
          child.parentData! as BoxParentData;
      childParentData.offset = Offset(x, y);
      
      child = childAfter(child);
      index++;
    }
    
    // Set own size
    size = constraints.constrain(const Size(
      constraints.maxWidth,
      constraints.maxHeight,
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

// 3. Using radial layout
class RadialExample extends StatelessWidget {
  const RadialExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 300,
      color: Colors.grey[200],
      child: RadialLayout(
        radius: 100,
        startAngle: 0,
        children: [
          _buildCircleItem(Colors.red, '1'),
          _buildCircleItem(Colors.green, '2'),
          _buildCircleItem(Colors.blue, '3'),
          _buildCircleItem(Colors.orange, '4'),
          _buildCircleItem(Colors.purple, '5'),
          _buildCircleItem(Colors.pink, '6'),
        ],
      ),
    );
  }

  Widget _buildCircleItem(Color color, String label) {
    return Container(
      width: 40,
      height: 40,
      decoration: BoxDecoration(
        color: color,
        shape: BoxShape.circle,
      ),
      child: Center(
        child: Text(
          label,
          style: const TextStyle(color: Colors.white),
        ),
      ),
    );
  }
}
```

What's happening here?
- Items positioned on a circle
- Custom positioning algorithm
- Trigonometric calculations
- Unique visual layout

---

# Custom Layout Example: Waterfall Grid

> **Creating a waterfall/pinterest grid with custom spacing.**

```dart
// Waterfall grid widget
class WaterfallGrid extends MultiChildRenderObjectWidget {
  const WaterfallGrid({
    super.key,
    required super.children,
    this.crossAxisCount = 2,
    this.mainAxisSpacing = 8,
    this.crossAxisSpacing = 8,
  });

  final int crossAxisCount;
  final double mainAxisSpacing;
  final double crossAxisSpacing;

  @override
  RenderObject createRenderObject(BuildContext context) {
    return WaterfallRenderObject(
      crossAxisCount: crossAxisCount,
      mainAxisSpacing: mainAxisSpacing,
      crossAxisSpacing: crossAxisSpacing,
    );
  }

  @override
  void updateRenderObject(
    BuildContext context,
    WaterfallRenderObject renderObject,
  ) {
    renderObject
      ..crossAxisCount = crossAxisCount
      ..mainAxisSpacing = mainAxisSpacing
      ..crossAxisSpacing = crossAxisSpacing;
  }
}

// Using waterfall grid
class WaterfallExample extends StatelessWidget {
  const WaterfallExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 400,
      color: Colors.grey[200],
      child: WaterfallGrid(
        crossAxisCount: 3,
        mainAxisSpacing: 8,
        crossAxisSpacing: 8,
        children: [
          _buildWaterfallItem(100, Colors.red),
          _buildWaterfallItem(150, Colors.green),
          _buildWaterfallItem(80, Colors.blue),
          _buildWaterfallItem(120, Colors.orange),
          _buildWaterfallItem(200, Colors.purple),
          _buildWaterfallItem(90, Colors.pink),
          _buildWaterfallItem(160, Colors.teal),
          _buildWaterfallItem(110, Colors.indigo),
        ],
      ),
    );
  }

  Widget _buildWaterfallItem(double height, Color color) {
    return Container(
      height: height,
      color: color,
      child: Center(
        child: Text(
          '${height.toInt()}px',
          style: const TextStyle(color: Colors.white),
        ),
      ),
    );
  }
}
```

What's happening here?
- Variable-height items in grid
- Optimal column placement
- Custom spacing control
- Pinterest-style layout

---

# Best Practices

## Use RenderObject Efficiently

```dart
// Good - Cache calculations
class CustomRenderObject extends RenderBox {
  List<Offset>? _cachedPositions;
  
  @override
  void performLayout() {
    if (_cachedPositions == null) {
      _cachedPositions = _calculatePositions();
    }
    // Use cached positions
  }
}
```

## Minimize Layout Work

```dart
// Good - Only relayout when needed
set spacing(double value) {
  if (_spacing == value) return;
  _spacing = value;
  markNeedsLayout(); // Only when value changes
}
```

## Use Mixins for Common Functionality

```dart
// Good - Use ContainerRenderObjectMixin
class CustomRenderObject extends RenderBox
    with ContainerRenderObjectMixin<RenderBox, BoxParentData>,
         RenderBoxContainerDefaultsMixin<RenderBox, BoxParentData> {
  // Gets many methods for free
}
```

---

# Summary

Custom layouts give you complete control over child positioning and sizing. Use MultiChildRenderObjectWidget and RenderObject to implement your own layout algorithms. This enables specialized layouts like masonry, radial, waterfall, and any custom arrangement you can imagine.

---

# Next Steps

- [Constraints](../framework-architecture/constraints.md)
- [Layout Process](../framework-architecture/layout-process.md)
- [Rendering Pipeline](../framework-architecture/rendering-pipeline.md)

---

# Did You Know?

- Custom layouts use RenderObject
- ContainerRenderObjectMixin manages children
- performLayout handles sizing and positioning
- paint handles visual rendering
- markNeedsLayout triggers relayout
- Custom layouts can optimize performance
- Layout algorithms can be complex
- Custom layouts enable unique designs