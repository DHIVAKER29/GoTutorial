# 20 - Arrays

> Fixed-size collections in Go - understanding the foundation before slices.

---

## 📌 What You'll Learn

- What arrays are and how they differ from other languages
- Why Go has both arrays AND slices
- Array declaration, initialization, and operations
- Memory layout and performance implications
- When to use arrays vs slices
- Common pitfalls and best practices
- Sample programs for every concept

---

## 🤔 What is an Array?

### Definition

> An **array** is a fixed-size, ordered collection of elements of the same type.

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ARRAY = APARTMENT BUILDING WITH FIXED UNITS                    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  SUNSHINE APARTMENTS (5 units)                          │   │
│  │                                                         │   │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐              │   │
│  │  │Unit │ │Unit │ │Unit │ │Unit │ │Unit │              │   │
│  │  │  0  │ │  1  │ │  2  │ │  3  │ │  4  │              │   │
│  │  │     │ │     │ │     │ │     │ │     │              │   │
│  │  │Alice│ │ Bob │ │Empty│ │Carol│ │Dave │              │   │
│  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘              │   │
│  │                                                         │   │
│  │  • Fixed size: Always 5 units                           │   │
│  │  • Same type: Only humans can live here                 │   │
│  │  • Indexed: Unit 0, 1, 2, 3, 4                         │   │
│  │  • Can't add Unit 5: Building is complete!              │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  In Go:                                                         │
│  var apartments [5]string                                       │
│  apartments[0] = "Alice"                                        │
│  apartments[1] = "Bob"                                          │
│  // apartments[5] = "Eve"  ❌ Index out of bounds!              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Array Memory Layout

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  HOW ARRAYS ARE STORED IN MEMORY                                │
│                                                                 │
│  var nums [5]int = {10, 20, 30, 40, 50}                        │
│                                                                 │
│  Memory (contiguous block):                                     │
│  ┌────────┬────────┬────────┬────────┬────────┐                │
│  │   10   │   20   │   30   │   40   │   50   │                │
│  └────────┴────────┴────────┴────────┴────────┘                │
│  Address:  1000     1008     1016     1024     1032             │
│  Index:      0        1        2        3        4              │
│                                                                 │
│  • Elements are CONTIGUOUS (next to each other)                 │
│  • Each int takes 8 bytes (on 64-bit system)                    │
│  • Total size: 5 × 8 = 40 bytes                                 │
│  • Access any element in O(1) - just calculate offset           │
│                                                                 │
│  Address of nums[i] = Base Address + (i × element size)         │
│  nums[3] = 1000 + (3 × 8) = 1024                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Array Declaration & Initialization

### All the Ways to Create Arrays

