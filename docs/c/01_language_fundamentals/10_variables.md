# Variables

## 1. Overview

### Definition
A variable is a named storage location in memory that holds a value which can be changed during program execution. In C, every variable has a **type**, a **name**, and a **value**; it must be declared before use, specifying its type and optionally an initial value.

### Purpose
Variables are the fundamental containers of data in any program. They allow you to store, retrieve, and manipulate information. Without variables, you could only work with literal constants, making any dynamic computation impossible. They are the building blocks of all stateful computation.

### Where It Fits
Variables are the most common element in C code. They appear in:
- **Declarations** – defining a variable's type and name.
- **Definitions** – allocating storage and optionally initialising.
- **Assignments** – changing the value.
- **Expressions** – reading the value for computation.
- **Function parameters** – receiving arguments.

---

## 2. Why It Exists

### The Problem Without Variables
Without variables, you could not store or process dynamic data. Every computation would have to use immediate values, making programs static and useless for anything beyond trivial arithmetic. You could not read user input, process files, or maintain state.

### The Solution: Named Storage
Variables provide a way to name a memory location, read from it, write to it, and change its value over time. They abstract away memory addresses, giving you a human‑readable name for a piece of data.

### Why It Was Introduced
Variables are a fundamental concept in all imperative programming languages. In C, they are closely tied to the underlying hardware – a variable corresponds to a memory location, and the type determines how that memory is interpreted. This low‑level transparency is part of C's systems‑programming philosophy.

---

## 3. Syntax / Basic Usage

### Declaring and Defining Variables
```c
#include <stdio.h>

// Declaration (extern – no storage allocated, just tells compiler about it)
extern int global_count;

// Definition (storage allocated)
int global_counter = 10;      // global variable with initialisation
static int file_scope_var;    // static global (file scope), zero‑initialised

int main(void) {
    // Automatic variables (local)
    int x;                    // uninitialised – contains indeterminate value
    int y = 5;                // initialised with literal
    int z = y + 2;            // initialised with expression
    int a, b, c;              // multiple declarations
    int d = 1, e = 2;         // multiple with initialisers

    // const variables
    const int MAX = 100;      // read‑only variable, must be initialised

    // Register variable (hint to compiler)
    register int fast = 10;   // may be stored in CPU register (C89, deprecated in C++)

    // static local variable
    static int call_count = 0; // retains value between function calls

    x = 42;                   // assignment
    a = b = c = 0;            // chained assignment

    printf("y: %d, z: %d, x: %d\n", y, z, x);
    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>   // Header for printf

extern int global_count;
// 'extern' declares that this variable is defined elsewhere.
// No storage is allocated; the linker resolves it.

int global_counter = 10;
// Definition: allocates storage, initialised to 10.
// This variable has external linkage (visible across files).

static int file_scope_var;
// 'static' gives internal linkage: only visible within this file.
// No explicit initialiser → zero‑initialised (by the runtime).

int main(void) {
    // Automatic storage duration: created on entry to main, destroyed on exit.
    int x;        // Uninitialised; its value is indeterminate.
                  // Reading it before assignment is undefined behaviour.

    int y = 5;    // Initialised to 5. This is a definition and initialisation.

    int z = y + 2; // Initialised with an expression using y's value (valid).

    int a, b, c;  // Multiple declarations on one line.
    int d = 1, e = 2; // Multiple with initialisers.

    const int MAX = 100;
    // 'const' makes MAX read‑only; must be initialised.
    // MAX = 200; // error: assignment of read‑only variable.

    register int fast = 10;
    // 'register' is a hint to the compiler to store this variable in a CPU register.
    // Modern compilers ignore this hint (they register‑allocate automatically).
    // You cannot take the address of a register variable (&fast is invalid).

    static int call_count = 0;
    // Static local: initialised once when the program starts.
    // Retains its value between function calls.
    // It is stored in static storage, not on the stack.

    x = 42;       // Assignment: changes x's value from indeterminate to 42.

    a = b = c = 0; // Chained assignment: right‑associative, sets all to 0.

    printf("y: %d, z: %d, x: %d\n", y, z, x);
    // Prints values; x is now 42, so it's safe.
    return 0;
}
```

