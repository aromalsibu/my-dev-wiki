# Flex Layout

Understand how flex layout works in Flutter for responsive designs.

---

# What is it?

Flex layout is a system for arranging widgets in a flexible, responsive way. It allows widgets to expand, shrink, and distribute space within a container. The flex layout in Flutter uses Row and Column widgets with Expanded and Flexible children to create adaptive layouts that respond to different screen sizes.

---

# Why does it exist?

Flex layout exists to:

- Create responsive layouts that adapt to screen size
- Distribute space evenly between widgets
- Allow widgets to expand and shrink flexibly
- Support complex UI arrangements
- Handle different device orientations
- Simplify responsive design
- Enable dynamic and adaptive interfaces

---

# Flex Basics

> **Row and Column** are the basic flex widgets.

```dart
// Basic flex layout
class BasicFlex extends StatelessWidget {
  const BasicFlex({super.key});

  @override
  Widget build(BuildContext context) {
    // Row - horizontal flex
    return Row(
      children: [
        Container(width: 50, color: Colors.red),
        Container(width: 50, color: Colors.green),
        Container(width: 50, color: Colors.blue),
      ],
    );
  }
}

// Column - vertical flex
class BasicColumn extends StatelessWidget {
  const BasicColumn({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Container(height: 50, color: Colors.red),
        Container(height: 50, color: Colors.green),
        Container(height: 50, color: Colors.blue),
      ],
    );
  }
}

// Flex properties:
// 1. direction - Axis.horizontal or Axis.vertical
// 2. mainAxisAlignment - How to align along main axis
// 3. crossAxisAlignment - How to align along cross axis
// 4. children - List of widgets
// 5. mainAxisSize - Min or max space
```

What's happening here?
- Row arranges children horizontally
- Column arranges children vertically
- Children are placed in order
- Main axis is direction of layout
- Cross axis is perpendicular

---

# MainAxisAlignment

> **MainAxisAlignment** controls alignment along the main axis.

```dart
// MainAxisAlignment options
class MainAxisAlignmentExample extends StatelessWidget {
  const MainAxisAlignmentExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Start - Align to start
        Row(
          mainAxisAlignment: MainAxisAlignment.start,
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            Container(width: 50, height: 50, color: Colors.green),
          ],
        ),
        
        // 2. Center - Align to center
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            Container(width: 50, height: 50, color: Colors.green),
          ],
        ),
        
        // 3. End - Align to end
        Row(
          mainAxisAlignment: MainAxisAlignment.end,
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            Container(width: 50, height: 50, color: Colors.green),
          ],
        ),
        
        // 4. SpaceBetween - Space between children
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            Container(width: 50, height: 50, color: Colors.green),
            Container(width: 50, height: 50, color: Colors.blue),
          ],
        ),
        
        // 5. SpaceAround - Space around children
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceAround,
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            Container(width: 50, height: 50, color: Colors.green),
            Container(width: 50, height: 50, color: Colors.blue),
          ],
        ),
        
        // 6. SpaceEvenly - Even space between and around
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            Container(width: 50, height: 50, color: Colors.green),
            Container(width: 50, height: 50, color: Colors.blue),
          ],
        ),
      ],
    );
  }
}

// Visual comparison:
// start:    [1][2]
// center:     [1][2]
// end:       [1][2]
// spaceBetween: [1]   [2]   [3]
// spaceAround:   [1]   [2]   [3]
// spaceEvenly:   [1]   [2]   [3]
```

What's happening here?
- start aligns children to beginning
- center centers children
- end aligns children to end
- spaceBetween puts space between children
- spaceAround puts space around children
- spaceEvenly distributes space evenly

---

# CrossAxisAlignment

> **CrossAxisAlignment** controls alignment along the cross axis.

