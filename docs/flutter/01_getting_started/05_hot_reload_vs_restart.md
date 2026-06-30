# Hot Reload vs Hot Restart

Understand the difference between hot reload and hot restart in Flutter development.

---

# What is it?

Hot reload and hot restart are two development features in Flutter that help you see changes quickly. Hot reload updates your app instantly while preserving state, whereas hot restart resets the entire application state but loads changes faster than a full rebuild.

---

# Why does it exist?

Hot reload and hot restart exist to:

- Speed up development by avoiding full rebuilds
- Preserve or reset state based on development needs
- Provide immediate feedback during coding
- Reduce development time significantly
- Enable rapid experimentation and iteration
- Maintain developer flow and productivity

---

# Hot Reload

> **Hot reload** injects updated source code files into the running Dart VM without restarting the app.

```dart
// Original code
class MyHomePage extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Counter App'),
      ),
      body: Center(
        child: Text('Hello World'),
      ),
    );
  }
}

// After hot reload - update the text
class MyHomePage extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Counter App'),
      ),
      body: Center(
        child: Text('Hello Flutter!'), // Changed text
      ),
    );
  }
}

// Save file (Ctrl+S / Cmd+S)
// or press 'r' in terminal
// App updates instantly - state preserved!
```

What's happening here?
- Hot reload injects new code into running app
- State of the app is preserved
- UI updates within 1-2 seconds
- Most changes are hot reloadable
- Does not restart the app
- Maintains navigation stack and variables

---

# Hot Restart

> **Hot restart** resets the app state and rebuilds the entire widget tree from scratch.

```dart
// Original code with counter state
int _counter = 0;

void _incrementCounter() {
  setState(() {
    _counter++;
  });
}

// After hot restart
// The counter resets to 0
// The app rebuilds from the beginning
// State is completely cleared

// Press 'R' in terminal (Shift+R)
// App restarts with state reset
// But faster than full app restart
```

What's happening here?
- Hot restart destroys all state
- Rebuilds widget tree from scratch
- Navigation stack is cleared
- Variables reset to initial values
- Faster than full app restart
- Useful when hot reload doesn't work

---

# When to Use Hot Reload

> **Hot reload is ideal for** UI changes that don't affect the app state.

**Good for hot reload:**

```dart
// 1. Changing text content
Text('Hello World') → Text('Hello Flutter')

// 2. Changing colors
Colors.blue → Colors.red

// 3. Changing layout
Column → Row

// 4. Adding simple widgets
Container() → Container(padding: EdgeInsets.all(8))

// 5. Changing styles
style: TextStyle(fontSize: 16) → style: TextStyle(fontSize: 20)
```

**Also good for:**

```dart
// 6. Adding new functions that don't affect state
void _helperFunction() { ... }

// 7. Modifying business logic
bool isValid → bool isActive

// 8. Updating asset references
Image.asset('assets/logo.png') → Image.asset('assets/new_logo.png')

// 9. Changing theme values
primaryColor: Colors.blue → primaryColor: Colors.green
```

What's happening here?
- Hot reload preserves user session
- Keeps navigation history intact
- Maintains form inputs and selections
- Stateful widgets keep their data
- Great for rapid UI iteration

---

# When to Use Hot Restart

> **Hot restart is needed when** changes affect the app state or initialization.

**Required for hot restart:**

```dart
// 1. Changing initState() methods
@override
void initState() {
  super.initState();
  fetchData(); // Changes to initialization
}

// 2. Changing global variables
var apiUrl = 'https://api.example.com'; // Changed URL

// 3. Changing main() function
void main() {
  runApp(MyApp()); // Changed to run different app
}

// 4. Changing app-level providers
runApp(Provider(create: (_) => AuthService()) // New provider

// 5. Changing theme initialization
theme: ThemeData.dark() → theme: ThemeData.light()

// 6. Navigation structure changes
home: HomeScreen() → home: LoginScreen()
```

**Also needed for:**

```dart
// 7. Changing static variables
static const String appName = 'MyApp'; // Changed

// 8. Changing app configuration
SharedPreferences preferences; // New initialization

// 9. State model changes
class User { String name; } → class User { String name; int age; }
```

What's happening here?
- Hot restart resets everything
- Re-runs all initializations
- Useful after major changes
- Ensures clean state
- Verifies app starts correctly

---

# Comparison Table

| Feature | Hot Reload | Hot Restart | Full Restart |
|---------|------------|-------------|--------------|
| Speed | Fastest (1-2s) | Medium (3-5s) | Slow (10-30s+) |
| Preserves State | ✅ Yes | ❌ No | ❌ No |
| Preserves Navigation | ✅ Yes | ❌ No | ❌ No |
| Rebuilds Widget Tree | ✅ Yes | ✅ Yes | ✅ Yes |
| Re-runs main() | ❌ No | ✅ Yes | ✅ Yes |
| Re-initializes State | ❌ No | ✅ Yes | ✅ Yes |
| Key | 'r' key | 'R' key | 'flutter run' |
| Use Case | UI changes | State changes | Major changes |

