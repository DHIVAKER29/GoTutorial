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

### What Break Does

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  break = "Exit this loop immediately"                           │
│                                                                 │
│  for i := 0; i < 10; i++ {                                      │
│      if i == 5 {                                                │
│          break  // Exit the loop                                │
│      }                                                          │
│      fmt.Println(i)                                             │
│  }                                                              │
│  // Output: 0, 1, 2, 3, 4                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Break

```go
// break_demo.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              BREAK STATEMENT                              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic break
    fmt.Println("\n📊 Basic Break:")
    fmt.Print("   ")
    for i := 0; i < 10; i++ {
        if i == 5 {
            break
        }
        fmt.Printf("%d ", i)
    }
    fmt.Println("← stopped at 5")
    
    // Break in switch
    fmt.Println("\n📊 Break in Switch:")
    x := 2
    switch x {
    case 1:
        fmt.Println("   One")
    case 2:
        fmt.Println("   Two")
        break  // Optional in switch (implicit)
        fmt.Println("   This won't print")
    case 3:
        fmt.Println("   Three")
    }
    
    // Break with condition
    fmt.Println("\n📊 Find First Even Number > 10:")
    numbers := []int{1, 5, 7, 12, 15, 18}
    for _, n := range numbers {
        if n > 10 && n%2 == 0 {
            fmt.Printf("   Found: %d\n", n)
            break
        }
    }
    
    // Break in nested loops (only inner!)
    fmt.Println("\n⚠️ Break in Nested Loops (only exits INNER loop):")
    for i := 1; i <= 3; i++ {
        for j := 1; j <= 3; j++ {
            if j == 2 {
                break  // Only exits inner loop!
            }
            fmt.Printf("   i=%d, j=%d\n", i, j)
        }
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              BREAK STATEMENT                              ║
╚══════════════════════════════════════════════════════════╝

📊 Basic Break:
   0 1 2 3 4 ← stopped at 5

📊 Break in Switch:
   Two

📊 Find First Even Number > 10:
   Found: 12

⚠️ Break in Nested Loops (only exits INNER loop):
   i=1, j=1
   i=2, j=1
   i=3, j=1
```

---

## ⏭️ Continue Statement

### What Continue Does

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  continue = "Skip rest of this iteration, go to next"          │
│                                                                 │
│  for i := 0; i < 5; i++ {                                       │
│      if i == 2 {                                                │
│          continue  // Skip i=2                                  │
│      }                                                          │
│      fmt.Println(i)                                             │
│  }                                                              │
│  // Output: 0, 1, 3, 4 (skips 2)                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Continue

```go
// continue_demo.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              CONTINUE STATEMENT                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic continue
    fmt.Println("\n📊 Basic Continue (skip even numbers):")
    fmt.Print("   Odd numbers: ")
    for i := 0; i < 10; i++ {
        if i%2 == 0 {
            continue  // Skip even
        }
        fmt.Printf("%d ", i)
    }
    fmt.Println()
    
    // Skip specific values
    fmt.Println("\n📊 Skip Specific Values:")
    skip := map[int]bool{3: true, 5: true, 7: true}
    fmt.Print("   Keeping: ")
    for i := 1; i <= 10; i++ {
        if skip[i] {
            continue
        }
        fmt.Printf("%d ", i)
    }
    fmt.Println()
    
    // Process only valid items
    fmt.Println("\n📊 Process Only Valid Items:")
    items := []string{"apple", "", "banana", "", "cherry"}
    for _, item := range items {
        if item == "" {
            continue  // Skip empty
        }
        fmt.Printf("   Processing: %s\n", item)
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              CONTINUE STATEMENT                           ║
╚══════════════════════════════════════════════════════════╝

📊 Basic Continue (skip even numbers):
   Odd numbers: 1 3 5 7 9 

📊 Skip Specific Values:
   Keeping: 1 2 4 6 8 9 10 

📊 Process Only Valid Items:
   Processing: apple
   Processing: banana
   Processing: cherry
```

---

## 🏷️ Labels

### Why Labels Exist

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  THE PROBLEM:                                                   │
│                                                                 │
│  for i := 0; i < 10; i++ {                                      │
│      for j := 0; j < 10; j++ {                                  │
│          if found {                                             │
│              break  // Only exits INNER loop!                   │
│          }                                                      │
│      }                                                          │
│      // Still in outer loop...                                  │
│  }                                                              │
│                                                                 │
│  THE SOLUTION: Labels                                           │
│                                                                 │
│  outer:                                                         │
│  for i := 0; i < 10; i++ {                                      │
│      for j := 0; j < 10; j++ {                                  │
│          if found {                                             │
│              break outer  // Exits OUTER loop!                  │
│          }                                                      │
│      }                                                          │
│  }                                                              │
│  // Outside both loops                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Labels

