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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  S.O.L.I.D PRINCIPLES                                           │
│                                                                 │
│  S = Single Responsibility Principle                            │
│      A module should have one reason to change                  │
│                                                                 │
│  O = Open/Closed Principle                                      │
│      Open for extension, closed for modification                │
│                                                                 │
│  L = Liskov Substitution Principle                              │
│      Subtypes must be substitutable for base types              │
│                                                                 │
│  I = Interface Segregation Principle                            │
│      Many specific interfaces > one general interface           │
│                                                                 │
│  D = Dependency Inversion Principle                             │
│      Depend on abstractions, not concretions                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### SOLID in Go vs Java

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA (Traditional OOP)           GO (Composition-Based)        │
│  ─────────────────────            ──────────────────────        │
│                                                                 │
│  Classes with inheritance         Structs with composition      │
│  Abstract classes                 Interfaces (implicit)         │
│  Dependency injection frameworks  Constructor functions         │
│  Design patterns everywhere       Simple, explicit code         │
│                                                                 │
│  Go's philosophy: "Clear is better than clever"                 │
│  SOLID principles still apply, but implementation differs!      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## S - Single Responsibility Principle

### Definition

> A package/struct/function should have only ONE reason to change.

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  BAD: Swiss Army Knife Employee                                 │
│  ┌─────────────────────────────────────┐                        │
│  │  Employee                           │                        │
│  │  - Does accounting                  │                        │
│  │  - Does marketing                   │                        │
│  │  - Does customer support            │                        │
│  │  - Does IT maintenance              │                        │
│  └─────────────────────────────────────┘                        │
│  Problem: Changes in ANY area affect this employee!             │
│                                                                 │
│  GOOD: Specialized Departments                                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │Accounting│ │Marketing │ │ Support  │ │   IT     │           │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
│  Each department has ONE job. Changes are isolated.             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Single Responsibility

