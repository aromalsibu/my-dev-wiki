# Constants

## 1. Overview

### Definition
Constants are fixed values that cannot be modified during program execution. They represent literal data such as numbers, characters, and strings, as well as named values defined using `const` or `#define`. Constants provide a way to embed immutable data directly into your source code.

### Purpose
Constants serve several essential purposes:
- Make code more readable by giving meaningful names to fixed values.
- Prevent accidental modification of values that should remain unchanged.
- Enable compiler optimisations by allowing values to be evaluated at compile time.
- Improve maintainability by centralising configuration values.
- Provide type safety when using `const` (unlike `#define`).

### Where It Fits
Constants are fundamental to almost every C program. They appear in:
- **Variable declarations** – `const` qualified variables.
- **Literal values** – numbers, characters, strings directly in code.
- **Preprocessor macros** – `#define` constants (handled before compilation).
- **Enumeration constants** – named integer constants in `enum` definitions.

---

## 2. Why It Exists

### The Problem Without Constants
Consider code that uses "magic numbers" – literal values with no explanation:
```c
float area = 3.14159 * radius * radius;  // What is 3.14159?
int status = 404;                         // What does 404 mean?
if (response == 200) { ... }             // What is 200?
```
Such code is:
- **Unreadable** – the meaning of the numbers is unclear.
- **Error‑prone** – you might mistype the value.
- **Hard to maintain** – if the value changes (e.g., update π precision), you must find every occurrence.

### The Solution: Named Constants
Constants provide a way to give a name and meaning to fixed values:
```c
#define PI 3.14159265359f
#define HTTP_OK 200
#define HTTP_NOT_FOUND 404

float area = PI * radius * radius;
if (response == HTTP_OK) { ... }
```
Now the code is self‑documenting, and changes are localised.

### Why It Was Introduced
Constants have been part of programming languages from the beginning. In C, `#define` was inherited from the preprocessor to support early constant definitions. The `const` qualifier was added in C89 to provide a type‑safe alternative that integrates with the language's type system. Enumerations (`enum`) were also present early on to define named integer constants. Each approach serves different needs, as we'll explore.

---

## 3. Syntax / Basic Usage

### Literal Constants
```c
#include <stdio.h>

int main(void) {
    // Integer literals
    int decimal = 42;      // decimal (base 10)
    int octal = 052;       // octal (base 8) – leading 0
    int hex = 0x2A;        // hexadecimal (base 16) – 0x prefix
    int binary = 0b101010; // binary (base 2) – C23 feature

    // Floating-point literals
    float f = 3.14f;       // float literal (f suffix)
    double d = 3.14159;    // double literal (default)
    long double ld = 3.14L; // long double literal (L suffix)

    // Character literals
    char c = 'A';          // single character
    char newline = '\n';   // escape sequence
    char hex_char = '\x41';// hexadecimal escape (ASCII 65 = 'A')

    // String literals
    char *str = "Hello, World!";

    printf("%d %f %c %s\n", decimal, d, c, str);
    return 0;
}
```

### Named Constants – `#define` (Preprocessor)
```c
#include <stdio.h>

#define MAX_BUFFER 1024
#define PI 3.14159265359
#define GREETING "Hello"
#define SQUARE(x) ((x) * (x))  // function-like macro (not a constant, but related)

int main(void) {
    int buffer[MAX_BUFFER];           // Used in array size
    float area = PI * 5 * 5;          // Used in expression
    printf("%s, world!\n", GREETING); // Used as string literal
    return 0;
}
```

### Named Constants – `const` (Type‑safe)
```c
#include <stdio.h>

int main(void) {
    const int MAX_BUFFER = 1024;     // const-qualified variable
    const float PI = 3.14159265359f;
    const char *GREETING = "Hello";  // pointer to const char

    // MAX_BUFFER = 2048;            // ❌ Error – cannot modify const
    float area = PI * 5 * 5;
    printf("%s, world!\n", GREETING);
    return 0;
}
```

