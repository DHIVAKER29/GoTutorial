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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO SYNCHRONIZATION PRIMITIVES                                  │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  LOCKS (Mutual Exclusion)                                 │ │
│  │  ─────────────────────────                                │ │
│  │  sync.Mutex     - Exclusive access (one at a time)        │ │
│  │  sync.RWMutex   - Multiple readers OR one writer          │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  COORDINATION                                             │ │
│  │  ────────────                                             │ │
│  │  sync.WaitGroup - Wait for N goroutines to complete       │ │
│  │  sync.Once      - Execute something exactly once          │ │
│  │  sync.Cond      - Wait for/signal conditions              │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  SPECIALIZED                                              │ │
│  │  ───────────                                              │ │
│  │  sync.Map       - Concurrent map (specific use cases)     │ │
│  │  sync.Pool      - Reusable object pool                    │ │
│  │  sync/atomic    - Lock-free atomic operations             │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  WHEN TO USE WHAT:                                              │
│  • Simple critical section → Mutex                              │
│  • Read-heavy workload → RWMutex                                │
│  • Wait for goroutines → WaitGroup                              │
│  • Initialize once → Once                                       │
│  • Wait for condition → Cond (or channels)                      │
│  • Reduce allocations → Pool                                    │
│  • Simple counters → atomic                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔒 sync.Mutex - Mutual Exclusion Lock

### What is a Mutex?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MUTEX = "MUTual EXclusion"                                     │
│                                                                 │
│  Only ONE goroutine can hold the lock at a time.                │
│  Others must WAIT until the lock is released.                   │
│                                                                 │
│  Without Mutex:                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Goroutine A: read counter (10) ──────► write (11)      │   │
│  │  Goroutine B: read counter (10) ──────► write (11)      │   │
│  │                       ↑                                 │   │
│  │              Both read 10, both write 11!               │   │
│  │              Expected: 12, Got: 11 (LOST UPDATE!)       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  With Mutex:                                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Goroutine A: LOCK → read (10) → write (11) → UNLOCK    │   │
│  │  Goroutine B: wait... LOCK → read (11) → write (12)     │   │
│  │                       ↑                                 │   │
│  │              B waits for A to finish                    │   │
│  │              Expected: 12, Got: 12 ✓                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete Mutex Example

```go
// mutex_complete.go
package main

import (
    "fmt"
    "sync"
    "time"
)

// SafeCounter is safe to use from multiple goroutines
type SafeCounter struct {
    mu    sync.Mutex  // Guards the fields below
    value int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()  // ALWAYS use defer
    c.value++
}

func (c *SafeCounter) Decrement() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value--
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           SYNC.MUTEX - COMPLETE GUIDE                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic lock/unlock
    fmt.Println("\n📊 Basic Lock/Unlock:")
    var mu sync.Mutex
    
    mu.Lock()
    fmt.Println("   Lock acquired")
    // Critical section
    mu.Unlock()
    fmt.Println("   Lock released")
    
    // Safe counter
    fmt.Println("\n📊 Thread-Safe Counter:")
    counter := &SafeCounter{}
    var wg sync.WaitGroup
    
    // 1000 goroutines incrementing
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter.Increment()
        }()
    }
    wg.Wait()
    fmt.Printf("   Final value: %d (expected 1000)\n", counter.Value())
    
    // Mutex cannot be copied!
    fmt.Println("\n⚠️ Mutex CANNOT be Copied:")
    fmt.Println("   type Counter struct { mu sync.Mutex; val int }")
    fmt.Println("   c1 := Counter{}")
    fmt.Println("   c2 := c1  // BAD! Copies the mutex!")
    fmt.Println("   ")
    fmt.Println("   // Use pointer instead:")
    fmt.Println("   c2 := &c1  // Share the same counter")
    
    // TryLock (Go 1.18+)
    fmt.Println("\n📊 TryLock (Go 1.18+) - Non-blocking:")
    var mu2 sync.Mutex
    
    mu2.Lock()
    go func() {
        if mu2.TryLock() {
            fmt.Println("   Got the lock!")
            mu2.Unlock()
        } else {
            fmt.Println("   Lock is busy, doing something else...")
        }
    }()
    time.Sleep(10 * time.Millisecond)
    mu2.Unlock()
    
    // Best practices
    fmt.Println("\n💡 Best Practices:")
    fmt.Println("   1. Always use defer mu.Unlock()")
    fmt.Println("   2. Keep critical sections SHORT")
    fmt.Println("   3. Don't copy mutex (use pointers)")
    fmt.Println("   4. Document what the mutex protects")
    fmt.Println("   5. Avoid holding mutex during I/O")
}
```

