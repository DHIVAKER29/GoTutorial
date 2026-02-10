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

Output:
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

```
┌─────────────────────────────────────────────────────────────────┐
│                    WHAT IS A PACKAGE?                           │
│                                                                 │
│  A package is a way to organize and group related code.         │
│                                                                 │
│  Real-World Analogy: Departments in a Company                   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    COMPANY                              │   │
│  │                                                         │   │
│  │   ┌──────────┐  ┌──────────┐  ┌──────────┐             │   │
│  │   │ Finance  │  │   HR     │  │Engineering│            │   │
│  │   │Department│  │Department│  │Department │            │   │
│  │   └──────────┘  └──────────┘  └──────────┘             │   │
│  │                                                         │   │
│  │   Each department:                                      │   │
│  │   • Has specific responsibilities                       │   │
│  │   • Exposes only what others need                       │   │
│  │   • Hides internal workings                             │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  In Go:                                                         │
│  • "fmt" package = Formatting/printing department               │
│  • "net/http" package = HTTP department                         │
│  • "main" package = The executable entry point                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Why `main`?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  package main   ← This is SPECIAL!                              │
│                                                                 │
│  It tells Go:                                                   │
│  "This package is meant to be an EXECUTABLE program,            │
│   not a library to be imported by other code."                  │
│                                                                 │
│  RULE: Every executable Go program MUST have:                   │
│        • package main                                           │
│        • func main()                                            │
│                                                                 │
│  If you use a different package name (like "package utils"),    │
│  it becomes a LIBRARY that others can import, but you           │
│  cannot run it directly.                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Java Comparison

```java
// Java - Package declares namespace
package com.example.myapp;

public class Main {
    public static void main(String[] args) {
        // Entry point determined by main method in any class
    }
}
```

```go
// Go - package main is SPECIAL
package main  // This IS the executable marker

func main() {
    // Entry point
}
```

**Key Difference:**
- Java: Any class with `main` method can be an entry point
- Go: Only `package main` with `func main()` can be an entry point

---

### Line 2: `import "fmt"`

```go
import "fmt"
```

#### What is Import?

```
┌─────────────────────────────────────────────────────────────────┐
│                    WHAT IS IMPORT?                              │
│                                                                 │
│  Import brings functionality from other packages into yours.    │
│                                                                 │
│  Real-World Analogy: Ordering Supplies                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  Your Office (package main)                             │   │
│  │  ┌────────────────────────────────────────┐             │   │
│  │  │                                        │             │   │
│  │  │  "I need printing supplies!"           │             │   │
│  │  │        │                               │             │   │
│  │  │        ▼                               │             │   │
│  │  │  import "fmt" ← Order from fmt dept    │             │   │
│  │  │                                        │             │   │
│  │  └────────────────────────────────────────┘             │   │
│  │         │                                               │   │
│  │         ▼                                               │   │
│  │  ┌────────────────────────────────────────┐             │   │
│  │  │  fmt package delivers:                 │             │   │
│  │  │  • Println() function                  │             │   │
│  │  │  • Printf() function                   │             │   │
│  │  │  • Sprintf() function                  │             │   │
│  │  │  • ... and more                        │             │   │
│  │  └────────────────────────────────────────┘             │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### What is `fmt`?

`fmt` stands for "format" and is Go's standard package for:
- Printing to console (`Println`, `Print`, `Printf`)
- Formatting strings (`Sprintf`, `Fprintf`)
- Scanning input (`Scan`, `Scanf`)

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
import (
    f "fmt"              // Use as f.Println()
    str "strings"        // Use as str.ToUpper()
)

// Blank import (side effects only)
import (
    _ "image/png"        // Registers PNG decoder
)
```

#### Java Comparison

```java
// Java import
import java.util.ArrayList;
import java.util.HashMap;
import static java.lang.System.out;  // Static import

// Usage
ArrayList<String> list = new ArrayList<>();
out.println("Hello");
```

```go
// Go import
import (
    "fmt"
    "container/list"
)

// Usage
fmt.Println("Hello")
l := list.New()
```

**Key Differences:**
- Java: Imports classes from packages
- Go: Imports entire packages, access via package name

---

### Line 3: Blank Line

```go
// (blank line here)
```

Go is not whitespace-sensitive like Python, but conventions matter:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO CODE ORGANIZATION                                           │
│                                                                 │
│  package main        ← Package declaration                      │
│                      ← Blank line                               │
│  import "fmt"        ← Imports                                  │
│                      ← Blank line                               │
│  func main() {       ← Code                                     │
│                                                                 │
│  This organization is ENFORCED by gofmt (Go formatter)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Line 4-6: `func main()`

```go
func main() {
    fmt.Println("Hello, World!")
}
```

#### What is a Function?

```
┌─────────────────────────────────────────────────────────────────┐
│                    ANATOMY OF A FUNCTION                        │
│                                                                 │
│  func main() {                                                  │
│   │    │   │ │                                                  │
│   │    │   │ └── Opening brace (MUST be on same line!)          │
│   │    │   └──── Parameters (empty in this case)                │
│   │    └──────── Function name                                  │
│   └───────────── func keyword                                   │
│                                                                 │
│      fmt.Println("Hello, World!")                               │
│       │    │           │                                        │
│       │    │           └── Argument (string to print)           │
│       │    └────────────── Function name from fmt package       │
│       └─────────────────── Package name                         │
│                                                                 │
│  }                                                              │
│  └── Closing brace                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Why is `main()` Special?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  func main() is the ENTRY POINT of your program.                │
│                                                                 │
│  When you run your program:                                     │
│                                                                 │
│  1. Operating System loads the binary                           │
│  2. Go runtime initializes                                      │
│  3. Go runtime calls main()                                     │
│  4. Your code runs                                              │
│  5. main() returns → Program exits                              │
│                                                                 │
│  RULES for main():                                              │
│  ✅ No parameters                                               │
│  ✅ No return value                                             │
│  ✅ Exactly one in package main                                 │
│                                                                 │
│  ❌ func main(args []string)      ← WRONG!                      │
│  ❌ func main() int               ← WRONG!                      │
│  ❌ func Main()                   ← WRONG! (capital M)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Java Comparison