### Named Constants – `enum`
```c
#include <stdio.h>

enum HttpStatus {
    HTTP_OK = 200,
    HTTP_NOT_FOUND = 404,
    HTTP_INTERNAL_ERROR = 500
};

enum Boolean { FALSE = 0, TRUE = 1 };

int main(void) {
    int status = HTTP_OK;
    if (status == HTTP_OK) {
        printf("Success\n");
    }
    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>

// #define constants – handled by preprocessor
#define MAX_BUFFER 1024
// This is a textual substitution. Every occurrence of MAX_BUFFER becomes 1024.
// No type checking; use with caution.

#define PI 3.14159265359
// Floating-point constant defined as a macro.

#define GREETING "Hello"
// String constant defined as a macro.

int main(void) {
    // Integer literals
    int decimal = 42;      // Decimal literal – base 10, value 42.
    int octal = 052;       // Octal literal – leading zero indicates base 8, value 42 decimal.
    int hex = 0x2A;        // Hexadecimal literal – 0x prefix, value 42 decimal.
    int binary = 0b101010; // Binary literal – 0b prefix (C23), value 42 decimal.

    // Floating-point literals
    float f = 3.14f;       // f suffix forces float type; without suffix, it's double.
    double d = 3.14159;    // Default floating-point type is double.
    long double ld = 3.14L;// L suffix for long double.

    // Character literals
    char c = 'A';          // Single quote for character; value is ASCII 65.
    char newline = '\n';   // Escape sequence: newline (ASCII 10).
    char hex_char = '\x41';// Hexadecimal escape: '\x' followed by hex digits.

    // String literals
    char *str = "Hello, World!";
    // String literal is stored in read-only memory (typically .rodata).
    // The pointer str points to the first character.

    // const-qualified constants
    const int MAX = 100;   // const variable – cannot be modified after initialisation.
    // MAX = 200;          // ❌ Error: assignment of read-only variable.

    // Enumeration constants
    enum Status { IDLE = 0, RUNNING = 1, STOPPED = 2 };
    enum Status current = RUNNING;  // enum variable with named constant.

    printf("%d %f %c %s\n", decimal, d, c, str);
    return 0;
}
```

---

## 4. Mental Model – Constants as Fixed Reference Points

Think of a city map. Constants are like:
- **Landmarks** – fixed points that everyone knows (`PI`, `HTTP_OK`). They never move.
- **Rulers** – fixed measurement units (e.g., `MAX_BUFFER = 1024`).
- **Signposts** – fixed text labels (`GREETING`).

Using literal values directly (like `3.14159` or `404`) is like using coordinates instead of landmark names – it's harder to navigate. Constants give you a name you can remember and understand.

**#define** is like a **stencil** – it stamps the value everywhere before the building begins. If you change the stencil, everywhere it was stamped changes too.

**const** is like a **locked box** – the value is inside, and the compiler enforces that you cannot change it. It's type‑safe and has a proper storage location.

**enum** is like a **colour‑coded label** – it defines a set of related named values (like traffic light colours: RED, YELLOW, GREEN).

---

## 5. Core Concepts

| Concept | Description | Examples |
|---------|-------------|----------|
| **Integer Literals** | Fixed integer values. Can be decimal, octal, hex, or binary. Suffixes (`u`, `l`, `ll`, etc.) specify type. | `42`, `052`, `0x2A`, `0b101010`, `100u`, `1000LL` |
| **Floating Literals** | Fixed floating-point values. `f`/`F` for float, `l`/`L` for long double. | `3.14f`, `2.718`, `1.0L`, `1e-5` |
| **Character Literals** | Single characters enclosed in single quotes. Escape sequences available. | `'A'`, `'\n'`, `'\x41'`, `'\077'` |
| **String Literals** | Sequences of characters in double quotes. Null-terminated. | `"Hello"`, `"Line1\nLine2"` |
| **`#define` Constants** | Preprocessor macros; textual substitution before compilation. No type safety, no scope. | `#define PI 3.14` |
| **`const` Constants** | Type‑safe constants; have type, scope, and storage. Enforced at compile time and runtime. | `const int MAX = 100;` |
| **Enumeration Constants** | Named integer constants; scope‑aware and type‑safe. | `enum { RED, GREEN, BLUE };` |
| **Enumeration Constants (C23)** | Can have explicit underlying types. | `enum : unsigned char { RED, GREEN };` |

