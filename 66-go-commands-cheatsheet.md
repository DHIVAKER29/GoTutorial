# 66 - Go Commands Cheatsheet

> Complete reference for all Go CLI commands with explanations.

---

## 📋 Quick Reference

```bash
# Most used commands
go run main.go          # Compile and run
go build                # Compile to binary
go test ./...           # Run all tests
go mod tidy             # Clean up dependencies
go fmt ./...            # Format all code
```

---

## 🏗️ Build & Run Commands

### `go run`
```bash
# Compile and execute in one step (doesn't create binary)
go run main.go                    # Run single file
go run .                          # Run package in current dir
go run ./cmd/api                  # Run specific package

# With build flags
go run -race main.go              # Run with race detector
go run -v main.go                 # Verbose output
```

### `go build`
```bash
# Compile to executable binary
go build                          # Build current package → binary named after dir
go build -o myapp                 # Specify output name
go build -o bin/myapp ./cmd/api   # Build specific package to path

# Build flags
go build -v                       # Verbose (show packages being compiled)
go build -race                    # Include race detector
go build -ldflags="-s -w"         # Strip debug info (smaller binary)
go build -ldflags="-X main.Version=1.0.0"  # Inject version

# Cross-compilation
GOOS=linux GOARCH=amd64 go build -o myapp-linux
GOOS=windows GOARCH=amd64 go build -o myapp.exe
GOOS=darwin GOARCH=arm64 go build -o myapp-mac-arm

# Common GOOS: linux, darwin, windows, freebsd
# Common GOARCH: amd64, arm64, arm, 386
```

### `go install`
```bash
# Compile and install to $GOPATH/bin (or $GOBIN)
go install                        # Install current package
go install ./...                  # Install all packages
go install github.com/user/tool@latest  # Install remote tool

# Example: Install golangci-lint
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

### `go clean`
```bash
go clean                          # Remove build artifacts
go clean -cache                   # Clear build cache
go clean -testcache               # Clear test cache
go clean -modcache                # Clear module cache (all downloaded deps!)
go clean -i                       # Remove installed binaries
```

---

## 📦 Module Commands

### `go mod`
```bash
# Initialize new module
go mod init github.com/user/project

# Dependency management
go mod tidy                       # Add missing, remove unused deps
go mod download                   # Download deps to cache
go mod verify                     # Verify deps match go.sum
go mod graph                      # Print dependency graph
go mod why github.com/pkg/errors  # Why is this dep needed?

# Editing go.mod
go mod edit -require github.com/pkg/errors@v1.0.0
go mod edit -droprequire github.com/old/pkg
go mod edit -replace github.com/old=github.com/new@v1.0.0
go mod edit -go=1.21              # Set Go version

# Vendoring
go mod vendor                     # Create vendor directory
go mod vendor -v                  # Verbose vendor output
```

### `go get`
```bash
# Add/update dependencies
go get github.com/gin-gonic/gin              # Latest version
go get github.com/gin-gonic/gin@v1.9.1       # Specific version
go get github.com/gin-gonic/gin@latest       # Latest release
go get github.com/gin-gonic/gin@master       # Latest commit on branch

# Update flags
go get -u github.com/gin-gonic/gin           # Update to latest minor/patch
go get -u=patch github.com/gin-gonic/gin     # Update patch only
go get -u ./...                              # Update all deps

# Remove dependency
go get github.com/old/pkg@none               # Remove from go.mod
```

### `go list`
```bash
go list                           # Current package import path
go list ./...                     # All packages in module
go list -m all                    # All modules (deps)
go list -m -versions github.com/gin-gonic/gin  # Available versions
go list -m -u all                 # Show available updates
go list -f '{{.ImportPath}}'      # Custom format

# JSON output
go list -json                     # Package info as JSON
go list -m -json all              # Module info as JSON
```

---

## 🧪 Testing Commands

### `go test`
```bash
# Basic testing
go test                           # Test current package
go test ./...                     # Test all packages
go test -v                        # Verbose output
go test -v ./pkg/...              # Test specific path

# Running specific tests
go test -run TestName             # Run tests matching pattern
go test -run TestAdd/positive     # Run subtest
go test -run "Test.*Error"        # Regex pattern

# Test modes
go test -race                     # Race detector
go test -short                    # Skip long tests
go test -count=5                  # Run N times
go test -parallel=4               # Max parallel tests
go test -timeout=30s              # Test timeout
go test -failfast                 # Stop on first failure

# Coverage
go test -cover                    # Show coverage %
go test -coverprofile=cover.out   # Generate coverage file
go tool cover -html=cover.out     # Open coverage in browser
go tool cover -func=cover.out     # Coverage by function

# Benchmarks
go test -bench=.                  # Run all benchmarks
go test -bench=BenchmarkAdd       # Specific benchmark
go test -bench=. -benchmem        # Show memory allocations
go test -bench=. -benchtime=5s    # Run for 5 seconds
go test -bench=. -count=10        # Run 10 times

# CPU/Memory profiling
go test -cpuprofile=cpu.out -bench=.
go test -memprofile=mem.out -bench=.
```

---

## 🔍 Analysis Commands

### `go vet`
```bash
# Static analysis for common errors
go vet                            # Current package
go vet ./...                      # All packages
go vet -v ./...                   # Verbose

# Specific checks
go vet -printf ./...              # Check printf format strings
go vet -shadow ./...              # Find shadowed variables
```

### `go fmt` / `gofmt`
```bash
# Format code
go fmt ./...                      # Format all .go files
gofmt -w .                        # Write changes in place
gofmt -d .                        # Show diff (don't change)
gofmt -s -w .                     # Simplify code + write

