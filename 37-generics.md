# 37 - Generics (Go 1.18+)

> Type parameters for writing flexible, reusable code.

---

## 📌 What You'll Learn

- What generics are and why Go added them
- Type parameters and constraints
- Generic functions and types
- The `any` and `comparable` constraints
- When to use generics (and when not to)

---

## 🤔 What Are Generics?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  THE PROBLEM: Repeating code for different types                │
│                                                                 │
│  func SumInts(nums []int) int {                                 │
│      total := 0                                                 │
│      for _, n := range nums { total += n }                      │
│      return total                                               │
│  }                                                              │
│                                                                 │
│  func SumFloat64s(nums []float64) float64 {                     │
│      total := 0.0                                               │
│      for _, n := range nums { total += n }                      │
│      return total                                               │
│  }                                                              │
│                                                                 │
│  // Same logic, different types! Code duplication!              │
│                                                                 │
│  THE SOLUTION: Generics                                         │
│                                                                 │
│  func Sum[T int | float64](nums []T) T {                        │
│      var total T                                                │
│      for _, n := range nums { total += n }                      │
│      return total                                               │
│  }                                                              │
│                                                                 │
│  Sum([]int{1, 2, 3})        // Works!                           │
│  Sum([]float64{1.1, 2.2})   // Works!                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Generic Functions

```go
// generic_functions.go
package main

import "fmt"

// Generic function with type parameter T
func Print[T any](value T) {
    fmt.Printf("Value: %v (type: %T)\n", value, value)
}

// Multiple type parameters
func Pair[K, V any](key K, value V) map[K]V {
    // Note: K must be comparable to use as map key
    // This won't compile - just for illustration
    return nil
}

// Constrained type parameter
func Min[T int | float64](a, b T) T {
    if a < b {
        return a
    }
    return b
}

// Using constraints package
type Number interface {
    int | int8 | int16 | int32 | int64 |
    float32 | float64
}

func Sum[T Number](nums []T) T {
    var total T
    for _, n := range nums {
        total += n
    }
    return total
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           GENERIC FUNCTIONS                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Type inference (compiler figures out T)
    fmt.Println("\n📊 Type Inference:")
    Print(42)           // T = int
    Print("hello")      // T = string
    Print([]int{1, 2})  // T = []int
    
    // Explicit type argument
    fmt.Println("\n📊 Explicit Type:")
    Print[float64](3.14)
    
    // Constrained generics
    fmt.Println("\n📊 Constrained Generic:")
    fmt.Printf("Min(3, 5) = %d\n", Min(3, 5))
    fmt.Printf("Min(3.14, 2.71) = %f\n", Min(3.14, 2.71))
    
    // Sum with different types
    fmt.Println("\n📊 Sum Generic:")
    fmt.Printf("Sum([]int) = %d\n", Sum([]int{1, 2, 3, 4, 5}))
    fmt.Printf("Sum([]float64) = %f\n", Sum([]float64{1.1, 2.2, 3.3}))
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           GENERIC FUNCTIONS                               ║
╚══════════════════════════════════════════════════════════╝

📊 Type Inference:
Value: 42 (type: int)
Value: hello (type: string)
Value: [1 2] (type: []int)

📊 Explicit Type:
Value: 3.14 (type: float64)

📊 Constrained Generic:
Min(3, 5) = 3
Min(3.14, 2.71) = 2.710000

📊 Sum Generic:
Sum([]int) = 15
Sum([]float64) = 6.600000
```

---

## 📦 Generic Types

```go
// generic_types.go
package main

import "fmt"

// Generic Stack
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}

func (s *Stack[T]) Peek() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    return s.items[len(s.items)-1], true
}

func (s *Stack[T]) Len() int {
    return len(s.items)
}

// Generic Result type (like Rust's Result)
type Result[T any] struct {
    value T
    err   error
}

func Ok[T any](value T) Result[T] {
    return Result[T]{value: value}
}

func Err[T any](err error) Result[T] {
    return Result[T]{err: err}
}

func (r Result[T]) Unwrap() (T, error) {
    return r.value, r.err
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           GENERIC TYPES                                   ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Integer stack
    fmt.Println("\n📊 Integer Stack:")
    intStack := &Stack[int]{}
    intStack.Push(1)
    intStack.Push(2)
    intStack.Push(3)
    
    for intStack.Len() > 0 {
        val, _ := intStack.Pop()
        fmt.Printf("   Popped: %d\n", val)
    }
    
    // String stack
    fmt.Println("\n📊 String Stack:")
    strStack := &Stack[string]{}
    strStack.Push("hello")
    strStack.Push("world")
    
    if val, ok := strStack.Peek(); ok {
        fmt.Printf("   Peek: %s\n", val)
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           GENERIC TYPES                                   ║
╚══════════════════════════════════════════════════════════╝

📊 Integer Stack:
   Popped: 3
   Popped: 2
   Popped: 1

📊 String Stack:
   Peek: world
```

---

## 🔒 Constraints

