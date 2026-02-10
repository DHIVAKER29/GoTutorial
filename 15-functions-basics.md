# 15 - Functions - Basics

> Understanding functions in Go: declaration, parameters, and return values.

---

## 📌 What You'll Learn

- Function declaration syntax
- Parameters and arguments
- Return values (single and multiple)
- Named return values
- The blank identifier `_`
- Go's unique patterns compared to Java
- Sample programs for each concept

---

## 🤔 What is a Function?

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  FUNCTION = A VENDING MACHINE                                   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     COFFEE MACHINE                       │   │
│  │                                                         │   │
│  │   INPUT (Parameters):                                   │   │
│  │   • Money: $2.00                                        │   │
│  │   • Selection: "Latte"                                  │   │
│  │                                                         │   │
│  │   PROCESSING (Function Body):                           │   │
│  │   • Check money                                         │   │
│  │   • Check selection                                     │   │
│  │   • Make coffee                                         │   │
│  │                                                         │   │
│  │   OUTPUT (Return Value):                                │   │
│  │   • Coffee cup ☕                                       │   │
│  │   • Change: $0.50                                       │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  In code:                                                       │
│  func getCoffee(money float64, selection string) (Coffee, float64)│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Function Declaration Syntax

### Basic Syntax

```go
func functionName(parameter1 type1, parameter2 type2) returnType {
    // function body
    return value
}
```

### Anatomy of a Function

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  func add(a int, b int) int {                                   │
│   │    │   │       │      │                                     │
│   │    │   │       │      └── Return type                       │
│   │    │   │       └── Parameter 2: name and type               │
│   │    │   └── Parameter 1: name and type                       │
│   │    └── Function name (lowercase = unexported)               │
│   └── func keyword                                              │
│                                                                 │
│      return a + b                                               │
│       │     │                                                   │
│       │     └── Return value                                    │
│       └── return keyword                                        │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Sample Program: Basic Functions

```go
// functions_basic.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              FUNCTIONS IN GO                              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Calling functions
    fmt.Println("\n📊 Calling Functions:")
    
    // No parameters, no return
    sayHello()
    
    // With parameters
    greet("Alice")
    
    // With return value
    sum := add(5, 3)
    fmt.Printf("   add(5, 3) = %d\n", sum)
    
    // With multiple parameters of same type
    product := multiply(4, 5)
    fmt.Printf("   multiply(4, 5) = %d\n", product)
    
    // Multiple return values
    quotient, remainder := divide(17, 5)
    fmt.Printf("   divide(17, 5) = %d remainder %d\n", quotient, remainder)
    
    // Named return values
    area, perimeter := rectangleInfo(5, 3)
    fmt.Printf("   rectangleInfo(5, 3): area=%d, perimeter=%d\n", area, perimeter)
    
    // Ignoring return values with _
    onlyArea, _ := rectangleInfo(10, 5)
    fmt.Printf("   Area only (ignoring perimeter): %d\n", onlyArea)
}

// No parameters, no return value
func sayHello() {
    fmt.Println("   Hello from sayHello()!")
}

// With parameter
func greet(name string) {
    fmt.Printf("   Hello, %s!\n", name)
}

// With return value
func add(a int, b int) int {
    return a + b
}

// Same type parameters - can be shortened
func multiply(a, b int) int {  // Both are int
    return a * b
}

// Multiple return values
func divide(dividend, divisor int) (int, int) {
    quotient := dividend / divisor
    remainder := dividend % divisor
    return quotient, remainder
}

// Named return values
func rectangleInfo(width, height int) (area int, perimeter int) {
    area = width * height        // Assign to named return
    perimeter = 2 * (width + height)
    return  // "Naked return" - returns named values
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              FUNCTIONS IN GO                              ║
╚══════════════════════════════════════════════════════════╝

📊 Calling Functions:
   Hello from sayHello()!
   Hello, Alice!
   add(5, 3) = 8
   multiply(4, 5) = 20
   divide(17, 5) = 3 remainder 2
   rectangleInfo(5, 3): area=15, perimeter=16
   Area only (ignoring perimeter): 50
```

---

## 🔀 Multiple Return Values

### Why Go Has Multiple Returns

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  THE PROBLEM IN OTHER LANGUAGES:                                │
│                                                                 │
│  Java: Need wrapper class or out parameters                     │
│  ────────────────────────────────────────                       │
│  class Result {                                                 │
│      int quotient;                                              │
│      int remainder;                                             │
│  }                                                              │
│  Result divide(int a, int b) { ... }                            │
│                                                                 │
│  Or use exceptions for errors:                                  │
│  int divide(int a, int b) throws ArithmeticException            │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  GO'S SOLUTION: Multiple return values                          │
│  ─────────────────────────────────────                          │
│  func divide(a, b int) (int, int) {                             │
│      return a / b, a % b                                        │
│  }                                                              │
│                                                                 │
│  quotient, remainder := divide(17, 5)                           │
│                                                                 │
│  For errors:                                                    │
│  result, err := someOperation()                                 │
│  if err != nil {                                                │
│      // handle error                                            │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Multiple Returns

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
    
    // Two values
    fmt.Println("\n📊 Two Return Values:")
    min, max := minMax(5, 3, 8, 1, 9, 2)
    fmt.Printf("   minMax(5,3,8,1,9,2): min=%d, max=%d\n", min, max)
    
    // Error as second return
    fmt.Println("\n📊 Error as Second Return (Go Pattern!):")
    
    result, err := safeDivide(10, 2)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   safeDivide(10, 2) = %.2f\n", result)
    }
    
    result, err = safeDivide(10, 0)
    if err != nil {
        fmt.Printf("   safeDivide(10, 0): Error - %v\n", err)
    }
    
    // Parsing with error
    fmt.Println("\n📊 Parsing with Error:")
    parseAndPrint("42")
    parseAndPrint("not-a-number")
    
    // Multiple values for complex operations
    fmt.Println("\n📊 Complex Operations:")
    user, found, err := findUser(1)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else if !found {
        fmt.Println("   User not found")
    } else {
        fmt.Printf("   Found user: %s\n", user)
    }
    
    user, found, err = findUser(99)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else if !found {
        fmt.Println("   User 99 not found")
    }
}

