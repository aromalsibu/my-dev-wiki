# Async Patterns

Understand common patterns for handling asynchronous operations in Flutter.

---

# What is it?

Async Patterns are reusable strategies for handling asynchronous operations in Flutter applications. They cover everything from simple Future handling to complex streams, error handling, cancellation, retry logic, and combining multiple async operations. These patterns help you write clean, maintainable, and robust async code.

---

# Why does it exist?

Async Patterns exist to:

- Handle async operations consistently
- Manage loading, error, and data states
- Combine multiple async operations
- Implement retry and cancellation
- Avoid common async pitfalls
- Improve code readability
- Handle complex async flows

---

# Basic Async Patterns

> **Common patterns** for handling Futures.

```dart
// Import required packages
import 'package:flutter/material.dart';

/// Basic async patterns
class BasicAsyncPatterns extends StatelessWidget {
  const BasicAsyncPatterns({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Async Patterns'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Simple Future with FutureBuilder
            const Text(
              '1. Simple Future',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const _SimpleFutureExample(),
            const SizedBox(height: 16),
            
            // 2. Future with error handling
            const Text(
              '2. Future with Error Handling',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const _ErrorHandlingExample(),
            const SizedBox(height: 16),
            
            // 3. Future with retry
            const Text(
              '3. Future with Retry',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const _RetryExample(),
          ],
        ),
      ),
    );
  }
}

/// 1. Simple Future example
class _SimpleFutureExample extends StatelessWidget {
  const _SimpleFutureExample();

  Future<String> _fetchData() async {
    await Future.delayed(const Duration(seconds: 1));
    return 'Data loaded successfully!';
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<String>(
      future: _fetchData(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const SizedBox(
            height: 50,
            child: Center(child: CircularProgressIndicator()),
          );
        }
        
        if (snapshot.hasError) {
          return Container(
            padding: const EdgeInsets.all(8),
            color: Colors.red[100],
            child: Text('Error: ${snapshot.error}'),
          );
        }
        
        return Container(
          padding: const EdgeInsets.all(8),
          color: Colors.green[100],
          child: Text(snapshot.data ?? 'No data'),
        );
      },
    );
  }
}

/// 2. Error handling example
class _ErrorHandlingExample extends StatelessWidget {
  const _ErrorHandlingExample();

  Future<String> _fetchData() async {
    await Future.delayed(const Duration(seconds: 1));
    // Simulate an error
    throw Exception('Failed to load data');
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<String>(
      future: _fetchData(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const SizedBox(
            height: 50,
            child: Center(child: CircularProgressIndicator()),
          );
        }
        
        if (snapshot.hasError) {
          return Container(
            padding: const EdgeInsets.all(8),
            color: Colors.red[100],
            child: Row(
              children: [
                const Icon(Icons.error, color: Colors.red),
                const SizedBox(width: 8),
                Expanded(
                  child: Text('Error: ${snapshot.error}'),
                ),
              ],
            ),
          );
        }
        
        return Container(
          padding: const EdgeInsets.all(8),
          color: Colors.green[100],
          child: Text(snapshot.data ?? 'No data'),
        );
      },
    );
  }
}

/// 3. Retry example
class _RetryExample extends StatefulWidget {
  const _RetryExample();

  @override
  State<_RetryExample> createState() => _RetryExampleState();
}

class _RetryExampleState extends State<_RetryExample> {
  int _attempts = 0;
  Future<String>? _future;

  Future<String> _fetchData() async {
    await Future.delayed(const Duration(seconds: 1));
    _attempts++;
    
    // Fail on first attempt, succeed on second
    if (_attempts == 1) {
      throw Exception('Temporary failure');
    }
    
    return 'Data loaded after $_attempts attempts';
  }

  void _retry() {
    setState(() {
      _future = _fetchData();
    });
  }

  @override
  void initState() {
    super.initState();
    _future = _fetchData();
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<String>(
      future: _future,
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const SizedBox(
            height: 50,
            child: Center(child: CircularProgressIndicator()),
          );
        }
        
        if (snapshot.hasError) {
          return Container(
            padding: const EdgeInsets.all(8),
            color: Colors.red[100],
            child: Row(
              children: [
                const Icon(Icons.error, color: Colors.red),
                const SizedBox(width: 8),
                Expanded(
                  child: Text('Error: ${snapshot.error}'),
                ),
                TextButton(
                  onPressed: _retry,
                  child: const Text('Retry'),
                ),
              ],
            ),
          );
        }
        
        return Container(
          padding: const EdgeInsets.all(8),
          color: Colors.green[100],
          child: Text(snapshot.data ?? 'No data'),
        );
      },
    );
  }
}
```