---

## 📖 sync.RWMutex - Read-Write Lock

### When to Use RWMutex

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  RWMUTEX: MULTIPLE READERS OR ONE WRITER                        │
│                                                                 │
│  SCENARIO: Cache with many readers, few writers                 │
│                                                                 │
│  With Mutex:                                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Reader A: LOCK → read → UNLOCK                         │   │
│  │  Reader B: wait...                                      │   │
│  │  Reader C: wait...                                      │   │
│  │  Reader D: wait...                                      │   │
│  │                                                         │   │
│  │  Readers block each other! Inefficient!                 │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  With RWMutex:                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Reader A: RLOCK → read ────────────────► RUNLOCK       │   │
│  │  Reader B: RLOCK → read ────────────────► RUNLOCK       │   │
│  │  Reader C: RLOCK → read ────────────────► RUNLOCK       │   │
│  │  Reader D: RLOCK → read ────────────────► RUNLOCK       │   │
│  │                                                         │   │
│  │  All readers can proceed simultaneously!                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  When Writer comes:                                             │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Writer: LOCK → waits for all readers → write → UNLOCK  │   │
│  │  Readers: wait until writer is done                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  USE RWMUTEX WHEN:                                              │
│  • Reads are MUCH more frequent than writes                     │
│  • Read operations take significant time                        │
│  • Multiple concurrent reads are beneficial                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete RWMutex Example

```go
// rwmutex_complete.go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Cache with read-heavy access pattern
type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

func NewCache() *Cache {
    return &Cache{data: make(map[string]string)}
}

// Get uses read lock - multiple readers can proceed
func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    val, ok := c.data[key]
    return val, ok
}

// Set uses write lock - exclusive access
func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = value
}

// Keys uses read lock
func (c *Cache) Keys() []string {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    keys := make([]string, 0, len(c.data))
    for k := range c.data {
        keys = append(keys, k)
    }
    return keys
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           SYNC.RWMUTEX - READ-WRITE LOCK                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    cache := NewCache()
    var wg sync.WaitGroup
    
    // Writer
    wg.Add(1)
    go func() {
        defer wg.Done()
        for i := 0; i < 5; i++ {
            key := fmt.Sprintf("key%d", i)
            cache.Set(key, fmt.Sprintf("value%d", i))
            fmt.Printf("   Writer: set %s\n", key)
            time.Sleep(50 * time.Millisecond)
        }
    }()
    
    // Multiple readers
    for r := 0; r < 3; r++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            for i := 0; i < 10; i++ {
                keys := cache.Keys()
                fmt.Printf("   Reader %d: sees %d keys\n", id, len(keys))
                time.Sleep(20 * time.Millisecond)
            }
        }(r)
    }
    
    wg.Wait()
    
    // RWMutex methods
    fmt.Println("\n📊 RWMutex Methods:")
    fmt.Println("   mu.Lock()      - Acquire write lock (exclusive)")
    fmt.Println("   mu.Unlock()    - Release write lock")
    fmt.Println("   mu.RLock()     - Acquire read lock (shared)")
    fmt.Println("   mu.RUnlock()   - Release read lock")
    fmt.Println("   mu.TryLock()   - Try write lock (Go 1.18+)")
    fmt.Println("   mu.TryRLock()  - Try read lock (Go 1.18+)")
    
    // When NOT to use RWMutex
    fmt.Println("\n⚠️ When NOT to Use RWMutex:")
    fmt.Println("   • Write-heavy workloads (use Mutex)")
    fmt.Println("   • Very fast read operations (overhead not worth it)")
    fmt.Println("   • When in doubt, benchmark!")
}
```

---

## 🔔 sync.Cond - Condition Variables

### What is a Condition Variable?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CONDITION VARIABLE: "Wait until something happens"            │
│                                                                 │
│  SCENARIO: Producer-Consumer with limited buffer                │
│                                                                 │
│  Producer: "I have data, but buffer is full!"                   │
│            → Wait until consumer takes something                │
│                                                                 │
│  Consumer: "I want data, but buffer is empty!"                  │
│            → Wait until producer adds something                 │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Without Cond (busy waiting - BAD):                     │   │
│  │  for len(buffer) == 0 {                                 │   │
│  │      // Waste CPU checking repeatedly!                  │   │
│  │  }                                                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  With Cond (efficient waiting):                         │   │
│  │  cond.L.Lock()                                          │   │
│  │  for len(buffer) == 0 {                                 │   │
│  │      cond.Wait()  // Suspend, release lock, wait        │   │
│  │  }                                                      │   │
│  │  // Process item                                        │   │
│  │  cond.L.Unlock()                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete Cond Example

