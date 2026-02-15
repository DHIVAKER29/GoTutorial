# 32 - Sync Package - Complete Guide

> All synchronization primitives in Go - locks, conditions, pools, and atomic operations.

---

## 📌 What You'll Learn

- All types of locks and when to use each
- sync.Mutex (mutual exclusion)
- sync.RWMutex (read-write lock)
- sync.WaitGroup (waiting for completion)
- sync.Once (one-time initialization)
- sync.Cond (condition variables)
- sync.Pool (object pooling)
- sync.Map (concurrent map)
- sync/atomic (lock-free operations)
- Deadlocks and how to avoid them

---

## 🔒 Overview of Synchronization Primitives

| Category | Primitive | Use Case |
|----------|-----------|----------|
| **Locks** | sync.Mutex | Exclusive access (one at a time) |
| | sync.RWMutex | Multiple readers OR one writer |
| **Coordination** | sync.WaitGroup | Wait for N goroutines to complete |
| | sync.Once | Execute something exactly once |
| | sync.Cond | Wait for/signal conditions |
| **Specialized** | sync.Map | Concurrent map (specific use cases) |
| | sync.Pool | Reusable object pool |
| | sync/atomic | Lock-free atomic operations |

**When to use what:** Simple critical section → Mutex. Read-heavy → RWMutex. Wait for goroutines → WaitGroup. Initialize once → Once. Wait for condition → Cond or channels. Reduce allocations → Pool. Simple counters → atomic.

---

## 🔒 sync.Mutex - Mutual Exclusion Lock

### What is a Mutex?

- **Mutex = Mutual Exclusion** — only ONE goroutine holds the lock at a time
- Without Mutex: race condition — two goroutines read same value, both write, one update is lost
- With Mutex: one goroutine locks, does read-modify-write, unlocks; others wait

### Basic Lock/Unlock

```go
var mu sync.Mutex
mu.Lock()
// Critical section
mu.Unlock()
// Output: (no output, demonstrates pattern)
```

### TryLock (Go 1.18+)

```go
var mu sync.Mutex
mu.Lock()
go func() {
    if mu.TryLock() {
        fmt.Println("Got the lock!")
        mu.Unlock()
    } else {
        fmt.Println("Lock is busy")
    }
}()
time.Sleep(10 * time.Millisecond)
mu.Unlock()
// Output: Lock is busy
```

### Complete Example: Safe Counter

```go
package main

import (
    "fmt"
    "sync"
)

type SafeCounter struct {
    mu    sync.Mutex
    value int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

func main() {
    counter := &SafeCounter{}
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter.Increment()
        }()
    }
    wg.Wait()
    fmt.Println("Final value:", counter.Value())
}
```

**Output:**
```
Final value: 1000
```

**Best practices:** Always use `defer mu.Unlock()`. Keep critical sections short. Don't copy mutex (use pointers). Avoid holding mutex during I/O.

---

## 📖 sync.RWMutex - Read-Write Lock

### When to Use

- **RWMutex:** Multiple readers OR one writer
- With Mutex: readers block each other
- With RWMutex: all readers proceed simultaneously; writer waits for readers
- Use when reads are much more frequent than writes

### RWMutex Methods

| Method | Purpose |
|--------|---------|
| `Lock()` / `Unlock()` | Write lock (exclusive) |
| `RLock()` / `RUnlock()` | Read lock (shared) |
| `TryLock()` / `TryRLock()` | Non-blocking (Go 1.18+) |

### Complete Example: Cache with RWMutex

```go
package main

import (
    "fmt"
    "sync"
)

type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    val, ok := c.data[key]
    return val, ok
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = value
}

func main() {
    cache := &Cache{data: make(map[string]string)}
    cache.Set("a", "1")
    cache.Set("b", "2")
    if v, ok := cache.Get("a"); ok {
        fmt.Println("a =", v)
    }
}
```

**Output:**
```
a = 1
```

---

## 🔔 sync.Cond - Condition Variables

### What is Cond?

- **Condition variable:** "Wait until something happens"
- Without Cond: busy waiting wastes CPU
- With Cond: `cond.Wait()` suspends, releases lock, waits; wakes when signaled

### Cond Methods

| Method | Purpose |
|--------|---------|
| `sync.NewCond(&mu)` | Create with existing lock |
| `cond.Wait()` | Release lock, wait, reacquire |
| `cond.Signal()` | Wake one goroutine |
| `cond.Broadcast()` | Wake all goroutines |

### Complete Example: Bounded Queue

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type BoundedQueue struct {
    mu       sync.Mutex
    notEmpty *sync.Cond
    notFull  *sync.Cond
    items    []int
    capacity int
}

func NewBoundedQueue(cap int) *BoundedQueue {
    q := &BoundedQueue{items: make([]int, 0, cap), capacity: cap}
    q.notEmpty = sync.NewCond(&q.mu)
    q.notFull = sync.NewCond(&q.mu)
    return q
}

func (q *BoundedQueue) Put(item int) {
    q.mu.Lock()
    defer q.mu.Unlock()
    for len(q.items) == q.capacity {
        q.notFull.Wait()
    }
    q.items = append(q.items, item)
    q.notEmpty.Signal()
}

