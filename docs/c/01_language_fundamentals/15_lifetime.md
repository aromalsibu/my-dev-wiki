# Lifetime

## 1. Overview

### Definition
Lifetime (also called **storage duration**) in C refers to the period during program execution when a variable actually exists in memory and retains its value. It determines when a variable is created, when it is destroyed, and how long its storage persists. Lifetime is closely related to scope but is a distinct concept.

### Purpose
Understanding lifetime is critical for:
- **Memory management** – knowing when variables are allocated and deallocated.
- **Avoiding bugs** – preventing use of variables after they've been destroyed.
- **Performance** – understanding stack vs heap allocation.
- **State management** – using `static` variables to persist state between function calls.
- **Multi‑threading** – managing thread‑local data.

### Where It Fits
Lifetime is determined by:
- **Storage duration** – automatic, static, thread, or allocated.
- **Storage class specifiers** – `auto`, `static`, `extern`, `register`, `_Thread_local`.
- **Allocation functions** – `malloc`, `calloc`, `realloc`, `free` for dynamic memory.

Lifetime works alongside:
- **Scope** – where a variable is visible.
- **Linkage** – whether a variable is shared across files.
- **Memory layout** – stack, heap, data segment, TLS.

---

## 2. Why It Exists

### The Problem Without Defined Lifetime
Without clear lifetime rules:
- You wouldn't know when memory is safe to use.
- Variables might persist too long (wasting memory) or not long enough (causing dangling pointers).
- Functions couldn't maintain state across calls.
- Dynamic memory allocation would be impossible to manage.
- Programs would leak memory or crash unpredictably.

### The Solution: Explicit Storage Durations
C defines four storage durations:
1. **Automatic** – variables are created when a block is entered and destroyed when it exits.
2. **Static** – variables exist for the entire program execution.
3. **Thread** – each thread gets its own copy, existing for the thread's lifetime.
4. **Allocated** – variables are created and destroyed explicitly via `malloc`/`free`.

Each duration gives the programmer control over how memory is managed, balancing performance, memory usage, and convenience.

### Why It Was Introduced
Lifetime rules have been fundamental to C from its inception. They reflect the underlying hardware:
- **Automatic** variables map to stack memory (fast allocation/deallocation).
- **Static** variables map to data segments (fixed addresses, persistent).
- **Allocated** variables map to the heap (flexible, but requires explicit management).
- **Thread** variables (C11) map to thread‑local storage (each thread has its own copy).

This design allows C to be both close to the hardware and provide high‑level abstractions.

---

## 3. Syntax / Basic Usage

### Automatic Storage Duration (Default for Locals)
```c
#include <stdio.h>

void func(void) {
    // Automatic storage duration (default)
    int x = 10;           // created when func is called
    // x exists here
}   // x is destroyed when func returns

int main(void) {
    func();
    // x no longer exists
    return 0;
}
```

### Static Storage Duration
```c
#include <stdio.h>

// Static storage duration (file scope)
static int file_static = 10;    // exists for entire program
int global = 20;                // also static duration (external linkage)

void counter(void) {
    // Static storage duration (block scope)
    static int count = 0;       // initialised once, persists across calls
    count++;
    printf("count: %d\n", count);
}

int main(void) {
    counter();  // count: 1
    counter();  // count: 2
    counter();  // count: 3
    // count persists, but is not visible outside counter()
    return 0;
}
```

### Thread Storage Duration (C11)
```c
#include <stdio.h>
#include <threads.h>

// Thread storage duration – each thread gets its own copy
_Thread_local int thread_data = 0;

int thread_func(void *arg) {
    thread_data = *(int *)arg;
    printf("Thread data: %d\n", thread_data);
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

### Allocated Storage Duration (Dynamic Memory)
```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    // Allocated storage duration
    int *p = (int *)malloc(sizeof(int));  // allocated on heap
    if (p == NULL) {
        return 1;
    }
    *p = 42;
    printf("Value: %d\n", *p);

    free(p);  // explicitly deallocated
    // p is now a dangling pointer – don't use it!
    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>
#include <stdlib.h>   // for malloc, free
#include <threads.h>  // for thrd_t, _Thread_local

