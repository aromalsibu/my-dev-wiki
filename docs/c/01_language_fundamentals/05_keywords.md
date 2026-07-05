# Keywords

## 1. Overview

### Definition
Keywords are reserved words in the C language that have a fixed, predefined meaning to the compiler. They are the fundamental building blocks of the language's syntax and cannot be used for any other purpose (like variable names, function names, or user‑defined types). Each keyword triggers a specific behaviour or declares a specific language construct.

### Purpose
Keywords form the vocabulary of C. They enable the compiler to recognise declarations, control flow, data types, storage classes, and other core language features. By reserving these words, the language ensures that there is no ambiguity between user‑defined identifiers and built‑in constructs.

### Where It Fits
Keywords are a subset of the token stream. When the lexer encounters a sequence of characters that matches a keyword, it classifies it as a keyword token, not an identifier. The parser then uses these tokens to apply the grammar rules of C. They are the syntactic anchors around which the rest of the program is built.

---

## 2. Why It Exists

### The Problem Without Keywords
If C did not reserve specific words, the compiler would have no way to distinguish, for instance, a variable named `if` from the conditional statement `if`. This would make the language ambiguous and essentially impossible to parse without complex contextual rules.

### The Solution: A Fixed Lexicon
By reserving a set of words, the language gains clarity. Every C programmer knows that `int` declares an integer type, `while` starts a loop, and `return` exits a function. The compiler can reliably recognise these constructs at tokenisation time, without needing to understand the surrounding context.

### Why It Was Introduced
This approach is common in most programming languages. For C, the list of keywords was kept intentionally small – just 32 in the original K&R C – to maintain simplicity. Over time, new standards added keywords to support new features while preserving backward compatibility. The language’s design philosophy of “trust the programmer” also meant that keywords are few and direct, reducing the cognitive load.

---

## 3. Syntax / Basic Usage

Keywords are used directly in the source code. Here is a small program that uses many of them.

```c
#include <stdio.h>   // Not a keyword – this is a preprocessor directive

int main(void) {     // int, void – type keywords; main is an identifier (not a keyword)
    int count = 0;   // int – type keyword
    for (int i = 0; i < 5; i++) {  // for, int – keywords; i is an identifier
        if (i % 2 == 0) {          // if – conditional keyword
            count += i;            // count is an identifier; += is an operator
        }
    }
    return count;    // return – keyword; count is an identifier
}
```

### Complete List of Standard C Keywords

#### C89/C90 Keywords (32)
```
auto    break   case    char    const   continue    default do
double  else    enum    extern  float   for         goto    if
int     long    register return  short   signed      sizeof  static
struct  switch  typedef union   unsigned void    volatile while
```

#### C99 Added (5)
```
inline  restrict   _Bool   _Complex   _Imaginary
```
(Note: `_Bool` is often used via `stdbool.h` as `bool`; `_Complex` via `complex.h` as `complex`.)

#### C11 Added (7)
```
_Alignas    _Alignof    _Atomic     _Generic    _Noreturn
_Static_assert  _Thread_local
```

#### C23 Added (10+)
```
alignas    alignof    bool    false    static_assert   thread_local   true
constexpr  nullptr    typeof   typeof_unqual   (some are new keywords, some are
                                              aliases introduced via headers)
```
(Note: C23 also introduces `bitint`, `_BitInt` type, but not a keyword per se.)

### Code Breakdown (with comments)
```c
#include <stdio.h>   
// #include is a preprocessor directive, not a C keyword.

int main(void) {
    // int – keyword for integer type.
    // main – identifier (function name).
    // void – keyword for “no type” – here it means no parameters.

    int count = 0;
    // int – keyword: declares count as an integer.

    for (int i = 0; i < 5; i++) {
        // for – keyword: starts a loop.
        // int – keyword: declares i within the for scope (C99+).
        // i++ – ++ is an operator, not a keyword.

        if (i % 2 == 0) {
            // if – keyword: conditional branch.
            // % – operator, == – operator.

            count += i;
            // += – operator.
        }
        // } – punctuator, not a keyword.
    }

    return count;
    // return – keyword: exits the function and returns a value.
}
```

---

## 4. Mental Model – Keywords as the Foundation of a House

