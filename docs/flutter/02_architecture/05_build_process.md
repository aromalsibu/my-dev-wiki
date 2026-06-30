# Build Process

Understand how Flutter builds widgets and manages the build lifecycle.

---

# What is it?

The build process in Flutter is the mechanism by which widgets are created, configured, and arranged into a widget tree. It involves calling the `build()` method on widgets, reconciling with the existing element tree, and updating the render tree. The build process is triggered by state changes, parent rebuilds, or other events.

---

# Why does it exist?

The build process exists to:

- Create and configure widgets based on current state
- Rebuild parts of the UI when state changes
- Optimize performance by only rebuilding changed widgets
- Maintain a consistent UI state
- Enable declarative UI programming
- Support hot reload and development features

---

# The Build Pipeline

> **The build process** follows a specific pipeline in Flutter.

```
1. Trigger Event
   ↓
2. Mark Dirty
   ↓
3. Schedule Build
   ↓
4. Build Widgets
   ↓
5. Update Elements
   ↓
6. Layout Render Tree
   ↓
7. Paint Render Tree
   ↓
8. Display on Screen
```

What's happening here?
- Events trigger the build process
- Widgets are marked as dirty
- Build is scheduled on the next frame
- Widgets are rebuilt and configured
- Elements are updated with new configuration
- Render tree is laid out and painted

---

# The Build Method

> **`build()` is called** to create the widget hierarchy.

```dart
// The build method
class MyWidget extends StatelessWidget {
  const MyWidget({super.key, required this.title});

  final String title;

  @override
  Widget build(BuildContext context) {
    // Called every time the widget needs to rebuild
    // Returns a widget tree
    
    return Scaffold(
      appBar: AppBar(
        title: Text(title),
      ),
      body: Center(
        child: Column(
          children: [
            const Text('Hello'),
            ElevatedButton(
              onPressed: () {
                // Triggers rebuild when state changes
                setState(() {});
              },
              child: const Text('Press'),
            ),
          ],
        ),
      ),
    );
  }
}

// When build() is called:
// 1. Initial build (after mounting)
// 2. State changes (setState)
// 3. Inherited widget changes
// 4. Parent widget rebuilds
// 5. Hot reload

// What build() should NOT do:
// - Heavy computations
// - Network requests
// - File operations
// - Database queries
// - Memory allocations
```

What's happening here?
- build() returns a widget tree
- Called when widget needs to update
- Should be pure and fast
- Avoid heavy operations in build
- build() creates new widgets

---

# Types of Build

> **Different types** of builds occur in Flutter.

```dart
// 1. Initial Build (mounting)
class InitialBuildWidget extends StatefulWidget {
  const InitialBuildWidget({super.key});

  @override
  State<InitialBuildWidget> createState() => _InitialBuildWidgetState();
}

class _InitialBuildWidgetState extends State<InitialBuildWidget> {
  @override
  void initState() {
    super.initState();
    // Initial build triggered
    // Widget is mounted to tree
    print('Initial build');
  }

  @override
  Widget build(BuildContext context) {
    return const Text('Initial');
  }
}

// 2. State Change Build
class StateChangeWidget extends StatefulWidget {
  const StateChangeWidget({super.key});

  @override
  State<StateChangeWidget> createState() => _StateChangeWidgetState();
}

class _StateChangeWidgetState extends State<StateChangeWidget> {
  int _counter = 0;

  void _increment() {
    setState(() {
      _counter++; // Triggers rebuild
    });
  }

  @override
  Widget build(BuildContext context) {
    // Called every time setState is called
    print('Build: $_counter');
    return Text('$_counter');
  }
}

// 3. Inherited Widget Build
class InheritedBuildWidget extends StatelessWidget {
  const InheritedBuildWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // Builds when inherited data changes
    final data = InheritedData.of(context);
    return Text(data.value);
  }
}

// 4. Parent Rebuild
class ParentWidget extends StatefulWidget {
  const ParentWidget({super.key});

  @override
  State<ParentWidget> createState() => _ParentWidgetState();
}

class _ParentWidgetState extends State<ParentWidget> {
  bool _showChild = true;

  @override
  Widget build(BuildContext context) {
    // Rebuilds child when parent rebuilds
    return Column(
      children: [
        ElevatedButton(
          onPressed: () {
            setState(() {
              _showChild = !_showChild;
            });
          },
          child: const Text('Toggle'),
        ),
        if (_showChild)
          const ChildWidget(), // Rebuilds with parent
      ],
    );
  }
}
```

What's happening here?
- Initial build happens on mount
- State changes trigger rebuilds
- Inherited widgets rebuild dependents
- Parent rebuilds affect children

---

# Build Scheduling

> **Builds are scheduled** to occur on the next frame.