// Static storage duration – program lifetime.
// Initialised once before main() starts.
static int file_static = 10;   // internal linkage (visible only in this file)

int global = 20;               // external linkage (visible to other files)

void counter(void) {
    // Automatic storage duration – created on entry, destroyed on exit.
    int auto_var = 0;          // Created each time counter() is called.
    auto_var++;

    // Static storage duration – but block scope.
    // Initialised once, persists across calls.
    static int static_count = 0;  // Initialised once, before main().
    static_count++;
    printf("auto_var: %d, static_count: %d\n", auto_var, static_count);
}

void example_malloc(void) {
    // Allocated storage duration – created by malloc, destroyed by free.
    int *arr = (int *)malloc(10 * sizeof(int));
    if (arr == NULL) {
        // Handle allocation failure.
        return;
    }
    // arr points to allocated memory on the heap.
    // The memory exists until free(arr) is called.
    free(arr);   // destroys the allocated memory.
    // After free, arr is a dangling pointer – don't dereference it.
}

// Thread storage duration – each thread gets its own copy.
_Thread_local int thread_local_var = 0;   // each thread has its own copy

int thread_func(void *arg) {
    int id = *(int *)arg;
    thread_local_var = id;   // stores thread‑specific data
    printf("Thread %d: %d\n", id, thread_local_var);
    return 0;
}

