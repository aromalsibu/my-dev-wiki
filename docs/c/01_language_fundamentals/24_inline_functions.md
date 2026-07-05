# Inline Functions

## 1. Overview

### Definition
An inline function is a function that is expanded directly at the point of call, rather than being called through the standard function call mechanism. When you declare a function as `inline`, you're suggesting to the compiler that it should replace the function call with the actual body of the function, avoiding the overhead of a function call. The `inline` keyword was introduced in C99 and is a hint, not a command—the compiler is free to ignore it.

### Purpose
Inline functions serve several important purposes:
- **Performance improvement** – eliminate function call overhead for small, frequently called functions.
- **Optimisation opportunities** – enable the compiler to perform optimisations across the function boundary.
- **Type safety** – unlike macros, inline functions are type‑checked and don't have the pitfalls of macro expansion.
- **Readability** – allow you to write small, single‑purpose functions without worrying about performance penalties.
- **Header‑friendly** – inline functions are often defined in header files, enabling the compiler to inline them across translation units.

### Where It Fits
Inline functions are part of the function mechanism but have special compilation semantics. They fit between:
- **Macros** – textual substitution (no type checking, prone to side‑effects).
- **Regular functions** – full function call with stack frame overhead.
- **Static inline functions** – a common idiom for header‑only implementations.

---

## 2. Why It Exists

### The Problem Before Inline Functions
Before C99, if you wanted to avoid function call overhead in C, your only option was to use macros:
```c
#define SQUARE(x) ((x) * (x))
```
While macros avoid call overhead, they have serious drawbacks:
- **No type checking** – `SQUARE("hello")` compiles (or fails mysteriously).
- **Side effects** – `SQUARE(x++)` expands to `((x++) * (x++))`, causing undefined behaviour.
- **Scope issues** – macros don't respect scope or block structure.
- **Debugging difficulties** – debugging macro expansions is hard.
- **Complexity** – multi‑statement macros require `do { ... } while (0)` tricks.

Regular functions provide type safety and scope, but they always incur call overhead, even for trivial operations like getting a value or performing a simple calculation.

### The Solution: Inline Functions
Inline functions give you the best of both worlds:
- **Type safety** – parameters and return values are type‑checked.
- **Call‑overhead avoidance** – if the compiler honours the suggestion, the function is expanded in place.
- **No side‑effect issues** – arguments are evaluated once, like regular functions.
- **Debugging support** – inline functions can be debugged (with `-g`).
- **Scope and visibility** – inline functions respect scope and can be `static` or `extern`.

### Why They Were Introduced
C99 introduced `inline` to address the limitations of macros for performance‑critical code and to bring C closer to C++ in terms of optimisation capabilities. The need for inline functions came from systems programming, where performance matters, but developers wanted the safety of functions over macros.

---

## 3. Syntax / Basic Usage

### Simple Inline Function
```c
#include <stdio.h>

// Inline function definition.
// The 'inline' keyword suggests to the compiler to expand it at call sites.
inline int square(int x) {
    return x * x;
}

int main(void) {
    int a = 5;
    // The call to square() may be expanded to a * a at compile time.
    int result = square(a);
    printf("5^2 = %d\n", result);   // 25

    // Even with a constant argument, the compiler can evaluate it at compile time.
    int constant_result = square(10);
    printf("10^2 = %d\n", constant_result); // 100

    return 0;
}
```

### Static Inline Function (Header‑only Idiom)
```c
// utils.h – header file
#ifndef UTILS_H
#define UTILS_H

// static inline is a common idiom for header‑only functions.
// 'static' ensures each translation unit gets its own copy; it doesn't conflict.
// 'inline' suggests inlining, but the compiler may not inline.
static inline int max(int a, int b) {
    return a > b ? a : b;
}

static inline int min(int a, int b) {
    return a < b ? a : b;
}

#endif
```

```c
// main.c
#include <stdio.h>
#include "utils.h"   // includes the static inline functions.

int main(void) {
    int x = 10, y = 20;
    printf("Max: %d\n", max(x, y));   // 20
    printf("Min: %d\n", min(x, y));   // 10
    return 0;
}
```

