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

**Real-World Analogy:** Like a traffic light—if green, go; if yellow, slow down; else stop. Your program needs to make decisions too.

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

- **No parentheses** needed (but allowed): `if x > 0 { }` is idiomatic Go
- **Braces are required**—even for single statements; `if x > 0 fmt.Println("x")` causes a compile error
- **Opening brace** must be on the same line as the condition
- **Condition must be boolean**—Go has no truthy/falsy; `if 1 { }` is invalid

---

## 📝 Basic If-Else Snippets

```go
// Basic if
age := 25
if age >= 18 {
    fmt.Println("You are an adult")
}
// Output: You are an adult
```

```go
// If-else
score := 75
if score >= 60 {
    fmt.Println("PASS")
} else {
    fmt.Println("FAIL")
}
// Output: PASS
```

```go
// If-else if-else (grade)
g := 82
if g >= 90 {
    fmt.Println("A")
} else if g >= 80 {
    fmt.Println("B")
} else if g >= 70 {
    fmt.Println("C")
} else {
    fmt.Println("F")
}
// Output: B
```

```go
// Multiple conditions
temperature, isRaining := 22, false
if temperature > 20 && !isRaining {
    fmt.Println("Perfect weather for a walk!")
}
// Output: Perfect weather for a walk!
```

### Complete Example: Basic If-Else

```go
package main

import "fmt"

func main() {
    age := 25
    if age >= 18 {
        fmt.Printf("Age %d: You are an adult\n", age)
    }

    score := 75
    if score >= 60 {
        fmt.Printf("Score %d: PASS\n", score)
    } else {
        fmt.Printf("Score %d: FAIL\n", score)
    }

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
        fmt.Printf("Score %d → Grade %s\n", g, grade)
    }
}
```

**Output:**
```
Age 25: You are an adult
Score 75: PASS
Score 95 → Grade A
Score 82 → Grade B
Score 73 → Grade C
Score 65 → Grade D
Score 45 → Grade F
```

---

## ⭐ If with Initialization (Go's Unique Feature!)

You can declare a variable right in the if statement. The variable is scoped to the if block only.

```go
if x := 10; x > 5 {
    fmt.Printf("x = %d is greater than 5\n", x)
}
// x is NOT available here (scoped to if block)
// Output: x = 10 is greater than 5
```

```go
// Error handling pattern
if num, err := strconv.Atoi("42"); err != nil {
    fmt.Printf("Error: %v\n", err)
} else {
    fmt.Printf("Parsed: %d\n", num)
}
// Output: Parsed: 42
```

```go
// Map lookup pattern
users := map[string]int{"alice": 25, "bob": 30}
if age, exists := users["alice"]; exists {
    fmt.Printf("Alice's age: %d\n", age)
} else {
    fmt.Println("Alice not found")
}
// Output: Alice's age: 25
```

**Scope benefit:** With init statement, variables don't leak into outer scope. Without it, `err` would remain in scope after the if block.

### Complete Example: If with Initialization

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    if x := 10; x > 5 {
        fmt.Printf("x = %d is greater than 5\n", x)
    }

    if num, err := strconv.Atoi("42"); err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Parsed: %d\n", num)
    }

    if num, err := strconv.Atoi("not-a-number"); err != nil {
        fmt.Printf("Error parsing: %v\n", err)
    } else {
        fmt.Printf("Parsed: %d\n", num)
    }

    users := map[string]int{"alice": 25, "bob": 30}
    if age, exists := users["alice"]; exists {
        fmt.Printf("Alice's age: %d\n", age)
    }
    if _, exists := users["charlie"]; !exists {
        fmt.Println("Charlie not found")
    }
}
```

**Output:**
```
x = 10 is greater than 5
Parsed: 42
Error parsing: strconv.Atoi: parsing "not-a-number": invalid syntax
Alice's age: 25
Charlie not found
```

---

## 🏭 Production Patterns

```go
// Guard clause (early return)
if err := validateUser(user); err != nil {
    return err
}
```

```go
// Nil check
if u == nil {
    fmt.Println("No user provided")
    return
}
fmt.Printf("User: %s\n", u.Name)
```

```go
// Default value from map
if value, exists := configs[key]; exists {
    return value
}
return "default"
```

### Complete Example: Production Patterns

```go
package main

import (
    "errors"
    "fmt"
)

type User struct {
    Name string
    Age  int
}

func validateUser(u User) error {
    if u.Name == "" {
        return errors.New("name is required")
    }
    if u.Age < 0 {
        return errors.New("age cannot be negative")
    }
    return nil
}

func main() {
    user := User{Name: "Alice", Age: 25}
    if err := validateUser(user); err != nil {
        fmt.Printf("Validation failed: %v\n", err)
    } else {
        fmt.Printf("User %s is valid\n", user.Name)
    }

    configs := map[string]string{"port": "8080", "host": "localhost"}
    if value, exists := configs["port"]; exists {
        fmt.Printf("Port: %s\n", value)
    }
    if _, exists := configs["missing"]; !exists {
        fmt.Println("Missing config: using default")
    }
}
```

**Output:**
```
User Alice is valid
Port: 8080
Missing config: using default
```

---

## 🆚 Java Comparison

| Aspect | Java | Go |
|--------|------|-----|
| Syntax | `if (x > 0) { }` | `if x > 0 { }` |
| Init in condition | `int x = getValue(); if (x > 0) { }` | `if x := getValue(); x > 0 { }` |
| Ternary | `result = x > 0 ? "yes" : "no"` | No ternary—use if-else |
| Truthy/falsy | `if (object)` for null check | `if object != nil` (strict boolean) |
| Zero check | `if (count)` | `if count != 0` |

---

## 🎯 Key Takeaways

1. **No parentheses** needed around condition
2. **Braces are required** (even for single statements)
3. **Opening brace** must be on same line
4. **Condition must be boolean** (no truthy/falsy!)
5. **Init statement** is Go's powerful feature: `if x := val; x > 0 {}`
6. **Variables in init** are scoped to the if block
7. **No ternary operator** in Go—use if-else

---

## ➡️ Next Steps

You now understand if-else. Let's explore switch statements.

**Next Topic:** [12 - Switch Statements](./12-switch.md)
