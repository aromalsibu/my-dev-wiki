# Identifiers

## 1. Overview

### Definition
An identifier is a name you give to a program element – a variable, function, structure, union, enumeration, typedef, or label. It is a sequence of characters that the compiler uses to refer to that element during compilation and linking. Identifiers are user‑defined (unlike keywords, which are built‑in).

### Purpose
Identifiers allow you to refer to memory locations, functions, and other objects in a human‑readable way. They make code self‑documenting and maintainable. Without identifiers, you would have to work with raw addresses – an impossibility for any non‑trivial program.

### Where It Fits
Identifiers are the most common token type in C source code. They appear in declarations, definitions, and expressions, and are resolved by the compiler’s symbol table. Understanding their rules, scope, and linkage is essential for writing correct and well‑organised code.

---

## 2. Why It Exists

### The Problem Without Identifiers
If everything were accessed by memory addresses, programming would be impossible. You would need to manually track where each variable is stored and update addresses when the program changes. This is exactly what assembly language does – and it is error‑prone, unreadable, and non‑portable.

### The Solution: Symbolic Naming
C provides symbolic names that the compiler translates to addresses. The linker resolves cross‑file references. The result: you can write `int result = a + b;` and the compiler handles the rest. Identifiers also enable abstraction – you can name functions to represent complex operations.

### Why It Was Introduced
Symbolic names have been a staple of high‑level languages since FORTRAN. C inherited the concept from B and BCPL, but added strict typing and linkage rules. The design choices – case sensitivity, allowed characters, and length – were influenced by the PDP‑11’s assembler and the desire for simplicity.

---

## 3. Syntax / Basic Usage

### Rules for Forming Identifiers
- Must begin with a letter (a‑z, A‑Z) or an underscore (`_`).
- Subsequent characters can be letters, digits (0‑9), or underscores.
- Case‑sensitive: `count`, `Count`, and `COUNT` are three distinct identifiers.
- No limit on length in the standard (though compilers may have internal limits, at least 31 significant characters are guaranteed for external identifiers – C89/90 – but modern compilers support much longer).

### Basic Example
```c
#include <stdio.h>

int main(void) {          // main – identifier (function)
    int my_variable = 42; // my_variable – identifier (variable)
    int _private = 100;   // _private – identifier (starts with underscore)
    int 2nd = 200;        // ❌ invalid – starts with digit
    int my-var = 300;     // ❌ invalid – hyphen is not allowed
    return 0;
}
```

### Complete Program with Various Identifiers
```c
#include <stdio.h>

// Function identifier: calculate_sum
int calculate_sum(int a, int b) {
    return a + b;   // a and b are parameter identifiers
}

// typedef identifier: Number
typedef int Number;

// struct identifier: Point
struct Point {
    int x;
    int y;
};

int main(void) {
    Number n1 = 10;           // n1 is a variable identifier
    Number n2 = 20;           // n2 is a variable identifier
    int result = calculate_sum(n1, n2); // result – variable, calculate_sum – function

    struct Point p1 = {3, 4}; // p1 – variable of struct type
    printf("Result: %d\n", result);
    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>   // stdio.h is a header name, not a C identifier

int calculate_sum(int a, int b) {
    // calculate_sum – identifier for the function.
    // a and b – identifiers for parameters (local scope).
    return a + b;
}

typedef int Number;
// Number – identifier for a type alias (typedef name).

struct Point {
    // Point – identifier for the structure tag.
    int x;   // x – identifier for a structure member
    int y;   // y – identifier for a structure member
};

int main(void) {
    Number n1 = 10;   // n1 – variable identifier
    Number n2 = 20;   // n2 – variable identifier
    int result = calculate_sum(n1, n2);
    // result – variable identifier.
    // calculate_sum – function identifier used in a call.

    struct Point p1 = {3, 4};
    // p1 – variable identifier; Point – structure tag.

    printf("Result: %d\n", result);
    return 0;
}
```

---

## 4. Mental Model – Identifiers as Name Tags

