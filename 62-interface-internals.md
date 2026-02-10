# 62 - Interface Internals

> How Go interfaces work under the hood.

---

## 📌 What You'll Learn

- Interface internal representation
- iface vs eface
- nil interface vs nil pointer
- Type assertions internals
- Performance implications

---

## 🔍 Interface Internal Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  TWO TYPES OF INTERFACES                                        │
│                                                                 │
│  1. eface (empty interface - interface{})                       │
│  ┌─────────────────────────────────────────────────┐           │
│  │  type eface struct {                            │           │
│  │      _type *_type  // Type information          │           │
│  │      data  unsafe.Pointer  // Pointer to data   │           │
│  │  }                                              │           │
│  └─────────────────────────────────────────────────┘           │
│                                                                 │
│  2. iface (interface with methods)                              │
│  ┌─────────────────────────────────────────────────┐           │
│  │  type iface struct {                            │           │
│  │      tab  *itab  // Type + method table         │           │
│  │      data unsafe.Pointer  // Pointer to data    │           │
│  │  }                                              │           │
│  │                                                 │           │
│  │  type itab struct {                             │           │
│  │      inter *interfacetype  // Interface type    │           │
│  │      _type *_type          // Concrete type     │           │
│  │      fun   [1]uintptr      // Method pointers   │           │
│  │  }                                              │           │
│  └─────────────────────────────────────────────────┘           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Visualizing Interfaces

```go
// interface_visual.go
package main

import (
    "fmt"
    "unsafe"
)

type Stringer interface {
    String() string
}

type MyString string

func (s MyString) String() string {
    return string(s)
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Interface Internals                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Empty interface (eface)
    fmt.Println("\n📊 Empty Interface (eface):")
    var any interface{}
    fmt.Printf("   Size of interface{}: %d bytes\n", unsafe.Sizeof(any))
    // 16 bytes on 64-bit: 8 for type + 8 for data pointer
    
    any = 42
    fmt.Printf("   interface{} holding int: type + data pointer\n")
    
    // Interface with methods (iface)
    fmt.Println("\n📊 Interface with Methods (iface):")
    var s Stringer
    fmt.Printf("   Size of Stringer: %d bytes\n", unsafe.Sizeof(s))
    
    s = MyString("hello")
    fmt.Printf("   Stringer holding MyString: itab + data pointer\n")
    
    // What's stored
    fmt.Println("\n📊 What's Stored:")
    fmt.Println("   ┌──────────────┐")
    fmt.Println("   │ interface{}  │")
    fmt.Println("   ├──────────────┤")
    fmt.Println("   │ type: *int   │──► type descriptor")
    fmt.Println("   │ data: 0x...  │──► actual value (42)")
    fmt.Println("   └──────────────┘")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Interface Internals                             ║
╚══════════════════════════════════════════════════════════╝

📊 Empty Interface (eface):
   Size of interface{}: 16 bytes
   interface{} holding int: type + data pointer

📊 Interface with Methods (iface):
   Size of Stringer: 16 bytes
   Stringer holding MyString: itab + data pointer

📊 What's Stored:
   ┌──────────────┐
   │ interface{}  │
   ├──────────────┤
   │ type: *int   │──► type descriptor
   │ data: 0x...  │──► actual value (42)
   └──────────────┘
```

---

## ⚠️ nil Interface vs nil Pointer

```go
// nil_interface.go
package main

import "fmt"

type MyError struct {
    msg string
}

func (e *MyError) Error() string {
    return e.msg
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           nil Interface vs nil Pointer                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Case 1: Truly nil interface
    fmt.Println("\n📊 Case 1: Truly nil Interface")
    var err1 error  // nil interface (type=nil, data=nil)
    fmt.Printf("   err1 == nil: %t\n", err1 == nil)  // true!
    
    // Case 2: Interface holding nil pointer
    fmt.Println("\n📊 Case 2: Interface with nil Pointer (GOTCHA!)")
    var myErr *MyError = nil  // nil pointer
    var err2 error = myErr    // interface holding nil pointer
    // err2: type=*MyError, data=nil
    
    fmt.Printf("   myErr == nil: %t\n", myErr == nil)  // true
    fmt.Printf("   err2 == nil:  %t\n", err2 == nil)   // FALSE! 😱
    
    // Why?
    fmt.Println("\n📊 Why the Difference?")
    fmt.Println("   err1: { type: nil,      data: nil } → nil interface")
    fmt.Println("   err2: { type: *MyError, data: nil } → NOT nil interface!")
    fmt.Println("   Interface is nil ONLY when both type AND data are nil")
    
    // Common mistake
    fmt.Println("\n📊 Common Mistake:")
    fmt.Println("   func getError() error {")
    fmt.Println("       var err *MyError = nil")
    fmt.Println("       return err  // Returns non-nil interface!")
    fmt.Println("   }")
    fmt.Println("")
    fmt.Println("   ✅ Correct way:")
    fmt.Println("   func getError() error {")
    fmt.Println("       return nil  // Returns nil interface")
    fmt.Println("   }")
    
    // How to check properly
    fmt.Println("\n📊 Safe Nil Check:")
    if err2 != nil {
        // Check if underlying value is actually nil
        if v, ok := err2.(*MyError); ok && v == nil {
            fmt.Println("   Interface holds nil pointer!")
        }
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           nil Interface vs nil Pointer                    ║
╚══════════════════════════════════════════════════════════╝

📊 Case 1: Truly nil Interface
   err1 == nil: true

📊 Case 2: Interface with nil Pointer (GOTCHA!)
   myErr == nil: true
   err2 == nil:  false

📊 Why the Difference?
   err1: { type: nil,      data: nil } → nil interface
   err2: { type: *MyError, data: nil } → NOT nil interface!
   Interface is nil ONLY when both type AND data are nil

📊 Common Mistake:
   func getError() error {
       var err *MyError = nil
       return err  // Returns non-nil interface!
   }

   ✅ Correct way:
   func getError() error {
       return nil  // Returns nil interface
   }

📊 Safe Nil Check:
   Interface holds nil pointer!
```

