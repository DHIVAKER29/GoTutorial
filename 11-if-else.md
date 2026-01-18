# 11 - If-Else Statements

> Making decisions in Go with conditional statements.

---

## 📌 What You'll Learn

- Basic if-else syntax
- If with initialization statement (Go's unique feature!)
- Nested conditions
- Common patterns and best practices
- Sample programs for each pattern

---

## 🤔 What is a Conditional?

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  TRAFFIC LIGHT DECISION                                         │
│                                                                 │
│  if light is green {                                            │
│      go                                                         │
│  } else if light is yellow {                                    │
│      slow down                                                  │
│  } else {                                                       │
│      stop                                                       │
│  }                                                              │
│                                                                 │
│  Your program needs to make decisions too!                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Basic Syntax

```go
// Basic if
if condition {
    // code
}

// If-else
if condition {
    // code when true
} else {
    // code when false
}

// If-else if-else
if condition1 {
    // code when condition1 is true
} else if condition2 {
    // code when condition2 is true
} else {
    // code when all conditions are false
}
```

### Key Rules

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO's if STATEMENT RULES                                        │
│                                                                 │
│  1. NO PARENTHESES needed (but allowed)                         │
│     if x > 0 { }    ✅ Go style                                 │
│     if (x > 0) { }  ✅ Works but not idiomatic                  │
│                                                                 │
│  2. BRACES are REQUIRED                                         │
│     if x > 0                                                    │
│         fmt.Println("positive")  ❌ COMPILE ERROR               │
│                                                                 │
│     if x > 0 {                                                  │
│         fmt.Println("positive")  ✅ Correct                     │
│     }                                                           │
│                                                                 │
│  3. Opening brace on SAME LINE                                  │
│     if x > 0                                                    │
│     {                            ❌ COMPILE ERROR               │
│     }                                                           │
│                                                                 │
│  4. Condition MUST be boolean                                   │
│     if 1 { }         ❌ COMPILE ERROR (no truthy/falsy!)        │
│     if x > 0 { }     ✅ Correct                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Sample Program: Basic If-Else

```go
// if_else_basic.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           IF-ELSE STATEMENTS IN GO                        ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic if
    fmt.Println("\n📊 Basic if:")
    age := 25
    if age >= 18 {
        fmt.Printf("   Age %d: You are an adult\n", age)
    }
    
    // If-else
    fmt.Println("\n📊 If-else:")
    score := 75
    if score >= 60 {
        fmt.Printf("   Score %d: PASS ✅\n", score)
    } else {
        fmt.Printf("   Score %d: FAIL ❌\n", score)
    }
    
    // If-else if-else (grade calculation)
    fmt.Println("\n📊 If-else if-else (Grade Calculator):")
    grades := []int{95, 82, 73, 65, 45}
    
    for _, g := range grades {
        var grade string
        if g >= 90 {
            grade = "A"
        } else if g >= 80 {
            grade = "B"
        } else if g >= 70 {
            grade = "C"
        } else if g >= 60 {
            grade = "D"
        } else {
            grade = "F"
        }
        fmt.Printf("   Score %d → Grade %s\n", g, grade)
    }
    
    // Multiple conditions
    fmt.Println("\n📊 Multiple Conditions:")
    temperature := 22
    isRaining := false
    
    if temperature > 20 && !isRaining {
        fmt.Println("   Perfect weather for a walk! 🚶")
    } else if temperature > 20 && isRaining {
        fmt.Println("   Warm but rainy. Take an umbrella! ☔")
    } else {
        fmt.Println("   Stay inside! 🏠")
    }
    
    // Comparison operators
    fmt.Println("\n📊 All Comparison Operators:")
    x, y := 10, 20
    fmt.Printf("   x = %d, y = %d\n", x, y)
    fmt.Printf("   x == y : %t\n", x == y)
    fmt.Printf("   x != y : %t\n", x != y)
    fmt.Printf("   x < y  : %t\n", x < y)
    fmt.Printf("   x > y  : %t\n", x > y)
    fmt.Printf("   x <= y : %t\n", x <= y)
    fmt.Printf("   x >= y : %t\n", x >= y)
    
    // Logical operators
    fmt.Println("\n📊 Logical Operators:")
    a, b := true, false
    fmt.Printf("   a = %t, b = %t\n", a, b)
    fmt.Printf("   a && b : %t (AND)\n", a && b)
    fmt.Printf("   a || b : %t (OR)\n", a || b)
    fmt.Printf("   !a     : %t (NOT)\n", !a)
}
```

---

## ⭐ If with Initialization (Go's Unique Feature!)

### The Powerful Pattern

```go
if initialization; condition {
    // code
}
```

This is one of Go's best features - you can declare a variable right in the if statement!

### Sample Program: If with Initialization

```go
// if_init.go
package main

import (
    "fmt"
    "os"
    "strconv"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║       IF WITH INITIALIZATION (GO's SPECIAL FEATURE)       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic pattern
    fmt.Println("\n📊 Basic Pattern:")
    if x := 10; x > 5 {
        fmt.Printf("   x = %d is greater than 5\n", x)
    }
    // x is NOT available here (scoped to if block)
    
    // Common pattern: Error handling
    fmt.Println("\n📊 Error Handling Pattern:")
    if num, err := strconv.Atoi("42"); err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   Parsed number: %d\n", num)
    }
    
    // With invalid input
    if num, err := strconv.Atoi("not-a-number"); err != nil {
        fmt.Printf("   Error parsing: %v\n", err)
    } else {
        fmt.Printf("   Parsed number: %d\n", num)
    }
    
    // File operations
    fmt.Println("\n📊 File Operations Pattern:")
    if file, err := os.Open("/etc/hosts"); err != nil {
        fmt.Printf("   Cannot open file: %v\n", err)
    } else {
        fmt.Printf("   File opened: %s\n", file.Name())
        file.Close()
    }
    
    // Map lookup
    fmt.Println("\n📊 Map Lookup Pattern:")
    users := map[string]int{
        "alice": 25,
        "bob":   30,
    }
    
    if age, exists := users["alice"]; exists {
        fmt.Printf("   Alice's age: %d\n", age)
    } else {
        fmt.Println("   Alice not found")
    }
    
    if age, exists := users["charlie"]; exists {
        fmt.Printf("   Charlie's age: %d\n", age)
    } else {
        fmt.Println("   Charlie not found")
    }
    
    // Type assertion
    fmt.Println("\n📊 Type Assertion Pattern:")
    var value interface{} = "hello"
    
    if str, ok := value.(string); ok {
        fmt.Printf("   It's a string: %q\n", str)
    } else {
        fmt.Println("   Not a string")
    }
    
    // Chained conditions
    fmt.Println("\n📊 Why This Pattern is Great:")
    fmt.Println("   1. Variable is scoped to if block only")
    fmt.Println("   2. No pollution of outer scope")
    fmt.Println("   3. Clear and concise")
    fmt.Println("   4. Error handling in one line")
}
```

### Scope Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  WITHOUT INIT STATEMENT:                                        │
│  ───────────────────────                                        │
│  err := someFunction()    // err leaks into outer scope         │
│  if err != nil {                                                │
│      return err                                                 │
│  }                                                              │
│  // err is still here! 😕                                       │
│                                                                 │
│  WITH INIT STATEMENT:                                           │
│  ─────────────────────                                          │
│  if err := someFunction(); err != nil {                         │
│      return err                                                 │
│  }                                                              │
│  // err is gone! Clean scope! 🎉                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🏭 Production Patterns

### Sample Program: Real-World Patterns

```go
// if_production.go
package main

import (
    "errors"
    "fmt"
)

// User struct for examples
type User struct {
    Name    string
    Email   string
    Age     int
    IsAdmin bool
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION IF-ELSE PATTERNS                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: Guard clauses (early return)
    fmt.Println("\n📊 Pattern 1: Guard Clauses")
    user := User{Name: "Alice", Email: "alice@example.com", Age: 25}
    
    if err := validateUser(user); err != nil {
        fmt.Printf("   Validation failed: %v\n", err)
    } else {
        fmt.Printf("   User %s is valid ✅\n", user.Name)
    }
    
    // Pattern 2: Authorization check
    fmt.Println("\n📊 Pattern 2: Authorization Check")
    admin := User{Name: "Admin", IsAdmin: true}
    regular := User{Name: "User", IsAdmin: false}
    
    checkAdminAccess(admin)
    checkAdminAccess(regular)
    
    // Pattern 3: Nil check
    fmt.Println("\n📊 Pattern 3: Nil Check")
    var nilUser *User
    validUser := &User{Name: "Bob"}
    
    printUserName(nilUser)
    printUserName(validUser)
    
    // Pattern 4: Default values
    fmt.Println("\n📊 Pattern 4: Default Values")
    config := getConfig("port")
    fmt.Printf("   Port config: %s\n", config)
    
    missing := getConfig("missing")
    fmt.Printf("   Missing config: %s\n", missing)
    
    // Pattern 5: Feature flags
    fmt.Println("\n📊 Pattern 5: Feature Flags")
    features := map[string]bool{
        "dark_mode":    true,
        "new_checkout": false,
    }
    
    if features["dark_mode"] {
        fmt.Println("   🌙 Dark mode enabled")
    }
    
    if features["new_checkout"] {
        fmt.Println("   🛒 New checkout enabled")
    } else {
        fmt.Println("   🛒 Using legacy checkout")
    }
}

// Guard clause pattern - return early on errors
func validateUser(u User) error {
    if u.Name == "" {
        return errors.New("name is required")
    }
    if u.Email == "" {
        return errors.New("email is required")
    }
    if u.Age < 0 {
        return errors.New("age cannot be negative")
    }
    return nil
}

// Authorization check
func checkAdminAccess(u User) {
    if !u.IsAdmin {
        fmt.Printf("   ❌ %s: Access denied\n", u.Name)
        return
    }
    fmt.Printf("   ✅ %s: Admin access granted\n", u.Name)
}

// Nil check pattern
func printUserName(u *User) {
    if u == nil {
        fmt.Println("   No user provided")
        return
    }
    fmt.Printf("   User name: %s\n", u.Name)
}

// Default value pattern
func getConfig(key string) string {
    configs := map[string]string{
        "port": "8080",
        "host": "localhost",
    }
    
    if value, exists := configs[key]; exists {
        return value
    }
    return "default"
}
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  if (x > 0) {                      if x > 0 {                   │
│      // code                           // code                  │
│  }                                 }                            │
│                                                                 │
│  // No init statement              if x := getValue(); x > 0 {  │
│  int x = getValue();                   // code                  │
│  if (x > 0) {                      }                            │
│      // code                                                    │
│  }                                                              │
│                                                                 │
│  // Ternary operator               // No ternary!               │
│  result = x > 0 ? "yes" : "no";    if x > 0 {                   │
│                                        result = "yes"           │
│                                    } else {                     │
│                                        result = "no"            │
│                                    }                            │
│                                                                 │
│  // Truthy/falsy                   // Strict boolean only       │
│  if (object)  // null check        if object != nil             │
│  if (count)   // zero check        if count != 0                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **No parentheses** needed around condition
2. **Braces are required** (even for single statements)
3. **Opening brace** must be on same line
4. **Condition must be boolean** (no truthy/falsy!)
5. **Init statement** is Go's powerful feature: `if x := val; x > 0 {}`
6. **Variables in init** are scoped to the if block
7. **No ternary operator** in Go - use if-else

---

## ➡️ Next Steps

You now understand if-else. Let's explore switch statements.

**Next Topic:** [12 - Switch Statements](./12-switch.md)

