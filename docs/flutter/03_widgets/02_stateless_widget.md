# StatelessWidget

Understand the simplest form of widget that has no mutable state.

---

# What is it?

A StatelessWidget is a widget that describes part of the user interface by building a constellation of other widgets. It is called "stateless" because it does not contain any mutable state - once created, it cannot change. The widget's properties are final and immutable, and it only rebuilds when its parent widget changes or when it receives new configuration.

---

# Why does it exist?

StatelessWidget exists to:

- Create UI elements that don't change over time
- Represent static content and layouts
- Improve performance through predictability
- Simplify widget creation and maintenance
- Enable composition of widgets
- Promote functional programming patterns

---

# StatelessWidget Basics

> **StatelessWidget** has no mutable state and only depends on its configuration.

```dart
// Basic StatelessWidget
class MyStatelessWidget extends StatelessWidget {
  // All properties must be final (immutable)
  final String title;
  final Color color;
  final VoidCallback? onTap;

  // Constructor with required parameters
  const MyStatelessWidget({
    super.key,
    required this.title,
    this.color = Colors.blue,
    this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    // Called when widget is created or parent changes
    // Returns a widget tree
    return GestureDetector(
      onTap: onTap,
      child: Container(
        color: color,
        padding: const EdgeInsets.all(16),
        child: Text(title),
      ),
    );
  }
}

// Using the stateless widget
class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Column(
            children: [
              // Each instance has its own configuration
              const MyStatelessWidget(
                title: 'Hello',
                color: Colors.blue,
              ),
              MyStatelessWidget(
                title: 'World',
                color: Colors.red,
                onTap: () {
                  print('Tapped!');
                },
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

What's happening here?
- Widget properties are final and immutable
- build() is the only required method
- Widget rebuilds when parent changes
- No mutable state to manage
- Pure UI representation

---

# When to Use StatelessWidget

> **Use StatelessWidget** when your widget doesn't need mutable state.

```dart
// 1. Static content
class StaticContent extends StatelessWidget {
  const StaticContent({super.key});

  @override
  Widget build(BuildContext context) {
    // Content never changes
    return const Text('Hello World');
  }
}

// 2. Layout widgets
class MyLayout extends StatelessWidget {
  const MyLayout({super.key});

  @override
  Widget build(BuildContext context) {
    // Layout structure only
    return Column(
      children: [
        const Header(),
        const Content(),
        const Footer(),
      ],
    );
  }
}

// 3. Presentation widgets
class UserProfile extends StatelessWidget {
  const UserProfile({
    super.key,
    required this.name,
    required this.email,
  });

  final String name;
  final String email;

  @override
  Widget build(BuildContext context) {
    // Only displays data passed in
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(name, style: const TextStyle(fontSize: 18)),
            Text(email, style: const TextStyle(color: Colors.grey)),
          ],
        ),
      ),
    );
  }
}

// 4. UI components
class CustomButton extends StatelessWidget {
  const CustomButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.isPrimary = true,
  });

  final String label;
  final VoidCallback onPressed;
  final bool isPrimary;

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed,
      style: ElevatedButton.styleFrom(
        backgroundColor: isPrimary ? Colors.blue : Colors.grey,
      ),
      child: Text(label),
    );
  }
}

// 5. Wrapper widgets
class PaddingWrapper extends StatelessWidget {
  const PaddingWrapper({
    super.key,
    required this.child,
  });

  final Widget child;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: child,
    );
  }
}
```

What's happening here?
- Static content doesn't change
- Layout structure is fixed
- Data comes from parent
- UI components are reusable
- Wrappers modify children

---

# StatelessWidget Properties

> **Properties must be final** in StatelessWidget.

```dart
// StatelessWidget with various property types
class PropertyExample extends StatelessWidget {
  // 1. All properties must be final
  final String stringProperty;
  final int intProperty;
  final double doubleProperty;
  final bool boolProperty;
  final Color colorProperty;
  final Widget? widgetProperty;
  final VoidCallback? callbackProperty;
  
  // 2. Default values can be set
  final String defaultProperty = 'Default';
  
  // 3. Constructor with required and optional parameters
  const PropertyExample({
    super.key,
    required this.stringProperty,
    this.intProperty = 0,
    this.doubleProperty = 0.0,
    this.boolProperty = false,
    this.colorProperty = Colors.blue,
    this.widgetProperty,
    this.callbackProperty,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      color: colorProperty,
      child: Column(
        children: [
          Text('String: $stringProperty'),
          Text('Int: $intProperty'),
          Text('Double: $doubleProperty'),
          Text('Bool: $boolProperty'),
          if (widgetProperty != null) widgetProperty!,
          if (callbackProperty != null)
            ElevatedButton(
              onPressed: callbackProperty,
              child: const Text('Callback'),
            ),
        ],
      ),
    );
  }
}

