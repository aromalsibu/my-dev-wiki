# Row

Understand how to use Row for horizontal layout.

---

# What is it?

Row is a widget that displays its children in a horizontal array. It's one of the most commonly used layout widgets in Flutter. Row arranges widgets from left to right (or right to left in RTL locales) and can control how they align, distribute space, and size themselves.

---

# Why does it exist?

Row exists to:

- Arrange widgets horizontally
- Create responsive horizontal layouts
- Control alignment and spacing
- Distribute available space
- Support scrolling when needed
- Build complex UI components
- Enable flexible and adaptive designs

---

# Basic Row

> **Row** arranges children horizontally.

```dart
// Basic Row usage
class BasicRow extends StatelessWidget {
  const BasicRow({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        // 1. Container with fixed size
        Container(
          width: 50,
          height: 50,
          color: Colors.red,
        ),
        
        // 2. Container with fixed size
        Container(
          width: 50,
          height: 50,
          color: Colors.green,
        ),
        
        // 3. Container with fixed size
        Container(
          width: 50,
          height: 50,
          color: Colors.blue,
        ),
      ],
    );
  }
}

// Row properties:
// 1. children - List of widgets
// 2. mainAxisAlignment - Horizontal alignment
// 3. crossAxisAlignment - Vertical alignment
// 4. mainAxisSize - Min or max space
// 5. textDirection - LTR or RTL
// 6. verticalDirection - Up or down
// 7. textBaseline - Text baseline
```

What's happening here?
- Children are arranged left to right
- Each child has fixed size
- Row takes the height of tallest child
- Width is sum of children + spacing
- Can control alignment and sizing

---

# MainAxisAlignment

> **MainAxisAlignment** controls horizontal alignment.

```dart
// MainAxisAlignment examples
class RowMainAxisAlignment extends StatelessWidget {
  const RowMainAxisAlignment({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Start - Left aligned
        _buildRowWithTitle(
          'Start',
          mainAxisAlignment: MainAxisAlignment.start,
        ),
        
        // 2. Center - Center aligned
        _buildRowWithTitle(
          'Center',
          mainAxisAlignment: MainAxisAlignment.center,
        ),
        
        // 3. End - Right aligned
        _buildRowWithTitle(
          'End',
          mainAxisAlignment: MainAxisAlignment.end,
        ),
        
        // 4. SpaceBetween - Space between
        _buildRowWithTitle(
          'SpaceBetween',
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
        ),
        
        // 5. SpaceAround - Space around
        _buildRowWithTitle(
          'SpaceAround',
          mainAxisAlignment: MainAxisAlignment.spaceAround,
        ),
        
        // 6. SpaceEvenly - Even space
        _buildRowWithTitle(
          'SpaceEvenly',
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        ),
      ],
    );
  }

  Widget _buildRowWithTitle(
    String title, {
    required MainAxisAlignment mainAxisAlignment,
  }) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          title,
          style: const TextStyle(fontWeight: FontWeight.bold),
        ),
        Container(
          width: double.infinity,
          color: Colors.grey[200],
          child: Row(
            mainAxisAlignment: mainAxisAlignment,
            children: [
              Container(width: 40, height: 40, color: Colors.red),
              Container(width: 40, height: 40, color: Colors.green),
              Container(width: 40, height: 40, color: Colors.blue),
            ],
          ),
        ),
        const SizedBox(height: 10),
      ],
    );
  }
}
```

What's happening here?
- start aligns children to the left
- center centers children
- end aligns children to the right
- spaceBetween distributes space between
- spaceAround distributes space around
- spaceEvenly distributes space evenly

---

# CrossAxisAlignment

> **CrossAxisAlignment** controls vertical alignment.

