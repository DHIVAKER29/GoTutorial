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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  IMPLICIT CONVERSIONS (Other Languages)                         │
│                                                                 │
│  JavaScript:                                                    │
│  "5" + 3 = "53"      // String concatenation!                   │
│  "5" - 3 = 2         // Numeric subtraction!                    │
│  "5" * "3" = 15      // Numeric multiplication!                 │
│                                                                 │
│  C:                                                             │
│  int i = 1000000;                                               │
│  short s = i;        // Silently truncates! s = 16960          │
│                                                                 │
│  Problems:                                                      │
│  ❌ Unexpected behavior                                         │
│  ❌ Silent data loss                                            │
│  ❌ Hard to debug                                               │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  GO: EXPLICIT CONVERSIONS ONLY                                  │
│                                                                 │
│  var i int = 5                                                  │
│  var f float64 = i   // ❌ COMPILE ERROR!                       │
│  var f float64 = float64(i)  // ✅ Explicit conversion          │
│                                                                 │
│  Benefits:                                                      │
│  ✅ No surprises                                                │
│  ✅ Intentional conversions are visible                         │
│  ✅ Compiler catches type mismatches                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔢 Numeric Conversions

### Basic Syntax

```go
T(value)  // Convert 'value' to type 'T'
```

### Sample Program: Numeric Conversions

```go
// numeric_conversions.go
package main

import (
    "fmt"
    "math"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           NUMERIC CONVERSIONS IN GO                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // int to float64
    fmt.Println("\n📊 int → float64")
    var i int = 42
    var f float64 = float64(i)
    fmt.Printf("   int %d → float64 %f\n", i, f)
    
    // float64 to int (truncates decimal!)
    fmt.Println("\n📊 float64 → int (TRUNCATES!)")
    var pi float64 = 3.14159
    var piInt int = int(pi)
    fmt.Printf("   float64 %f → int %d\n", pi, piInt)
    
    var negative float64 = -3.7
    var negInt int = int(negative)
    fmt.Printf("   float64 %f → int %d (truncates toward zero)\n", negative, negInt)
    
    // int32 to int64 (safe, larger type)
    fmt.Println("\n📊 Smaller → Larger (Safe)")
    var small int32 = 100
    var large int64 = int64(small)
    fmt.Printf("   int32 %d → int64 %d ✅\n", small, large)
    
    // int64 to int32 (DANGER: overflow possible!)
    fmt.Println("\n⚠️ Larger → Smaller (DANGER!)")
    var big int64 = 3000000000  // 3 billion
    var truncated int32 = int32(big)
    fmt.Printf("   int64 %d → int32 %d (OVERFLOW!)\n", big, truncated)
    
    // Demonstrating safe conversion check
    fmt.Println("\n✅ Safe Conversion Pattern:")
    safeBig := int64(1000000)
    if safeBig >= math.MinInt32 && safeBig <= math.MaxInt32 {
        safeSmall := int32(safeBig)
        fmt.Printf("   int64 %d → int32 %d (safe)\n", safeBig, safeSmall)
    } else {
        fmt.Printf("   int64 %d would overflow int32!\n", safeBig)
    }
    
    // byte to int
    fmt.Println("\n📊 byte ↔ int")
    var b byte = 65  // ASCII 'A'
    var intVal int = int(b)
    fmt.Printf("   byte %d → int %d\n", b, intVal)
    
    // rune to int
    fmt.Println("\n📊 rune ↔ int")
    var r rune = '中'  // Chinese character
    var runeInt int = int(r)
    fmt.Printf("   rune '%c' → int %d (U+%04X)\n", r, runeInt, runeInt)
    
    // Different int sizes
    fmt.Println("\n📊 Same 'int' family still needs conversion")
    var a int = 10
    var b32 int32 = 20
    // sum := a + b32  // ❌ COMPILE ERROR!
    sum := a + int(b32)  // ✅ Must convert
    fmt.Printf("   int %d + int32 %d = int %d\n", a, b32, sum)
}
```

