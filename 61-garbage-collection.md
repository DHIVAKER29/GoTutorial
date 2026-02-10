# 61 - Garbage Collection

> Understanding Go's garbage collector and how to work with it.

---

## 📌 What You'll Learn

- How Go's GC works
- GC phases and pauses
- GOGC environment variable
- Memory management tips
- Reducing GC pressure

---

## 🗑️ How Go's GC Works

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO'S GARBAGE COLLECTOR                                         │
│                                                                 │
│  TYPE: Concurrent, Tri-color, Mark-and-Sweep                    │
│                                                                 │
│  GOALS:                                                         │
│  • Low latency (short pauses)                                   │
│  • Concurrent with application                                  │
│  • Simple and predictable                                       │
│                                                                 │
│  PHASES:                                                        │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  1. MARK SETUP (STW ~10-30μs)                             │ │
│  │     Stop-the-world, enable write barrier                  │ │
│  │                                                           │ │
│  │  2. MARKING (Concurrent)                                  │ │
│  │     Mark live objects while app runs                      │ │
│  │     Uses ~25% CPU                                         │ │
│  │                                                           │ │
│  │  3. MARK TERMINATION (STW ~10-30μs)                       │ │
│  │     Stop-the-world, finish marking, disable barrier       │ │
│  │                                                           │ │
│  │  4. SWEEPING (Concurrent)                                 │ │
│  │     Reclaim unmarked memory while app runs                │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  STW = Stop-The-World (all goroutines paused)                   │
│  Modern Go: STW pauses typically < 1ms                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎨 Tri-Color Algorithm

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  TRI-COLOR MARKING                                              │
│                                                                 │
│  ⚪ WHITE: Not yet visited (potentially garbage)                │
│  ⬛ BLACK: Visited and all children visited (definitely alive)  │
│  🔘 GRAY:  Visited but children not yet visited                 │
│                                                                 │
│  ALGORITHM:                                                     │
│  1. Start: All objects WHITE                                    │
│  2. Mark roots (stack, globals) as GRAY                         │
│  3. Loop:                                                       │
│     • Pick GRAY object                                          │
│     • Mark its references GRAY                                  │
│     • Mark the object BLACK                                     │
│  4. When no GRAY left:                                          │
│     • BLACK = alive                                             │
│     • WHITE = garbage (sweep)                                   │
│                                                                 │
│  ┌─────┐      ┌─────┐      ┌─────┐                             │
│  │Root │─────►│  A  │─────►│  B  │                             │
│  └─────┘      └──┬──┘      └─────┘                             │
│                  │                                              │
│                  ▼                                              │
│               ┌─────┐                                           │
│               │  C  │                                           │
│               └─────┘                                           │
│                                                                 │
│  If reachable from root → BLACK (alive)                         │
│  If not reachable → WHITE (garbage)                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Controlling GC

```go
// gc_control.go
package main

import (
    "fmt"
    "runtime"
    "runtime/debug"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Controlling GC                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Get current memory stats
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    fmt.Println("\n📊 Memory Stats:")
    fmt.Printf("   Alloc:      %d MB (currently allocated)\n", m.Alloc/1024/1024)
    fmt.Printf("   TotalAlloc: %d MB (total allocated ever)\n", m.TotalAlloc/1024/1024)
    fmt.Printf("   Sys:        %d MB (obtained from OS)\n", m.Sys/1024/1024)
    fmt.Printf("   NumGC:      %d (number of GC cycles)\n", m.NumGC)
    
    // Force GC (usually not recommended)
    fmt.Println("\n📊 Force GC:")
    runtime.GC()
    fmt.Println("   GC triggered manually")
    
    // GOGC setting (default 100)
    fmt.Println("\n📊 GOGC:")
    fmt.Printf("   Current: %d%%\n", debug.SetGCPercent(-1))
    debug.SetGCPercent(100)  // Reset to default
    fmt.Println("   GOGC=100 means: GC when heap doubles")
    fmt.Println("   GOGC=50  means: GC when heap grows 50%")
    fmt.Println("   GOGC=200 means: GC when heap triples")
    
    // Memory limit (Go 1.19+)
    fmt.Println("\n📊 Memory Limit (Go 1.19+):")
    fmt.Println("   debug.SetMemoryLimit(1 << 30)  // 1GB limit")
    
    // Free OS memory
    fmt.Println("\n📊 Free Memory to OS:")
    debug.FreeOSMemory()
    fmt.Println("   Released unused memory to OS")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Controlling GC                                  ║
╚══════════════════════════════════════════════════════════╝

📊 Memory Stats:
   Alloc:      0 MB (currently allocated)
   TotalAlloc: 0 MB (total allocated ever)
   Sys:        0 MB (obtained from OS)
   NumGC:      0 (number of GC cycles)

📊 Force GC:
   GC triggered manually

📊 GOGC:
   Current: 100%
   GOGC=100 means: GC when heap doubles
   GOGC=50  means: GC when heap grows 50%
   GOGC=200 means: GC when heap triples

📊 Memory Limit (Go 1.19+):
   debug.SetMemoryLimit(1 << 30)  // 1GB limit

📊 Free Memory to OS:
   Released unused memory to OS
```

