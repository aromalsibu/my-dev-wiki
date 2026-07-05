# Header Files

## 1. Overview

### Definition
A header file in C is a text file containing declarations—such as function prototypes, type definitions (structs, enums, typedefs), macro definitions, and global variable declarations—that are meant to be shared across multiple source files. Header files typically have the extension `.h` and are included into source files using the `#include` preprocessor directive. They form the interface between different translation units, enabling modular program design.

### Purpose
Header files serve several essential purposes:
- **Interface declaration** – expose the public API of a module without revealing implementation details.
- **Code reuse** – share declarations across multiple source files, avoiding duplication.
- **Type safety** – ensure consistent type definitions across translation units.
- **Modularity** – separate interface from implementation, making large projects manageable.
- **Compilation speed** – precompiled headers can speed up compilation.
- **Library distribution** – provide headers so users can compile against your library without access to source.

### Where It Fits
Header files are a key part of the C compilation model. They sit at the boundary between the preprocessor and the compiler:
- **Preprocessor** – processes `#include` directives, inserting the content of header files into the source file before compilation.
- **Compiler** – sees the combined translation unit (source file + all included headers) and compiles it.
- **Linker** – resolves external references across object files, using the declarations from headers to match definitions.

---

## 2. Why It Exists

### The Problem Without Headers
Without header files, every source file would need to repeat all declarations:
- Every `.c` file would have to copy the prototypes of every function it calls.
- Every `.c` file would have to duplicate type definitions (structs, enums, typedefs).
- Changing a type definition would require updating every source file manually.
- There would be no central place to define shared macros or constants.
- Code duplication would lead to inconsistencies and maintenance headaches.

### The Solution: Declarations in One Place
Header files centralise declarations:
- One file (the header) contains all public declarations.
- Source files include the header to access those declarations.
- Changes to the interface only require updating the header and the implementation file.
- The compiler ensures consistency because the header is included in both the implementation and the client code.

### Why They Were Introduced
Header files were part of C from its earliest days, designed to support separate compilation. In the early Unix environment, programs were built from multiple source files, and headers were essential for sharing declarations. The `#include` directive, inherited from BCPL, allowed the preprocessor to insert header content into source files, enabling a clean separation of interface and implementation.

---

## 3. Syntax / Basic Usage

### A Simple Header File
```c
// mymath.h – header file
#ifndef MYMATH_H          // Include guard – prevents multiple inclusions
#define MYMATH_H

// Function prototypes – declarations, not definitions.
int add(int a, int b);
int subtract(int a, int b);
int multiply(int a, int b);

// Macro constants
#define PI 3.14159265359

// Type definitions
typedef struct {
    double x;
    double y;
} Point;

#endif  // MYMATH_H
```

### Corresponding Implementation File
```c
// mymath.c – implementation source file
#include "mymath.h"   // Include the header to ensure consistency.

// Definitions of the functions declared in the header.
int add(int a, int b) {
    return a + b;
}

int subtract(int a, int b) {
    return a - b;
}

int multiply(int a, int b) {
    return a * b;
}
```

### Using the Header in Another Source File
```c
// main.c
#include <stdio.h>
#include "mymath.h"   // Include custom header.

int main(void) {
    int sum = add(5, 3);
    int diff = subtract(5, 3);
    int prod = multiply(5, 3);
    Point p = {1.0, 2.0};

    printf("Sum: %d, Diff: %d, Prod: %d\n", sum, diff, prod);
    printf("Point: (%f, %f)\n", p.x, p.y);
    return 0;
}
```

### Using System Headers
```c
#include <stdio.h>   // Angle brackets search system directories.
#include <stdlib.h>
#include <string.h>
#include <math.h>    // Need -lm flag for linking.
```

### Using Standard Library Headers Correctly
```c
#include <stddef.h>   // For size_t, ptrdiff_t.
#include <stdbool.h>  // For bool, true, false (C99).
#include <stdint.h>   // For fixed-width integer types (C99).
#include <inttypes.h> // For printing fixed-width types.
#include <limits.h>   // For INT_MAX, etc.
#include <float.h>    // For FLT_MAX, DBL_MIN, etc.
```

