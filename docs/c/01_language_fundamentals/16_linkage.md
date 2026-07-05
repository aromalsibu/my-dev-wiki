# Linkage

## 1. Overview

### Definition
Linkage in C determines whether an identifier (variable, function, or type) refers to the same entity across different translation units (source files) or within the same translation unit. It controls how identifiers are connected (or "linked") during the linking phase of compilation. C defines three types of linkage: **external**, **internal**, and **none**.

### Purpose
Linkage is essential for:
- **Modular programming** – sharing functions and variables across multiple source files.
- **Encapsulation** – hiding implementation details from other files.
- **Avoiding name conflicts** – preventing duplicate symbols at link time.
- **Controlling visibility** – deciding which parts of your program are public or private.

### Where It Fits
Linkage works alongside:
- **Scope** – where an identifier is visible.
- **Lifetime** – how long a variable exists.
- **Storage classes** – `static`, `extern`, which control linkage.
- **Translation units** – each `.c` file (after preprocessing) is a separate translation unit.
- **Linker** – resolves external references during the linking stage.

---

## 2. Why It Exists

### The Problem Without Linkage Rules
Without linkage rules, every identifier would be either:
- **Globally visible** – causing name clashes across files.
- **Completely private** – making it impossible to share code across files.
- **Ambiguous** – leading to multiple definition errors or unresolved symbols.

### The Solution: Three Levels of Linkage
C provides fine-grained control:
- **External linkage** – identifier is visible across all translation units.
- **Internal linkage** – identifier is visible only within its own translation unit.
- **No linkage** – identifier is visible only within its own scope.

This allows you to:
- Share public APIs (external linkage).
- Hide implementation details (internal linkage with `static`).
- Create local variables that don't interfere with other code (no linkage).

### Why It Was Introduced
Linkage rules were fundamental to C's design for supporting modular programming. They were present from the earliest versions, enabling the development of large systems like Unix by allowing multiple source files to be compiled separately and linked together.

---

## 3. Syntax / Basic Usage

### External Linkage (Default for File-Scope Variables and Functions)
```c
// file1.c
#include <stdio.h>

// External linkage – default for file-scope variables.
int global_count = 10;        // visible to other files

// External linkage – default for functions.
void print_global(void) {
    printf("global_count: %d\n", global_count);
}

// file2.c
#include <stdio.h>

// Declaration – refers to global_count in file1.c.
extern int global_count;

// Declaration – refers to print_global in file1.c.
extern void print_global(void);

int main(void) {
    print_global();           // calls function from file1.c
    printf("global_count: %d\n", global_count);
    return 0;
}
```

### Internal Linkage (Using `static`)
```c
// file1.c
#include <stdio.h>

// Internal linkage – only visible in file1.c.
static int private_count = 20;

// Internal linkage – only visible in file1.c.
static void private_helper(void) {
    printf("private_count: %d\n", private_count);
}

// External linkage – visible to other files.
void public_function(void) {
    private_helper();         // can call internal function
    private_count++;          // can modify internal variable
}

// file2.c
#include <stdio.h>

// ❌ Error – private_count is static in file1.c, not visible here.
// extern int private_count;

// ❌ Error – private_helper is static in file1.c, not visible here.
// extern void private_helper(void);

// ✅ OK – public_function has external linkage.
extern void public_function(void);

int main(void) {
    public_function();        // works
    return 0;
}
```

### No Linkage (Automatic Variables)
```c
#include <stdio.h>

int main(void) {
    // No linkage – only visible within main.
    int local = 10;

    {
        // No linkage – only visible within this block.
        int inner = 20;
        printf("inner: %d\n", inner);
    }
    // inner is not visible here (different scope).

    // ❌ Error – cannot reference local across files (obviously).
    return 0;
}
```

### Code Breakdown (with comments)
```c
// file1.c
#include <stdio.h>

// External linkage (default for file-scope identifiers).
// Visible to all translation units.
int global_count = 10;

// Internal linkage – 'static' limits visibility to this file.
// Only file1.c can access private_count.
static int private_count = 20;

// Internal linkage – helper function not visible outside file1.c.
static void private_helper(void) {
    // private_count is accessible here (same file).
    private_count++;
}

// External linkage – public function visible to other files.
void public_function(void) {
    // Can call internal helper function.
    private_helper();

    // Can access internal variable.
    printf("private_count: %d\n", private_count);
}

// file2.c
#include <stdio.h>

// Declaration with extern – tells compiler that global_count
// is defined in another translation unit.
extern int global_count;

// Declaration – public_function is defined elsewhere.
extern void public_function(void);

int main(void) {
    // Access external variables and functions.
    printf("global_count: %d\n", global_count);
    public_function();

    // ❌ Cannot access private_count – it's static in file1.c.
    // printf("%d\n", private_count); // Compilation error.

    // Local variable – no linkage.
    int local = 5;

    return 0;
}
```

