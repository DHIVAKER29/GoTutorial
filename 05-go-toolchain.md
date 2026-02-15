# 05 - Go Toolchain

> Understanding all the commands and tools that come with Go.

---

## 📌 What You'll Learn

- All `go` commands and when to use them
- How `go build`, `go run`, and `go install` differ
- How to format, test, and vet your code
- Cross-compilation for different platforms
- Real production build patterns

---

## 🔧 The go Command

When you install Go, you get a single command: `go`. This command does everything:

**$ go \<command\> [arguments]**

| Category | Commands |
|----------|----------|
| **Building Code** | `go build` — Compile packages and dependencies |
| | `go run` — Compile and run Go program |
| | `go install` — Compile and install packages and dependencies |
| | `go generate` — Run code generators |
| **Managing Dependencies** | `go mod` — Module maintenance |
| | `go get` — Download and install packages |
| | `go work` — Workspace maintenance |
| **Testing & Quality** | `go test` — Run tests |
| | `go vet` — Report suspicious code |
| | `go fmt` — Format source code |
| **Information** | `go doc` — Show documentation |
| | `go env` — Print Go environment |
| | `go list` — List packages or modules |
| | `go version` — Print Go version |
| **Debugging** | `go tool` — Run specified go tool |

---

## 🏗️ go build

### What it Does

Compiles your Go source code into an executable binary.

### Basic Usage

```bash
# Build current directory (creates binary with module name)
go build

# Build and name the output
go build -o myapp

# Build a specific package
go build ./cmd/api

# Build with path
go build -o bin/api ./cmd/api
```

### The Compilation Process

**go build ./cmd/api**

Source Files (.go) → Compiler (fast!) → Binary (portable) api

**What goes INTO the binary:**
- Your compiled code
- All dependency code
- Go runtime (GC, scheduler, etc.)
- Standard library parts you use

**What is NOT needed to RUN:**
- Go installation
- Source files
- Any external dependencies

### Build Flags

```bash
# Verbose output
go build -v ./...

# Print commands being run
go build -x ./...

# Race detector (for testing)
go build -race ./...

# Strip debug info (smaller binary)
go build -ldflags="-s -w" ./...

# Set version at build time
go build -ldflags="-X main.version=1.0.0" ./...
```

---

## 🏃 go run

### What it Does

Compiles AND runs in one step. The binary is temporary.

### Basic Usage

```bash
# Run a single file
go run main.go

# Run multiple files
go run main.go helper.go

# Run a package
go run ./cmd/api

# Pass arguments to your program
go run main.go arg1 arg2
```

### go run vs go build

| Command | What it does | Use for |
|---------|--------------|---------|
| **go run main.go** | 1. Compile to /tmp/go-build-xxx/main<br>2. Execute the binary<br>3. Delete the binary | Quick testing during development. Don't use for production deployment. |
| **go build -o main main.go** | 1. Compile to ./main (permanent)<br>2. (You run it manually later) | Production builds, distribution |

---

## 📦 go install

### What it Does

Compiles and installs the binary to `$GOPATH/bin` (or `$GOBIN`).

### Basic Usage

```bash
# Install from current directory
go install

# Install a specific package
go install ./cmd/api

# Install from remote (tools)
go install golang.org/x/tools/gopls@latest
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

### Where Binaries Go

1. `$GOBIN` if set
2. `$GOPATH/bin` (default: ~/go/bin)

Make sure this is in your PATH:
```bash
export PATH=$PATH:$(go env GOPATH)/bin
```

Then you can run installed tools anywhere:
```bash
$ gopls version
$ golangci-lint run
```

---

## 📚 go get

### What it Does

Downloads and installs packages and dependencies.

### Basic Usage

```bash
# Add a dependency (latest version)
go get github.com/gorilla/mux

# Add a specific version
go get github.com/gorilla/mux@v1.8.0

# Update to latest
go get -u github.com/gorilla/mux

# Update all dependencies
go get -u ./...