```java
// Java entry point
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

```go
// Go entry point
package main

func main() {
    fmt.Println("Hello, World!")
}
```

| Aspect | Java | Go |
|--------|------|-----|
| Access modifier | `public static` | Not needed |
| Class wrapper | Required | Not needed |
| Command line args | `String[] args` parameter | Use `os.Args` package |
| Return type | `void` | Implicit (none) |
| Ceremony | High | Minimal |

---

### Line 5: `fmt.Println("Hello, World!")`

```go
fmt.Println("Hello, World!")
```

#### Breaking It Down

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  fmt.Println("Hello, World!")                                   │
│   │    │       │           │                                    │
│   │    │       │           └── ) closing parenthesis            │
│   │    │       └────────────── String literal (UTF-8!)          │
│   │    └────────────────────── Println = Print + Line           │
│   └─────────────────────────── fmt package                      │
│                                                                 │
│  What Println does:                                             │
│  1. Takes any number of arguments                               │
│  2. Converts them to strings                                    │
│  3. Prints them separated by spaces                             │
│  4. Adds a newline at the end                                   │
│                                                                 │
│  Examples:                                                      │
│  fmt.Println("Hello")           → Hello\n                       │
│  fmt.Println("Hello", "World")  → Hello World\n                 │
│  fmt.Println(1, 2, 3)           → 1 2 3\n                       │
│  fmt.Println()                  → \n (just newline)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Other Print Functions

```go
// Print - no newline at end
fmt.Print("Hello")
fmt.Print("World")
// Output: HelloWorld

// Println - adds newline
fmt.Println("Hello")
fmt.Println("World")
// Output:
// Hello
// World

// Printf - formatted output (like C)
name := "Go"
version := 1.22
fmt.Printf("Hello, %s version %.2f!\n", name, version)
// Output: Hello, Go version 1.22!
```

---

## 🏃 Running Your Program

### Method 1: `go run` (Development)

```bash
go run main.go
```

What happens:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  go run main.go                                                 │
│                                                                 │
│  1. Compiles main.go to temporary binary                        │
│  2. Runs the temporary binary                                   │
│  3. Deletes the temporary binary                                │
│                                                                 │
│  Use for: Quick testing, development                            │
│  Don't use for: Production                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Method 2: `go build` (Production)

```bash
go build -o myprogram main.go
./myprogram
```

What happens:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  go build -o myprogram main.go                                  │
│                                                                 │
│  1. Compiles main.go to binary named "myprogram"                │
│  2. Binary is kept on disk                                      │
│  3. You can run it anytime: ./myprogram                         │
│  4. You can distribute it to others                             │
│                                                                 │
│  Use for: Production, distribution                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Cross-Compilation (Build for Other OS)

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

// Constants
const appName = "MyApp"

// Package-level variables
var startTime time.Time

// init runs before main
func init() {
    startTime = time.Now()
    fmt.Println("Initializing", appName)
}

// main is the entry point
func main() {
    // Get command line arguments
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

*Note: When run with arguments (e.g., `go run main.go Alice`), output would be "Hello, Alice" instead. Timestamp will vary.*

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

```java
// Java: HelloWorld.java
package com.example;

import java.time.LocalDateTime;

public class HelloWorld {
    private static final String APP_NAME = "MyApp";
    
    public static void main(String[] args) {
        System.out.println("Hello, World!");
        System.out.println("Time: " + LocalDateTime.now());
    }
}

// Compile: javac HelloWorld.java
// Run: java com.example.HelloWorld
// Requires: JDK installed, classpath set
```

```go
// Go: main.go
package main

import (
    "fmt"
    "time"
)

const appName = "MyApp"

func main() {
    fmt.Println("Hello, World!")
    fmt.Println("Time:", time.Now())
}

// Run: go run main.go
// Build: go build -o hello main.go
// Requires: Just Go
```

---

## ➡️ Next Steps

You've written and run your first Go program! Now let's understand how to organize code into packages and modules.

**Next Topic:** [04 - Packages & Modules](./04-packages-and-modules.md)