```go
// array_declaration.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           ARRAY DECLARATION & INITIALIZATION              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Method 1: Declare with zero values
    fmt.Println("\n📊 Method 1: Declaration (zero values)")
    var arr1 [5]int  // All elements are 0
    fmt.Printf("   var arr1 [5]int = %v\n", arr1)
    
    var arr2 [3]string  // All elements are ""
    fmt.Printf("   var arr2 [3]string = %q\n", arr2)
    
    var arr3 [2]bool  // All elements are false
    fmt.Printf("   var arr3 [2]bool = %v\n", arr3)
    
    // Method 2: Declaration with initialization
    fmt.Println("\n📊 Method 2: Declaration with values")
    var arr4 [3]int = [3]int{10, 20, 30}
    fmt.Printf("   var arr4 [3]int = [3]int{10, 20, 30} = %v\n", arr4)
    
    // Method 3: Short declaration with values
    fmt.Println("\n📊 Method 3: Short declaration")
    arr5 := [4]string{"apple", "banana", "cherry", "date"}
    fmt.Printf("   arr5 := [4]string{...} = %v\n", arr5)
    
    // Method 4: Compiler counts the elements
    fmt.Println("\n📊 Method 4: Inferred length with ...")
    arr6 := [...]int{1, 2, 3, 4, 5, 6, 7}  // Compiler counts: 7
    fmt.Printf("   arr6 := [...]int{...} = %v (len=%d)\n", arr6, len(arr6))
    
    // Method 5: Indexed initialization
    fmt.Println("\n📊 Method 5: Indexed initialization")
    arr7 := [5]int{0: 100, 2: 200, 4: 300}  // Specific indices
    fmt.Printf("   arr7 := [5]int{0:100, 2:200, 4:300} = %v\n", arr7)
    
    // Method 6: Mixed indexed and sequential
    fmt.Println("\n📊 Method 6: Mixed initialization")
    arr8 := [5]int{1, 2, 4: 99}  // 1, 2, 0, 0, 99
    fmt.Printf("   arr8 := [5]int{1, 2, 4:99} = %v\n", arr8)
    
    // Method 7: Last index determines length
    fmt.Println("\n📊 Method 7: Length from last index")
    arr9 := [...]int{9: 100}  // Creates [10]int with arr9[9]=100
    fmt.Printf("   arr9 := [...]int{9:100} = %v (len=%d)\n", arr9, len(arr9))
    
    // Multi-dimensional arrays
    fmt.Println("\n📊 Multi-dimensional Arrays:")
    matrix := [2][3]int{
        {1, 2, 3},
        {4, 5, 6},
    }
    fmt.Printf("   matrix := [2][3]int = %v\n", matrix)
    fmt.Printf("   matrix[1][2] = %d\n", matrix[1][2])
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           ARRAY DECLARATION & INITIALIZATION              ║
╚══════════════════════════════════════════════════════════╝

📊 Method 1: Declaration (zero values)
   var arr1 [5]int = [0 0 0 0 0]
   var arr2 [3]string = ["" "" ""]
   var arr3 [2]bool = [false false]

📊 Method 2: Declaration with values
   var arr4 [3]int = [3]int{10, 20, 30} = [10 20 30]

📊 Method 3: Short declaration
   arr5 := [4]string{...} = [apple banana cherry date]

📊 Method 4: Inferred length with ...
   arr6 := [...]int{...} = [1 2 3 4 5 6 7] (len=7)

📊 Method 5: Indexed initialization
   arr7 := [5]int{0:100, 2:200, 4:300} = [100 0 200 0 300]

📊 Method 6: Mixed initialization
   arr8 := [5]int{1, 2, 4:99} = [1 2 0 0 99]

📊 Method 7: Length from last index
   arr9 := [...]int{9:100} = [0 0 0 0 0 0 0 0 0 100] (len=10)

📊 Multi-dimensional Arrays:
   matrix := [2][3]int = [[1 2 3] [4 5 6]]
   matrix[1][2] = 6
```

---

## 🔧 Array Operations

### Accessing, Modifying, and Iterating

```go
// array_operations.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              ARRAY OPERATIONS                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    arr := [5]int{10, 20, 30, 40, 50}
    
    // Accessing elements
    fmt.Println("\n📊 Accessing Elements:")
    fmt.Printf("   arr = %v\n", arr)
    fmt.Printf("   arr[0] = %d (first element)\n", arr[0])
    fmt.Printf("   arr[4] = %d (last element)\n", arr[4])
    fmt.Printf("   arr[len(arr)-1] = %d (last element, dynamic)\n", arr[len(arr)-1])
    
    // Modifying elements
    fmt.Println("\n📊 Modifying Elements:")
    arr[2] = 300
    fmt.Printf("   arr[2] = 300 → %v\n", arr)
    
    // Length
    fmt.Println("\n📊 Length:")
    fmt.Printf("   len(arr) = %d\n", len(arr))
    
    // Iteration Method 1: Classic for loop
    fmt.Println("\n📊 Iteration - Classic for loop:")
    for i := 0; i < len(arr); i++ {
        fmt.Printf("   arr[%d] = %d\n", i, arr[i])
    }
    
    // Iteration Method 2: Range (preferred)
    fmt.Println("\n📊 Iteration - Range (preferred):")
    for index, value := range arr {
        fmt.Printf("   arr[%d] = %d\n", index, value)
    }
    
    // Iteration Method 3: Range, index only
    fmt.Println("\n📊 Iteration - Index only:")
    for i := range arr {
        fmt.Printf("   Index: %d\n", i)
    }
    
    // Iteration Method 4: Range, value only
    fmt.Println("\n📊 Iteration - Value only:")
    fmt.Print("   Values: ")
    for _, value := range arr {
        fmt.Printf("%d ", value)
    }
    fmt.Println()
    
    // Comparing arrays
    fmt.Println("\n📊 Comparing Arrays:")
    a := [3]int{1, 2, 3}
    b := [3]int{1, 2, 3}
    c := [3]int{1, 2, 4}
    
    fmt.Printf("   a = %v, b = %v, c = %v\n", a, b, c)
    fmt.Printf("   a == b : %t\n", a == b)
    fmt.Printf("   a == c : %t\n", a == c)
    
    // Finding max/min
    fmt.Println("\n📊 Finding Max Value:")
    max := arr[0]
    for _, v := range arr {
        if v > max {
            max = v
        }
    }
    fmt.Printf("   Max of %v = %d\n", arr, max)
    
    // Sum of elements
    fmt.Println("\n📊 Sum of Elements:")
    sum := 0
    for _, v := range arr {
        sum += v
    }
    fmt.Printf("   Sum of %v = %d\n", arr, sum)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              ARRAY OPERATIONS                             ║
╚══════════════════════════════════════════════════════════╝

📊 Accessing Elements:
   arr = [10 20 30 40 50]
   arr[0] = 10 (first element)
   arr[4] = 50 (last element)
   arr[len(arr)-1] = 50 (last element, dynamic)

📊 Modifying Elements:
   arr[2] = 300 → [10 20 300 40 50]

📊 Length:
   len(arr) = 5

📊 Iteration - Classic for loop:
   arr[0] = 10
   arr[1] = 20
   arr[2] = 300
   arr[3] = 40
   arr[4] = 50

📊 Iteration - Range (preferred):
   arr[0] = 10
   arr[1] = 20
   arr[2] = 300
   arr[3] = 40
   arr[4] = 50

📊 Iteration - Index only:
   Index: 0
   Index: 1
   Index: 2
   Index: 3
   Index: 4

📊 Iteration - Value only:
   Values: 10 20 300 40 50 

📊 Comparing Arrays:
   a = [1 2 3], b = [1 2 3], c = [1 2 4]
   a == b : true
   a == c : false

📊 Finding Max Value:
   Max of [10 20 300 40 50] = 300

📊 Sum of Elements:
   Sum of [10 20 300 40 50] = 420
```

