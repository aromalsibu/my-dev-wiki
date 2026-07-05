# Data Types

## 1. Overview

### Definition
A data type in C defines the set of values a variable can hold, the operations that can be performed on it, and the way it is stored in memory. Every variable, function parameter, and return value in C must have a type, which is checked at compile time. Types are the foundation of C's static type system.

### Purpose
Data types serve several critical purposes:
- **Memory allocation** – determine how many bytes are reserved.
- **Interpretation** – specify how the bit pattern is interpreted (integer, floating‑point, etc.).
- **Operation validity** – restrict which operations are allowed (e.g., you cannot add a pointer to a struct).
- **Type safety** – the compiler checks that operations are meaningful, preventing many bugs.
- **Performance** – choosing the right type can improve speed and memory usage.

### Where It Fits
Data types are the core of C's type system. They appear in:
- **Variable declarations** – `int x;`
- **Function signatures** – `float add(float a, float b);`
- **Structures and unions** – defining aggregates.
- **Pointer types** – `int *p;`
- **Type definitions** – `typedef`.

---

## 2. Why It Exists

### The Problem Without Types
In very low‑level languages (e.g., assembly), everything is just bytes or words. There is no distinction between an integer, a floating‑point number, a pointer, or a character. This leads to:
- **Bugs** – accidentally treating a pointer as an integer.
- **No safety** – no compile‑time checks.
- **Hard‑to‑read code** – you have to remember what each memory location represents.
- **Non‑portable** – sizes and representations vary across machines.

### The Solution: A Type System
C provides a set of built‑in types that map to hardware representations but abstract away the details. The compiler enforces type rules, preventing many errors. Additionally, you can create your own types (`struct`, `typedef`), allowing high‑level abstractions while remaining efficient.

### Why It Was Introduced
C inherited types from ALGOL and BCPL but added more precision. Dennis Ritchie introduced types to make system programming safer and more portable while still allowing low‑level manipulation via pointers and casts. The type system is deliberately simple and close to the hardware, reflecting C's philosophy of trust the programmer.

---

## 3. Syntax / Basic Usage

### Fundamental (Basic) Data Types
```c
#include <stdio.h>
#include <stdbool.h>   // for bool, true, false (C99)
#include <stdint.h>    // for exact‑width types

int main(void) {
    // Integer types
    char c = 'A';                    // 1 byte, typically signed or unsigned (implementation‑defined)
    signed char sc = -10;            // explicitly signed
    unsigned char uc = 255;          // 0 to 255

    short s = -100;                  // at least 2 bytes
    unsigned short us = 65535;       // 0 to 65535

    int i = -1000;                   // at least 2 bytes, typically 4
    unsigned int ui = 4000000000U;   // 0 to 4,294,967,295 (if 32-bit)
    long l = -100000L;               // at least 4 bytes
    unsigned long ul = 100000UL;

    long long ll = -100000000000LL;  // at least 8 bytes
    unsigned long long ull = 100000000000ULL;

    // Floating‑point types
    float f = 3.14f;                 // 4 bytes, single precision
    double d = 3.14159;              // 8 bytes, double precision
    long double ld = 3.141592653L;   // at least 10 bytes, often 12 or 16

    // Boolean (C99)
    bool b = true;                   // from <stdbool.h>, effectively int

    // Character
    char ch = 'A';                   // actually integer type

    // void – used for functions returning nothing or generic pointers
    void *ptr = NULL;                // pointer to anything

    // Exact‑width integer types (C99)
    int8_t i8 = 10;                  // exactly 8 bits signed
    uint16_t u16 = 65535;            // exactly 16 bits unsigned
    int32_t i32 = -1000000;          // exactly 32 bits
    uint64_t u64 = 100000000000ULL;  // exactly 64 bits

    printf("char: %d bytes\n", sizeof(char));
    printf("short: %zu bytes\n", sizeof(short));
    printf("int: %zu bytes\n", sizeof(int));
    printf("long: %zu bytes\n", sizeof(long));
    printf("long long: %zu bytes\n", sizeof(long long));
    printf("float: %zu bytes\n", sizeof(float));
    printf("double: %zu bytes\n", sizeof(double));
    printf("long double: %zu bytes\n", sizeof(long double));
    printf("pointer: %zu bytes\n", sizeof(void *));

    return 0;
}
```