---

## 4. Mental Model – Linkage as Communication Channels

Imagine your program as a building with multiple rooms (translation units):

- **External linkage** – like a **public announcement system**. Anyone in any room can hear the announcement. If you define something with external linkage, it's like putting a message on a central bulletin board that everyone can see.

- **Internal linkage** – like a **private memo** that only circulates within one room. People in other rooms cannot see it. This is achieved with `static` at file scope.

- **No linkage** – like a **personal note** that only one person can see, and only in a specific location. Local variables have no linkage.

The **linker** is like the building manager who ensures that when someone says "global_count", they are all referring to the same thing. If you have two things with the same name and external linkage, the linker will complain (multiple definition error). If you have `static`, the linker ignores it – it's not on the public channel.

---

## 5. Core Concepts

### Summary of Linkage Types

| Linkage | Visibility | Examples | Storage Class |
|---------|------------|----------|---------------|
| **External** | All translation units | File-scope variables (default), functions (default) | `extern` (explicit), or no specifier |
| **Internal** | Current translation unit only | File-scope variables/functions with `static` | `static` |
| **None** | Current scope only | Local variables, function parameters | `auto` (default), `register` |

### Key Points
- **External linkage** is the default for:
  - File-scope variables (not `static`).
  - Functions (not `static`).
  - Enum constants (they have external linkage in C).
  
- **Internal linkage** is achieved with:
  - The `static` keyword at file scope.
  - It limits visibility to the current translation unit.

- **No linkage** applies to:
  - Local variables (automatic storage).
  - Function parameters.
  - Block-scope variables (even with `static`, they have no linkage but static lifetime).

### Linkage and Storage Classes

| Storage Class | File Scope | Block Scope |
|---------------|------------|-------------|
| **Default** | External | No linkage |
| **`static`** | Internal | No linkage (but static lifetime) |
| **`extern`** | External (declaration) | External (if already defined) |
| **`auto`** | N/A (not allowed) | No linkage |
| **`register`** | N/A (not allowed) | No linkage |

### Important Rule
- An identifier with **internal linkage** in one translation unit is completely separate from an identifier with the same name in another translation unit (they are distinct).
- An identifier with **external linkage** refers to the **same entity** across all translation units.

---

## 6. How It Works – Linker and Symbol Resolution

### Compilation Phase
1. Each translation unit (`.c` file) is compiled independently.
2. The compiler generates an object file (`.o` or `.obj`) containing:
   - **Machine code** – for functions.
   - **Data** – for variables.
   - **Symbol table** – listing all identifiers with their linkage and addresses.
3. Symbols with **external linkage** are marked as "global" in the symbol table.
4. Symbols with **internal linkage** are marked as "local" (not exported).

### Linking Phase
1. The linker takes all object files and libraries.
2. It collects all **global symbols** (external linkage) from each object file.
3. It resolves **references** to these symbols:
   - If a symbol is defined in one file and referenced in another, the linker connects them.
   - If a symbol is defined in multiple files, the linker reports a **multiple definition error**.
   - If a symbol is referenced but not defined anywhere, the linker reports an **undefined reference error**.
4. Internal symbols (`static`) are not visible to the linker – they are resolved within the object file itself.

### ASCII Diagram – Linkage and Symbol Resolution
```
file1.c (compiled)          file2.c (compiled)
       │                           │
       ▼                           ▼
   file1.o                    file2.o
   ┌──────────────┐          ┌──────────────┐
   │ global_count │          │ extern       │
   │ (external)   │          │ global_count │
   │ public_func  │──────────│ (reference)  │
   │ (external)   │          │              │
   │ private_count│          │              │
   │ (internal)   │          └──────────────┘
   │ private_func │
   │ (internal)   │
   └──────────────┘
          │
          ▼
      Linker
          │
          ▼
   Executable (resolved)
   ┌──────────────┐
   │ global_count │  ← same address
   │ public_func  │  ← same address
   │ private_count│  ← internal to file1
   │ private_func │  ← internal to file1
   └──────────────┘
```

---

## 7. Internal Architecture – Symbol Tables

