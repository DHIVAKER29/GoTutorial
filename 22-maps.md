# 22 - Maps

> Key-value data structures in Go - fast lookups, dynamic sizing, and unordered storage.

---

## 📌 What You'll Learn

- What maps are and how they work internally
- Creating, reading, writing, and deleting entries
- The comma-ok idiom for safe lookups
- Iteration (unordered!)
- Common patterns and production usage
- Sample programs for every concept

---

## 🤔 What is a Map?

### Definition

> A **map** is an unordered collection of key-value pairs where each key is unique. Also known as hash table, dictionary, or associative array.

### Real-World Analogy

Like a phone contact list: each name (key) maps to one number (value). Keys are unique. Lookup is fast (O(1)). Order doesn't matter. You can add or remove contacts anytime.

```go
contacts := map[string]string{
    "Alice": "555-0101",
    "Bob":   "555-0102",
}
phone := contacts["Alice"]  // "555-0101"
```

---

## 📊 Map Internals

- **Hash table**: Key → hash function → bucket index → store value
- **Time complexity**: O(1) average for lookup, insert, delete
- **Key requirements**: Keys must be comparable (`==`, `!=`)
  - ✅ `string`, `int`, `float`, `bool`, pointers, structs (with comparable fields)
  - ❌ slices, maps, functions (not comparable)

---

## 📝 Creating Maps

```go
ages := map[string]int{
    "Alice": 25,
    "Bob":   30,
}
fmt.Println(ages)  // Output: map[Alice:25 Bob:30]
```

```go
scores := map[string]int{}
scores["Alice"] = 95
```

```go
users := make(map[int]string)
users[1] = "Alice"
```

```go
largeMap := make(map[string]int, 1000)  // Capacity hint
```

```go
var nilMap map[string]int
fmt.Println(nilMap == nil)  // Output: true
// nilMap["key"] = 1  // ❌ PANIC! Cannot write to nil map
```

```go
type Point struct{ X, Y int }
pointLabels := map[Point]string{{0, 0}: "origin"}
fmt.Println(pointLabels)  // Output: map[{0 0}:origin]
```

### Complete Example: Creating Maps

```go
package main

import "fmt"

func main() {
    ages := map[string]int{"Alice": 25, "Bob": 30}
    scores := map[string]int{}
    scores["Charlie"] = 95

    fmt.Println(ages)
    fmt.Println(scores)
}
```

**Output:**
```
map[Alice:25 Bob:30]
map[Charlie:95]
```

---

## 🔍 Reading and Writing

```go
ages := map[string]int{"Alice": 25, "Bob": 30}
fmt.Println(ages["Alice"])    // Output: 25
fmt.Println(ages["Charlie"])  // Output: 0 (missing → zero value!)
```

**Problem**: Missing key returns zero value. How to distinguish "key exists with value 0" from "key doesn't exist"?

**Solution**: Comma-ok idiom.

```go
value, exists := ages["Alice"]
fmt.Println(value, exists)  // Output: 25 true

value, exists = ages["Eve"]
fmt.Println(value, exists)  // Output: 0 false
```

```go
if age, ok := ages["Alice"]; ok {
    fmt.Printf("Alice is %d years old\n", age)
} else {
    fmt.Println("Alice not found")
}
```

```go
ages["Charlie"] = 28  // Add new
ages["Alice"] = 26    // Update existing
delete(ages, "Bob")
fmt.Println(len(ages))  // Output: 2
```

### Complete Example: Map Operations

```go
package main

import "fmt"

func main() {
    ages := map[string]int{"Alice": 25, "Bob": 30}
    ages["Charlie"] = 28
    ages["Alice"] = 26
    delete(ages, "Bob")

    if age, ok := ages["Alice"]; ok {
        fmt.Printf("Alice: %d\n", age)
    }
    fmt.Println(ages)
    fmt.Println(len(ages))
}
```

**Output:**
```
Alice: 26
map[Alice:26 Charlie:28]
2
```

---

## 🔄 Iteration

Map iteration order is **random** in Go. Never rely on order.

```go
scores := map[string]int{"Alice": 95, "Bob": 87, "Charlie": 92}
for name, score := range scores {
    fmt.Printf("%s: %d\n", name, score)
}
```

```go
for name := range scores {
    fmt.Println(name)
}
```

**Sorted iteration**: Collect keys, sort, then iterate.

```go
keys := make([]string, 0, len(scores))
for k := range scores {
    keys = append(keys, k)
}
sort.Strings(keys)
for _, name := range keys {
    fmt.Printf("%s: %d\n", name, scores[name])
}
```

**Warning**: Do not add or delete map entries while iterating—undefined behavior.

---

## 🏭 Production Patterns

```go
// Set (map[T]bool)
seen := make(map[string]bool)
for _, item := range items {
    seen[item] = true
}
```

```go
// Set with struct{} (0 bytes per value)
set := make(map[string]struct{})
set["apple"] = struct{}{}
if _, exists := set["apple"]; exists {
    fmt.Println("apple in set")
}
```

```go
// Word frequency
freq := make(map[string]int)
for _, word := range words {
    freq[word]++
}
```

```go
// Grouping
byCity := make(map[string][]string)
for _, p := range people {
    byCity[p.City] = append(byCity[p.City], p.Name)
}
```

```go
// Get with default
func getOrDefault(m map[string]string, key, defaultVal string) string {
    if val, ok := m[key]; ok {
        return val
    }
    return defaultVal
}
```

### Complete Example: Production Patterns

```go
package main

import "fmt"

func main() {
    // Word frequency
    words := []string{"the", "quick", "brown", "fox", "the", "quick"}
    freq := make(map[string]int)
    for _, word := range words {
        freq[word]++
    }
    fmt.Println(freq)

    // Grouping
    type Person struct{ Name, City string }
    people := []Person{{"Alice", "NYC"}, {"Bob", "LA"}, {"Charlie", "NYC"}}
    byCity := make(map[string][]string)
    for _, p := range people {
        byCity[p.City] = append(byCity[p.City], p.Name)
    }
    fmt.Println(byCity)
}
```

**Output:**
```
map[brown:1 fox:1 quick:2 the:2]
map[LA:[Bob] NYC:[Alice Charlie]]
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `Map<String, Integer> map = new HashMap<>();` | `m := map[string]int{}` |
| `map.put("key", 1);` | `m["key"] = 1` |
| `map.get("key");` returns null if missing | `m["key"]` returns zero value |
| `map.containsKey("key");` | `_, ok := m["key"]` |
| `map.remove("key");` | `delete(m, "key")` |
| `map.size();` | `len(m)` |
| `for (var e : map.entrySet())` | `for k, v := range m` |
| `map.getOrDefault("k", 0);` | `if v, ok := m["k"]; ok { return v }; return 0` |

---

## 🎯 Key Takeaways

1. **map[K]V** - K is key type, V is value type
2. **Keys must be comparable** (no slices, maps, functions)
3. **Zero value is nil** - reading ok, writing panics!
4. **Comma-ok idiom** - `val, ok := m[key]` for safe lookups
5. **Iteration order is random** - never rely on order
6. **O(1) operations** - fast lookup, insert, delete
7. **Use `make()`** or literal, never leave nil for writable maps
8. **Thread-unsafe** - use sync.Map or mutex for concurrent access

---

## ➡️ Next Steps

**Next Topic:** [23 - Structs](./23-structs.md)