*Note: Actual memory values may vary based on runtime state.*

```bash
# Environment variables

# GOGC: GC trigger threshold (default 100)
GOGC=50 ./myapp   # More frequent GC
GOGC=200 ./myapp  # Less frequent GC
GOGC=off ./myapp  # Disable GC (dangerous!)

# GOMEMLIMIT (Go 1.19+): Soft memory limit
GOMEMLIMIT=1GiB ./myapp

# GODEBUG: GC tracing
GODEBUG=gctrace=1 ./myapp
```

---

## 📊 Reading GC Trace

```bash
$ GODEBUG=gctrace=1 ./myapp

gc 1 @0.012s 2%: 0.026+0.23+0.004 ms clock, 0.21+0.12/0.31/0.52+0.036 ms cpu, 4->4->0 MB, 5 MB goal, 8 P
│    │       │   │                │                                      │         │         │
│    │       │   │                │                                      │         │         └─ # of processors
│    │       │   │                │                                      │         └─ heap goal
│    │       │   │                │                                      └─ heap: before->after->live
│    │       │   │                └─ CPU time (assist/background/idle)
│    │       │   └─ Wall clock time (STW1 + concurrent + STW2)
│    │       └─ % of time in GC
│    └─ Elapsed since start
└─ GC cycle number
```

---

## 💡 Reducing GC Pressure

```go
// reduce_gc.go
package main

import (
    "sync"
)

// TIP 1: Reuse objects with sync.Pool
var bufferPool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 1024)
    },
}

func ProcessWithPool() {
    buf := bufferPool.Get().([]byte)
    defer bufferPool.Put(buf)
    
    // Use buf...
}

// TIP 2: Pre-allocate slices
func GoodSlice(n int) []int {
    result := make([]int, 0, n)  // Pre-allocate!
    for i := 0; i < n; i++ {
        result = append(result, i)
    }
    return result
}

// TIP 3: Avoid unnecessary pointers
type Good struct {
    Data [100]byte  // Embedded, one allocation
}

type Bad struct {
    Data *[100]byte  // Pointer, extra allocation
}

// TIP 4: Use value types for small data
func ByValue(x int) int {
    return x * 2  // No allocation
}

func ByPointer(x *int) *int {
    result := *x * 2  // Allocation!
    return &result
}

// TIP 5: Batch allocations
func BatchAlloc() {
    // Bad: 1000 small allocations
    // for i := 0; i < 1000; i++ {
    //     data := make([]byte, 100)
    //     process(data)
    // }
    
    // Good: 1 large allocation
    allData := make([]byte, 100*1000)
    for i := 0; i < 1000; i++ {
        data := allData[i*100 : (i+1)*100]
        process(data)
    }
}

func process(data []byte) {}

func main() {}
```

---

## 🎯 Key Takeaways

1. **Go's GC** is concurrent with low pause times
2. **Tri-color marking** identifies live objects
3. **STW pauses** are typically < 1ms
4. **GOGC** controls how often GC runs
5. **sync.Pool** reduces allocations
6. **Pre-allocate** slices and maps
7. **Avoid unnecessary pointers** to reduce GC work
8. **Profile first** - don't optimize blindly

---

## ➡️ Next Steps

**Next Topic:** [62 - Interface Internals](./62-interface-internals.md)