---

## 6. How It Works – Constant Handling

### Literal Constants
1. **Lexer** recognises literal patterns (e.g., digits for integers, quotes for chars).
2. **Parser** assigns a type based on the literal's form and suffix.
3. **Compiler** stores the value in the object file:
   - Integer literals are embedded in the instruction stream or data section.
   - Floating literals are stored in `.rodata` or `.data`.
   - String literals are placed in `.rodata` (read‑only) and have static storage duration.
4. **Runtime** accesses the literal as a fixed value.

### `#define` Constants
1. **Preprocessor** performs textual substitution before the compiler sees the code.
2. No token remains after preprocessing (the value is inserted directly).
3. No type checking; the literal value is inserted at the point of use.

### `const` Constants
1. **Compiler** treats `const` as a type qualifier.
2. It enforces that the variable cannot be modified (compiler error if you try).
3. For constants with static storage, the value may be placed in `.rodata` (read‑only memory).
4. For local `const` variables, the compiler may optimise them away, using the value directly.

### `enum` Constants
1. **Compiler** treats enumeration constants as integer constants.
2. They are resolved at compile time and can be used in constant expressions.
3. They have their own scope (if defined inside a function or structure, they are visible there).

### ASCII Diagram – Constant Handling Pipeline
```
#define PI 3.14          const float PI = 3.14f;    enum { PI = 3.14f };

     ▼                      ▼                             ▼
Preprocessor       Compiler                      Compiler
(textual replace)  (type-checked)               (type-checked)
     ▼                      ▼                             ▼
PI becomes 3.14   const variable in .rodata     Enum constant in symbol table
     ▼                      ▼                             ▼
No token remains   Runtime: read-only memory    Compile-time: value substituted
```

---

## 7. Internal Architecture – Storage of Constants

### Literal Constants
```
┌─────────────────────────────────────────────┐
│               .rodata (read-only)           │
│  ┌─────────────────────────────────────┐    │
│  │ "Hello, World!" (string literal)    │    │
│  │ 3.14159 (floating literal)          │    │
│  │ 42 (integer literal in instructions)│    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

### `const` Constants
```c
const int global_const = 42;  // stored in .rodata
static const int file_const = 100; // stored in .rodata

int main(void) {
    const int local_const = 50;  // may be optimised to immediate value
    // Or stored on stack; cannot be modified.
}
```

### `enum` Constants
- Enumerators are handled at compile time; they do not occupy storage.
- They are replaced by their integer values, like `#define`.

---

## 8. Lifecycle / Workflow of a Constant

### Literal Constants
1. **Written** – source code contains literal.
2. **Lexed** – recognised as a constant token.
3. **Compiled** – embedded in object file.
4. **Linked** – merged into final executable.
5. **Executed** – value is loaded from memory or immediate operand.

### `#define` Constants
1. **Written** – macro defined.
2. **Preprocessed** – replaced with literal value.
3. **Compiled** – value used directly.
4. **Executed** – value appears in machine code as immediate operand or data.

### `const` Constants
1. **Written** – declared with `const`.
2. **Compiled** – type-checked, storage allocated.
3. **Linked** – address resolved (if global/static).
4. **Executed** – value is accessed from memory (if not optimised away).

---

## 9. Practical Examples

### Example 1: All Integer Literal Types
```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    // Decimal literals
    int dec = 42;                    // int
    long l = 42L;                    // long
    unsigned long ul = 42UL;         // unsigned long
    long long ll = 42LL;             // long long
    unsigned long long ull = 42ULL;  // unsigned long long

    // Hexadecimal literals
    int hex = 0xFF;                  // 255 decimal
    unsigned int hex_u = 0xFFFFu;    // 65535
    long hex_l = 0xFFFFFFFFL;        // 4294967295 (on 64-bit)

    // Octal literals
    int oct = 010;                   // 8 decimal

    // Binary literals (C23)
    int bin = 0b1010;                // 10 decimal

    printf("dec: %d, hex: %d, oct: %d, bin: %d\n", dec, hex, oct, bin);
    printf("long: %ld, unsigned long: %lu\n", l, ul);
    printf("long long: %lld, unsigned long long: %llu\n", ll, ull);

    return 0;
}
```
**Code Breakdown:**
- Suffixes control the type of integer literal:
  - `L` or `l` – `long`.
  - `LL` or `ll` – `long long`.
  - `U` or `u` – `unsigned`.
  - `UL` – `unsigned long`.
  - `ULL` – `unsigned long long`.
