# 29 - Goroutines & Concurrency Fundamentals

> Understanding concurrency, parallelism, the Go scheduler, and goroutines in depth.

---

## 📌 What You'll Learn

- Concurrency vs Parallelism (the critical difference!)
- Processes, Threads, and Goroutines
- How the Go Runtime Scheduler works (M:N scheduling)
- GOMAXPROCS and controlling parallelism
- Creating and managing goroutines
- Common gotchas and best practices
- Race conditions introduction

---

## 🧠 Concurrency vs Parallelism

### The Critical Difference

- **Concurrency** = "Dealing with many things at once" — structure, code organization
- **Parallelism** = "Doing many things at once" — execution, hardware

### Real-World Analogy: Coffee Shop

- **Concurrency (1 barista):** One person handles multiple orders by switching — take order A, start brewing, take order B, serve A when done, etc. Tasks interleave but don't run simultaneously.
- **Parallelism (3 baristas):** Three people each make different orders at the same time. Tasks run simultaneously on multiple CPUs.

### Key Insight

- Concurrency is about structure; parallelism is about execution
- You can have concurrency WITHOUT parallelism (single CPU)
- Go gives you concurrency; runtime decides parallelism

### In Programming Terms

- **Single core:** Tasks take turns (time-slicing). Only one task runs at any moment.
- **Multi-core:** Multiple tasks run simultaneously on different CPUs.

---

## 🔄 Processes vs Threads vs Goroutines

| | Process | Thread | Goroutine |
|--|---------|--------|-----------|
| Stack size | N/A | 1-8 MB | 2 KB (grows) |
| Creation time | ~100ms | ~100µs | ~0.3µs |
| Context switch | ~1000µs | ~1-10µs | ~0.2µs |
| Max practical | ~100s | ~10,000 | ~1,000,000+ |
| Managed by | OS | OS | Go Runtime |
| Scheduling | Preemptive | Preemptive | Cooperative* |

*Go 1.14+ added preemptive scheduling for goroutines.

**Goroutine advantages:** Lightweight stack, managed by Go runtime (not OS), multiplexed onto OS threads (M:N scheduling).

---

## ⚙️ Go Runtime Scheduler (GMP Model)

- **G** = Goroutine (the work to be done)
- **M** = Machine (OS thread that executes)
- **P** = Processor (scheduling context, holds runqueue)

**How it works:**
1. Goroutines (G) are created and added to queues
2. Processors (P) pick up Gs from their local queue
3. Machines (M) execute the G that P gives them
4. Number of Ps = GOMAXPROCS (default = CPU cores)
5. Work stealing: if P's local queue is empty, steal from other Ps

---

## 🔧 GOMAXPROCS

### What it controls

- Max number of OS threads running Go code simultaneously
- Default = number of CPU cores (since Go 1.5)
- `GOMAXPROCS(1)` = no parallelism (concurrency only)
- `GOMAXPROCS(4)` = up to 4 goroutines run in parallel

### When to change

- Usually don't — default is optimal
- Container limits: may need to match cgroup CPU limit
- Debugging: `GOMAXPROCS=1` to simplify race condition debugging

### Complete Example

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    fmt.Printf("NumCPU: %d\n", runtime.NumCPU())
    fmt.Printf("GOMAXPROCS: %d\n", runtime.GOMAXPROCS(0))
    fmt.Printf("NumGoroutine: %d\n", runtime.NumGoroutine())
}
```

**Output:**
```
NumCPU: 8
GOMAXPROCS: 8
NumGoroutine: 1
```

---

## 📝 Creating Goroutines

### Method 1: Named function

```go
go namedFunction(&wg)
```

### Method 2: Anonymous function

```go
go func() {
    defer wg.Done()
    fmt.Println("Hello from goroutine!")
}()
```

### Method 3: With parameters (IMPORTANT!)

```go
for i := 1; i <= 3; i++ {
    wg.Add(1)
    go func(n int) {  // n is a COPY of i
        defer wg.Done()
        fmt.Printf("Received: %d\n", n)
    }(i)  // Pass i here!
}
// Output: Received: 1, Received: 2, Received: 3
```

### Method 4: Method as goroutine

```go
worker := &Worker{ID: 1}
go worker.DoWork(&wg)
```

### Complete Example

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    wg.Add(3)
    for i := 1; i <= 3; i++ {
        go func(n int) {
            defer wg.Done()
            fmt.Printf("Goroutine received: %d\n", n)
        }(i)
    }
    wg.Wait()
}
```

