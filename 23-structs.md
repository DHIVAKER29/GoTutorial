# 23 - Structs

> Go's way of grouping related data together - the foundation of data modeling.

---

## 📌 What You'll Learn

- What structs are (Go's alternative to classes)
- Defining and creating structs
- Accessing and modifying fields
- Struct embedding (Go's composition)
- Anonymous structs
- Struct tags (for JSON, database, etc.)
- Sample programs for every concept

---

## 🤔 What is a Struct?

### Definition

> A **struct** (structure) is a composite data type that groups together zero or more values of different types under a single name.

### The Java Developer's Mental Model

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  FROM JAVA CLASSES TO GO STRUCTS                                │
│                                                                 │
│  JAVA (class = data + behavior bundled)                         │
│  ┌─────────────────────────────────────┐                       │
│  │  class Person {                     │                       │
│  │      String name;                   │ ← Data (fields)       │
│  │      int age;                       │                       │
│  │                                     │                       │
│  │      void greet() { ... }           │ ← Behavior (methods)  │
│  │      int getAge() { ... }           │                       │
│  │  }                                  │                       │
│  └─────────────────────────────────────┘                       │
│                                                                 │
│  GO (data and behavior are SEPARATE)                            │
│  ┌─────────────────────────────────────┐                       │
│  │  type Person struct {               │                       │
│  │      Name string                    │ ← Just data           │
│  │      Age  int                       │                       │
│  │  }                                  │                       │
│  └─────────────────────────────────────┘                       │
│                                                                 │
│  ┌─────────────────────────────────────┐                       │
│  │  func (p Person) Greet() { ... }    │ ← Behavior attached   │
│  │  func (p Person) GetAge() { ... }   │   via methods         │
│  └─────────────────────────────────────┘                       │
│                                                                 │
│  KEY INSIGHT: In Go, you define data first, then attach         │
│  behavior. This is more flexible than bundling everything.      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Struct Memory Layout

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  type Person struct {                                           │
│      Name string  // 16 bytes (pointer + length)                │
│      Age  int     // 8 bytes                                    │
│      Active bool  // 1 byte + padding                           │
│  }                                                              │
│                                                                 │
│  Memory Layout:                                                 │
│  ┌────────────────────────────────────────────────────────────┐│
│  │ Name (string)      │ Age (int) │ Active│ Padding │          ││
│  │ pointer + length   │           │       │         │          ││
│  │ 16 bytes           │ 8 bytes   │ 1 byte│ 7 bytes │          ││
│  └────────────────────────────────────────────────────────────┘│
│                                                                 │
│  • Fields are stored CONTIGUOUSLY                               │
│  • Compiler may add PADDING for alignment                       │
│  • Total size depends on field order (memory efficiency)        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Defining Structs

```go
// struct_definition.go
package main

import "fmt"

// Simple struct
type Person struct {
    Name    string
    Age     int
    Email   string
    Active  bool
}

// Struct with same-type fields grouped
type Rectangle struct {
    Width, Height float64  // Both are float64
}

// Nested struct
type Address struct {
    Street  string
    City    string
    Country string
    ZipCode string
}

type Employee struct {
    ID      int
    Name    string
    Address Address  // Nested struct
    Salary  float64
}

// Empty struct (zero size!)
type Empty struct{}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              DEFINING STRUCTS                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 Simple Struct:")
    var p Person
    fmt.Printf("   Zero value: %+v\n", p)
    
    fmt.Println("\n📊 Grouped Fields:")
    r := Rectangle{Width: 10, Height: 5}
    fmt.Printf("   Rectangle: %+v\n", r)
    
    fmt.Println("\n📊 Nested Struct:")
    emp := Employee{
        ID:   1,
        Name: "Alice",
        Address: Address{
            Street:  "123 Main St",
            City:    "NYC",
            Country: "USA",
            ZipCode: "10001",
        },
        Salary: 75000,
    }
    fmt.Printf("   Employee: %+v\n", emp)
    
    fmt.Println("\n📊 Empty Struct (size = 0 bytes!):")
    var e Empty
    fmt.Printf("   Empty struct size: %d bytes\n", unsafe_Sizeof(e))
    fmt.Println("   Use case: Sets (map[string]struct{})")
}

// Simulating unsafe.Sizeof for demo
func unsafe_Sizeof(x interface{}) int {
    return 0  // Would use unsafe.Sizeof in real code
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              DEFINING STRUCTS                             ║
╚══════════════════════════════════════════════════════════╝

📊 Simple Struct:
   Zero value: {Name: Age:0 Email: Active:false}

📊 Grouped Fields:
   Rectangle: {Width:10 Height:5}

📊 Nested Struct:
   Employee: {ID:1 Name:Alice Address:{Street:123 Main St City:NYC Country:USA ZipCode:10001} Salary:75000}

📊 Empty Struct (size = 0 bytes!):
   Empty struct size: 0 bytes
   Use case: Sets (map[string]struct{})
```

---

## 🏗️ Creating Struct Instances

```go
// struct_creation.go
package main

import "fmt"

type Person struct {
    Name  string
    Age   int
    Email string
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              CREATING STRUCT INSTANCES                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Method 1: Zero value
    fmt.Println("\n📊 Method 1: Zero Value (var)")
    var p1 Person
    fmt.Printf("   var p1 Person = %+v\n", p1)
    
    // Method 2: Named fields (RECOMMENDED)
    fmt.Println("\n📊 Method 2: Named Fields (recommended)")
    p2 := Person{
        Name:  "Alice",
        Age:   25,
        Email: "alice@example.com",
    }
    fmt.Printf("   p2 = %+v\n", p2)
    
    // Method 3: Positional (fragile, not recommended)
    fmt.Println("\n📊 Method 3: Positional (not recommended)")
    p3 := Person{"Bob", 30, "bob@example.com"}
    fmt.Printf("   p3 = %+v\n", p3)
    fmt.Println("   ⚠️ Warning: Breaks if field order changes!")
    
    // Method 4: Partial initialization
    fmt.Println("\n📊 Method 4: Partial (others get zero values)")
    p4 := Person{Name: "Charlie"}
    fmt.Printf("   p4 = %+v\n", p4)
    
    // Method 5: Pointer with &
    fmt.Println("\n📊 Method 5: Pointer with &")
    p5 := &Person{Name: "Diana", Age: 28}
    fmt.Printf("   p5 (pointer) = %+v\n", p5)
    fmt.Printf("   *p5 (value) = %+v\n", *p5)
    
    // Method 6: new() function
    fmt.Println("\n📊 Method 6: new() - returns pointer to zero value")
    p6 := new(Person)
    p6.Name = "Eve"
    p6.Age = 22
    fmt.Printf("   p6 = %+v\n", p6)
    
    // Method 7: Constructor function (idiomatic Go)
    fmt.Println("\n📊 Method 7: Constructor Function (idiomatic)")
    p7 := NewPerson("Frank", 35, "frank@example.com")
    fmt.Printf("   p7 = %+v\n", p7)
}

// Constructor function - returns pointer
func NewPerson(name string, age int, email string) *Person {
    // Can add validation here
    if age < 0 {
        age = 0
    }
    return &Person{
        Name:  name,
        Age:   age,
        Email: email,
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              CREATING STRUCT INSTANCES                    ║
╚══════════════════════════════════════════════════════════╝

📊 Method 1: Zero Value (var)
   var p1 Person = {Name: Age:0 Email:}

📊 Method 2: Named Fields (recommended)
   p2 = {Name:Alice Age:25 Email:alice@example.com}

📊 Method 3: Positional (not recommended)
   p3 = {Name:Bob Age:30 Email:bob@example.com}
   ⚠️ Warning: Breaks if field order changes!

📊 Method 4: Partial (others get zero values)
   p4 = {Name:Charlie Age:0 Email:}

📊 Method 5: Pointer with &
   p5 (pointer) = &{Name:Diana Age:28 Email:}
   *p5 (value) = {Name:Diana Age:28 Email:}

📊 Method 6: new() - returns pointer to zero value
   p6 = &{Name:Eve Age:22 Email:}

📊 Method 7: Constructor Function (idiomatic)
   p7 = &{Name:Frank Age:35 Email:frank@example.com}
```

---

## 🔧 Accessing and Modifying Fields

```go
// struct_fields.go
package main

import "fmt"

type Person struct {
    Name  string
    Age   int
    Email string
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           ACCESSING AND MODIFYING FIELDS                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    p := Person{Name: "Alice", Age: 25, Email: "alice@example.com"}
    
    // Accessing fields
    fmt.Println("\n📊 Accessing Fields:")
    fmt.Printf("   p.Name = %q\n", p.Name)
    fmt.Printf("   p.Age = %d\n", p.Age)
    fmt.Printf("   p.Email = %q\n", p.Email)
    
    // Modifying fields
    fmt.Println("\n📊 Modifying Fields:")
    p.Age = 26
    fmt.Printf("   After p.Age = 26: %+v\n", p)
    
    // Pointer to struct
    fmt.Println("\n📊 Pointer to Struct:")
    ptr := &p
    
    // These are EQUIVALENT:
    fmt.Printf("   (*ptr).Name = %q\n", (*ptr).Name)
    fmt.Printf("   ptr.Name = %q (automatic dereference!)\n", ptr.Name)
    
    // Modify through pointer
    ptr.Age = 27  // Modifies original!
    fmt.Printf("   After ptr.Age = 27: p = %+v\n", p)
    
    // Structs are values (copied!)
    fmt.Println("\n⚠️ Structs are Values (copied!):")
    original := Person{Name: "Bob", Age: 30}
    copy := original  // COPY!
    copy.Age = 31
    fmt.Printf("   original = %+v\n", original)
    fmt.Printf("   copy = %+v (independent)\n", copy)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           ACCESSING AND MODIFYING FIELDS                  ║
╚══════════════════════════════════════════════════════════╝

📊 Accessing Fields:
   p.Name = "Alice"
   p.Age = 25
   p.Email = "alice@example.com"

📊 Modifying Fields:
   After p.Age = 26: {Name:Alice Age:26 Email:alice@example.com}

📊 Pointer to Struct:
   (*ptr).Name = "Alice"
   ptr.Name = "Alice" (automatic dereference!)

📊 Modifying Through Pointer:
   After ptr.Age = 27: p = {Name:Alice Age:27 Email:alice@example.com}

⚠️ Structs are Values (copied!):
   original = {Name:Bob Age:30 Email:}
   copy = {Name:Bob Age:31 Email:} (independent)
```

---

## 🧩 Struct Embedding (Composition)

```go
// struct_embedding.go
package main

import "fmt"

// Base "component"
type Address struct {
    Street  string
    City    string
    Country string
}

// Another "component"
type ContactInfo struct {
    Email string
    Phone string
}

// Embedding (composition, not inheritance!)
type Person struct {
    Name string
    Age  int
    Address     // Embedded - no field name!
    ContactInfo // Embedded
}

// Traditional nesting (for comparison)
type PersonNested struct {
    Name    string
    Age     int
    Address Address
    Contact ContactInfo
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              STRUCT EMBEDDING                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // With embedding
    fmt.Println("\n📊 With Embedding:")
    p := Person{
        Name: "Alice",
        Age:  25,
        Address: Address{
            Street:  "123 Main St",
            City:    "NYC",
            Country: "USA",
        },
        ContactInfo: ContactInfo{
            Email: "alice@example.com",
            Phone: "555-0101",
        },
    }
    
    // PROMOTED FIELDS - access directly!
    fmt.Printf("   p.Name = %q\n", p.Name)
    fmt.Printf("   p.City = %q (promoted from Address!)\n", p.City)
    fmt.Printf("   p.Email = %q (promoted from ContactInfo!)\n", p.Email)
    
    // Can still access via embedded type
    fmt.Printf("   p.Address.City = %q\n", p.Address.City)
    
    // Without embedding (nested)
    fmt.Println("\n📊 Without Embedding (nested):")
    pn := PersonNested{
        Name: "Bob",
        Age:  30,
        Address: Address{
            City: "LA",
        },
        Contact: ContactInfo{
            Email: "bob@example.com",
        },
    }
    
    // Must access through field name
    fmt.Printf("   pn.Address.City = %q (must use field name)\n", pn.Address.City)
    // fmt.Printf("   pn.City = ...  // ❌ Won't work!
    
    fmt.Println("\n💡 Embedding = Composition + Field Promotion")
    fmt.Println("   Not inheritance! Go has no inheritance.")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              STRUCT EMBEDDING                             ║
╚══════════════════════════════════════════════════════════╝

📊 With Embedding:
   p.Name = "Alice"
   p.City = "NYC" (promoted from Address!)
   p.Email = "alice@example.com" (promoted from ContactInfo!)
   p.Address.City = "NYC"

📊 Without Embedding (nested):
   pn.Address.City = "LA" (must use field name)

💡 Embedding = Composition + Field Promotion
   Not inheritance! Go has no inheritance.
```

---

## 🏷️ Struct Tags

```go
// struct_tags.go
package main

import (
    "encoding/json"
    "fmt"
)

// Struct with tags for JSON
type User struct {
    ID        int    `json:"id"`
    FirstName string `json:"first_name"`
    LastName  string `json:"last_name"`
    Email     string `json:"email,omitempty"`  // Omit if empty
    Password  string `json:"-"`                 // Never include
    Age       int    `json:"age,omitempty"`
}

// Multiple tags
type DBUser struct {
    ID   int    `json:"id" db:"user_id" validate:"required"`
    Name string `json:"name" db:"user_name" validate:"min=2,max=50"`
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              STRUCT TAGS                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // JSON serialization
    fmt.Println("\n📊 JSON Serialization:")
    user := User{
        ID:        1,
        FirstName: "Alice",
        LastName:  "Smith",
        Email:     "alice@example.com",
        Password:  "secret123",  // Won't appear in JSON!
        Age:       0,            // Will be omitted (omitempty)
    }
    
    jsonBytes, _ := json.MarshalIndent(user, "   ", "  ")
    fmt.Println("   " + string(jsonBytes))
    
    // JSON deserialization
    fmt.Println("\n📊 JSON Deserialization:")
    jsonStr := `{"id": 2, "first_name": "Bob", "last_name": "Jones"}`
    var user2 User
    json.Unmarshal([]byte(jsonStr), &user2)
    fmt.Printf("   Parsed: %+v\n", user2)
    
    // Tag syntax
    fmt.Println("\n📊 Tag Syntax:")
    fmt.Println("   `key:\"value\" key2:\"value2\"`")
    fmt.Println("")
    fmt.Println("   Common tags:")
    fmt.Println("   • json:\"field_name\"        - JSON field name")
    fmt.Println("   • json:\"-\"                 - Exclude from JSON")
    fmt.Println("   • json:\",omitempty\"        - Omit if zero value")
    fmt.Println("   • db:\"column_name\"         - Database column")
    fmt.Println("   • validate:\"required,min=1\" - Validation rules")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              STRUCT TAGS                                  ║
╚══════════════════════════════════════════════════════════╝

📊 JSON Serialization:
   {
     "id": 1,
     "first_name": "Alice",
     "last_name": "Smith",
     "email": "alice@example.com"
   }

📊 JSON Deserialization:
   Parsed: {ID:2 FirstName:Bob LastName:Jones Email: Age:0}

📊 Tag Syntax:
   `key:"value" key2:"value2"`

   Common tags:
   • json:"field_name"        - JSON field name
   • json:"-"                 - Exclude from JSON
   • json:",omitempty"        - Omit if zero value
   • db:"column_name"         - Database column
   • validate:"required,min=1" - Validation rules
```

---

## 📋 Anonymous Structs

```go
// anonymous_structs.go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              ANONYMOUS STRUCTS                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Inline anonymous struct
    fmt.Println("\n📊 Inline Anonymous Struct:")
    person := struct {
        Name string
        Age  int
    }{
        Name: "Alice",
        Age:  25,
    }
    fmt.Printf("   person = %+v\n", person)
    
    // Useful for one-off data structures
    fmt.Println("\n📊 Use Case: Quick JSON Parsing")
    jsonStr := `{"status": "success", "data": {"id": 1, "name": "Widget"}}`
    
    var response struct {
        Status string `json:"status"`
        Data   struct {
            ID   int    `json:"id"`
            Name string `json:"name"`
        } `json:"data"`
    }
    
    json.Unmarshal([]byte(jsonStr), &response)
    fmt.Printf("   Status: %s\n", response.Status)
    fmt.Printf("   Data.Name: %s\n", response.Data.Name)
    
    // Useful in tests
    fmt.Println("\n📊 Use Case: Table-Driven Tests")
    testCases := []struct {
        input    int
        expected int
    }{
        {1, 2},
        {2, 4},
        {3, 6},
    }
    
    for _, tc := range testCases {
        result := tc.input * 2
        if result == tc.expected {
            fmt.Printf("   ✅ double(%d) = %d\n", tc.input, result)
        }
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              ANONYMOUS STRUCTS                            ║
╚══════════════════════════════════════════════════════════╝

📊 Inline Anonymous Struct:
   person = {Name:Alice Age:25}

📊 Use Case: Quick JSON Parsing
   Status: success
   Data.Name: Widget

📊 Use Case: Table-Driven Tests
   ✅ double(1) = 2
   ✅ double(2) = 4
   ✅ double(3) = 6
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
│      int age;                          Age  int                 │
│  }                                 }                            │
│                                                                 │
│  Person p = new Person();          p := Person{}                │
│                                    // or                        │
│  // Must use new                   p := Person{Name: "Alice"}   │
│                                                                 │
│  p.name = "Alice";                 p.Name = "Alice"             │
│                                                                 │
│  // Inheritance                    // COMPOSITION (embedding)   │
│  class Employee extends Person     type Employee struct {       │
│                                        Person  // embedded      │
│                                        Salary float64           │
│                                    }                            │
│                                                                 │
│  // Private by default             // Lowercase = private       │
│  private String name;              name string                  │
│  // Public with modifier           // Uppercase = public        │
│  public String name;               Name string                  │
│                                                                 │
│  // Getters/Setters               // Just access directly!     │
│  getName(), setName(...)           p.Name                       │
│                                                                 │
│  // Annotations                    // Struct tags               │
│  @JsonProperty("name")             `json:"name"`                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Structs are value types** - copied on assignment/passing
2. **No constructors** - use factory functions (`NewXxx()`)
3. **No inheritance** - use composition via embedding
4. **Field visibility** - Uppercase = public, lowercase = private
5. **Struct tags** - metadata for JSON, DB, validation
6. **Anonymous structs** - useful for one-off data
7. **Embedded fields** get promoted to outer struct
8. **Pointer to struct** - automatic dereference with `.`

---

## ➡️ Next Steps

**Next Topic:** [24 - Methods](./24-methods.md)

