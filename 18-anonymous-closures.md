# 18 - Anonymous Functions & Closures

> Functions as values, anonymous functions, and closures in Go.

---

## 📌 What You'll Learn

- Functions as first-class values
- Anonymous (lambda) functions
- Closures and captured variables
- Common patterns

---

## 📦 Functions as Values

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  IN GO, FUNCTIONS ARE VALUES                                    │
│                                                                 │
│  • Assign to variables                                          │
│  • Pass as arguments                                            │
│  • Return from functions                                        │
│  • Store in data structures                                     │
│                                                                 │
│  var fn func(int) int   // fn is a function variable           │
│  fn = double            // Assign a function                    │
│  result := fn(5)        // Call it: result = 10                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Sample Program

```go
// closures.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║        ANONYMOUS FUNCTIONS & CLOSURES                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Function as variable
    fmt.Println("\n📊 Function as Variable:")
    double := func(x int) int {
        return x * 2
    }
    fmt.Printf("   double(5) = %d\n", double(5))
    
    // Anonymous function (immediately invoked)
    fmt.Println("\n📊 Immediately Invoked:")
    result := func(a, b int) int {
        return a + b
    }(3, 4)  // Called immediately!
    fmt.Printf("   Result: %d\n", result)
    
    // Closure - captures outer variable
    fmt.Println("\n📊 Closure (captures counter):")
    counter := makeCounter()
    fmt.Printf("   counter() = %d\n", counter())
    fmt.Printf("   counter() = %d\n", counter())
    fmt.Printf("   counter() = %d\n", counter())
    
    // Each closure has its own state
    counter2 := makeCounter()
    fmt.Printf("   counter2() = %d (separate state)\n", counter2())
    
    // Passing function as argument
    fmt.Println("\n📊 Function as Argument:")
    nums := []int{1, 2, 3, 4, 5}
    doubled := mapInts(nums, func(x int) int {
        return x * 2
    })
    fmt.Printf("   Doubled: %v\n", doubled)
    
    // Deferred anonymous function
    fmt.Println("\n📊 Deferred Anonymous Function:")
    deferDemo()
    
    // Practical: Timer callback
    fmt.Println("\n📊 Timer Callback:")
    time.AfterFunc(100*time.Millisecond, func() {
        fmt.Println("   Timer fired!")
    })
    time.Sleep(200 * time.Millisecond)
}

// Returns a function (closure)
func makeCounter() func() int {
    count := 0  // This is captured!
    return func() int {
        count++
        return count
    }
}

// Higher-order function
func mapInts(nums []int, fn func(int) int) []int {
    result := make([]int, len(nums))
    for i, n := range nums {
        result[i] = fn(n)
    }
    return result
}

func deferDemo() {
    message := "initial"
    
    defer func() {
        fmt.Printf("   Deferred sees: %s\n", message)
    }()
    
    message = "modified"
    fmt.Printf("   Current: %s\n", message)
}
```

---

## 🔒 Closure Gotcha

```go
// ⚠️ COMMON BUG: Loop variable capture
for i := 0; i < 3; i++ {
    go func() {
        fmt.Println(i)  // All print 3! (i is shared)
    }()
}

// ✅ FIX: Pass as parameter
for i := 0; i < 3; i++ {
    go func(n int) {
        fmt.Println(n)  // Prints 0, 1, 2
    }(i)
}
```

---

## 🎯 Key Points

1. **Functions are values** - assign, pass, return them
2. **Anonymous functions** - functions without names
3. **Closures** capture variables from outer scope
4. **Each closure** has its own copy of captured state
5. **Watch loop variables** - pass to goroutines by value

---

## ➡️ Next Steps

**Next Topic:** [19 - Defer, Panic & Recover](./19-defer-panic-recover.md)

