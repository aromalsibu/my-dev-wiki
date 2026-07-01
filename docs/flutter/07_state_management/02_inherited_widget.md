# InheritedWidget

Understand how to share data down the widget tree using InheritedWidget.

---

# What is it?

InheritedWidget is a special type of widget that allows data to be passed down the widget tree efficiently. It enables widgets to access shared data without having to pass it through every parent widget. InheritedWidget is the foundation for many state management solutions in Flutter, including Provider, Riverpod, and BLoC.

---

# Why does it exist?

InheritedWidget exists to:

- Share data down the widget tree
- Avoid passing data through intermediate widgets
- Enable efficient updates and rebuilds
- Provide a foundation for state management
- Support reactive programming patterns
- Reduce boilerplate code
- Enable global app state access

---

# Basic InheritedWidget

> **Creating and using** an InheritedWidget.

```dart
// Basic InheritedWidget implementation
class MyInheritedWidget extends InheritedWidget {
  const MyInheritedWidget({
    super.key,
    required this.data,
    required super.child,
  });

  final String data;

  // Static method to access the data
  static MyInheritedWidget? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<MyInheritedWidget>();
  }

  // Determine if widget needs to rebuild dependents
  @override
  bool updateShouldNotify(MyInheritedWidget oldWidget) {
    return data != oldWidget.data;
  }
}

// Using the InheritedWidget
class InheritedWidgetExample extends StatelessWidget {
  const InheritedWidgetExample({super.key});

  @override
  Widget build(BuildContext context) {
    return MyInheritedWidget(
      data: 'Shared Data',
      child: MaterialApp(
        home: const HomeScreen(),
      ),
    );
  }
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('InheritedWidget'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Access inherited data
            Text(
              'Data: ${MyInheritedWidget.of(context)?.data ?? 'No data'}',
              style: const TextStyle(fontSize: 20),
            ),
            const SizedBox(height: 16),
            const ChildWidget(),
          ],
        ),
      ),
    );
  }
}

class ChildWidget extends StatelessWidget {
  const ChildWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // Access inherited data from anywhere in the tree
    final data = MyInheritedWidget.of(context)?.data;
    
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.blue[100],
      child: Text(
        'Child Widget: $data',
        style: const TextStyle(fontSize: 16),
      ),
    );
  }
}
```

What's happening here?
- InheritedWidget holds shared data
- of() method provides access to data
- updateShouldNotify controls rebuilds
- Children can access data without props

---

# InheritedWidget with Mutable State

> **Managing mutable** state with InheritedWidget.

```dart
// Mutable state with InheritedWidget
class CounterState extends InheritedWidget {
  const CounterState({
    super.key,
    required this.data,
    required this.onIncrement,
    required super.child,
  });

  final int data;
  final VoidCallback onIncrement;

  static CounterState? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<CounterState>();
  }

  @override
  bool updateShouldNotify(CounterState oldWidget) {
    return data != oldWidget.data;
  }
}

class CounterInheritedExample extends StatefulWidget {
  const CounterInheritedExample({super.key});

  @override
  State<CounterInheritedExample> createState() => _CounterInheritedExampleState();
}

class _CounterInheritedExampleState extends State<CounterInheritedExample> {
  int _counter = 0;

  void _increment() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return CounterState(
      data: _counter,
      onIncrement: _increment,
      child: MaterialApp(
        home: const CounterScreen(),
      ),
    );
  }
}

class CounterScreen extends StatelessWidget {
  const CounterScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Inherited Counter'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const CounterDisplay(),
            const SizedBox(height: 16),
            const CounterButton(),
          ],
        ),
      ),
    );
  }
}

class CounterDisplay extends StatelessWidget {
  const CounterDisplay({super.key});

  @override
  Widget build(BuildContext context) {
    final counter = CounterState.of(context);
    
    return Text(
      'Count: ${counter?.data ?? 0}',
      style: const TextStyle(fontSize: 24),
    );
  }
}

class CounterButton extends StatelessWidget {
  const CounterButton({super.key});

  @override
  Widget build(BuildContext context) {
    final counter = CounterState.of(context);
    
    return ElevatedButton(
      onPressed: counter?.onIncrement,
      child: const Text('Increment'),
    );
  }
}
```

What's happening here?
- State lives in parent widget
- InheritedWidget provides access
- Callbacks update state
- Rebuilds propagate efficiently

---

# InheritedWidget with Multiple Values

> **Sharing multiple** values through InheritedWidget.