# Check if formatted
gofmt -l .                        # List unformatted files
test -z "$(gofmt -l .)"           # Exit 1 if unformatted
```

### `go doc`
```bash
# View documentation
go doc fmt                        # Package docs
go doc fmt.Println                # Function docs
go doc -all fmt                   # All docs for package
go doc -src fmt.Println           # Show source code

# Start doc server
godoc -http=:6060                 # Browse docs at localhost:6060
```

---

## 🛠️ Tool Commands

### `go tool`
```bash
# List available tools
go tool

# Compilation analysis
go tool compile -S main.go        # Show assembly
go tool compile -m main.go        # Escape analysis

# Binary analysis
go tool nm myapp                  # List symbols
go tool objdump myapp             # Disassemble binary
go tool objdump -s main.main myapp  # Specific function

# Profiling
go tool pprof cpu.out             # Analyze CPU profile
go tool pprof -http=:8080 cpu.out # Web UI for profile
go tool trace trace.out           # Analyze trace

# Coverage
go tool cover -html=cover.out     # Coverage visualization
go tool cover -func=cover.out     # Coverage by function
```

### `go generate`
```bash
# Run code generators
go generate                       # Current package
go generate ./...                 # All packages
go generate -v ./...              # Verbose
go generate -n ./...              # Dry run (show commands)

# In source file:
//go:generate stringer -type=Status
//go:generate mockgen -source=interface.go -destination=mock.go
```

### `go fix`
```bash
# Update code for new Go versions
go fix ./...
```

---

## 🔧 Environment Commands

### `go env`
```bash
# View environment
go env                            # All variables
go env GOPATH                     # Specific variable
go env GOROOT GOPATH GOOS GOARCH  # Multiple variables

# Set environment
go env -w GOPROXY=https://proxy.golang.org,direct
go env -w GOPRIVATE=github.com/mycompany/*
go env -w GO111MODULE=on
go env -w GOBIN=$HOME/bin

# Unset
go env -u GOPROXY

# Common variables
GOROOT      # Go installation directory
GOPATH      # Workspace (legacy) / module cache
GOBIN       # Where 'go install' puts binaries
GOPROXY     # Module proxy URL
GOPRIVATE   # Private module patterns
GONOSUMDB   # Skip checksum for these modules
GOOS        # Target OS
GOARCH      # Target architecture
CGO_ENABLED # Enable/disable CGO
```

### `go version`
```bash
go version                        # Go version
go version -m myapp               # Show build info of binary
```

---

## 📊 Profiling Flags

```bash
# Build with profiling
go build -gcflags="-m"            # Escape analysis decisions
go build -gcflags="-m -m"         # More verbose escape analysis
go build -gcflags="-S"            # Assembly output
go build -gcflags="-N -l"         # Disable optimizations

# Runtime profiling (via HTTP)
import _ "net/http/pprof"
# Then: go tool pprof http://localhost:6060/debug/pprof/profile

# Generate SSA HTML
GOSSAFUNC=main go build           # Creates ssa.html
```

---

## 🚀 Common Workflows

### New Project
```bash
mkdir myproject && cd myproject
go mod init github.com/user/myproject
# Write code...
go mod tidy
go build
```

### Add Dependency
```bash
go get github.com/gin-gonic/gin@latest
go mod tidy
```

### Run Tests with Coverage
```bash
go test -v -race -cover -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

### Build for Production
```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
  go build -ldflags="-s -w -X main.Version=1.0.0" \
  -o bin/myapp ./cmd/api
```

### Update All Dependencies
```bash
go get -u ./...
go mod tidy
go mod verify
```

### Check Code Quality
```bash
go fmt ./...
go vet ./...
go test -race ./...
golangci-lint run
```

---

## 📝 Makefile Example

```makefile
.PHONY: build test clean

APP_NAME := myapp
VERSION := $(shell git describe --tags --always)

build:
	CGO_ENABLED=0 go build -ldflags="-s -w -X main.Version=$(VERSION)" -o bin/$(APP_NAME) ./cmd/api

test:
	go test -v -race -cover ./...

lint:
	go fmt ./...
	go vet ./...
	golangci-lint run

clean:
	go clean
	rm -rf bin/

deps:
	go mod download
	go mod verify

tidy:
	go mod tidy

docker:
	docker build -t $(APP_NAME):$(VERSION) .
```

---

## 🎯 Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────┐
│  GO COMMANDS QUICK REFERENCE                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BUILD & RUN                                                    │
│    go run main.go         Compile and execute                  │
│    go build               Create binary                        │
│    go install             Build and install to GOBIN           │
│                                                                 │
│  MODULES                                                        │
│    go mod init            Initialize module                    │
│    go mod tidy            Sync dependencies                    │
│    go get pkg@version     Add/update dependency                │
│    go mod vendor          Create vendor directory              │
│                                                                 │
│  TESTING                                                        │
│    go test ./...          Run all tests                        │
│    go test -v -race       Verbose with race detector           │
│    go test -cover         Show coverage                        │
│    go test -bench=.       Run benchmarks                       │
│                                                                 │
│  CODE QUALITY                                                   │
│    go fmt ./...           Format code                          │
│    go vet ./...           Static analysis                      │
│    go doc pkg             View documentation                   │
│                                                                 │
│  ENVIRONMENT                                                    │
│    go env                 Show environment                     │
│    go version             Show Go version                      │
│    go list -m all         List all modules                     │
│                                                                 │
│  CROSS-COMPILE                                                  │
│    GOOS=linux GOARCH=amd64 go build                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎉 You now have a complete Go command reference!

Print this cheatsheet or keep it handy while developing! 🚀

