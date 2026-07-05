# Center

Understand how to center a child widget within its parent.

---

# What is it?

Center is a widget that positions its child in the center of itself. It's a specialized version of Align that always positions the child at the center. Center is one of the simplest and most commonly used layout widgets in Flutter.

---

# Why does it exist?

Center exists to:

- Center a child widget easily
- Provide a simple alternative to Align for centering
- Make code more readable
- Center both horizontally and vertically
- Create centered layouts quickly
- Handle responsive centering
- Simplify common UI patterns

---

# Basic Center

> **Center** places its child in the middle.

```dart
// Basic Center usage
class BasicCenter extends StatelessWidget {
  const BasicCenter({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 200,
      color: Colors.grey[200],
      child: const Center(
        child: Text(
          'Centered Text',
          style: TextStyle(fontSize: 24),
        ),
      ),
    );
  }
}

// Center properties:
// 1. child - The widget to center
// 2. widthFactor - Scale child width (optional)
// 3. heightFactor - Scale child height (optional)
```

What's happening here?
- Child is centered both horizontally and vertically
- Child keeps its own size
- Parent bounds determine centering area
- Simple and intuitive

---

# Center vs Align

> **Center** is a specialized Align.

```dart
// Center vs Align comparison
class CenterVsAlign extends StatelessWidget {
  const CenterVsAlign({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Center widget
        Container(
          width: 200,
          height: 100,
          color: Colors.grey[200],
          child: const Center(
            child: Text('Center'),
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Align with center
        Container(
          width: 200,
          height: 100,
          color: Colors.grey[200],
          child: const Align(
            alignment: Alignment.center,
            child: Text('Align Center'),
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Container alignment
        Container(
          width: 200,
          height: 100,
          color: Colors.grey[200],
          alignment: Alignment.center,
          child: const Text('Container Align'),
        ),
      ],
    );
  }
}

// Center = Align(alignment: Alignment.center)
// Center is simpler and more readable
// Use Center when you only need centering
// Use Align for other alignments
```

What's happening here?
- Center is equivalent to Align.center
- Center is more readable
- All three achieve the same result
- Choose based on clarity

---

# Center with Size Factors

> **Size factors** control child sizing.

```dart
// Size factor examples
class CenterSizeFactor extends StatelessWidget {
  const CenterSizeFactor({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: 300,
      height: 200,
      color: Colors.grey[200],
      child: Column(
        children: [
          // 1. No factor (child keeps size)
          Expanded(
            child: Center(
              child: Container(
                color: Colors.red,
                child: const Text('No Factor'),
              ),
            ),
          ),
          
          // 2. Width factor 0.5
          Expanded(
            child: Center(
              widthFactor: 0.5,
              child: Container(
                color: Colors.green,
                child: const Text('Width 0.5'),
              ),
            ),
          ),
          
          // 3. Both factors
          Expanded(
            child: Center(
              widthFactor: 0.5,
              heightFactor: 0.5,
              child: Container(
                color: Colors.blue,
                child: const Text('Both 0.5'),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// Size factor effects:
// widthFactor: null (use child width)
// widthFactor: 0.5 (child = 50% of parent)
// widthFactor: 1.0 (child fills parent width)
// heightFactor: null (use child height)
// heightFactor: 0.5 (child = 50% of parent)
// heightFactor: 1.0 (child fills parent height)
```

What's happening here?
- Size factors scale the child
- Null uses child's natural size
- 0.5 makes child half the parent
- 1.0 fills the parent dimension
- Useful for responsive centering

---

# Center in Layouts

> **Center** in different layout contexts.

```dart
// Center in various layouts
class CenterInLayouts extends StatelessWidget {
  const CenterInLayouts({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 1. Center in Row
        Container(
          height: 100,
          color: Colors.grey[200],
          child: Row(
            children: [
              const Text('Left'),
              const Expanded(
                child: Center(
                  child: Text('Centered in Row'),
                ),
              ),
              const Text('Right'),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 2. Center in Column
        Container(
          height: 150,
          color: Colors.grey[200],
          child: Row(
            children: [
              Expanded(
                child: Column(
                  children: [
                    const Text('Top'),
                    const Expanded(
                      child: Center(
                        child: Text('Centered in Column'),
                      ),
                    ),
                    const Text('Bottom'),
                  ],
                ),
              ),
            ],
          ),
        ),
        
        const SizedBox(height: 10),
        
        // 3. Center in Stack
        Container(
          width: 200,
          height: 200,
          color: Colors.grey[200],
          child: Stack(
            children: [
              Container(color: Colors.blue),
              const Center(
                child: Text(
                  'Centered in Stack',
                  style: TextStyle(color: Colors.white),
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
- Center works in any layout context
- Row with centered text
- Column with centered content
- Stack with centered overlay

---

# Real-World Examples

> **Common patterns** using Center.

```dart
// 1. Loading indicator
class LoadingIndicator extends StatelessWidget {
  const LoadingIndicator({super.key, this.message});

