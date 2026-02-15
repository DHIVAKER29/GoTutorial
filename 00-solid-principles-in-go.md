# 00 - SOLID Principles in Go

> How to write clean, maintainable Go code using SOLID principles adapted for Go's paradigm.

---

## 📌 What You'll Learn

- What SOLID principles are
- How each principle applies to Go (not OOP!)
- Why Go's design naturally encourages good patterns
- Sample programs demonstrating each principle
- Real-world production patterns

---

## 🤔 What is SOLID?

SOLID is a set of 5 design principles that help you write maintainable, flexible, and understandable code.

### The Five Principles

| Letter | Principle | Definition |
|-------|-----------|------------|
| **S** | Single Responsibility | A module should have one reason to change |
| **O** | Open/Closed | Open for extension, closed for modification |
| **L** | Liskov Substitution | Subtypes must be substitutable for base types |
| **I** | Interface Segregation | Many specific interfaces > one general interface |
| **D** | Dependency Inversion | Depend on abstractions, not concretions |

### SOLID in Go vs Java

| Aspect | Java (Traditional OOP) | Go (Composition-Based) |
|--------|------------------------|--------------------------|
| Structure | Classes with inheritance | Structs with composition |
| Abstractions | Abstract classes | Interfaces (implicit) |
| Dependency injection | Frameworks | Constructor functions |
| Philosophy | Design patterns everywhere | Simple, explicit code |

*Go's philosophy: "Clear is better than clever." SOLID principles still apply, but implementation differs!*

---

## S - Single Responsibility Principle

### Definition

> A package/struct/function should have only ONE reason to change.

### Real-World Analogy

- **BAD:** Swiss Army Knife Employee — does accounting, marketing, support, IT. Changes in any area affect this employee.
- **GOOD:** Specialized Departments — each department has one job. Changes are isolated.

### Bad Example: One Struct Doing Too Much

```go
type BadUser struct {
    ID       int
    Name     string
    Email    string
    Password string
}

// This struct handles: data, validation, DB, email, logging — too many!
func (u *BadUser) Validate() error { /* ... */ return nil }
func (u *BadUser) SaveToDatabase() error { /* ... */ return nil }
func (u *BadUser) SendWelcomeEmail() error { /* ... */ return nil }
func (u *BadUser) Log(message string) { /* ... */ }
```

### Good Example: Separated Responsibilities

```go
// User: ONLY stores data
type User struct {
    ID        int
    Name      string
    Email     string
    CreatedAt time.Time
}

// UserValidator: ONLY validates
type UserValidator struct{}
func (v *UserValidator) Validate(u *User) error { /* ... */ return nil }

// UserRepository: ONLY handles database
type UserRepository struct{}
func (r *UserRepository) Save(u *User) error { /* ... */ return nil }

// EmailService: ONLY handles emails
type EmailService struct{}
func (e *EmailService) SendWelcome(u *User) error { /* ... */ return nil }
```

### Complete Example: Single Responsibility

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

type User struct {
    ID        int
    Name      string
    Email     string
    CreatedAt time.Time
}

type UserValidator struct{}
func (v *UserValidator) Validate(u *User) error {
    if u.Name == "" { return fmt.Errorf("name is required") }
    if u.Email == "" { return fmt.Errorf("email is required") }
    return nil
}

type UserRepository struct{}
func (r *UserRepository) Save(u *User) error {
    fmt.Printf("Saving user %s to database...\n", u.Name)
    return nil
}

type UserService struct {
    validator  *UserValidator
    repository *UserRepository
}

func NewUserService() *UserService {
    return &UserService{
        validator:  &UserValidator{},
        repository: &UserRepository{},
    }
}

func (s *UserService) RegisterUser(name, email string) (*User, error) {
    user := &User{ID: 1, Name: name, Email: email, CreatedAt: time.Now()}
    if err := s.validator.Validate(user); err != nil { return nil, err }
    if err := s.repository.Save(user); err != nil { return nil, err }
    return user, nil
}

func main() {
    service := NewUserService()
    user, _ := service.RegisterUser("Bob", "bob@example.com")
    data, _ := json.MarshalIndent(user, "", "  ")
    fmt.Printf("Created user: %s\n", data)
}
```

**Output:**
```
Saving user Bob to database...
Created user: {
  "ID": 1,
  "Name": "Bob",
  "Email": "bob@example.com",
  "CreatedAt": "2026-02-10T12:34:56.123456789Z"
}
```

*Note: Timestamp will vary based on execution time.*

---

## O - Open/Closed Principle

### Definition

> Software should be **open for extension** but **closed for modification**.

### Real-World Analogy

Think of a **power strip**: open for extension (plug in any device), closed for modification (you don't rewire it for each device). In code: use interfaces to allow new implementations without modifying existing code.

### Bad Example: Must Modify to Add Payment Methods

```go
func BadProcessPayment(method string, amount float64) error {
    switch method {
    case "credit_card": fmt.Printf("Processing credit card: $%.2f\n", amount)
    case "paypal": fmt.Printf("Processing PayPal: $%.2f\n", amount)
    // To add "crypto", we MUST modify this function!
    default: return fmt.Errorf("unknown payment method: %s", method)
    }
    return nil
}
```

### Good Example: Interface for Extension

```go
type PaymentProcessor interface {
    ProcessPayment(amount float64) error
    Name() string
}