---

# Keyboard Shortcuts

> **During development,** use these shortcuts to control the app.

```bash
# In terminal window where 'flutter run' is running:

r   # Hot reload (preserves state)
R   # Hot restart (resets state)
q   # Quit the app
h   # Show help

# Visual Studio Code shortcuts:
Ctrl+S (Windows) / Cmd+S (Mac)    # Hot reload on save
Ctrl+Shift+F5 (Windows)            # Hot restart
F5 (Windows)                       # Start debugging

# Android Studio shortcuts:
Ctrl+S (Windows) / Cmd+S (Mac)    # Hot reload on save
Ctrl+Shift+R (Windows)             # Hot restart
Shift+F10 (Windows)                # Run app
```

What's happening here?
- 'r' is for quick hot reload
- 'R' for complete hot restart
- 'q' to quit the app entirely
- IDEs have their own shortcuts
- Shortcuts work across all platforms

---

# Managing State During Hot Reload

> **Hot reload preserves state,** but you need to handle it properly.

```dart
// Stateful widget with preserved state
class CounterWidget extends StatefulWidget {
  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: () {
            setState(() {
              _count++; // State preserved during hot reload
            });
          },
          child: Text('Increment'),
        ),
      ],
    );
  }
}

// During hot reload:
// - _count value is preserved
// - Button state is preserved
// - UI updates with new changes
// - App stays at current screen
```

What's happening here?
- State values are maintained
- Variables keep their values
- Button presses still work
- No data loss during UI updates
- Perfect for iterative development

---

# Hot Reload Limitations

> **Not all changes** can be hot reloaded.

```dart
// Changes that WON'T work with hot reload:

// 1. Changing main() function
void main() {
  runApp(MyApp()); // Won't update
}

// 2. Changing initState()
@override
void initState() {
  super.initState();
  // Changes here won't apply
}

// 3. Changing app-level initialization
runApp(MyApp()); // Won't reflect

// 4. Changing global variable initializations
String apiKey = 'old_key'; // Won't update

// 5. Changing static fields
static const String appName = 'MyApp'; // Won't change

// 6. Changing assets
Image.asset('assets/logo.png'); // New asset might not load

// 7. Changing app theme
theme: ThemeData.light(); // Won't apply
```

What's happening here?
- Some changes require full restart
- App initialization only runs once
- Static variables are not updated
- Asset changes may not be detected
- Use hot restart for these changes

---

# Best Practices

## Use Hot Reload for UI

```dart
// Good: Use hot reload for UI changes
@override
Widget build(BuildContext context) {
  return Container(
    color: Colors.blue, // Change to Colors.red
    child: Text('Hello'), // Change to 'Hello World'
  );
}
```

## Use Hot Restart for State Changes

```dart
// Good: Use hot restart for state changes
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: HomeScreen(), // Change to LoginScreen()
    );
  }
}
```

## Save Frequently

```dart
// Save files frequently for hot reload
// Enable auto-save in IDE
// Watch for reload messages in terminal
// Check for errors in console
```

---

# Common Mistakes

## Not Checking Hot Reload Status

Wrong:
```dart
// Don't assume hot reload worked
// Look for console messages
// "Reloaded X of Y libraries"
// "Performing hot reload..."
```

Correct:
```bash
# Look for these messages:
Performing hot reload... 
Reloaded 2 of 2 libraries in 1,234ms.
```

---

## Using Hot Reload for Breaking Changes

Wrong:
```dart
// Trying to hot reload initialization changes
@override
void initState() { // Change initialization
  super.initState();
  fetchData(); // This won't run
}
```

Correct:
```dart
// Use hot restart for these changes
@override
void initState() {
  super.initState();
  fetchData(); // Works after hot restart
}
```

---

# Summary

Hot reload is faster and preserves state, making it ideal for UI changes. Hot restart resets the app state and is needed for initialization changes. Understanding when to use each feature speeds up development significantly.

---

# Next Steps

- [Flutter Architecture](../framework-architecture/flutter-architecture.md)
- [What is a Widget?](../widgets/what-is-a-widget.md)
- [StatelessWidget](../widgets/statelesswidget.md)

---

# Did You Know?

- Hot reload can update code in less than a second
- The record for hot reload speed is 0.5 seconds
- Hot reload works on physical devices too
- State restoration can be customized
- Some changes automatically trigger hot reload on save
- You can disable hot reload if needed