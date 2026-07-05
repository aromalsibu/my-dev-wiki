# The C Standard

## 1. Overview

### Definition
The C Standard is the formal specification of the C programming language, maintained by the ISO/IEC JTC1/SC22/WG14 working group (commonly known as the C Standard Committee). It defines the syntax, semantics, constraints, and library requirements for C implementations. Compilers, libraries, and tools that claim to support "C" must conform to a specific version of this standard. The standard is not a book you read cover‑to‑cover; it's a technical document that defines the language's behaviour in precise terms.

### Purpose
The C Standard serves several critical purposes:
- **Portability** – code written for a standard‑conforming compiler will behave consistently across platforms.
- **Interoperability** – libraries and tools can rely on common language features.
- **Language definition** – provides an unambiguous reference for compiler implementers and programmers.
- **Evolution** – provides a mechanism for adding new features while preserving backward compatibility.

### Where It Fits
The C Standard sits at the very foundation of the language. Everything in this handbook—syntax, semantics, the standard library, undefined behaviour—is defined by the standard. Compilers interpret the standard to decide how to translate code to machine instructions. When you write C code, you're writing against a specific version of this standard.

---

## 2. Why It Exists

### The Problem Without a Standard
Before the standardisation of C, there were many variations of the language:
- Different compilers had different features and behaviours.
- Code written for one compiler might not work on another.
- There was no authoritative reference for what was "valid C".
- Portability was difficult; you had to write code for a specific compiler.

### The Solution: A Formal Standard
The C Standard provides a single, unified definition of the language, allowing:
- **Portable code** – write once, compile anywhere (with standard‑conforming compilers).
- **Competition** – multiple compiler vendors can implement the same standard.
- **Longevity** – code written today will compile for decades.
- **Innovation** – the standard evolves to include new features while maintaining compatibility.

### Why It Was Introduced
C was standardised to address the fragmentation that occurred as the language grew in popularity. The first ANSI standard (C89) formalised the language, codifying what was already common practice in the K&R version. The ISO took over, producing C90, C99, C11, C17, and C23. The standard ensures that C remains a viable language for systems programming for decades to come.

---

## 3. Standard Versions and Evolution

### Timeline of C Standards

| Version | Year | Common Name | Key Features |
|---------|------|-------------|--------------|
| **K&R C** | 1978 | K&R C | Original language as described in the first edition of "The C Programming Language". No standard. |
| **C89** | 1989 | ANSI C | First formal standard. Added function prototypes, `const`, `volatile`, standard library. |
| **C90** | 1990 | ISO C90 | Identical to C89, but published by ISO. |
| **C95** | 1995 | AMD1 | Amendment 1: added wide characters, `wchar_t`, `<wctype.h>`. |
| **C99** | 1999 | C99 | Added `inline`, `restrict`, variable‑length arrays, fixed‑width integer types (`<stdint.h>`), `stdbool.h`, `_Complex`, `_Bool`, `//` comments. |
| **C11** | 2011 | C11 | Added multi‑threading support (`<threads.h>`), atomic operations (`<stdatomic.h>`), `_Generic`, `_Alignas`, `_Alignof`, `_Noreturn`, `_Static_assert`. |
| **C17** | 2018 | C17 | Bug‑fix release; no new language features. Also known as C18. |
| **C23** | 2024 | C23 | Added `constexpr`, `nullptr`, `typeof`, `typeof_unqual`, `bool` keyword, attributes (`[[...]]`), enhanced `#embed`, `bitint`, and more. |

### Comparison of Key Features Across Standards

