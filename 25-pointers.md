# 25 - Pointers

> Understanding memory addresses and references in Go.

---

## 📌 What You'll Learn

- What pointers are and why they matter
- Creating and using pointers
- When to use pointers vs values
- Pointers and structs
- Nil pointers and safety
- Sample programs for every concept

---

## 🤔 What is a Pointer?

### Definition

> A **pointer** is a variable that stores the memory address of another variable.

### Real-World Analogy

- **Value** = The actual person (lives at 123 Main St)
- **Pointer** = Address card ("123 Main St") — cheap to copy
- You can give someone the address card; they visit the person at that address (dereference)
- Multiple pointers can point to the same value

In Go:
- `alice := Person{Name: "Alice"}` — the actual value
- `ptr := &alice` — address (pointer)
- `(*ptr).Name = "Alice Smith"` or `ptr.Name = "Alice Smith"` — visit and modify

---

## 📊 Memory Layout

- `x := 42` — variable x holds 42 at some address (e.g., 0x1000)
- `ptr := &x` — ptr holds the address of x (0x1000)
- `&x` = "address of x"
- `*ptr` = "value at ptr" = 42

---

## 📝 Pointer Basics

### Creating a pointer

```go
x := 42
ptr := &x  // & = "address of"
fmt.Printf("x = %d, ptr = %p, *ptr = %d\n", x, ptr, *ptr)
// Output: x = 42, ptr = 0xc0000140a0, *ptr = 42
```

### Modifying through pointer

```go
*ptr = 100  // * = "value at"
fmt.Println(x) // Output: 100 (changed!)
```

### new() function

```go
p := new(int)  // Allocates, returns pointer to zero value
*p = 50
fmt.Println(*p) // Output: 50
```

### Operators summary

- `&` = Address-of operator (get pointer)
- `*` = Dereference operator (get value)
- `*T` = Pointer type (e.g., *int, *string)

### Complete Example

```go
package main

import "fmt"

func main() {
    x := 42
    ptr := &x
    fmt.Printf("x = %d, *ptr = %d\n", x, *ptr)
    *ptr = 100
    fmt.Printf("After *ptr = 100: x = %d\n", x)
}
```

**Output:**
```
x = 42, *ptr = 42
After *ptr = 100: x = 100
```

---

## 🔄 Pointers and Functions

### Pass by value (doesn't modify)

```go
func doubleValue(n int) {
    n = n * 2  // Modifies local copy
}
x := 10
doubleValue(x)
fmt.Println(x) // Output: 10 (unchanged!)
```

### Pass by pointer (modifies)

```go
func doublePointer(n *int) {
    *n = *n * 2
}
y := 10
doublePointer(&y)
fmt.Println(y) // Output: 20
```

### Swap example

```go
func swap(a, b *int) {
    *a, *b = *b, *a
}
a, b := 5, 10
swap(&a, &b)
fmt.Printf("a=%d, b=%d\n", a, b) // Output: a=10, b=5
```

### Returning pointer (safe in Go!)

```go
func createInt(n int) *int {
    result := n
    return &result  // Returning pointer to local is SAFE in Go!
}
ptr := createInt(42)
fmt.Println(*ptr) // Output: 42
```

### Complete Example

```go
package main

import "fmt"

func doublePointer(n *int) {
    *n = *n * 2
}

func swap(a, b *int) {
    *a, *b = *b, *a
}

func main() {
    y := 10
    doublePointer(&y)
    fmt.Println(y)
    a, b := 5, 10
    swap(&a, &b)
    fmt.Printf("a=%d, b=%d\n", a, b)
}
```

**Output:**
```
20
a=10, b=5
```

---

## 🏗️ Pointers and Structs

### Struct value vs pointer

```go
p1 := Person{Name: "Alice", Age: 25}  // Value
p2 := &Person{Name: "Bob", Age: 30}   // Pointer
```

### Accessing fields (auto-dereference)

```go
fmt.Println(p2.Name)      // Go auto-dereferences
fmt.Println((*p2).Name)   // Explicit dereference
```

### Pass struct by pointer to modify

```go
func updateAgePointer(p *Person) {
    p.Age = 99
}
person := Person{Name: "Charlie", Age: 28}
updateAgePointer(&person)
fmt.Println(person.Age) // Output: 99
```

### Constructor pattern

```go
func NewPerson(name string, age int) *Person {
    return &Person{Name: name, Age: age}
}
```

### Complete Example

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func updateAgePointer(p *Person) {
    p.Age = 99
}

func main() {
    person := Person{Name: "Charlie", Age: 28}
    updateAgePointer(&person)
    fmt.Printf("%+v\n", person)
}
```

**Output:**
```
{Name:Charlie Age:99}
```

---

## ⚠️ Nil Pointers

### Nil check

```go
var ptr *int
fmt.Println(ptr == nil) // Output: true
```

### Dereferencing nil = PANIC

```go
// *ptr  // This would panic!
```

### Safe nil check

```go
if ptr != nil {
    fmt.Println(*ptr)
} else {
    fmt.Println("Pointer is nil")
}
```

### Methods can handle nil receivers

```go
func (p *Person) SafeGreet() string {
    if p == nil {
        return "Hello, stranger"
    }
    return "Hello, " + p.Name
}
var p *Person
fmt.Println(p.SafeGreet()) // Output: Hello, stranger
```

### Complete Example

```go
package main

import "fmt"

type Person struct {
    Name string
}

func (p *Person) SafeGreet() string {
    if p == nil {
        return "Hello, stranger"
    }
    return "Hello, " + p.Name
}

func main() {
    var p *Person
    fmt.Println(p.SafeGreet())
}
```

**Output:**
```
Hello, stranger
```

---

## 🆚 Pointers: Go vs Java vs C

| Aspect | Java | Go | C |
|--------|------|-----|-----|
| Objects | Always references (pointers under hood) | Explicit: choose value or pointer | Explicit pointers |
| Primitives | Always values | All types can be value OR pointer | Values |
| Syntax | No explicit pointer syntax | `&` and `*` operators | `&` and `*` |
| Memory | Garbage collected | Garbage collected (safe) | Manual management |
| Pointer arithmetic | N/A | Not allowed | Allowed (unsafe) |

**Go's sweet spot:** Explicit like C (you choose), safe like Java (garbage collected, no arithmetic).

---

## 🎯 When to Use Pointers

| Use Pointers When | Use Values When |
|-------------------|-----------------|
| You need to modify the original | Data is small (int, bool, small structs) |
| Data is large (avoid copying) | You want immutability |
| Representing "optional" or "missing" (nil) | You want a copy |
| Sharing data across goroutines | Type is already a reference (slice, map, channel) |

---

## 🎯 Key Takeaways

1. **&** gets the address (creates pointer)
2. **\*** dereferences (gets value at address)
3. **Go is memory-safe** — no pointer arithmetic
4. **Nil pointers** — check before dereferencing!
5. **Struct pointers** auto-dereference with `.`
6. **Returning local pointers** is safe (Go manages memory)
7. **Use pointers** for large structs and when modification is needed
8. **Garbage collection** frees you from manual memory management

---

## ➡️ Next Steps

**Next Topic:** [26 - Interfaces](./26-interfaces.md)
