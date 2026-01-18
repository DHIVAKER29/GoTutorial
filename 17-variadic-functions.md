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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  VARIADIC = "Variable number of arguments"                      │
│                                                                 │
│  func sum(nums ...int) int {                                    │
│           ↑                                                     │
│           nums becomes []int inside function                    │
│  }                                                              │
│                                                                 │
│  sum(1, 2, 3)        // Works!                                  │
│  sum(1, 2, 3, 4, 5)  // Works!                                  │
│  sum()               // Works! (empty slice)                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Sample Program

```go
// variadic.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           VARIADIC FUNCTIONS                              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic variadic
    fmt.Println("\n📊 Basic Variadic:")
    fmt.Printf("   sum()        = %d\n", sum())
    fmt.Printf("   sum(1)       = %d\n", sum(1))
    fmt.Printf("   sum(1,2,3)   = %d\n", sum(1, 2, 3))
    fmt.Printf("   sum(1,2,3,4,5) = %d\n", sum(1, 2, 3, 4, 5))
    
    // Passing slice with ...
    fmt.Println("\n📊 Passing Slice with ...:")
    numbers := []int{10, 20, 30, 40}
    fmt.Printf("   sum(numbers...) = %d\n", sum(numbers...))
    
    // Mixed parameters
    fmt.Println("\n📊 Mixed Parameters:")
    greetAll("Hello", "Alice", "Bob", "Charlie")
    
    // fmt.Println is variadic!
    fmt.Println("\n📊 fmt.Println is Variadic:")
    fmt.Println("Multiple", "values", "separated", "by", "spaces")
    
    // append is variadic
    fmt.Println("\n📊 append() is Variadic:")
    slice := []int{1, 2}
    slice = append(slice, 3, 4, 5)
    fmt.Printf("   append(slice, 3, 4, 5) = %v\n", slice)
    
    // Spread another slice
    more := []int{6, 7, 8}
    slice = append(slice, more...)  // Spread!
    fmt.Printf("   append(slice, more...) = %v\n", slice)
}

func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

func greetAll(greeting string, names ...string) {
    for _, name := range names {
        fmt.Printf("   %s, %s!\n", greeting, name)
    }
}
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

