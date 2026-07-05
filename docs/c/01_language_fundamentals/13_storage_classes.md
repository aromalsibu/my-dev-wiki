# Storage Classes

## 1. Overview

### Definition
Storage classes in C determine the **scope** (visibility), **lifetime** (duration), **storage location**, and **linkage** of variables and functions. They are specified using keywords that appear before the type in a declaration. C provides five storage class specifiers: `auto`, `register`, `static`, `extern`, and `_Thread_local` (C11).

### Purpose
Storage classes give you control over:
- **Where** a variable is stored (register, stack, static data, or thread‑local storage).
- **How long** it lives (function call, program lifetime, or thread lifetime).
- **Where** it is visible (within a block, file, or across files).
- **How** it is shared (internal or external linkage).

This control is essential for memory management, performance optimisation, and structuring large programs.

### Where It Fits
Storage classes are part of variable declarations. They work alongside data types and type qualifiers. They are fundamental to understanding how C manages memory and how to organise code across multiple files.

---

## 2. Why They Exist

### The Problem Without Storage Classes
Without storage classes, all variables would have the same behaviour:
- All variables would be globally visible (cluttering the namespace).
- All variables would persist for the entire program (wasting memory).
- There would be no way to optimise frequently used variables.
- Sharing variables across files would be messy.

### The Solution: Different Storage Classes
C provides a range of storage classes to match different needs:
- **`auto`** – the default for local variables; short‑lived and block‑scoped.
- **`register`** – hint to store in a CPU register for speed.
- **`static`** – gives variables internal linkage and static lifetime; useful for file‑private data and persistent state in functions.
- **`extern`** – declares a variable defined elsewhere; enables sharing across files.
- **`_Thread_local`** – each thread gets its own copy; enables thread‑safe global state.

### Why They Were Introduced
These storage classes were designed to give programmers low‑level control over memory and visibility, aligning with C's philosophy of trust the programmer and closeness to the hardware. They were present in early C (except `_Thread_local`, which was added in C11 for multi‑threading).

---

## 3. Syntax / Basic Usage

### Declaring Variables with Storage Classes
```c
#include <stdio.h>
#include <threads.h>   // for thrd_t, _Thread_local

// Storage class specifiers appear before the type.
auto int x = 10;           // auto is the default for local variables
register int y = 20;       // hint to store in a register
static int z = 30;         // static storage duration, internal linkage
extern int global_var;     // declaration (not definition) – defined elsewhere
_Thread_local int tls = 0; // thread‑local storage (C11)

// They can also be used in function declarations.
static void helper(void);  // internal linkage – only visible in this file
extern int add(int, int);  // declares a function defined elsewhere

int main(void) {
    // auto is implicit – you rarely write it.
    int local = 5;         // same as 'auto int local = 5;'

    // register – cannot take its address.
    register int fast = 10;

    // static inside a function – retains value between calls.
    static int call_count = 0;
    call_count++;

    // _Thread_local inside a function – each thread gets its own copy.
    _Thread_local int thread_id = 0;

    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>
#include <threads.h>   // for _Thread_local, thrd_t, etc.

// auto – default for local variables. You rarely write it explicitly.
// It gives automatic storage duration (stack) and block scope.
auto int x = 10;       // x exists only within its block (file scope here, but auto at file scope is valid but useless)

// register – a hint to the compiler to store the variable in a CPU register.
// Cannot take the address of a register variable.
register int y = 20;

// static – at file scope, it gives internal linkage and static storage duration.
// The variable is visible only within this translation unit.
static int z = 30;     // file‑private global

// extern – declares a variable defined elsewhere (another translation unit).
// Does not allocate storage; it's a reference.
extern int global_var; // defined in another .c file

// _Thread_local – each thread gets its own copy of the variable.
// Storage duration is the lifetime of the thread.
_Thread_local int tls = 0; // each thread has its own tls variable

// static function – internal linkage; only visible in this file.
static void helper(void) {
    // This function cannot be called from other files.
}

// extern function – defined in another file.
extern int add(int a, int b);

int main(void) {
    // auto is the default for local variables; you can omit it.
    int local = 5;         // automatic storage, block scope, no linkage.

    // register local variable – hint to store in register.
    register int fast = 10;
    // &fast; // ❌ Error: address of register variable requested.

    // static local variable – static storage duration, block scope, no linkage.
    // Initialised only once, before program startup.
    static int call_count = 0;
    call_count++;          // retains value between function calls.

    // _Thread_local local variable – each thread has its own copy.
    _Thread_local int thread_id = 0;
    // This is useful for thread‑specific data.

    return 0;
}
```

