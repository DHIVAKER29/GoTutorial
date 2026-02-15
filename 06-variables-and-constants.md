# 06 - Variables & Constants

> Understanding how Go stores and manages data with variables and constants.

---

## 📌 What You'll Learn

- What variables are and why we need them
- All ways to declare variables in Go
- Zero values and their importance
- Constants and when to use them
- Visibility (exported vs unexported)
- Variable shadowing and its dangers
- Sample programs for each concept

---

## 🤔 What is a Variable?

### The Problem: Storing Information

Without variables, you'd write `fmt.Println(25 * 365)` — but what is 25? What is 365? What if the age changes or you need it multiple times?

**The solution:** Variables give names to values so code is clear and reusable.

```go
age := 25
daysPerYear := 365
ageInDays := age * daysPerYear
fmt.Println(ageInDays) // Output: 9125
```

### Real-World Analogy

A variable is like a **labeled box**:
- **Name** — identifies the box (e.g., `age`, `name`, `price`)
- **Type** — what kind of content it holds (e.g., `int`, `string`, `float64`)
- **Value** — the current contents

You can put something in (assign), look inside (read), or replace contents (reassign). Only values of the same type fit — Go enforces type safety.

---

## 📝 Declaring Variables

### Method 1: var with Type (Explicit)

```go
var name string
var age int
fmt.Println(name, age) // Output:  0
```

The variable gets a **zero value** (explained later).

### Method 2: var with Initialization

```go
var name string = "Alice"
var age int = 25
fmt.Println(name, age) // Output: Alice 25
```

### Method 3: var with Type Inference

```go
var name = "Alice"  // Go infers: string
var age = 25        // Go infers: int
fmt.Println(name, age) // Output: Alice 25
```

### Method 4: Short Declaration `:=` (Most Common!)

```go
name := "Alice"  // Declare AND initialize
age := 25
fmt.Println(name, age) // Output: Alice 25
```

### Grouped Declaration

```go
var (
    name     string  = "Alice"
    age      int     = 25
    isActive bool    = true
)
fmt.Println(name, age, isActive) // Output: Alice 25 true
```

### When to Use Which?

| Declaration | Use When |
|-------------|----------|
| `:=` (Short) | Inside functions, when you have an initial value. Cannot use at package level. |
| `var` with Type | Package-level variables, when you want zero value, when type isn't obvious. |
| `var` with Inference | When the type is obvious from the value, package level with initialization. |

---

## 🎯 Complete Example: Variable Declarations

```go
package main

import "fmt"

var appName = "My App"
var maxUsers int  // Zero value: 0

func main() {
    var count int
    var message string
    fmt.Println(count, message) // 0

    name := "Alice"
    age := 25
    fmt.Println(name, age) // Alice 25

    x, y, z := 1, 2, 3
    fmt.Println(x, y, z) // 1 2 3

    count = 10
    count += 5
    fmt.Println(count) // 15

    fmt.Println(appName, maxUsers) // My App 0
}
```

**Output:**
```
0 
Alice 25
1 2 3
15
My App 0
```

---

## 🔢 Zero Values

### What is a Zero Value?

When you declare a variable without initializing it, Go automatically assigns a **zero value** based on its type.

### Zero Values Table

| Type | Zero Value | Example |
|------|------------|---------|
| `int`, `int8`–`int64` | `0` | `var x int` → `0` |
| `uint`, `uint8`–`uint64` | `0` | `var x uint` → `0` |
| `float32`, `float64` | `0.0` | `var x float64` → `0.0` |
| `bool` | `false` | `var x bool` → `false` |
| `string` | `""` | `var x string` → `""` |
| `pointer`, `slice`, `map`, `channel`, `interface`, `function` | `nil` | `var x *int` → `nil` |
| `struct` | All fields zero | `var x MyStruct` → each field is zero |

### Why Zero Values Matter

- **No undefined behavior** — In C/C++, uninitialized variables hold garbage. In Go, you always get a predictable zero value.
- **Ready to use** — `var count int` means `count` is 0; you can use it immediately (e.g., `count++`).
- **Useful defaults** — `var sb strings.Builder` is ready to use without initialization.
- **Detect unset values** — `if name == ""` checks whether a string was set.

### Complete Example: Zero Values

```go
package main

import "fmt"

func main() {
    var intVar int
    var boolVar bool
    var stringVar string
    var sliceVar []int

    fmt.Println(intVar, boolVar, stringVar, sliceVar) // 0 false  []

    var counter int
    counter++
    counter++
    fmt.Println(counter) // 2

    var middleName string
    if middleName == "" {
        fmt.Println("Middle name: (not provided)") // Middle name: (not provided)
    }
}
```