Imagine a large warehouse (the computer’s memory). Inside, there are many shelves (memory locations). Instead of remembering shelf numbers like “0x7FFF1234”, you attach a name tag to each shelf – that name tag is an identifier.

- You can have `box_count`, `box_count2`, `BOX_COUNT` – each is a different tag.
- You cannot have a tag that starts with a digit (e.g., `1st_box`) – the warehouse system forbids it.
- Some tags are reserved for the warehouse staff (keywords like `int`) – you cannot use them.
- Different areas of the warehouse have different visibility: a tag in a shipping container (`static`) is not visible outside, while a tag on a pallet in the main hall (`extern`) is visible everywhere.

---

## 5. Core Concepts

| Concept | Explanation |
|---------|-------------|
| **Character set** | Identifiers can use letters, digits, and underscores; must start with a letter or underscore. |
| **Case sensitivity** | `foo`, `Foo`, `FOO` are distinct. |
| **Reserved names** | Cannot be a keyword; also, some names are reserved for the implementation (starting with `_` and a capital, or with `__`). |
| **Scope** | Where the identifier is visible: block scope, file scope, function scope (for labels), or function prototype scope. |
| **Linkage** | How the identifier is shared across translation units: external, internal, or none. |
| **Lifetime** | When the object exists: static, automatic, or allocated. |
| **Name spaces** | C has separate name spaces for different categories (e.g., structure tags vs. ordinary identifiers). |
| **Significant characters** | The standard guarantees that at least 31 characters are significant for internal identifiers; external (linker) may be less (commonly 31 or 63). |

---

## 6. How It Works – Identifier Resolution

### Step‑by‑step
1. **Lexer** reads characters and recognises an identifier token.
2. **Parser** matches it according to grammar (e.g., declaration, expression).
3. **Compiler’s symbol table** records the identifier if it is a new declaration, or looks it up if it is a use.
   - The symbol table stores the identifier’s type, scope, linkage, and storage location.
4. **Semantic analysis** checks that the identifier is used correctly (type matching, scope visibility).
5. **Linker** resolves identifiers with external linkage across object files, replacing them with final addresses.

### ASCII Diagram – Identifier Processing
```
Source: int count = 5;
           │
           ▼
   Lexer: "int" – keyword; "count" – identifier; "=" – operator; "5" – constant
           │
           ▼
   Parser: declaration – type "int", declarator "count", initialiser "5"
           │
           ▼
   Symbol table: insert "count", type int, scope block, storage auto
           │
           ▼
   Code generation: allocate storage for count (stack), emit init code
           │
           ▼
   Linker: if "count" were extern, resolve its address; otherwise no action.
```

---

## 7. Internal Architecture – Symbol Tables and Name Mangling

- The **compiler’s symbol table** is a data structure (often a hash table) that maps identifiers to their attributes.
- For external identifiers, the compiler may append an underscore (common in some ABIs) but does not “mangle” names like C++ does.
- The **linker’s symbol table** contains global identifiers (functions and variables) with their addresses; it resolves references across object files.

### Separate Name Spaces
C maintains several disjoint name spaces – the same identifier can be used for different purposes in different contexts without conflict:
- **Labels** – (used with `goto`) – in their own space.
- **Structure/union/enum tags** – after `struct`, `union`, or `enum` (e.g., `struct Point`).
- **Members of structures/unions** – each struct/union has its own name space.
- **Ordinary identifiers** – variables, functions, typedef names, enumeration constants.

Example:
```c
int point = 10;        // ordinary identifier
struct point { int x; }; // tag (different name space)
point p; // invalid – need struct point
```

---

## 8. Lifecycle / Workflow of an Identifier

1. **Declaration** – introduces the identifier and its type (optionally with storage class).
2. **Definition** – allocates storage (for variables) or provides function body.
3. **Use** – the identifier appears in expressions, assignments, or calls.
4. **Scope exit** – the identifier may become invisible (if local) or remain (if global/static).
5. **Program termination** – the object’s lifetime ends; storage is reclaimed.

---