| Feature | C89 | C99 | C11 | C17 | C23 |
|---------|-----|-----|-----|-----|-----|
| `//` comments | ❌ | ✅ | ✅ | ✅ | ✅ |
| Function prototypes | ✅ | ✅ | ✅ | ✅ | ✅ |
| `inline` | ❌ | ✅ | ✅ | ✅ | ✅ |
| `restrict` | ❌ | ✅ | ✅ | ✅ | ✅ |
| Variable‑length arrays | ❌ | ✅ | ✅ | ✅ | ✅ (optional) |
| `stdbool.h` | ❌ | ✅ | ✅ | ✅ | ✅ |
| `stdint.h` | ❌ | ✅ | ✅ | ✅ | ✅ |
| Thread support | ❌ | ❌ | ✅ | ✅ | ✅ |
| Atomics | ❌ | ❌ | ✅ | ✅ | ✅ |
| `_Generic` | ❌ | ❌ | ✅ | ✅ | ✅ |
| `constexpr` | ❌ | ❌ | ❌ | ❌ | ✅ |
| `nullptr` | ❌ | ❌ | ❌ | ❌ | ✅ |
| `typeof` | ❌ | ❌ | ❌ | ❌ | ✅ |
| Boolean keyword | ❌ | ❌ | ❌ | ❌ | ✅ (`bool` now keyword) |
| Attributes (`[[...]]`) | ❌ | ❌ | ❌ | ❌ | ✅ |

---

## 4. Core Concepts

### Standardisation Bodies
- **ANSI** (American National Standards Institute) – developed the original C standard.
- **ISO** (International Organization for Standardization) – maintains the international standard.
- **WG14** (Working Group 14) – the ISO committee responsible for C.

### The Standard Document
The standard is a large document, typically over 500 pages, divided into:
- **Language specification** – syntax, semantics, constraints.
- **Library specification** – the standard library functions and headers.
- **Annexes** – informative, normative, and optional sections.

### Conformance
- **Strictly conforming program** – uses only features defined by the standard, produces predictable output, and is portable.
- **Conforming program** – may use implementation‑defined features but still compiles with a standard‑conforming compiler.
- **Conforming implementation** – a compiler or environment that complies with the standard.

### Implementation‑Defined Behaviour
- Behaviour that the standard leaves to the implementation to define (e.g., size of `int`, endianness).
- The implementation must document its behaviour.
- Code relying on implementation‑defined behaviour is not fully portable.

### Unspecified Behaviour
- The standard provides two or more possibilities; the implementation may choose any, but doesn't need to document it (e.g., order of evaluation of function arguments).

### Undefined Behaviour
- Behaviour that is not defined by the standard. The compiler can do anything—crash, produce unexpected results, or even format your hard drive (in theory). Examples:
  - Signed integer overflow.
  - Null pointer dereference.
  - Accessing an array out of bounds.
  - Modifying the same variable multiple times between sequence points.

### Common Extensions
- Many compilers provide extensions beyond the standard (e.g., `#pragma once`, `__attribute__`, `inline` keyword extensions). Code using these is not strictly conforming but may be portable across compilers that support the extensions.

### Standard Headers (Annex B)
- The standard defines a set of mandatory headers (e.g., `<stdio.h>`, `<stdlib.h>`).
- Additional headers are optional (e.g., `<threads.h>` in C11, `<stdatomic.h>`).
- C23 adds new headers and updates existing ones.

---

## 5. How It Works – Compiler and Standard Conformance

### The Compiler's Role
- A compiler that claims to conform to a particular version of the standard must implement the language features and library functions defined in that version.
- Compilers may provide flags to select the standard version: `-std=c99`, `-std=c11`, `-std=c17`, `-std=c23` (GCC/Clang).
- Some compilers default to a particular standard (e.g., GCC historically defaulted to GNU extensions; now may default to a recent standard).

### The Standard and Undefined Behaviour
- The standard defines rules; violating them leads to undefined behaviour.
- Compilers may assume undefined behaviour never occurs and optimise accordingly.
- This can lead to surprises if you rely on undefined behaviour.

### Portability
- Strictly conforming programs should work on any standard‑conforming compiler.
- In practice, most programs rely on some implementation‑defined features (e.g., sizes of types) but can still be portable if they check for them.
- `#ifdef` and feature test macros can be used to handle platform differences.

---

## 6. Internal Architecture – How the Standard Is Developed

### WG14 Working Group
- Meets regularly (twice a year) to discuss proposals.
- Proposals are submitted as "papers" (document numbers).
- The committee votes on proposals; approval requires consensus.
- New features are added after careful consideration of backward compatibility.

