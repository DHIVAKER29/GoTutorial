# 09 - Type Conversions

> Understanding how Go converts between types safely and explicitly.

---

## 📌 What You'll Learn

- Why Go requires explicit type conversions
- How to convert between numeric types
- String conversions (strconv package)
- Type assertions for interfaces
- Common pitfalls and how to avoid them
- Sample programs for each conversion

---

## 🤔 Why Explicit Conversions?

### The Problem with Implicit Conversions

**JavaScript:** `"5" + 3 = "53"` (string concat), `"5" - 3 = 2` (numeric) — same operator, different behavior.

**C:** `short s = i` when `i` is large — silently truncates.

**Problems:** Unexpected behavior, silent data loss, hard to debug.

### Go: Explicit Conversions Only

```go
var i int = 5
var f float64 = i        // ❌ COMPILE ERROR!
var f float64 = float64(i) // ✅ Explicit conversion
```

**Benefits:** No surprises, intentional conversions are visible, compiler catches mismatches.

---

## 🔢 Numeric Conversions

### Basic Syntax

```go
T(value)  // Convert 'value' to type 'T'
```

### Snippets

```go
var i int = 42
var f float64 = float64(i)
fmt.Println(f) // Output: 42
```

```go
pi := 3.14159
piInt := int(pi)
fmt.Println(piInt) // Output: 3 (truncates, doesn't round!)
```

```go
var big int64 = 3000000000
var small int32 = int32(big)
fmt.Println(small) // Output: -1294967296 (overflow!)
```

```go
var a int = 10
var b32 int32 = 20
sum := a + int(b32)  // Must convert
fmt.Println(sum) // Output: 30
```

### Numeric Conversion Rules

| Rule | Example |
|------|---------|
| Float → Int truncates | `int(3.9)` = 3, not 4 |
| Larger → Smaller may overflow | `int32(3000000000)` wraps |
| Smaller → Larger is safe | `int8` → `int64` ✅ |
| Signed → Unsigned: watch negatives | `uint(-1)` = huge number |
| Different int types need conversion | `int + int32` = error |

### Complete Example: Numeric Conversions

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    i := 42
    f := float64(i)
    fmt.Println(f) // 42

    pi := 3.14159
    fmt.Println(int(pi)) // 3

    big := int64(3000000000)
    small := int32(big)
    fmt.Println(small) // -1294967296 (overflow)

    safeBig := int64(1000000)
    if safeBig >= math.MinInt32 && safeBig <= math.MaxInt32 {
        safe := int32(safeBig)
        fmt.Println(safe) // 1000000 (safe conversion)
    }
}
```

**Output:**
```
42
3
-1294967296
1000000
```

---

## 📝 String Conversions

### Using strconv Package

```go
import "strconv"

num := 42
str := strconv.Itoa(num)
fmt.Println(str) // Output: 42
```

```go
strNum := "123"
intNum, err := strconv.Atoi(strNum)
fmt.Println(intNum, err) // Output: 123 <nil>
```

```go
n := int64(255)
fmt.Println(strconv.FormatInt(n, 10))  // 255
fmt.Println(strconv.FormatInt(n, 16))  // ff
fmt.Println(strconv.FormatInt(n, 2))   // 11111111
```

```go
pi := 3.14159
piStr := strconv.FormatFloat(pi, 'f', 2, 64)
fmt.Println(piStr) // Output: 3.14
```

```go
parsed, err := strconv.ParseFloat("3.14", 64)
fmt.Println(parsed, err) // Output: 3.14 <nil>
```

```go
fmt.Println(strconv.FormatBool(true))   // true
parsed, _ := strconv.ParseBool("true")
fmt.Println(parsed) // Output: true
```

### strconv Quick Reference

| Function | Purpose |
|----------|---------|
| `strconv.Itoa(int)` | int → string |
| `strconv.Atoi(string)` | string → int |
| `strconv.FormatInt(int64, base)` | int64 → string |
| `strconv.ParseInt(string, base, bitSize)` | string → int64 |
| `strconv.FormatFloat(...)` | float64 → string |
| `strconv.ParseFloat(string, bitSize)` | string → float64 |
| `strconv.FormatBool(bool)` | bool → string |
| `strconv.ParseBool(string)` | string → bool |

### Complete Example: String Conversions

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    fmt.Println(strconv.Itoa(42))
    fmt.Println(strconv.FormatInt(255, 16))

    n, _ := strconv.Atoi("123")
    fmt.Println(n)

    piStr := strconv.FormatFloat(3.14159, 'f', 2, 64)
    fmt.Println(piStr)

    b, _ := strconv.ParseBool("true")
    fmt.Println(b)

    _, err := strconv.Atoi("not-a-number")
    fmt.Println(err)
}
```

