# 03 - Your First Go Program

> Writing, understanding, and running your first Go program step by step.

---

## 📌 What You'll Learn

- How to write a complete Go program
- Understanding every line of a Go program
- How `package`, `import`, and `func main()` work
- How to run and build Go programs

---

## 🚀 The Complete "Hello World"

Create a file called `main.go`:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

Run it:
```bash
go run main.go
```

**Output:**
```
Hello, World!
```

**That's it!** But let's understand every single line.

---

## 🔍 Line-by-Line Breakdown

### Line 1: `package main`

```go
package main
```

#### What is a Package?

A package is a way to organize and group related code. Think of it like departments in a company — each has specific responsibilities and exposes only what others need.

- `fmt` package = Formatting/printing
- `net/http` package = HTTP
- `main` package = The executable entry point

#### Why `main`?

`package main` tells Go: "This package is meant to be an **executable program**, not a library."

**Rule:** Every executable Go program MUST have:
- `package main`
- `func main()`

If you use a different package name (like `package utils`), it becomes a library that others can import, but you cannot run it directly.

#### Java Comparison

| Aspect | Java | Go |
|--------|------|-----|
| Package | `package com.example.myapp;` | `package main` |
| Entry point | Any class with `main` method | Only `package main` with `func main()` |

---

### Line 2: `import "fmt"`

```go
import "fmt"
```

#### What is Import?

Import brings functionality from other packages into yours. The `fmt` package provides:
- `Println`, `Print`, `Printf` — printing to console
- `Sprintf`, `Fprintf` — formatting strings
- `Scan`, `Scanf` — scanning input

#### Import Syntax Variations

```go
// Single import
import "fmt"

// Multiple imports (grouped)
import (
    "fmt"
    "strings"
    "net/http"
)

// Import with alias
import f "fmt"  // Use as f.Println()

// Blank import (side effects only)
import _ "image/png"  // Registers PNG decoder
```

#### Java Comparison

| Aspect | Java | Go |
|--------|------|-----|
| Import | `import java.util.ArrayList;` | `import "fmt"` |
| Access | Import classes | Import packages, access via package name |

---

### Line 3: Blank Line

Go is not whitespace-sensitive like Python, but conventions matter. The organization (package → imports → code) is enforced by `gofmt`.

---

### Line 4-6: `func main()`

```go
func main() {
    fmt.Println("Hello, World!")
}
```

#### Anatomy of a Function

| Part | Meaning |
|------|---------|
| `func` | Function keyword |
| `main` | Function name |
| `()` | Parameters (empty here) |
| `{` | Opening brace (MUST be on same line!) |

#### Why is `main()` Special?

`func main()` is the **entry point** of your program:

1. OS loads the binary
2. Go runtime initializes
3. Go runtime calls `main()`
4. Your code runs
5. `main()` returns → Program exits

**Rules for main():**
- No parameters
- No return value
- Exactly one in package main

#### Java Comparison

| Aspect | Java | Go |
|--------|------|-----|
| Access modifier | `public static` | Not needed |
| Class wrapper | Required | Not needed |
| Command line args | `String[] args` parameter | Use `os.Args` package |
| Ceremony | High | Minimal |

---

### Line 5: `fmt.Println("Hello, World!")`

```go
fmt.Println("Hello, World!")
```

**What Println does:**
1. Takes any number of arguments
2. Converts them to strings
3. Prints them separated by spaces
4. Adds a newline at the end

**Other Print Functions:**

```go
// Print - no newline
fmt.Print("Hello")
fmt.Print("World")
// Output: HelloWorld

// Println - adds newline
fmt.Println("Hello")
fmt.Println("World")
// Output:
// Hello
// World

// Printf - formatted output
fmt.Printf("Hello, %s version %.2f!\n", "Go", 1.22)
// Output: Hello, Go version 1.22!
```

---

## 🏃 Running Your Program

### Method 1: `go run` (Development)

```bash
go run main.go
```

- Compiles to temporary binary
- Runs it
- Deletes the binary
- **Use for:** Quick testing, development
- **Don't use for:** Production

### Method 2: `go build` (Production)

```bash
go build -o myprogram main.go
./myprogram
```

- Compiles to permanent binary
- **Use for:** Production, distribution

### Cross-Compilation

```bash
# Build for Windows from Mac/Linux
GOOS=windows GOARCH=amd64 go build -o myprogram.exe main.go

# Build for Linux from Mac/Windows
GOOS=linux GOARCH=amd64 go build -o myprogram main.go

# Build for Mac from Linux/Windows
GOOS=darwin GOARCH=amd64 go build -o myprogram main.go
```

---

## 📁 Complete Program Structure

### Minimal Program

```go
package main

func main() {
    println("Hello")  // Built-in, no import needed (but limited)
}
```

**Output:**
```
Hello
```

### Standard Program

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

**Output:**
```
Hello, World!
```

### Real-World Program

```go
package main

import (
    "fmt"
    "os"
    "time"
)

const appName = "MyApp"

var startTime time.Time

func init() {
    startTime = time.Now()
    fmt.Println("Initializing", appName)
}

func main() {
    args := os.Args[1:]  // Skip program name
    
    if len(args) > 0 {
        fmt.Println("Hello,", args[0])
    } else {
        fmt.Println("Hello, World!")
    }
    
    fmt.Println("Started at:", startTime.Format(time.RFC3339))
}
```

**Output:**
```
Initializing MyApp
Hello, World!
Started at: 2026-02-10T12:34:56+07:00
```

*Note: When run with arguments (e.g., `go run main.go Alice`), output would be "Hello, Alice". Timestamp will vary.*

---

## 🎯 Key Takeaways

1. **`package main`** = This is an executable program
2. **`import`** = Bring in functionality from other packages
3. **`func main()`** = Entry point, no params, no return
4. **`fmt.Println()`** = Print with newline
5. **`go run`** = Compile and run (development)
6. **`go build`** = Compile to binary (production)

---

## 🆚 Java vs Go: Complete Comparison

| Aspect | Java | Go |
|--------|------|-----|
| Entry point | `public static void main(String[] args)` | `func main()` |
| Class required | Yes | No |
| Compile | `javac HelloWorld.java` | `go build main.go` |
| Run | `java com.example.HelloWorld` | `go run main.go` or `./myprogram` |
| Dependencies | JDK, classpath | Just Go |

---

## ➡️ Next Steps

You've written and run your first Go program! Now let's understand how to organize code into packages and modules.

**Next Topic:** [04 - Packages & Modules](./04-packages-and-modules.md)
