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

### Without Types (Dynamic Languages)

In languages like JavaScript: `x = 5` then `x = "hello"` — same variable, different type. `x + 10` can fail at **runtime**. Errors are discovered too late, and it's unclear what a variable holds.

### With Types (Go)

```go
var x int = 5
x = "hello"  // COMPILE ERROR! Type mismatch
```

**Benefits:** Errors caught at compile time, clear what each variable holds, IDE can autocomplete and catch mistakes.

---

## 📊 Numeric Types Overview

**Integers (signed):** `int8` (-128 to 127), `int16`, `int32`, `int64`, `int` (platform-dependent)

**Integers (unsigned):** `uint8` (0–255, alias `byte`), `uint16`, `uint32`, `uint64`, `uint`

**Special:** `byte` = `uint8`, `rune` = `int32` (Unicode code point)

**Floating point:** `float32` (~7 digits), `float64` (~15 digits, **default**)

**Complex:** `complex64`, `complex128`

---

## 🔢 Integer Types

### When to Use Each

| Type | Size | Range | Use Case |
|------|------|-------|----------|
| `int` | 32/64 bit | Platform dependent | **DEFAULT choice** |
| `int8` | 8 bit | -128 to 127 | Small numbers, memory-critical |
| `int32` | 32 bit | -2B to 2B | Specific protocols |
| `int64` | 64 bit | Huge range | Timestamps, large IDs, money (paisa) |
| `byte` | 8 bit | 0 to 255 | Raw bytes, binary data |

### Snippets

```go
var count int = 42
fmt.Println(count) // Output: 42
```

```go
var amountInPaisa int64 = 99950  // ₹999.50
fmt.Printf("₹%.2f\n", float64(amountInPaisa)/100) // Output: ₹999.50
```

```go
var rawByte byte = 'A'
fmt.Printf("%d %c\n", rawByte, rawByte) // Output: 65 A
```

### Why `int64` for Money?

Floats have precision issues: `0.1 + 0.2 = 0.30000000000000004`. Store money in smallest units (paisa/cents) as `int64` for exact arithmetic and no rounding errors.

### Complete Example: Integer Types

```go
package main

import "fmt"

func main() {
    var count int = 42
    var amountInPaisa int64 = 99950
    var rawByte byte = 'A'

    fmt.Println(count)                                    // 42
    fmt.Printf("₹%.2f\n", float64(amountInPaisa)/100)     // ₹999.50
    fmt.Printf("%d = %c\n", rawByte, rawByte)             // 65 = A
}
```

**Output:**
```
42
₹999.50
65 = A
```

---

## 🔢 Floating Point Types

### When to Use

| Type | Precision | Use Case |
|------|-----------|----------|
| `float64` | ~15 digits | **DEFAULT** — Most calculations |
| `float32` | ~7 digits | Memory-critical, graphics |

### Snippets

```go
pi := 3.141592653589793
fmt.Printf("%.15f\n", pi) // Output: 3.141592653589793
```

```go
result := 0.1 + 0.2
fmt.Printf("%.17f\n", result) // Output: 0.30000000000000004
fmt.Println(result == 0.3)    // Output: false
```

```go
import "math"
result := 0.1 + 0.2
areEqual := math.Abs(result-0.3) < 0.0000001
fmt.Println(areEqual) // Output: true (use epsilon for float comparison)
```

### Complete Example: Float Types

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    pi := 3.14159
    fmt.Printf("Pi: %.5f\n", pi)

    result := 0.1 + 0.2
    fmt.Printf("0.1 + 0.2 = %.17f\n", result)
    fmt.Println("Equal to 0.3?", math.Abs(result-0.3) < 0.0000001)

    fmt.Printf("Max float64: %e\n", math.MaxFloat64)
}
```

**Output:**
```
Pi: 3.14159
0.1 + 0.2 = 0.30000000000000004
Equal to 0.3? true
Max float64: 1.797693e+308
```

---

## ✅ Boolean Type

### Definition

A `bool` can only be `true` or `false`.

### Snippets

```go
var isActive bool = true
var hasPermission bool  // Zero value: false
fmt.Println(isActive, hasPermission) // Output: true false
```

```go
x, y := 10, 20
fmt.Println(x == y, x < y) // Output: false true
```

```go
a, b := true, false
fmt.Println(a && b, a || b, !a) // Output: false true false
```

### No Implicit Boolean Conversion

In JavaScript: `if (1)` → true. In Go: `if 1` → **COMPILE ERROR!** Go requires explicit bool expressions: `if count > 0`, `if name != ""`, `if ptr != nil`.

### Complete Example: Boolean Type

```go
package main

import "fmt"

func main() {
    var isActive bool = true
    var hasPermission bool
    fmt.Println(isActive, hasPermission) // true false

    x, y := 10, 20
    fmt.Println(x == y, x < y) // false true

    a, b := true, false
    fmt.Println(a && b, a || b) // false true
}
```

**Output:**
```
true false
false true
false true
```

---

## 📝 String Type

### Definition

A `string` is an immutable sequence of bytes (typically UTF-8 encoded text).

### Key Characteristics

| Property | Description |
|----------|-------------|
| Immutable | Cannot change individual characters |
| UTF-8 | Default encoding for Go source files |
| Zero value | `""` (empty string) |
| Comparison | Uses `==`, `<`, `>` (lexicographic) |

### Snippets

```go
greeting := "Hello, World!"
fmt.Println(greeting) // Output: Hello, World!
```

```go
text := "Hello"
hindi := "नमस्ते"
fmt.Println(len(text), len(hindi)) // Output: 5 18 (bytes, not characters!)
```

```go
first, second := "Hello", "World"
combined := first + ", " + second + "!"
fmt.Println(combined) // Output: Hello, World!
```

```go
raw := `Multi-line
with \n literal`
// Prints two lines; \n is literal, not escape
fmt.Println(raw)
```

```go
word := "Hello"
fmt.Printf("%d %c\n", word[0], word[0]) // Output: 72 H (returns byte!)
```

### Complete Example: String Basics

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    greeting := "Hello, Go World!"
    fmt.Println(greeting)

    fmt.Println(len("Hello"), len("नमस्ते")) // 5 18

    sample := "  Hello, Go!  "
    fmt.Println(strings.TrimSpace(sample))
    fmt.Println(strings.ToUpper(sample))
    fmt.Println(strings.Contains(sample, "Go"))

    parts := strings.Split("apple,banana,cherry", ",")
    fmt.Println(strings.Join(parts, " | "))
}
```

**Output:**
```
Hello, Go World!
5 18
Hello, Go!
  HELLO, GO WORLD!  
true
apple | banana | cherry
```

---

## 🆚 Java Comparison: Data Types

| Aspect | Java | Go |
|--------|------|-----|
| Integers | `byte`, `short`, `int`, `long` | `int8`, `int16`, `int32`, `int64` |
| Floats | `float`, `double` | `float32`, `float64` (default) |
| Boolean | `boolean` | `bool` |
| String | Object, nullable | Value type, zero = `""` |
| Char | `char` (16-bit UTF-16) | `rune` (32-bit Unicode) |
| int size | Java `int` always 32-bit | Go `int` platform-dependent |
| String null | Can be null | Cannot be null (zero is `""`) |

---

## 🎯 Key Takeaways

1. **`int`** is the default for integers (use unless you need specific size)
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
