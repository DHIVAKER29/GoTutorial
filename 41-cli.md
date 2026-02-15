# 41 - Command Line Tools

> Building CLI applications with flags, arguments, and environment variables.

---

## 📌 What You'll Learn

- Parsing command line arguments
- Using the flag package
- Environment variables
- Building professional CLIs

---

## 📝 Command Line Arguments

### os.Args Basics

```go
// os.Args contains all command line arguments
// Run: go run main.go hello world 123
package main

import (
    "fmt"
    "os"
)

func main() {
    fmt.Printf("Program name: %s\n", os.Args[0])
    fmt.Printf("All args: %v\n", os.Args)
    fmt.Printf("Arg count: %d\n", len(os.Args))
}
// Output:
// Program name: /tmp/go-build.../exe/main
// All args: [.../main hello world 123]
// Arg count: 4
```

### Iterating User Arguments

```go
package main

import "fmt"
import "os"

func main() {
    for i, arg := range os.Args[1:] {
        fmt.Printf("Arg %d: %s\n", i+1, arg)
    }
}
// Output (when run with: go run main.go hello world 123):
// Arg 1: hello
// Arg 2: world
// Arg 3: 123
```

---

## 🚩 Flag Package

### Defining Flags

```go
package main

import (
    "flag"
    "fmt"
)

func main() {
    name := flag.String("name", "World", "Name to greet")
    age := flag.Int("age", 0, "Your age")
    verbose := flag.Bool("verbose", false, "Enable verbose output")

    var email string
    flag.StringVar(&email, "email", "", "Your email address")

    flag.Parse()

    fmt.Printf("Name: %s\n", *name)
    fmt.Printf("Age: %d\n", *age)
    fmt.Printf("Verbose: %t\n", *verbose)
    fmt.Printf("Email: %s\n", email)
}
// Output (run: go run main.go -name=Alice -age=30 -verbose):
// Name: Alice
// Age: 30
// Verbose: true
// Email:
```

### Remaining Arguments After Flags

```go
package main

import (
    "flag"
    "fmt"
)

func main() {
    flag.String("name", "", "Name")
    flag.Parse()

    fmt.Printf("Args: %v\n", flag.Args())
    fmt.Printf("Count: %d\n", flag.NArg())
}
// Output (run: go run main.go -name=Alice file1.txt file2.txt):
// Args: [file1.txt file2.txt]
// Count: 2
```

---

## 🌍 Environment Variables

### Get and Lookup

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    home := os.Getenv("HOME")
    fmt.Printf("HOME: %s\n", home)

    apiKey, exists := os.LookupEnv("API_KEY")
    if exists {
        fmt.Printf("API_KEY: %s\n", apiKey)
    } else {
        fmt.Println("API_KEY: (not set)")
    }
}
// Output:
// HOME: /Users/username
// API_KEY: (not set)
```

### Set and List

```go
package main

import (
    "fmt"
    "os"
    "strings"
)

func main() {
    os.Setenv("MY_VAR", "hello")
    fmt.Printf("MY_VAR: %s\n", os.Getenv("MY_VAR"))

    for i, env := range os.Environ() {
        if i >= 3 {
            fmt.Println("...")
            break
        }
        parts := strings.SplitN(env, "=", 2)
        fmt.Printf("%s=...\n", parts[0])
    }
}
// Output:
// MY_VAR: hello
// PATH=...
// HOME=...
// USER=...
// ...
```

---

## 🏭 Production CLI Pattern

### Complete Example

```go
package main

import (
    "flag"
    "fmt"
    "os"
)

var (
    Version   = "dev"
    BuildTime = "unknown"
)

type Config struct {
    Host  string
    Port  int
    Debug bool
}

func main() {
    if len(os.Args) < 2 {
        printUsage()
        os.Exit(1)
    }

    switch os.Args[1] {
    case "serve":
        serveCmd(os.Args[2:])
    case "version":
        fmt.Printf("Version: %s\nBuild: %s\n", Version, BuildTime)
    case "help":
        printUsage()
    default:
        fmt.Printf("Unknown command: %s\n", os.Args[1])
        printUsage()
        os.Exit(1)
    }
}

func serveCmd(args []string) {
    fs := flag.NewFlagSet("serve", flag.ExitOnError)
    host := fs.String("host", getEnvOrDefault("HOST", "localhost"), "Server host")
    port := fs.Int("port", 8080, "Server port")
    debug := fs.Bool("debug", false, "Enable debug mode")
    fs.Parse(args)

    cfg := Config{Host: *host, Port: *port, Debug: *debug}
    fmt.Printf("Starting server on %s:%d (debug=%t)\n", cfg.Host, cfg.Port, cfg.Debug)
}

func getEnvOrDefault(key, defaultVal string) string {
    if val := os.Getenv(key); val != "" {
        return val
    }
    return defaultVal
}

func printUsage() {
    fmt.Println("Usage: myapp <command> [options]")
    fmt.Println("\nCommands:")
    fmt.Println("  serve     Start the server")
    fmt.Println("  version   Show version info")
    fmt.Println("  help      Show this help")
}
```

**Output** (when running `./myapp serve -port=3000 -debug`):
```
Starting server on localhost:3000 (debug=true)
```

**Output** (when running `./myapp version`):
```
Version: 1.0.0
Build: 2026-02-10T12:34:56
```

**Output** (when running `./myapp` without arguments):
```
Usage: myapp <command> [options]

Commands:
  serve     Start the server
  version   Show version info
  help      Show this help
```

**Build with version:**
```bash
go build -ldflags "-X main.Version=1.0.0 -X main.BuildTime=$(date -u +%Y-%m-%dT%H:%M:%S)" -o myapp
```

---

## 🎯 Key Takeaways

1. **os.Args** for raw arguments
2. **flag** package for parsed flags
3. **os.Getenv/LookupEnv** for env vars
4. **flag.NewFlagSet** for subcommands
5. **ldflags** to inject version at build time

---

## ➡️ Next Steps

**Next Topic:** [42 - Logging](./42-logging.md)