---

## 4. Mental Model – Storage Classes as Variable "Cloaks"

Imagine each variable has a "cloak" that determines its behaviour:

- **`auto`** – a **disposable cloak**. The variable appears when you enter a room (function) and disappears when you leave. Each time you enter, you get a fresh one.

- **`register`** – a **fast cloak**. It's kept in a pocket (CPU register) for quick access, but you can't attach a label to it (cannot take its address). The compiler may ignore this hint.

- **`static`** – a **permanent cloak**. The variable persists for the entire program. At file scope, it's hidden from other rooms (internal linkage). Inside a function, it remembers its state between visits.

- **`extern`** – a **shared cloak**. The variable is defined in another room; you're just borrowing it. You must declare it with `extern` to use it.

- **`_Thread_local`** – a **thread‑specific cloak**. Each person (thread) has their own copy. They don't interfere with each other.

---

## 5. Core Concepts

### Summary Table of Storage Classes

| Keyword | Storage Duration | Scope | Linkage | Typical Use |
|---------|------------------|-------|---------|-------------|
| `auto` | Automatic (stack) | Block | None | Local variables (default) |
| `register` | Automatic (may be register) | Block | None | Performance‑critical locals |
| `static` (file scope) | Static (program lifetime) | File | Internal | File‑private globals |
| `static` (block scope) | Static (program lifetime) | Block | None | Persistent local state |
| `extern` (declaration) | N/A (defined elsewhere) | Varies | External | Sharing variables across files |
| `_Thread_local` | Thread (thread lifetime) | Block/File | Depends on other specifiers | Per‑thread data |

### Storage Duration
- **Automatic** – allocated on entry to the block, deallocated on exit.
- **Static** – allocated at program startup, deallocated at program termination.
- **Thread** – allocated when a thread is created, deallocated when the thread terminates.

### Scope
- **Block scope** – visible from the point of declaration to the end of the enclosing block.
- **File scope** – visible from the point of declaration to the end of the file.
- **Function scope** – only for labels (used with `goto`).

### Linkage
- **External** – visible across translation units (default for file‑scope variables and functions).
- **Internal** – visible only within the current translation unit (using `static`).
- **None** – visible only within the current block (local variables).

---

## 6. How It Works – Compiler and Runtime Handling

### `auto` (Automatic Storage)
1. **Compile time** – the compiler records the variable's offset on the stack frame.
2. **Runtime** – when the function is called, space is allocated on the stack.
3. **Initialisation** – if an initialiser is present, it is executed each time the block is entered.
4. **Destruction** – when the block exits, the stack pointer is restored, making the variable inaccessible.

### `register`
1. **Compile time** – the compiler attempts to allocate the variable in a CPU register.
2. **If not possible** – it falls back to automatic storage (on the stack).
3. **Address unavailable** – taking the address of a `register` variable is not allowed.
4. **Modern compilers** – mostly ignore the `register` keyword (they are better at register allocation).

### `static`
#### File‑scope static:
1. **Compile time** – allocated in `.data` (if initialised) or `.bss` (if zero‑initialised).
2. **Linkage** – internal; the linker does not export the symbol.
3. **Initialisation** – occurs before `main` is called (static initialisation).

#### Block‑scope static:
1. **Compile time** – allocated in `.data` or `.bss` (not on the stack).
2. **Initialisation** – once, before program startup.
3. **Visibility** – only within the block, but the storage persists.

### `extern`
1. **Declaration** – tells the compiler that the variable is defined in another translation unit.
2. **No storage** – the declaration does not allocate storage.
3. **Linker** – resolves the reference at link time.

### `_Thread_local`
1. **Runtime** – each thread gets its own copy of the variable.
2. **Storage** – allocated in thread‑local storage (TLS) when the thread starts.
3. **Initialisation** – each thread's copy is initialised separately.
4. **Destruction** – when the thread exits, its copy is deallocated.

