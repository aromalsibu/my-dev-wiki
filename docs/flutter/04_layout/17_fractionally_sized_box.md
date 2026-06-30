# FractionallySizedBox

Understand how to size widgets as a fraction of available space.

---

# What is it?

FractionallySizedBox is a widget that sizes its child to a fraction of the available space. You can specify width and height factors (e.g., 0.5 for 50%, 0.75 for 75%) to create responsive widgets that scale with their parent. This is perfect for creating proportional layouts.

---

# Why does it exist?

FractionallySizedBox exists to:

- Size widgets as a percentage of parent
- Create responsive layouts
- Build proportional designs
- Simplify percentage-based sizing
- Handle different screen sizes
- Create adaptive UIs
- Replace complex calculations

---

# Basic FractionallySizedBox

> **FractionallySizedBox** sizes child as a fraction of parent.

```dart
// Basic FractionallySizedBox usage
class BasicFractionallySizedBox extends StatelessWidget {
  const BasicFractionallySizedBox({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 200,
      color: Colors.grey[200],
      child: Column(
        children: [
          // 1. 50% width and height
          FractionallySizedBox(
            widthFactor: 0.5,
            heightFactor: 0.5,
            child: Container(
              color: Colors.blue,
              child: const Center(child: Text('50%')),
            ),
          ),
          
          const SizedBox(height: 10),
          
          // 2. 75% width only
          FractionallySizedBox(
            widthFactor: 0.75,
            child: Container(
              color: Colors.green,
              height: 40,
              child: const Center(child: Text('75% width')),
            ),
          ),
          
          const SizedBox(height: 10),
          
          // 3. 50% height only
          FractionallySizedBox(
            heightFactor: 0.5,
            child: Container(
              color: Colors.red,
              child: const Center(child: Text('50% height')),
            ),
          ),
        ],
      ),
    );
  }
}

// FractionallySizedBox properties:
// 1. widthFactor - Fraction of parent width (0.0 to 1.0)
// 2. heightFactor - Fraction of parent height (0.0 to 1.0)
// 3. child - The widget to size
// 4. alignment - How to align child (default: center)
```

What's happening here?
- widthFactor: fraction of parent width
- heightFactor: fraction of parent height
- Child fills the sized box
- Responsive to parent size
- Values from 0.0 to 1.0

---

# Size Factors Explained

> **Understanding** widthFactor and heightFactor.

```dart
// Size factor examples
class SizeFactorsExplained extends StatelessWidget {
  const SizeFactorsExplained({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 200,
      color: Colors.grey[200],
      child: Column(
        children: [
          // 1. widthFactor: 0.25 (25%)
          FractionallySizedBox(
            widthFactor: 0.25,
            heightFactor: 0.5,
            child: Container(
              color: Colors.red,
              child: const Center(child: Text('25%')),
            ),
          ),
          
          const SizedBox(height: 4),
          
          // 2. widthFactor: 0.5 (50%)
          FractionallySizedBox(
            widthFactor: 0.5,
            heightFactor: 0.5,
            child: Container(
              color: Colors.green,
              child: const Center(child: Text('50%')),
            ),
          ),
          
          const SizedBox(height: 4),
          
          // 3. widthFactor: 0.75 (75%)
          FractionallySizedBox(
            widthFactor: 0.75,
            heightFactor: 0.5,
            child: Container(
              color: Colors.blue,
              child: const Center(child: Text('75%')),
            ),
          ),
          
          const SizedBox(height: 4),
          
          // 4. widthFactor: 1.0 (100%)
          FractionallySizedBox(
            widthFactor: 1.0,
            heightFactor: 0.5,
            child: Container(
              color: Colors.orange,
              child: const Center(child: Text('100%')),
            ),
          ),
        ],
      ),
    );
  }
}

// Size factor values:
// 0.0 = 0% (invisible)
// 0.25 = 25%
// 0.5 = 50%
// 0.75 = 75%
// 1.0 = 100% (fills parent)
// null = use child's natural size
```

What's happening here?
- 0.0 = 0% of parent
- 0.5 = 50% of parent
- 1.0 = 100% of parent
- null = natural size
- Combined for proportional sizing

