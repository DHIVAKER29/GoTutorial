# 02 - Setting Up Go

> Installing Go, understanding the environment, GOPATH evolution, and configuring your development setup.

---

## 📌 What You'll Learn

- How to install Go on your operating system
- The complete Go directory structure (bin, src, pkg)
- GOPATH: The problem it solved and the problems it created
- How Go Modules solved GOPATH's problems
- Setting up your code editor
- Verifying your installation with sample programs

---

## 🖥️ Installing Go

### Step 1: Download Go

Go to the official Go website: **https://go.dev/dl/**

Download the installer for your operating system:

| Operating System | File Type | Example |
|------------------|-----------|---------|
| Windows | `.msi` | go1.22.0.windows-amd64.msi |
| macOS (Intel) | `.pkg` | go1.22.0.darwin-amd64.pkg |
| macOS (Apple Silicon) | `.pkg` | go1.22.0.darwin-arm64.pkg |
| Linux | `.tar.gz` | go1.22.0.linux-amd64.tar.gz |

### Step 2: Install

#### Windows
1. Double-click the `.msi` file
2. Follow the installation wizard
3. Go is installed to `C:\Go` by default

#### macOS
1. Double-click the `.pkg` file
2. Follow the installation wizard
3. Go is installed to `/usr/local/go`

#### Linux
```bash
# Remove any previous installation
sudo rm -rf /usr/local/go

# Extract the archive
sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH=$PATH:/usr/local/go/bin
```

### Step 3: Verify Installation

```bash
go version
```

Expected output:
```
go version go1.22.0 darwin/arm64
```

---

## 📁 Go Installation Directory Structure (GOROOT)

When you install Go, it creates a specific directory structure. This is called **GOROOT**.

### The Complete GOROOT Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GOROOT (Where Go itself is installed)                          │
│  ─────────────────────────────────────                          │
│                                                                 │
│  /usr/local/go/  (or C:\Go on Windows)                          │
│  │                                                              │
│  ├── bin/                    ← GO EXECUTABLES                   │
│  │   ├── go                  ← The main go command              │
│  │   └── gofmt               ← The formatter                    │
│  │                                                              │
│  ├── src/                    ← STANDARD LIBRARY SOURCE          │
│  │   ├── fmt/                ← fmt package source               │
│  │   │   ├── format.go                                          │
│  │   │   ├── print.go                                           │
│  │   │   └── scan.go                                            │
│  │   ├── net/                ← net package source               │
│  │   │   ├── http/           ← net/http subpackage              │
│  │   │   │   ├── server.go                                      │
│  │   │   │   └── client.go                                      │
│  │   │   └── ...                                                │
│  │   ├── os/                 ← os package source                │
│  │   ├── io/                 ← io package source                │
│  │   └── ...                 ← All standard library             │
│  │                                                              │
│  ├── pkg/                    ← COMPILED STANDARD LIBRARY        │
│  │   └── darwin_arm64/       ← Platform-specific                │
│  │       ├── fmt.a           ← Compiled fmt package             │
│  │       ├── net/                                               │
│  │       │   └── http.a      ← Compiled net/http                │
│  │       └── ...                                                │
│  │                                                              │
│  ├── doc/                    ← DOCUMENTATION                    │
│  │   ├── go_spec.html        ← Language specification           │
│  │   └── ...                                                    │
│  │                                                              │
│  ├── lib/                    ← SUPPORTING LIBRARIES             │
│  │   └── time/               ← Timezone data                    │
│  │                                                              │
│  ├── misc/                   ← MISCELLANEOUS                    │
│  │   ├── cgo/                ← C interop examples               │
│  │   └── wasm/               ← WebAssembly support              │
│  │                                                              │
│  └── api/                    ← API COMPATIBILITY FILES          │
│      └── go1.22.txt          ← Lists all public APIs            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Understanding Each Directory

| Directory | Purpose | Example Contents |
|-----------|---------|------------------|
| `bin/` | Executable programs | `go`, `gofmt` |
| `src/` | Standard library source code | `fmt/print.go` |
| `pkg/` | Compiled standard library | `fmt.a` (archive) |
| `doc/` | Documentation | Language spec |
| `lib/` | Supporting data files | Timezone database |
| `misc/` | Miscellaneous resources | Examples, tools |
| `api/` | API compatibility | Public API lists |

