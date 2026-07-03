# FutureBuilder

Understand how to handle asynchronous data and future operations in Flutter using FutureBuilder.

---

# What is it?

FutureBuilder is a widget that builds itself based on the latest snapshot of interaction with a Future. It allows you to easily handle asynchronous operations like network requests, database queries, or any other Future-based operation by providing different UI states for loading, error, and data.

---

# Why does it exist?

FutureBuilder exists to:

- Handle asynchronous operations easily
- Manage loading, error, and data states
- Simplify async UI development
- Provide reactive UI updates
- Reduce boilerplate code
- Handle Future operations declaratively

---

# Basic FutureBuilder

> **Creating** a basic FutureBuilder.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic FutureBuilder example
class BasicFutureBuilderExample extends StatelessWidget {
  const BasicFutureBuilderExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('FutureBuilder'),
      ),
      body: Center(
        child: FutureBuilder<String>(
          // 1. The Future that will provide data
          future: _fetchData(),
          
          // 2. Initial data (optional)
          initialData: 'Loading...',
          
          // 3. Builder function that builds UI based on snapshot
          builder: (BuildContext context, AsyncSnapshot<String> snapshot) {
            // 4. Check if data is ready
            if (snapshot.connectionState == ConnectionState.waiting) {
              return const CircularProgressIndicator();
            }
            
            // 5. Check for errors
            if (snapshot.hasError) {
              return Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.error, color: Colors.red, size: 48),
                  const SizedBox(height: 8),
                  Text('Error: ${snapshot.error}'),
                  const SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () {
                      // You could retry here
                    },
                    child: const Text('Retry'),
                  ),
                ],
              );
            }
            
            // 6. Data is ready
            return Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Text(
                  'Data loaded successfully!',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 8),
                Text(
                  snapshot.data ?? 'No data',
                  style: const TextStyle(fontSize: 20),
                ),
              ],
            );
          },
        ),
      ),
    );
  }

  // Simulate an asynchronous operation
  Future<String> _fetchData() async {
    // Simulate network delay
    await Future.delayed(const Duration(seconds: 2));
    return 'Hello, World!';
  }
}
```

What's happening here?
- FutureBuilder builds based on Future state
- AsyncSnapshot provides the current state
- connectionState tracks loading status
- hasError indicates error state
- data contains the result

---

# FutureBuilder with Different States

> **Handling all** Future states.

```dart
/// FutureBuilder with different states
class FutureBuilderStatesExample extends StatefulWidget {
  const FutureBuilderStatesExample({super.key});

  @override
  State<FutureBuilderStatesExample> createState() => _FutureBuilderStatesExampleState();
}

class _FutureBuilderStatesExampleState extends State<FutureBuilderStatesExample> {
  // 1. Different Future examples
  Future<String> _successFuture = _fetchSuccess();
  Future<String> _errorFuture = _fetchError();
  Future<String> _delayedFuture = _fetchDelayed();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('FutureBuilder States'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Success state
            const Text(
              'Success State',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 4),
            _buildSuccessExample(),
            const SizedBox(height: 16),
            
            // 2. Error state
            const Text(
              'Error State',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 4),
            _buildErrorExample(),
            const SizedBox(height: 16),
            
            // 3. Delayed state (shows loading)
            const Text(
              'Delayed State',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 4),
            _buildDelayedExample(),
          ],
        ),
      ),
    );
  }

  Widget _buildSuccessExample() {
    return FutureBuilder<String>(
      future: _successFuture,
      builder: (context, snapshot) {
        // 2. ConnectionState handling
        switch (snapshot.connectionState) {
          case ConnectionState.none:
            return const Text('No connection');
          case ConnectionState.waiting:
            return const SizedBox(
              height: 20,
              width: 20,
              child: CircularProgressIndicator(),
            );
          case ConnectionState.active:
            return const Text('Active');
          case ConnectionState.done:
            if (snapshot.hasError) {
              return Text('Error: ${snapshot.error}');
            }
            return Container(
              padding: const EdgeInsets.all(8),
              color: Colors.green[100],
              child: Text(
                '✅ ${snapshot.data ?? 'No data'}',
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
            );
        }
      },
    );
  }

  Widget _buildErrorExample() {
    return FutureBuilder<String>(
      future: _errorFuture,
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const SizedBox(
            height: 20,
            width: 20,
            child: CircularProgressIndicator(),
          );
        }
        
        if (snapshot.hasError) {
          return Container(
            padding: const EdgeInsets.all(8),
            color: Colors.red[100],
            child: Text(
              '❌ Error: ${snapshot.error}',
              style: const TextStyle(fontWeight: FontWeight.bold, color: Colors.red),
            ),
          );
        }
        
        return Container(
          padding: const EdgeInsets.all(8),
          color: Colors.green[100],
          child: Text('✅ ${snapshot.data ?? 'No data'}'),
        );
      },
    );
  }

  Widget _buildDelayedExample() {
    return FutureBuilder<String>(
      future: _delayedFuture,
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const SizedBox(
            height: 20,
            width: 20,
            child: CircularProgressIndicator(),
          );
        }
        
        if (snapshot.hasError) {
          return Text('Error: ${snapshot.error}');
        }
        
        return Container(
          padding: const EdgeInsets.all(8),
          color: Colors.blue[100],
          child: Text('✅ ${snapshot.data ?? 'No data'}'),
        );
      },
    );
  }

  // Simulate a successful fetch
  static Future<String> _fetchSuccess() async {
    return 'Data loaded successfully!';
  }

  // Simulate an error fetch
  static Future<String> _fetchError() async {
    await Future.delayed(const Duration(milliseconds: 100));
    throw Exception('Failed to load data');
  }

  // Simulate a delayed fetch
  static Future<String> _fetchDelayed() async {
    await Future.delayed(const Duration(seconds: 2));
    return 'Data loaded after delay!';
  }
}
```

What's happening here?
- ConnectionState.none: No future provided
- ConnectionState.waiting: Loading
- ConnectionState.active: Loading with data
- ConnectionState.done: Completed

---

# FutureBuilder with Data Models

> **Using FutureBuilder** with complex data.

```dart
/// Data model for user
class User {
  final int id;
  final String name;
  final String email;

