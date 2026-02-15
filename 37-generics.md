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

**The problem:** Repeating code for different types:
```go
func SumInts(nums []int) int { ... }
func SumFloat64s(nums []float64) float64 { ... }
// Same logic, different types — code duplication!
```

**The solution:** Generics
```go
func Sum[T int | float64](nums []T) T {
    var total T
    for _, n := range nums { total += n }
    return total
}
Sum([]int{1, 2, 3})        // Works!
Sum([]float64{1.1, 2.2})   // Works!
```

---

## 📝 Generic Functions

**Basic generic with `any`:**
```go
func Print[T any](value T) {
    fmt.Printf("Value: %v (type: %T)\n", value, value)
}
Print(42)           // T = int
Print("hello")      // T = string
// Output: Value: 42 (type: int)
// Output: Value: hello (type: string)
```

**Constrained type parameter:**
```go
func Min[T int | float64](a, b T) T {
    if a < b { return a }
    return b
}
fmt.Println(Min(3, 5))         // 3
fmt.Println(Min(3.14, 2.71))   // 2.71
```

**Custom constraint:**
```go
type Number interface {
    int | int8 | int16 | int32 | int64 | float32 | float64
}
func Sum[T Number](nums []T) T {
    var total T
    for _, n := range nums { total += n }
    return total
}
fmt.Println(Sum([]int{1, 2, 3, 4, 5}))  // 15
```

### Complete Example: Generic Functions

```go
package main

import "fmt"

func Print[T any](value T) {
    fmt.Printf("Value: %v (type: %T)\n", value, value)
}

func Min[T int | float64](a, b T) T {
    if a < b { return a }
    return b
}

type Number interface {
    int | float64
}

func Sum[T Number](nums []T) T {
    var total T
    for _, n := range nums { total += n }
    return total
}

func main() {
    Print(42)
    Print("hello")
    fmt.Printf("Min(3, 5) = %d\n", Min(3, 5))
    fmt.Printf("Sum([]int{1,2,3,4,5}) = %d\n", Sum([]int{1, 2, 3, 4, 5}))
}
```

**Output:**
```
Value: 42 (type: int)
Value: hello (type: string)
Min(3, 5) = 3
Sum([]int{1,2,3,4,5}) = 15
```

---

## 📦 Generic Types

**Generic Stack:**
```go
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
```

**Usage:**
```go
intStack := &Stack[int]{}
intStack.Push(1)
intStack.Push(2)
val, ok := intStack.Pop()  // val=2, ok=true
```

### Complete Example: Generic Stack

```go
package main

import "fmt"

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

func main() {
    s := &Stack[int]{}
    s.Push(1)
    s.Push(2)
    s.Push(3)
    for val, ok := s.Pop(); ok; val, ok = s.Pop() {
        fmt.Printf("Popped: %d\n", val)
    }
}
```

**Output:**
```
Popped: 3
Popped: 2
Popped: 1
```

---

## 🔒 Constraints

| Constraint | Description |
|------------|-------------|
| `any` | Any type (same as `interface{}`) |
| `comparable` | Types that support `==` and `!=` (map keys) |
| `~T` | Includes types with underlying type T |

**comparable:**
```go
func Contains[T comparable](slice []T, target T) bool {
    for _, item := range slice {
        if item == target { return true }
    }
    return false
}
Contains([]int{1, 2, 3}, 2)  // true
```

**Underlying type (~):**
```go
type Numeric interface {
    ~int | ~float64
}
type Meters float64
var d Meters = 100.5
Max(d, 50.0)  // Works — Meters has underlying type float64
```

---

## 🏭 Practical Generic Patterns

**Map (transform):**
```go
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice { result[i] = fn(v) }
    return result
}
doubled := Map([]int{1, 2, 3}, func(n int) int { return n * 2 })
// [2 4 6]
```

**Filter:**
```go
func Filter[T any](slice []T, fn func(T) bool) []T {
    result := make([]T, 0)
    for _, v := range slice {
        if fn(v) { result = append(result, v) }
    }
    return result
}
evens := Filter([]int{1, 2, 3, 4}, func(n int) bool { return n%2 == 0 })
// [2 4]
```

**Keys/Values:**
```go
func Keys[K comparable, V any](m map[K]V) []K {
    keys := make([]K, 0, len(m))
    for k := range m { keys = append(keys, k) }
    return keys
}
Keys(map[string]int{"Alice": 30, "Bob": 25})  // [Alice Bob]
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `public <T> void print(T val)` | `func Print[T any](val T)` |
| `class Stack<T> { List<T> items; }` | `type Stack[T any] struct { items []T }` |
| `<T extends Number>` | `[T Number]` (custom constraint) |
| `List<? extends Animal>` | No wildcards (simpler model) |
| Type erasure at runtime | Monomorphization (specialized code) |

---

## ⚠️ When NOT to Use Generics

- **When types don't matter** — use `interface{}`/`any`
- **When behavior differs by type** — use interfaces
- **Simple cases** — sometimes copy-paste is clearer
- **When it hurts readability** — simplify

**Rule:** Use generics when you'd otherwise write the same code for multiple types.

---

## 🎯 Key Takeaways

1. **Type parameters** in square brackets: `func Name[T any]()`
2. **any** = accepts any type
3. **comparable** = types that support `==` and `!=`
4. **~T** = includes types with underlying type T
5. **Type inference** often makes explicit types unnecessary
6. **Don't overuse** — generics aren't always the answer
7. **Standard library** has generic slices/maps packages

---

## ➡️ Next Steps

**Next Topic:** [38 - File I/O](./38-file-io.md)