```go
// cond_complete.go
package main

import (
    "fmt"
    "sync"
    "time"
)

// BoundedQueue with condition variables
type BoundedQueue struct {
    mu       sync.Mutex
    notEmpty *sync.Cond
    notFull  *sync.Cond
    items    []int
    capacity int
}

func NewBoundedQueue(capacity int) *BoundedQueue {
    q := &BoundedQueue{
        items:    make([]int, 0, capacity),
        capacity: capacity,
    }
    q.notEmpty = sync.NewCond(&q.mu)
    q.notFull = sync.NewCond(&q.mu)
    return q
}

func (q *BoundedQueue) Put(item int) {
    q.mu.Lock()
    defer q.mu.Unlock()
    
    // Wait while queue is full
    for len(q.items) == q.capacity {
        q.notFull.Wait()  // Release lock, wait, reacquire lock
    }
    
    q.items = append(q.items, item)
    q.notEmpty.Signal()  // Wake one waiting consumer
}

func (q *BoundedQueue) Get() int {
    q.mu.Lock()
    defer q.mu.Unlock()
    
    // Wait while queue is empty
    for len(q.items) == 0 {
        q.notEmpty.Wait()
    }
    
    item := q.items[0]
    q.items = q.items[1:]
    q.notFull.Signal()  // Wake one waiting producer
    return item
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           SYNC.COND - CONDITION VARIABLES                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    queue := NewBoundedQueue(3)
    var wg sync.WaitGroup
    
    // Producer
    wg.Add(1)
    go func() {
        defer wg.Done()
        for i := 1; i <= 10; i++ {
            queue.Put(i)
            fmt.Printf("   Produced: %d\n", i)
            time.Sleep(20 * time.Millisecond)
        }
    }()
    
    // Consumer
    wg.Add(1)
    go func() {
        defer wg.Done()
        for i := 1; i <= 10; i++ {
            item := queue.Get()
            fmt.Printf("   Consumed: %d\n", item)
            time.Sleep(50 * time.Millisecond)
        }
    }()
    
    wg.Wait()
    
    // Cond methods
    fmt.Println("\n📊 Cond Methods:")
    fmt.Println("   cond := sync.NewCond(&mu)  - Create with existing lock")
    fmt.Println("   cond.Wait()    - Release lock, wait, reacquire lock")
    fmt.Println("   cond.Signal()  - Wake ONE waiting goroutine")
    fmt.Println("   cond.Broadcast() - Wake ALL waiting goroutines")
    
    // When to use
    fmt.Println("\n💡 When to Use Cond:")
    fmt.Println("   • Complex waiting conditions")
    fmt.Println("   • Multiple goroutines waiting on same condition")
    fmt.Println("   • Bounded buffers/queues")
    fmt.Println("   • Often better: use channels instead!")
}
```

---

## 🏊 sync.Pool - Object Pooling

### What is sync.Pool?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  SYNC.POOL: REDUCE ALLOCATION PRESSURE                          │
│                                                                 │
│  PROBLEM: Creating objects is expensive                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  for each request {                                     │   │
│  │      buffer := make([]byte, 64*1024)  // Allocate 64KB  │   │
│  │      processRequest(buffer)                             │   │
│  │      // buffer becomes garbage                          │   │
│  │  }                                                      │   │
│  │  // Lots of GC pressure!                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  SOLUTION: Reuse objects with Pool                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  for each request {                                     │   │
│  │      buffer := pool.Get().([]byte)  // Get from pool    │   │
│  │      processRequest(buffer)                             │   │
│  │      pool.Put(buffer)               // Return to pool   │   │
│  │  }                                                      │   │
│  │  // Objects reused, less GC!                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  IMPORTANT: Pool is NOT a cache!                                │
│  • Objects may be cleared at any GC                             │
│  • Don't rely on objects being in pool                          │
│  • Only use for temporary objects                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete Pool Example