int main(void) {
    // Automatic storage – created when main starts, destroyed when main ends.
    int main_local = 10;

    counter();  // auto_var: 1, static_count: 1
    counter();  // auto_var: 1 (fresh), static_count: 2
    counter();  // auto_var: 1, static_count: 3

    // Thread storage – create two threads.
    thrd_t t1, t2;
    int id1 = 1, id2 = 2;
    thrd_create(&t1, thread_func, &id1);
    thrd_create(&t2, thread_func, &id2);
    thrd_join(t1, NULL);
    thrd_join(t2, NULL);

    // Allocated storage – dynamic array.
    int *dynamic_array = (int *)malloc(5 * sizeof(int));
    if (dynamic_array == NULL) {
        return 1;
    }
    dynamic_array[0] = 100;
    printf("Dynamic: %d\n", dynamic_array[0]);
    free(dynamic_array);

    return 0;
}
```

---

## 4. Mental Model – Lifetime as a Variable's Lifespan

Think of a variable as a character in a play:

- **Automatic duration** – an **extra** who appears on stage only during a specific scene (block). They are created when the scene starts and exit when it ends. They don't remember anything from previous scenes (no persistent state).

- **Static duration** – a **main character** who is present from the start of the play to the end. They remember everything that happened in earlier acts. At file scope, they are backstage (visible only to the crew in that file). Inside a function, they are a character with a "memory" between scenes.

- **Thread duration** – a **character in a parallel play** (each thread is its own performance). Each performance has its own copy of the character; they don't interact across performances.

- **Allocated duration** – a **prop** that you create when needed (with `malloc`) and explicitly destroy when you're done (with `free`). You control when it appears and disappears.

The **lifetime** is *when* the character exists. The **scope** is *which rooms* they can be seen from. You can have a character that exists for the whole play (static) but is only visible in one room (block scope).

---

## 5. Core Concepts

### Storage Durations Summary

| Duration | Keyword | Creation | Destruction | Storage Location |
|----------|---------|----------|-------------|------------------|
| **Automatic** | `auto` (default) | Block entry | Block exit | Stack |
| **Static** | `static` | Program startup | Program exit | `.data` / `.bss` |
| **Thread** | `_Thread_local` | Thread creation | Thread exit | Thread‑Local Storage (TLS) |
| **Allocated** | `malloc` / `calloc` / `realloc` | Explicit allocation (`malloc`) | Explicit deallocation (`free`) | Heap |

### Key Characteristics

#### Automatic Duration
- **Stack allocation** – very fast.
- **No explicit initialisation** – values are indeterminate unless initialised.
- **Scoped** – lifetime matches block scope.
- **Nested** – inner blocks create new variables.
- **Recursion** – each recursive call has its own set of automatic variables.

#### Static Duration
- **Program lifetime** – exists from before `main()` starts until after it exits.
- **Zero‑initialised** – if no explicit initialiser, set to `0` (or `NULL` for pointers).
- **Initialisation once** – initialisers are evaluated only once, at program startup.
- **Persistent state** – retains value between function calls (block‑scope static).
- **File‑private** – `static` at file scope gives internal linkage.

#### Thread Duration (C11)
- **Per‑thread lifetime** – each thread has its own copy.
- **Created at thread start** – initialised when the thread begins.
- **Destroyed at thread exit** – cleaned up when the thread terminates.
- **Use for thread‑specific data** – avoids race conditions.

#### Allocated Duration
- **Explicit control** – you decide when to allocate and free.
- **Heap storage** – flexible size, but slower than stack.
- **Persists across function calls** – memory remains until `free` is called.
- **No automatic cleanup** – must be manually managed (memory leaks, double frees).
- **Error handling** – `malloc` returns `NULL` on failure.

### Lifetime vs Scope vs Linkage

| Concept | Definition | Example |
|---------|------------|---------|
| **Lifetime** | When a variable exists in memory. | A `static` local exists for the whole program. |
| **Scope** | Where a variable is visible. | A `static` local is only visible inside its function. |
| **Linkage** | Whether a variable can be shared across translation units. | A global has external linkage; `static` has internal linkage. |

A variable can have:
- **Static lifetime, block scope, no linkage** – `static` local.
- **Static lifetime, file scope, internal linkage** – `static` global.
- **Static lifetime, file scope, external linkage** – global (default).
- **Automatic lifetime, block scope, no linkage** – local variable.

---

## 6. How It Works – Runtime Memory Management

### Automatic Variables (Stack)
1. **Function call** – the compiler generates code to reserve space on the stack (adjusts the stack pointer).
2. **Variable creation** – each automatic variable is assigned an offset within the stack frame.
3. **Initialisation** – if an initialiser is present, the value is stored.
4. **Use** – variables are accessed via stack offsets.
5. **Function return** – the stack pointer is restored, making the variables inaccessible (they are "destroyed").

### Static Variables (Data Segment)
1. **Compile time** – the compiler allocates space in `.data` (initialised) or `.bss` (zero‑initialised).
2. **Program startup** – the runtime initialises `.data` with explicit values and zeroes `.bss`.
3. **Use** – variables are accessed via fixed addresses.
4. **Program exit** – memory is reclaimed by the OS.

### Thread Variables (TLS)
1. **Thread creation** – the runtime allocates thread‑local storage for each thread.
2. **Initialisation** – each thread's copy is initialised.
3. **Use** – accessed via thread‑local storage mechanisms (e.g., `fs` segment on x86).
4. **Thread exit** – the thread‑local storage is deallocated.

### Allocated Variables (Heap)
1. **Allocation** – `malloc` finds a block of memory on the heap and returns a pointer.
2. **Use** – memory is accessed via the pointer.
3. **Deallocation** – `free` returns the memory to the heap for reuse.
4. **Memory management** – the heap manager tracks free and used blocks.

### ASCII Diagram – Lifetime Timeline
```
Program Start         main() calls func()      Program End
        │                    │                       │
        ▼                    ▼                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Static variables exist ──────────────────────────────────── │
├─────────────────────────────────────────────────────────────┤
│        Thread 1 TLS exists ──────────────────────────────  │
│        Thread 2 TLS exists ──────────────────────────────  │
├─────────────────────────────────────────────────────────────┤
│        func() called           func() returns              │
│        Automatic vars created  Automatic vars destroyed    │
├─────────────────────────────────────────────────────────────┤
│        malloc() called         free() called               │
│        Allocated memory exists ──────────────────────────  │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Internal Architecture – Storage Sections