---

## ⚠️ Critical: Size is Part of the Type!

```go
// array_type.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           ARRAY SIZE IS PART OF TYPE!                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n⚠️ [3]int and [4]int are DIFFERENT TYPES!")
    
    var a [3]int = [3]int{1, 2, 3}
    var b [4]int = [4]int{1, 2, 3, 4}
    
    fmt.Printf("   Type of a: %T\n", a)
    fmt.Printf("   Type of b: %T\n", b)
    
    // This would be a COMPILE ERROR:
    // a = b  // ❌ cannot use b (type [4]int) as type [3]int
    
    fmt.Println("\n📊 Why This Matters:")
    fmt.Println("   • Cannot assign [4]int to [3]int variable")
    fmt.Println("   • Cannot pass [3]int to function expecting [4]int")
    fmt.Println("   • Cannot compare [3]int with [4]int")
    
    // Function example
    fmt.Println("\n📊 Function Parameters:")
    arr3 := [3]int{10, 20, 30}
    processArray3(arr3)  // ✅ Works
    
    // processArray3(b)  // ❌ COMPILE ERROR: wrong type
    
    fmt.Println("\n💡 This is why SLICES are preferred!")
    fmt.Println("   Slices work with any length.")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           ARRAY SIZE IS PART OF TYPE!                     ║
╚══════════════════════════════════════════════════════════╝

⚠️ [3]int and [4]int are DIFFERENT TYPES!
   Type of a: [3]int
   Type of b: [4]int

📊 Why This Matters:
   • Cannot assign [4]int to [3]int variable
   • Cannot pass [3]int to function expecting [4]int
   • Cannot compare [3]int with [4]int

📊 Function Parameters:
   Processing [3]int: [10 20 30]

💡 This is why SLICES are preferred!
   Slices work with any length.
```