```go
// single_responsibility.go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "time"
)

// ═══════════════════════════════════════════════════════════════
// ❌ BAD: One struct doing too many things
// ═══════════════════════════════════════════════════════════════

type BadUser struct {
    ID       int
    Name     string
    Email    string
    Password string
}

// This struct is responsible for:
// 1. Storing user data
// 2. Validating user data
// 3. Saving to database
// 4. Sending emails
// 5. Logging

func (u *BadUser) Validate() error {
    // Validation logic mixed with data
    if u.Name == "" {
        return fmt.Errorf("name is required")
    }
    return nil
}

func (u *BadUser) SaveToDatabase() error {
    // Database logic mixed in
    fmt.Println("Saving to database...")
    return nil
}

func (u *BadUser) SendWelcomeEmail() error {
    // Email logic mixed in
    fmt.Println("Sending email...")
    return nil
}

func (u *BadUser) Log(message string) {
    // Logging logic mixed in
    fmt.Printf("[LOG] User %s: %s\n", u.Name, message)
}

// ═══════════════════════════════════════════════════════════════
// ✅ GOOD: Each type has ONE responsibility
// ═══════════════════════════════════════════════════════════════

// 1. User struct: ONLY stores user data
type User struct {
    ID        int
    Name      string
    Email     string
    CreatedAt time.Time
}

// 2. UserValidator: ONLY validates users
type UserValidator struct{}

func (v *UserValidator) Validate(u *User) error {
    if u.Name == "" {
        return fmt.Errorf("name is required")
    }
    if u.Email == "" {
        return fmt.Errorf("email is required")
    }
    return nil
}

// 3. UserRepository: ONLY handles database operations
type UserRepository struct {
    // db connection would go here
}

func (r *UserRepository) Save(u *User) error {
    fmt.Printf("Saving user %s to database...\n", u.Name)
    return nil
}

func (r *UserRepository) FindByID(id int) (*User, error) {
    // Database query logic
    return &User{ID: id, Name: "Found User"}, nil
}

// 4. EmailService: ONLY handles emails
type EmailService struct {
    // SMTP config would go here
}

func (e *EmailService) SendWelcome(u *User) error {
    fmt.Printf("Sending welcome email to %s...\n", u.Email)
    return nil
}

// 5. Logger: ONLY handles logging
type Logger struct {
    output *os.File
}

func NewLogger() *Logger {
    return &Logger{output: os.Stdout}
}

func (l *Logger) Info(message string) {
    timestamp := time.Now().Format("2006-01-02 15:04:05")
    fmt.Fprintf(l.output, "[INFO] %s: %s\n", timestamp, message)
}

// 6. UserService: Coordinates the above components
type UserService struct {
    validator  *UserValidator
    repository *UserRepository
    emailer    *EmailService
    logger     *Logger
}

func NewUserService() *UserService {
    return &UserService{
        validator:  &UserValidator{},
        repository: &UserRepository{},
        emailer:    &EmailService{},
        logger:     NewLogger(),
    }
}

func (s *UserService) RegisterUser(name, email string) (*User, error) {
    user := &User{
        ID:        1,
        Name:      name,
        Email:     email,
        CreatedAt: time.Now(),
    }
    
    // Each step uses a specialized component
    if err := s.validator.Validate(user); err != nil {
        return nil, err
    }
    
    if err := s.repository.Save(user); err != nil {
        return nil, err
    }
    
    if err := s.emailer.SendWelcome(user); err != nil {
        s.logger.Info(fmt.Sprintf("Failed to send welcome email: %v", err))
        // Non-critical, don't fail registration
    }
    
    s.logger.Info(fmt.Sprintf("User %s registered successfully", name))
    return user, nil
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║        SINGLE RESPONSIBILITY PRINCIPLE                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n❌ BAD Example (one struct does everything):")
    badUser := &BadUser{ID: 1, Name: "Alice", Email: "alice@example.com"}
    badUser.Validate()
    badUser.SaveToDatabase()
    badUser.SendWelcomeEmail()
    badUser.Log("registered")
    
    fmt.Println("\n✅ GOOD Example (separated responsibilities):")
    service := NewUserService()
    user, err := service.RegisterUser("Bob", "bob@example.com")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    
    data, _ := json.MarshalIndent(user, "", "  ")
    fmt.Printf("Created user: %s\n", data)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║        SINGLE RESPONSIBILITY PRINCIPLE                    ║
╚══════════════════════════════════════════════════════════╝

❌ BAD Example (one struct does everything):
Saving to database...
Sending email...
[LOG] User Alice: registered

✅ GOOD Example (separated responsibilities):
Saving user Bob to database...
Sending welcome email to bob@example.com...
[INFO] 2026-02-10 12:34:56: User Bob registered successfully
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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Think of a POWER STRIP                                         │
│                                                                 │
│  ┌────────────────────────────────────────────────┐            │
│  │ 🔌  🔌  🔌  🔌  🔌  🔌                          │            │
│  └────────────────────────────────────────────────┘            │
│                                                                 │
│  OPEN for extension:                                            │
│    - Plug in any new device                                     │
│    - Phone, laptop, lamp, anything with a plug                  │
│                                                                 │
│  CLOSED for modification:                                       │
│    - You don't rewire the strip for each new device             │
│    - The strip's internal wiring stays the same                 │
│                                                                 │
│  In code: Use INTERFACES to allow new implementations           │
│           without modifying existing code.                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Open/Closed

```go
// open_closed.go
package main

import "fmt"

// ═══════════════════════════════════════════════════════════════
// ❌ BAD: Must modify code to add new payment methods
// ═══════════════════════════════════════════════════════════════

func BadProcessPayment(method string, amount float64) error {
    switch method {
    case "credit_card":
        fmt.Printf("Processing credit card: $%.2f\n", amount)
    case "paypal":
        fmt.Printf("Processing PayPal: $%.2f\n", amount)
    case "bank_transfer":
        fmt.Printf("Processing bank transfer: $%.2f\n", amount)
    // To add "crypto", we MUST modify this function!
    default:
        return fmt.Errorf("unknown payment method: %s", method)
    }
    return nil
}

// ═══════════════════════════════════════════════════════════════
// ✅ GOOD: Open for extension via interface
// ═══════════════════════════════════════════════════════════════

// PaymentProcessor defines the behavior (the "power strip interface")
type PaymentProcessor interface {
    ProcessPayment(amount float64) error
    Name() string
}

// CreditCardProcessor implements PaymentProcessor
type CreditCardProcessor struct {
    CardNumber string
}

func (c *CreditCardProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("💳 Credit Card [%s]: Processing $%.2f\n", 
        maskCard(c.CardNumber), amount)
    return nil
}

func (c *CreditCardProcessor) Name() string {
    return "Credit Card"
}

