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

**go.sum** = Security ledger for dependencies

| Purpose | Description |
|---------|-------------|
| Verify | Downloaded modules haven't been tampered |
| Reproducible | Lock to exact content (not just version) |
| Detect | Supply chain attacks |
| go.mod | "I need package X v1.2.3" |
| go.sum | "v1.2.3 should have THIS exact content (hash)" |

---

## 📝 go.sum Format

```
github.com/gin-gonic/gin v1.9.1 h1:4idEAncQnU5cB7BeOkPtxjfCSye0AAm1R0RVIqFPSa=
github.com/gin-gonic/gin v1.9.1/go.mod h1:hPrL76YYWbnrHYAT8A+VLU87c8Zy5C8G1zYN=
```

| Part | Meaning |
|------|---------|
| Module path | github.com/gin-gonic/gin |
| Version | v1.9.1 |
| h1 | SHA256 hash algorithm |
| Hash | Base64 encoded checksum |

**Two entries per module:** Content hash + go.mod hash

---

## 🔄 How Verification Works

1. **Download** module from proxy or source
2. **Calculate** SHA256 of content and go.mod
3. **Verify** against go.sum or sum.golang.org
4. **Match** → proceed | **Mismatch** → ERROR (possible tampering)
5. **Store** in module cache

---

## 🌐 Go Checksum Database

**sum.golang.org** - Global checksum database

- Operated by Google
- Merkle tree for integrity
- Ensures everyone gets same checksum
- Attack: modified content → hash mismatch → download refused

---

## 🛡️ Supply Chain Security

**Without go.sum:** Attacker modifies package → you download malware

**With go.sum:** Attacker modifies → hash mismatch → "SECURITY ERROR: checksum mismatch" → attack blocked

---

## 📋 go.mod vs go.sum

| go.mod | go.sum |
|--------|--------|
| Human-editable | Auto-generated |
| Direct dependencies | All dependencies |
| Minimum version | Cryptographic hashes |
| "What I need" | "Exact content verification" |

**Commit both to version control!**

---

## 🔧 Common Commands

```bash
go mod verify
go mod tidy
go mod download
go mod why github.com/some/module
go mod graph
go clean -modcache
```

---

## ⚠️ Common Issues

| Issue | Fix |
|-------|-----|
| Checksum mismatch | `go clean -modcache` then `go mod download` |
| Missing go.sum entry | `go mod tidy` |
| Private module fails | `GOPRIVATE=github.com/mycompany/*` |
| Skip checksum for private | `GONOSUMDB=github.com/mycompany/*` |

---

## 🎯 Key Takeaways

1. **go.sum** = cryptographic verification
2. **Two entries** per module (content + go.mod)
3. **sum.golang.org** = global checksum database
4. **Detects tampering** and supply chain attacks
5. **Commit to VCS** - essential for reproducible builds
6. **GOPRIVATE** for internal modules
7. **go mod verify** to check integrity

---

## 🎉 Tutorial Complete!

You've now covered **66 comprehensive chapters** on Go!

**You are fully prepared for Go development and interviews!** 🚀
