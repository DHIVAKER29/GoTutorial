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

In Go, functions are first-class values. You can:

- Assign them to variables
- Pass them as arguments
- Return them from functions
- Store them in data structures

```go
var fn func(int) int   // fn is a function variable
fn = func(x int) int { return x * 2 }
result := fn(5)        // Call it: result = 10
```

---

## 📝 Function as Variable

```go
double := func(x int) int {
    return x * 2
}
fmt.Println(double(5))  // 10
```

---

## 📝 Immediately Invoked Function

```go
result := func(a, b int) int {
    return a + b
}(3, 4)  // Called immediately!
fmt.Println(result)  // 7
```

---

## 📝 Closures (Captured Variables)

```go
func makeCounter() func() int {
    count := 0  // Captured by closure
    return func() int {
        count++
        return count
    }
}

counter := makeCounter()
fmt.Println(counter(), counter(), counter())  // 1 2 3
counter2 := makeCounter()
fmt.Println(counter2())  // 1 (separate state)
```

---

## 📝 Function as Argument (Higher-Order)

```go
func mapInts(nums []int, fn func(int) int) []int {
    result := make([]int, len(nums))
    for i, n := range nums {
        result[i] = fn(n)
    }
    return result
}

nums := []int{1, 2, 3, 4, 5}
doubled := mapInts(nums, func(x int) int { return x * 2 })
// doubled = [2 4 6 8 10]
```

---

## 📝 Deferred Anonymous Function

```go
func deferDemo() {
    message := "initial"
    defer func() {
        fmt.Printf("Deferred sees: %s\n", message)
    }()
    message = "modified"
    fmt.Printf("Current: %s\n", message)
}
// Output: Current: modified
// Output: Deferred sees: modified
```

---

## 📝 Complete Example

```go
package main

import (
    "fmt"
    "time"
)

func makeCounter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

func main() {
    double := func(x int) int { return x * 2 }
    fmt.Println(double(5))

    counter := makeCounter()
    fmt.Println(counter(), counter(), counter())

    time.AfterFunc(100*time.Millisecond, func() {
        fmt.Println("Timer fired!")
    })
    time.Sleep(200 * time.Millisecond)
}
```

**Output:**
```
10
1 2 3
Timer fired!
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