### Memory Sections and Storage Durations
```
┌──────────────────────────────────────────────────────────────┐
│ .text (code)                                                │
├──────────────────────────────────────────────────────────────┤
│ .rodata (read‑only data)                                   │
│   └── string literals, const data (static duration)         │
├──────────────────────────────────────────────────────────────┤
│ .data (initialised static data)                            │
│   └── int global = 10;      (static duration)              │
│   └── static int x = 5;     (static duration)              │
├──────────────────────────────────────────────────────────────┤
│ .bss (zero‑initialised static data)                        │
│   └── int global;           (zero‑initialised)             │
│   └── static int y;         (zero‑initialised)             │
├──────────────────────────────────────────────────────────────┤
│ Heap (allocated duration)                                   │
│   └── malloc(), calloc(), realloc(), free()                │
├──────────────────────────────────────────────────────────────┤
│ Stack (automatic duration)                                  │
│   └── local variables, function call frames                │
├──────────────────────────────────────────────────────────────┤
│ Thread‑Local Storage (thread duration)                     │
│   └── _Thread_local variables                              │
└──────────────────────────────────────────────────────────────┘
```

### Initialisation by Storage Duration

| Storage Duration | Without Explicit Initialiser | With Explicit Initialiser |
|------------------|------------------------------|---------------------------|
| **Automatic** | Indeterminate (garbage) | Stored on stack |
| **Static** | Zero‑initialised (`.bss`) | Stored in `.data` |
| **Thread** | Zero‑initialised | Initialised per thread |
| **Allocated** | Indeterminate (garbage) | Must be manually set |

---

## 8. Lifecycle / Workflow of Each Duration

### Automatic Lifetime Workflow
1. **Block entry** – variable is created (stack frame is set up).
2. **Initialisation** – if specified, value is assigned.
3. **Use** – variable is accessed.
4. **Block exit** – variable is destroyed (stack pointer is restored).

### Static Lifetime Workflow
1. **Program startup** – variable is allocated in `.data` or `.bss`.
2. **Initialisation** – `.data` is initialised, `.bss` is zeroed.
3. **Use** – variable is accessed throughout program.
4. **Program exit** – variable is destroyed (OS reclaims memory).

### Thread Lifetime Workflow
1. **Thread creation** – variable is allocated in TLS.
2. **Initialisation** – each thread's copy is initialised.
3. **Use** – variable is accessed by that thread only.
4. **Thread exit** – variable is destroyed (TLS is freed).

### Allocated Lifetime Workflow
1. **`malloc` / `calloc`** – memory is allocated on the heap.
2. **Use** – variable is accessed via the pointer.
3. **`free`** – memory is deallocated.
4. **After `free`** – pointer is dangling; don't use it.

---

## 9. Practical Examples

### Example 1: Automatic vs Static Local
```c
#include <stdio.h>

void demo(void) {
    // Automatic – created fresh each call.
    int auto_count = 0;
    auto_count++;

    // Static – persists across calls.
    static int static_count = 0;
    static_count++;

    printf("auto: %d, static: %d\n", auto_count, static_count);
}

int main(void) {
    demo();  // auto: 1, static: 1
    demo();  // auto: 1 (fresh), static: 2
    demo();  // auto: 1 (fresh), static: 3
    return 0;
}
```
**Code Breakdown:**
- `auto_count` is created and destroyed each time `demo()` is called.
- `static_count` is created once at program startup and retains its value.
- Automatic variables are initialised each call; static variables are initialised once.

### Example 2: Static Initialisation Order
```c
#include <stdio.h>

// File‑scope static – zero‑initialised by default.
static int zero_initialised;   // 0

// File‑scope static – explicitly initialised.
static int explicit = 42;

int main(void) {
    printf("zero_initialised: %d\n", zero_initialised);  // 0
    printf("explicit: %d\n", explicit);                 // 42

    // Function‑scope static – initialised once at program startup.
    static int local_static = 100;
    printf("local_static: %d\n", local_static);
    return 0;
}
```
**Code Breakdown:**
- File‑scope static variables without an initialiser are zero‑initialised.
- File‑scope static variables with an initialiser are initialised to that value.
- Function‑scope static variables are initialised at program startup (before `main` runs).

### Example 3: Dynamic Allocation (Heap)
```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    // Allocate memory for 5 integers.
    int *arr = (int *)malloc(5 * sizeof(int));
    if (arr == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return 1;
    }

    // Use allocated memory.
    for (int i = 0; i < 5; i++) {
        arr[i] = i * 10;
        printf("arr[%d] = %d\n", i, arr[i]);
    }

    // Free allocated memory.
    free(arr);

    // arr is now a dangling pointer – don't dereference it!
    // arr[0] = 100; // ❌ Undefined behaviour

    return 0;
}
```
**Code Breakdown:**
- `malloc` allocates memory on the heap and returns a pointer.
- The memory persists until `free` is called.
- After `free`, the pointer is invalid (dangling).
- Always check `malloc`'s return value for `NULL`.

