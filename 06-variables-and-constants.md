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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  THE PROBLEM                                                    │
│                                                                 │
│  You want to calculate a person's age in days.                  │
│  Without variables:                                             │
│                                                                 │
│  fmt.Println(25 * 365)  // What is 25? What is 365?             │
│                                                                 │
│  • What if age changes?                                         │
│  • What if we need age multiple times?                          │
│  • What does 25 mean? 365?                                      │
│                                                                 │
│  THE SOLUTION: Variables                                        │
│                                                                 │
│  age := 25                                                      │
│  daysPerYear := 365                                             │
│  ageInDays := age * daysPerYear                                 │
│  fmt.Println(ageInDays)  // Clear and reusable!                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  VARIABLE = LABELED BOX                                         │
│                                                                 │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐           │
│  │    age      │   │   name      │   │   price     │           │
│  │  ┌───────┐  │   │  ┌───────┐  │   │  ┌───────┐  │           │
│  │  │  25   │  │   │  │"Alice"│  │   │  │ 99.99 │  │           │
│  │  └───────┘  │   │  └───────┘  │   │  └───────┘  │           │
│  │    int      │   │   string    │   │   float64   │           │
│  └─────────────┘   └─────────────┘   └─────────────┘           │
│        ↑                 ↑                 ↑                    │
│     Name of           Type of           Current                 │
│     the box           content           value                   │
│                                                                 │
│  You can:                                                       │
│  • Put something in (assign value)                              │
│  • Look at what's inside (read value)                           │
│  • Replace contents (reassign)                                  │
│  • But only same type fits! (type safety)                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Declaring Variables

### Method 1: var with Type (Explicit)

```go
var name string
var age int
var price float64
var isActive bool
```

The variable gets a **zero value** (explained later).

### Method 2: var with Initialization

```go
var name string = "Alice"
var age int = 25
var price float64 = 99.99
var isActive bool = true
```

### Method 3: var with Type Inference

```go
var name = "Alice"      // Go infers: string
var age = 25            // Go infers: int
var price = 99.99       // Go infers: float64
var isActive = true     // Go infers: bool
```

### Method 4: Short Declaration `:=` (Most Common!)

```go
name := "Alice"         // Declare AND initialize
age := 25
price := 99.99
isActive := true
```

### Grouped Declaration

```go
var (
    name     string  = "Alice"
    age      int     = 25
    price    float64 = 99.99
    isActive bool    = true
)

// Or with inference
var (
    name     = "Alice"
    age      = 25
    price    = 99.99
    isActive = true
)
```

### When to Use Which?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  WHICH DECLARATION TO USE?                                      │
│                                                                 │
│  `:=` (Short Declaration)                                       │
│  ─────────────────────────                                      │
│  ✅ Inside functions (most common)                              │
│  ✅ When you have an initial value                              │
│  ❌ Cannot use at package level                                 │
│                                                                 │
│  `var` with Type                                                │
│  ─────────────────                                              │
│  ✅ Package-level variables                                     │
│  ✅ When you want zero value                                    │
│  ✅ When type isn't obvious from value                          │
│                                                                 │
│  `var` with Inference                                           │
│  ─────────────────────                                          │
│  ✅ When the type is obvious                                    │
│  ✅ Package level with initialization                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Sample Program: All Variable Declarations

