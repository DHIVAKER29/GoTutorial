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

**Reflection** = Examining types at runtime.

- **Normal code (compile time):** `var x int = 10` — type known at compile time
- **Reflection (runtime):** `func process(v interface{})` — we don't know the type at compile time; use `reflect.TypeOf(v)` to inspect it

**Use cases:**
- JSON marshaling/unmarshaling
- ORM (database mapping)
- Dependency injection
- Validation frameworks
- Generic programming (pre-Go 1.18)

---

## 📝 Type and Value

### reflect.TypeOf

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x int = 42
    t := reflect.TypeOf(x)
    fmt.Printf("Type: %v\n", t)
    fmt.Printf("Name: %s\n", t.Name())
    fmt.Printf("Kind: %s\n", t.Kind())
}
// Output:
// Type: int
// Name: int
// Kind: int
```

### reflect.ValueOf

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x int = 42
    v := reflect.ValueOf(x)
    fmt.Printf("Value: %v\n", v)
    fmt.Printf("Type: %v\n", v.Type())
    fmt.Printf("Int: %d\n", v.Int())
}
// Output:
// Value: 42
// Type: int
// Int: 42
```

### Kind vs Type

```go
package main

import (
    "fmt"
    "reflect"
)

type MyInt int

func main() {
    var y MyInt = 100
    ty := reflect.TypeOf(y)
    fmt.Printf("Type: %v (custom type name)\n", ty)
    fmt.Printf("Kind: %v (underlying type)\n", ty.Kind())
}
// Output:
// Type: main.MyInt (custom type name)
// Kind: int (underlying type)
```

### Various Types

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    values := []interface{}{42, 3.14, "hello", true, []int{1, 2, 3}, map[string]int{"a": 1}}
    for _, val := range values {
        t := reflect.TypeOf(val)
        fmt.Printf("%v: Type=%v, Kind=%v\n", val, t, t.Kind())
    }
}
// Output:
// 42: Type=int, Kind=int
// 3.14: Type=float64, Kind=float64
// hello: Type=string, Kind=string
// true: Type=bool, Kind=bool
// [1 2 3]: Type=[]int, Kind=slice
// map[a:1]: Type=map[string]int, Kind=map
```

---

## 🔍 Inspecting Structs

### Struct Fields and Tags

```go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    ID       int    `json:"id" db:"user_id"`
    Name     string `json:"name" validate:"required"`
    Email    string `json:"email" validate:"email"`
    password string
}

func main() {
    user := User{ID: 1, Name: "Alice", Email: "alice@example.com", password: "secret"}
    t := reflect.TypeOf(user)
    v := reflect.ValueOf(user)

    fmt.Printf("Type: %v, NumField: %d\n", t, t.NumField())
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        fmt.Printf("  %s: %v, Exported: %t", field.Name, field.Type, field.IsExported())
        if value.CanInterface() {
            fmt.Printf(", Value: %v", value.Interface())
        }
        if tag := field.Tag.Get("json"); tag != "" {
            fmt.Printf(", json:%s", tag)
        }
        fmt.Println()
    }
}
// Output:
// Type: main.User, NumField: 4
//   ID: int, Exported: true, Value: 1, json:id
//   Name: string, Exported: true, Value: Alice, json:name
//   Email: string, Exported: true, Value: alice@example.com, json:email
//   password: string, Exported: false
```

### Get Field by Name

```go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    ID    int
    Name  string
    Email string
}

func main() {
    t := reflect.TypeOf(User{})
    if field, ok := t.FieldByName("Email"); ok {
        fmt.Printf("Email field type: %v\n", field.Type)
    }
}
// Output:
// Email field type: string
```

---

## ✏️ Modifying Values

### Modifying Simple Value

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    x := 10
    v := reflect.ValueOf(&x).Elem()
    fmt.Printf("Before: %d\n", x)
    if v.CanSet() {
        v.SetInt(20)
    }
    fmt.Printf("After: %d\n", x)
}
// Output:
// Before: 10
// After: 20
```

### Modifying Struct Fields

```go
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
    cfg := Config{Host: "localhost", Port: 8080}
    cfgValue := reflect.ValueOf(&cfg).Elem()
    cfgValue.FieldByName("Host").SetString("0.0.0.0")
    cfgValue.FieldByName("Port").SetInt(3000)
    fmt.Printf("After: %+v\n", cfg)
}
// Output:
// After: {Host:0.0.0.0 Port:3000}
```

### CanSet Rules

- Must reflect on a **pointer**
- Use `.Elem()` to get addressable value
- Field must be **exported** (uppercase)

---

## ⚠️ When NOT to Use Reflection

| Prefer | Use Reflection For |
|--------|--------------------|
| Generics (Go 1.18+) for type-safe code | Serialization (JSON, XML) |
| Interfaces for polymorphism | ORM / Database mapping |
| Code generation for performance | Validation frameworks |
| | Testing utilities |
| | When types unknown at compile time |

**Downsides:** Slow, no compile-time checks, hard to read, brittle.

---

## 🎯 Key Takeaways

1. **reflect.TypeOf** for type information
2. **reflect.ValueOf** for value access
3. **Kind** = underlying type (int, struct, slice)
4. **Type** = actual type name
5. **Use pointer + Elem()** to modify values
6. **Avoid unless necessary** — slow and fragile

---

## ➡️ Next Steps

**Next Topic:** [44 - Embedding Files](./44-embedding.md)
