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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  POINTER = HOME ADDRESS                                         │
│                                                                 │
│  Imagine you have a friend Alice:                               │
│                                                                 │
│  ┌────────────────┐                                            │
│  │  Alice (Value) │  ← The actual person                       │
│  │  Lives at      │                                            │
│  │  123 Main St   │                                            │
│  └────────────────┘                                            │
│                                                                 │
│  ┌────────────────┐                                            │
│  │  Address Card  │  ← A pointer to Alice                      │
│  │  "123 Main St" │                                            │
│  └────────────────┘                                            │
│                                                                 │
│  You can:                                                       │
│  • Give someone the ADDRESS CARD (cheap to copy)                │
│  • They can VISIT Alice at that address (dereference)           │
│  • Multiple cards can point to same Alice                       │
│                                                                 │
│  In Go:                                                         │
│  alice := Person{Name: "Alice"}  // The actual person          │
│  ptr := &alice                    // Address card (pointer)     │
│  (*ptr).Name = "Alice Smith"      // Visit and modify          │
│  ptr.Name = "Alice Smith"         // Same (Go shorthand)       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Memory Visualization

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MEMORY LAYOUT                                                  │
│                                                                 │
│  x := 42                                                        │
│  ptr := &x                                                      │
│                                                                 │
│  MEMORY:                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Address 0x1000  │ Address 0x1008  │ ...                 │   │
│  │ (variable x)    │ (variable ptr)  │                     │   │
│  │                 │                 │                     │   │
│  │     42          │    0x1000       │                     │   │
│  │     ↑           │       │         │                     │   │
│  └─────────────────│───────│─────────────────────────────┘   │
│                    │       │                                    │
│                    └───────┘                                    │
│                    ptr points to x                              │
│                                                                 │
│  &x   = "address of x" = 0x1000                                │
│  *ptr = "value at ptr" = 42                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Pointer Basics

```go
// pointer_basics.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              POINTER BASICS                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Creating a pointer
    fmt.Println("\n📊 Creating Pointers:")
    x := 42
    ptr := &x  // & = "address of"
    
    fmt.Printf("   x = %d\n", x)
    fmt.Printf("   &x (address) = %p\n", &x)
    fmt.Printf("   ptr = %p\n", ptr)
    fmt.Printf("   *ptr (value at ptr) = %d\n", *ptr)
    
    // Modifying through pointer
    fmt.Println("\n📊 Modifying Through Pointer:")
    *ptr = 100  // * = "value at"
    fmt.Printf("   After *ptr = 100:\n")
    fmt.Printf("   x = %d (changed!)\n", x)
    fmt.Printf("   *ptr = %d\n", *ptr)
    
    // Pointer types
    fmt.Println("\n📊 Pointer Types:")
    var intPtr *int     // Pointer to int (nil)
    var strPtr *string  // Pointer to string (nil)
    
    fmt.Printf("   var intPtr *int = %v\n", intPtr)
    fmt.Printf("   var strPtr *string = %v\n", strPtr)
    
    // new() function
    fmt.Println("\n📊 new() Function:")
    p := new(int)  // Allocates, returns pointer to zero value
    fmt.Printf("   p := new(int)\n")
    fmt.Printf("   p = %p\n", p)
    fmt.Printf("   *p = %d (zero value)\n", *p)
    
    *p = 50
    fmt.Printf("   *p = %d (after assignment)\n", *p)
    
    // Operators summary
    fmt.Println("\n📊 Operators Summary:")
    fmt.Println("   & = Address-of operator (get pointer)")
    fmt.Println("   * = Dereference operator (get value)")
    fmt.Println("   *T = Pointer type (e.g., *int, *string)")
}
```

---

## 🔄 Pointers and Functions

```go
// pointer_functions.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           POINTERS AND FUNCTIONS                          ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pass by value (copy)
    fmt.Println("\n⚠️ Pass by Value (doesn't modify original):")
    x := 10
    fmt.Printf("   Before: x = %d\n", x)
    doubleValue(x)
    fmt.Printf("   After doubleValue(x): x = %d (unchanged!)\n", x)
    
    // Pass by pointer (reference)
    fmt.Println("\n✅ Pass by Pointer (modifies original):")
    y := 10
    fmt.Printf("   Before: y = %d\n", y)
    doublePointer(&y)
    fmt.Printf("   After doublePointer(&y): y = %d (doubled!)\n", y)
    
    // Multiple values
    fmt.Println("\n📊 Swap Example:")
    a, b := 5, 10
    fmt.Printf("   Before: a=%d, b=%d\n", a, b)
    swap(&a, &b)
    fmt.Printf("   After swap(&a, &b): a=%d, b=%d\n", a, b)
    
    // Returning pointer
    fmt.Println("\n📊 Returning Pointer:")
    ptr := createInt(42)
    fmt.Printf("   createInt(42) returned pointer to %d\n", *ptr)
}

func doubleValue(n int) {
    n = n * 2  // Modifies local copy
}

func doublePointer(n *int) {
    *n = *n * 2  // Modifies original through pointer
}

func swap(a, b *int) {
    *a, *b = *b, *a
}

func createInt(n int) *int {
    result := n  // Local variable
    return &result  // Returning pointer to local is SAFE in Go!
}
```

---

## 🏗️ Pointers and Structs

