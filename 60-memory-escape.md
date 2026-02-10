# 60 - Go Memory & Escape Analysis

> Understanding how Go manages memory and where variables are allocated.

---

## 📌 What You'll Learn

- Stack vs Heap allocation
- Escape analysis
- How to check if variables escape
- Performance implications
- Memory optimization tips

---

## 🧠 Stack vs Heap

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MEMORY LAYOUT                                                  │
│                                                                 │
│  ┌─────────────────────────────────────────────┐               │
│  │                 STACK                        │               │
│  │  • Fast allocation (just move pointer)       │               │
│  │  • Automatic cleanup (function returns)      │               │
│  │  • Fixed size per goroutine (~2KB initial)   │               │
│  │  • Local variables, function params          │               │
│  │  • No GC overhead                            │               │
│  └─────────────────────────────────────────────┘               │
│                                                                 │
│  ┌─────────────────────────────────────────────┐               │
│  │                 HEAP                         │               │
│  │  • Slower allocation                         │               │
│  │  • Garbage Collected (GC overhead)           │               │
│  │  • Dynamic size                              │               │
│  │  • Shared between goroutines                 │               │
│  │  • Variables that "escape"                   │               │
│  └─────────────────────────────────────────────┘               │
│                                                                 │
│  GOAL: Keep as much on STACK as possible!                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔍 What is Escape Analysis?

```go
// escape_analysis.go
package main

import "fmt"

// Escape analysis determines WHERE variables are allocated
// The compiler analyzes if a variable "escapes" the function

// Example 1: Does NOT escape (stays on stack)
func createLocal() int {
    x := 42  // Allocated on stack
    return x // Value is copied, x can be cleaned up
}

// Example 2: ESCAPES to heap
func createPointer() *int {
    x := 42   // Must be on heap!
    return &x // Pointer returned - x outlives function
}

// Example 3: Might escape (depends on usage)
func process(data []byte) {
    // data might be on stack or heap
    // depends on where it came from
    _ = data
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Escape Analysis                                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Stack allocation
    val := createLocal()
    fmt.Printf("   Value (stack): %d\n", val)
    
    // Heap allocation
    ptr := createPointer()
    fmt.Printf("   Pointer (heap): %d\n", *ptr)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Escape Analysis                                 ║
╚══════════════════════════════════════════════════════════╝
   Value (stack): 42
   Pointer (heap): 42
```

---

## 🛠️ Checking Escape Analysis

```bash
# See escape analysis decisions
go build -gcflags="-m" main.go

# More verbose
go build -gcflags="-m -m" main.go

# Example output:
# ./main.go:10:2: x escapes to heap
# ./main.go:15:2: y does not escape
```

```go
// escape_check.go
package main

type User struct {
    Name string
    Age  int
}

// Does NOT escape - returned by value
func NewUserValue() User {
    return User{Name: "Alice", Age: 30}
}

// ESCAPES - pointer returned
func NewUserPointer() *User {
    return &User{Name: "Bob", Age: 25}  // &User escapes to heap
}

// ESCAPES - stored in interface
func AsInterface() interface{} {
    x := 42
    return x  // x escapes (interface boxing)
}

// ESCAPES - closure captures variable
func Closure() func() int {
    x := 42
    return func() int {
        return x  // x escapes (captured by closure)
    }
}

// ESCAPES - slice too large for stack
func LargeSlice() []byte {
    return make([]byte, 1024*1024)  // 1MB - too large for stack
}

// Does NOT escape - small slice
func SmallSlice() [64]byte {
    var arr [64]byte  // Array on stack (fixed size, small)
    return arr
}

func main() {
    _ = NewUserValue()
    _ = NewUserPointer()
    _ = AsInterface()
    _ = Closure()
    _ = LargeSlice()
    _ = SmallSlice()
}

/*
Run: go build -gcflags="-m" escape_check.go

Output:
./escape_check.go:14:9: &User{...} escapes to heap
./escape_check.go:19:2: x escapes to heap
./escape_check.go:24:2: x escapes to heap
./escape_check.go:25:9: func literal escapes to heap
./escape_check.go:31:13: make([]byte, 1048576) escapes to heap
*/
```