```go
// pool_complete.go
package main

import (
    "bytes"
    "fmt"
    "sync"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           SYNC.POOL - OBJECT POOLING                      ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Buffer pool
    fmt.Println("\n📊 Buffer Pool Example:")
    bufferPool := &sync.Pool{
        New: func() interface{} {
            fmt.Println("   Creating new buffer")
            return new(bytes.Buffer)
        },
    }
    
    // First Get - creates new
    buf1 := bufferPool.Get().(*bytes.Buffer)
    buf1.WriteString("Hello")
    fmt.Printf("   buf1: %q\n", buf1.String())
    
    // Return to pool
    buf1.Reset()  // ALWAYS reset before returning!
    bufferPool.Put(buf1)
    fmt.Println("   Returned buf1 to pool")
    
    // Second Get - reuses existing
    buf2 := bufferPool.Get().(*bytes.Buffer)
    buf2.WriteString("World")
    fmt.Printf("   buf2: %q (reused!)\n", buf2.String())
    
    // Concurrent usage
    fmt.Println("\n📊 Concurrent Usage:")
    var wg sync.WaitGroup
    
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            buf := bufferPool.Get().(*bytes.Buffer)
            buf.Reset()
            buf.WriteString(fmt.Sprintf("Worker %d", id))
            
            // Do work...
            
            bufferPool.Put(buf)
        }(i)
    }
    wg.Wait()
    
    // Best practices
    fmt.Println("\n💡 Best Practices:")
    fmt.Println("   1. Always reset objects before Put()")
    fmt.Println("   2. Don't hold references after Put()")
    fmt.Println("   3. Only for truly temporary objects")
    fmt.Println("   4. Benchmark to verify benefit!")
    
    // Common use cases
    fmt.Println("\n📊 Common Use Cases:")
    fmt.Println("   • bytes.Buffer pools")
    fmt.Println("   • JSON encoder/decoder pools")
    fmt.Println("   • Large byte slice pools")
    fmt.Println("   • Connection wrapper pools")
}
```

---

## ⚛️ sync/atomic - Lock-Free Operations

### Complete Atomic Guide

```go
// atomic_complete.go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           SYNC/ATOMIC - LOCK-FREE OPERATIONS              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Atomic counter (int64)
    fmt.Println("\n📊 Atomic Counter (int64):")
    var counter int64
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            atomic.AddInt64(&counter, 1)  // Atomic increment
        }()
    }
    wg.Wait()
    fmt.Printf("   Counter: %d\n", atomic.LoadInt64(&counter))
    
    // All atomic operations
    fmt.Println("\n📊 Atomic Operations:")
    var val int64 = 10
    
    // Load - atomically read
    loaded := atomic.LoadInt64(&val)
    fmt.Printf("   Load: %d\n", loaded)
    
    // Store - atomically write
    atomic.StoreInt64(&val, 20)
    fmt.Printf("   Store(20): %d\n", val)
    
    // Add - atomically add (can subtract with negative)
    atomic.AddInt64(&val, 5)
    fmt.Printf("   Add(5): %d\n", val)
    
    atomic.AddInt64(&val, -3)
    fmt.Printf("   Add(-3): %d\n", val)
    
    // Swap - atomically replace and return old
    old := atomic.SwapInt64(&val, 100)
    fmt.Printf("   Swap(100): old=%d, new=%d\n", old, val)
    
    // CompareAndSwap - atomically compare and set
    swapped := atomic.CompareAndSwapInt64(&val, 100, 200)
    fmt.Printf("   CAS(100→200): success=%t, val=%d\n", swapped, val)
    
    swapped = atomic.CompareAndSwapInt64(&val, 100, 300)
    fmt.Printf("   CAS(100→300): success=%t, val=%d (not 100!)\n", swapped, val)
    
    // Atomic boolean (using int32)
    fmt.Println("\n📊 Atomic Boolean:")
    var flag int32
    
    // Set true
    atomic.StoreInt32(&flag, 1)
    if atomic.LoadInt32(&flag) == 1 {
        fmt.Println("   Flag is true")
    }
    
    // atomic.Value for any type
    fmt.Println("\n📊 atomic.Value (any type):")
    var config atomic.Value
    
    type Config struct {
        Server string
        Port   int
    }
    
    // Store
    config.Store(&Config{Server: "localhost", Port: 8080})
    
    // Load
    cfg := config.Load().(*Config)
    fmt.Printf("   Config: %+v\n", cfg)
    
    // Swap (Go 1.17+)
    oldCfg := config.Swap(&Config{Server: "127.0.0.1", Port: 9090})
    fmt.Printf("   Old config: %+v\n", oldCfg.(*Config))
    fmt.Printf("   New config: %+v\n", config.Load().(*Config))
    
    // When to use atomic
    fmt.Println("\n💡 When to Use Atomic:")
    fmt.Println("   ✅ Simple counters")
    fmt.Println("   ✅ Flags (bool as int32)")
    fmt.Println("   ✅ Lock-free data structures")
    fmt.Println("   ❌ Complex invariants (use Mutex)")
    fmt.Println("   ❌ Multiple related variables (use Mutex)")
}
```

