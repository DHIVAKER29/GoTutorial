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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO COMPILATION PIPELINE                                        │
│                                                                 │
│  Source Code (.go files)                                        │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────┐                                           │
│  │  1. LEXING      │  Break into tokens                        │
│  │     (Scanner)   │  "func", "main", "(", ")", "{" ...        │
│  └────────┬────────┘                                           │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                           │
│  │  2. PARSING     │  Build Abstract Syntax Tree (AST)         │
│  │     (Parser)    │  Tree structure of your code              │
│  └────────┬────────┘                                           │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                           │
│  │  3. TYPE CHECK  │  Verify types, resolve names              │
│  │                 │  Check interfaces, method calls           │
│  └────────┬────────┘                                           │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                           │
│  │  4. IR GEN      │  Generate Intermediate Representation     │
│  │     (SSA Form)  │  Static Single Assignment form            │
│  └────────┬────────┘                                           │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                           │
│  │  5. OPTIMIZATION│  Dead code elimination                    │
│  │                 │  Inlining, escape analysis                │
│  └────────┬────────┘                                           │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                           │
│  │  6. CODE GEN    │  Generate machine code                    │
│  │                 │  Platform-specific (amd64, arm64)         │
│  └────────┬────────┘                                           │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                           │
│  │  7. LINKING     │  Combine with runtime, create binary      │
│  │                 │  Resolve external symbols                 │
│  └────────┬────────┘                                           │
│           │                                                     │
│           ▼                                                     │
│      Executable Binary                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Phase 1: Lexical Analysis (Lexing)

```go
// What the lexer sees:
package main

import "fmt"

func main() {
    fmt.Println("Hello")
}

// Lexer produces tokens:
// PACKAGE, IDENT("main"), SEMICOLON
// IMPORT, STRING("fmt"), SEMICOLON
// FUNC, IDENT("main"), LPAREN, RPAREN, LBRACE
// IDENT("fmt"), PERIOD, IDENT("Println"), LPAREN, STRING("Hello"), RPAREN
// RBRACE
```

```
┌────────────────────────────────────────────────────┐
│  TOKEN TYPES                                       │
│                                                    │
│  Keywords:  package, import, func, var, const...  │
│  Identifiers: main, fmt, Println, myVar           │
│  Literals: "Hello", 42, 3.14, true                │
│  Operators: +, -, *, /, ==, !=, <, >              │
│  Delimiters: (, ), {, }, [, ], ;, ,               │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 🌳 Phase 2: Parsing (AST Generation)

```go
// Source code:
func add(a, b int) int {
    return a + b
}

// Abstract Syntax Tree (simplified):
/*
FuncDecl {
    Name: "add"
    Params: [
        Field{Names: ["a", "b"], Type: "int"}
    ]
    Results: [
        Field{Type: "int"}
    ]
    Body: BlockStmt {
        Stmts: [
            ReturnStmt {
                Results: [
                    BinaryExpr {
                        Op: "+"
                        X: Ident("a")
                        Y: Ident("b")
                    }
                ]
            }
        ]
    }
}
*/
```

```bash
# View AST of your code:
go build -gcflags="-W" main.go  # Show AST before optimizations
```

---

## ✅ Phase 3: Type Checking

```go
// Type checker verifies:

// 1. Variable types match
var x int = "hello"  // ERROR: cannot use string as int

// 2. Function signatures
func add(a, b int) int { return a + b }
add(1, "2")  // ERROR: cannot use string as int

// 3. Interface satisfaction
type Reader interface { Read([]byte) (int, error) }
var r Reader = os.Stdin  // OK: os.Stdin implements Reader

// 4. Method calls are valid
var s string
s.Foo()  // ERROR: string has no method Foo

