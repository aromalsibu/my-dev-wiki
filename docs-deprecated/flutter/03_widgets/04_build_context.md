# BuildContext

Understand how BuildContext provides access to the widget tree.

---

# What is it?

BuildContext is a handle to the location of a widget in the widget tree. It provides access to the widget's position, parent widgets, and various services like Theme, Navigator, and MediaQuery. Every widget has a BuildContext that is passed to the build method, and it's essential for navigating the widget hierarchy and accessing inherited data.

---

# Why does it exist?

BuildContext exists to:

- Provide access to the widget tree location
- Enable navigation between screens
- Access inherited widgets and data
- Retrieve theme and styling information
- Get screen dimensions and device information
- Manage focus and form validation
- Enable widget-to-widget communication

---

# Understanding BuildContext

> **BuildContext** represents where a widget is in the tree.

```dart
// BuildContext in a widget
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // 'context' is the BuildContext
    // It represents this widget's position in the tree
    
    return Container(
      child: Text('Hello'),
    );
  }
}

// Context hierarchy:
// MaterialApp Context
//   └── Scaffold Context
//         └── MyWidget Context
//               └── Container Context
//                     └── Text Context

// Each widget has its own context
// Context provides access to:
// 1. Widget's location in tree
// 2. Ancestor widgets
// 3. Inherited widgets
// 4. Theme, Navigator, MediaQuery, etc.
```

What's happening here?
- Context is passed to build method
- Each widget has unique context
- Context represents widget position
- Context enables tree navigation
- Context provides access to services

---

# Common Uses of BuildContext

> **BuildContext is used for** various operations.

```dart
// 1. Theme access
class ThemeAccess extends StatelessWidget {
  const ThemeAccess({super.key});

  @override
  Widget build(BuildContext context) {
    // Access theme data
    final theme = Theme.of(context);
    final primaryColor = theme.primaryColor;
    final textStyle = theme.textTheme.bodyLarge;
    
    return Text(
      'Themed Text',
      style: textStyle?.copyWith(color: primaryColor),
    );
  }
}

// 2. Navigation
class NavigationExample extends StatelessWidget {
  const NavigationExample({super.key});

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        // Navigate to new screen
        Navigator.push(
          context,
          MaterialPageRoute(builder: (_) => const NewScreen()),
        );
      },
      child: const Text('Navigate'),
    );
  }
}

// 3. MediaQuery
class MediaQueryExample extends StatelessWidget {
  const MediaQueryExample({super.key});

  @override
  Widget build(BuildContext context) {
    final mediaQuery = MediaQuery.of(context);
    final screenSize = mediaQuery.size;
    final isDarkMode = mediaQuery.platformBrightness == Brightness.dark;
    
    return Container(
      width: screenSize.width * 0.8,
      height: screenSize.height * 0.5,
      color: isDarkMode ? Colors.grey[800] : Colors.grey[200],
    );
  }
}

// 4. InheritedWidget access
class InheritedAccess extends StatelessWidget {
  const InheritedAccess({super.key});

  @override
  Widget build(BuildContext context) {
    // Access custom inherited widget
    final data = MyInheritedWidget.of(context);
    final userData = UserProvider.of(context);
    
    return Text(data?.value ?? 'No data');
  }
}

// 5. Form validation
class FormValidation extends StatelessWidget {
  const FormValidation({super.key});

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        // Access form state via context
        final form = Form.of(context);
        if (form?.validate() ?? false) {
          // Form is valid
        }
      },
      child: const Text('Validate'),
    );
  }
}
```

What's happening here?
- Theme.of() accesses styling
- Navigator.push() navigates
- MediaQuery.of() gets screen info
- Inherited widgets share data
- Form.of() accesses form state

---

# Finding Widgets with Context

> **Context can be used to find** widgets in the tree.

```dart
// Finding ancestor widgets
class FindWidgetExample extends StatelessWidget {
  const FindWidgetExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Builder(
      builder: (context) {
        // 1. Find specific widget type
        final scaffold = Scaffold.of(context);
        
        // 2. Find inherited widget
        final theme = Theme.of(context);
        
        // 3. Find widget with specific type
        final navigator = Navigator.of(context);
        
        // 4. Find widget with key
        // final widget = context.findAncestorWidgetOfExactType<MyWidget>();
        
        // 5. Find render object
        // final renderObject = context.findRenderObject();
        
        return Column(
          children: [
            Text('Found widgets using context'),
            ElevatedButton(
              onPressed: () {
                // Show snackbar using scaffold
                scaffold.showSnackBar(
                  const SnackBar(content: Text('Found!')),
                );
              },
              child: const Text('Show SnackBar'),
            ),
          ],
        );
      },
    );
  }
}

// BuildContext methods for finding widgets:
// 1. context.dependOnInheritedWidgetOfExactType<T>()
// 2. context.findAncestorWidgetOfExactType<T>()
// 3. context.findAncestorStateOfType<T>()
// 4. context.findAncestorRenderObjectOfType<T>()
// 5. context.getElementForInheritedWidgetOfExactType<T>()
// 6. context.visitAncestorElements()
```