### Inline Functions and `extern`
```c
// file1.c
#include <stdio.h>

// Inline definition (with external linkage).
// If the compiler doesn't inline, it generates an external definition.
inline int add(int a, int b) {
    return a + b;
}

// Another inline function that calls add().
inline int add_and_double(int a, int b) {
    return add(a, b) * 2;
}

int main(void) {
    printf("add: %d\n", add(5, 3));           // 8
    printf("add_and_double: %d\n", add_and_double(5, 3)); // 16
    return 0;
}
```

### Inline Function with Static and Extern (C99 Semantics)
```c
// file1.c
#include <stdio.h>

// Inline definition with no 'static' or 'extern' – provides an inline definition.
inline int inc(int x) {
    return x + 1;
}

// In another file, you could provide an extern definition if needed.
// file2.c
extern int inc(int x);   // declaration (may use the definition from file1.c or provide its own)

// In file1.c, if the compiler needs a non‑inline version (e.g., for function pointers),
// it will generate an external definition as well, unless you use 'static inline'.
```

### Code Breakdown (with comments)
```c
#include <stdio.h>

// Inline function – a suggestion to the compiler to expand the function body
// at each call site, avoiding the overhead of a function call.
inline int square(int x) {
    // The body is simple – a good candidate for inlining.
    return x * x;
}

// static inline – often used in header files.
// 'static' gives internal linkage (each translation unit gets its own copy).
// 'inline' suggests expansion.
static inline int cube(int x) {
    return x * x * x;
}

int main(void) {
    // Call to inline function – may be expanded to: (a * a)
    int a = 5;
    int result = square(a);   // If inlined, no function call overhead.

    // Calling static inline – also may be expanded.
    int cubed = cube(a);

    printf("square: %d, cube: %d\n", result, cubed);

    // Taking the address of an inline function forces the compiler
    // to generate a non‑inline version (if it hasn't already).
    int (*func_ptr)(int) = square;   // Address of square
    int via_ptr = func_ptr(10);      // Calls through the pointer – cannot be inlined.
    printf("via pointer: %d\n", via_ptr);

    return 0;
}
```

---

## 4. Mental Model – Inline Functions as the "Copy‑and‑Paste" Suggestion

Imagine you have a chef (the compiler) who follows a recipe (your source code). A function call is like telling the chef: "Go to the other room, look at the recipe for `square`, follow it, and bring me the result." This takes time (overhead).

**Inline functions** are like telling the chef: "Just write the instructions for `square` here, where you are." The chef copies the recipe steps directly into the current location. This is faster because the chef doesn't have to go to another room.

**Macros** are like telling the chef to do a crude text replacement—"wherever you see `SQUARE(x)`, write `((x) * (x))`." This can lead to mistakes if `x` has side effects.

**Regular functions** are like a professional chef following a recipe—always correct, but with the overhead of going to the other room.

**Inline functions** give you the correctness of the professional chef (type checking, scope), but the speed of staying in the same room (if the compiler chooses to copy the steps).

---

## 5. Core Concepts

### Inline Keyword Semantics (C99+)
| Declaration | Linkage | Effect |
|-------------|---------|--------|
| `inline int f(void) { ... }` | External | Provides an inline definition. The compiler may inline calls, but an external definition is also generated if needed (e.g., function pointers, non‑inlined calls). |
| `static inline int f(void) { ... }` | Internal (per translation unit) | Provides an inline definition with internal linkage. Each translation unit has its own copy. This is the safest and most common usage, especially in headers. |
| `extern inline int f(void) { ... }` | External | Provides an external definition. This is rarely used; it's mainly for providing a non‑inline definition in a separate file. |
| `inline` without `static` or `extern` | External | Provides an inline definition, but an external definition may also be emitted. This can lead to multiple definitions if not handled carefully. |

### How Inlining Works
1. **Compiler suggestion** – `inline` is a hint, not a command. The compiler decides whether to inline based on:
   - **Function size** – small functions are more likely to be inlined.
   - **Optimisation level** – higher optimisation (`-O2`, `-O3`) increases inlining.
   - **Function complexity** – functions with loops, recursion, or many calls are less likely to be inlined.
   - **Compiler heuristics** – each compiler has its own rules.

2. **Translation unit** – for a function to be inlined across translation units, its definition must be visible in the header (usually `static inline`).

3. **Function pointers** – if you take the address of an inline function, the compiler must generate a non‑inline version, so the function cannot be inlined at that call site.

### When Inlining Happens
- **Compile‑time** – the compiler expands the function at the call site.
- **Link‑time** – some compilers support link‑time optimisation (LTO), which can inline functions across translation units even if they weren't marked `inline`.

