# 43 - Reflection

> Runtime type inspection and manipulation in Go.

---

## 📌 What You'll Learn

- What reflection is and when to use it
- reflect.Type and reflect.Value
- Inspecting structs at runtime
- Dynamic value manipulation
- Struct tags

---

## 🤔 What Is Reflection?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  REFLECTION = Examining types at RUNTIME                        │
│                                                                 │
│  NORMAL CODE (compile time):                                    │
│  ┌─────────────────┐                                           │
│  │ var x int = 10  │  Type known at compile time               │
│  └─────────────────┘                                           │
│                                                                 │
│  REFLECTION (runtime):                                          │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ func process(v interface{}) {                       │       │
│  │     // What type is v? What fields does it have?    │       │
│  │     // We don't know at compile time!               │       │
│  │     t := reflect.TypeOf(v)                          │       │
│  │     fmt.Println(t.Name())  // "User"                │       │
│  │ }                                                   │       │
│  └─────────────────────────────────────────────────────┘       │
│                                                                 │
│  USE CASES:                                                     │
│  • JSON marshaling/unmarshaling                                 │
│  • ORM (database mapping)                                       │
│  • Dependency injection                                         │
│  • Validation frameworks                                        │
│  • Generic programming (pre-Go 1.18)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Type and Value

```go
// reflect_basics.go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           REFLECTION BASICS                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // TypeOf - get type information
    fmt.Println("\n📊 reflect.TypeOf:")
    var x int = 42
    t := reflect.TypeOf(x)
    fmt.Printf("   Type: %v\n", t)
    fmt.Printf("   Name: %s\n", t.Name())
    fmt.Printf("   Kind: %s\n", t.Kind())
    
    // ValueOf - get value information
    fmt.Println("\n📊 reflect.ValueOf:")
    v := reflect.ValueOf(x)
    fmt.Printf("   Value: %v\n", v)
    fmt.Printf("   Type: %v\n", v.Type())
    fmt.Printf("   Kind: %v\n", v.Kind())
    fmt.Printf("   Int: %d\n", v.Int())
    
    // Kind vs Type
    fmt.Println("\n📊 Kind vs Type:")
    type MyInt int
    var y MyInt = 100
    ty := reflect.TypeOf(y)
    fmt.Printf("   Type: %v (custom type name)\n", ty)
    fmt.Printf("   Kind: %v (underlying type)\n", ty.Kind())
    
    // Different types
    fmt.Println("\n📊 Various Types:")
    values := []interface{}{
        42,
        3.14,
        "hello",
        true,
        []int{1, 2, 3},
        map[string]int{"a": 1},
    }
    for _, val := range values {
        t := reflect.TypeOf(val)
        fmt.Printf("   %v: Type=%v, Kind=%v\n", val, t, t.Kind())
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           REFLECTION BASICS                               ║
╚══════════════════════════════════════════════════════════╝

📊 reflect.TypeOf:
   Type: int
   Name: int
   Kind: int

📊 reflect.ValueOf:
   Value: 42
   Type: int
   Kind: int
   Int: 42

📊 Kind vs Type:
   Type: main.MyInt (custom type name)
   Kind: int (underlying type)

📊 Various Types:
   42: Type=int, Kind=int
   3.14: Type=float64, Kind=float64
   hello: Type=string, Kind=string
   true: Type=bool, Kind=bool
   [1 2 3]: Type=[]int, Kind=slice
   map[a:1]: Type=map[string]int, Kind=map
```

---

## 🔍 Inspecting Structs