---

# Alignment in FractionallySizedBox

> **Alignment** controls where the child sits.

```dart
// Alignment with FractionallySizedBox
class AlignmentFractionallySizedBox extends StatelessWidget {
  const AlignmentFractionallySizedBox({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 200,
      color: Colors.grey[200],
      child: Stack(
        children: [
          // 1. Top-left aligned
          FractionallySizedBox(
            widthFactor: 0.5,
            heightFactor: 0.3,
            alignment: Alignment.topLeft,
            child: Container(
              color: Colors.red,
              child: const Center(child: Text('Top Left')),
            ),
          ),
          
          // 2. Center aligned
          FractionallySizedBox(
            widthFactor: 0.5,
            heightFactor: 0.3,
            alignment: Alignment.center,
            child: Container(
              color: Colors.green,
              child: const Center(child: Text('Center')),
            ),
          ),
          
          // 3. Bottom-right aligned
          FractionallySizedBox(
            widthFactor: 0.5,
            heightFactor: 0.3,
            alignment: Alignment.bottomRight,
            child: Container(
              color: Colors.blue,
              child: const Center(child: Text('Bottom Right')),
            ),
          ),
        ],
      ),
    );
  }
}

// Alignment options:
// 1. Alignment.topLeft
// 2. Alignment.topCenter
// 3. Alignment.topRight
// 4. Alignment.centerLeft
// 5. Alignment.center
// 6. Alignment.centerRight
// 7. Alignment.bottomLeft
// 8. Alignment.bottomCenter
// 9. Alignment.bottomRight
```

What's happening here?
- alignment controls child position
- Default is center
- Works like Align widget
- Multiple alignments in Stack

---

# FractionallySizedBox vs Other Widgets

> **Comparing** FractionallySizedBox with similar widgets.

```dart
// FractionallySizedBox vs others
class ComparisonFractionallySizedBox extends StatelessWidget {
  const ComparisonFractionallySizedBox({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 200,
      color: Colors.grey[200],
      child: Column(
        children: [
          // 1. FractionallySizedBox - Percentage of parent
          FractionallySizedBox(
            widthFactor: 0.5,
            heightFactor: 0.3,
            child: Container(
              color: Colors.blue,
              child: const Center(child: Text('50% x 30%')),
            ),
          ),
          
          const SizedBox(height: 4),
          
          // 2. SizedBox - Fixed size
          const SizedBox(
            width: 100,
            height: 30,
            child: ColoredBox(
              color: Colors.green,
              child: Center(child: Text('Fixed 100x30')),
            ),
          ),
          
          const SizedBox(height: 4),
          
          // 3. Expanded - Fills remaining space
          Row(
            children: [
              const Text('Left'),
              Expanded(
                child: Container(
                  color: Colors.red,
                  height: 30,
                  child: const Center(child: Text('Expanded')),
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }
}

// When to use:
// FractionallySizedBox: Percentage-based sizing
// SizedBox: Fixed sizing
// Expanded: Fill remaining space
// Flexible: Flexible sizing
```

What's happening here?
- FractionallySizedBox: percentage-based
- SizedBox: fixed size
- Expanded: fills remaining
- Flexible: flexible sizing

---

# Real-World Examples

> **Common patterns** using FractionallySizedBox.

