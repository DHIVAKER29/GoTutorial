# 14 - Break, Continue & Labels

> Controlling loop flow with break, continue, and labeled statements.

---

## 📌 What You'll Learn

- How `break` exits loops
- How `continue` skips iterations
- Using labels for nested loops
- `goto` statement (rare but exists)
- When to use each

---

## 🛑 Break Statement

**break** exits the innermost loop or switch immediately.

```go
for i := 0; i < 10; i++ {
    if i == 5 {
        break
    }
    fmt.Print(i, " ")
}
// Output: 0 1 2 3 4
```

```go
// Break with condition
numbers := []int{1, 5, 7, 12, 15, 18}
for _, n := range numbers {
    if n > 10 && n%2 == 0 {
        fmt.Printf("Found: %d\n", n)
        break
    }
}
// Output: Found: 12
```

**Important:** In nested loops, `break` exits only the **inner** loop.

### Complete Example: Break

```go
package main

import "fmt"

func main() {
    fmt.Print("Basic break: ")
    for i := 0; i < 10; i++ {
        if i == 5 {
            break
        }
        fmt.Printf("%d ", i)
    }
    fmt.Println("← stopped at 5")

    numbers := []int{1, 5, 7, 12, 15, 18}
    for _, n := range numbers {
        if n > 10 && n%2 == 0 {
            fmt.Printf("First even > 10: %d\n", n)
            break
        }
    }

    fmt.Println("Nested break (exits inner only):")
    for i := 1; i <= 3; i++ {
        for j := 1; j <= 3; j++ {
            if j == 2 {
                break
            }
            fmt.Printf("  i=%d, j=%d\n", i, j)
        }
    }
}
```

**Output:**
```
Basic break: 0 1 2 3 4 ← stopped at 5
First even > 10: 12
Nested break (exits inner only):
  i=1, j=1
  i=2, j=1
  i=3, j=1
```

---

## ⏭️ Continue Statement

**continue** skips the rest of the current iteration and moves to the next one.

```go
for i := 0; i < 5; i++ {
    if i == 2 {
        continue
    }
    fmt.Print(i, " ")
}
// Output: 0 1 3 4
```

```go
// Skip empty items
items := []string{"apple", "", "banana", "", "cherry"}
for _, item := range items {
    if item == "" {
        continue
    }
    fmt.Printf("Processing: %s\n", item)
}
// Output: Processing: apple
//         Processing: banana
//         Processing: cherry
```

### Complete Example: Continue

```go
package main

import "fmt"

func main() {
    fmt.Print("Odd numbers: ")
    for i := 0; i < 10; i++ {
        if i%2 == 0 {
            continue
        }
        fmt.Printf("%d ", i)
    }
    fmt.Println()

    items := []string{"apple", "", "banana", "", "cherry"}
    for _, item := range items {
        if item == "" {
            continue
        }
        fmt.Printf("Processing: %s\n", item)
    }
}
```

**Output:**
```
Odd numbers: 1 3 5 7 9 
Processing: apple
Processing: banana
Processing: cherry
```

---

## 🏷️ Labels

**Problem:** In nested loops, `break` only exits the inner loop. **Solution:** Use labels to break or continue an outer loop.

```go
outer:
for i := 0; i < 10; i++ {
    for j := 0; j < 10; j++ {
        if found {
            break outer  // Exits outer loop
        }
    }
}
```

```go
// Continue with label skips to next outer iteration
outerLoop:
for i := 1; i <= 3; i++ {
    for j := 1; j <= 3; j++ {
        if j == 2 {
            continue outerLoop
        }
        fmt.Printf("i=%d, j=%d\n", i, j)
    }
}
```

### Complete Example: Labels

```go
package main

import "fmt"

func main() {
    fmt.Println("Break with label:")
outer:
    for i := 1; i <= 3; i++ {
        for j := 1; j <= 3; j++ {
            if i == 2 && j == 2 {
                fmt.Println("Breaking outer at i=2, j=2")
                break outer
            }
            fmt.Printf("  i=%d, j=%d\n", i, j)
        }
    }
    fmt.Println("Outside both loops")

    fmt.Println("\nSearch in 2D array:")
    matrix := [][]int{{1, 2, 3}, {4, 5, 6}, {7, 8, 9}}
    target := 5
search:
    for i, row := range matrix {
        for j, val := range row {
            if val == target {
                fmt.Printf("Found %d at [%d][%d]\n", target, i, j)
                break search
            }
        }
    }
}
```

**Output:**
```
Break with label:
  i=1, j=1
  i=1, j=2
  i=1, j=3
  i=2, j=1
Breaking outer at i=2, j=2
Outside both loops

Search in 2D array:
Found 5 at [1][1]
```

---

## 🚀 Goto Statement

`goto` exists in Go but is rarely needed. Use it sparingly—prefer labeled break/continue, defer for cleanup, or early return.

**Restrictions:**
- Cannot jump over variable declarations
- Cannot jump into blocks
- Cannot jump between functions

```go
func processWithGoto() error {
    fmt.Println("Allocating resource 1...")
    if err != nil {
        goto cleanup
    }
    fmt.Println("Allocating resource 2...")
    return nil
cleanup:
    fmt.Println("Cleaning up...")
    return fmt.Errorf("failed")
}
```

**Better alternatives:** labeled break/continue, defer for cleanup, early return.

---

## 🏭 Production Patterns

```go
// Early exit on error
for i, item := range items {
    if item == "ERROR" {
        fmt.Printf("Error at index %d\n", i)
        break
    }
    process(item)
}
```

```go
// Skip invalid items
for _, n := range data {
    if n <= 0 {
        continue
    }
    fmt.Print(n, " ")
}
```

```go
// Find in nested structure with label
findPerm:
for role, perms := range users {
    for _, perm := range perms {
        if perm == targetPerm {
            fmt.Printf("Role '%s' has permission\n", role)
            break findPerm
        }
    }
}
```

### Complete Example: Production Patterns

```go
package main

import "fmt"

func main() {
    items := []string{"valid", "valid", "ERROR", "valid"}
    for i, item := range items {
        if item == "ERROR" {
            fmt.Printf("Error at index %d, stopping\n", i)
            break
        }
        fmt.Printf("Processing: %s\n", item)
    }

    data := []int{1, -5, 3, -2, 7, 0, 4}
    fmt.Print("Positive numbers: ")
    for _, n := range data {
        if n <= 0 {
            continue
        }
        fmt.Printf("%d ", n)
    }
    fmt.Println()

    users := map[string][]string{
        "admin":  {"read", "write", "delete"},
        "editor": {"read", "write"},
    }
    targetPerm := "delete"
findPerm:
    for role, perms := range users {
        for _, perm := range perms {
            if perm == targetPerm {
                fmt.Printf("Role '%s' has '%s' permission\n", role, targetPerm)
                break findPerm
            }
        }
    }
}
```

**Output:**
```
Processing: valid
Processing: valid
Error at index 2, stopping
Positive numbers: 1 3 7 4 
Role 'admin' has 'delete' permission
```

---

## 🎯 Key Takeaways

1. **`break`** exits the innermost loop or switch
2. **`continue`** skips to the next iteration
3. **Labels** allow breaking/continuing outer loops
4. **`goto`** exists but rarely needed (use defer instead)
5. **Use labels** for nested loop control
6. **Prefer early return** over goto for cleanup

---

## ➡️ Next Steps

**Next Topic:** [15 - Functions - Basics](./15-functions-basics.md)