### Derived and User‑Defined Types
```c
#include <stdio.h>

// Array type
int arr[10];                     // array of 10 ints

// Pointer type
int *p;                          // pointer to int

// Structure type
struct Point {
    int x;
    int y;
};

// Union type
union Data {
    int i;
    float f;
    char str[20];
};

// Enumeration type
enum Color { RED, GREEN, BLUE };

// Typedef – alias for an existing type
typedef unsigned long ulong;
typedef struct Point Point;      // now you can use Point without "struct"

int main(void) {
    ulong u = 100UL;
    Point pt = {10, 20};
    enum Color c = RED;
    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>
#include <stdbool.h>   // provides bool, true, false
#include <stdint.h>    // provides int8_t, uint16_t, etc.

int main(void) {
    // char – smallest addressable unit; often used for characters and small integers.
    char c = 'A';       // 'A' is an integer constant (65 in ASCII).
    // signed char – explicitly signed; range typically -128 to 127.
    signed char sc = -10;
    // unsigned char – range 0 to 255; useful for raw bytes.
    unsigned char uc = 255;

    // short – at least 16 bits; on many platforms exactly 16 bits.
    short s = -100;
    unsigned short us = 65535;   // max for 16-bit unsigned.

    // int – the "natural" integer size; at least 16 bits, typically 32 bits.
    int i = -1000;
    unsigned int ui = 4000000000U; // suffix U for unsigned.

    // long – at least 32 bits; on Windows 32, on Linux 64 (LP64 data model).
    long l = -100000L;
    unsigned long ul = 100000UL;

    // long long – at least 64 bits; introduced in C99.
    long long ll = -100000000000LL; // LL suffix for long long.
    unsigned long long ull = 100000000000ULL;

    // Floating‑point: float (single), double (double), long double (extended).
    float f = 3.14f;   // f suffix for float; without suffix, it's double.
    double d = 3.14159; // default floating type.
    long double ld = 3.141592653L; // L suffix for long double.

    // bool – from stdbool.h; actually an int with 0/1 values.
    bool b = true;     // true expands to 1.

    // void – incomplete type; used for functions returning nothing and generic pointers.
    void *ptr = NULL;  // generic pointer, can hold any address.

    // Fixed‑width types from stdint.h – guarantee exact bit widths.
    int8_t i8 = 10;      // exactly 8 bits signed.
    uint16_t u16 = 65535; // exactly 16 bits unsigned.
    int32_t i32 = -1000000;
    uint64_t u64 = 100000000000ULL; // exactly 64 bits.

    // sizeof operator returns size_t (unsigned integer) – use %zu to print.
    printf("char: %zu bytes\n", sizeof(char));        // always 1
    printf("short: %zu bytes\n", sizeof(short));
    printf("int: %zu bytes\n", sizeof(int));
    printf("long: %zu bytes\n", sizeof(long));
    printf("long long: %zu bytes\n", sizeof(long long));
    printf("float: %zu bytes\n", sizeof(float));
    printf("double: %zu bytes\n", sizeof(double));
    printf("long double: %zu bytes\n", sizeof(long double));
    printf("pointer: %zu bytes\n", sizeof(void *));    // 4 or 8 bytes

    return 0;
}
```

---

## 4. Mental Model – Types as Stencils

Think of data types as different-sized stencils or moulds:

- **char** – a tiny mould for one byte; fits small numbers or a single character.
- **int** – a medium mould; fits whole numbers up to a few billion.
- **float** – a mould for numbers with a decimal point, but with limited precision.
- **double** – a bigger mould for more precise decimal numbers.
- **pointers** – a mould that holds memory addresses; like a label that points to another mould.
- **structs** – a custom mould that combines several smaller moulds into a single block.
- **unions** – a mould that can hold different types at different times, but only one at a time.
- **arrays** – a row of identical moulds next to each other.

The compiler uses the mould to decide:
- How much space to allocate.
- How to interpret the bits.
- Which operations are allowed (you can't put a floating‑point number into an integer mould without a cast).

---

## 5. Core Concepts

### Basic Types and Their Properties

| Type | Minimum Size | Typical Size | Range (Typical) |
|------|--------------|--------------|-----------------|
| `char` | 1 byte | 1 byte | -128 to 127 or 0 to 255 |
| `signed char` | 1 byte | 1 byte | -128 to 127 |
| `unsigned char` | 1 byte | 1 byte | 0 to 255 |
| `short` | 2 bytes | 2 bytes | -32,768 to 32,767 |
| `unsigned short` | 2 bytes | 2 bytes | 0 to 65,535 |
| `int` | 2 bytes | 4 bytes | -2,147,483,648 to 2,147,483,647 |
| `unsigned int` | 2 bytes | 4 bytes | 0 to 4,294,967,295 |
| `long` | 4 bytes | 4 (Win) / 8 (Linux) | varies |
| `unsigned long` | 4 bytes | varies | varies |
| `long long` | 8 bytes | 8 bytes | -9.22e18 to 9.22e18 |
| `unsigned long long` | 8 bytes | 8 bytes | 0 to 1.84e19 |
| `float` | 4 bytes | 4 bytes | ±1.2e-38 to ±3.4e38 |
| `double` | 8 bytes | 8 bytes | ±2.3e-308 to ±1.8e308 |
| `long double` | ≥ 8 bytes | 10/12/16 | extended precision |

### Type Modifiers
- **`signed`** – explicitly allow negative values (default for char, int, etc. except on some platforms char is unsigned).
- **`unsigned`** – only non‑negative values; range doubles the positive side.
- **`short`** – reduce size (minimum 16 bits).
- **`long`** – increase size (minimum 32 bits).
- **`long long`** – larger (minimum 64 bits).

### Type Qualifiers
- **`const`** – read‑only variable.
- **`volatile`** – prevents optimisations; value may change unexpectedly.
- **`restrict`** (C99) – for pointers; indicates no aliasing.

### Derived Types
- **Array** – contiguous sequence of elements of the same type.
- **Pointer** – holds memory address.
- **Function** – defines signature.
- **Structure** – aggregate of possibly different types.
- **Union** – overlapping storage for different types.
- **Enum** – set of named integer constants.

### Standard Headers
- **`<stdint.h>`** – exact‑width integer types (int8_t, uint16_t, etc.).
- **`<inttypes.h>`** – macros for printing exact‑width types.
- **`<stdbool.h>`** – bool, true, false.
- **`<limits.h>`** – macros for type ranges (INT_MAX, etc.).
- **`<float.h>`** – macros for floating‑point properties.
- **`<stddef.h>`** – size_t, ptrdiff_t, NULL.

---

## 6. How It Works – Type Representation

### Integer Representation (Two's Complement)
- Signed integers use two's complement: the most significant bit is the sign bit.
- Unsigned integers use pure binary.
- The size affects the range.

### Floating‑Point Representation (IEEE 754)
- `float` – 32 bits: 1 sign, 8 exponent, 23 mantissa.
- `double` – 64 bits: 1 sign, 11 exponent, 52 mantissa.
- `long double` – varies; often 80‑bit extended precision (x86) or 128‑bit.

### Pointer Representation
- Pointers store memory addresses; size depends on the architecture (4 bytes on 32‑bit, 8 on 64‑bit).
- `void *` is a generic pointer that can hold any address but cannot be dereferenced without casting.

### Structure Alignment and Padding
- The compiler may insert padding between members to satisfy alignment requirements.
- Alignment ensures that members are accessed efficiently (e.g., 4‑byte ints aligned on 4‑byte boundaries).
- Ordering members by size can reduce padding.

---

## 7. Internal Architecture – Type Information

- **Compile‑time**: The compiler stores type information in its symbol table. It uses it for type checking and to generate correct code.
- **No runtime type information** – unlike Java or C++, C does not preserve type information at runtime (except for some debug info). This makes C efficient but less safe.
- **`sizeof`** is a compile‑time operator (except for variable‑length arrays, C99) that yields the size in bytes.
- **Type conversions** – implicit (promotion) and explicit (cast) conversions follow rules defined by the standard.

---

## 8. Lifecycle / Workflow of a Type

1. **Declaration** – the type appears in a variable declaration, function signature, etc.
2. **Type checking** – the compiler verifies that operations are valid for the type.
3. **Storage allocation** – based on type size, memory is reserved.
4. **Initialisation** – values are stored according to type interpretation.
5. **Operations** – arithmetic, comparisons, etc., are performed according to type semantics.
6. **Conversions** – when necessary, values are converted to other types (implicitly or explicitly).
7. **Program termination** – storage is released.

---

## 9. Practical Examples

### Example 1: Signed vs Unsigned and Overflow
```c
#include <stdio.h>
#include <limits.h>

int main(void) {
    // Signed overflow is undefined behaviour.
    int x = INT_MAX;
    printf("INT_MAX = %d\n", x);
    // x + 1 may wrap around, but it's undefined – don't do this.
    // Instead, check before addition.

    // Unsigned overflow wraps around modulo (2^n).
    unsigned int y = UINT_MAX;
    printf("UINT_MAX = %u\n", y);
    unsigned int z = y + 1;  // wraps to 0 (defined behaviour)
    printf("UINT_MAX + 1 = %u\n", z);

    // Signed vs unsigned comparison can be tricky.
    int a = -1;
    unsigned int b = 1;
    if (a < b) {
        printf("a < b\n");  // This may NOT print! Because a is converted to unsigned.
    } else {
        printf("a >= b\n"); // Actually prints: -1 becomes UINT_MAX, which > 1.
    }
    // To avoid: compare signed values or cast appropriately.
    if (a < (int)b) {
        printf("a < (int)b\n"); // works
    }

    return 0;
}
```
**Code Breakdown:**
- Signed integer overflow is **undefined** – never rely on it.
- Unsigned overflow is well‑defined: wraps modulo `2^n`.
- Mixed signed/unsigned comparisons can produce surprising results because of implicit conversions; always be explicit.

### Example 2: Size and Range of Types
```c
#include <stdio.h>
#include <limits.h>
#include <float.h>
#include <stdint.h>

int main(void) {
    printf("char: %zu bytes, min: %d, max: %d\n",
           sizeof(char), CHAR_MIN, CHAR_MAX);
    printf("unsigned char: %zu bytes, max: %u\n",
           sizeof(unsigned char), UCHAR_MAX);
    printf("short: %zu bytes, min: %hd, max: %hd\n",
           sizeof(short), SHRT_MIN, SHRT_MAX);
    printf("int: %zu bytes, min: %d, max: %d\n",
           sizeof(int), INT_MIN, INT_MAX);
    printf("long: %zu bytes, min: %ld, max: %ld\n",
           sizeof(long), LONG_MIN, LONG_MAX);
    printf("long long: %zu bytes, min: %lld, max: %lld\n",
           sizeof(long long), LLONG_MIN, LLONG_MAX);
    printf("float: %zu bytes, min: %e, max: %e\n",
           sizeof(float), FLT_MIN, FLT_MAX);
    printf("double: %zu bytes, min: %e, max: %e\n",
           sizeof(double), DBL_MIN, DBL_MAX);
    printf("long double: %zu bytes, min: %Le, max: %Le\n",
           sizeof(long double), LDBL_MIN, LDBL_MAX);
    return 0;
}
```
**Code Breakdown:**
- Use `<limits.h>` and `<float.h>` for portable limits.
- The `%zu` format specifier for `size_t`.
- Use `%ld`, `%lld`, `%Le` appropriately for different integer and floating types.

### Example 3: Structures and Padding
```c
#include <stdio.h>

struct Unpadded {
    char c;     // 1 byte
    int i;      // 4 bytes
    double d;   // 8 bytes
}; // likely size 16 or 24 (padding)

struct Padded {
    double d;   // 8 bytes
    int i;      // 4 bytes
    char c;     // 1 byte
}; // likely size 16 (8+4+1+padding to align)

int main(void) {
    printf("Size of Unpadded: %zu\n", sizeof(struct Unpadded));
    printf("Size of Padded: %zu\n", sizeof(struct Padded));

    // Offset of members
    struct Unpadded u;
    printf("Offset of c: %zu\n", (char *)&u.c - (char *)&u);
    printf("Offset of i: %zu\n", (char *)&u.i - (char *)&u);
    printf("Offset of d: %zu\n", (char *)&u.d - (char *)&u);
    return 0;
}
```
**Code Breakdown:**
- The compiler aligns members to their natural alignment (e.g., `int` on 4‑byte boundary).
- Reordering members can reduce padding and save memory.
- Use `offsetof` from `<stddef.h>` to get offsets.

### Example 4: Using `_Generic` for Type‑Generic Code (C11)
```c
#include <stdio.h>
#include <math.h>

// Macro that uses _Generic to select the appropriate absolute function.
#define ABS(x) _Generic((x), \
    int: abs, \
    float: fabsf, \
    double: fabs, \
    long double: fabsl, \
    default: fabs \
)(x)

int main(void) {
    int i = -5;
    float f = -3.14f;
    double d = -2.718;
    long double ld = -1.414L;

    printf("abs(%d) = %d\n", i, ABS(i));
    printf("abs(%f) = %f\n", f, ABS(f));
    printf("abs(%f) = %f\n", d, ABS(d));
    printf("abs(%Lf) = %Lf\n", ld, ABS(ld));
    return 0;
}
```
**Code Breakdown:**
- `_Generic` selects a compile‑time expression based on the type of the controlling expression.
- This is a powerful way to write type‑generic macros in C11.

### Example 5: Type Conversions (Implicit and Explicit)
```c
#include <stdio.h>

int main(void) {
    // Implicit conversion (promotion)
    int a = 10;
    double b = a;  // int → double, safe

    // Implicit conversion (narrowing) – may lose data
    double pi = 3.14159;
    int c = pi;    // double → int, truncates; compiler may warn

    // Explicit cast
    int d = (int)pi; // same as c but explicit

    // Pointer conversions
    int x = 42;
    void *vp = &x;      // int* → void* (safe, implicit)
    int *ip = (int *)vp; // void* → int* (explicit cast needed in C++ but not C)

    // Integer promotion: char, short are promoted to int in expressions
    char ch1 = 'A';
    char ch2 = 'B';
    int sum = ch1 + ch2; // ch1, ch2 promoted to int before addition

    printf("a: %d, b: %f, c: %d, d: %d, sum: %d\n", a, b, c, d, sum);
    return 0;
}
```
**Code Breakdown:**
- Implicit conversions happen in assignments, function arguments, and expressions.
- Narrowing conversions may lose information; many compilers warn.
- Explicit casts tell the compiler you know what you're doing.

---

## 10. Common Use Cases

| Use Case | Type | Example |
|----------|------|---------|
| **Loop counters** | `int` | `for (int i = 0; i < n; i++)` |
| **Large counts** | `size_t` | `size_t len = strlen(str);` |
| **Memory addresses** | Pointers | `void *ptr = malloc(100);` |
| **File sizes** | `size_t` or `off_t` | `size_t file_size;` |
| **Precise arithmetic** | `double` | `double result = sqrt(x);` |
| **Single‑precision performance** | `float` | `float dot_product;` |
| **Exact‑width network protocols** | `uint32_t`, `uint16_t` | `uint32_t ip_address;` |
| **Characters** | `char` | `char c = getchar();` |
| **Boolean flags** | `bool` | `bool is_valid = true;` |
| **Polymorphic data** | `void *` | `void *data;` |
| **Bit fields** | `unsigned int` | `unsigned int flags : 3;` |
| **Complex numbers** | `_Complex` (C99) | `double complex z;` |

---

## 11. Best Practices

- **Use exact‑width types** (`<stdint.h>`) for data with specific requirements (e.g., file formats, network protocols).
- **Use `size_t` for sizes and indices** – it is unsigned and large enough for any object size.
- **Prefer `double` over `float`** for general floating‑point arithmetic to avoid precision loss.
- **Use `const` for read‑only parameters and variables** – documents intent and helps optimisation.
- **Order structure members from largest to smallest** to minimise padding.
- **Avoid signed/unsigned mixed comparisons** – cast to a common type or use explicit checks.
- **Use `bool` from `<stdbool.h>`** for boolean values (C99).
- **Be aware of implicit integer promotions** – `char` and `short` are promoted to `int` in expressions.
- **Use `typedef` to create meaningful aliases** – e.g., `typedef uint32_t ipv4_addr_t;`.
- **Check for overflow** before performing arithmetic on signed integers.
- **Use `_Generic` for type‑generic macros** (C11) instead of relying on `#define` with casts.

---

## 12. Common Mistakes

### Mistake 1: Assuming Fixed Sizes
```c
// ❌ Wrong – sizes are implementation‑defined.
int arr[100];  // int might be 16, 32, or 64 bits.
// ✅ Correct – use exact‑width types if you need a specific size.
#include <stdint.h>
int32_t arr[100];  // exactly 32 bits each.
```

### Mistake 2: Signed/Unsigned Comparison
```c
// ❌ Wrong – surprising result.
int a = -1;
unsigned int b = 1;
if (a < b) { ... } // false because a becomes UINT_MAX.
// ✅ Correct – cast to signed or use a larger signed type.
if (a < (int)b) { ... }
```

### Mistake 3: Ignoring Padding in Structs
```c
// ❌ Wrong – may not be contiguous as expected.
struct MyStruct {
    char c;   // 1 byte
    int i;    // 4 bytes, but c is padded to 4 bytes alignment -> total 8 bytes
};
// ✅ Correct – reorder members to reduce padding.
struct MyStruct {
    int i;
    char c;   // now 4 + 1 + 3 padding = 8 bytes (still 8, but different order)
};
```

### Mistake 4: Using `char` for Binary Data
```c
// ❌ Wrong – char may be signed, causing sign extension when used as int.
char buffer[256];
int b = buffer[0]; // if char is signed and value > 127, it becomes negative.
// ✅ Correct – use unsigned char for binary data.
unsigned char buffer[256];
```

### Mistake 5: Forgetting that `char` may be Signed or Unsigned
```c
// ❌ Wrong – assumption of signedness.
char c = 200;    // if char is signed, this overflows (implementation‑defined).
// ✅ Correct – use signed char or unsigned char explicitly.
unsigned char c = 200;
```

### Mistake 6: Not Using `restrict` When Beneficial
```c
void add(int *a, int *b, int n) {   // compiler may not optimise due to aliasing.
    for (int i=0; i<n; i++) a[i] += b[i];
}
// ✅ Better – use restrict if you know they don't alias.
void add(int *restrict a, int *restrict b, int n) { ... }
```

---

## 13. Performance Considerations

- **Size matters** – larger types use more memory and may have slower access due to cache misses. Use the smallest type that fits your needs.
- **Alignment** – misaligned access can be slower (or cause faults). The compiler handles it, but padding affects memory usage.
- **Floating‑point** – `double` operations are often as fast as `float` on modern CPUs, but may use more memory bandwidth. Use `float` for large arrays.
- **Integer promotion** – `char` and `short` are promoted to `int` in expressions; this may have performance implications but is usually negligible.
- **`restrict`** – can enable vectorisation and other optimisations.
- **`const`** – allows constant propagation and can reduce memory accesses.

---

## 14. Security Considerations

- **Integer overflow** – signed overflow is undefined, leading to vulnerabilities. Always check before arithmetic.
- **Type confusion** – using a pointer of one type to access memory of another (via casts) can lead to undefined behaviour and security holes.
- **Signedness** – mixing signed and unsigned can lead to unexpected comparisons, which may be exploitable.
- **Buffer overflows** – using `char` arrays without proper bounds checking.
- **`volatile`** – not a security feature; for shared memory, use atomic types.
- **Padding** – sensitive data in structs may have padding that contains leftovers; always zero‑initialise structs.

---

## 15. Debugging Tips

- **Use `-Wall -Wextra`** – catches many type‑related warnings.
- **Use `-Wconversion`** – warns about implicit conversions that may change values.
- **Use `-Wsign-compare`** – warns about signed/unsigned comparisons.
- **Use `-Wpadded`** (GCC) – warns about padding in structs.
- **Use `sizeof` liberally** – to understand memory usage.
- **Use `offsetof`** from `<stddef.h>` to debug struct layouts.
- **Use `gdb`** – `print sizeof(struct MyStruct)`; `print &member`.
- **Use static analysers** – like Clang Static Analyzer, Coverity, etc.

---

## 16. When to Use

- **Always** – every variable must have a type.
- **Use the most appropriate type** for the data you are representing.
- **Use standard types** for portability; use exact‑width types when needed.

---

## 17. When Not to Use

- **Avoid `char` for arithmetic** unless you explicitly need it; use `int` for numeric values.
- **Avoid `float`** when precision is critical – use `double`.
- **Avoid `long`** for portability unless you need it; use `int32_t` or `int64_t` from `<stdint.h>`.
- **Avoid `void *`** unless necessary (generic data); it discards type safety.

---

## 18. Related Concepts

- **Type conversions** – implicit and explicit.
- **Type qualifiers** – `const`, `volatile`, `restrict`.
- **Storage classes** – `auto`, `static`, `extern`, `register`, `_Thread_local`.
- **Alignment** – how types are aligned in memory.
- **Endianness** – byte order affects multi‑byte types.
- **Padding** – structure member alignment.
- **`sizeof`** – operator to get size.
- **`_Generic`** – type‑generic selection.

---

## 19. Did You Know?

- The C standard only specifies minimum sizes for types; `int` can be 16, 32, or 64 bits. This was intentional to match the hardware.
- `char` is the only type guaranteed to be exactly 1 byte.
- `_Bool` (C99) is an unsigned integer type that can only hold 0 or 1.
- `long double` on x86 is often 80 bits (10 bytes) but may be padded to 12 or 16 bytes for alignment.
- C23 introduces `_BitInt(N)` for arbitrary‑precision integer types (N bits).
- C23 also adds `nullptr` constant and `typeof` operator.

---

## 20. Summary

- **Data types define** the set of values, memory layout, and operations for a variable.
- **C provides basic types** (char, int, float, double), with modifiers (short, long, signed, unsigned).
- **Exact‑width types** (`<stdint.h>`) offer portable, fixed‑size integers.
- **Derived types** include arrays, pointers, structs, unions, enums, and functions.
- **Type qualifiers** (`const`, `volatile`, `restrict`) add extra semantics.
- **Memory layout** is influenced by type sizes and alignment; struct padding can waste space.
- **Best practices**: use fixed‑width types for portability, avoid signed/unsigned mixing, use `size_t` for sizes, order struct members wisely.
- **Common mistakes**: assuming sizes, ignoring padding, using `char` for binary data, signed/unsigned confusion.
- **Understanding data types** is essential for writing correct, portable, and efficient C code.