### Process
1. **Proposal** – a paper is submitted.
2. **Discussion** – the committee debates the proposal.
3. **Revision** – the proposal is refined based on feedback.
4. **Vote** – the committee votes to adopt or reject.
5. **Draft** – revisions are incorporated into the working draft.
6. **Ballot** – the draft is sent for final approval.
7. **Publication** – the standard is officially published.

### Frequency of New Standards
- Historically, a new standard has been published every 10 years or so.
- C23 is the most recent major revision.
- The committee aims to be more frequent (every few years) for future revisions.

---

## 7. Lifecycle / Workflow of a Standard

1. **Research** – the committee identifies areas for improvement or new features.
2. **Proposal** – someone submits a paper.
3. **Discussion** – the committee debates.
4. **Revision** – the paper is updated.
5. **Vote** – the committee votes to adopt.
6. **Integration** – the feature is added to the working draft.
7. **Finalisation** – after all features are added, the draft is finalised.
8. **Publication** – the standard is published.
9. **Implementation** – compiler vendors implement the new features.
10. **Adoption** – developers start using the new features.

---

## 8. Practical Examples

### Example 1: Specifying a Standard with GCC
```c
#include <stdio.h>

int main(void) {
    // Use a feature from C99 – the inline keyword.
    // To compile with C89, this would cause an error.
    // Use -std=c99 to enable C99 features.
    printf("Hello, C99!\n");
    return 0;
}
```
**Compilation Commands:**
```bash
# Compile with C99 standard
gcc -std=c99 main.c -o main

# Compile with C11 standard
gcc -std=c11 main.c -o main

# Compile with GNU extensions (default on GCC for many versions)
gcc main.c -o main
```

### Example 2: Checking for Feature Test Macros
```c
#include <stdio.h>

int main(void) {
    // Check if the compiler supports C99 or later.
    #ifdef __STDC_VERSION__
        #if __STDC_VERSION__ >= 199901L
            printf("C99 or later (__STDC_VERSION__ = %ld)\n", __STDC_VERSION__);
        #else
            printf("C89 or earlier\n");
        #endif
    #else
        printf("Pre‑ANSI C\n");
    #endif

    #if __STDC_VERSION__ >= 201112L
        printf("C11 or later\n");
    #endif

    #if __STDC_VERSION__ >= 201710L
        printf("C17 or later\n");
    #endif

    #if __STDC_VERSION__ >= 202311L
        printf("C23 or later\n");
    #endif

    return 0;
}
```
**Code Breakdown:**
- `__STDC_VERSION__` is a predefined macro that indicates the standard version.
- Values: `199901L` for C99, `201112L` for C11, `201710L` for C17, `202311L` for C23.
- You can use this macro to conditionally compile code that uses features from newer standards.

### Example 3: Using C11 Threads (Optional Feature)
```c
#include <stdio.h>
#include <threads.h>   // C11 feature – not available in C99.

int thread_func(void *arg) {
    printf("Hello from thread!\n");
    return 0;
}

int main(void) {
    thrd_t t;
    thrd_create(&t, thread_func, NULL);
    thrd_join(t, NULL);
    return 0;
}
```
**Code Breakdown:**
- `<threads.h>` was introduced in C11.
- To compile, you need a C11‑compliant compiler and library.
- Use `-std=c11` (or later) and link with `-pthread` on Unix.

