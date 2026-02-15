# 12 - Switch Statements

> Go's powerful switch statement with unique features.

---

## 📌 What You'll Learn

- Basic switch syntax
- Switch without expression (cleaner if-else)
- Type switch for interfaces
- Fallthrough (opt-in, not default!)
- Sample programs for each pattern

---

## 🔀 Switch Basics

Go's switch differs from C/Java:

- **No automatic fallthrough**—each case stops by default (unlike C/Java where you need `break`)
- **Multiple values per case**: `case 1, 2, 3:`
- **Cases can be expressions** (not just constants)
- **Switch without expression** = cleaner if-else chain
- **Type switch** for interface values

---

## 📝 Basic Switch Snippets

```go
// Basic switch
day := time.Saturday
switch day {
case time.Saturday, time.Sunday:
    fmt.Println("It's the weekend!")
case time.Monday:
    fmt.Println("Monday blues...")
default:
    fmt.Printf("Regular day: %s\n", day)
}
// Output: It's the weekend!
```

```go
// Switch with expression (no value after switch)
score := 85
switch {
case score >= 90:
    fmt.Println("Grade: A")
case score >= 80:
    fmt.Println("Grade: B")
case score >= 70:
    fmt.Println("Grade: C")
default:
    fmt.Println("Grade: F")
}
// Output: Grade: B
```

```go
// Switch with init statement
switch hour := 14; {
case hour < 12:
    fmt.Println("Good morning!")
case hour < 17:
    fmt.Println("Good afternoon!")
default:
    fmt.Println("Good evening!")
}
// Output: Good afternoon!
```

```go
// Multiple values per case
char := 'a'
switch char {
case 'a', 'e', 'i', 'o', 'u':
    fmt.Printf("'%c' is a vowel\n", char)
default:
    fmt.Printf("'%c' is a consonant\n", char)
}
// Output: 'a' is a vowel
```

### Complete Example: Switch Basics

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    day := time.Now().Weekday()
    switch day {
    case time.Saturday, time.Sunday:
        fmt.Println("It's the weekend!")
    case time.Monday:
        fmt.Println("Monday blues...")
    case time.Friday:
        fmt.Println("TGIF!")
    default:
        fmt.Printf("Regular day: %s\n", day)
    }

    score := 85
    switch {
    case score >= 90:
        fmt.Println("Grade: A")
    case score >= 80:
        fmt.Println("Grade: B")
    case score >= 70:
        fmt.Println("Grade: C")
    case score >= 60:
        fmt.Println("Grade: D")
    default:
        fmt.Println("Grade: F")
    }

    switch hour := time.Now().Hour(); {
    case hour < 12:
        fmt.Println("Good morning!")
    case hour < 17:
        fmt.Println("Good afternoon!")
    default:
        fmt.Println("Good evening!")
    }
}
```

**Output:**
```
Regular day: Wednesday
Grade: B
Good afternoon!
```

---

## 🔽 Fallthrough

In Go, fallthrough is **opt-in**. Use the `fallthrough` keyword to continue to the next case. By default, each case stops automatically.

```go
num := 1
switch num {
case 1:
    fmt.Println("One")
    fallthrough  // Continue to next case
case 2:
    fmt.Println("Two (via fallthrough)")
    fallthrough
case 3:
    fmt.Println("Three (via fallthrough)")
}
// Output: One
//         Two (via fallthrough)
//         Three (via fallthrough)
```

**Practical use:** Cumulative permission levels—higher levels include lower-level permissions.

### Complete Example: Fallthrough

```go
package main

import "fmt"

