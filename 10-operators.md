# 10 - Operators

> Understanding all operators in Go: arithmetic, comparison, logical, bitwise, and assignment.

---

## 📌 What You'll Learn

- All operator types in Go
- Operator precedence
- What Go DOESN'T have (ternary, ++i)
- Sample programs for each operator type

---

## 📊 Operator Categories

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO OPERATORS                                                   │
│                                                                 │
│  1. ARITHMETIC     +  -  *  /  %                                │
│  2. COMPARISON     ==  !=  <  >  <=  >=                         │
│  3. LOGICAL        &&  ||  !                                    │
│  4. BITWISE        &  |  ^  <<  >>  &^                          │
│  5. ASSIGNMENT     =  +=  -=  *=  /=  %=  etc.                  │
│  6. ADDRESS        &  *                                         │
│  7. RECEIVE        <-                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ➕ Arithmetic Operators

```go
// arithmetic_operators.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           ARITHMETIC OPERATORS                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    a, b := 17, 5
    
    fmt.Printf("\n📊 Basic Arithmetic (a=%d, b=%d):\n", a, b)
    fmt.Printf("   a + b = %d (addition)\n", a+b)
    fmt.Printf("   a - b = %d (subtraction)\n", a-b)
    fmt.Printf("   a * b = %d (multiplication)\n", a*b)
    fmt.Printf("   a / b = %d (integer division)\n", a/b)
    fmt.Printf("   a %% b = %d (modulo/remainder)\n", a%b)
    
    // Float division
    fmt.Println("\n📊 Float Division:")
    x, y := 17.0, 5.0
    fmt.Printf("   %.1f / %.1f = %.2f\n", x, y, x/y)
    
    // Increment/Decrement (statements, not expressions!)
    fmt.Println("\n📊 Increment/Decrement:")
    count := 10
    fmt.Printf("   count = %d\n", count)
    count++
    fmt.Printf("   count++ → %d\n", count)
    count--
    fmt.Printf("   count-- → %d\n", count)
    
    // ⚠️ Go difference: ++/-- are STATEMENTS, not expressions
    fmt.Println("\n⚠️ Go Difference:")
    fmt.Println("   ❌ x = count++  (not allowed!)")
    fmt.Println("   ❌ ++count      (no prefix form!)")
    fmt.Println("   ✅ count++      (only postfix statement)")
    
    // Unary operators
    fmt.Println("\n📊 Unary Operators:")
    num := 42
    fmt.Printf("   +num = %d (positive)\n", +num)
    fmt.Printf("   -num = %d (negation)\n", -num)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           ARITHMETIC OPERATORS                            ║
╚══════════════════════════════════════════════════════════╝

📊 Basic Arithmetic (a=17, b=5):
   a + b = 22 (addition)
   a - b = 12 (subtraction)
   a * b = 85 (multiplication)
   a / b = 3 (integer division)
   a % b = 2 (modulo/remainder)

📊 Float Division:
   17.0 / 5.0 = 3.40

📊 Increment/Decrement:
   count = 10
   count++ → 11
   count-- → 10

⚠️ Go Difference:
   ❌ x = count++  (not allowed!)
   ❌ ++count      (no prefix form!)
   ✅ count++      (only postfix statement)

📊 Unary Operators:
   +num = 42 (positive)
   -num = -42 (negation)
```

---

## ⚖️ Comparison Operators

```go
// comparison_operators.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           COMPARISON OPERATORS                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    a, b := 10, 20
    
    fmt.Printf("\n📊 Numeric Comparison (a=%d, b=%d):\n", a, b)
    fmt.Printf("   a == b : %t (equal)\n", a == b)
    fmt.Printf("   a != b : %t (not equal)\n", a != b)
    fmt.Printf("   a < b  : %t (less than)\n", a < b)
    fmt.Printf("   a > b  : %t (greater than)\n", a > b)
    fmt.Printf("   a <= b : %t (less or equal)\n", a <= b)
    fmt.Printf("   a >= b : %t (greater or equal)\n", a >= b)
    
    // String comparison (lexicographic)
    fmt.Println("\n📊 String Comparison (lexicographic):")
    s1, s2 := "apple", "banana"
    fmt.Printf("   %q == %q : %t\n", s1, s2, s1 == s2)
    fmt.Printf("   %q < %q  : %t (a comes before b)\n", s1, s2, s1 < s2)
    fmt.Printf("   %q > %q  : %t\n", s1, s2, s1 > s2)
    
    // Comparing same types only!
    fmt.Println("\n⚠️ Type Safety:")
    fmt.Println("   var x int = 5")
    fmt.Println("   var y int64 = 5")
    fmt.Println("   x == y  ❌ COMPILE ERROR! Different types")
    fmt.Println("   x == int(y) ✅ Must convert")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           COMPARISON OPERATORS                            ║
╚══════════════════════════════════════════════════════════╝

📊 Numeric Comparison (a=10, b=20):
   a == b : false (equal)
   a != b : true (not equal)
   a < b  : true (less than)
   a > b  : false (greater than)
   a <= b : true (less or equal)
   a >= b : false (greater or equal)

📊 String Comparison (lexicographic):
   "apple" == "banana" : false
   "apple" < "banana"  : true (a comes before b)
   "apple" > "banana"  : false

⚠️ Type Safety:
   var x int = 5
   var y int64 = 5
   x == y  ❌ COMPILE ERROR! Different types
   x == int(y) ✅ Must convert
```

