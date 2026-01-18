# 63 - Common Go Interview Questions

> Top Go interview questions with answers - your final review.

---

## 📌 Quick Reference for Interviews

This chapter covers the most frequently asked Go interview questions. Use it as a final review before your interview!

---

## 🔤 Language Basics

### Q1: What is the zero value in Go?
```go
// Every type has a zero value (default when not initialized)
var i int       // 0
var f float64   // 0.0
var b bool      // false
var s string    // "" (empty string)
var p *int      // nil
var sl []int    // nil
var m map[K]V   // nil
var ch chan int // nil
var fn func()   // nil
var iface interface{} // nil

// Struct: all fields get their zero values
type User struct {
    Name string  // ""
    Age  int     // 0
}
var u User // User{Name: "", Age: 0}
```

### Q2: What's the difference between var and :=?
```go
// var: explicit declaration, can be package-level
var x int = 10
var y = 10     // Type inferred
var z int      // Zero value

// :=  short declaration, only inside functions
x := 10        // Declare + assign, type inferred

// := requires at least one NEW variable on left
x := 10        // OK: x is new
x, y := 10, 20 // OK: y is new
x, y := 20, 30 // ERROR: no new variables
```

### Q3: Difference between make and new?
```go
// new(T): allocates zeroed memory, returns *T
p := new(int)    // *int pointing to 0
s := new([]int)  // *[]int pointing to nil slice!

// make(T, args): only for slice, map, channel
// Returns T (not pointer), initializes internal structures
sl := make([]int, 5)    // []int with len=5
m := make(map[K]V)      // Ready-to-use map
ch := make(chan int, 5) // Buffered channel

// Key difference: new gives you pointer to zero value
// make initializes internal structures
```

---

## 📦 Slices and Maps

### Q4: What happens when you append to a slice beyond capacity?
```go
s := make([]int, 0, 2)
// len=0, cap=2

s = append(s, 1, 2)  // len=2, cap=2 (fits)
s = append(s, 3)     // len=3, cap=4 (NEW underlying array!)

// When capacity exceeded:
// 1. New array allocated (usually 2x capacity)
// 2. Existing elements copied
// 3. New slice points to new array
// 4. Old array eventually GC'd
```

### Q5: Can you use a map concurrently?
```go
// NO! Maps are NOT thread-safe

// This will panic or corrupt:
// go func() { m["a"] = 1 }()
// go func() { m["b"] = 2 }()

// Solutions:
// 1. sync.Mutex
var mu sync.Mutex
mu.Lock()
m["key"] = value
mu.Unlock()

// 2. sync.RWMutex (for read-heavy)
var rw sync.RWMutex
rw.RLock()
v := m["key"]
rw.RUnlock()

// 3. sync.Map (built-in concurrent map)
var sm sync.Map
sm.Store("key", "value")
v, ok := sm.Load("key")
```

### Q6: What's the difference between nil slice and empty slice?
```go
var nilSlice []int       // nil (no underlying array)
emptySlice := []int{}    // Non-nil, len=0, cap=0
madeSlice := make([]int, 0) // Non-nil, len=0, cap=0

fmt.Println(nilSlice == nil)   // true
fmt.Println(emptySlice == nil) // false

// Practically: both work the same for append, len, range
// But: nil serializes differently in JSON (null vs [])
```

---

## 🔌 Interfaces

### Q7: How are Go interfaces different from Java?
```go
// Go: Implicit satisfaction
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Any type with this method satisfies Reader
type MyReader struct{}
func (r MyReader) Read(p []byte) (int, error) { return 0, nil }

// Java: Explicit "implements"
// class MyReader implements Reader { ... }

// Key differences:
// - No "implements" keyword
// - Interface satisfied by having methods, not declaring
// - Can satisfy interfaces from other packages
// - Interfaces can be satisfied after type is defined
```

### Q8: Explain nil interface vs interface holding nil
```go
var err1 error = nil  // nil interface (type=nil, data=nil)
fmt.Println(err1 == nil) // true

var ptr *MyError = nil
var err2 error = ptr  // Interface with type but nil data
fmt.Println(err2 == nil) // FALSE!

// err2 has: type=*MyError, data=nil
// Interface is nil ONLY when BOTH type AND data are nil

// Common bug:
func getError() error {
    var e *MyError = nil
    return e  // Returns non-nil interface!
}

// Fix:
func getError() error {
    return nil  // Returns nil interface
}
```

---

## 🔄 Concurrency

