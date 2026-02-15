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

**Build constraints** = Compile code conditionally

| Use Case | Example |
|----------|---------|
| Platform-specific | Windows vs Linux |
| Architecture-specific | amd64 vs arm |
| Build modes | debug vs release |
| Optional features | Custom tags |

**Two ways to specify:**
1. **File naming:** `file_linux.go`, `file_windows.go`, `file_darwin.go`
2. **//go:build directive:** `//go:build linux && amd64`

---

## 📝 Platform-Specific Code

```go
// main.go - shared code
package main
func main() {
    println(getSystemInfo())
}
```

```go
// system_linux.go
//go:build linux
package main
func getSystemInfo() string { return "Running on Linux" }
```

```go
// system_darwin.go
//go:build darwin
package main
func getSystemInfo() string { return "Running on macOS" }
```

```go
// system_windows.go
//go:build windows
package main
func getSystemInfo() string { return "Running on Windows" }
```

**Output:** Varies by `GOOS` build target (e.g., "Running on Linux")

---

## 🏷️ Build Tags Syntax

```go
//go:build linux
//go:build linux || darwin
//go:build linux && amd64
//go:build !windows
//go:build (linux || darwin) && amd64
//go:build debug
```

```bash
go build
go build -tags debug
go build -tags "debug integration"
GOOS=linux go build
GOOS=windows GOARCH=amd64 go build
```

---

## 🎭 Debug vs Release

```go
// debug.go
//go:build debug
package main
func debugLog(msg string) { log.Printf("[DEBUG] %s\n", msg) }
const DebugMode = true
```

```go
// release.go
//go:build !debug
package main
func debugLog(msg string) {}
const DebugMode = false
```

```bash
go build -tags debug -o myapp-debug
go build -o myapp
```

---

## ⚙️ go:generate

```go
//go:generate stringer -type=Status
//go:generate mockgen -source=interface.go -destination=mock_interface.go
```

```bash
go generate ./...
go generate generate.go
```

---

## 📂 File Naming Conventions

| Pattern | Example | When compiled |
|---------|---------|----------------|
| `*_GOOS.go` | `file_linux.go` | Linux only |
| `*_GOARCH.go` | `file_amd64.go` | amd64 only |
| `*_GOOS_GOARCH.go` | `file_linux_amd64.go` | Linux amd64 only |
| `*_test.go` | `server_test.go` | Test build only |

**Common values:** GOOS: linux, darwin, windows | GOARCH: amd64, arm64, arm, 386

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
