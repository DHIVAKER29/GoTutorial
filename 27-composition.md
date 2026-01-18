# 27 - Composition Over Inheritance

> How Go achieves code reuse without inheritance.

---

## 📌 What You'll Learn

- Why Go has no inheritance
- Composition through embedding
- Method forwarding
- When to embed vs when to use fields
- Achieving polymorphism without inheritance

---

## 🚫 Go Has No Inheritance!

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA INHERITANCE                   GO COMPOSITION              │
│                                                                 │
│  ┌─────────┐                        ┌─────────┐                │
│  │ Animal  │ (parent)               │ Animal  │                │
│  └────┬────┘                        └─────────┘                │
│       │ extends                           ↓                     │
│  ┌────┴────┐                        ┌─────────────┐            │
│  │   Dog   │ (child)                │    Dog      │            │
│  └─────────┘                        │ ┌─────────┐ │            │
│                                     │ │ Animal  │ │ embedded  │
│  Dog IS-A Animal                    │ └─────────┘ │            │
│                                     └─────────────┘            │
│                                                                 │
│                                     Dog HAS-A Animal            │
│                                     (but with method promotion) │
│                                                                 │
│  PROBLEMS WITH INHERITANCE:                                     │
│  • Fragile base class problem                                   │
│  • Tight coupling                                               │
│  • Diamond problem (multiple inheritance)                       │
│  • Forces IS-A when HAS-A is more appropriate                   │
│                                                                 │
│  GO'S PHILOSOPHY:                                               │
│  "Prefer composition over inheritance"                          │
│  - Gang of Four Design Patterns                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Embedding in Action

```go
// composition.go
package main

import "fmt"

// Base "component"
type Engine struct {
    Horsepower int
    Type       string
}

func (e Engine) Start() {
    fmt.Printf("   Engine starting: %s (%d HP)\n", e.Type, e.Horsepower)
}

func (e Engine) Stop() {
    fmt.Println("   Engine stopping")
}

// Another component
type Wheels struct {
    Count int
    Size  string
}

func (w Wheels) Rotate() {
    fmt.Printf("   %d wheels rotating (%s)\n", w.Count, w.Size)
}

// Composed type (NOT inheritance!)
type Car struct {
    Brand  string
    Model  string
    Engine        // Embedded - methods promoted!
    Wheels        // Embedded
}

// Car can have its own methods
func (c Car) Drive() {
    fmt.Printf("   Driving %s %s\n", c.Brand, c.Model)
    c.Start()   // Promoted from Engine!
    c.Rotate()  // Promoted from Wheels!
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           COMPOSITION THROUGH EMBEDDING                   ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    car := Car{
        Brand:  "Tesla",
        Model:  "Model S",
        Engine: Engine{Horsepower: 670, Type: "Electric"},
        Wheels: Wheels{Count: 4, Size: "21 inch"},
    }
    
    fmt.Println("\n📊 Using Composed Type:")
    car.Drive()
    
    fmt.Println("\n📊 Accessing Embedded Fields:")
    fmt.Printf("   car.Brand = %s\n", car.Brand)
    fmt.Printf("   car.Horsepower = %d (from Engine)\n", car.Horsepower)
    fmt.Printf("   car.Count = %d (from Wheels)\n", car.Count)
    
    fmt.Println("\n📊 Calling Promoted Methods:")
    car.Start()  // From Engine
    car.Stop()   // From Engine
    car.Rotate() // From Wheels
    
    fmt.Println("\n📊 Accessing Embedded Type Explicitly:")
    fmt.Printf("   car.Engine.Type = %s\n", car.Engine.Type)
}
```

---

## 🔄 Method Overriding

```go
// method_override.go
package main

import "fmt"

type Animal struct {
    Name string
}

func (a Animal) Speak() string {
    return "..."
}

func (a Animal) Info() string {
    return "I am " + a.Name
}

type Dog struct {
    Animal  // Embedded
    Breed   string
}

// "Override" the Speak method
func (d Dog) Speak() string {
    return "Woof!"
}

// Dog.Info() is inherited (promoted) from Animal

type Cat struct {
    Animal
}

func (c Cat) Speak() string {
    return "Meow!"
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           METHOD 'OVERRIDING'                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    dog := Dog{Animal: Animal{Name: "Buddy"}, Breed: "Golden Retriever"}
    cat := Cat{Animal: Animal{Name: "Whiskers"}}
    
    fmt.Println("\n📊 Method Override:")
    fmt.Printf("   dog.Speak() = %q (overridden)\n", dog.Speak())
    fmt.Printf("   cat.Speak() = %q (overridden)\n", cat.Speak())
    
    fmt.Println("\n📊 Promoted Method (not overridden):")
    fmt.Printf("   dog.Info() = %q (from Animal)\n", dog.Info())
    fmt.Printf("   cat.Info() = %q (from Animal)\n", cat.Info())
    
    fmt.Println("\n📊 Accessing 'Parent' Method:")
    fmt.Printf("   dog.Animal.Speak() = %q (original)\n", dog.Animal.Speak())
}
```