```dart
// CrossAxisAlignment options
class CrossAxisAlignmentExample extends StatelessWidget {
  const CrossAxisAlignmentExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 200,
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Container(width: 50, height: 50, color: Colors.red),
          Container(width: 50, height: 100, color: Colors.green),
          Container(width: 50, height: 75, color: Colors.blue),
        ],
      ),
    );
  }
}

// CrossAxisAlignment options:
// 1. CrossAxisAlignment.start - Align to top
// 2. CrossAxisAlignment.center - Align to center
// 3. CrossAxisAlignment.end - Align to bottom
// 4. CrossAxisAlignment.stretch - Stretch to fill
// 5. CrossAxisAlignment.baseline - Align by text baseline

// Example with different cross alignments:
class CrossAlignmentComparison extends StatelessWidget {
  const CrossAlignmentComparison({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Start
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Row(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Container(width: 50, height: 30, color: Colors.red),
              Container(width: 50, height: 50, color: Colors.green),
              Container(width: 50, height: 40, color: Colors.blue),
            ],
          ),
        ),
        
        // 2. Center
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Row(
            crossAxisAlignment: CrossAxisAlignment.center,
            children: [
              Container(width: 50, height: 30, color: Colors.red),
              Container(width: 50, height: 50, color: Colors.green),
              Container(width: 50, height: 40, color: Colors.blue),
            ],
          ),
        ),
        
        // 3. End
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Row(
            crossAxisAlignment: CrossAxisAlignment.end,
            children: [
              Container(width: 50, height: 30, color: Colors.red),
              Container(width: 50, height: 50, color: Colors.green),
              Container(width: 50, height: 40, color: Colors.blue),
            ],
          ),
        ),
        
        // 4. Stretch
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Row(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              Container(width: 50, color: Colors.red),
              Container(width: 50, color: Colors.green),
              Container(width: 50, color: Colors.blue),
            ],
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- start aligns to beginning of cross axis
- center centers on cross axis
- end aligns to end of cross axis
- stretch fills cross axis space
- baseline aligns by text baseline

---

# Expanded and Flexible

> **Expanded and Flexible** control how children take space.

```dart
// Expanded - Takes all available space
class ExpandedExample extends StatelessWidget {
  const ExpandedExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        // Fixed width
        Container(
          width: 50,
          color: Colors.red,
          child: const Text('Fixed'),
        ),
        
        // Takes remaining space
        Expanded(
          child: Container(
            color: Colors.blue,
            child: const Text('Expanded'),
          ),
        ),
        
        // Also takes remaining space
        Expanded(
          child: Container(
            color: Colors.green,
            child: const Text('Expanded'),
          ),
        ),
      ],
    );
  }
}

// Flexible - Flexible with limits
class FlexibleExample extends StatelessWidget {
  const FlexibleExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        // Flexible with tight fit
        Flexible(
          fit: FlexFit.tight,
          child: Container(
            color: Colors.red,
            child: const Text('Tight'),
          ),
        ),
        
        // Flexible with loose fit
        Flexible(
          fit: FlexFit.loose,
          child: Container(
            color: Colors.blue,
            child: const Text('Loose'),
          ),
        ),
      ],
    );
  }
}

// Expanded vs Flexible:
// Expanded: Always takes all available space
// Flexible: Can take space, but has limits
// Expanded = Flexible(fit: FlexFit.tight)
```

What's happening here?
- Expanded forces child to fill remaining space
- Flexible allows child to flex but has limits
- Tight fit forces child to fill
- Loose fit allows child to size itself
- Expanded is commonly used for responsive layouts

---

# Flex Factors

> **Flex factors** control proportional space distribution.

```dart
// Flex factors example
class FlexFactorsExample extends StatelessWidget {
  const FlexFactorsExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Equal flex factors (1:1:1)
        Row(
          children: [
            Expanded(
              flex: 1,
              child: Container(
                color: Colors.red,
                height: 50,
                child: const Center(child: Text('1')),
              ),
            ),
            Expanded(
              flex: 1,
              child: Container(
                color: Colors.green,
                height: 50,
                child: const Center(child: Text('1')),
              ),
            ),
            Expanded(
              flex: 1,
              child: Container(
                color: Colors.blue,
                height: 50,
                child: const Center(child: Text('1')),
              ),
            ),
          ],
        ),
        
        const SizedBox(height: 10),
        
        // Different flex factors (1:2:3)
        Row(
          children: [
            Expanded(
              flex: 1,
              child: Container(
                color: Colors.red,
                height: 50,
                child: const Center(child: Text('1')),
              ),
            ),
            Expanded(
              flex: 2,
              child: Container(
                color: Colors.green,
                height: 50,
                child: const Center(child: Text('2')),
              ),
            ),
            Expanded(
              flex: 3,
              child: Container(
                color: Colors.blue,
                height: 50,
                child: const Center(child: Text('3')),
              ),
            ),
          ],
        ),
        
