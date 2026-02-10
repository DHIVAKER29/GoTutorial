# 07 - Data Types - Basics

> Understanding Go's fundamental data types: numbers, booleans, and strings.

---

## 📌 What You'll Learn

- All numeric types in Go (int, float, etc.)
- Boolean type and its uses
- String type basics
- Type defaults and when to use each
- Sample programs for every type
- Java/OOP comparison

---

## 🤔 Why Data Types Matter

### The Problem Without Types

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  WITHOUT TYPES (Dynamic Languages)                              │
│                                                                 │
│  x = 5                                                          │
│  x = "hello"      # Same variable, now a string?                │
│  x = x + 10       # Error at RUNTIME!                           │
│                                                                 │
│  Problems:                                                      │
│  • Errors discovered when code runs (too late!)                 │
│  • Hard to understand what a variable contains                  │
│  • No compile-time help from IDE                                │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  WITH TYPES (Go)                                                │
│                                                                 │
│  var x int = 5                                                  │
│  x = "hello"      // COMPILE ERROR! Type mismatch               │
│                                                                 │
│  Benefits:                                                      │
│  • Errors caught at compile time (before running)               │
│  • Clear what each variable holds                               │
│  • IDE can autocomplete and catch mistakes                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Numeric Types Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO NUMERIC TYPES                                               │
│                                                                 │
│  INTEGERS (Whole Numbers)                                       │
│  ─────────────────────────                                      │
│  Signed (can be negative):                                      │
│  • int8   : -128 to 127                                         │
│  • int16  : -32,768 to 32,767                                   │
│  • int32  : -2.1 billion to 2.1 billion                         │
│  • int64  : -9.2 quintillion to 9.2 quintillion                 │
│  • int    : Platform dependent (32 or 64 bit)                   │
│                                                                 │
│  Unsigned (positive only):                                      │
│  • uint8  : 0 to 255 (also called byte)                         │
│  • uint16 : 0 to 65,535                                         │
│  • uint32 : 0 to 4.2 billion                                    │
│  • uint64 : 0 to 18.4 quintillion                               │
│  • uint   : Platform dependent                                  │
│                                                                 │
│  Special:                                                       │
│  • byte   : Alias for uint8 (raw data)                          │
│  • rune   : Alias for int32 (Unicode code point)                │
│  • uintptr: Pointer arithmetic (advanced)                       │
│                                                                 │
│  FLOATING POINT (Decimals)                                      │
│  ─────────────────────────                                      │
│  • float32 : ~7 decimal digits precision                        │
│  • float64 : ~15 decimal digits precision (DEFAULT)             │
│                                                                 │
│  COMPLEX NUMBERS                                                │
│  ───────────────                                                │
│  • complex64  : float32 real and imaginary parts                │
│  • complex128 : float64 real and imaginary parts                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔢 Integer Types

### When to Use Each

| Type | Size | Range | Use Case |
|------|------|-------|----------|
| `int` | 32/64 bit | Platform dependent | **DEFAULT choice** |
| `int8` | 8 bit | -128 to 127 | Small numbers, memory-critical |
| `int16` | 16 bit | -32K to 32K | Rarely used |
| `int32` | 32 bit | -2B to 2B | Specific protocols |
| `int64` | 64 bit | Huge range | Timestamps, large IDs, money (paisa) |
| `uint` | 32/64 bit | Platform dependent | Positive only |
| `byte` | 8 bit | 0 to 255 | Raw bytes, binary data |

### Sample Program: Integer Types

