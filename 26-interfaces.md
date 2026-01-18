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

### The Revolutionary Difference

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA: EXPLICIT Interface Implementation                        │
│  ──────────────────────────────────────────                     │
│  interface Speaker {                                            │
│      void speak();                                              │
│  }                                                              │
│                                                                 │
│  class Dog implements Speaker {  // Must say "implements"       │
│      public void speak() {                                      │
│          System.out.println("Woof!");                          │
│      }                                                          │
│  }                                                              │
│                                                                 │
│  GO: IMPLICIT Interface Satisfaction                            │
│  ───────────────────────────────────                            │
│  type Speaker interface {                                       │
│      Speak() string                                             │
│  }                                                              │
│                                                                 │
│  type Dog struct{}                                              │
│                                                                 │
│  func (d Dog) Speak() string {  // Just implement the method   │
│      return "Woof!"             // No "implements" needed!      │
│  }                                                              │
│                                                                 │
│  // Dog automatically satisfies Speaker!                        │
│                                                                 │
│  WHY THIS IS POWERFUL:                                          │
│  • Can create interfaces for types you don't control            │
│  • Decouples definition from implementation                     │
│  • Enables "duck typing" with type safety                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Interface Basics

```go
// interface_basics.go
package main

import (
    "fmt"
    "math"
)

// Interface definition
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Types that satisfy Shape (no "implements" keyword!)
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

// Function accepting interface
func PrintShape(s Shape) {
    fmt.Printf("   Area: %.2f, Perimeter: %.2f\n", s.Area(), s.Perimeter())
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              INTERFACE BASICS                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    rect := Rectangle{Width: 10, Height: 5}
    circle := Circle{Radius: 7}
    
    fmt.Println("\n📊 Same Interface, Different Types:")
    fmt.Println("   Rectangle:")
    PrintShape(rect)
    fmt.Println("   Circle:")
    PrintShape(circle)
    
    // Slice of interfaces
    fmt.Println("\n📊 Slice of Interface (polymorphism):")
    shapes := []Shape{rect, circle}
    
    totalArea := 0.0
    for _, s := range shapes {
        totalArea += s.Area()
    }
    fmt.Printf("   Total area of all shapes: %.2f\n", totalArea)
    
    // Interface variable
    fmt.Println("\n📊 Interface Variable:")
    var s Shape
    s = rect
    fmt.Printf("   s = rect → %T, Area: %.2f\n", s, s.Area())
    s = circle
    fmt.Printf("   s = circle → %T, Area: %.2f\n", s, s.Area())
}
```

---

## 🦆 Duck Typing

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  "IF IT WALKS LIKE A DUCK AND QUACKS LIKE A DUCK,              │
│   THEN IT'S A DUCK"                                             │
│                                                                 │
│  type Quacker interface {                                       │
│      Quack() string                                             │
│  }                                                              │
│                                                                 │
│  type Duck struct{}                                             │
│  func (d Duck) Quack() string { return "Quack!" }               │
│                                                                 │
│  type Person struct{}                                           │
│  func (p Person) Quack() string { return "I'm quacking!" }      │
│                                                                 │
│  type Robot struct{}                                            │
│  func (r Robot) Quack() string { return "QUACK.WAV" }           │
│                                                                 │
│  // All three satisfy Quacker!                                  │
│  var q Quacker                                                  │
│  q = Duck{}   // ✅                                             │
│  q = Person{} // ✅                                             │
│  q = Robot{}  // ✅                                             │
│                                                                 │
│  They all "quack like a duck" so they ARE ducks (Quackers)!    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔲 Empty Interface

