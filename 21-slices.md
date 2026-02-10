# 21 - Slices

> Dynamic, flexible views into arrays - the most commonly used collection in Go.

---

## 📌 What You'll Learn

- What slices are and how they differ from arrays
- Slice internals: length, capacity, and underlying array
- Creating, modifying, and growing slices
- The `append` function and how it works
- Slice expressions and sub-slices
- Common pitfalls (shared backing arrays!)
- Sample programs for every concept

---

## 🤔 What is a Slice?

### Definition

> A **slice** is a dynamically-sized, flexible view into an underlying array. It provides a more powerful and convenient interface than arrays.

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  SLICE = WINDOW INTO A BOOKSHELF                                │
│                                                                 │
│  Imagine a bookshelf (the underlying array):                    │
│                                                                 │
│  BOOKSHELF (underlying array):                                  │
│  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐ │
│  │Book │Book │Book │Book │Book │Book │Book │Book │Book │Book │ │
│  │  0  │  1  │  2  │  3  │  4  │  5  │  6  │  7  │  8  │  9  │ │
│  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘ │
│                                                                 │
│  SLICE = A window/view into part of the shelf:                  │
│                                                                 │
│  Slice A: books[2:6]                                            │
│                ┌─────────────────────────┐                      │
│                │ View: Book 2,3,4,5      │                      │
│                │ Length: 4               │                      │
│                │ Capacity: 8 (can expand)│                      │
│                └─────────────────────────┘                      │
│                    │                                            │
│  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐ │
│  │  0  │  1  │  2  │  3  │  4  │  5  │  6  │  7  │  8  │  9  │ │
│  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘ │
│                ↑─────────────────↑       ↑                      │
│              Start             End    Capacity ends             │
│                                                                 │
│  Key insight:                                                   │
│  • Slice doesn't OWN the data                                   │
│  • It's a VIEW into existing data                               │
│  • Changes through slice affect original!                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Slice Internals: The Slice Header

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  SLICE HEADER (what a slice variable actually contains)         │
│                                                                 │
│  ┌─────────────────────────────────┐                           │
│  │        SLICE HEADER             │                           │
│  │  ┌─────────────────────────┐    │                           │
│  │  │ Pointer to first element │────┼────┐                     │
│  │  └─────────────────────────┘    │    │                      │
│  │  ┌─────────────────────────┐    │    │                      │
│  │  │ Length (len)        = 4 │    │    │                      │
│  │  └─────────────────────────┘    │    │                      │
│  │  ┌─────────────────────────┐    │    │                      │
│  │  │ Capacity (cap)      = 6 │    │    │                      │
│  │  └─────────────────────────┘    │    │                      │
│  └─────────────────────────────────┘    │                      │
│                                         │                      │
│                                         ▼                      │
│  UNDERLYING ARRAY:                                             │
│  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐            │
│  │  10 │  20 │  30 │  40 │  50 │  60 │  70 │  80 │            │
│  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘            │
│            ↑                       ↑                           │
│         Pointer                Capacity ends                   │
│         starts                                                 │
│         here                                                   │
│                                                                │
│  slice = [20, 30, 40, 50]                                      │
│  len(slice) = 4 (elements in view)                             │
│  cap(slice) = 6 (room to grow before reallocation)             │
│                                                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Creating Slices