### Code Breakdown (with comments)
```c
// mymath.h
#ifndef MYMATH_H   // Header guard: if MYMATH_H is not defined,
#define MYMATH_H   // define it and process the header; else skip.

// Function prototypes: declare functions but don't define them.
// These tell the compiler that these functions exist elsewhere.
int add(int a, int b);

// Macro: textual substitution.
#define PI 3.14159265359

// Type alias: defined here so all files see the same definition.
typedef struct {
    double x;
    double y;
} Point;

#endif   // End of header guard.
```

### Code Breakdown – Implementation File
```c
// mymath.c
#include "mymath.h"   // Include the header; ensures prototypes match definitions.

// Definitions: provide the actual function bodies.
int add(int a, int b) {
    return a + b;
}
// The prototype in the header matched this definition,
// so the compiler can check consistency.
```

### Code Breakdown – Main File
```c
// main.c
#include <stdio.h>    // System header with angle brackets.
#include "mymath.h"   // User header with double quotes.

// The compiler sees declarations from mymath.h and can check calls.
int main(void) {
    int sum = add(5, 3);   // add is declared in mymath.h.
    return 0;
}
```

---

## 4. Mental Model – Header Files as a Blueprint or Menu

Think of a header file as a **blueprint** or a **menu**:

- **Blueprint** – a header file is like the blueprint for a building. It shows what's available (rooms, doors, windows) without revealing the internal construction details. The implementation file (`.c`) is like the actual construction, hidden inside the walls.

- **Menu** – a header file is like a restaurant menu. It lists the dishes (functions) you can order, but you don't need to know how the chef prepares them. The menu is the interface; the kitchen is the implementation.

- **Include guard** – like a "do not duplicate" sign on the menu. If you try to copy the menu multiple times, the guard prevents confusion.

- **System headers** – like a standardised international menu (`<stdio.h>` provides common I/O "dishes" that work everywhere).

- **User headers** – like a restaurant's own specials menu, unique to that project.

When you `#include` a header, you're essentially saying: "Tell me about all the things available in this module, so I can use them correctly."

---

## 5. Core Concepts

### Header Contents
| What | Examples | Note |
|------|----------|------|
| **Function prototypes** | `int add(int a, int b);` | Declare, not define. |
| **Type definitions** | `typedef struct { ... } MyType;` | Define types for all to use. |
| **Macros** | `#define MAX 100` | Constants and function‑like macros. |
| **Global variable declarations** | `extern int global_count;` | Use `extern` to declare, not define. |
| **Inline function definitions** | `static inline int square(int x) { return x*x; }` | Safe in headers. |
| **Enumerations** | `enum Color { RED, GREEN };` | Define named constants. |
| **Structure/union definitions** | `struct Point { int x; int y; };` | Define composite types. |
| **Include directives** | `#include <stddef.h>` | Nested inclusions. |
| **Conditional compilation** | `#ifdef FEATURE_X`, `#endif` | Platform/feature‑specific code. |

### Include Guards (Header Guards)
- **Purpose** – prevent a header from being included multiple times in the same translation unit, which would cause redefinition errors.
- **Traditional**: `#ifndef HEADER_H`, `#define HEADER_H`, `#endif`.
- **Modern**: `#pragma once` (supported by most compilers) – simpler and less error‑prone, but not standard.
- **Always use include guards** for all headers.

### Angle Brackets vs Double Quotes
- `#include <header.h>` – searches system include paths (compiler‑specific, typically `/usr/include` and standard library locations).
- `#include "header.h"` – searches the current directory first, then system paths.
- For project headers, always use double quotes; for standard library, use angle brackets.

### Circular Includes
- **Problem**: header A includes header B, and header B includes header A – causes infinite recursion.
- **Solution**: use include guards and forward declarations to break the cycle.

### Forward Declarations
- Declaring a type without defining it, to break circular dependencies:
  ```c
  struct B;   // Forward declaration.
  struct A {
      struct B *b;   // Use pointer without needing full definition.
  };
  ```
- Allows using pointers to incomplete types.

### Standard Header Conventions
- System headers usually have no extension? (Actually `.h`). They are placed in system directories.
- Some compilers support `#include <stdio.h>` without the `.h` for C++ (but for C, use `.h`).

---

## 6. How It Works – The Preprocessor

### Step‑by‑Step Inclusion
1. **Preprocessor** reads the source file (`main.c`).
2. When it encounters `#include <stdio.h>`, it:
   - Locates the file `stdio.h` in the system include paths.
   - Inserts the entire content of `stdio.h` (and all its nested includes) into the source stream.
