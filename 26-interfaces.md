# 26 - Interfaces

> Go's most powerful feature for abstraction and polymorphism.

---

## 📌 What You'll Learn

- What interfaces are and why they're special in Go
- Implicit interface implementation (no "implements" keyword!)
- Empty interface (any type)
- Type assertions and type switches
- Interface composition
- Best practices and production patterns

---

## 🤔 What is an Interface?

### Definition

> An **interface** is a set of method signatures. Any type that implements ALL those methods automatically satisfies the interface.

### Java vs Go

- **Java:** Must declare `implements Speaker` — explicit, compile-time coupling
- **Go:** Just implement the methods — no "implements" needed! Types satisfy interfaces implicitly

**Why this is powerful:**
- Can create interfaces for types you don't control
- Decouples definition from implementation
- Enables "duck typing" with type safety

---

## 📝 Interface Basics

### Interface definition

```go
type Shape interface {
    Area() float64
    Perimeter() float64
}
```

### Types satisfy implicitly (no "implements")

```go
type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}
// Rectangle automatically satisfies Shape!
```

### Function accepting interface

```go
func PrintShape(s Shape) {
    fmt.Printf("Area: %.2f, Perimeter: %.2f\n", s.Area(), s.Perimeter())
}
rect := Rectangle{Width: 10, Height: 5}
PrintShape(rect) // Output: Area: 50.00, Perimeter: 30.00
```

### Slice of interfaces (polymorphism)

```go
shapes := []Shape{rect, circle}
totalArea := 0.0
for _, s := range shapes {
    totalArea += s.Area()
}
```

### Complete Example

```go
package main

import (
    "fmt"
    "math"
)

type Shape interface {
    Area() float64
    Perimeter() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}
    circle := Circle{Radius: 7}
    shapes := []Shape{rect, circle}
    totalArea := 0.0
    for _, s := range shapes {
        totalArea += s.Area()
    }
    fmt.Printf("Total area: %.2f\n", totalArea)
}
```

**Output:**
```
Total area: 203.94
```

---

## 🦆 Duck Typing

"If it walks like a duck and quacks like a duck, then it's a duck."

```go
type Quacker interface {
    Quack() string
}

type Duck struct{}
func (d Duck) Quack() string { return "Quack!" }

type Person struct{}
func (p Person) Quack() string { return "I'm quacking!" }
// Both satisfy Quacker — they "quack like a duck"
```

---

## 🔲 Empty Interface

### interface{} or any — accepts any type

```go
var anything interface{}
anything = 42
anything = "hello"
anything = []int{1, 2, 3}
// All valid!
```

### any (Go 1.18+)

```go
var x any = "works the same!"
// any is alias for interface{}
```

### Slice of any

```go
mixed := []any{1, "two", 3.0, true}
for i, v := range mixed {
    fmt.Printf("[%d] %T: %v\n", i, v, v)
}
```

**Note:** `fmt.Println` uses `func Println(a ...interface{})` — that's why it accepts any type!

### Complete Example

```go
package main

import "fmt"

func main() {
    var anything interface{}
    anything = 42
    fmt.Printf("Type: %T, Value: %v\n", anything, anything)
    anything = "hello"
    fmt.Printf("Type: %T, Value: %v\n", anything, anything)
}
```

**Output:**
```
Type: int, Value: 42
Type: string, Value: hello
```

---

## 🔍 Type Assertions

### Basic type assertion (can panic)

```go
var i interface{} = "hello"
s := i.(string)
fmt.Println(s) // Output: hello
// n := i.(int)  // PANIC!
```

### Safe type assertion (comma-ok)

```go
if s, ok := i.(string); ok {
    fmt.Printf("i is a string: %q\n", s)
} else {
    fmt.Println("i is not a string")
}
```

### Type switch

```go
func describeType(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Integer: %d\n", v)
    case string:
        fmt.Printf("String: %q\n", v)
    default:
        fmt.Printf("Unknown: %T\n", v)
    }
}
describeType(42)   // Integer: 42
describeType("hi") // String: "hi"
```

### Complete Example

```go
package main

import "fmt"

func main() {
    var i interface{} = "hello"
    if s, ok := i.(string); ok {
        fmt.Printf("i is string: %q\n", s)
    }
    if _, ok := i.(int); !ok {
        fmt.Println("i is not an int")
    }
}
```

**Output:**
```
i is string: "hello"
i is not an int
```

---

## 🧩 Interface Composition

### Small, focused interfaces

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}
```

### Composed interfaces

```go
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

**Go's io package uses this:** `io.Reader`, `io.Writer`, `io.ReadWriter`, `io.ReadWriteCloser`

### Complete Example

```go
package main

import "fmt"

type Reader interface {
    Read() string
}

type Writer interface {
    Write(s string)
}

type ReadWriter interface {
    Reader
    Writer
}

type File struct{ name string }

func (f *File) Read() string  { return "data from " + f.name }
func (f *File) Write(s string) { fmt.Println("Writing:", s) }

func main() {
    f := &File{name: "data.txt"}
    var rw ReadWriter = f
    fmt.Println(rw.Read())
    rw.Write("hello")
}
```

**Output:**
```
data from data.txt
Writing: hello
```

---

## 🏭 Production Patterns

### Dependency injection

```go
type UserRepository interface {
    GetByID(id int) (*User, error)
    Save(user *User) error
}

type UserService struct {
    repo UserRepository  // Interface, not concrete type!
}

// Production: &UserService{repo: &PostgresUserRepo{}}
// Testing:   &UserService{repo: &MockUserRepo{}}
```

### Accept interfaces, return concrete types

```go
func ProcessData(data string, logger Logger) string {
    logger.Log("Processing: " + data)
    return strings.ToUpper(data)
}
```

**Key principles:**
- Accept interfaces, return concrete types
- Keep interfaces small (1-3 methods)
- Define interfaces where they're used, not where implemented

### Complete Example

```go
package main

import (
    "fmt"
    "strings"
)

type Logger interface {
    Log(message string)
}

type ConsoleLogger struct{}

func (l ConsoleLogger) Log(message string) {
    fmt.Println("[LOG]", message)
}

func ProcessData(data string, logger Logger) string {
    logger.Log("Processing: " + data)
    return strings.ToUpper(data)
}

func main() {
    result := ProcessData("hello", ConsoleLogger{})
    fmt.Println(result)
}
```

**Output:**
```
[LOG] Processing: hello
HELLO
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `interface Shape { double area(); }` | `type Shape interface { Area() float64 }` |
| `class Circle implements Shape` | `func (c Circle) Area() float64 {...}` — no "implements" |
| Must import/know interface at implementation time | Can satisfy after-the-fact without touching original |
| `Object` (top type) | `interface{}` or `any` |
| `instanceof` | Type assertion `x.(Type)` or type switch |

---

## 🎯 Key Takeaways

1. **Implicit satisfaction** — no "implements" keyword
2. **Duck typing** — if it has the methods, it satisfies
3. **Empty interface** (`interface{}` or `any`) accepts any type
4. **Type assertions** — extract concrete type from interface
5. **Small interfaces** are better (Reader, Writer, Stringer)
6. **Composition** — build large interfaces from small ones
7. **Accept interfaces, return concrete types**
8. **Define interfaces at the consumer**, not provider

---

## ➡️ Next Steps

**Next Topic:** [27 - Composition](./27-composition.md)