// 5. Unused variables/imports
import "fmt"  // No usage
var x int     // No usage
// Both are errors in Go!
```

---

## 🔧 Phase 4 & 5: SSA and Optimizations

```
┌────────────────────────────────────────────────────┐
│  SSA = Static Single Assignment                    │
│                                                    │
│  Each variable assigned exactly once               │
│  Enables powerful optimizations                    │
│                                                    │
│  Original:              SSA Form:                  │
│  x = 1                  x₁ = 1                     │
│  x = x + 1              x₂ = x₁ + 1                │
│  y = x * 2              y₁ = x₂ * 2                │
│                                                    │
└────────────────────────────────────────────────────┘
```

```bash
# View SSA output:
GOSSAFUNC=main go build main.go
# Creates ssa.html - open in browser to see all SSA phases
```

**Optimizations performed:**
- **Dead code elimination** - Remove unreachable code
- **Constant folding** - `x := 2 + 3` → `x := 5`
- **Inlining** - Replace function call with function body
- **Escape analysis** - Decide stack vs heap allocation
- **Bounds check elimination** - Remove unnecessary slice checks
- **Common subexpression elimination** - Reuse computed values

---

## 💻 Phase 6: Code Generation

```
┌────────────────────────────────────────────────────┐
│  CODE GENERATION                                   │
│                                                    │
│  SSA IR → Machine Code                             │
│                                                    │
│  Target architectures:                             │
│  • amd64 (x86-64) - Intel/AMD 64-bit              │
│  • arm64 - Apple Silicon, ARM servers             │
│  • arm - Raspberry Pi, mobile                     │
│  • 386 - 32-bit x86                               │
│  • wasm - WebAssembly                             │
│                                                    │
│  Each architecture has different:                  │
│  • Registers (rax, rbx... vs x0, x1...)           │
│  • Instructions (MOV, ADD, JMP...)                │
│  • Calling conventions                             │
│                                                    │
└────────────────────────────────────────────────────┘
```

```bash
# View generated assembly:
go build -gcflags="-S" main.go

# Or use go tool:
go tool compile -S main.go
```

---

## 🔗 Phase 7: Linking

```
┌────────────────────────────────────────────────────┐
│  LINKING                                           │
│                                                    │
│  Combines:                                         │
│  • Your compiled code (.o files)                  │
│  • Go runtime (scheduler, GC, etc.)               │
│  • Standard library packages                       │
│  • External dependencies                           │
│                                                    │
│  Resolves:                                         │
│  • External symbol references                      │
│  • Runtime initialization                          │
│  • Main entry point                                │
│                                                    │
│  Output:                                           │
│  • Single static binary (no dependencies!)         │
│  • Contains everything needed to run               │
│                                                    │
└────────────────────────────────────────────────────┘
```

```bash
# See what's in your binary:
go tool nm myapp | head -20

# See size breakdown:
go tool objdump myapp | head -50
```

---

## 🌍 Cross-Compilation

```bash
# Go makes cross-compilation trivial!

# Build for Linux on Mac:
GOOS=linux GOARCH=amd64 go build -o myapp-linux

# Build for Windows on Mac:
GOOS=windows GOARCH=amd64 go build -o myapp.exe

# Build for ARM (Raspberry Pi):
GOOS=linux GOARCH=arm go build -o myapp-arm

# Build for WebAssembly:
GOOS=js GOARCH=wasm go build -o myapp.wasm

# Common GOOS values:
# linux, darwin (macOS), windows, freebsd, android, ios, js

# Common GOARCH values:
# amd64, arm64, arm, 386, wasm
```

---

## 🛠️ Compiler Flags

```bash
# Optimization flags:
go build -ldflags="-s -w" myapp  # Strip debug info (smaller binary)

# Debugging flags:
go build -gcflags="-N -l" myapp  # Disable optimizations (for debugging)

# Analysis flags:
go build -gcflags="-m" myapp     # Show escape analysis decisions
go build -gcflags="-m -m" myapp  # More verbose escape analysis
go build -gcflags="-S" myapp     # Show assembly output

# Race detector:
go build -race myapp             # Build with race detector

# Version info:
go build -ldflags="-X main.Version=1.0.0" myapp
```

---

## 🆚 Go vs Java Compilation

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│                                                                 │
│  .java → .class (bytecode)         .go → binary (machine code)  │
│  Needs JVM to run                  Runs directly on OS          │
│  JIT compilation at runtime        AOT compilation              │
│                                                                 │
│  javac → bytecode                  go build → executable        │
│  java myapp                        ./myapp                      │
│                                                                 │
│  Pros: Platform independent        Pros: No runtime needed      │
│        Runtime optimizations              Fast startup          │
│  Cons: JVM overhead                       Single binary         │
│        Slower startup              Cons: Larger binary          │
│                                          Rebuild for each OS    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Lexing** → Tokens
2. **Parsing** → AST (Abstract Syntax Tree)
3. **Type Check** → Verify correctness
4. **SSA** → Intermediate representation
5. **Optimize** → Inlining, escape analysis, dead code
6. **Code Gen** → Machine code
7. **Linking** → Single static binary
8. **Cross-compile** → `GOOS` and `GOARCH`

---

## ➡️ Next Steps

**Next Topic:** [65 - go.sum Deep Dive](./65-go-sum.md)

