# Expanded

Understand how to make widgets fill available space in flex layouts.

---

# What is it?

Expanded is a widget that expands a child of a Row, Column, or Flex to fill the available space along the main axis. It's used to make widgets take up remaining space proportionally. Expanded is essential for creating responsive layouts where widgets need to share or fill available space.

---

# Why does it exist?

Expanded exists to:

- Fill available space in flex layouts
- Create proportional layouts
- Make widgets responsive
- Share space between children
- Simplify flexible layouts
- Handle different screen sizes
- Create adaptive UIs

---

# Basic Expanded

> **Expanded** fills available space in a flex container.

```dart
// Basic Expanded usage
class BasicExpanded extends StatelessWidget {
  const BasicExpanded({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Expanded in Row
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Row(
            children: [
              Container(
                width: 50,
                color: Colors.red,
              ),
              Expanded(
                child: Container(
                  color: Colors.blue,
                  child: const Center(child: Text('Expanded')),
                ),
              ),
              Container(
                width: 50,
                color: Colors.green,
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 20),
        
        // 2. Expanded in Column
        Container(
          height: 200,
          color: Colors.grey[200],
          child: Column(
            children: [
              Container(
                height: 50,
                color: Colors.red,
              ),
              Expanded(
                child: Container(
                  color: Colors.blue,
                  child: const Center(child: Text('Expanded')),
                ),
              ),
              Container(
                height: 50,
                color: Colors.green,
              ),
            ],
          ),
        ),
      ],
    );
  }
}

// Expanded properties:
// 1. flex - How much space to take (default: 1)
// 2. child - The widget to expand
```

What's happening here?
- Expanded fills remaining space
- Works in Row, Column, Flex
- Child expands to fill space
- Multiple Expanded share space
- flex controls proportion

---

# Flex Factors

> **Flex factors** control space distribution.

```dart
// Flex factor examples
class FlexFactorExample extends StatelessWidget {
  const FlexFactorExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Equal flex (1:1:1)
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Expanded(
                flex: 1,
                child: Container(
                  color: Colors.red,
                  child: const Center(child: Text('1')),
                ),
              ),
              Expanded(
                flex: 1,
                child: Container(
                  color: Colors.green,
                  child: const Center(child: Text('1')),
                ),
              ),
              Expanded(
                flex: 1,
                child: Container(
                  color: Colors.blue,
                  child: const Center(child: Text('1')),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Different flex (1:2:3)
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Expanded(
                flex: 1,
                child: Container(
                  color: Colors.red,
                  child: const Center(child: Text('1')),
                ),
              ),
              Expanded(
                flex: 2,
                child: Container(
                  color: Colors.green,
                  child: const Center(child: Text('2')),
                ),
              ),
              Expanded(
                flex: 3,
                child: Container(
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
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Container(
                width: 80,
                color: Colors.red,
                child: const Center(child: Text('Fixed')),
              ),
              Expanded(
                flex: 2,
                child: Container(
                  color: Colors.green,
                  child: const Center(child: Text('Flex 2')),
                ),
              ),
              Expanded(
                flex: 1,
                child: Container(
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

// Flex factor calculation:
// Total flex = sum of all flex factors
// Space per flex = available space / total flex
// Child space = flex factor * space per flex
```

What's happening here?
- Higher flex = more space
- Equal flex = equal space
- Flex factors proportionally distribute
- Fixed widgets maintain their size
- Expanded fills remaining space

---

# Expanded vs Flexible

> **Comparing Expanded** with Flexible.

```dart
// Expanded vs Flexible comparison
class ExpandedVsFlexible extends StatelessWidget {
  const ExpandedVsFlexible({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Expanded - always fills space
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Expanded(
                child: Container(
                  color: Colors.red,
                  child: const Center(child: Text('Expanded')),
                ),
              ),
              Expanded(
                child: Container(
                  color: Colors.blue,
                  child: const Center(child: Text('Expanded')),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Flexible - can fit to content
        Container(
          height: 80,
          color: Colors.grey[200],
          child: Row(
            children: [
              Flexible(
                fit: FlexFit.loose,
                child: Container(
                  color: Colors.green,
                  child: const Text('Flexible Loose'),
                ),
              ),
              Flexible(
                fit: FlexFit.tight,
                child: Container(
                  color: Colors.orange,
                  child: const Text('Flexible Tight'),
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. When to use each
        Row(
          children: [
            // Expanded: Force fill
            Expanded(
              child: Container(
                color: Colors.purple,
                child: const Text('Expanded'),
              ),
            ),
            
            // Flexible: Allow natural size
            Flexible(
              fit: FlexFit.loose,
              child: Container(
                color: Colors.pink,
                child: const Text('Flexible'),
              ),
            ),
          ],
        ),
      ],
    );
  }
}

// Expanded = Flexible(fit: FlexFit.tight)
// Expanded always fills space
// Flexible can be loose or tight
// Use Expanded when you want to fill
// Use Flexible when you want flexibility
```

