# Restoration

Understand how to preserve and restore state in Flutter applications.

---

# What is it?

Restoration is a mechanism in Flutter that allows you to save and restore application state across app restarts, process kills, or device reboots. It enables users to pick up exactly where they left off, preserving navigation stack, form data, scroll positions, and other user interactions. This is essential for creating professional, user-friendly applications.

---

# Why does it exist?

Restoration exists to:

- Preserve user state across app restarts
- Provide seamless user experience
- Handle system-initiated app kills
- Support Android's "Don't keep activities"
- Enable state persistence in iOS
- Improve user retention and satisfaction
- Allow users to continue from where they left off

---

# Restoration Basics

> **Understanding** state restoration in Flutter.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Main application entry point with restoration support
void main() {
  runApp(const MyApp());
}

/// Root application widget that enables restoration
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Restoration Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      
      // Enable restoration by providing a RestorationScope
      // This allows child widgets to participate in state restoration
      restorationScopeId: 'app',
      
      // Set the home widget that will be restored
      home: const HomeScreen(),
    );
  }
}

/// Home screen with restoration support
class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

/// State with restoration capability
class _HomeScreenState extends State<HomeScreen> with RestorationMixin {
  // 1. Provide a restoration ID for this widget
  // This ID is used to identify saved state for this widget
  @override
  String get restorationId => 'home_screen';

  // 2. Create Restorable values
  // These values will be automatically saved and restored
  
  // Restorable integer for counter
  // The initial value is 0, which is restored if no saved state exists
  final RestorableInt _counter = RestorableInt(0);
  
  // Restorable string for text input
  final RestorableString _text = RestorableString('');

  // Restorable boolean for toggle
  final RestorableBool _isEnabled = RestorableBool(false);

  // 3. Register restorable properties
  @override
  void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
    // Register each restorable property with a unique key
    // The key identifies the property in the saved state
    registerForRestoration(_counter, 'counter');
    registerForRestoration(_text, 'text');
    registerForRestoration(_isEnabled, 'is_enabled');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Restoration Demo'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Counter display and controls
            // Shows the restored counter value
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    // Display current counter value
                    // This value is restored when the app restarts
                    Text(
                      'Counter: ${_counter.value}',
                      style: const TextStyle(fontSize: 24),
                    ),
                    const SizedBox(height: 8),
                    
                    // Increment and decrement buttons
                    // Changes to _counter are automatically saved
                    Row(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        // Decrement button
                        ElevatedButton(
                          onPressed: () {
                            // Update the counter value
                            // This change is automatically saved
                            _counter.value--;
                          },
                          child: const Text('-'),
                        ),
                        const SizedBox(width: 8),
                        
                        // Increment button
                        ElevatedButton(
                          onPressed: () {
                            // Update the counter value
                            // This change is automatically saved
                            _counter.value++;
                          },
                          child: const Text('+'),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // Text input restoration
            // User input is preserved across app restarts
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    // Display current text value
                    // This is restored when the app restarts
                    Text(
                      'Text: ${_text.value.isEmpty ? 'Empty' : _text.value}',
                    ),
                    const SizedBox(height: 8),
                    
                    // Text field with restoration
                    TextField(
                      // The controller is not restored directly
                      // Instead, we use _text to store the value
                      onChanged: (value) {
                        // Update the restorable string
                        // This change is automatically saved
                        _text.value = value;
                      },
                      decoration: const InputDecoration(
                        border: OutlineInputBorder(),
                        hintText: 'Enter some text...',
                      ),
                    ),
                  ],
                ),
              ),
            ),
            
            const SizedBox(height: 16),
            
            // Toggle switch restoration
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: SwitchListTile(
                  title: const Text('Enabled'),
                  subtitle: Text(
                    _isEnabled.value ? 'ON' : 'OFF',
                  ),
                  // The switch value is restored automatically
                  value: _isEnabled.value,
                  onChanged: (value) {
                    // Update the restorable boolean
                    // This change is automatically saved
                    _isEnabled.value = value;
                  },
                ),
              ),
            ),
            
            const Spacer(),
            
            // Info about restoration
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Text(
                'Try restarting the app - your state will be preserved!',
                textAlign: TextAlign.center,
                style: TextStyle(fontSize: 14),
              ),
            ),
          ],
        ),
      ),
    );
  }

  // 4. Optional: Clean up restorable objects
  // Called when the widget is disposed
  @override
  void dispose() {
    // Dispose of restorable objects to prevent memory leaks
    _counter.dispose();
    _text.dispose();
    _isEnabled.dispose();
    super.dispose();
  }
}
```

What's happening here?
- RestorationMixin enables state restoration
- RestorableInt, RestorableString, RestorableBool store values
- registerForRestoration connects values to restoration IDs
- dispose() prevents memory leaks
- State is automatically saved and restored

---

# Restoration with Navigation

> **Restoring navigation** stack and routes.

```dart
/// Detail screen with restoration support
class DetailScreen extends StatefulWidget {
  const DetailScreen({super.key});