3. When it encounters `#include "mymath.h"`:
   - Searches the current directory (or the user‑specified include path) for `mymath.h`.
   - Inserts its content.
4. The `#ifndef` guard in the header prevents multiple processing if the header is included again.
5. The resulting expanded source is passed to the compiler.
6. The compiler compiles the single translation unit (`.c` + all expanded headers).

### Include Guard Mechanism
```
First inclusion:
  #ifndef MYMATH_H   -> TRUE (MYMATH_H not defined)
  #define MYMATH_H   -> define it
  ... header content ...
  #endif              -> end of guarded block.

Second inclusion:
  #ifndef MYMATH_H   -> FALSE (MYMATH_H already defined)
  ... skips to #endif ...
  (no content processed)
```

### ASCII Diagram – Preprocessing and Compilation
```
main.c
   │
   ├─ #include <stdio.h>   → inserts stdio.h content
   │       ├─ #include <stddef.h>  → etc.
   │       └─ ... (all system declarations)
   │
   ├─ #include "mymath.h"  → inserts mymath.h content
   │       ├─ #ifndef MYMATH_H  (guard)
   │       ├─ int add(...); etc.
   │       └─ #endif
   │
   └─ int main() { ... }

   ↓ (after preprocessing)
Expanded source (translation unit) → compiler → object file.
```

---

## 7. Internal Architecture – Header File Processing in the Compiler

### Preprocessor
- The preprocessor is a separate stage (or integrated) that handles `#include` directives.
- It maintains a stack of include paths and recursively expands each header.
- It expands macros, evaluates conditional directives (`#ifdef`, etc.), and handles `#pragma` directives.
- It also removes comments and performs tokenisation.

### Compiler
- The compiler receives the fully expanded source.
- It parses the declarations, builds a symbol table, and generates code.
- It doesn't "see" the header files as separate entities; only the expanded content matters.

### Linker
- The linker resolves external references (functions, global variables) across object files.
- Headers provide the declarations; the linker matches them to definitions in other object files or libraries.

### Dependencies
- When a header changes, all source files that include it (directly or indirectly) must be recompiled.
- Build systems (like Make, CMake) track these dependencies.

---

## 8. Lifecycle / Workflow of a Header File

1. **Creation** – the programmer writes a header file with declarations.
2. **Inclusion** – the header is included into one or more source files via `#include`.
3. **Preprocessing** – the header content is expanded into each translation unit.
4. **Compilation** – each translation unit (including expanded header content) is compiled into an object file.
5. **Linking** – the linker resolves external references using the declarations from the header.
6. **Distribution** – if distributing a library, the header files are shipped along with the library binary.
7. **Maintenance** – when the interface changes, the header is updated, and dependent source files are recompiled.

---

## 9. Practical Examples

### Example 1: Standard Header Guard Pattern
```c
// utils.h
#ifndef UTILS_H
#define UTILS_H

#include <stddef.h>   // Include standard headers if needed.

// Function prototypes.
size_t string_length(const char *str);
char *string_copy(char *dest, const char *src);

// Macro constants.
#define MAX_BUFFER 1024

// Type definition.
typedef struct {
    int id;
    char name[MAX_BUFFER];
} User;

#endif  // UTILS_H
```
**Code Breakdown:**
- `#ifndef UTILS_H` – checks if the macro is not defined.
- `#define UTILS_H` – defines it, so subsequent inclusions skip the content.
- The header includes `stddef.h` because it uses `size_t`.
- All declarations are consistent and visible to any `.c` file that includes this header.

### Example 2: Using `#pragma once` (Compiler Extension)
```c
// utils.h
#pragma once   // Non‑standard but widely supported; ensures single inclusion.

#include <stddef.h>

size_t string_length(const char *str);
// ... other declarations
```
**Code Breakdown:**
- `#pragma once` is a common extension (GCC, Clang, MSVC) that makes the header included only once per translation unit.
- It's simpler and less error‑prone than include guards, but not part of the C standard (though widely implemented).
- Some projects use it for speed and conciseness, but for maximum portability, use traditional guards.