### Example 4: Using C23 Features
```c
#include <stdio.h>

// C23: constexpr indicates compile‑time evaluation.
constexpr int square(int x) {
    return x * x;
}

// C23: bool is now a keyword (previously needed #include <stdbool.h>).
bool is_positive(int x) {
    return x > 0;
}

int main(void) {
    // C23: constexpr functions can be evaluated at compile time.
    const int SIZE = square(5);
    int arr[SIZE];   // 5 * 5 = 25, valid at compile time.

    // C23: nullptr constant (replaces NULL).
    int *p = nullptr;

    // C23: typeof operator.
    typeof(arr) another_arr;   // another_arr is int[25].

    // C23: attributes.
    [[nodiscard]] int result = square(10);

    printf("Size: %d\n", SIZE);
    printf("is_positive(5): %d\n", is_positive(5));

    return 0;
}
```
**Code Breakdown:**
- `constexpr` functions are new in C23, allowing compile‑time evaluation.
- `bool` is a keyword in C23 (no longer requires `<stdbool.h>`).
- `nullptr` is a new keyword for a null pointer constant.
- `typeof` is a new operator to get the type of an expression.
- Attributes (`[[...]]`) are a new syntax for annotations.

---

## 9. Common Use Cases

| Use Case | Example |
|----------|---------|
| **Writing portable code** | Use standard features only, avoid undefined behaviour. |
| **Compiler selection** | Choose a standard version for your project. |
| **Feature detection** | Use `__STDC_VERSION__` to conditionally compile. |
| **Library development** | Follow the standard to ensure broad compatibility. |
| **Performance** | Use standard features that enable optimisations (`restrict`, `inline`). |

---

## 10. Best Practices

### General
- **Choose a standard version** and stick to it for your project.
- **Avoid undefined behaviour** – it can be exploited by compiler optimisations and lead to bugs.
- **Use `-pedantic`** (GCC/Clang) to enforce strict standard conformance.
- **Use `-Wall -Wextra`** to catch many standard‑related issues.
- **Prefer standard features** over compiler extensions for portability.

### Portability
- **Check type sizes** with `sizeof` and use `<stdint.h>` for fixed‑width types.
- **Use feature test macros** (`__STDC_VERSION__`) to conditionally compile.
- **Avoid assuming endianness** or alignment unless necessary.
- **Use standard library functions**; avoid platform‑specific ones.

### Compiler Flags
```bash
# Enforce strict C99 conformance
gcc -std=c99 -pedantic -Wall -Wextra main.c -o main

# Enforce strict C11 conformance
gcc -std=c11 -pedantic -Wall -Wextra main.c -o main
```

### Standard Library
- **Know which headers are standard** and which are optional.
- **Use `#include` with angle brackets** for standard headers.
- **Be aware of optional features** (e.g., `<threads.h>` may not be available on all platforms).

---

## 11. Common Mistakes

### Mistake 1: Assuming a Specific Standard Version
```c
// ❌ Wrong – assumes C99 or later.
inline int square(int x) { return x * x; }
// This is a syntax error in C89.
// ✅ Correct – use feature test macro.
#ifdef __STDC_VERSION__
    #if __STDC_VERSION__ >= 199901L
        inline int square(int x) { return x * x; }
    #endif
#endif
```

### Mistake 2: Using Undefined Behaviour
```c
// ❌ Wrong – signed overflow is undefined.
int x = INT_MAX;
int y = x + 1;   // Undefined behaviour.
// ✅ Correct – avoid overflow.
if (x < INT_MAX) {
    y = x + 1;
} else {
    // handle overflow.
}
```

### Mistake 3: Assuming Implementation‑Defined Behaviour
```c
// ❌ Wrong – assumes `int` is 32 bits.
int x = 1 << 31;   // Shift of signed int; implementation‑defined.
// ✅ Correct – use unsigned and fixed‑width types.
uint32_t x = 1U << 31;
```

### Mistake 4: Not Using `-std` Flag
- Compilers often default to an older standard or enable extensions. Always specify the standard you want.
```bash
# ❌ Wrong – may not use the standard you expect.
gcc main.c -o main

# ✅ Correct – specify the standard.
gcc -std=c17 main.c -o main
```

### Mistake 5: Relying on Extensions
```c
// ❌ Wrong – uses GCC extension (declaration inside for loop? Actually allowed in C99).
for (int i = 0; i < n; i++) { ... }   // OK in C99; but if you're targeting C89, it's an extension.
// ✅ Correct – use the standard version you've chosen.
```

