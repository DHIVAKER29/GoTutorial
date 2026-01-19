# 04 - Packages & Modules

> Understanding how Go organizes code and manages dependencies.

---

## 📌 What You'll Learn

- What is a Package and why it matters
- What is a Module and how it differs from a Package
- How `go.mod` and `go.sum` work
- How to create and use packages
- Real production patterns

---

## 🤔 The Problem: Organizing Code

### Without Organization

Imagine writing a large program in a single file:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  main.go (10,000 lines!)                                        │
│  ─────────────────────────                                      │
│  • Database connection code                                     │
│  • User authentication code                                     │
│  • Payment processing code                                      │
│  • Email sending code                                           │
│  • Logging code                                                 │
│  • Utility functions                                            │
│  • HTTP handlers                                                │
│  • ... and more                                                 │
│                                                                 │
│  PROBLEMS:                                                      │
│  ❌ Impossible to navigate                                      │
│  ❌ Can't work on same file with team                           │
│  ❌ Can't reuse code in other projects                          │
│  ❌ Can't test individual pieces                                │
│  ❌ Naming conflicts everywhere                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### With Packages

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  myproject/                                                     │
│  ├── go.mod                 ← Module definition                 │
│  ├── main.go                ← Entry point (package main)        │
│  ├── database/              ← package database                  │
│  │   ├── connection.go                                          │
│  │   └── queries.go                                             │
│  ├── auth/                  ← package auth                      │
│  │   ├── login.go                                               │
│  │   └── token.go                                               │
│  ├── payment/               ← package payment                   │
│  │   ├── process.go                                             │
│  │   └── refund.go                                              │
│  └── utils/                 ← package utils                     │
│      └── helpers.go                                             │
│                                                                 │
│  BENEFITS:                                                      │
│  ✅ Clear organization                                          │
│  ✅ Team can work in parallel                                   │
│  ✅ Code is reusable                                            │
│  ✅ Easy to test pieces                                         │
│  ✅ Naming scoped to package                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📦 What is a Package?

### Definition

> A **Package** is a collection of source files in the same directory that are compiled together. It's Go's basic unit of code organization.

### Real-World Analogy: Departments in a Company

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  COMPANY (Project)                                              │
│  ═════════════════                                              │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   Finance   │  │   HR        │  │  Engineering│              │
│  │  Department │  │  Department │  │  Department │              │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤              │
│  │ • Payroll   │  │ • Hiring    │  │ • Backend   │              │
│  │ • Budget    │  │ • Reviews   │  │ • Frontend  │              │
│  │ • Taxes     │  │ • Benefits  │  │ • Database  │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                 │
│  In Go:                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   finance   │  │     auth    │  │   handler   │              │
│  │   package   │  │   package   │  │   package   │              │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤              │
│  │ payroll.go  │  │  login.go   │  │  user.go    │              │
│  │ budget.go   │  │  token.go   │  │  order.go   │              │
│  │ taxes.go    │  │  verify.go  │  │  payment.go │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                 │
│  Each package:                                                  │
│  • Has a specific responsibility                                │
│  • Exposes only what others need (exported = Capital letter)    │
│  • Hides internal details (unexported = lowercase)              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Package Rules

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  PACKAGE RULES                                                  │
│                                                                 │
│  1. All files in a directory must have same package name        │
│                                                                 │
│     myproject/utils/                                            │
│       ├── strings.go   ← package utils                          │
│       ├── numbers.go   ← package utils                          │
│       └── dates.go     ← package utils                          │
│                                                                 │
│     ❌ WRONG: mixing package names in same directory            │
│       ├── strings.go   ← package utils                          │
│       └── numbers.go   ← package helpers   ← ERROR!             │
│                                                                 │
│  2. Package name = directory name (convention, not required)    │
│                                                                 │
│     myproject/utils/    ← directory "utils"                     │
│       └── helpers.go    ← package utils  ✅ matches             │
│                                                                 │
│  3. Exception: package main can be in any directory name        │
│                                                                 │
│     myproject/cmd/api/  ← directory "api"                       │
│       └── main.go       ← package main  ✅ OK (executable)      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📁 What is a Module?

### Definition

> A **Module** is a collection of packages with a `go.mod` file at the root. It defines a project's dependencies and identity.