What's happening here?
- Simple Future with loading state
- Error handling with retry
- Attempt tracking
- Retry logic

---

# Combining Async Operations

> **Combining multiple** async operations.

```dart
/// Combining async operations
class CombiningAsyncPatterns extends StatefulWidget {
  const CombiningAsyncPatterns({super.key});

  @override
  State<CombiningAsyncPatterns> createState() => _CombiningAsyncPatternsState();
}

class _CombiningAsyncPatternsState extends State<CombiningAsyncPatterns> {
  String _result = 'Ready';
  bool _isLoading = false;

  // 1. Sequential operations
  Future<void> _sequentialOperations() async {
    setState(() {
      _isLoading = true;
      _result = 'Starting sequential operations...';
    });

    try {
      final result1 = await _operation1();
      setState(() => _result = 'Operation 1: $result1');

      final result2 = await _operation2(result1);
      setState(() => _result = 'Operation 2: $result2');

      final result3 = await _operation3(result2);
      setState(() => _result = 'Operation 3: $result3');

      setState(() => _result = 'All operations complete!');
    } catch (e) {
      setState(() => _result = 'Error: $e');
    } finally {
      setState(() => _isLoading = false);
    }
  }

  // 2. Parallel operations
  Future<void> _parallelOperations() async {
    setState(() {
      _isLoading = true;
      _result = 'Starting parallel operations...';
    });

    try {
      // Run operations in parallel
      final results = await Future.wait([
        _operation1(),
        _operation2(''),
        _operation3(''),
      ]);

      setState(() {
        _result = 'Parallel results: ${results.join(", ")}';
      });
    } catch (e) {
      setState(() => _result = 'Error: $e');
    } finally {
      setState(() => _isLoading = false);
    }
  }

  // 3. First completed (race)
  Future<void> _raceOperations() async {
    setState(() {
      _isLoading = true;
      _result = 'Racing operations...';
    });

    try {
      final result = await Future.any([
        _operation1(),
        _operation2(''),
        _operation3(''),
      ]);

      setState(() {
        _result = 'First completed: $result';
      });
    } catch (e) {
      setState(() => _result = 'Error: $e');
    } finally {
      setState(() => _isLoading = false);
    }
  }

  Future<String> _operation1() async {
    await Future.delayed(const Duration(seconds: 1));
    return 'Data from op1';
  }

  Future<String> _operation2(String input) async {
    await Future.delayed(const Duration(seconds: 2));
    return 'Data from op2 (input: $input)';
  }

  Future<String> _operation3(String input) async {
    await Future.delayed(const Duration(seconds: 1));
    return 'Data from op3 (input: $input)';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Combining Async'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Result display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  const Text(
                    'Result:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  if (_isLoading)
                    const CircularProgressIndicator()
                  else
                    Text(_result),
                ],
              ),
            ),
            const SizedBox(height: 16),
            
            // Operation buttons
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isLoading ? null : _sequentialOperations,
                    child: const Text('Sequential'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isLoading ? null : _parallelOperations,
                    child: const Text('Parallel'),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 8),
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isLoading ? null : _raceOperations,
                    child: const Text('Race'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isLoading
                        ? null
                        : () {
                            setState(() {
                              _result = 'Ready';
                            });
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
- Sequential operations run one after another
- Parallel operations run simultaneously
- Race completes when first operation finishes
- Different patterns for different scenarios

---

# Async Error Handling

> **Advanced error** handling patterns.

```dart
/// Async error handling patterns
class AsyncErrorHandling extends StatefulWidget {
  const AsyncErrorHandling({super.key});