### Advantages Over Macros
- **Type checking** – parameters and return types are validated.
- **Side‑effect safety** – arguments are evaluated exactly once, like regular functions.
- **Debugging** – you can set breakpoints inside inline functions (if not fully inlined).
- **Syntax** – they look like regular functions, not a substitution language.

---

## 6. How It Works – Compiler Inlining Mechanics

### Step‑by‑Step
1. **Parser** – the compiler sees the `inline` keyword and records it in the function's symbol.
2. **Optimisation pass** – when the compiler sees a call to an inline function, it evaluates whether to inline it:
   - The function body is substituted at the call site.
   - Arguments are evaluated (once).
   - Local variables are expanded into the caller's scope.
   - The return statement is replaced with an assignment to the return value.
3. **If not inlined** – the function is compiled as a regular function, with a call instruction.
4. **External definition** – if an external definition is needed (e.g., function pointers, or `inline` without `static`), the compiler emits a separate function body.

### Example – Inlining Expansion
```c
// Source:
inline int square(int x) {
    return x * x;
}

int main() {
    int a = 5;
    int b = square(a);
}

// After inlining (conceptual):
int main() {
    int a = 5;
    int b = a * a;   // square() body expanded here.
}
```

### Conditions for Inlining
The compiler may inline if:
- The function body is small enough.
- The function is not recursive.
- The function is not accessed via a function pointer at the call site.
- There are no variable‑length arrays or other complex constructs.

---

## 7. Internal Architecture – Symbol Handling

### With `static inline`
- The function has **internal linkage**.
- Each translation unit that includes the header gets its own copy.
- No external symbol is exported.
- This is the safest for headers.

### With `inline` (non‑static)
- The function has **external linkage**.
- The compiler may generate an external definition if:
  - The function is not inlined at a call site.
  - The function's address is taken.
  - The compiler decides it needs a separate definition.
- This can cause **multiple definition errors** if the same non‑`static` `inline` function is defined in multiple translation units (but C99 has special rules to avoid this if you follow the pattern).

### C99 Inline Semantics (Advanced)
In C99, the semantics of `inline` are a bit complex:
- If you write `inline int f(void) { ... }`, this is an **inline definition**.
- It doesn't provide an external definition unless needed.
- If no external definition is provided elsewhere, and the compiler needs one, it may generate one (with a "weak" symbol).
- The common idiom for header files is `static inline`, which avoids all complexity.

---

## 8. Lifecycle / Workflow of an Inline Function

1. **Written** – programmer writes `inline int max(int a, int b) { ... }`.
2. **Compiled** – the compiler processes the function:
   - If called, it may inline the body.
   - If not inlined, it emits a function definition.
   - If `static`, it's local to the translation unit.
3. **Optimised** – the inlined code is optimised further in the context of the caller.
4. **Executed** – the inlined code runs as part of the caller's code.
5. **Program termination** – no function call was made, so no stack frame was created for the function.

---

## 9. Practical Examples

### Example 1: Inline Function for Performance
```c
#include <stdio.h>
#include <time.h>

// Inline function for a simple operation.
static inline int max_int(int a, int b) {
    return a > b ? a : b;
}

// Regular function for comparison.
int max_int_regular(int a, int b) {
    return a > b ? a : b;
}

int main(void) {
    int x = 10, y = 20;
    int result1 = max_int(x, y);          // Likely inlined
    int result2 = max_int_regular(x, y);  // Function call

    printf("Inline max: %d\n", result1);
    printf("Regular max: %d\n", result2);

    // In a tight loop, the inline version can be significantly faster.
    clock_t start = clock();
    for (int i = 0; i < 100000000; i++) {
        volatile int r = max_int(i, i + 1);   // volatile to prevent optimisation
    }
    clock_t end = clock();
    printf("Time with inline: %f seconds\n", (double)(end - start) / CLOCKS_PER_SEC);

    start = clock();
    for (int i = 0; i < 100000000; i++) {
        volatile int r = max_int_regular(i, i + 1);
    }
    end = clock();
    printf("Time with regular: %f seconds\n", (double)(end - start) / CLOCKS_PER_SEC);

    return 0;
}
```
**Code Breakdown:**
- `max_int` is `static inline` – likely to be expanded inline.
- The benchmark compares performance; the inline version should be faster (less call overhead).
- `volatile` prevents the compiler from optimising away the loop.

