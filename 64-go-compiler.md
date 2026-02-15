# 64 - The Go Compiler

> How Go transforms source code into executable binaries.

---

## 📌 What You'll Learn

- Go compilation pipeline
- Phases of compilation
- Lexing, parsing, type checking
- SSA and optimizations
- Linking and final binary
- Cross-compilation

---

## 🔄 Compilation Pipeline Overview

| Phase | Output |
|-------|--------|
| 1. **Lexing** | Tokens (func, main, (, ), {) |
| 2. **Parsing** | AST (Abstract Syntax Tree) |
| 3. **Type Check** | Verify types, resolve names |
| 4. **IR Gen** | SSA (Static Single Assignment) |
| 5. **Optimization** | Dead code, inlining, escape analysis |
| 6. **Code Gen** | Machine code (amd64, arm64) |
| 7. **Linking** | Single executable binary |

---

## 📝 Phase 1: Lexical Analysis

```go
// Source
func main() {
    fmt.Println("Hello")
}

// Tokens: FUNC, IDENT("main"), LPAREN, RPAREN, LBRACE,
// IDENT("fmt"), PERIOD, IDENT("Println"), LPAREN, STRING("Hello"), RPAREN, RBRACE
```

---

## 🌳 Phase 2: Parsing (AST)

```go
// FuncDecl { Name: "add", Params: [...], Body: BlockStmt { ... } }
```

```bash
go build -gcflags="-W" main.go  # View AST
```

---

## ✅ Phase 3: Type Checking

- Variable types match
- Function signatures
- Interface satisfaction
- Method calls valid
- Unused variables/imports (errors in Go)

---

## 🔧 Phase 4 & 5: SSA and Optimizations

**SSA:** Each variable assigned exactly once; enables optimizations.

**Optimizations:** Dead code elimination, constant folding, inlining, escape analysis, bounds check elimination.

```bash
GOSSAFUNC=main go build main.go  # Creates ssa.html
```

---

## 💻 Phase 6: Code Generation

**Targets:** amd64, arm64, arm, 386, wasm

```bash
go build -gcflags="-S" main.go
go tool compile -S main.go
```

---

## 🔗 Phase 7: Linking

**Combines:** Your code, Go runtime, standard library, dependencies.

**Output:** Single static binary (no external dependencies).

```bash
go tool nm myapp | head -20
```

---

## 🌍 Cross-Compilation

```bash
GOOS=linux GOARCH=amd64 go build -o myapp-linux
GOOS=windows GOARCH=amd64 go build -o myapp.exe
GOOS=linux GOARCH=arm go build -o myapp-arm
GOOS=js GOARCH=wasm go build -o myapp.wasm
```

---

## 🛠️ Compiler Flags

```bash
go build -gcflags="-m"       # Escape analysis
go build -gcflags="-S"       # Assembly output
go build -gcflags="-N -l"    # Disable optimizations
go build -ldflags="-s -w"    # Strip debug info
go build -race               # Race detector
```

---

## 🆚 Go vs Java Compilation

| Java | Go |
|------|-----|
| .java → .class (bytecode) | .go → binary (machine code) |
| Needs JVM | Runs directly |
| JIT at runtime | AOT compilation |
| Platform independent | Rebuild per OS |
| Slower startup | Fast startup |

---

## 🎯 Key Takeaways

1. **Lexing** → Tokens
2. **Parsing** → AST
3. **Type Check** → Verify correctness
4. **SSA** → Intermediate representation
5. **Optimize** → Inlining, escape analysis
6. **Code Gen** → Machine code
7. **Linking** → Single static binary
8. **Cross-compile** → GOOS, GOARCH

---

## ➡️ Next Steps

**Next Topic:** [65 - go.sum Deep Dive](./65-go-sum.md)