### ASCII Diagram – Storage Class Placement in Memory
```
┌──────────────────────────────────────────────────────────────┐
│                    Virtual Memory Layout                      │
├──────────────────────────────────────────────────────────────┤
│  .text (code)                                                │
├──────────────────────────────────────────────────────────────┤
│  .rodata (read‑only data)                                   │
├──────────────────────────────────────────────────────────────┤
│  .data (initialised static data)                            │
│    └── static int x = 10;     (file or block scope)         │
│    └── int global = 20;       (external linkage)            │
├──────────────────────────────────────────────────────────────┤
│  .bss (zero‑initialised static data)                        │
│    └── static int y;          (zero‑initialised)            │
├──────────────────────────────────────────────────────────────┤
│  Heap (dynamically allocated)                               │
├──────────────────────────────────────────────────────────────┤
│  Stack (automatic variables)                                │
│    └── int z;                 (auto)                        │
│    └── register int r;        (may be in register)          │
├──────────────────────────────────────────────────────────────┤
│  Thread‑Local Storage (TLS)                                 │
│    └── _Thread_local int t;   (each thread has its own)    │
└──────────────────────────────────────────────────────────────┘
```

---

## 7. Internal Architecture – Symbol Tables and Linker

### Compiler Symbol Table
- Records variables with their storage class, type, scope, and linkage.
- For `static` variables, the symbol is marked as internal (not exported).
- For `extern` declarations, the symbol is marked as unresolved (to be resolved by the linker).

### Linker Handling
- **External symbols** – the linker resolves references to definitions in other object files.
- **Internal symbols** (`static`) – are not visible to the linker; they are local to the object file.
- **TLS symbols** – handled specially; the runtime allocates per‑thread copies.

---

## 8. Lifecycle / Workflow of Each Storage Class

### `auto`
1. **Declaration** – appears inside a block.
2. **Entry to block** – memory allocated on stack.
3. **Initialisation** – if provided, value stored.
4. **Use** – variable can be read/written.
5. **Exit block** – memory deallocated.

### `register`
1. **Declaration** – with `register` keyword.
2. **Compile‑time** – compiler tries to allocate a register.
3. **Use** – if in register, operations are faster; address cannot be taken.
4. **Block exit** – register is freed (if allocated).

### `static` (file scope)
1. **Declaration** – at file scope with `static`.
2. **Program startup** – storage allocated in `.data`/`.bss`, initialised.
3. **Use** – visible only within the file.
4. **Program termination** – storage deallocated.

### `static` (block scope)
1. **Declaration** – inside a block with `static`.
2. **Program startup** – storage allocated, initialised once.
3. **Use** – visible only within the block, but persists across calls.
4. **Program termination** – storage deallocated.

### `extern`
1. **Declaration** – with `extern` (usually in a header file).
2. **Compile‑time** – no storage allocated; symbol marked as external.
3. **Link‑time** – linker resolves the symbol to a definition.
4. **Runtime** – accesses the storage of the defined variable.

### `_Thread_local`
1. **Declaration** – with `_Thread_local`.
2. **Thread creation** – each thread gets its own copy.
3. **Use** – accessed via TLS mechanisms.
4. **Thread termination** – copy is deallocated.

---

## 9. Practical Examples

### Example 1: `auto` – The Default
```c
#include <stdio.h>

int main(void) {
    // auto is the default; rarely written explicitly.
    auto int x = 10;    // same as 'int x = 10;'
    int y = 20;         // auto by default

    {
        // New scope – different variables.
        int y = 30;     // this shadows the outer y
        printf("Inner: x=%d, y=%d\n", x, y); // x=10, y=30
    }

    printf("Outer: x=%d, y=%d\n", x, y); // x=10, y=20
    return 0;
}
```
**Code Breakdown:**
- `auto` variables exist only within their block.
- The inner `y` shadows the outer `y`; both have automatic storage.
- `auto` is implicit and rarely used explicitly.