### Example 4: Allocated Memory Across Functions
```c
#include <stdio.h>
#include <stdlib.h>

int *create_array(int size) {
    // Allocated memory – persists after create_array returns.
    int *arr = (int *)malloc(size * sizeof(int));
    return arr;   // pointer to heap memory
}

void use_array(int *arr, int size) {
    for (int i = 0; i < size; i++) {
        arr[i] = i * 2;
    }
}

int main(void) {
    int *my_array = create_array(5);
    if (my_array == NULL) {
        return 1;
    }

    use_array(my_array, 5);

    for (int i = 0; i < 5; i++) {
        printf("%d ", my_array[i]);
    }
    printf("\n");

    free(my_array);   // Clean up.
    return 0;
}
```
**Code Breakdown:**
- `create_array` allocates memory and returns a pointer to it.
- The memory persists even after `create_array` returns (unlike automatic variables).
- `use_array` can modify the memory.
- The caller is responsible for freeing the memory.

### Example 5: Thread‑Local Storage (C11)
```c
#include <stdio.h>
#include <threads.h>
#include <stdlib.h>

_Thread_local int thread_id = 0;

int thread_func(void *arg) {
    int id = *(int *)arg;
    thread_id = id;   // each thread has its own copy
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

    // thread_id in main is still 0 (each thread has its own copy).
    printf("Main: thread_id = %d\n", thread_id);
    return 0;
}
```
**Code Breakdown:**
- `_Thread_local` gives each thread its own copy of `thread_id`.
- Thread 1 and Thread 2 each modify their own copy.
- The main thread's copy remains unchanged.

### Example 6: Lifetime Issues – Returning Pointer to Local
```c
#include <stdio.h>

// ❌ WRONG – returning pointer to automatic variable.
int *bad_function(void) {
    int local = 42;
    return &local;   // local is destroyed when function returns!
}

int main(void) {
    int *p = bad_function();
    // p points to memory that is no longer valid (dangling pointer).
    printf("%d\n", *p);   // Undefined behaviour – may crash or print garbage.
    return 0;
}
```
**Code Breakdown:**
- `local` has automatic storage duration – it's destroyed when `bad_function` returns.
- Returning its address gives a dangling pointer.
- Dereferencing it is undefined behaviour.
- **Fix:** use `static` or allocate memory with `malloc`.

### Example 7: Correctly Returning a Valid Pointer
```c
#include <stdio.h>
#include <stdlib.h>

// ✅ Correct – using static.
int *good_static(void) {
    static int value = 42;   // static – persists.
    return &value;           // safe to return address of static.
}

// ✅ Correct – using malloc.
int *good_malloc(void) {
    int *p = (int *)malloc(sizeof(int));
    if (p) {
        *p = 42;
    }
    return p;   // caller must free.
}

int main(void) {
    int *s = good_static();
    printf("static: %d\n", *s);

    int *m = good_malloc();
    if (m) {
        printf("malloc: %d\n", *m);
        free(m);
    }
    return 0;
}
```
**Code Breakdown:**
- Returning a pointer to a `static` variable is safe (static lifetime).
- Returning a pointer to `malloc`‑ed memory is safe (caller must free).
- Never return a pointer to an automatic variable.

---

## 10. Common Use Cases

| Use Case | Storage Duration | Example |
|----------|------------------|---------|
| **Local variables** | Automatic | `int x;` inside a function |
| **Loop counters** | Automatic | `for (int i = 0; ...)` |
| **Function state** | Static (block) | `static int call_count;` |
| **File‑private data** | Static (file) | `static int helper_count;` |
| **Global configuration** | Static (file or external) | `int config;` at file scope |
| **Dynamic arrays** | Allocated (heap) | `int *arr = malloc(n * sizeof(int));` |
| **Data structures** | Allocated (heap) | Linked lists, trees, graphs |
| **Thread‑specific data** | Thread (TLS) | `_Thread_local int errno;` |
| **String literals** | Static | `"Hello"` (read‑only) |