```dart
// Build scheduling
class BuildScheduler extends StatefulWidget {
  const BuildScheduler({super.key});

  @override
  State<BuildScheduler> createState() => _BuildSchedulerState();
}

class _BuildSchedulerState extends State<BuildScheduler> {
  int _counter = 0;

  void _updateCounter() {
    // setState schedules a build
    setState(() {
      _counter++;
    });
    // Build doesn't happen immediately
    // It's scheduled for the next frame
    print('State updated, build scheduled');
  }

  @override
  Widget build(BuildContext context) {
    // Called on the next frame
    print('Build executing');
    return Text('Counter: $_counter');
  }
}

// Multiple setState calls are batched
void _batchUpdate() {
  // All three setState calls
  // Result in a single build
  setState(() { _counter++; });
  setState(() { _counter++; });
  setState(() { _counter++; });
}

// Manual build triggering
void _triggerBuild() {
  // Rarely needed
  // WidgetsBinding.instance.allowReassemble();
}

// Build phases:
// 1. Mark dirty (setState)
// 2. Schedule build (end of frame)
// 3. Execute build (next frame)
// 4. Perform layout
// 5. Perform painting
```

What's happening here?
- setState schedules a build
- Builds are batched together
- Builds happen on the next frame
- Multiple changes = one build
- Efficient update system

---

# Build Context

> **BuildContext** provides information during build.

```dart
// Using BuildContext in build method
class ContextWidget extends StatelessWidget {
  const ContextWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Access theme
    final theme = Theme.of(context);
    
    // 2. Access media query
    final mediaQuery = MediaQuery.of(context);
    final size = mediaQuery.size;
    final isDark = mediaQuery.platformBrightness == Brightness.dark;
    
    // 3. Access inherited widgets
    final data = InheritedData.of(context);
    
    // 4. Access navigator
    final navigator = Navigator.of(context);
    
    // 5. Access focus scope
    final focusScope = FocusScope.of(context);
    
    // 6. Access scaffold
    final scaffold = Scaffold.of(context);
    
    return Container(
      width: size.width * 0.5,
      color: theme.primaryColor,
      child: Text(data?.value ?? 'No data'),
    );
  }
}

// Context limitations:
// 1. Only available during build
// 2. Cannot be used after dispose
// 3. Must be used in current widget
// 4. May not exist at certain times

// Safe context usage:
if (mounted) {
  // Safe to use context
  Navigator.push(context, route);
}
```

What's happening here?
- Context provides tree information
- Access theme, media, inherited data
- Used for navigation and actions
- Only valid during build and events

---

# Build Optimization

> **Optimizing builds** improves performance.

```dart
// 1. Use const widgets
class ConstWidget extends StatelessWidget {
  const ConstWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // const widgets are reused
    return const Column(
      children: [
        Text('Hello'),
        Text('World'),
      ],
    );
  }
}

// 2. Use const constructors
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // const children don't rebuild
    return const Padding(
      padding: EdgeInsets.all(8.0),
      child: Text('Hello'),
    );
  }
}

// 3. Split widgets
class SplitWidget extends StatefulWidget {
  const SplitWidget({super.key});

  @override
  State<SplitWidget> createState() => _SplitWidgetState();
}

class _SplitWidgetState extends State<SplitWidget> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // This widget rebuilds when counter changes
        CounterDisplay(counter: _counter),
        
        // This widget doesn't rebuild (const)
        const ControlsWidget(),
        
        // This widget is independent
        ElevatedButton(
          onPressed: () {
            setState(() {
              _counter++;
            });
          },
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

// 4. Use child caching
class CachedWidget extends StatelessWidget {
  const CachedWidget({super.key, required this.child});

  final Widget child;

  @override
  Widget build(BuildContext context) {
    // Child is cached and not rebuilt
    return Container(
      child: child,
    );
  }
}

// 5. Use keys for stable identity
class ListWidget extends StatelessWidget {
  const ListWidget({super.key, required this.items});

  final List<String> items;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: items.map((item) {
        return Text(
          item,
          key: ValueKey(item), // Stable identity
        );
      }).toList(),
    );
  }
}
```

What's happening here?
- const widgets are reused
- Split widgets to isolate rebuilds
- Child caching prevents rebuilds
- Keys provide stable identity
- Smaller widgets rebuild faster

---

# Build Performance

> **Measuring and improving** build performance.

