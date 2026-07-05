# Literals

## 1. Overview

### Definition
A literal is a syntactic token that directly represents a fixed value in source code. Unlike named constants (`const`, `#define`, `enum`), literals are the **actual values** you write – numbers, characters, strings, and compound initialisers. They are the raw data from which constants and variables are built.

### Purpose
Literals are the simplest way to embed data into your program. They provide immediate, readable values that the compiler can directly use. They form the foundation of every expression and initialisation. Without literals, you could never set a variable to a specific initial value or embed a message in your code.

### Where It Fits
Literals appear everywhere in C source:
- **Initialisers** – `int x = 42;`
- **Expressions** – `return a + 3.14;`
- **Function arguments** – `printf("Hello");`
- **Array initialisers** – `int arr[] = {1, 2, 3};`
- **Case labels** – `case 5:`
- **Enumerators** – `enum { RED = 1 };`

They are fundamental tokens that the lexer recognises and the compiler converts into internal representations.

---

## 2. Why It Exists

### The Problem Without Literals
Without literals, you could not write any constant data directly. Every value would have to come from a variable or a function, making even the simplest program cumbersome. You would need to pre‑initialise every value elsewhere, reducing readability and defeating the purpose of high‑level languages.

### The Solution: Literals as Immediate Data
Literals provide a way to write data values inline. The compiler interprets them according to their syntax and generates the appropriate machine code or data storage. They are the building blocks of all initialisation and constant propagation.

### Why They Were Introduced
Literals have existed in every high‑level language since the earliest days. In C, they are designed to be concise and flexible, allowing you to express values in decimal, octal, hexadecimal, binary (C23), with suffixes to control type, and with escape sequences for characters. This reflects C’s systems‑programming heritage – you can specify values exactly as they appear in hardware.

---

## 3. Syntax / Basic Usage

### Integer Literals
```c
// Decimal (base 10) – default
int dec = 42;

// Octal (base 8) – leading zero
int oct = 052;    // 42 decimal

// Hexadecimal (base 16) – 0x prefix
int hex = 0x2A;   // 42 decimal

// Binary (base 2) – 0b prefix (C23)
int bin = 0b101010; // 42 decimal

// Suffixes to specify type
unsigned int u = 42u;
long l = 42L;
unsigned long ul = 42UL;
long long ll = 42LL;
unsigned long long ull = 42ULL;

// Combined type and size
long long big = 123456789012345LL;
```

### Floating‑Point Literals
```c
// Double (default)
double d1 = 3.14;
double d2 = .5;       // 0.5
double d3 = 1.;       // 1.0
double d4 = 1e-3;     // 0.001 (scientific notation)

// Float – f or F suffix
float f = 3.14f;

// Long double – l or L suffix
long double ld = 3.14L;

// Hexadecimal floating constants (C99) – 0x prefix with p exponent
double hex_float = 0x1.2p3; // 1.125 * 2^3 = 9.0
```

### Character Literals
```c
// Simple character
char c = 'A';

// Escape sequences
char newline = '\n';
char tab = '\t';
char backslash = '\\';
char single_quote = '\'';
char double_quote = '"';

// Octal escape: \ooo (3 octal digits max)
char octal = '\101';     // 'A' (octal 101 = decimal 65)

// Hexadecimal escape: \xhh (any number of hex digits)
char hex_char = '\x41';  // 'A' (hex 41 = decimal 65)

// Universal character names (C99) – for wide characters
wchar_t euro = L'\u20AC'; // Unicode Euro sign (if wchar_t supports it)
```

### String Literals
```c
// Ordinary string literal
char *str = "Hello, World!";

// Wide string literal (C89)
wchar_t *wstr = L"Hello";

// UTF‑8 string literal (C11)
char *utf8 = u8"Hello";   // type char[]

// UTF‑16 string literal (C11)
char16_t *utf16 = u"Hello";

// UTF‑32 string literal (C11)
char32_t *utf32 = U"Hello";

// String concatenation (adjacent literals are joined)
char *concat = "Hello " "World"; // "Hello World"

// Multi‑line string (using backslash or adjacent literals)
char *multi = "Line 1\n"
              "Line 2\n";
```