---

## 11. Best Practices

### Automatic Duration
- **Initialise local variables** – they have indeterminate values by default.
- **Keep variables in smallest scope** – minimises lifetime and reduces memory usage.
- **Use `const` for read‑only locals** – documents intent.

### Static Duration
- **Use `static` for file‑private data** – encapsulation and to avoid name conflicts.
- **Use `static` locals for persistent state** – but beware of thread safety.
- **Prefer `static` over globals** – when possible, to limit visibility.
- **Initialise static variables explicitly** – even though they're zero‑initialised, explicit is clearer.

### Allocated Duration
- **Always check `malloc` return value** – it can return `NULL` on failure.
- **Free memory when done** – prevent memory leaks.
- **Set pointers to `NULL` after `free`** – to avoid dangling pointer bugs.
- **Use `calloc` for zero‑initialised allocations** – it initialises memory to `0`.
- **Use `realloc` carefully** – it may move the memory; save the result.

### Thread Duration
- **Use `_Thread_local` for thread‑specific data** – avoids race conditions.
- **Initialise thread‑local variables with constants** – dynamic initialisation can be tricky.
- **Use `_Thread_local` with `static`** – for file‑private thread data.

### General
- **Understand the lifetime** of every variable you use.
- **Never return pointers to automatic variables** – dangling pointers.
- **Use memory management tools** – Valgrind, AddressSanitizer to detect leaks and errors.
- **Be mindful in recursive functions** – each call has its own automatic variables.

---

## 12. Common Mistakes

### Mistake 1: Returning Pointer to Local
```c
// ❌ Wrong – returning pointer to automatic variable.
int *get_pointer(void) {
    int x = 10;
    return &x;   // x is destroyed when function returns.
}
// ✅ Correct – use static or allocate memory.
static int x = 10;
int *get_pointer(void) {
    return &x;
}
```

### Mistake 2: Using Memory After `free`
```c
// ❌ Wrong – use‑after‑free.
int *p = malloc(sizeof(int));
*p = 42;
free(p);
*p = 100;   // Undefined behaviour – p is dangling.
// ✅ Correct – set p to NULL after free.
free(p);
p = NULL;
// Then check before using: if (p != NULL) *p = 100;
```

### Mistake 3: Not Freeing Memory (Memory Leak)
```c
// ❌ Wrong – memory leak.
void leak(void) {
    int *p = malloc(100 * sizeof(int));
    // forgot to free(p);
}
// ✅ Correct – free when done.
void no_leak(void) {
    int *p = malloc(100 * sizeof(int));
    // use p
    free(p);
}
```

### Mistake 4: Assuming Static Locals Are Thread‑Safe
```c
// ❌ Wrong – static local is shared across threads.
void increment(void) {
    static int counter = 0;
    counter++;   // Not thread‑safe – data race.
}
// ✅ Correct – use atomic or thread‑local.
void increment(void) {
    static atomic_int counter = 0;
    atomic_fetch_add(&counter, 1);   // thread‑safe
}
```

### Mistake 5: Misunderstanding Static Initialisation
```c
#include <stdio.h>

int *get_value(void) {
    static int value;   // zero‑initialised (0)
    value++;
    return &value;
}

int main(void) {
    printf("%d\n", *get_value());  // 1
    printf("%d\n", *get_value());  // 2
    // value is initialised to 0 only once, at program startup.
    return 0;
}
```

### Mistake 6: Dangling Pointer After `realloc`
```c
// ❌ Wrong – if realloc fails, p is lost.
int *p = malloc(10 * sizeof(int));
p = realloc(p, 20 * sizeof(int));   // if realloc fails, p is NULL and old memory is lost.
// ✅ Correct – use a temporary variable.
int *new_p = realloc(p, 20 * sizeof(int));
if (new_p == NULL) {
    // Handle error, p is still valid.
    free(p);
    return;
}
p = new_p;
```

---

## 13. Performance Considerations

