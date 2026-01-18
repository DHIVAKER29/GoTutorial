# 16 - Multiple Return Values

> Deep dive into Go's multiple return values and the error pattern.

---

## 📌 What You'll Learn

- Why multiple return values exist
- The (result, error) pattern
- Named return values in depth
- Best practices

---

## 🔄 The Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  THE GO ERROR PATTERN                                           │
│                                                                 │
│  result, err := doSomething()                                   │
│  if err != nil {                                                │
│      return err  // or handle it                                │
│  }                                                              │
│  // use result                                                  │
│                                                                 │
│  This pattern appears EVERYWHERE in Go!                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Sample Program

```go
// multiple_returns.go
package main

import (
    "errors"
    "fmt"
    "strconv"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           MULTIPLE RETURN VALUES                          ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Standard error pattern
    fmt.Println("\n📊 Standard Error Pattern:")
    if result, err := divide(10, 2); err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   10 / 2 = %.2f\n", result)
    }
    
    if result, err := divide(10, 0); err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   Result: %.2f\n", result)
    }
    
    // Multiple values
    fmt.Println("\n📊 Multiple Related Values:")
    min, max, avg := stats([]int{5, 2, 8, 1, 9, 3})
    fmt.Printf("   Stats: min=%d, max=%d, avg=%.2f\n", min, max, avg)
    
    // Value and exists pattern
    fmt.Println("\n📊 Value, Exists Pattern:")
    users := map[string]int{"alice": 25, "bob": 30}
    
    if age, exists := users["alice"]; exists {
        fmt.Printf("   Alice's age: %d\n", age)
    }
    
    if age, exists := users["charlie"]; exists {
        fmt.Printf("   Charlie's age: %d\n", age)
    } else {
        fmt.Println("   Charlie not found")
    }
    
    // Type assertion pattern
    fmt.Println("\n📊 Type Assertion Pattern:")
    var val interface{} = "hello"
    if str, ok := val.(string); ok {
        fmt.Printf("   It's a string: %q\n", str)
    }
    
    // Ignoring values
    fmt.Println("\n📊 Ignoring Values with _:")
    quotient, _ := divideInt(17, 5)  // Ignore remainder
    fmt.Printf("   17 / 5 = %d (remainder ignored)\n", quotient)
}

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

func stats(nums []int) (min, max int, avg float64) {
    if len(nums) == 0 {
        return 0, 0, 0
    }
    
    min, max = nums[0], nums[0]
    sum := 0
    
    for _, n := range nums {
        if n < min {
            min = n
        }
        if n > max {
            max = n
        }
        sum += n
    }
    
    avg = float64(sum) / float64(len(nums))
    return  // Named return
}

func divideInt(a, b int) (int, int) {
    return a / b, a % b
}
```

---

## 🎯 Key Patterns

| Pattern | Example | Use Case |
|---------|---------|----------|
| (result, error) | `file, err := os.Open()` | Most common |
| (value, bool) | `val, ok := map[key]` | Map lookup |
| (value, bool) | `val, ok := x.(Type)` | Type assertion |
| Multiple values | `min, max := minMax()` | Related values |

---

## ➡️ Next Steps

**Next Topic:** [17 - Variadic Functions](./17-variadic-functions.md)