Think of a C program as a house. The keywords are the **load‑bearing walls** and **fixtures** that define the structure:

- **Data type keywords** (`int`, `float`, `char`, `struct`) – like the concrete types of rooms (bedroom, kitchen, bathroom).
- **Storage class keywords** (`static`, `extern`, `register`) – like whether a room is private to the house (static) or shared across a complex (extern).
- **Control flow keywords** (`if`, `else`, `for`, `while`, `switch`) – like doors, hallways, and staircases that direct the flow of movement.
- **Function‑related keywords** (`return`, `void`) – like exits and entry points.
- **Type qualifiers** (`const`, `volatile`) – like “do not alter” signs or “may change unexpectedly” warnings.
- **Miscellaneous keywords** (`sizeof`, `typedef`) – like tape measures and renaming tools.

Every house is built using the same set of standard parts; you cannot use a wall to name a door – that would break the design. Similarly, you cannot use `if` as a variable name because it would confuse the builder (compiler).

---

## 5. Core Concepts

| Keyword | Category | Purpose | Example |
|---------|----------|---------|---------|
| `int`, `char`, `float`, `double` | Data types | Declare variables of basic types. | `int x;` |
| `short`, `long`, `unsigned`, `signed` | Type modifiers | Modify range/size of basic types. | `unsigned long int y;` |
| `_Bool`, `_Complex`, `_Imaginary` | Special types | Boolean, complex numbers (C99+). | `_Bool flag = 1;` |
| `struct`, `union`, `enum` | Derived types | Group data; choice of memory layout. | `struct Point {int x; int y;};` |
| `typedef` | Type alias | Create new name for an existing type. | `typedef int age_t;` |
| `void` | Special type | No value (functions, generic pointers). | `void *ptr;` |
| `const`, `volatile` | Qualifiers | Immutable or hardware‑modified data. | `const int MAX = 100;` |
| `auto`, `register`, `static`, `extern`, `_Thread_local` | Storage classes | Control lifetime, linkage, storage. | `static int counter;` |
| `if`, `else`, `switch`, `case`, `default` | Conditionals | Branching. | `if (x > 0) ...` |
| `for`, `while`, `do` | Loops | Iteration. | `for (;;) { ... }` |
| `break`, `continue` | Loop control | Exit iteration or skip rest. | `break;` |
| `return` | Function return | Exit function with/without value. | `return 0;` |
| `goto` | Unconditional jump | Jump to label (discouraged). | `goto error;` |
| `sizeof` | Operator (looks like keyword) | Get size in bytes. | `sizeof(int)` |
| `inline` | Function specifier | Suggest inlining (C99+). | `inline int add(int a, int b) { return a+b; }` |
| `_Alignas`, `_Alignof` | Alignment | Control memory alignment (C11+). | `_Alignas(16) int x;` |
| `_Atomic` | Atomic | Atomic operations (C11+). | `_Atomic int counter;` |
| `_Generic` | Generic selection | Type‑generic macros (C11+). | `_Generic(x, int: ...)` |
| `_Noreturn` | Function specifier | Function does not return (C11+). | `_Noreturn void panic(void);` |
| `_Static_assert` | Compile‑time assertion | Check condition at compile time (C11+). | `_Static_assert(sizeof(int) == 4, "int size");` |
| `restrict` | Pointer qualifier | Pointer aliasing hint (C99+). | `int * restrict p;` |
| `constexpr` | Constant expression | Function that can be evaluated at compile time (C23). | `constexpr int square(int x) { return x*x; }` |
| `typeof`, `typeof_unqual` | Type operator | Obtain type of expression (C23). | `typeof(5) y;` |

---

## 6. How It Works – Lexer and Parser Integration

1. **Lexer** scans the source and reads a sequence of characters.
2. **It checks if the sequence matches a keyword** – typically via a trie or hash table.
   - If it does, a **keyword token** is emitted (with a specific token ID).
   - If not, it is treated as an **identifier**.
3. **Parser** consumes keyword tokens according to the grammar:
   - `int` triggers a declaration rule.
   - `if` triggers a conditional rule, expecting a parenthesised expression and a statement.
   - `return` expects an optional expression and ends the function.