### Example 2: Inline Functions in Headers (Common Pattern)
```c
// math_utils.h
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

// static inline is the safest way to define functions in headers.
static inline int add(int a, int b) {
    return a + b;
}

static inline int subtract(int a, int b) {
    return a - b;
}

static inline int multiply(int a, int b) {
    return a * b;
}

// A more complex inline function – may be inlined or not, depending on the compiler.
static inline int clamp(int value, int min, int max) {
    if (value < min) return min;
    if (value > max) return max;
    return value;
}

#endif
```

```c
// main.c
#include <stdio.h>
#include "math_utils.h"   // includes the inline functions.

int main(void) {
    int a = 10, b = 20;
    printf("Add: %d\n", add(a, b));
    printf("Sub: %d\n", subtract(a, b));
    printf("Mul: %d\n", multiply(a, b));

    printf("Clamp(15, 10, 20): %d\n", clamp(15, 10, 20));
    printf("Clamp(5, 10, 20): %d\n", clamp(5, 10, 20));
    printf("Clamp(25, 10, 20): %d\n", clamp(25, 10, 20));

    return 0;
}
```
**Code Breakdown:**
- `static inline` in a header means each translation unit that includes it gets its own copy.
- This is type‑safe and avoids the macro pitfalls.
- The compiler may inline these functions, making them fast.

### Example 3: Inline Function with `extern` Definition
```c
// file1.c
#include <stdio.h>

// Inline definition (external linkage).
// This provides an inline version, but if needed, an external definition is also generated.
inline int square(int x) {
    return x * x;
}

int main(void) {
    int result = square(5);   // May be inlined.
    printf("Square: %d\n", result);

    // Taking the address forces an external definition.
    int (*func)(int) = square;
    int result2 = func(10);
    printf("Via function pointer: %d\n", result2);

    return 0;
}

// file2.c (optional) – could provide an extern definition if needed.
// But with C99, the inline definition in file1.c is sufficient.
```

**Code Breakdown:**
- `inline` without `static` provides an inline definition with external linkage.
- If the compiler cannot inline a call (e.g., via a function pointer), it may use an external definition.
- The compiler may generate a "weak" external definition to avoid multiple definition errors if another file provides one.

### Example 4: `static inline` vs Regular Function for Accessor
```c
#include <stdio.h>
#include <stdlib.h>

// A simple struct.
typedef struct {
    int x;
    int y;
} Point;

// Static inline accessor – likely to be inlined.
static inline int get_x(const Point *p) {
    return p->x;
}

// Static inline setter – likely inlined.
static inline void set_x(Point *p, int value) {
    p->x = value;
}

// Regular function for comparison.
int get_x_regular(const Point *p) {
    return p->x;
}

void set_x_regular(Point *p, int value) {
    p->x = value;
}

int main(void) {
    Point p = {10, 20};

    // Using inline accessors – no function call overhead.
    printf("x: %d\n", get_x(&p));
    set_x(&p, 30);
    printf("x: %d\n", get_x(&p));

    // Using regular functions – function call overhead.
    printf("x: %d\n", get_x_regular(&p));
    set_x_regular(&p, 40);
    printf("x: %d\n", get_x_regular(&p));

    return 0;
}
```
**Code Breakdown:**
- `static inline` accessors are common in C for performance‑critical getters/setters.
- They provide a clean interface with no runtime overhead if inlined.
- Regular functions are still type‑safe but incur call overhead.

### Example 5: Inline Function with Static Local Variable
```c
#include <stdio.h>

// Inline function with static local – this is unusual and may not behave as expected.
static inline int next_id(void) {
    static int id = 0;   // Static local – persists across calls.
    return id++;
}

int main(void) {
    printf("ID: %d\n", next_id());   // 0
    printf("ID: %d\n", next_id());   // 1
    printf("ID: %d\n", next_id());   // 2
    // The static local works even if the function is inlined.
    // Each translation unit will have its own copy of the static variable
    // because it's static inline.
    return 0;
}
```
**Code Breakdown:**
- `static inline` with a static local variable: each translation unit gets its own copy of the static local.
- If you need a single instance across translation units, use a regular function.