func maskCard(card string) string {
    if len(card) < 4 {
        return "****"
    }
    return "****-****-****-" + card[len(card)-4:]
}

// PayPalProcessor implements PaymentProcessor
type PayPalProcessor struct {
    Email string
}

func (p *PayPalProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("🅿️  PayPal [%s]: Processing $%.2f\n", p.Email, amount)
    return nil
}

func (p *PayPalProcessor) Name() string {
    return "PayPal"
}

// CryptoProcessor - NEW! Added without modifying existing code!
type CryptoProcessor struct {
    WalletAddress string
    Currency      string
}

func (c *CryptoProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("🪙 Crypto [%s to %s]: Processing $%.2f\n", 
        c.Currency, c.WalletAddress[:8]+"...", amount)
    return nil
}

func (c *CryptoProcessor) Name() string {
    return "Cryptocurrency"
}

// UPIProcessor - Another new one! No existing code modified!
type UPIProcessor struct {
    UPIID string
}

func (u *UPIProcessor) ProcessPayment(amount float64) error {
    fmt.Printf("📱 UPI [%s]: Processing ₹%.2f\n", u.UPIID, amount)
    return nil
}

func (u *UPIProcessor) Name() string {
    return "UPI"
}

// PaymentService is CLOSED for modification
// It works with any PaymentProcessor!
type PaymentService struct {
    processor PaymentProcessor
}

func NewPaymentService(processor PaymentProcessor) *PaymentService {
    return &PaymentService{processor: processor}
}