---

## 🔀 Logical Operators

```go
// logical_operators.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           LOGICAL OPERATORS                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    a, b := true, false
    
    fmt.Printf("\n📊 Logical Operations (a=%t, b=%t):\n", a, b)
    fmt.Printf("   a && b : %t (AND - both must be true)\n", a && b)
    fmt.Printf("   a || b : %t (OR - at least one true)\n", a || b)
    fmt.Printf("   !a     : %t (NOT - inverts)\n", !a)
    fmt.Printf("   !b     : %t\n", !b)
    
    // Truth tables
    fmt.Println("\n📊 AND Truth Table (&&):")
    fmt.Println("   true  && true  = true")
    fmt.Println("   true  && false = false")
    fmt.Println("   false && true  = false")
    fmt.Println("   false && false = false")
    
    fmt.Println("\n📊 OR Truth Table (||):")
    fmt.Println("   true  || true  = true")
    fmt.Println("   true  || false = true")
    fmt.Println("   false || true  = true")
    fmt.Println("   false || false = false")
    
    // Short-circuit evaluation
    fmt.Println("\n📊 Short-Circuit Evaluation:")
    fmt.Println("   false && expensiveFunc() → expensiveFunc NOT called")
    fmt.Println("   true || expensiveFunc()  → expensiveFunc NOT called")
    
    // Practical example
    fmt.Println("\n💡 Practical Example:")
    age := 25
    hasLicense := true
    
    if age >= 18 && hasLicense {
        fmt.Println("   Can drive! ✅")
    }
    
    isWeekend := true
    isHoliday := false
    
    if isWeekend || isHoliday {
        fmt.Println("   Day off! 🎉")
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           LOGICAL OPERATORS                               ║
╚══════════════════════════════════════════════════════════╝

📊 Logical Operations (a=true, b=false):
   a && b : false (AND - both must be true)
   a || b : true (OR - at least one true)
   !a     : false (NOT - inverts)
   !b     : true

📊 AND Truth Table (&&):
   true  && true  = true
   true  && false = false
   false && true  = false
   false && false = false

📊 OR Truth Table (||):
   true  || true  = true
   true  || false = true
   false || true  = true
   false || false = false

📊 Short-Circuit Evaluation:
   false && expensiveFunc() → expensiveFunc NOT called
   true || expensiveFunc()  → expensiveFunc NOT called

💡 Practical Example:
   Can drive! ✅
   Day off! 🎉
```

---

## 🔢 Bitwise Operators

```go
// bitwise_operators.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           BITWISE OPERATORS                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    a, b := 12, 10  // 1100, 1010 in binary
    
    fmt.Printf("\n📊 Bitwise Operations (a=%d [%04b], b=%d [%04b]):\n", a, a, b, b)
    fmt.Printf("   a & b  = %2d [%04b] (AND)\n", a&b, a&b)
    fmt.Printf("   a | b  = %2d [%04b] (OR)\n", a|b, a|b)
    fmt.Printf("   a ^ b  = %2d [%04b] (XOR)\n", a^b, a^b)
    fmt.Printf("   a &^ b = %2d [%04b] (AND NOT / bit clear)\n", a&^b, a&^b)
    
    // Shift operators
    fmt.Println("\n📊 Shift Operators:")
    x := 4  // 100 in binary
    fmt.Printf("   x = %d [%04b]\n", x, x)
    fmt.Printf("   x << 1 = %2d [%04b] (left shift = multiply by 2)\n", x<<1, x<<1)
    fmt.Printf("   x << 2 = %2d [%04b] (left shift 2 = multiply by 4)\n", x<<2, x<<2)
    fmt.Printf("   x >> 1 = %2d [%04b] (right shift = divide by 2)\n", x>>1, x>>1)
    
    // Practical: Flags
    fmt.Println("\n💡 Practical: Permission Flags")
    const (
        Read    = 1 << iota  // 001
        Write                 // 010
        Execute               // 100
    )
    
    permission := Read | Write  // 011
    fmt.Printf("   Read|Write = %d [%03b]\n", permission, permission)
    
    hasRead := permission&Read != 0
    hasExecute := permission&Execute != 0
    fmt.Printf("   Has Read? %t\n", hasRead)
    fmt.Printf("   Has Execute? %t\n", hasExecute)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           BITWISE OPERATORS                               ║
╚══════════════════════════════════════════════════════════╝

📊 Bitwise Operations (a=12 [1100], b=10 [1010]):
   a & b  =  8 [1000] (AND)
   a | b  = 14 [1110] (OR)
   a ^ b  =  6 [0110] (XOR)
   a &^ b =  4 [0100] (AND NOT / bit clear)

📊 Shift Operators:
   x = 4 [0100]
   x << 1 =  8 [1000] (left shift = multiply by 2)
   x << 2 = 16 [10000] (left shift 2 = multiply by 4)
   x >> 1 =  2 [0010] (right shift = divide by 2)

💡 Practical: Permission Flags
   Read|Write = 3 [011]
   Has Read? true
   Has Execute? false
```

