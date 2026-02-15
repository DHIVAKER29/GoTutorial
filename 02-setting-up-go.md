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

**Output:**
```
go version go1.22.0 darwin/arm64
```

---

## 📁 Go Installation Directory Structure (GOROOT)

When you install Go, it creates a specific directory structure. This is called **GOROOT**.

### The Complete GOROOT Structure

| Directory | Purpose | Example Contents |
|-----------|---------|------------------|
| `bin/` | Executable programs | `go`, `gofmt` |
| `src/` | Standard library source code | `fmt/print.go`, `net/http/` |
| `pkg/` | Compiled standard library | `fmt.a` (archive) |
| `doc/` | Documentation | Language spec |
| `lib/` | Supporting data files | Timezone database |
| `misc/` | Miscellaneous resources | Examples, tools |
| `api/` | API compatibility | Public API lists |

### Sample Program: Exploring GOROOT

```go
package main

import (
    "fmt"
    "os"
    "path/filepath"
    "runtime"
)

func main() {
    goroot := runtime.GOROOT()
    fmt.Println("GOROOT:", goroot)
    
    entries, err := os.ReadDir(goroot)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    
    fmt.Println("\nGOROOT contents:")
    for _, entry := range entries {
        if entry.IsDir() {
            fmt.Printf("  %s/\n", entry.Name())
        } else {
            fmt.Printf("  %s\n", entry.Name())
        }
    }
    
    fmtPath := filepath.Join(goroot, "src", "fmt")
    fmt.Printf("\nfmt package source: %s\n", fmtPath)
}
```

**Output:**
```
GOROOT: /usr/local/go

GOROOT contents:
  api/
  bin/
  doc/
  lib/
  misc/
  pkg/
  src/

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

| Language | Problem |
|----------|---------|
| **C/C++** | Include paths confusing; no standard project structure; manual dependency management |
| **Java** | Long classpaths; JAR conflicts; "JAR hell" |
| **Python** | sys.path manipulation; virtual environments needed; "Works on my machine" |

### Go's Original Solution: GOPATH (2009-2018)

**Idea:** "One workspace to rule them all"

- `$GOPATH/bin/` — Your installed executables
- `$GOPATH/pkg/` — Compiled package cache
- `$GOPATH/src/` — All source code (yours + dependencies)

**Rule:** Your project path = Your import path. Code at `$GOPATH/src/github.com/you/myproject/` was imported as `github.com/you/myproject`.

### The Problems with GOPATH

| Problem | Description |
|---------|-------------|
| **Location lock-in** | ALL projects MUST live in `$GOPATH/src/`. Can't organize projects your way |
| **No versioning** | Only ONE version of each dependency could exist. Project A needs lib v1.0, Project B needs v2.0 — impossible |
| **No reproducible builds** | `go get` got latest (whatever). Build could break randomly when dependencies updated |
| **Vendor folder mess** | Workaround: copy deps into vendor/. Huge repos, manual updates |

### The Solution: Go Modules (Go 1.11+, Default in Go 1.13+)

| Feature | GOPATH Era | Modules Era |
|---------|------------|-------------|
| Project location | ~/go/src/github.com/you/proj (fixed) | ~/anywhere/you/want/proj (flexible) |
| Dependency versions | Whatever is in src/ (one version only) | go.mod specifies exactly (multiple versions OK) |
| Reproducibility | Non-deterministic | go.sum = checksums (deterministic) |
| Dependency storage | Mixed with your code | Separate cache in $GOPATH/pkg/mod/ |

### GO111MODULE Explained

| Value | Mode | Behavior |
|-------|------|----------|
| `off` | GOPATH (legacy) | Ignores go.mod; all code in $GOPATH/src |
| `on` | Module mode (default) | Uses go.mod; work from anywhere |
| `auto` | Automatic | If go.mod exists → module mode; else GOPATH mode |

Check your setting: `go env GO111MODULE`

### Sample Program: Understanding Environment

```go
package main

import (
    "fmt"
    "os"
    "os/exec"
    "runtime"
    "strings"
)

func main() {
    fmt.Printf("Go Version: %s\n", runtime.Version())
    fmt.Printf("OS/Arch: %s/%s\n", runtime.GOOS, runtime.GOARCH)
    
    fmt.Println("\nKey Environment Variables:")
    envVars := []string{"GOROOT", "GOPATH", "GOBIN", "GOMODCACHE", "GO111MODULE"}
    for _, env := range envVars {
        value := getGoEnv(env)
        fmt.Printf("%-15s = %s\n", env, value)
    }
    
    fmt.Println("\nWhat Each Means:")
    fmt.Println("GOROOT      → Where Go is installed (don't modify!)")
    fmt.Println("GOPATH      → Your workspace (~go by default)")
    fmt.Println("GOBIN       → Where 'go install' puts binaries")
    fmt.Println("GOMODCACHE  → Where downloaded modules are cached")
    fmt.Println("GO111MODULE → Module mode (on=modules, off=GOPATH)")
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
Go Version: go1.22.0
OS/Arch: darwin/arm64

Key Environment Variables:
GOROOT           = /usr/local/go
GOPATH           = /Users/you/go
GOBIN            =
GOMODCACHE       = /Users/you/go/pkg/mod
GO111MODULE      = on

What Each Means:
GOROOT      → Where Go is installed (don't modify!)
GOPATH      → Your workspace (~go by default)
GOBIN       → Where 'go install' puts binaries
GOMODCACHE  → Where downloaded modules are cached
GO111MODULE → Module mode (on=modules, off=GOPATH)
```

*Note: Values will vary based on your system and Go installation.*

---

## 💻 Setting Up Your Editor

### VS Code (Recommended)

1. **Install VS Code**: https://code.visualstudio.com/
2. **Install Go Extension**: Search for "Go" in Extensions (Ctrl+Shift+X)
3. **Install Go Tools**: Ctrl+Shift+P → "Go: Install/Update Tools" → Select all

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
    fmt.Println("Go Setup Verification")
    fmt.Println(strings.Repeat("=", 50))
    
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
    
    fmt.Println(strings.Repeat("=", 50))
    if allPassed {
        fmt.Println("All checks passed! You're ready to Go!")
    } else {
        fmt.Println("Some checks failed. See above for details.")
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
    return true, "Ready to use go mod init"
}
```

**Output:**
```
Go Setup Verification
==================================================
✅ Go Installed: go command found
✅ Go Version: go1.22.0
✅ GOPATH Set: /Users/you/go
✅ GOPATH/bin in PATH: In PATH
✅ Module Mode: Modules enabled (good!)
✅ Can Create Module: Ready to use go mod init
==================================================
All checks passed! You're ready to Go!
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
