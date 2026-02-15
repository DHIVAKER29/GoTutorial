# 20 - Arrays

> Fixed-size collections in Go - understanding the foundation before slices.

---

## 📌 What You'll Learn

- What arrays are and how they differ from other languages
- Why Go has both arrays AND slices
- Array declaration, initialization, and operations
- Memory layout and performance implications
- When to use arrays vs slices
- Common pitfalls and best practices
- Sample programs for every concept

---

## 🤔 What is an Array?

### Definition

> An **array** is a fixed-size, ordered collection of elements of the same type.

### Real-World Analogy

Think of an array like an apartment building with a fixed number of units. Each unit holds one tenant (element). You cannot add more units—the building is complete. All units are the same type (only humans can live there). You access units by index (Unit 0, 1, 2, etc.).

```go
var apartments [5]string
apartments[0] = "Alice"
apartments[1] = "Bob"
// apartments[5] = "Eve"  // ❌ Index out of bounds!
```

---

## 📊 Array Memory Layout

- **Contiguous storage**: Elements are stored next to each other in memory
- **Fixed size**: Each `int` takes 8 bytes (on 64-bit); total = length × element size
- **O(1) access**: Address of `arr[i]` = base + (i × element size)
- **Example**: `var nums [5]int = [5]int{10, 20, 30, 40, 50}` uses 40 bytes contiguous

---

## 📝 Array Declaration & Initialization

### Declaration methods

```go
var arr1 [5]int
fmt.Println(arr1)  // Output: [0 0 0 0 0]
```

```go
arr2 := [3]int{10, 20, 30}
fmt.Println(arr2)  // Output: [10 20 30]
```

```go
arr3 := [...]int{1, 2, 3, 4, 5}
fmt.Println(len(arr3))  // Output: 5
```

```go
arr4 := [5]int{0: 100, 2: 200, 4: 300}
fmt.Println(arr4)  // Output: [100 0 200 0 300]
```

```go
matrix := [2][3]int{{1, 2, 3}, {4, 5, 6}}
fmt.Println(matrix[1][2])  // Output: 6
```

### Complete Example: Array Declaration

```go
package main

import "fmt"

func main() {
    var arr1 [5]int
    arr2 := [3]int{10, 20, 30}
    arr3 := [...]int{1, 2, 3, 4, 5}
    arr4 := [5]int{0: 100, 2: 200, 4: 300}
    matrix := [2][3]int{{1, 2, 3}, {4, 5, 6}}

    fmt.Println(arr1)        // [0 0 0 0 0]
    fmt.Println(arr2)        // [10 20 30]
    fmt.Println(len(arr3))   // 5
    fmt.Println(arr4)        // [100 0 200 0 300]
    fmt.Println(matrix[1][2]) // 6
}
```

**Output:**
```
[0 0 0 0 0]
[10 20 30]
5
[100 0 200 0 300]
6
```

---

## 🔧 Array Operations

### Accessing and modifying

```go
arr := [5]int{10, 20, 30, 40, 50}
fmt.Println(arr[0])           // Output: 10
fmt.Println(arr[len(arr)-1])  // Output: 50
arr[2] = 300
fmt.Println(arr)  // Output: [10 20 300 40 50]
```

### Iteration

```go
arr := [3]int{10, 20, 30}
for i, v := range arr {
    fmt.Printf("%d:%d ", i, v)
}
// Output: 0:10 1:20 2:30
```

### Comparing arrays

```go
a := [3]int{1, 2, 3}
b := [3]int{1, 2, 3}
c := [3]int{1, 2, 4}
fmt.Println(a == b, a == c)  // Output: true false
```

### Complete Example: Array Operations

```go
package main

import "fmt"

func main() {
    arr := [5]int{10, 20, 30, 40, 50}
    arr[2] = 300

    sum := 0
    for _, v := range arr {
        sum += v
    }

    fmt.Println(arr)  // [10 20 300 40 50]
    fmt.Println(len(arr))  // 5
    fmt.Println(sum)  // 420
}
```

**Output:**
```
[10 20 300 40 50]
5
420
```

---

## ⚠️ Critical: Size is Part of the Type!

`[3]int` and `[4]int` are different types. You cannot assign one to the other or pass one to a function expecting the other.