4. **Semantic analysis** checks keyword usage: e.g., `break` must be inside a loop or switch.

### ASCII Diagram – Keyword Recognition
```
Source: "int x;"
   │
   ▼
Lexer reads 'i','n','t' – matches keyword token "int"
   │
   ▼
Lexer continues with space, then 'x' – matches identifier
   │
   ▼
Parser sees "int" keyword → expects a declarator (identifier) and semicolon.
```

---

## 7. Internal Architecture – Keyword Handling in a Compiler

- **Keyword table** – often a perfect hash table or a trie for O(1) or O(m) lookup (m = keyword length).
- **Token IDs** – each keyword is assigned a unique integer constant (e.g., `TK_INT`).
- **Preprocessor distinction** – keywords are not recognised inside preprocessor directives; only after macro expansion.
- **New keywords vs. reserved identifiers** – C standards introduce new keywords as **reserved identifiers** starting with an underscore and a capital letter (e.g., `_Bool`) to avoid breaking existing code. Then headers like `stdbool.h` map them to familiar names (e.g., `bool`).
- **Context‑sensitive keywords** – some newer features (e.g., `typeof` in C23) are technically keywords in C23, but in earlier standards they could be used as identifiers. Compilers must handle version‑specific sets.

---

## 8. Lifecycle / Workflow of a Keyword

1. **Written** by the programmer in source code.
2. **Tokenised** by the lexer – classified as a keyword token.
3. **Parsed** by the parser – triggers a grammar rule.
4. **Semantically checked** – e.g., `return` must be inside a function; `break` must be inside a loop/switch.
5. **Code generation** – the keyword influences the generated machine code (e.g., `for` generates conditional jumps; `static` influences symbol visibility).
6. **Final binary** – the keyword itself does not appear; its effects are compiled away.

---

## 9. Practical Examples

### Example 1: `const` and `volatile`
```c
#include <stdio.h>

// const – the value cannot be modified after initialisation
// volatile – the value may change outside the program (e.g., memory‑mapped I/O)
void read_sensor(const volatile int *sensor_reg) {
    // sensor_reg points to a hardware register.
    // const: we won't modify it; volatile: compiler must re‑read each time.
    int value = *sensor_reg;   // read
    int next = *sensor_reg;    // read again (volatile prevents optimisation)
    printf("Value: %d, Next: %d\n", value, next);
}

int main(void) {
    // Example hardware address (for illustration)
    volatile int *reg = (volatile int *)0x4000;  // cast to pointer
    read_sensor(reg);
    return 0;
}
```
**Code Breakdown:**
- `const volatile int *sensor_reg` – pointer to integer that is both const (read‑only) and volatile (may change unexpectedly).
- `*sensor_reg` – each read generates a memory access because of `volatile`; the compiler cannot cache the value.
- This pattern is common in embedded systems for hardware registers.

### Example 2: `static` and `extern`
```c
// file1.c
#include <stdio.h>

static int file_scope_var = 10;  // visible only in this file
extern int global_var;           // declared elsewhere (in file2.c)

void use_static(void) {
    printf("file_scope_var = %d\n", file_scope_var);
}

void use_global(void) {
    printf("global_var = %d\n", global_var);
}

// file2.c
int global_var = 42;  // definition

int main(void) {
    use_static();
    use_global();
    return 0;
}
```
**Code Breakdown:**
- `static` – limits `file_scope_var` to file1.c; prevents name collisions.
- `extern` – declares `global_var` without defining it; the linker resolves it with the definition in file2.c.

### Example 3: `restrict` (C99)
```c
#include <stdio.h>

// restrict tells the compiler that pointers do not alias
int add_arrays(int *restrict a, int *restrict b, int n) {
    for (int i = 0; i < n; i++) {
        a[i] += b[i];
    }
    return 0;
}

int main(void) {
    int x[5] = {1,2,3,4,5};
    int y[5] = {10,20,30,40,50};
    add_arrays(x, y, 5);
    // x now contains {11,22,33,44,55}
    return 0;
}
```
**Code Breakdown:**
- `restrict` – a promise that `a` and `b` do not point to overlapping memory.
- The compiler can optimise loops more aggressively (e.g., vectorisation) because it knows there's no aliasing.