// Why properties must be final:
// 1. Widgets are immutable
// 2. Prevents accidental modification
// 3. Enables performance optimizations
// 4. Makes widgets predictable
// 5. Hot reload works properly
```

What's happening here?
- All properties are final
- Constructor is const
- Immutable configuration
- Hot reload compatible
- Predictable behavior

---

# StatelessWidget Lifecycle

> **StatelessWidget has a simple lifecycle** - just construction and build.

```dart
// StatelessWidget lifecycle demonstration
class LifecycleDemo extends StatelessWidget {
  const LifecycleDemo({
    super.key,
    required this.message,
  });

  final String message;

  // 1. Constructor called
  // Widget is created
  const LifecycleDemo({super.key, required this.message}) {
    print('1. Constructor called');
  }

  @override
  Widget build(BuildContext context) {
    // 2. build() called
    // Called after constructor
    // Called when parent rebuilds
    // Called when dependencies change
    print('2. build() called');
    
    return Container(
      child: Text(message),
    );
  }

  // Lifecycle events:
  // 1. Widget is created (constructor)
  // 2. Widget is mounted to tree
  // 3. build() is called
  // 4. Widget is updated (if needed)
  // 5. Widget is removed from tree

  // No mutable state lifecycle methods:
  // No initState()
  // No setState()
  // No dispose()
  // No didUpdateWidget()
}

// Multiple stateless widgets in a tree:
class ParentWidget extends StatelessWidget {
  const ParentWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // When parent rebuilds:
    // 1. Child constructor called
    // 2. Child build() called
    // 3. Child is updated
    
    return Column(
      children: [
        const ChildWidget(message: 'Child 1'),
        const ChildWidget(message: 'Child 2'),
        const ChildWidget(message: 'Child 3'),
      ],
    );
  }
}
```

What's happening here?
- Simple lifecycle
- Constructor creates widget
- build() builds UI
- No state management
- Rebuilds when needed

---

# Stateless vs Stateful

> **Comparing** StatelessWidget and StatefulWidget.

```dart
// StatelessWidget (no mutable state)
class StatelessCounter extends StatelessWidget {
  const StatelessCounter({
    super.key,
    required this.count,
    required this.onIncrement,
  });

  final int count;
  final VoidCallback onIncrement;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: onIncrement,
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

// StatefulWidget (has mutable state)
class StatefulCounter extends StatefulWidget {
  const StatefulCounter({super.key});

  @override
  State<StatefulCounter> createState() => _StatefulCounterState();
}

class _StatefulCounterState extends State<StatefulCounter> {
  int _count = 0; // Mutable state

  void _increment() {
    setState(() {
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: _increment,
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

// When to use which:
// StatelessWidget:
// ✓ UI that doesn't change
// ✓ Content from parent
// ✓ Layout widgets
// ✓ Presentation widgets
// ✓ Static components

// StatefulWidget:
// ✓ UI that changes over time
// ✓ User interactions
// ✓ Animations
// ✓ Forms and input
// ✓ Data that updates
```

What's happening here?
- Stateless: data from parent
- Stateful: data managed internally
- Stateless: simpler and faster
- Stateful: handles user interaction
- Choose based on need

---

# Performance Considerations

> **StatelessWidget is more performant** than StatefulWidget.

```dart
// Performance optimization
class OptimizedStatelessWidget extends StatelessWidget {
  const OptimizedStatelessWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Use const for static content
    return const Column(
      children: [
        // 2. Const widgets are cached
        Text('Static text'), // Won't rebuild
        Text('More static text'), // Won't rebuild
      ],
    );
  }
}

// 3. Split into smaller widgets
class SplitWidget extends StatelessWidget {
  const SplitWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const HeaderWidget(), // Independent
        const ContentWidget(), // Independent
        const FooterWidget(), // Independent
      ],
    );
  }
}

// 4. Use const constructors
class ConstWidget extends StatelessWidget {
  const ConstWidget({super.key}); // Allows const usage