What's happening here?
- Expanded: always fills space
- Flexible: can fit to content
- Expanded = Flexible tight
- Flexible offers more control
- Choose based on behavior needed

---

# Expanded in Different Layouts

> **Expanded works** in various flex layouts.

```dart
// Expanded in different contexts
class ExpandedInLayouts extends StatelessWidget {
  const ExpandedInLayouts({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. In Row
        Container(
          height: 60,
          color: Colors.grey[200],
          child: Row(
            children: [
              const Text('Left'),
              Expanded(
                child: Container(
                  color: Colors.blue,
                  child: const Center(child: Text('Center')),
                ),
              ),
              const Text('Right'),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. In Column
        Container(
          height: 150,
          color: Colors.grey[200],
          child: Column(
            children: [
              const Text('Top'),
              Expanded(
                child: Container(
                  color: Colors.green,
                  child: const Center(child: Text('Middle')),
                ),
              ),
              const Text('Bottom'),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Nested Expanded
        Container(
          height: 150,
          color: Colors.grey[200],
          child: Row(
            children: [
              Expanded(
                flex: 2,
                child: Container(
                  color: Colors.red,
                  child: const Center(child: Text('Left')),
                ),
              ),
              Expanded(
                flex: 3,
                child: Column(
                  children: [
                    Expanded(
                      child: Container(
                        color: Colors.green,
                        child: const Center(child: Text('Top')),
                      ),
                    ),
                    Expanded(
                      child: Container(
                        color: Colors.blue,
                        child: const Center(child: Text('Bottom')),
                      ),
                    ),
                  ],
                ),
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
- Works in both Row and Column
- Can be nested
- Flexible in any direction
- Creates complex layouts
- Responsive and adaptive

---

# Real-World Examples

> **Common patterns** using Expanded.

```dart
// 1. App bar with title
class AppBarWithTitle extends StatelessWidget {
  const AppBarWithTitle({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 56,
      color: Colors.blue,
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Row(
        children: [
          const Icon(Icons.menu, color: Colors.white),
          const SizedBox(width: 16),
          // Title expands to fill center space
          Expanded(
            child: const Text(
              'App Title',
              style: TextStyle(
                color: Colors.white,
                fontSize: 20,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          const Icon(Icons.search, color: Colors.white),
          const SizedBox(width: 16),
          const Icon(Icons.more_vert, color: Colors.white),
        ],
      ),
    );
  }
}

// 2. Login screen with flexible layout
class LoginScreen extends StatelessWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // Logo takes some space
          const SizedBox(height: 60),
          const Icon(Icons.flutter_dash, size: 80, color: Colors.blue),
          const SizedBox(height: 20),
          const Text(
            'Welcome',
            style: TextStyle(
              fontSize: 28,
              fontWeight: FontWeight.bold,
            ),
          ),
          
          // Form takes remaining space
          Expanded(
            child: Padding(
              padding: const EdgeInsets.all(24),
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const TextField(
                    decoration: InputDecoration(
                      labelText: 'Email',
                      border: OutlineInputBorder(),
                    ),
                  ),
                  const SizedBox(height: 16),
                  const TextField(
                    decoration: InputDecoration(
                      labelText: 'Password',
                      border: OutlineInputBorder(),
                    ),
                    obscureText: true,
                  ),
                  const SizedBox(height: 24),
                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton(
                      onPressed: () {},
                      child: const Text('Login'),
                    ),
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

// 3. Card with expandable content
class ExpandableCard extends StatelessWidget {
  const ExpandableCard({super.key});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: Column(
        children: [
          // Fixed header
          Container(
            padding: const EdgeInsets.all(16),
            color: Colors.blue,
            child: const Row(
              children: [
                Icon(Icons.info, color: Colors.white),
                SizedBox(width: 8),
                Text(
                  'Card Header',
                  style: TextStyle(
                    color: Colors.white,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
          ),
          
          // Expandable content
          Expanded(
            child: Container(
              padding: const EdgeInsets.all(16),
              child: const Text(
                'This content expands to fill available space. '
                'It will take up all remaining space in the card.',
              ),
            ),
          ),
          
          // Fixed footer
          Container(
            padding: const EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.end,
              children: [
                TextButton(
                  onPressed: () {},
                  child: const Text('Cancel'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () {},
                  child: const Text('Save'),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// 4. Equal-height columns with Expanded
class EqualHeightColumns extends StatelessWidget {
  const EqualHeightColumns({super.key});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: 200,
      child: Row(
        children: [
          // Each column has equal height
          Expanded(
            child: Container(
              color: Colors.red[100],
              padding: const EdgeInsets.all(16),
              child: Column(
                children: [
                  const Icon(Icons.star, size: 40),
                  const SizedBox(height: 8),
                  const Text('Column 1'),
                  Expanded(
                    child: Container(
                      color: Colors.red[200],
                      child: const Center(
                        child: Text('Content 1'),
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
          const SizedBox(width: 4),
          Expanded(
            child: Container(
              color: Colors.green[100],
              padding: const EdgeInsets.all(16),
              child: Column(
                children: [
                  const Icon(Icons.favorite, size: 40),
                  const SizedBox(height: 8),
                  const Text('Column 2'),
                  Expanded(
                    child: Container(
                      color: Colors.green[200],
                      child: const Center(
                        child: Text('Content 2'),
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
          const SizedBox(width: 4),
          Expanded(
            child: Container(
              color: Colors.blue[100],
              padding: const EdgeInsets.all(16),
              child: Column(
                children: [
                  const Icon(Icons.person, size: 40),
                  const SizedBox(height: 8),
                  const Text('Column 3'),
                  Expanded(
                    child: Container(
                      color: Colors.blue[200],
                      child: const Center(
                        child: Text('Content 3'),
                      ),
                    ),
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
- App bar with centered title
- Login screen with flexible form
- Card with expandable content
- Equal-height columns with Expanded

---

# Best Practices

## Use Expanded for Remaining Space

```dart
// Good - Fills remaining space
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      Text('Fixed'),
      Expanded(
        child: Text('This expands'),
      ),
    ],
  );
}
```

## Use Flex Factors for Proportions

```dart
// Good - Proportional sizing
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      Expanded(
        flex: 2,
        child: Container(color: Colors.red),
      ),
      Expanded(
        flex: 1,
        child: Container(color: Colors.blue),
      ),
    ],
  );
}
```

## Avoid Overusing Expanded

```dart
// Good - Appropriate use
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      // Only one Expanded needed
      Text('Fixed'),
      Expanded(child: Text('Expands')),
      Text('Fixed'),
    ],
  );
}

// Bad - Unnecessary Expanded
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      Expanded(child: Text('All expanded')), // No need
      Expanded(child: Text('All expanded')), // No need
    ],
  );
}
```

---

# Common Mistakes

## Using Expanded Outside Flex

Wrong:
```dart
// Error: Expanded must be inside Flex
Container(
  child: Expanded(
    child: Text('Error'),
  ),
)
```

Correct:
```dart
// Expanded inside Row/Column/Flex
Row(
  children: [
    Expanded(
      child: Text('Correct'),
    ),
  ],
)
```

## Not Using flex Factor

Wrong:
```dart
// All equal without reason
Row(
  children: [
    Expanded(child: Text('Short')),
    Expanded(child: Text('Very long text')),
  ],
)
```

Correct:
```dart
// Use flex for proportions
Row(
  children: [
    Expanded(
      flex: 1,
      child: Text('Short'),
    ),
    Expanded(
      flex: 3,
      child: Text('Very long text'),
    ),
  ],
)
```

---

# Summary

Expanded fills available space in flex layouts. Use Expanded for remaining space, flex factors for proportions, and combine with fixed widgets for balanced layouts. Expanded is essential for creating responsive, adaptive UIs.

---

# Next Steps

- [Flexible](flexible.md)
- [Spacer](spacer.md)
- [AspectRatio](aspectratio.md)

---

# Did You Know?

- Expanded = Flexible(fit: FlexFit.tight)
- Multiple Expanded share space proportionally
- flex factor controls space distribution
- Expanded only works in Flex (Row/Column)
- Expanded fills remaining space after fixed children
- flex default is 1
- Expanded can be nested
- Expanded is essential for responsive layouts