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

**Java inheritance:** Dog extends Animal → Dog IS-A Animal. Problems include fragile base class, tight coupling, diamond problem, and forcing IS-A when HAS-A fits better.

**Go composition:** Dog embeds Animal → Dog HAS-A Animal (with method promotion). Go follows "prefer composition over inheritance" (Gang of Four).

| Java | Go |
|------|-----|
| Dog extends Animal | Dog embeds Animal |
| Dog IS-A Animal | Dog HAS-A Animal (with promoted methods) |
| Inheritance hierarchy | Flat composition |

---

## 📝 Embedding in Action

Embed a type by declaring it without a field name. Its fields and methods are promoted to the outer type.

```go
type Engine struct {
    Horsepower int
    Type       string
}

func (e Engine) Start() {
    fmt.Printf("Engine: %s (%d HP)\n", e.Type, e.Horsepower)
}

type Car struct {
    Brand  string
    Engine  // Embedded—methods promoted
}

// Car can call c.Start() directly
```

```go
car := Car{
    Brand:  "Tesla",
    Engine: Engine{Horsepower: 670, Type: "Electric"},
}
car.Start()           // Promoted from Engine
fmt.Println(car.Horsepower)  // Promoted field
fmt.Println(car.Engine.Type) // Explicit access
// Output: Engine: Electric (670 HP)
//         670
//         Electric
```

### Complete Example: Embedding

```go
package main

import "fmt"

type Engine struct {
    Horsepower int
    Type       string
}

func (e Engine) Start() {
    fmt.Printf("Engine starting: %s (%d HP)\n", e.Type, e.Horsepower)
}

type Wheels struct {
    Count int
    Size  string
}

func (w Wheels) Rotate() {
    fmt.Printf("%d wheels rotating (%s)\n", w.Count, w.Size)
}

type Car struct {
    Brand  string
    Model  string
    Engine
    Wheels
}

func (c Car) Drive() {
    fmt.Printf("Driving %s %s\n", c.Brand, c.Model)
    c.Start()
    c.Rotate()
}

func main() {
    car := Car{
        Brand:  "Tesla",
        Model:  "Model S",
        Engine: Engine{Horsepower: 670, Type: "Electric"},
        Wheels: Wheels{Count: 4, Size: "21 inch"},
    }

    car.Drive()
    fmt.Printf("car.Horsepower = %d (from Engine)\n", car.Horsepower)
    fmt.Printf("car.Engine.Type = %s\n", car.Engine.Type)
}
```

**Output:**
```
Driving Tesla Model S
Engine starting: Electric (670 HP)
4 wheels rotating (21 inch)
car.Horsepower = 670 (from Engine)
car.Engine.Type = Electric
```

---

## 🔄 Method Overriding

Define a method on the outer type with the same name to "override" the embedded type's method. Access the original via `outer.Embedded.Method()`.

```go
type Animal struct {
    Name string
}

func (a Animal) Speak() string {
    return "..."
}

type Dog struct {
    Animal
}

func (d Dog) Speak() string {
    return "Woof!"
}

dog := Dog{Animal: Animal{Name: "Buddy"}}
fmt.Println(dog.Speak())        // "Woof!" (overridden)
fmt.Println(dog.Animal.Speak()) // "..." (original)
```

### Complete Example: Method Overriding

```go
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
    Animal
    Breed string
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

func main() {
    dog := Dog{Animal: Animal{Name: "Buddy"}, Breed: "Golden Retriever"}
    cat := Cat{Animal: Animal{Name: "Whiskers"}}

    fmt.Printf("dog.Speak() = %q (overridden)\n", dog.Speak())
    fmt.Printf("cat.Speak() = %q (overridden)\n", cat.Speak())
    fmt.Printf("dog.Info() = %q (promoted)\n", dog.Info())
    fmt.Printf("dog.Animal.Speak() = %q (original)\n", dog.Animal.Speak())
}
```

**Output:**
```
dog.Speak() = "Woof!" (overridden)
cat.Speak() = "Meow!" (overridden)
dog.Info() = "I am Buddy" (promoted)
dog.Animal.Speak() = "..." (original)
```

---

## 🎭 Polymorphism with Interfaces

Composition provides code reuse; interfaces provide polymorphism. Together they replace inheritance.

```go
type Speaker interface {
    Speak() string
}

func MakeSpeak(s Speaker) {
    fmt.Println(s.Speak())
}

animals := []Speaker{
    Dog{Animal{Name: "Buddy"}},
    Cat{Animal{Name: "Whiskers"}},
}
for _, a := range animals {
    MakeSpeak(a)
}
// Output: Woof!
//         Meow!
```

### Complete Example: Polymorphism

```go
package main

import "fmt"

type Speaker interface {
    Speak() string
}

type Animal struct {
    Name string
}

type Dog struct{ Animal }
func (d Dog) Speak() string { return "Woof!" }

type Cat struct{ Animal }
func (c Cat) Speak() string { return "Meow!" }

type Bird struct{ Animal }
func (b Bird) Speak() string { return "Tweet!" }

func MakeSpeak(s Speaker) {
    fmt.Println(s.Speak())
}

func main() {
    animals := []Speaker{
        Dog{Animal{Name: "Buddy"}},
        Cat{Animal{Name: "Whiskers"}},
        Bird{Animal{Name: "Tweety"}},
    }
    for _, a := range animals {
        MakeSpeak(a)
    }
}
```

**Output:**
```
Woof!
Meow!
Tweet!
```

---

## 🏭 Production Pattern

Embed reusable components (e.g., `Identifiable`, `Timestamps`) into domain models.

```go
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

type User struct {
    Identifiable
    Timestamps
    Name  string
    Email string
}

type Order struct {
    Identifiable
    Timestamps
    UserID string
    Total  float64
}
```

### Complete Example: Production Pattern

```go
package main

import (
    "fmt"
    "time"
)

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

type User struct {
    Identifiable
    Timestamps
    Name  string
    Email string
}

type Order struct {
    Identifiable
    Timestamps
    UserID string
    Total  float64
}

func main() {
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
    }

    fmt.Printf("user.ID = %s\n", user.ID)
    fmt.Printf("order.ID = %s\n", order.ID)
    order.Touch()
    fmt.Println("order.Touch() called")
}
```

**Output:**
```
user.ID = user-001
order.ID = order-001
order.Touch() called
```

---

## 🎯 Key Takeaways

1. **No inheritance** in Go—use composition
2. **Embedding** promotes fields and methods
3. **Override** by defining method on outer type
4. **Access original** via `outer.Embedded.Method()`
5. **Interfaces** provide polymorphism
6. **Composition + Interfaces** = Go's OOP
7. **Reusable components** through embedding

---

## ➡️ Next Steps

**Next Topic:** [28 - Error Handling Best Practices](./28-error-handling.md)