### Compiler Symbol Table (per Translation Unit)
```
Symbol Table (file1.o):
┌───────────────────┬──────────────┬──────────────┐
│ Symbol Name       │ Type         │ Linkage      │
├───────────────────┼──────────────┼──────────────┤
│ global_count      │ Variable     │ External     │
│ public_function   │ Function     │ External     │
│ private_count     │ Variable     │ Internal     │
│ private_helper    │ Function     │ Internal     │
└───────────────────┴──────────────┴──────────────┘
```

### Linker Symbol Table (after linking)
```
Global Symbol Table:
┌───────────────────┬──────────────┬──────────────┐
│ Symbol Name       │ Defined In   │ Address      │
├───────────────────┼──────────────┼──────────────┤
│ global_count      │ file1.o      │ 0x1000       │
│ public_function   │ file1.o      │ 0x2000       │
└───────────────────┴──────────────┴──────────────┘
```

### Multiple Definition Error Example
```c
// file1.c
int count = 10;       // external linkage

// file2.c
int count = 20;       // ❌ Error: multiple definitions of 'count'
```
**Why it's wrong:** Both definitions have external linkage. The linker sees two definitions for the same symbol and reports an error.

### Fix: Use `static` or `extern`
```c
// file1.c
int count = 10;       // external linkage (definition)

// file2.c
extern int count;     // declaration (not a definition) – OK
```

---

## 8. Lifecycle / Workflow of Linkage

### For External Linkage
1. **Declaration** – appears in source (file scope, no `static`).
2. **Compilation** – symbol is exported to the object file as a global symbol.
3. **Linking** – the linker resolves references to this symbol.
4. **Runtime** – all references point to the same memory location.

### For Internal Linkage
1. **Declaration** – appears in source with `static` at file scope.
2. **Compilation** – symbol is marked as local; not exported.
3. **Linking** – linker does not see this symbol.
4. **Runtime** – only visible within the translation unit.

### For No Linkage
1. **Declaration** – appears inside a block.
2. **Compilation** – symbol is not recorded in the object file's symbol table (only stack offsets are generated).
3. **Linking** – irrelevant; it's resolved at compile time to a stack location.
4. **Runtime** – exists only on the stack.

---

## 9. Practical Examples

### Example 1: Sharing Variables Across Files
**config.h**
```c
#ifndef CONFIG_H
#define CONFIG_H

// Declaration – tells other files that DEBUG exists.
extern int DEBUG;

#endif
```

**config.c**
```c
#include "config.h"

// Definition – allocates storage.
int DEBUG = 1;
```

**main.c**
```c
#include "config.h"
#include <stdio.h>

int main(void) {
    // DEBUG is accessible because of the extern declaration in config.h.
    printf("Debug mode: %d\n", DEBUG);
    DEBUG = 0;
    printf("Debug mode now: %d\n", DEBUG);
    return 0;
}
```
**Code Breakdown:**
- `config.h` declares `DEBUG` as `extern` – a reference.
- `config.c` defines `DEBUG` – allocates storage.
- `main.c` includes `config.h` and can access `DEBUG`.
- This is the standard pattern for global configuration variables.

### Example 2: Functions with External and Internal Linkage
**helper.h**
```c
#ifndef HELPER_H
#define HELPER_H

// External linkage – visible to other files.
void public_helper(void);

#endif
```

**helper.c**
```c
#include <stdio.h>
#include "helper.h"

// Internal linkage – only visible in helper.c.
static void private_helper(void) {
    printf("Private helper called\n");
}

// External linkage – visible to other files.
void public_helper(void) {
    printf("Public helper called\n");
    private_helper();   // calls internal function
}
```

**main.c**
```c
#include "helper.h"
#include <stdio.h>

int main(void) {
    public_helper();    // ✅ OK – public function
    // private_helper(); // ❌ Error – static function not visible
    return 0;
}
```
**Code Breakdown:**
- `public_helper` has external linkage; it's declared in `helper.h` and defined in `helper.c`.
- `private_helper` has internal linkage (`static`); it's only visible within `helper.c`.
- This pattern encapsulates implementation details.

### Example 3: `static` vs Global Variables
```c
// file1.c
#include <stdio.h>

int global = 10;               // external linkage
static int file_static = 20;   // internal linkage

void print_values(void) {
    printf("global: %d, file_static: %d\n", global, file_static);
}

// file2.c
#include <stdio.h>

extern int global;             // refers to global in file1.c
// extern int file_static;     // ❌ Error – static in file1.c

void modify_global(void) {
    global = 100;
    // file_static = 200;      // ❌ Error – not accessible
}

int main(void) {
    print_values();            // from file1.c: 10, 20
    modify_global();
    print_values();            // from file1.c: 100, 20
    return 0;
}
```
**Code Breakdown:**
- `global` is shared across files.
- `file_static` is private to file1.c.
- `modify_global` can change `global` because it has external linkage.

