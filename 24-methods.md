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

### The Mental Model

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  FUNCTION vs METHOD                                             │
│                                                                 │
│  FUNCTION: Standalone behavior                                  │
│  ┌────────────────────────────────────────────────────────────┐│
│  │  func Greet(name string) string {                          ││
│  │      return "Hello, " + name                               ││
│  │  }                                                         ││
│  │                                                            ││
│  │  Greet("Alice")  // Call directly                          ││
│  └────────────────────────────────────────────────────────────┘│
│                                                                 │
│  METHOD: Behavior attached to a type                            │
│  ┌────────────────────────────────────────────────────────────┐│
│  │  func (p Person) Greet() string {                          ││
│  │        ↑                                                   ││
│  │     RECEIVER                                               ││
│  │       return "Hello, I'm " + p.Name                        ││
│  │  }                                                         ││
│  │                                                            ││
│  │  alice.Greet()  // Call on instance                        ││
│  └────────────────────────────────────────────────────────────┘│
│                                                                 │
│  The receiver (p Person) is like "this" or "self" in other     │
│  languages, but explicit and named.                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Basic Method Syntax

```go
// method_basics.go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

// Method with value receiver
func (p Person) Greet() string {
    return "Hello, I'm " + p.Name
}

// Method with parameters
func (p Person) GreetWith(greeting string) string {
    return greeting + ", I'm " + p.Name
}

// Method with multiple return values
func (p Person) Info() (string, int) {
    return p.Name, p.Age
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              BASIC METHODS                                ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    alice := Person{Name: "Alice", Age: 25}
    
    // Call methods
    fmt.Println("\n📊 Calling Methods:")
    fmt.Printf("   alice.Greet() = %q\n", alice.Greet())
    fmt.Printf("   alice.GreetWith(\"Hi\") = %q\n", alice.GreetWith("Hi"))
    
    name, age := alice.Info()
    fmt.Printf("   alice.Info() = %q, %d\n", name, age)
    
    // Method is just syntactic sugar!
    fmt.Println("\n📊 Method is Syntactic Sugar:")
    fmt.Printf("   alice.Greet() = %q\n", alice.Greet())
    fmt.Printf("   Person.Greet(alice) = %q (same thing!)\n", Person.Greet(alice))
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              BASIC METHODS                                ║
╚══════════════════════════════════════════════════════════╝

📊 Calling Methods:
   alice.Greet() = "Hello, I'm Alice"
   alice.GreetWith("Hi") = "Hi, I'm Alice"
   alice.Info() = "Alice", 25

📊 Method is Syntactic Sugar:
   alice.Greet() = "Hello, I'm Alice"
   Person.Greet(alice) = "Hello, I'm Alice" (same thing!)
```

---

## ⚡ Value Receiver vs Pointer Receiver

```go
// receivers.go
package main

import "fmt"

type Counter struct {
    Value int
}

// Value receiver - gets a COPY
func (c Counter) GetValue() int {
    return c.Value
}

// Value receiver - modifying the COPY (doesn't affect original!)
func (c Counter) IncrementBad() {
    c.Value++  // Modifies the copy!
    fmt.Printf("   Inside IncrementBad: %d\n", c.Value)
}

// Pointer receiver - gets the ACTUAL struct
func (c *Counter) Increment() {
    c.Value++  // Modifies the original!
}

// Pointer receiver - efficient for large structs
func (c *Counter) Reset() {
    c.Value = 0
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║        VALUE vs POINTER RECEIVERS                         ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    c := Counter{Value: 10}
    
    // Value receiver - reading
    fmt.Println("\n📊 Value Receiver (reading):")
    fmt.Printf("   c.GetValue() = %d\n", c.GetValue())
    
    // Value receiver - trying to modify (DOESN'T WORK!)
    fmt.Println("\n⚠️ Value Receiver (modifying - doesn't work!):")
    fmt.Printf("   Before: c.Value = %d\n", c.Value)
    c.IncrementBad()
    fmt.Printf("   After: c.Value = %d (unchanged!)\n", c.Value)
    
    // Pointer receiver - modifying (WORKS!)
    fmt.Println("\n✅ Pointer Receiver (modifying - works!):")
    fmt.Printf("   Before: c.Value = %d\n", c.Value)
    c.Increment()
    fmt.Printf("   After: c.Value = %d (incremented!)\n", c.Value)
    
    // Go is smart - automatic conversion
    fmt.Println("\n💡 Go Auto-Converts:")
    fmt.Println("   c.Increment() works even though c is not a pointer")
    fmt.Println("   Go automatically does (&c).Increment()")
    
    ptr := &Counter{Value: 5}
    fmt.Println("\n💡 And Vice Versa:")
    fmt.Printf("   ptr.GetValue() = %d\n", ptr.GetValue())
    fmt.Println("   ptr.GetValue() works even though GetValue has value receiver")
    fmt.Println("   Go automatically does (*ptr).GetValue()")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║        VALUE vs POINTER RECEIVERS                         ║
╚══════════════════════════════════════════════════════════╝

📊 Value Receiver (reading):
   c.GetValue() = 10

⚠️ Value Receiver (modifying - doesn't work!):
   Before: c.Value = 10
   Inside IncrementBad: 11
   After: c.Value = 10 (unchanged!)

✅ Pointer Receiver (modifying - works!):
   Before: c.Value = 10
   After: c.Value = 11 (incremented!)

💡 Go Auto-Converts:
   c.Increment() works even though c is not a pointer
   Go automatically does (&c).Increment()

💡 And Vice Versa:
   ptr.GetValue() = 5
   ptr.GetValue() works even though GetValue has value receiver
   Go automatically does (*ptr).GetValue()
```