  @override
  Widget build(BuildContext context) {
    return const Text('Hello');
  }
}

// Performance benefits:
// 1. No state to manage
// 2. No setState calls
// 3. No lifecycle overhead
// 4. Optimized rebuilds
// 5. Better performance

// Performance comparison:
// StatelessWidget: Faster
// StatefulWidget: Slower (has more overhead)
// Use StatelessWidget when possible
```

What's happening here?
- const widgets are cached
- Smaller widgets = better performance
- No state overhead
- Faster rebuilds
- Prefer Stateless when possible

---

# Common Patterns

> **Common patterns** with StatelessWidget.

```dart
// 1. Factory pattern
class ButtonFactory {
  static Widget primaryButton(String label, VoidCallback onPressed) {
    return CustomButton(
      label: label,
      onPressed: onPressed,
      isPrimary: true,
    );
  }

  static Widget secondaryButton(String label, VoidCallback onPressed) {
    return CustomButton(
      label: label,
      onPressed: onPressed,
      isPrimary: false,
    );
  }
}

// 2. Builder pattern
class ContentBuilder extends StatelessWidget {
  const ContentBuilder({
    super.key,
    required this.builder,
  });

  final Widget Function(BuildContext) builder;

  @override
  Widget build(BuildContext context) {
    return builder(context);
  }
}

// 3. Composition pattern
class ComposedWidget extends StatelessWidget {
  const ComposedWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      child: Row(
        children: [
          const Avatar(),
          const SizedBox(width: 8),
          const UserInfo(),
          const Spacer(),
          const ActionButton(),
        ],
      ),
    );
  }
}

// 4. Conditional rendering
class ConditionalWidget extends StatelessWidget {
  const ConditionalWidget({
    super.key,
    required this.showContent,
  });

  final bool showContent;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Using if statement
        if (showContent)
          const Text('Content visible'),
        
        // Using conditional operator
        showContent 
          ? const Text('Visible')
          : const Text('Hidden'),
      ],
    );
  }
}
```

What's happening here?
- Factory creates widgets
- Builder pattern for dynamic content
- Composition builds complex UI
- Conditional rendering

---

# Best Practices

## Keep Widgets Pure

```dart
// Good - Pure function
@override
Widget build(BuildContext context) {
  return Text(title);
}

// Bad - Side effects
@override
Widget build(BuildContext context) {
  print('Building widget'); // Side effect
  fetchData(); // Side effect
  return Text(title);
}
```

## Use const Constructors

```dart
// Good - Const constructor
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});
  
  @override
  Widget build(BuildContext context) {
    return const Text('Hello');
  }
}

// Bad - Missing const
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Text('Hello'); // Not const
  }
}
```

## Split Large Widgets

```dart
// Good - Split widgets
class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: const [
        ProfileHeader(),
        ProfileContent(),
        ProfileFooter(),
      ],
    );
  }
}

// Bad - Large monolithic widget
class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Everything in one widget
    return Column(
      children: [
        // Header
        // Content
        // Footer
        // All in one place
      ],
    );
  }
}
```

---

# Common Mistakes

## Using Stateful When Not Needed

Wrong:
```dart
// Overkill - Stateful when Stateless would work
class StaticText extends StatefulWidget {
  const StaticText({super.key});

  @override
  State<StaticText> createState() => _StaticTextState();
}

class _StaticTextState extends State<StaticText> {
  @override
  Widget build(BuildContext context) {
    return const Text('Hello');
  }
}
```

Correct:
```dart
// Simple - Stateless is enough
class StaticText extends StatelessWidget {
  const StaticText({super.key});

  @override
  Widget build(BuildContext context) {
    return const Text('Hello');
  }
}
```

## Mutating Properties

Wrong:
```dart
// Can't mutate final properties
class MyWidget extends StatelessWidget {
  final String title;
  
  void updateTitle() {
    // Error: title is final
    title = 'New title';
  }
}
```

Correct:
```dart
// Properties are final
class MyWidget extends StatelessWidget {
  final String title;
  
  const MyWidget({required this.title});
  
  void updateTitle() {
    // Can't update, create new widget instead
  }
}
```

---

# Summary

StatelessWidget is the simplest widget type in Flutter. It has no mutable state, only immutable configuration. It's used for static content, presentation widgets, and layout structure. StatelessWidgets are performant and easy to reason about.

---

# Next Steps

- [StatefulWidget](statefulwidget.md)
- [BuildContext](buildcontext.md)
- [Keys](keys.md)

---

# Did You Know?

- StatelessWidget is more performant than StatefulWidget
- All properties in StatelessWidget must be final
- StatelessWidget can be const
- The build method is called whenever the widget updates
- StatelessWidget is the most common widget type
- Everything in Flutter starts as a StatelessWidget
- StatelessWidgets are easier to test
- You can compose multiple StatelessWidgets together