---

## 💀 Deadlocks

### Understanding Deadlocks

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  DEADLOCK: Two or more goroutines waiting for each other        │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  Goroutine A:           Goroutine B:                    │   │
│  │  1. Lock(mu1)           1. Lock(mu2)                    │   │
│  │  2. Lock(mu2) ← WAIT    2. Lock(mu1) ← WAIT             │   │
│  │         │                      │                        │   │
│  │         └──────────────────────┘                        │   │
│  │              Waiting for each other!                    │   │
│  │              Neither can proceed!                       │   │
│  │              DEADLOCK!                                  │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  FOUR CONDITIONS FOR DEADLOCK (all must be true):               │
│  1. Mutual exclusion - resources can't be shared                │
│  2. Hold and wait - holding one, waiting for another            │
│  3. No preemption - can't force release                         │
│  4. Circular wait - A waits for B, B waits for A                │
│                                                                 │
│  PREVENT BY BREAKING ANY CONDITION:                             │
│  • Always acquire locks in same ORDER                           │
│  • Use timeouts (TryLock)                                       │
│  • Avoid nested locks when possible                             │
│  • Use channels instead of locks                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Deadlock Prevention

```go
// deadlock_prevention.go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           DEADLOCK PREVENTION                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Example of deadlock-prone code (DON'T RUN - will deadlock!)
    fmt.Println("\n❌ Deadlock Example (conceptual):")
    fmt.Println("   var mu1, mu2 sync.Mutex")
    fmt.Println("   go func() { mu1.Lock(); mu2.Lock(); ... }()")
    fmt.Println("   go func() { mu2.Lock(); mu1.Lock(); ... }()")
    
    // Prevention 1: Lock ordering
    fmt.Println("\n✅ Prevention 1: Consistent Lock Ordering")
    var mu1, mu2 sync.Mutex
    
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
    
    time.Sleep(100 * time.Millisecond)
    fmt.Println("   Always lock in same order: mu1 → mu2")
    
    // Prevention 2: TryLock with backoff
    fmt.Println("\n✅ Prevention 2: TryLock with Backoff")
    fmt.Println("   if !mu.TryLock() {")
    fmt.Println("       time.Sleep(backoff)")
    fmt.Println("       continue  // Retry")
    fmt.Println("   }")
    
    // Prevention 3: Avoid holding locks during I/O
    fmt.Println("\n✅ Prevention 3: Minimize Lock Scope")
    fmt.Println("   mu.Lock()")
    fmt.Println("   data := copy(sharedData)  // Copy")
    fmt.Println("   mu.Unlock()")
    fmt.Println("   processData(data)  // I/O outside lock!")
    
    // Prevention 4: Use channels
    fmt.Println("\n✅ Prevention 4: Use Channels Instead")
    fmt.Println("   ch <- request  // Send to worker goroutine")
    fmt.Println("   result := <-resultCh  // Wait for result")
    fmt.Println("   // No locks needed!")
    
    // Go's deadlock detector
    fmt.Println("\n📊 Go's Deadlock Detector:")
    fmt.Println("   Go detects when ALL goroutines are blocked")
    fmt.Println("   Prints 'fatal error: all goroutines are asleep - deadlock!'")
    fmt.Println("   BUT: doesn't detect partial deadlocks")
}
```

---

## 🆚 Choosing the Right Primitive

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  DECISION TREE: WHICH SYNCHRONIZATION TO USE?                   │
│                                                                 │
│  Need to protect shared state?                                  │
│  │                                                              │
│  ├─► Simple counter → atomic.AddInt64()                         │
│  │                                                              │
│  ├─► Complex state with mostly reads → sync.RWMutex             │
│  │                                                              │
│  ├─► Complex state with balanced read/write → sync.Mutex        │
│  │                                                              │
│  └─► Coordinate goroutines → channels                           │
│                                                                 │
│  Need to wait for goroutines?                                   │
│  │                                                              │
│  ├─► Wait for N to complete → sync.WaitGroup                    │
│  │                                                              │
│  ├─► Wait for condition → sync.Cond or channels                 │
│  │                                                              │
│  └─► One-time setup → sync.Once                                 │
│                                                                 │
│  Need to reduce allocations? → sync.Pool                        │
│                                                                 │
│  RULE OF THUMB:                                                 │
│  "When in doubt, use channels"                                  │
│  "Share memory by communicating, don't communicate by sharing"  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

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