```go
// slice_creation.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CREATING SLICES                                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Method 1: Slice literal
    fmt.Println("\n📊 Method 1: Slice Literal")
    s1 := []int{10, 20, 30, 40, 50}
    fmt.Printf("   s1 := []int{10, 20, 30, 40, 50}\n")
    fmt.Printf("   s1 = %v, len=%d, cap=%d\n", s1, len(s1), cap(s1))
    
    // Method 2: make() function
    fmt.Println("\n📊 Method 2: make([]T, length, capacity)")
    s2 := make([]int, 5)  // len=5, cap=5 (default)
    fmt.Printf("   s2 := make([]int, 5)\n")
    fmt.Printf("   s2 = %v, len=%d, cap=%d\n", s2, len(s2), cap(s2))
    
    s3 := make([]int, 3, 10)  // len=3, cap=10
    fmt.Printf("   s3 := make([]int, 3, 10)\n")
    fmt.Printf("   s3 = %v, len=%d, cap=%d\n", s3, len(s3), cap(s3))
    
    // Method 3: Slice from array
    fmt.Println("\n📊 Method 3: Slice from Array")
    arr := [6]int{10, 20, 30, 40, 50, 60}
    s4 := arr[1:4]  // Elements at index 1, 2, 3
    fmt.Printf("   arr := [6]int{10, 20, 30, 40, 50, 60}\n")
    fmt.Printf("   s4 := arr[1:4] = %v, len=%d, cap=%d\n", s4, len(s4), cap(s4))
    
    // Method 4: Slice from slice
    fmt.Println("\n📊 Method 4: Slice from Slice")
    s5 := s1[1:3]
    fmt.Printf("   s5 := s1[1:3] = %v, len=%d, cap=%d\n", s5, len(s5), cap(s5))
    
    // Method 5: nil slice
    fmt.Println("\n📊 Method 5: Nil Slice (zero value)")
    var s6 []int  // nil slice
    fmt.Printf("   var s6 []int\n")
    fmt.Printf("   s6 = %v, len=%d, cap=%d, nil=%t\n", s6, len(s6), cap(s6), s6 == nil)
    
    // Method 6: Empty slice (not nil!)
    fmt.Println("\n📊 Method 6: Empty Slice (not nil)")
    s7 := []int{}
    s8 := make([]int, 0)
    fmt.Printf("   s7 := []int{} → nil=%t\n", s7 == nil)
    fmt.Printf("   s8 := make([]int, 0) → nil=%t\n", s8 == nil)
    
    // Nil vs empty behavior
    fmt.Println("\n💡 Nil vs Empty: Same behavior!")
    fmt.Printf("   len(nil slice) = %d\n", len(s6))
    fmt.Printf("   len(empty slice) = %d\n", len(s7))
    // Both work with append, range, etc.
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           CREATING SLICES                                 ║
╚══════════════════════════════════════════════════════════╝

📊 Method 1: Slice Literal
   s1 := []int{10, 20, 30, 40, 50}
   s1 = [10 20 30 40 50], len=5, cap=5

📊 Method 2: make([]T, length, capacity)
   s2 := make([]int, 5)
   s2 = [0 0 0 0 0], len=5, cap=5
   s3 := make([]int, 3, 10)
   s3 = [0 0 0], len=3, cap=10

📊 Method 3: Slice from Array
   arr := [6]int{10, 20, 30, 40, 50, 60}
   s4 := arr[1:4] = [20 30 40], len=3, cap=5

📊 Method 4: Slice from Slice
   s5 := s1[1:3] = [20 30], len=2, cap=4

📊 Method 5: Nil Slice (zero value)
   var s6 []int
   s6 = [], len=0, cap=0, nil=true

📊 Method 6: Empty Slice (not nil)
   s7 := []int{} → nil=false
   s8 := make([]int, 0) → nil=false

💡 Nil vs Empty: Same behavior!
   len(nil slice) = 0
   len(empty slice) = 0
```

---

## 🔄 Slice Expressions

```go
// slice_expressions.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           SLICE EXPRESSIONS                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    s := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    fmt.Printf("\n📊 Original: s = %v\n", s)
    
    // Basic slicing: s[low:high]
    fmt.Println("\n📊 Basic Slicing: s[low:high]")
    fmt.Printf("   s[2:5] = %v (indices 2, 3, 4)\n", s[2:5])
    fmt.Printf("   s[0:3] = %v (first 3)\n", s[0:3])
    fmt.Printf("   s[7:10] = %v (last 3)\n", s[7:10])
    
    // Omitting indices
    fmt.Println("\n📊 Omitting Indices:")
    fmt.Printf("   s[:5]  = %v (first 5)\n", s[:5])
    fmt.Printf("   s[5:]  = %v (from index 5 to end)\n", s[5:])
    fmt.Printf("   s[:]   = %v (full slice, copy of header)\n", s[:])
    
    // Full slice expression: s[low:high:max]
    fmt.Println("\n📊 Full Slice Expression: s[low:high:max]")
    s2 := s[2:5:7]  // Limits capacity!
    fmt.Printf("   s[2:5:7] = %v, len=%d, cap=%d\n", s2, len(s2), cap(s2))
    fmt.Println("   (max limits capacity to prevent accidental sharing)")
    
    // Capacity calculation
    fmt.Println("\n📊 Capacity Calculation:")
    fmt.Printf("   For s[low:high], cap = len(s) - low\n")
    sub := s[3:6]
    fmt.Printf("   s[3:6] = %v, len=%d, cap=%d (10-3=7)\n", sub, len(sub), cap(sub))
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           SLICE EXPRESSIONS                               ║
╚══════════════════════════════════════════════════════════╝

📊 Original: s = [0 1 2 3 4 5 6 7 8 9]

📊 Basic Slicing: s[low:high]
   s[2:5] = [2 3 4] (indices 2, 3, 4)
   s[0:3] = [0 1 2] (first 3)
   s[7:10] = [7 8 9] (last 3)

📊 Omitting Indices:
   s[:5]  = [0 1 2 3 4] (first 5)
   s[5:]  = [5 6 7 8 9] (from index 5 to end)
   s[:]   = [0 1 2 3 4 5 6 7 8 9] (full slice, copy of header)

📊 Full Slice Expression: s[low:high:max]
   s[2:5:7] = [2 3 4], len=3, cap=5
   (max limits capacity to prevent accidental sharing)

📊 Capacity Calculation:
   For s[low:high], cap = len(s) - low
   s[3:6] = [3 4 5], len=3, cap=7 (10-3=7)
```