- Prefixes control the base:
  - `0x` – hexadecimal.
  - `0` – octal.
  - `0b` – binary (C23).
- Without suffix, the type is the smallest that can hold the value (`int`, `long`, or `long long`).

### Example 2: Floating‑Point Literal Types
```c
#include <stdio.h>
#include <float.h>

int main(void) {
    float f = 3.14f;          // float (f suffix)
    double d = 3.14;          // double (default)
    long double ld = 3.14L;   // long double (L suffix)

    // Scientific notation
    double scientific = 1.23e-4;   // 0.000123
    double exponent = 1e10;        // 10,000,000,000

    // Special values
    float inf = INFINITY;          // from math.h
    float nan = NAN;               // from math.h

    printf("float: %f\n", f);
    printf("double: %f\n", d);
    printf("long double: %Lf\n", ld);
    printf("scientific: %e\n", scientific);
    printf("exponent: %e\n", exponent);

    return 0;
}
```
**Code Breakdown:**
- `f` suffix forces `float` type (otherwise `double`).
- `L` suffix forces `long double`.
- Scientific notation with `e` or `E` (e.g., `1.23e-4`).
- `INFINITY` and `NAN` from `<math.h>` are not literals but macros for constants.

### Example 3: Character Literals and Escape Sequences
```c
#include <stdio.h>

int main(void) {
    // Simple characters
    char a = 'A';
    char newline = '\n';
    char tab = '\t';
    char backslash = '\\';
    char single_quote = '\'';
    char double_quote = '"';   // Inside single quotes, " is just a character

    // Octal escape (ASCII code in octal)
    char octal_a = '\101';     // 'A' (octal 101 = decimal 65)
    char octal_newline = '\012'; // newline (octal 12 = decimal 10)

    // Hexadecimal escape
    char hex_a = '\x41';       // 'A' (hex 41 = decimal 65)
    char hex_newline = '\x0A'; // newline

    // Universal character names (C99+)
    char euro = '\u20AC';      // Unicode € (if char supports it, usually wchar_t)

    printf("%c%c", a, newline);    // Outputs "A\n"
    printf("Tab\tbetween\n");
    printf("Backslash: %c\n", backslash);
    printf("Octal A: %c, Hex A: %c\n", octal_a, hex_a);

    return 0;
}
```
**Code Breakdown:**
- Escape sequences start with `\`:
  - `\n` – newline.
  - `\t` – tab.
  - `\\` – backslash.
  - `\'` – single quote.
  - `\"` – double quote.
  - `\ooo` – octal value (3 digits max).
  - `\xhh` – hexadecimal value (any number of hex digits).
  - `\u` – Unicode (C99+, 4 hex digits).
  - `\U` – Unicode (C99+, 8 hex digits).

### Example 4: `const` vs `#define` – Practical Comparison
```c
#include <stdio.h>

#define MAX_USERS 100      // Preprocessor constant
const int MAX_CONNECTIONS = 50; // const constant

void process_connections(int n) {
    // MAX_USERS is replaced by 100 before compilation
    // MAX_CONNECTIONS is a const variable
    if (n > MAX_CONNECTIONS) {
        printf("Exceeds max connections (%d)\n", MAX_CONNECTIONS);
    }
}

int main(void) {
    // Array size must be a constant expression
    int user_ids[MAX_USERS];  // OK – preprocessor substitutes 100
    // int ids[MAX_CONNECTIONS]; // ❌ Error in C89/C90 (allowed in C99+ with VLA if const? Actually const is not a constant expression in C89; use enum)

    // 'const' variable can be used where a read-only variable is needed
    const float PI = 3.14159f;
    float area = PI * 5 * 5;   // OK – PI is const, value known

    printf("Users: %d, Connections: %d\n", MAX_USERS, MAX_CONNECTIONS);
    return 0;
}
```
**Code Breakdown:**
- `#define` is pure textual substitution; it works anywhere a literal value works.
- `const` creates a read‑only variable; it has type and address (except when optimised).
- In C89, a `const` variable is not a "constant expression"; you cannot use it for array sizes. Use `#define` or `enum` for that.