## 9. Practical Examples

### Example 1: Scope and Visibility
```c
#include <stdio.h>

int global = 100;          // file scope – visible throughout the file

void func(void) {
    int local = 20;        // block scope – only inside func
    {
        int local = 30;    // new variable shadows the outer one (valid)
        printf("%d\n", local); // prints 30
    }
    printf("%d\n", local); // prints 20 (outer local restored)
}

int main(void) {
    int global = 5;        // this local shadows the global variable
    printf("%d\n", global); // prints 5 – uses local
    // To access the global, you can use :: in C++ but not in C – no scope resolution.
    // In C, you cannot access a global if a local hides it.
    func();
    return 0;
}
```
**Code Breakdown:**
- `global` at file scope is accessible in `func` and `main`, but `main` declares a local `global` that hides it.
- Shadowing is allowed but considered bad practice – it can confuse.
- Within `func`, the inner block declares a new `local` that shadows the outer `local`; after the block, the outer one is visible again.

### Example 2: Linkage – `static` vs `extern`
```c
// fileA.c
#include <stdio.h>

int shared = 42;          // external linkage – visible to other files
static int hidden = 100;  // internal linkage – only visible in fileA.c

void print_shared(void) {
    printf("shared = %d\n", shared);
}

// fileB.c
#include <stdio.h>

extern int shared;        // declaration, not definition – refers to fileA's shared
// extern int hidden;     // would cause linker error because hidden is static

int main(void) {
    printf("shared from fileB: %d\n", shared);
    return 0;
}
```
**Code Breakdown:**
- `shared` has **external linkage** – any file can access it if declared `extern`.
- `hidden` has **internal linkage** (`static`) – it is private to fileA.c.
- The linker resolves `shared` to the definition in fileA.c.

### Example 3: Reserved Names and Implementation
```c
#include <stdio.h>

// Names starting with underscore are reserved for the implementation
// in certain contexts. It's safe to use _my_var at file scope, but
// avoid names starting with two underscores or underscore + capital letter.

int _my_var = 10;      // fine, but not recommended for portability
// int _Atomic = 20;   // _Atomic is a keyword in C11+ – reserved.

int main(void) {
    printf("%d\n", _my_var);
    return 0;
}
```
**Code Breakdown:**
- The C standard reserves identifiers beginning with an underscore for use by the implementation (compiler/library). Using them can cause name clashes. It’s safer to avoid them in user code.

### Example 4: Identifiers in Structure Members
```c
#include <stdio.h>

struct Data {
    int value;      // member identifier
    int value;      // error – duplicate member name within the same struct
};

struct Another {
    int value;      // this is in a different name space – allowed
};

int main(void) {
    struct Data d1 = {10};
    struct Another d2 = {20};
    printf("%d %d\n", d1.value, d2.value); // both are accessible
    return 0;
}
```
**Code Breakdown:**
- Member names are local to their structure; they do not conflict with other structures or with ordinary identifiers.

---

## 10. Common Use Cases

- **Variable names** – store data.
- **Function names** – encapsulate logic.
- **Type names** (via `typedef`) – create short, meaningful aliases.
- **Structure/union/enum tags** – name composite types.
- **Labels** – used with `goto` (rare).
- **Preprocessor macros** – not C identifiers per se (handled by preprocessor), but they follow similar naming rules.

---

## 11. Best Practices

- **Use meaningful names** – avoid single‑letter names (except loop counters) and cryptic abbreviations.
- **Follow a consistent naming convention** – e.g., `snake_case` for variables/functions, `CamelCase` for types, `ALL_CAPS` for macros.
- **Avoid names that start with underscore** – to prevent collisions with compiler‑reserved names.
- **Avoid shadowing** – do not reuse names in nested scopes; it leads to confusion.
- **Keep names reasonably short** – but not at the cost of clarity.
- **Use `static` to limit scope** – prevents name clashes with other translation units.
- **Prefix or suffix global identifiers** – e.g., `g_count` to distinguish globals.

---

## 12. Common Mistakes