### Example 6: Inline Function and Macro Comparison
```c
#include <stdio.h>

// Inline function – type‑safe and side‑effect safe.
static inline int square_inline(int x) {
    return x * x;
}

// Macro – not type‑safe, side effects can cause issues.
#define SQUARE_MACRO(x) ((x) * (x))

int main(void) {
    int a = 5;

    // Inline function: x is evaluated once.
    int result1 = square_inline(a++);
    printf("square_inline: result = %d, a = %d\n", result1, a);   // result=25, a=6

    // Macro: a++ is expanded twice, causing undefined behaviour.
    a = 5;
    int result2 = SQUARE_MACRO(a++);
    printf("SQUARE_MACRO: result = %d, a = %d\n", result2, a);   // Undefined!

    // Type safety: inline function catches mismatched types.
    // square_inline("hello");   // ❌ Error – compile-time type check.

    // Macro: this compiles (or fails in unexpected ways).
    // SQUARE_MACRO("hello");    // ❌ Compiles but does nonsense.

    return 0;
}
```
**Code Breakdown:**
- The inline function is safe with side effects (`a++` is evaluated once).
- The macro has side‑effect issues – `a++` expands twice.
- The inline function provides type checking; the macro does not.

---

## 10. Common Use Cases

| Use Case | Example |
|----------|---------|
| **Simple accessors** | `static inline int get_value(Obj *o) { return o->value; }` |
| **Small mathematical functions** | `static inline int max(int a, int b) { return a > b ? a : b; }` |
| **Header‑only libraries** | `static inline` functions in headers for reusable utilities. |
| **Performance‑critical loops** | Functions called inside tight loops. |
| **Type‑safe macros** | Replacing macros with inline functions. |
| **Zero‑overhead abstractions** | Wrapping struct fields with accessors. |

---

## 11. Best Practices

### General
- **Use `static inline`** for functions defined in headers – it avoids linking issues.
- **Keep inline functions small** – one or two lines, simple operations.
- **Use for performance‑critical code** – but measure to confirm benefits.
- **Prefer inline functions over macros** – they're safer and more maintainable.
- **Avoid inline for complex functions** – loops, recursion, large bodies won't be inlined.

### Header Files
- **Always use `static inline`** – each translation unit gets its own copy, no multiple definition errors.
- **Consider `extern inline`** only if you need a single external definition (rare).
- **Keep header‑inline functions simple** – to avoid code bloat.

### Compiler Flags
- **Compile with `-O2` or `-O3`** – inlining is more aggressive with optimisation.
- **Use `-finline-functions`** (GCC) – to force more aggressive inlining.
- **Use `-fno-inline`** – to disable inlining for debugging.

### Debugging
- **Compile with `-g`** – debug symbols are still available for inline functions.
- **Use `-O0` during debugging** – inlining is disabled, making debugging easier.

---

## 12. Common Mistakes

### Mistake 1: Defining Inline Functions in Headers Without `static`
```c
// ❌ Wrong – can cause multiple definition errors.
inline int add(int a, int b) { return a + b; }

// ✅ Correct – use static inline.
static inline int add(int a, int b) { return a + b; }
```
**Why it's wrong:** If the header is included in multiple translation units, the compiler may generate an external definition in each, leading to linker errors (multiple definitions).

### Mistake 2: Assuming Inline Means Always Inlined
```c
// ❌ Wrong – inline is a hint, not a command.
inline int complex_function(int n) {
    if (n <= 1) return 1;
    return n * complex_function(n - 1);   // Recursion – won't be inlined.
}
// The compiler will likely ignore the inline keyword for recursive functions.
```

### Mistake 3: Using Inline for Large Functions
```c
// ❌ Wrong – large functions are unlikely to be inlined.
inline void huge_function(void) {
    // hundreds of lines of code...
}
// The compiler will likely ignore inline for this function.
```

### Mistake 4: Taking the Address of an Inline Function
```c
static inline int square(int x) { return x * x; }

int main(void) {
    int (*func)(int) = square;   // Address taken.
    int result = func(5);        // This call cannot be inlined.
}
```
**Why it's wrong:** When you take the address of a function, the compiler must generate a non‑inline version. Calls through the pointer cannot be inlined.

### Mistake 5: Forgetting `static` for Header‑Only Inline Functions
```c
// utils.h
inline int add(int a, int b) { return a + b; }   // ❌ Wrong.

// utils.h
static inline int add(int a, int b) { return a + b; }   // ✅ Correct.
```