# Downgrade a version
go get github.com/gorilla/mux@v1.7.0
```

### Semantic Versioning

**Version format: vMAJOR.MINOR.PATCH**

Example: v1.8.3
- **MAJOR** — Breaking changes (incompatible)
- **MINOR** — New features (compatible)
- **PATCH** — Bug fixes (compatible)

**Version selection:**
- `@v1.8.0` — Exact version
- `@latest` — Latest stable version
- `@v1` — Latest v1.x.x
- `@master` — Latest commit on master (not recommended)

---

## 📁 go mod

### What it Does

Module management commands.

### All Subcommands

```bash
# Initialize a new module
go mod init github.com/username/project

# Add missing, remove unused dependencies
go mod tidy

# Download dependencies to local cache
go mod download

# Verify dependencies haven't been modified
go mod verify

# Show module graph
go mod graph

# Explain why a module is needed
go mod why github.com/some/package

# Create a vendor directory
go mod vendor

# Edit go.mod (rarely used directly)
go mod edit -require=github.com/pkg/errors@v0.9.1
```

### go mod tidy - The Most Used

**Does TWO things:**

1. **ADDS missing dependencies**
   - Your code: `import "github.com/gorilla/mux"`
   - go.mod: (missing)
   - After tidy: `require github.com/gorilla/mux v1.8.0`

2. **REMOVES unused dependencies**
   - Your code: (no longer uses github.com/old/lib)
   - go.mod: `require github.com/old/lib v1.0.0`
   - After tidy: (removed)

**ALWAYS run this before committing!**

---

## 🧹 go fmt

### What it Does

Formats Go source code to the standard style.

### Basic Usage

```bash
# Format a file (prints to stdout)
go fmt main.go

# Format and overwrite files
gofmt -w main.go

# Format all files in project
go fmt ./...

# Format with imports (better)
goimports -w .
```

### Before and After

```go
// Before: Messy code
package main
import "fmt"
func main(){
x:=1
if x==1{
fmt.Println("one")
}}

// After: go fmt
package main

import "fmt"

func main() {
    x := 1
    if x == 1 {
        fmt.Println("one")
    }
}
```

### Why Formatting Matters

**Go's philosophy on formatting:**

> "Gofmt's style is no one's favorite, yet gofmt is everyone's favorite."

**Benefits:**
- No debates about tabs vs spaces
- No debates about brace placement
- All Go code looks the same
- Easy to read others' code
- Automated in CI/CD pipelines

---

## ✅ go test

### What it Does

Runs tests in files ending with `_test.go`.

### Basic Usage

```bash
# Run all tests in current package
go test

# Run all tests in project
go test ./...

# Verbose output
go test -v ./...

# Run specific test
go test -run TestMyFunction

# With coverage
go test -cover ./...

# Generate coverage report
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

### Test File Example

```go
// math.go
package math

func Add(a, b int) int {
    return a + b
}
```

```go
// math_test.go
package math

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Add(2, 3) = %d; want 5", result)
    }
}
```

---

## 🔍 go vet

### What it Does

Reports likely bugs in your code.

### Basic Usage

```bash
# Check current package
go vet

# Check all packages
go vet ./...
```

### What It Catches

```go
// go vet catches these:

// 1. Printf format errors
fmt.Printf("Name: %d", "Alice")  // %d for string

// 2. Unreachable code
func example() {
    return
    fmt.Println("Never runs")  // Unreachable
}

// 3. Invalid struct tags
type User struct {
    Name string `json:name`  // Missing quotes
}

// 4. Copying locks
var mu sync.Mutex
mu2 := mu  // Copying a mutex (bad!)
```

---

## 🌍 Cross-Compilation

### What it Is

Building for a different operating system or architecture.

### How to Do It