### Module vs Package

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MODULE vs PACKAGE                                              │
│                                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  MODULE (go.mod)                                           │ │
│  │  ════════════════                                          │ │
│  │  The whole project                                         │ │
│  │  Like a book                                               │ │
│  │                                                            │ │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐              │ │
│  │  │  Package   │ │  Package   │ │  Package   │              │ │
│  │  │  ════════  │ │  ════════  │ │  ════════  │              │ │ 
│  │  │  One       │ │  One       │ │  One       │              │ │
│  │  │  chapter   │ │  chapter   │ │  chapter   │              │ │
│  │  └────────────┘ └────────────┘ └────────────┘              │ │
│  │                                                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  Real Example:                                                  │
│                                                                 │
│  github.com/razorpay/catalyst   ← MODULE (the project)          │
│    ├── internal/mgst            ← PACKAGE (one feature)         │
│    ├── internal/boot            ← PACKAGE (initialization)      │
│    ├── pkg/utils                ← PACKAGE (utilities)           │
│    └── cmd/api                  ← PACKAGE (main entry point)    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📄 The go.mod File

### What is go.mod?

The `go.mod` file is the heart of a Go module. It declares:
- The module's name (import path)
- The Go version
- Dependencies

### Structure

```go
// go.mod file
module github.com/razorpay/catalyst

go 1.23

require (
    github.com/gorilla/mux v1.8.0
    github.com/lib/pq v1.10.9
    google.golang.org/grpc v1.58.3
)

require (
    // indirect dependencies (pulled in by direct ones)
    golang.org/x/net v0.17.0 // indirect
    golang.org/x/text v0.13.0 // indirect
)
```