// Returns multiple values of same type
func minMax(nums ...int) (int, int) {
    if len(nums) == 0 {
        return 0, 0
    }
    
    min, max := nums[0], nums[0]
    for _, n := range nums[1:] {
        if n < min {
            min = n
        }
        if n > max {
            max = n
        }
    }
    return min, max
}

// Standard Go pattern: (result, error)
func safeDivide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

func parseAndPrint(s string) {
    num, err := strconv.Atoi(s)
    if err != nil {
        fmt.Printf("   Parse %q: Error - %v\n", s, err)
        return
    }
    fmt.Printf("   Parse %q: Success - %d\n", s, num)
}

// Three return values
func findUser(id int) (string, bool, error) {
    users := map[int]string{
        1: "Alice",
        2: "Bob",
    }
    
    if id < 0 {
        return "", false, errors.New("invalid user ID")
    }
    
    name, found := users[id]
    return name, found, nil
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           MULTIPLE RETURN VALUES                          ║
╚══════════════════════════════════════════════════════════╝

📊 Two Return Values:
   minMax(5,3,8,1,9,2): min=1, max=9

📊 Error as Second Return (Go Pattern!):
   safeDivide(10, 2) = 5.00
   safeDivide(10, 0): Error - division by zero

📊 Parsing with Error:
   Parse "42": Success - 42
   Parse "not-a-number": Error - strconv.Atoi: parsing "not-a-number": invalid syntax

📊 Complex Operations:
   Found user: Alice
   User 99 not found
```

---

## 🏷️ Named Return Values

### What Are They?

```go
// Named return values
func divide(a, b int) (quotient int, remainder int) {
    quotient = a / b      // Assign directly to named return
    remainder = a % b
    return                // "Naked return" - returns named values
}

// Or with explicit return
func divide2(a, b int) (quotient int, remainder int) {
    quotient = a / b
    remainder = a % b
    return quotient, remainder  // Explicit return also works
}
```

### When to Use Named Returns

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  USE NAMED RETURNS WHEN:                                        │
│                                                                 │
│  1. DOCUMENTATION - Names clarify what's returned               │
│     func getCoords() (latitude, longitude float64)              │
│                                                                 │
│  2. DEFER with modifications                                    │
│     func doSomething() (err error) {                            │
│         defer func() {                                          │
│             if err != nil {                                     │
│                 // Can modify err here!                         │
│             }                                                   │
│         }()                                                     │
│     }                                                           │
│                                                                 │
│  AVOID NAMED RETURNS WHEN:                                      │
│                                                                 │
│  1. Short functions - adds no clarity                           │
│  2. When you might shadow them accidentally                     │
│  3. When naked return is far from function signature            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ➖ The Blank Identifier `_`

### Why It Exists

```go
// When you don't need a return value
result, _ := functionReturningTwo()  // Ignore second value

// In range loops
for _, value := range slice {  // Ignore index
    fmt.Println(value)
}

for index := range slice {  // Ignore value (even simpler!)
    fmt.Println(index)
}
```

### Sample Program: Blank Identifier

```go
// blank_identifier.go
package main

import (
    "fmt"
    "os"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           THE BLANK IDENTIFIER _                          ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Ignoring return values
    fmt.Println("\n📊 Ignoring Return Values:")
    
    quot, rem := divide(17, 5)
    fmt.Printf("   Both values: quotient=%d, remainder=%d\n", quot, rem)
    
    quotOnly, _ := divide(17, 5)  // Ignore remainder
    fmt.Printf("   Quotient only: %d\n", quotOnly)
    
    _, remOnly := divide(17, 5)   // Ignore quotient
    fmt.Printf("   Remainder only: %d\n", remOnly)
    
    // Ignoring index in range
    fmt.Println("\n📊 In Range Loops:")
    fruits := []string{"apple", "banana", "cherry"}
    
    fmt.Println("   Values only (ignore index):")
    for _, fruit := range fruits {
        fmt.Printf("     %s\n", fruit)
    }
    
    // Check if operation succeeded without using result
    fmt.Println("\n📊 Checking Success Only:")
    _, err := os.Open("/nonexistent/file")
    if err != nil {
        fmt.Printf("   File check failed: %v\n", err)
    }
    
    // Import for side effects only
    fmt.Println("\n📊 Import for Side Effects:")
    fmt.Println("   import _ \"image/png\"  // Registers PNG decoder")
    fmt.Println("   The _ means: import for side effects, don't use directly")
}

func divide(a, b int) (int, int) {
    return a / b, a % b
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           THE BLANK IDENTIFIER _                          ║
╚══════════════════════════════════════════════════════════╝

📊 Ignoring Return Values:
   Both values: quotient=3, remainder=2
   Quotient only: 3
   Remainder only: 2

📊 In Range Loops:
   Values only (ignore index):
     apple
     banana
     cherry

📊 Checking Success Only:
   File check failed: open /nonexistent/file: no such file or directory

📊 Import for Side Effects:
   import _ "image/png"  // Registers PNG decoder
   The _ means: import for side effects, don't use directly
```

---

## 🏭 Production Patterns

### Sample Program: Real-World Function Patterns

```go
// functions_production.go
package main

import (
    "errors"
    "fmt"
    "time"
)

// User represents a user in the system
type User struct {
    ID        int
    Name      string
    Email     string
    CreatedAt time.Time
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION FUNCTION PATTERNS                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: Constructor function
    fmt.Println("\n📊 Pattern 1: Constructor Function")
    user := NewUser(1, "Alice", "alice@example.com")
    fmt.Printf("   Created: %+v\n", user)
    
    // Pattern 2: Validation function
    fmt.Println("\n📊 Pattern 2: Validation")
    if err := ValidateUser(user); err != nil {
        fmt.Printf("   Invalid: %v\n", err)
    } else {
        fmt.Println("   User is valid ✅")
    }
    
    invalidUser := NewUser(0, "", "")
    if err := ValidateUser(invalidUser); err != nil {
        fmt.Printf("   Invalid user: %v\n", err)
    }
    
    // Pattern 3: Option struct for many parameters
    fmt.Println("\n📊 Pattern 3: Options Pattern")
    config := ServerConfig{
        Host:    "localhost",
        Port:    8080,
        Timeout: 30,
    }
    StartServer(config)
    
    // Pattern 4: Functional options
    fmt.Println("\n📊 Pattern 4: Functional Options")
    server := NewServer(
        WithHost("0.0.0.0"),
        WithPort(9000),
        WithTimeout(60),
    )
    fmt.Printf("   Server: %+v\n", server)
}

// Pattern 1: Constructor function (like Java's constructor)
func NewUser(id int, name, email string) *User {
    return &User{
        ID:        id,
        Name:      name,
        Email:     email,
        CreatedAt: time.Now(),
    }
}

// Pattern 2: Validation function
func ValidateUser(u *User) error {
    if u.ID <= 0 {
        return errors.New("invalid user ID")
    }
    if u.Name == "" {
        return errors.New("name is required")
    }
    if u.Email == "" {
        return errors.New("email is required")
    }
    return nil
}

// Pattern 3: Config struct for many parameters
type ServerConfig struct {
    Host    string
    Port    int
    Timeout int
}

func StartServer(config ServerConfig) {
    fmt.Printf("   Starting server on %s:%d (timeout: %ds)\n",
        config.Host, config.Port, config.Timeout)
}

// Pattern 4: Functional options pattern
type Server struct {
    host    string
    port    int
    timeout int
}

type ServerOption func(*Server)

func WithHost(host string) ServerOption {
    return func(s *Server) {
        s.host = host
    }
}

func WithPort(port int) ServerOption {
    return func(s *Server) {
        s.port = port
    }
}

func WithTimeout(timeout int) ServerOption {
    return func(s *Server) {
        s.timeout = timeout
    }
}

func NewServer(opts ...ServerOption) *Server {
    // Defaults
    s := &Server{
        host:    "localhost",
        port:    8080,
        timeout: 30,
    }
    
    // Apply options
    for _, opt := range opts {
        opt(s)
    }
    
    return s
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           PRODUCTION FUNCTION PATTERNS                    ║
╚══════════════════════════════════════════════════════════╝

📊 Pattern 1: Constructor Function
   Created: &{ID:1 Name:Alice Email:alice@example.com CreatedAt:2026-02-10 ...}

📊 Pattern 2: Validation
   User is valid ✅
   Invalid user: invalid user ID

📊 Pattern 3: Options Pattern
   Starting server on localhost:8080 (timeout: 30s)

📊 Pattern 4: Functional Options
   Server: &{host:0.0.0.0 port:9000 timeout:60}
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  // Method in class                // Function (can be anywhere)│
│  public int add(int a, int b) {    func add(a, b int) int {     │
│      return a + b;                     return a + b             │
│  }                                 }                            │
│                                                                 │
│  // Access modifiers               // Capital letter = exported │
│  public void doSomething()         func DoSomething()           │
│  private void helper()             func helper()                │
│                                                                 │
│  // Single return value            // Multiple returns!         │
│  public Result divide(int a, b)    func divide(a, b int) (q, r int)│
│                                                                 │
│  // Throw exceptions               // Return error              │
│  throws ArithmeticException        returns (result, error)      │
│                                                                 │
│  // Overloading                    // No overloading!           │
│  add(int, int)                     // Use different names or    │
│  add(int, int, int)                // variadic functions        │
│  add(double, double)                                            │
│                                                                 │
│  // Constructor                    // Factory function          │
│  public User(String name) { }      func NewUser(name string) *User│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **`func`** keyword for all functions
2. **Type comes after** parameter name: `func add(a int)`
3. **Same type parameters** can be shortened: `func add(a, b int)`
4. **Multiple return values** are idiomatic: `(result, error)`
5. **Named returns** document what's returned
6. **Blank identifier `_`** ignores unwanted values
7. **No overloading** - use different names or variadic functions
8. **No exceptions** - return errors instead

---

## ➡️ Next Steps

You now understand basic functions. Let's explore multiple return values in depth.

**Next Topic:** [16 - Multiple Return Values](./16-multiple-returns.md)