```go
// empty_interface.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              EMPTY INTERFACE (any)                        ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // interface{} = no methods = ANY type satisfies it
    fmt.Println("\n📊 Empty Interface (interface{} or 'any'):")
    
    var anything interface{}
    
    anything = 42
    fmt.Printf("   anything = 42 → Type: %T, Value: %v\n", anything, anything)
    
    anything = "hello"
    fmt.Printf("   anything = \"hello\" → Type: %T, Value: %v\n", anything, anything)
    
    anything = []int{1, 2, 3}
    fmt.Printf("   anything = []int{1,2,3} → Type: %T, Value: %v\n", anything, anything)
    
    // 'any' is alias for interface{} (Go 1.18+)
    fmt.Println("\n📊 'any' Keyword (Go 1.18+):")
    var x any = "works the same!"
    fmt.Printf("   var x any = ... → Type: %T\n", x)
    
    // Slice of any type
    fmt.Println("\n📊 Slice of any:")
    mixed := []any{1, "two", 3.0, true}
    for i, v := range mixed {
        fmt.Printf("   [%d] Type: %T, Value: %v\n", i, v, v)
    }
    
    // fmt.Println uses empty interface!
    fmt.Println("\n💡 fmt.Println signature:")
    fmt.Println("   func Println(a ...interface{}) (n int, err error)")
    fmt.Println("   That's why it accepts any type!")
}
```

---

## 🔍 Type Assertions

```go
// type_assertions.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              TYPE ASSERTIONS                              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    var i interface{} = "hello"
    
    // Basic type assertion (dangerous!)
    fmt.Println("\n📊 Basic Type Assertion:")
    s := i.(string)
    fmt.Printf("   i.(string) = %q\n", s)
    
    // Wrong type = PANIC!
    fmt.Println("\n⚠️ Wrong Type = PANIC:")
    fmt.Println("   n := i.(int)  // Would panic!")
    
    // Safe type assertion (with ok)
    fmt.Println("\n✅ Safe Type Assertion (comma-ok):")
    
    if s, ok := i.(string); ok {
        fmt.Printf("   i is a string: %q\n", s)
    } else {
        fmt.Println("   i is not a string")
    }
    
    if n, ok := i.(int); ok {
        fmt.Printf("   i is an int: %d\n", n)
    } else {
        fmt.Println("   i is not an int")
    }
    
    // Type switch
    fmt.Println("\n📊 Type Switch:")
    describeType(42)
    describeType("hello")
    describeType(3.14)
    describeType([]int{1, 2, 3})
    describeType(true)
}

func describeType(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("   Integer: %d (doubled: %d)\n", v, v*2)
    case string:
        fmt.Printf("   String: %q (length: %d)\n", v, len(v))
    case float64:
        fmt.Printf("   Float: %.2f\n", v)
    case bool:
        fmt.Printf("   Boolean: %t\n", v)
    default:
        fmt.Printf("   Unknown type: %T\n", v)
    }
}
```

---

## 🧩 Interface Composition

```go
// interface_composition.go
package main

import "fmt"

// Small, focused interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

// Composed interfaces
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

// Implementation
type File struct {
    name string
}

func (f *File) Read(p []byte) (n int, err error) {
    fmt.Printf("   Reading from %s\n", f.name)
    return len(p), nil
}

func (f *File) Write(p []byte) (n int, err error) {
    fmt.Printf("   Writing to %s\n", f.name)
    return len(p), nil
}

func (f *File) Close() error {
    fmt.Printf("   Closing %s\n", f.name)
    return nil
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           INTERFACE COMPOSITION                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    file := &File{name: "data.txt"}
    
    fmt.Println("\n📊 File satisfies multiple interfaces:")
    
    // File as Reader
    var r Reader = file
    r.Read(nil)
    
    // File as Writer
    var w Writer = file
    w.Write(nil)
    
    // File as ReadWriteCloser
    var rwc ReadWriteCloser = file
    rwc.Read(nil)
    rwc.Write(nil)
    rwc.Close()
    
    fmt.Println("\n💡 Go's io Package Uses This Pattern:")
    fmt.Println("   io.Reader, io.Writer, io.Closer")
    fmt.Println("   io.ReadWriter = Reader + Writer")
    fmt.Println("   io.ReadWriteCloser = Reader + Writer + Closer")
}
```

