# 48 - Build Constraints and Tags

> Conditional compilation for different platforms and build configurations.

---

## 📌 What You'll Learn

- Build constraints (build tags)
- Platform-specific code
- Custom build tags
- go:generate directive

---

## 🔧 Build Constraints

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  BUILD CONSTRAINTS = Compile code conditionally                 │
│                                                                 │
│  USE CASES:                                                     │
│  • Platform-specific code (Windows vs Linux)                    │
│  • Architecture-specific (amd64 vs arm)                         │
│  • Build modes (debug vs release)                               │
│  • Optional features                                            │
│                                                                 │
│  TWO WAYS TO SPECIFY:                                           │
│                                                                 │
│  1. File naming convention:                                     │
│     file_linux.go     → Linux only                              │
│     file_windows.go   → Windows only                            │
│     file_darwin.go    → macOS only                              │
│                                                                 │
│  2. //go:build directive:                                       │
│     //go:build linux && amd64                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Platform-Specific Code

```go
// main.go - shared code
package main

func main() {
    info := getSystemInfo()
    println(info)
}
```

```go
// system_linux.go
//go:build linux

package main

func getSystemInfo() string {
    return "Running on Linux"
}
```

```go
// system_darwin.go
//go:build darwin

package main

func getSystemInfo() string {
    return "Running on macOS"
}
```

```go
// system_windows.go
//go:build windows

package main

func getSystemInfo() string {
    return "Running on Windows"
}
```

**Output:** (varies by GOOS build target)
```
Running on Linux
```
(or "Running on macOS" for darwin, "Running on Windows" for windows)

---

## 🏷️ Build Tags Syntax

```go
// Build tag examples

// Single constraint
//go:build linux

// OR (either condition)
//go:build linux || darwin

// AND (both conditions)
//go:build linux && amd64

// NOT
//go:build !windows

// Complex expression
//go:build (linux || darwin) && amd64

// Custom build tags
//go:build debug

// Multiple constraints
//go:build linux && cgo && !race
```

```bash
# Build commands
go build                          # Normal build
go build -tags debug              # With debug tag
go build -tags "debug integration"  # Multiple tags
GOOS=linux go build               # Cross-compile
GOOS=windows GOARCH=amd64 go build
```

---

## 🎭 Debug vs Release

```go
// debug.go
//go:build debug

package main

import "log"

func debugLog(msg string) {
    log.Printf("[DEBUG] %s\n", msg)
}

const DebugMode = true
```

```go
// release.go
//go:build !debug

package main

func debugLog(msg string) {
    // No-op in release
}

const DebugMode = false
```

```go
// main.go
package main

import "fmt"

func main() {
    debugLog("Starting application...")
    
    if DebugMode {
        fmt.Println("Running in debug mode")
    }
    
    fmt.Println("Hello, World!")
}
```

**Output:**
```
(When built with -tags debug:)
[DEBUG] Starting application...
Running in debug mode
Hello, World!

(When built without -tags debug:)
Hello, World!
```

```bash
# Build for debugging
go build -tags debug -o myapp-debug

# Build for release (default)
go build -o myapp
```

---

## ⚙️ go:generate

```go
// generate.go
package main

// Run code generators
//go:generate stringer -type=Status
//go:generate mockgen -source=interface.go -destination=mock_interface.go
//go:generate protoc --go_out=. user.proto

type Status int

const (
    StatusPending Status = iota
    StatusActive
    StatusCompleted
)
```

```bash
# Run all generators in package
go generate ./...

# Run generators in specific file
go generate generate.go
```

---

## 📂 File Naming Conventions

```
File naming patterns (auto-applied):

*_GOOS.go           → file_linux.go, file_darwin.go
*_GOARCH.go         → file_amd64.go, file_arm64.go
*_GOOS_GOARCH.go    → file_linux_amd64.go
*_test.go           → Test files only

Examples:
  server.go              → Always compiled
  server_linux.go        → Linux only
  server_linux_amd64.go  → Linux 64-bit only
  server_test.go         → Only in test build

Common GOOS values:
  linux, darwin, windows, freebsd, android

Common GOARCH values:
  amd64, arm64, arm, 386
```

---

## 🎯 Key Takeaways

1. **//go:build** for compile-time constraints
2. **File naming** (`_linux.go`) for platform code
3. **Custom tags** with `-tags` flag
4. **go generate** for code generation
5. **GOOS/GOARCH** for cross-compilation

---

## ➡️ Next Steps

**Next Topic:** [49 - Module Management](./49-modules.md)

