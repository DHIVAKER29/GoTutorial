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

| Part | Meaning | Example |
|------|---------|---------|
| **MAJOR** | Breaking changes | v1.2.3 → v2.0.0 |
| **MINOR** | New features (backward compatible) | v1.2.3 → v1.3.0 |
| **PATCH** | Bug fixes | v1.2.3 → v1.2.4 |

- **v0.x.x** = Unstable
- **v1.x.x+** = Stable (follows SemVer)

---

## 📝 go.mod Directives

```go
module github.com/myorg/myapp
go 1.21

require (
    github.com/gin-gonic/gin v1.9.1
    golang.org/x/crypto v0.14.0 // indirect
)

replace github.com/myorg/mylib => ../mylib
exclude github.com/problematic/module v1.5.0
retract v1.0.0
```

---

## 🔢 Major Version Modules (v2+)

| Version | Import path |
|---------|-------------|
| v0.x.x, v1.x.x | `import "github.com/user/pkg"` |
| v2.x.x+ | `import "github.com/user/pkg/v2"` |

**Why:** v1 and v2 can coexist; gradual migration; no dependency hell.

```go
// go.mod for v2+ module
module github.com/myorg/mylib/v2
```

---

## 🔧 Common Module Commands

```bash
go mod init github.com/myorg/myapp
go get github.com/gin-gonic/gin@latest
go get -u github.com/gin-gonic/gin
go mod tidy
go mod download
go mod vendor
go mod verify
go mod graph
go mod why github.com/some/dependency
```

---

## 🔒 Private Modules

```bash
export GOPRIVATE=github.com/myorg/*,gitlab.mycompany.com/*
export GONOPROXY=github.com/myorg/*
export GONOSUMDB=github.com/myorg/*
```

---

## 📁 Vendoring

```bash
go mod vendor
go build -mod=vendor
```

---

## 📊 Version Selection (MVS)

**Minimal Version Selection:** Go picks the **minimum** version that satisfies all requirements.

| Your app | Dep A | Dep B | Go selects |
|----------|-------|-------|------------|
| lib v1.2.0 | lib v1.3.0 | lib v1.1.0 | lib v1.3.0 |

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
