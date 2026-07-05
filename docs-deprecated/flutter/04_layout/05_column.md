# Column

Understand how to use Column for vertical layout.

---

# What is it?

Column is a widget that displays its children in a vertical array. It's one of the most commonly used layout widgets in Flutter. Column arranges widgets from top to bottom and can control how they align, distribute space, and size themselves. It is the vertical equivalent of the Row widget.

---

# Why does it exist?

Column exists to:

- Arrange widgets vertically
- Create responsive vertical layouts
- Control alignment and spacing
- Distribute available space
- Support scrolling when needed
- Build complex UI components
- Enable flexible and adaptive designs

---

# Basic Column

> **Column** arranges children vertically.

```dart
// Basic Column usage
class BasicColumn extends StatelessWidget {
  const BasicColumn({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
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

// Column properties:
// 1. children - List of widgets
// 2. mainAxisAlignment - Vertical alignment
// 3. crossAxisAlignment - Horizontal alignment
// 4. mainAxisSize - Min or max space
// 5. verticalDirection - Top to bottom or bottom to top
// 6. textBaseline - Text baseline
```

What's happening here?
- Children are arranged top to bottom
- Each child has fixed size
- Column takes the width of widest child
- Height is sum of children + spacing
- Can control alignment and sizing

---

# MainAxisAlignment

> **MainAxisAlignment** controls vertical alignment.

```dart
// MainAxisAlignment examples
class ColumnMainAxisAlignment extends StatelessWidget {
  const ColumnMainAxisAlignment({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Start - Top aligned
        _buildColumnWithTitle(
          'Start',
          mainAxisAlignment: MainAxisAlignment.start,
        ),
        
        // 2. Center - Center aligned
        _buildColumnWithTitle(
          'Center',
          mainAxisAlignment: MainAxisAlignment.center,
        ),
        
        // 3. End - Bottom aligned
        _buildColumnWithTitle(
          'End',
          mainAxisAlignment: MainAxisAlignment.end,
        ),
        
        // 4. SpaceBetween - Space between
        _buildColumnWithTitle(
          'SpaceBetween',
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
        ),
        
        // 5. SpaceAround - Space around
        _buildColumnWithTitle(
          'SpaceAround',
          mainAxisAlignment: MainAxisAlignment.spaceAround,
        ),
        
        // 6. SpaceEvenly - Even space
        _buildColumnWithTitle(
          'SpaceEvenly',
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        ),
      ],
    );
  }

  Widget _buildColumnWithTitle(
    String title, {
    required MainAxisAlignment mainAxisAlignment,
  }) {
    return Container(
      height: 150,
      color: Colors.grey[200],
      margin: const EdgeInsets.only(bottom: 8),
      child: Column(
        mainAxisAlignment: mainAxisAlignment,
        children: [
          Container(width: 40, height: 30, color: Colors.red),
          Container(width: 40, height: 30, color: Colors.green),
          Container(width: 40, height: 30, color: Colors.blue),
        ],
      ),
    );
  }
}
```

What's happening here?
- start aligns children to the top
- center centers children vertically
- end aligns children to the bottom
- spaceBetween distributes space between
- spaceAround distributes space around
- spaceEvenly distributes space evenly

---

# CrossAxisAlignment

> **CrossAxisAlignment** controls horizontal alignment.

```dart
// CrossAxisAlignment examples
class ColumnCrossAxisAlignment extends StatelessWidget {
  const ColumnCrossAxisAlignment({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 200,
      color: Colors.grey[200],
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Container(width: 30, height: 30, color: Colors.red),
          Container(width: 60, height: 30, color: Colors.green),
          Container(width: 90, height: 30, color: Colors.blue),
        ],
      ),
    );
  }
}

// CrossAxisAlignment options:
// 1. CrossAxisAlignment.start - Align to left
// 2. CrossAxisAlignment.center - Align to center
// 3. CrossAxisAlignment.end - Align to right
// 4. CrossAxisAlignment.stretch - Stretch to fill
// 5. CrossAxisAlignment.baseline - Align by text baseline
```

What's happening here?
- start aligns children to the left
- center centers children horizontally
- end aligns children to the right
- stretch stretches children to fill width
- baseline aligns by text baseline

---

# MainAxisSize

> **MainAxisSize** controls how much space Column takes.

