# Program Structure

## 1. Overview

### Definition
Program structure in C refers to the fundamental anatomy of a C source file—the rules for organising declarations, definitions, preprocessor directives, and the mandatory entry point (`main`). It is the blueprint that dictates how a C program is laid out, compiled, and executed.

### Purpose
Mastering program structure ensures that your code compiles without errors, behaves predictably, and is readable by other engineers. It establishes a consistent pattern for every C project, from a single‑file utility to a multi‑module operating system component.

### Where It Fits
Program structure is the first concrete concept you apply after learning the compilation model. It sits at the intersection of:
- **Preprocessor directives** – inclusion of headers and macro definitions.
- **Global declarations** – external variables, function prototypes, and type definitions.
- **Function definitions** – the implementation of logic, culminating in `main`.
- **Program lifetime** – startup, execution, and termination.

---

## 2. Why It Exists

### The Problem Without a Standard Structure
Before C formalised a layout, early system software was often written in ad‑hoc assembly with no consistent entry point or clear separation of declarations and definitions. This led to:
- Unpredictable program startup.
- Confusion about which functions were externally visible.
- Frequent linker errors due to missing or duplicated symbols.

### The Solution: The C Program Blueprint
C introduced a **mandatory `main` function** as the entry point, and a **flexible but idiomatic ordering** of sections that ensures:
- All types and functions are declared before use (enabling proper type checking).
- Global resources are initialised predictably.
- The linker can resolve external references without ambiguity.
- Developers can quickly understand the high‑level organisation of any C project.

### Why It Was Introduced
This structure was designed alongside Unix to provide a uniform interface between the operating system and user programs. The `main` function’s signature (`int main(void)` or `int main(int argc, char *argv[])`) gives the OS a standard way to pass command‑line arguments and receive an exit code, making every C program interoperable with the shell.

---

## 3. Syntax / Basic Usage

The canonical structure of a C source file follows this order:

```c
// 1. Preprocessor directives (includes and macros)
#include <stdio.h>      // Include standard library headers
#include "my_header.h"  // Include project‑specific headers
#define MAX_SIZE 1024   // Macro definitions

// 2. Type declarations (structs, unions, enums, typedefs)
typedef struct {
    int id;
    char name[50];
} User;

// 3. Global variable declarations (with optional initialisation)
int global_counter = 0;          // Global with external linkage
static int file_scope_counter;   // Static global – only visible in this file

// 4. Function prototypes (declarations)
void process_user(User *u);      // Declare function before use

// 5. main() – the entry point
int main(void) {
    User alice = {1, "Alice"};
    process_user(&alice);
    return 0;                    // Success exit code
}

// 6. Function definitions (implementations)
void process_user(User *u) {
    printf("Processing user: %s (ID: %d)\n", u->name, u->id);
}
```

### Code Breakdown
```c
#include <stdio.h>      
// The preprocessor copies the entire stdio.h header here.
// This gives us access to printf, fprintf, etc.

#include "my_header.h"   
// Quotes indicate the current directory; used for project headers.
// Ensures declarations from other parts of the project are visible.

#define MAX_SIZE 1024   
// Textual substitution; every occurrence of MAX_SIZE becomes 1024.
// No semicolon – it’s a directive, not a statement.

typedef struct { ... } User;
// Defines a new type alias so we can write `User` instead of `struct User`.
// Placed before functions that use it.

int global_counter = 0;
// A global variable accessible from any translation unit (unless static).
// Initialised to 0 before main runs.

static int file_scope_counter;
// Static global – only visible within this file. Helps avoid name collisions.

void process_user(User *u);
// A function prototype (declaration) – tells the compiler:
// "There is a function called process_user that takes a User* and returns void."
// Without this, the compiler would assume an implicit int return.

int main(void) {
// The mandatory entry point. `void` means no arguments.
// The OS calls this function when the program starts.
// It must return an int – 0 for success, non‑zero for failure.
    User alice = {1, "Alice"};
    // Declare and initialise a local variable (automatic storage).
    // The struct initialiser sets id=1, name[0]='A', etc.

    process_user(&alice);
    // Call the function we defined later – prototype allows this.

    return 0;
    // Return 0 to the OS, indicating successful termination.
    // The exit status can be checked in a shell (echo $?).
}

void process_user(User *u) {
// Implementation of the function that was prototyped above.
// The definition must match the prototype exactly.
    printf("Processing user: %s (ID: %d)\n", u->name, u->id);
    // printf is from stdio.h. %s for string, %d for integer.
}
```

