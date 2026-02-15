# 17 - Variadic Functions

> Functions that accept a variable number of arguments.

---

## 📌 What You'll Learn

- What variadic functions are
- The `...` syntax
- How to pass slices to variadic functions
- Common examples (fmt.Println, append)

---

## 🔢 Variadic Basics

**VARIADIC** = "Variable number of arguments"

- The `...T` syntax makes the last parameter accept zero or more values of type `T`
- Inside the function, it becomes a slice `[]T`
- Examples: `sum(1, 2, 3)`, `sum(1, 2, 3, 4, 5)`, and `sum()` (empty slice) all work

```go
func sum(nums ...int) int {
    // nums is []int inside the function
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}
// sum(1, 2, 3) → 6
// sum() → 0
```

---

## 📝 Basic Variadic Examples

```go
fmt.Printf("sum() = %d\n", sum())           // 0
fmt.Printf("sum(1) = %d\n", sum(1))          // 1
fmt.Printf("sum(1,2,3) = %d\n", sum(1, 2, 3))  // 6
// Output: sum() = 0
// Output: sum(1) = 1
// Output: sum(1,2,3) = 6
```

---

## 📝 Passing Slice with ...

```go
numbers := []int{10, 20, 30, 40}
fmt.Printf("sum(numbers...) = %d\n", sum(numbers...))
// Output: sum(numbers...) = 100
```

---

## 📝 Mixed Parameters

```go
func greetAll(greeting string, names ...string) {
    for _, name := range names {
        fmt.Printf("%s, %s!\n", greeting, name)
    }
}
greetAll("Hello", "Alice", "Bob", "Charlie")
// Output: Hello, Alice!
// Output: Hello, Bob!
// Output: Hello, Charlie!
```

---

## 📝 fmt.Println and append are Variadic

```go
fmt.Println("Multiple", "values", "separated", "by", "spaces")
// Output: Multiple values separated by spaces

slice := []int{1, 2}
slice = append(slice, 3, 4, 5)
more := []int{6, 7, 8}
slice = append(slice, more...)  // Spread with ...
// slice is now [1 2 3 4 5 6 7 8]
```

---

## 📝 Complete Example

```go
package main

import "fmt"

func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

func main() {
    fmt.Println(sum(), sum(1), sum(1, 2, 3))
    numbers := []int{10, 20, 30}
    fmt.Println(sum(numbers...))
}
```

**Output:**
```
0 1 6
60
```

---

## 🎯 Key Points

1. **`...T`** after the last parameter makes it variadic
2. Inside function, it's a **slice** `[]T`
3. **`slice...`** spreads a slice into arguments
4. **Must be last** parameter in function signature
5. Common in: `fmt.Println`, `append`, `errors.Join`

---

## ➡️ Next Steps

**Next Topic:** [18 - Anonymous Functions & Closures](./18-anonymous-closures.md)

