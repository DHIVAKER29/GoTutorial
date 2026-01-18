# 53 - strings and bytes Packages

> Essential string and byte slice manipulation utilities.

---

## 📌 What You'll Learn

- strings package functions
- bytes package functions
- strings.Builder for efficient concatenation
- Common string operations

---

## 📜 strings Package

```go
// strings_package.go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           strings Package                                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    s := "Hello, World!"
    
    // Searching
    fmt.Println("\n📊 Searching:")
    fmt.Printf("   Contains 'World': %t\n", strings.Contains(s, "World"))
    fmt.Printf("   ContainsAny 'aeiou': %t\n", strings.ContainsAny(s, "aeiou"))
    fmt.Printf("   HasPrefix 'Hello': %t\n", strings.HasPrefix(s, "Hello"))
    fmt.Printf("   HasSuffix '!': %t\n", strings.HasSuffix(s, "!"))
    fmt.Printf("   Index 'o': %d\n", strings.Index(s, "o"))
    fmt.Printf("   LastIndex 'o': %d\n", strings.LastIndex(s, "o"))
    fmt.Printf("   Count 'l': %d\n", strings.Count(s, "l"))
    
    // Case conversion
    fmt.Println("\n📊 Case Conversion:")
    fmt.Printf("   ToUpper: %s\n", strings.ToUpper(s))
    fmt.Printf("   ToLower: %s\n", strings.ToLower(s))
    fmt.Printf("   Title: %s\n", strings.ToTitle("hello world"))
    
    // Trimming
    fmt.Println("\n📊 Trimming:")
    padded := "  Hello  "
    fmt.Printf("   TrimSpace: %q\n", strings.TrimSpace(padded))
    fmt.Printf("   Trim: %q\n", strings.Trim("!!!hello!!!", "!"))
    fmt.Printf("   TrimPrefix: %q\n", strings.TrimPrefix("Hello, World", "Hello, "))
    fmt.Printf("   TrimSuffix: %q\n", strings.TrimSuffix("file.txt", ".txt"))
    
    // Splitting
    fmt.Println("\n📊 Splitting:")
    csv := "a,b,c,d"
    fmt.Printf("   Split: %v\n", strings.Split(csv, ","))
    fmt.Printf("   SplitN (limit 2): %v\n", strings.SplitN(csv, ",", 2))
    fmt.Printf("   Fields: %v\n", strings.Fields("  a  b  c  "))
    
    // Joining
    fmt.Println("\n📊 Joining:")
    parts := []string{"a", "b", "c"}
    fmt.Printf("   Join: %s\n", strings.Join(parts, "-"))
    
    // Replacing
    fmt.Println("\n📊 Replacing:")
    fmt.Printf("   Replace: %s\n", strings.Replace("oink oink", "oink", "moo", 1))
    fmt.Printf("   ReplaceAll: %s\n", strings.ReplaceAll("oink oink", "oink", "moo"))
    
    // Repeating
    fmt.Println("\n📊 Repeating:")
    fmt.Printf("   Repeat: %s\n", strings.Repeat("ab", 3))
    
    // Comparison
    fmt.Println("\n📊 Comparison:")
    fmt.Printf("   EqualFold (case-insensitive): %t\n", 
        strings.EqualFold("Hello", "hello"))
}
```

---

## 🔧 strings.Builder (Efficient Concatenation)

```go
// strings_builder.go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           strings.Builder                                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Problem with += 
    fmt.Println("\n📊 Problem with += :")
    fmt.Println("   s += \"x\"  // Creates new string each time!")
    fmt.Println("   O(n²) for n concatenations")
    
    // Solution: strings.Builder
    fmt.Println("\n📊 Solution: strings.Builder")
    var builder strings.Builder
    
    builder.WriteString("Hello")
    builder.WriteString(", ")
    builder.WriteString("World")
    builder.WriteByte('!')
    builder.WriteRune('🚀')
    
    result := builder.String()
    fmt.Printf("   Result: %s\n", result)
    fmt.Printf("   Length: %d\n", builder.Len())
    
    // Pre-allocate for known size
    fmt.Println("\n📊 Pre-allocation (for known size):")
    var b strings.Builder
    b.Grow(100)  // Pre-allocate 100 bytes
    for i := 0; i < 10; i++ {
        fmt.Fprintf(&b, "item%d ", i)
    }
    fmt.Printf("   Result: %s\n", b.String())
    
    // Performance comparison
    fmt.Println("\n📊 Performance:")
    fmt.Println("   += operator:      ~1000ms for 100k concatenations")
    fmt.Println("   strings.Builder:  ~1ms for 100k concatenations")
    fmt.Println("   1000x faster!")
}
```