---

## 🔄 Type Assertions Internals

```go
// type_assertions.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Type Assertions Internals                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    var x interface{} = "hello"
    
    // Type assertion: x.(T)
    // Compiler checks if x's type == T
    // If eface: compare _type pointer
    // If iface: compare itab pointer (cached!)
    
    fmt.Println("\n📊 Type Assertion (fast):")
    if s, ok := x.(string); ok {
        fmt.Printf("   x is string: %s\n", s)
    }
    
    // Type switch: uses cached itabs
    fmt.Println("\n📊 Type Switch:")
    switch v := x.(type) {
    case string:
        fmt.Printf("   string: %s\n", v)
    case int:
        fmt.Printf("   int: %d\n", v)
    }
    
    // Performance note
    fmt.Println("\n📊 Performance:")
    fmt.Println("   Type assertion: O(1) - just pointer comparison")
    fmt.Println("   Type switch: O(n) in worst case, but optimized")
    fmt.Println("   itabs are cached globally - fast lookup")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Type Assertions Internals                       ║
╚══════════════════════════════════════════════════════════╝

📊 Type Assertion (fast):
   x is string: hello

📊 Type Switch:
   string: hello

📊 Performance:
   Type assertion: O(1) - just pointer comparison
   Type switch: O(n) in worst case, but optimized
   itabs are cached globally - fast lookup
```

---

## ⚡ Performance Implications

```go
// interface_performance.go
package main

import (
    "fmt"
)

type Adder interface {
    Add(int) int
}

type IntAdder int

func (a IntAdder) Add(x int) int {
    return int(a) + x
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Interface Performance                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 Performance Costs:")
    
    fmt.Println("\n   1. Allocation (Boxing):")
    fmt.Println("      var i interface{} = 42  // May allocate")
    fmt.Println("      Small values might be optimized (stored inline)")
    
    fmt.Println("\n   2. Indirect Method Call:")
    fmt.Println("      Concrete: direct function call")
    fmt.Println("      Interface: lookup method in itab, then call")
    fmt.Println("      ~few nanoseconds overhead")
    
    fmt.Println("\n   3. Can't Inline:")
    fmt.Println("      Concrete types: compiler can inline methods")
    fmt.Println("      Interfaces: can't inline (dynamic dispatch)")
    
    fmt.Println("\n📊 When to Use Interfaces:")
    fmt.Println("   ✅ Abstraction boundaries (package APIs)")
    fmt.Println("   ✅ Dependency injection")
    fmt.Println("   ✅ Testing (mock implementations)")
    fmt.Println("   ❌ Hot paths where nanoseconds matter")
    fmt.Println("   ❌ Simple value passing")
    
    fmt.Println("\n📊 Optimization Tips:")
    fmt.Println("   • Use concrete types in tight loops")
    fmt.Println("   • Profile before optimizing")
    fmt.Println("   • Interfaces at boundaries, concrete internally")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Interface Performance                           ║
╚══════════════════════════════════════════════════════════╝

📊 Performance Costs:

   1. Allocation (Boxing):
      var i interface{} = 42  // May allocate
      Small values might be optimized (stored inline)

   2. Indirect Method Call:
      Concrete: direct function call
      Interface: lookup method in itab, then call
      ~few nanoseconds overhead

   3. Can't Inline:
      Concrete types: compiler can inline methods
      Interfaces: can't inline (dynamic dispatch)

📊 When to Use Interfaces:
   ✅ Abstraction boundaries (package APIs)
   ✅ Dependency injection
   ✅ Testing (mock implementations)
   ❌ Hot paths where nanoseconds matter
   ❌ Simple value passing

📊 Optimization Tips:
   • Use concrete types in tight loops
   • Profile before optimizing
   • Interfaces at boundaries, concrete internally
```

---

## 🎯 Key Takeaways

1. **eface** = empty interface (type + data pointer)
2. **iface** = interface with methods (itab + data pointer)
3. **nil interface** ≠ interface holding nil pointer!
4. **Type assertions** are O(1) pointer comparisons
5. **itabs are cached** globally for performance
6. **Interfaces have overhead** - use concrete types in hot paths
7. **Boxing** may cause allocation

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│                                                                 │
│  Object (base class)               interface{} (empty interface)│
│                                                                 │
│  Explicit implements               Implicit satisfaction        │
│                                                                 │
│  Interface reference can be null   nil interface vs nil pointer │
│  (null == null always)             (interface{nil ptr} != nil)  │
│                                                                 │
│  vtable per class                  itab per (type, interface)   │
│                                    pair, cached globally        │
│                                                                 │
│  Boxing always for primitives      Small values may be inline   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ➡️ Next Steps

**Next Topic:** [63 - Common Interview Questions](./63-interview-questions.md)