```go
// constraints.go
package main

import (
    "fmt"
    "golang.org/x/exp/constraints"
)

// Built-in constraints
// any       = interface{} (any type)
// comparable = types that support == and != (can be map keys)

// Custom constraint
type Numeric interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64
}

// ~ means "underlying type" (includes type aliases)
type MyInt int  // MyInt has underlying type int

// Using comparable
func Contains[T comparable](slice []T, target T) bool {
    for _, item := range slice {
        if item == target {
            return true
        }
    }
    return false
}

// Using custom constraint
func Max[T Numeric](a, b T) T {
    if a > b {
        return a
    }
    return b
}

// Using constraints package (golang.org/x/exp/constraints)
func Abs[T constraints.Signed](n T) T {
    if n < 0 {
        return -n
    }
    return n
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CONSTRAINTS                                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // comparable
    fmt.Println("\n📊 comparable Constraint:")
    fmt.Printf("   Contains([1,2,3], 2) = %t\n", Contains([]int{1, 2, 3}, 2))
    fmt.Printf("   Contains([\"a\",\"b\"], \"c\") = %t\n", Contains([]string{"a", "b"}, "c"))
    
    // Custom numeric constraint
    fmt.Println("\n📊 Numeric Constraint:")
    fmt.Printf("   Max(10, 20) = %d\n", Max(10, 20))
    fmt.Printf("   Max(3.14, 2.71) = %f\n", Max(3.14, 2.71))
    
    // Underlying type (~)
    fmt.Println("\n📊 Underlying Type (~):")
    type Meters float64
    var distance Meters = 100.5
    fmt.Printf("   Max(Meters) = %v\n", Max(distance, 50.0))
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           CONSTRAINTS                                     ║
╚══════════════════════════════════════════════════════════╝

📊 comparable Constraint:
   Contains([1,2,3], 2) = true
   Contains(["a","b"], "c") = false

📊 Numeric Constraint:
   Max(10, 20) = 20
   Max(3.14, 2.71) = 3.140000

📊 Underlying Type (~):
   Max(Meters) = 100.5
```

---

## 🏭 Practical Generic Patterns

```go
// generic_patterns.go
package main

import "fmt"

// Map function (like JavaScript map)
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Filter function
func Filter[T any](slice []T, fn func(T) bool) []T {
    result := make([]T, 0)
    for _, v := range slice {
        if fn(v) {
            result = append(result, v)
        }
    }
    return result
}

// Reduce function
func Reduce[T, U any](slice []T, initial U, fn func(U, T) U) U {
    result := initial
    for _, v := range slice {
        result = fn(result, v)
    }
    return result
}

// Keys extracts map keys
func Keys[K comparable, V any](m map[K]V) []K {
    keys := make([]K, 0, len(m))
    for k := range m {
        keys = append(keys, k)
    }
    return keys
}

// Values extracts map values
func Values[K comparable, V any](m map[K]V) []V {
    vals := make([]V, 0, len(m))
    for _, v := range m {
        vals = append(vals, v)
    }
    return vals
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRACTICAL GENERIC PATTERNS                      ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    numbers := []int{1, 2, 3, 4, 5}
    
    // Map
    fmt.Println("\n📊 Map (transform):")
    doubled := Map(numbers, func(n int) int { return n * 2 })
    fmt.Printf("   Doubled: %v\n", doubled)
    
    strings := Map(numbers, func(n int) string {
        return fmt.Sprintf("num-%d", n)
    })
    fmt.Printf("   To strings: %v\n", strings)
    
    // Filter
    fmt.Println("\n📊 Filter:")
    evens := Filter(numbers, func(n int) bool { return n%2 == 0 })
    fmt.Printf("   Evens: %v\n", evens)
    
    // Reduce
    fmt.Println("\n📊 Reduce:")
    sum := Reduce(numbers, 0, func(acc, n int) int { return acc + n })
    fmt.Printf("   Sum: %d\n", sum)
    
    // Keys/Values
    fmt.Println("\n📊 Map Utilities:")
    ages := map[string]int{"Alice": 30, "Bob": 25}
    fmt.Printf("   Keys: %v\n", Keys(ages))
    fmt.Printf("   Values: %v\n", Values(ages))
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           PRACTICAL GENERIC PATTERNS                      ║
╚══════════════════════════════════════════════════════════╝

📊 Map (transform):
   Doubled: [2 4 6 8 10]
   To strings: [num-1 num-2 num-3 num-4 num-5]

📊 Filter:
   Evens: [2 4]

📊 Reduce:
   Sum: 15

📊 Map Utilities:
   Keys: [Alice Bob]
   Values: [30 25]
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│                                                                 │
│  public <T> void print(T val)      func Print[T any](val T)     │
│                                                                 │
│  class Stack<T> {                  type Stack[T any] struct {   │
│      private List<T> items;            items []T                │
│  }                                 }                            │
│                                                                 │
│  <T extends Number>                [T Numeric] (custom)         │
│                                    [T constraints.Ordered]      │
│                                                                 │
│  List<? extends Animal>            No wildcards in Go           │
│                                    (simpler model)              │
│                                                                 │
│  // Type erasure at runtime        // Monomorphization          │
│  // (info lost)                    // (specialized code)        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚠️ When NOT to Use Generics

```go
// When to avoid generics:

// 1. When types don't matter (use interface{}/any)
func PrintAnything(v any) { fmt.Println(v) }

// 2. When behavior differs by type (use interfaces)
type Stringer interface {
    String() string
}

// 3. Simple cases where duplication is minimal
// Sometimes copy-paste is clearer than generics!

// 4. When it hurts readability
// If the generic signature is confusing, simplify

// RULE: Use generics when you'd otherwise write
// the same code for multiple types
```

---

## 🎯 Key Takeaways

1. **Type parameters** in square brackets: `func Name[T any]()`
2. **any** = accepts any type
3. **comparable** = types that support `==` and `!=`
4. **~T** = includes types with underlying type T
5. **Type inference** often makes explicit types unnecessary
6. **Don't overuse** - generics aren't always the answer
7. **Standard library** has generic slices/maps packages

---

## ➡️ Next Steps

**Next Topic:** [38 - File I/O](./38-file-io.md)