---

## 📦 bytes Package

```go
// bytes_package.go
package main

import (
    "bytes"
    "fmt"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           bytes Package                                   ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // bytes has same functions as strings, but for []byte
    b := []byte("Hello, World!")
    
    fmt.Println("\n📊 bytes Functions (same as strings):")
    fmt.Printf("   Contains: %t\n", bytes.Contains(b, []byte("World")))
    fmt.Printf("   ToUpper: %s\n", bytes.ToUpper(b))
    fmt.Printf("   Split: %v\n", bytes.Split(b, []byte(", ")))
    
    // bytes.Buffer (io.Reader AND io.Writer)
    fmt.Println("\n📊 bytes.Buffer:")
    var buf bytes.Buffer
    
    // Write to buffer
    buf.WriteString("Hello ")
    buf.Write([]byte("World"))
    buf.WriteByte('!')
    
    fmt.Printf("   Contents: %s\n", buf.String())
    fmt.Printf("   Bytes: %v\n", buf.Bytes())
    fmt.Printf("   Len: %d\n", buf.Len())
    
    // Read from buffer
    fmt.Println("\n📊 Reading from Buffer:")
    data := make([]byte, 5)
    n, _ := buf.Read(data)
    fmt.Printf("   Read %d bytes: %s\n", n, data)
    fmt.Printf("   Remaining: %s\n", buf.String())
    
    // Comparison
    fmt.Println("\n📊 bytes.Compare:")
    a := []byte("abc")
    b1 := []byte("abc")
    b2 := []byte("abd")
    
    fmt.Printf("   Equal(abc, abc): %t\n", bytes.Equal(a, b1))
    fmt.Printf("   Compare(abc, abd): %d\n", bytes.Compare(a, b2)) // -1
}
```

---

## 🔄 strings.Reader

```go
// strings_reader.go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           strings.Reader                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Create reader from string
    r := strings.NewReader("Hello, World!")
    
    fmt.Println("\n📊 strings.Reader:")
    fmt.Printf("   Size: %d\n", r.Size())
    fmt.Printf("   Len (remaining): %d\n", r.Len())
    
    // Read bytes
    buf := make([]byte, 5)
    n, _ := r.Read(buf)
    fmt.Printf("   Read %d bytes: %s\n", n, buf)
    fmt.Printf("   Remaining: %d\n", r.Len())
    
    // Seek
    r.Seek(0, io.SeekStart)  // Back to beginning
    fmt.Printf("   After Seek(0): Len=%d\n", r.Len())
    
    // ReadAll
    r.Seek(0, io.SeekStart)
    all, _ := io.ReadAll(r)
    fmt.Printf("   ReadAll: %s\n", all)
}
```

---

## 🎯 Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│  STRINGS/BYTES QUICK REFERENCE                                  │
├─────────────────────────────────────────────────────────────────┤
│  SEARCHING                                                      │
│    Contains, ContainsAny, HasPrefix, HasSuffix                  │
│    Index, LastIndex, Count                                      │
│                                                                 │
│  TRANSFORMING                                                   │
│    ToUpper, ToLower, ToTitle                                    │
│    TrimSpace, Trim, TrimPrefix, TrimSuffix                      │
│    Replace, ReplaceAll                                          │
│                                                                 │
│  SPLITTING/JOINING                                              │
│    Split, SplitN, Fields                                        │
│    Join                                                         │
│                                                                 │
│  BUILDERS/BUFFERS                                               │
│    strings.Builder - efficient string building                  │
│    bytes.Buffer    - read/write byte buffer                     │
│    strings.Reader  - read from string                           │
│                                                                 │
│  TIP: bytes package mirrors strings for []byte                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## ➡️ Next Steps

**Next Topic:** [54 - sort Package](./54-sorting.md)