What's happening here?
- Find specific widget types
- Access inherited data
- Find ancestor widgets
- Find render objects
- Visit ancestor elements

---

# Context Types

> **Different contexts** serve different purposes.

```dart
// 1. BuildContext from build method
class BuildMethodContext extends StatelessWidget {
  const BuildMethodContext({super.key});

  @override
  Widget build(BuildContext context) {
    // Standard BuildContext
    // Available in all widgets
    return Container();
  }
}

// 2. Builder context
class BuilderContextExample extends StatelessWidget {
  const BuilderContextExample({super.key});

  @override
  Widget build(BuildContext context) {
    // This context is from the current widget
    return Builder(
      builder: (context) {
        // This is a new context for the Builder widget
        // Useful when you need a different context
        return ElevatedButton(
          onPressed: () {
            // Use the Builder's context
            Scaffold.of(context).showSnackBar(
              const SnackBar(content: Text('Hello')),
            );
          },
          child: const Text('Show SnackBar'),
        );
      },
    );
  }
}

// 3. Context from State
class StateContextExample extends StatefulWidget {
  const StateContextExample({super.key});

  @override
  State<StateContextExample> createState() => _StateContextExampleState();
}

class _StateContextExampleState extends State<StateContextExample> {
  @override
  Widget build(BuildContext context) {
    // context from State
    // Same as widget's context
    return Container();
  }
}

// 4. GlobalKey context
class GlobalKeyExample extends StatefulWidget {
  const GlobalKeyExample({super.key});

  @override
  State<GlobalKeyExample> createState() => _GlobalKeyExampleState();
}

class _GlobalKeyExampleState extends State<GlobalKeyExample> {
  final GlobalKey _key = GlobalKey();

  void _accessContext() {
    // Access context from GlobalKey
    final context = _key.currentContext;
    if (context != null) {
      // Use context
    }
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      key: _key,
      child: const Text('GlobalKey Context'),
    );
  }
}
```

What's happening here?
- Build method provides context
- Builder creates new context
- State has context
- GlobalKey provides context access

---

# Context Limitations

> **BuildContext has limitations** to be aware of.

```dart
// Context limitations
class ContextLimitations extends StatelessWidget {
  const ContextLimitations({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Context can't be used after dispose
    // ❌ Don't store context for later use
    
    // 2. Context must be from same widget tree
    // ❌ Can't use context from different tree
    
    // 3. Context changes during rebuild
    // ⚠️ Context can change when widget moves
    
    // 4. Not all widgets are accessible
    // ❌ Can't access widgets that don't exist
    
    return Container();
  }
}

// Safe context usage
class SafeContextUsage extends StatefulWidget {
  const SafeContextUsage({super.key});

  @override
  State<SafeContextUsage> createState() => _SafeContextUsageState();
}

class _SafeContextUsageState extends State<SafeContextUsage> {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        // 1. Check if mounted before using context
        if (mounted) {
          // Safe to use context
          Navigator.push(
            context,
            MaterialPageRoute(builder: (_) => const NewScreen()),
          );
        }
      },
      child: const Text('Navigate'),
    );
  }

  // 2. Don't store context
  // ❌ BuildContext? _storedContext;
  
  // 3. Use context synchronously
  void _useContext(BuildContext context) {
    // ✓ Use context immediately
    final theme = Theme.of(context);
  }
}

// When context is invalid:
// 1. After widget is disposed
// 2. During async operations (check mounted)
// 3. When widget is removed from tree
// 4. During hot reload
```

What's happening here?
- Context can expire
- Check mounted before use
- Don't store context
- Use context synchronously
- Context changes can occur

---

# Context and InheritedWidget

> **Context is essential** for InheritedWidget communication.