```go
// integer_types.go
package main

import (
    "fmt"
    "math"
    "unsafe"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              INTEGER TYPES IN GO                          ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Default int type
    var count int = 42
    fmt.Printf("\n📊 int (default):\n")
    fmt.Printf("   Value: %d\n", count)
    fmt.Printf("   Size:  %d bytes\n", unsafe.Sizeof(count))
    
    // Specific size integers
    var tiny int8 = 127     // Max value for int8
    var small int16 = 32767
    var medium int32 = 2147483647
    var large int64 = 9223372036854775807
    
    fmt.Println("\n📊 Signed Integers (can be negative):")
    fmt.Printf("   int8  : %d (max: %d, min: %d)\n", tiny, math.MaxInt8, math.MinInt8)
    fmt.Printf("   int16 : %d (max: %d)\n", small, math.MaxInt16)
    fmt.Printf("   int32 : %d (max: %d)\n", medium, math.MaxInt32)
    fmt.Printf("   int64 : %d\n", large)
    
    // Unsigned integers
    var uTiny uint8 = 255   // Max value for uint8
    var uSmall uint16 = 65535
    var uMedium uint32 = 4294967295
    
    fmt.Println("\n📊 Unsigned Integers (positive only):")
    fmt.Printf("   uint8  : %d (max: %d)\n", uTiny, math.MaxUint8)
    fmt.Printf("   uint16 : %d (max: %d)\n", uSmall, math.MaxUint16)
    fmt.Printf("   uint32 : %d (max: %d)\n", uMedium, math.MaxUint32)
    
    // byte is alias for uint8
    var rawByte byte = 'A'  // ASCII value of 'A' is 65
    fmt.Println("\n📊 byte (alias for uint8):")
    fmt.Printf("   byte 'A' = %d (decimal) = %c (character)\n", rawByte, rawByte)
    
    // Practical examples
    fmt.Println("\n💡 Practical Use Cases:")
    
    // Money in paisa (avoid floating point!)
    var amountInPaisa int64 = 99950  // ₹999.50
    fmt.Printf("   Amount: ₹%.2f (stored as %d paisa)\n", 
        float64(amountInPaisa)/100, amountInPaisa)
    
    // Unix timestamp
    var timestamp int64 = 1705555200  // Jan 18, 2024
    fmt.Printf("   Timestamp: %d seconds since 1970\n", timestamp)
    
    // Counting
    var visitors int = 1000000
    fmt.Printf("   Visitors: %d\n", visitors)
    
    // Negative temperatures
    var temperature int = -15
    fmt.Printf("   Temperature: %d°C\n", temperature)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              INTEGER TYPES IN GO                          ║
╚══════════════════════════════════════════════════════════╝

📊 int (default):
   Value: 42
   Size:  8 bytes

📊 Signed Integers (can be negative):
   int8  : 127 (max: 127, min: -128)
   int16 : 32767 (max: 32767)
   int32 : 2147483647 (max: 2147483647)
   int64 : 9223372036854775807

📊 Unsigned Integers (positive only):
   uint8  : 255 (max: 255)
   uint16 : 65535 (max: 65535)
   uint32 : 4294967295 (max: 4294967295)

📊 byte (alias for uint8):
   byte 'A' = 65 (decimal) = A (character)

💡 Practical Use Cases:
   Amount: ₹999.50 (stored as 99950 paisa)
   Timestamp: 1705555200 seconds since 1970
   Visitors: 1000000
   Temperature: -15°C
```

