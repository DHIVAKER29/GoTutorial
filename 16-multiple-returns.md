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

The standard Go error pattern:

```go
result, err := doSomething()
if err != nil {
    return err  // or handle it
}
// use result
```

This pattern appears **everywhere** in Go!

---

## 📝 Snippets by Concept

### Standard Error Pattern

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Usage:
result, err := divide(10, 2)
if err != nil {
    fmt.Printf("Error: %v\n", err)
} else {
    fmt.Printf("10 / 2 = %.2f\n", result)
}
// Output: 10 / 2 = 5.00
```

### Multiple Related Values

```go
func stats(nums []int) (min, max int, avg float64) {
    if len(nums) == 0 {
        return 0, 0, 0
    }
    min, max = nums[0], nums[0]
    sum := 0
    for _, n := range nums {
        if n < min { min = n }
        if n > max { max = n }
        sum += n
    }
    avg = float64(sum) / float64(len(nums))
    return  // Named return
}

// Usage:
min, max, avg := stats([]int{5, 2, 8, 1, 9, 3})
fmt.Printf("min=%d, max=%d, avg=%.2f\n", min, max, avg)
// Output: min=1, max=9, avg=4.67
```

### Value, Exists Pattern (Map Lookup)

```go
users := map[string]int{"alice": 25, "bob": 30}

if age, exists := users["alice"]; exists {
    fmt.Printf("Alice's age: %d\n", age)
}
// Output: Alice's age: 25

if _, exists := users["charlie"]; !exists {
    fmt.Println("Charlie not found")
}
// Output: Charlie not found
```

### Type Assertion Pattern

```go
var val interface{} = "hello"
if str, ok := val.(string); ok {
    fmt.Printf("It's a string: %q\n", str)
}
// Output: It's a string: "hello"
```

### Ignoring Values with _

```go
func divideInt(a, b int) (int, int) {
    return a / b, a % b
}

quotient, _ := divideInt(17, 5)  // Ignore remainder
fmt.Printf("17 / 5 = %d (remainder ignored)\n", quotient)
// Output: 17 / 5 = 3 (remainder ignored)
```

---

## 📝 Complete Example

```go
package main

import (
    "errors"
    "fmt"
    "strconv"
)

func main() {
    // Standard error pattern
    if result, err := divide(10, 2); err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("10 / 2 = %.2f\n", result)
    }
    
    if _, err := divide(10, 0); err != nil {
        fmt.Printf("Error: %v\n", err)
    }
    
    // Multiple values
    min, max, avg := stats([]int{5, 2, 8, 1, 9, 3})
    fmt.Printf("Stats: min=%d, max=%d, avg=%.2f\n", min, max, avg)
    
    // Value, exists pattern
    users := map[string]int{"alice": 25, "bob": 30}
    if age, exists := users["alice"]; exists {
        fmt.Printf("Alice's age: %d\n", age)
    }
    
    // Ignoring values
    quotient, _ := divideInt(17, 5)
    fmt.Printf("17 / 5 = %d (remainder ignored)\n", quotient)
    
    _ = strconv.Itoa(1)  // Use strconv
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
        if n < min { min = n }
        if n > max { max = n }
        sum += n
    }
    avg = float64(sum) / float64(len(nums))
    return
}

func divideInt(a, b int) (int, int) {
    return a / b, a % b
}
```

**Output:**
```
10 / 2 = 5.00
Error: division by zero
Stats: min=1, max=9, avg=4.67
Alice's age: 25
17 / 5 = 3 (remainder ignored)
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
