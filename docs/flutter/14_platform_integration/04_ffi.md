# FFI (Foreign Function Interface)

Understand how to call native C/C++ code directly from Dart using FFI.

---

# What is it?

FFI (Foreign Function Interface) is a mechanism in Dart that allows you to call native C/C++ code directly from Dart without going through platform channels. This provides higher performance and lower latency compared to MethodChannel, making it ideal for computationally intensive tasks, game engines, and performance-critical operations.

---

# Why does it exist?

FFI exists to:

- Call native C/C++ code directly
- Achieve higher performance
- Reduce latency
- Access existing C libraries
- Implement performance-critical operations
- Integrate with game engines
- Use native data processing

---

# FFI Setup

> **Setting up** FFI in your Flutter project.

```dart
// Import required packages
import 'dart:ffi' as ffi;
import 'dart:typed_data';
import 'package:ffi/ffi.dart';

/// 1. Basic FFI setup
class FFISetup {
  // 1. Load the native library
  static final ffi.DynamicLibrary nativeLibrary = _loadLibrary();

  static ffi.DynamicLibrary _loadLibrary() {
    // Platform-specific library loading
    if (Platform.isAndroid) {
      return ffi.DynamicLibrary.open('libnative_lib.so');
    } else if (Platform.isIOS) {
      return ffi.DynamicLibrary.process();
    } else if (Platform.isWindows) {
      return ffi.DynamicLibrary.open('native_lib.dll');
    } else if (Platform.isMacOS) {
      return ffi.DynamicLibrary.open('libnative_lib.dylib');
    } else if (Platform.isLinux) {
      return ffi.DynamicLibrary.open('libnative_lib.so');
    } else {
      throw Exception('Unsupported platform');
    }
  }
}

/// 2. Define native function signatures
class NativeFunctions {
  // 2.1 Function that returns an int
  static int add(int a, int b) {
    // Look up the native function and call it
    final addFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Int32 Function(ffi.Int32, ffi.Int32)>>('add');
    final add = addFunc.asFunction<int Function(int, int)>();
    return add(a, b);
  }

  // 2.2 Function that returns a double
  static double multiply(double a, double b) {
    final multiplyFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Double Function(ffi.Double, ffi.Double)>>('multiply');
    final multiply = multiplyFunc.asFunction<double Function(double, double)>();
    return multiply(a, b);
  }

  // 2.3 Function that modifies a string
  static String reverseString(String input) {
    // Convert Dart string to C string
    final cString = input.toNativeUtf8();
    
    final reverseFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Pointer<ffi.Char> Function(ffi.Pointer<ffi.Char>)>>('reverseString');
    final reverse = reverseFunc.asFunction<ffi.Pointer<ffi.Char> Function(ffi.Pointer<ffi.Char>)>();
    
    final result = reverse(cString);
    final resultString = result.cast<ffi.Uint8>().toDartString();
    
    // Free memory
    calloc.free(cString);
    
    return resultString;
  }

  // 2.4 Function that processes an array
  static int sumArray(List<int> numbers) {
    // Create a C array
    final cArray = calloc<ffi.Int32>(numbers.length);
    for (int i = 0; i < numbers.length; i++) {
      cArray[i] = numbers[i];
    }

    final sumFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Int32 Function(ffi.Pointer<ffi.Int32>, ffi.Int32)>>('sumArray');
    final sum = sumFunc.asFunction<int Function(ffi.Pointer<ffi.Int32>, int)>();
    
    final result = sum(cArray, numbers.length);
    
    // Free memory
    calloc.free(cArray);
    
    return result;
  }
}
```

What's happening here?
- DynamicLibrary loads native code
- lookup finds native functions
- asFunction converts to Dart function
- Pointers manage memory
- Native strings require conversion

---

# Native C/C++ Implementation

> **Implementing** native functions.