### Example 3: Header with `extern` Declarations for Global Variables
```c
// config.h
#ifndef CONFIG_H
#define CONFIG_H

// Declare global variables as extern (not definitions).
extern int debug_level;
extern const char *app_name;

// Function prototype.
void load_config(void);

#endif
```

```c
// config.c
#include "config.h"

// Define the global variables (allocate storage).
int debug_level = 0;
const char *app_name = "MyApp";

void load_config(void) {
    // Load configuration from file, etc.
}
```

```c
// main.c
#include "config.h"
#include <stdio.h>

int main(void) {
    printf("App: %s, Debug: %d\n", app_name, debug_level);
    load_config();
    return 0;
}
```
**Code Breakdown:**
- In the header, we use `extern` to declare the variables, so they're visible but not defined.
- In `config.c`, we define them (no `extern`), allocating storage.
- In `main.c`, we include the header and can access the variables.
- This is the standard pattern for sharing globals across files.

### Example 4: Inline Functions in Headers
```c
// math_utils.h
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

// static inline is safe in headers.
static inline int max_int(int a, int b) {
    return a > b ? a : b;
}

static inline int min_int(int a, int b) {
    return a < b ? a : b;
}

// Also functions defined inline without static (external linkage) are rare.
// For header‑only, always use static inline.

#endif
```
**Code Breakdown:**
- `static inline` functions in headers are a common idiom.
- Each translation unit that includes the header gets its own copy (due to `static`), avoiding multiple definition errors.
- The compiler may inline the function, eliminating call overhead.

### Example 5: Forward Declarations to Break Circular Includes
```c
// player.h
#ifndef PLAYER_H
#define PLAYER_H

// Forward declaration of the Game structure.
struct Game;

typedef struct {
    int id;
    char name[32];
    struct Game *current_game;   // pointer to incomplete type.
} Player;

void player_join(Player *p, struct Game *g);

#endif
```

```c
// game.h
#ifndef GAME_H
#define GAME_H

// Forward declaration of Player.
typedef struct Player Player;

typedef struct {
    int id;
    Player *players[10];
    int player_count;
} Game;

void game_add_player(Game *g, Player *p);

#endif
```

```c
// game.c
#include "game.h"
#include "player.h"   // Now Player is fully defined for implementation.

void game_add_player(Game *g, Player *p) {
    if (g->player_count < 10) {
        g->players[g->player_count++] = p;
    }
}
```

```c
// player.c
#include "player.h"
#include "game.h"   // Now Game is fully defined.

void player_join(Player *p, Game *g) {
    game_add_player(g, p);
}
```
**Code Breakdown:**
- `player.h` declares `struct Game;` to avoid needing `game.h` for the pointer.
- `game.h` declares `typedef struct Player Player;` to refer to Player.
- This breaks the circular dependency, as each header only needs an incomplete type of the other.
- The implementation files include both headers for full definitions.

### Example 6: Platform‑Specific Headers with Conditional Compilation
```c
// platform.h
#ifndef PLATFORM_H
#define PLATFORM_H

#ifdef _WIN32
    #define PLATFORM "Windows"
    #include <windows.h>
#elif defined(__linux__)
    #define PLATFORM "Linux"
    #include <unistd.h>
#elif defined(__APPLE__)
    #define PLATFORM "macOS"
    #include <unistd.h>
#else
    #error "Unsupported platform"
#endif

// Common functions for all platforms.
void platform_sleep(int seconds);

#endif
```
**Code Breakdown:**
- Uses predefined macros (`_WIN32`, `__linux__`) to detect the platform.
- Includes different system headers for each platform.
- Defines a common interface (`platform_sleep`) that is implemented differently per platform.
- This is a common pattern for cross‑platform code.

### Example 7: Self‑Contained Header with Only Declarations
```c
// string_utils.h
#ifndef STRING_UTILS_H
#define STRING_UTILS_H

#include <stddef.h>   // Depend on another header.

// This header only declares – no definitions.
size_t my_strlen(const char *s);
char *my_strdup(const char *s);
char *my_strcat(char *dest, const char *src);

#endif
```
**Code Breakdown:**
- The header includes `<stddef.h>` because it uses `size_t`.
- It declares functions but doesn't define them.
- The implementation file (`string_utils.c`) will provide the definitions.

---

## 10. Common Use Cases