```dart
// 1. Responsive profile header
class ResponsiveProfileHeader extends StatelessWidget {
  const ResponsiveProfileHeader({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.blue,
      child: Row(
        children: [
          // Avatar takes 20% of screen width
          FractionallySizedBox(
            widthFactor: 0.2,
            child: const CircleAvatar(
              radius: 30,
              backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
            ),
          ),
          const SizedBox(width: 16),
          // Name takes remaining space
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text(
                  'John Doe',
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                Text(
                  'Flutter Developer',
                  style: TextStyle(
                    color: Colors.white.withOpacity(0.8),
                    fontSize: 14,
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// 2. Progress bar
class ProgressBar extends StatelessWidget {
  const ProgressBar({
    super.key,
    required this.progress,
    required this.color,
  });

  final double progress;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 20,
      decoration: BoxDecoration(
        color: Colors.grey[200],
        borderRadius: BorderRadius.circular(10),
      ),
      child: FractionallySizedBox(
        widthFactor: progress.clamp(0.0, 1.0),
        child: Container(
          decoration: BoxDecoration(
            color: color,
            borderRadius: BorderRadius.circular(10),
          ),
          child: Center(
            child: Text(
              '${(progress * 100).round()}%',
              style: const TextStyle(
                color: Colors.white,
                fontSize: 12,
              ),
            ),
          ),
        ),
      ),
    );
  }
}

// 3. Responsive card grid
class ResponsiveCardGrid extends StatelessWidget {
  const ResponsiveCardGrid({super.key});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: 200,
      child: Row(
        children: [
          // Each card takes 30% of width
          ...List.generate(3, (index) {
            return FractionallySizedBox(
              widthFactor: 0.3,
              child: Card(
                margin: const EdgeInsets.symmetric(horizontal: 4),
                child: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(
                        Icons.star,
                        size: 40,
                        color: Colors.blue[100 * (index + 1)],
                      ),
                      const SizedBox(height: 8),
                      Text('Card ${index + 1}'),
                    ],
                  ),
                ),
              ),
            );
          }),
        ],
      ),
    );
  }
}

// 4. Centered button
class CenteredButton extends StatelessWidget {
  const CenteredButton({super.key});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: 100,
      child: FractionallySizedBox(
        widthFactor: 0.5,
        child: ElevatedButton(
          onPressed: () {},
          child: const Text('Centered Button'),
        ),
      ),
    );
  }
}
```

What's happening here?
- Responsive profile avatar
- Progress bar with percentage
- Responsive card grid
- Centered button with proportional width

---

# Best Practices

## Use for Responsive Sizing

```dart
// Good - Responsive sizing
@override
Widget build(BuildContext context) {
  return FractionallySizedBox(
    widthFactor: 0.8,
    child: Container(
      color: Colors.blue,
      child: const Text('80% width'),
    ),
  );
}
```

## Use with LayoutBuilder

```dart
// Good - Responsive with LayoutBuilder
@override
Widget build(BuildContext context) {
  return LayoutBuilder(
    builder: (context, constraints) {
      return FractionallySizedBox(
        widthFactor: constraints.maxWidth > 600 ? 0.5 : 0.9,
        child: Container(color: Colors.blue),
      );
    },
  );
}
```

## Use for Progress Indicators

```dart
// Good - Progress bar
@override
Widget build(BuildContext context) {
  return FractionallySizedBox(
    widthFactor: _progress,
    child: Container(color: Colors.blue),
  );
}
```

---

# Common Mistakes

## Values Outside 0-1 Range

Wrong:
```dart
// Values > 1.0 cause overflow
FractionallySizedBox(
  widthFactor: 1.5, // 150% - causes overflow
  child: Container(color: Colors.blue),
)
```

Correct:
```dart
// Values between 0 and 1
FractionallySizedBox(
  widthFactor: 0.8, // 80%
  child: Container(color: Colors.blue),
)
```

## No Parent Constraints

Wrong:
```dart
// No constraints
FractionallySizedBox(
  widthFactor: 0.5,
  child: Container(color: Colors.blue),
)
```

Correct:
```dart
// With parent constraints
Container(
  width: 300,
  child: FractionallySizedBox(
    widthFactor: 0.5,
    child: Container(color: Colors.blue),
  ),
)
```

---

# Summary

FractionallySizedBox sizes children as a percentage of available space. Use it for responsive layouts, progress bars, proportional sizing, and creating adaptive UIs. Values range from 0.0 to 1.0, and alignment controls child positioning.

---

# Next Steps

- [Intrinsic Widgets](intrinsic-widgets.md)
- [Custom Layouts](custom-layouts.md)
- [Layout Process](../framework-architecture/layout-process.md)

---

# Did You Know?

- FractionallySizedBox works with fractions
- 0.0 = 0%, 1.0 = 100%
- null uses natural child size
- Alignment controls child position
- FractionallySizedBox is responsive
- Perfect for progress indicators
- Works with any child widget
- Combines well with LayoutBuilder