---

## 🎯 When to Use Which Receiver

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  WHEN TO USE VALUE RECEIVER (T)                                 │
│                                                                 │
│  ✅ Method doesn't modify the receiver                          │
│  ✅ Receiver is small (int, bool, small struct)                 │
│  ✅ You want a copy (immutable semantics)                       │
│  ✅ Receiver is a map, func, or chan (already references)       │
│                                                                 │
│  WHEN TO USE POINTER RECEIVER (*T)                              │
│                                                                 │
│  ✅ Method modifies the receiver                                │
│  ✅ Receiver is large (avoid copying)                           │
│  ✅ Consistency (if one method uses *T, all should)             │
│  ✅ Receiver contains sync.Mutex or similar                     │
│                                                                 │
│  RULE OF THUMB:                                                 │
│  "When in doubt, use a pointer receiver"                        │
│                                                                 │
│  CONSISTENCY RULE:                                              │
│  "If one method has *T receiver, ALL methods should use *T"     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📦 Methods on Non-Struct Types

```go
// methods_on_types.go
package main

import (
    "fmt"
    "strings"
)

// Custom string type
type MyString string

func (s MyString) Upper() string {
    return strings.ToUpper(string(s))
}

func (s MyString) Len() int {
    return len(s)
}

// Custom int type
type Celsius float64
type Fahrenheit float64

func (c Celsius) ToFahrenheit() Fahrenheit {
    return Fahrenheit(c*9/5 + 32)
}

func (f Fahrenheit) ToCelsius() Celsius {
    return Celsius((f - 32) * 5 / 9)
}

// Custom slice type
type IntSlice []int

func (s IntSlice) Sum() int {
    total := 0
    for _, v := range s {
        total += v
    }
    return total
}

func (s IntSlice) Max() int {
    if len(s) == 0 {
        return 0
    }
    max := s[0]
    for _, v := range s {
        if v > max {
            max = v
        }
    }
    return max
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           METHODS ON NON-STRUCT TYPES                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Custom string
    fmt.Println("\n📊 Methods on Custom String Type:")
    s := MyString("hello world")
    fmt.Printf("   s = %q\n", s)
    fmt.Printf("   s.Upper() = %q\n", s.Upper())
    fmt.Printf("   s.Len() = %d\n", s.Len())
    
    // Temperature conversion
    fmt.Println("\n📊 Methods on Custom Float Type:")
    temp := Celsius(100)
    fmt.Printf("   %v°C = %v°F\n", temp, temp.ToFahrenheit())
    
    fTemp := Fahrenheit(212)
    fmt.Printf("   %v°F = %v°C\n", fTemp, fTemp.ToCelsius())
    
    // Custom slice
    fmt.Println("\n📊 Methods on Custom Slice Type:")
    nums := IntSlice{5, 2, 8, 1, 9}
    fmt.Printf("   nums = %v\n", nums)
    fmt.Printf("   nums.Sum() = %d\n", nums.Sum())
    fmt.Printf("   nums.Max() = %d\n", nums.Max())
    
    fmt.Println("\n⚠️ Limitation:")
    fmt.Println("   Cannot define methods on built-in types directly")
    fmt.Println("   Must create a new type based on the built-in type")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           METHODS ON NON-STRUCT TYPES                     ║
╚══════════════════════════════════════════════════════════╝

📊 Methods on Custom String Type:
   s = "hello world"
   s.Upper() = "HELLO WORLD"
   s.Len() = 11

📊 Methods on Custom Float Type:
   100°C = 212°F
   212°F = 100°C

📊 Methods on Custom Slice Type:
   nums = [5 2 8 1 9]
   nums.Sum() = 25
   nums.Max() = 9

⚠️ Limitation:
   Cannot define methods on built-in types directly
   Must create a new type based on the built-in type
```

