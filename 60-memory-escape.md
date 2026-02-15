# 60 - Go Memory & Escape Analysis

> Understanding how Go manages memory and where variables are allocated.

---

## 📌 What You'll Learn

- Stack vs Heap allocation
- Escape analysis
- How to check if variables escape
- Performance implications
- Memory optimization tips

---

## 🧠 Stack vs Heap

| Feature | Stack | Heap |
|---------|-------|------|
| Speed | Fast (move pointer) | Slower |
| Cleanup | Automatic (function return) | GC |
| Size | ~2KB per goroutine | Dynamic |
| Contents | Local vars, params | Escaped variables |
| Goal | Keep as much on stack as possible | Minimize |

---

## 🔍 Escape Analysis

```go
// Does NOT escape (stays on stack)
func createLocal() int {
    x := 42
    return x
}

// ESCAPES to heap
func createPointer() *int {
    x := 42
    return &x
}

// Check: go build -gcflags="-m" main.go
```

---

## 📋 Escape Analysis Rules

**Escapes to heap when:**
1. Pointer to local variable is returned
2. Variable stored in interface
3. Variable captured by escaping closure
4. Slice/map too large for stack
5. Variable assigned to heap-allocated struct field
6. Variable sent to channel

**Stays on stack when:**
1. Only used within function
2. Small, fixed-size types
3. Returned by value (not pointer)
4. Compiler can prove lifetime

---

## 🛠️ Checking Escape Analysis

```bash
go build -gcflags="-m" main.go
go build -gcflags="-m -m" main.go
# Output: ./main.go:10:2: x escapes to heap
```

---

## ⚡ Performance Implications

```go
// Stack: no allocations
func ByValue() Data {
    var d Data
    d.Value[0] = 42
    return d
}

// Heap: 1 allocation
func ByPointer() *Data {
    d := &Data{}
    d.Value[0] = 42
    return d
}
// Benchmark: ByValue ~25ns/op 0 allocs, ByPointer ~80ns/op 1 alloc
```

---

## 💡 Optimization Tips

```go
// Prefer value receivers for small structs
func (p Point) Distance() float64 { return p.X*p.X + p.Y*p.Y }

// Pre-allocate slices
result := make([]int, 0, n)

// Use arrays for fixed small sizes
func UseArray() [10]int { var arr [10]int; return arr }

// Avoid interface{} when possible
func ProcessInt(x int) int { return x * 2 }

// Pass large structs by pointer to avoid copy
func ProcessLarge(s *LargeStruct) { _ = s.Data[0] }
```

---

## 🎯 Key Takeaways

1. **Stack = fast, Heap = slower (GC)**
2. **Escape analysis** decides allocation location
3. **Returning pointers** causes heap allocation
4. **Interface boxing** causes heap allocation
5. **Use `-gcflags="-m"`** to see escape decisions
6. **Small values by value** = stack = fast
7. **Don't over-optimize** - profile first!

---

## ➡️ Next Steps

**Next Topic:** [61 - Garbage Collection](./61-garbage-collection.md)
