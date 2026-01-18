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

> `defer` schedules a function call to be run AFTER the current function returns.

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  DEFER = "REMIND ME TO DO THIS BEFORE I LEAVE"                  │
│                                                                 │
│  When you enter a room:                                         │
│  1. defer "turn off lights"                                     │
│  2. defer "close window"                                        │
│  3. defer "lock door"                                           │
│  4. ... do work in the room ...                                 │
│                                                                 │
│  When you leave (function returns):                             │
│  • lock door     (last deferred = first executed)               │
│  • close window                                                 │
│  • turn off lights                                              │
│                                                                 │
│  ORDER: LIFO (Last In, First Out) - like a stack!               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Defer Basics

```go
// defer_basics.go
package main

import (
    "fmt"
    "os"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║                    DEFER IN GO                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic defer
    fmt.Println("\n📊 Basic Defer:")
    basicDefer()
    
    // LIFO order
    fmt.Println("\n📊 LIFO Order (Stack):")
    lifoDefer()
    
    // Defer with file handling
    fmt.Println("\n📊 File Handling Pattern:")
    readFile()
    
    // Defer evaluates arguments immediately
    fmt.Println("\n📊 Arguments Evaluated Immediately:")
    argumentEvaluation()
    
    // Defer with return value modification
    fmt.Println("\n📊 Modifying Named Return Values:")
    result := namedReturnDefer()
    fmt.Printf("   Result: %d\n", result)
    
    // Loop with defer
    fmt.Println("\n📊 Defer in Loop (Careful!):")
    loopDefer()
}

func basicDefer() {
    fmt.Println("   Start of function")
    defer fmt.Println("   This is deferred (runs last)")
    fmt.Println("   End of function")
}

func lifoDefer() {
    defer fmt.Println("   1st deferred (runs 3rd)")
    defer fmt.Println("   2nd deferred (runs 2nd)")
    defer fmt.Println("   3rd deferred (runs 1st)")
    fmt.Println("   Main code")
}

func readFile() {
    // Create a temp file
    file, err := os.CreateTemp("", "example")
    if err != nil {
        fmt.Printf("   Error creating file: %v\n", err)
        return
    }
    defer file.Close()  // Will run when function returns
    defer os.Remove(file.Name())  // Cleanup
    
    // Write to file
    file.WriteString("Hello, defer!")
    fmt.Printf("   Created and will cleanup: %s\n", file.Name())
    
    // File will be closed and removed after function returns
}

func argumentEvaluation() {
    x := 10
    defer fmt.Printf("   Deferred value of x: %d\n", x)  // x is evaluated NOW
    x = 20
    fmt.Printf("   Current value of x: %d\n", x)
    // Output: Current: 20, Deferred: 10
}

func namedReturnDefer() (result int) {
    result = 10
    defer func() {
        result = result + 5  // Modifies the return value!
    }()
    return result  // Returns 15, not 10!
}

func loopDefer() {
    // ⚠️ CAUTION: Deferred calls accumulate until function returns!
    for i := 0; i < 3; i++ {
        defer fmt.Printf("   Deferred %d\n", i)
    }
    fmt.Println("   After loop, before function ends")
    // All defers execute when function returns (not when loop ends)
}
```

### Common Defer Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  COMMON DEFER PATTERNS                                          │
│                                                                 │
│  1. RESOURCE CLEANUP                                            │
│     ─────────────────                                           │
│     file, _ := os.Open("file.txt")                              │
│     defer file.Close()                                          │
│                                                                 │
│  2. UNLOCK MUTEX                                                │
│     ────────────────                                            │
│     mutex.Lock()                                                │
│     defer mutex.Unlock()                                        │
│                                                                 │
│  3. CLOSE DATABASE CONNECTION                                   │
│     ─────────────────────────                                   │
│     db, _ := sql.Open("mysql", connStr)                         │
│     defer db.Close()                                            │
│                                                                 │
│  4. FINISH TRACING SPAN                                         │
│     ─────────────────────                                       │
│     span := tracer.StartSpan("operation")                       │
│     defer span.Finish()                                         │
│                                                                 │
│  5. RECOVER FROM PANIC                                          │
│     ─────────────────────                                       │
│     defer func() {                                              │
│         if r := recover(); r != nil {                           │
│             log.Printf("Recovered: %v", r)                      │
│         }                                                       │
│     }()                                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🚨 Panic