### Q9: Difference between goroutine and thread?
```
Threads (OS):
- Heavy (~1MB stack)
- Managed by OS kernel
- Context switch is expensive
- Limited number (thousands)

Goroutines (Go):
- Lightweight (~2KB stack, grows)
- Managed by Go runtime
- Fast context switch (user-space)
- Millions possible

Go uses M:N scheduling (M goroutines on N threads)
```

### Q10: Buffered vs Unbuffered channels?
```go
// Unbuffered: synchronous, blocks until receiver ready
ch := make(chan int)
ch <- 42  // Blocks until someone receives

// Buffered: async up to capacity
ch := make(chan int, 3)
ch <- 1  // Doesn't block (buffer has space)
ch <- 2  // Doesn't block
ch <- 3  // Doesn't block
ch <- 4  // BLOCKS (buffer full)

// Use unbuffered for: synchronization, handoff
// Use buffered for: decoupling, batch processing
```

### Q11: What is a race condition? How to detect?
```go
// Race: two goroutines access same variable, at least one writes
var counter int

go func() { counter++ }()  // Write
go func() { counter++ }()  // Write
// Result is unpredictable!

// Detection: run with -race flag
// go run -race main.go
// go test -race ./...

// Fixes:
// 1. Mutex
// 2. Channels
// 3. sync/atomic
// 4. Avoid shared state
```

### Q12: What causes a deadlock?
```go
// Deadlock: goroutines waiting for each other forever

// Example 1: Unbuffered channel, no receiver
ch := make(chan int)
ch <- 42  // DEADLOCK: no one to receive

// Example 2: Mutex, locking twice
var mu sync.Mutex
mu.Lock()
mu.Lock()  // DEADLOCK: already locked

// Example 3: Circular wait
// A waits for B, B waits for A

// Prevention:
// - Always have matching send/receive
// - Use defer mu.Unlock()
// - Lock ordering
// - Timeouts with select
```

---

## ⚠️ Error Handling

### Q13: How does Go handle errors?
```go
// Errors are values, not exceptions
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Caller must check error
result, err := divide(10, 0)
if err != nil {
    log.Fatal(err)
}

// Error wrapping (Go 1.13+)
return fmt.Errorf("divide failed: %w", err)

// Checking wrapped errors
if errors.Is(err, ErrDivisionByZero) { ... }

// Extracting error type
var myErr *MyError
if errors.As(err, &myErr) { ... }
```

### Q14: When to use panic vs error?
```go
// Error: normal failure cases
// - File not found
// - Invalid input
// - Network timeout
func readFile(path string) ([]byte, error)

// Panic: programmer errors, unrecoverable
// - Index out of bounds
// - Nil pointer (bug)
// - Impossible states
if x == nil {
    panic("x should never be nil")
}

// Rule: Errors for expected failures, panic for bugs
// Don't use panic for control flow!
```

---

## 🏭 Production

### Q15: What is context.Context used for?
```go
// Context carries: deadlines, cancellation, values

// 1. Cancellation
ctx, cancel := context.WithCancel(context.Background())
go worker(ctx)
cancel()  // Signals worker to stop

// 2. Timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

// 3. Request-scoped values
ctx = context.WithValue(ctx, "userID", 123)

// All blocking operations should accept context:
func Query(ctx context.Context, query string) (*Result, error)
```

### Q16: How do you implement graceful shutdown?
```go
// 1. Catch shutdown signal
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

// 2. Start server
go server.ListenAndServe()

// 3. Wait for signal
<-quit

// 4. Graceful shutdown with timeout
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
server.Shutdown(ctx)  // Waits for active requests
```

---

## 🎯 Quick Fire Round

| Question | One-Line Answer |
|----------|-----------------|
| Is Go OOP? | Go has methods and interfaces, but no inheritance |
| What's a method receiver? | First parameter that binds method to type |
| Value vs pointer receiver? | Value for read-only, pointer for mutation |
| What's defer? | Executes when function returns, LIFO order |
| What's init()? | Runs automatically before main() |
| Exported vs unexported? | Capital letter = public, lowercase = private |
| What's GOPATH? | Legacy workspace, now modules preferred |
| What's go.mod? | Module definition, dependencies |
| What's go.sum? | Checksums for dependencies |
| What's GOMAXPROCS? | Number of OS threads for goroutines |

---

## 🎉 You're Ready!

You've completed the comprehensive Go tutorial with **64 chapters**!

**Final Tips:**
1. Practice coding, not just reading
2. Build real projects
3. Read standard library code
4. Explain concepts out loud (rubber duck debugging)
5. Be honest about what you don't know

**Good luck with your interview! 🚀**