### Example 4: Linkage of Enum Constants
```c
// file1.c
#include <stdio.h>

enum Color { RED, GREEN, BLUE };   // Enum constants have external linkage.

void print_color(void) {
    printf("RED: %d\n", RED);      // RED is accessible.
}

// file2.c
#include <stdio.h>

// Enum constants have external linkage – they are visible across files.
// No need for extern declaration (they are not variables).

int main(void) {
    printf("GREEN: %d\n", GREEN);   // GREEN is accessible!
    return 0;
}
```
**Code Breakdown:**
- Enumeration constants have **external linkage** in C.
- They are visible across translation units without requiring `extern` declarations.
- This is different from variables – enum constants are compile‑time constants.

### Example 5: `extern` with Functions
```c
// calculator.h
#ifndef CALCULATOR_H
#define CALCULATOR_H

// Function declarations – by default, they have external linkage.
int add(int a, int b);
int subtract(int a, int b);

#endif
```

**calculator.c**
```c
#include "calculator.h"

// Definitions – external linkage by default.
int add(int a, int b) {
    return a + b;
}

int subtract(int a, int b) {
    return a - b;
}
```

**main.c**
```c
#include "calculator.h"
#include <stdio.h>

// We could also explicitly declare with extern (optional).
// extern int add(int, int);

int main(void) {
    printf("Add: %d\n", add(5, 3));        // 8
    printf("Subtract: %d\n", subtract(5, 3)); // 2
    return 0;
}
```
**Code Breakdown:**
- Functions have external linkage by default.
- The `extern` keyword is optional for functions (unlike variables, where you need it for declarations).
- `calculator.h` declares the functions; `calculator.c` defines them.

### Example 6: `static` Functions in Headers (Anti-pattern)
```c
// utils.h
#ifndef UTILS_H
#define UTILS_H

// ❌ Wrong – static function in header.
static void helper(void) {
    // This function is defined in every translation unit that includes this header.
    printf("Helper\n");
}

#endif

// file1.c
#include "utils.h"

void func1(void) {
    helper();   // calls file1.c's copy of helper
}

// file2.c
#include "utils.h"

void func2(void) {
    helper();   // calls file2.c's copy of helper (different function!)
}
```
**Code Breakdown:**
- `static` in a header means each `.c` file gets its own copy of the function.
- This wastes space and can lead to confusing behaviour if the function has static locals.
- **Fix:** Use `inline` or put the function in a `.c` file with external linkage.

---

## 10. Common Use Cases

| Scenario | Linkage | Example |
|----------|---------|---------|
| **Public API functions** | External | `int process_data(int *data, size_t n);` |
| **Global configuration** | External | `extern int debug_level;` (defined in one .c) |
| **File‑private helpers** | Internal | `static void internal_helper(void);` |
| **File‑private data** | Internal | `static int cache[100];` |
| **Local variables** | None | `int x;` inside a function |
| **Constants shared across files** | External (enum) | `enum { MAX = 100 };` |
| **Constants private to a file** | Internal | `static const int MAX = 100;` |

---

## 11. Best Practices

### General
- **Use `static` at file scope** – for functions and variables that are not part of your public API. This is a form of encapsulation and prevents name conflicts.
- **Use `extern` in header files** – to declare variables defined in a `.c` file. This ensures consistency across files.
- **Define variables in exactly one `.c` file** – to avoid multiple definition errors.
- **Use `static const` for file‑private constants** – instead of `#define`, for type safety and scope control.
- **Avoid `static` functions in headers** – each translation unit gets its own copy; use `inline` or move to a `.c` file.