| Use Case | Example |
|----------|---------|
| **Library interfaces** | `#include <stdio.h>`, `#include "mylib.h"` |
| **Project‑wide types** | `typedef` and `struct` definitions in a common header. |
| **Configuration constants** | `#define MAX_USERS 100` in `config.h`. |
| **Global variable sharing** | `extern int g_debug;` in header, defined in one `.c`. |
| **Inline function definitions** | `static inline int square(int x)` in header. |
| **Conditional compilation** | `#ifdef __cplusplus extern "C" { #endif` for C++ compatibility. |
| **Platform abstraction** | Different headers for different OSes. |
| **Module interfaces** | Each module has its own `.h` file. |

---

## 11. Best Practices

### General
- **Always use include guards** (`#ifndef`/`#define`/`#endif` or `#pragma once`).
- **Include only what you need** – avoid including headers unnecessarily to reduce compile times.
- **Keep headers self‑contained** – a header should compile on its own (include all necessary headers).
- **Use forward declarations** to reduce dependencies and break cycles.
- **Define only declarations in headers** – not definitions (except `static inline` functions and macros).
- **Use `extern` for global variables** – never define variables in headers (unless `static`).

### Organisation
- **Group related declarations** – one header per module or logical group.
- **Use consistent naming** – e.g., `module_name.h` for public API, `module_name_private.h` for internal (if needed).
- **Place `#include` directives in source files** for implementation details, not in headers unless needed for declarations.

### Documentation
- **Document public headers** – describe the purpose of functions, parameters, return values.
- **Use comments** to explain non‑obvious aspects of the interface.

### Compilation
- **Use `-I` to add include directories** – for project headers.
- **Use `-MMD`** (GCC) to generate dependency files for Make.
- **Precompiled headers** – for large projects, use precompiled headers to speed compilation.

### Portability
- **Use standard headers** when possible (`<stdint.h>`, `<stdbool.h>`).
- **Avoid compiler‑specific `#pragma`** unless necessary.
- **Use `#ifdef __cplusplus`** to allow C++ to include C headers.

---

## 12. Common Mistakes

### Mistake 1: Defining Variables in Headers
```c
// header.h
int global_counter = 0;   // ❌ Wrong – this is a definition, not a declaration.
// If included in multiple files, linker will have multiple definitions.

// ✅ Correct – use extern.
extern int global_counter;   // declaration only.
```
**Why it's wrong:** The variable is defined (storage allocated) in every translation unit that includes the header, leading to multiple definition errors at link time.

### Mistake 2: Missing Include Guards
```c
// header.h
int add(int a, int b);   // No guard – if included twice, redefinition error.

// ✅ Correct – add include guards.
#ifndef HEADER_H
#define HEADER_H
int add(int a, int b);
#endif
```
**Why it's wrong:** If the header is included multiple times (directly or indirectly), the declarations are duplicated, causing compile errors.

### Mistake 3: Circular Includes Without Forward Declarations
```c
// a.h
#include "b.h"
struct A { struct B *b; };

// b.h
#include "a.h"
struct B { struct A *a; };

// ❌ Compilation error – infinite recursion.
// ✅ Use forward declarations to break the cycle.
```

### Mistake 4: Including Source Files Instead of Headers
```c
// main.c
#include "helper.c"   // ❌ Wrong – includes source file, not header.

// ✅ Correct – include the header.
#include "helper.h"
```
**Why it's wrong:** Including a `.c` file brings in definitions, leading to multiple definitions when the `.c` file is also compiled separately.

### Mistake 5: Not Including Necessary Headers in a Header
```c
// utils.h
size_t string_length(const char *str);   // ❌ size_t not defined.
// ✅ Include <stddef.h>.
#include <stddef.h>
size_t string_length(const char *str);
```
**Why it's wrong:** The header uses `size_t` but doesn't include `<stddef.h>`, so any file including `utils.h` without including `<stddef.h>` first will get a compilation error.

### Mistake 6: Using `#include` with Double Quotes for System Headers
```c
#include "stdio.h"   // ❌ This searches the current directory first, may find wrong file.
#include <stdio.h>   // ✅ Correct for system headers.
```

### Mistake 7: Placing Non‑`static` Inline Functions in Headers
```c
// header.h
inline int add(int a, int b) { return a + b; }   // ❌ May cause multiple definitions.
// ✅ Use static inline.
static inline int add(int a, int b) { return a + b; }
```
**Why it's wrong:** Without `static`, an inline function with external linkage may cause multiple definition errors if included in multiple translation units.