func (q *BoundedQueue) Get() int {
    q.mu.Lock()
    defer q.mu.Unlock()
    for len(q.items) == 0 {
        q.notEmpty.Wait()
    }
    item := q.items[0]
    q.items = q.items[1:]
    q.notFull.Signal()
    return item
}

func main() {
    q := NewBoundedQueue(3)
    go func() {
        for i := 1; i <= 5; i++ {
            q.Put(i)
            fmt.Println("Produced:", i)
        }
    }()
    time.Sleep(100 * time.Millisecond)
    for i := 0; i < 5; i++ {
        fmt.Println("Consumed:", q.Get())
    }
}
```

**Output:**
```
Produced: 1
Produced: 2
Produced: 3
Consumed: 1
...
```

---

## 🏊 sync.Pool - Object Pooling

### What is sync.Pool?

- Reduces allocation pressure by reusing objects
- **Important:** Pool is NOT a cache — objects may be cleared at GC
- Only use for temporary objects

### Basic Usage

```go
pool := &sync.Pool{
    New: func() interface{} { return new(bytes.Buffer) },
}
buf := pool.Get().(*bytes.Buffer)
buf.Reset()
buf.WriteString("Hello")
pool.Put(buf)  // Always reset before Put!
```

### Complete Example: Buffer Pool

```go
package main

import (
    "bytes"
    "fmt"
    "sync"
)

func main() {
    pool := &sync.Pool{
        New: func() interface{} {
            fmt.Println("Creating new buffer")
            return new(bytes.Buffer)
        },
    }
    buf1 := pool.Get().(*bytes.Buffer)
    buf1.WriteString("Hello")
    fmt.Println("buf1:", buf1.String())
    buf1.Reset()
    pool.Put(buf1)

    buf2 := pool.Get().(*bytes.Buffer)
    buf2.WriteString("World")
    fmt.Println("buf2:", buf2.String())
}
```

**Output:**
```
Creating new buffer
buf1: Hello
buf2: World
```

---

## ⚛️ sync/atomic - Lock-Free Operations

### Atomic Operations

| Operation | Purpose |
|-----------|---------|
| `LoadInt64(&v)` | Atomically read |
| `StoreInt64(&v, x)` | Atomically write |
| `AddInt64(&v, n)` | Atomically add |
| `SwapInt64(&v, x)` | Replace, return old |
| `CompareAndSwapInt64(&v, old, new)` | Set if current == old |

### Complete Example: Atomic Counter

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    var counter int64
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            atomic.AddInt64(&counter, 1)
        }()
    }
    wg.Wait()
    fmt.Println("Counter:", atomic.LoadInt64(&counter))

    var val int64 = 10
    old := atomic.SwapInt64(&val, 100)
    fmt.Println("Swap: old=", old, "new=", val)
    swapped := atomic.CompareAndSwapInt64(&val, 100, 200)
    fmt.Println("CAS success:", swapped, "val:", val)
}
```

**Output:**
```
Counter: 1000
Swap: old= 10 new= 100
CAS success: true val: 200
```

**When to use:** Simple counters, flags, lock-free structures. Not for complex invariants or multiple related variables.

---

## 💀 Deadlocks

### Understanding Deadlocks

- **Deadlock:** Two or more goroutines waiting for each other
- Example: Goroutine A holds mu1, waits for mu2; Goroutine B holds mu2, waits for mu1
- **Prevention:** Consistent lock ordering, TryLock with backoff, minimize lock scope, use channels

### Prevention: Lock Ordering

```go
// Always lock in same order: mu1 then mu2
go func() {
    mu1.Lock()
    mu2.Lock()
    // work
    mu2.Unlock()
    mu1.Unlock()
}()
go func() {
    mu1.Lock()  // Same order!
    mu2.Lock()
    // work
    mu2.Unlock()
    mu1.Unlock()
}()
```

### Go's Deadlock Detector

- Detects when ALL goroutines are blocked
- Prints: `fatal error: all goroutines are asleep - deadlock!`
- Does not detect partial deadlocks

---

## 🆚 Choosing the Right Primitive

| Need | Use |
|------|-----|
| Simple counter | atomic.AddInt64() |
| Complex state, mostly reads | sync.RWMutex |
| Complex state, balanced R/W | sync.Mutex |
| Coordinate goroutines | channels |
| Wait for N goroutines | sync.WaitGroup |
| Wait for condition | sync.Cond or channels |
| One-time setup | sync.Once |
| Reduce allocations | sync.Pool |

**Rule of thumb:** "When in doubt, use channels." "Share memory by communicating."

---

## 🎯 Key Takeaways

1. **Mutex** - exclusive access, use for most cases
2. **RWMutex** - multiple readers OR one writer
3. **WaitGroup** - wait for goroutines to finish
4. **Once** - lazy initialization, singleton
5. **Cond** - wait for conditions (prefer channels usually)
6. **Pool** - reduce GC pressure with object reuse
7. **atomic** - fast, lock-free for simple values
8. **Prevent deadlocks** - consistent lock ordering
9. **Always defer Unlock()** - never forget to unlock!

---

## ➡️ Next Steps

**Next Topic:** [33 - Context](./33-context.md)
