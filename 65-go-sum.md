# 65 - go.sum Deep Dive

> Understanding Go's dependency verification and supply chain security.

---

## 📌 What You'll Learn

- What go.sum is and why it exists
- Checksum format and structure
- How Go verifies dependencies
- Supply chain security
- Common issues and solutions

---

## 🔐 What is go.sum?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  go.sum = SECURITY LEDGER for your dependencies                 │
│                                                                 │
│  PURPOSE:                                                       │
│  • Verify downloaded modules haven't been tampered              │
│  • Ensure reproducible builds                                   │
│  • Detect supply chain attacks                                  │
│  • Lock dependencies to exact content (not just version)        │
│                                                                 │
│  ANALOGY:                                                       │
│  • go.mod = "I need package X version 1.2.3"                   │
│  • go.sum = "Package X version 1.2.3 should have THIS exact    │
│              content (verified by cryptographic hash)"          │
│                                                                 │
│  Like a bank verifying signatures before releasing funds!       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 go.sum Format

```
# go.sum file format:
# <module> <version> <hash>
# <module> <version>/go.mod <hash>

github.com/gin-gonic/gin v1.9.1 h1:4idEAncQnU5cB7BeOkPtxjfCSye0AAm1R0RVIqFPSa=
github.com/gin-gonic/gin v1.9.1/go.mod h1:hPrL76YYWbnrHYAT8A+VLU87c8Zy5C8G1zYN=
```

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  ENTRY BREAKDOWN                                               │
│                                                                │
│  github.com/gin-gonic/gin v1.9.1 h1:4idEAnc...=                │
│  │                        │      │  │                         │
│  │                        │      │  └─ Base64 encoded hash    │
│  │                        │      └─ Hash algorithm (h1=SHA256)│
│  │                        └─ Version                          │
│  └─ Module path                                                │
│                                                                │
│  TWO ENTRIES PER MODULE:                                       │
│  1. v1.9.1      → Hash of entire module content               │
│  2. v1.9.1/go.mod → Hash of just the go.mod file              │
│                                                                │
│  Why two? go.mod can be fetched separately (faster resolution)│
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 🔄 How Verification Works

```go
// When you run: go get github.com/gin-gonic/gin@v1.9.1

/*
STEP 1: Download module
┌───────────────────────────────────────────┐
│  Go downloads gin v1.9.1 from            │
│  proxy.golang.org or direct source       │
└───────────────────────────────────────────┘
           │
           ▼
STEP 2: Calculate hash
┌───────────────────────────────────────────┐
│  Go calculates SHA256 of:                 │
│  • All files in module                    │
│  • go.mod file separately                 │
└───────────────────────────────────────────┘
           │
           ▼
STEP 3: Verify against go.sum
┌───────────────────────────────────────────┐
│  If go.sum exists:                        │
│    Compare calculated hash vs stored hash │
│    MATCH → proceed                        │
│    MISMATCH → ERROR (possible tampering!) │
│                                           │
│  If go.sum doesn't have entry:            │
│    Query checksum database (sum.golang.org)│
│    Add new entry to go.sum                │
└───────────────────────────────────────────┘
           │
           ▼
STEP 4: Use module
┌───────────────────────────────────────────┐
│  Store in module cache                    │
│  Use for compilation                      │
└───────────────────────────────────────────┘
*/
```

---

## 🌐 Go Checksum Database

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  sum.golang.org = Global Checksum Database                      │
│                                                                 │
│  • Operated by Google                                           │
│  • Stores checksums for all public modules                      │
│  • Uses Merkle tree (like blockchain) for integrity             │
│  • Ensures everyone gets the same checksum                      │
│                                                                 │
│  SECURITY GUARANTEE:                                            │
│  If attacker replaces module content:                           │
│  • Their hash won't match database hash                         │
│  • Go will refuse to download                                   │
│  • Attack detected!                                             │
│                                                                 │
│  FLOW:                                                          │
│  1. You: go get github.com/pkg/errors                          │
│  2. Go: Check sum.golang.org for official hash                 │
│  3. Go: Download module, calculate local hash                   │
│  4. Go: Compare → Must match!                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛡️ Supply Chain Security