### Example 4: `_Static_assert` (C11)
```c
#include <stdio.h>
#include <stddef.h>  // for offsetof

struct MyStruct {
    int a;
    char b;
    double c;
};

// Compile‑time check: ensure alignment is as expected.
_Static_assert(offsetof(struct MyStruct, b) == sizeof(int),
               "b should be immediately after a (no padding)");

int main(void) {
    printf("Size of struct: %zu\n", sizeof(struct MyStruct));
    return 0;
}
```
**Code Breakdown:**
- `_Static_assert` – if the condition is false, compilation fails with the given message.
- This helps enforce invariants at compile time without runtime overhead.

### Example 5: `_Generic` (C11) – Type‑generic Macro
```c
#include <stdio.h>
#include <math.h>

// Macro that prints the absolute value using the appropriate function.
#define ABS(x) _Generic((x), \
    int: abs, \
    float: fabsf, \
    double: fabs, \
    default: fabs \
)(x)

int main(void) {
    int i = -5;
    float f = -3.14f;
    double d = -2.718;

    printf("ABS(%d) = %d\n", i, ABS(i));        // uses abs
    printf("ABS(%f) = %f\n", f, ABS(f));        // uses fabsf
    printf("ABS(%f) = %f\n", d, ABS(d));        // uses fabs
    return 0;
}
```
**Code Breakdown:**
- `_Generic` – a keyword that selects an expression based on the type of the controlling expression.
- It is evaluated at compile time, enabling type‑safe generic macros.
- `ABS(x)` expands to a call to the appropriate absolute‑value function.

---

## 10. Common Use Cases

- **Data types** – every variable declaration uses type keywords.
- **Control flow** – loops and conditionals are ubiquitous.
- **Storage control** – `static` for file‑private variables, `extern` for sharing.
- **Optimisation hints** – `restrict`, `inline` to guide the compiler.
- **Safety and correctness** – `const` for read‑only data, `_Static_assert` for compile‑time checks.
- **Portability** – `stdint.h` uses `typedef` to create platform‑independent types.

---

## 11. Best Practices

- **Do not use keywords as identifiers** – the compiler will reject it. Choose descriptive names instead.
- **Prefer the standard header aliases** for new keywords (e.g., `bool` from `stdbool.h` instead of `_Bool`, `alignas` from `stdalign.h` instead of `_Alignas`).
- **Use `static` for file‑private functions and variables** – this reduces global namespace pollution and improves encapsulation.
- **Use `const` wherever possible** – makes intent clear and enables compiler optimisations.
- **Use `restrict` only when you are certain pointers do not alias** – misusing it can lead to undefined behaviour.
- **Leverage `_Generic` for type‑generic programming** – reduces code duplication.
- **Avoid `goto`** – use structured control flow; it’s considered harmful in modern practice.

---

## 12. Common Mistakes

### Mistake 1: Using a Keyword as an Identifier
```c
// ❌ Wrong
int return = 5;   // return is a keyword
// ✅ Correct
int result = 5;
```

### Mistake 2: Forgetting to Include Headers for Alias Keywords
```c
// ❌ Wrong – _Bool works, but using bool without stdbool.h gives an error.
bool flag = true;   // bool and true are not defined unless you include stdbool.h
// ✅ Correct
#include <stdbool.h>
bool flag = true;
```

### Mistake 3: Using `restrict` Incorrectly
```c
// ❌ Wrong – a and b may alias in some calls, leading to undefined behaviour.
void add(int *restrict a, int *restrict b, int n) {
    for (int i = 0; i < n; i++) a[i] += b[i];
}
int main() {
    int x[10];
    add(x, x, 10);  // a and b point to the same array – violates restrict promise
}
// ✅ Correct – only use restrict when you guarantee no aliasing.
```

### Mistake 4: Misusing `static` Inside a Function
```c
// ❌ Wrong – static local variable retains value across calls; may be unintended.
int counter(void) {
    static int count = 0;  // initialised once
    return count++;
}
// If you wanted a simple counter, this is fine; but beware of thread safety.
```