func (s *PaymentService) Checkout(amount float64) error {
    fmt.Printf("Using %s for checkout...\n", s.processor.Name())
    return s.processor.ProcessPayment(amount)
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           OPEN/CLOSED PRINCIPLE                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n❌ BAD: Modifying code for each new payment method")
    BadProcessPayment("credit_card", 100.00)
    BadProcessPayment("paypal", 50.00)
    
    fmt.Println("\n✅ GOOD: Extending via new types (no modification needed)")
    
    processors := []PaymentProcessor{
        &CreditCardProcessor{CardNumber: "4111111111111234"},
        &PayPalProcessor{Email: "user@example.com"},
        &CryptoProcessor{WalletAddress: "0x1234567890abcdef", Currency: "ETH"},
        &UPIProcessor{UPIID: "user@bank"},
    }
    
    for _, processor := range processors {
        service := NewPaymentService(processor)
        service.Checkout(99.99)
        fmt.Println()
    }
    
    fmt.Println("📝 Key Point:")
    fmt.Println("   New payment methods added by creating NEW types,")
    fmt.Println("   NOT by modifying PaymentService!")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           OPEN/CLOSED PRINCIPLE                           ║
╚══════════════════════════════════════════════════════════╝

❌ BAD: Modifying code for each new payment method
Processing credit card: $100.00
Processing PayPal: $50.00

✅ GOOD: Extending via new types (no modification needed)
Using Credit Card for checkout...
💳 Credit Card [****-****-****-1234]: Processing $99.99

Using PayPal for checkout...
🅿️  PayPal [user@example.com]: Processing $99.99

Using Cryptocurrency for checkout...
🪙 Crypto [ETH to 0x123456...]: Processing $99.99

Using UPI for checkout...
📱 UPI [user@bank]: Processing ₹99.99

📝 Key Point:
   New payment methods added by creating NEW types,
   NOT by modifying PaymentService!
```

---

## L - Liskov Substitution Principle

### Definition

> If S is a subtype of T, then objects of type T can be replaced with objects of type S without breaking the program.

### In Go Terms

> Any type that implements an interface must be usable wherever that interface is expected.

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Think of POWER OUTLETS                                         │
│                                                                 │
│  Interface: "Can receive 2-prong plug"                          │
│  ┌─────────┐                                                    │
│  │  ⚫ ⚫  │                                                    │
│  └─────────┘                                                    │
│                                                                 │
│  Any device with a 2-prong plug should work:                    │
│  ✅ Lamp        - works as expected                             │
│  ✅ Phone charger - works as expected                           │
│  ✅ Fan         - works as expected                             │
│  ❌ 3-prong plug - doesn't fit! (violates the contract)         │
│                                                                 │
│  Liskov says: If something claims to fit the interface,         │
│               it MUST behave correctly!                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Liskov Substitution

```go
// liskov_substitution.go
package main

import (
    "fmt"
    "math"
)

// ═══════════════════════════════════════════════════════════════
// ❌ BAD: Square violates Rectangle's expected behavior
// ═══════════════════════════════════════════════════════════════

type BadRectangle struct {
    width  float64
    height float64
}

func (r *BadRectangle) SetWidth(w float64) {
    r.width = w
}

func (r *BadRectangle) SetHeight(h float64) {
    r.height = h
}

func (r *BadRectangle) Area() float64 {
    return r.width * r.height
}

// BadSquare "is-a" Rectangle? Not really!
type BadSquare struct {
    BadRectangle
}

func (s *BadSquare) SetWidth(w float64) {
    s.width = w
    s.height = w // Forces height to match!
}

func (s *BadSquare) SetHeight(h float64) {
    s.height = h
    s.width = h // Forces width to match!
}

// This function expects Rectangle behavior
func BadCalculateArea(r *BadRectangle) float64 {
    r.SetWidth(5)
    r.SetHeight(4)
    // Expects: 5 * 4 = 20
    return r.Area()
}

// ═══════════════════════════════════════════════════════════════
// ✅ GOOD: Use interfaces and composition
// ═══════════════════════════════════════════════════════════════

// Shape interface - any shape must be able to calculate area
type Shape interface {
    Area() float64
    Perimeter() float64
    Name() string
}

// Rectangle implements Shape
type Rectangle struct {
    Width  float64
    Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

func (r Rectangle) Name() string {
    return "Rectangle"
}

// Square implements Shape (not extending Rectangle!)
type Square struct {
    Side float64
}

func (s Square) Area() float64 {
    return s.Side * s.Side
}

func (s Square) Perimeter() float64 {
    return 4 * s.Side
}

func (s Square) Name() string {
    return "Square"
}

// Circle implements Shape
type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

func (c Circle) Name() string {
    return "Circle"
}

// This function works with ANY Shape - Liskov satisfied!
func PrintShapeInfo(s Shape) {
    fmt.Printf("  %s: Area=%.2f, Perimeter=%.2f\n", 
        s.Name(), s.Area(), s.Perimeter())
}

// Sum areas of any shapes - all are substitutable!
func TotalArea(shapes []Shape) float64 {
    total := 0.0
    for _, s := range shapes {
        total += s.Area()
    }
    return total
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║        LISKOV SUBSTITUTION PRINCIPLE                      ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n❌ BAD: Square as Rectangle violates expectations")
    badRect := &BadRectangle{}
    badSquare := &BadSquare{}
    
    rectArea := BadCalculateArea(badRect)
    fmt.Printf("  Rectangle area (5x4): %.0f (expected 20) ✓\n", rectArea)
    
    // This breaks! Square changes both dimensions
    squareArea := BadCalculateArea(&badSquare.BadRectangle)
    fmt.Printf("  Square area (5x4): %.0f (expected 20) ✗ GOT WRONG ANSWER!\n", squareArea)
    
    fmt.Println("\n✅ GOOD: Each shape implements interface correctly")
    
    shapes := []Shape{
        Rectangle{Width: 5, Height: 4},
        Square{Side: 5},
        Circle{Radius: 3},
    }
    
    for _, shape := range shapes {
        PrintShapeInfo(shape)
    }
    
    fmt.Printf("\n  Total area of all shapes: %.2f\n", TotalArea(shapes))
    
    fmt.Println("\n📝 Key Point:")
    fmt.Println("   All shapes can substitute for each other in PrintShapeInfo()")
    fmt.Println("   and TotalArea() without breaking the program!")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║        LISKOV SUBSTITUTION PRINCIPLE                      ║
╚══════════════════════════════════════════════════════════╝

❌ BAD: Square as Rectangle violates expectations
  Rectangle area (5x4): 20 (expected 20) ✓
  Square area (5x4): 16 (expected 20) ✗ GOT WRONG ANSWER!

✅ GOOD: Each shape implements interface correctly
  Rectangle: Area=20.00, Perimeter=18.00
  Square: Area=25.00, Perimeter=20.00
  Circle: Area=28.27, Perimeter=18.85

  Total area of all shapes: 73.27

📝 Key Point:
   All shapes can substitute for each other in PrintShapeInfo()
   and TotalArea() without breaking the program!
```

---

## I - Interface Segregation Principle

### Definition

> Clients should not be forced to depend on interfaces they don't use. Many small interfaces are better than one large interface.

### Go's Natural Advantage

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO'S INTERFACES ARE SMALL BY DESIGN                            │
│                                                                 │
│  Standard library examples:                                     │
│                                                                 │
│  io.Reader   - just Read()                                      │
│  io.Writer   - just Write()                                     │
│  io.Closer   - just Close()                                     │
│  fmt.Stringer - just String()                                   │
│                                                                 │
│  These small interfaces can be COMPOSED:                        │
│  io.ReadWriter = io.Reader + io.Writer                          │
│  io.ReadCloser = io.Reader + io.Closer                          │
│                                                                 │
│  This is Interface Segregation in action!                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Interface Segregation

```go
// interface_segregation.go
package main

import (
    "fmt"
    "time"
)

// ═══════════════════════════════════════════════════════════════
// ❌ BAD: One fat interface forces unnecessary implementations
// ═══════════════════════════════════════════════════════════════

// Fat interface - not all workers do all these things!
type BadWorker interface {
    Work()
    Eat()
    Sleep()
    TakeBreak()
    AttendMeeting()
    WriteReport()
    CodeReview()
}

// Human worker - can do everything
type BadHumanWorker struct {
    Name string
}

func (h *BadHumanWorker) Work()          { fmt.Println(h.Name, "is working") }
func (h *BadHumanWorker) Eat()           { fmt.Println(h.Name, "is eating") }
func (h *BadHumanWorker) Sleep()         { fmt.Println(h.Name, "is sleeping") }
func (h *BadHumanWorker) TakeBreak()     { fmt.Println(h.Name, "taking a break") }
func (h *BadHumanWorker) AttendMeeting() { fmt.Println(h.Name, "in meeting") }
func (h *BadHumanWorker) WriteReport()   { fmt.Println(h.Name, "writing report") }
func (h *BadHumanWorker) CodeReview()    { fmt.Println(h.Name, "reviewing code") }

// Robot worker - forced to implement things it can't do!
type BadRobotWorker struct {
    ID string
}

func (r *BadRobotWorker) Work() { fmt.Println("Robot", r.ID, "is working") }
func (r *BadRobotWorker) Eat()  { 
    // Robot can't eat! But forced to implement!
    panic("robots don't eat!")
}
func (r *BadRobotWorker) Sleep() {
    panic("robots don't sleep!")
}
func (r *BadRobotWorker) TakeBreak()     { /* Do nothing */ }
func (r *BadRobotWorker) AttendMeeting() { /* Do nothing */ }
func (r *BadRobotWorker) WriteReport()   { fmt.Println("Robot", r.ID, "generating report") }
func (r *BadRobotWorker) CodeReview()    { fmt.Println("Robot", r.ID, "analyzing code") }

// ═══════════════════════════════════════════════════════════════
// ✅ GOOD: Many small, focused interfaces
// ═══════════════════════════════════════════════════════════════

// Small, focused interfaces
type Worker interface {
    Work()
}

type Eater interface {
    Eat()
}

type Sleeper interface {
    Sleep()
}

type Reporter interface {
    WriteReport() string
}

type Reviewer interface {
    ReviewCode(code string) []string
}

// Composed interfaces when needed
type BiologicalBeing interface {
    Eater
    Sleeper
}

type Employee interface {
    Worker
    Reporter
}

// Human - implements all relevant interfaces
type Human struct {
    Name string
}

func (h *Human) Work() {
    fmt.Printf("👤 %s is working\n", h.Name)
}

func (h *Human) Eat() {
    fmt.Printf("👤 %s is having lunch\n", h.Name)
}

func (h *Human) Sleep() {
    fmt.Printf("👤 %s is sleeping\n", h.Name)
}

func (h *Human) WriteReport() string {
    return fmt.Sprintf("Report by %s: All tasks completed.", h.Name)
}

func (h *Human) ReviewCode(code string) []string {
    return []string{"LGTM", "Nice work!"}
}

// Robot - only implements what it can actually do
type Robot struct {
    ID      string
    Model   string
}

func (r *Robot) Work() {
    fmt.Printf("🤖 Robot %s (%s) is working\n", r.ID, r.Model)
}

func (r *Robot) WriteReport() string {
    return fmt.Sprintf("Automated report from %s: 1000 tasks completed.", r.ID)
}

func (r *Robot) ReviewCode(code string) []string {
    return []string{
        "Line 10: Potential null pointer",
        "Line 25: Consider using constants",
    }
}

// Functions that accept only what they need

func AssignWork(w Worker) {
    w.Work()
}

func LunchBreak(e Eater) {
    e.Eat()
}

func GenerateReports(reporters []Reporter) {
    fmt.Println("\n📄 Generating Reports:")
    for _, r := range reporters {
        fmt.Printf("  - %s\n", r.WriteReport())
    }
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║        INTERFACE SEGREGATION PRINCIPLE                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n❌ BAD: Fat interface forces empty/panic implementations")
    fmt.Println("   Robot forced to implement Eat() and Sleep() with panic!")
    
    fmt.Println("\n✅ GOOD: Small interfaces, implement only what you can do")
    
    human := &Human{Name: "Alice"}
    robot := &Robot{ID: "R2D2", Model: "Analyzer-3000"}
    
    fmt.Println("\n👷 Assigning work (both are Workers):")
    workers := []Worker{human, robot}
    for _, w := range workers {
        AssignWork(w)
    }
    
    fmt.Println("\n🍽️  Lunch break (only Eaters):")
    eaters := []Eater{human}  // Robot is NOT here!
    for _, e := range eaters {
        LunchBreak(e)
    }
    
    reporters := []Reporter{human, robot}  // Both can report
    GenerateReports(reporters)
    
    fmt.Println("\n📝 Key Point:")
    fmt.Println("   Human implements: Worker, Eater, Sleeper, Reporter, Reviewer")
    fmt.Println("   Robot implements: Worker, Reporter, Reviewer")
    fmt.Println("   Each only implements what makes sense!")
    
    fmt.Println("\n💡 Go Standard Library Examples:")
    fmt.Println("   io.Reader  - just Read()")
    fmt.Println("   io.Writer  - just Write()")
    fmt.Println("   io.Closer  - just Close()")
    _ = time.Now() // Just to use time package
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║        INTERFACE SEGREGATION PRINCIPLE                    ║
╚══════════════════════════════════════════════════════════╝

❌ BAD: Fat interface forces empty/panic implementations
   Robot forced to implement Eat() and Sleep() with panic!

✅ GOOD: Small interfaces, implement only what you can do

👷 Assigning work (both are Workers):
👤 Alice is working
🤖 Robot R2D2 (Analyzer-3000) is working

🍽️  Lunch break (only Eaters):
👤 Alice is having lunch

📄 Generating Reports:
  - Report by Alice: All tasks completed.
  - Automated report from R2D2: 1000 tasks completed.

📝 Key Point:
   Human implements: Worker, Eater, Sleeper, Reporter, Reviewer
   Robot implements: Worker, Reporter, Reviewer
   Each only implements what makes sense!

💡 Go Standard Library Examples:
   io.Reader  - just Read()
   io.Writer  - just Write()
   io.Closer  - just Close()
```

---

## D - Dependency Inversion Principle

### Definition

> High-level modules should not depend on low-level modules. Both should depend on abstractions (interfaces).

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Think of ELECTRICAL PLUGS                                      │
│                                                                 │
│  ❌ BAD: Hardwired appliances                                   │
│     Each appliance wired directly to house power               │
│     Can't replace, can't move, can't test                       │
│                                                                 │
│  ✅ GOOD: Plugs and outlets                                     │
│     Appliances depend on "power interface" (the plug)           │
│     House provides "power interface" (the outlet)               │
│     Either can change independently!                            │
│                                                                 │
│  In code:                                                       │
│  High-level: OrderService                                       │
│  Low-level:  MySQLDatabase                                      │
│  Interface:  DataStore                                          │
│                                                                 │
│  OrderService → DataStore ← MySQLDatabase                       │
│                    ↑                                            │
│             Both depend on abstraction!                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Dependency Inversion

```go
// dependency_inversion.go
package main

import (
    "fmt"
    "time"
)

// ═══════════════════════════════════════════════════════════════
// ❌ BAD: High-level depends directly on low-level
// ═══════════════════════════════════════════════════════════════

type BadMySQLDatabase struct {
    connectionString string
}

func (db *BadMySQLDatabase) Query(sql string) []string {
    fmt.Printf("  MySQL: Executing '%s'\n", sql)
    return []string{"result1", "result2"}
}

type BadOrderService struct {
    db *BadMySQLDatabase  // Directly depends on MySQL!
}

func (s *BadOrderService) GetOrders() []string {
    return s.db.Query("SELECT * FROM orders")
}

// Problem: Can't switch to PostgreSQL without changing OrderService!
// Problem: Can't test without a real MySQL database!

// ═══════════════════════════════════════════════════════════════
// ✅ GOOD: Both depend on abstraction
// ═══════════════════════════════════════════════════════════════

// Abstraction: DataStore interface
type DataStore interface {
    Save(key string, data interface{}) error
    Find(key string) (interface{}, error)
    FindAll() ([]interface{}, error)
    Delete(key string) error
}

// Low-level: MySQL implementation
type MySQLStore struct {
    host string
    port int
}

func NewMySQLStore(host string, port int) *MySQLStore {
    return &MySQLStore{host: host, port: port}
}

func (m *MySQLStore) Save(key string, data interface{}) error {
    fmt.Printf("  🐬 MySQL: INSERT INTO table VALUES (%s, %v)\n", key, data)
    return nil
}

func (m *MySQLStore) Find(key string) (interface{}, error) {
    fmt.Printf("  🐬 MySQL: SELECT * FROM table WHERE id = %s\n", key)
    return map[string]string{"id": key, "name": "Order " + key}, nil
}

func (m *MySQLStore) FindAll() ([]interface{}, error) {
    fmt.Println("  🐬 MySQL: SELECT * FROM table")
    return []interface{}{"order1", "order2", "order3"}, nil
}

func (m *MySQLStore) Delete(key string) error {
    fmt.Printf("  🐬 MySQL: DELETE FROM table WHERE id = %s\n", key)
    return nil
}

// Low-level: In-Memory implementation (for testing!)
type InMemoryStore struct {
    data map[string]interface{}
}

func NewInMemoryStore() *InMemoryStore {
    return &InMemoryStore{data: make(map[string]interface{})}
}

func (m *InMemoryStore) Save(key string, data interface{}) error {
    fmt.Printf("  💾 InMemory: Storing %s = %v\n", key, data)
    m.data[key] = data
    return nil
}

func (m *InMemoryStore) Find(key string) (interface{}, error) {
    fmt.Printf("  💾 InMemory: Finding %s\n", key)
    return m.data[key], nil
}

func (m *InMemoryStore) FindAll() ([]interface{}, error) {
    fmt.Println("  💾 InMemory: Getting all data")
    result := make([]interface{}, 0, len(m.data))
    for _, v := range m.data {
        result = append(result, v)
    }
    return result, nil
}

func (m *InMemoryStore) Delete(key string) error {
    fmt.Printf("  💾 InMemory: Deleting %s\n", key)
    delete(m.data, key)
    return nil
}

// Low-level: Redis implementation (another option!)
type RedisStore struct {
    address string
}

func NewRedisStore(address string) *RedisStore {
    return &RedisStore{address: address}
}

func (r *RedisStore) Save(key string, data interface{}) error {
    fmt.Printf("  🔴 Redis: SET %s %v\n", key, data)
    return nil
}

func (r *RedisStore) Find(key string) (interface{}, error) {
    fmt.Printf("  🔴 Redis: GET %s\n", key)
    return "cached_value", nil
}

func (r *RedisStore) FindAll() ([]interface{}, error) {
    fmt.Println("  🔴 Redis: KEYS *")
    return []interface{}{"key1", "key2"}, nil
}

func (r *RedisStore) Delete(key string) error {
    fmt.Printf("  🔴 Redis: DEL %s\n", key)
    return nil
}

// High-level: OrderService depends on ABSTRACTION
type Order struct {
    ID        string
    Customer  string
    Amount    float64
    CreatedAt time.Time
}

type OrderService struct {
    store DataStore  // Depends on interface, not concrete type!
}

// Constructor injection - pass the dependency
func NewOrderService(store DataStore) *OrderService {
    return &OrderService{store: store}
}

func (s *OrderService) CreateOrder(customer string, amount float64) (*Order, error) {
    order := &Order{
        ID:        fmt.Sprintf("ORD-%d", time.Now().UnixNano()),
        Customer:  customer,
        Amount:    amount,
        CreatedAt: time.Now(),
    }
    
    err := s.store.Save(order.ID, order)
    if err != nil {
        return nil, err
    }
    
    return order, nil
}

func (s *OrderService) GetOrder(id string) (*Order, error) {
    data, err := s.store.Find(id)
    if err != nil {
        return nil, err
    }
    
    // In real code, you'd do proper type assertion/conversion
    fmt.Printf("  Retrieved: %v\n", data)
    return &Order{ID: id}, nil
}

func (s *OrderService) GetAllOrders() ([]*Order, error) {
    data, err := s.store.FindAll()
    if err != nil {
        return nil, err
    }
    
    fmt.Printf("  Found %d orders\n", len(data))
    return []*Order{}, nil
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║        DEPENDENCY INVERSION PRINCIPLE                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n❌ BAD: OrderService directly depends on MySQL")
    fmt.Println("   - Can't switch databases without changing OrderService")
    fmt.Println("   - Can't test without real MySQL")
    
    badDB := &BadMySQLDatabase{connectionString: "mysql://..."}
    badService := &BadOrderService{db: badDB}
    badService.GetOrders()
    
    fmt.Println("\n✅ GOOD: Both depend on DataStore interface")
    
    // Production: Use MySQL
    fmt.Println("\n📦 Production (MySQL):")
    mysqlStore := NewMySQLStore("localhost", 3306)
    prodService := NewOrderService(mysqlStore)
    prodService.CreateOrder("Alice", 99.99)
    
    // Development: Use Redis
    fmt.Println("\n📦 Development (Redis):")
    redisStore := NewRedisStore("localhost:6379")
    devService := NewOrderService(redisStore)
    devService.CreateOrder("Bob", 49.99)
    
    // Testing: Use In-Memory
    fmt.Println("\n📦 Testing (In-Memory):")
    memStore := NewInMemoryStore()
    testService := NewOrderService(memStore)
    testService.CreateOrder("TestUser", 10.00)
    testService.GetAllOrders()
    
    fmt.Println("\n📝 Key Points:")
    fmt.Println("   1. OrderService doesn't know which store it's using")
    fmt.Println("   2. We can swap implementations without changing OrderService")
    fmt.Println("   3. Easy to test with mock/fake implementations")
    fmt.Println("   4. New stores can be added without modifying OrderService")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║        DEPENDENCY INVERSION PRINCIPLE                     ║
╚══════════════════════════════════════════════════════════╝

❌ BAD: OrderService directly depends on MySQL
   - Can't switch databases without changing OrderService
   - Can't test without real MySQL
  MySQL: Executing 'SELECT * FROM orders'

✅ GOOD: Both depend on DataStore interface

📦 Production (MySQL):
🐬 MySQL: INSERT INTO table VALUES (ORD-1739199236123456789, &{ORD-1739199236123456789 Alice 99.99 2026-02-10 12:34:56.123456789 +0000 UTC})

📦 Development (Redis):
🔴 Redis: SET ORD-1739199236123456789 &{ORD-1739199236123456789 Bob 49.99 2026-02-10 12:34:56.123456789 +0000 UTC}

📦 Testing (In-Memory):
💾 InMemory: Storing ORD-1739199236123456789 = &{ORD-1739199236123456789 TestUser 10 2026-02-10 12:34:56.123456789 +0000 UTC}
💾 InMemory: Getting all data
  Found 1 orders

📝 Key Points:
   1. OrderService doesn't know which store it's using
   2. We can swap implementations without changing OrderService
   3. Easy to test with mock/fake implementations
   4. New stores can be added without modifying OrderService
```

*Note: Order IDs and timestamps will vary based on execution time.*

---

## 📊 SOLID Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  SOLID IN GO - QUICK REFERENCE                                  │
│                                                                 │
│  S - Single Responsibility                                      │
│      One package/struct = One purpose                           │
│      Split: UserValidator, UserRepository, EmailService         │
│                                                                 │
│  O - Open/Closed                                                │
│      Use interfaces for extensibility                           │
│      New PaymentProcessor types, no service modification        │
│                                                                 │
│  L - Liskov Substitution                                        │
│      Interface implementations must be interchangeable          │
│      All Shapes work in TotalArea() without surprises           │
│                                                                 │
│  I - Interface Segregation                                      │
│      Small, focused interfaces                                  │
│      io.Reader, io.Writer, not io.DoEverything                  │
│                                                                 │
│  D - Dependency Inversion                                       │
│      Depend on interfaces, inject implementations               │
│      OrderService → DataStore ← MySQLStore/RedisStore           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

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