---

## 4. Mental Model – Variables as Labelled Boxes

Imagine a warehouse (computer memory) filled with many boxes. Each box has:
- **A label** – the variable name.
- **A type** – what kind of item can be stored (int, float, etc.).
- **A capacity** – how many bytes (determined by the type).
- **Current contents** – the value.

You can:
- **Declare** a box: “We will have a box called `count` for integers.”
- **Define** it: actually place the box on a shelf and maybe put an initial item in it.
- **Assign** to it: replace the contents with a new item.
- **Read** from it: look at what’s inside.

Some boxes are:
- **Automatic** – set up when you enter a room (function) and discarded when you leave.
- **Static** – permanent boxes in a dedicated storage area, retaining their contents.
- **Global** – boxes accessible from any room.
- **Const** – boxes with a lock; you cannot change the contents after initial placement.

---

## 5. Core Concepts

| Concept | Explanation |
|---------|-------------|
| **Declaration** | Introduces the variable name and type; does not allocate storage (except when also a definition). |
| **Definition** | Allocates storage for the variable; may include initialisation. |
| **Initialisation** | Assigning an initial value at the point of definition. |
| **Assignment** | Changing the value of an already‑defined variable. |
| **Type** | Determines the size, representation, and allowed operations. |
| **Storage duration** | When the variable exists: automatic, static, allocated (heap), or thread‑local. |
| **Scope** | Where the variable name is visible: block scope, file scope, function scope, function prototype scope. |
| **Linkage** | How the variable is shared across translation units: external, internal, or none. |
| **Storage class specifiers** | `auto`, `register`, `static`, `extern`, `_Thread_local` (C11). |
| **Type qualifiers** | `const`, `volatile`, `restrict` (C99). |

### Storage Duration Summary
| Storage Duration | Keyword | Lifetime | Scope | Typical Use |
|------------------|---------|----------|-------|-------------|
| **Automatic** | `auto` (default) | Function entry to exit | Block | Local variables |
| **Static** | `static` | Entire program | File or block | Persistent locals, file‑private globals |
| **External** | `extern` (def) | Entire program | Global | Shared variables |
| **Thread‑local** | `_Thread_local` | Thread lifetime | Block or file | Per‑thread data (C11) |

---

## 6. How It Works – Variable Lifecycle

### Automatic Variables (Stack)
1. **Declaration** – the compiler reserves space on the stack when the function is called.
2. **Initialisation** – if an initialiser is provided, the value is written to that stack slot.
3. **Use** – the variable is accessed via its stack offset.
4. **Destruction** – when the function returns, the stack pointer is adjusted, and the variable's memory is no longer accessible (it may be overwritten by later calls).

### Static Variables (Static Storage)
1. **Compile time** – the compiler allocates space in the `.data` or `.bss` section.
2. **Program startup** – the runtime initialises them (zero for `.bss`, explicit value for `.data`) before `main` is called.
3. **Use** – they are accessed via a fixed address.
4. **Program termination** – the memory is reclaimed when the process ends.

### External Variables (Global)
- Similar to static, but with **external linkage** – the symbol is visible to the linker.
- Other translation units can access them via `extern` declarations.

