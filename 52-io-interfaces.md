# 52 - io.Reader and io.Writer Interfaces

> The fundamental interfaces that power all I/O in Go.

---

## 📌 What You'll Learn

- io.Reader and io.Writer interfaces
- Why they're so powerful
- Common implementations
- Composing readers and writers
- The Stringer interface

---

## 🤔 The Power of Simple Interfaces

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  io.Reader and io.Writer = Go's MOST IMPORTANT interfaces       │
│                                                                 │
│  type Reader interface {                                        │
│      Read(p []byte) (n int, err error)                          │
│  }                                                              │
│                                                                 │
│  type Writer interface {                                        │
│      Write(p []byte) (n int, err error)                         │
│  }                                                              │
│                                                                 │
│  WHY SO POWERFUL?                                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Everything is a stream of bytes!                       │   │
│  │                                                         │   │
│  │  Files        → io.Reader, io.Writer                    │   │
│  │  Network      → io.Reader, io.Writer                    │   │
│  │  HTTP Body    → io.Reader                               │   │
│  │  Buffers      → io.Reader, io.Writer                    │   │
│  │  Compression  → io.Reader, io.Writer                    │   │
│  │  Encryption   → io.Reader, io.Writer                    │   │
│  │  Strings      → io.Reader (strings.Reader)              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Write code ONCE, works with ANY source/destination!            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Basic Usage

```go
// io_basics.go
package main

import (
    "bytes"
    "fmt"
    "io"
    "os"
    "strings"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           io.Reader and io.Writer                         ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // io.Reader examples
    fmt.Println("\n📊 io.Reader Examples:")
    
    // 1. strings.Reader - read from string
    strReader := strings.NewReader("Hello from string!")
    
    // 2. bytes.Buffer - read from bytes
    bufReader := bytes.NewBufferString("Hello from buffer!")
    
    // 3. os.Stdin - read from keyboard
    // 4. os.File - read from file
    // 5. http.Response.Body - read from HTTP
    
    // Read into buffer
    buf := make([]byte, 100)
    n, _ := strReader.Read(buf)
    fmt.Printf("   Read %d bytes: %s\n", n, buf[:n])
    
    n, _ = bufReader.Read(buf)
    fmt.Printf("   Read %d bytes: %s\n", n, buf[:n])
    
    // io.Writer examples
    fmt.Println("\n📊 io.Writer Examples:")
    
    // 1. os.Stdout - write to console
    os.Stdout.Write([]byte("   Written to stdout\n"))
    
    // 2. bytes.Buffer - write to memory
    var buffer bytes.Buffer
    buffer.Write([]byte("Hello "))
    buffer.WriteString("World!")
    fmt.Printf("   Buffer: %s\n", buffer.String())
    
    // 3. os.File - write to file
    // 4. http.ResponseWriter - write HTTP response
}
```

---

## 🔄 io.Copy - The Universal Copier

```go
// io_copy.go
package main

import (
    "bytes"
    "fmt"
    "io"
    "os"
    "strings"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           io.Copy - Universal Data Transfer               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Copy from Reader to Writer
    fmt.Println("\n📊 io.Copy:")
    
    // String to stdout
    src := strings.NewReader("Hello, io.Copy!\n")
    io.Copy(os.Stdout, src)
    
    // String to buffer
    src = strings.NewReader("Data to copy")
    var dst bytes.Buffer
    n, _ := io.Copy(&dst, src)
    fmt.Printf("   Copied %d bytes to buffer: %s\n", n, dst.String())
    
    // This is how you'd copy files:
    fmt.Println("\n📊 File Copy Pattern:")
    fmt.Println(`   srcFile, _ := os.Open("source.txt")`)
    fmt.Println(`   dstFile, _ := os.Create("dest.txt")`)
    fmt.Println(`   io.Copy(dstFile, srcFile)`)
    
    // Copy with limit
    fmt.Println("\n📊 io.CopyN (with limit):")
    src = strings.NewReader("Only copy first 5 bytes")
    var limited bytes.Buffer
    io.CopyN(&limited, src, 5)
    fmt.Printf("   Result: %q\n", limited.String())
}
```