### Compound Literals (C99)
```c
// Compound literal – creates an unnamed object of the given type
int *p = (int []){1, 2, 3};  // pointer to array
struct Point { int x, y; };
struct Point pt = (struct Point){10, 20};

// Can be used as function arguments
void print_point(struct Point p);
print_point((struct Point){3, 4});
```

---

## 4. Mental Model – Literals as Direct Imprints

Imagine you are carving a sculpture. Literals are like the chisel marks directly shaping the material:

- **Integer literals** are like precise measurements etched in stone – `42` is a fixed number.
- **Floating literals** are like exact weight measurements – `3.14` is a specific quantity.
- **Character literals** are like single letters stamped into the surface – `'A'` is that letter.
- **String literals** are like full sentences inscribed – `"Hello"` is that sequence of characters.
- **Compound literals** are like pre‑assembled blocks – `(int []){1,2,3}` is a ready‑made array.

Unlike named constants, literals are not given a name – they are the raw value itself, used directly.

---

## 5. Core Concepts

| Literal Type | Syntax | Examples | Type (default) |
|--------------|--------|----------|----------------|
| **Integer** | Decimal, octal (0), hex (0x), binary (0b) | `42`, `052`, `0x2A`, `0b101010` | `int` (or larger if needed) |
| **Floating** | Decimal, scientific, hex (C99) | `3.14`, `1e-3`, `0x1.2p3` | `double` (unless suffixed) |
| **Character** | Single quotes, with escapes | `'A'`, `'\n'`, `'\x41'` | `int` (in expressions), `char` (when assigned) |
| **String** | Double quotes, with escapes | `"Hello"`, `"Line1\nLine2"` | `char[]` (array of char) |
| **Compound** | `(type){ initialisers }` | `(int []){1,2,3}` | Unnamed object of specified type |
| **Wide / UTF‑8 / UTF‑16 / UTF‑32** | Prefixes: `L`, `u8`, `u`, `U` | `L"wide"`, `u8"UTF-8"` | `wchar_t[]`, `char[]`, `char16_t[]`, `char32_t[]` |

### Suffixes for Integer Literals
| Suffix | Type |
|--------|------|
| `u` or `U` | `unsigned int` |
| `l` or `L` | `long` |
| `ul`, `UL`, `uL`, `Ul` | `unsigned long` |
| `ll` or `LL` | `long long` |
| `ull`, `ULL`, `uLL`, `Ull` | `unsigned long long` |
| `z` or `Z` (C23) | `size_t` (the result of `sizeof`) |

### Suffixes for Floating Literals
| Suffix | Type |
|--------|------|
| `f` or `F` | `float` |
| `l` or `L` | `long double` |
| (no suffix) | `double` |

### Escape Sequences in Character and String Literals
| Sequence | Meaning |
|----------|---------|
| `\a` | Alert (bell) |
| `\b` | Backspace |
| `\f` | Form feed |
| `\n` | Newline |
| `\r` | Carriage return |
| `\t` | Tab |
| `\v` | Vertical tab |
| `\\` | Backslash |
| `\'` | Single quote |
| `\"` | Double quote |
| `\?` | Question mark (used to avoid trigraphs) |
| `\ooo` | Octal value (1‑3 octal digits) |
| `\xhh` | Hexadecimal value (any number of hex digits) |
| `\u` | Unicode 16‑bit (C99, 4 hex digits) |
| `\U` | Unicode 32‑bit (C99, 8 hex digits) |

---

## 6. How It Works – Lexical Analysis and Typing

1. **Lexer** reads the source and identifies a literal token based on its pattern:
   - Digits, optional `0x`, `0b`, etc. → integer or floating.
   - Single quote → character literal.
   - Double quote → string literal.
   - Parenthesised type with braces → compound literal.