---

## 🎭 Polymorphism with Interfaces

```go
// polymorphism.go
package main

import "fmt"

// Interface defines behavior
type Speaker interface {
    Speak() string
}

// Base component
type Animal struct {
    Name string
}

// Different types with embedded Animal
type Dog struct {
    Animal
}

func (d Dog) Speak() string {
    return "Woof!"
}

type Cat struct {
    Animal
}

func (c Cat) Speak() string {
    return "Meow!"
}

type Bird struct {
    Animal
}

func (b Bird) Speak() string {
    return "Tweet!"
}

// Function accepting interface (polymorphism!)
func MakeSpeak(s Speaker) {
    fmt.Printf("   %s\n", s.Speak())
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           POLYMORPHISM WITH INTERFACES                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    animals := []Speaker{
        Dog{Animal{Name: "Buddy"}},
        Cat{Animal{Name: "Whiskers"}},
        Bird{Animal{Name: "Tweety"}},
    }
    
    fmt.Println("\n📊 Polymorphic Behavior:")
    for _, animal := range animals {
        MakeSpeak(animal)
    }
    
    fmt.Println("\n💡 Key Insight:")
    fmt.Println("   • Composition gives us code reuse (embedded fields/methods)")
    fmt.Println("   • Interfaces give us polymorphism")
    fmt.Println("   • Together they replace inheritance!")
}
```

---

## 🏭 Production Pattern

```go
// composition_production.go
package main

import (
    "fmt"
    "time"
)

// Reusable components
type Timestamps struct {
    CreatedAt time.Time
    UpdatedAt time.Time
}

func (t *Timestamps) Touch() {
    t.UpdatedAt = time.Now()
}

type Identifiable struct {
    ID string
}

// Domain models compose common behaviors
type User struct {
    Identifiable  // Has ID
    Timestamps    // Has timestamps
    Name  string
    Email string
}

type Order struct {
    Identifiable
    Timestamps
    UserID string
    Total  float64
    Status string
}

type Product struct {
    Identifiable
    Timestamps
    Name  string
    Price float64
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION COMPOSITION PATTERN                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    now := time.Now()
    
    user := User{
        Identifiable: Identifiable{ID: "user-001"},
        Timestamps:   Timestamps{CreatedAt: now, UpdatedAt: now},
        Name:         "Alice",
        Email:        "alice@example.com",
    }
    
    order := Order{
        Identifiable: Identifiable{ID: "order-001"},
        Timestamps:   Timestamps{CreatedAt: now, UpdatedAt: now},
        UserID:       user.ID,
        Total:        99.99,
        Status:       "pending",
    }
    
    fmt.Println("\n📊 All Models Have ID and Timestamps:")
    fmt.Printf("   user.ID = %s\n", user.ID)
    fmt.Printf("   user.CreatedAt = %s\n", user.CreatedAt.Format("2006-01-02"))
    
    fmt.Printf("   order.ID = %s\n", order.ID)
    fmt.Printf("   order.CreatedAt = %s\n", order.CreatedAt.Format("2006-01-02"))
    
    fmt.Println("\n📊 Promoted Methods:")
    order.Touch()  // From Timestamps
    fmt.Printf("   order.Touch() → UpdatedAt changed\n")
}
```

---

## 🎯 Key Takeaways

1. **No inheritance** in Go - use composition
2. **Embedding** promotes fields and methods
3. **Override** by defining method on outer type
4. **Access original** via `outer.Embedded.Method()`
5. **Interfaces** provide polymorphism
6. **Composition + Interfaces** = Go's OOP
7. **Reusable components** through embedding

---

## ➡️ Next Steps

**Next Topic:** [28 - Error Handling Best Practices](./28-error-handling.md)