```go
// pointer_structs.go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           POINTERS AND STRUCTS                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Struct value vs pointer
    fmt.Println("\n📊 Struct Value vs Pointer:")
    
    p1 := Person{Name: "Alice", Age: 25}  // Value
    p2 := &Person{Name: "Bob", Age: 30}    // Pointer
    
    fmt.Printf("   p1 (value): %+v, type: %T\n", p1, p1)
    fmt.Printf("   p2 (pointer): %+v, type: %T\n", p2, p2)
    
    // Accessing fields
    fmt.Println("\n📊 Accessing Fields:")
    fmt.Printf("   p1.Name = %q\n", p1.Name)
    fmt.Printf("   p2.Name = %q (auto-dereference!)\n", p2.Name)
    fmt.Printf("   (*p2).Name = %q (explicit dereference)\n", (*p2).Name)
    
    // Modifying through pointer
    fmt.Println("\n📊 Modifying Through Pointer:")
    p2.Age = 31
    fmt.Printf("   p2.Age = 31 → %+v\n", p2)
    
    // Pass struct by value (copied!)
    fmt.Println("\n⚠️ Pass Struct by Value (copied!):")
    person := Person{Name: "Charlie", Age: 28}
    updateAgeValue(person)
    fmt.Printf("   After updateAgeValue: %+v (unchanged)\n", person)
    
    // Pass struct by pointer (modified!)
    fmt.Println("\n✅ Pass Struct by Pointer (modified!):")
    updateAgePointer(&person)
    fmt.Printf("   After updateAgePointer: %+v (changed)\n", person)
    
    // Constructor pattern
    fmt.Println("\n📊 Constructor Returns Pointer:")
    newPerson := NewPerson("Diana", 35)
    fmt.Printf("   NewPerson() = %+v\n", newPerson)
}

func updateAgeValue(p Person) {
    p.Age = 99  // Modifies copy
}

func updateAgePointer(p *Person) {
    p.Age = 99  // Modifies original
}

func NewPerson(name string, age int) *Person {
    return &Person{
        Name: name,
        Age:  age,
    }
}
```

---

## ⚠️ Nil Pointers

```go
// nil_pointers.go
package main

import "fmt"

type Person struct {
    Name string
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              NIL POINTERS                                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Nil pointer
    fmt.Println("\n📊 Nil Pointer:")
    var ptr *int
    fmt.Printf("   var ptr *int = %v\n", ptr)
    fmt.Printf("   ptr == nil: %t\n", ptr == nil)
    
    // Dereferencing nil = PANIC!
    fmt.Println("\n⚠️ Dereferencing Nil = PANIC!")
    fmt.Println("   *ptr  // This would panic!")
    
    // Safe nil check
    fmt.Println("\n✅ Safe Nil Check:")
    ptr = nil
    if ptr != nil {
        fmt.Printf("   Value: %d\n", *ptr)
    } else {
        fmt.Println("   Pointer is nil, skipping dereference")
    }
    
    // Methods can handle nil receivers!
    fmt.Println("\n💡 Methods Can Handle Nil Receivers:")
    var p *Person
    result := p.SafeGreet()
    fmt.Printf("   nil person.SafeGreet() = %q\n", result)
    
    // Common pattern: check nil in methods
    fmt.Println("\n📊 Common Pattern:")
    fmt.Println("   func (p *Person) Method() {")
    fmt.Println("       if p == nil {")
    fmt.Println("           return  // Handle nil gracefully")
    fmt.Println("       }")
    fmt.Println("       // Normal logic")
    fmt.Println("   }")
}

func (p *Person) SafeGreet() string {
    if p == nil {
        return "Hello, stranger"
    }
    return "Hello, " + p.Name
}
```

---

## 🆚 Pointers: Go vs Java vs C

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO vs JAVA vs C                                                │
│                                                                 │
│  JAVA:                                                          │
│  • Objects are ALWAYS references (pointers under the hood)      │
│  • Primitives are always values                                 │
│  • No explicit pointer syntax                                   │
│  • Garbage collected                                            │
│                                                                 │
│  GO:                                                            │
│  • EXPLICIT control: choose value or pointer                    │
│  • All types can be value OR pointer                            │
│  • & and * operators                                            │
│  • Garbage collected (safe!)                                    │
│  • No pointer arithmetic                                        │
│                                                                 │
│  C:                                                             │
│  • Explicit pointers like Go                                    │
│  • UNSAFE: pointer arithmetic allowed                           │
│  • Manual memory management                                     │
│  • Easy to corrupt memory                                       │
│                                                                 │
│  GO'S SWEET SPOT:                                               │
│  • Explicit like C (you choose)                                 │
│  • Safe like Java (garbage collected, no arithmetic)            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 When to Use Pointers

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  USE POINTERS WHEN:                                             │
│                                                                 │
│  1. You need to MODIFY the original value                       │
│     func update(p *Person) { p.Age++ }                          │
│                                                                 │
│  2. The data is LARGE (avoid copying)                           │
│     func process(data *LargeStruct) { ... }                     │
│                                                                 │
│  3. Representing "OPTIONAL" or "missing"                        │
│     var ptr *int = nil  // No value                             │
│                                                                 │
│  4. Sharing data across goroutines                              │
│     go processData(sharedPtr)                                   │
│                                                                 │
│  USE VALUES WHEN:                                               │
│                                                                 │
│  1. Data is SMALL (int, bool, small structs)                    │
│  2. You want IMMUTABILITY (function can't modify)               │
│  3. You want a COPY of the data                                 │
│  4. The type is already a reference (slice, map, channel)       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **&** gets the address (creates pointer)
2. **\*** dereferences (gets value at address)
3. **Go is memory-safe** - no pointer arithmetic
4. **Nil pointers** - check before dereferencing!
5. **Struct pointers** auto-dereference with `.`
6. **Returning local pointers** is safe (Go manages memory)
7. **Use pointers** for large structs and when modification is needed
8. **Garbage collection** frees you from manual memory management

---

## ➡️ Next Steps

**Next Topic:** [26 - Interfaces](./26-interfaces.md)