---

## 4. Mental Model – The Program as a House

Think of a C program’s structure as the blueprint for building a house:

- **Preprocessor directives** – The permits and building codes (`#include` brings in external rules, `#define` sets material dimensions).
- **Type definitions** – The architectural plans for rooms (structs) and tools (enums).
- **Global declarations** – The foundations and infrastructure (visible to all rooms).
- **`main` function** – The front door. The building inspector (OS) enters here and expects to exit through the same door with a sign (return code).
- **Other function definitions** – The rooms themselves; each has a door (prototype) and a purpose (implementation). They can be built later in the blueprint, but their doors must be shown earlier so the rest of the house knows how to enter.

---

## 5. Core Concepts

| Concept | Explanation |
|---------|-------------|
| **Translation unit** | A `.c` file plus all included headers after preprocessing; the compiler processes exactly one translation unit at a time. |
| **External vs static linkage** | `extern` (default) makes symbols visible across translation units; `static` restricts visibility to the current file. |
| **Prototypes** | Declare a function’s signature before its definition, enabling the compiler to perform type‑checking at call sites. |
| **`main` signatures** | `int main(void)` or `int main(int argc, char *argv[])`. The latter receives command‑line arguments. C23 also allows `int main(int argc, char *argv[], char *envp[])` as a non‑standard but common extension. |
| **Exit codes** | `return` from `main` is equivalent to calling `exit(return_value)`. Zero conventionally means success. |
| **Startup and cleanup** | Before `main` runs, the runtime initialises static and global variables. After `main` returns, the runtime calls `atexit` handlers and flushes output buffers. |

---

## 6. How It Works – Step‑by‑Step Execution Flow

1. **The OS loads the executable** into memory.
2. **The C runtime (CRT) startup code** runs:
   - Sets up the stack and heap.
   - Initialises `argc`, `argv`, and `envp`.
   - Calls constructors for global objects (C++ only; C has no constructors, but static data is zero‑initialised).
3. **`main` is called** with the appropriate arguments.
4. **Your code executes** sequentially inside `main` and any functions it calls.
5. **`main` returns** an integer exit code.
6. **The CRT shutdown code** runs:
   - Calls functions registered with `atexit` (in reverse order).
   - Flushes `stdout` and `stderr`.
   - Returns control to the OS with the exit code.

### ASCII Diagram – Program Lifecycle
```
OS Loader → CRT Startup → main() → ... → return → CRT Shutdown → OS
                 │                           │
                 ├── init .data/.bss         └── atexit calls
                 ├── prepare argc/argv           flush buffers
                 └── call main                   return exit code
```

---

## 7. Internal Architecture – The Sections in Memory

A compiled C program typically contains these sections, arranged by the linker:

```
┌─────────────────────────────────────┐
│          .text (code)               │  ← Machine instructions
├─────────────────────────────────────┤
│          .rodata (read‑only data)   │  ← String literals, const data
├─────────────────────────────────────┤
│          .data (initialised data)   │  ← Global/static with init values
├─────────────────────────────────────┤
│          .bss (uninitialised data)  │  ← Zero‑initialised globals/statics
├─────────────────────────────────────┤
│          heap (grows upward)        │  ← malloc/free
├─────────────────────────────────────┤
│          stack (grows downward)     │  ← Local variables, call frames
└─────────────────────────────────────┘
```

- **.text** – where the code of `main` and all functions resides.
- **.rodata** – stores `"Processing user..."` string literals.
- **.data** – holds `global_counter = 0`.
- **.bss** – holds `file_scope_counter` (zero‑initialised).
- **Stack** – stores `alice`, `argc`, `argv`, and return addresses.
- **Heap** – used if we call `malloc` (not shown here).

---

## 8. Lifecycle / Workflow of a Source File

When you write a C source file, you follow this natural progression:

1. **Write includes** – bring in external declarations.
2. **Define constants and macros** – configure behaviour.
3. **Declare types** – structs, enums, typedefs.
4. **Declare global variables** – with care, favouring `static` when possible.
5. **Write prototypes** – so functions can call each other.
6. **Implement `main`** – the core logic, often at the top for readability.
7. **Implement helper functions** – placed after `main` or in separate files.