func processArray3(arr [3]int) {  // Only accepts [3]int!
    fmt.Printf("   Processing [3]int: %v\n", arr)
}
```

---

## ⚠️ Arrays are Values (Copied!)

```go
// array_copy.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           ARRAYS ARE VALUES (COPIED!)                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 Assignment Creates a Copy:")
    original := [5]int{1, 2, 3, 4, 5}
    copy := original  // FULL COPY! Not a reference!
    
    fmt.Printf("   original = %v\n", original)
    fmt.Printf("   copy     = %v\n", copy)
    
    // Modify the copy
    copy[0] = 100
    
    fmt.Println("\n📊 After modifying copy[0] = 100:")
    fmt.Printf("   original = %v (unchanged!)\n", original)
    fmt.Printf("   copy     = %v (changed)\n", copy)
    
    // Passing to function
    fmt.Println("\n📊 Passing to Function (also copied!):")
    arr := [3]int{10, 20, 30}
    fmt.Printf("   Before function: %v\n", arr)
    modifyArray(arr)
    fmt.Printf("   After function:  %v (unchanged!)\n", arr)
    
    // Using pointer to modify
    fmt.Println("\n📊 Using Pointer to Modify Original:")
    modifyArrayPtr(&arr)
    fmt.Printf("   After function with pointer: %v (changed!)\n", arr)
    
    // Performance implication
    fmt.Println("\n⚠️ Performance Implication:")
    fmt.Println("   Large arrays copied on assignment/function call")
    fmt.Println("   [1000000]int = ~8MB copied each time!")
    fmt.Println("   Solution: Use pointers or slices")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           ARRAYS ARE VALUES (COPIED!)                     ║
╚══════════════════════════════════════════════════════════╝

📊 Assignment Creates a Copy:
   original = [1 2 3 4 5]
   copy     = [1 2 3 4 5]

📊 After modifying copy[0] = 100:
   original = [1 2 3 4 5] (unchanged!)
   copy     = [100 2 3 4 5] (changed)

📊 Passing to Function (also copied!):
   Before function: [10 20 30]
   Inside function: [999 20 30]
   After function:  [10 20 30] (unchanged!)

📊 Using Pointer to Modify Original:
   After function with pointer: [999 20 30] (changed!)

⚠️ Performance Implication:
   Large arrays copied on assignment/function call
   [1000000]int = ~8MB copied each time!
   Solution: Use pointers or slices
```

func modifyArray(arr [3]int) {  // Receives a COPY
    arr[0] = 999  // Modifies the copy, not original
    fmt.Printf("   Inside function: %v\n", arr)
}

func modifyArrayPtr(arr *[3]int) {  // Receives a POINTER
    arr[0] = 999  // Modifies through pointer
}
```

---

## 🆚 Arrays vs Slices

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ARRAYS vs SLICES                                               │
│                                                                 │
│  ┌─────────────────┬───────────────────────────────────────┐   │
│  │     FEATURE     │   ARRAY          │   SLICE           │   │
│  ├─────────────────┼───────────────────┼───────────────────┤   │
│  │ Size            │ Fixed            │ Dynamic           │   │
│  │ Type includes   │ Yes [5]int       │ No []int          │   │
│  │ Passed by       │ Value (copy)     │ Reference         │   │
│  │ Comparable (==) │ Yes              │ No                │   │
│  │ Can grow        │ No               │ Yes (append)      │   │
│  │ Zero value      │ [n]T with zeros  │ nil               │   │
│  │ Use frequency   │ Rare (~5%)       │ Common (~95%)     │   │
│  └─────────────────┴───────────────────┴───────────────────┘   │
│                                                                 │
│  WHEN TO USE ARRAYS:                                            │
│  • Fixed-size data (e.g., IP address [4]byte)                   │
│  • Cryptographic keys (e.g., [32]byte)                          │
│  • Matrix operations with known dimensions                      │
│  • When you need value semantics (copy on assign)               │
│                                                                 │
│  WHEN TO USE SLICES (almost always):                            │
│  • Dynamic-size collections                                     │
│  • Function parameters (accept any size)                        │
│  • Growing/shrinking data                                       │
│  • When you want reference semantics                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🏭 Production Use Cases