---

## ➕ The append Function

```go
// append_demo.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           THE APPEND FUNCTION                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic append
    fmt.Println("\n📊 Basic Append:")
    s := []int{1, 2, 3}
    fmt.Printf("   Before: s = %v, len=%d, cap=%d\n", s, len(s), cap(s))
    
    s = append(s, 4)  // MUST reassign!
    fmt.Printf("   After append(s, 4): s = %v, len=%d, cap=%d\n", s, len(s), cap(s))
    
    // Append multiple values
    fmt.Println("\n📊 Append Multiple Values:")
    s = append(s, 5, 6, 7)
    fmt.Printf("   append(s, 5, 6, 7) = %v\n", s)
    
    // Append slice to slice (spread operator)
    fmt.Println("\n📊 Append Slice to Slice:")
    more := []int{8, 9, 10}
    s = append(s, more...)  // Note the ...
    fmt.Printf("   append(s, more...) = %v\n", s)
    
    // Capacity growth
    fmt.Println("\n📊 Capacity Growth (watch it double!):")
    growth := make([]int, 0)
    for i := 0; i < 10; i++ {
        growth = append(growth, i)
        fmt.Printf("   len=%d, cap=%d\n", len(growth), cap(growth))
    }
    
    // Append to nil slice
    fmt.Println("\n📊 Append to Nil Slice (works!):")
    var nilSlice []int
    nilSlice = append(nilSlice, 1, 2, 3)
    fmt.Printf("   nil slice after append: %v\n", nilSlice)
    
    // ⚠️ MUST reassign!
    fmt.Println("\n⚠️ MUST Reassign Result of append:")
    fmt.Println("   s = append(s, x)  ✅ Correct")
    fmt.Println("   append(s, x)      ❌ Result lost!")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           THE APPEND FUNCTION                             ║
╚══════════════════════════════════════════════════════════╝

📊 Basic Append:
   Before: s = [1 2 3], len=3, cap=3
   After append(s, 4): s = [1 2 3 4], len=4, cap=6

📊 Append Multiple Values:
   append(s, 5, 6, 7) = [1 2 3 4 5 6 7]

📊 Append Slice to Slice:
   append(s, more...) = [1 2 3 4 5 6 7 8 9 10]

📊 Capacity Growth (watch it double!):
   len=1, cap=1
   len=2, cap=2
   len=3, cap=4
   len=4, cap=4
   len=5, cap=8
   len=6, cap=8
   len=7, cap=8
   len=8, cap=8
   len=9, cap=16
   len=10, cap=16

📊 Append to Nil Slice (works!):
   nil slice after append: [1 2 3]

⚠️ MUST Reassign Result of append:
   s = append(s, x)  ✅ Correct
   append(s, x)      ❌ Result lost!
```

---

## ⚠️ Critical: Shared Backing Array!