This order minimises forward references and makes the file self‑documenting.

---

## 9. Practical Examples

### Example 1: Program with Command‑Line Arguments
```c
#include <stdio.h>      // For printf and fprintf
#include <stdlib.h>     // For atoi

int main(int argc, char *argv[]) {
    // argc: number of arguments including the program name.
    // argv: array of C‑strings; argv[0] is the program name itself.

    if (argc < 2) {
        // Print error message to stderr (standard error stream).
        fprintf(stderr, "Usage: %s <number>\n", argv[0]);
        return 1;        // Non‑zero exit code indicates failure.
    }

    // Convert the first argument (argv[1]) to an integer.
    int number = atoi(argv[1]);  // atoi is unsafe; use strtol for production.

    printf("You entered: %d\n", number);
    return 0;            // Success.
}
```
**Code Breakdown:**
- `argc` is at least 1 (the program name). If less than 2, we show usage.
- `argv[0]` is the program name; we use it in the error message.
- `fprintf(stderr, ...)` prints to the error stream, which is typically displayed but can be redirected separately.
- `return 1` indicates failure to the OS.
- `atoi` converts a string to an integer; it returns 0 on error, so in production we would use `strtol` for proper validation.

### Example 2: Multi‑File Structure (Production Style)
**main.c**
```c
#include <stdio.h>
#include "calculator.h"   // Declares add, subtract, etc.

int main(void) {
    int a = 10, b = 5;
    printf("Add: %d\n", add(a, b));
    printf("Sub: %d\n", subtract(a, b));
    return 0;
}
```

**calculator.h**
```c
#ifndef CALCULATOR_H
#define CALCULATOR_H

int add(int x, int y);      // Prototype
int subtract(int x, int y);

#endif
```

**calculator.c**
```c
#include "calculator.h"

int add(int x, int y) {
    return x + y;
}

int subtract(int x, int y) {
    return x - y;
}
```
**Code Breakdown:**
- Header guard prevents multiple inclusion.
- `calculator.h` declares the interface; `calculator.c` implements it.
- `main.c` includes `calculator.h`, so the compiler knows the signatures of `add` and `subtract`.
- This separation allows `calculator.c` to be compiled independently and reused in other projects.

### Example 3: Using Global Variables (with Caution)
```c
#include <stdio.h>

// Global variable with external linkage – accessible from other files.
int global_count = 0;

// Static global – only visible in this file.
static int internal_tracker = 0;

void increment(void) {
    global_count++;          // Changes persist across calls.
    internal_tracker++;
}

void report(void) {
    // Both are accessible here.
    printf("Global: %d, Internal: %d\n", global_count, internal_tracker);
}

int main(void) {
    increment();
    increment();
    report();                // Output: Global: 2, Internal: 2
    return 0;
}
```
**Code Breakdown:**
- `global_count` has **external linkage** – it can be declared `extern` in other files and shared.
- `internal_tracker` has **internal linkage** (`static`) – it cannot be used outside this file, reducing name conflicts.
- Global variables are initialised to zero by default (if not explicitly initialised, they go to `.bss`).
- Overuse of global variables makes code hard to test – prefer passing parameters.

---

## 10. Common Use Cases

- **Single‑file utilities** – structure is simple, with `main` at the top and helpers below.
- **Libraries** – expose a public header, hide implementation details with `static`.
- **Embedded firmware** – `main` is often an infinite loop that never returns.
- **Command‑line tools** – parse `argc/argv` to implement flags and subcommands.
- **Servers / daemons** – `main` initialises resources, sets up signal handlers, and enters an event loop.

---

## 11. Best Practices

- **Always use function prototypes** – place them in headers or at the top of the file to avoid implicit declarations.
- **Keep `main` lean** – delegate high‑level logic to helper functions, making tests easier.
- **Use `static` for file‑private data** – this minimises global namespace pollution and improves encapsulation.
- **Initialise variables** – uninitialised automatic variables have indeterminate values; always set them.
- **Follow a consistent order** – includes, macros, types, globals, prototypes, `main`, definitions. This makes every file predictable.
- **Document `main`'s exit codes** – define constants (e.g., `EXIT_SUCCESS`, `EXIT_FAILURE` from `stdlib.h`) instead of magic numbers.
- **Compile with `-Wall -Wextra`** – catch missing prototypes, mismatched signatures, and unreachable code.

---

## 12. Common Mistakes