- **Automatic** – fastest allocation (stack pointer adjustment). Minimal overhead.
- **Static** – fixed addresses, no allocation overhead. Fast access.
- **Allocated** – slower (heap management), but flexible.
- **Thread** – TLS access is slightly slower than regular static (requires TLS lookup).
- **Memory locality** – stack variables are cache‑friendly; heap variables can be fragmented.
- **`static` vs `auto`** – static uses less stack space, but access may be slower due to data segment addressing.
- **Dynamic allocation** – can cause fragmentation; reuse memory where possible.

---

## 14. Security Considerations

- **Dangling pointers** – can be exploited (use‑after‑free vulnerabilities). Always nullify after `free`.
- **Memory leaks** – can exhaust resources, leading to denial of service.
- **Uninitialised automatic variables** – contain garbage, which can leak sensitive information.
- **Heap overflow** – writing past allocated memory can corrupt heap metadata, leading to exploits.
- **Static locals** – can be a problem in reentrant code or signal handlers; be careful.
- **Thread‑local storage** – avoids data races, improving security in multi‑threaded code.

---

## 15. Debugging Tips

- **Use Valgrind** – detects memory leaks, use‑after‑free, and uninitialised memory.
- **Use AddressSanitizer (`-fsanitize=address`)** – catches heap and stack buffer overflows, use‑after‑free.
- **Use `-Wall -Wextra`** – warns about uninitialised variables.
- **Set pointers to `NULL` after `free`** – makes use‑after‑free easier to detect (segfault instead of silent corruption).
- **Use `static` analysis tools** – like Clang Static Analyzer, Coverity.
- **Debug with `gdb`** – `print` variables, `watch` memory addresses.
- **Use `malloc` debugging** – `MALLOC_CHECK_` environment variable on glibc.

---

## 16. When to Use

- **Automatic** – for most local variables; fast and safe.
- **Static** – for persistent state (function‑scope) and file‑private data (file‑scope).
- **Allocated** – when size is unknown at compile time, or when data must outlive the function that created it.
- **Thread** – for thread‑specific data, replacing globals with locks.

---

## 17. When Not to Use

- **Avoid static** for large arrays if they are only needed temporarily – they occupy memory for the entire program.
- **Avoid allocated** for small, short‑lived objects – stack is faster and safer.
- **Avoid thread‑local** for single‑threaded programs – unnecessary overhead.
- **Avoid automatic** for objects that must persist across function calls – use `static` or allocated.

---

## 18. Related Concepts

- **Scope** – where a variable is visible.
- **Linkage** – whether a variable is shared across files.
- **Storage classes** – `auto`, `static`, `extern`, `register`, `_Thread_local`.
- **Memory management** – `malloc`, `calloc`, `realloc`, `free`.
- **Stack and heap** – memory regions for different lifetimes.
- **Data segments** – `.data`, `.bss`, `.rodata`.
- **Thread‑Local Storage (TLS)** – implementation of thread lifetime.
- **Undefined behaviour** – accessing variables outside their lifetime.

---

## 19. Did You Know?

- In C, the lifetime of a variable is not the same as its scope. A `static` local has block scope but static lifetime.
- The `register` storage class suggests automatic storage but with a hint to put it in a register (lifetime is still automatic).
- `_Thread_local` variables are initialised before the thread function is called (C11).
- String literals have static storage duration – they exist for the entire program.
- `const` does not affect lifetime – a `const` variable can be automatic, static, or allocated.
- C23 may introduce `constexpr` for compile‑time constants, but lifetime rules remain unchanged.

---

## 20. Summary

- **Lifetime (storage duration)** defines how long a variable exists in memory.
- **Four storage durations**: automatic, static, thread, and allocated.
- **Automatic** – created at block entry, destroyed at block exit (stack).
- **Static** – exists for the entire program (`.data` / `.bss`).
- **Thread** – each thread has its own copy (TLS).
- **Allocated** – created with `malloc`, destroyed with `free` (heap).
- **Lifetime vs Scope** – lifetime is *when* it exists; scope is *where* it's visible.
- **Best practices**: initialise variables, check `malloc` return, free memory, avoid dangling pointers.
- **Common mistakes**: returning pointers to locals, use‑after‑free, memory leaks, uninitialised variables.
- **Understanding lifetime** is crucial for memory safety, performance, and correct program behaviour.