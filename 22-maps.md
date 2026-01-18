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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MAP = PHONE CONTACT LIST                                       │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  CONTACTS                                               │   │
│  │  ─────────────────────────────────────────────────────   │   │
│  │  "Alice"   → "555-0101"                                 │   │
│  │  "Bob"     → "555-0102"                                 │   │
│  │  "Charlie" → "555-0103"                                 │   │
│  │  "Diana"   → "555-0104"                                 │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Key properties:                                                │
│  • Each name (KEY) is unique                                    │
│  • Quick lookup: "What's Alice's number?" → O(1)                │
│  • Order doesn't matter                                         │
│  • Can add/remove contacts anytime                              │
│                                                                 │
│  In Go:                                                         │
│  contacts := map[string]string{                                 │
│      "Alice": "555-0101",                                       │
│      "Bob":   "555-0102",                                       │
│  }                                                              │
│  phone := contacts["Alice"]  // "555-0101"                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Map Internals

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  HOW MAPS WORK (Hash Table)                                     │
│                                                                 │
│  1. Key → Hash Function → Bucket Index                          │
│  2. Store value in that bucket                                  │
│  3. Lookup: same hash → same bucket → find value                │
│                                                                 │
│  "Alice" ──► hash("Alice") ──► bucket 3                         │
│                                   ↓                             │
│  ┌─────────┬─────────┬─────────┬─────────────────┬───────────┐ │
│  │Bucket 0 │Bucket 1 │Bucket 2 │     Bucket 3    │ Bucket 4  │ │
│  │ (empty) │  "Bob"  │ (empty) │"Alice":"555-0101│  "Diana"  │ │
│  │         │"555-02" │         │                 │ "555-04"  │ │
│  └─────────┴─────────┴─────────┴─────────────────┴───────────┘ │
│                                                                 │
│  TIME COMPLEXITY:                                               │
│  • Lookup: O(1) average                                         │
│  • Insert: O(1) average                                         │
│  • Delete: O(1) average                                         │
│                                                                 │
│  KEY REQUIREMENTS:                                              │
│  • Must be comparable (==, !=)                                  │
│  • ✅ string, int, float, bool, pointers, structs (comparable)  │
│  • ❌ slices, maps, functions (not comparable)                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Creating Maps

```go
// map_creation.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              CREATING MAPS                                ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Method 1: Map literal
    fmt.Println("\n📊 Method 1: Map Literal")
    ages := map[string]int{
        "Alice":   25,
        "Bob":     30,
        "Charlie": 28,
    }
    fmt.Printf("   ages = %v\n", ages)
    
    // Method 2: Empty map literal
    fmt.Println("\n📊 Method 2: Empty Map Literal")
    scores := map[string]int{}
    scores["Alice"] = 95
    fmt.Printf("   scores = %v\n", scores)
    
    // Method 3: make() function
    fmt.Println("\n📊 Method 3: make()")
    users := make(map[int]string)  // map[keyType]valueType
    users[1] = "Alice"
    users[2] = "Bob"
    fmt.Printf("   users = %v\n", users)
    
    // Method 4: make() with capacity hint
    fmt.Println("\n📊 Method 4: make() with Capacity")
    largeMap := make(map[string]int, 1000)  // Hint: ~1000 entries
    fmt.Printf("   Created map with capacity hint 1000\n")
    _ = largeMap
    
    // Method 5: nil map (CAUTION!)
    fmt.Println("\n⚠️ Method 5: Nil Map (careful!)")
    var nilMap map[string]int
    fmt.Printf("   var nilMap map[string]int\n")
    fmt.Printf("   nilMap == nil: %t\n", nilMap == nil)
    fmt.Printf("   Reading nil map: nilMap[\"key\"] = %d (returns zero value)\n", nilMap["key"])
    // nilMap["key"] = 1  // ❌ PANIC! Cannot write to nil map
    
    // Various key and value types
    fmt.Println("\n📊 Various Types:")
    
    // String → int
    wordCount := map[string]int{"hello": 5, "world": 3}
    fmt.Printf("   map[string]int: %v\n", wordCount)
    
    // Int → string
    errorCodes := map[int]string{404: "Not Found", 500: "Server Error"}
    fmt.Printf("   map[int]string: %v\n", errorCodes)
    
    // String → bool (set-like usage)
    seen := map[string]bool{"apple": true, "banana": true}
    fmt.Printf("   map[string]bool (set): %v\n", seen)
    
    // Struct as key
    type Point struct{ X, Y int }
    pointLabels := map[Point]string{
        {0, 0}: "origin",
        {1, 0}: "unit-x",
    }
    fmt.Printf("   map[Point]string: %v\n", pointLabels)
}
```