### What is Panic?

> `panic` is a built-in function that stops the normal execution flow and begins panicking.

### When to Use Panic

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  USE PANIC FOR:                                                 │
│  ──────────────                                                 │
│  ✅ Unrecoverable errors (programming bugs)                     │
│  ✅ Should-never-happen situations                              │
│  ✅ Invalid function arguments (during development)             │
│  ✅ Initialization failures (can't continue)                    │
│                                                                 │
│  DON'T USE PANIC FOR:                                           │
│  ────────────────────                                           │
│  ❌ Expected errors (file not found, network timeout)           │
│  ❌ User input validation                                       │
│  ❌ Business logic errors                                       │
│  ❌ Anything that can be handled gracefully                     │
│                                                                 │
│  Go philosophy: "Don't panic!"                                  │
│  Use error returns for normal error handling.                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Panic

```go
// panic_demo.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║                    PANIC IN GO                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Panic example (with recovery)
    fmt.Println("\n📊 Panic with Recovery:")
    safeDivide(10, 2)
    safeDivide(10, 0)  // Would panic, but we recover
    
    // Array out of bounds (runtime panic)
    fmt.Println("\n📊 Runtime Panics:")
    fmt.Println("   Examples of runtime panics:")
    fmt.Println("   • arr[100] on small array → index out of range")
    fmt.Println("   • nil pointer dereference → nil pointer")
    fmt.Println("   • type assertion failure → interface conversion")
    
    // When to use panic
    fmt.Println("\n📊 When to Use Panic:")
    fmt.Println("   ✅ Initialization failure: panic(\"config not found\")")
    fmt.Println("   ✅ Programming bug: panic(\"this should never happen\")")
    fmt.Println("   ❌ File not found: return err (not panic)")
    fmt.Println("   ❌ Invalid user input: return err (not panic)")
    
    fmt.Println("\n✅ Program continues after recovered panics!")
}

func safeDivide(a, b int) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("   Recovered from panic: %v\n", r)
        }
    }()
    
    if b == 0 {
        panic("division by zero")
    }
    
    result := a / b
    fmt.Printf("   %d / %d = %d\n", a, b, result)
}
```

---

## 🔄 Recover

### What is Recover?

> `recover` is a built-in function that regains control of a panicking goroutine.

### Key Rules

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  RECOVER RULES                                                  │
│                                                                 │
│  1. MUST be called inside a DEFERRED function                   │
│     ────────────────────────────────────────                    │
│     ✅ defer func() { recover() }()                             │
│     ❌ recover()  // Does nothing if not in defer               │
│                                                                 │
│  2. Only recovers panic in CURRENT goroutine                    │
│     ───────────────────────────────────────                     │
│     Can't recover panic from another goroutine                  │
│                                                                 │
│  3. Returns nil if NO panic is happening                        │
│     ───────────────────────────────────                         │
│     if r := recover(); r != nil { // only if panicking }        │
│                                                                 │
│  4. Returns the panic VALUE if panicking                        │
│     ─────────────────────────────────────                       │
│     panic("error message") → recover() returns "error message"  │
│     panic(err) → recover() returns err                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Recover

```go
// recover_demo.go
package main

import (
    "errors"
    "fmt"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║                   RECOVER IN GO                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic recovery pattern
    fmt.Println("\n📊 Basic Recovery Pattern:")
    fmt.Printf("   Result: %v\n", safeCall(normalFunc))
    fmt.Printf("   Result: %v\n", safeCall(panicFunc))
    
    // Converting panic to error
    fmt.Println("\n📊 Convert Panic to Error:")
    result, err := safeOperation()
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   Result: %v\n", result)
    }
    
    // HTTP handler pattern
    fmt.Println("\n📊 HTTP Handler Pattern:")
    simulateHTTPHandler()
    
    fmt.Println("\n✅ All panics were recovered!")
}

func normalFunc() {
    fmt.Println("   Normal function executed")
}

func panicFunc() {
    panic("something went wrong!")
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
    return "Function completed"
}

// Converting panic to error (common pattern)
func safeOperation() (result string, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic recovered: %v", r)
        }
    }()
    
    // Simulate something that panics
    riskyOperation()
    
    return "success", nil
}

func riskyOperation() {
    panic(errors.New("internal error"))
}

// HTTP handler pattern (like in production)
func simulateHTTPHandler() {
    // This is how gRPC/HTTP servers recover from panics
    handler := func(request string) {
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("   Handler panic recovered: %v\n", r)
                fmt.Println("   Returning 500 Internal Server Error")
            }
        }()
        
        if request == "bad" {
            panic("bad request handling")
        }
        
        fmt.Printf("   Handled request: %s\n", request)
    }
    
    handler("good-request")
    handler("bad")  // Panics but recovers
    handler("another-good")  // Still works!
}
```

---

## 🏭 Production Pattern

### Real Production Recovery (from Catalyst)

```go
// production_panic_handler.go
package main

import (
    "fmt"
    "runtime/debug"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION PANIC HANDLING                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Simulate gRPC interceptor pattern
    fmt.Println("\n📊 gRPC Interceptor Pattern:")
    simulateGRPCHandler("ValidRequest")
    simulateGRPCHandler("PanicRequest")
    simulateGRPCHandler("AnotherRequest")
    
    fmt.Println("\n✅ Server continues running after panics!")
}

// This pattern is similar to grpc_recovery interceptor in production
func simulateGRPCHandler(request string) {
    // Recovery wrapper
    func() {
        defer func() {
            if r := recover(); r != nil {
                // 1. Log the panic
                fmt.Printf("   🚨 PANIC RECOVERED: %v\n", r)
                
                // 2. Log stack trace
                fmt.Println("   Stack trace:")
                lines := string(debug.Stack())
                // In production, this goes to logging service
                fmt.Printf("   %s...\n", lines[:200])
                
                // 3. Return error to client
                fmt.Println("   → Returning: Internal Server Error")
                
                // 4. Increment panic counter (metrics)
                fmt.Println("   → Incrementing panic metric counter")
                
                // 5. Alert (in production)
                fmt.Println("   → Sending alert to on-call team")
            }
        }()
        
        // Simulate handler
        handleRequest(request)
    }()
}

func handleRequest(request string) {
    if request == "PanicRequest" {
        panic("simulated panic in handler")
    }
    fmt.Printf("   ✅ Handled: %s\n", request)
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
│  // try-finally                    // defer                     │
│  try {                             file := openFile()           │
│      file = openFile();            defer file.Close()           │
│      // use file                   // use file                  │
│  } finally {                       // Close called at return    │
│      file.close();                                              │
│  }                                                              │
│                                                                 │
│  // throw exception                // panic                     │
│  throw new RuntimeException();     panic("error")               │
│                                                                 │
│  // catch exception                // recover (in defer)        │
│  try {                             defer func() {               │
│      riskyOperation();                 if r := recover(); r != nil│
│  } catch (Exception e) {               // handle               │
│      // handle                     }()                          │
│  }                                 riskyOperation()             │
│                                                                 │
│  // Use exceptions for errors      // Use error returns!        │
│  throws IOException                return data, err             │
│                                                                 │
│  KEY DIFFERENCE:                                                │
│  Java uses exceptions for ALL errors.                           │
│  Go uses error returns for expected errors,                     │
│      panic/recover for unexpected errors only.                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **`defer`** schedules code to run when function returns (LIFO order)
2. **Arguments are evaluated immediately** when defer is called
3. **`panic`** stops normal execution - use for unrecoverable errors only
4. **`recover`** must be called inside a deferred function
5. **`recover` returns nil** if not panicking
6. **Error returns** are preferred over panic for normal errors
7. **Production pattern**: Use recover in HTTP/gRPC interceptors

---

## ➡️ Next Steps

You now understand defer, panic, and recover. Let's explore arrays.

**Next Topic:** [20 - Arrays](./20-arrays.md)