### Sample Program: Exploring GOROOT

```go
// explore_goroot.go
package main

import (
    "fmt"
    "os"
    "path/filepath"
    "runtime"
)

func main() {
    // GOROOT is where Go is installed
    goroot := runtime.GOROOT()
    fmt.Println("GOROOT:", goroot)
    
    // List the main directories
    entries, err := os.ReadDir(goroot)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    
    fmt.Println("\nGOROOT contents:")
    for _, entry := range entries {
        if entry.IsDir() {
            fmt.Printf("  📁 %s/\n", entry.Name())
        } else {
            fmt.Printf("  📄 %s\n", entry.Name())
        }
    }
    
    // Show where fmt package source is
    fmtPath := filepath.Join(goroot, "src", "fmt")
    fmt.Printf("\nfmt package source: %s\n", fmtPath)
}
```

**Output:**
```
GOROOT: /usr/local/go

GOROOT contents:
  📁 api/
  📁 bin/
  📁 doc/
  📁 lib/
  📁 misc/
  📁 pkg/
  📁 src/

fmt package source: /usr/local/go/src/fmt
```

*Note: GOROOT path and directory contents may vary based on your Go installation.*

Run it:
```bash
go run explore_goroot.go
```

---

## 🔧 GOPATH: The Complete History

### The Problem Before GOPATH (Before 2009)