```dart
// MainAxisSize examples
class ColumnMainAxisSize extends StatelessWidget {
  const ColumnMainAxisSize({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Min - Takes minimum space
        Container(
          color: Colors.grey[200],
          child: Column(
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
          height: 200,
          color: Colors.grey[200],
          child: Column(
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
// Min: Column is as tall as children + spacing
// Max: Column expands to fill parent height
// Default is MainAxisSize.max
// Use min for tight wrapping, max for full height
```

What's happening here?
- min takes minimum space needed
- max expands to fill available space
- Default is max
- min is useful for tight layouts

---

# Column with Expanded

> **Expanded** makes children fill available space.

```dart
// Column with Expanded children
class ColumnWithExpanded extends StatelessWidget {
  const ColumnWithExpanded({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 300,
      color: Colors.grey[200],
      child: Column(
        children: [
          // 1. Equal distribution
          Expanded(
            child: Container(
              width: double.infinity,
              color: Colors.red,
              child: const Center(child: Text('1')),
            ),
          ),
          Expanded(
            child: Container(
              width: double.infinity,
              color: Colors.green,
              child: const Center(child: Text('1')),
            ),
          ),
          Expanded(
            child: Container(
              width: double.infinity,
              color: Colors.blue,
              child: const Center(child: Text('1')),
            ),
          ),
        ],
      ),
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

# Real-World Examples

> **Common patterns** using Column.

```dart
// 1. Profile card
class ProfileCard extends StatelessWidget {
  const ProfileCard({super.key});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(16),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.center,
          children: [
            const CircleAvatar(
              radius: 50,
              backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
            ),
            const SizedBox(height: 12),
            const Text(
              'John Doe',
              style: TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 4),
            Text(
              'Flutter Developer',
              style: TextStyle(
                color: Colors.grey[600],
                fontSize: 16,
              ),
            ),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildStat('Posts', '150'),
                _buildStat('Followers', '2.5K'),
                _buildStat('Following', '350'),
              ],
            ),
            const SizedBox(height: 16),
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: () {},
                child: const Text('Follow'),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildStat(String label, String value) {
    return Column(
      children: [
        Text(
          value,
          style: const TextStyle(
            fontSize: 18,
            fontWeight: FontWeight.bold,
          ),
        ),
        Text(
          label,
          style: TextStyle(
            color: Colors.grey[600],
            fontSize: 14,
          ),
        ),
      ],
    );
  }
}

// 2. Login form
class LoginForm extends StatelessWidget {
  const LoginForm({super.key});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          const Text(
            'Welcome Back',
            style: TextStyle(
              fontSize: 28,
              fontWeight: FontWeight.bold,
            ),
            textAlign: TextAlign.center,
          ),
          const SizedBox(height: 8),
          Text(
            'Sign in to continue',
            style: TextStyle(
              color: Colors.grey[600],
              fontSize: 16,
            ),
            textAlign: TextAlign.center,
          ),
          const SizedBox(height: 32),
          const TextField(
            decoration: InputDecoration(
              labelText: 'Email',
              border: OutlineInputBorder(),
              prefixIcon: Icon(Icons.email),
            ),
          ),
          const SizedBox(height: 16),
          const TextField(
            decoration: InputDecoration(
              labelText: 'Password',
              border: OutlineInputBorder(),
              prefixIcon: Icon(Icons.lock),
            ),
            obscureText: true,
          ),
          const SizedBox(height: 24),
          ElevatedButton(
            onPressed: () {},
            style: ElevatedButton.styleFrom(
              padding: const EdgeInsets.symmetric(vertical: 16),
            ),
            child: const Text('Sign In'),
          ),
          const SizedBox(height: 16),
          TextButton(
            onPressed: () {},
            child: const Text('Forgot Password?'),
          ),
        ],
      ),
    );
  }
}

