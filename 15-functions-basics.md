# 15 - Functions - Basics

> Understanding functions in Go: declaration, parameters, and return values.

---

## 📌 What You'll Learn

- Function declaration syntax
- Parameters and arguments
- Return values (single and multiple)
- Named return values
- The blank identifier `_`
- Go's unique patterns compared to Java
- Sample programs for each concept

---

## 🤔 What is a Function?

A function takes inputs (parameters), does work, and returns outputs. Think of it like a vending machine: you put in money and selection (inputs), it processes, and gives you coffee and change (outputs).

---

## 📝 Function Declaration Syntax

```go
func functionName(parameter1 type1, parameter2 type2) returnType {
    return value
}
```

- `func` — keyword
- `functionName` — lowercase = unexported (package-private)
- Parameters: name followed by type
- Return type after the closing parenthesis

---

## Basic Function Examples

No parameters, no return:

```go
func sayHello() {
    fmt.Println("Hello!")
}
sayHello()  // Output: Hello!
```

With parameters:

```go
func greet(name string) {
    fmt.Printf("Hello, %s!\n", name)
}
greet("Alice")  // Output: Hello, Alice!
```

With return value:

```go
func add(a int, b int) int {
    return a + b
}
fmt.Println(add(5, 3))  // Output: 8
```

Same-type parameters can be shortened:

```go
func multiply(a, b int) int {
    return a * b
}
fmt.Println(multiply(4, 5))  // Output: 20
```

### Complete Example: Basic Functions

```go
package main

import "fmt"

func main() {
    sayHello()
    greet("Alice")
    fmt.Println(add(5, 3))
    quotient, remainder := divide(17, 5)
    fmt.Println(quotient, remainder)
}

func sayHello() {
    fmt.Println("Hello!")
}

func greet(name string) {
    fmt.Printf("Hello, %s!\n", name)
}

func add(a, b int) int {
    return a + b
}

func divide(dividend, divisor int) (int, int) {
    return dividend / divisor, dividend % divisor
}
```

**Output:**
```
Hello!
Hello, Alice!
8
3 2
```

---

## 🔀 Multiple Return Values

Go supports multiple return values natively—no wrapper types needed.

```go
func divide(a, b int) (int, int) {
    return a / b, a % b
}
q, r := divide(17, 5)
fmt.Println(q, r)  // Output: 3 2
```

**Error handling pattern:** Return `(result, error)`:

```go
func safeDivide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
result, err := safeDivide(10, 2)
if err != nil {
    fmt.Println(err)
} else {
    fmt.Println(result)  // Output: 5
}
```

### Complete Example: Multiple Returns

```go
package main

import (
    "errors"
    "fmt"
)

func main() {
    q, r := divide(17, 5)
    fmt.Println(q, r)
    result, err := safeDivide(10, 0)
    if err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(result)
    }
}

func divide(a, b int) (int, int) {
    return a / b, a % b
}

func safeDivide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

**Output:**
```
3 2
division by zero
```

---

## 🏷️ Named Return Values

Named returns let you assign to variables that are automatically returned.

```go
func rectangleInfo(width, height int) (area int, perimeter int) {
    area = width * height
    perimeter = 2 * (width + height)
    return  // "naked return" - returns area and perimeter
}
a, p := rectangleInfo(5, 3)
fmt.Println(a, p)  // Output: 15 16
```

**When to use:**
- Documentation: names clarify what's returned
- With `defer`: you can modify named returns in deferred functions

**When to avoid:** Short functions where they add no clarity, or when shadowing is a risk.

---

## ➖ The Blank Identifier `_`

Use `_` to ignore return values you don't need.

```go
quotient, _ := divide(17, 5)  // Ignore remainder
_, remainder := divide(17, 5)  // Ignore quotient
fmt.Println(quotient, remainder)  // Output: 3 2
```

In range loops:

```go
for _, value := range []string{"a", "b"} {
    fmt.Print(value)
}
// Output: ab

for i := range []string{"a", "b"} {
    fmt.Print(i)
}
// Output: 01
```

### Complete Example: Blank Identifier

```go
package main

import "fmt"

func main() {
    q, _ := divide(17, 5)
    fmt.Println("Quotient only:", q)
    for _, v := range []string{"apple", "banana"} {
        fmt.Print(v, " ")
    }
}

func divide(a, b int) (int, int) {
    return a / b, a % b
}
```

**Output:**
```
Quotient only: 3
apple banana
```

---

## 🏭 Production Patterns

**Constructor function:**

```go
func NewUser(id int, name string) *User {
    return &User{ID: id, Name: name, CreatedAt: time.Now()}
}
```

**Validation function:**

```go
func ValidateUser(u *User) error {
    if u.Name == "" {
        return errors.New("name is required")
    }
    return nil
}
```

**Options struct for many parameters:**

```go
type ServerConfig struct {
    Host string
    Port int
}
func StartServer(cfg ServerConfig) {
    fmt.Printf("Starting %s:%d\n", cfg.Host, cfg.Port)
}
```

### Complete Example: Production Pattern

```go
package main

import (
    "errors"
    "fmt"
)

type User struct {
    ID   int
    Name string
}

func main() {
    u := NewUser(1, "Alice")
    fmt.Println(u)
    if err := ValidateUser(u); err != nil {
        fmt.Println(err)
    } else {
        fmt.Println("Valid")
    }
}

func NewUser(id int, name string) *User {
    return &User{ID: id, Name: name}
}

func ValidateUser(u *User) error {
    if u.Name == "" {
        return errors.New("name is required")
    }
    return nil
}
```

**Output:**
```
&{1 Alice}
Valid
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| Method in class | Function (can be anywhere) |
| `public int add(int a, int b)` | `func add(a, b int) int` |
| `public` / `private` | Capital letter = exported, lowercase = unexported |
| Single return or wrapper class | Multiple returns: `(int, int)` |
| `throws Exception` | Return `(result, error)` |
| Method overloading | No overloading; use different names or variadic |
| Constructor | Factory function: `func NewUser(name string) *User` |

---

## 🎯 Key Takeaways

1. **`func`** keyword for all functions
2. **Type comes after** parameter name: `func add(a int)`
3. **Same type parameters** can be shortened: `func add(a, b int)`
4. **Multiple return values** are idiomatic: `(result, error)`
5. **Named returns** document what's returned
6. **Blank identifier `_`** ignores unwanted values
7. **No overloading** - use different names or variadic functions
8. **No exceptions** - return errors instead

---

## ➡️ Next Steps

You now understand basic functions. Let's explore multiple return values in depth.

**Next Topic:** [16 - Multiple Return Values](./16-multiple-returns.md)