```go
// reflect_struct.go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    ID       int    `json:"id" db:"user_id"`
    Name     string `json:"name" validate:"required"`
    Email    string `json:"email" validate:"email"`
    password string // unexported
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           INSPECTING STRUCTS                              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    user := User{
        ID:       1,
        Name:     "Alice",
        Email:    "alice@example.com",
        password: "secret",
    }
    
    t := reflect.TypeOf(user)
    v := reflect.ValueOf(user)
    
    fmt.Println("\n📊 Struct Info:")
    fmt.Printf("   Type: %v\n", t)
    fmt.Printf("   NumField: %d\n", t.NumField())
    
    fmt.Println("\n📊 Iterating Fields:")
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        
        fmt.Printf("   Field %d:\n", i)
        fmt.Printf("      Name: %s\n", field.Name)
        fmt.Printf("      Type: %v\n", field.Type)
        fmt.Printf("      Exported: %t\n", field.IsExported())
        
        if value.CanInterface() {
            fmt.Printf("      Value: %v\n", value.Interface())
        } else {
            fmt.Printf("      Value: (unexported)\n")
        }
        
        // Tags
        if tag := field.Tag.Get("json"); tag != "" {
            fmt.Printf("      json tag: %s\n", tag)
        }
        if tag := field.Tag.Get("validate"); tag != "" {
            fmt.Printf("      validate tag: %s\n", tag)
        }
    }
    
    // Get field by name
    fmt.Println("\n📊 Get Field by Name:")
    if field, ok := t.FieldByName("Email"); ok {
        fmt.Printf("   Email field type: %v\n", field.Type)
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           INSPECTING STRUCTS                              ║
╚══════════════════════════════════════════════════════════╝

📊 Struct Info:
   Type: main.User
   NumField: 4

📊 Iterating Fields:
   Field 0:
      Name: ID
      Type: int
      Exported: true
      Value: 1
      json tag: id
   Field 1:
      Name: Name
      Type: string
      Exported: true
      Value: Alice
      json tag: name
      validate tag: required
   Field 2:
      Name: Email
      Type: string
      Exported: true
      Value: alice@example.com
      json tag: email
      validate tag: email
   Field 3:
      Name: password
      Type: string
      Exported: false
      Value: (unexported)

📊 Get Field by Name:
   Email field type: string
```

---

## ✏️ Modifying Values

```go
// reflect_modify.go
package main

import (
    "fmt"
    "reflect"
)

type Config struct {
    Host string
    Port int
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           MODIFYING VALUES                                ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Must use pointer to modify!
    fmt.Println("\n📊 Modifying Simple Value:")
    x := 10
    v := reflect.ValueOf(&x).Elem()  // Must pass pointer!
    
    fmt.Printf("   Before: %d\n", x)
    if v.CanSet() {
        v.SetInt(20)
    }
    fmt.Printf("   After: %d\n", x)
    
    // Modifying struct fields
    fmt.Println("\n📊 Modifying Struct Fields:")
    cfg := Config{Host: "localhost", Port: 8080}
    
    cfgValue := reflect.ValueOf(&cfg).Elem()
    
    fmt.Printf("   Before: %+v\n", cfg)
    
    hostField := cfgValue.FieldByName("Host")
    if hostField.CanSet() {
        hostField.SetString("0.0.0.0")
    }
    
    portField := cfgValue.FieldByName("Port")
    if portField.CanSet() {
        portField.SetInt(3000)
    }
    
    fmt.Printf("   After: %+v\n", cfg)
    
    // CanSet rules
    fmt.Println("\n📊 CanSet Rules:")
    fmt.Println("   • Must reflect on a POINTER")
    fmt.Println("   • Use .Elem() to get addressable value")
    fmt.Println("   • Field must be exported (uppercase)")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           MODIFYING VALUES                                ║
╚══════════════════════════════════════════════════════════╝

📊 Modifying Simple Value:
   Before: 10
   After: 20

📊 Modifying Struct Fields:
   Before: {Host:localhost Port:8080}
   After: {Host:0.0.0.0 Port:3000}

📊 CanSet Rules:
   • Must reflect on a POINTER
   • Use .Elem() to get addressable value
   • Field must be exported (uppercase)
```

---

## ⚠️ When NOT to Use Reflection

```go
/*
REFLECTION DOWNSIDES:

1. SLOW - much slower than direct access
2. NO COMPILE-TIME CHECKS - errors at runtime
3. HARD TO READ - complex, easy to mess up
4. BRITTLE - changes to struct break reflection code

PREFER:
• Generics (Go 1.18+) for type-safe generic code
• Interfaces for polymorphism
• Code generation for performance-critical cases

USE REFLECTION FOR:
• Serialization (JSON, XML, etc.)
• ORM / Database mapping
• Validation frameworks
• Testing utilities
• When you truly don't know types at compile time
*/
```

---

## 🎯 Key Takeaways

1. **reflect.TypeOf** for type information
2. **reflect.ValueOf** for value access
3. **Kind** = underlying type (int, struct, slice)
4. **Type** = actual type name
5. **Use pointer + Elem()** to modify values
6. **Avoid unless necessary** - slow and fragile

---

## ➡️ Next Steps

**Next Topic:** [44 - Embedding Files](./44-embedding.md)