---

## 📋 Escape Analysis Rules

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ESCAPES TO HEAP WHEN:                                          │
│                                                                 │
│  1. Pointer to local variable is returned                       │
│     func f() *int { x := 1; return &x }                         │
│                                                                 │
│  2. Variable stored in interface                                │
│     var i interface{} = x                                       │
│                                                                 │
│  3. Variable captured by escaping closure                       │
│     return func() { use(x) }                                    │
│                                                                 │
│  4. Slice/map too large for stack                               │
│     make([]byte, 10_000_000)                                    │
│                                                                 │
│  5. Variable assigned to heap-allocated struct field            │
│     heapStruct.field = &x                                       │
│                                                                 │
│  6. Variable sent to channel                                    │
│     ch <- x                                                     │
│                                                                 │
│  STAYS ON STACK WHEN:                                           │
│                                                                 │
│  1. Only used within function                                   │
│  2. Small, fixed-size types                                     │
│  3. Returned by value (not pointer)                             │
│  4. Compiler can prove lifetime                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚡ Performance Implications

```go
// performance.go
package main

import (
    "testing"
)

type Data struct {
    Value [100]int
}

// Benchmark: Return by value (stack)
func ByValue() Data {
    var d Data
    d.Value[0] = 42
    return d
}

// Benchmark: Return by pointer (heap)
func ByPointer() *Data {
    d := &Data{}
    d.Value[0] = 42
    return d
}

func BenchmarkByValue(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = ByValue()
    }
}

func BenchmarkByPointer(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = ByPointer()
    }
}

/*
Results (approximate):
BenchmarkByValue-8     50000000    25 ns/op    0 B/op   0 allocs/op
BenchmarkByPointer-8   20000000    80 ns/op  800 B/op   1 allocs/op

Value return: NO allocations (stack)
Pointer return: 1 allocation (heap + GC pressure)
*/
```

---

## 💡 Optimization Tips

```go
// optimization_tips.go
package main

// TIP 1: Prefer value receivers for small structs
type Point struct {
    X, Y float64
}

func (p Point) Distance() float64 {  // Value receiver - no escape
    return p.X*p.X + p.Y*p.Y
}

// TIP 2: Pre-allocate slices with known size
func GoodSlice(n int) []int {
    result := make([]int, 0, n)  // Pre-allocate capacity
    for i := 0; i < n; i++ {
        result = append(result, i)
    }
    return result
}

// TIP 3: Use arrays instead of slices for fixed small sizes
func UseArray() [10]int {
    var arr [10]int  // Stack allocated
    return arr
}

// TIP 4: Avoid interface{} when possible
func ProcessInt(x int) int {     // Good - concrete type
    return x * 2
}

func ProcessAny(x interface{}) interface{} {  // Bad - forces heap
    return x
}

// TIP 5: Reuse buffers with sync.Pool
// (Covered in chapter 32)

// TIP 6: Pass large structs by pointer to AVOID copying
// (But they'll be on heap anyway if passed around)
type LargeStruct struct {
    Data [10000]byte
}

func ProcessLarge(s *LargeStruct) {  // Pointer avoids copy
    _ = s.Data[0]
}

func main() {}
```

---

## 🎯 Key Takeaways

1. **Stack = fast, Heap = slower (GC)**
2. **Escape analysis** decides allocation location
3. **Returning pointers** causes heap allocation
4. **Interface boxing** causes heap allocation
5. **Use `-gcflags="-m"`** to see escape decisions
6. **Small values by value** = stack = fast
7. **Don't over-optimize** - profile first!

---

## ➡️ Next Steps

**Next Topic:** [61 - Garbage Collection](./61-garbage-collection.md)