```dart
// InheritedWidget with context
class MyInheritedWidget extends InheritedWidget {
  const MyInheritedWidget({
    super.key,
    required this.data,
    required super.child,
  });

  final String data;

  static MyInheritedWidget? of(BuildContext context) {
    // Use context to find inherited widget
    return context.dependOnInheritedWidgetOfExactType<MyInheritedWidget>();
  }

  @override
  bool updateShouldNotify(MyInheritedWidget oldWidget) {
    return data != oldWidget.data;
  }
}

// Using inherited data
class DataConsumer extends StatelessWidget {
  const DataConsumer({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Access data via context
    final inheritedData = MyInheritedWidget.of(context);
    
    // 2. Access multiple inherited widgets
    final theme = Theme.of(context);
    final mediaQuery = MediaQuery.of(context);
    
    return Column(
      children: [
        Text('Data: ${inheritedData?.data ?? 'No data'}'),
        Text('Theme: ${theme.primaryColor}'),
        Text('Screen: ${mediaQuery.size}'),
      ],
    );
  }
}

// Context dependency tracking
class DependentWidget extends StatefulWidget {
  const DependentWidget({super.key});

  @override
  State<DependentWidget> createState() => _DependentWidgetState();
}

class _DependentWidgetState extends State<DependentWidget> {
  @override
  Widget build(BuildContext context) {
    // This widget depends on Theme
    final theme = Theme.of(context);
    
    // This widget depends on MediaQuery
    final size = MediaQuery.of(context).size;
    
    // This widget depends on custom inherited
    final data = MyInheritedWidget.of(context);
    
    // When any of these change, widget rebuilds
    return Container(
      color: theme.primaryColor,
      width: size.width * 0.5,
      child: Text(data?.data ?? ''),
    );
  }
}
```

What's happening here?
- Context finds inherited widgets
- Dependencies are tracked
- Widgets rebuild on dependency changes
- Multiple inherited widgets can be used
- Context enables data sharing

---

# Best Practices

## Use Appropriate Context

```dart
// Good - Using correct context
@override
Widget build(BuildContext context) {
  return Builder(
    builder: (context) {
      // Use Builder context for Scaffold
      return ElevatedButton(
        onPressed: () {
          Scaffold.of(context).showSnackBar(
            const SnackBar(content: Text('Hello')),
          );
        },
        child: const Text('Show'),
      );
    },
  );
}

// Bad - Using wrong context
@override
Widget build(BuildContext context) {
  return ElevatedButton(
    onPressed: () {
      // This context might not find Scaffold
      Scaffold.of(context).showSnackBar(
        const SnackBar(content: Text('Hello')),
      );
    },
    child: const Text('Show'),
  );
}
```

## Check Mounted Before Async Context Use

```dart
// Good - Checking mounted
Future<void> _navigate() async {
  await someAsync();
  if (mounted) {
    Navigator.push(context, route);
  }
}

// Bad - No mounted check
Future<void> _navigate() async {
  await someAsync();
  Navigator.push(context, route);
}
```

## Use Context in Build Only

```dart
// Good - Context in build
@override
Widget build(BuildContext context) {
  final theme = Theme.of(context);
  return Container();
}

// Bad - Storing context
@override
void initState() {
  super.initState();
  // ❌ Don't store context
  // _context = context;
}
```

---

# Common Mistakes

## Using Context After Dispose

Wrong:
```dart
Future<void> _doSomething() async {
  await Future.delayed(const Duration(seconds: 2));
  // Context might be disposed
  Navigator.push(context, route);
}
```

Correct:
```dart
Future<void> _doSomething() async {
  await Future.delayed(const Duration(seconds: 2));
  if (mounted) {
    Navigator.push(context, route);
  }
}
```

## Wrong Context for Scaffold

Wrong:
```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: ElevatedButton(
      onPressed: () {
        // This context is from the scaffold's parent
        Scaffold.of(context).showSnackBar(...);
      },
    ),
  );
}
```

Correct:
```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: Builder(
      builder: (context) {
        // This context is from the scaffold's child
        return ElevatedButton(
          onPressed: () {
            Scaffold.of(context).showSnackBar(...);
          },
        );
      },
    ),
  );
}
```

---

# Summary

BuildContext is a handle to a widget's location in the tree. It provides access to theme, navigation, media queries, inherited widgets, and more. Always check mounted before using context in async operations and use the appropriate context for the task.

---

# Next Steps

- [Keys](keys.md)
- [Widget Lifecycle](widget-lifecycle.md)
- [Widget Composition](widget-composition.md)

---

# Did You Know?

- Every widget has its own BuildContext
- Context represents a widget's position in the tree
- Context can find ancestor widgets
- Context dependencies trigger rebuilds
- Check mounted before using context
- Builder creates a new context
- GlobalKey provides context access
- Context is essential for navigation