  final String? message;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const CircularProgressIndicator(),
          if (message != null) ...[
            const SizedBox(height: 16),
            Text(
              message!,
              style: const TextStyle(
                color: Colors.grey,
                fontSize: 16,
              ),
            ),
          ],
        ],
      ),
    );
  }
}

// 2. Empty state
class EmptyState extends StatelessWidget {
  const EmptyState({
    super.key,
    required this.icon,
    required this.title,
    required this.subtitle,
    this.action,
  });

  final IconData icon;
  final String title;
  final String subtitle;
  final Widget? action;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              icon,
              size: 64,
              color: Colors.grey[400],
            ),
            const SizedBox(height: 16),
            Text(
              title,
              style: const TextStyle(
                fontSize: 20,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              subtitle,
              textAlign: TextAlign.center,
              style: TextStyle(
                color: Colors.grey[600],
                fontSize: 16,
              ),
            ),
            if (action != null) ...[
              const SizedBox(height: 16),
              action!,
            ],
          ],
        ),
      ),
    );
  }
}

// 3. Splash screen
class SplashScreen extends StatelessWidget {
  const SplashScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        decoration: const BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topCenter,
            end: Alignment.bottomCenter,
            colors: [Colors.blue, Colors.purple],
          ),
        ),
        child: const Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(
                Icons.flutter_dash,
                size: 80,
                color: Colors.white,
              ),
              SizedBox(height: 16),
              Text(
                'My App',
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 32,
                  fontWeight: FontWeight.bold,
                ),
              ),
              SizedBox(height: 8),
              CircularProgressIndicator(
                valueColor: AlwaysStoppedAnimation<Color>(Colors.white),
              ),
            ],
          ),
        ),
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
      width: double.infinity,
      height: 100,
      child: Center(
        child: ElevatedButton(
          onPressed: () {},
          style: ElevatedButton.styleFrom(
            padding: const EdgeInsets.symmetric(
              horizontal: 40,
              vertical: 16,
            ),
          ),
          child: const Text('Centered Button'),
        ),
      ),
    );
  }
}
```

What's happening here?
- Loading indicator centered
- Empty state with icon and text
- Splash screen with centered content
- Centered button in container

---

# Best Practices

## Use Center for Simplicity

```dart
// Good - Simple and readable
@override
Widget build(BuildContext context) {
  return Center(
    child: const Text('Hello'),
  );
}
```

## Combine with Other Widgets

```dart
// Good - Center with Column for vertical layout
@override
Widget build(BuildContext context) {
  return Center(
    child: Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text('Title'),
        Text('Subtitle'),
      ],
    ),
  );
}
```

## Use Size Factors Responsively

```dart
// Good - Responsive sizing
@override
Widget build(BuildContext context) {
  return Center(
    widthFactor: 0.8,
    child: Container(
      color: Colors.blue,
      child: const Text('Responsive'),
    ),
  );
}
```

---

# Common Mistakes

## Center Without Bounds

Wrong:
```dart
// Center needs bounded parent
Center(
  child: Container(width: 100, height: 100),
)
```

Correct:
```dart
// Give Center bounds
Container(
  width: 300,
  height: 200,
  child: Center(
    child: Container(width: 100, height: 100),
  ),
)
```

## Overusing Center

Wrong:
```dart
// Unnecessary Center
Row(
  children: [
    Center(child: Text('Left')), // Not needed
    Center(child: Text('Right')), // Not needed
  ],
)
```

Correct:
```dart
// Use appropriate alignment
Row(
  children: [
    Text('Left'),
    Text('Right'),
  ],
)
```

---

# Summary

Center positions a child in the center of its parent. It's a simplified version of Align specifically for centering. Use Center for simple centering needs, loading indicators, empty states, and any UI that needs centered content.

---

# Next Steps

- [SizedBox](sizedbox.md)
- [Expanded](expanded.md)
- [Flexible](flexible.md)

---

# Did You Know?

- Center = Align(alignment: Alignment.center)
- Center is one of the most used widgets
- Center works in any layout context
- widthFactor and heightFactor scale child size
- Center requires bounded parent
- Center can be nested for complex layouts
- Center is used in many Flutter widgets
- Center is simple but powerful