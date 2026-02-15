# 24 - Methods

> Attaching behavior to types - Go's way of object-oriented programming.

---

## 📌 What You'll Learn

- What methods are and how they differ from functions
- Value receivers vs pointer receivers
- When to use each receiver type
- Methods on any type (not just structs!)
- Method sets and interfaces
- Sample programs for every concept

---

## 🤔 What is a Method?

### Definition

> A **method** is a function with a special receiver argument that associates the function with a type.

### Function vs Method

- **Function:** Standalone behavior — call directly: `Greet("Alice")`
- **Method:** Behavior attached to a type — call on instance: `alice.Greet()`
- The receiver `(p Person)` is like "this" or "self" in other languages, but explicit and named

---

## 📝 Basic Method Syntax

### Value receiver

```go
type Person struct {
    Name string
    Age  int
}

func (p Person) Greet() string {
    return "Hello, I'm " + p.Name
}

alice := Person{Name: "Alice", Age: 25}
fmt.Println(alice.Greet()) // Output: Hello, I'm Alice
```

### Method with parameters

```go
func (p Person) GreetWith(greeting string) string {
    return greeting + ", I'm " + p.Name
}
fmt.Println(alice.GreetWith("Hi")) // Output: Hi, I'm Alice
```

### Method is syntactic sugar

```go
alice.Greet()           // Output: Hello, I'm Alice
Person.Greet(alice)     // Same thing! Output: Hello, I'm Alice
```

### Complete Example

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func (p Person) Greet() string {
    return "Hello, I'm " + p.Name
}

func (p Person) Info() (string, int) {
    return p.Name, p.Age
}

func main() {
    alice := Person{Name: "Alice", Age: 25}
    fmt.Println(alice.Greet())
    name, age := alice.Info()
    fmt.Printf("%s, %d\n", name, age)
}
```

**Output:**
```
Hello, I'm Alice
Alice, 25
```

---

## ⚡ Value Receiver vs Pointer Receiver

### Value receiver — gets a copy

```go
func (c Counter) GetValue() int {
    return c.Value
}
// Reading works fine
```

### Value receiver — modifying fails (modifies copy only)

```go
func (c Counter) IncrementBad() {
    c.Value++  // Modifies the copy, not the original!
}
c := Counter{Value: 10}
c.IncrementBad()
fmt.Println(c.Value) // Output: 10 (unchanged!)
```

### Pointer receiver — modifies original

```go
func (c *Counter) Increment() {
    c.Value++
}
c := Counter{Value: 10}
c.Increment()
fmt.Println(c.Value) // Output: 11
```

### Go auto-converts

- `c.Increment()` works even though `c` is not a pointer — Go does `(&c).Increment()`
- `ptr.GetValue()` works with value receiver — Go does `(*ptr).GetValue()`

### Complete Example

```go
package main

import "fmt"

type Counter struct {
    Value int
}

func (c Counter) GetValue() int {
    return c.Value
}

func (c *Counter) Increment() {
    c.Value++
}

func main() {
    c := Counter{Value: 10}
    fmt.Println(c.GetValue())
    c.Increment()
    fmt.Println(c.Value)
}
```

**Output:**
```
10
11
```

---

## 🎯 When to Use Which Receiver

| Value Receiver (T) | Pointer Receiver (*T) |
|-------------------|----------------------|
| Method doesn't modify the receiver | Method modifies the receiver |
| Receiver is small (int, bool, small struct) | Receiver is large (avoid copying) |
| You want immutable semantics | Consistency (if one uses *T, all should) |
| Receiver is map, func, or chan (already references) | Receiver contains sync.Mutex |

**Rule of thumb:** When in doubt, use a pointer receiver.

**Consistency rule:** If one method has *T receiver, ALL methods should use *T.

---

## 📦 Methods on Non-Struct Types

### Custom string type

```go
type MyString string

func (s MyString) Upper() string {
    return strings.ToUpper(string(s))
}
s := MyString("hello")
fmt.Println(s.Upper()) // Output: HELLO
```

### Custom numeric type

```go
type Celsius float64

func (c Celsius) ToFahrenheit() Fahrenheit {
    return Fahrenheit(c*9/5 + 32)
}
temp := Celsius(100)
fmt.Println(temp.ToFahrenheit()) // Output: 212
```

### Custom slice type

```go
type IntSlice []int

func (s IntSlice) Sum() int {
    total := 0
    for _, v := range s {
        total += v
    }
    return total
}
nums := IntSlice{5, 2, 8, 1, 9}
fmt.Println(nums.Sum()) // Output: 25
```

**Limitation:** Cannot define methods on built-in types directly — must create a new type based on the built-in type.

### Complete Example

```go
package main

import (
    "fmt"
    "strings"
)

type MyString string

func (s MyString) Upper() string {
    return strings.ToUpper(string(s))
}

type IntSlice []int

func (s IntSlice) Sum() int {
    total := 0
    for _, v := range s {
        total += v
    }
    return total
}

func main() {
    s := MyString("hello world")
    fmt.Println(s.Upper())
    nums := IntSlice{5, 2, 8, 1, 9}
    fmt.Println(nums.Sum())
}
```

**Output:**
```
HELLO WORLD
25
```

---

## 🏭 Production Patterns

### Domain model with methods

```go
type Order struct {
    ID     string
    Items  []OrderItem
    Status OrderStatus
}

func (o *Order) Total() float64 {
    total := 0.0
    for _, item := range o.Items {
        total += item.Price * float64(item.Quantity)
    }
    return total
}

func (o *Order) CanCancel() bool {
    return o.Status == OrderPending || o.Status == OrderConfirmed
}
```

### Complete Example

```go
package main

import "fmt"

type Order struct {
    ID     string
    Items  []OrderItem
    Status string
}

type OrderItem struct {
    ProductID string
    Quantity  int
    Price     float64
}

func (o *Order) Total() float64 {
    total := 0.0
    for _, item := range o.Items {
        total += item.Price * float64(item.Quantity)
    }
    return total
}

func main() {
    order := &Order{
        ID:     "ORD-001",
        Status: "confirmed",
        Items: []OrderItem{
            {ProductID: "PROD-1", Quantity: 2, Price: 29.99},
            {ProductID: "PROD-2", Quantity: 1, Price: 49.99},
        },
    }
    fmt.Printf("Total: $%.2f\n", order.Total())
}
```

**Output:**
```
Total: $109.97
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `class Person { String name; }` | `type Person struct { Name string }` |
| Method inside class | Method outside struct |
| `return this.name` (implicit) | `return p.Name` (explicit receiver) |
| Method overloading supported | No overloading — use different names |
| Overriding via inheritance | Use embedding + interfaces |
| `@Override void method()` | No inheritance |

---

## 🎯 Key Takeaways

1. **Methods** are functions with a receiver argument
2. **Value receiver** gets a copy, **pointer receiver** gets the original
3. **Use pointer receivers** when modifying or for large types
4. **Consistency** — if one method uses *T, all should
5. **Can add methods** to any type you define (not just structs)
6. **Go auto-converts** between value and pointer for method calls
7. **No method overloading** — use different names
8. **String() method** is used by `fmt.Printf("%s", obj)`

---

## ➡️ Next Steps

**Next Topic:** [25 - Pointers](./25-pointers.md)