### Example 2: `register` – A Hint for Speed
```c
#include <stdio.h>

int main(void) {
    // 'register' is just a hint; the compiler may ignore it.
    register int counter = 0;

    // You cannot take the address of a register variable.
    // int *p = &counter; // ❌ Error: address of register variable requested

    // In a loop, registering a variable might help performance.
    for (register int i = 0; i < 1000000; i++) {
        counter += i;
    }

    printf("Counter: %d\n", counter);
    return 0;
}
```
**Code Breakdown:**
- `register` suggests storing the variable in a CPU register.
- Modern compilers are good at register allocation; `register` is often ignored.
- The address operator (`&`) cannot be used on `register` variables.

### Example 3: `static` at File Scope
**file1.c**
```c
#include <stdio.h>

// static global – visible only in this file.
static int file_counter = 0;

// non‑static global – visible across files (external linkage).
int global_counter = 0;

static void helper(void) {
    file_counter++;
}

void increment(void) {
    helper();
    global_counter++;
}

void print_counters(void) {
    printf("file_counter: %d, global_counter: %d\n",
           file_counter, global_counter);
}
```

**file2.c**
```c
#include <stdio.h>

// Declare global_counter from file1.c.
extern int global_counter;

// We cannot access file_counter because it's static in file1.c.

// We can also declare functions from file1.c.
extern void increment(void);
extern void print_counters(void);

int main(void) {
    increment();
    increment();
    print_counters(); // prints: file_counter: 2, global_counter: 2
    return 0;
}
```
**Code Breakdown:**
- `static` at file scope limits visibility to the file; it's a form of encapsulation.
- `extern` declarations in file2.c refer to symbols defined in file1.c.
- `file_counter` and `helper` are not visible outside file1.c.

### Example 4: `static` inside a Function
```c
#include <stdio.h>

int next_id(void) {
    // static local – initialised once, retains value between calls.
    static int id = 0;
    return id++;
}

int main(void) {
    printf("%d\n", next_id()); // 0
    printf("%d\n", next_id()); // 1
    printf("%d\n", next_id()); // 2
    // The variable 'id' persists across calls, unlike an automatic variable.
    return 0;
}
```
**Code Breakdown:**
- `static` local variables are initialised once, before program startup.
- They retain their value between function calls.
- They are not visible outside the function.

### Example 5: `extern` for Sharing Variables
**config.h**
```c
#ifndef CONFIG_H
#define CONFIG_H

// Declaration – tells other files that 'debug_mode' exists.
extern int debug_mode;

#endif
```

**config.c**
```c
#include "config.h"

// Definition – allocates storage.
int debug_mode = 0;
```

**main.c**
```c
#include "config.h"
#include <stdio.h>

int main(void) {
    debug_mode = 1;   // modifies the global variable
    printf("Debug mode: %d\n", debug_mode);
    return 0;
}
```
**Code Breakdown:**
- `config.h` declares `debug_mode` as `extern` – a reference.
- `config.c` defines `debug_mode` – allocates storage.
- `main.c` includes `config.h` and can access `debug_mode`.
- This pattern is standard for global configuration variables.

### Example 6: `_Thread_local` (C11)
```c
#include <stdio.h>
#include <threads.h>
#include <stdlib.h>

// Thread‑local variable – each thread gets its own copy.
_Thread_local int thread_counter = 0;

int thread_func(void *arg) {
    int id = *(int *)arg;
    for (int i = 0; i < 3; i++) {
        thread_counter++;
        printf("Thread %d: counter = %d\n", id, thread_counter);
    }
    return 0;
}

int main(void) {
    thrd_t t1, t2;
    int id1 = 1, id2 = 2;

    thrd_create(&t1, thread_func, &id1);
    thrd_create(&t2, thread_func, &id2);

    thrd_join(t1, NULL);
    thrd_join(t2, NULL);

    // thread_counter in main is still 0 (each thread has its own copy).
    printf("Main thread counter: %d\n", thread_counter);
    return 0;
}
```
**Code Breakdown:**
- `_Thread_local` gives each thread its own copy of `thread_counter`.
- Thread 1 increments its copy; thread 2 increments its own copy.
- The main thread's copy remains unchanged.
- This is useful for thread‑specific data, like `errno`.