```dart
// Multiple values in InheritedWidget
class AppState extends InheritedWidget {
  const AppState({
    super.key,
    required this.user,
    required this.theme,
    required this.onThemeChange,
    required super.child,
  });

  final User user;
  final ThemeData theme;
  final ValueChanged<ThemeMode> onThemeChange;

  static AppState? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<AppState>();
  }

  @override
  bool updateShouldNotify(AppState oldWidget) {
    return user != oldWidget.user || 
           theme != oldWidget.theme;
  }
}

class User {
  final String name;
  final String email;

  const User({required this.name, required this.email});

  @override
  bool operator ==(Object other) {
    return other is User && 
           other.name == name && 
           other.email == email;
  }

  @override
  int get hashCode => name.hashCode ^ email.hashCode;
}

class MultiValueInheritedExample extends StatefulWidget {
  const MultiValueInheritedExample({super.key});

  @override
  State<MultiValueInheritedExample> createState() => _MultiValueInheritedExampleState();
}

class _MultiValueInheritedExampleState extends State<MultiValueInheritedExample> {
  User _user = const User(name: 'John Doe', email: 'john@example.com');
  ThemeMode _themeMode = ThemeMode.light;
  ThemeData _themeData = ThemeData.light();

  void _updateUser() {
    setState(() {
      _user = User(
        name: 'Jane Doe',
        email: 'jane@example.com',
      );
    });
  }

  void _updateTheme() {
    setState(() {
      _themeMode = _themeMode == ThemeMode.light 
          ? ThemeMode.dark 
          : ThemeMode.light;
      _themeData = _themeMode == ThemeMode.light 
          ? ThemeData.light() 
          : ThemeData.dark();
    });
  }

  @override
  Widget build(BuildContext context) {
    return AppState(
      user: _user,
      theme: _themeData,
      onThemeChange: (_) => _updateTheme(),
      child: MaterialApp(
        theme: _themeData,
        home: const MultiValueScreen(),
      ),
    );
  }
}

class MultiValueScreen extends StatelessWidget {
  const MultiValueScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final appState = AppState.of(context);
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Multi-Value State'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Card(
              child: ListTile(
                title: Text(appState?.user.name ?? ''),
                subtitle: Text(appState?.user.email ?? ''),
                leading: const Icon(Icons.person),
              ),
            ),
            const SizedBox(height: 16),
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: appState?.onThemeChange,
                    child: const Text('Toggle Theme'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: () {
                      appState?._updateUser(); // Access from state
                    },
                    child: const Text('Update User'),
                  ),
                ),
              ],
            ),
            // Theme info
            const SizedBox(height: 16),
            Text(
              'Current Theme: ${appState?.theme.brightness == Brightness.light ? 'Light' : 'Dark'}',
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- Multiple values in one InheritedWidget
- Different data types
- Callbacks for updates
- Theme and user state management

---

# InheritedWidget with Listeners

> **Adding listeners** to InheritedWidget.

```dart
// InheritedWidget with change listeners
class DataChangeNotifier extends ChangeNotifier {
  String _data = 'Initial Data';
  
  String get data => _data;
  
  void updateData(String newData) {
    _data = newData;
    notifyListeners();
  }
}

class DataProvider extends InheritedWidget {
  const DataProvider({
    super.key,
    required this.data,
    required super.child,
  });

  final DataChangeNotifier data;

  static DataChangeNotifier? of(BuildContext context) {
    return context
        .dependOnInheritedWidgetOfExactType<DataProvider>()
        ?.data;
  }

  @override
  bool updateShouldNotify(DataProvider oldWidget) {
    return data != oldWidget.data;
  }
}

class ListenerInheritedExample extends StatefulWidget {
  const ListenerInheritedExample({super.key});

  @override
  State<ListenerInheritedExample> createState() => _ListenerInheritedExampleState();
}

class _ListenerInheritedExampleState extends State<ListenerInheritedExample> {
  final DataChangeNotifier _dataNotifier = DataChangeNotifier();

  @override
  void dispose() {
    _dataNotifier.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return DataProvider(
      data: _dataNotifier,
      child: MaterialApp(
        home: const ListenerScreen(),
      ),
    );
  }
}