```go
// shared_backing.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║        ⚠️ SHARED BACKING ARRAY (GOTCHA!)                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // The problem
    fmt.Println("\n⚠️ The Problem:")
    original := []int{1, 2, 3, 4, 5}
    sub := original[1:4]  // [2, 3, 4]
    
    fmt.Printf("   original = %v\n", original)
    fmt.Printf("   sub = original[1:4] = %v\n", sub)
    
    // Modifying sub affects original!
    sub[0] = 999
    
    fmt.Println("\n   After sub[0] = 999:")
    fmt.Printf("   original = %v (ALSO CHANGED!)\n", original)
    fmt.Printf("   sub = %v\n", sub)
    
    // Why?
    fmt.Println("\n   Why? Both point to SAME underlying array!")
    
    // The solution: copy
    fmt.Println("\n✅ Solution: Use copy()")
    original2 := []int{1, 2, 3, 4, 5}
    sub2 := make([]int, 3)
    copy(sub2, original2[1:4])  // Independent copy
    
    sub2[0] = 999
    fmt.Printf("   original2 = %v (unchanged!)\n", original2)
    fmt.Printf("   sub2 = %v\n", sub2)
    
    // Append can also share
    fmt.Println("\n⚠️ Append Can Also Share:")
    base := make([]int, 3, 10)  // len=3, cap=10
    base[0], base[1], base[2] = 1, 2, 3
    
    slice1 := append(base, 4)  // Still uses same backing array!
    slice2 := append(base, 5)  // Also uses same backing array!
    
    fmt.Printf("   base = %v\n", base)
    fmt.Printf("   slice1 = append(base, 4) = %v\n", slice1)
    fmt.Printf("   slice2 = append(base, 5) = %v\n", slice2)
    fmt.Println("   Notice: slice1[3] was overwritten by slice2's append!")
    
    // Full slice expression prevents this
    fmt.Println("\n✅ Prevention: Full Slice Expression")
    base2 := make([]int, 3, 10)
    base2[0], base2[1], base2[2] = 1, 2, 3
    
    limited := base2[0:3:3]  // cap = 3 (forces new allocation on append)
    slice3 := append(limited, 4)
    slice4 := append(limited, 5)
    
    fmt.Printf("   slice3 = %v\n", slice3)
    fmt.Printf("   slice4 = %v (independent!)\n", slice4)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║        ⚠️ SHARED BACKING ARRAY (GOTCHA!)                  ║
╚══════════════════════════════════════════════════════════╝

⚠️ The Problem:
   original = [1 2 3 4 5]
   sub = original[1:4] = [2 3 4]

   After sub[0] = 999:
   original = [1 999 3 4 5] (ALSO CHANGED!)
   sub = [999 3 4]

   Why? Both point to SAME underlying array!

✅ Solution: Use copy()
   original2 = [1 2 3 4 5] (unchanged!)
   sub2 = [999 3 4]

⚠️ Append Can Also Share:
   base = [1 2 3]
   slice1 = append(base, 4) = [1 2 3 4]
   slice2 = append(base, 5) = [1 2 3 5]
   Notice: slice1[3] was overwritten by slice2's append!

✅ Prevention: Full Slice Expression
   slice3 = [1 2 3 4]
   slice4 = [1 2 3 5] (independent!)
```

---

## 📋 The copy Function

```go
// copy_demo.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           THE COPY FUNCTION                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic copy
    fmt.Println("\n📊 Basic copy:")
    src := []int{1, 2, 3, 4, 5}
    dst := make([]int, len(src))
    
    n := copy(dst, src)  // Returns number copied
    fmt.Printf("   copy(dst, src) copied %d elements\n", n)
    fmt.Printf("   src = %v\n", src)
    fmt.Printf("   dst = %v (independent copy)\n", dst)
    
    // Copy to smaller destination
    fmt.Println("\n📊 Copy to Smaller Destination:")
    small := make([]int, 3)
    n = copy(small, src)
    fmt.Printf("   Copied %d elements to smaller slice\n", n)
    fmt.Printf("   small = %v\n", small)
    
    // Copy overlapping (within same slice)
    fmt.Println("\n📊 Copy Overlapping (shift elements):")
    s := []int{0, 1, 2, 3, 4, 5}
    fmt.Printf("   Before: %v\n", s)
    copy(s[0:], s[2:])  // Shift left
    fmt.Printf("   After copy(s[0:], s[2:]): %v\n", s)
    
    // Copy string to byte slice
    fmt.Println("\n📊 Copy String to []byte:")
    bytes := make([]byte, 10)
    copy(bytes, "Hello")
    fmt.Printf("   bytes = %v (%q)\n", bytes, bytes)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           THE COPY FUNCTION                               ║
╚══════════════════════════════════════════════════════════╝

📊 Basic copy:
   copy(dst, src) copied 5 elements
   src = [1 2 3 4 5]
   dst = [1 2 3 4 5] (independent copy)

📊 Copy to Smaller Destination:
   Copied 3 elements to smaller slice
   small = [1 2 3]

📊 Copy Overlapping (shift elements):
   Before: [0 1 2 3 4 5]
   After copy(s[0:], s[2:]): [2 3 4 5 4 5]

📊 Copy String to []byte:
   bytes = [72 101 108 108 111 0 0 0 0 0] ("Hello")
```

---

## 🏭 Production Patterns