**Output:**
```
0 false  []
2
Middle name: (not provided)
```

---

## 🔒 Constants

### What is a Constant?

A constant is a value that **cannot change** after declaration.

### Why Use Constants?

- **Values that never change** — e.g., `pi`, `daysInWeek`
- **Prevent accidental modification** — `maxRetries = 5` causes a compile error
- **Self-documenting code** — `statusActive` is clearer than `"active"`
- **Compile-time evaluation** — `minutesInDay = hoursInDay * 60` is computed at compile time

### Declaring Constants

```go
const pi = 3.14159
const maxSize int = 100

const (
    statusPending  = "pending"
    statusActive   = "active"
    statusComplete = "complete"
)

const (
    KB = 1024
    MB = KB * 1024
    GB = MB * 1024
)
```

### iota: The Constant Generator

```go
const (
    Sunday = iota    // 0
    Monday           // 1
    Tuesday          // 2
)
fmt.Println(Sunday, Monday, Tuesday) // Output: 0 1 2
```

```go
const (
    _  = iota             // 0 (ignored)
    KB = 1 << (10 * iota)  // 1024
    MB                     // 1048576
    GB                     // 1073741824
)
fmt.Println(KB, MB, GB) // Output: 1024 1048576 1073741824
```

### Complete Example: Constants

```go
package main

import "fmt"

const AppName = "My Application"
const MaxRetries = 3

const (
    Sunday = iota
    Monday
    Tuesday
)

func main() {
    fmt.Println(AppName, MaxRetries) // My Application 3
    fmt.Println(Sunday, Monday, Tuesday) // 0 1 2

    retries := MaxRetries
    retries = 5
    fmt.Println(retries) // 5 (variable can change; constant cannot)
}
```

**Output:**
```
My Application 3
0 1 2
5
```

---

## 🔍 Visibility: Exported vs Unexported

### The Capital Letter Rule

| Style | Visibility | Access |
|-------|-------------|--------|
| **Capital** letter | Exported | Other packages can access |
| **lowercase** letter | Unexported | Only this package can access |

Examples: `MaxUsers` (exported), `minAge` (unexported), `ValidateUser()` (exported), `hashPassword()` (unexported).

### Complete Example: Visibility

```go
package main

import "fmt"

var MaxConnections = 100
var defaultTimeout = 30

type User struct {
    Name string
    age  int
}

func main() {
    fmt.Println(MaxConnections, defaultTimeout) // 100 30

    user := User{Name: "Alice", age: 25}
    fmt.Println(user.Name, user.age) // Alice 25
}
```

**Output:**
```
100 30
Alice 25
```

---

## ⚠️ Variable Shadowing

### What is Shadowing?

Shadowing occurs when a variable in an inner scope has the same name as one in an outer scope, "hiding" the outer variable.

```go
message := "original"
if true {
    message := "inside if"  // Shadows outer message!
    fmt.Println(message)   // Output: inside if
}
fmt.Println(message) // Output: original (unchanged!)
```

**Correct approach** — use `=` to reassign, not `:=`:

```go
result := "original"
if true {
    result = "modified"  // Uses outer variable
}
fmt.Println(result) // Output: modified
```

### Common Bug: Shadowing `err`

```go
var err error
if true {
    _, err := someOperation()  // Shadows outer err!
    fmt.Println(err)           // Has value here
}
fmt.Println(err) // Still nil! Bug!
```

**Fix:** Use `=` or a different variable name.

---

## 🆚 Java Comparison: Variables & Constants

| Aspect | Java | Go |
|--------|------|-----|
| Variables | `int age = 25;` | `age := 25` |
| Constants | `final int MAX = 100;` | `const Max = 100` |
| Visibility | `public String name;` | `Name string` (Capital) |
| Visibility | `private int age;` | `age int` (lowercase) |
| Default values | `int x;` → 0 | `var x int` → 0 |
| Default values | `String s;` → null | `var s string` → `""` |
| Type inference | `var name = "Alice";` (Java 10+) | `name := "Alice"` |

---

## 🎯 Key Takeaways

1. **`:=`** = Short declaration (most common inside functions)
2. **`var`** = Explicit declaration (package level or zero value)
3. **Zero values** = Every type has a default value (no undefined!)
4. **`const`** = Immutable values, evaluated at compile time
5. **`iota`** = Auto-incrementing constant generator
6. **Capital letter** = Exported (public)
7. **lowercase** = Unexported (private)
8. **Shadowing** = Be careful with `:=` in inner scopes!

---

## ➡️ Next Steps

You now understand variables and constants. Let's explore Go's data types in depth.

**Next Topic:** [07 - Data Types - Basics](./07-data-types-basics.md)