  User({
    required this.id,
    required this.name,
    required this.email,
  });

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }
}

/// FutureBuilder with data models
class FutureBuilderDataModelExample extends StatefulWidget {
  const FutureBuilderDataModelExample({super.key});

  @override
  State<FutureBuilderDataModelExample> createState() => _FutureBuilderDataModelExampleState();
}

class _FutureBuilderDataModelExampleState extends State<FutureBuilderDataModelExample> {
  // 1. Future that returns a list of users
  Future<List<User>> _fetchUsers() async {
    // Simulate network request
    await Future.delayed(const Duration(seconds: 2));
    
    // Simulate response data
    return [
      User(id: 1, name: 'John Doe', email: 'john@example.com'),
      User(id: 2, name: 'Jane Smith', email: 'jane@example.com'),
      User(id: 3, name: 'Bob Johnson', email: 'bob@example.com'),
    ];
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('FutureBuilder with Data'),
        actions: [
          // 2. Refresh button
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () {
              // Trigger rebuild to refetch
              setState(() {});
            },
          ),
        ],
      ),
      body: FutureBuilder<List<User>>(
        future: _fetchUsers(),
        builder: (context, snapshot) {
          // 3. Loading state
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  CircularProgressIndicator(),
                  SizedBox(height: 16),
                  Text('Loading users...'),
                ],
              ),
            );
          }

          // 4. Error state
          if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.error, color: Colors.red, size: 48),
                  const SizedBox(height: 16),
                  Text('Error: ${snapshot.error}'),
                  const SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () {
                      setState(() {});
                    },
                    child: const Text('Retry'),
                  ),
                ],
              ),
            );
          }

          // 5. Data ready
          final users = snapshot.data ?? [];
          
          if (users.isEmpty) {
            return const Center(
              child: Text('No users found'),
            );
          }

          return ListView.builder(
            itemCount: users.length,
            itemBuilder: (context, index) {
              final user = users[index];
              return Card(
                margin: const EdgeInsets.symmetric(
                  horizontal: 16,
                  vertical: 4,
                ),
                child: ListTile(
                  leading: CircleAvatar(
                    child: Text('${user.id}'),
                  ),
                  title: Text(user.name),
                  subtitle: Text(user.email),
                  trailing: const Icon(Icons.arrow_forward),
                  onTap: () {
                    // Navigate to user detail
                  },
                ),
              );
            },
          );
        },
      ),
    );
  }
}
```

What's happening here?
- Future returns List<User>
- Loading, error, and data states
- Refresh functionality
- Data display with ListView

---

# Real-World Examples

> **Common patterns** with FutureBuilder.

```dart
/// 1. API call with caching
class ApiCallWithCache extends StatefulWidget {
  const ApiCallWithCache({super.key});

  @override
  State<ApiCallWithCache> createState() => _ApiCallWithCacheState();
}

class _ApiCallWithCacheState extends State<ApiCallWithCache> {
  // Cache the result
  Future<String>? _cachedFuture;

  @override
  void initState() {
    super.initState();
    _cachedFuture = _fetchData();
  }

  Future<String> _fetchData() async {
    // Simulate API call
    await Future.delayed(const Duration(seconds: 2));
    return 'Data from API: ${DateTime.now().toLocal()}';
  }