**Output:**
```
42
ff
123
3.14
true
strconv.Atoi: parsing "not-a-number": invalid syntax
```

---

## ⚡ Type Assertions (for Interfaces)

### Snippets

```go
var anything interface{} = 42
num := anything.(int)
fmt.Println(num) // Output: 42
```

```go
str, ok := anything.(string)
if ok {
    fmt.Println(str)
} else {
    fmt.Println("Not a string") // Output: Not a string
}
```

```go
switch v := x.(type) {
case int:
    fmt.Printf("int: %d\n", v)
case string:
    fmt.Printf("string: %s\n", v)
default:
    fmt.Printf("unknown: %T\n", v)
}
```

### Complete Example: Type Assertions

```go
package main

import "fmt"

func main() {
    var x interface{} = 42
    num := x.(int)
    fmt.Println(num)

    _, ok := x.(string)
    fmt.Println(ok) // false

    describe(42)
    describe("hello")
}

func describe(x interface{}) {
    switch v := x.(type) {
    case int:
        fmt.Printf("int: %d\n", v)
    case string:
        fmt.Printf("string: %q\n", v)
    default:
        fmt.Printf("unknown: %T\n", v)
    }
}
```

**Output:**
```
42
false
int: 42
string: "hello"
```

---

## 🏭 Production Patterns

```go
func ConvertRupeesToPaisa(rupees float64) int64 {
    return int64(rupees*100 + 0.5)
}

func ParseIntOrDefault(s string, defaultVal int) int {
    val, err := strconv.Atoi(s)
    if err != nil {
        return defaultVal
    }
    return val
}
```

### Complete Example: Production Conversions

```go
package main

import (
    "fmt"
    "strconv"
)

func ConvertRupeesToPaisa(rupees float64) int64 {
    return int64(rupees*100 + 0.5)
}

func ParseIntOrDefault(s string, defaultVal int) int {
    val, err := strconv.Atoi(s)
    if err != nil {
        return defaultVal
    }
    return val
}

func main() {
    paisa := ConvertRupeesToPaisa(999.99)
    fmt.Printf("₹999.99 = %d paisa\n", paisa)

    fmt.Println(ParseIntOrDefault("42", 0))
    fmt.Println(ParseIntOrDefault("bad", 0))
}
```

**Output:**
```
₹999.99 = 99999 paisa
42
0
```

---

## ⚠️ Common Pitfalls

| Pitfall | Example | Fix |
|---------|---------|-----|
| Float → Int truncates | `int(3.9)` = 3 | Use `math.Round()` if needed |
| Overflow is silent | `int32(3000000000)` wraps | Check bounds before narrowing |
| `string(int)` is rune! | `string(65)` = "A" | Use `strconv.Itoa(65)` for "65" |
| Type assertion can panic | `x.(int)` | Use `x, ok := x.(int)` |
| Float precision loss | `float32(3.14159...)` | Prefer `float64` |

---

## 🆚 Java Comparison

| Aspect | Java | Go |
|--------|------|-----|
| Widening | `long l = i;` (implicit) | `var l int64 = int64(i)` (explicit) |
| Narrowing | `int i = (int) d;` | `var i int = int(d)` |
| String parsing | `Integer.parseInt("42")` | `strconv.Atoi("42")` |
| To string | `String.valueOf(42)` | `strconv.Itoa(42)` |
| Type check | `obj instanceof String` | `_, ok := obj.(string)` |

---

## 🎯 Key Takeaways

1. **All conversions are explicit** in Go — no surprises!
2. **`T(value)`** syntax for basic type conversions
3. **`strconv`** package for string ↔ number conversions
4. **Type assertions** (`x.(T)`) for interface values
5. **Always use comma-ok** idiom to avoid panics
6. **Watch for overflow** when converting to smaller types
7. **`string(int)`** treats int as a rune, not digits!

---

## ➡️ Next Steps

You now understand type conversions. Let's explore operators in Go.

**Next Topic:** [10 - Operators](./10-operators.md)
