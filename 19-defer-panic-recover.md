# 19 - Defer, Panic & Recover

> Go's unique mechanisms for cleanup, error handling, and recovery from critical errors.

---

## 📌 What You'll Learn

- What `defer` is and how it works
- LIFO order of deferred functions
- How `panic` and `recover` work
- When to use each mechanism
- Sample programs with real examples

---

## 🔄 Defer

### What is Defer?

`defer` schedules a function call to run **after** the current function returns.

**Analogy:** When you enter a room, you might "defer" turning off the lights, closing the window, and locking the door. When you leave, these run in reverse order (LIFO): lock door first, then close window, then turn off lights.

### Basic Defer

```go
func example() {
    fmt.Println("Start")
    defer fmt.Println("Deferred (runs last)")
    fmt.Println("End")
}
example()
// Output: Start
//         End
//         Deferred (runs last)
```

### LIFO Order

Deferred calls execute in reverse order (last deferred = first executed):

```go
func lifoExample() {
    defer fmt.Println("1st deferred (runs 3rd)")
    defer fmt.Println("2nd deferred (runs 2nd)")
    defer fmt.Println("3rd deferred (runs 1st)")
    fmt.Println("Main code")
}
lifoExample()
// Output: Main code
//         3rd deferred (runs 1st)
//         2nd deferred (runs 2nd)
//         1st deferred (runs 3rd)
```

### Arguments Evaluated Immediately

The arguments to `defer` are evaluated when the `defer` statement runs, not when the deferred function executes:

```go
x := 10
defer fmt.Println(x)  // x is 10 now
x = 20
fmt.Println(x)
// Output: 20
//         10
```

### Modifying Named Return Values

A deferred function can modify named return values:

```go
func namedReturnDefer() (result int) {
    result = 10
    defer func() {
        result = result + 5
    }()
    return result
}
fmt.Println(namedReturnDefer())  // Output: 15
```

### Common Defer Patterns

- **Resource cleanup:** `defer file.Close()`
- **Unlock mutex:** `defer mutex.Unlock()`
- **Close DB connection:** `defer db.Close()`
- **Finish tracing span:** `defer span.Finish()`
- **Recover from panic:** `defer func() { recover() }()`

### Complete Example: Defer

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    basicDefer()
    lifoDefer()
    file, _ := os.CreateTemp("", "ex")
    defer file.Close()
    defer os.Remove(file.Name())
    file.WriteString("hello")
    fmt.Println("File created, will cleanup on return")
}

func basicDefer() {
    fmt.Println("Start")
    defer fmt.Println("Deferred")
    fmt.Println("End")
}

func lifoDefer() {
    defer fmt.Println("third")
    defer fmt.Println("second")
    defer fmt.Println("first")
    fmt.Println("main")
}
```

**Output:**
```
Start
End
Deferred
main
first
second
third
File created, will cleanup on return
```

---

## 🚨 Panic

### What is Panic?

`panic` stops normal execution and begins panicking. If not recovered, the program exits.

### When to Use Panic

**Use for:**
- Unrecoverable errors (programming bugs)
- Should-never-happen situations
- Initialization failures where the program cannot continue

**Don't use for:**
- Expected errors (file not found, network timeout)
- User input validation
- Business logic errors
- Anything that can be handled gracefully

Go philosophy: use error returns for normal errors; reserve panic for truly unexpected cases.

### Panic Example

```go
func safeDivide(a, b int) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("Recovered: %v\n", r)
        }
    }()
    if b == 0 {
        panic("division by zero")
    }
    fmt.Println(a / b)
}
safeDivide(10, 2)  // Output: 5
safeDivide(10, 0)  // Output: Recovered: division by zero
```

---

## 🔄 Recover

### What is Recover?

`recover` regains control of a panicking goroutine. It only works inside a deferred function.

### Recover Rules

- Must be called inside a **deferred** function
- Only recovers panic in the **current** goroutine
- Returns `nil` if no panic is happening
- Returns the panic value if panicking

### Basic Recovery

```go
func safeCall(fn func()) (result string) {
    defer func() {
        if r := recover(); r != nil {
            result = fmt.Sprintf("Recovered: %v", r)
        } else {
            result = "Success"
        }
    }()
    fn()
    return "completed"
}
fmt.Println(safeCall(func() { fmt.Println("OK") }))   // Output: OK\nSuccess
fmt.Println(safeCall(func() { panic("oops") }))       // Output: Recovered: oops
```

### Convert Panic to Error

```go
func safeOperation() (result string, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic recovered: %v", r)
        }
    }()
    panic("internal error")
    return "success", nil
}
_, err := safeOperation()
fmt.Println(err)  // Output: panic recovered: internal error
```

### Complete Example: Panic and Recover

```go
package main