```dart
// Debug build performance
class DebugWidget extends StatelessWidget {
  const DebugWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // 1. Track build time
    final stopwatch = Stopwatch()..start();
    
    // Build widget
    final widget = _buildWidget();
    
    stopwatch.stop();
    if (stopwatch.elapsedMilliseconds > 16) {
      // Build took too long (exceeds frame budget)
      print('Slow build: ${stopwatch.elapsedMilliseconds}ms');
    }
    
    return widget;
  }

  Widget _buildWidget() {
    // Heavy build
    return const Text('Hello');
  }
}

// Profile build performance
class ProfileBuild extends StatelessWidget {
  const ProfileBuild({super.key});

  @override
  Widget build(BuildContext context) {
    // Use Flutter DevTools to profile:
    // 1. Open DevTools
    // 2. Select Performance tab
    // 3. Record timeline
    // 4. Check build costs
    // 5. Identify expensive widgets
    
    return const Text('Profile me');
  }
}

// Common performance issues:
// 1. Building too many widgets
// 2. Heavy computation in build
// 3. Deep widget trees
// 4. Not using const
// 5. Rebuilding unchanged widgets
```

What's happening here?
- Track build time for optimization
- Use DevTools for profiling
- Keep builds under 16ms
- Avoid heavy computations
- Rebuild only what changed

---

# Build in Different Scenarios

> **Build behavior** varies based on context.

```dart
// 1. Hot Reload Build
class HotReloadWidget extends StatelessWidget {
  const HotReloadWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // During hot reload:
    // 1. Widget rebuilds
    // 2. State is preserved
    // 3. Elements are updated
    // 4. Render tree is updated
    
    return const Text('Hot reload works!');
  }
}

// 2. Release Build
class ReleaseWidget extends StatelessWidget {
  const ReleaseWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // In release mode:
    // 1. Assertions are removed
    // 2. Debug flags are disabled
    // 3. Optimized build
    // 4. Faster performance
    
    return const Text('Release mode');
  }
}

// 3. Profile Build
class ProfileWidget extends StatelessWidget {
  const ProfileWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // In profile mode:
    // 1. Debugging disabled
    // 2. Performance tracking
    // 3. Tracing enabled
    // 4. Best for performance testing
    
    return const Text('Profile mode');
  }
}

// 4. Debug Build
class DebugBuildWidget extends StatelessWidget {
  const DebugBuildWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // In debug mode:
    // 1. Assertions enabled
    // 2. Debug flags on
    // 3. Slower performance
    // 4. Better error messages
    
    assert(true, 'Debug mode');
    return const Text('Debug mode');
  }
}
```

What's happening here?
- Different builds for different purposes
- Hot reload preserves state
- Release mode is optimized
- Profile mode for performance
- Debug mode for development

---

# Best Practices

## Keep Build Methods Pure

```dart
// Good - Pure build method
@override
Widget build(BuildContext context) {
  return Text('Hello');
}

// Bad - Side effects in build
@override
Widget build(BuildContext context) {
  fetchData(); // Side effect - DON'T DO THIS
  return Text('Hello');
}
```

## Use const Where Possible

```dart
// Good - const widgets
@override
Widget build(BuildContext context) {
  return const Padding(
    padding: EdgeInsets.all(8.0),
    child: Text('Hello'),
  );
}

// Bad - Not using const
@override
Widget build(BuildContext context) {
  return Padding(
    padding: EdgeInsets.all(8.0), // Creates new object every time
    child: Text('Hello'),
  );
}
```

## Split Large Widgets

```dart
// Good - Split widgets
class ParentWidget extends StatelessWidget {
  const ParentWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const HeaderWidget(),
        const ContentWidget(),
        const FooterWidget(),
      ],
    );
  }
}

// Bad - One large widget
class LargeWidget extends StatelessWidget {
  const LargeWidget({super.key});

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

## Doing Heavy Work in Build

Wrong:
```dart
@override
Widget build(BuildContext context) {
  final data = heavyComputation(); // Runs every build
  return Text(data);
}
```

Correct:
```dart
final data = heavyComputation(); // Runs once

@override
Widget build(BuildContext context) {
  return Text(data);
}
```

## Not Using Keys

Wrong:
```dart
// No keys for list items
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return Text(items[index]);
  },
)
```

Correct:
```dart
// Keys for list items
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return Text(
      items[index],
      key: ValueKey(index),
    );
  },
)
```

---

# Summary

The build process creates and updates widgets based on state changes. Understanding when and how builds occur helps optimize performance. Keep build methods pure, use const widgets, split large widgets, and avoid heavy computations.

---

# Next Steps

- [Rendering Pipeline](rendering-pipeline.md)
- [Frame Scheduling](frame-scheduling.md)
- [Constraints](constraints.md)

---

# Did You Know?

- Build methods are called every frame for dirty widgets
- Const widgets are pre-compiled and never rebuild
- Builds are batched together for efficiency
- Widgets can rebuild multiple times per frame
- The build process is optimized for performance
- Hot reload creates a fresh build
- Release builds skip debug checks