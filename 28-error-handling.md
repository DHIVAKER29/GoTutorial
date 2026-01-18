# 28 - Error Handling Best Practices

> Go's explicit error handling philosophy and production patterns.

---

## 📌 What You'll Learn

- Go's error philosophy (errors as values)
- The error interface
- Creating custom errors
- Error wrapping and unwrapping (Go 1.13+)
- Production error handling patterns
- When to use panic vs error

---

## 🤔 Go's Error Philosophy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA: Exceptions (hidden control flow)                         │
│                                                                 │
│  try {                                                          │
│      result = riskyOperation();                                 │
│      // Exception can fly from ANYWHERE                         │
│  } catch (Exception e) {                                        │
│      // Handle it here, maybe                                   │
│  }                                                              │
│                                                                 │
│  GO: Errors as Values (explicit control flow)                   │
│                                                                 │
│  result, err := riskyOperation()                                │
│  if err != nil {                                                │
│      // Must handle it HERE, now, explicitly                    │
│      return err                                                 │
│  }                                                              │
│  // Success path continues                                      │
│                                                                 │
│  WHY GO CHOSE THIS:                                             │
│  • Errors are explicit - you see them in the code               │
│  • No hidden jumps - control flow is clear                      │
│  • Forces handling - can't ignore (well, shouldn't)             │
│  • Simple - error is just a value, like any other               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 The error Interface

```go
// error_interface.go
package main

import (
    "errors"
    "fmt"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           THE ERROR INTERFACE                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // The error interface is simple:
    // type error interface {
    //     Error() string
    // }
    
    fmt.Println("\n📊 Creating Errors:")
    
    // Method 1: errors.New()
    err1 := errors.New("something went wrong")
    fmt.Printf("   errors.New(): %v\n", err1)
    
    // Method 2: fmt.Errorf()
    name := "file.txt"
    err2 := fmt.Errorf("could not open %s", name)
    fmt.Printf("   fmt.Errorf(): %v\n", err2)
    
    // Method 3: Sentinel errors (package-level)
    fmt.Println("\n📊 Sentinel Errors (predefined):")
    fmt.Println("   var ErrNotFound = errors.New(\"not found\")")
    fmt.Println("   Used with errors.Is() for comparison")
    
    // Using errors
    fmt.Println("\n📊 Standard Pattern:")
    result, err := divide(10, 0)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   Result: %.2f\n", result)
    }
    
    result, err = divide(10, 2)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   Result: %.2f\n", result)
    }
}

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

---

## 🎨 Custom Errors

```go
// custom_errors.go
package main

import (
    "fmt"
)

// Custom error type with additional context
type ValidationError struct {
    Field   string
    Message string
}

// Implement error interface
func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on field '%s': %s", e.Field, e.Message)
}

// Error with status code
type HTTPError struct {
    StatusCode int
    Message    string
}

func (e *HTTPError) Error() string {
    return fmt.Sprintf("HTTP %d: %s", e.StatusCode, e.Message)
}

// Helper method
func (e *HTTPError) IsClientError() bool {
    return e.StatusCode >= 400 && e.StatusCode < 500
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CUSTOM ERRORS                                   ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Custom error
    fmt.Println("\n📊 Custom Validation Error:")
    err := validateUser("", 15)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        
        // Type assertion to access fields
        if vErr, ok := err.(*ValidationError); ok {
            fmt.Printf("   Field: %s\n", vErr.Field)
        }
    }
    
    // HTTP error
    fmt.Println("\n📊 Custom HTTP Error:")
    err = fetchData("invalid-url")
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        
        if httpErr, ok := err.(*HTTPError); ok {
            fmt.Printf("   Is client error: %t\n", httpErr.IsClientError())
        }
    }
}

func validateUser(name string, age int) error {
    if name == "" {
        return &ValidationError{Field: "name", Message: "cannot be empty"}
    }
    if age < 18 {
        return &ValidationError{Field: "age", Message: "must be 18 or older"}
    }
    return nil
}

func fetchData(url string) error {
    // Simulating HTTP error
    return &HTTPError{StatusCode: 404, Message: "Resource not found"}
}
```

---

## 🔗 Error Wrapping (Go 1.13+)

```go
// error_wrapping.go
package main

import (
    "errors"
    "fmt"
    "os"
)

// Sentinel errors
var ErrNotFound = errors.New("not found")
var ErrPermissionDenied = errors.New("permission denied")

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           ERROR WRAPPING (Go 1.13+)                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Wrapping errors with %w
    fmt.Println("\n📊 Wrapping Errors with %w:")
    err := readConfig("config.json")
    if err != nil {
        fmt.Printf("   Wrapped error: %v\n", err)
    }
    
    // Unwrapping
    fmt.Println("\n📊 errors.Unwrap():")
    innerErr := errors.Unwrap(err)
    fmt.Printf("   Unwrapped: %v\n", innerErr)
    
    // errors.Is() - check if error IS (or wraps) a specific error
    fmt.Println("\n📊 errors.Is() - Checking Error Type:")
    if errors.Is(err, os.ErrNotExist) {
        fmt.Println("   ✅ Error is (or wraps) os.ErrNotExist")
    }
    
    // errors.As() - extract specific error type
    fmt.Println("\n📊 errors.As() - Extracting Error Type:")
    err2 := processFile()
    if err2 != nil {
        var pathErr *os.PathError
        if errors.As(err2, &pathErr) {
            fmt.Printf("   Path: %s\n", pathErr.Path)
            fmt.Printf("   Op: %s\n", pathErr.Op)
        }
    }
    
    // Chain of wrapped errors
    fmt.Println("\n📊 Error Chain:")
    err3 := serviceCall()
    fmt.Printf("   Full chain: %v\n", err3)
    fmt.Printf("   Is ErrNotFound? %t\n", errors.Is(err3, ErrNotFound))
}