### Variables
- **Prefer `extern` declarations in headers** – never define variables in headers (unless they're `static const` for file‑private use).
- **Use `static` for variables that should be private to the file** – prevents accidental external access.

### Functions
- **Functions are external by default** – if you need a private function, mark it `static`.
- **Declare functions in headers** – and define them in `.c` files.
- **Use `static` for helper functions** – that are only used within a single `.c` file.

---

## 12. Common Mistakes

### Mistake 1: Defining a Variable in a Header
```c
// header.h
#ifndef HEADER_H
#define HEADER_H
int global = 10;   // ❌ Wrong – multiple definitions if included in multiple files.
#endif

// ✅ Correct – declare in header, define in .c.
// header.h
extern int global;

// file.c
int global = 10;
```

### Mistake 2: Forgetting `extern` for Variables
```c
// file1.c
int shared = 10;   // definition

// file2.c
int shared = 20;   // ❌ Error – multiple definitions

// ✅ Correct – use extern in file2.c
extern int shared;   // declaration
```

### Mistake 3: Using `static` in a Header (Accidental Copies)
```c
// header.h
static int count = 0;   // ❌ Each file gets its own copy!

// ✅ Correct – use extern in header, define in one .c.
extern int count;
```

### Mistake 4: Not Understanding Enum Constants Have External Linkage
```c
// file1.c
enum { MAX = 100 };   // MAX has external linkage

// file2.c
// You can access MAX without any declaration!
int arr[MAX];   // OK – MAX is visible.
```
**Why it matters:** If you want a file‑private enum constant, use `static enum { MAX = 100 };` or `static const int MAX = 100;`.

### Mistake 5: Redeclaring `static` as `extern`
```c
// file1.c
static int x = 10;

// file2.c
extern int x;   // ❌ Error – x is static in file1.c, not visible.
```
**Why it's wrong:** `static` gives internal linkage; `extern` cannot override that.

### Mistake 6: Assuming `static` Functions Are Not in the Object File
```c
// static functions are still in the object file, but they are not exported.
// The linker ignores them when resolving external references.
static void helper(void) { ... }   // in object file, but marked local.
```

---

## 13. Performance Considerations

- **No runtime performance difference** – linkage affects compile/link time, not runtime.
- **Internal linkage** – may slightly reduce link time (fewer symbols to process).
- **External linkage** – requires symbol resolution at link time; negligible overhead.
- **`static` in headers** – can increase code size (multiple copies) and cache pressure.
- **Inline functions** – often used for performance, but have special linkage rules (C99).

---

## 14. Security Considerations

- **`static`** – limits visibility, reducing the attack surface (symbols not exposed).
- **External linkage** – symbols can be referenced from anywhere; be careful with sensitive data.
- **Multiple definitions** – can lead to linker errors, but also to subtle bugs if `static` is misused.
- **Enum constants** – external linkage means they are globally visible; not a security issue, but good to know.

---

## 15. Debugging Tips

- **Use `nm`** – to inspect symbols in object files:
  ```bash
  nm file.o
  ```
  - `T` – function with external linkage.
  - `t` – function with internal linkage (`static`).
  - `D` – data with external linkage.
  - `d` – data with internal linkage.

- **Use `objdump`** – to see symbols and sections.

- **Linker errors**:
  - "undefined reference" – you've declared something with `extern` but haven't defined it.
  - "multiple definition" – you've defined the same symbol in multiple files (probably a header with a definition).

- **Use `-Wall -Wextra`** – catches some linkage‑related issues.

---

## 16. When to Use

- **External linkage** – for public APIs (functions and global variables that need to be shared).
- **Internal linkage** – for implementation‑private functions and variables (use `static`).
- **No linkage** – for local variables (default).

---

## 17. When Not to Use

- **Avoid external linkage** for variables that should be file‑private – use `static`.
- **Avoid `static` in headers** – it creates multiple copies; use `extern` for declarations.
- **Avoid defining variables in headers** – unless they are `static const` (file‑private).

---

## 18. Related Concepts

- **Scope** – where an identifier is visible.
- **Lifetime** – how long a variable exists.
- **Storage classes** – `auto`, `static`, `extern`, `register`, `_Thread_local`.
- **Translation units** – the unit of compilation.
- **Linker** – resolves external references.
- **Symbol tables** – compiler and linker data structures.
- **Header files** – where external declarations typically reside.
- **Multiple definition errors** – caused by duplicate external symbols.

---

## 19. Did You Know?

- In C, `extern` is optional for functions – they have external linkage by default.
- Enumeration constants have **external linkage** in C (unlike C++, where they have no linkage).
- A file‑scope variable without `static` or `extern` has external linkage.
- `static` inside a function means "static lifetime", not "internal linkage" – it still has no linkage.
- C23 introduces `constexpr`, but it doesn't change linkage rules.
- The linker doesn't see identifiers with no linkage – they are resolved at compile time.

---

## 20. Summary

- **Linkage determines visibility** of identifiers across translation units.
- **Three types**: external, internal, and none.
- **External linkage** – visible across all files (default for file‑scope variables and functions).
- **Internal linkage** – visible only within the current file (achieved with `static` at file scope).
- **No linkage** – visible only within the current scope (local variables).
- **Best practices**: use `static` for file‑private data and functions, use `extern` in headers for shared globals, define variables in exactly one `.c` file.
- **Common mistakes**: defining variables in headers, using `static` in headers (multiple copies), forgetting `extern` for variables.
- **Understanding linkage** is essential for modular programming, avoiding name conflicts, and correctly organising multi‑file C projects.