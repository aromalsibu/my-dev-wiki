# Type Qualifiers

## 1. Overview

### Definition
Type qualifiers are keywords that modify the properties of a type, adding constraints or providing hints to the compiler about how the object should be treated. C provides four type qualifiers: `const`, `volatile`, `restrict` (C99), and `_Atomic` (C11). These qualifiers appear in declarations and affect the semantics of the type they qualify.

### Purpose
Type qualifiers serve different purposes:
- **`const`** – enforces immutability, preventing modification.
- **`volatile`** – prevents optimisation, ensuring each access goes to memory.
- **`restrict`** (C99) – hints that a pointer is the sole reference to an object, enabling optimisations.
- **`_Atomic`** (C11) – ensures atomic operations on shared variables in multi‑threaded code.

### Where It Fits
Type qualifiers are part of the type system. They appear in variable declarations, function parameters, and pointer types. They are distinct from storage class specifiers (`static`, `extern`, etc.) and type modifiers (`short`, `long`, etc.). They refine the behaviour of the type without changing its underlying representation.

---

## 2. Why They Exist

### The Problem Without Type Qualifiers
Without qualifiers, the compiler would treat all data uniformly, missing opportunities for optimisation and failing to enforce important constraints:
- No way to protect read‑only data from accidental modification.
- No way to prevent the compiler from optimising away accesses to memory‑mapped I/O registers.
- No way to inform the compiler that pointers don't alias, limiting performance.
- No built‑in support for thread‑safe atomic operations.

### The Solution: Specialised Qualifiers
Type qualifiers give programmers precise control over compiler behaviour:
- **`const`** – enables safer code by enforcing immutability.
- **`volatile`** – ensures correctness for hardware registers and variables modified by interrupts.
- **`restrict`** – allows the compiler to generate optimal machine code.
- **`_Atomic`** – provides a portable way to write lock‑free, thread‑safe code.