### Conversion Rules

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  NUMERIC CONVERSION RULES                                       │
│                                                                 │
│  1. Float → Int: TRUNCATES (doesn't round!)                     │
│     3.9 → 3                                                     │
│     -3.9 → -3 (toward zero)                                     │
│                                                                 │
│  2. Larger → Smaller: May OVERFLOW!                             │
│     int64(3000000000) → int32(-1294967296)                      │
│     Check bounds before converting!                             │
│                                                                 │
│  3. Smaller → Larger: Always SAFE                               │
│     int8 → int32 → int64 ✅                                     │
│                                                                 │
│  4. Signed → Unsigned: Watch for negatives!                     │
│     int(-1) → uint = huge number (wrap around)                  │
│                                                                 │
│  5. Different int types ALWAYS need conversion                  │
│     int + int32 = ERROR (must convert one)                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 String Conversions

### Using strconv Package

```go
// string_conversions.go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           STRING CONVERSIONS (strconv)                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // int to string
    fmt.Println("\n📊 Int → String")
    num := 42
    str := strconv.Itoa(num)  // Integer to ASCII
    fmt.Printf("   strconv.Itoa(%d) = %q\n", num, str)
    
    // Alternative: FormatInt for more control
    bigNum := int64(1234567890)
    strBig := strconv.FormatInt(bigNum, 10)  // base 10
    fmt.Printf("   strconv.FormatInt(%d, 10) = %q\n", bigNum, strBig)
    
    // Different bases
    fmt.Println("\n📊 Int → String (Different Bases)")
    n := int64(255)
    fmt.Printf("   Decimal:     %s\n", strconv.FormatInt(n, 10))
    fmt.Printf("   Binary:      %s\n", strconv.FormatInt(n, 2))
    fmt.Printf("   Octal:       %s\n", strconv.FormatInt(n, 8))
    fmt.Printf("   Hexadecimal: %s\n", strconv.FormatInt(n, 16))
    
    // String to int
    fmt.Println("\n📊 String → Int")
    strNum := "123"
    intNum, err := strconv.Atoi(strNum)  // ASCII to Integer
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   strconv.Atoi(%q) = %d\n", strNum, intNum)
    }
    
    // ParseInt for more control
    strBigNum := "9223372036854775807"  // Max int64
    parsedBig, err := strconv.ParseInt(strBigNum, 10, 64)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   strconv.ParseInt(%q) = %d\n", strBigNum, parsedBig)
    }
    
    // Float to string
    fmt.Println("\n📊 Float → String")
    pi := 3.14159265358979
    piStr := strconv.FormatFloat(pi, 'f', 2, 64)  // format, precision, bitSize
    fmt.Printf("   strconv.FormatFloat(pi, 'f', 2, 64) = %q\n", piStr)
    
    piStrE := strconv.FormatFloat(pi, 'e', 4, 64)  // scientific notation
    fmt.Printf("   strconv.FormatFloat(pi, 'e', 4, 64) = %q\n", piStrE)
    
    // String to float
    fmt.Println("\n📊 String → Float")
    floatStr := "3.14159"
    parsedFloat, err := strconv.ParseFloat(floatStr, 64)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   strconv.ParseFloat(%q) = %f\n", floatStr, parsedFloat)
    }
    
    // Bool to string
    fmt.Println("\n📊 Bool ↔ String")
    boolVal := true
    boolStr := strconv.FormatBool(boolVal)
    fmt.Printf("   strconv.FormatBool(%t) = %q\n", boolVal, boolStr)
    
    parsedBool, _ := strconv.ParseBool("true")
    fmt.Printf("   strconv.ParseBool(%q) = %t\n", "true", parsedBool)
    
    // Error handling
    fmt.Println("\n⚠️ Error Handling (invalid input)")
    invalid := "not-a-number"
    _, err = strconv.Atoi(invalid)
    if err != nil {
        fmt.Printf("   strconv.Atoi(%q) error: %v\n", invalid, err)
    }
}
```

### Quick Reference: strconv Functions

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  strconv QUICK REFERENCE                                        │
│                                                                 │
│  INT ↔ STRING:                                                  │
│  ───────────────                                                │
│  strconv.Itoa(int) string          // int → string              │
│  strconv.Atoi(string) (int, error) // string → int              │
│                                                                 │
│  More control:                                                  │
│  strconv.FormatInt(int64, base) string                          │
│  strconv.ParseInt(string, base, bitSize) (int64, error)         │
│                                                                 │
│  FLOAT ↔ STRING:                                                │
│  ─────────────────                                              │
│  strconv.FormatFloat(float64, fmt, prec, bitSize) string        │
│  strconv.ParseFloat(string, bitSize) (float64, error)           │
│                                                                 │
│  BOOL ↔ STRING:                                                 │
│  ────────────────                                               │
│  strconv.FormatBool(bool) string                                │
│  strconv.ParseBool(string) (bool, error)                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚡ Type Assertions (for Interfaces)

### Sample Program: Type Assertions

```go
// type_assertions.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           TYPE ASSERTIONS                                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // interface{} can hold any value
    var anything interface{}
    
    // Store an int
    anything = 42
    fmt.Println("\n📊 Type Assertion Basics:")
    fmt.Printf("   anything = %v (type: %T)\n", anything, anything)
    
    // Type assertion: get the concrete value
    num := anything.(int)  // Assert it's an int
    fmt.Printf("   anything.(int) = %d\n", num)
    
    // Wrong assertion causes PANIC!
    fmt.Println("\n⚠️ Wrong Assertion = PANIC!")
    fmt.Println("   // anything.(string) would PANIC!")
    
    // Safe type assertion with comma-ok
    fmt.Println("\n✅ Safe Type Assertion (comma-ok idiom):")
    str, ok := anything.(string)
    if ok {
        fmt.Printf("   It's a string: %s\n", str)
    } else {
        fmt.Printf("   Not a string! ok = %t\n", ok)
    }
    
    intVal, ok := anything.(int)
    if ok {
        fmt.Printf("   It's an int: %d\n", intVal)
    }
    
    // Type switch
    fmt.Println("\n📊 Type Switch:")
    describeType(42)
    describeType("hello")
    describeType(3.14)
    describeType(true)
    describeType([]int{1, 2, 3})
}

func describeType(x interface{}) {
    switch v := x.(type) {
    case int:
        fmt.Printf("   %v is an int (doubled: %d)\n", v, v*2)
    case string:
        fmt.Printf("   %q is a string (length: %d)\n", v, len(v))
    case float64:
        fmt.Printf("   %v is a float64\n", v)
    case bool:
        fmt.Printf("   %v is a bool\n", v)
    default:
        fmt.Printf("   %v is unknown type %T\n", v, v)
    }
}
```

---

## 🏭 Production Patterns

### Sample Program: Real-World Conversions

```go
// production_conversions.go
package main

import (
    "encoding/json"
    "fmt"
    "strconv"
)

// Production pattern: Money conversion
func ConvertRupeesToPaisa(rupees float64) int64 {
    // Multiply by 100 and add 0.5 for rounding
    return int64(rupees*100 + 0.5)
}

func ConvertPaisaToRupees(paisa int64) float64 {
    return float64(paisa) / 100
}

// Production pattern: Safe string to int with default
func ParseIntOrDefault(s string, defaultVal int) int {
    val, err := strconv.Atoi(s)
    if err != nil {
        return defaultVal
    }
    return val
}

// Production pattern: JSON number handling
type Payment struct {
    Amount json.Number `json:"amount"` // Can be int or float in JSON
    ID     string      `json:"id"`
}

func (p Payment) GetAmountAsInt() (int64, error) {
    return p.Amount.Int64()
}

func (p Payment) GetAmountAsFloat() (float64, error) {
    return p.Amount.Float64()
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION CONVERSION PATTERNS                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Money conversion
    fmt.Println("\n💰 Money Conversion:")
    rupees := 999.99
    paisa := ConvertRupeesToPaisa(rupees)
    fmt.Printf("   ₹%.2f → %d paisa\n", rupees, paisa)
    fmt.Printf("   %d paisa → ₹%.2f\n", paisa, ConvertPaisaToRupees(paisa))
    
    // Safe parsing
    fmt.Println("\n📊 Safe Parsing with Default:")
    fmt.Printf("   ParseIntOrDefault(\"42\", 0) = %d\n", ParseIntOrDefault("42", 0))
    fmt.Printf("   ParseIntOrDefault(\"bad\", 0) = %d\n", ParseIntOrDefault("bad", 0))
    fmt.Printf("   ParseIntOrDefault(\"\", -1) = %d\n", ParseIntOrDefault("", -1))
    
    // Query parameter handling
    fmt.Println("\n📊 Query Parameter Pattern:")
    queryParams := map[string]string{
        "page":  "5",
        "limit": "invalid",
    }
    page := ParseIntOrDefault(queryParams["page"], 1)
    limit := ParseIntOrDefault(queryParams["limit"], 10)
    fmt.Printf("   page = %d, limit = %d\n", page, limit)
    
    // JSON number handling
    fmt.Println("\n📊 JSON Number Handling:")
    jsonData := `{"amount": 99.99, "id": "PAY001"}`
    var payment Payment
    json.Unmarshal([]byte(jsonData), &payment)
    
    amountInt, _ := payment.GetAmountAsInt()
    amountFloat, _ := payment.GetAmountAsFloat()
    fmt.Printf("   JSON amount as int: %d\n", amountInt)
    fmt.Printf("   JSON amount as float: %.2f\n", amountFloat)
}
```

---

## ⚠️ Common Pitfalls

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  COMMON CONVERSION PITFALLS                                     │
│                                                                 │
│  1. FLOAT → INT TRUNCATES (doesn't round!)                      │
│     int(3.9) = 3, NOT 4                                         │
│     Use math.Round() if you want rounding                       │
│                                                                 │
│  2. OVERFLOW IS SILENT                                          │
│     int32(3000000000) wraps around silently                     │
│     Always check bounds before narrowing conversion             │
│                                                                 │
│  3. string(int) DOESN'T WORK AS EXPECTED                        │
│     string(65) = "A" (treats as rune/code point!)               │
│     Use strconv.Itoa(65) = "65"                                 │
│                                                                 │
│  4. INTERFACE ASSERTION CAN PANIC                               │
│     x.(int) panics if x is not int                              │
│     Always use x, ok := x.(int)                                 │
│                                                                 │
│  5. FLOAT PRECISION LOSS                                        │
│     float32(3.14159265358979) loses precision                   │
│     Prefer float64 for calculations                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  IMPLICIT (widening):              ALWAYS EXPLICIT:             │
│  int i = 5;                        var i int = 5                │
│  long l = i;  // auto              var l int64 = int64(i)       │
│                                                                 │
│  EXPLICIT (narrowing):             EXPLICIT:                    │
│  double d = 3.14;                  var d float64 = 3.14         │
│  int i = (int) d;                  var i int = int(d)           │
│                                                                 │
│  STRING PARSING:                   STRING PARSING:              │
│  Integer.parseInt("42")            strconv.Atoi("42")           │
│  Double.parseDouble("3.14")        strconv.ParseFloat("3.14")   │
│                                                                 │
│  TO STRING:                        TO STRING:                   │
│  String.valueOf(42)                strconv.Itoa(42)             │
│  Integer.toString(42)              strconv.FormatInt(42, 10)    │
│                                                                 │
│  TYPE CHECKING:                    TYPE ASSERTION:              │
│  if (obj instanceof String)        if s, ok := obj.(string); ok │
│  Object to (String) cast           obj.(string)                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **All conversions are explicit** in Go - no surprises!
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