### Line-by-Line Breakdown

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  module github.com/razorpay/catalyst                            │
│  ─────────────────────────────────────                          │
│  • MODULE PATH: unique identifier for this project              │
│  • Used when importing packages from this module                │
│  • Usually a URL where code is hosted (but doesn't have to be)  │
│                                                                 │
│  go 1.23                                                        │
│  ─────────                                                      │
│  • Minimum Go version required                                  │
│  • Enables version-specific features                            │
│                                                                 │
│  require (                                                      │
│      github.com/gorilla/mux v1.8.0                              │
│  )                                                              │
│  ───────────────────────────────────                            │
│  • DIRECT DEPENDENCIES: packages your code directly imports     │
│  • Each has a specific version (v1.8.0)                         │
│  • Versions use semantic versioning (vMAJOR.MINOR.PATCH)        │
│                                                                 │
│  require (                                                      │
│      golang.org/x/net v0.17.0 // indirect                       │
│  )                                                              │
│  ────────────────────────────────────────                       │
│  • INDIRECT DEPENDENCIES: dependencies of your dependencies     │
│  • You don't import these, but they're needed                   │
│  • Marked with "// indirect" comment                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Creating a go.mod

```bash
# Create a new module
go mod init github.com/username/myproject

# This creates go.mod:
# module github.com/username/myproject
# 
# go 1.22

# Add dependencies (automatically updates go.mod)
go get github.com/gorilla/mux@v1.8.0

# Remove unused dependencies
go mod tidy

# Download all dependencies
go mod download
```

---

## 📄 The go.sum File

### What is go.sum?

The `go.sum` file is a security feature. It contains checksums (cryptographic hashes) of all dependencies.

### Why We Need It

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  THE PROBLEM: Supply Chain Attacks                              │
│                                                                 │
│  Scenario:                                                      │
│  1. You depend on "awesome-lib v1.0.0"                          │
│  2. Attacker hacks the library's repository                     │
│  3. Attacker modifies v1.0.0 to include malware                 │
│  4. You download "v1.0.0" (now with malware)                    │
│  5. Your app is compromised!                                    │
│                                                                 │
│  THE SOLUTION: Checksums                                        │
│                                                                 │
│  1. First time you download "awesome-lib v1.0.0"                │
│  2. Go calculates a checksum (hash) of the content              │
│  3. Checksum is saved in go.sum                                 │
│  4. Next download, Go checks: does content match checksum?      │
│  5. If not → ERROR! Someone tampered with the code!             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Structure

```
// go.sum file
github.com/gorilla/mux v1.8.0 h1:i40aqfkR1h2SlN9hojwV5ZA91wcXFOvkdNIeFDP5koI=
github.com/gorilla/mux v1.8.0/go.mod h1:DVbg23sWSpFRCP0SfiEN6jmj59UnW/n46BH5rLB71So=
```

### Line Breakdown

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  github.com/gorilla/mux v1.8.0 h1:i40aqf...koI=                 │
│  ─────────────────────────────────────────────────              │
│  │                    │      │                                  │
│  │                    │      └── h1: = hash of the MODULE ZIP   │
│  │                    └── Version                               │
│  └── Module path                                                │
│                                                                 │
│  github.com/gorilla/mux v1.8.0/go.mod h1:DVbg23...71So=         │
│  ───────────────────────────────────────────────────────        │
│  │                              │                               │
│  │                              └── /go.mod = hash of go.mod    │
│  └── Module path + version                file only             │
│                                                                 │
│  Two hashes per dependency:                                     │
│  1. Hash of entire module content                               │
│  2. Hash of just the go.mod file                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CHECKSUM = SEAL ON A MEDICINE BOTTLE                           │
│                                                                 │
│  When you buy medicine:                                         │
│  ┌─────────────────┐                                            │
│  │  💊 Aspirin     │  ← Sealed wrapper                          │
│  │  ──────────────  │                                           │
│  │  If seal broken │  → Don't use! May be tampered!             │
│  │  If seal intact │  → Safe to use!                            │
│  └─────────────────┘                                            │
│                                                                 │
│  In Go:                                                         │
│  ┌─────────────────┐                                            │
│  │  📦 gorilla/mux │  ← Checksum in go.sum                      │
│  │  ──────────────  │                                           │
│  │  Checksum match │  → Safe! Use it!                           │
│  │  Checksum fail  │  → ERROR! Don't use!                       │
│  └─────────────────┘                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🆚 Java Comparison: Dependency Management

### Java (Maven)

```xml
<!-- pom.xml -->
<project>
    <groupId>com.razorpay</groupId>
    <artifactId>catalyst</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.9</version>
        </dependency>
    </dependencies>
</project>
```

### Go

```go
// go.mod
module github.com/razorpay/catalyst

go 1.23

require github.com/gorilla/mux v1.8.0
```

### Comparison

| Aspect | Java (Maven) | Go |
|--------|-------------|-----|
| File format | XML | Simple text |
| Verbosity | High | Minimal |
| Lock file | pom.xml.lock (optional) | go.sum (always) |
| Repository | Maven Central | GitHub/anywhere |
| Version syntax | 5.3.9 | v5.3.9 (v prefix) |
| Install tool | Maven/Gradle | Built-in (go get) |

---

## 🔍 Visibility: Exported vs Unexported

### The Capital Letter Rule

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO'S VISIBILITY RULE                                           │
│                                                                 │
│  Capital letter   = Exported (Public)   = Other packages CAN    │
│  Lowercase letter = Unexported (Private) = Other packages CANNOT│
│                                                                 │
│  // utils/helpers.go                                            │
│  package utils                                                  │
│                                                                 │
│  func Add(a, b int) int {      ← Exported (Capital A)           │
│      return a + b                 Other packages can use        │
│  }                                                              │
│                                                                 │
│  func subtract(a, b int) int { ← Unexported (lowercase s)       │
│      return a - b                 Only this package can use     │
│  }                                                              │
│                                                                 │
│  var MaxValue = 100            ← Exported variable              │
│  var minValue = 0              ← Unexported variable            │
│                                                                 │
│  type User struct {            ← Exported type                  │
│      Name  string              ← Exported field                 │
│      email string              ← Unexported field               │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Java Comparison

```java
// Java: Explicit access modifiers
public class User {
    public String name;        // Anyone can access
    private String email;      // Only this class
    protected int age;         // Subclasses can access
    String phone;              // Package-private (default)
}
```

```go
// Go: Capital letter = public, lowercase = private
type User struct {
    Name  string    // Exported (like public)
    email string    // Unexported (like private)
    // No protected concept
    // No package-private concept
}
```

---

## 📂 Creating and Using Packages

### Step 1: Project Structure

```
myproject/
├── go.mod                  ← Module definition
├── main.go                 ← Entry point
└── greeting/               ← Our package
    └── greet.go            ← Package code
```

### Step 2: Create go.mod

```bash
cd myproject
go mod init github.com/username/myproject
```

### Step 3: Create the Package

```go
// greeting/greet.go
package greeting

import "fmt"

// Hello returns a greeting message (Exported - Capital H)
func Hello(name string) string {
    return fmt.Sprintf("Hello, %s!", name)
}

// goodbye is unexported (lowercase g)
func goodbye(name string) string {
    return fmt.Sprintf("Goodbye, %s!", name)
}
```

### Step 4: Use the Package

```go
// main.go
package main

import (
    "fmt"
    "github.com/username/myproject/greeting"  // Import our package
)

func main() {
    message := greeting.Hello("Go Developer")  // ✅ Works (exported)
    fmt.Println(message)
    
    // greeting.goodbye("Go")  // ❌ ERROR! unexported function
}
```

### Step 5: Run

```bash
go run main.go
# Output: Hello, Go Developer!
```

---

## 📂 Package Patterns in Production

### Pattern 1: cmd/ for Multiple Entry Points

```
catalyst/
├── cmd/
│   ├── api/
│   │   └── main.go        ← go run ./cmd/api
│   ├── worker/
│   │   └── main.go        ← go run ./cmd/worker
│   └── migration/
│       └── main.go        ← go run ./cmd/migration
├── internal/              ← Private to this module
│   ├── handler/
│   ├── service/
│   └── repository/
├── pkg/                   ← Public, can be imported
│   └── utils/
└── go.mod
```

### Pattern 2: internal/ for Private Code

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  THE "internal" DIRECTORY IS SPECIAL                            │
│                                                                 │
│  mymodule/                                                      │
│    ├── internal/         ← MAGIC DIRECTORY!                     │
│    │   └── secrets/      Packages here can ONLY be imported by  │
│    │       └── key.go    packages in mymodule (same module)     │
│    │                                                            │
│    └── pkg/              ← Normal directory                     │
│        └── utils/        Packages here can be imported by       │
│            └── helper.go anyone                                 │
│                                                                 │
│  Example:                                                       │
│  ✅ mymodule/cmd/api can import mymodule/internal/secrets       │
│  ❌ othermodule cannot import mymodule/internal/secrets         │
│     (Go compiler enforces this!)                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🏭 Real Catalyst Example

### Project Structure

```
github.com/razorpay/catalyst/
├── go.mod                            ← Module: github.com/razorpay/catalyst
├── go.sum
├── cmd/
│   └── api/
│       └── main.go                   ← package main
├── internal/
│   ├── boot/
│   │   └── boot.go                   ← package boot
│   ├── mgst/
│   │   ├── manager.go                ← package mgst
│   │   └── entity/
│   │       └── mgst.go               ← package entity
│   └── server/
│       └── server.go                 ← package server
└── pkg/
    └── utils/
        └── utils.go                  ← package utils
```

### How Imports Work

```go
// cmd/api/main.go
package main

import (
    // Standard library
    "context"
    "fmt"
    
    // Our internal packages (use full module path)
    "github.com/razorpay/catalyst/internal/boot"
    "github.com/razorpay/catalyst/internal/server"
    
    // External dependencies
    "google.golang.org/grpc"
)

func main() {
    ctx := context.Background()
    
    // Using our internal package
    boot.Initialize(ctx)
    
    // Using external package
    grpcServer := grpc.NewServer()
    
    server.Start(grpcServer)
}
```

---

## 🎯 Key Takeaways

1. **Package** = Directory of related .go files with same package name
2. **Module** = Collection of packages with go.mod at root
3. **go.mod** = Defines module identity and dependencies
4. **go.sum** = Security checksums for dependencies
5. **Capital letter** = Exported (public), **lowercase** = Unexported (private)
6. **internal/** = Private to module (enforced by compiler)
7. **Import path** = Module path + relative path to package

---

## 🔧 Common Commands

```bash
# Initialize a new module
go mod init github.com/username/project

# Add a dependency
go get github.com/gorilla/mux@v1.8.0

# Remove unused dependencies
go mod tidy

# Download dependencies
go mod download

# Show dependency graph
go mod graph

# Verify dependencies against go.sum
go mod verify

# Show why a dependency is needed
go mod why github.com/some/package
```

---

## ➡️ Next Steps

You now understand how Go organizes code with packages and modules. Let's explore the Go toolchain in depth.

**Next Topic:** [05 - Go Toolchain](./05-go-toolchain.md)