---

## 🧩 Composing Readers and Writers

```go
// io_compose.go
package main

import (
    "compress/gzip"
    "encoding/base64"
    "fmt"
    "io"
    "os"
    "strings"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Composing Readers/Writers                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Stack writers: data → gzip → base64 → stdout
    fmt.Println("\n📊 Writer Chain:")
    fmt.Println("   data → gzip compress → base64 encode → stdout")
    
    // Build the chain (bottom to top)
    b64Writer := base64.NewEncoder(base64.StdEncoding, os.Stdout)
    gzWriter := gzip.NewWriter(b64Writer)
    
    // Write data (goes through entire chain!)
    gzWriter.Write([]byte("Hello, compressed and encoded!"))
    gzWriter.Close()
    b64Writer.Close()
    fmt.Println()
    
    // Multi-reader
    fmt.Println("\n📊 io.MultiReader (concatenate readers):")
    r1 := strings.NewReader("First ")
    r2 := strings.NewReader("Second ")
    r3 := strings.NewReader("Third")
    
    multi := io.MultiReader(r1, r2, r3)
    io.Copy(os.Stdout, multi)
    fmt.Println()
    
    // Tee reader (split to two destinations)
    fmt.Println("\n📊 io.TeeReader (read + copy):")
    src := strings.NewReader("Data for two places")
    var buf strings.Builder
    tee := io.TeeReader(src, &buf)
    
    io.Copy(os.Stdout, tee)
    fmt.Println()
    fmt.Printf("   Also captured: %s\n", buf.String())
}
```

---

## 📜 The Stringer Interface

```go
// stringer.go
package main

import "fmt"

// Stringer interface (from fmt package)
// type Stringer interface {
//     String() string
// }

type User struct {
    ID    int
    Name  string
    Email string
}

// Implement Stringer for custom string representation
func (u User) String() string {
    return fmt.Sprintf("User{%d: %s <%s>}", u.ID, u.Name, u.Email)
}

type Status int

const (
    StatusPending Status = iota
    StatusActive
    StatusCompleted
)

// Stringer for enum-like types
func (s Status) String() string {
    switch s {
    case StatusPending:
        return "PENDING"
    case StatusActive:
        return "ACTIVE"
    case StatusCompleted:
        return "COMPLETED"
    default:
        return "UNKNOWN"
    }
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Stringer Interface                              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Without Stringer: {1 Alice alice@example.com}
    // With Stringer: User{1: Alice <alice@example.com>}
    
    user := User{ID: 1, Name: "Alice", Email: "alice@example.com"}
    fmt.Println("\n📊 Custom String():")
    fmt.Printf("   %v\n", user)   // Uses String() method
    fmt.Printf("   %s\n", user)   // Also uses String()
    
    fmt.Println("\n📊 Enum with String():")
    status := StatusActive
    fmt.Printf("   Status: %v\n", status)  // "ACTIVE" not "1"
    
    // Great for logging!
    fmt.Println("\n📊 Useful for Logging:")
    fmt.Printf("   Processing user: %v with status: %v\n", user, status)
}

/*
TIP: Use `stringer` tool to auto-generate String() for enums:

//go:generate stringer -type=Status

Then run: go generate ./...
*/
```

---

## 🎯 Key Takeaways

1. **io.Reader/Writer** = Go's most important interfaces
2. **One method each** - incredibly simple yet powerful
3. **Everything is bytes** - files, network, buffers
4. **io.Copy** moves data between any reader/writer
5. **Composable** - chain readers/writers together
6. **Stringer** for custom string representation
7. **Write to interface** for maximum flexibility

---

## ➡️ Next Steps

**Next Topic:** [53 - strings and bytes Packages](./53-strings-bytes.md)

