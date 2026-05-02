# Go Low-Level Fundamentals: Memory, Data Structures &amp; Algorithms

A comprehensive, study-friendly reorganization of meeting notes.

## Table of Contents
1. [How Computers Store Data: Bytes &amp; Bits](#how-computers-store-data-bytes--bits)
2. [Variables &amp; Pointers](#variables--pointers)
3. [Big O Notation](#big-o-notation)
4. [Foundations of Go](#foundations-of-go)
5. [Arrays](#arrays)
6. [Slices](#slices)
7. [Code Examples](#code-examples)
8. [Maps](#maps)
9. [Go File System API](#go-file-system-api)

## How Computers Store Data: Bytes & Bits

**Summary**: The smallest addressable unit is a **byte** (8 bits, 256 combos: 2^8). Bits are 0/1.

### Integer Types Table
| Type    | Signed? | Bits | Range (Signed)                          | Range (Unsigned)                  |
|---------|---------|------|-----------------------------------------|-----------------------------------|
| int8   | Yes    | 8    | -128 to 127                            | uint8: 0-255                      |
| int16  | Yes    | 16   | -32,768 to 32,767                      | uint16: 0 to 65,535               |
| int32  | Yes    | 32   | -2^31 to 2^31-1                        | uint32: 0 to 4,294,967,295        |
| int64  | Yes    | 64   | -9.22e18 to 9.22e18                     | uint64: 0 to 18.44e18             |
| int    | Arch   | 32/64| Platform-dependent                      | -                                 |

**Original Examples**:
```go
import (
	"fmt"
	"unsafe"
)

var a int8 = 127
fmt.Println(a + 1)  // -128 (overflow)
fmt.Println(a + 122) // -1 (overflow)
```

**bool**: 1 byte (`unsafe.Sizeof(bool) == 1`).

**int**: 8 bytes on 64-bit (`unsafe.Sizeof(int) == 8`).

**unsafe package**: Low-level memory ops (pointer arith, C interop). **Caution**: Undefined behavior if misused.

```go
var b bool = true
fmt.Println(unsafe.Sizeof(b)) // 1

var c int = 7
fmt.Println(unsafe.Sizeof(c)) // 8 (binary: 00000111 padded)
```

**Senior Mastery**:
- **Overflow wraps around** (two's complement).
- **Bit ops**: `& | ^ &^ << >>` for masks/shifts.
- **Endianness**: Little-endian on x86/ARM. Use `binary.BigEndian` for networks.
- **Alignment**: Types padded (e.g., struct fields align to size for cache efficiency).

**Key Takeaway**: Know ranges to avoid overflows. Use `unsafe` sparingly for perf-critical (e.g., serialization).

## Variables & Pointers

**Summary**: Containers for data. Name + type + value. Memory address via `&`, deref `*`.

**Original Notes**:
- Hex address runtime-changes.
- **Call-by-value** (default): Copy value.
- **Call-by-ref**: Pointers share memory.

```go
x := 42
fmt.Println(x)   // 42
fmt.Println(&x)  // e.g., 0xc0000140a0 (changes per run)

p := &x
fmt.Println(*p)  // 42
```

**Senior Mastery**:
- **Stack/Heap**: Vars on stack (fast). Escape analysis promotes to heap if lives beyond func/loop.
- **Pointer pitfalls**: Nil deref panic. Use `new(T)` or `&T{}`.
- **Call semantics deep**: Structs copied fully (expensive for large).

**Key Takeaway**: Pointers for mutation/efficiency. Go hides refcounting (GC).

## Big O Notation

**Summary**: Time complexity.

**O(n) Loop**:
```go
func addUp(n int) {
	sum := 0
	for i := 1; i <= n; i++ {  // Linear
		sum += i
	}
}
```

**O(1) Formula**:
```go
func addUp2(n int) {
	sum := n * (n + 1) / 2  // Constant!
}
```

**Gauss Story**: Young Gauss summed 1..100 in O(1): pair 1+100=101, 2+99=101, ..., 50 pairs = `50*101=5050`. General: `n*(n+1)/2`.

**Senior**: Append slices **amortized O(1)**. Realloc doubles capacity:
```
Old cap 5 -> new 10 (copy 5 els)
Append cost: O(n) rare, avg O(1) over many.
Proof: Total cost for n appends ~2n copies.
```

**Key**: Optimize worst-case, but understand amortized.

## Foundations of Go

**Original**: `=` assign, `:=` declare+assign (local). `var` explicit/global.

| Decl       | Scope | Type? | Notes |
|------------|-------|-------|-------|
| `var x int`| Any  | Yes  | Global ok |
| `x := 42` | Local| Infer| Concise |
| `= `      | -    | No   | Reassign |

**Added Senior**:
- **Packages**: `import "fmt"`. Create local: `mkdir mypkg; mypkg/mypkg.go` with `package mypkg; func Foo()`. Import as `import "./mypkg"`. `go mod init` for modules.
```go
// main.go
import "./mypkg"
func main() { mypkg.Foo() }
```
- **go run vs go build**:
  | Command   | Does?                  | Use |
  |-----------|------------------------|-----|
  | `go run .`| Compile+run in-memory | Dev/test |
  | `go build`| Binary executable     | Deploy/prod |
  `go build` links deps statically.
- **Multi-main**: Separate dirs (`cmd/server`, `cmd/cli`), each `package main`. Build separately: `go build ./cmd/server`.

**Strings**: `[]byte` internally (immutable).

**Key**: Modules > GOPATH. Multi-main for CLI+server.

## Arrays

**Properties** (Original):
1. Fixed size.
2. Same type.
3. Contiguous mem.
4. 0-index.
5. Value type.
6. Len in type.
7. No resize.

**Syntax**:
```go
var numbers [5]int = [5]int{1,2,3,4,5}
fmt.Println(numbers) // [1 2 3 4 5]
```

**len/cap**: Equal, fixed.

**O(1) Access**:
```
Addr[i] = base + i * sizeof(T)
```

**Original unsafe Ex**:
```go
import "unsafe"
x := [5]int{1,2,3,4,5}
fmt.Println(unsafe.Sizeof(x)) // 40 (5*8)
base := unsafe.Pointer(&x[0])
size := unsafe.Sizeof(x[0])
elemAddr := uintptr(base) + 2*size
val := *(*int)(unsafe.Pointer(elemAddr)) // 3
```

**Senior**: Use for buffers (e.g., [256]byte). Multidim: `[3][3]int`.

**Pitfalls**: Decay to slice on assign.

## Slices

**vs Arrays**:
| Feature| Array | Slice |
|--------|-------|-------|
| Size  | Fixed| Dynamic |
| Mem   | Value| Ref (24 bytes header) |
| Resize| No   | append |

**Header Anatomy** (ASCII):
```
Slice: [Ptr ──┐
        Len: 3 │ Underlying Array: [0xa0:1][0xa8:2][0xb0:3][0xb8:4][0xc0:5]
        Cap: 5 ]               ↑ptr     ↑len=3
```

**Syntax**: `[]T`, `make([]T, len, cap)`.

**Slicing `[low:high]`** (max optional):
- low incl, high excl.
- Mods shared!

**Append/Realloc**:
1. Enough cap? Add O(1).
2. No? New array ~2x cap, copy, update ptr.
Strategy: Double to ~1024, then +25%.

**Exs** (Original):
```go
s1 := []int{1,2,3,4,5,6}
s2 := s1[1:3]     // len=2, cap=5
s2 = append(s2, 100,200,300) // Mods s1 up to cap!
```

**Senior Pitfalls**:
- Nil slice: `var s []int; s == nil` (append ok).
- `copy(dst, src)` prevents shared mem.
- Bounds panic (no wrap).
- Escape: Large slices heap-alloc.

## Code Examples

**formatUserName** (Complete):
```go
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

func capitalizeEachWord(s string) string {
	b := []byte(s)
	for i := 0; i < len(b); i++ {
		if i == 0 || b[i-1] == ' ' {
			b[i] = b[i] - 32 // ASCII upper
		}
	}
	return string(b)
}

func main() {
	fmt.Println(formatUserName("john doe", "admin")) // JOHN DOE_ADMIN
}
```

**converter** (Fixed w/ import/outputs):
```go
// Converts bin/hex slice to dec sum, counts invalid.
import (
	"fmt"
	"strconv"
)

func converter(s []string) []int {
	sum, flag := 0, 0
	for _, str := range s {
		v, err := strconv.ParseInt(str, 2, 64)
		if err != nil {
			v, err = strconv.ParseInt(str, 16, 64)
			if err != nil {
				flag++
				continue
			}
		}
		sum += int(v)
	}
	return []int{sum, flag}
}
```
**Vault Decoder**: Never crash, sum valids, count skips.

**Anagram (242)**: O(n) map.
```go
func isAnagram(s, t string) bool {
	if len(s) != len(t) { return false }
	seen := make(map[rune]int)
	for _, c := range s { seen[c]++ }
	for _, c := range t {
		seen[c]--
		if seen[c] < 0 { return false }
	}
	return true
}
```

## Maps

**Original**: Ref type, hash table. Key→value contract.

```go
m := make(map[int]string)
m[1] = "a" // Overwrite if exists
val, ok := m[3] // "", false if missing
delete(m, 1)
```

**Senior**:
- `len(m)` O(1), iter random (security).
- Capacity: `make(map[K]V, hint)`.
- Zero val on miss.
- Concurrency: `sync.Map`.

## Go File System API

**os pkg**: Create/Open/Read/Write/Remove/Mkdir/Chmod.

**Senior**: Use `ioutil.ReadFile` simple; `os.DirFS` for embed.FS.

## Key Takeaways & Practice
- **Memory**: Everything addresses/bytes.
- **Slices/Maps**: Refs, watch sharing.
- Practice: Benchmark append realloc, unsafe perf.
- Resources: Go spec, `go doc unsafe`.

This file is now **mastery-ready**! Open in Markdown viewer.