### Example 5: Enumeration Constants
```c
#include <stdio.h>

// Default values: RED=0, GREEN=1, BLUE=2
enum Color { RED, GREEN, BLUE };

// Explicit values
enum Status { IDLE = 0, RUNNING = 1, STOPPED = 2, ERROR = -1 };

// Sequential values with some explicit
enum Month { JAN = 1, FEB, MAR, APR, MAY, JUN,
             JUL, AUG, SEP, OCT, NOV, DEC };
// FEB=2, MAR=3, ..., DEC=12

int main(void) {
    enum Color c = RED;
    enum Status s = RUNNING;
    enum Month m = DEC;

    printf("Color: %d, Status: %d, Month: %d\n", c, s, m);

    // Enum constants are integers
    for (enum Month month = JAN; month <= DEC; month++) {
        printf("%d ", month);
    }
    printf("\n");

    return 0;
}
```
**Code Breakdown:**
- Enumeration constants are compile‑time integer constants.
- They have scope: if defined inside a function, they are visible only there.
- They are not macros; they are part of the language and have type `int`.
- You can define specific values; subsequent values increment by 1.

---

## 10. Common Use Cases

| Use Case | Approach | Example |
|----------|----------|---------|
| **Array sizes** | `#define` or `enum` | `#define MAX 100`<br>`int arr[MAX];` |
| **Configuration values** | `#define` or `const` | `#define TIMEOUT 30` |
| **Error codes** | `enum` | `enum { ERR_NONE, ERR_INVALID, ERR_TIMEOUT };` |
| **Read-only parameters** | `const` | `void print(const char *str);` |
| **Hardware addresses** | `#define` | `#define REG_BASE 0x4000` |
| **Magic numbers** | `#define` or `const` | `#define PI 3.14159f` |
| **Mathematical constants** | `const` or `#define` | `const double E = 2.71828;` |
| **Strings** | String literals | `"Hello"` |

---

## 11. Best Practices

- **Prefer `const` over `#define` for type safety** – `const` variables have a type and are checked by the compiler.
- **Use `#define` for values that must be constant expressions** – array sizes, preprocessor conditions.
- **Use `enum` for related sets of integer constants** – status codes, flags, options.
- **Use `static const` for file‑private constants** – limits visibility.
- **Use meaningful names** – `MAX_BUFFER` is better than `MAX`.
- **Use uppercase for macro constants** – convention to distinguish them from variables.
- **Use `const` for function parameters that should not be modified** – documents intent and prevents bugs.
- **Do not use magic numbers** – define constants for all non‑obvious numeric values.
- **Use `const` with pointers carefully** – `const char *p` means the data pointed to is const; `char * const p` means the pointer is const.

---

## 12. Common Mistakes

### Mistake 1: Using `#define` Without Parentheses
```c
// ❌ Wrong – operator precedence issues
#define MULTIPLY(a, b) a * b
int result = MULTIPLY(2 + 3, 4 + 5); // Expands to 2 + 3 * 4 + 5 = 19 (wrong!)
// ✅ Correct – wrap in parentheses
#define MULTIPLY(a, b) ((a) * (b))
```

### Mistake 2: Modifying `const` Variables
```c
// ❌ Wrong – compilation error
const int x = 10;
x = 20;   // Error: assignment of read-only variable
// ✅ Correct – initialise and never modify
const int x = 10;
```

### Mistake 3: Using `const` for Array Size in C89
```c
// ❌ Wrong – const is not a constant expression in C89
const int SIZE = 10;
int arr[SIZE];   // Error in C89; allowed in C99+ (VLA)
// ✅ Correct – use #define or enum
#define SIZE 10
int arr[SIZE];
```