```dart
// CrossAxisAlignment examples
class RowCrossAxisAlignment extends StatelessWidget {
  const RowCrossAxisAlignment({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Start - Align to top
        _buildRowWithTitle(
          'Start',
          crossAxisAlignment: CrossAxisAlignment.start,
        ),
        
        // 2. Center - Align to center
        _buildRowWithTitle(
          'Center',
          crossAxisAlignment: CrossAxisAlignment.center,
        ),
        
        // 3. End - Align to bottom
        _buildRowWithTitle(
          'End',
          crossAxisAlignment: CrossAxisAlignment.end,
        ),
        
        // 4. Stretch - Stretch to fill
        _buildRowWithTitle(
          'Stretch',
          crossAxisAlignment: CrossAxisAlignment.stretch,
        ),
        
        // 5. Baseline - Align by text baseline
        _buildRowWithTitle(
          'Baseline',
          crossAxisAlignment: CrossAxisAlignment.baseline,
          textBaseline: TextBaseline.alphabetic,
        ),
      ],
    );
  }

  Widget _buildRowWithTitle(
    String title, {
    required CrossAxisAlignment crossAxisAlignment,
    TextBaseline? textBaseline,
  }) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          title,
          style: const TextStyle(fontWeight: FontWeight.bold),
        ),
        Container(
          height: 100,
          width: double.infinity,
          color: Colors.grey[200],
          child: Row(
            crossAxisAlignment: crossAxisAlignment,
            textBaseline: textBaseline,
            children: [
              Container(width: 40, height: 30, color: Colors.red),
              Container(width: 40, height: 60, color: Colors.green),
              Container(width: 40, height: 45, color: Colors.blue),
            ],
          ),
        ),
        const SizedBox(height: 10),
      ],
    );
  }
}
```

What's happening here?
- start aligns children to the top
- center centers children vertically
- end aligns children to the bottom
- stretch stretches children to fill height
- baseline aligns by text baseline

---

# MainAxisSize

> **MainAxisSize** controls how much space Row takes.

```dart
// MainAxisSize examples
class RowMainAxisSize extends StatelessWidget {
  const RowMainAxisSize({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Min - Takes minimum space
        Container(
          color: Colors.grey[200],
          child: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Container(width: 50, height: 50, color: Colors.red),
              Container(width: 50, height: 50, color: Colors.green),
              Container(width: 50, height: 50, color: Colors.blue),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Max - Takes maximum space
        Container(
          color: Colors.grey[200],
          child: Row(
            mainAxisSize: MainAxisSize.max,
            children: [
              Container(width: 50, height: 50, color: Colors.red),
              Container(width: 50, height: 50, color: Colors.green),
              Container(width: 50, height: 50, color: Colors.blue),
            ],
          ),
        ),
      ],
    );
  }
}

// MainAxisSize effects:
// Min: Row is as wide as children + spacing
// Max: Row expands to fill parent width
// Default is MainAxisSize.max
// Use min for tight wrapping, max for full width
```

What's happening here?
- min takes minimum space needed
- max expands to fill available space
- Default is max
- min is useful for tight layouts

---

# Row with Expanded

> **Expanded** makes children fill available space.

