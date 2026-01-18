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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  init() = Special function that runs AUTOMATICALLY              │
│           before main() starts                                  │
│                                                                 │
│  EXECUTION ORDER:                                               │
│                                                                 │
│  1. Import dependencies (recursively)                           │
│  2. Initialize package-level variables                          │
│  3. Run init() functions (in order of appearance)               │
│  4. Run main() (only in main package)                           │
│                                                                 │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐           │
│  │  pkg A      │   │  pkg B      │   │  main       │           │
│  │  ────────   │   │  ────────   │   │  ────────   │           │
│  │  1. vars    │──►│  1. vars    │──►│  1. vars    │           │
│  │  2. init()  │   │  2. init()  │   │  2. init()  │           │
│  └─────────────┘   └─────────────┘   │  3. main()  │           │
│                                       └─────────────┘           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Basic init() Usage

```go
// init_basic.go
package main

import "fmt"

// Package-level variables (initialized first)
var config = loadConfig()

func loadConfig() string {
    fmt.Println("1. Loading config...")
    return "config-loaded"
}

// init() runs after variables, before main()
func init() {
    fmt.Println("2. init() running...")
    fmt.Printf("   Config: %s\n", config)
}

func main() {
    fmt.Println("3. main() running...")
}

/*
Output:
    1. Loading config...
    2. init() running...
       Config: config-loaded
    3. main() running...
*/
```

---

## 🔢 Multiple init() Functions

```go
// multiple_init.go
package main

import "fmt"

// You can have MULTIPLE init() functions!
// They run in order of appearance in the file

func init() {
    fmt.Println("First init()")
}

func init() {
    fmt.Println("Second init()")
}

func init() {
    fmt.Println("Third init()")
}

func main() {
    fmt.Println("main()")
}

/*
Output:
    First init()
    Second init()
    Third init()
    main()
*/
```

---

## 📦 Cross-Package Initialization

```go
// ========== pkg/database/db.go ==========
package database

import "fmt"

var Connection string

func init() {
    fmt.Println("database: init() - connecting...")
    Connection = "db-connected"
}

// ========== pkg/cache/cache.go ==========
package cache

import (
    "fmt"
    "myapp/pkg/database"  // Import triggers database init first
)

var Client string

func init() {
    fmt.Println("cache: init() - initializing...")
    // database.Connection is already available!
    Client = "cache-ready-" + database.Connection
}

// ========== main.go ==========
package main

import (
    "fmt"
    "myapp/pkg/cache"     // Imports cache, which imports database
)

func init() {
    fmt.Println("main: init()")
}

func main() {
    fmt.Println("main: main()")
    fmt.Printf("Cache client: %s\n", cache.Client)
}

/*
Output:
    database: init() - connecting...
    cache: init() - initializing...
    main: init()
    main: main()
    Cache client: cache-ready-db-connected
*/
```

---

## 🎯 Common Use Cases

```go
// use_cases.go
package main

import (
    "database/sql"
    "log"
    "os"
    "regexp"
    
    _ "github.com/go-sql-driver/mysql"  // Side-effect import!
)

// 1. REGISTER DRIVERS (most common!)
// The _ import runs mysql driver's init() to register itself
// with database/sql package

// 2. COMPILE REGEX PATTERNS
var emailPattern *regexp.Regexp

func init() {
    // Compile once at startup, not every call
    emailPattern = regexp.MustCompile(`^[\w.]+@[\w.]+\.\w+$`)
}

// 3. VALIDATE CONFIGURATION
var requiredEnvVars = []string{"DATABASE_URL", "API_KEY"}

func init() {
    for _, env := range requiredEnvVars {
        if os.Getenv(env) == "" {
            log.Fatalf("Required environment variable %s not set", env)
        }
    }
}

// 4. INITIALIZE SINGLETONS
var db *sql.DB

func init() {
    var err error
    db, err = sql.Open("mysql", os.Getenv("DATABASE_URL"))
    if err != nil {
        log.Fatalf("Failed to connect to database: %v", err)
    }
}

// 5. SETUP LOGGING
func init() {
    log.SetFlags(log.Ldate | log.Ltime | log.Lshortfile)
    log.SetPrefix("[MYAPP] ")
}
```

---

## 🔄 Side-Effect Imports

```go
// The blank identifier import pattern
import (
    _ "github.com/go-sql-driver/mysql"  // Just for init()
    _ "net/http/pprof"                  // Registers pprof handlers
    _ "image/png"                       // Registers PNG decoder
)

/*
WHY?
- These packages have init() that registers something
- We don't call any functions from them directly
- The _ prevents "unused import" error

COMMON SIDE-EFFECT IMPORTS:
- Database drivers (mysql, postgres, sqlite)
- Image format decoders (png, jpeg, gif)
- Profiling (net/http/pprof)
*/
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│                                                                 │
│  // Static initializer block       // init() function           │
│  static {                          func init() {                │
│      config = loadConfig();            config = loadConfig()    │
│  }                                 }                            │
│                                                                 │
│  // Static factory                 // No equivalent needed      │
│  public static MyClass             // init() runs automatically │
│      getInstance() {                                            │
│      if (instance == null)                                      │
│          instance = new MyClass();                              │
│      return instance;                                           │
│  }                                                              │
│                                                                 │
│  DIFFERENCES:                                                   │
│  • Java: explicit class loading                                 │
│  • Go: automatic package initialization                         │
│  • Java: static blocks per class                                │
│  • Go: init() per package (multiple allowed)                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚠️ init() Best Practices

```go
// DO's and DON'Ts

// ✅ DO: Use for one-time setup
func init() {
    // Compile regex once
    pattern = regexp.MustCompile(`...`)
}

// ✅ DO: Register drivers/handlers
import _ "github.com/lib/pq"  // Register postgres driver

// ✅ DO: Validate required config
func init() {
    if os.Getenv("API_KEY") == "" {
        log.Fatal("API_KEY required")
    }
}

// ❌ DON'T: Long-running operations
func init() {
    // BAD: Blocks program startup
    time.Sleep(10 * time.Second)
}

// ❌ DON'T: Start goroutines that never stop
func init() {
    // BAD: Hard to test and control
    go infiniteWorker()
}

// ❌ DON'T: Complex business logic
func init() {
    // BAD: Hard to test
    processAllOrders()
}

// ❌ DON'T: Depend on init order across packages
func init() {
    // BAD: Fragile, order may change
    otherPkg.SomeGlobal = "value"
}
```

---

## 🧪 Testing with init()

```go
// The problem: init() runs before tests too!

// Solution 1: Use a flag
var skipInit = false

func init() {
    if skipInit {
        return
    }
    // Normal init logic
}

// Solution 2: Extract logic to callable function
func init() {
    if err := Initialize(); err != nil {
        log.Fatal(err)
    }
}

func Initialize() error {
    // Testable initialization logic
    return nil
}

// Solution 3: Lazy initialization
var db *sql.DB
var dbOnce sync.Once

func GetDB() *sql.DB {
    dbOnce.Do(func() {
        db, _ = sql.Open(...)
    })
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

