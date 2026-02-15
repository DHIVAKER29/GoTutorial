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

**eface (empty interface):**
- `_type` - Type information
- `data` - Pointer to data

**iface (interface with methods):**
- `tab` - Type + method table (itab)
- `data` - Pointer to data

**iface itab:** `inter` (interface type), `_type` (concrete type), `fun` (method pointers)

---

## 📝 Size and Storage

```go
var any interface{}
// Size: 16 bytes (64-bit) - 8 for type + 8 for data

any = 42
// Holds: type descriptor + pointer to 42

var s Stringer
s = MyString("hello")
// Holds: itab + data pointer
```

---

## ⚠️ nil Interface vs nil Pointer

```go
// Case 1: Truly nil interface
var err1 error
fmt.Println(err1 == nil)  // true

// Case 2: Interface holding nil pointer (GOTCHA!)
var myErr *MyError = nil
var err2 error = myErr
fmt.Println(err2 == nil)  // FALSE!

// err2 has: type=*MyError, data=nil
// Interface is nil ONLY when BOTH type AND data are nil

// Common bug:
func getError() error {
    var e *MyError = nil
    return e  // Returns non-nil interface!
}

// Fix:
func getError() error {
    return nil
}
```

---

## 🔄 Type Assertions Internals

```go
var x interface{} = "hello"

if s, ok := x.(string); ok {
    fmt.Println(s)
}
// Type assertion: O(1) - pointer comparison
// itabs are cached globally - fast lookup

switch v := x.(type) {
case string:
    fmt.Println(v)
case int:
    fmt.Println(v)
}
```

---

## ⚡ Performance Implications

```go
// 1. Allocation (boxing): var i interface{} = 42
// 2. Indirect method call: lookup in itab, then call
// 3. Can't inline: interfaces use dynamic dispatch

// When to use interfaces:
// ✅ Abstraction boundaries, dependency injection, testing
// ❌ Hot paths where nanoseconds matter
// ✅ Interfaces at boundaries, concrete internally
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| Object (base class) | interface{} |
| Explicit implements | Implicit satisfaction |
| null == null always | nil interface ≠ interface holding nil pointer |
| vtable per class | itab per (type, interface) pair, cached |
| Boxing always for primitives | Small values may be inline |

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

## ➡️ Next Steps

**Next Topic:** [63 - Common Interview Questions](./63-interview-questions.md)