import "fmt"

func main() {
    safeDivide(10, 2)
    safeDivide(10, 0)
    fmt.Println(safeCall(normalFunc))
    fmt.Println(safeCall(panicFunc))
}

func normalFunc() {
    fmt.Println("Normal")
}

func panicFunc() {
    panic("oops")
}

func safeDivide(a, b int) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("Recovered: %v\n", r)
        }
    }()
    if b == 0 {
        panic("division by zero")
    }
    fmt.Println(a / b)
}

func safeCall(fn func()) (result string) {
    defer func() {
        if r := recover(); r != nil {
            result = fmt.Sprintf("Recovered: %v", r)
        } else {
            result = "Success"
        }
    }()
    fn()
    return "completed"
}
```

**Output:**
```
5
Recovered: division by zero
Normal
Success
Recovered: oops
```

---

## 🏭 Production Pattern

HTTP/gRPC servers often wrap handlers with a recovery interceptor:

```go
func withRecovery(handler func()) {
    defer func() {
        if r := recover(); r != nil {
            log.Printf("Panic recovered: %v", r)
            log.Printf("Stack: %s", debug.Stack())
            // Return 500 to client, increment metrics, alert
        }
    }()
    handler()
}
```

### Complete Example: Production Recovery

```go
package main

import (
    "fmt"
    "runtime/debug"
)

func main() {
    handleRequest("ValidRequest")
    handleRequest("PanicRequest")
    handleRequest("AnotherRequest")
}

func handleRequest(req string) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("Panic recovered: %v\n", r)
            fmt.Printf("Stack: %s\n", string(debug.Stack())[:100])
            fmt.Println("Returning 500")
        }
    }()
    if req == "PanicRequest" {
        panic("handler panic")
    }
    fmt.Printf("Handled: %s\n", req)
}
```

**Output:**
```
Handled: ValidRequest
Panic recovered: handler panic
Stack: goroutine 1 [running]:
runtime/debug.Stack(...)
...
Returning 500
Handled: AnotherRequest
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `try { ... } finally { file.close(); }` | `defer file.Close()` |
| `throw new RuntimeException()` | `panic("error")` |
| `try { ... } catch (Exception e) { ... }` | `defer func() { if r := recover(); r != nil { ... } }()` |
| Exceptions for all errors | Error returns for expected errors; panic for unexpected only |

**Key difference:** Java uses exceptions for many errors. Go prefers error returns for expected errors and reserves panic/recover for truly unexpected situations.

---

## 🎯 Key Takeaways

1. **`defer`** schedules code to run when the function returns (LIFO order)
2. **Arguments are evaluated immediately** when defer is called
3. **`panic`** stops normal execution—use only for unrecoverable errors
4. **`recover`** must be called inside a deferred function
5. **`recover` returns nil** if not panicking
6. **Error returns** are preferred over panic for normal errors
7. **Production pattern:** Use recover in HTTP/gRPC interceptors

---

## ➡️ Next Steps

You now understand defer, panic, and recover. Let's explore arrays.

**Next Topic:** [20 - Arrays](./20-arrays.md)
