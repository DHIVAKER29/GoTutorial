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

**Type:** Concurrent, tri-color, mark-and-sweep

**Phases:**
1. **Mark setup** (STW ~10-30μs) - Enable write barrier
2. **Marking** (concurrent) - Mark live objects while app runs
3. **Mark termination** (STW ~10-30μs) - Finish marking
4. **Sweeping** (concurrent) - Reclaim unmarked memory

**STW pauses typically < 1ms** in modern Go.

---

## 🎨 Tri-Color Algorithm

| Color | Meaning |
|-------|---------|
| **White** | Not yet visited (potentially garbage) |
| **Gray** | Visited, children not yet visited |
| **Black** | Visited, all children visited (alive) |

**Algorithm:** Start all white → Mark roots gray → Process gray to black → White = garbage (sweep)

---

## 🔧 Controlling GC

```go
var m runtime.MemStats
runtime.ReadMemStats(&m)
// m.Alloc, m.TotalAlloc, m.NumGC

runtime.GC()  // Force GC (usually not recommended)

debug.SetGCPercent(100)  // Default: GC when heap doubles
// GOGC=50: more frequent, GOGC=200: less frequent

debug.SetMemoryLimit(1 << 30)  // 1GB limit (Go 1.19+)
debug.FreeOSMemory()
```

```bash
GOGC=50 ./myapp
GOMEMLIMIT=1GiB ./myapp
GODEBUG=gctrace=1 ./myapp
```

---

## 💡 Reducing GC Pressure

```go
// sync.Pool for object reuse
var bufferPool = sync.Pool{
    New: func() interface{} { return make([]byte, 1024) },
}

// Pre-allocate slices
result := make([]int, 0, n)

// Avoid unnecessary pointers
type Good struct { Data [100]byte }
type Bad struct { Data *[100]byte }

// Use value types for small data
func ByValue(x int) int { return x * 2 }

// Batch allocations
allData := make([]byte, 100*1000)
for i := 0; i < 1000; i++ {
    data := allData[i*100 : (i+1)*100]
    process(data)
}
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
