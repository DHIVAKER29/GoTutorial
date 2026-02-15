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

### Java vs Go Mental Model

- **Java**: Class bundles data + behavior (fields + methods)
- **Go**: Data and behavior are separate. Define struct (data first), then attach methods
- **Key insight**: In Go, you define data first, then attach behavior. More flexible than bundling everything.

---

## 📊 Struct Memory Layout

- Fields are stored **contiguously** in memory
- Compiler may add **padding** for alignment
- Total size depends on field order (affects memory efficiency)
- Example: `Person{Name string, Age int, Active bool}` — string is 16 bytes, int 8 bytes, bool 1 byte + padding

---

## 📝 Defining Structs

```go
type Person struct {
    Name   string
    Age    int
    Email  string
    Active bool
}
```

```go
type Rectangle struct {
    Width, Height float64  // Grouped same-type fields
}
```

```go
type Address struct {
    Street  string
    City    string
    Country string
}

type Employee struct {
    ID      int
    Name    string
    Address Address  // Nested struct
    Salary  float64
}
```

```go
type Empty struct{}  // Zero size! Use for sets: map[string]struct{}
```

### Complete Example: Defining Structs

```go
package main

import (
    "fmt"
    "unsafe"
)

type Person struct {
    Name   string
    Age    int
    Email  string
}

type Employee struct {
    ID      int
    Name    string
    Address struct {
        Street string
        City   string
    }
}

func main() {
    var p Person
    fmt.Printf("Zero value: %+v\n", p)

    emp := Employee{
        ID:   1,
        Name: "Alice",
        Address: struct {
            Street string
            City   string
        }{Street: "123 Main", City: "NYC"},
    }
    fmt.Printf("Employee: %+v\n", emp)

    var e struct{}
    fmt.Printf("Empty struct size: %d bytes\n", unsafe.Sizeof(e))
}
```

**Output:**
```
Zero value: {Name: Age:0 Email: }
Employee: {ID:1 Name:Alice Address:{Street:123 Main City:NYC}}
Empty struct size: 0 bytes
```

---

## 🏗️ Creating Struct Instances

```go
var p1 Person  // Zero value
```

```go
p2 := Person{
    Name:  "Alice",
    Age:   25,
    Email: "alice@example.com",
}
```

```go
p3 := Person{"Bob", 30, "bob@example.com"}  // Positional (fragile!)
```

```go
p4 := Person{Name: "Charlie"}  // Partial; Age=0, Email=""
```

```go
p5 := &Person{Name: "Diana", Age: 28}  // Pointer
```

```go
p6 := new(Person)  // Returns pointer to zero value
p6.Name = "Eve"
```

```go
func NewPerson(name string, age int, email string) *Person {
    return &Person{Name: name, Age: age, Email: email}
}
p7 := NewPerson("Frank", 35, "frank@example.com")
```

### Complete Example: Creating Structs

```go
package main

import "fmt"

type Person struct {
    Name  string
    Age   int
    Email string
}

func main() {
    var p1 Person
    p2 := Person{Name: "Alice", Age: 25, Email: "alice@example.com"}
    p3 := &Person{Name: "Bob", Age: 30}

    fmt.Printf("p1: %+v\n", p1)
    fmt.Printf("p2: %+v\n", p2)
    fmt.Printf("p3: %+v\n", p3)
}
```

**Output:**
```
p1: {Name: Age:0 Email:}
p2: {Name:Alice Age:25 Email:alice@example.com}
p3: &{Name:Bob Age:30 Email:}
```

---

## 🔧 Accessing and Modifying Fields

```go
p := Person{Name: "Alice", Age: 25}
fmt.Println(p.Name, p.Age)  // Output: Alice 25
p.Age = 26
```

```go
ptr := &p
fmt.Println((*ptr).Name)  // Explicit dereference
fmt.Println(ptr.Name)     // Automatic dereference (same)
ptr.Age = 27  // Modifies original
```

```go
original := Person{Name: "Bob", Age: 30}
copy := original
copy.Age = 31
fmt.Println(original.Age)  // Output: 30 (unchanged - structs are values!)
```

### Complete Example: Field Access

