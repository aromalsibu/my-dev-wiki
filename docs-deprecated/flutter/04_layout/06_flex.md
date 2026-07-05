# Flex

Understand the Flex widget and how it provides flexible layouts.

---

# What is it?

Flex is the base widget that powers both Row and Column. It provides the core functionality for laying out children along a specific axis (horizontal or vertical) with flexible sizing, alignment, and spacing. While you typically use Row and Column directly, understanding Flex gives you deeper insight into Flutter's layout system and enables custom flex layouts.

---

# Why does it exist?

Flex exists to:

- Provide the foundation for Row and Column
- Enable custom axis layouts
- Control flex behavior and sizing
- Manage layout direction and alignment
- Support both horizontal and vertical layouts
- Enable responsive and adaptive designs
- Provide low-level flex layout control

---

# Flex Basics

> **Flex is the parent** of Row and Column.

```dart
// Flex vs Row/Column
class FlexBasics extends StatelessWidget {
  const FlexBasics({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Row uses Flex with horizontal direction
        Row(
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            Container(width: 50, height: 50, color: Colors.green),
          ],
        ),
        
        const SizedBox(height: 20),
        
        // 2. Column uses Flex with vertical direction
        Column(
          children: [
            Container(width: 50, height: 50, color: Colors.blue),
            Container(width: 50, height: 50, color: Colors.yellow),
          ],
        ),
        
        const SizedBox(height: 20),
        
        // 3. Flex directly with direction
        Flex(
          direction: Axis.horizontal,
          children: [
            Container(width: 50, height: 50, color: Colors.purple),
            Container(width: 50, height: 50, color: Colors.orange),
          ],
        ),
      ],
    );
  }
}

// Flex properties:
// 1. direction - Axis.horizontal or Axis.vertical
// 2. children - List of widgets
// 3. mainAxisAlignment - Alignment along main axis
// 4. crossAxisAlignment - Alignment along cross axis
// 5. mainAxisSize - Min or max space
// 6. textDirection - LTR or RTL
// 7. verticalDirection - Up or down
```

What's happening here?
- Row = Flex(direction: Axis.horizontal)
- Column = Flex(direction: Axis.vertical)
- Flex can be used directly for custom layouts
- All Row/Column properties work with Flex

---

# Custom Flex Layouts

> **Using Flex directly** for custom layouts.

```dart
// Custom flex layouts
class CustomFlexLayout extends StatelessWidget {
  const CustomFlexLayout({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Dynamic direction based on condition
        _buildDynamicFlex(),
        
        const SizedBox(height: 20),
        
        // 2. Custom spacing
        Flex(
          direction: Axis.horizontal,
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            Container(width: 50, height: 50, color: Colors.red),
            Container(width: 50, height: 50, color: Colors.green),
            Container(width: 50, height: 50, color: Colors.blue),
          ],
        ),
        
        const SizedBox(height: 20),
        
        // 3. Flex with children containing Expanded
        Flex(
          direction: Axis.horizontal,
          children: [
            Expanded(
              flex: 2,
              child: Container(color: Colors.red, height: 50),
            ),
            Expanded(
              flex: 1,
              child: Container(color: Colors.green, height: 50),
            ),
          ],
        ),
      ],
    );
  }

  Widget _buildDynamicFlex() {
    // Flexible layout based on condition
    final bool isHorizontal = true;
    
    return Flex(
      direction: isHorizontal ? Axis.horizontal : Axis.vertical,
      children: [
        Container(width: 50, height: 50, color: Colors.blue),
        Container(width: 50, height: 50, color: Colors.yellow),
      ],
    );
  }
}
```

What's happening here?
- Flex can change direction dynamically
- Control axis based on conditions
- Same properties as Row/Column
- More control over layout behavior

---

# FlexDirection

> **Direction determines** how children are arranged.