2. **Parser** assigns a type to the literal:
   - For integers, the type is the smallest that can hold the value, influenced by suffixes.
   - For floats, the type is `double` unless suffixed.
   - For characters, the type is `int` in expressions, but assigning to `char` narrows.
   - For strings, the type is `char[]` (or `wchar_t[]`, etc.) with static storage duration.

3. **Compiler** stores the literal:
   - Integer/floating literals may become immediate operands in instructions.
   - String literals are placed in `.rodata` and are null‑terminated.
   - Character literals are converted to their integer value (ASCII or Unicode).

4. **Semantic analysis** checks that the literal is used correctly (e.g., no overflow, proper type matching).

### ASCII Diagram – Literal Processing
```
Source: int x = 42;
   │
   ▼
Lexer: sees digits '4', '2' – integer literal token (42)
   │
   ▼
Parser: constant expression – type int, value 42
   │
   ▼
Compiler: generates mov instruction with immediate 42
   │
   ▼
Machine code: mov eax, 42
```

---

## 7. Internal Architecture – Storage of Literals

### String Literals
```
┌─────────────────────────────────────────────┐
│               .rodata                       │
│  ┌─────────────────────────────────────┐    │
│  │ "Hello, World!" (null-terminated)   │    │
│  │ "Line1\nLine2"                      │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```
- String literals have **static storage duration** – they exist for the entire program lifetime.
- They are stored in read‑only memory; attempts to modify them are undefined behaviour.

### Integer and Floating Literals
- Often embedded directly in the instruction stream as immediate operands (e.g., `mov eax, 42`).
- For larger constants or if their address is taken (e.g., `&(int){42}`), they may be stored in `.data` or `.rodata`.

### Character Literals
- Treated as integer constants; they become immediate operands (e.g., `mov al, 'A'`).
- No separate storage; they are part of the instruction.

### Compound Literals
- If used at file scope, they are stored in `.data` or `.rodata`.
- If used inside a function, they are allocated on the stack (like local variables) but with initialiser.

---

## 8. Lifecycle / Workflow of a Literal

1. **Written** – the programmer types the literal in source.
2. **Lexed** – recognised as a token of a specific literal type.
3. **Parsed** – its value and type are determined.
4. **Compiled** – the value is encoded in the object code or data section.
5. **Linked** – string literals are merged, addresses resolved.
6. **Executed** – the value is loaded from memory or used as an immediate operand.

---

## 9. Practical Examples

### Example 1: Integer Literals with Different Bases and Suffixes
```c
#include <stdio.h>
#include <stdint.h>

int main(void) {
    // Decimal
    int dec = 42;
    printf("Dec: %d\n", dec);

    // Octal (leading zero)
    int oct = 052;        // 42 decimal
    printf("Oct: %d\n", oct);

    // Hexadecimal (0x prefix)
    int hex = 0x2A;       // 42 decimal
    printf("Hex: %d\n", hex);

    // Binary (C23) – 0b prefix
    int bin = 0b101010;   // 42 decimal
    printf("Bin: %d\n", bin);

    // Suffixes
    unsigned int u = 42u;        // unsigned int
    long l = 42L;                // long
    unsigned long ul = 42UL;     // unsigned long
    long long ll = 42LL;         // long long
    unsigned long long ull = 42ULL; // unsigned long long

    printf("u: %u, l: %ld, ul: %lu, ll: %lld, ull: %llu\n",
           u, l, ul, ll, ull);

    // Hexadecimal with suffix
    unsigned int hex_u = 0xFFFFu;
    long hex_l = 0xFFFFFFFFL;   // may be long or long long depending on platform
    printf("hex_u: %u, hex_l: %ld\n", hex_u, hex_l);

    return 0;
}
```
**Code Breakdown:**
- Each literal's suffix explicitly sets the type.
- Octal and hexadecimal are convenient for bitwise operations or memory addresses.
- C23 binary literals improve readability for bit masks.