```c
// native_lib.c

#include <string.h>
#include <stdlib.h>

// 1. Simple addition
int add(int a, int b) {
    return a + b;
}

// 2. Multiplication
double multiply(double a, double b) {
    return a * b;
}

// 3. String reversal
char* reverseString(const char* input) {
    int length = strlen(input);
    char* result = (char*)malloc((length + 1) * sizeof(char));
    
    for (int i = 0; i < length; i++) {
        result[i] = input[length - 1 - i];
    }
    result[length] = '\0';
    
    return result;
}

// 4. Array sum
int sumArray(int* arr, int length) {
    int sum = 0;
    for (int i = 0; i < length; i++) {
        sum += arr[i];
    }
    return sum;
}

// 5. Image processing (blur effect)
void applyBlur(unsigned char* imageData, int width, int height) {
    // Simple box blur algorithm
    unsigned char* temp = (unsigned char*)malloc(width * height * sizeof(unsigned char));
    
    for (int y = 0; y < height; y++) {
        for (int x = 0; x < width; x++) {
            int sum = 0;
            int count = 0;
            
            for (int dy = -1; dy <= 1; dy++) {
                for (int dx = -1; dx <= 1; dx++) {
                    int px = x + dx;
                    int py = y + dy;
                    
                    if (px >= 0 && px < width && py >= 0 && py < height) {
                        sum += imageData[py * width + px];
                        count++;
                    }
                }
            }
            
            temp[y * width + x] = sum / count;
        }
    }
    
    // Copy back
    memcpy(imageData, temp, width * height * sizeof(unsigned char));
    free(temp);
}
```

What's happening here?
- C functions implement logic
- Memory management with malloc/free
- Pointers for arrays and strings
- Data processing in C

---

# FFI with Structs

> **Using structs** with FFI.

```dart
/// 1. Define C struct in Dart
class Point extends ffi.Struct {
  @ffi.Double()
  double x;

  @ffi.Double()
  double y;
}

/// 2. Native functions with structs
class NativeStructFunctions {
  static double distance(Point p1, Point p2) {
    final distanceFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Double Function(ffi.Pointer<Point>, ffi.Pointer<Point>)>>('distance');
    final distance = distanceFunc.asFunction<double Function(ffi.Pointer<Point>, ffi.Pointer<Point>)>();

    // Create Point objects in native memory
    final p1Ptr = calloc<Point>();
    final p2Ptr = calloc<Point>();

    p1Ptr.ref.x = p1.x;
    p1Ptr.ref.y = p1.y;
    p2Ptr.ref.x = p2.x;
    p2Ptr.ref.y = p2.y;

    final result = distance(p1Ptr, p2Ptr);

    // Free memory
    calloc.free(p1Ptr);
    calloc.free(p2Ptr);

    return result;
  }

  static Point midpoint(Point p1, Point p2) {
    final midpointFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Pointer<Point> Function(ffi.Pointer<Point>, ffi.Pointer<Point>)>>('midpoint');
    final midpoint = midpointFunc.asFunction<ffi.Pointer<Point> Function(ffi.Pointer<Point>, ffi.Pointer<Point>)>();

    final p1Ptr = calloc<Point>();
    final p2Ptr = calloc<Point>();

    p1Ptr.ref.x = p1.x;
    p1Ptr.ref.y = p1.y;
    p2Ptr.ref.x = p2.x;
    p2Ptr.ref.y = p2.y;

    final resultPtr = midpoint(p1Ptr, p2Ptr);
    final result = Point()
      ..x = resultPtr.ref.x
      ..y = resultPtr.ref.y;

    calloc.free(p1Ptr);
    calloc.free(p2Ptr);

    return result;
  }
}

/// 3. Native struct definitions (C)
// typedef struct {
//   double x;
//   double y;
// } Point;
//
// double distance(Point* p1, Point* p2) {
//   double dx = p1->x - p2->x;
//   double dy = p1->y - p2->y;
//   return sqrt(dx * dx + dy * dy);
// }
//
// Point* midpoint(Point* p1, Point* p2) {
//   Point* result = (Point*)malloc(sizeof(Point));
//   result->x = (p1->x + p2->x) / 2;
//   result->y = (p1->y + p2->y) / 2;
//   return result;
// }
```

