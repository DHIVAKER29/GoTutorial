# 51 - The init() Function

> Understanding Go's special initialization function.

---

## 📌 What You'll Learn

- What init() is and when it runs
- Multiple init() functions
- Initialization order
- Common use cases
- Init vs constructors (Java comparison)

---

## 🤔 What Is init()?

**init()** = Special function that runs automatically before main() starts

**Execution order:**
1. Import dependencies (recursively)
2. Initialize package-level variables
3. Run init() functions (in order of appearance)
4. Run main() (only in main package)

---

## 📝 Basic init() Usage

```go
package main

import "fmt"

var config = loadConfig()

func loadConfig() string {
    fmt.Println("1. Loading config...")
    return "config-loaded"
}

func init() {
    fmt.Println("2. init() running...")
    fmt.Printf("   Config: %s\n", config)
}

func main() {
    fmt.Println("3. main() running...")
}
// Output:
// 1. Loading config...
// 2. init() running...
//    Config: config-loaded
// 3. main() running...
```

---

## 🔢 Multiple init() Functions

```go
func init() { fmt.Println("First init()") }
func init() { fmt.Println("Second init()") }
func init() { fmt.Println("Third init()") }

func main() { fmt.Println("main()") }
// Output: First init(), Second init(), Third init(), main()
```

---

## 📦 Cross-Package Initialization

```go
// database/db.go
func init() {
    fmt.Println("database: init() - connecting...")
    Connection = "db-connected"
}

// cache/cache.go (imports database)
func init() {
    fmt.Println("cache: init() - initializing...")
    Client = "cache-ready-" + database.Connection
}

// main.go (imports cache)
func main() {
    fmt.Printf("Cache client: %s\n", cache.Client)
}
// Output: database init → cache init → main init → main
```

---

## 🎯 Common Use Cases

```go
// 1. Register drivers (side-effect import)
import _ "github.com/go-sql-driver/mysql"

// 2. Compile regex patterns once
var emailPattern *regexp.Regexp
func init() {
    emailPattern = regexp.MustCompile(`^[\w.]+@[\w.]+\.\w+$`)
}

// 3. Validate configuration
func init() {
    if os.Getenv("API_KEY") == "" {
        log.Fatal("API_KEY required")
    }
}

// 4. Setup logging
func init() {
    log.SetFlags(log.Ldate | log.Ltime | log.Lshortfile)
}
```

---

## 🔄 Side-Effect Imports

```go
import (
    _ "github.com/go-sql-driver/mysql"  // Registers driver
    _ "net/http/pprof"                  // Registers pprof handlers
    _ "image/png"                       // Registers PNG decoder
)
// _ prevents "unused import" error; init() runs for registration
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `static { config = loadConfig(); }` | `func init() { config = loadConfig() }` |
| Explicit class loading | Automatic package initialization |
| Static blocks per class | init() per package (multiple allowed) |
| `getInstance()` singleton | init() or lazy init with sync.Once |

---

## ⚠️ init() Best Practices

**DO:** One-time setup, register drivers, validate config, compile regex once.

**DON'T:** Long-running operations, start goroutines, complex business logic, depend on init order across packages.

---

## 🧪 Testing with init()

```go
// Extract logic to callable function
func init() {
    if err := Initialize(); err != nil {
        log.Fatal(err)
    }
}
func Initialize() error { return nil }  // Testable

// Or use lazy initialization
var dbOnce sync.Once
func GetDB() *sql.DB {
    dbOnce.Do(func() { db, _ = sql.Open(...) })
    return db
}
```

---

## 🎯 Key Takeaways

1. **init() runs automatically** before main()
2. **Multiple init()** functions allowed per package
3. **Order**: imports → variables → init() → main()
4. **Side-effect imports** use `_` identifier
5. **Keep init() simple** - validation, registration only
6. **Don't use for** complex logic or goroutines
7. **Hard to test** - prefer explicit initialization

---

## ➡️ Next Steps

**Next Topic:** [52 - io.Reader and io.Writer Interfaces](./52-io-interfaces.md)