```go
// ATTACK SCENARIO (without go.sum):
/*
1. Attacker gains access to github.com/popular/package
2. Attacker modifies v1.5.0 to include malware
3. You run: go get github.com/popular/package@v1.5.0
4. Malware installed in your project!
*/

// WITH go.sum PROTECTION:
/*
1. Attacker modifies v1.5.0
2. You run: go get github.com/popular/package@v1.5.0
3. Go calculates hash of downloaded code
4. Hash doesn't match go.sum or checksum database
5. ERROR: "SECURITY ERROR: checksum mismatch"
6. Attack BLOCKED!
*/
```

```bash
# Example error message:
verifying github.com/popular/package@v1.5.0: checksum mismatch
    downloaded: h1:AAAA...
    go.sum:     h1:BBBB...

SECURITY ERROR
This download does NOT match an earlier download recorded in go.sum.
This could indicate:
- the module was modified since first download
- a malicious module was substituted
```

---

## 📋 go.mod vs go.sum

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  go.mod                           go.sum                        │
│  ──────                           ──────                        │
│                                                                 │
│  • Human-editable                 • Auto-generated              │
│  • Lists direct dependencies      • Lists ALL dependencies      │
│  • Specifies minimum version      • Cryptographic hashes        │
│  • "What I need"                  • "Exact content verification"│
│                                                                 │
│  module myapp                     github.com/pkg/a v1.0.0 h1:xx │
│                                   github.com/pkg/a v1.0.0/go.mod│
│  require (                        github.com/pkg/b v2.0.0 h1:yy │
│    github.com/pkg/a v1.0.0        github.com/pkg/b v2.0.0/go.mod│
│  )                                github.com/pkg/c v1.5.0 h1:zz │
│                                   ... (transitive deps too)     │
│                                                                 │
│  COMMIT BOTH to version control!                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Common Commands

```bash
# Verify all dependencies match go.sum
go mod verify
# Output: all modules verified

# Tidy up (add missing, remove unused)
go mod tidy

# Download dependencies (populates cache)
go mod download

# Show why a module is needed
go mod why github.com/some/module

# View module graph
go mod graph

# Clear module cache
go clean -modcache
```

---

## ⚠️ Common Issues

```bash
# Issue 1: Checksum mismatch
# Cause: Module content changed, or corrupted download
# Fix:
go clean -modcache
go mod download

# Issue 2: Missing go.sum entry
# Cause: New dependency not in go.sum
# Fix:
go mod tidy

# Issue 3: go.sum has entries not in go.mod
# Cause: Removed dependency but go.sum not cleaned
# Fix:
go mod tidy

# Issue 4: Private module checksum fails
# Cause: Private modules aren't in public checksum DB
# Fix:
export GOPRIVATE=github.com/mycompany/*
# or
export GONOSUMDB=github.com/mycompany/*
```

---

## 🏢 Private Modules

```bash
# For private/internal modules, skip checksum database:

# Option 1: GOPRIVATE (skip proxy AND checksum)
export GOPRIVATE=github.com/mycompany/*

# Option 2: GONOSUMDB (skip only checksum)
export GONOSUMDB=github.com/mycompany/*

# Option 3: GONOPROXY (skip only proxy)
export GONOPROXY=github.com/mycompany/*

# Multiple patterns:
export GOPRIVATE=github.com/mycompany/*,gitlab.internal.com/*
```

---

## 📊 go.sum Best Practices

```
✅ DO:
• Commit go.sum to version control
• Run go mod tidy before commits
• Run go mod verify in CI/CD
• Review go.sum changes in PRs

❌ DON'T:
• Delete go.sum (security risk!)
• Ignore checksum errors
• Edit go.sum manually
• Skip verification for speed
```

---

## 🎯 Key Takeaways

1. **go.sum** = cryptographic verification of dependencies
2. **Two entries per module** (content + go.mod)
3. **sum.golang.org** = global checksum database
4. **Detects tampering** and supply chain attacks
5. **Commit to VCS** - essential for reproducible builds
6. **GOPRIVATE** for internal modules
7. **go mod verify** to check integrity

---

## 🎉 Tutorial Complete!

You've now covered **66 comprehensive chapters** on Go!

This tutorial now covers:
- ✅ All language features
- ✅ Concurrency deep dive
- ✅ Standard library
- ✅ Production patterns
- ✅ Go internals (memory, GC, interfaces)
- ✅ Compiler pipeline
- ✅ Dependency management & security
- ✅ Interview questions

**You are fully prepared for Go development and interviews!** 🚀