```dart
// Row with Expanded children
class RowWithExpanded extends StatelessWidget {
  const RowWithExpanded({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Equal distribution
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Expanded(
                child: Container(
                  height: 50,
                  color: Colors.red,
                  child: const Center(child: Text('1')),
                ),
              ),
              Expanded(
                child: Container(
                  height: 50,
                  color: Colors.green,
                  child: const Center(child: Text('1')),
                ),
              ),
              Expanded(
                child: Container(
                  height: 50,
                  color: Colors.blue,
                  child: const Center(child: Text('1')),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Proportional distribution
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Expanded(
                flex: 1,
                child: Container(
                  height: 50,
                  color: Colors.red,
                  child: const Center(child: Text('1')),
                ),
              ),
              Expanded(
                flex: 2,
                child: Container(
                  height: 50,
                  color: Colors.green,
                  child: const Center(child: Text('2')),
                ),
              ),
              Expanded(
                flex: 3,
                child: Container(
                  height: 50,
                  color: Colors.blue,
                  child: const Center(child: Text('3')),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Mixed fixed and expanded
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Container(
                width: 80,
                height: 50,
                color: Colors.red,
                child: const Center(child: Text('Fixed')),
              ),
              Expanded(
                child: Container(
                  height: 50,
                  color: Colors.green,
                  child: const Center(child: Text('Expanded')),
                ),
              ),
              Container(
                width: 80,
                height: 50,
                color: Colors.blue,
                child: const Center(child: Text('Fixed')),
              ),
            ],
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- Expanded children fill remaining space
- Flex factors control proportion
- Fixed children keep their size
- Mixed layouts combine fixed and flexible
- Very common for responsive layouts

---

# Row with Flexible

> **Flexible** gives more control than Expanded.

```dart
// Row with Flexible children
class RowWithFlexible extends StatelessWidget {
  const RowWithFlexible({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Flexible with tight fit
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  height: 50,
                  color: Colors.red,
                  child: const Center(child: Text('Tight')),
                ),
              ),
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  height: 50,
                  color: Colors.green,
                  child: const Center(child: Text('Tight')),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Flexible with loose fit
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  height: 50,
                  color: Colors.red,
                  child: const Center(child: Text('Loose')),
                ),
              ),
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  height: 50,
                  color: Colors.green,
                  child: const Center(child: Text('Loose')),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Flexible with different flex values
        Container(
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                flex: 2,
                fit: FlexFit.tight,
                child: Container(
                  height: 50,
                  color: Colors.red,
                  child: const Center(child: Text('Flex 2')),
                ),
              ),
              Flexible(
                flex: 1,
                fit: FlexFit.tight,
                child: Container(
                  height: 50,
                  color: Colors.blue,
                  child: const Center(child: Text('Flex 1')),
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

// Expanded vs Flexible:
// Expanded = Flexible(fit: FlexFit.tight)
// Flexible allows more control over fitting behavior
// Loose fit allows child to size naturally
// Tight fit forces child to fill space
```

What's happening here?
- Flexible controls flex behavior
- Tight fit forces fill
- Loose fit allows natural size
- Flex factors work the same

---

# Row with Spacing

> **Adding spacing** between children.

```dart
// Row spacing options
class RowSpacing extends StatelessWidget {
  const RowSpacing({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Using SizedBox
        Row(
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            const SizedBox(width: 10),
            Container(width: 50, height: 50, color: Colors.green),
            const SizedBox(width: 20),
            Container(width: 50, height: 50, color: Colors.blue),
          ],
        ),
        
        const SizedBox(height: 10),
        
        // 2. Using Spacer
        Row(
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            const Spacer(),
            Container(width: 50, height: 50, color: Colors.green),
            const Spacer(),
            Container(width: 50, height: 50, color: Colors.blue),
          ],
        ),
        
        const SizedBox(height: 10),
        
        // 3. Using MainAxisAlignment
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            Container(width: 50, height: 50, color: Colors.green),
            Container(width: 50, height: 50, color: Colors.blue),
          ],
        ),
        
        const SizedBox(height: 10),
        
        // 4. Using Padding on children
        Row(
          children: [
            Padding(
              padding: const EdgeInsets.only(right: 10),
              child: Container(width: 50, height: 50, color: Colors.red),
            ),
            Padding(
              padding: const EdgeInsets.only(right: 10),
              child: Container(width: 50, height: 50, color: Colors.green),
            ),
            Container(width: 50, height: 50, color: Colors.blue),
          ],
        ),
      ],
    );
  }
}
```

What's happening here?
- SizedBox adds fixed spacing
- Spacer distributes flexible space
- MainAxisAlignment spaces automatically
- Padding adds spacing around children

---

# Real-World Examples

> **Common patterns** using Row.

```dart
// 1. App Bar
class AppBarRow extends StatelessWidget {
  const AppBarRow({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      color: Colors.blue,
      child: Row(
        children: [
          // Left: Back button
          const Icon(Icons.arrow_back, color: Colors.white),
          
          // Middle: Title
          const Expanded(
            child: Text(
              'App Title',
              style: TextStyle(
                color: Colors.white,
                fontSize: 20,
                fontWeight: FontWeight.bold,
              ),
              textAlign: TextAlign.center,
            ),
          ),
          
          // Right: Action buttons
          const Icon(Icons.search, color: Colors.white),
          const SizedBox(width: 8),
          const Icon(Icons.more_vert, color: Colors.white),
        ],
      ),
    );
  }
}

// 2. Card with icon and text
class CardWithRow extends StatelessWidget {
  const CardWithRow({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(8),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 8,
          ),
        ],
      ),
      child: Row(
        children: [
          // Icon
          Container(
            width: 50,
            height: 50,
            decoration: const BoxDecoration(
              color: Colors.blue,
              shape: BoxShape.circle,
            ),
            child: const Icon(
              Icons.person,
              color: Colors.white,
            ),
          ),
          
          const SizedBox(width: 16),
          
          // Text content
          const Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  'John Doe',
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    fontSize: 16,
                  ),
                ),
                Text(
                  'Flutter Developer',
                  style: TextStyle(
                    color: Colors.grey,
                    fontSize: 14,
                  ),
                ),
              ],
            ),
          ),
          
          // Action button
          const Icon(Icons.chevron_right, color: Colors.grey),
        ],
      ),
    );
  }
}

