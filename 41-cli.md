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

```go
// cli_args.go
package main

import (
    "fmt"
    "os"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           COMMAND LINE ARGUMENTS                          ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // os.Args contains all command line arguments
    fmt.Println("\n📊 os.Args:")
    fmt.Printf("   Program name: %s\n", os.Args[0])
    fmt.Printf("   All args: %v\n", os.Args)
    fmt.Printf("   Arg count: %d\n", len(os.Args))
    
    // Arguments (skip program name)
    if len(os.Args) > 1 {
        fmt.Println("\n📊 User Arguments:")
        for i, arg := range os.Args[1:] {
            fmt.Printf("   Arg %d: %s\n", i+1, arg)
        }
    }
}

/*
Run with:
  go run cli_args.go hello world 123

Output:
  Program name: /tmp/go-build.../exe/cli_args
  All args: [.../cli_args hello world 123]
  User Arguments:
    Arg 1: hello
    Arg 2: world
    Arg 3: 123
*/
```

---

## 🚩 Flag Package

```go
// cli_flags.go
package main

import (
    "flag"
    "fmt"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           FLAG PACKAGE                                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Define flags
    name := flag.String("name", "World", "Name to greet")
    age := flag.Int("age", 0, "Your age")
    verbose := flag.Bool("verbose", false, "Enable verbose output")
    
    // Pointer version
    var email string
    flag.StringVar(&email, "email", "", "Your email address")
    
    // Parse command line
    flag.Parse()
    
    // Use flags
    fmt.Println("\n📊 Parsed Flags:")
    fmt.Printf("   Name:    %s\n", *name)
    fmt.Printf("   Age:     %d\n", *age)
    fmt.Printf("   Verbose: %t\n", *verbose)
    fmt.Printf("   Email:   %s\n", email)
    
    // Remaining arguments (after flags)
    fmt.Println("\n📊 Remaining Args:")
    fmt.Printf("   Args: %v\n", flag.Args())
    fmt.Printf("   Count: %d\n", flag.NArg())
    
    if *verbose {
        fmt.Println("\n📊 Verbose mode enabled!")
    }
}

/*
Run with:
  go run cli_flags.go -name=Alice -age=30 -verbose file1.txt file2.txt
  go run cli_flags.go --help

Output:
  Parsed Flags:
    Name:    Alice
    Age:     30
    Verbose: true
  Remaining Args:
    Args: [file1.txt file2.txt]
*/
```

---

## 🌍 Environment Variables

```go
// cli_env.go
package main

import (
    "fmt"
    "os"
    "strings"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           ENVIRONMENT VARIABLES                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Get single variable
    fmt.Println("\n📊 Get Variable:")
    home := os.Getenv("HOME")
    fmt.Printf("   HOME: %s\n", home)
    
    // Check if exists
    fmt.Println("\n📊 Check Existence:")
    apiKey, exists := os.LookupEnv("API_KEY")
    if exists {
        fmt.Printf("   API_KEY: %s\n", apiKey)
    } else {
        fmt.Println("   API_KEY: (not set)")
    }
    
    // Set variable
    fmt.Println("\n📊 Set Variable:")
    os.Setenv("MY_VAR", "hello")
    fmt.Printf("   MY_VAR: %s\n", os.Getenv("MY_VAR"))
    
    // All environment variables
    fmt.Println("\n📊 All Variables (first 5):")
    for i, env := range os.Environ() {
        if i >= 5 {
            fmt.Println("   ...")
            break
        }
        parts := strings.SplitN(env, "=", 2)
        fmt.Printf("   %s=%s...\n", parts[0], truncate(parts[1], 20))
    }
}

func truncate(s string, n int) string {
    if len(s) <= n {
        return s
    }
    return s[:n]
}
```

---

## 🏭 Production CLI Pattern

```go
// cli_production.go
package main

import (
    "flag"
    "fmt"
    "os"
)

// Version info (set at build time)
var (
    Version   = "dev"
    BuildTime = "unknown"
)

// Config from flags and env
type Config struct {
    Host      string
    Port      int
    Debug     bool
    LogLevel  string
}

func main() {
    // Subcommands
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
    
    // Flags with env fallback
    host := fs.String("host", getEnvOrDefault("HOST", "localhost"), "Server host")
    port := fs.Int("port", 8080, "Server port")
    debug := fs.Bool("debug", false, "Enable debug mode")
    
    fs.Parse(args)
    
    cfg := Config{
        Host:  *host,
        Port:  *port,
        Debug: *debug,
    }
    
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

/*
Build with version:
  go build -ldflags "-X main.Version=1.0.0 -X main.BuildTime=$(date -u +%Y-%m-%dT%H:%M:%S)" -o myapp

Run:
  ./myapp serve -port=3000 -debug
  ./myapp version
*/
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

