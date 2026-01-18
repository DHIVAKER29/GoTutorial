# 47 - Profiling and Performance

> Measuring and optimizing Go application performance.

---

## 📌 What You'll Learn

- CPU profiling
- Memory profiling
- Trace analysis
- Benchmark-driven optimization
- Common performance tips

---

## 📊 pprof Basics

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO PROFILING TOOLS                                             │
│                                                                 │
│  pprof:                                                         │
│  • CPU profiling (where time is spent)                          │
│  • Memory profiling (allocations)                               │
│  • Goroutine profiling (blocking)                               │
│                                                                 │
│  trace:                                                         │
│  • Execution tracer (goroutine scheduling)                      │
│  • GC events                                                    │
│  • System calls                                                 │
│                                                                 │
│  PROFILING WORKFLOW:                                            │
│  1. Collect profile data                                        │
│  2. Analyze with go tool pprof                                  │
│  3. Identify hotspots                                           │
│  4. Optimize                                                    │
│  5. Measure again (verify improvement)                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔥 CPU Profiling

```go
// cpu_profile.go
package main

import (
    "fmt"
    "os"
    "runtime/pprof"
    "time"
)

func heavyComputation() {
    result := 0
    for i := 0; i < 100000000; i++ {
        result += i * i
    }
    _ = result
}

func main() {
    // Create CPU profile file
    f, err := os.Create("cpu.prof")
    if err != nil {
        panic(err)
    }
    defer f.Close()
    
    // Start CPU profiling
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()
    
    // Your application code
    fmt.Println("Running heavy computation...")
    start := time.Now()
    heavyComputation()
    fmt.Printf("Done in %v\n", time.Since(start))
}

/*
Run and analyze:
    go run cpu_profile.go
    go tool pprof cpu.prof
    
pprof commands:
    top        - Show top functions by CPU time
    top10      - Top 10 functions
    list func  - Show source code for function
    web        - Open interactive graph in browser
    svg        - Generate SVG graph
*/
```

---

## 🧠 Memory Profiling

```go
// memory_profile.go
package main

import (
    "os"
    "runtime"
    "runtime/pprof"
)

func allocateMemory() [][]byte {
    slices := make([][]byte, 1000)
    for i := range slices {
        slices[i] = make([]byte, 10000)  // 10KB each
    }
    return slices
}

func main() {
    // Allocate memory
    data := allocateMemory()
    
    // Force GC
    runtime.GC()
    
    // Create heap profile
    f, _ := os.Create("mem.prof")
    defer f.Close()
    
    pprof.WriteHeapProfile(f)
    
    // Keep data alive
    _ = data
}

/*
Analyze:
    go tool pprof mem.prof
    
Commands:
    top        - Top memory consumers
    allocs     - Show allocations
    inuse      - Show in-use memory
*/
```

---

## 🌐 HTTP Server Profiling

```go
// http_pprof.go
package main

import (
    "fmt"
    "net/http"
    _ "net/http/pprof"  // Import for side effects!
)

func main() {
    // pprof endpoints automatically registered:
    // /debug/pprof/
    // /debug/pprof/cmdline
    // /debug/pprof/profile
    // /debug/pprof/symbol
    // /debug/pprof/trace
    // /debug/pprof/heap
    // /debug/pprof/goroutine
    
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello!")
    })
    
    fmt.Println("Server running on :8080")
    fmt.Println("pprof at http://localhost:8080/debug/pprof/")
    http.ListenAndServe(":8080", nil)
}

/*
Profile running server:
    # CPU profile (30 seconds)
    go tool pprof http://localhost:8080/debug/pprof/profile?seconds=30
    
    # Heap profile
    go tool pprof http://localhost:8080/debug/pprof/heap
    
    # Goroutine profile
    go tool pprof http://localhost:8080/debug/pprof/goroutine
*/
```

---

## 📈 Benchmark Profiling

```go
// bench_test.go
package main

import "testing"

func BenchmarkStringConcat(b *testing.B) {
    for i := 0; i < b.N; i++ {
        s := ""
        for j := 0; j < 100; j++ {
            s += "x"
        }
    }
}

/*
Run with profiling:
    go test -bench=. -cpuprofile=cpu.prof
    go test -bench=. -memprofile=mem.prof
    go test -bench=. -benchmem
    
Analyze:
    go tool pprof cpu.prof
*/
```

---

## ⚡ Performance Tips

```go
// performance_tips.go
package main

import (
    "bytes"
    "fmt"
    "strings"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PERFORMANCE TIPS                                ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // 1. String concatenation
    fmt.Println("\n📊 String Concatenation:")
    fmt.Println("   ❌ Slow: s += \"x\" (creates new string each time)")
    fmt.Println("   ✅ Fast: strings.Builder or bytes.Buffer")
    
    var builder strings.Builder
    for i := 0; i < 100; i++ {
        builder.WriteString("x")
    }
    _ = builder.String()
    
    // 2. Pre-allocate slices
    fmt.Println("\n📊 Slice Pre-allocation:")
    fmt.Println("   ❌ Slow: append grows slice multiple times")
    fmt.Println("   ✅ Fast: make([]T, 0, expectedSize)")
    
    data := make([]int, 0, 1000)  // Pre-allocate capacity
    for i := 0; i < 1000; i++ {
        data = append(data, i)
    }
    
    // 3. Avoid interface{} when possible
    fmt.Println("\n📊 Avoid interface{}:")
    fmt.Println("   ❌ Slow: func process(v interface{})")
    fmt.Println("   ✅ Fast: func process(v int) or generics")
    
    // 4. sync.Pool for frequently allocated objects
    fmt.Println("\n📊 sync.Pool:")
    fmt.Println("   Use for frequently allocated/deallocated objects")
    fmt.Println("   Reduces GC pressure")
    
    // 5. Avoid copying large structs
    fmt.Println("\n📊 Pointer vs Value:")
    fmt.Println("   Large structs: pass by pointer")
    fmt.Println("   Small structs (<64 bytes): pass by value")
    
    // 6. Use buffered I/O
    fmt.Println("\n📊 Buffered I/O:")
    fmt.Println("   Use bufio.Reader/Writer for file/network I/O")
    
    _ = bytes.Buffer{}  // Suppress unused warning
}
```

---

## 🎯 Key Takeaways

1. **pprof** for CPU and memory profiling
2. **net/http/pprof** for HTTP servers
3. **go tool pprof** to analyze profiles
4. **Benchmark before optimizing**
5. **Pre-allocate** slices and maps
6. **strings.Builder** for string concatenation
7. **sync.Pool** for object reuse

---

## ➡️ Next Steps

**Next Topic:** [48 - Build Constraints](./48-build-constraints.md)