### Mistake 6: Expecting Inline to Reduce Code Size
```c
// Inlining can increase code size because the function body is duplicated at each call site.
// For large functions or many call sites, inlining can bloat the executable.
```

---

## 13. Performance Considerations

- **Function call overhead** – inlining eliminates the call and return instructions, saving a few cycles.
- **Register allocation** – inlining allows the compiler to optimise across the function boundary, potentially improving register usage.
- **Code bloat** – inlining can increase code size, hurting instruction cache performance.
- **Optimisation level** – inlining is most effective with `-O2` or `-O3`.
- **Micro‑benchmarks** – always measure; the benefit of inlining is often negligible except in tight loops.

### When Inlining Helps Most
- **Frequently called, small functions** – accessors, simple arithmetic.
- **Functions called inside loops** – avoids call overhead per iteration.
- **Functions with constant arguments** – inlining can enable constant propagation.

### When Inlining Hurts
- **Large functions** – code bloat can reduce instruction cache hit rates.
- **Functions called infrequently** – the overhead of duplication outweighs the call overhead.
- **Functions with many call sites** – significant code bloat.

---

## 14. Security Considerations

- **Inline functions** don't introduce new security risks; they're a performance optimisation.
- **Macro vs inline** – inline functions are safer than macros because they avoid side‑effect issues and type mismatches.
- **Code bloat** – excessive inlining can increase the attack surface (more code, more potential vulnerabilities), but this is a minor concern.

---

## 15. Debugging Tips

- **Compile with `-O0`** – disables inlining, making debugging easier.
- **Compile with `-g`** – debug symbols are available even for inline functions.
- **Use `-fno-inline`** (GCC) – to disable inlining globally.
- **Use `__attribute__((noinline))`** (GCC/Clang) – to prevent inlining a specific function.
- **Use `__inline__` or `__attribute__((always_inline))`** – to force inlining (compiler‑specific).

---

## 16. When to Use

- **Small, frequently called functions** – accessors, simple math, comparisons.
- **Header‑only libraries** – `static inline` functions are a common pattern.
- **Performance‑critical code** – measured hot paths.
- **When you need type safety but want to avoid macro pitfalls** – replace macros with inline functions.

---

## 17. When Not to Use

- **Large functions** – they're unlikely to be inlined and will bloat code.
- **Recursive functions** – cannot be inlined.
- **Functions with many call sites** – may cause code bloat.
- **Performance is not critical** – regular functions are fine and easier to debug.
- **Portability concerns** – `inline` is C99+; for C89, use macros or compiler extensions.

---

## 18. Related Concepts

- **Functions** – the base concept.
- **Macros** – the predecessor to inline functions for avoiding call overhead.
- **`static`** – controls linkage and visibility.
- **`extern`** – external linkage for functions.
- **Compiler optimisation** – inlining is one of many optimisation techniques.
- **Link‑time optimisation (LTO)** – enables inlining across translation units.
- **Function pointers** – force non‑inline versions.

---

## 19. Did You Know?

- The `inline` keyword was introduced in C99; before that, compilers used language extensions (`__inline`, `__inline__`).
- In C++, `inline` has slightly different semantics – it's used to allow multiple definitions of the same function across translation units.
- Some compilers (like GCC) have an `always_inline` attribute to force inlining, overriding the compiler's heuristics.
- Link‑time optimisation (LTO) can inline functions across translation units, even if they weren't marked `inline`.
- Inline functions can be recursive, but the compiler will rarely inline recursive calls.
- The Linux kernel makes extensive use of `static inline` for performance‑critical operations.

---

## 20. Summary

- **Inline functions** are functions that the compiler may expand at the call site, avoiding function call overhead.
- **They are type‑safe** and avoid the pitfalls of macros (side effects, lack of type checking).
- **The `inline` keyword is a hint** – the compiler decides whether to inline.
- **`static inline`** is the safest pattern for header‑only functions – each translation unit gets its own copy.
- **Best for small, frequently called functions** – accessors, simple arithmetic.
- **Not suitable for large functions** – unlikely to be inlined and may cause code bloat.
- **Common mistakes**: using `inline` without `static` in headers, assuming inlining always happens, taking the address of an inline function.
- **Performance**: measure before and after; inlining can help in tight loops but can also bloat code.
- **Understanding inline functions** gives you a tool for writing efficient, type‑safe, and maintainable C code, especially in performance‑critical contexts.