### Mistake 5: Forgetting `return` in a Non‑void Function
```c
// ❌ Wrong – undefined behaviour if the caller uses the return value.
int add(int a, int b) {
    // missing return
}
// ✅ Correct
int add(int a, int b) {
    return a + b;
}
```

---

## 13. Performance Considerations

- **`inline`** – suggests the compiler to replace a function call with its body, potentially improving performance for small functions (reduces call overhead). But it is only a suggestion; the compiler may ignore it.
- **`register`** – historically suggested storage in a CPU register; now mostly ignored by modern compilers, but still allowed (deprecated in C++).
- **`restrict`** – can enable significant optimisations (loop unrolling, vectorisation) by telling the compiler that pointers do not alias.
- **`const` and `volatile`** – affect optimisation; `const` allows more aggressive constant propagation; `volatile` forces memory reads, preventing optimisations that would eliminate them.

---

## 14. Security Considerations

- **`const`** – helps protect against accidental modification of read‑only data (e.g., string literals).
- **`volatile`** – crucial for memory‑mapped I/O and signal handlers; without it, the compiler might optimise away critical reads.
- **`static`** – reduces attack surface by hiding internal functions and variables from external linkage.
- **`_Atomic`** – ensures thread‑safe operations; use for shared variables in multi‑threaded code.
- **`_Noreturn`** – indicates functions that do not return (e.g., `abort`); helps the compiler optimise and avoid warnings.

---

## 15. Debugging Tips

- **Keyword‑related errors** – often appear as “expected identifier” or “syntax error” when a keyword is misused.
- **Use `-std=c11` or `-std=c23`** to enable new keywords; otherwise, they may be treated as identifiers.
- **Check headers** – if a keyword alias like `bool` is not recognised, ensure you have included the appropriate header (`stdbool.h`).
- **Preprocessor output** – use `-E` to see expanded code; sometimes macro expansions introduce keywords incorrectly.

---

## 16. When to Use

- **All the time** – keywords are mandatory for writing valid C code.
- **Use the appropriate standard** – choose the language version that provides the keywords you need (e.g., C11 for `_Atomic`, C23 for `constexpr`).

---

## 17. When Not to Use

- **You cannot avoid them** – they are part of the language.
- **Avoid over‑using obscure keywords** – stick to common ones (`int`, `if`, etc.) unless you have a specific need.

---

## 18. Related Concepts

- **Tokens** – keywords are a specific token type.
- **Identifiers** – user‑defined names that contrast with keywords.
- **Data types** – type keywords define the basic building blocks.
- **Storage classes** – keywords like `static`, `extern`.
- **Control flow** – keywords that direct program execution.
- **Preprocessor** – directives are not keywords; they are handled before tokenisation.

---

## 19. Did You Know?

- The original K&R C had only 32 keywords. The latest C23 has over 50 if you include new ones.
- Some keywords are “soft” – they can be used as identifiers in earlier standards but become reserved in newer ones (e.g., `inline` was not a keyword in C89 but is in C99).
- The `_` prefix for many new keywords (e.g., `_Bool`, `_Atomic`) is a convention to avoid breaking existing code that might use those names.
- In C, `sizeof` is an operator, not a function, even though it looks like one. It does not require parentheses when used with a type (`sizeof int` is invalid; `sizeof(int)` is required for types, but for expressions you can write `sizeof x` without parentheses).
- `goto` is rarely used but still in the language; in fact, it is often used in error‑handling patterns in the Linux kernel to jump to cleanup code.

---

## 20. Summary

- **Keywords are reserved words** with fixed meanings in C; they cannot be used as identifiers.
- **They belong to several categories**: data types, type modifiers, storage classes, control flow, qualifiers, function specifiers, and more.
- **The list has grown over time** – from 32 in C89 to over 50 in C23, with new keywords introduced in each standard.
- **New keywords often start with an underscore** and a capital letter to avoid breaking existing code; headers like `stdbool.h` provide user‑friendly aliases.
- **Using keywords correctly** is essential for writing valid C code; misusing them leads to syntax errors.
- **Best practices** include using `static` for encapsulation, `const` for read‑only data, and leveraging new keywords for type safety and optimisation.
- **Understanding keywords** is fundamental to reading and writing C – they are the vocabulary of the language.