### Example 2: Floating‑Point Literals and Precision
```c
#include <stdio.h>
#include <float.h>

int main(void) {
    // double (default)
    double d = 3.141592653589793;
    printf("double: %.15f\n", d);

    // float (suffix f)
    float f = 3.141592653589793f;
    printf("float: %.15f\n", f);   // less precision

    // long double (suffix L)
    long double ld = 3.141592653589793L;
    printf("long double: %.15Lf\n", ld);

    // Scientific notation
    double sci = 1.23e-4;   // 0.000123
    double big = 1e10;      // 10,000,000,000
    printf("sci: %e, big: %e\n", sci, big);

    // Hexadecimal floating (C99) – exact representation
    double hex_float = 0x1.2p3;  // 1.125 * 2^3 = 9.0
    printf("hex_float: %f\n", hex_float);

    return 0;
}
```
**Code Breakdown:**
- `double` literals have the highest default precision.
- `f` suffix forces `float`, which may lose precision.
- `L` suffix gives `long double` (if supported).
- Scientific notation is useful for very large or small numbers.
- Hex floating constants can represent exact binary fractions, useful for avoiding rounding errors.

### Example 3: Character Literals and Escape Sequences
```c
#include <stdio.h>
#include <wchar.h>
#include <locale.h>

int main(void) {
    // Basic characters
    char c = 'A';
    printf("char: %c, ASCII: %d\n", c, c);

    // Escape sequences
    char tab = '\t';
    char newline = '\n';
    char backslash = '\\';
    char single_quote = '\'';
    char double_quote = '"';

    printf("Tab%cbetween%c", tab, newline);
    printf("Backslash: %c, Quote: %c, Double: %c%c",
           backslash, single_quote, double_quote, newline);

    // Octal and hex escapes
    char octal_a = '\101';   // 'A'
    char hex_a = '\x41';     // 'A'
    printf("Octal A: %c, Hex A: %c\n", octal_a, hex_a);

    // Null character (string terminator)
    char null_char = '\0';
    printf("Null char value: %d\n", null_char); // prints 0

    // Wide character literal (C89)
    wchar_t wc = L'Ω';
    setlocale(LC_ALL, "");
    wprintf(L"Wide char: %lc\n", wc);

    // Universal character name (C99)
    wchar_t euro = L'\u20AC';
    wprintf(L"Euro sign: %lc\n", euro);

    return 0;
}
```
**Code Breakdown:**
- Escape sequences let you represent non‑printable characters.
- Octal and hex escapes are useful for arbitrary byte values.
- Wide character literals (`L'...'`) store multibyte characters (type `wchar_t`).
- Universal character names (`\u` and `\U`) represent Unicode code points.

### Example 4: String Literals – Types and Concatenation
```c
#include <stdio.h>
#include <wchar.h>
#include <locale.h>

int main(void) {
    // Ordinary string
    char *s1 = "Hello, World!";
    printf("%s\n", s1);

    // String with escape sequences
    char *s2 = "Line 1\nLine 2\tTabbed";
    printf("%s\n", s2);

    // Adjacent string concatenation
    char *s3 = "Hello " "World";   // "Hello World"
    printf("%s\n", s3);

    // Multi‑line using adjacent literals
    char *s4 = "This is a long string "
               "that spans multiple lines "
               "in the source code.";
    printf("%s\n", s4);

    // Wide string literal
    wchar_t *ws = L"Wide string";
    setlocale(LC_ALL, "");
    wprintf(L"%ls\n", ws);

    // UTF‑8 string literal (C11)
    char *utf8 = u8"UTF-8 string with Unicode: こんにちは";
    printf("%s\n", utf8);

    // UTF‑16 and UTF‑32 (C11)
    char16_t *utf16 = u"UTF-16 string";
    char32_t *utf32 = U"UTF-32 string";
    // Note: printing these requires appropriate handling, not shown.

    return 0;
}
```
**Code Breakdown:**
- Adjacent string literals are concatenated at compile time – useful for formatting.
- Wide strings use `L` prefix; type `wchar_t[]`.
- UTF‑8, UTF‑16, UTF‑32 prefixes (`u8`, `u`, `U`) were added in C11.
- String literals are read‑only; modifying them is undefined behaviour.

