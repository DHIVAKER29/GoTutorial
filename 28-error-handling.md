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

**Java:** Exceptions—hidden control flow. Exceptions can propagate from anywhere; you catch them (maybe).

**Go:** Errors as values—explicit control flow. You get `(result, err)` and must handle it explicitly. Errors are visible in the code, control flow is clear, and you're encouraged not to ignore them.

---

## 📝 The error Interface

```go
type error interface {
    Error() string
}
```

```go
// Create errors
err1 := errors.New("something went wrong")
err2 := fmt.Errorf("could not open %s", name)
// Output: something went wrong
//         could not open file.txt
```

```go
// Standard pattern
result, err := divide(10, 0)
if err != nil {
    fmt.Printf("Error: %v\n", err)
} else {
    fmt.Printf("Result: %.2f\n", result)
}
// Output: Error: division by zero
```

### Complete Example: The error Interface

```go
package main

import (
    "errors"
    "fmt"
)

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

func main() {
    err1 := errors.New("something went wrong")
    fmt.Printf("errors.New(): %v\n", err1)

    err2 := fmt.Errorf("could not open %s", "file.txt")
    fmt.Printf("fmt.Errorf(): %v\n", err2)

    result, err := divide(10, 0)
    if err != nil {
        fmt.Printf("divide(10,0): %v\n", err)
    } else {
        fmt.Printf("Result: %.2f\n", result)
    }

    result, err = divide(10, 2)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Result: %.2f\n", result)
    }
}
```

**Output:**
```
errors.New(): something went wrong
fmt.Errorf(): could not open file.txt
divide(10,0): division by zero
Result: 5.00
```

---

## 🎨 Custom Errors

Define structs that implement the `error` interface for domain-specific errors with extra context.

```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on '%s': %s", e.Field, e.Message)
}

// Usage
return &ValidationError{Field: "name", Message: "cannot be empty"}
```

```go
// Type assertion to access fields
if vErr, ok := err.(*ValidationError); ok {
    fmt.Printf("Field: %s\n", vErr.Field)
}
```

### Complete Example: Custom Errors

```go
package main

import "fmt"

type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on '%s': %s", e.Field, e.Message)
}

type HTTPError struct {
    StatusCode int
    Message    string
}

func (e *HTTPError) Error() string {
    return fmt.Sprintf("HTTP %d: %s", e.StatusCode, e.Message)
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

func main() {
    err := validateUser("", 15)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        if vErr, ok := err.(*ValidationError); ok {
            fmt.Printf("Field: %s\n", vErr.Field)
        }
    }

    err = &HTTPError{StatusCode: 404, Message: "Resource not found"}
    fmt.Printf("HTTP Error: %v\n", err)
}
```

**Output:**
```
Error: validation error on 'name': cannot be empty
Field: name
HTTP Error: HTTP 404: Resource not found
```

---

## 🔗 Error Wrapping (Go 1.13+)

Use `%w` in `fmt.Errorf` to wrap errors with context. Use `errors.Is()` and `errors.As()` to inspect wrapped errors.

```go
// Wrap with context
return fmt.Errorf("reading config %s: %w", filename, err)
```

```go
// errors.Is() - check if error is (or wraps) a specific error
if errors.Is(err, os.ErrNotExist) {
    fmt.Println("File not found")
}
```

```go
// errors.As() - extract specific error type
var pathErr *os.PathError
if errors.As(err, &pathErr) {
    fmt.Printf("Path: %s, Op: %s\n", pathErr.Path, pathErr.Op)
}
```

### Complete Example: Error Wrapping

```go
package main

import (
    "errors"
    "fmt"
    "os"
)

var ErrNotFound = errors.New("not found")

func readConfig(filename string) error {
    _, err := os.Open(filename)
    if err != nil {
        return fmt.Errorf("reading config %s: %w", filename, err)
    }
    return nil
}

func main() {
    err := readConfig("config.json")
    if err != nil {
        fmt.Printf("Wrapped error: %v\n", err)
    }

    innerErr := errors.Unwrap(err)
    fmt.Printf("Unwrapped: %v\n", innerErr)

    if errors.Is(err, os.ErrNotExist) {
        fmt.Println("Error is (or wraps) os.ErrNotExist")
    }
}
```

**Output:**
```
Wrapped error: reading config config.json: open config.json: no such file or directory
Unwrapped: open config.json: no such file or directory
Error is (or wraps) os.ErrNotExist
```

---

## 🏭 Production Patterns

- **Handle at appropriate level**—don't swallow; return or handle
- **Don't ignore errors**—avoid `result, _ := doSomething()`
- **Wrap with context**—`return fmt.Errorf("getting user %d: %w", id, err)`
- **Use errors.Is/As** for comparison and extraction
- **Define domain errors** as package-level sentinels: `var ErrNotFound = errors.New("not found")`

### Complete Example: Production Patterns

```go
package main

import (
    "errors"
    "fmt"
)

var (
    ErrUserNotFound = errors.New("user not found")
    ErrUnauthorized = errors.New("unauthorized")
)

type User struct {
    ID   int
    Name string
}

func getUser(id int) (*User, error) {
    if id == 1 {
        return &User{ID: 1, Name: "Alice"}, nil
    }
    return nil, fmt.Errorf("database query: %w", ErrUserNotFound)
}

func main() {
    user, err := getUser(1)
    if err == nil {
        fmt.Printf("Found user: %+v\n", user)
    }

    _, err = getUser(999)
    if errors.Is(err, ErrUserNotFound) {
        fmt.Println("User 999 not found (404)")
    }
}
```

**Output:**
```
Found user: &{ID:1 Name:Alice}
User 999 not found (404)
```

---

## 🆚 Error vs Panic

| Use **error** (return err) for | Use **panic** for |
|--------------------------------|-------------------|
| Expected failures (file not found, timeout) | Programming errors (bugs) |
| Invalid user input | Impossible states |
| Business logic failures | Init failures (can't start app) |
| Recoverable cases | Index out of bounds (automatic) |
| **~99% of cases** | **~1% of cases** |

**Rule of thumb:** If in doubt, use error.

---

## 🎯 Key Takeaways

1. **Errors are values**—handle them explicitly
2. **Check errors immediately** after function calls
3. **Don't ignore errors**—use `_ = err` only when intentional
4. **Wrap errors** with context using `%w`
5. **Use errors.Is()** for sentinel error comparison
6. **Use errors.As()** to extract custom error types
7. **Define domain errors** as package-level sentinels
8. **Panic is for bugs**, not regular errors

---

## ➡️ Next Steps

**Next Topic:** [29 - Goroutines](./29-goroutines.md)
