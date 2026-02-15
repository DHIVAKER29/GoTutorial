# 59 - make vs new

> Understanding Go's two allocation functions and when to use each.

---

## 📌 What You'll Learn

- Difference between make and new
- When to use each
- Memory allocation details
- Common mistakes

---

## 🤔 The Difference

| | new(T) | make(T, args...) |
|---|--------|-------------------|
| **Works with** | Any type | Only slice, map, channel |
| **Returns** | *T (pointer) | T (value) |
| **Memory** | Zeroed | Allocated + initialized |
| **new([]int)** | *[]int (nil slice!) | N/A |
| **make([]int, 5)** | N/A | []int ready to use |

**Remember:** new = allocate + zero | make = allocate + initialize internal structures

---

## 📝 new() in Detail

```go
intPtr := new(int)
// Output: *intPtr == 0

userPtr := new(User)
// Output: &User{Name: "", Age: 0}

slicePtr := new([]int)
// Output: *slicePtr == nil (GOTCHA! Don't use new for slices)
// (*slicePtr)[0] = 1  // PANIC!

// new(User) == &User{}
```

---

## 🏗️ make() in Detail

```go
// Slice
s1 := make([]int, 5)        // len=5, cap=5
s2 := make([]int, 5, 10)    // len=5, cap=10
s1[0] = 100
// Output: [100 0 0 0 0]

// Map
m := make(map[string]int)
m["key"] = 42
// Output: map[key:42]

// Channel
ch1 := make(chan int)      // Unbuffered
ch2 := make(chan int, 5)   // Buffered
// Output: ch1 cap=0, ch2 cap=5
```

---

## ⚠️ Common Mistakes

```go
// ❌ new([]int) - pointer to nil slice
slicePtr := new([]int)
// (*slicePtr)[0] = 1  // PANIC!

// ✅ make([]int, 5)
slice := make([]int, 5)
slice[0] = 1  // Works!
// Output: [1 0 0 0 0]

// ❌ new(map[string]int) - pointer to nil map
// ✅ make(map[string]int)

// make returns value, not pointer
s := make([]int, 5)
// s is []int (slices are reference types)
```

---

## 🧩 Quick Reference

| Use | When |
|-----|------|
| **new(T)** | Get pointer to zero value; basic types, structs; rarely used - prefer &T{} |
| **make(T)** | Slices, maps, channels - ALWAYS use |

**Literal alternatives:** `&User{}`, `&User{Name:"A"}`, `[]int{1,2,3}`, `map[K]V{...}`

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `new User()` | `new(User)` or `&User{}` |
| `new ArrayList<>()` | `make([]int, 0)` |
| `new HashMap<>()` | `make(map[string]int)` |
| Always returns reference | new = *T, make = T |

---

## ➡️ Next Steps

**Next Topic:** [60 - Go Memory & Escape Analysis](./60-memory-escape.md)