func main() {
    num := 1
    switch num {
    case 1:
        fmt.Println("One")
        fallthrough
    case 2:
        fmt.Println("Two (via fallthrough)")
        fallthrough
    case 3:
        fmt.Println("Three (via fallthrough)")
    case 4:
        fmt.Println("Four (not reached)")
    }

    level := 2
    fmt.Printf("Level %d permissions:\n", level)
    switch level {
    case 3:
        fmt.Println("- Super Admin access")
        fallthrough
    case 2:
        fmt.Println("- Admin access")
        fallthrough
    case 1:
        fmt.Println("- User access")
        fallthrough
    case 0:
        fmt.Println("- Guest access")
    }
}
```

**Output:**
```
One
Two (via fallthrough)
Three (via fallthrough)
Level 2 permissions:
- Admin access
- User access
- Guest access
```

---

## 🔍 Type Switch

Use a type switch to branch on the dynamic type of an interface value.

```go
var value interface{} = 42
switch v := value.(type) {
case int:
    fmt.Printf("Integer: %d\n", v)
case string:
    fmt.Printf("String: %q\n", v)
case nil:
    fmt.Println("nil value")
default:
    fmt.Printf("Unknown: %T\n", v)
}
// Output: Integer: 42
```

### Complete Example: Type Switch

```go
package main

import "fmt"

func describe(value interface{}) {
    switch v := value.(type) {
    case nil:
        fmt.Println("nil value")
    case int:
        fmt.Printf("Integer: %d (doubled: %d)\n", v, v*2)
    case string:
        fmt.Printf("String: %q (length: %d)\n", v, len(v))
    case float64:
        fmt.Printf("Float: %f\n", v)
    case bool:
        fmt.Printf("Boolean: %t\n", v)
    default:
        fmt.Printf("Unknown type: %T = %v\n", v, v)
    }
}

func main() {
    describe(42)
    describe("hello")
    describe(3.14)
    describe(true)
    describe(nil)
}
```

**Output:**
```
Integer: 42 (doubled: 84)
String: "hello" (length: 5)
Float: 3.140000
Boolean: true
nil value
```

---

## 🏭 Production Patterns

```go
// HTTP status handling
switch code {
case http.StatusOK:
    fmt.Println("Success")
case http.StatusNotFound:
    fmt.Println("Not Found")
case http.StatusInternalServerError:
    fmt.Println("Server Error")
default:
    fmt.Printf("Other: %d\n", code)
}
```

```go
// Expression switch (replaces if-else chain)
switch {
case age < 0:
    fmt.Println("Invalid")
case age < 13:
    fmt.Println("Child")
case age < 20:
    fmt.Println("Teenager")
case age < 65:
    fmt.Println("Adult")
default:
    fmt.Println("Senior")
}
```

### Complete Example: Production Patterns

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    handleStatus(200)
    handleStatus(404)
    handleStatus(500)

    checkAge(5)
    checkAge(25)
    checkAge(70)
}

func handleStatus(code int) {
    switch code {
    case http.StatusOK:
        fmt.Printf("%d: Success\n", code)
    case http.StatusNotFound:
        fmt.Printf("%d: Not Found\n", code)
    case http.StatusInternalServerError:
        fmt.Printf("%d: Server Error\n", code)
    default:
        fmt.Printf("%d: Other status\n", code)
    }
}

func checkAge(age int) {
    switch {
    case age < 0:
        fmt.Printf("Age %d: Invalid\n", age)
    case age < 13:
        fmt.Printf("Age %d: Child\n", age)
    case age < 20:
        fmt.Printf("Age %d: Teenager\n", age)
    case age < 65:
        fmt.Printf("Age %d: Adult\n", age)
    default:
        fmt.Printf("Age %d: Senior\n", age)
    }
}
```

**Output:**
```
200: Success
404: Not Found
500: Server Error
Age 5: Child
Age 25: Adult
Age 70: Senior
```

---

## 🆚 Java Comparison

| Aspect | Java | Go |
|--------|------|-----|
| Break | Required in each case | Not needed—automatic |
| Multiple values | `case 1: case 2: case 3:` | `case 1, 2, 3:` |
| Expression switch | Must use if-else | `switch { case x > 0: }` |
| Fallthrough | Default (need break) | Opt-in (need fallthrough) |

---

## 🎯 Key Takeaways

1. **No automatic fallthrough**—Go is safer by default
2. **No break needed**—each case stops automatically
3. **Multiple values** per case: `case 1, 2, 3:`
4. **Expression switch** (`switch { case x > 0: }`) replaces if-else chains
5. **Type switch** for interface values: `switch v := x.(type)`
6. **Init statement** supported: `switch x := getValue(); x {`
7. **fallthrough** is explicit and opt-in

---

## ➡️ Next Steps

**Next Topic:** [13 - For Loops](./13-for-loops.md)