```dart
// Flex direction examples
class FlexDirectionExample extends StatelessWidget {
  const FlexDirectionExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Horizontal direction (Row)
        Container(
          color: Colors.grey[200],
          padding: const EdgeInsets.all(8),
          child: Flex(
            direction: Axis.horizontal,
            children: [
              Container(width: 40, height: 40, color: Colors.red),
              Container(width: 40, height: 40, color: Colors.green),
              Container(width: 40, height: 40, color: Colors.blue),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Vertical direction (Column)
        Container(
          color: Colors.grey[200],
          padding: const EdgeInsets.all(8),
          child: Flex(
            direction: Axis.vertical,
            children: [
              Container(width: 40, height: 40, color: Colors.red),
              Container(width: 40, height: 40, color: Colors.green),
              Container(width: 40, height: 40, color: Colors.blue),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Vertical direction with alignment
        Container(
          height: 150,
          color: Colors.grey[200],
          padding: const EdgeInsets.all(8),
          child: Flex(
            direction: Axis.vertical,
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Container(width: 40, height: 40, color: Colors.red),
              Container(width: 40, height: 40, color: Colors.green),
              Container(width: 40, height: 40, color: Colors.blue),
            ],
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- Horizontal = Row behavior
- Vertical = Column behavior
- Direction determines main axis
- Properties adjust accordingly

---

# Flex with Expanded

> **Expanded in Flex** controls space distribution.

```dart
// Flex with Expanded children
class FlexWithExpanded extends StatelessWidget {
  const FlexWithExpanded({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Horizontal flex
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Flex(
            direction: Axis.horizontal,
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
                flex: 1,
                child: Container(color: Colors.blue),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Vertical flex
        Container(
          height: 150,
          color: Colors.grey[200],
          child: Flex(
            direction: Axis.vertical,
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
                flex: 1,
                child: Container(color: Colors.blue),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Mixed fixed and flexible
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Flex(
            direction: Axis.horizontal,
            children: [
              Container(
                width: 50,
                color: Colors.red,
              ),
              Expanded(
                child: Container(color: Colors.green),
              ),
              Container(
                width: 50,
                color: Colors.blue,
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
- Expanded works with Flex
- Flex factors control proportion
- Works in both directions
- Mix fixed and flexible children

---

# Flex with Flexible

> **Flexible** provides more control than Expanded.

```dart
// Flex with Flexible children
class FlexWithFlexible extends StatelessWidget {
  const FlexWithFlexible({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Flexible with tight fit
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Flex(
            direction: Axis.horizontal,
            children: [
              Flexible(
                fit: FlexFit.tight,
                child: Container(color: Colors.red),
              ),
              Flexible(
                fit: FlexFit.tight,
                child: Container(color: Colors.green),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Flexible with loose fit
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Flex(
            direction: Axis.horizontal,
            children: [
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.red,
                  child: const Text('Loose'),
                ),
              ),
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.green,
                  child: const Text('Loose'),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Flexible with different flex values
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Flex(
            direction: Axis.horizontal,
            children: [
              Flexible(
                flex: 2,
                fit: FlexFit.tight,
                child: Container(color: Colors.red),
              ),
              Flexible(
                flex: 1,
                fit: FlexFit.tight,
                child: Container(color: Colors.blue),
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
- Tight fit forces fill
- Loose fit allows natural size
- Flex factors work the same
- More control than Expanded

---

# Flex Alignment

> **Alignment in Flex** controls child positioning.

```dart
// Flex alignment examples
class FlexAlignment extends StatelessWidget {
  const FlexAlignment({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Main alignment (horizontal)
        Container(
          color: Colors.grey[200],
          child: Flex(
            direction: Axis.horizontal,
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              Container(width: 40, height: 40, color: Colors.red),
              Container(width: 40, height: 40, color: Colors.green),
              Container(width: 40, height: 40, color: Colors.blue),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Cross alignment (vertical)
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Flex(
            direction: Axis.horizontal,
            crossAxisAlignment: CrossAxisAlignment.end,
            children: [
              Container(width: 40, height: 30, color: Colors.red),
              Container(width: 40, height: 60, color: Colors.green),
              Container(width: 40, height: 45, color: Colors.blue),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Both alignments
        Container(
          height: 150,
          color: Colors.grey[200],
          child: Flex(
            direction: Axis.vertical,
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.end,
            children: [
              Container(width: 30, height: 30, color: Colors.red),
              Container(width: 60, height: 30, color: Colors.green),
              Container(width: 90, height: 30, color: Colors.blue),
            ],
          ),
        ),
      ],
    );
  }
}
```

What's happening here?
- mainAxisAlignment along main axis
- crossAxisAlignment along cross axis
- Works in both directions
- Full control over positioning

---

# Flex in Real-World

> **Practical use cases** for Flex.

```dart
// 1. Responsive toolbar
class ResponsiveToolbar extends StatelessWidget {
  const ResponsiveToolbar({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // Switch between horizontal and vertical
        final isWide = constraints.maxWidth > 600;
        
        return Flex(
          direction: isWide ? Axis.horizontal : Axis.vertical,
          mainAxisAlignment: isWide 
              ? MainAxisAlignment.spaceBetween 
              : MainAxisAlignment.center,
          children: [
            const Text('Logo', style: TextStyle(fontSize: 24)),
            if (isWide) ...[
              TextButton(child: const Text('Home'), onPressed: () {}),
              TextButton(child: const Text('About'), onPressed: () {}),
              TextButton(child: const Text('Contact'), onPressed: () {}),
            ],
            IconButton(
              icon: const Icon(Icons.menu),
              onPressed: () {},
            ),
          ],
        );
      },
    );
  }
}

// 2. Card layout with flex
class CardWithFlex extends StatelessWidget {
  const CardWithFlex({super.key});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(16),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Flex(
          direction: Axis.horizontal,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Left: Icon
            Container(
              width: 60,
              height: 60,
              decoration: const BoxDecoration(
                color: Colors.blue,
                shape: BoxShape.circle,
              ),
              child: const Icon(Icons.person, color: Colors.white),
            ),
            const SizedBox(width: 16),
            
            // Right: Content
            Expanded(
              child: Flex(
                direction: Axis.vertical,
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    'John Doe',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    'Flutter Developer',
                    style: TextStyle(
                      color: Colors.grey[600],
                      fontSize: 14,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Flex(
                    direction: Axis.horizontal,
                    children: [
                      Icon(Icons.star, color: Colors.yellow[700]),
                      Icon(Icons.star, color: Colors.yellow[700]),
                      Icon(Icons.star, color: Colors.yellow[700]),
                      Icon(Icons.star, color: Colors.yellow[700]),
                      Icon(Icons.star, color: Colors.yellow[700]),
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

// 3. Dynamic flex grid
class DynamicFlexGrid extends StatelessWidget {
  const DynamicFlexGrid({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        final crossAxisCount = (constraints.maxWidth / 120).floor().clamp(2, 4);
        
        return Flex(
          direction: Axis.vertical,
          children: List.generate(3, (rowIndex) {
            return Flex(
              direction: Axis.horizontal,
              children: List.generate(crossAxisCount, (colIndex) {
                return Expanded(
                  child: Container(
                    margin: const EdgeInsets.all(4),
                    height: 100,
                    color: Colors.blue[100 * ((rowIndex + colIndex) % 9 + 1)],
                    child: Center(
                      child: Text('${rowIndex * crossAxisCount + colIndex + 1}'),
                    ),
                  ),
                );
              }),
            );
          }),
        );
      },
    );
  }
}
```

What's happening here?
- Responsive toolbar switching direction
- Card with mixed flex directions
- Dynamic grid with Flex
- Real-world flex usage

---

# Best Practices

## Use Flex for Dynamic Direction

```dart
// Good - Dynamic direction
@override
Widget build(BuildContext context) {
  final isHorizontal = MediaQuery.of(context).size.width > 600;
  
  return Flex(
    direction: isHorizontal ? Axis.horizontal : Axis.vertical,
    children: [
      Container(color: Colors.red),
      Container(color: Colors.green),
    ],
  );
}
```

## Keep Flex Trees Shallow

```dart
// Good - Shallow flex tree
@override
Widget build(BuildContext context) {
  return Flex(
    direction: Axis.horizontal,
    children: [
      Expanded(child: Container(color: Colors.red)),
      Expanded(child: Container(color: Colors.green)),
    ],
  );
}

// Bad - Deep flex tree
@override
Widget build(BuildContext context) {
  return Flex(
    direction: Axis.horizontal,
    children: [
      Flex(
        direction: Axis.vertical,
        children: [
          Flex(
            direction: Axis.horizontal,
            children: [
              // Very deep nesting
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

## Wrong Direction

Wrong:
```dart
// Using Flex horizontal for vertical layout
Flex(
  direction: Axis.horizontal,
  mainAxisAlignment: MainAxisAlignment.center,
  children: [
    Container(height: 50, color: Colors.red),
    Container(height: 50, color: Colors.green),
  ],
)
```

Correct:
```dart
// Use appropriate direction
Flex(
  direction: Axis.vertical,
  mainAxisAlignment: MainAxisAlignment.center,
  children: [
    Container(height: 50, color: Colors.red),
    Container(height: 50, color: Colors.green),
  ],
)
```

## Overusing Flex

Wrong:
```dart
// Using Flex when Row/Column would be clearer
Flex(
  direction: Axis.horizontal,
  children: [
    Container(color: Colors.red),
  ],
)
```

Correct:
```dart
// Use Row/Column for readability
Row(
  children: [
    Container(color: Colors.red),
  ],
)
```

---

# Summary

Flex is the foundation for Row and Column, providing flexible layout capabilities in any direction. Use Flex directly when you need dynamic direction or custom behavior. Understanding Flex helps build more flexible and responsive layouts.

---

# Next Steps

- [Stack](stack.md)
- [Wrap](wrap.md)
- [Flow](flow.md)

---

# Did You Know?

- Row = Flex(direction: Axis.horizontal)
- Column = Flex(direction: Axis.vertical)
- Flex can change direction dynamically
- All Row/Column properties work with Flex
- Flex is lower-level than Row/Column
- Flex supports custom flex layouts
- Flex is used throughout Flutter
- Understanding Flex helps debug layout issues