```go
var a [3]int = [3]int{1, 2, 3}
var b [4]int = [4]int{1, 2, 3, 4}
// a = b  // ❌ compile error: cannot use b (type [4]int) as type [3]int
fmt.Printf("%T %T\n", a, b)  // Output: [3]int [4]int
```

---

## ⚠️ Arrays are Values (Copied!)

Assignment and function passing copy the entire array. Use pointers to modify the original.

```go
original := [3]int{1, 2, 3}
copy := original
copy[0] = 100
fmt.Println(original)  // Output: [1 2 3] (unchanged!)
fmt.Println(copy)      // Output: [100 2 3]
```

```go
func modifyArray(arr [3]int) {
    arr[0] = 999  // Modifies copy only
}
arr := [3]int{10, 20, 30}
modifyArray(arr)
fmt.Println(arr)  // Output: [10 20 30] (unchanged!)
```

**Performance**: Large arrays (e.g. `[1000000]int` ≈ 8MB) are copied on each assignment or function call. Use pointers or slices for large data.

---

## 🆚 Arrays vs Slices

| Feature | Array | Slice |
|---------|-------|-------|
| Size | Fixed | Dynamic |
| Type includes size | Yes `[5]int` | No `[]int` |
| Passed by | Value (copy) | Reference |
| Comparable (`==`) | Yes | No |
| Can grow | No | Yes (`append`) |
| Zero value | `[n]T` with zeros | `nil` |
| Use frequency | Rare (~5%) | Common (~95%) |

**When to use arrays**: Fixed-size data (IP address `[4]byte`), crypto keys `[32]byte`, matrix operations, value semantics.

**When to use slices**: Dynamic collections, function parameters, growing/shrinking data, reference semantics.

---

## 🏭 Production Use Cases

```go
var ipv4 [4]byte = [4]byte{192, 168, 1, 1}
// IPv4 addresses are always 4 bytes
```

```go
hash := sha256.Sum256([]byte("Hello"))  // Returns [32]byte
fmt.Printf("%T\n", hash)  // Output: [32]uint8
```

```go
type RGB [3]uint8
red := RGB{255, 0, 0}
```

```go
daysInMonth := [13]int{0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}
fmt.Println(daysInMonth[2])  // Output: 28 (February)
```

### Complete Example: Production Use Cases

```go
package main

import (
    "crypto/sha256"
    "fmt"
)

func main() {
    var ipv4 [4]byte = [4]byte{192, 168, 1, 1}
    hash := sha256.Sum256([]byte("Hello"))
    daysInMonth := [13]int{0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}

    fmt.Printf("IPv4: %d.%d.%d.%d\n", ipv4[0], ipv4[1], ipv4[2], ipv4[3])
    fmt.Printf("Hash type: %T\n", hash)
    fmt.Printf("Days in February: %d\n", daysInMonth[2])
}
```

**Output:**
```
IPv4: 192.168.1.1
Hash type: [32]uint8
Days in February: 28
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `int[] arr = new int[5];` | `var arr [5]int` |
| Size not in type | Size IS the type |
| `a == b` → false (reference) | `a == b` → true (value) |
| `void process(int[] arr)` works with any size | `func process(arr [3]int)` only `[3]int` |
| `arr.length` | `len(arr)` |
| `int[] b = a` → same reference | `b := a` → full copy |
| `b[0] = 99` → `a[0]` also 99 | `b[0] = 99` → `a` unchanged |

**Key difference**: Java arrays are reference types; Go arrays are value types (copied on assignment).

---

## 🎯 Key Takeaways

1. **Fixed size** - cannot grow or shrink after creation
2. **Size is part of type** - `[3]int` ≠ `[4]int`
3. **Value type** - assignment/passing creates a copy
4. **Zero-indexed** - first element is `arr[0]`
5. **Comparable** - can use `==` to compare
6. **Contiguous memory** - fast, cache-friendly
7. **Use slices instead** for most cases (~95%)
8. **Arrays for**: fixed-size data, crypto keys, buffers

---

## ➡️ Next Steps

Arrays are the foundation. Now let's learn about slices - the much more common and flexible data structure.

**Next Topic:** [21 - Slices](./21-slices.md)