---

## 🔍 Reading and Writing

```go
// map_operations.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              MAP OPERATIONS                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    ages := map[string]int{
        "Alice": 25,
        "Bob":   30,
    }
    
    // Read existing key
    fmt.Println("\n📊 Reading Values:")
    fmt.Printf("   ages[\"Alice\"] = %d\n", ages["Alice"])
    
    // Read missing key (returns zero value!)
    fmt.Printf("   ages[\"Charlie\"] = %d (missing key → zero value!)\n", ages["Charlie"])
    
    // ⚠️ Problem: How to tell missing from actual zero?
    fmt.Println("\n⚠️ The Problem:")
    ages["Dave"] = 0  // Dave is 0 years old (valid!)
    fmt.Printf("   ages[\"Dave\"] = %d (exists but is 0)\n", ages["Dave"])
    fmt.Printf("   ages[\"Eve\"] = %d (doesn't exist, also 0!)\n", ages["Eve"])
    
    // ✅ Solution: Comma-ok idiom
    fmt.Println("\n✅ Solution: Comma-Ok Idiom")
    value, exists := ages["Dave"]
    fmt.Printf("   ages[\"Dave\"]: value=%d, exists=%t\n", value, exists)
    
    value, exists = ages["Eve"]
    fmt.Printf("   ages[\"Eve\"]: value=%d, exists=%t\n", value, exists)
    
    // Common pattern
    fmt.Println("\n📊 Common Pattern:")
    if age, ok := ages["Alice"]; ok {
        fmt.Printf("   Alice is %d years old\n", age)
    } else {
        fmt.Println("   Alice not found")
    }
    
    if age, ok := ages["Eve"]; ok {
        fmt.Printf("   Eve is %d years old\n", age)
    } else {
        fmt.Println("   Eve not found")
    }
    
    // Writing values
    fmt.Println("\n📊 Writing Values:")
    ages["Charlie"] = 28  // Add new
    ages["Alice"] = 26    // Update existing
    fmt.Printf("   After updates: %v\n", ages)
    
    // Deleting values
    fmt.Println("\n📊 Deleting Values:")
    delete(ages, "Bob")
    fmt.Printf("   After delete(ages, \"Bob\"): %v\n", ages)
    
    // Delete non-existent key (no error!)
    delete(ages, "NonExistent")  // Does nothing, no panic
    fmt.Println("   delete() on missing key: no error")
    
    // Length
    fmt.Println("\n📊 Length:")
    fmt.Printf("   len(ages) = %d\n", len(ages))
}
```

---

## 🔄 Iteration

```go
// map_iteration.go
package main

import (
    "fmt"
    "sort"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              MAP ITERATION                                ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    scores := map[string]int{
        "Alice":   95,
        "Bob":     87,
        "Charlie": 92,
        "Diana":   88,
    }
    
    // Iterate over key-value pairs
    fmt.Println("\n📊 Iterate Key-Value (order is RANDOM!):")
    for name, score := range scores {
        fmt.Printf("   %s: %d\n", name, score)
    }
    
    fmt.Println("\n   Run again - different order!")
    for name, score := range scores {
        fmt.Printf("   %s: %d\n", name, score)
    }
    
    // Iterate keys only
    fmt.Println("\n📊 Iterate Keys Only:")
    for name := range scores {
        fmt.Printf("   %s\n", name)
    }
    
    // Sorted iteration
    fmt.Println("\n📊 Sorted Iteration (collect keys, sort, iterate):")
    keys := make([]string, 0, len(scores))
    for k := range scores {
        keys = append(keys, k)
    }
    sort.Strings(keys)
    
    for _, name := range keys {
        fmt.Printf("   %s: %d\n", name, scores[name])
    }
    
    // ⚠️ Cannot modify map while iterating (undefined behavior)
    fmt.Println("\n⚠️ Modifying While Iterating:")
    fmt.Println("   Adding/deleting during iteration = undefined!")
    fmt.Println("   Solution: Collect keys first, then modify")
}
```