// 3. Button row
class ButtonRow extends StatelessWidget {
  const ButtonRow({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        Expanded(
          child: ElevatedButton(
            onPressed: () {},
            child: const Text('Cancel'),
          ),
        ),
        const SizedBox(width: 8),
        Expanded(
          child: ElevatedButton(
            onPressed: () {},
            style: ElevatedButton.styleFrom(
              backgroundColor: Colors.blue,
            ),
            child: const Text('Confirm'),
          ),
        ),
      ],
    );
  }
}

// 4. Header with badge
class HeaderWithBadge extends StatelessWidget {
  const HeaderWithBadge({super.key});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        // Left: Label
        const Text(
          'Notifications',
          style: TextStyle(
            fontSize: 18,
            fontWeight: FontWeight.bold,
          ),
        ),
        
        const Spacer(),
        
        // Right: Badge
        Container(
          padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
          decoration: BoxDecoration(
            color: Colors.red,
            borderRadius: BorderRadius.circular(12),
          ),
          child: const Text(
            '5 new',
            style: TextStyle(
              color: Colors.white,
              fontSize: 12,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- App bar with left, center, right sections
- Card with icon, text, and action
- Button row with equal buttons
- Header with badge counter

---

# Best Practices

## Use Expanded for Responsive Layout

```dart
// Good - Responsive
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      const Icon(Icons.star),
      const SizedBox(width: 8),
      Expanded(
        child: Text('Long text that adapts'),
      ),
    ],
  );
}

// Bad - Fixed width
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      const Icon(Icons.star),
      const SizedBox(width: 8),
      Container(
        width: 200, // May overflow on smaller screens
        child: const Text('Long text'),
      ),
    ],
  );
}
```

## Keep Row Children Consistent

```dart
// Good - Consistent heights
@override
Widget build(BuildContext context) {
  return Row(
    crossAxisAlignment: CrossAxisAlignment.center,
    children: [
      Container(height: 50, color: Colors.red),
      Container(height: 50, color: Colors.green),
      Container(height: 50, color: Colors.blue),
    ],
  );
}

// Bad - Inconsistent heights
@override
Widget build(BuildContext context) {
  return Row(
    crossAxisAlignment: CrossAxisAlignment.center,
    children: [
      Container(height: 30, color: Colors.red),
      Container(height: 60, color: Colors.green),
      Container(height: 45, color: Colors.blue),
    ],
  );
}
```

## Use Appropriate Spacing

```dart
// Good - Using MainAxisAlignment
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

// Bad - Hardcoded spacing
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

---

# Common Mistakes

## Overusing Expanded

Wrong:
```dart
// All children expanded
Row(
  children: [
    Expanded(child: Text('Long text')),
    Expanded(child: Text('Another text')),
    Expanded(child: Text('Third text')),
  ],
)
```

Correct:
```dart
// Use flex factors
Row(
  children: [
    Expanded(
      flex: 2,
      child: Text('Long text'),
    ),
    Expanded(
      flex: 1,
      child: Text('Another text'),
    ),
    Expanded(
      flex: 1,
      child: Text('Third text'),
    ),
  ],
)
```

## No Expanded for Long Text

Wrong:
```dart
// Text might overflow
Row(
  children: [
    Icon(Icons.star),
    Text('Very long text that might not fit'),
  ],
)
```

Correct:
```dart
// Use Expanded for text
Row(
  children: [
    Icon(Icons.star),
    Expanded(
      child: Text('Very long text that will fit'),
    ),
  ],
)
```

---

# Summary

Row arranges children horizontally with control over alignment, spacing, and sizing. Use Expanded for responsive layouts, MainAxisAlignment for spacing, and CrossAxisAlignment for vertical alignment. Row is essential for building horizontal UI components.

---

# Next Steps

- [Column](column.md)
- [Flex](flex.md)
- [Stack](stack.md)

---

# Did You Know?

- Row and Column are both Flex widgets
- Row takes the height of its tallest child
- CrossAxisAlignment controls vertical alignment
- Expanded and Flexible create responsive layouts
- MainAxisAlignment supports 6 alignment options
- Row can be nested inside Row
- Rows support RTL text direction