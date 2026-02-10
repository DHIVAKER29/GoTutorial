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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  new(T)                          make(T, args...)               │
│  ───────                         ────────────────               │
│                                                                 │
│  • Works with ANY type           • Only: slice, map, channel    │
│  • Allocates zeroed memory       • Allocates AND initializes    │
│  • Returns *T (pointer)          • Returns T (not pointer!)     │
│  • Memory is zeroed              • Internal structures ready    │
│                                                                 │
│  new(int)     → *int (value=0)   make([]int, 5) → []int         │
│  new(User)    → *User (zeroed)   make(map[K]V)  → map[K]V       │
│  new([]int)   → *[]int (nil!)    make(chan int) → chan int      │
│                                                                 │
│  REMEMBER:                                                      │
│  • new = allocate + zero                                        │
│  • make = allocate + initialize internal structures             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 new() in Detail

```go
// new_examples.go
package main

import "fmt"

type User struct {
    Name  string
    Age   int
    Email string
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           new() Function                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // new returns a pointer to zeroed memory
    
    // new with basic types
    fmt.Println("\n📊 new with Basic Types:")
    intPtr := new(int)
    fmt.Printf("   new(int): %v, value: %d\n", intPtr, *intPtr)  // 0
    
    floatPtr := new(float64)
    fmt.Printf("   new(float64): %v, value: %f\n", floatPtr, *floatPtr)  // 0.0
    
    boolPtr := new(bool)
    fmt.Printf("   new(bool): %v, value: %t\n", boolPtr, *boolPtr)  // false
    
    strPtr := new(string)
    fmt.Printf("   new(string): %v, value: %q\n", strPtr, *strPtr)  // ""
    
    // new with struct
    fmt.Println("\n📊 new with Struct:")
    userPtr := new(User)
    fmt.Printf("   new(User): %+v\n", userPtr)  // All zero values
    
    // Use the pointer
    userPtr.Name = "Alice"
    userPtr.Age = 30
    fmt.Printf("   After assignment: %+v\n", userPtr)
    
    // new with slice (CAREFUL!)
    fmt.Println("\n📊 new with Slice (GOTCHA!):")
    slicePtr := new([]int)
    fmt.Printf("   new([]int): %v\n", slicePtr)    // Pointer to nil slice!
    fmt.Printf("   *slicePtr == nil: %t\n", *slicePtr == nil)  // true!
    // (*slicePtr)[0] = 1  // PANIC! nil slice
    
    // new is equivalent to &T{}
    fmt.Println("\n📊 new vs &T{}:")
    u1 := new(User)
    u2 := &User{}
    fmt.Printf("   new(User): %+v\n", u1)
    fmt.Printf("   &User{}:   %+v\n", u2)
    fmt.Println("   Both are equivalent!")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           new() Function                                  ║
╚══════════════════════════════════════════════════════════╝

📊 new with Basic Types:
   new(int): 0xc0000120a8, value: 0
   new(float64): 0xc0000120b0, value: 0.000000
   new(bool): 0xc0000120b8, value: false
   new(string): 0xc0000120c0, value: ""

📊 new with Struct:
   new(User): &{Name: Age:0 Email:}
   After assignment: &{Name:Alice Age:30 Email:}

📊 new with Slice (GOTCHA!):
   new([]int): &[]
   *slicePtr == nil: true

📊 new vs &T{}:
   new(User): &{Name: Age:0 Email:}
   &User{}:   &{Name: Age:0 Email:}
   Both are equivalent!
```

*Note: Pointer addresses vary on each run.*

---

## 🏗️ make() in Detail

```go
// make_examples.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           make() Function                                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // make only works with: slice, map, channel
    // Returns the TYPE itself (not a pointer!)
    
    // make slice
    fmt.Println("\n📊 make Slice:")
    s1 := make([]int, 5)        // len=5, cap=5
    s2 := make([]int, 5, 10)    // len=5, cap=10
    
    fmt.Printf("   make([]int, 5):     len=%d, cap=%d, %v\n", 
        len(s1), cap(s1), s1)
    fmt.Printf("   make([]int, 5, 10): len=%d, cap=%d, %v\n", 
        len(s2), cap(s2), s2)
    
    // Slice is ready to use!
    s1[0] = 100
    fmt.Printf("   After s1[0]=100: %v\n", s1)
    
    // make map
    fmt.Println("\n📊 make Map:")
    m1 := make(map[string]int)       // Empty map, ready to use
    m2 := make(map[string]int, 100)  // Hint: expect ~100 entries
    
    fmt.Printf("   make(map[string]int): %v\n", m1)
    
    // Map is ready to use!
    m1["key"] = 42
    fmt.Printf("   After m1[\"key\"]=42: %v\n", m1)
    
    _ = m2  // Capacity hint doesn't change behavior
    
    // make channel
    fmt.Println("\n📊 make Channel:")
    ch1 := make(chan int)      // Unbuffered
    ch2 := make(chan int, 5)   // Buffered, capacity 5
    
    fmt.Printf("   make(chan int):    unbuffered, cap=%d\n", cap(ch1))
    fmt.Printf("   make(chan int, 5): buffered,   cap=%d\n", cap(ch2))
    
    // Channel is ready to use!
    go func() { ch1 <- 42 }()
    value := <-ch1
    fmt.Printf("   Received: %d\n", value)
    
    // Why make exists
    fmt.Println("\n📊 Why make is Necessary:")
    fmt.Println("   Slices, maps, channels have internal structures:")
    fmt.Println("   • Slice: pointer + length + capacity")
    fmt.Println("   • Map: hash table pointer")
    fmt.Println("   • Channel: circular buffer + locks")
    fmt.Println("   make() initializes these internal structures!")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           make() Function                                 ║
╚══════════════════════════════════════════════════════════╝

📊 make Slice:
   make([]int, 5):     len=5, cap=5, [0 0 0 0 0]
   make([]int, 5, 10): len=5, cap=10, [0 0 0 0 0]
   After s1[0]=100: [100 0 0 0 0]

📊 make Map:
   make(map[string]int): map[]
   After m1["key"]=42: map[key:42]

📊 make Channel:
   make(chan int):    unbuffered, cap=0
   make(chan int, 5): buffered,   cap=5
   Received: 42

📊 Why make is Necessary:
   Slices, maps, channels have internal structures:
   • Slice: pointer + length + capacity
   • Map: hash table pointer
   • Channel: circular buffer + locks
   make() initializes these internal structures!
```