**Output:**
```
Goroutine received: 1
Goroutine received: 2
Goroutine received: 3
```

*(Order may vary)*

---

## ⚠️ Goroutine Gotchas

### GOTCHA 1: Loop variable capture

```go
// BAD — all print same value
for _, v := range values {
    go func() {
        fmt.Println(v)  // v is SHARED!
    }()
}

// GOOD — pass as parameter
for _, v := range values {
    go func(val int) {
        fmt.Println(val)
    }(v)
}
```

### GOTCHA 2: Main exits = goroutines die

```go
func main() {
    go doWork()  // May NEVER run!
}  // main exits immediately

// FIX: Use WaitGroup, channels, or select{}
```

### GOTCHA 3: Goroutine leak

```go
ch := make(chan int)
go func() {
    val := <-ch  // Blocks forever if nothing sent!
}()
// Forgot to send or close ch — goroutine leaks!

// FIX: Always ensure goroutines can exit; use context for cancellation
```

### GOTCHA 4: Data race

```go
counter := 0
for i := 0; i < 1000; i++ {
    go func() { counter++ }()  // RACE!
}
// FIX: Use sync.Mutex, sync/atomic, or channels
// DETECT: go run -race program.go
```

### GOTCHA 5: Panic in goroutine crashes everything

```go
go func() {
    panic("oops")  // Crashes ENTIRE program!
}()

// FIX: Use recover in deferred function
go func() {
    defer func() {
        if r := recover(); r != nil {
            log.Printf("Recovered: %v", r)
        }
    }()
    // risky code here
}()
```

---

## 🔄 sync.WaitGroup

### Basic pattern

```go
var wg sync.WaitGroup
for i := 1; i <= 3; i++ {
    wg.Add(1)  // BEFORE launching goroutine!
    go func(id int) {
        defer wg.Done()  // ALWAYS use defer
        fmt.Printf("Worker %d\n", id)
    }(i)
}
wg.Wait()  // Block until counter = 0
```

### Common mistake

```go
// WRONG — race condition!
go func() {
    wg.Add(1)
    defer wg.Done()
}()

// RIGHT
wg.Add(1)
go func() {
    defer wg.Done()
}()
```

### Complete Example

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            fmt.Printf("Worker %d done\n", id)
        }(i)
    }
    wg.Wait()
    fmt.Println("All workers completed!")
}
```

**Output:**
```
Worker 1 done
Worker 2 done
Worker 3 done
All workers completed!
```

*(Order may vary)*

---

## 🏎️ Race Condition Detection

### What is a data race?

When two goroutines access the same memory, at least one is a write, and they're not synchronized.

### Race detector

```
go run -race program.go
go test -race ./...
go build -race program.go
```

### Fixing races

- **Mutex:** `mu.Lock()` / `mu.Unlock()` around critical section
- **Atomic:** `sync/atomic` for simple counters
- **Channels:** Single owner of data, communicate via channels

### Complete Example

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    counter := 0
    var mu sync.Mutex
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            mu.Lock()
            counter++
            mu.Unlock()
        }()
    }
    wg.Wait()
    fmt.Printf("Counter: %d\n", counter)
}
```

**Output:**
```
Counter: 1000
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `new Thread(() -> doWork()).start()` | `go func() { doWork() }()` |
| `ExecutorService pool = Executors.newFixedThreadPool(n)` | Just use goroutines — Go runtime handles it |
| `synchronized (lock) { ... }` | `mu.Lock()` / `mu.Unlock()` |
| `thread.join()` | `wg.Wait()` |
| 10,000 threads ≈ 80GB RAM, slow | 1,000,000 goroutines ≈ 2GB RAM, fast |
| `Thread.currentThread()` | `runtime.NumGoroutine()` |
| `Runtime.getRuntime().availableProcessors()` | `runtime.NumCPU()` |

---

## 🎯 Key Takeaways

1. **Concurrency ≠ Parallelism** — structure vs execution
2. **Goroutines are cheap** — 2KB stack, create millions
3. **GMP Model** — Go schedules G on M via P
4. **GOMAXPROCS** — controls max parallel execution (default = CPU cores)
5. **Always use WaitGroup** or channels to synchronize
6. **Pass loop variables** as parameters to avoid capture bugs
7. **Use `go run -race`** to detect data races
8. **Main exits = goroutines die** — ensure proper synchronization

---

## ➡️ Next Steps

**Next Topic:** [30 - Channels](./30-channels.md)