  @override
  State<DetailScreen> createState() => _DetailScreenState();
}

/// State with restoration support for navigation
class _DetailScreenState extends State<DetailScreen> with RestorationMixin {
  // Restoration ID for this screen
  @override
  String get restorationId => 'detail_screen';

  // Restorable string for the item title
  final RestorableString _itemTitle = RestorableString('Item');

  // Restorable integer for the item index
  final RestorableInt _itemIndex = RestorableInt(0);

  @override
  void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
    // Register both restorable values
    registerForRestoration(_itemTitle, 'item_title');
    registerForRestoration(_itemIndex, 'item_index');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Detail Screen'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Display the restored item info
            Text(
              'Item: ${_itemTitle.value}',
              style: const TextStyle(fontSize: 24),
            ),
            Text(
              'Index: ${_itemIndex.value}',
              style: const TextStyle(fontSize: 18),
            ),
            const SizedBox(height: 16),
            
            // Buttons to modify the item
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  // Change the item title
                  onPressed: () {
                    _itemTitle.value = 'Item ${_itemIndex.value + 1}';
                  },
                  child: const Text('Change Title'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  // Increment the item index
                  onPressed: () {
                    _itemIndex.value++;
                  },
                  child: const Text('Next Item'),
                ),
              ],
            ),
            const SizedBox(height: 16),
            
            // Back button
            ElevatedButton(
              onPressed: () {
                // Pop the screen
                Navigator.pop(context);
              },
              child: const Text('Go Back'),
            ),
          ],
        ),
      ),
    );
  }

  @override
  void dispose() {
    _itemTitle.dispose();
    _itemIndex.dispose();
    super.dispose();
  }
}

/// Main app with restoration-enabled navigation
class NavigationRestorationApp extends StatelessWidget {
  const NavigationRestorationApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Navigation Restoration',
      restorationScopeId: 'app',
      home: const NavigationHomeScreen(),
      // Use onGenerateRoute for restoration
      // This ensures restoration works with navigation
      onGenerateRoute: (settings) {
        // If we have restoration data, we can restore specific screens
        if (settings.name == '/detail') {
          return MaterialPageRoute(
            builder: (context) => const DetailScreen(),
            settings: settings, // Pass settings for restoration
          );
        }
        return null;
      },
    );
  }
}