```bash
# Build for Linux from Mac
GOOS=linux GOARCH=amd64 go build -o myapp-linux

# Build for Windows from Mac
GOOS=windows GOARCH=amd64 go build -o myapp.exe

# Build for Mac ARM (M1/M2) from Intel Mac
GOOS=darwin GOARCH=arm64 go build -o myapp-arm

# Build for Raspberry Pi (ARM)
GOOS=linux GOARCH=arm GOARM=7 go build -o myapp-pi
```

### Supported Platforms

| GOOS (Operating System) | GOARCH (Architecture) |
|-------------------------|------------------------|
| linux | amd64 (64-bit Intel/AMD) |
| darwin (macOS) | arm64 (64-bit ARM, M1/M2) |
| windows | 386 (32-bit Intel) |
| freebsd | arm (32-bit ARM) |
| android | ppc64 (PowerPC) |
| ios | wasm (WebAssembly) |

See all combinations:
```bash
$ go tool dist list
```

### Why This is Amazing

| Language | Cross-compilation |
|----------|-------------------|
| **C++** | Need cross-compiler toolchain for each target. Often need target OS libraries. Complex setup. |
| **Java** | "Write once, run anywhere" but requires JVM on target. JVM is 200+ MB. |
| **Go** | Just set GOOS and GOARCH. Works out of the box. Binary includes everything. No runtime needed on target. |

---

## 🏭 Production Build Example (Makefile)

### From Real Catalyst Codebase

```makefile
# Build flags for production
CGO_ENABLED=0
GOOS=linux
GOARCH=amd64
LDFLAGS=-s -w -X main.version=$(VERSION)

# Build API binary
go-build-api:
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
	go build -ldflags="$(LDFLAGS)" \
	-o bin/api ./cmd/api

# Build Worker binary
go-build-worker:
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
	go build -ldflags="$(LDFLAGS)" \
	-o bin/worker ./cmd/worker

# Build all binaries
go-build-all: go-build-api go-build-worker
```

### Build Flags Explained

| Flag | Meaning |
|------|---------|
| **CGO_ENABLED=0** | Disable C code compilation. Creates a fully static binary. Works better in Docker containers. |
| **-ldflags="-s -w"** | -s: Omit symbol table (smaller binary). -w: Omit DWARF debug info (smaller binary). Result: ~30% smaller binary. |
| **-X main.version=$(VERSION)** | Inject version at build time. In code: `var version string`. After build: version = "1.0.0" |

---

## 📊 Summary: When to Use Each Command

| Command | When to Use |
|---------|-------------|
| `go run` | Quick testing during development |
| `go build` | Create production binaries |
| `go install` | Install tools to $GOPATH/bin |
| `go get` | Add or update dependencies |
| `go mod init` | Start a new project |
| `go mod tidy` | Before every commit |
| `go fmt` | Before every commit (or auto-save) |
| `go vet` | Before every commit |
| `go test` | Before every commit |
| `go doc` | Look up documentation |

---

## 🆚 Java Comparison

| Go Command | Java Equivalent | Notes |
|------------|-----------------|-------|
| `go build` | `mvn package` or `javac` | Go is faster |
| `go run` | `mvn exec:java` | Go is simpler |
| `go test` | `mvn test` or `junit` | Built-in |
| `go fmt` | Checkstyle/SpotBugs | Enforced, not optional |
| `go get` | Maven dependencies | In go.mod |
| `go mod init` | `mvn archetype:generate` | Much simpler |
| `go doc` | Javadoc | Built-in |

---

## 🎯 Key Takeaways

1. **`go build`** = Compile to binary (production)
2. **`go run`** = Compile + run (development)
3. **`go install`** = Compile + install globally (tools)
4. **`go mod tidy`** = Always run before committing
5. **`go fmt`** = Mandatory formatting (no debates!)
6. **`go test`** = Built-in testing
7. **Cross-compilation** = Just set GOOS and GOARCH

---

## ➡️ Next Steps

You now understand the Go toolchain. Let's dive into variables and constants.

**Next Topic:** [06 - Variables & Constants](./06-variables-and-constants.md)