---

## 🏭 Production Patterns

```go
// method_production.go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

// Domain model with methods
type Order struct {
    ID        string
    Items     []OrderItem
    Status    OrderStatus
    CreatedAt time.Time
}

type OrderItem struct {
    ProductID string
    Quantity  int
    Price     float64
}

type OrderStatus string

const (
    OrderPending   OrderStatus = "pending"
    OrderConfirmed OrderStatus = "confirmed"
    OrderShipped   OrderStatus = "shipped"
    OrderDelivered OrderStatus = "delivered"
)

// Business logic methods
func (o *Order) Total() float64 {
    total := 0.0
    for _, item := range o.Items {
        total += item.Price * float64(item.Quantity)
    }
    return total
}

func (o *Order) ItemCount() int {
    count := 0
    for _, item := range o.Items {
        count += item.Quantity
    }
    return count
}

func (o *Order) CanCancel() bool {
    return o.Status == OrderPending || o.Status == OrderConfirmed
}

func (o *Order) Ship() error {
    if o.Status != OrderConfirmed {
        return fmt.Errorf("cannot ship order in %s status", o.Status)
    }
    o.Status = OrderShipped
    return nil
}

// String representation
func (o Order) String() string {
    return fmt.Sprintf("Order{ID: %s, Items: %d, Total: $%.2f, Status: %s}",
        o.ID, o.ItemCount(), o.Total(), o.Status)
}

// JSON helper
func (o Order) ToJSON() ([]byte, error) {
    return json.MarshalIndent(o, "", "  ")
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION METHOD PATTERNS                      ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    order := &Order{
        ID:        "ORD-001",
        Status:    OrderConfirmed,
        CreatedAt: time.Now(),
        Items: []OrderItem{
            {ProductID: "PROD-1", Quantity: 2, Price: 29.99},
            {ProductID: "PROD-2", Quantity: 1, Price: 49.99},
        },
    }
    
    fmt.Println("\n📊 Business Logic Methods:")
    fmt.Printf("   %s\n", order)
    fmt.Printf("   Total: $%.2f\n", order.Total())
    fmt.Printf("   Can Cancel: %t\n", order.CanCancel())
    
    fmt.Println("\n📊 State Transition:")
    if err := order.Ship(); err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   Shipped! New status: %s\n", order.Status)
    }
    fmt.Printf("   Can Cancel now: %t\n", order.CanCancel())
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           PRODUCTION METHOD PATTERNS                      ║
╚══════════════════════════════════════════════════════════╝

📊 Business Logic Methods:
   Order{ID: ORD-001, Items: 3, Total: $109.97, Status: confirmed}
   Total: $109.97
   Can Cancel: true

📊 State Transition:
   Shipped! New status: shipped
   Can Cancel now: false
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  class Person {                    type Person struct {         │
│      String name;                      Name string              │
│                                    }                            │
│      // Method inside class                                     │
│      String greet() {              // Method outside struct     │
│          return "Hi, " + name;     func (p Person) Greet() {    │
│      }                                 return "Hi, " + p.Name   │
│  }                                 }                            │
│                                                                 │
│  // 'this' is implicit             // Receiver is EXPLICIT      │
│  return this.name;                 return p.Name                │
│                                                                 │
│  // Overloading                    // NO OVERLOADING            │
│  void process(int x)               func (t T) ProcessInt(x int) │
│  void process(String x)            func (t T) ProcessStr(x str) │
│                                                                 │
│  // Overriding (inheritance)       // NO INHERITANCE            │
│  @Override                         // Use embedding + interfaces│
│  void method() { ... }                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Methods** are functions with a receiver argument
2. **Value receiver** gets a copy, **pointer receiver** gets the original
3. **Use pointer receivers** when modifying or for large types
4. **Consistency** - if one method uses *T, all should
5. **Can add methods** to any type you define (not just structs)
6. **Go auto-converts** between value and pointer for method calls
7. **No method overloading** - use different names
8. **String() method** is used by fmt.Printf("%s", obj)

---

## ➡️ Next Steps

**Next Topic:** [25 - Pointers](./25-pointers.md)