type CreditCardProcessor struct{ CardNumber string }
func (c *CreditCardProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("Credit Card: Processing $%.2f\n", amount)
    return nil
}
func (c *CreditCardProcessor) Name() string { return "Credit Card" }

// CryptoProcessor - NEW! Added without modifying existing code!
type CryptoProcessor struct{ WalletAddress, Currency string }
func (c *CryptoProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("Crypto [%s]: Processing $%.2f\n", c.Currency, amount)
    return nil
}
func (c *CryptoProcessor) Name() string { return "Cryptocurrency" }
```

### Complete Example: Open/Closed

```go
package main

import "fmt"

type PaymentProcessor interface {
    ProcessPayment(amount float64) error
    Name() string
}

type CreditCardProcessor struct{ CardNumber string }
func (c *CreditCardProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("Credit Card: Processing $%.2f\n", amount)
    return nil
}
func (c *CreditCardProcessor) Name() string { return "Credit Card" }

type PayPalProcessor struct{ Email string }
func (p *PayPalProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("PayPal [%s]: Processing $%.2f\n", p.Email, amount)
    return nil
}
func (p *PayPalProcessor) Name() string { return "PayPal" }

type PaymentService struct{ processor PaymentProcessor }
func NewPaymentService(processor PaymentProcessor) *PaymentService {
    return &PaymentService{processor: processor}
}
func (s *PaymentService) Checkout(amount float64) error {
    fmt.Printf("Using %s for checkout...\n", s.processor.Name())
    return s.processor.ProcessPayment(amount)
}

func main() {
    processors := []PaymentProcessor{
        &CreditCardProcessor{CardNumber: "4111111111111234"},
        &PayPalProcessor{Email: "user@example.com"},
    }
    for _, p := range processors {
        NewPaymentService(p).Checkout(99.99)
    }
}
```

**Output:**
```
Using Credit Card for checkout...
Credit Card: Processing $99.99

Using PayPal for checkout...
PayPal [user@example.com]: Processing $99.99
```

---

## L - Liskov Substitution Principle

### Definition

> If S is a subtype of T, then objects of type T can be replaced with objects of type S without breaking the program.

**In Go terms:** Any type that implements an interface must be usable wherever that interface is expected.

### Real-World Analogy

Think of **power outlets**: any device with a 2-prong plug should work. A 3-prong plug doesn't fit (violates the contract). Liskov says: if something claims to fit the interface, it MUST behave correctly.

### Complete Example: Liskov Substitution

```go
package main

import (
    "fmt"
    "math"
)

type Shape interface {
    Area() float64
    Perimeter() float64
    Name() string
}

type Rectangle struct{ Width, Height float64 }
func (r Rectangle) Area() float64      { return r.Width * r.Height }
func (r Rectangle) Perimeter() float64 { return 2 * (r.Width + r.Height) }
func (r Rectangle) Name() string      { return "Rectangle" }

type Square struct{ Side float64 }
func (s Square) Area() float64      { return s.Side * s.Side }
func (s Square) Perimeter() float64 { return 4 * s.Side }
func (s Square) Name() string       { return "Square" }

type Circle struct{ Radius float64 }
func (c Circle) Area() float64      { return math.Pi * c.Radius * c.Radius }
func (c Circle) Perimeter() float64 { return 2 * math.Pi * c.Radius }
func (c Circle) Name() string      { return "Circle" }

func PrintShapeInfo(s Shape) {
    fmt.Printf("  %s: Area=%.2f, Perimeter=%.2f\n", s.Name(), s.Area(), s.Perimeter())
}

func main() {
    shapes := []Shape{
        Rectangle{Width: 5, Height: 4},
        Square{Side: 5},
        Circle{Radius: 3},
    }
    for _, shape := range shapes {
        PrintShapeInfo(shape)
    }
}
```

**Output:**
```
  Rectangle: Area=20.00, Perimeter=18.00
  Square: Area=25.00, Perimeter=20.00
  Circle: Area=28.27, Perimeter=18.85
