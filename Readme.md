# 🐹 Go Low-Level Fundamentals
### Memory · Data Structures · Algorithms — Complete Study Guide

> **How to use this file**: Read each section top-to-bottom. Key concepts come first, then runnable examples, then senior-level mastery notes, and finally pitfalls & practice at the end of each topic.

---

## Table of Contents

1. [How Computers Store Data: Bytes & Bits](#1-how-computers-store-data-bytes--bits)
2. [Variables & Pointers](#2-variables--pointers)
3. [Big O Notation](#3-big-o-notation)
4. [Foundations of Go](#4-foundations-of-go)
5. [Arrays](#5-arrays)
6. [Slices](#6-slices)
7. [Maps](#7-maps)
8. [Practical Code Examples](#8-practical-code-examples)
9. [Go File System API](#9-go-file-system-api)
10. [Key Takeaways & Practice Problems](#10-key-takeaways--practice-problems)

---

## 1. How Computers Store Data: Bytes & Bits

### 📌 Core Concepts

| Term | Definition |
|------|-----------|
| **Bit** | Smallest unit of data — either `0` or `1` |
| **Byte** | 8 bits grouped together; the smallest *addressable* memory unit |
| **Combinations** | 1 byte = 2⁸ = **256** possible values |
| **Use** | Bytes represent characters, numbers, and all other data |

---

### Integer Types in Go

| Type | Signed? | Bits | Min Value | Max Value |
|------|---------|------|-----------|-----------|
| `int8` | ✅ Yes | 8 | -128 | 127 |
| `int16` | ✅ Yes | 16 | -32,768 | 32,767 |
| `int32` | ✅ Yes | 32 | -2,147,483,648 | 2,147,483,647 |
| `int64` | ✅ Yes | 64 | -9,223,372,036,854,775,808 | 9,223,372,036,854,775,807 |
| `uint8` | ❌ No | 8 | 0 | 255 |
| `uint16` | ❌ No | 16 | 0 | 65,535 |
| `uint32` | ❌ No | 32 | 0 | 4,294,967,295 |
| `uint64` | ❌ No | 64 | 0 | 18,446,744,073,709,551,615 |
| `int` | Arch | 32 or 64 | Arch-dependent | Arch-dependent |

> 💡 **Rule of thumb**: On a modern 64-bit system, `int` behaves like `int64`.

---

### Runnable Examples

```go
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    // ── Overflow: int8 max is 127 ──────────────────────────────
    var a int8 = 127
    fmt.Println(a + 1)   // Output: -128  ← wraps around (overflow)
    fmt.Println(a + 122) // Output: -7    ← also overflows

    // ── Bool size ─────────────────────────────────────────────
    var b bool = true
    fmt.Println(unsafe.Sizeof(b)) // Output: 1 byte (smallest addressable unit)

    // ── int size on 64-bit system ─────────────────────────────
    var c int = 7
    fmt.Println(unsafe.Sizeof(c))
    // Output: 8 (bytes)
    // Binary: 00000000 00000000 00000000 00000000
    //         00000000 00000000 00000000 00000111
}
```

---

### The `unsafe` Package

**Purpose**: Provides low-level memory operations that bypass Go's type safety.

| Function | What it does |
|----------|-------------|
| `unsafe.Sizeof(x)` | Returns size of `x` in bytes |
| `unsafe.Alignof(x)` | Returns memory alignment requirement |
| `unsafe.Pointer(p)` | Converts any pointer to a raw pointer |

> ⚠️ **Warning**: `unsafe` bypasses type safety. Only use it when you truly need low-level performance or C interop.

---

### 🎓 Senior Mastery: Two's Complement & Bitwise Ops

**Why overflow wraps around — Two's Complement:**

```
int8: 127 = 0111 1111
       + 1
      ─────────────
           1000 0000 = -128  ← bit pattern wraps!
```

Formula for negative: `~value + 1` gives the two's complement.

**Bitwise Operations:**

```go
u := uint64(0xAA) // 1010 1010
v := uint64(0xCC) // 1100 1100

u & v   // AND  → 0x88  (1000 1000) — both bits must be 1
u | v   // OR   → 0xEE  (1110 1110) — at least one bit must be 1
u ^ v   // XOR  → 0x66  (0110 0110) — bits must differ
^u      // NOT  → flips all 64 bits
u &^ v  // AND NOT (clear bits) → 0x22
u << 2  // Left shift  → multiply by 4
u >> 2  // Right shift → divide by 4
```

> 🚀 **Performance**: Bitwise ops run at >10 GB/s vs ~2 GB/s for arithmetic on modern CPUs. Use them for flags, masks, and performance-critical code.

**Endianness (byte order):**

```go
import "encoding/binary"

buf := make([]byte, 4)
binary.BigEndian.PutUint32(buf, 1)
// buf = [0, 0, 0, 1]  ← most significant byte first (network order)

binary.LittleEndian.PutUint32(buf, 1)
// buf = [1, 0, 0, 0]  ← least significant byte first (amd64/x86 default)
```

**Memory Alignment:**

```go
type Padded struct {
    a byte   // 1 byte + 7 bytes padding added by compiler
    b int64  // 8 bytes — must align to 8-byte boundary
}
fmt.Println(unsafe.Sizeof(Padded{}))   // 16 (not 9!)
fmt.Println(unsafe.Alignof(Padded{}))  // 8
```

> 💡 Misaligned memory access can cause **50% performance loss** on some CPUs.

**Benchmark — bit ops vs arithmetic:**

```go
func BenchmarkAnd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = (1 & 1) | 2
    }
}
// Run: go test -bench=.
// Result: Bit ops >10 GB/s vs arithmetic ~2 GB/s on M1
```

**Useful `math/bits` functions** (used in crypto & hashing):

```go
import "math/bits"

bits.OnesCount64(0xFF)   // 8  — count set bits (popcount)
bits.Len64(255)          // 8  — bit length
bits.ReverseBytes64(x)   // reverse byte order
```

### ⚠️ Common Pitfalls

- Signed right shift sign-extends (use `uint` if you want logical shift)
- Overflow in constant expressions causes a **compile-time panic**
- Use `atomic.Uint64` for bit flags in concurrent code

### 🏋️ Practice

- Implement a bit-matrix transpose
- Write a popcount histogram
- Build a custom CRC32 using only bit shifts

---

## 2. Variables & Pointers

### 📌 Core Concepts

A **variable** is a named container that holds a value in memory. It has:
- a **name** (identifier)
- a **type** (determines memory size and valid operations)
- a **value** (the data stored)
- an **address** (where it lives in RAM)

```
Variable x = 42
┌─────────────────────┐
│ Name:    x          │
│ Type:    int        │
│ Value:   42         │
│ Address: 0xc0000140a0 │
└─────────────────────┘
```

---

### Call by Value vs Call by Reference

| Mechanism | How it works | Effect on original |
|-----------|-------------|-------------------|
| **Call by Value** | Copies the value | No effect — original unchanged |
| **Call by Reference** | Copies the memory address (pointer) | Original IS modified |

> Go's **default is call by value**. To share/mutate, use pointers.

---

### Runnable Examples

```go
package main

import "fmt"

func main() {
    x := 42
    fmt.Println(x)  // Output: 42
    fmt.Println(&x) // Output: 0xc0000140a0 (memory address — changes every run!)

    // Pointer: p stores the address of x
    p := &x
    fmt.Println(*p) // Output: 42 (dereference: read value AT that address)

    // Modifying through pointer
    *p = 100
    fmt.Println(x)  // Output: 100 — x was changed via pointer!
}
```

> 💡 **Why does the address change every run?** ASLR (Address Space Layout Randomization) — a security feature of modern OSes that randomizes where programs are placed in memory.

---

### 🎓 Senior Mastery: Pointers Deep Dive

**Variable declaration styles:**

```go
var x int = 10   // explicit type, works globally
y := 20          // short declaration, type inferred, local only
x = 30           // reassignment (no :=)

// Shadowing — dangerous but legal:
x := 1
if true {
    x := 2       // NEW variable that shadows outer x
    _ = x
}
fmt.Println(x)   // Still 1 — outer x untouched
```

**Call by Value — structs copy everything:**

```go
type Big struct{ data [1000]int }

func byVal(b Big) {
    b.data[0] = 99   // modifies the copy, not the original
}

b := Big{}
byVal(b)
fmt.Println(b.data[0]) // 0 — original unchanged
```

**Call by Reference — pointer receivers:**

```go
func byRef(b *Big) {
    b.data[0] = 99   // modifies the ORIGINAL
}

b := Big{}
byRef(&b)
fmt.Println(b.data[0]) // 99 — original modified!
```

**Stack vs Heap — Escape Analysis:**

```go
// Heap escape: value returned as pointer, compiler moves it to heap
func escape() *int {
    x := new(int) // allocated on heap
    return x
}

// Stack: value returned by copy, stays on stack
func noEscape() int {
    i := 1
    return i // copied out, stack frame freed
}
```

To see escape decisions:

```bash
go build -gcflags="-m=2" ./...
# Output shows: "x escapes to heap"
```

**Benchmark — value copy vs pointer:**

```go
func BenchmarkCopyBig(b *testing.B) {
    type Large [1 << 10]int64  // 8 KB struct
    l := Large{}
    for i := 0; i < b.N; i++ {
        var copy Large = l   // copies 8 KB every iteration!
    }
}

func BenchmarkPtr(b *testing.B) {
    type Large [1 << 10]int64
    l := Large{}
    p := &l
    for i := 0; i < b.N; i++ {
        _ = *p   // just follows a pointer — near-zero cost
    }
}
// Result: Pointer is ~100x faster for large structs
```

**Unsafe pointer arithmetic (advanced):**

```go
import "unsafe"

arr := [5]int{10, 20, 30, 40, 50}
base := unsafe.Pointer(&arr[0])
size := unsafe.Sizeof(arr[0])

// Access arr[2] via raw pointer math
elemAddr := uintptr(base) + 2*size
val := *(*int)(unsafe.Pointer(elemAddr))
fmt.Println(val) // 30
```

**nil pointer panic — what it looks like:**

```
panic: runtime error: invalid memory address or nil pointer dereference
goroutine 1 [running]:
main.bad(...)
    /tmp/main.go:8
```

Always check: `if p != nil { ... }` before dereferencing.

### ⚠️ Common Pitfalls

- Forgetting `*` to dereference: `p` is an address, `*p` is the value
- Passing large structs by value in hot loops (use pointers instead)
- nil pointer dereference — most common Go runtime panic
- Variable shadowing hiding bugs in nested scopes

### 🏋️ Practice

- Implement a singly linked list using pointers
- Write an escape analysis benchmark suite
- Implement a cycle detector for a linked list (Floyd's algorithm)

---

## 3. Big O Notation

### 📌 Core Concepts

Big O describes **how runtime (or memory) scales** as input size `n` grows. It answers: *"If the input doubles, what happens to the time/space?"*

| Notation | Name | Example | Growth |
|----------|------|---------|--------|
| O(1) | Constant | Array index, formula | Same regardless of n |
| O(log n) | Logarithmic | Binary search | Doubles input → +1 step |
| O(n) | Linear | Single loop | Doubles input → doubles time |
| O(n log n) | Linearithmic | Merge sort | Slightly worse than linear |
| O(n²) | Quadratic | Nested loops | Doubles input → 4x time |
| O(2ⁿ) | Exponential | Recursive Fibonacci | Very slow, avoid |
| O(n!) | Factorial | Permutations | Catastrophically slow |

```
Speed:  O(1) > O(log n) > O(n) > O(n log n) > O(n²) > O(2ⁿ) > O(n!)
        FAST ────────────────────────────────────────────────── SLOW
```

---

### The Classic Example: O(n) vs O(1)

**O(n) — loop approach:**

```go
package main

import "fmt"

// O(n): time grows linearly with n
// For n=100: 100 iterations. For n=1,000,000: 1,000,000 iterations
func addUp(n int) {
    sum := 0
    for i := 1; i <= n; i++ {
        sum += i
    }
    fmt.Println(sum)
}
```

**O(1) — mathematical formula:**

```go
// O(1): always exactly 3 operations (multiply, add, divide)
// regardless of whether n = 10 or n = 1,000,000,000
func addUp2(n int) {
    sum := n * (n + 1) / 2
    fmt.Println(sum)
}

func main() {
    addUp(100)   // 5050 — loop ran 100 times
    addUp2(100)  // 5050 — computed instantly
}
```

> 📖 **Gauss's Story**: At age 8, Carl Friedrich Gauss was asked to sum 1 to 100. He noticed pairs: (1+100), (2+99), (3+98)... = 50 pairs × 101 = **5050**. This gives the formula: **S = n(n+1)/2**.

---

### 🎓 Senior Mastery

**Assembly view of the O(n) loop** (`go tool objdump`):

```asm
LOOP:
  CMP  i, n     ; compare i to n
  JGE  END      ; if i >= n, exit
  ADD  sum, i   ; sum += i
  INC  i        ; i++
  JMP  LOOP     ; go back
; ≈ 4 ops × n iterations = O(n)
```

**O(1) proof:**

```
sum = n * (n + 1) / 2
   = 1 multiply + 1 add + 1 divide
   = exactly 3 ops — always. No loop. O(1).
```

**Slice `append` — amortized O(1):**

Go's `append` is special. When the slice is full it doubles the capacity:

```
Capacities: 1 → 2 → 4 → 8 → ... → n
Copies made: 1 + 2 + 4 + ... + n/2 = n - 1
Average cost per append = (n-1)/n ≈ 1 = O(1) amortized
```

Go's growth strategy (from `src/runtime/slice.go`):

```go
// Simplified:
newcap := oldcap
for newcap < 1024 {
    newcap *= 2          // double until 1024
}
for newcap < wantcap*2 {
    newcap = (newcap + 1) * 3 / 2  // then grow by ~25%
}
```

**Append benchmark:**

```go
func BenchmarkAppend(b *testing.B) {
    for i := 0; i < b.N; i++ {
        s := []int{}
        for j := 0; j < 1000; j++ {
            s = append(s, j)  // amortized O(1) per append
        }
    }
}
// Run: go test -bench=BenchmarkAppend
```

> 💡 **Pre-allocate to avoid reallocations**: `s := make([]int, 0, 1000)` eliminates all copies when you know the final size.

**Proof of Gauss's formula by induction:**

```
Base case: n=1 → S = 1*(1+1)/2 = 1 ✓

Inductive step: Assume S(k) = k(k+1)/2
Show S(k+1) = (k+1)(k+2)/2:

S(k+1) = S(k) + (k+1)
       = k(k+1)/2 + (k+1)
       = (k+1)(k/2 + 1)
       = (k+1)(k+2)/2  ✓
```

### ⚠️ Common Pitfalls

- Big O ignores constants: O(3n) = O(n) for complexity purposes
- **Space complexity** matters too — a loop can be O(1) time but O(n) space
- Loop concurrency bugs — use the race detector: `go test -race`

### 🏋️ Practice

- Implement Fibonacci in O(1) using matrix exponentiation
- Analyze the Big O of your last project's core algorithm
- Benchmark an O(n²) vs O(n log n) sort on the same dataset

---

## 4. Foundations of Go

### 📌 Variable Declaration Cheat Sheet

| Syntax | Scope | Type Inferred? | Use case |
|--------|-------|----------------|----------|
| `var x int = 10` | Global or local | ❌ No | Explicit types, global vars |
| `var x = 10` | Global or local | ✅ Yes | When type is obvious |
| `x := 10` | Local only | ✅ Yes | Most common — idiomatic Go |
| `x = 10` | Local only | — | Reassignment only |

> ⚠️ `:=` **cannot be used at package (global) level** — that's a compile error.

---

### Packages

```go
// Creating a local package:
// File: mypkg/mypkg.go
package mypkg

func Foo() string {
    return "hello from mypkg"
}
```

```go
// main.go — using the local package
package main

import (
    "fmt"
    "mymodule/mypkg"
)

func main() {
    fmt.Println(mypkg.Foo())
}
```

```bash
go mod init mymodule   # initialize module
go run .               # run
```

---

### `go run` vs `go build`

| Command | What it does | When to use |
|---------|-------------|-------------|
| `go run .` | Compiles + runs in memory (no binary) | Development & testing |
| `go build` | Produces a binary executable | Deployment / production |
| `go build ./cmd/server` | Build a specific sub-command | Multi-binary projects |

**Multi-binary project layout:**

```
myproject/
├── cmd/
│   ├── server/
│   │   └── main.go   (package main — the web server)
│   └── cli/
│       └── main.go   (package main — the CLI tool)
└── internal/
    └── logic/
        └── logic.go  (shared business logic)
```

```bash
go build ./cmd/server   # builds the server binary
go build ./cmd/cli      # builds the CLI binary
```

---

### Strings in Go

- Strings are **immutable** sequences of bytes (`[]byte` internally)
- Indexing gives **bytes**, not characters: `s[0]` is a `byte`
- Use `range` to iterate over **runes** (Unicode code points):

```go
s := "hello, 世界"

// Byte iteration (wrong for multi-byte chars):
for i := 0; i < len(s); i++ {
    fmt.Printf("%x ", s[i])
}

// Rune iteration (correct for Unicode):
for i, r := range s {
    fmt.Printf("%d: %c\n", i, r)
}
```

---

## 5. Arrays

### 📌 Core Properties

1. **Fixed size** — set at compile time, cannot change
2. **Same type** — all elements must be the same type
3. **Contiguous memory** — elements are adjacent in RAM
4. **Zero-indexed** — first element at index `0`
5. **Value type** — assigning copies the entire array
6. **Length is part of type** — `[3]int` ≠ `[4]int`
7. **No resize** — need more space? Use a slice

---

### Syntax

```go
// Declaration and initialization
var numbers [5]int = [5]int{1, 2, 3, 4, 5}
fmt.Println(numbers)    // [1 2 3 4 5]
fmt.Println(len(numbers)) // 5
fmt.Println(cap(numbers)) // 5 (== len for arrays, always)

// Short syntax with inferred length
primes := [...]int{2, 3, 5, 7, 11}

// Zero-initialized array
var zeros [10]int
fmt.Println(zeros) // [0 0 0 0 0 0 0 0 0 0]
```

---

### O(1) Random Access — How it works

```
Array base address: 0x1000
Element size: 8 bytes (int64)

Element at index i:
  Address = base + i × sizeof(T)
  arr[0] → 0x1000 + 0 × 8 = 0x1000
  arr[2] → 0x1000 + 2 × 8 = 0x1010
  arr[4] → 0x1000 + 4 × 8 = 0x1020

One arithmetic operation → O(1) regardless of array size
```

---

### `unsafe` Pointer Walk Through Array

```go
import "unsafe"

x := [5]int{1, 2, 3, 4, 5}
fmt.Println(unsafe.Sizeof(x)) // 40 (5 elements × 8 bytes each)

base := unsafe.Pointer(&x[0])
size := unsafe.Sizeof(x[0])

// Access x[2] via raw pointer arithmetic
elemAddr := uintptr(base) + 2*size
val := *(*int)(unsafe.Pointer(elemAddr))
fmt.Println(val) // 3
```

---

### 🎓 Senior Mastery

**Arrays as fixed buffers:**

```go
// Efficient fixed-size buffer — no heap allocation
var buf [256]byte
n, _ := reader.Read(buf[:])
process(buf[:n])
```

**Multidimensional arrays:**

```go
// 3×3 matrix
var matrix [3][3]int = [3][3]int{
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9},
}
fmt.Println(matrix[1][2]) // 6
```

**Value type gotcha:**

```go
a := [3]int{1, 2, 3}
b := a         // b is a COPY of a
b[0] = 99
fmt.Println(a) // [1 2 3] — a is unchanged
fmt.Println(b) // [99 2 3]
```

### ⚠️ Common Pitfalls

- Arrays are value types — passing to functions copies all elements
- `[3]int` and `[4]int` are **different types** — not interchangeable
- When you assign an array to a slice (`s := arr[:]`), the slice points into the array's memory — modifications through the slice will change the array

---

## 6. Slices

### 📌 Arrays vs Slices

| Feature | Array | Slice |
|---------|-------|-------|
| Size | Fixed at compile time | Dynamic |
| Memory | Value type (copy on assign) | Reference type (shared backing array) |
| Resize | ❌ Cannot | ✅ `append()` |
| Header size | N/A | 24 bytes (ptr + len + cap) |
| Use case | Fixed buffers, low-level | Almost everything else |

---

### The Slice Header (Internal Anatomy)

Every slice is a 3-field struct in Go's runtime:

```
┌─────────────────────────────────────────────────┐
│  Slice Header (24 bytes)                        │
│  ┌──────────┬──────────┬──────────┐             │
│  │ Ptr      │ Len      │ Cap      │             │
│  │ (8 bytes)│ (8 bytes)│ (8 bytes)│             │
│  └────┬─────┴──────────┴──────────┘             │
│       │                                         │
└───────┼─────────────────────────────────────────┘
        │
        ▼  Underlying Array (in heap or stack)
   ┌────────┬────────┬────────┬────────┬────────┐
   │  [0]   │  [1]   │  [2]   │  [3]   │  [4]   │
   │   1    │   2    │   3    │   4    │   5    │
   └────────┴────────┴────────┴────────┴────────┘
    0xa0     0xa8     0xb0     0xb8     0xc0
        ↑
       ptr (Len covers [0..2], Cap covers [0..4])
```

---

### Creating Slices

```go
// Literal
s := []int{1, 2, 3, 4, 5}

// make(type, len, cap) — preferred when size is known
s2 := make([]int, 3, 10)   // len=3, cap=10
fmt.Println(len(s2), cap(s2)) // 3 10

// From array (shares memory!)
arr := [5]int{10, 20, 30, 40, 50}
sl := arr[1:4]  // [20, 30, 40]  len=3, cap=4

// nil slice (valid, zero value)
var ns []int
fmt.Println(ns == nil) // true
fmt.Println(len(ns))   // 0 — safe to use
```

---

### Slicing Syntax `[low:high:max]`

```go
s := []int{0, 1, 2, 3, 4, 5}

s[1:4]   // [1, 2, 3]           len=3, cap=5 (shares backing array)
s[:3]    // [0, 1, 2]           len=3, cap=6
s[3:]    // [3, 4, 5]           len=3, cap=3
s[:]     // [0, 1, 2, 3, 4, 5]  full slice

// Three-index slice (limits cap — prevents unwanted sharing):
s[1:3:4] // [1, 2]              len=2, cap=3 (cannot grow into s[3:])
```

Rules: `low` is inclusive, `high` is exclusive.

---

### `append` and Reallocation

```go
s := []int{1, 2, 3}     // len=3, cap=3

// When cap is sufficient → O(1), no allocation
s = append(s, 4)        // works in-place

// When cap is full → allocate new array (~2x cap), copy, update ptr
s = append(s, 5, 6, 7)  // new backing array allocated!
```

**The shared-memory trap:**

```go
s1 := []int{1, 2, 3, 4, 5, 6}   // len=6, cap=6
s2 := s1[1:3]                    // [2, 3]  len=2, cap=5 — shares s1's array!

s2 = append(s2, 100, 200, 300)   // writes into s1's backing array!
fmt.Println(s1)                   // [1, 2, 3, 100, 200, 300] — s1 was mutated!
```

**Fix — use `copy` to prevent sharing:**

```go
s1 := []int{1, 2, 3, 4, 5}
s2 := make([]int, len(s1[1:3]))
copy(s2, s1[1:3])                 // independent copy
s2 = append(s2, 100)
fmt.Println(s1)                   // [1, 2, 3, 4, 5] — s1 safe
```

---

### 🎓 Senior Mastery

**Nil slice vs empty slice:**

```go
var ns []int       // nil slice: nil == true, len=0, cap=0
es := []int{}      // empty slice: nil == false, len=0, cap=0

// Both are safe to append to:
ns = append(ns, 1)
es = append(es, 1)

// But they serialize differently in JSON:
// nil slice → null
// empty slice → []
```

**Pre-allocate for performance:**

```go
// BAD — reallocates many times
result := []int{}
for i := 0; i < 10000; i++ {
    result = append(result, i)
}

// GOOD — pre-allocate, zero reallocations
result := make([]int, 0, 10000)
for i := 0; i < 10000; i++ {
    result = append(result, i)
}
```

**Slice tricks:**

```go
s := []int{1, 2, 3, 4, 5}

// Delete element at index 2 (order preserved):
s = append(s[:2], s[3:]...)  // [1, 2, 4, 5]

// Delete element at index 2 (order NOT preserved, faster):
s[2] = s[len(s)-1]
s = s[:len(s)-1]             // [1, 2, 5, 4]

// Insert at index 2:
s = append(s[:2+1], s[2:]...)
s[2] = 99
```

### ⚠️ Common Pitfalls

- Sub-slices share the backing array — mutations propagate unexpectedly
- Bounds panic: accessing `s[len(s)]` is a runtime panic (no wraparound)
- Large slices escape to the heap — can increase GC pressure
- `append` may return a **new** slice header — always reassign: `s = append(s, x)`

---

## 7. Maps

### 📌 Core Concepts

A **map** is Go's built-in hash table. It provides O(1) average-case lookups, insertions, and deletions.

```
Key ──hash──► bucket ──► Value
```

Maps are **reference types** — passing a map to a function shares it (like slices).

---

### Syntax

```go
// Create
m := make(map[string]int)
m2 := map[string]int{"a": 1, "b": 2}  // literal

// Insert / update
m["hello"] = 42     // insert
m["hello"] = 99     // update (overwrites silently)

// Read
val := m["hello"]          // 99
val, ok := m["missing"]    // val=0, ok=false (zero value if missing)

// Delete
delete(m, "hello")

// Check existence
if v, ok := m["key"]; ok {
    fmt.Println("found:", v)
}

// Iterate (order is random — by design!)
for k, v := range m {
    fmt.Printf("%s → %d\n", k, v)
}

// Length
fmt.Println(len(m)) // O(1)
```

---

### 🎓 Senior Mastery

**Capacity hint for performance:**

```go
// Pre-size the map to avoid rehashing
m := make(map[string]int, 1000)
```

**Map iteration is intentionally random** — Go randomizes it on purpose (security: prevents hash-flooding attacks). Never rely on iteration order.

**Zero value on missing key:**

```go
m := map[string]int{}
m["count"]++    // safe! missing key returns 0, then increments to 1
fmt.Println(m["count"]) // 1
```

**Concurrent maps — use `sync.Map`:**

```go
import "sync"

var m sync.Map
m.Store("key", 42)
val, ok := m.Load("key")
m.Delete("key")
m.Range(func(k, v any) bool {
    fmt.Println(k, v)
    return true // continue
})
```

> ⚠️ Regular `map` is **NOT safe for concurrent use** — causes a fatal race condition. Use `sync.Map` or protect with a `sync.RWMutex`.

**Valid key types** — must be comparable with `==`:

| Valid | Invalid |
|-------|---------|
| `int`, `string`, `bool` | `slice` |
| `struct` (if all fields comparable) | `map` |
| pointers | `func` |

### ⚠️ Common Pitfalls

- Reading a missing key returns zero value — always use the two-value form `v, ok := m[k]` when existence matters
- Never write to a nil map — causes a panic: `assignment to entry in nil map`
- Maps aren't ordered — use `sort.Slice` + a keys slice if you need sorted output
- Concurrent access without synchronization → **fatal error: concurrent map read and map write**

---

## 8. Practical Code Examples

### `formatUserName` — String Manipulation

```go
package main

import (
    "fmt"
    "strings"
)

func formatUserName(name, role string) string {
    switch role {
    case "admin":
        return fmt.Sprintf("%s_ADMIN", strings.ToUpper(name))
    case "guest":
        return fmt.Sprintf("guest_%s", strings.ToLower(name))
    case "mod":
        return capitalizeEachWord(name) + ".mod"
    default:
        return "unknown role"
    }
}

// capitalizeEachWord capitalizes the first letter of each word
// by operating directly on bytes (ASCII only)
func capitalizeEachWord(s string) string {
    b := []byte(s)
    for i := 0; i < len(b); i++ {
        if i == 0 || b[i-1] == ' ' {
            b[i] = b[i] - 32  // ASCII: lowercase - 32 = uppercase
        }
    }
    return string(b)
}

func main() {
    fmt.Println(formatUserName("john doe", "admin"))  // JOHN DOE_ADMIN
    fmt.Println(formatUserName("jane", "guest"))      // guest_jane
    fmt.Println(formatUserName("bob smith", "mod"))   // Bob Smith.mod
    fmt.Println(formatUserName("alice", "unknown"))   // unknown role
}
```

> 💡 **`b[i] - 32`**: In ASCII, `'a'` = 97 and `'A'` = 65. The difference is always 32. Subtracting 32 from a lowercase letter gives the uppercase version. Only works for ASCII — use `strings.Title` or `unicode.ToUpper` for Unicode.

---

### `converter` — Binary/Hex String Parser

```go
package main

import (
    "fmt"
    "strconv"
)

// converter takes a slice of strings representing binary or hex numbers,
// sums the valid ones, and counts the invalid ones.
// Returns []int{sum, invalidCount}
func converter(s []string) []int {
    sum, flag := 0, 0
    for _, str := range s {
        // Try parsing as binary (base 2)
        v, err := strconv.ParseInt(str, 2, 64)
        if err != nil {
            // Try parsing as hexadecimal (base 16)
            v, err = strconv.ParseInt(str, 16, 64)
            if err != nil {
                flag++    // invalid in both bases
                continue
            }
        }
        sum += int(v)
    }
    return []int{sum, flag}
}

func main() {
    input := []string{"1010", "FF", "invalid", "1111", "AB"}
    result := converter(input)
    fmt.Printf("Sum: %d, Invalid count: %d\n", result[0], result[1])
    // Sum: 10 + 255 + 15 + 171 = 451, Invalid: 1
}
```

---

### `isAnagram` — LeetCode 242, O(n) Solution

```go
package main

import "fmt"

// isAnagram returns true if s and t contain the exact same characters
// with the same frequencies.
// O(n) time, O(1) space (at most 26 unique chars for lowercase ASCII)
func isAnagram(s, t string) bool {
    if len(s) != len(t) {
        return false
    }
    seen := make(map[rune]int)
    for _, c := range s {
        seen[c]++   // count chars in s
    }
    for _, c := range t {
        seen[c]--
        if seen[c] < 0 {
            return false  // t has more of this char than s
        }
    }
    return true
}

func main() {
    fmt.Println(isAnagram("anagram", "nagaram"))  // true
    fmt.Println(isAnagram("rat", "car"))          // false
    fmt.Println(isAnagram("listen", "silent"))    // true
}
```

---

## 9. Go File System API

### `os` Package — Common Operations

```go
import (
    "fmt"
    "os"
)

// ── Create / Write ────────────────────────────────────────────
f, err := os.Create("file.txt")
if err != nil { panic(err) }
defer f.Close()
f.WriteString("Hello, Go!\n")

// ── Open / Read ───────────────────────────────────────────────
data, err := os.ReadFile("file.txt")  // simplest: whole file at once
if err != nil { panic(err) }
fmt.Println(string(data))

// ── Append to existing file ───────────────────────────────────
f2, _ := os.OpenFile("file.txt", os.O_APPEND|os.O_WRONLY, 0644)
defer f2.Close()
f2.WriteString("Another line\n")

// ── Remove ────────────────────────────────────────────────────
os.Remove("file.txt")

// ── Directory ────────────────────────────────────────────────
os.Mkdir("mydir", 0755)
os.MkdirAll("a/b/c", 0755)  // creates all intermediate dirs

// ── List directory ────────────────────────────────────────────
entries, _ := os.ReadDir(".")
for _, e := range entries {
    fmt.Println(e.Name(), e.IsDir())
}

// ── File info ────────────────────────────────────────────────
info, _ := os.Stat("file.txt")
fmt.Println(info.Size(), info.Mode())

// ── Permissions ──────────────────────────────────────────────
os.Chmod("file.txt", 0600)  // owner read/write only
```

**Common `os.OpenFile` flags:**

| Flag | Meaning |
|------|---------|
| `os.O_RDONLY` | Read only |
| `os.O_WRONLY` | Write only |
| `os.O_RDWR` | Read + write |
| `os.O_CREATE` | Create if not exists |
| `os.O_APPEND` | Append to file |
| `os.O_TRUNC` | Truncate to zero length |

**Senior tip — `embed.FS` for bundling files into the binary:**

```go
import _ "embed"

//go:embed config.yaml
var config []byte   // config.yaml baked into the binary at compile time
```

---

## 10. Key Takeaways & Practice Problems

### 🧠 Core Mental Models

| Concept | Remember This |
|---------|-------------|
| **Memory** | Everything is addresses and bytes. `&x` = address, `*p` = value at address |
| **Arrays** | Fixed, value type, contiguous. O(1) access via `base + i × size` |
| **Slices** | 24-byte header (ptr + len + cap). Shares backing array until realloc |
| **Maps** | Hash table, reference type, random iteration order, not thread-safe |
| **Big O** | Measures growth rate — O(1) > O(log n) > O(n) > O(n²) |
| **Pointers** | Default is call-by-value. Use `*T` to share/mutate across functions |
| **Overflow** | Wraps around silently at runtime. Check types carefully |

---

### 📚 Essential Commands

```bash
go run .                          # run project
go build ./...                    # build all
go test ./...                     # run all tests
go test -bench=. ./...            # run benchmarks
go test -race ./...               # race condition detector
go build -gcflags="-m=2" ./...    # show escape analysis
go tool objdump -s main.main ./binary  # view assembly
```

---

### 🏋️ Practice Problems (Ordered by Difficulty)

**Beginner:**
- [ ] Write a function that counts how many times each character appears in a string using a map
- [ ] Implement `reverseSlice` in-place without allocating a new slice
- [ ] Demonstrate variable shadowing with a test that catches the bug

**Intermediate:**
- [ ] Implement a singly linked list with pointer-based nodes
- [ ] Write a safe `pop` function for slices that panics gracefully with a message
- [ ] Parse a log file using `os.ReadFile` and group lines by severity using a map

**Advanced:**
- [ ] Benchmark append reallocation: pre-allocated vs dynamic — measure ns/op
- [ ] Implement a concurrent word counter using `sync.Map`
- [ ] Write a generic `Filter[T any]` function using Go generics
- [ ] Implement Floyd's cycle detection on a pointer-based linked list
- [ ] Build a custom binary serializer using `encoding/binary` and `unsafe.Sizeof`

---

### 📖 Further Reading

| Resource | What it covers |
|----------|---------------|
| [The Go Specification](https://go.dev/ref/spec) | Full language reference |
| `go doc unsafe` | unsafe package documentation |
| `go doc sync` | sync.Map, Mutex, RWMutex |
| [Go Blog: Slices internals](https://go.dev/blog/slices-intro) | Deep dive on slices |
| [Go Blog: Laws of Reflection](https://go.dev/blog/laws-of-reflection) | Types at runtime |
| [Effective Go](https://go.dev/doc/effective_go) | Idiomatic patterns |

---

*Study tip: Re-read one section per day, run every example, and do one practice problem. You'll have mastered this material in under two weeks.*
