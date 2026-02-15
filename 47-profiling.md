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

| Tool | Purpose |
|------|---------|
| **pprof** | CPU, memory, goroutine profiling |
| **trace** | Execution tracer, GC events, system calls |

**Workflow:** 1. Collect profile → 2. Analyze with `go tool pprof` → 3. Identify hotspots → 4. Optimize → 5. Measure again

---

## 🔥 CPU Profiling

```go
// cpu_profile.go
package main

import (
    "os"
    "runtime/pprof"
)

func heavyComputation() {
    result := 0
    for i := 0; i < 100000000; i++ {
        result += i * i
    }
    _ = result
}

func main() {
    f, _ := os.Create("cpu.prof")
    defer f.Close()
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()

    heavyComputation()
}
// Output: Creates cpu.prof file
```

```bash
go run cpu_profile.go
go tool pprof cpu.prof
# Commands: top, top10, list func, web, svg
```

---

## 🧠 Memory Profiling

```go
// memory_profile.go
func allocateMemory() [][]byte {
    slices := make([][]byte, 1000)
    for i := range slices {
        slices[i] = make([]byte, 10000)
    }
    return slices
}

func main() {
    data := allocateMemory()
    runtime.GC()
    f, _ := os.Create("mem.prof")
    defer f.Close()
    pprof.WriteHeapProfile(f)
    _ = data
}
// Output: Creates mem.prof file
```

```bash
go tool pprof mem.prof
# Commands: top, allocs, inuse
```

---

## 🌐 HTTP Server Profiling

```go
import _ "net/http/pprof"

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello!")
    })
    http.ListenAndServe(":8080", nil)
}
// pprof endpoints: /debug/pprof/, /debug/pprof/profile, /debug/pprof/heap, etc.
```

```bash
go tool pprof http://localhost:8080/debug/pprof/profile?seconds=30
go tool pprof http://localhost:8080/debug/pprof/heap
```

---

## 📈 Benchmark Profiling

```go
func BenchmarkStringConcat(b *testing.B) {
    for i := 0; i < b.N; i++ {
        s := ""
        for j := 0; j < 100; j++ {
            s += "x"
        }
    }
}
```

```bash
go test -bench=. -cpuprofile=cpu.prof
go test -bench=. -memprofile=mem.prof
go test -bench=. -benchmem
```

---

## ⚡ Performance Tips

| Tip | Avoid | Prefer |
|-----|-------|--------|
| String concat | `s += "x"` | `strings.Builder` or `bytes.Buffer` |
| Slices | `append` without pre-alloc | `make([]T, 0, expectedSize)` |
| Types | `func process(v interface{})` | `func process(v int)` or generics |
| Objects | Frequent alloc/dealloc | `sync.Pool` |
| Structs | Large by value | Pass by pointer |
| I/O | Unbuffered | `bufio.Reader/Writer` |

```go
// strings.Builder example
var builder strings.Builder
for i := 0; i < 100; i++ {
    builder.WriteString("x")
}
result := builder.String()
// Output: Efficient concatenation
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