```go
// array_production.go
package main

import (
    "crypto/sha256"
    "fmt"
    "net"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION USE CASES FOR ARRAYS                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Use Case 1: IP Address (always 4 bytes for IPv4)
    fmt.Println("\n📊 Use Case 1: IP Address")
    var ipv4 [4]byte = [4]byte{192, 168, 1, 1}
    fmt.Printf("   IPv4: %d.%d.%d.%d\n", ipv4[0], ipv4[1], ipv4[2], ipv4[3])
    
    // Go's net package uses this
    ip := net.IPv4(192, 168, 1, 1)
    fmt.Printf("   net.IPv4: %s\n", ip)
    
    // Use Case 2: Cryptographic Hash (always 32 bytes for SHA256)
    fmt.Println("\n📊 Use Case 2: SHA256 Hash")
    data := []byte("Hello, World!")
    hash := sha256.Sum256(data)  // Returns [32]byte
    fmt.Printf("   Hash type: %T\n", hash)
    fmt.Printf("   Hash: %x\n", hash)
    
    // Use Case 3: RGB Color (always 3 components)
    fmt.Println("\n📊 Use Case 3: RGB Color")
    type RGB [3]uint8
    red := RGB{255, 0, 0}
    green := RGB{0, 255, 0}
    blue := RGB{0, 0, 255}
    fmt.Printf("   Red:   RGB%v\n", red)
    fmt.Printf("   Green: RGB%v\n", green)
    fmt.Printf("   Blue:  RGB%v\n", blue)
    
    // Use Case 4: Coordinates (2D or 3D)
    fmt.Println("\n📊 Use Case 4: Coordinates")
    type Point2D [2]float64
    type Point3D [3]float64
    
    p2 := Point2D{3.5, 4.5}
    p3 := Point3D{1.0, 2.0, 3.0}
    fmt.Printf("   2D Point: %v\n", p2)
    fmt.Printf("   3D Point: %v\n", p3)
    
    // Use Case 5: Fixed buffer
    fmt.Println("\n📊 Use Case 5: Fixed Buffer")
    var buffer [1024]byte  // 1KB buffer
    copy(buffer[:], "Hello")
    fmt.Printf("   Buffer (first 10 bytes): %v\n", buffer[:10])
    
    // Use Case 6: Lookup tables
    fmt.Println("\n📊 Use Case 6: Lookup Table (Days in Month)")
    // Index 1-12 for months (0 unused)
    daysInMonth := [13]int{0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}
    fmt.Printf("   Days in February: %d\n", daysInMonth[2])
    fmt.Printf("   Days in December: %d\n", daysInMonth[12])
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           PRODUCTION USE CASES FOR ARRAYS                 ║
╚══════════════════════════════════════════════════════════╝

📊 Use Case 1: IP Address
   IPv4: 192.168.1.1
   net.IPv4: 192.168.1.1

📊 Use Case 2: SHA256 Hash
   Hash type: [32]uint8
   Hash: 65a8e27d8879283831b664bd8b7f0ad4...

📊 Use Case 3: RGB Color
   Red:   RGB[255 0 0]
   Green: RGB[0 255 0]
   Blue:  RGB[0 0 255]

📊 Use Case 4: Coordinates
   2D Point: [3.5 4.5]
   3D Point: [1 2 3]

📊 Use Case 5: Fixed Buffer
   Buffer (first 10 bytes): [72 101 108 108 111 0 0 0 0 0]

📊 Use Case 6: Lookup Table (Days in Month)
   Days in February: 28
   Days in December: 31
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  int[] arr = new int[5];           var arr [5]int               │
│  // Size not in type               // Size IS the type!         │
│                                                                 │
│  int[] a = {1, 2, 3};              a := [3]int{1, 2, 3}         │
│  int[] b = {1, 2, 3};              b := [3]int{1, 2, 3}         │
│  a == b  // false (reference)      a == b  // true (value)      │
│                                                                 │
│  void process(int[] arr)           func process(arr [3]int)     │
│  // Works with any size            // Only works with [3]int!   │
│                                                                 │
│  arr.length                        len(arr)                     │
│                                                                 │
│  // Arrays are reference types     // Arrays are VALUE types    │
│  int[] b = a;  // Same reference   b := a  // Full copy!        │
│  b[0] = 99;    // a[0] also 99     b[0] = 99  // a unchanged    │
│                                                                 │
│  KEY DIFFERENCE:                                                │
│  Java arrays are reference types (objects on heap)              │
│  Go arrays are value types (copied on assignment)               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Fixed size** - cannot grow or shrink after creation
2. **Size is part of type** - `[3]int` ≠ `[4]int`
3. **Value type** - assignment/passing creates a copy
4. **Zero-indexed** - first element is `arr[0]`
5. **Comparable** - can use `==` to compare
6. **Contiguous memory** - fast, cache-friendly
7. **Use slices instead** for most cases (~95%)
8. **Arrays for**: fixed-size data, crypto keys, buffers

---

## ➡️ Next Steps

Arrays are the foundation. Now let's learn about slices - the much more common and flexible data structure.

**Next Topic:** [21 - Slices](./21-slices.md)