        const SizedBox(height: 10),
        
        // Mixed fixed and flexible
        Row(
          children: [
            Container(
              width: 80,
              color: Colors.red,
              height: 50,
              child: const Center(child: Text('Fixed')),
            ),
            Expanded(
              flex: 2,
              child: Container(
                color: Colors.green,
                height: 50,
                child: const Center(child: Text('Flex 2')),
              ),
            ),
            Expanded(
              flex: 1,
              child: Container(
                color: Colors.blue,
                height: 50,
                child: const Center(child: Text('Flex 1')),
              ),
            ),
          ],
        ),
      ],
    );
  }
}

// Flex factor calculation:
// Total flex = sum of all flex factors
// Space per flex = available space / total flex
// Child space = flex factor * space per flex

// Example: flex factors 1,2,3
// Total flex = 6
// Space per flex = 100% / 6 = 16.67%
// Child 1 = 16.67%
// Child 2 = 33.33%
// Child 3 = 50%
```

What's happening here?
- Flex factors distribute space proportionally
- Higher flex = more space
- Total flex is sum of all factors
- Available space divided by total flex
- Children get space proportionally

---

# Intrinsic Sizing

> **Intrinsic sizing** constrains children naturally.

```dart
// Intrinsic sizing examples
class IntrinsicSizingExample extends StatelessWidget {
  const IntrinsicSizingExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Without intrinsic sizing
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Container(
                width: 100,
                height: 100,
                color: Colors.red,
                child: const Text('Fixed'),
              ),
              Container(
                color: Colors.blue,
                child: const Text(
                  'Long text that determines width',
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // With intrinsic sizing
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              IntrinsicWidth(
                child: Container(
                  height: 100,
                  color: Colors.red,
                  child: const Text('Fixed'),
                ),
              ),
              Expanded(
                child: Container(
                  color: Colors.blue,
                  child: const Text(
                    'Long text that determines width',
                  ),
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

// Intrinsic widgets:
// 1. IntrinsicWidth - Sizes to child's intrinsic width
// 2. IntrinsicHeight - Sizes to child's intrinsic height
// 3. IntrinsicDimension - Custom intrinsic sizing

// Use cases:
// 1. When children have different intrinsic sizes
// 2. When you want children to be sized by content
// 3. When you need consistent sizing
```

What's happening here?
- IntrinsicWidth sizes to content width
- IntrinsicHeight sizes to content height
- Useful for consistent sizing
- Can affect performance with complex layouts

---

# Responsive Flex Layouts

> **Making flex layouts** responsive to screen size.

```dart
// Responsive flex layout
class ResponsiveFlexLayout extends StatelessWidget {
  const ResponsiveFlexLayout({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // 1. Check available space
        final isWide = constraints.maxWidth > 600;
        
        // 2. Choose layout based on width
        if (isWide) {
          // Wide layout - row
          return Row(
            children: [
              Expanded(
                flex: 2,
                child: Container(
                  color: Colors.blue,
                  padding: const EdgeInsets.all(20),
                  child: const Text('Sidebar'),
                ),
              ),
              Expanded(
                flex: 5,
                child: Container(
                  color: Colors.grey[200],
                  padding: const EdgeInsets.all(20),
                  child: const Text('Content'),
                ),
              ),
            ],
          );
        } else {
          // Narrow layout - column
          return Column(
            children: [
              Container(
                width: double.infinity,
                color: Colors.blue,
                padding: const EdgeInsets.all(20),
                child: const Text('Header'),
              ),
              Expanded(
                child: Container(
                  width: double.infinity,
                  color: Colors.grey[200],
                  padding: const EdgeInsets.all(20),
                  child: const Text('Content'),
                ),
              ),
            ],
          );
        }
      },
    );
  }
}

// Adaptive flex layout
class AdaptiveFlexLayout extends StatelessWidget {
  const AdaptiveFlexLayout({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Wrap for adaptive sizing
        Wrap(
          spacing: 8,
          runSpacing: 8,
          children: [
            _buildChip('Android'),
            _buildChip('iOS'),
            _buildChip('Web'),
            _buildChip('Desktop'),
            _buildChip('Mobile'),
            _buildChip('Flutter'),
          ],
        ),
        
        const SizedBox(height: 20),
        
        // Flexible grid
        LayoutBuilder(
          builder: (context, constraints) {
            final crossAxisCount = (constraints.maxWidth / 150).floor();
            return GridView.builder(
              shrinkWrap: true,
              gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: crossAxisCount.clamp(2, 4),
                crossAxisSpacing: 8,
                mainAxisSpacing: 8,
              ),
              itemCount: 6,
              itemBuilder: (context, index) {
                return Container(
                  color: Colors.blue[100 * (index + 1)],
                  child: Center(
                    child: Text('Item $index'),
                  ),
                );
              },
            );
          },
        ),
      ],
    );
  }

  Widget _buildChip(String label) {
    return Chip(
      label: Text(label),
      backgroundColor: Colors.blue[100],
    );
  }
}
```

What's happening here?
- LayoutBuilder for responsive decisions
- Wrap for adaptive content
- Dynamic grid based on width
- Different layouts for different screen sizes
- Flexible and adaptive design

---

# Best Practices

## Use Appropriate Alignment

```dart
// Good - Using alignment
@override
Widget build(BuildContext context) {
  return Row(
    mainAxisAlignment: MainAxisAlignment.spaceEvenly,
    children: [
      Icon(Icons.home),
      Icon(Icons.search),
      Icon(Icons.person),
    ],
  );
}

// Bad - Using hardcoded spacing
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      Icon(Icons.home),
      const SizedBox(width: 20),
      Icon(Icons.search),
      const SizedBox(width: 20),
      Icon(Icons.person),
    ],
  );
}
```

## Use Expanded Wisely

```dart
// Good - Using Expanded
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      const Icon(Icons.star),
      const SizedBox(width: 8),
      Expanded(
        child: Text('Very long text that will expand'),
      ),
    ],
  );
}