```go
// slice_production.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION SLICE PATTERNS                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: Pre-allocate when size known
    fmt.Println("\n📊 Pattern 1: Pre-allocate (avoid reallocations)")
    n := 1000
    // Bad: starts with cap 0, many reallocations
    // result := []int{}
    // Good: pre-allocate
    result := make([]int, 0, n)
    for i := 0; i < n; i++ {
        result = append(result, i)
    }
    fmt.Printf("   Pre-allocated: len=%d, cap=%d\n", len(result), cap(result))
    
    // Pattern 2: Filter in place
    fmt.Println("\n📊 Pattern 2: Filter in Place (no allocation)")
    data := []int{1, -2, 3, -4, 5, -6, 7}
    fmt.Printf("   Before filter: %v\n", data)
    data = filterPositive(data)
    fmt.Printf("   After filter: %v\n", data)
    
    // Pattern 3: Stack operations
    fmt.Println("\n📊 Pattern 3: Stack (LIFO)")
    stack := []int{}
    // Push
    stack = append(stack, 1)
    stack = append(stack, 2)
    stack = append(stack, 3)
    fmt.Printf("   Stack after push 1,2,3: %v\n", stack)
    // Pop
    top := stack[len(stack)-1]
    stack = stack[:len(stack)-1]
    fmt.Printf("   Popped: %d, Stack: %v\n", top, stack)
    
    // Pattern 4: Queue operations
    fmt.Println("\n📊 Pattern 4: Queue (FIFO)")
    queue := []int{}
    // Enqueue
    queue = append(queue, 1)
    queue = append(queue, 2)
    queue = append(queue, 3)
    fmt.Printf("   Queue after enqueue 1,2,3: %v\n", queue)
    // Dequeue
    front := queue[0]
    queue = queue[1:]
    fmt.Printf("   Dequeued: %d, Queue: %v\n", front, queue)
    
    // Pattern 5: Remove element by index
    fmt.Println("\n📊 Pattern 5: Remove by Index")
    items := []string{"a", "b", "c", "d", "e"}
    fmt.Printf("   Before: %v\n", items)
    items = removeAt(items, 2)  // Remove "c"
    fmt.Printf("   After removeAt(2): %v\n", items)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           PRODUCTION SLICE PATTERNS                       ║
╚══════════════════════════════════════════════════════════╝

📊 Pattern 1: Pre-allocate (avoid reallocations)
   Pre-allocated: len=1000, cap=1000

📊 Pattern 2: Filter in Place (no allocation)
   Before filter: [1 -2 3 -4 5 -6 7]
   After filter: [1 3 5 7]

📊 Pattern 3: Stack (LIFO)
   Stack after push 1,2,3: [1 2 3]
   Popped: 3, Stack: [1 2]

📊 Pattern 4: Queue (FIFO)
   Queue after enqueue 1,2,3: [1 2 3]
   Dequeued: 1, Queue: [2 3]

📊 Pattern 5: Remove by Index
   Before: [a b c d e]
   After removeAt(2): [a b d e]
```

// Filter positive numbers in place
func filterPositive(nums []int) []int {
    n := 0
    for _, v := range nums {
        if v > 0 {
            nums[n] = v
            n++
        }
    }
    return nums[:n]
}

// Remove element at index
func removeAt(s []string, i int) []string {
    return append(s[:i], s[i+1:]...)
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
│  ArrayList<Integer> list           []int slice                  │
│  = new ArrayList<>();              := []int{}                   │
│                                                                 │
│  list.add(x);                      slice = append(slice, x)     │
│                                                                 │
│  list.get(i);                      slice[i]                     │
│                                                                 │
│  list.size();                      len(slice)                   │
│                                                                 │
│  list.subList(1, 4);               slice[1:4]                   │
│                                                                 │
│  // Sublist is a VIEW             // Slice is a VIEW            │
│  // (changes affect original)      // (changes affect original) │
│                                                                 │
│  // Copy:                          // Copy:                     │
│  new ArrayList<>(list)             copy(dst, src)               │
│                                                                 │
│  int[] arr = new int[5];           arr := make([]int, 5)        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Slice = pointer + length + capacity** (not the data itself!)
2. **Dynamic size** - can grow with `append`
3. **Views share data** - modifications affect original!
4. **Use `copy()`** for independent copies
5. **`append` MUST be reassigned** - `s = append(s, x)`
6. **nil slice is valid** - len=0, cap=0, works with append
7. **Pre-allocate** when size is known for performance
8. **Full slice expression** `s[low:high:max]` limits capacity

---

## ➡️ Next Steps

**Next Topic:** [22 - Maps](./22-maps.md)