---

## 🏭 Production Patterns

```go
// interface_production.go
package main

import (
    "fmt"
    "strings"
)

// Pattern 1: Dependency Injection
type UserRepository interface {
    GetByID(id int) (*User, error)
    Save(user *User) error
}

type User struct {
    ID   int
    Name string
}

// Production implementation
type PostgresUserRepo struct{}

func (r *PostgresUserRepo) GetByID(id int) (*User, error) {
    return &User{ID: id, Name: "From Postgres"}, nil
}

func (r *PostgresUserRepo) Save(user *User) error {
    return nil
}

// Test implementation
type MockUserRepo struct {
    users map[int]*User
}

func (r *MockUserRepo) GetByID(id int) (*User, error) {
    return r.users[id], nil
}

func (r *MockUserRepo) Save(user *User) error {
    r.users[user.ID] = user
    return nil
}

// Service depends on interface, not implementation
type UserService struct {
    repo UserRepository  // Interface!
}

func (s *UserService) GetUser(id int) (*User, error) {
    return s.repo.GetByID(id)
}

// Pattern 2: Accept interfaces, return concrete types
type Logger interface {
    Log(message string)
}

type ConsoleLogger struct{}

func (l ConsoleLogger) Log(message string) {
    fmt.Println("[LOG]", message)
}

// Accepts interface
func ProcessData(data string, logger Logger) string {
    logger.Log("Processing: " + data)
    return strings.ToUpper(data)
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION PATTERNS                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: Swap implementations
    fmt.Println("\n📊 Pattern 1: Dependency Injection")
    
    // Production
    prodService := &UserService{repo: &PostgresUserRepo{}}
    user, _ := prodService.GetUser(1)
    fmt.Printf("   Production: %+v\n", user)
    
    // Testing
    mockRepo := &MockUserRepo{users: map[int]*User{1: {ID: 1, Name: "Test User"}}}
    testService := &UserService{repo: mockRepo}
    user, _ = testService.GetUser(1)
    fmt.Printf("   Testing: %+v\n", user)
    
    // Pattern 2: Accept interface
    fmt.Println("\n📊 Pattern 2: Accept Interface, Return Concrete")
    result := ProcessData("hello", ConsoleLogger{})
    fmt.Printf("   Result: %s\n", result)
    
    fmt.Println("\n💡 Key Principles:")
    fmt.Println("   • Accept interfaces, return concrete types")
    fmt.Println("   • Keep interfaces small (1-3 methods)")
    fmt.Println("   • Define interfaces where they're used, not implemented")
}
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  interface Shape {                 type Shape interface {       │
│      double area();                    Area() float64           │
│  }                                 }                            │
│                                                                 │
│  class Circle implements Shape {   type Circle struct{...}      │
│      public double area() {...}    func (c Circle) Area() {...} │
│  }                                 // No "implements"!          │
│                                                                 │
│  // Must import/know interface     // Can satisfy after-the-fact│
│  // at implementation time         // without touching original │
│                                                                 │
│  Object (top type)                 interface{} or any           │
│                                                                 │
│  instanceof                        Type assertion x.(Type)      │
│                                    Type switch: switch x.(type) │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Implicit satisfaction** - no "implements" keyword
2. **Duck typing** - if it has the methods, it satisfies
3. **Empty interface** (`interface{}` or `any`) accepts any type
4. **Type assertions** - extract concrete type from interface
5. **Small interfaces** are better (Reader, Writer, Stringer)
6. **Composition** - build large interfaces from small ones
7. **Accept interfaces, return concrete types**
8. **Define interfaces at the consumer**, not provider

---

## ➡️ Next Steps

**Next Topic:** [27 - Composition](./27-composition.md)