### Example 5: Compound Literals (C99)
```c
#include <stdio.h>

struct Point {
    int x;
    int y;
};

void print_point(struct Point p) {
    printf("(%d, %d)\n", p.x, p.y);
}

int main(void) {
    // Compound literal for a struct
    struct Point p = (struct Point){10, 20};
    print_point(p);

    // Passing compound literal directly
    print_point((struct Point){30, 40});

    // Compound literal for an array
    int *arr = (int []){1, 2, 3, 4, 5};
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    // Using sizeof on compound literal (C99)
    size_t size = sizeof((int []){1, 2, 3});
    printf("Size of compound literal array: %zu\n", size);

    // Compound literal with designated initialisers (C99)
    struct Point p2 = (struct Point){.y = 5, .x = 10};
    print_point(p2);

    return 0;
}
```
**Code Breakdown:**
- Compound literals create unnamed objects of a given type.
- They are useful for passing temporary values to functions.
- They can be used with designated initialisers (C99) to initialise specific members.
- The lifetime depends on scope: if at block scope, it lasts until the block exits.

---

## 10. Common Use Cases

- **Initialisation** – setting variables to starting values.
- **Magic numbers** – when you need a constant value and you name it (though better to use named constants).
- **Printf format strings** – passing a literal format string.
- **Case labels** – using integer literals in `switch` statements.
- **Array sizes** – using integer literals for array dimensions.
- **Enumeration values** – specifying custom values.
- **String messages** – output messages, error strings, prompts.

---

## 11. Best Practices

- **Use meaningful constants over raw literals** – named constants improve readability, except for obvious values like `0` or `1` in common contexts.
- **Use suffixes to avoid overflow** – when a literal exceeds `int` range, use `L`, `LL`, or `U`.
- **Prefer decimal for normal integer values** – use hex for bit masks, octal for permissions (Unix file modes), binary for flags (C23).
- **Use `f` suffix for float constants** – otherwise they are `double` and may cause conversion warnings.
- **Use `const` for string literals when using pointers** – `const char *s = "hello";` avoids accidental modification.
- **Escape sequences for clarity** – use `\n`, `\t` instead of raw control characters.
- **Use adjacent string literals for long strings** – improves readability and formatting.
- **Use compound literals sparingly** – they are powerful but can obscure code if overused.
- **Be mindful of platform‑dependent sizes** – `wchar_t` size varies; use fixed‑width types when possible.

---

## 12. Common Mistakes

### Mistake 1: Using `char` to Store a Wide Character
```c
// ❌ Wrong – char cannot hold multibyte characters
char c = 'Ω';
// ✅ Correct – use wchar_t for wide characters
wchar_t wc = L'Ω';
```

### Mistake 2: Modifying a String Literal
```c
// ❌ Wrong – undefined behaviour (string literal is read-only)
char *s = "hello";
s[0] = 'H';   // may crash
// ✅ Correct – use an array if you need to modify
char s[] = "hello";
s[0] = 'H';   // OK
```

### Mistake 3: Integer Literal Too Large Without Suffix
```c
// ❌ Wrong – overflow if int is 32-bit
int x = 3000000000;   // may overflow (since > 2^31-1)
// ✅ Correct – use long long or unsigned
long long x = 3000000000LL;
unsigned int y = 3000000000u; // if unsigned int can hold it
```

### Mistake 4: Mixing Integer and Float Literals in Comparisons
```c
// ❌ Subtle – integer division when expecting float
float ratio = 1 / 2;   // result is 0.0 (integer division)
// ✅ Correct – use floating literal
float ratio = 1.0f / 2.0f;
```

