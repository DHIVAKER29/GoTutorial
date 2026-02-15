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

Go operators fall into these categories:

- **Arithmetic**: `+` `-` `*` `/` `%`
- **Comparison**: `==` `!=` `<` `>` `<=` `>=`
- **Logical**: `&&` `||` `!`
- **Bitwise**: `&` `|` `^` `<<` `>>` `&^`
- **Assignment**: `=` `+=` `-=` `*=` `/=` `%=` etc.
- **Address**: `&` `*`
- **Receive**: `<-`

---

## ➕ Arithmetic Operators

Basic arithmetic works as expected. Integer division truncates toward zero.

```go
a, b := 17, 5
fmt.Println(a + b)  // Output: 22
fmt.Println(a - b)  // Output: 12
fmt.Println(a * b)  // Output: 85
fmt.Println(a / b)  // Output: 3 (integer division)
fmt.Println(a % b)  // Output: 2 (modulo)
```

Float division produces a float:

```go
x, y := 17.0, 5.0
fmt.Printf("%.2f\n", x/y)  // Output: 3.40
```

Increment and decrement are **statements only** (not expressions):

```go
count := 10
count++
fmt.Println(count)  // Output: 11
count--
fmt.Println(count)  // Output: 10
```

**Go difference:** `++` and `--` are postfix only. No `++i`, and you cannot use them in expressions like `x = count++`.

Unary operators:

```go
num := 42
fmt.Println(+num)  // Output: 42
fmt.Println(-num)  // Output: -42
```

### Complete Example: Arithmetic

```go
package main

import "fmt"

func main() {
    a, b := 17, 5
    fmt.Printf("a+b=%d a-b=%d a*b=%d a/b=%d a%%b=%d\n", a+b, a-b, a*b, a/b, a%b)
    count := 10
    count++
    fmt.Println(count)
}
```

**Output:**
```
a+b=22 a-b=12 a*b=85 a/b=3 a%b=2
11
```

---

## ⚖️ Comparison Operators

Comparison operators always return `bool`.

```go
a, b := 10, 20
fmt.Println(a == b)  // Output: false
fmt.Println(a != b)  // Output: true
fmt.Println(a < b)   // Output: true
fmt.Println(a > b)   // Output: false
fmt.Println(a <= b)  // Output: true
fmt.Println(a >= b)  // Output: false
```

Strings compare lexicographically:

```go
s1, s2 := "apple", "banana"
fmt.Println(s1 == s2)  // Output: false
fmt.Println(s1 < s2)   // Output: true
```

**Type safety:** You cannot compare different types directly. `var x int = 5` and `var y int64 = 5` cannot be compared with `x == y`; use `x == int(y)`.

### Complete Example: Comparison

```go
package main

import "fmt"

func main() {
    a, b := 10, 20
    fmt.Println(a < b, a == b)
    s1, s2 := "apple", "banana"
    fmt.Println(s1 < s2)
}
```

**Output:**
```
true false
true
```

---

## 🔀 Logical Operators

Logical operators work on boolean values with short-circuit evaluation.

```go
a, b := true, false
fmt.Println(a && b)  // Output: false (AND)
fmt.Println(a || b)  // Output: true (OR)
fmt.Println(!a)      // Output: false (NOT)
```

Short-circuit: `false && expensiveFunc()` never calls `expensiveFunc()`; `true || expensiveFunc()` never calls it either.

### Complete Example: Logical

```go
package main

import "fmt"

func main() {
    age := 25
    hasLicense := true
    if age >= 18 && hasLicense {
        fmt.Println("Can drive!")
    }
}
```

**Output:**
```
Can drive!
```

---

## 🔢 Bitwise Operators

Bitwise operators work on integer types.

```go
a, b := 12, 10  // 1100, 1010 in binary
fmt.Println(a & b)   // Output: 8  (AND)
fmt.Println(a | b)   // Output: 14 (OR)
fmt.Println(a ^ b)   // Output: 6  (XOR)
fmt.Println(a &^ b)  // Output: 4  (AND NOT / bit clear)
```

Shift operators:

```go
x := 4
fmt.Println(x << 1)  // Output: 8  (multiply by 2)
fmt.Println(x << 2)  // Output: 16 (multiply by 4)
fmt.Println(x >> 1)  // Output: 2  (divide by 2)
```

### Complete Example: Bitwise

```go
package main

import "fmt"

func main() {
    const Read, Write, Execute = 1, 2, 4
    permission := Read | Write
    fmt.Println(permission&Read != 0, permission&Execute != 0)
}
```

**Output:**
```
true false
```

---

## 📝 Assignment Operators

Short declaration and compound assignment:

```go
x := 10
fmt.Println(x)  // Output: 10

n := 10
n += 5
fmt.Println(n)  // Output: 15
n -= 3
fmt.Println(n)  // Output: 12
```

Compound operators: `+=` `-=` `*=` `/=` `%=` and bitwise variants like `&=` `|=` `<<=`.

### Complete Example: Assignment

```go
package main

import "fmt"

func main() {
    n := 10
    n += 5
    n -= 3
    n *= 2
    fmt.Println(n)
}
```

**Output:**
```
24
```

---

## ⚠️ What Go DOESN'T Have

| Feature | Go | Alternative |
|---------|-----|-------------|
| Ternary operator `?:` | ❌ | Use `if`-`else` |
| Prefix increment `++i` | ❌ | Use postfix `i++` only |
| Increment as expression `y = x++` | ❌ | Use separate statements: `x++; y = x` |
| Comma operator in for | ❌ | Use `for i, j := 0, 10; i < j; i, j = i+1, j-1` |

---

## 📊 Operator Precedence

| Precedence (high to low) | Operators |
|--------------------------|-----------|
| 1 | `*` `/` `%` `<<` `>>` `&` `&^` |
| 2 | `+` `-` `\|` `^` |
| 3 | `==` `!=` `<` `<=` `>` `>=` |
| 4 | `&&` |
| 5 | `\|\|` |

**Tip:** When in doubt, use parentheses: `(a + b) * c` is clearer than `a + b * c`.

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