```go
// labels_demo.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              LABELS                                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Break with label
    fmt.Println("\n📊 Break with Label (exit outer loop):")
    
outer:
    for i := 1; i <= 3; i++ {
        for j := 1; j <= 3; j++ {
            if i == 2 && j == 2 {
                fmt.Println("   Breaking outer at i=2, j=2")
                break outer
            }
            fmt.Printf("   i=%d, j=%d\n", i, j)
        }
    }
    fmt.Println("   Outside both loops")
    
    // Continue with label
    fmt.Println("\n📊 Continue with Label (skip to next outer iteration):")
    
outerLoop:
    for i := 1; i <= 3; i++ {
        for j := 1; j <= 3; j++ {
            if j == 2 {
                fmt.Printf("   Skipping rest of inner at i=%d\n", i)
                continue outerLoop
            }
            fmt.Printf("   i=%d, j=%d\n", i, j)
        }
    }
    
    // Practical: Search in 2D array
    fmt.Println("\n💡 Practical: Search in 2D Array:")
    matrix := [][]int{
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9},
    }
    target := 5
    
search:
    for i, row := range matrix {
        for j, val := range row {
            if val == target {
                fmt.Printf("   Found %d at [%d][%d]\n", target, i, j)
                break search
            }
        }
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              LABELS                                       ║
╚══════════════════════════════════════════════════════════╝

📊 Break with Label (exit outer loop):
   i=1, j=1
   i=1, j=2
   i=1, j=3
   i=2, j=1
   Breaking outer at i=2, j=2
   Outside both loops

📊 Continue with Label (skip to next outer iteration):
   i=1, j=1
   Skipping rest of inner at i=1
   i=2, j=1
   Skipping rest of inner at i=2
   i=3, j=1
   Skipping rest of inner at i=3

💡 Practical: Search in 2D Array:
   Found 5 at [1][1]
```

---

## 🚀 Goto Statement

### When to Use (Rarely!)

```go
// goto_demo.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              GOTO STATEMENT (Use Rarely!)                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 Goto for Cleanup (rare use case):")
    
    err := processWithGoto()
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    }
    
    fmt.Println("\n⚠️ Goto Restrictions:")
    fmt.Println("   ❌ Cannot jump over variable declarations")
    fmt.Println("   ❌ Cannot jump into blocks")
    fmt.Println("   ❌ Cannot jump between functions")
    
    fmt.Println("\n💡 Better Alternatives:")
    fmt.Println("   • Use labeled break/continue")
    fmt.Println("   • Use defer for cleanup")
    fmt.Println("   • Use early return")
}

func processWithGoto() error {
    // Allocate resources
    fmt.Println("   Allocating resource 1...")
    
    // Check for error
    if false {  // Simulate error
        goto cleanup
    }
    
    fmt.Println("   Allocating resource 2...")
    
    if false {  // Simulate error
        goto cleanup
    }
    
    fmt.Println("   All resources allocated!")
    return nil

cleanup:
    fmt.Println("   Cleaning up...")
    return fmt.Errorf("resource allocation failed")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              GOTO STATEMENT (Use Rarely!)                 ║
╚══════════════════════════════════════════════════════════╝

📊 Goto for Cleanup (rare use case):
   Allocating resource 1...
   Allocating resource 2...
   All resources allocated!

⚠️ Goto Restrictions:
   ❌ Cannot jump over variable declarations
   ❌ Cannot jump into blocks
   ❌ Cannot jump between functions

💡 Better Alternatives:
   • Use labeled break/continue
   • Use defer for cleanup
   • Use early return
```

---

## 🏭 Production Patterns

```go
// loop_control_production.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION PATTERNS                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: Early exit on error
    fmt.Println("\n📊 Pattern 1: Early Exit on Error")
    items := []string{"valid", "valid", "ERROR", "valid"}
    
    for i, item := range items {
        if item == "ERROR" {
            fmt.Printf("   Error at index %d, stopping\n", i)
            break
        }
        fmt.Printf("   Processing: %s\n", item)
    }
    
    // Pattern 2: Skip invalid items
    fmt.Println("\n📊 Pattern 2: Skip Invalid Items")
    data := []int{1, -5, 3, -2, 7, 0, 4}
    fmt.Print("   Positive numbers: ")
    for _, n := range data {
        if n <= 0 {
            continue
        }
        fmt.Printf("%d ", n)
    }
    fmt.Println()
    
    // Pattern 3: Find in nested structure
    fmt.Println("\n📊 Pattern 3: Find in Nested Structure")
    users := map[string][]string{
        "admin":  {"read", "write", "delete"},
        "editor": {"read", "write"},
        "viewer": {"read"},
    }
    
    targetPerm := "delete"
findPerm:
    for role, perms := range users {
        for _, perm := range perms {
            if perm == targetPerm {
                fmt.Printf("   Role '%s' has '%s' permission\n", role, targetPerm)
                break findPerm
            }
        }
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           PRODUCTION PATTERNS                             ║
╚══════════════════════════════════════════════════════════╝

📊 Pattern 1: Early Exit on Error
   Processing: valid
   Processing: valid
   Error at index 2, stopping

📊 Pattern 2: Skip Invalid Items
   Positive numbers: 1 3 7 4 

📊 Pattern 3: Find in Nested Structure
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