  @override
  State<AsyncErrorHandling> createState() => _AsyncErrorHandlingState();
}

class _AsyncErrorHandlingState extends State<AsyncErrorHandling> {
  String _result = 'Ready';
  bool _isLoading = false;

  // 1. Try-catch with specific errors
  Future<void> _specificErrorHandling() async {
    setState(() {
      _isLoading = true;
      _result = 'Loading...';
    });

    try {
      await _throwSpecificError();
    } on FormatException catch (e) {
      setState(() => _result = 'Format error: $e');
    } on TimeoutException catch (e) {
      setState(() => _result = 'Timeout error: $e');
    } catch (e) {
      setState(() => _result = 'Unknown error: $e');
    } finally {
      setState(() => _isLoading = false);
    }
  }

  // 2. Error with fallback
  Future<void> _errorWithFallback() async {
    setState(() {
      _isLoading = true;
      _result = 'Loading...';
    });

    try {
      final result = await _throwError();
      setState(() => _result = result);
    } catch (e) {
      // Use fallback data
      setState(() => _result = 'Fallback data (error: $e)');
    } finally {
      setState(() => _isLoading = false);
    }
  }

  // 3. Multiple error handling
  Future<void> _multipleErrorHandling() async {
    setState(() {
      _isLoading = true;
      _result = 'Loading...';
    });

    try {
      await _throwMultipleErrors();
    } on Exception catch (e) {
      setState(() => _result = 'Handled exception: $e');
    } catch (e) {
      setState(() => _result = 'Handled error: $e');
    } finally {
      setState(() => _isLoading = false);
    }
  }

  Future<void> _throwSpecificError() async {
    await Future.delayed(const Duration(seconds: 1));
    throw FormatException('Invalid format');
  }

  Future<String> _throwError() async {
    await Future.delayed(const Duration(seconds: 1));
    throw Exception('Network error');
  }

