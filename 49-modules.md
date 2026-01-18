# 49 - Module Management

> Advanced Go modules, versioning, and dependency management.

---

## 📌 What You'll Learn

- Semantic versioning
- Module versioning (v2+)
- Replace and exclude directives
- Private modules
- Vendoring

---

## 📦 Semantic Versioning

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  SEMANTIC VERSIONING (SemVer)                                   │
│                                                                 │
│         v1.2.3                                                  │
│         │ │ │                                                   │
│         │ │ └── PATCH: Bug fixes (backward compatible)          │
│         │ └──── MINOR: New features (backward compatible)       │
│         └────── MAJOR: Breaking changes                         │
│                                                                 │
│  EXAMPLES:                                                      │
│  v1.2.3 → v1.2.4  Patch: fixed a bug                           │
│  v1.2.3 → v1.3.0  Minor: added new function                    │
│  v1.2.3 → v2.0.0  Major: changed API, breaks old code          │
│                                                                 │
│  v0.x.x = Unstable (anything can change)                        │
│  v1.x.x+ = Stable (follows SemVer)                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 go.mod Directives

```go
// go.mod example with all directives

module github.com/myorg/myapp

go 1.21

require (
    github.com/gin-gonic/gin v1.9.1
    github.com/lib/pq v1.10.9
    
    // Indirect dependencies (required by direct dependencies)
    golang.org/x/crypto v0.14.0 // indirect
)

// Replace module with local copy (development)
replace github.com/myorg/mylib => ../mylib

// Replace with different version
replace github.com/old/module => github.com/new/module v2.0.0

// Exclude problematic version
exclude github.com/problematic/module v1.5.0

// Retract broken versions (in your own module)
retract (
    v1.0.0 // Contains critical bug
    [v1.1.0, v1.2.0] // Range of bad versions
)
```

---

## 🔢 Major Version Modules (v2+)

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO IMPORT COMPATIBILITY RULE                                   │
│                                                                 │
│  Different major versions = Different modules!                  │
│                                                                 │
│  v0.x.x, v1.x.x:                                               │
│    import "github.com/user/pkg"                                │
│                                                                 │
│  v2.x.x and above:                                             │
│    import "github.com/user/pkg/v2"   ← /v2 suffix!             │
│    import "github.com/user/pkg/v3"                             │
│                                                                 │
│  WHY?                                                           │
│  • v1 and v2 can coexist in same project                       │
│  • Gradual migration possible                                   │
│  • No "dependency hell"                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```go
// go.mod for v2+ module
module github.com/myorg/mylib/v2  // Note: /v2 suffix

go 1.21

// Using v2 module
import "github.com/myorg/mylib/v2"
```

---

## 🔧 Common Module Commands

```bash
# Initialize new module
go mod init github.com/myorg/myapp

# Add dependency
go get github.com/gin-gonic/gin@latest
go get github.com/gin-gonic/gin@v1.9.1

# Update dependency
go get -u github.com/gin-gonic/gin          # Latest minor/patch
go get -u=patch github.com/gin-gonic/gin    # Latest patch only

# Update all dependencies
go get -u ./...

# Remove unused dependencies
go mod tidy

# Download dependencies to cache
go mod download

# Create vendor directory
go mod vendor

# Verify dependencies (checksum)
go mod verify

# Show dependency graph
go mod graph

# Show why dependency is needed
go mod why github.com/some/dependency

# Edit go.mod programmatically
go mod edit -require github.com/new/dep@v1.0.0
go mod edit -replace github.com/old=github.com/new@v1.0.0
go mod edit -droprequire github.com/unused/dep
```

---

## 🔒 Private Modules

```bash
# Configure Go to access private repos

# Method 1: GOPRIVATE environment variable
export GOPRIVATE=github.com/myorg/*,gitlab.mycompany.com/*

# Method 2: .gitconfig for authentication
git config --global url."https://${GITHUB_TOKEN}@github.com/".insteadOf "https://github.com/"

# Method 3: .netrc file
# ~/.netrc
machine github.com
login YOUR_USERNAME
password YOUR_TOKEN

# For Go 1.13+: GONOPROXY and GONOSUMDB
export GONOPROXY=github.com/myorg/*
export GONOSUMDB=github.com/myorg/*
```

---

## 📁 Vendoring

```bash
# Create vendor directory
go mod vendor

# Build using vendor
go build -mod=vendor

# Always use vendor (in go.mod)
# Add: go 1.21
# Vendor mode is auto-detected when vendor/ exists

# Verify vendor matches go.sum
go mod verify
```

```
myproject/
├── go.mod
├── go.sum
├── vendor/
│   ├── modules.txt          # Manifest of vendored modules
│   ├── github.com/
│   │   └── gin-gonic/
│   │       └── gin/
│   └── ...
└── main.go
```

---

## 📊 Version Selection

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MINIMAL VERSION SELECTION (MVS)                                │
│                                                                 │
│  Go picks the MINIMUM version that satisfies all requirements   │
│                                                                 │
│  Example:                                                       │
│    Your app → requires lib v1.2.0                               │
│    Dependency A → requires lib v1.3.0                           │
│    Dependency B → requires lib v1.1.0                           │
│                                                                 │
│    Go selects: lib v1.3.0 (minimum that works for all)         │
│                                                                 │
│  Unlike other systems that pick "latest compatible"             │
│  Go picks "minimum required" - more reproducible!               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **SemVer** for version numbers
2. **go mod tidy** to clean up
3. **v2+** needs `/v2` in import path
4. **GOPRIVATE** for private repos
5. **vendor/** for reproducible builds
6. **MVS** picks minimum viable version

---

## ➡️ Next Steps

**Next Topic:** [50 - Best Practices](./50-best-practices.md)