```go
// variables_demo.go
package main

import "fmt"

// Package-level variables (must use var, not :=)
var appName = "My App"
var version string = "1.0.0"
var maxUsers int  // Zero value: 0

// Grouped package-level variables
var (
    debug     = false
    logLevel  = "info"
    timeout   = 30
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           VARIABLE DECLARATIONS IN GO                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Method 1: var with type (gets zero value)
    var count int        // 0
    var message string   // ""
    var active bool      // false
    
    fmt.Println("\n📦 Method 1: var with type (zero values)")
    fmt.Printf("  count = %d, message = %q, active = %t\n", count, message, active)
    
    // Method 2: var with initialization
    var name string = "Alice"
    var age int = 25
    
    fmt.Println("\n📦 Method 2: var with initialization")
    fmt.Printf("  name = %s, age = %d\n", name, age)
    
    // Method 3: var with type inference
    var city = "New York"  // Go knows it's string
    var temp = 72.5        // Go knows it's float64
    
    fmt.Println("\n📦 Method 3: var with type inference")
    fmt.Printf("  city = %s (type: %T)\n", city, city)
    fmt.Printf("  temp = %.1f (type: %T)\n", temp, temp)
    
    // Method 4: Short declaration (most common!)
    country := "USA"
    population := 331000000
    isLarge := true
    
    fmt.Println("\n📦 Method 4: Short declaration :=")
    fmt.Printf("  country = %s\n", country)
    fmt.Printf("  population = %d\n", population)
    fmt.Printf("  isLarge = %t\n", isLarge)
    
    // Multiple variables at once
    x, y, z := 1, 2, 3
    firstName, lastName := "John", "Doe"
    
    fmt.Println("\n📦 Multiple variables at once")
    fmt.Printf("  x, y, z = %d, %d, %d\n", x, y, z)
    fmt.Printf("  Name: %s %s\n", firstName, lastName)
    
    // Reassignment
    count = 10           // = for reassignment (already declared)
    count = count + 5    // Can use current value
    count += 5           // Shorthand
    
    fmt.Println("\n📦 Reassignment")
    fmt.Printf("  count after modifications = %d\n", count)
    
    // Package-level variables
    fmt.Println("\n📦 Package-level variables")
    fmt.Printf("  appName = %s\n", appName)
    fmt.Printf("  version = %s\n", version)
    fmt.Printf("  maxUsers = %d (zero value)\n", maxUsers)
    fmt.Printf("  debug = %t\n", debug)
}
```

**Run it:**
```bash
go run variables_demo.go
```

---

## 🔢 Zero Values

### What is a Zero Value?

> When you declare a variable without initializing it, Go automatically assigns a **zero value** based on its type.

### Zero Values Table

| Type | Zero Value | Example |
|------|------------|---------|
| `int`, `int8`, `int16`, `int32`, `int64` | `0` | `var x int` → `0` |
| `uint`, `uint8`, `uint16`, `uint32`, `uint64` | `0` | `var x uint` → `0` |
| `float32`, `float64` | `0.0` | `var x float64` → `0.0` |
| `bool` | `false` | `var x bool` → `false` |
| `string` | `""` (empty string) | `var x string` → `""` |
| `pointer` | `nil` | `var x *int` → `nil` |
| `slice` | `nil` | `var x []int` → `nil` |
| `map` | `nil` | `var x map[string]int` → `nil` |
| `channel` | `nil` | `var x chan int` → `nil` |
| `interface` | `nil` | `var x interface{}` → `nil` |
| `function` | `nil` | `var x func()` → `nil` |
| `struct` | All fields zero | `var x MyStruct` → each field is zero |

### Why Zero Values Matter

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  WHY ZERO VALUES ARE IMPORTANT                                  │
│                                                                 │
│  1. NO UNDEFINED BEHAVIOR                                       │
│     In C/C++: uninitialized = garbage value (dangerous!)        │
│     In Go: always predictable zero value                        │
│                                                                 │
│  2. READY TO USE                                                │
│     var count int  // count is 0, can use immediately           │
│     count++        // Works! count is now 1                     │
│                                                                 │
│  3. ZERO VALUES ARE USEFUL                                      │
│     var sb strings.Builder  // Ready to use!                    │
│     sb.WriteString("Hello") // Works without initialization     │
│                                                                 │
│  4. DETECT UNSET VALUES                                         │
│     var name string                                             │
│     if name == "" {                                             │
│         fmt.Println("Name not set!")                            │
│     }                                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Zero Values