### Example 7: Combining Storage Classes with Other Specifiers
```c
#include <stdio.h>

// static and const – file‑private read‑only variable.
static const int MAX_BUFFER = 1024;

// extern and const – a read‑only variable defined elsewhere.
extern const char *APP_NAME;

// static and _Thread_local – each thread gets its own file‑private variable.
static _Thread_local int thread_error = 0;

int main(void) {
    // static const local – read‑only persistent variable.
    static const int local_max = 100;

    // _Thread_local and static – persistent per‑thread state.
    static _Thread_local int per_thread_state = 0;
    per_thread_state++;

    printf("MAX_BUFFER: %d\n", MAX_BUFFER);
    printf("Per‑thread state: %d\n", per_thread_state);
    return 0;
}
```
**Code Breakdown:**
- Storage classes can be combined with `const` and other qualifiers.
- `static const` – a file‑private constant.
- `static _Thread_local` – a thread‑local variable with internal linkage.

---

## 10. Common Use Cases

| Storage Class | Use Case | Example |
|---------------|----------|---------|
| `auto` (default) | Most local variables | `int x = 5;` |
| `register` | Performance‑critical loops (rarely needed) | `register int i;` |
| `static` (file scope) | File‑private variables and functions | `static int helper_count;` |
| `static` (block scope) | Persistent state in functions | `static int call_count = 0;` |
| `extern` | Sharing globals across files | `extern int config;` |
| `extern` | Function declarations in headers | `extern void init(void);` |
| `_Thread_local` | Per‑thread data (e.g., `errno`) | `_Thread_local int errno;` |
| Combinations | Read‑only file‑private data | `static const int MAX = 100;` |
| Combinations | Per‑thread file‑private data | `static _Thread_local int state;` |

---

## 11. Best Practices

### `auto`
- **Do not write `auto` explicitly** – it's the default; just omit it.
- **Use `auto` (implicitly) for all local variables** unless you need `static` or `_Thread_local`.

### `register`
- **Avoid `register`** – modern compilers ignore it and manage registers better.
- **Do not use `register`** if you need the variable's address.

### `static`
- **Use `static` at file scope** for functions and variables that should be private to the file – it's a form of encapsulation.
- **Use `static` inside functions** for persistent state (e.g., counters, caches).
- **Initialise `static` variables explicitly** – they are zero‑initialised if not, but explicit is clearer.

### `extern`
- **Use `extern` in header files** to declare global variables defined in a `.c` file.
- **Define the variable once** in a `.c` file (without `extern`).
- **Avoid using `extern` in source files** – use header files for consistency.

### `_Thread_local`
- **Use `_Thread_local` for thread‑specific data** – instead of global variables with locks.
- **Initialise with a constant** – dynamic initialisation for TLS is not always supported.
- **Use `static _Thread_local`** to limit visibility to the current file.

### General
- **Combine with `const`** for read‑only data.
- **Use meaningful names** – `static` variables at file scope are often prefixed with `s_` or similar.

---

## 12. Common Mistakes

### Mistake 1: Using `auto` at File Scope
```c
// ❌ Wrong – auto at file scope is valid but useless (and rarely used).
auto int x = 10;   // Same as 'int x = 10;' – file scope, external linkage.
// ✅ Correct – omit auto for file‑scope variables.
int x = 10;
```

### Mistake 2: Taking the Address of a `register` Variable
```c
// ❌ Wrong – cannot take address of register variable.
register int x = 5;
int *p = &x;   // error: address of register variable requested
// ✅ Correct – remove register if you need the address.
int x = 5;
int *p = &x;
```

### Mistake 3: Forgetting `extern` in One File
```c
// file1.c
int global = 10;   // definition

// file2.c
int global = 20;   // ❌ Wrong – multiple definitions (linker error)
// ✅ Correct – use extern in file2.c
extern int global;   // declaration, not definition
```

### Mistake 4: Using `static` in a Header (Accidental)
```c
// header.h
#ifndef HEADER_H
#define HEADER_H
static int count = 0;   // ❌ Wrong – each file including this gets its own copy!
#endif
// ✅ Correct – use extern in header, define in one .c file.
extern int count;
```
**Why it's wrong:** Each translation unit that includes the header gets its own `static` variable – a common source of confusion.

### Mistake 5: Not Realising `static` Local Initialisation Happens Once
```c
#include <stdio.h>

int counter(void) {
    static int count;   // zero‑initialised once
    return count++;
}

int main(void) {
    printf("%d\n", counter()); // 0
    printf("%d\n", counter()); // 1
    // The initialisation of 'count' happens only at program startup.
    return 0;
}
```
**Key point:** `static` locals are initialised once, not each time the function is called.