---

## ⚠️ Common Mistakes

```go
// mistakes.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Common Mistakes                                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Mistake 1: Using new for slice/map/channel
    fmt.Println("\n❌ Mistake 1: new([]int)")
    slicePtr := new([]int)
    fmt.Printf("   Result: pointer to NIL slice!\n")
    fmt.Printf("   *slicePtr == nil: %t\n", *slicePtr == nil)
    // (*slicePtr)[0] = 1  // PANIC!
    
    // Correct way
    fmt.Println("\n✅ Correct: make([]int, 5)")
    slice := make([]int, 5)
    slice[0] = 1  // Works!
    fmt.Printf("   Result: %v\n", slice)
    
    // Mistake 2: Using new for map
    fmt.Println("\n❌ Mistake 2: new(map[string]int)")
    mapPtr := new(map[string]int)
    fmt.Printf("   Result: pointer to NIL map!\n")
    // (*mapPtr)["key"] = 1  // PANIC!
    
    // Correct way
    fmt.Println("\n✅ Correct: make(map[string]int)")
    m := make(map[string]int)
    m["key"] = 1  // Works!
    fmt.Printf("   Result: %v\n", m)
    
    // Mistake 3: Forgetting make returns value, not pointer
    fmt.Println("\n❌ Mistake 3: Thinking make returns pointer")
    s := make([]int, 5)
    fmt.Printf("   Type of s: %T (not *[]int!)\n", s)
    
    // This is fine - slices are already reference types
    modifySlice(s)
    fmt.Printf("   After modify: %v\n", s)
}

func modifySlice(s []int) {
    s[0] = 999
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Common Mistakes                                 ║
╚══════════════════════════════════════════════════════════╝

❌ Mistake 1: new([]int)
   Result: pointer to NIL slice!
   *slicePtr == nil: true

✅ Correct: make([]int, 5)
   Result: [1 0 0 0 0]

❌ Mistake 2: new(map[string]int)
   Result: pointer to NIL map!

✅ Correct: make(map[string]int)
   Result: map[key:1]

❌ Mistake 3: Thinking make returns pointer
   Type of s: []int (not *[]int!)

   After modify: [999 0 0 0 0]
```

---

## 🎯 Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  WHEN TO USE WHAT                                               │
│                                                                 │
│  new(T)                                                         │
│  ───────                                                        │
│  • Get a pointer to zero value                                  │
│  • Basic types: new(int), new(string)                           │
│  • Structs: new(MyStruct) or &MyStruct{} (equivalent)           │
│  • Rarely used - usually &T{} is preferred                      │
│                                                                 │
│  make(T, args...)                                               │
│  ────────────────                                               │
│  • Slices: make([]T, length) or make([]T, len, cap)             │
│  • Maps: make(map[K]V) or make(map[K]V, hint)                   │
│  • Channels: make(chan T) or make(chan T, buffer)               │
│  • ALWAYS use for slice/map/channel!                            │
│                                                                 │
│  LITERAL SYNTAX (Alternative to new)                            │
│  ─────────────────────────────────────                          │
│  • &User{}         instead of new(User)                         │
│  • &User{Name:"A"} with initialization                          │
│  • []int{1,2,3}    instead of make + append                     │
│  • map[K]V{...}    instead of make + assign                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│                                                                 │
│  User u = new User();              u := new(User)  // or &User{}│
│                                                                 │
│  ArrayList<Integer> list =         s := make([]int, 0)          │
│      new ArrayList<>();                                         │
│                                                                 │
│  HashMap<String, Integer> map =    m := make(map[string]int)    │
│      new HashMap<>();                                           │
│                                                                 │
│  // Java: always new               // Go: new vs make           │
│  // returns reference              // new = *T, make = T        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ➡️ Next Steps

**Next Topic:** [60 - Go Memory & Escape Analysis](./60-memory-escape.md)