### Mistake 6: Forgetting to Include Standard Headers
```c
// ❌ Wrong – uses printf without including stdio.h.
int main(void) {
    printf("Hello\n");   // Implicit declaration – invalid in C99+.
}
// ✅ Correct – include <stdio.h>.
#include <stdio.h>
int main(void) { printf("Hello\n"); return 0; }
```

---

## 12. Performance Considerations

- **The standard itself doesn't affect performance**; the compiler implementation does.
- **Undefined behaviour** can lead to surprising optimisations (compilers assume it never happens).
- **Implementation‑defined behaviour** can affect performance (e.g., `sizeof(int)` affects how many bytes are processed).
- **Newer standards often include optimisation aids** (`restrict`, `inline`, `constexpr`).
- **Using `-std`** with a recent standard may enable newer optimisation features.

---

## 13. Security Considerations

- **Undefined behaviour** is a major source of security vulnerabilities (buffer overflows, integer overflows, use‑after‑free).
- **Implementation‑defined behaviour** can vary across platforms, leading to unintended security differences.
- **Newer standards** add safety features (`_Static_assert`, `constexpr`, type‑generic macros) to help catch issues early.
- **Compiler flags** like `-fstack-protector-strong`, `-fsanitize=undefined` can help catch undefined behaviour.

---

## 14. Debugging Tips

- **Use `-std`** to select a standard version; mismatches can cause errors.
- **Use `-pedantic -Wall -Wextra`** to catch violations of the standard.
- **Use `-fsanitize=undefined`** (GCC/Clang) to catch undefined behaviour at runtime.
- **Use `__STDC_VERSION__`** in debug output to see which standard the compiler is using.
- **Check the compiler's documentation** for its conformance status.

---

## 15. When to Use

- **Always** – every C program is written against some version of the standard.
- **Choose the latest standard** that your compiler supports and that meets your needs (C23, C17, C11, C99).
- **Use `-std`** to enforce a specific standard.

---

## 16. When Not to Use

- **Avoid using an older standard** if your compiler supports a newer one (unless you need compatibility).
- **Avoid relying on extensions** unless you're targeting a specific compiler and you accept the non‑portability.

---

## 17. Related Concepts

- **Compilers** – implement the standard.
- **Preprocessor** – handles directives like `#define`, `#ifdef`.
- **Undefined behaviour** – violations of the standard.
- **Implementation‑defined behaviour** – documented choices by the compiler.
- **Portability** – writing code that works across platforms.
- **Feature test macros** – `__STDC_VERSION__`, `__STDC__`.
- **Standard library** – the library defined by the standard.

---

## 18. Did You Know?

- The C standard was originally developed by ANSI (X3J11 committee) in the 1980s and was known as ANSI C.
- The first ISO standard was C90, which is essentially identical to C89.
- C95 added wchar_t and wide character support, but is rarely used.
- C99 was a major update, but some features (like variable‑length arrays) were made optional in C11.
- C11 added multi‑threading support, but many compilers still default to an older standard.
- C17 was a bug‑fix release with no new language features.
- C23 added many features, including `constexpr`, `nullptr`, `typeof`, and attributes, bringing C closer to C++ in some areas.
- The standard is a huge document, but many C programmers have never read it; they rely on books, tutorials, and compiler behaviour.

---

## 19. Summary

- **The C Standard is the formal specification** of the C programming language, maintained by ISO/WG14.
- **Versions**: C89, C90, C95, C99, C11, C17, C23 (latest).
- **Conformance**: Strictly conforming programs use only standard features; conforming programs may use implementation‑defined features.
- **Implementation‑defined, unspecified, and undefined behaviour** are important categories to understand.
- **The standard evolves** – C23 adds features like `constexpr`, `nullptr`, `typeof`, and attributes.
- **Best practices**: choose a standard version, avoid undefined behaviour, use `-std` and `-pedantic`, prefer portable code.
- **Common mistakes**: assuming a standard version, using undefined behaviour, relying on implementation‑defined behaviour without checking, forgetting to include headers.
- **Understanding the C Standard** is essential for writing portable, reliable, and maintainable C code.