### ASCII Diagram – Variable Storage
```
┌──────────────────────────────────────────────────────────────┐
│                    Virtual Memory Layout                      │
├──────────────────────────────────────────────────────────────┤
│  .text (code)                                                │
├──────────────────────────────────────────────────────────────┤
│  .rodata (read‑only data) – string literals, const globals   │
├──────────────────────────────────────────────────────────────┤
│  .data (initialised data) – globals with explicit init value │
│    └── static int x = 10;                                    │
├──────────────────────────────────────────────────────────────┤
│  .bss (zero‑initialised data) – globals with no init        │
│    └── static int y;   // zero-initialised                  │
├──────────────────────────────────────────────────────────────┤
│  Heap (dynamically allocated)                                │
├──────────────────────────────────────────────────────────────┤
│  Stack (automatic variables, call frames)                    │
│    └── int z;   // local variable (stack)                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 7. Internal Architecture – Variable Representation

### Memory Layout
- Each variable is assigned a memory address.
- The size depends on the type and platform:
  - `int` – typically 4 bytes.
  - `char` – 1 byte.
  - `float` – 4 bytes.
  - `double` – 8 bytes.
  - Pointers – 4 or 8 bytes (32‑bit or 64‑bit).

- For structs, the layout includes padding for alignment.

### Linker Symbols
- Global variables with external linkage appear in the symbol table.
- Static variables have internal linkage – the linker does not expose them to other object files.
- Local variables do not appear in the symbol table; they are stack‑relative.

### Initialisation
- **Implicit zero‑initialisation** – for static storage duration variables without an explicit initialiser.
- **Uninitialised automatic variables** – have indeterminate values; using them is undefined behaviour.

---

## 8. Lifecycle / Workflow of a Variable

1. **Declaration** – the compiler records the variable’s type and name.
2. **Definition** – storage is allocated (or reserved).
3. **Initialisation** – if provided, the initial value is stored.
4. **Use** – the variable is read or written in expressions.
5. **Modification** – assignment changes its value.
6. **End of lifetime** – for automatic variables, this happens when the block ends; for static/global, at program termination.

---

## 9. Practical Examples

### Example 1: Automatic vs Static Local Variables
```c
#include <stdio.h>

void counter(void) {
    // Automatic variable – recreated each call, initialised each time
    int auto_count = 0;
    auto_count++;
    printf("auto_count: %d\n", auto_count);

    // Static local – initialised once, retains value
    static int static_count = 0;
    static_count++;
    printf("static_count: %d\n", static_count);
}

int main(void) {
    counter();  // auto:1, static:1
    counter();  // auto:1 (starts fresh), static:2
    counter();  // auto:1, static:3
    return 0;
}
```
**Code Breakdown:**
- `auto_count` – automatic, each call creates a new instance, initialised to 0 each time; output always 1.
- `static_count` – static storage duration; initialised once at program startup; retains its incremented value across calls.

### Example 2: Global Variables and `extern`
**file1.c**
```c
#include <stdio.h>

int shared = 42;        // global definition (external linkage)

void print_shared(void) {
    printf("shared: %d\n", shared);
}
```

**file2.c**
```c
#include <stdio.h>

extern int shared;      // declaration – refers to shared in file1.c

int main(void) {
    printf("In main: %d\n", shared);
    shared = 100;
    print_shared();     // prints 100
    return 0;
}
```
**Code Breakdown:**
- `shared` is defined in file1.c with external linkage.
- In file2.c, `extern int shared;` tells the compiler that `shared` exists elsewhere.
- The linker resolves the reference.

### Example 3: `const` and `volatile` Qualifiers
```c
#include <stdio.h>
#include <time.h>   // for clock()

int main(void) {
    // const: cannot be modified after initialisation
    const double PI = 3.14159;
    // PI = 3.0;   // error: assignment of read‑only variable

    // volatile: tells compiler not to optimise away reads/writes
    // Typically used for hardware registers or variables changed by interrupts.
    volatile int sensor_value = 0;
    // In a real program, sensor_value might be updated by an interrupt.
    // Without volatile, the compiler might cache it.
    int reading = sensor_value;  // compiler must read from memory

    // Note: volatile does not make the variable thread‑safe; use atomics.
    return 0;
}
```
**Code Breakdown:**
- `const` – makes the variable read‑only; protects against accidental modification.
- `volatile` – prevents the compiler from optimising away accesses, ensuring that each read/write goes to memory. Useful for memory‑mapped I/O and signal handlers.

### Example 4: `restrict` Pointer (C99)
```c
#include <stdio.h>

// restrict tells the compiler that pointers do not alias.
void add_arrays(int *restrict a, int *restrict b, int n) {
    for (int i = 0; i < n; i++) {
        a[i] += b[i];
    }
}