### Mistake 8: Forgetting to Include a Header in the Implementation File
```c
// mymath.c
int add(int a, int b) { return a + b; }   // No include of mymath.h.
// ✅ Include the header to check consistency.
#include "mymath.h"
```
**Why it's wrong:** The compiler won't check that the definition matches the prototype, potentially causing mismatched signatures (different return types, parameters).

---

## 13. Performance Considerations

- **Compile time** – including many large headers increases compilation time. Use forward declarations and minimise includes where possible.
- **Precompiled headers** – can dramatically speed up compilation for large projects.
- **Dependency management** – if a header changes, all files including it must be recompiled. Keep headers minimal.
- **`#pragma once`** – can be slightly faster than traditional guards because the compiler doesn't need to reopen the header to check the guard.
- **Header ordering** – irrelevant for performance but important for correctness.

---

## 14. Security Considerations

- **Header exposure** – when distributing libraries, headers reveal the API. Ensure no sensitive information (like passwords or internal details) is in public headers.
- **Include path injection** – an attacker could trick the compiler into including a malicious header if the include path is not secured. Use absolute or controlled include directories.
- **Macro definitions** – macros in headers can be overwritten by the user; ensure macros are used consistently and not a security risk.

---

## 15. Debugging Tips

- **Use `-E`** (GCC/Clang) to see the preprocessor output and verify what the compiler sees:
  ```bash
  gcc -E main.c -o main.i
  ```
- **Use `-H`** to see which headers are included and their nesting depth.
- **Use `-MD` or `-MMD`** to generate dependency files for build systems.
- **Inspect include paths** with `gcc -v` or `-I` options.
- **If you get linker errors** about undefined symbols, check that the header declares `extern` variables and that the matching definitions exist in the source files.

---

## 16. When to Use

- **Always** for declarations that need to be shared across multiple source files.
- **For every module** – create a `.h` file for its public interface.
- **For global constants and macros** – place them in headers with appropriate guards.
- **For inline functions** – use `static inline` in headers.

---

## 17. When Not to Use

- **Do not put function definitions in headers** (except `static inline`).
- **Do not put variable definitions in headers** – use `extern`.
- **Avoid large headers** – split into smaller, focused headers.
- **Avoid including headers that are not needed** – it increases dependencies and compile time.

---

## 18. Related Concepts

- **Preprocessor** – `#include` is a preprocessor directive.
- **Translation unit** – a source file after preprocessing.
- **Linkage** – `extern` and `static` affect visibility across files.
- **Functions** – prototypes in headers.
- **Macros** – `#define` constants in headers.
- **Inline functions** – commonly defined in headers.
- **Build systems** – handle dependencies (Make, CMake).
- **Libraries** – headers are part of the library interface.

---

## 19. Did You Know?

- The `#include` directive is a textual inclusion – the preprocessor literally pastes the content of the header into the source file.
- Standard headers like `<stdio.h>` often include other headers (e.g., `<stddef.h>`), so you might not need to include them explicitly.
- Some headers are provided by the compiler itself (e.g., `<stdint.h>`) and may contain compiler‑specific implementations.
- The term "header file" comes from the idea that it contains the "heading" declarations of a module.
- `#pragma once` is supported by all major compilers (GCC, Clang, MSVC) but is not part of the C standard. However, it's widely used in practice.
- In the early days of C, headers sometimes contained code, but modern practice strictly separates interface (`.h`) from implementation (`.c`).

---

## 20. Summary

- **Header files (.h)** contain declarations—function prototypes, types, macros, and `extern` variable declarations.
- **They enable modular programming** by providing a clean interface between modules.
- **Include guards** (`#ifndef`/`#define`) prevent multiple inclusions, avoiding redefinition errors.
- **System headers** use angle brackets (`< >`); **project headers** use double quotes (`" "`).
- **Always include what you need** – headers should be self‑contained and minimal.
- **Use forward declarations** to break circular dependencies.
- **Never define variables or non‑`static` functions** in headers – use `extern` and `static inline` where appropriate.
- **Common mistakes**: missing guards, defining variables, circular includes, and failing to include needed headers.
- **Header files are essential** for organising large C projects, sharing code, and implementing libraries.