### Mistake 1: Missing Prototype
```c
// ❌ Wrong – implicit declaration: the compiler assumes int func()
int main(void) {
    double result = square(5.0);   // Compiler thinks square returns int – wrong!
    return 0;
}
double square(double x) { return x * x; }
// ✅ Correct – add prototype before main
double square(double x);
```

### Mistake 2: Incorrect `main` Signature
```c
// ❌ Wrong – non‑standard in strict C89/C90 (though allowed in many compilers)
void main(void) { ... }
// ✅ Correct – always return int
int main(void) { ... }
```

### Mistake 3: Assuming Globals Are Initialised to Zero for Automatic Variables
```c
// ❌ Wrong – local variable uninitialised
int main(void) {
    int count;        // count has indeterminate value
    printf("%d", count); // Undefined behaviour
    return 0;
}
// ✅ Correct – initialise local variables
int count = 0;
```

### Mistake 4: Including Source Files Instead of Headers
```c
// ❌ Wrong – this compiles but leads to multiple definition errors
#include "helper.c"  // Do not do this!
// ✅ Correct – include headers only
#include "helper.h"
```

---

## 13. Performance Considerations

- **Global vs local** – local variables are stored on the stack and accessed faster than globals (which require indirection via the data segment). Use locals when possible.
- **Static globals** – have the same performance as non‑static globals but avoid symbol table pollution.
- **`main`'s return** – returning from `main` causes the program to exit normally; calling `exit` does the same but can be done from any function.
- **Startup overhead** – large numbers of static initialisers can increase program startup time (rarely an issue in C, but relevant for embedded).

---

## 14. Security Considerations

- **`argc/argv` injection** – never trust command‑line arguments; validate length and content.
- **Global mutable state** – if global variables are used across threads (in multi‑threaded programs), they require synchronisation.
- **Static variables in functions** – retain state between calls; this can be a hazard in reentrant or threaded contexts.
- **`main` return code** – avoid returning sensitive information in exit codes; they are visible to the caller.

---

## 15. Debugging Tips

- **`-g` flag** – compile with debugging symbols to step through `main` in GDB.
- **Breakpoint on `main`** – `gdb ./program`, `break main`, `run` to stop at the entry point.
- **Check exit codes** – in the shell, `echo $?` after running your program.
- **Use `fprintf(stderr, ...)`** for debugging messages – stderr is unbuffered and appears immediately.
- **`atexit`** – register a function to debug cleanup (e.g., print a message when the program exits).

---

## 16. When to Use

- **Every C program** – this structure is mandatory; you cannot avoid it.
- **When writing reusable libraries** – the structure becomes a public header and private implementation.
- **For embedded systems** – `main` is often declared with `void` and never returns.

---

## 17. When Not to Use

- **In header files** – header files should not contain function definitions (except `static inline` functions) or `main`.
- **For test drivers** – sometimes you write multiple `main` functions in separate test files, but only one can be linked per executable.

---

## 18. Related Concepts

- **Translation Units and Linkage** – how `main` relates to other files.
- **Preprocessor** – the role of `#include` and `#define` in the structure.
- **Startup code** – `crt0.o` and the runtime environment.
- **`exit` and `atexit`** – program termination beyond `return`.
- **Command‑line arguments** – parsing `argc/argv`.

---

## 19. Did You Know?

- The `main` function is not actually the first code executed in a C program – the startup routine (`_start` or `crt0`) runs first and then calls `main`.
- In C23, `main` can be declared with the `noreturn` specifier if it never returns.
- Some embedded systems do not have an OS, so `main` is called directly by the reset vector and must never return.
- The `argc` and `argv` parameters are not required; `int main(void)` is perfectly valid even if you do not need arguments.

---

## 20. Summary

- **Every C program must have a `main` function** – it is the entry point called by the OS.
- **The typical structure** is: preprocessor directives → type definitions → global declarations → prototypes → `main` → function definitions.
- **`main` returns an integer exit code** – `0` for success, non‑zero for failure.
- **Function prototypes** enable proper type checking and allow definitions to appear after usage.
- **Global variables** should be used sparingly; prefer local variables and pass parameters.
- **`static`** restricts visibility to the current translation unit, improving encapsulation.
- **Command‑line arguments** are accessed via `argc` and `argv` – always validate them.
- **The structure ensures portability, reliability, and maintainability** across all C platforms.