---

## 📝 Assignment Operators

```go
// assignment_operators.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           ASSIGNMENT OPERATORS                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 Basic Assignment:")
    x := 10  // Short declaration
    fmt.Printf("   x := 10 → x = %d\n", x)
    
    var y int = 20  // Explicit
    fmt.Printf("   var y int = 20 → y = %d\n", y)
    
    fmt.Println("\n📊 Compound Assignment:")
    n := 10
    fmt.Printf("   n = %d\n", n)
    
    n += 5
    fmt.Printf("   n += 5  → n = %d\n", n)
    
    n -= 3
    fmt.Printf("   n -= 3  → n = %d\n", n)
    
    n *= 2
    fmt.Printf("   n *= 2  → n = %d\n", n)
    
    n /= 4
    fmt.Printf("   n /= 4  → n = %d\n", n)
    
    n %= 3
    fmt.Printf("   n %%= 3  → n = %d\n", n)
    
    // Bitwise compound
    fmt.Println("\n📊 Bitwise Compound Assignment:")
    b := 12
    fmt.Printf("   b = %d [%04b]\n", b, b)
    
    b &= 10
    fmt.Printf("   b &= 10 → b = %d [%04b]\n", b, b)
    
    b |= 1
    fmt.Printf("   b |= 1  → b = %d [%04b]\n", b, b)
    
    b <<= 1
    fmt.Printf("   b <<= 1 → b = %d [%04b]\n", b, b)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           ASSIGNMENT OPERATORS                            ║
╚══════════════════════════════════════════════════════════╝

📊 Basic Assignment:
   x := 10 → x = 10
   var y int = 20 → y = 20

📊 Compound Assignment:
   n = 10
   n += 5  → n = 15
   n -= 3  → n = 12
   n *= 2  → n = 24
   n /= 4  → n = 6
   n %= 3  → n = 0

📊 Bitwise Compound Assignment:
   b = 12 [1100]
   b &= 10 → b = 8 [1000]
   b |= 1  → b = 9 [1001]
   b <<= 1 → b = 18 [10010]
```

---

## ⚠️ What Go DOESN'T Have

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  OPERATORS GO DOESN'T HAVE                                      │
│                                                                 │
│  1. TERNARY OPERATOR (?:)                                       │
│     ❌ result = x > 0 ? "positive" : "negative"                 │
│     ✅ Use if-else instead:                                     │
│        if x > 0 {                                               │
│            result = "positive"                                  │
│        } else {                                                 │
│            result = "negative"                                  │
│        }                                                        │
│                                                                 │
│  2. PREFIX INCREMENT/DECREMENT                                  │
│     ❌ ++i, --i                                                 │
│     ✅ i++ and i-- only (postfix)                               │
│                                                                 │
│  3. INCREMENT AS EXPRESSION                                     │
│     ❌ y = x++                                                  │
│     ✅ x++; y = x  (separate statements)                        │
│                                                                 │
│  4. COMMA OPERATOR                                              │
│     ❌ for (i=0, j=10; i<j; i++, j--)                           │
│     ✅ for i, j := 0, 10; i < j; i, j = i+1, j-1                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Operator Precedence

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  OPERATOR PRECEDENCE (highest to lowest)                        │
│                                                                 │
│  1.  *  /  %  <<  >>  &  &^                                     │
│  2.  +  -  |  ^                                                 │
│  3.  ==  !=  <  <=  >  >=                                       │
│  4.  &&                                                         │
│  5.  ||                                                         │
│                                                                 │
│  TIP: When in doubt, use parentheses!                           │
│       (a + b) * c  is clearer than  a + b * c                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Arithmetic**: `+ - * / %` work as expected
2. **Comparison**: Always returns `bool`
3. **Logical**: `&& || !` with short-circuit evaluation
4. **Bitwise**: `& | ^ << >> &^` for bit manipulation
5. **No ternary operator** - use if-else
6. **`++` and `--`** are statements only, not expressions
7. **Type safety**: Can't compare different types directly

---

## ➡️ Next Steps

**Next Topic:** [11 - If-Else Statements](./11-if-else.md)