```go
package main

import "fmt"

type Person struct {
    Name  string
    Age   int
    Email string
}

func main() {
    p := Person{Name: "Alice", Age: 25}
    p.Age = 26

    ptr := &p
    ptr.Age = 27
    fmt.Printf("p = %+v\n", p)
}
```

**Output:**
```
p = {Name:Alice Age:27 Email:}
```

---

## 🧩 Struct Embedding (Composition)

Embedding promotes fields to the outer struct. Access embedded fields directly or via the type name.

```go
type Address struct {
    Street  string
    City    string
    Country string
}

type Person struct {
    Name    string
    Age     int
    Address  // Embedded - no field name!
}

p := Person{
    Name: "Alice",
    Address: Address{Street: "123 Main", City: "NYC", Country: "USA"},
}
fmt.Println(p.City)          // Output: NYC (promoted!)
fmt.Println(p.Address.City)  // Also valid
```

**Embedding = composition + field promotion**. Go has no inheritance.

### Complete Example: Struct Embedding

```go
package main

import "fmt"

type Address struct {
    Street  string
    City    string
}

type Person struct {
    Name    string
    Age     int
    Address
}

func main() {
    p := Person{
        Name:    "Alice",
        Age:     25,
        Address: Address{Street: "123 Main", City: "NYC"},
    }
    fmt.Println(p.Name, p.City, p.Address.City)
}
```

**Output:**
```
Alice NYC NYC
```

---

## 🏷️ Struct Tags

Tags provide metadata for JSON, databases, validation, etc.

```go
type User struct {
    ID        int    `json:"id"`
    FirstName string `json:"first_name"`
    Email     string `json:"email,omitempty"`
    Password  string `json:"-"`  // Never include in JSON
}
```

```go
type DBUser struct {
    ID   int    `json:"id" db:"user_id" validate:"required"`
    Name string `json:"name" db:"user_name"`
}
```

### Complete Example: Struct Tags

```go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    ID        int    `json:"id"`
    FirstName string `json:"first_name"`
    LastName  string `json:"last_name"`
    Password  string `json:"-"`
}

func main() {
    user := User{
        ID:        1,
        FirstName: "Alice",
        LastName:  "Smith",
        Password:  "secret",
    }
    jsonBytes, _ := json.MarshalIndent(user, "", "  ")
    fmt.Println(string(jsonBytes))
}
```

**Output:**
```json
{
  "id": 1,
  "first_name": "Alice",
  "last_name": "Smith"
}
```

---

## 📋 Anonymous Structs

Use for one-off data structures: quick JSON parsing, table-driven tests.

```go
person := struct {
    Name string
    Age  int
}{
    Name: "Alice",
    Age:  25,
}
fmt.Printf("%+v\n", person)
```

```go
var response struct {
    Status string `json:"status"`
    Data   struct {
        ID   int    `json:"id"`
        Name string `json:"name"`
    } `json:"data"`
}
json.Unmarshal([]byte(`{"status":"ok","data":{"id":1,"name":"Widget"}}`), &response)
fmt.Println(response.Status, response.Data.Name)
```

```go
testCases := []struct {
    input    int
    expected int
}{
    {1, 2},
    {2, 4},
}
```

### Complete Example: Anonymous Structs

```go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    person := struct {
        Name string
        Age  int
    }{Name: "Alice", Age: 25}
    fmt.Printf("%+v\n", person)

    var response struct {
        Status string `json:"status"`
        ID     int    `json:"id"`
    }
    json.Unmarshal([]byte(`{"status":"success","id":42}`), &response)
    fmt.Println(response.Status, response.ID)
}
```

**Output:**
```
{Name:Alice Age:25}
success 42
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `class Person { String name; int age; }` | `type Person struct { Name string; Age int }` |
| `Person p = new Person();` | `p := Person{}` or `p := Person{Name: "Alice"}` |
| `p.name = "Alice";` | `p.Name = "Alice"` |
| Inheritance: `class Employee extends Person` | Composition: `type Employee struct { Person; Salary float64 }` |
| `private String name;` / `public String name;` | `name` (private) / `Name` (public) |
| `getName()`, `setName()` | Direct access: `p.Name` |
| `@JsonProperty("name")` | `` `json:"name"` `` |

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
