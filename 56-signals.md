# 56 - Signal Handling and External Commands

> Handling OS signals and running external commands.

---

## 📌 What You'll Learn

- Handling SIGINT, SIGTERM
- Graceful shutdown patterns
- Running external commands
- Capturing command output

---

## 🚦 Signal Handling

```go
// signal_handling.go
package main

import (
    "context"
    "fmt"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Signal Handling                                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 Common Signals:")
    fmt.Println("   SIGINT  (2)  - Ctrl+C")
    fmt.Println("   SIGTERM (15) - kill command, container stop")
    fmt.Println("   SIGKILL (9)  - Cannot be caught! Immediate kill")
    fmt.Println("   SIGHUP  (1)  - Terminal hangup")
    
    // Create channel for signals
    sigChan := make(chan os.Signal, 1)
    
    // Notify on SIGINT and SIGTERM
    signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
    
    fmt.Println("\n   Press Ctrl+C to trigger SIGINT...")
    
    // Wait for signal
    sig := <-sigChan
    fmt.Printf("\n   Received signal: %v\n", sig)
    
    // Cleanup
    fmt.Println("   Cleaning up...")
    time.Sleep(1 * time.Second)
    fmt.Println("   Goodbye!")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Signal Handling                                 ║
╚══════════════════════════════════════════════════════════╝

📊 Common Signals:
   SIGINT  (2)  - Ctrl+C
   SIGTERM (15) - kill command, container stop
   SIGKILL (9)  - Cannot be caught! Immediate kill
   SIGHUP  (1)  - Terminal hangup

   Press Ctrl+C to trigger SIGINT...

   (Program blocks here until user presses Ctrl+C or sends SIGTERM)
   Received signal: interrupt
   Cleaning up...
   Goodbye!
```

---

## 🏭 Graceful Shutdown Pattern

```go
// graceful_shutdown.go
package main

import (
    "context"
    "fmt"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Graceful Shutdown                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Create server
    server := &http.Server{
        Addr: ":8080",
        Handler: http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            time.Sleep(2 * time.Second)  // Simulate work
            fmt.Fprintf(w, "Hello!")
        }),
    }
    
    // Start server in goroutine
    go func() {
        fmt.Println("   Server starting on :8080")
        if err := server.ListenAndServe(); err != http.ErrServerClosed {
            fmt.Printf("   Server error: %v\n", err)
        }
    }()
    
    // Wait for shutdown signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    
    fmt.Println("\n   Shutting down gracefully...")
    
    // Create shutdown context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    // Shutdown server (waits for active connections)
    if err := server.Shutdown(ctx); err != nil {
        fmt.Printf("   Shutdown error: %v\n", err)
    }
    
    fmt.Println("   Server stopped")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Graceful Shutdown                               ║
╚══════════════════════════════════════════════════════════╝

   Server starting on :8080

   (Program blocks until user presses Ctrl+C or sends SIGTERM)
   Shutting down gracefully...
   Server stopped
```

---

## 💻 Running External Commands

```go
// exec_commands.go
package main

import (
    "bytes"
    "context"
    "fmt"
    "os/exec"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Running External Commands                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Simple command
    fmt.Println("\n📊 Simple Command:")
    out, err := exec.Command("echo", "Hello from Go!").Output()
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    }
    fmt.Printf("   Output: %s", out)
    
    // Command with output
    fmt.Println("\n📊 List Files:")
    out, _ = exec.Command("ls", "-la").Output()
    fmt.Printf("%s", out[:min(200, len(out))])
    fmt.Println("   ...")
    
    // Capture stdout and stderr separately
    fmt.Println("\n📊 Stdout and Stderr:")
    cmd := exec.Command("ls", "-la", "/nonexistent")
    var stdout, stderr bytes.Buffer
    cmd.Stdout = &stdout
    cmd.Stderr = &stderr
    err = cmd.Run()
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        fmt.Printf("   Stderr: %s\n", stderr.String())
    }
    
    // Command with timeout
    fmt.Println("\n📊 Command with Timeout:")
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    cmd = exec.CommandContext(ctx, "sleep", "2")
    err = cmd.Run()
    if ctx.Err() == context.DeadlineExceeded {
        fmt.Println("   Command timed out!")
    } else if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Println("   Command completed")
    }
    
    // Combined output
    fmt.Println("\n📊 CombinedOutput (stdout + stderr):")
    out, _ = exec.Command("date").CombinedOutput()
    fmt.Printf("   %s", out)
    
    // Check if command exists
    fmt.Println("\n📊 Check Command Exists:")
    path, err := exec.LookPath("git")
    if err != nil {
        fmt.Println("   git not found")
    } else {
        fmt.Printf("   git found at: %s\n", path)
    }
}

func min(a, b int) int {
    if a < b { return a }
    return b
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Running External Commands                       ║
╚══════════════════════════════════════════════════════════╝

📊 Simple Command:
   Output: Hello from Go!

📊 List Files:
total 24
drwxr-x   ...
   ...

📊 Stdout and Stderr:
   Error: ...

   Stderr: ls: /nonexistent: No such file or directory

📊 Command with Timeout:
   Command completed

📊 CombinedOutput (stdout + stderr):
   Tue Feb 10 12:00:00 PST 2025

📊 Check Command Exists:
   git found at: /usr/bin/git
```

*Note: Output varies by system (ls listing, date, git path).*

---

## 🔧 Advanced Command Execution

```go
// exec_advanced.go
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
    "os/exec"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Advanced Command Execution                      ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pipe input
    fmt.Println("\n📊 Pipe Input:")
    cmd := exec.Command("grep", "Go")
    cmd.Stdin = strings.NewReader("Hello Go\nHello Python\nI love Go\n")
    out, _ := cmd.Output()
    fmt.Printf("%s", out)
    
    // Stream output line by line
    fmt.Println("\n📊 Stream Output:")
    cmd = exec.Command("ping", "-c", "3", "localhost")
    stdout, _ := cmd.StdoutPipe()
    cmd.Start()
    
    scanner := bufio.NewScanner(stdout)
    for scanner.Scan() {
        fmt.Printf("   > %s\n", scanner.Text())
    }
    cmd.Wait()
    
    // Run with environment variables
    fmt.Println("\n📊 With Environment:")
    cmd = exec.Command("sh", "-c", "echo $MY_VAR")
    cmd.Env = append(os.Environ(), "MY_VAR=Hello from Go!")
    out, _ = cmd.Output()
    fmt.Printf("   %s", out)
    
    // Set working directory
    fmt.Println("\n📊 Set Working Directory:")
    cmd = exec.Command("pwd")
    cmd.Dir = "/tmp"
    out, _ = cmd.Output()
    fmt.Printf("   %s", out)
}

import "strings"
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           Advanced Command Execution                      ║
╚══════════════════════════════════════════════════════════╝

📊 Pipe Input:
Hello Go
I love Go

📊 Stream Output:
   > PING localhost (...): 56 data bytes
   > 64 bytes from localhost: ...
   > ...

📊 With Environment:
   Hello from Go!

📊 Set Working Directory:
   /tmp

```

*Note: Stream output (ping) varies by system and platform.*

---

## 🎯 Key Takeaways

1. **signal.Notify** to catch OS signals
2. **SIGINT** (Ctrl+C) and **SIGTERM** for graceful shutdown
3. **exec.Command** to run external programs
4. **CommandContext** for timeouts
5. **StdoutPipe/StderrPipe** for streaming output
6. Always handle **graceful shutdown** in production

---

## ➡️ Next Steps

**Next Topic:** [57 - Concurrency Patterns](./57-concurrency-patterns.md)

