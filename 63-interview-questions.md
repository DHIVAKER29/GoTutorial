# 63 - Common Go Interview Questions

> Top Go interview questions with answers - your final review.

---

## 📌 Quick Reference for Interviews

This chapter covers the most frequently asked Go interview questions. Use it as a final review before your interview!

---

## 🔤 Language Basics

### Q1: What is the zero value in Go?
```go
var i int       // 0
var f float64   // 0.0
var b bool      // false
var s string    // ""
var p *int      // nil
var sl []int    // nil
var m map[K]V   // nil
var ch chan int // nil
var u User      // User{Name: "", Age: 0}
```

### Q2: What's the difference between var and :=?
```go
var x int = 10   // Explicit, can be package-level
x := 10         // Short decl, only inside functions, type inferred
x, y := 10, 20  // OK: at least one new variable required
x, y := 20, 30  // ERROR: no new variables
```

### Q3: Difference between make and new?
```go
// new(T): returns *T, zeroed memory
p := new(int)    // *int = 0
s := new([]int)  // *[]int = nil slice!

// make(T, args): only slice, map, channel, returns T
sl := make([]int, 5)
m := make(map[K]V)
ch := make(chan int, 5)
```

---

## 📦 Slices and Maps

### Q4: What happens when you append beyond capacity?
```go
s := make([]int, 0, 2)
s = append(s, 1, 2)  // len=2, cap=2
s = append(s, 3)     // New array! len=3, cap=4
// 1. New array (usually 2x cap)
// 2. Copy existing
// 3. Slice points to new array
```

### Q5: Can you use a map concurrently?
```go
// NO! Maps are NOT thread-safe
// Solutions: sync.Mutex, sync.RWMutex, sync.Map
```

### Q6: nil slice vs empty slice?
```go
var nilSlice []int        // nil
emptySlice := []int{}    // Non-nil, len=0
madeSlice := make([]int, 0)  // Non-nil, len=0

// Both work for append, len, range
// JSON: nil → null, empty → []
```

---

## 🔌 Interfaces

### Q7: How are Go interfaces different from Java?
```go
// Go: Implicit - any type with methods satisfies
type Reader interface { Read(p []byte) (int, error) }
type MyReader struct{}
func (r MyReader) Read(p []byte) (int, error) { return 0, nil }
// MyReader satisfies Reader automatically

// Java: Explicit "implements Reader"
```

### Q8: nil interface vs interface holding nil?
```go
var err1 error = nil     // nil interface
fmt.Println(err1 == nil) // true

var ptr *MyError = nil
var err2 error = ptr    // Interface with type, data=nil
fmt.Println(err2 == nil) // FALSE!
// Interface nil only when BOTH type AND data are nil
```

---

## 🔄 Concurrency

### Q9: Goroutine vs thread?

| Threads | Goroutines |
|---------|------------|
| Heavy (~1MB stack) | Lightweight (~2KB) |
| OS-managed | Go runtime |
| Expensive context switch | Fast (user-space) |
| Thousands | Millions |
| M:N scheduling | M goroutines on N threads |

### Q10: Buffered vs unbuffered channels?
```go
ch := make(chan int)      // Unbuffered: blocks until receiver
ch := make(chan int, 3)  // Buffered: async up to capacity
```

### Q11: Race condition? How to detect?
```go
// Race: concurrent access, at least one write
// Detection: go run -race, go test -race
// Fixes: Mutex, channels, atomic, avoid shared state
```

### Q12: What causes deadlock?
```go
// Unbuffered channel, no receiver
ch := make(chan int)
ch <- 42  // DEADLOCK

// Mutex locked twice
mu.Lock()
mu.Lock()  // DEADLOCK

// Circular wait: A waits B, B waits A
```

---

## ⚠️ Error Handling

### Q13: How does Go handle errors?
```go
// Errors are values
result, err := divide(10, 0)
if err != nil { ... }

// Wrapping: fmt.Errorf("failed: %w", err)
// Check: errors.Is(err, ErrNotFound), errors.As(err, &myErr)
```

### Q14: When panic vs error?
```go
// Error: expected failures (file not found, invalid input)
// Panic: programmer errors, unrecoverable (nil pointer, impossible state)
// Don't use panic for control flow!
```

---

## 🏭 Production

### Q15: What is context.Context for?
```go
// Cancellation, deadlines, request-scoped values
ctx, cancel := context.WithCancel(context.Background())
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
ctx = context.WithValue(ctx, "userID", 123)
```

### Q16: Graceful shutdown?
```go
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit

ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
server.Shutdown(ctx)
```

---

## 🎯 Quick Fire Round

| Question | One-Line Answer |
|----------|-----------------|
| Is Go OOP? | Methods and interfaces, no inheritance |
| Method receiver? | First param that binds method to type |
| Value vs pointer receiver? | Value for read-only, pointer for mutation |
| defer? | Executes when function returns, LIFO |
| init()? | Runs automatically before main() |
| Exported vs unexported? | Capital = public, lowercase = private |
| go.mod? | Module definition, dependencies |
| go.sum? | Checksums for dependencies |
| GOMAXPROCS? | Number of OS threads for goroutines |

---

## 🎉 You're Ready!

**Final Tips:** Practice coding, build real projects, read standard library code, explain concepts out loud, be honest about what you don't know.

**Good luck with your interview! 🚀**