int main(void) {
    int x[5] = {1,2,3,4,5};
    int y[5] = {10,20,30,40,50};
    add_arrays(x, y, 5);  // x and y do not overlap
    for (int i = 0; i < 5; i++) {
        printf("%d ", x[i]);
    }
    printf("\n");
    return 0;
}
```
**Code Breakdown:**
- `restrict` is a type qualifier for pointers.
- It indicates that the pointer is the only reference to the object during its lifetime – no aliasing.
- This allows the compiler to perform more aggressive optimisations (e.g., vectorisation).

### Example 5: Thread‑Local Storage (C11)
```c
#include <stdio.h>
#include <threads.h>
#include <stdatomic.h>

// thread‑local variable: each thread gets its own copy
_Thread_local int thread_id = 0;

int thread_func(void *arg) {
    int id = *(int *)arg;
    thread_id = id;   // store per‑thread
    printf("Thread %d: thread_id = %d\n", id, thread_id);
    return 0;
}

int main(void) {
    thrd_t t1, t2;
    int id1 = 1, id2 = 2;
    thrd_create(&t1, thread_func, &id1);
    thrd_create(&t2, thread_func, &id2);
    thrd_join(t1, NULL);
    thrd_join(t2, NULL);
    return 0;
}
```
**Code Breakdown:**
- `_Thread_local` (or `thread_local` from `<threads.h>`) gives each thread its own instance of the variable.
- Each thread writes and reads its own copy, avoiding data races.

---

## 10. Common Use Cases

| Use Case | Variable Type | Example |
|----------|---------------|---------|
| **Loop counters** | Automatic local | `for (int i = 0; i < n; i++)` |
| **Function state** | Static local | `static int call_count;` |
| **Shared data** | Global / extern | `int g_config;` in one file, `extern` in others |
| **Configuration constants** | `const` global | `const int BUFFER_SIZE = 1024;` |
| **Hardware registers** | `volatile` pointer | `volatile uint32_t *reg = (volatile uint32_t *)0x4000;` |
| **Performance‑critical** | `register` (rarely needed) | `register int sum = 0;` |
| **Per‑thread data** | `_Thread_local` | `_Thread_local int errno;` (in libc) |

---

## 11. Best Practices

- **Always initialise variables** – especially automatic ones, to avoid undefined behaviour.
- **Minimise scope** – declare variables as close to their first use as possible (C99/C11 allows declarations anywhere in a block).
- **Use `const` for read‑only variables** – documents intent and enables optimisations.
- **Prefer local variables over globals** – reduces coupling and improves testability.
- **Use `static` for file‑private globals** – limits visibility, preventing accidental external access.
- **Use meaningful names** – `count`, `index`, `temperature` instead of `x`, `y`, `z` (except in short loops).
- **Avoid `register`** – modern compilers are better at register allocation.
- **Use `_Thread_local` for thread‑specific data** – instead of using global variables with locks.
- **Be aware of alignment** – struct member ordering can affect padding and size.
- **Use `size_t` for sizes and indices** – ensures portability across platforms.

---

## 12. Common Mistakes

### Mistake 1: Using Uninitialised Variable
```c
// ❌ Wrong – undefined behaviour
int x;
printf("%d\n", x);  // x has indeterminate value
// ✅ Correct – initialise
int x = 0;
```

### Mistake 2: Shadowing a Variable
```c
// ❌ Confusing – shadows outer variable
int x = 5;
if (1) {
    int x = 10;  // shadows outer x
    printf("%d\n", x); // prints 10
}
printf("%d\n", x); // still 5 – hard to read
// ✅ Better – use distinct names
```

### Mistake 3: Forgetting `extern` Declaration
```c
// file1.c
int global = 5;
// file2.c
int global = 10;   // ❌ error: multiple definitions
// ✅ Correct – declare as extern in file2.c
extern int global;  // refers to file1's global
```

### Mistake 4: Modifying `const` Variable
```c
const int MAX = 100;
MAX = 200;  // ❌ error: assignment of read‑only variable
```

### Mistake 5: Taking Address of `register` Variable
```c
register int x = 5;
int *p = &x;   // ❌ error: address of register variable requested
```

### Mistake 6: Misunderstanding Static Local Initialisation
```c
void func(void) {
    static int count = 0;
    count++;
    // count retains its value across calls
}
// The initialisation "= 0" happens only once, at program startup.
```

---

## 13. Performance Considerations

- **Local variables** – on the stack, extremely fast access (cache‑friendly).
- **Global/static variables** – in data section, access may be slightly slower (indirect addressing).
- **`const`** – can enable compiler optimisations (e.g., constant propagation).
- **`volatile`** – prevents optimisations; use only when necessary.
- **`register`** – mostly ignored; rely on compiler optimisations (`-O2`).
- **Static local** – like global, but with restricted scope; same performance as global.
- **Padding** – struct alignment can waste memory; order members from largest to smallest to minimise padding.

---

## 14. Security Considerations

- **Uninitialised variables** – can leak sensitive data; always initialise.
- **Global variables** – can be accessed by any function; protect with locks in multi‑threaded code.
- **`volatile`** – does not provide atomicity; use `_Atomic` for thread‑safe shared variables.
- **Static locals** – in functions, they retain state; be careful in reentrant or threaded environments.
- **String buffers** – ensure they are properly terminated; use safe functions.

---

## 15. Debugging Tips

- **Use `-Wall -Wextra`** – catches uninitialised variables, signed/unsigned mismatches.
- **Use `-g`** – debug symbols allow you to inspect variable values in GDB.
- **`gdb` commands**:
  - `print x` – show value of x.
  - `info locals` – show all local variables.
  - `watch x` – break when x changes.
- **Use `valgrind`** – detects uninitialised value usage.
- **Add debug prints** – but remove them in production.

---

## 16. When to Use

- **All the time** – variables are unavoidable.
- **Use the appropriate storage class** – automatic for local, static for persistent local, global for shared data.
- **Use `const`** whenever a value should not change.

---

## 17. When Not to Use

- **Avoid global variables** unless absolutely necessary (e.g., configuration, shared state).
- **Avoid `register`** – it is obsolete.
- **Avoid `volatile`** for thread synchronisation – use atomic operations.
- **Avoid `static` local for thread‑unsafe state** – use thread‑local storage.

---

## 18. Related Concepts

- **Data types** – the type of a variable.
- **Scope** – where a variable is visible.
- **Lifetime** – when a variable exists.
- **Storage classes** – `auto`, `static`, `extern`, `register`, `_Thread_local`.
- **Type qualifiers** – `const`, `volatile`, `restrict`.
- **Pointers** – variables that hold addresses.
- **Memory layout** – stack, heap, data, bss, rodata.
- **Initialisation** – zero‑initialisation, static initialisation, dynamic initialisation.

---

## 19. Did You Know?

- In C, a variable declared at file scope without `static` has external linkage by default.
- The `register` keyword is deprecated in C++17, but still in C – though it is largely a no‑op.
- `_Thread_local` was introduced in C11; before that, you had to use compiler‑specific extensions.
- Automatic variables are not initialised by default; their values are whatever is on the stack.
- You can declare variables anywhere in a block, not just at the top, since C99.
- `const` does not mean “constant expression” – you cannot use `const int` for array sizes in C89; use `enum` or `#define`.

---

## 20. Summary

- **Variables are named memory locations** that store data with a specific type.
- **They have storage duration** (automatic, static, thread‑local, allocated) and **scope** (block, file).
- **Storage class specifiers** (`auto`, `static`, `extern`, `register`, `_Thread_local`) control lifetime and linkage.
- **Type qualifiers** (`const`, `volatile`, `restrict`) add extra semantics.
- **Best practices**: initialise variables, minimise scope, prefer locals over globals, use `static` for file‑private data, use `const` for read‑only values.
- **Common mistakes**: uninitialised variables, shadowing, multiple definitions, modifying `const`.
- **Understanding variables** is foundational to writing any C program – they are the essence of data manipulation.