### Mistake 1: Starting an Identifier with a Digit
```c
// ❌ Wrong – invalid identifier
int 1st_number = 5;
// ✅ Correct
int first_number = 5;
```

### Mistake 2: Using a Keyword as an Identifier
```c
// ❌ Wrong
int int = 10;   // int is a keyword
// ✅ Correct
int value = 10;
```

### Mistake 3: Shadowing with Confusing Names
```c
// ❌ Confusing – hard to know which x is used
int x = 5;
int main() {
    int x = 10;
    {
        int x = 20;
        printf("%d", x); // prints 20 – which one? ambiguous for readers
    }
}
// ✅ Better – use distinct names
```

### Mistake 4: Forgetting Case Sensitivity
```c
int Count = 10;
printf("%d", count); // error – count is not defined (different case)
```

### Mistake 5: Using Hyphens or Spaces
```c
// ❌ Wrong – not allowed
int my-var = 5;
// ✅ Correct
int my_var = 5;
```

---

## 13. Performance Considerations

- **Identifier length** does not affect runtime performance; it only affects compilation speed (slightly) and symbol table size.
- **Linkage** – `static` identifiers do not have to be exported to the symbol table, which can slightly improve linking time and reduce binary size (they are not visible outside).
- **Name mangling** – C does not mangle names, so function names are direct symbols; this is partly why C is used for system APIs (easier to call from other languages).

---

## 14. Security Considerations

- **Exposed symbol names** – in a release build, symbol names may be stripped (with `-s` or `strip`) to reduce attack surface.
- **Reserved names** – using names that the compiler/library uses can lead to subtle bugs (especially if they are macros).
- **Global identifiers** – if you use common names like `read`, `write`, they may conflict with standard library functions; avoid naming conflicts.

---

## 15. Debugging Tips

- **Use `-g`** to include debug symbols – the debugger shows the original identifier names.
- **If you get “undefined reference” errors**, check spelling, case, and linkage (`static` vs `extern`).
- **If you get “redefinition” errors**, check for multiple definitions or missing header guards.
- **Use `nm` on object files** to see exported symbols – e.g., `nm myfile.o` – `T` indicates a defined function, `U` an undefined reference.

---

## 16. When to Use

- **Always** – every variable, function, type, and label requires an identifier.
- **Choose identifiers carefully** – they are the primary means of communication in code.

---

## 17. When Not to Use

- **Avoid long, overly descriptive names** in tight loops (no performance impact, but may harm readability).
- **Avoid names that are too similar** – `count1`, `count2`, `count3` – use arrays or meaningful suffixes.

---

## 18. Related Concepts

- **Keywords** – the reserved set you cannot use.
- **Scope** – where an identifier is visible.
- **Linkage** – how identifiers connect across files.
- **Symbol tables** – compiler and linker data structures.
- **Name spaces** – separate contexts for different identifier categories.

---

## 19. Did You Know?

- The C standard guarantees that identifiers are case‑sensitive; this differs from some older languages like FORTRAN (case‑insensitive).
- Some compilers treat identifiers with leading underscores differently – e.g., `_` prefix is often used for system internals.
- In C, you can have an identifier that is a keyword in another language – e.g., `class` is not a keyword in C, so you can use it (though not recommended).
- The maximum length of an external identifier is implementation‑defined; the standard recommends 31 characters for external, but many compilers support 255+.

---

## 20. Summary

- **Identifiers are user‑defined names** for program elements – variables, functions, types, etc.
- **They must start with a letter or underscore**; only letters, digits, and underscores are allowed.
- **C is case‑sensitive** – `foo`, `Foo`, `FOO` are distinct.
- **Identifiers have scope, linkage, and lifetime** – these determine visibility, sharing, and storage duration.
- **Best practices**: use meaningful names, avoid leading underscores, avoid shadowing, and use `static` to limit visibility.
- **Common mistakes**: using keywords, starting with digits, using special characters (hyphens, spaces).
- **Understanding identifiers** is fundamental to writing readable, maintainable, and correct C code.