  Future<void> _throwMultipleErrors() async {
    await Future.delayed(const Duration(seconds: 1));
    throw 'String error';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Async Error Handling'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Result display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  const Text(
                    'Result:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  if (_isLoading)
                    const CircularProgressIndicator()
                  else
                    Text(_result),
                ],
              ),
            ),
            const SizedBox(height: 16),
            
            // Buttons
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isLoading ? null : _specificErrorHandling,
                    child: const Text('Specific Errors'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isLoading ? null : _errorWithFallback,
                    child: const Text('Fallback'),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 8),
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isLoading ? null : _multipleErrorHandling,
                    child: const Text('Multiple Errors'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isLoading
                        ? null
                        : () {
                            setState(() {
                              _result = 'Ready';
                            });
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
- Specific error handling with catch blocks
- Fallback data on error
- Multiple error types handling
- Proper cleanup with finally

---

# Async with Cancellation

> **Cancelling async** operations.

```dart
/// Async cancellation patterns
class AsyncCancellationPattern extends StatefulWidget {
  const AsyncCancellationPattern({super.key});

  @override
  State<AsyncCancellationPattern> createState() => _AsyncCancellationPatternState();
}

class _AsyncCancellationPatternState extends State<AsyncCancellationPattern> {
  // 1. Use CancelableOperation for cancellation
  CancelableOperation? _cancelableOperation;
  String _result = 'Ready';
  bool _isLoading = false;

  @override
  void dispose() {
    _cancelableOperation?.cancel();
    super.dispose();
  }

  // 2. Cancelable operation
  void _startCancelableOperation() async {
    // Cancel any existing operation
    _cancelableOperation?.cancel();

    setState(() {
      _isLoading = true;
      _result = 'Loading... (can cancel)';
    });

    // Create cancelable operation
    _cancelableOperation = CancelableOperation.fromFuture(
      _longRunningOperation(),
      onCancel: () {
        setState(() {
          _result = 'Cancelled!';
          _isLoading = false;
        });
      },
    );

    try {
      final result = await _cancelableOperation!.value;
      if (result != null) {
        setState(() {
          _result = result;
          _isLoading = false;
        });
      }
    } catch (e) {
      if (e is! CancelableOperationError) {
        setState(() {
          _result = 'Error: $e';
          _isLoading = false;
        });
      }
    }
  }

  // 3. Cancel operation
  void _cancelOperation() {
    _cancelableOperation?.cancel();
  }

  // 4. Simulate long-running operation
  Future<String> _longRunningOperation() async {
    int count = 0;
    while (count < 10) {
      await Future.delayed(const Duration(milliseconds: 500));
      count++;
      // Check if cancelled (will throw on cancellation)
      if (count == 10) {
        return 'Operation completed successfully!';
      }
    }
    return 'Operation completed!';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Async Cancellation'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Result display
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Column(
                children: [
                  const Text(
                    'Result:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  if (_isLoading)
                    Column(
                      children: [
                        const CircularProgressIndicator(),
                        const SizedBox(height: 8),
                        ElevatedButton(
                          onPressed: _cancelOperation,
                          child: const Text('Cancel'),
                        ),
                      ],
                    )
                  else
                    Text(_result),
                ],
              ),
            ),
            const SizedBox(height: 16),
            
            // Buttons
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isLoading ? null : _startCancelableOperation,
                    child: const Text('Start Operation'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isLoading ? null : () {
                      setState(() {
                        _result = 'Ready';
                      });
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
- CancelableOperation for cancellation
- onCancel callback for cleanup
- Check for cancellation
- Proper disposal

---

# Best Practices

## Use Appropriate Async Pattern

```dart
// Good - Choose the right pattern
// Sequential: await each operation
// Parallel: Future.wait
// Race: Future.any
// Cancellable: CancelableOperation
```

## Handle Errors Properly

```dart
// Good - Comprehensive error handling
try {
  final result = await asyncOperation();
} on FormatException catch (e) {
  // Handle format errors
} on TimeoutException catch (e) {
  // Handle timeout errors
} catch (e) {
  // Handle other errors
} finally {
  // Cleanup
}
```

## Clean Up Resources

```dart
// Good - Dispose and cancel
@override
void dispose() {
  _cancelableOperation?.cancel();
  _streamSubscription?.cancel();
  super.dispose();
}
```

---

# Common Mistakes

## Forgetting to Cancel

Wrong:
```dart
// No cancellation
@override
void dispose() {
  super.dispose();
}
```

Correct:
```dart
// Cancel operations
@override
void dispose() {
  _cancelableOperation?.cancel();
  _streamSubscription?.cancel();
  super.dispose();
}
```

## Not Handling Errors

Wrong:
```dart
// No error handling
final result = await asyncOperation();
```

Correct:
```dart
// With error handling
try {
  final result = await asyncOperation();
} catch (e) {
  // Handle error
}
```

---

# Summary

Async Patterns provide reusable strategies for handling asynchronous operations. Use sequential, parallel, and race patterns for combining operations. Handle errors with try-catch, implement cancellation with CancelableOperation, and clean up resources properly.

---

# Next Steps

- [Provider](../state-management/provider.md)
- [Riverpod](../state-management/riverpod.md)
- [BLoC](../state-management/bloc.md)

---

# Did You Know?

- Future.wait runs operations in parallel
- Future.any completes on first success
- CancelableOperation enables cancellation
- Error handling should be comprehensive
- Sequential operations run one after another
- Proper cleanup prevents memory leaks
- Retry logic handles transient failures
- Async patterns improve code readability