  void _refresh() {
    setState(() {
      _cachedFuture = _fetchData();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('API with Cache'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _refresh,
          ),
        ],
      ),
      body: FutureBuilder<String>(
        future: _cachedFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  CircularProgressIndicator(),
                  SizedBox(height: 16),
                  Text('Loading data...'),
                ],
              ),
            );
          }

          if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.error, color: Colors.red, size: 48),
                  const SizedBox(height: 16),
                  Text('Error: ${snapshot.error}'),
                  const SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: _refresh,
                    child: const Text('Retry'),
                  ),
                ],
              ),
            );
          }

          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Icon(Icons.check_circle, color: Colors.green, size: 48),
                const SizedBox(height: 16),
                Text(
                  snapshot.data ?? 'No data',
                  style: const TextStyle(fontSize: 18),
                ),
                const SizedBox(height: 8),
                const Text(
                  'Pull refresh to update',
                  style: TextStyle(color: Colors.grey, fontSize: 14),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}

/// 2. FutureBuilder with stream-like updates
class PeriodicUpdateExample extends StatefulWidget {
  const PeriodicUpdateExample({super.key});

  @override
  State<PeriodicUpdateExample> createState() => _PeriodicUpdateExampleState();
}

class _PeriodicUpdateExampleState extends State<PeriodicUpdateExample> {
  Future<String> _fetchData() async {
    // Simulate a periodic update
    await Future.delayed(const Duration(seconds: 1));
    return 'Update at: ${DateTime.now().toLocal()}';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Periodic Updates'),
      ),
      body: FutureBuilder<String>(
        future: _fetchData(),
        builder: (context, snapshot) {
          // Check for loading state
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          // Check for error
          if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.error, color: Colors.red, size: 48),
                  const SizedBox(height: 16),
                  Text('Error: ${snapshot.error}'),
                ],
              ),
            );
          }

          // Display data with auto-update
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Text(
                  'Latest Update:',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 8),
                Text(
                  snapshot.data ?? 'No data',
                  style: const TextStyle(fontSize: 18),
                ),
                const SizedBox(height: 16),
                // Auto-update with timer
                _buildAutoUpdateTimer(),
              ],
            ),
          );
        },
      ),
    );
  }

  Widget _buildAutoUpdateTimer() {
    // Timer to refresh data every 5 seconds
    return StatefulBuilder(
      builder: (context, setState) {
        Timer.periodic(const Duration(seconds: 5), (timer) {
          setState(() {
            // This will trigger the FutureBuilder to rebuild
            // causing _fetchData() to run again
          });
        });
        return const Text(
          '⏰ Auto-updates every 5 seconds',
          style: TextStyle(color: Colors.grey, fontSize: 14),
        );
      },
    );
  }
}
```

What's happening here?
- API call with caching
- Periodic updates with FutureBuilder
- Refresh and retry functionality
- Auto-updating data

---

# Best Practices

## Use FutureBuilder for Simple Futures

```dart
// Good - Simple Future
FutureBuilder<String>(
  future: _fetchData(),
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return Text(snapshot.data!);
    }
    return CircularProgressIndicator();
  },
)
```

## Handle All States

```dart
// Good - All states handled
switch (snapshot.connectionState) {
  case ConnectionState.none:
    return Text('No connection');
  case ConnectionState.waiting:
    return CircularProgressIndicator();
  case ConnectionState.active:
    return Text('Loading...');
  case ConnectionState.done:
    if (snapshot.hasError) {
      return Text('Error');
    }
    return Text(snapshot.data ?? 'No data');
}
```

## Cache Future Results

```dart
// Good - Cache the Future
Future<String>? _cachedFuture;

@override
void initState() {
  super.initState();
  _cachedFuture = _fetchData();
}
```

---

# Common Mistakes

## Creating Future in build

Wrong:
```dart
// Future created every rebuild
FutureBuilder(
  future: _fetchData(), // Called on every rebuild
  builder: (context, snapshot) => ...
)
```

Correct:
```dart
// Future created once
final Future<String> _future = _fetchData();

FutureBuilder(
  future: _future,
  builder: (context, snapshot) => ...
)
```

## Not Handling All States

Wrong:
```dart
// Only handles data state
FutureBuilder(
  future: _fetchData(),
  builder: (context, snapshot) {
    return Text(snapshot.data ?? '');
  },
)
```

Correct:
```dart
// Handles all states
FutureBuilder(
  future: _fetchData(),
  builder: (context, snapshot) {
    if (snapshot.hasError) {
      return Text('Error');
    }
    if (!snapshot.hasData) {
      return CircularProgressIndicator();
    }
    return Text(snapshot.data!);
  },
)
```

---

# Summary

FutureBuilder simplifies handling asynchronous operations in Flutter. Use it to manage loading, error, and data states. Cache Futures when possible, handle all ConnectionStates, and show appropriate UI for each state.

---

# Next Steps

- [StreamBuilder](streambuilder.md)
- [ValueListenableBuilder](valuelistenablebuilder.md)
- [Async Patterns](async-patterns.md)

---

# Did You Know?

- FutureBuilder handles async operations
- AsyncSnapshot provides state info
- ConnectionState tracks loading status
- hasError indicates errors
- data contains the result
- Initial data can be provided
- Cache Futures for performance
- FutureBuilder is declarative