```go
// zero_values.go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              ZERO VALUES IN GO                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Numeric types
    var intVar int
    var int8Var int8
    var int64Var int64
    var floatVar float64
    
    fmt.Println("\n📊 Numeric Zero Values:")
    fmt.Printf("  int     = %d\n", intVar)
    fmt.Printf("  int8    = %d\n", int8Var)
    fmt.Printf("  int64   = %d\n", int64Var)
    fmt.Printf("  float64 = %f\n", floatVar)
    
    // Boolean
    var boolVar bool
    fmt.Println("\n🔘 Boolean Zero Value:")
    fmt.Printf("  bool = %t\n", boolVar)
    
    // String
    var stringVar string
    fmt.Println("\n📝 String Zero Value:")
    fmt.Printf("  string = %q (empty string)\n", stringVar)
    fmt.Printf("  length = %d\n", len(stringVar))
    
    // Pointer
    var ptrVar *int
    fmt.Println("\n👆 Pointer Zero Value:")
    fmt.Printf("  *int = %v (nil)\n", ptrVar)
    
    // Slice
    var sliceVar []int
    fmt.Println("\n📋 Slice Zero Value:")
    fmt.Printf("  []int = %v (nil slice)\n", sliceVar)
    fmt.Printf("  length = %d, capacity = %d\n", len(sliceVar), cap(sliceVar))
    fmt.Printf("  is nil? %t\n", sliceVar == nil)
    
    // Map
    var mapVar map[string]int
    fmt.Println("\n🗺️ Map Zero Value:")
    fmt.Printf("  map[string]int = %v (nil map)\n", mapVar)
    fmt.Printf("  is nil? %t\n", mapVar == nil)
    
    // Struct
    var personVar Person
    fmt.Println("\n👤 Struct Zero Value:")
    fmt.Printf("  Person = %+v\n", personVar)
    fmt.Printf("  Name = %q (empty string)\n", personVar.Name)
    fmt.Printf("  Age = %d\n", personVar.Age)
    
    // Practical use: zero values are useful!
    fmt.Println("\n✅ Why Zero Values Are Useful:")
    
    // Counter starts at 0, ready to use
    var counter int
    counter++
    counter++
    fmt.Printf("  Counter after 2 increments: %d\n", counter)
    
    // Empty string for optional values
    var middleName string
    if middleName == "" {
        fmt.Println("  Middle name: (not provided)")
    }
    
    // Boolean flags default to false
    var debugMode bool
    if !debugMode {
        fmt.Println("  Debug mode: disabled (default)")
    }
}
```

---

## 🔒 Constants

### What is a Constant?

> A constant is a value that **cannot change** after declaration.

### Why Use Constants?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  WHY CONSTANTS?                                                 │
│                                                                 │
│  1. VALUES THAT NEVER CHANGE                                    │
│     const pi = 3.14159                                          │
│     const daysInWeek = 7                                        │
│                                                                 │
│  2. PREVENT ACCIDENTAL MODIFICATION                             │
│     const maxRetries = 3                                        │
│     maxRetries = 5  // ❌ COMPILE ERROR!                        │
│                                                                 │
│  3. SELF-DOCUMENTING CODE                                       │
│     const statusActive = "active"                               │
│     const statusInactive = "inactive"                           │
│     // Better than: if status == "active"                       │
│                                                                 │
│  4. COMPILE-TIME EVALUATION                                     │
│     const hoursInDay = 24                                       │
│     const minutesInDay = hoursInDay * 60  // Calculated at      │
│                                           // compile time!      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Declaring Constants

```go
// Single constant
const pi = 3.14159

// With explicit type
const maxSize int = 100

// Grouped constants
const (
    statusPending  = "pending"
    statusActive   = "active"
    statusComplete = "complete"
)

// Computed constants (compile-time only!)
const (
    KB = 1024
    MB = KB * 1024
    GB = MB * 1024
)
```

### iota: The Constant Generator

```go
// iota starts at 0 and increments
const (
    Sunday = iota    // 0
    Monday           // 1
    Tuesday          // 2
    Wednesday        // 3
    Thursday         // 4
    Friday           // 5
    Saturday         // 6
)

// Skip values
const (
    _ = iota         // 0 (ignored)
    One              // 1
    Two              // 2
    _                // 3 (ignored)
    Four             // 4
)

// Expressions with iota
const (
    _  = iota             // 0 (ignored)
    KB = 1 << (10 * iota) // 1 << 10 = 1024
    MB                    // 1 << 20 = 1048576
    GB                    // 1 << 30 = 1073741824
)
```

### Sample Program: Constants