/// Home screen that navigates to detail screen
class NavigationHomeScreen extends StatelessWidget {
  const NavigationHomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Navigation Restoration'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'This app preserves navigation state',
              style: TextStyle(fontSize: 18),
            ),
            const SizedBox(height: 16),
            
            // Navigate to detail screen
            ElevatedButton(
              onPressed: () {
                // Push the detail screen
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const DetailScreen(),
                    // Important: Pass the settings for restoration
                    settings: const RouteSettings(name: '/detail'),
                  ),
                );
              },
              child: const Text('Go to Detail'),
            ),
            
            const SizedBox(height: 24),
            
            // Info box
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Text(
                'The navigation stack is preserved across app restarts',
                textAlign: TextAlign.center,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Navigation stack is preserved
- Detail screen state is restored
- RouteSettings enable restoration
- onGenerateRoute handles restoration

---

# Restoration with Scroll Position

> **Restoring scroll** position in lists.

```dart
/// List screen with scroll position restoration
class ScrollRestorationScreen extends StatefulWidget {
  const ScrollRestorationScreen({super.key});

  @override
  State<ScrollRestorationScreen> createState() => _ScrollRestorationScreenState();
}

/// State with scroll position restoration
class _ScrollRestorationScreenState extends State<ScrollRestorationScreen> 
    with RestorationMixin {
  // Restoration ID for this screen
  @override
  String get restorationId => 'scroll_screen';

  // Scroll controller for the list
  // We'll save and restore its position
  final ScrollController _scrollController = ScrollController();

  // Restorable value for the scroll position
  // This stores the current scroll offset
  final RestorableDouble _scrollOffset = RestorableDouble(0);

  @override
  void initState() {
    super.initState();
    
    // Listen to scroll changes
    _scrollController.addListener(() {
      // Update the restorable scroll offset
      // This value is automatically saved
      _scrollOffset.value = _scrollController.position.pixels;
    });
  }

  @override
  void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
    // Register the scroll offset for restoration
    registerForRestoration(_scrollOffset, 'scroll_offset');
    
    // If we have a saved scroll offset, restore it
    // This ensures the list scrolls to the saved position
    WidgetsBinding.instance.addPostFrameCallback((_) {
      // Jump to the saved scroll position
      _scrollController.jumpTo(_scrollOffset.value);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Scroll Restoration'),
      ),
      body: Column(
        children: [
          // Scroll position indicator
          Container(
            padding: const EdgeInsets.all(8),
            color: Colors.blue[50],
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                const Text(
                  'Scroll Position:',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                Text(
                  _scrollOffset.value.toStringAsFixed(2),
                  style: const TextStyle(fontWeight: FontWeight.bold),
                ),
              ],
            ),
          ),
          
          // Scrollable list
          Expanded(
            child: ListView.builder(
              controller: _scrollController,
              itemCount: 100,
              itemBuilder: (context, index) {
                return Container(
                  height: 60,
                  margin: const EdgeInsets.all(4),
                  decoration: BoxDecoration(
                    color: Colors.blue[100 * (index % 9 + 1)],
                    borderRadius: BorderRadius.circular(4),
                  ),
                  child: Center(
                    child: Text(
                      'Item $index',
                      style: const TextStyle(
                        fontWeight: FontWeight.bold,
                        color: Colors.white,
                      ),
                    ),
                  ),
                );
              },
            ),
          ),
          
          // Control buttons
          Container(
            padding: const EdgeInsets.all(8),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Scroll to top
                ElevatedButton(
                  onPressed: () {
                    _scrollController.animateTo(
                      0,
                      duration: const Duration(milliseconds: 500),
                      curve: Curves.easeInOut,
                    );
                  },
                  child: const Text('Top'),
                ),
                const SizedBox(width: 8),
                
                // Scroll to middle
                ElevatedButton(
                  onPressed: () {
                    // Scroll to the middle of the list
                    final maxScroll = _scrollController.position.maxScrollExtent;
                    _scrollController.animateTo(
                      maxScroll / 2,
                      duration: const Duration(milliseconds: 500),
                      curve: Curves.easeInOut,
                    );
                  },
                  child: const Text('Middle'),
                ),
                const SizedBox(width: 8),
                
                // Scroll to bottom
                ElevatedButton(
                  onPressed: () {
                    // Scroll to the bottom of the list
                    final maxScroll = _scrollController.position.maxScrollExtent;
                    _scrollController.animateTo(
                      maxScroll,
                      duration: const Duration(milliseconds: 500),
                      curve: Curves.easeInOut,
                    );
                  },
                  child: const Text('Bottom'),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  @override
  void dispose() {
    // Dispose controllers and restorable objects
    _scrollController.dispose();
    _scrollOffset.dispose();
    super.dispose();
  }
}
```

What's happening here?
- Scroll position is saved as RestorableDouble
- Scroll listener updates the saved position
- Post-frame callback restores scroll position
- List continues where user left off

---

# Restoration with Forms

> **Restoring form** data and validation state.

```dart
/// Form screen with restoration support
class FormRestorationScreen extends StatefulWidget {
  const FormRestorationScreen({super.key});

  @override
  State<FormRestorationScreen> createState() => _FormRestorationScreenState();
}

/// State with full form restoration
class _FormRestorationScreenState extends State<FormRestorationScreen> 
    with RestorationMixin {
  // Restoration ID
  @override
  String get restorationId => 'form_screen';

  // Form key for validation
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  // Restorable form fields
  // Each field's value is saved and restored
  final RestorableString _name = RestorableString('');
  final RestorableString _email = RestorableString('');
  final RestorableString _phone = RestorableString('');
  final RestorableBool _agreeToTerms = RestorableBool(false);

  // Non-restorable state (UI only)
  bool _submitted = false;

  @override
  void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
    // Register all form fields for restoration
    registerForRestoration(_name, 'form_name');
    registerForRestoration(_email, 'form_email');
    registerForRestoration(_phone, 'form_phone');
    registerForRestoration(_agreeToTerms, 'form_agree_terms');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Form Restoration'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              // Name field
              // The value is restored when the app restarts
              TextFormField(
                initialValue: _name.value,
                decoration: const InputDecoration(
                  labelText: 'Name',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your name';
                  }
                  return null;
                },
                onChanged: (value) {
                  // Save the value for restoration
                  _name.value = value;
                },
              ),
              const SizedBox(height: 16),
              
              // Email field
              TextFormField(
                initialValue: _email.value,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your email';
                  }
                  if (!value.contains('@')) {
                    return 'Please enter a valid email';
                  }
                  return null;
                },
                onChanged: (value) {
                  // Save the value for restoration
                  _email.value = value;
                },
              ),
              const SizedBox(height: 16),
              
              // Phone field
              TextFormField(
                initialValue: _phone.value,
                decoration: const InputDecoration(
                  labelText: 'Phone',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.phone,
                onChanged: (value) {
                  // Save the value for restoration
                  _phone.value = value;
                },
              ),
              const SizedBox(height: 16),
              
              // Terms and conditions checkbox
              CheckboxListTile(
                title: const Text('I agree to the terms and conditions'),
                // The checkbox state is restored
                value: _agreeToTerms.value,
                onChanged: (value) {
                  // Save the value for restoration
                  _agreeToTerms.value = value ?? false;
                },
              ),
              const SizedBox(height: 16),
              
              // Submit button with validation
              ElevatedButton(
                onPressed: () {
                  // Validate the form
                  if (_formKey.currentState!.validate()) {
                    setState(() {
                      _submitted = true;
                    });
                    
                    // Show success message
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('Form submitted successfully!'),
                        backgroundColor: Colors.green,
                      ),
                    );
                  }
                },
                child: const Text('Submit'),
              ),
              
              // Show submission status
              if (_submitted) ...[
                const SizedBox(height: 16),
                Container(
                  padding: const EdgeInsets.all(16),
                  decoration: BoxDecoration(
                    color: Colors.green[50],
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: const Text(
                    'Form has been submitted!',
                    style: TextStyle(color: Colors.green),
                  ),
                ),
              ],
              
              const Spacer(),
              
              // Display current values
              Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.blue[50],
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Column(
                  children: [
                    const Text(
                      'Saved Values:',
                      style: TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 4),
                    Text('Name: ${_name.value}'),
                    Text('Email: ${_email.value}'),
                    Text('Phone: ${_phone.value}'),
                    Text('Agreed: ${_agreeToTerms.value}'),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    // Dispose all restorable objects
    _name.dispose();
    _email.dispose();
    _phone.dispose();
    _agreeToTerms.dispose();
    super.dispose();
  }
}
```

What's happening here?
- All form fields are restored
- Validation state is preserved
- Checkbox state is restored
- User doesn't lose entered data

---

# Best Practices

## Use RestorationMixin Correctly

```dart
// Good - Proper restoration implementation
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> with RestorationMixin {
  @override
  String get restorationId => 'my_widget';
  
  final RestorableInt _value = RestorableInt(0);
  
  @override
  void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
    registerForRestoration(_value, 'value');
  }
}
```

## Dispose Restorable Objects

```dart
// Good - Proper disposal
@override
void dispose() {
  _value.dispose();
  _text.dispose();
  super.dispose();
}
```

## Use Appropriate Restoration IDs

```dart
// Good - Unique restoration IDs
@override
String get restorationId => '${widget.id}_screen';

// Bad - Duplicate IDs can cause conflicts
@override
String get restorationId => 'screen';
```

---

# Common Mistakes

## Forgetting RestorationMixin

Wrong:
```dart
// Missing RestorationMixin
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}
```

Correct:
```dart
// With RestorationMixin
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> with RestorationMixin {
  // Implementation
}
```

## Not Disposing Restorable Objects

Wrong:
```dart
// Memory leak
final RestorableInt _value = RestorableInt(0);
// No dispose
```

Correct:
```dart
// Proper disposal
@override
void dispose() {
  _value.dispose();
  super.dispose();
}
```

---

# Summary

Restoration preserves user state across app restarts. Use RestorationMixin with Restorable types to save and restore UI state, navigation, scroll positions, and form data. This creates a seamless user experience and handles system-initiated app kills gracefully.

---

# Next Steps

- [State Management Overview](state-management-overview.md)
- [Provider](provider.md)
- [Riverpod](riverpod.md)

---

# Did You Know?

- Restoration works across process kills
- Android "Don't keep activities" is handled
- Navigation stack can be restored
- Scroll positions are preserved
- Form data is maintained
- Restorable types are automatically saved
- Restoration IDs must be unique
- dispose() prevents memory leaks