// 3. Article card
class ArticleCard extends StatelessWidget {
  const ArticleCard({super.key});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(8),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Image
          Container(
            height: 200,
            decoration: const BoxDecoration(
              image: DecorationImage(
                image: NetworkImage('https://example.com/image.jpg'),
                fit: BoxFit.cover,
              ),
              borderRadius: BorderRadius.only(
                topLeft: Radius.circular(4),
                topRight: Radius.circular(4),
              ),
            ),
          ),
          // Content
          Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  'Article Title',
                  style: const TextStyle(
                    fontSize: 20,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                Text(
                  'Lorem ipsum dolor sit amet, consectetur adipiscing elit. '
                  'Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.',
                  style: TextStyle(
                    color: Colors.grey[600],
                    fontSize: 14,
                    height: 1.5,
                  ),
                ),
                const SizedBox(height: 16),
                Row(
                  children: [
                    const CircleAvatar(
                      radius: 16,
                      backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
                    ),
                    const SizedBox(width: 8),
                    Text(
                      'John Doe',
                      style: TextStyle(
                        color: Colors.grey[700],
                        fontSize: 14,
                      ),
                    ),
                    const Spacer(),
                    Text(
                      '2 hours ago',
                      style: TextStyle(
                        color: Colors.grey[500],
                        fontSize: 12,
                      ),
                    ),
                  ],
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

What's happening here?
- Profile card with avatar and stats
- Login form with fields and buttons
- Article card with image and content
- Common real-world patterns

---

# Best Practices

## Use Expanded for Responsive Layout

```dart
// Good - Responsive
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      const Text('Header'),
      Expanded(
        child: Container(
          color: Colors.blue,
          child: const Text('Content that fills space'),
        ),
      ),
      const Text('Footer'),
    ],
  );
}

// Bad - Fixed height
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      const Text('Header'),
      Container(
        height: 200, // Fixed height may overflow
        color: Colors.blue,
        child: const Text('Content'),
      ),
      const Text('Footer'),
    ],
  );
}
```

## Keep Column Children Consistent

```dart
// Good - Consistent widths
@override
Widget build(BuildContext context) {
  return Column(
    crossAxisAlignment: CrossAxisAlignment.stretch,
    children: [
      Container(height: 50, color: Colors.red),
      Container(height: 50, color: Colors.green),
      Container(height: 50, color: Colors.blue),
    ],
  );
}

// Bad - Inconsistent widths
@override
Widget build(BuildContext context) {
  return Column(
    crossAxisAlignment: CrossAxisAlignment.center,
    children: [
      Container(width: 30, height: 50, color: Colors.red),
      Container(width: 60, height: 50, color: Colors.green),
      Container(width: 90, height: 50, color: Colors.blue),
    ],
  );
}
```

## Use Appropriate Spacing

```dart
// Good - Using MainAxisAlignment
@override
Widget build(BuildContext context) {
  return Column(
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
  return Column(
    children: [
      Icon(Icons.home),
      const SizedBox(height: 20),
      Icon(Icons.search),
      const SizedBox(height: 20),
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
Column(
  children: [
    Expanded(child: Container(color: Colors.red)),
    Expanded(child: Container(color: Colors.green)),
    Expanded(child: Container(color: Colors.blue)),
  ],
)
```

Correct:
```dart
// Use flex factors
Column(
  children: [
    Expanded(
      flex: 2,
      child: Container(color: Colors.red),
    ),
    Expanded(
      flex: 1,
      child: Container(color: Colors.green),
    ),
    Expanded(
      flex: 1,
      child: Container(color: Colors.blue),
    ),
  ],
)
```

## No Expanded for Long Content

Wrong:
```dart
// Content might overflow
Column(
  children: [
    Text('Very long text that might not fit'),
    Text('Another long text'),
  ],
)
```

Correct:
```dart
// Use Expanded for content
Column(
  children: [
    Expanded(
      child: Text('Very long text that will fit'),
    ),
    Expanded(
      child: Text('Another long text that will fit'),
    ),
  ],
)
```

---

# Summary

Column arranges widgets vertically with control over alignment, spacing, and sizing. Use Expanded for responsive layouts, MainAxisAlignment for spacing, and CrossAxisAlignment for horizontal alignment. Column is essential for building vertical UI components.

---

# Next Steps

- [Flex](flex.md)
- [Stack](stack.md)
- [Wrap](wrap.md)

---

# Did You Know?

- Column and Row are both Flex widgets
- Column takes the width of its widest child
- CrossAxisAlignment controls horizontal alignment
- Expanded and Flexible create responsive layouts
- MainAxisAlignment supports 6 alignment options
- Column can be nested inside Column
- Column supports scrolling when wrapped in SingleChildScrollView