```

---

## I - Interface Segregation Principle

### Definition

> Clients should not be forced to depend on interfaces they don't use. Many small interfaces are better than one large interface.

### Go's Natural Advantage

Go's standard library uses small interfaces by design:

| Interface | Methods |
|-----------|---------|
| `io.Reader` | `Read()` |
| `io.Writer` | `Write()` |
| `io.Closer` | `Close()` |
| `fmt.Stringer` | `String()` |

These can be composed: `io.ReadWriter = io.Reader + io.Writer`

### Complete Example: Interface Segregation

```go
package main

import "fmt"

type Worker interface{ Work() }
type Eater interface{ Eat() }
type Reporter interface{ WriteReport() string }

type Human struct{ Name string }
func (h *Human) Work() { fmt.Printf("%s is working\n", h.Name) }
func (h *Human) Eat() { fmt.Printf("%s is having lunch\n", h.Name) }
func (h *Human) WriteReport() string { return fmt.Sprintf("Report by %s", h.Name) }

type Robot struct{ ID string }
func (r *Robot) Work() { fmt.Printf("Robot %s is working\n", r.ID) }
func (r *Robot) WriteReport() string { return fmt.Sprintf("Automated report from %s", r.ID) }

func AssignWork(w Worker) { w.Work() }
func LunchBreak(e Eater) { e.Eat() }

func main() {
    human := &Human{Name: "Alice"}
    robot := &Robot{ID: "R2D2"}
    
    workers := []Worker{human, robot}
    for _, w := range workers { AssignWork(w) }
    
    LunchBreak(human)  // Robot is NOT an Eater
}
```

**Output:**
```
Alice is working
Robot R2D2 is working
Alice is having lunch
```

---

## D - Dependency Inversion Principle

### Definition

> High-level modules should not depend on low-level modules. Both should depend on abstractions (interfaces).

### Real-World Analogy

- **BAD:** Hardwired appliances — can't replace, can't test
- **GOOD:** Plugs and outlets — both depend on "power interface." Either can change independently.

### Complete Example: Dependency Inversion

```go
package main

import (
    "fmt"
    "time"
)

type DataStore interface {
    Save(key string, data interface{}) error
    Find(key string) (interface{}, error)
}

type MySQLStore struct{}
func (m *MySQLStore) Save(key string, data interface{}) error {
    fmt.Printf("MySQL: INSERT %s\n", key)
    return nil
}
func (m *MySQLStore) Find(key string) (interface{}, error) {
    return map[string]string{"id": key}, nil
}

type InMemoryStore struct{ data map[string]interface{} }
func NewInMemoryStore() *InMemoryStore { return &InMemoryStore{data: make(map[string]interface{})} }
func (m *InMemoryStore) Save(key string, data interface{}) error {
    m.data[key] = data
    return nil
}
func (m *InMemoryStore) Find(key string) (interface{}, error) {
    return m.data[key], nil
}

type OrderService struct{ store DataStore }
func NewOrderService(store DataStore) *OrderService { return &OrderService{store: store} }
func (s *OrderService) CreateOrder(customer string, amount float64) error {
    id := fmt.Sprintf("ORD-%d", time.Now().UnixNano())
    return s.store.Save(id, map[string]interface{}{"customer": customer, "amount": amount})
}

func main() {
    fmt.Println("Production (MySQL):")
    NewOrderService(&MySQLStore{}).CreateOrder("Alice", 99.99)
    
    fmt.Println("Testing (In-Memory):")
    NewOrderService(NewInMemoryStore()).CreateOrder("TestUser", 10.00)
}
```

**Output:**
```
Production (MySQL):
MySQL: INSERT ORD-1739199236123456789

Testing (In-Memory):
```

---

## 📊 SOLID Summary

| Principle | In Go |
|-----------|-------|
| **S** - Single Responsibility | One package/struct = one purpose. Split: UserValidator, UserRepository, EmailService |
| **O** - Open/Closed | Use interfaces for extensibility. New PaymentProcessor types, no service modification |
| **L** - Liskov Substitution | Interface implementations must be interchangeable. All Shapes work in TotalArea() |
| **I** - Interface Segregation | Small, focused interfaces. io.Reader, io.Writer, not io.DoEverything |
| **D** - Dependency Inversion | Depend on interfaces, inject implementations. OrderService → DataStore ← MySQLStore |

---

## 🎯 Key Takeaways

1. **Go naturally encourages SOLID** through its design (small interfaces, composition)
2. **SRP**: One package = one responsibility
3. **OCP**: Use interfaces to extend without modifying
4. **LSP**: All implementations must be substitutable
5. **ISP**: Keep interfaces small (Go stdlib does this!)
6. **DIP**: Inject dependencies via constructor functions

---

## ➡️ Continue Learning

Apply these principles as you learn more Go concepts!

**Next Topic:** [01 - Introduction to Go](./01-introduction-to-go.md)