### Mistake 6: Using `_Thread_local` Without Including Headers
```c
// ❌ Wrong – missing headers.
_Thread_local int x = 0;
// ✅ Correct – include <threads.h> (or <stdthreads.h> in C11).
#include <threads.h>
_Thread_local int x = 0;
```

---

## 13. Performance Considerations

- **`auto`** – stack allocation is very fast (just moving the stack pointer).
- **`register`** – if the compiler honours it, access is faster (CPU register). But modern compilers ignore it.
- **`static`** – access to static data is slightly slower than stack (indirect addressing), but still fast.
- **`extern`** – same as `static` (global data), but may involve additional indirection if defined in another file.
- **`_Thread_local`** – access is slower than regular static (requires TLS lookup). Only use when necessary.

---

## 14. Security Considerations

- **`static`** – internal linkage reduces the attack surface (symbols are not exported).
- **`extern`** – shared globals can be a source of race conditions; use locking or atomics.
- **`_Thread_local`** – thread‑safe by design (each thread has its own copy).
- **`register`** – no security implications.
- **`static` locals** – can be problematic in reentrant code; use with care in multi‑threaded environments.

---

## 15. Debugging Tips

- **`static` variables** – are not visible in `nm` (symbol table) if they have internal linkage.
- **`extern` variables** – if you get "undefined reference", ensure you have a definition.
- **`_Thread_local`** – may require special debugger support; use `print` in GDB (may show thread‑local values).
- **Use `-Wall -Wextra`** – catches many storage‑class‑related issues.
- **Use `nm`** – to inspect symbols: `nm file.o` shows `T` for external functions, `t` for static.

---

## 16. When to Use

- **`auto`** – all local variables (implicit).
- **`register`** – almost never (let the compiler optimise).
- **`static` at file scope** – for file‑private functions and globals (always a good practice).
- **`static` at block scope** – for persistent state inside a function.
- **`extern`** – for sharing globals across translation units.
- **`_Thread_local`** – for per‑thread data.

---

## 17. When Not to Use

- **Avoid `register`** – it's obsolete; rely on compiler optimisations.
- **Avoid `static` in headers** – gives each file a separate copy; use `extern` instead.
- **Avoid `_Thread_local` for single‑threaded code** – unnecessary overhead.
- **Avoid `extern` for functions** – functions are extern by default; you only need `extern` for variables.

---

## 18. Related Concepts

- **Scope** – visibility of identifiers.
- **Lifetime** – storage duration of objects.
- **Linkage** – external, internal, none.
- **Memory layout** – .text, .data, .bss, heap, stack, TLS.
- **Type qualifiers** – `const`, `volatile`, `restrict`.
- **Threads** – multi‑threaded programming in C11.
- **Header files** – where `extern` declarations often reside.

---

## 19. Did You Know?

- The `auto` keyword is rarely used because it's the default – but it's still part of the language.
- `register` has been deprecated in C++17 but is still in C (though largely ignored).
- `static` at file scope was originally the only way to limit visibility; before that, all file‑scope variables were external.
- `_Thread_local` is often used in the implementation of `errno` – each thread has its own `errno`.
- The combination `static _Thread_local` gives a variable thread‑local storage with internal linkage.
- C23 introduces `constexpr` and `nullptr`, but does not add new storage classes (so far).

---

## 20. Summary

- **Storage classes control** scope, lifetime, storage location, and linkage.
- **`auto`** – default for local variables (automatic storage, block scope, no linkage).
- **`register`** – hint to store in a CPU register (rarely needed).
- **`static`** – static storage duration; at file scope gives internal linkage; at block scope retains value between calls.
- **`extern`** – declares a variable defined elsewhere; enables sharing across files.
- **`_Thread_local`** (C11) – each thread gets its own copy; thread storage duration.
- **Best practices**: use `static` for encapsulation, `extern` for sharing, `_Thread_local` for thread‑specific data.
- **Common mistakes**: using `register` unnecessarily, using `static` in headers, forgetting `extern` declarations.
- **Understanding storage classes** is essential for managing memory, organising code, and writing efficient, maintainable C programs.