---

## 🏭 Production Patterns

```go
// map_production.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION MAP PATTERNS                         ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: Set implementation
    fmt.Println("\n📊 Pattern 1: Set (using map[T]bool)")
    seen := make(map[string]bool)
    items := []string{"apple", "banana", "apple", "cherry", "banana"}
    
    for _, item := range items {
        seen[item] = true
    }
    fmt.Printf("   Unique items: ")
    for item := range seen {
        fmt.Printf("%s ", item)
    }
    fmt.Println()
    
    // Pattern 2: Set with struct{} (memory efficient)
    fmt.Println("\n📊 Pattern 2: Set with struct{} (0 bytes per value)")
    type void struct{}
    set := make(map[string]void)
    set["apple"] = void{}
    set["banana"] = void{}
    
    if _, exists := set["apple"]; exists {
        fmt.Println("   apple is in set")
    }
    
    // Pattern 3: Counting/Frequency
    fmt.Println("\n📊 Pattern 3: Word Frequency")
    words := []string{"the", "quick", "brown", "fox", "the", "quick"}
    freq := make(map[string]int)
    
    for _, word := range words {
        freq[word]++  // Auto-initializes to 0, then increments
    }
    fmt.Printf("   Frequency: %v\n", freq)
    
    // Pattern 4: Grouping
    fmt.Println("\n📊 Pattern 4: Grouping")
    type Person struct {
        Name string
        City string
    }
    people := []Person{
        {"Alice", "NYC"},
        {"Bob", "LA"},
        {"Charlie", "NYC"},
        {"Diana", "LA"},
    }
    
    byCity := make(map[string][]string)
    for _, p := range people {
        byCity[p.City] = append(byCity[p.City], p.Name)
    }
    fmt.Printf("   Grouped by city: %v\n", byCity)
    
    // Pattern 5: Cache/Memoization
    fmt.Println("\n📊 Pattern 5: Memoization Cache")
    cache := make(map[int]int)
    fib := func(n int) int {
        if cached, ok := cache[n]; ok {
            return cached
        }
        // Calculate...
        result := n * 2  // Simplified
        cache[n] = result
        return result
    }
    fmt.Printf("   fib(10) = %d (calculated)\n", fib(10))
    fmt.Printf("   fib(10) = %d (cached)\n", fib(10))
    
    // Pattern 6: Default values
    fmt.Println("\n📊 Pattern 6: Get with Default")
    config := map[string]string{"host": "localhost"}
    
    host := getOrDefault(config, "host", "127.0.0.1")
    port := getOrDefault(config, "port", "8080")
    fmt.Printf("   host=%s, port=%s\n", host, port)
}

func getOrDefault(m map[string]string, key, defaultVal string) string {
    if val, ok := m[key]; ok {
        return val
    }
    return defaultVal
}
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  Map<String, Integer> map          map[string]int               │
│  = new HashMap<>();                                             │
│                                                                 │
│  map.put("key", 1);                m["key"] = 1                 │
│                                                                 │
│  map.get("key");                   m["key"]                     │
│  // Returns null if missing        // Returns zero value        │
│                                                                 │
│  map.containsKey("key");           _, ok := m["key"]            │
│                                                                 │
│  map.remove("key");                delete(m, "key")             │
│                                                                 │
│  map.size();                       len(m)                       │
│                                                                 │
│  for (var e : map.entrySet())      for k, v := range m          │
│                                                                 │
│  map.getOrDefault("k", 0);         if v, ok := m["k"]; ok {     │
│                                        return v                 │
│                                    }                            │
│                                    return 0                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **map[K]V** - K is key type, V is value type
2. **Keys must be comparable** (no slices, maps, functions)
3. **Zero value is nil** - reading ok, writing panics!
4. **Comma-ok idiom** - `val, ok := m[key]` for safe lookups
5. **Iteration order is random** - never rely on order
6. **O(1) operations** - fast lookup, insert, delete
7. **Use `make()`** or literal, never leave nil
8. **Thread-unsafe** - use sync.Map or mutex for concurrent access

---

## ➡️ Next Steps

**Next Topic:** [23 - Structs](./23-structs.md)

