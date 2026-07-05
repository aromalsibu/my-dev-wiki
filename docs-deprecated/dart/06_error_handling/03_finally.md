# finally

Understand how to use the finally block for cleanup operations in Dart.

---

# What is it?

The `finally` block is a section of code that always executes after a `try-catch` block, regardless of whether an exception was thrown or caught. It's used for cleanup operations like closing files, releasing resources, or resetting states.

---

# Why does it exist?

`finally` exists to:

- Ensure cleanup code always runs
- Release resources (files, connections, etc.)
- Prevent resource leaks
- Reset state after operations
- Provide guaranteed execution
- Handle cleanup regardless of success/failure

---

# Basic finally

## Simple finally

> **The `finally` block** always executes, whether an exception occurs or not.

```dart
void main() {
  print('Starting...');
  
  try {
    print('Inside try');
    // No exception here
    int result = 10 ~/ 2;
    print('Result: $result');
  } catch (e) {
    print('Error caught: $e');
  } finally {
    print('Finally always executes');
  }
  
  print('After try-catch-finally');
}

// Output:
// Starting...
// Inside try
// Result: 5
// Finally always executes
// After try-catch-finally
```

> **What's happening here?**
> - `try` block executes successfully
> - No exception is thrown
> - `finally` executes regardless
> - Program continues after the block

---

## finally with Exception

> **When an exception occurs**, `finally` still executes after the `catch` block.

```dart
void main() {
  print('Starting...');
  
  try {
    print('Inside try');
    int result = 10 ~/ 0; // Throws exception
    print('This never executes');
  } catch (e) {
    print('Error caught: $e');
  } finally {
    print('Finally always executes even with error');
  }
  
  print('After try-catch-finally');
}

// Output:
// Starting...
// Inside try
// Error caught: IntegerDivisionByZeroException
// Finally always executes even with error
// After try-catch-finally
```

> **What's happening here?**
> - Exception is thrown in `try`
> - Execution jumps to `catch`
> - `catch` handles the exception
> - `finally` executes after `catch`
> - Program continues normally

---

# finally with Rethrow

## finally with rethrow

> **When using `rethrow`**, the `finally` block still executes before the exception propagates.

```dart
void riskyOperation() {
  try {
    print('Inside risky operation');
    throw Exception('Something went wrong');
  } catch (e) {
    print('Caught in risky: $e');
    // Add context
    rethrow; // Propagate the exception
  } finally {
    print('Cleanup in risky operation');
  }
}

void main() {
  try {
    riskyOperation();
  } catch (e) {
    print('Caught in main: $e');
  }
}

// Output:
// Inside risky operation
// Caught in risky: Exception: Something went wrong
// Cleanup in risky operation
// Caught in main: Exception: Something went wrong
```

> **Key insights:**
> - `finally` executes before `rethrow`
> - Cleanup happens even when propagating
> - The exception still reaches the caller
> - This ensures proper cleanup

---

# Real-World Examples

## File Operations

> **`finally` is essential** for closing files and releasing resources.

```dart
import 'dart:io';

class FileProcessor {
  void processFile(String path) {
    File file = File(path);
    RandomAccessFile? raf;
    
    try {
      print('Opening file: $path');
      raf = file.openSync();
      
      // Read and process file
      var content = raf.readStringSync();
      print('File content: $content');
      
      // Could throw an exception
      if (content.isEmpty) {
        throw Exception('File is empty');
      }
      
      print('File processed successfully');
    } catch (e) {
      print('Error processing file: $e');
    } finally {
      // Always close the file
      if (raf != null) {
        print('Closing file');
        raf.closeSync();
      }
    }
  }
}

void main() {
  var processor = FileProcessor();
  
  // Try with existing file
  try {
    processor.processFile('data.txt');
  } catch (e) {
    print('Main caught: $e');
  }
}
```

---

## Database Connection

> **`finally` ensures** database connections are always closed.