### Why `int64` for Money?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MONEY: WHY int64 NOT float64?                                  │
│                                                                 │
│  PROBLEM WITH FLOATS:                                           │
│  0.1 + 0.2 = 0.30000000000000004  (NOT 0.3!)                    │
│                                                                 │
│  This happens because floats use binary representation.         │
│  Some decimals can't be represented exactly.                    │
│                                                                 │
│  SOLUTION: Store as smallest unit (paisa/cents)                 │
│                                                                 │
│  ₹999.99 → 99999 paisa (int64)                                  │
│                                                                 │
│  Benefits:                                                      │
│  ✅ Exact arithmetic                                            │
│  ✅ No rounding errors                                          │
│  ✅ Can store huge amounts (9 quintillion paisa!)               │
│                                                                 │
│  In Catalyst codebase:                                          │
│  type Payment struct {                                          │
│      Amount int64 `json:"amount"` // in paisa                   │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔢 Floating Point Types

### When to Use

| Type | Precision | Use Case |
|------|-----------|----------|
| `float64` | ~15 digits | **DEFAULT** - Most calculations |
| `float32` | ~7 digits | Memory-critical, graphics |

### Sample Program: Float Types

```go
// float_types.go
package main

import (
    "fmt"
    "math"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           FLOATING POINT TYPES IN GO                      ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Default float (float64)
    pi := 3.14159265358979323846
    fmt.Printf("\n📊 float64 (default):\n")
    fmt.Printf("   Pi = %.15f\n", pi)
    fmt.Printf("   Precision: ~15 decimal digits\n")
    
    // float32 vs float64
    var f32 float32 = 1.123456789012345
    var f64 float64 = 1.123456789012345
    
    fmt.Println("\n📊 Precision Comparison:")
    fmt.Printf("   float32: %.15f (loses precision after ~7 digits)\n", f32)
    fmt.Printf("   float64: %.15f (accurate to ~15 digits)\n", f64)
    
    // Special values
    fmt.Println("\n📊 Special Float Values:")
    fmt.Printf("   +Inf: %f\n", math.Inf(1))
    fmt.Printf("   -Inf: %f\n", math.Inf(-1))
    fmt.Printf("   NaN (Not a Number): %f\n", math.NaN())
    fmt.Printf("   Max float64: %e\n", math.MaxFloat64)
    fmt.Printf("   Smallest positive: %e\n", math.SmallestNonzeroFloat64)
    
    // The famous floating point problem
    fmt.Println("\n⚠️ Famous Floating Point Problem:")
    result := 0.1 + 0.2
    fmt.Printf("   0.1 + 0.2 = %.17f\n", result)
    fmt.Printf("   Expected:   0.30000000000000000\n")
    fmt.Printf("   Equal to 0.3? %t\n", result == 0.3)
    
    // How to compare floats
    fmt.Println("\n✅ How to Compare Floats:")
    epsilon := 0.0000001
    areEqual := math.Abs(result-0.3) < epsilon
    fmt.Printf("   Using epsilon comparison: %t\n", areEqual)
    
    // Practical examples
    fmt.Println("\n💡 Practical Use Cases:")
    
    // Percentages
    percentage := 75.5
    fmt.Printf("   Discount: %.1f%%\n", percentage)
    
    // Scientific calculations
    distance := 1.496e8  // 149.6 million km (Earth to Sun)
    fmt.Printf("   Earth-Sun distance: %.3e km\n", distance)
    
    // Coordinates
    lat, lon := 12.9716, 77.5946
    fmt.Printf("   Bangalore coordinates: %.4f, %.4f\n", lat, lon)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           FLOATING POINT TYPES IN GO                      ║
╚══════════════════════════════════════════════════════════╝

📊 float64 (default):
   Pi = 3.141592653589793
   Precision: ~15 decimal digits

📊 Precision Comparison:
   float32: 1.123456835746765 (loses precision after ~7 digits)
   float64: 1.123456789012345 (accurate to ~15 digits)

📊 Special Float Values:
   +Inf: +Inf
   -Inf: -Inf
   NaN (Not a Number): NaN
   Max float64: 1.797693e+308
   Smallest positive: 5.000000e-324

⚠️ Famous Floating Point Problem:
   0.1 + 0.2 = 0.30000000000000004
   Expected:   0.30000000000000000
   Equal to 0.3? false

✅ How to Compare Floats:
   Using epsilon comparison: true

💡 Practical Use Cases:
   Discount: 75.5%
   Earth-Sun distance: 1.496e+08 km
   Bangalore coordinates: 12.9716, 77.5946
```

---

## ✅ Boolean Type

### Definition

> A boolean (`bool`) can only be `true` or `false`.

### Sample Program: Boolean Type

```go
// boolean_types.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              BOOLEAN TYPE IN GO                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Declaration
    var isActive bool = true
    var isDeleted bool = false
    var hasPermission bool  // Zero value is false
    
    fmt.Println("\n📊 Boolean Basics:")
    fmt.Printf("   isActive: %t\n", isActive)
    fmt.Printf("   isDeleted: %t\n", isDeleted)
    fmt.Printf("   hasPermission (zero value): %t\n", hasPermission)
    
    // Comparison operators return bool
    fmt.Println("\n📊 Comparison Operators (return bool):")
    x, y := 10, 20
    fmt.Printf("   %d == %d : %t\n", x, y, x == y)
    fmt.Printf("   %d != %d : %t\n", x, y, x != y)
    fmt.Printf("   %d > %d  : %t\n", x, y, x > y)
    fmt.Printf("   %d < %d  : %t\n", x, y, x < y)
    fmt.Printf("   %d >= %d : %t\n", x, y, x >= y)
    fmt.Printf("   %d <= %d : %t\n", x, y, x <= y)
    
    // Logical operators
    fmt.Println("\n📊 Logical Operators:")
    a, b := true, false
    fmt.Printf("   %t && %t  = %t (AND)\n", a, b, a && b)
    fmt.Printf("   %t || %t  = %t (OR)\n", a, b, a || b)
    fmt.Printf("   !%t       = %t (NOT)\n", a, !a)
    fmt.Printf("   !%t      = %t (NOT)\n", b, !b)
    
    // Short-circuit evaluation
    fmt.Println("\n📊 Short-Circuit Evaluation:")
    fmt.Println("   If first part of && is false, second part not evaluated")
    fmt.Println("   If first part of || is true, second part not evaluated")
    
    count := 0
    _ = false && increment(&count)  // increment not called!
    fmt.Printf("   false && func() - func called? count = %d\n", count)
    
    _ = true || increment(&count)   // increment not called!
    fmt.Printf("   true || func() - func called? count = %d\n", count)
    
    // Practical examples
    fmt.Println("\n💡 Practical Use Cases:")
    
    // Feature flags
    isFeatureEnabled := true
    if isFeatureEnabled {
        fmt.Println("   New feature is enabled!")
    }
    
    // Validation
    isValid := len("hello") > 0 && len("hello") < 100
    fmt.Printf("   Input valid? %t\n", isValid)
    
    // Status checks
    isLoggedIn := true
    isAdmin := false
    canAccessAdmin := isLoggedIn && isAdmin
    fmt.Printf("   Can access admin panel? %t\n", canAccessAdmin)
    
    // No implicit conversion!
    fmt.Println("\n⚠️ Go vs Other Languages:")
    fmt.Println("   In JavaScript: if (1) → true")
    fmt.Println("   In Go: if 1 → COMPILE ERROR!")
    fmt.Println("   Go requires explicit bool expressions")
}

func increment(count *int) bool {
    *count++
    return true
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              BOOLEAN TYPE IN GO                           ║
╚══════════════════════════════════════════════════════════╝

📊 Boolean Basics:
   isActive: true
   isDeleted: false
   hasPermission (zero value): false

📊 Comparison Operators (return bool):
   10 == 20 : false
   10 != 20 : true
   10 > 20  : false
   10 < 20  : true
   10 >= 20 : false
   10 <= 20 : true

📊 Logical Operators:
   true && false  = false (AND)
   true || false  = true (OR)
   !true       = false (NOT)
   !false      = true (NOT)

📊 Short-Circuit Evaluation:
   If first part of && is false, second part not evaluated
   If first part of || is true, second part not evaluated
   false && func() - func called? count = 0
   true || func() - func called? count = 0

💡 Practical Use Cases:
   New feature is enabled!
   Input valid? true
   Can access admin panel? false

⚠️ Go vs Other Languages:
   In JavaScript: if (1) → true
   In Go: if 1 → COMPILE ERROR!
   Go requires explicit bool expressions
```

### No Implicit Boolean Conversion

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO IS STRICT ABOUT BOOLEANS                                    │
│                                                                 │
│  JavaScript/Python:                                             │
│  if (1)       → true (truthy)                                   │
│  if (0)       → false (falsy)                                   │
│  if ("hello") → true (truthy)                                   │
│  if ("")      → false (falsy)                                   │
│  if (null)    → false (falsy)                                   │
│                                                                 │
│  Go:                                                            │
│  if 1         → COMPILE ERROR!                                  │
│  if count     → COMPILE ERROR!                                  │
│  if name      → COMPILE ERROR!                                  │
│                                                                 │
│  Must be explicit:                                              │
│  if count > 0     → OK                                          │
│  if name != ""    → OK                                          │
│  if ptr != nil    → OK                                          │
│                                                                 │
│  Why? Prevents bugs from accidental truthiness!                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 String Type

### Definition

> A `string` is an immutable sequence of bytes (typically UTF-8 encoded text).

### Key Characteristics

| Property | Description |
|----------|-------------|
| Immutable | Cannot change individual characters |
| UTF-8 | Default encoding for Go source files |
| Zero value | `""` (empty string) |
| Comparison | Uses `==`, `<`, `>` (lexicographic) |

### Sample Program: String Basics

```go
// string_types.go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              STRING TYPE IN GO                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // String declaration
    var greeting string = "Hello, World!"
    name := "Go Developer"
    var empty string  // Zero value is ""
    
    fmt.Println("\n📊 String Basics:")
    fmt.Printf("   greeting = %q\n", greeting)
    fmt.Printf("   name = %q\n", name)
    fmt.Printf("   empty = %q (zero value)\n", empty)
    
    // String length
    fmt.Println("\n📊 String Length:")
    text := "Hello"
    hindi := "नमस्ते"
    emoji := "👋"
    
    fmt.Printf("   len(%q) = %d bytes\n", text, len(text))
    fmt.Printf("   len(%q) = %d bytes (not 6 characters!)\n", hindi, len(hindi))
    fmt.Printf("   len(%q) = %d bytes (not 1 character!)\n", emoji, len(emoji))
    
    // String concatenation
    fmt.Println("\n📊 String Concatenation:")
    first := "Hello"
    second := "World"
    combined := first + ", " + second + "!"
    fmt.Printf("   %q + %q = %q\n", first, second, combined)
    
    // Multi-line strings (raw strings)
    fmt.Println("\n📊 Raw Strings (backticks):")
    raw := `This is a
    multi-line string.
    Special chars like \n are literal.`
    fmt.Printf("   Raw string:\n%s\n", raw)
    
    // String indexing (returns byte, not character!)
    fmt.Println("\n📊 String Indexing:")
    word := "Hello"
    fmt.Printf("   word[0] = %d (%c) - returns byte!\n", word[0], word[0])
    fmt.Printf("   word[4] = %d (%c)\n", word[4], word[4])
    
    // Strings are immutable
    fmt.Println("\n📊 Strings are Immutable:")
    fmt.Println("   s := \"Hello\"")
    fmt.Println("   s[0] = 'h'  // ❌ COMPILE ERROR!")
    fmt.Println("   s = \"hello\" // ✅ Reassign entire string is OK")
    
    // String comparison
    fmt.Println("\n📊 String Comparison:")
    a, b := "apple", "banana"
    fmt.Printf("   %q == %q : %t\n", a, a, a == a)
    fmt.Printf("   %q != %q : %t\n", a, b, a != b)
    fmt.Printf("   %q < %q  : %t (lexicographic)\n", a, b, a < b)
    
    // Common string operations
    fmt.Println("\n💡 Common String Operations:")
    sample := "  Hello, Go World!  "
    
    fmt.Printf("   Original: %q\n", sample)
    fmt.Printf("   Trimmed:  %q\n", strings.TrimSpace(sample))
    fmt.Printf("   Upper:    %q\n", strings.ToUpper(sample))
    fmt.Printf("   Lower:    %q\n", strings.ToLower(sample))
    fmt.Printf("   Contains 'Go': %t\n", strings.Contains(sample, "Go"))
    fmt.Printf("   Replace:  %q\n", strings.Replace(sample, "Go", "Golang", 1))
    
    // String splitting
    csv := "apple,banana,cherry"
    parts := strings.Split(csv, ",")
    fmt.Printf("   Split %q: %v\n", csv, parts)
    
    // String joining
    joined := strings.Join(parts, " | ")
    fmt.Printf("   Join with ' | ': %q\n", joined)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              STRING TYPE IN GO                            ║
╚══════════════════════════════════════════════════════════╝

📊 String Basics:
   greeting = "Hello, World!"
   name = "Go Developer"
   empty = "" (zero value)

📊 String Length:
   len("Hello") = 5 bytes
   len("नमस्ते") = 18 bytes (not 6 characters!)
   len("👋") = 4 bytes (not 1 character!)

📊 String Concatenation:
   "Hello" + "World" = "Hello, World!"

📊 Raw Strings (backticks):
   Raw string:
This is a
    multi-line string.
    Special chars like \n are literal.

📊 String Indexing:
   word[0] = 72 (H) - returns byte!
   word[4] = 111 (o)

📊 Strings are Immutable:
   s := "Hello"
   s[0] = 'h'  // ❌ COMPILE ERROR!
   s = "hello" // ✅ Reassign entire string is OK

📊 String Comparison:
   "apple" == "apple" : true
   "apple" != "banana" : true
   "apple" < "banana"  : true (lexicographic)

💡 Common String Operations:
   Original: "  Hello, Go World!  "
   Trimmed:  "Hello, Go World!"
   Upper:    "  HELLO, GO WORLD!  "
   Lower:    "  hello, go world!  "
   Contains 'Go': true
   Replace:  "  Hello, Golang World!  "
   Split "apple,banana,cherry": [apple banana cherry]
   Join with ' | ': "apple | banana | cherry"
```

---

## 🆚 Java Comparison: Data Types

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  INTEGERS:                                                      │
│  byte    (8-bit)                   int8                         │
│  short   (16-bit)                  int16                        │
│  int     (32-bit)                  int32 or int                 │
│  long    (64-bit)                  int64                        │
│                                                                 │
│  FLOATING POINT:                                                │
│  float   (32-bit)                  float32                      │
│  double  (64-bit)                  float64 (default)            │
│                                                                 │
│  BOOLEAN:                                                       │
│  boolean (true/false)              bool (true/false)            │
│                                                                 │
│  STRING:                                                        │
│  String (object, null-able)        string (value, zero="")      │
│                                                                 │
│  CHAR:                                                          │
│  char (16-bit UTF-16)              rune (32-bit Unicode)        │
│                                                                 │
│  KEY DIFFERENCES:                                               │
│  • Java int is always 32-bit, Go int is platform-dependent      │
│  • Java String can be null, Go string cannot (zero is "")       │
│  • Java char is 16-bit, Go rune is 32-bit (full Unicode)        │
│  • Go has unsigned types (uint), Java doesn't (mostly)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **`int`** is the default for integers (use it unless you need specific size)
2. **`int64`** for money, timestamps, large IDs
3. **`float64`** is the default for decimals (not `float32`)
4. **`bool`** is strictly `true` or `false` (no truthy/falsy!)
5. **`string`** is immutable and UTF-8 encoded
6. **`len(string)`** returns bytes, not characters!
7. **Zero values**: `int=0`, `float64=0.0`, `bool=false`, `string=""`

---

## 🔢 Type Defaults Summary

| Need | Use | Why |
|------|-----|-----|
| Counting, indexing | `int` | Platform-optimized |
| Money (paisa/cents) | `int64` | Exact, no rounding |
| Timestamps | `int64` | Unix epoch fits |
| General decimals | `float64` | Better precision |
| Yes/No values | `bool` | Only true/false |
| Text | `string` | UTF-8, immutable |
| Raw bytes | `[]byte` | Binary data |

---

## ➡️ Next Steps

Now that you understand basic types, let's dive deep into strings, bytes, and runes.

**Next Topic:** [08 - Strings, Bytes & Runes](./08-strings-bytes-runes.md)