### Mistake 5: Forgetting the Null Terminator in String Literals
```c
// ❌ Wrong – missing null terminator (string literal always has it)
char s[3] = "abc";   // array size 3, but "abc" needs 4 bytes (with \0)
// ✅ Correct – let the compiler size it
char s[] = "abc";    // size 4
```

---

## 13. Performance Considerations

- **Literals are compile‑time constants** – no runtime overhead for their creation.
- **String literals** – stored once in `.rodata`; repeated use of the same literal may be merged by the compiler, saving memory.
- **Integer literals** – often embedded as immediate operands, fastest possible access.
- **Floating literals** – may be loaded from memory (if not embedded in instruction), but still fast.
- **Compound literals** – create an object with storage; may involve copying if used in assignments.

---

## 14. Security Considerations

- **String literals** are read‑only; attempting to write them can lead to crashes (segmentation fault) – this is a security feature (prevents buffer overflows from modifying the string pool).
- **Hexadecimal and octal literals** can be used to represent arbitrary memory addresses; use with caution.
- **Integer overflow** – be careful with literals that exceed the range of the intended type; use suffixes or check before use.
- **Avoid exposing sensitive data in literals** – they are embedded in the binary and can be extracted (strings are easily visible).

---

## 15. Debugging Tips

- **Use `-Wall -Wextra`** to catch warnings about integer overflow, sign conversion, and mismatched types.
- **Inspect object files** with `objdump -s -j .rodata` to see string literals.
- **Use `gdb`** to examine values of literals (they are immediate or in memory).
- **Compile with `-save-temps`** to see preprocessed output – you can check how literals are interpreted.

---

## 16. When to Use

- **Everywhere** – literals are unavoidable.
- **Use named constants for non‑obvious values** – but the literal itself is still written in the definition.
- **Use literals directly for common values** – `0`, `1`, `-1`, `'\0'`, `NULL` are often better as literals.

---

## 17. When Not to Use

- **Avoid raw literals for values that may change** – use named constants for maintainability.
- **Avoid literals that are hard to understand** – `0xDEADBEEF` might be better as a macro `#define MAGIC 0xDEADBEEF`.

---

## 18. Related Concepts

- **Constants** – named values (`const`, `#define`, `enum`).
- **Data types** – the type of a literal determines how it is stored and interpreted.
- **Escape sequences** – special characters in literals.
- **String handling** – functions like `printf`, `strcpy` work with string literals.
- **Compound literals** – C99 feature for unnamed objects.
- **Character encoding** – ASCII, Unicode.

---

## 19. Did You Know?

- In C, a character literal like `'A'` actually has type `int`, not `char`. That’s why you can write `int x = 'A';`.
- Adjacent string literals are concatenated at compile time, even if they have different prefixes (e.g., `"Hello " L"World"` – but the result is a wide string).
- The C standard does not specify the character set; it could be ASCII, EBCDIC, or others, but ASCII is ubiquitous.
- String literals are not modifiable, but you can create a modifiable copy using arrays.
- The `u8` prefix for UTF‑8 string literals was introduced in C11; before that, you had to rely on the compiler’s support.
- Octal literals (leading zero) can be confusing; many style guides discourage using them.

---

## 20. Summary

- **Literals are fixed values directly written in source code** – numbers, characters, strings, and compound objects.
- **They come in several types**: integer, floating, character, string, and compound.
- **Integer literals** can be decimal, octal, hex, binary (C23), with suffixes for type control.
- **Floating literals** default to `double`; use `f` or `L` suffixes for `float` or `long double`.
- **Character literals** use single quotes; escape sequences allow special characters and arbitrary byte values.
- **String literals** are double‑quoted, read‑only arrays of characters, with wide and UTF variants.
- **Compound literals** (C99) create unnamed objects of any type, useful for temporary values.
- **Best practices**: use named constants for non‑obvious values, avoid modifying string literals, use suffixes to control type, and understand the storage of literals.
- **Literals are the foundation** of all data initialisation and constant expressions in C.