```dart
class DatabaseConnection {
  bool _isConnected = false;
  
  void connect() {
    print('Connecting to database...');
    _isConnected = true;
  }
  
  void query(String sql) {
    if (!_isConnected) {
      throw Exception('Not connected to database');
    }
    print('Executing: $sql');
    if (sql.contains('error')) {
      throw Exception('Query execution failed');
    }
  }
  
  void disconnect() {
    print('Disconnecting from database...');
    _isConnected = false;
  }
}

void performDatabaseOperations() {
  var db = DatabaseConnection();
  
  try {
    db.connect();
    db.query('SELECT * FROM users');
    db.query('UPDATE users SET active = true');
    // This will cause an error
    db.query('DELETE FROM error');
  } catch (e) {
    print('Database error: $e');
  } finally {
    // Always disconnect
    db.disconnect();
  }
}

void main() {
  print('Starting database operations...');
  performDatabaseOperations();
  print('Operations completed');
}
```

---

# finally with return

## finally and Return Statements

> **`finally` executes** even when there's a `return` statement in `try` or `catch`.

```dart
String processData(String? input) {
  try {
    if (input == null) {
      throw ArgumentError('Input cannot be null');
    }
    return 'Processed: $input';
  } catch (e) {
    print('Error: $e');
    return 'Default value';
  } finally {
    print('Cleanup always runs');
  }
}

void main() {
  var result = processData('Hello');
  print('Result: $result');
  
  print('---');
  
  var result2 = processData(null);
  print('Result: $result2');
}

// Output:
// Cleanup always runs
// Result: Processed: Hello
// ---
// Error: ArgumentError: Input cannot be null
// Cleanup always runs
// Result: Default value
```

> **What's happening here?**
> - `finally` executes before `return`
> - This ensures cleanup happens
> - Even with `return`, `finally` runs
> - Useful for resource cleanup

---

# try-finally without catch

## finally Alone

> **`try-finally` can be used** without a `catch` block when you don't want to handle the exception locally.

```dart
void riskyOperation() {
  try {
    print('Performing risky operation');
    throw Exception('Error occurred');
  } finally {
    print('Cleaning up resources');
  }
}

void main() {
  try {
    riskyOperation();
  } catch (e) {
    print('Caught in main: $e');
  }
}

// Output:
// Performing risky operation
// Cleaning up resources
// Caught in main: Exception: Error occurred
```

> **Key insights:**
> - `finally` runs before exception propagates
> - The exception goes to the caller
> - Useful when you want cleanup but not handling
> - Combines with caller's error handling

---

# Best Practices

## Always Clean Up Resources

```dart
// Good: Resource cleanup in finally
void processFile() {
  var file = File('data.txt');
  var raf = file.openSync();
  try {
    // Process file
  } finally {
    raf.closeSync(); // Always closes
  }
}

// Bad: Resource leak
void badProcessFile() {
  var file = File('data.txt');
  var raf = file.openSync();
  // If exception occurs, file is never closed
}
```

---

## Use finally with try-catch

```dart
// Good: Full error handling
void operation() {
  try {
    riskyOp();
  } catch (e) {
    handleError(e);
  } finally {
    cleanup();
  }
}

// Bad: Missing finally
void badOperation() {
  try {
    riskyOp();
  } catch (e) {
    handleError(e);
  }
  cleanup(); // May not run if exception isn't caught
}
```

---

# Common Mistakes

## Modifying Return in finally

Wrong:
```dart
String getData() {
  try {
    return 'Success';
  } finally {
    return 'Finally'; // Overrides return value!
  }
}
```

Correct:
```dart
String getData() {
  try {
    return 'Success';
  } finally {
    print('Cleanup'); // Don't modify return
  }
}
```

---

## Resource Leak

Wrong:
```dart
void process() {
  var connection = getConnection();
  try {
    connection.send('data');
  } catch (e) {
    print('Error');
  }
  // connection never closed if exception occurs
}
```

Correct:
```dart
void process() {
  var connection = getConnection();
  try {
    connection.send('data');
  } catch (e) {
    print('Error');
  } finally {
    connection.close(); // Always closes
  }
}
```

---

# Summary

`finally` guarantees that cleanup code runs regardless of exceptions. Use it for resource cleanup, state reset, and guaranteed execution.

---

# Next Steps

Now that you understand finally, continue to:

- [throw](throw.md)
- [Custom Exceptions](custom-exceptions.md)
- [Assertions](assertions.md)
- [Libraries](libraries.md)

---

# Did You Know?

- `finally` always executes, even with `return`
- `finally` runs before `rethrow`
- `finally` works without `catch`
- Use `finally` for resource cleanup
- `finally` cannot modify return values
- `finally` executes after `try` or `catch`
- Always close resources in `finally`

---