func readConfig(filename string) error {
    _, err := os.Open(filename)
    if err != nil {
        // Wrap with context using %w
        return fmt.Errorf("reading config %s: %w", filename, err)
    }
    return nil
}

func processFile() error {
    _, err := os.Open("/nonexistent/file.txt")
    if err != nil {
        return fmt.Errorf("processing: %w", err)
    }
    return nil
}

func serviceCall() error {
    err := repositoryCall()
    if err != nil {
        return fmt.Errorf("service layer: %w", err)
    }
    return nil
}

func repositoryCall() error {
    err := databaseCall()
    if err != nil {
        return fmt.Errorf("repository layer: %w", err)
    }
    return nil
}

func databaseCall() error {
    return fmt.Errorf("database query failed: %w", ErrNotFound)
}
```

---

## 🏭 Production Patterns

```go
// error_production.go
package main

import (
    "errors"
    "fmt"
    "log"
)

// Domain errors
var (
    ErrUserNotFound    = errors.New("user not found")
    ErrInvalidInput    = errors.New("invalid input")
    ErrUnauthorized    = errors.New("unauthorized")
    ErrInternalServer  = errors.New("internal server error")
)

type User struct {
    ID   int
    Name string
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION ERROR PATTERNS                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: Handle at appropriate level
    fmt.Println("\n📊 Pattern 1: Handle at Appropriate Level")
    handleRequest(1)
    handleRequest(999)
    
    // Pattern 2: Don't ignore errors!
    fmt.Println("\n⚠️ Pattern 2: DON'T Ignore Errors")
    fmt.Println("   // BAD:")
    fmt.Println("   result, _ := doSomething()  // Ignored!")
    fmt.Println("")
    fmt.Println("   // GOOD:")
    fmt.Println("   result, err := doSomething()")
    fmt.Println("   if err != nil { ... }")
    
    // Pattern 3: Wrap with context
    fmt.Println("\n📊 Pattern 3: Wrap with Context")
    fmt.Println("   return fmt.Errorf(\"getting user %d: %w\", id, err)")
    
    // Pattern 4: Use errors.Is/As for comparison
    fmt.Println("\n📊 Pattern 4: Compare with errors.Is()")
    fmt.Println("   if errors.Is(err, ErrNotFound) { ... }")
}

func handleRequest(userID int) {
    user, err := getUser(userID)
    
    // Handle different error types differently
    switch {
    case err == nil:
        fmt.Printf("   Found user: %+v\n", user)
    case errors.Is(err, ErrUserNotFound):
        fmt.Printf("   User %d not found (404)\n", userID)
    case errors.Is(err, ErrUnauthorized):
        fmt.Printf("   Unauthorized access (401)\n")
    default:
        // Log unexpected errors
        log.Printf("   Unexpected error: %v", err)
        fmt.Printf("   Internal server error (500)\n")
    }
}

func getUser(id int) (*User, error) {
    // Simulate database lookup
    if id == 1 {
        return &User{ID: 1, Name: "Alice"}, nil
    }
    return nil, fmt.Errorf("database query: %w", ErrUserNotFound)
}
```

---

## 🆚 Error vs Panic

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  USE ERROR (return err) FOR:                                    │
│  • Expected failures (file not found, network timeout)          │
│  • Invalid user input                                           │
│  • Business logic failures                                      │
│  • Anything recoverable                                         │
│  • 99% of error cases!                                          │
│                                                                 │
│  USE PANIC FOR:                                                 │
│  • Programming errors (bugs)                                    │
│  • Impossible states ("this should never happen")               │
│  • Initialization failures (can't start app)                    │
│  • Index out of bounds (automatic)                              │
│  • 1% of error cases!                                           │
│                                                                 │
│  RULE OF THUMB:                                                 │
│  "If in doubt, use error"                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Errors are values** - handle them explicitly
2. **Check errors immediately** after function calls
3. **Don't ignore errors** - use `_ = err` only when intentional
4. **Wrap errors** with context using `%w`
5. **Use errors.Is()** for sentinel error comparison
6. **Use errors.As()** to extract custom error types
7. **Define domain errors** as package-level sentinels
8. **Panic is for bugs**, not regular errors

---

## ➡️ Next Steps

**Next Topic:** [29 - Goroutines](./29-goroutines.md)