// Bad - Not using Expanded
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      const Icon(Icons.star),
      const SizedBox(width: 8),
      Text('Very long text that might overflow'), // Could overflow
    ],
  );
}
```

## Keep Flex Trees Shallow

```dart
// Good - Shallow flex tree
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      Expanded(child: Child1()),
      Expanded(child: Child2()),
      Expanded(child: Child3()),
    ],
  );
}

// Bad - Deep flex tree
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      Row(
        children: [
          Column(
            children: [
              Row(
                children: [
                  // Very deep
                ],
              ),
            ],
          ),
        ],
      ),
    ],
  );
}
```

---

# Common Mistakes

## Overusing Expanded

Wrong:
```dart
// Every child expanded
Row(
  children: [
    Expanded(child: Container(color: Colors.red)),
    Expanded(child: Container(color: Colors.green)),
    Expanded(child: Container(color: Colors.blue)),
  ],
)
```

Correct:
```dart
// Use flex factors for proportional sizes
Row(
  children: [
    Expanded(
      flex: 1,
      child: Container(color: Colors.red),
    ),
    Expanded(
      flex: 2,
      child: Container(color: Colors.green),
    ),
    Expanded(
      flex: 3,
      child: Container(color: Colors.blue),
    ),
  ],
)
```

## Wrong CrossAxisAlignment

Wrong:
```dart
// Row with unexpected heights
Row(
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [
    Container(height: 50, color: Colors.red),
    Container(height: 100, color: Colors.green), // Different height
    Container(height: 30, color: Colors.blue),
  ],
)
```

Correct:
```dart
// Consistent heights or use stretch
Row(
  crossAxisAlignment: CrossAxisAlignment.stretch,
  children: [
    Container(width: 50, color: Colors.red),
    Container(width: 50, color: Colors.green),
    Container(width: 50, color: Colors.blue),
  ],
)
```

---

# Summary

Flex layout in Flutter uses Row and Column widgets with Expanded and Flexible children to create responsive layouts. Understanding mainAxisAlignment, crossAxisAlignment, and flex factors helps build adaptive designs. Use flex layouts for dynamic, responsive UIs.

---

# Next Steps

- [Row](row.md)
- [Column](column.md)
- [Flex](flex.md)

---

# Did You Know?

- Row and Column are both Flex widgets
- Expanded forces children to fill available space
- Flex factors distribute space proportionally
- CrossAxisAlignment controls vertical alignment in Row
- MainAxisAlignment controls horizontal alignment in Row
- Wrap is useful for responsive chip layouts
- Flex layouts are used in most UI designs
- Intrinsic sizing can optimize layouts