### Why They Were Introduced
- **`const`** and **`volatile`** were added in C89 to improve safety and systems programming.
- **`restrict`** was introduced in C99 to enable high‑performance computing optimisations (inspired by Fortran's semantics).
- **`_Atomic`** was added in C11 as part of the memory model to support multi‑threaded programming.

---

## 3. Syntax / Basic Usage

### `const` – Immutability
```c
#include <stdio.h>

int main(void) {
    // const variable – must be initialised, cannot be modified
    const int MAX = 100;
    // MAX = 200;  // ❌ Error: assignment of read‑only variable

    // const pointer to const data
    const int value = 42;
    const int *ptr1 = &value;   // pointer to const int (data is read‑only)
    // *ptr1 = 50;              // ❌ Error: cannot modify through ptr1
    ptr1 = NULL;                // ✅ OK: the pointer itself can change

    // const pointer to non‑const data
    int x = 10;
    int *const ptr2 = &x;       // const pointer (address cannot change)
    *ptr2 = 20;                 // ✅ OK: data can be modified
    // ptr2 = NULL;             // ❌ Error: pointer is const

    // const pointer to const data
    const int *const ptr3 = &value;
    // *ptr3 = 50;              // ❌ Error: data is const
    // ptr3 = NULL;             // ❌ Error: pointer is const

    return 0;
}
```

### `volatile` – No Optimisation
```c
#include <stdio.h>

int main(void) {
    // volatile variable – compiler must access memory each time
    volatile int sensor_value = 0;

    // In a real program, this might be updated by an interrupt or hardware.
    // Without volatile, the compiler might optimise this to a single read.
    int reading1 = sensor_value;  // forced memory read
    int reading2 = sensor_value;  // forced memory read again

    printf("Readings: %d, %d\n", reading1, reading2);
    return 0;
}
```

### `restrict` (C99) – No Aliasing
```c
#include <stdio.h>

// restrict says: a, b, and c do not point to overlapping memory.
void add_arrays(int *restrict a, int *restrict b, int *restrict c, int n) {
    for (int i = 0; i < n; i++) {
        c[i] = a[i] + b[i];   // compiler can assume no aliasing, enabling optimisations
    }
}

int main(void) {
    int x[10] = {1,2,3,4,5,6,7,8,9,10};
    int y[10] = {10,9,8,7,6,5,4,3,2,1};
    int z[10];
    add_arrays(x, y, z, 10);   // x, y, z are distinct – valid use of restrict
    for (int i = 0; i < 10; i++) {
        printf("%d ", z[i]);
    }
    printf("\n");
    return 0;
}
```

### `_Atomic` (C11) – Atomic Operations
```c
#include <stdio.h>
#include <stdatomic.h>
#include <threads.h>

atomic_int counter = 0;   // _Atomic int or atomic_int

int increment(void *arg) {
    for (int i = 0; i < 1000; i++) {
        atomic_fetch_add(&counter, 1);  // atomic increment
    }
    return 0;
}

int main(void) {
    thrd_t t1, t2;
    thrd_create(&t1, increment, NULL);
    thrd_create(&t2, increment, NULL);
    thrd_join(t1, NULL);
    thrd_join(t2, NULL);
    printf("Counter: %d\n", counter);  // Always 2000
    return 0;
}
```

### Code Breakdown (with comments)
```c
#include <stdio.h>
#include <stdatomic.h>   // for atomic_int and atomic_fetch_add
#include <threads.h>     // for threads

// const – makes a variable read‑only.
const int MAX = 100;     // Must be initialised; cannot be modified later.

// const with pointers – two dimensions: data and pointer itself.
const int *ptr_to_const;   // Pointer to constant data (data cannot change).
int *const const_ptr;      // Constant pointer (address cannot change).
const int *const const_both; // Both constant.

// volatile – tells the compiler that the value may change unexpectedly.
// Used for memory‑mapped I/O registers, variables modified by interrupt handlers,
// or variables shared between threads without locks (but prefer atomics).
volatile int reg = 0;      // Compiler will not optimise away accesses.

// restrict – C99 keyword for pointers.
// It says: this pointer is the only way to access the object.
// Use it to enable optimisations, but misuse leads to undefined behaviour.
int *restrict ptr;         // No other pointer aliases the object ptr points to.

// _Atomic – C11 keyword for atomic operations.
_Atomic int counter;       // Atomic int; operations on it are thread‑safe.
// Alternative: atomic_int counter; (from <stdatomic.h>).

int main(void) {
    // const: data is read‑only
    const int value = 42;
    // value = 50; // error: assignment of read‑only variable

    // volatile: prevent optimisation
    volatile int sensor = 0;
    int a = sensor;   // compiler generates a memory read
    int b = sensor;   // compiler generates another memory read
    // Without volatile, the compiler might have reused the value of a.

    // restrict: promise no aliasing
    int *restrict p = &a;
    // int *restrict q = &a; // undefined behaviour if p and q both access a
    // because restrict says p is the only reference.

    // _Atomic: thread‑safe operations
    atomic_int shared = 0;
    atomic_fetch_add(&shared, 1); // atomically increments

    return 0;
}
```

---

## 4. Mental Model – Qualifiers as Locks and Sticky Notes

Think of type qualifiers as annotations attached to variables:

- **`const`** – like a **lock** on a box. Once you put something in, you cannot open it again. The compiler enforces this: any attempt to write is a compile‑time error.

- **`volatile`** – like a sticky note saying **"DO NOT IGNORE ME"**. The compiler normally tries to be smart and remember values, but `volatile` forces it to look at the actual box every time.

- **`restrict`** – like a sign saying **"ONLY I MAY TOUCH THIS"**. It's a promise to the compiler that nobody else is touching this box, so it can optimise without worrying about conflicts.

- **`_Atomic`** – like a box with a **"HANDLE WITH CARE"** sign. Operations on it are done carefully so that if two people try to touch it at the same time, they don't mess it up.

These are not changes to the box itself (the type); they are instructions on how to treat the box.

---

## 5. Core Concepts

### `const` – Immutability
- **Purpose** – prevent modification; enforce read‑only semantics.
- **Can be applied to** – variables, function parameters, return types, pointers.
- **Important notes**:
  - `const` does not make a compile‑time constant (except for array sizes in some contexts).
  - `const` can be cast away (unsafe), so it's a tool for correctness, not security.
  - Use `const` in function parameters to indicate read‑only access.

### `volatile` – No Optimisation
- **Purpose** – prevent the compiler from optimising accesses.
- **Use cases**:
  - Memory‑mapped I/O registers.
  - Variables modified by interrupt handlers.
  - Variables shared between threads without explicit synchronisation (though `_Atomic` is better).
- **Important notes**:
  - `volatile` does not provide atomicity.
  - It does not imply ordering of operations.
  - Using it incorrectly can make code slower.

### `restrict` (C99) – No Aliasing
- **Purpose** – allow optimisations by asserting pointers do not alias.
- **Applicable only to pointers**.
- **Important notes**:
  - The programmer must ensure the promise is true; violating it is undefined behaviour.
  - `restrict` can enable vectorisation and other aggressive optimisations.
  - Typically used in performance‑critical code (e.g., numerical libraries).

### `_Atomic` (C11) – Atomic Operations
- **Purpose** – ensure operations on shared variables are atomic and thread‑safe.
- **Use cases** – lock‑free programming, multi‑threaded shared state.
- **Important notes**:
  - Requires `<stdatomic.h>` header.
  - `_Atomic` can be used as a qualifier (`_Atomic int x`) or as a type specifier (`atomic_int x`).
  - Operations like `atomic_fetch_add`, `atomic_load`, `atomic_store` are provided.
  - Provides memory ordering semantics (relaxed, acquire, release, etc.).

### Combination of Qualifiers
Qualifiers can be combined:
- `const volatile int x;` – read‑only but may change unexpectedly (e.g., read‑only hardware register).
- `const int * restrict ptr;` – pointer to const data, no aliasing.

---

## 6. How It Works – Compiler and Runtime Handling

### `const`
- **Compile time** – the compiler checks that no assignment or modification is attempted.
- **Runtime** – the variable may be placed in read‑only memory (`.rodata`) if it has static storage duration. Local `const` variables are usually on the stack but cannot be modified.
- **Optimisation** – the compiler may propagate the value, eliminating memory accesses.

### `volatile`
- **Compile time** – the compiler disables certain optimisations (e.g., common subexpression elimination, reordering, removing unused reads).
- **Runtime** – each access generates a memory read or write. It does not affect memory layout or storage.
- **Important** – `volatile` does not guarantee atomicity; use `_Atomic` for that.

### `restrict`
- **Compile time** – the compiler can assume no aliasing, enabling optimisations like reordering, vectorisation, and register allocation.
- **Undefined behaviour** – if the promise is broken (e.g., two `restrict` pointers point to the same object), the program is ill‑formed.

### `_Atomic`
- **Compile time** – the compiler generates special instructions (e.g., `lock` prefix on x86) to ensure atomicity.
- **Runtime** – operations are indivisible; no data races.
- **Memory ordering** – the default is `memory_order_seq_cst` (sequential consistency), but you can specify weaker orderings.

### ASCII Diagram – Effects on Compiler Optimisation
```
                        ┌─────────────────────┐
                        │   Compiler View      │
                        └─────────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
   ┌──────────┐              ┌──────────┐              ┌──────────┐
   │  const   │              │ volatile │              │ restrict │
   ├──────────┤              ├──────────┤              ├──────────┤
   │ Can be   │              │ Cannot be│              │ Can      │
   │ optimised│              │ optimised│              │ optimise │
   │ away     │              │ or reor- │              │ more     │
   │ (constant│              │ dered    │              │ aggres-  │
   │ propa-   │              │          │              │ sively   │
   │ gation)  │              │          │              │          │
   └──────────┘              └──────────┘              └──────────┘
                                   │
                                   ▼
                              ┌──────────┐
                              │ _Atomic  │
                              ├──────────┤
                              │ Special  │
                              │ hardware │
                              │ instr-   │
                              │ uctions  │
                              └──────────┘
```

---

## 7. Internal Architecture – Storage and Semantics

### Storage
- **`const`** – may be placed in `.rodata` (read‑only) for globals/statics.
- **`volatile`** – no special storage, but accesses are always to memory.
- **`restrict`** – no storage effect; only a compile‑time hint.
- **`_Atomic`** – storage is the same as the underlying type, but may have additional alignment requirements (e.g., lock‑free instructions).

### Semantic Rules
- **`const`** – cannot be assigned; cannot be used as lvalue in certain contexts.
- **`volatile`** – accesses are considered side effects; the compiler cannot eliminate them.
- **`restrict`** – applies to pointer declarations; the object is accessed only through that pointer (or its derived pointers).
- **`_Atomic`** – operations are indivisible; the compiler ensures atomicity.

---

## 8. Lifecycle / Workflow of a Qualified Variable

1. **Declaration** – the qualifier appears in the declaration.
2. **Type checking** – the compiler enforces qualifier rules (e.g., `const` cannot be assigned).
3. **Storage allocation** – based on type and qualifier.
4. **Initialisation** – `const` must be initialised.
5. **Use** – operations are restricted/guided by the qualifier.
6. **Program termination** – storage is released.

---

## 9. Practical Examples

### Example 1: `const` with Pointers
```c
#include <stdio.h>

int main(void) {
    int x = 10;
    int y = 20;

    // Pointer to const int: data is read‑only through this pointer.
    const int *p1 = &x;
    // *p1 = 15;  // ❌ Error – cannot modify data
    p1 = &y;      // ✅ OK – pointer itself can change

    // const pointer to int: pointer is fixed, data can change.
    int *const p2 = &x;
    *p2 = 15;     // ✅ OK – data can change
    // p2 = &y;   // ❌ Error – pointer cannot change

    // const pointer to const int: both fixed.
    const int *const p3 = &x;
    // *p3 = 15;  // ❌ Error – data cannot change
    // p3 = &y;   // ❌ Error – pointer cannot change

    printf("x: %d, y: %d\n", x, y);
    return 0;
}
```
**Code Breakdown:**
- Reading `const int *p1` from right to left: "p1 is a pointer to a const int".
- Reading `int *const p2` from right to left: "p2 is a const pointer to an int".
- Use the convention: `const` modifies what is immediately to its left, except when it's at the far left (then it modifies the rightmost thing).

### Example 2: `volatile` for Hardware Register
```c
#include <stdio.h>
#include <stdint.h>

// Simulate a hardware register at a fixed address
#define STATUS_REG ((volatile uint32_t *)0x40000000)
#define DATA_REG   ((volatile uint32_t *)0x40000004)

int main(void) {
    // Wait for status bit 0 to become 1
    while ((*STATUS_REG & 0x01) == 0) {
        // Without volatile, the compiler might optimise this to an infinite loop
        // because it would think the value never changes.
    }

    // Read data register
    uint32_t data = *DATA_REG;
    printf("Data: %u\n", data);

    // Write to data register
    *DATA_REG = 0xFF;

    return 0;
}
```
**Code Breakdown:**
- `volatile uint32_t *` – the pointer points to a volatile uint32_t.
- Every read of `STATUS_REG` generates a memory load; the compiler cannot cache the value.
- This pattern is common in embedded systems programming.

### Example 3: `restrict` for Performance
```c
#include <stdio.h>

// Without restrict – compiler must assume a and b may alias.
void add_no_restrict(int *a, int *b, int n) {
    for (int i = 0; i < n; i++) {
        a[i] += b[i];
    }
}

// With restrict – compiler knows they don't alias.
void add_restrict(int *restrict a, int *restrict b, int n) {
    for (int i = 0; i < n; i++) {
        a[i] += b[i];
    }
}

int main(void) {
    int x[5] = {1,2,3,4,5};
    int y[5] = {10,20,30,40,50};

    add_no_restrict(x, y, 5);
    // add_restrict(x, y, 5); // also works

    for (int i = 0; i < 5; i++) {
        printf("%d ", x[i]);
    }
    printf("\n");
    return 0;
}
```
**Code Breakdown:**
- `restrict` promises that the pointers do not overlap.
- The compiler can vectorise the loop or use SIMD instructions.
- Without `restrict`, the compiler must generate safe code that handles aliasing.

### Example 4: `_Atomic` for Thread‑Safe Counter
```c
#include <stdio.h>
#include <stdatomic.h>
#include <threads.h>
#include <stdlib.h>

#define NUM_THREADS 10
#define ITERATIONS 1000

atomic_int total = 0;

int worker(void *arg) {
    for (int i = 0; i < ITERATIONS; i++) {
        atomic_fetch_add(&total, 1);  // Atomic increment
    }
    return 0;
}

int main(void) {
    thrd_t threads[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++) {
        thrd_create(&threads[i], worker, NULL);
    }

    for (int i = 0; i < NUM_THREADS; i++) {
        thrd_join(threads[i], NULL);
    }

    // Guaranteed to be NUM_THREADS * ITERATIONS = 10000
    printf("Total: %d\n", atomic_load(&total));
    return 0;
}
```
**Code Breakdown:**
- `atomic_int` is a type alias for `_Atomic int`.
- `atomic_fetch_add` atomically adds 1 and returns the old value.
- Without `_Atomic`, this would have a data race and produce unpredictable results.
- The final total is always correct.

### Example 5: Combining Qualifiers
```c
#include <stdio.h>

// Example: read‑only hardware register (const + volatile)
// The value is read‑only but may change unexpectedly.
const volatile uint32_t *hardware_status = (const volatile uint32_t *)0x4000;

int main(void) {
    uint32_t status = *hardware_status; // reads from memory
    // *hardware_status = 0; // ❌ Error: cannot write (const)
    printf("Status: %#x\n", status);
    return 0;
}
```
**Code Breakdown:**
- `const volatile` – the value is read‑only (`const`) and may change unexpectedly (`volatile`).
- The compiler cannot cache the value and cannot allow writes.
- Common for read‑only hardware registers.

---

## 10. Common Use Cases

| Qualifier | Use Case | Example |
|-----------|----------|---------|
| `const` | Read‑only parameters | `void print(const char *s);` |
| `const` | Configuration data | `const int MAX_BUFFER = 1024;` |
| `const` | String literals | `const char *msg = "Hello";` |
| `volatile` | Hardware registers | `volatile uint32_t *reg;` |
| `volatile` | Interrupt‑modified variables | `volatile int flag;` |
| `volatile` | Signal handlers | `volatile sig_atomic_t sig;` |
| `restrict` | High‑performance algorithms | `void matmul(float *restrict a, ...);` |
| `restrict` | Compiler optimisations | `memcpy(dest, src, n);` (often uses restrict) |
| `_Atomic` | Lock‑free data structures | `atomic_int counter;` |
| `_Atomic` | Shared counters | `atomic_fetch_add(&total, 1);` |
| Combined | Read‑only hardware registers | `const volatile uint32_t *reg;` |

---

## 11. Best Practices

### `const`
- **Use `const` for function parameters that are read‑only** – documents intent and allows optimisation.
- **Use `const` for global configuration values** – prevents accidental modification.
- **Use `const` when you don't need to modify a variable** – catches errors early.
- **Do not cast away `const`** – it's unsafe and leads to undefined behaviour.

### `volatile`
- **Use `volatile` only for variables that can change outside normal program flow** – e.g., hardware registers, interrupt‑modified variables.
- **Do not use `volatile` for thread synchronisation** – use `_Atomic` or mutexes.
- **Be aware that `volatile` does not provide atomicity** – you still need locks for compound operations.

### `restrict`
- **Use `restrict` when you are absolutely sure pointers do not alias** – misuse causes undefined behaviour.
- **Use `restrict` in performance‑critical code** – e.g., numerical kernels, standard library functions.
- **Be careful when passing `restrict` pointers to functions** – the caller must respect the promise.

### `_Atomic`
- **Prefer `_Atomic` over `volatile` for shared variables** – it provides atomicity and memory ordering.
- **Use `memory_order` parameters for performance** – sequential consistency is safe but may be slower.
- **Use `atomic_*` functions from `<stdatomic.h>`** – they provide portable atomic operations.

### General
- **Combining qualifiers** – e.g., `const volatile` is valid and common for hardware registers.
- **Qualifiers in type definitions** – can be used with `typedef` to create meaningful types.

---

## 12. Common Mistakes

### Mistake 1: Casting Away `const`
```c
// ❌ Wrong – unsafe, undefined behaviour if the object is truly const.
const int x = 10;
int *p = (int *)&x;
*p = 20;   // Undefined behaviour – x is const
// ✅ Correct – don't cast away const.
```

### Mistake 2: Using `volatile` for Thread Synchronisation
```c
// ❌ Wrong – volatile does not guarantee atomicity or ordering.
volatile int flag = 0;
// Thread 1:
flag = 1;
// Thread 2:
while (flag == 0) { }   // may work, but not guaranteed
// ✅ Correct – use atomic_int or a mutex.
atomic_int flag = 0;
```

### Mistake 3: Misusing `restrict`
```c
// ❌ Wrong – a and b may alias.
void add(int *restrict a, int *restrict b, int n) {
    for (int i = 0; i < n; i++) {
        a[i] += b[i];
    }
}
int main() {
    int x[10];
    add(x, x, 10);  // undefined behaviour – a and b alias
}
// ✅ Correct – only use restrict when you know they don't alias.
```

### Mistake 4: Forgetting to Include Headers for `_Atomic`
```c
// ❌ Wrong – missing header.
_Atomic int x = 0;
// ✅ Correct – include <stdatomic.h>.
#include <stdatomic.h>
atomic_int x = 0;
```

### Mistake 5: Assuming `const` Means "Compile‑Time Constant"
```c
// ❌ Wrong – const variable is not a constant expression in C.
const int SIZE = 10;
int arr[SIZE];   // May work in C99+ (VLA), but not a true constant expression.
// ✅ Correct – use #define or enum for true constants.
#define SIZE 10
int arr[SIZE];
```

### Mistake 6: Not Using `volatile` for Signal Handlers
```c
#include <signal.h>
#include <stdio.h>

int flag = 0;   // ❌ Should be volatile sig_atomic_t

void handler(int sig) {
    flag = 1;   // May be optimised away or partially read
}

int main() {
    signal(SIGINT, handler);
    while (flag == 0) { }  // May loop forever if flag is not volatile
}
// ✅ Correct:
volatile sig_atomic_t flag = 0;
```

---

## 13. Performance Considerations

- **`const`** – can enable constant propagation and dead‑code elimination, improving performance.
- **`volatile`** – disables optimisations, which may slow down code. Use only when necessary.
- **`restrict`** – can significantly improve performance by enabling vectorisation and other optimisations.
- **`_Atomic`** – operations may be slower than non‑atomic ones due to memory barriers and locks. Use only when needed.

---

## 14. Security Considerations

- **`const`** – not a security feature; `const` can be cast away. It's a tool for correctness, not a barrier.
- **`volatile`** – not thread‑safe; do not rely on it for security‑critical code.
- **`restrict`** – misuse can lead to undefined behaviour, which may be exploitable.
- **`_Atomic`** – helps prevent data races, improving security in multi‑threaded code.

---

## 15. Debugging Tips

- **`const`** – compile with `-Wcast-qual` to warn when casting away `const`.
- **`volatile`** – watch out for optimisations; use `-O0` when debugging to preserve `volatile` semantics.
- **`restrict`** – if you suspect aliasing issues, use `-Wrestrict` (GCC) to catch potential problems.
- **`_Atomic`** – use `-O2` to see performance improvements; atomic operations are correctly handled.
- **Use debuggers** – you can inspect variables even if they are `volatile` or `const`.

---

## 16. When to Use

- **Always consider `const`** – it's almost always a good idea for read‑only data.
- **Use `volatile`** – for hardware registers, interrupt variables, and signal handlers.
- **Use `restrict`** – in performance‑critical code where you control pointer aliasing.
- **Use `_Atomic`** – for shared variables in multi‑threaded code.

---

## 17. When Not to Use

- **Avoid `volatile` for thread synchronisation** – use `_Atomic` or mutexes.
- **Avoid `restrict` in public APIs** – unless you can enforce the no‑aliasing contract.
- **Avoid `_Atomic` for single‑threaded code** – unnecessary overhead.

---

## 18. Related Concepts

- **Type system** – qualifiers modify types.
- **Storage classes** – `static`, `extern`, `auto`, `register`, `_Thread_local`.
- **Memory model** – (C11) defines how `_Atomic` and `volatile` behave.
- **Optimisation** – how qualifiers affect compiler optimisations.
- **Undefined behaviour** – `restrict` and `volatile` have strict rules.
- **Signal handling** – `volatile sig_atomic_t` for safe access in signal handlers.

---

## 19. Did You Know?

- **`const`** was added in C89; before that, `#define` was used for constants.
- **`volatile`** was also added in C89, primarily for systems programming.
- **`restrict`** was inspired by Fortran's semantics, which assume no aliasing.
- **`_Atomic`** in C11 is part of a larger memory model that also defines `memory_order` sequences.
- **`volatile`** does not guarantee atomicity, but it does guarantee that reads/writes are not reordered by the compiler (though CPU may still reorder).
- **`const volatile`** is a common combination for hardware registers that are read‑only but can change (e.g., a status register).
- C23 may introduce new qualifiers like `_Unused`, though not standard yet.

---

## 20. Summary

- **Type qualifiers** refine the behaviour of types: `const` (immutable), `volatile` (no optimisation), `restrict` (no aliasing), `_Atomic` (atomicity).
- **`const`** – prevents modification, improves safety and optimisation.
- **`volatile`** – forces memory reads/writes, used for hardware and interrupts.
- **`restrict`** (C99) – promises no aliasing, enables aggressive optimisation.
- **`_Atomic`** (C11) – provides thread‑safe atomic operations.
- **Combinations** are allowed and useful (e.g., `const volatile`).
- **Best practices**: use `const` for read‑only, `volatile` for hardware, `restrict` for performance, `_Atomic` for threading.
- **Common mistakes**: casting away `const`, using `volatile` for threading, misusing `restrict`.
- **Understanding qualifiers** is essential for writing correct, safe, and performant C code.