class ListenerScreen extends StatelessWidget {
  const ListenerScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final dataNotifier = DataProvider.of(context);
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Inherited with Listeners'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Display data with listener
            ValueListenableBuilder(
              valueListenable: dataNotifier!,
              builder: (context, value, child) {
                return Card(
                  child: Padding(
                    padding: const EdgeInsets.all(16),
                    child: Text(
                      'Data: ${dataNotifier.data}',
                      style: const TextStyle(fontSize: 20),
                    ),
                  ),
                );
              },
            ),
            const SizedBox(height: 16),
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: () {
                      dataNotifier.updateData('Updated at ${DateTime.now()}');
                    },
                    child: const Text('Update Data'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: () {
                      dataNotifier.updateData('Reset');
                    },
                    child: const Text('Reset'),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

What's happening here?
- ChangeNotifier for data updates
- ValueListenableBuilder for reactive UI
- Clean separation of data and UI
- Automatic rebuilds on data change

---

# Real-World Examples

> **Common patterns** with InheritedWidget.

```dart
// 1. Theme provider
class ThemeProvider extends InheritedWidget {
  const ThemeProvider({
    super.key,
    required this.themeData,
    required this.toggleTheme,
    required super.child,
  });

  final ThemeData themeData;
  final VoidCallback toggleTheme;

  static ThemeProvider? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<ThemeProvider>();
  }

  @override
  bool updateShouldNotify(ThemeProvider oldWidget) {
    return themeData != oldWidget.themeData;
  }
}

// 2. User provider
class UserProvider extends InheritedWidget {
  const UserProvider({
    super.key,
    required this.user,
    required this.updateUser,
    required super.child,
  });

  final User user;
  final Function(User) updateUser;

  static UserProvider? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<UserProvider>();
  }

  @override
  bool updateShouldNotify(UserProvider oldWidget) {
    return user != oldWidget.user;
  }
}

// 3. Locale provider
class LocaleProvider extends InheritedWidget {
  const LocaleProvider({
    super.key,
    required this.locale,
    required this.updateLocale,
    required super.child,
  });

  final Locale locale;
  final Function(Locale) updateLocale;

  static LocaleProvider? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<LocaleProvider>();
  }

  @override
  bool updateShouldNotify(LocaleProvider oldWidget) {
    return locale != oldWidget.locale;
  }
}

// Combined providers
class RootProvider extends StatelessWidget {
  const RootProvider({
    super.key,
    required this.child,
  });

  final Widget child;

  @override
  Widget build(BuildContext context) {
    // Combine multiple providers
    return ThemeProvider(
      themeData: ThemeData.light(),
      toggleTheme: () {},
      child: UserProvider(
        user: const User(name: 'John', email: 'john@example.com'),
        updateUser: (user) {},
        child: LocaleProvider(
          locale: const Locale('en'),
          updateLocale: (locale) {},
          child: child,
        ),
      ),
    );
  }
}
```

What's happening here?
- Theme, User, Locale providers
- Combined providers pattern
- Reusable provider widgets
- Type-safe data access

---

# Best Practices

## Use of() Method

```dart
// Good - Static of method
class MyInheritedWidget extends InheritedWidget {
  static MyInheritedWidget? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<MyInheritedWidget>();
  }
}

// Bad - Direct access
MyInheritedWidget? widget = context.dependOnInheritedWidgetOfExactType<MyInheritedWidget>();
```

## Implement updateShouldNotify Correctly

```dart
// Good - Only rebuild when needed
@override
bool updateShouldNotify(MyInheritedWidget oldWidget) {
  return data != oldWidget.data;
}

// Bad - Always rebuild
@override
bool updateShouldNotify(MyInheritedWidget oldWidget) {
  return true; // Rebuilds unnecessarily
}
```

## Use const Constructors

```dart
// Good - const constructor
const MyInheritedWidget({
  super.key,
  required this.data,
  required super.child,
});
```

---

# Common Mistakes

## Not Calling dependOnInheritedWidgetOfExactType

Wrong:
```dart
// Doesn't create dependency
static MyInheritedWidget? of(BuildContext context) {
  return context.findAncestorWidgetOfExactType<MyInheritedWidget>();
}
```

Correct:
```dart
// Creates dependency
static MyInheritedWidget? of(BuildContext context) {
  return context.dependOnInheritedWidgetOfExactType<MyInheritedWidget>();
}
```

## Forgetting to Update on Changes

Wrong:
```dart
// No update notify
class MyWidget extends InheritedWidget {
  final String data;
  // updateShouldNotify missing
}
```

Correct:
```dart
// With update notification
class MyWidget extends InheritedWidget {
  final String data;
  
  @override
  bool updateShouldNotify(MyWidget oldWidget) {
    return data != oldWidget.data;
  }
}
```

---

# Summary

InheritedWidget efficiently shares data down the widget tree. It provides a foundation for state management in Flutter, enabling widgets to access shared data without passing it through intermediate widgets. Use it for app-wide data like themes, user info, and settings.

---

# Next Steps

- [InheritedModel](inheritedmodel.md)
- [ValueNotifier](valuenotifier.md)
- [ChangeNotifier](changenotifier.md)

---

# Did You Know?

- InheritedWidget is the foundation of Provider
- dependOnInheritedWidgetOfExactType creates dependencies
- updateShouldNotify controls rebuilds
- InheritedWidgets are immutable
- Widgets rebuild when data changes
- InheritedWidget avoids passing data through props
- Multiple InheritedWidgets can be combined
- InheritedWidget is efficient and performant