What's happening here?
- Structs map C structures to Dart
- Pointers manage native memory
- calloc allocates memory
- ref accesses struct fields

---

# Real-World Examples

> **Common patterns** with FFI.

```dart
/// 1. Image processing with FFI
class ImageProcessor {
  static Uint8List applyBlur(Uint8List imageData, int width, int height) {
    // 1. Create native buffer
    final buffer = calloc<ffi.Uint8>(imageData.length);
    
    // 2. Copy data to native buffer
    final nativeArray = buffer.asTypedList(imageData.length);
    nativeArray.setAll(0, imageData);
    
    // 3. Call native function
    final blurFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Void Function(ffi.Pointer<ffi.Uint8>, ffi.Int32, ffi.Int32)>>('applyBlur');
    final blur = blurFunc.asFunction<void Function(ffi.Pointer<ffi.Uint8>, int, int)>();
    
    blur(buffer, width, height);
    
    // 4. Copy result back
    final result = Uint8List.fromList(buffer.asTypedList(imageData.length));
    
    // 5. Free memory
    calloc.free(buffer);
    
    return result;
  }
}

/// 2. Native math operations
class NativeMath {
  static int factorial(int n) {
    final factorialFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Int64 Function(ffi.Int32)>>('factorial');
    final factorial = factorialFunc.asFunction<int Function(int)>();
    return factorial(n);
  }

  static double sin(double angle) {
    final sinFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Double Function(ffi.Double)>>('sin');
    final sin = sinFunc.asFunction<double Function(double)>();
    return sin(angle);
  }

  static double cos(double angle) {
    final cosFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Double Function(ffi.Double)>>('cos');
    final cos = cosFunc.asFunction<double Function(double)>();
    return cos(angle);
  }

  static double sqrt(double value) {
    final sqrtFunc = FFISetup.nativeLibrary
        .lookup<ffi.NativeFunction<ffi.Double Function(ffi.Double)>>('sqrt');
    final sqrt = sqrtFunc.asFunction<double Function(double)>();
    return sqrt(value);
  }
}
```

What's happening here?
- Image processing with FFI
- Native math operations
- Memory management
- Performance optimization

---

# Best Practices

## Manage Memory Properly

```dart
// Good - Free allocated memory
final buffer = calloc<ffi.Uint8>(size);
try {
  // Use buffer
} finally {
  calloc.free(buffer);
}
```

## Use TypedData for Performance

```dart
// Good - Use TypedData for arrays
final buffer = calloc<ffi.Int32>(length);
final array = buffer.asTypedList(length);
```

## Handle Errors

```dart
// Good - Error handling
try {
  final result = NativeFunctions.add(5, 3);
} catch (e) {
  print('Native error: $e');
}
```

---

# Common Mistakes

## Memory Leaks

Wrong:
```dart
// Memory leak
final buffer = calloc<ffi.Uint8>(size);
// Use buffer
// Not freeing memory
```

Correct:
```dart
// Free memory
final buffer = calloc<ffi.Uint8>(size);
try {
  // Use buffer
} finally {
  calloc.free(buffer);
}
```

## Incorrect Type Mapping

Wrong:
```dart
// Wrong type mapping
final func = library.lookup<ffi.NativeFunction<ffi.Int32 Function(ffi.Float)>>('function');
```

Correct:
```dart
// Correct type mapping
final func = library.lookup<ffi.NativeFunction<ffi.Int32 Function(ffi.Double)>>('function');
```

---

# Summary

FFI enables direct calling of native C/C++ code from Dart. Use DynamicLibrary to load native libraries, lookup to find functions, and asFunction to convert to Dart functions. Manage memory properly and handle errors gracefully for reliable FFI integration.

---

# Next Steps

- [Plugins](plugins.md)
- [Platform Views](platform-views.md)
- [MethodChannel](methodchannel.md)

---

# Did You Know?

- FFI calls native code directly
- DynamicLibrary loads native libraries
- Structs map C structures
- Pointers manage memory
- calloc allocates memory
- asFunction converts functions
- FFI is faster than MethodChannel
- FFI supports C, C++, and Objective-C