```go
// constants_demo.go
package main

import "fmt"

// Package-level constants
const AppName = "My Application"
const Version = "1.0.0"

// Grouped constants
const (
    MaxRetries    = 3
    TimeoutSecs   = 30
    MaxUploadSize = 10 * 1024 * 1024 // 10 MB
)

// Typed constants (for type safety)
type Status string

const (
    StatusPending  Status = "pending"
    StatusActive   Status = "active"
    StatusComplete Status = "complete"
)

// Days of week with iota
const (
    Sunday = iota // 0
    Monday        // 1
    Tuesday       // 2
    Wednesday     // 3
    Thursday      // 4
    Friday        // 5
    Saturday      // 6
)

// File sizes with iota
const (
    _  = iota             // ignore first value
    KB = 1 << (10 * iota) // 1 << 10 = 1024
    MB                    // 1 << 20 = 1048576
    GB                    // 1 << 30 = 1073741824
    TB                    // 1 << 40 = 1099511627776
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              CONSTANTS IN GO                              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic constants
    fmt.Println("\n📌 Basic Constants:")
    fmt.Printf("  AppName = %s\n", AppName)
    fmt.Printf("  Version = %s\n", Version)
    
    // Grouped constants
    fmt.Println("\n📌 Configuration Constants:")
    fmt.Printf("  MaxRetries    = %d\n", MaxRetries)
    fmt.Printf("  TimeoutSecs   = %d seconds\n", TimeoutSecs)
    fmt.Printf("  MaxUploadSize = %d bytes (%.0f MB)\n", MaxUploadSize, float64(MaxUploadSize)/float64(MB))
    
    // Typed constants
    fmt.Println("\n📌 Typed Constants (Status):")
    currentStatus := StatusActive
    fmt.Printf("  Current status: %s\n", currentStatus)
    
    // This provides type safety:
    // currentStatus = "invalid" // ❌ Would be compile error!
    
    // iota for enums
    fmt.Println("\n📌 Days of Week (iota):")
    fmt.Printf("  Sunday    = %d\n", Sunday)
    fmt.Printf("  Monday    = %d\n", Monday)
    fmt.Printf("  Tuesday   = %d\n", Tuesday)
    fmt.Printf("  Wednesday = %d\n", Wednesday)
    fmt.Printf("  Thursday  = %d\n", Thursday)
    fmt.Printf("  Friday    = %d\n", Friday)
    fmt.Printf("  Saturday  = %d\n", Saturday)
    
    // iota for sizes
    fmt.Println("\n📌 File Sizes (iota with expressions):")
    fmt.Printf("  KB = %d bytes\n", KB)
    fmt.Printf("  MB = %d bytes\n", MB)
    fmt.Printf("  GB = %d bytes\n", GB)
    fmt.Printf("  TB = %d bytes\n", TB)
    
    // Computed at compile time
    const SecondsInDay = 24 * 60 * 60
    const SecondsInWeek = SecondsInDay * 7
    
    fmt.Println("\n📌 Compile-time Computation:")
    fmt.Printf("  Seconds in day  = %d\n", SecondsInDay)
    fmt.Printf("  Seconds in week = %d\n", SecondsInWeek)
    
    // Constants vs Variables demonstration
    fmt.Println("\n⚠️ Constants Cannot Be Changed:")
    fmt.Println("  // const MaxRetries = 3")
    fmt.Println("  // MaxRetries = 5  ← COMPILE ERROR!")
    
    retries := MaxRetries // Copy to variable
    retries = 5           // Variable can change
    fmt.Printf("  Variable retries = %d (changed from constant)\n", retries)
}
```

---

## 🔍 Visibility: Exported vs Unexported

### The Capital Letter Rule

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO'S VISIBILITY RULE                                           │
│                                                                 │
│  CAPITAL letter   = EXPORTED   = Other packages CAN access      │
│  lowercase letter = UNEXPORTED = Only this package can access   │
│                                                                 │
│  // In package "user"                                           │
│                                                                 │
│  var MaxUsers = 100        // Exported (Capital M)              │
│  var minAge = 18           // Unexported (lowercase m)          │
│                                                                 │
│  const APIKey = "..."      // Exported                          │
│  const secretKey = "..."   // Unexported                        │
│                                                                 │
│  func ValidateUser() {}    // Exported                          │
│  func hashPassword() {}    // Unexported                        │
│                                                                 │
│  type User struct {}       // Exported type                     │
│  type session struct {}    // Unexported type                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Visibility