Before Go existed, each programming language had different approaches:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  C/C++: Header files everywhere                                 │
│  ─────────────────────────────────                              │
│  #include <stdio.h>       ← System includes                     │
│  #include "myheader.h"    ← Local includes                      │
│                                                                 │
│  Problems:                                                      │
│  • Include paths are confusing                                  │
│  • No standard project structure                                │
│  • Dependency management is manual                              │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  Java: CLASSPATH nightmare                                      │
│  ─────────────────────────                                      │
│  java -cp lib/*:classes:. MyApp                                 │
│                                                                 │
│  Problems:                                                      │
│  • Long classpaths                                              │
│  • JAR conflicts                                                │
│  • "JAR hell"                                                   │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  Python: Import chaos                                           │
│  ──────────────────────                                         │
│  import mymodule  # Where is it?                                │
│  sys.path.append('/some/random/path')                           │
│                                                                 │
│  Problems:                                                      │
│  • sys.path manipulation                                        │
│  • Virtual environments needed                                  │
│  • "Works on my machine"                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Go's Original Solution: GOPATH (2009-2018)

When Go was created, they introduced **GOPATH** as a single workspace for all Go code:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  THE GOPATH SOLUTION (Go 1.0 - Go 1.10)                         │
│                                                                 │
│  IDEA: "One workspace to rule them all"                         │
│                                                                 │
│  $GOPATH = ~/go (default)                                       │
│  │                                                              │
│  ├── bin/            ← YOUR installed executables               │
│  │   ├── myapp       ← After: go install myproject              │
│  │   ├── cobra       ← After: go install github.com/spf13/cobra │
│  │   └── gopls       ← After: go install golang.org/x/tools/... │
│  │                                                              │
│  ├── pkg/            ← Compiled package cache                   │
│  │   └── darwin_arm64/                                          │
│  │       └── github.com/                                        │
│  │           └── gorilla/                                       │
│  │               └── mux.a  ← Cached compilation                │
│  │                                                              │
│  └── src/            ← ALL SOURCE CODE (yours + deps)           │
│      ├── github.com/                                            │
│      │   ├── yourusername/                                      │
│      │   │   ├── project1/      ← Your project                  │
│      │   │   │   └── main.go                                    │
│      │   │   └── project2/      ← Another project               │
│      │   │       └── main.go                                    │
│      │   ├── gorilla/                                           │
│      │   │   └── mux/           ← Dependency                    │
│      │   │       └── mux.go                                     │
│      │   └── aws/                                               │
│      │       └── aws-sdk-go/    ← AWS SDK                       │
│      └── golang.org/                                            │
│          └── x/                                                 │
│              └── crypto/        ← Extended stdlib               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### GOPATH Directory Deep Dive

#### bin/ Directory

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  $GOPATH/bin/                                                   │
│                                                                 │
│  PURPOSE: Store executable binaries you install                 │
│                                                                 │
│  HOW IT WORKS:                                                  │
│  $ go install github.com/golangci/golangci-lint/cmd/...@latest  │
│                                                                 │
│  Creates:                                                       │
│  $GOPATH/bin/golangci-lint  ← Ready to use!                     │
│                                                                 │
│  TO USE THESE BINARIES:                                         │
│  Add to your shell profile (~/.bashrc or ~/.zshrc):             │
│  export PATH=$PATH:$(go env GOPATH)/bin                         │
│                                                                 │
│  Then you can run from anywhere:                                │
│  $ golangci-lint run                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### pkg/ Directory

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  $GOPATH/pkg/                                                   │
│                                                                 │
│  PURPOSE: Cache compiled packages (speeds up builds)            │
│                                                                 │
│  STRUCTURE:                                                     │
│  pkg/                                                           │
│  ├── mod/                    ← MODULE CACHE (Go 1.11+)          │
│  │   ├── cache/              ← Download cache                   │
│  │   │   └── download/                                          │
│  │   │       └── github.com/                                    │
│  │   │           └── gorilla/                                   │
│  │   │               └── mux/                                   │
│  │   │                   └── @v/                                │
│  │   │                       ├── v1.8.0.zip                     │
│  │   │                       └── v1.8.0.mod                     │
│  │   └── github.com/         ← Extracted modules                │
│  │       └── gorilla/                                           │
│  │           └── mux@v1.8.0/ ← Version-specific!                │
│  │               ├── mux.go                                     │
│  │               └── go.mod                                     │
│  │                                                              │
│  └── darwin_arm64/           ← COMPILED PACKAGES (legacy)       │
│      └── github.com/                                            │
│          └── gorilla/                                           │
│              └── mux.a       ← Compiled archive                 │
│                                                                 │
│  NOTE: pkg/mod/ is where go.sum checksums are verified!         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### src/ Directory (Legacy)

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  $GOPATH/src/ (LEGACY - Before Go Modules)                      │
│                                                                 │
│  PURPOSE: All source code lived here                            │
│                                                                 │
│  RULE: Your project path = Your import path                     │
│                                                                 │
│  If your code was at:                                           │
│  $GOPATH/src/github.com/yourusername/myproject/                 │
│                                                                 │
│  Then you imported it as:                                       │
│  import "github.com/yourusername/myproject"                     │
│                                                                 │
│  This meant:                                                    │
│  ✅ Simple mental model                                         │
│  ✅ No configuration needed                                     │
│  ❌ ALL code must be in GOPATH                                  │
│  ❌ Only ONE version of each dependency                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### The Problems with GOPATH

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GOPATH PROBLEMS (Why it failed)                                │
│                                                                 │
│  PROBLEM 1: Location Lock-in                                    │
│  ─────────────────────────────                                  │
│  ❌ ALL projects MUST live in $GOPATH/src/                      │
│  ❌ Can't have code on Desktop, Documents, anywhere else        │
│  ❌ Can't organize projects your way                            │
│                                                                 │
│  Developer: "I want my project in ~/projects/myapp"             │
│  Go: "NO! It must be in ~/go/src/github.com/you/myapp"          │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  PROBLEM 2: No Versioning                                       │
│  ─────────────────────────────                                  │
│  Scenario:                                                      │
│  • Project A needs library X version 1.0                        │
│  • Project B needs library X version 2.0                        │
│                                                                 │
│  $GOPATH/src/github.com/lib/X/  ← Only ONE version can exist!   │
│                                                                 │
│  Result: You had to choose one, breaking the other project      │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  PROBLEM 3: No Reproducible Builds                              │
│  ─────────────────────────────────                              │
│  $ go get github.com/some/package  ← Gets latest (whatever)     │
│                                                                 │
│  • Today: Gets v1.5.0                                           │
│  • Tomorrow: Gets v1.6.0 (breaking change!)                     │
│  • Your build breaks randomly                                   │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  PROBLEM 4: Vendor Folder Mess                                  │
│  ─────────────────────────────                                  │
│  Workaround: Copy dependencies into vendor/ folder              │
│                                                                 │
│  myproject/                                                     │
│  └── vendor/                                                    │
│      └── github.com/                                            │
│          └── gorilla/                                           │
│              └── mux/  ← Copied here                            │
│                                                                 │
│  Problems:                                                      │
│  • Huge repositories (vendor/ committed to git)                 │
│  • Manual updates                                               │
│  • Conflicts with tools                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### The Solution: Go Modules (Go 1.11+, Default in Go 1.13+)

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO MODULES: THE FIX                                            │
│                                                                 │
│  Introduced: Go 1.11 (2018)                                     │
│  Default: Go 1.13 (2019)                                        │
│  Mandatory: Go 1.17+ (2021)                                     │
│                                                                 │
│  HOW IT WORKS:                                                  │
│                                                                 │
│  1. Work from ANYWHERE                                          │
│     ~/Desktop/myproject/     ← Works!                           │
│     ~/Documents/code/app/    ← Works!                           │
│     /tmp/experiment/         ← Works!                           │
│                                                                 │
│  2. Add go.mod file                                             │
│     module github.com/you/myproject                             │
│     go 1.22                                                     │
│     require github.com/gorilla/mux v1.8.0  ← VERSIONED!         │
│                                                                 │
│  3. Dependencies cached in GOPATH/pkg/mod/                      │
│     ~/go/pkg/mod/github.com/gorilla/mux@v1.8.0/                 │
│     ~/go/pkg/mod/github.com/gorilla/mux@v1.7.0/                 │
│     ↑ Multiple versions can coexist!                            │
│                                                                 │
│  4. Checksums in go.sum                                         │
│     Ensures everyone gets EXACT same bytes                      │
│     Security against tampering                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Before vs After Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  BEFORE (GOPATH Era)            AFTER (Modules Era)             │
│  ─────────────────────          ─────────────────────           │
│                                                                 │
│  Project location:              Project location:               │
│  ~/go/src/github.com/you/proj   ~/anywhere/you/want/proj        │
│  ❌ Fixed                       ✅ Flexible                     │
│                                                                 │
│  Dependency versions:           Dependency versions:            │
│  Whatever is in src/            go.mod specifies exactly        │
│  ❌ One version only            ✅ Multiple versions OK         │
│                                                                 │
│  Reproducibility:               Reproducibility:                │
│  "go get" = latest              go.sum = checksums              │
│  ❌ Non-deterministic           ✅ Deterministic                │
│                                                                 │
│  Dependency storage:            Dependency storage:             │
│  $GOPATH/src/                   $GOPATH/pkg/mod/                │
│  ❌ Mixed with your code        ✅ Separate cache               │
│                                                                 │
│  Environment variable:          Environment variable:           │
│  GO111MODULE=off (implicit)     GO111MODULE=on (default)        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### GO111MODULE Explained

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO111MODULE ENVIRONMENT VARIABLE                               │
│                                                                 │
│  This controls which mode Go uses:                              │
│                                                                 │
│  GO111MODULE=off                                                │
│  ─────────────────                                              │
│  GOPATH mode (legacy)                                           │
│  • Ignores go.mod files                                         │
│  • All code must be in $GOPATH/src                              │
│  • Dependencies from $GOPATH/src                                │
│                                                                 │
│  GO111MODULE=on (DEFAULT since Go 1.16)                         │
│  ───────────────────────────────────────                        │
│  Module mode                                                    │
│  • Uses go.mod                                                  │
│  • Work from anywhere                                           │
│  • Dependencies from $GOPATH/pkg/mod                            │
│                                                                 │
│  GO111MODULE=auto                                               │
│  ─────────────────                                              │
│  Automatic detection                                            │
│  • If go.mod exists → module mode                               │
│  • If in $GOPATH/src without go.mod → GOPATH mode               │
│                                                                 │
│  CHECK YOUR SETTING:                                            │
│  $ go env GO111MODULE                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: Understanding Environment

```go
// env_explorer.go
package main

import (
    "fmt"
    "os"
    "os/exec"
    "runtime"
    "strings"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              GO ENVIRONMENT EXPLORER                      ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic info
    fmt.Printf("\n📦 Go Version: %s\n", runtime.Version())
    fmt.Printf("🖥️  OS/Arch: %s/%s\n", runtime.GOOS, runtime.GOARCH)
    
    // Key environment variables
    fmt.Println("\n🔧 Key Environment Variables:")
    fmt.Println(strings.Repeat("─", 60))
    
    envVars := []string{"GOROOT", "GOPATH", "GOBIN", "GOMODCACHE", "GO111MODULE"}
    for _, env := range envVars {
        value := getGoEnv(env)
        fmt.Printf("%-15s = %s\n", env, value)
    }
    
    // Explain what each means
    fmt.Println("\n📖 What Each Means:")
    fmt.Println(strings.Repeat("─", 60))
    fmt.Println("GOROOT      → Where Go is installed (don't modify!)")
    fmt.Println("GOPATH      → Your workspace (~go by default)")
    fmt.Println("GOBIN       → Where 'go install' puts binaries")
    fmt.Println("GOMODCACHE  → Where downloaded modules are cached")
    fmt.Println("GO111MODULE → Module mode (on=modules, off=GOPATH)")
    
    // Check module mode
    fmt.Println("\n✅ Current Mode:")
    fmt.Println(strings.Repeat("─", 60))
    moduleMode := getGoEnv("GO111MODULE")
    switch moduleMode {
    case "on", "":
        fmt.Println("🎉 Module mode (RECOMMENDED)")
        fmt.Println("   You can work from anywhere with go.mod")
    case "off":
        fmt.Println("⚠️  GOPATH mode (LEGACY)")
        fmt.Println("   Code must be in $GOPATH/src")
    case "auto":
        fmt.Println("🔄 Auto mode")
        fmt.Println("   Go decides based on go.mod presence")
    }
}

func getGoEnv(name string) string {
    cmd := exec.Command("go", "env", name)
    output, err := cmd.Output()
    if err != nil {
        return os.Getenv(name)
    }
    return strings.TrimSpace(string(output))
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              GO ENVIRONMENT EXPLORER                      ║
╚══════════════════════════════════════════════════════════╝

📦 Go Version: go1.22.0
🖥️  OS/Arch: darwin/arm64

🔧 Key Environment Variables:
────────────────────────────────────────────────────────────
GOROOT           = /usr/local/go
GOPATH           = /Users/you/go
GOBIN            =
GOMODCACHE       = /Users/you/go/pkg/mod
GO111MODULE      = on

📖 What Each Means:
────────────────────────────────────────────────────────────
GOROOT      → Where Go is installed (don't modify!)
GOPATH      → Your workspace (~go by default)
GOBIN       → Where 'go install' puts binaries
GOMODCACHE  → Where downloaded modules are cached
GO111MODULE → Module mode (on=modules, off=GOPATH)

✅ Current Mode:
────────────────────────────────────────────────────────────
🎉 Module mode (RECOMMENDED)
   You can work from anywhere with go.mod
```

*Note: Values will vary based on your system and Go installation.*

---

## 💻 Setting Up Your Editor

### VS Code (Recommended)

1. **Install VS Code**: https://code.visualstudio.com/

2. **Install Go Extension**:
   - Open VS Code
   - Press `Ctrl+Shift+X` (or `Cmd+Shift+X` on Mac)
   - Search for "Go"
   - Install the official "Go" extension by Go Team at Google

3. **Install Go Tools**:
   - Open VS Code
   - Press `Ctrl+Shift+P` (or `Cmd+Shift+P` on Mac)
   - Type "Go: Install/Update Tools"
   - Select all tools and click OK

### What Gets Installed

| Tool | Purpose | Installed To |
|------|---------|--------------|
| `gopls` | Language server | $GOPATH/bin/gopls |
| `dlv` | Debugger | $GOPATH/bin/dlv |
| `gofumpt` | Code formatter | $GOPATH/bin/gofumpt |
| `staticcheck` | Linter | $GOPATH/bin/staticcheck |
| `gotests` | Generate tests | $GOPATH/bin/gotests |

---

## ✅ Verifying Your Setup

### Complete Verification Program

```go
// verify_setup.go
package main

import (
    "fmt"
    "os"
    "os/exec"
    "path/filepath"
    "runtime"
    "strings"
)

func main() {
    fmt.Println("🔍 Go Setup Verification")
    fmt.Println(strings.Repeat("═", 50))
    
    checks := []struct {
        name  string
        check func() (bool, string)
    }{
        {"Go Installed", checkGoInstalled},
        {"Go Version", checkGoVersion},
        {"GOPATH Set", checkGoPath},
        {"GOPATH/bin in PATH", checkGoPathBin},
        {"Module Mode", checkModuleMode},
        {"Can Create Module", checkCanCreateModule},
    }
    
    allPassed := true
    for _, c := range checks {
        passed, msg := c.check()
        status := "✅"
        if !passed {
            status = "❌"
            allPassed = false
        }
        fmt.Printf("%s %s: %s\n", status, c.name, msg)
    }
    
    fmt.Println(strings.Repeat("═", 50))
    if allPassed {
        fmt.Println("🎉 All checks passed! You're ready to Go!")
    } else {
        fmt.Println("⚠️  Some checks failed. See above for details.")
    }
}

func checkGoInstalled() (bool, string) {
    _, err := exec.LookPath("go")
    if err != nil {
        return false, "go command not found in PATH"
    }
    return true, "go command found"
}

func checkGoVersion() (bool, string) {
    return true, runtime.Version()
}

func checkGoPath() (bool, string) {
    gopath := os.Getenv("GOPATH")
    if gopath == "" {
        gopath = filepath.Join(os.Getenv("HOME"), "go")
    }
    if _, err := os.Stat(gopath); os.IsNotExist(err) {
        return false, fmt.Sprintf("%s does not exist", gopath)
    }
    return true, gopath
}

func checkGoPathBin() (bool, string) {
    gopath := os.Getenv("GOPATH")
    if gopath == "" {
        gopath = filepath.Join(os.Getenv("HOME"), "go")
    }
    binPath := filepath.Join(gopath, "bin")
    
    pathEnv := os.Getenv("PATH")
    if strings.Contains(pathEnv, binPath) {
        return true, "In PATH"
    }
    return false, fmt.Sprintf("Add %s to PATH", binPath)
}

func checkModuleMode() (bool, string) {
    cmd := exec.Command("go", "env", "GO111MODULE")
    output, _ := cmd.Output()
    mode := strings.TrimSpace(string(output))
    if mode == "on" || mode == "" {
        return true, "Modules enabled (good!)"
    }
    return false, fmt.Sprintf("Mode is '%s', should be 'on'", mode)
}

func checkCanCreateModule() (bool, string) {
    // This just checks if we could create a module
    return true, "Ready to use go mod init"
}
```

**Output:**
```
🔍 Go Setup Verification
══════════════════════════════════════════════════════
✅ Go Installed: go command found
✅ Go Version: go1.22.0
✅ GOPATH Set: /Users/you/go
✅ GOPATH/bin in PATH: In PATH
✅ Module Mode: Modules enabled (good!)
✅ Can Create Module: Ready to use go mod init
══════════════════════════════════════════════════════
🎉 All checks passed! You're ready to Go!
```

*Note: Output may vary if some checks fail or paths differ on your system.*

---

## 🎯 Key Takeaways

1. **GOROOT** = Where Go is installed (bin/, src/, pkg/)
2. **GOPATH** = Your workspace (~/go by default)
3. **GOPATH has 3 directories**: bin/ (executables), pkg/ (cache), src/ (legacy source)
4. **GOPATH problems**: Location lock-in, no versioning, no reproducibility
5. **Go Modules solved everything**: Work anywhere, versioned dependencies, checksums
6. **GO111MODULE=on** is the default and recommended
7. **Add $GOPATH/bin to PATH** to use installed tools

---

## 🆚 Java Comparison: Environment Setup

| Aspect | Java | Go |
|--------|------|-----|
| Installation | JAVA_HOME, complex | Simple, one directory |
| Workspace | No fixed structure | GOPATH (legacy) or anywhere (modules) |
| Dependencies | ~/.m2/repository | ~/go/pkg/mod |
| Build tools | Maven/Gradle (separate) | Built-in (`go build`) |
| Version manager | SDKMAN, jenv | goenv, gvm (optional) |

---

## ➡️ Next Steps

Your Go environment is ready! Let's write your first real Go program.

**Next Topic:** [03 - Your First Go Program](./03-first-program.md)