### Mistake 4: String Literal Modification
```c
// ❌ Wrong – undefined behaviour (string literal is read-only)
char *str = "Hello";
str[0] = 'h';   // May crash or silently fail
// ✅ Correct – use array if you need to modify
char str[] = "Hello";
str[0] = 'h';   // OK – array is modifiable
```

### Mistake 5: Mixing Integer Literal Types Incorrectly
```c
// ❌ Wrong – overflow or unexpected type
unsigned int x = 50000;   // OK if int is 32-bit
long long y = 5000000000; // OK for long long
int z = 5000000000;       // Overflow if int is 32-bit
// ✅ Correct – use appropriate suffix
long long y = 5000000000LL; // Explicit type
```

---

## 13. Performance Considerations

- **`#define` constants** – no runtime overhead; value is substituted directly.
- **`const` variables** – may be optimised away (if the compiler can determine the value at compile time) or stored in `.rodata`. Minimal overhead.
- **`enum` constants** – compile‑time values; no runtime overhead.
- **String literals** – stored in read‑only memory; no runtime allocation.
- **Integer literals** – embedded in instructions as immediate operands.

---

## 14. Security Considerations

- **String literals** – are read‑only; attempting to write causes undefined behaviour (often a segmentation fault).
- **`const`** – helps enforce read‑only access, preventing accidental modification of sensitive data.
- **`#define`** – can be used to embed sensitive values, but they will be visible in the binary (strings are easily extractable).
- **Avoid hard‑coding passwords or secrets** – even with `const` or `#define`, they are visible in the binary.

---

## 15. Debugging Tips

- **Use `-E`** to see `#define` expansions.
- **Inspect `.rodata`** with `objdump -s -j .rodata myprogram` to see string literals and constants.
- **Check symbol tables** with `nm` to see `const` variables and their storage.
- **Use `gdb`** to examine `const` variables – they are in memory (or optimised away).

---

## 16. When to Use

- **Always** – use constants instead of magic numbers.
- **When you need a compile‑time constant** – use `#define` or `enum`.
- **When you need type safety** – use `const`.
- **When you need a set of related values** – use `enum`.
- **For string literals** – use double quotes directly.

---

## 17. When Not to Use

- **Avoid `#define` for type‑sensitive values** – use `const` instead.
- **Avoid `#define` for values that need scope** – it has no scope; `const` and `enum` do.
- **Avoid `const` for array sizes in C89** – it's not a constant expression; use `#define` or `enum`.

---

## 18. Related Concepts

- **Literals** – the raw values.
- **`const` qualifier** – read‑only variables.
- **`#define`** – preprocessor macros.
- **`enum`** – enumeration constants.
- **Storage classes** – how constants are stored.
- **Type system** – how constants are typed.

---

## 19. Did You Know?

- In C, `const` does not mean "constant" in the sense of a compile‑time value; it means "read‑only". A `const` variable is not a constant expression (except in C++).
- The `#define` directive does not have a scope; it applies from the point of definition to the end of the file (or until `#undef`).
- Enumeration constants have `int` type. In C23, you can specify a different underlying type (`enum : unsigned char { ... }`).
- String literals are actually arrays of `char` with static storage duration; they can be used to initialise `char` arrays.
- The `L` prefix creates a wide string literal: `L"wide"` (type `wchar_t[]`). Introduced in C89.

---

## 20. Summary

- **Constants are fixed values** that do not change during program execution.
- **C provides several ways to define constants**: literals, `#define`, `const`, and `enum`.
- **Literals** are the actual values written in code (e.g., `42`, `3.14`, `'A'`, `"Hello"`).
- **`#define`** is a preprocessor macro for textual substitution; no type checking.
- **`const`** is a type‑safe read‑only variable; it has type, scope, and storage.
- **`enum`** defines a set of named integer constants; scope‑aware and type‑safe.
- **Best practices**: use `const` for type safety, `#define` for constant expressions, `enum` for related constants.
- **Avoid magic numbers** – always use named constants for clarity and maintainability.
- **Understanding constants** is fundamental to writing robust, readable, and maintainable C code.