```go
// visibility_demo.go
package main

import "fmt"

// Exported (would be visible to other packages)
var MaxConnections = 100
const AppVersion = "2.0"

// Unexported (only visible in this package)
var defaultTimeout = 30
const internalSecret = "shhh"

// Exported type with mixed field visibility
type User struct {
    Name  string // Exported field
    Email string // Exported field
    age   int    // Unexported field (other packages can't access)
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              VISIBILITY IN GO                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📖 The Rule:")
    fmt.Println("  CAPITAL letter   → Exported (public)")
    fmt.Println("  lowercase letter → Unexported (private)")
    
    fmt.Println("\n📦 Variables:")
    fmt.Printf("  MaxConnections = %d (Exported - Capital M)\n", MaxConnections)
    fmt.Printf("  defaultTimeout = %d (Unexported - lowercase d)\n", defaultTimeout)
    
    fmt.Println("\n📦 Constants:")
    fmt.Printf("  AppVersion     = %s (Exported - Capital A)\n", AppVersion)
    fmt.Printf("  internalSecret = %s (Unexported - lowercase i)\n", internalSecret)
    
    fmt.Println("\n📦 Struct Fields:")
    user := User{
        Name:  "Alice",
        Email: "alice@example.com",
        age:   25, // Can set because we're in same package
    }
    fmt.Printf("  User.Name  = %s (Exported)\n", user.Name)
    fmt.Printf("  User.Email = %s (Exported)\n", user.Email)
    fmt.Printf("  User.age   = %d (Unexported - only accessible here)\n", user.age)
    
    fmt.Println("\n🆚 Java Comparison:")
    fmt.Println("  Java:   public String name;     →  Go: Name string")
    fmt.Println("  Java:   private int age;        →  Go: age int")
    fmt.Println("  Java:   public class User {}    →  Go: type User struct {}")
    fmt.Println("  Java:   private class Helper {} →  Go: type helper struct {}")
}
```

---

## ⚠️ Variable Shadowing

### What is Shadowing?

> Shadowing occurs when a variable in an inner scope has the same name as a variable in an outer scope, "hiding" the outer variable.

### Sample Program: Shadowing Dangers

```go
// shadowing_demo.go
package main

import "fmt"

var globalCount = 100 // Package level

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              VARIABLE SHADOWING                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Printf("\n📦 Global globalCount = %d\n", globalCount)
    
    // Shadowing the global variable
    globalCount := 50 // Creates NEW local variable, shadows global!
    fmt.Printf("📦 After shadowing, globalCount = %d\n", globalCount)
    
    // The global is still unchanged!
    fmt.Printf("📦 But global is still: %d (use package.variable to check)\n", 100)
    
    // Shadowing in if statement
    fmt.Println("\n⚠️ Dangerous shadowing in if:")
    
    message := "original"
    fmt.Printf("  Before if: message = %q\n", message)
    
    if true {
        message := "inside if" // SHADOWS outer message!
        fmt.Printf("  Inside if: message = %q\n", message)
    }
    
    fmt.Printf("  After if:  message = %q (unchanged!)\n", message)
    
    // How to avoid shadowing in if
    fmt.Println("\n✅ Correct approach (no shadowing):")
    
    result := "original"
    fmt.Printf("  Before if: result = %q\n", result)
    
    if true {
        result = "modified" // No := so uses outer variable
    }
    
    fmt.Printf("  After if:  result = %q (changed!)\n", result)
    
    // Common bug with err
    fmt.Println("\n🐛 Common bug: shadowing err")
    demonstrateBuggyErrorHandling()
}

func demonstrateBuggyErrorHandling() {
    var err error
    fmt.Printf("  Initial err: %v\n", err)
    
    if true {
        _, err := someOperation() // SHADOWS outer err!
        fmt.Printf("  Inside if, err: %v\n", err)
    }
    
    fmt.Printf("  After if, err: %v (still nil! Bug!)\n", err)
    
    // Correct way
    fmt.Println("\n  ✅ Fixed version:")
    if true {
        var innerErr error
        _, innerErr = someOperation() // Different name, or use = not :=
        err = innerErr
    }
    fmt.Printf("  After fix, err: %v\n", err)
}

func someOperation() (string, error) {
    return "result", fmt.Errorf("some error")
}
```

---

## 🆚 Java Comparison: Variables & Constants

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  VARIABLES:                                                     │
│  int age = 25;                     age := 25                    │
│  String name = "Alice";            name := "Alice"              │
│  private int count;                var count int                │
│                                                                 │
│  CONSTANTS:                                                     │
│  final int MAX = 100;              const Max = 100              │
│  static final String API = "...";  const API = "..."            │
│                                                                 │
│  VISIBILITY:                                                    │
│  public String name;               Name string (Capital)        │
│  private int age;                  age int (lowercase)          │
│  protected float price;            (no equivalent)              │
│                                                                 │
│  DEFAULT VALUES:                                                │
│  int x; // 0                       var x int // 0               │
│  String s; // null                 var s string // ""           │
│  boolean b; // false               var b bool // false          │
│                                                                 │
│  TYPE INFERENCE:                                                │
│  var name = "Alice"; (Java 10+)    name := "Alice"              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

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

