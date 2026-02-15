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

| Signal | Value | Trigger |
|--------|-------|---------|
| SIGINT | 2 | Ctrl+C |
| SIGTERM | 15 | kill command, container stop |
| SIGKILL | 9 | Cannot be caught! Immediate kill |
| SIGHUP | 1 | Terminal hangup |

```go
sigChan := make(chan os.Signal, 1)
signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)

sig := <-sigChan
fmt.Printf("Received: %v\n", sig)
// Cleanup...
// Output: Received: interrupt (on Ctrl+C)
```

---

## 🏭 Graceful Shutdown Pattern

```go
server := &http.Server{Addr: ":8080", Handler: handler}

go func() {
    server.ListenAndServe()
}()

quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit

ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
server.Shutdown(ctx)
// Output: Server stops gracefully, waits for active connections
```

---

## 💻 Running External Commands

```go
// Simple command
out, err := exec.Command("echo", "Hello from Go!").Output()
// Output: "Hello from Go!\n"

// Capture stdout and stderr
cmd := exec.Command("ls", "-la", "/nonexistent")
var stdout, stderr bytes.Buffer
cmd.Stdout = &stdout
cmd.Stderr = &stderr
err = cmd.Run()
// stderr contains error message

// Command with timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
cmd = exec.CommandContext(ctx, "sleep", "2")
err = cmd.Run()
// ctx.Err() == context.DeadlineExceeded if timed out

// Check if command exists
path, err := exec.LookPath("git")
// Output: "/usr/bin/git" or error
```

---

## 🔧 Advanced Command Execution

```go
// Pipe input
cmd := exec.Command("grep", "Go")
cmd.Stdin = strings.NewReader("Hello Go\nHello Python\nI love Go\n")
out, _ := cmd.Output()
// Output: "Hello Go\nI love Go\n"

// Stream output
cmd = exec.Command("ping", "-c", "3", "localhost")
stdout, _ := cmd.StdoutPipe()
cmd.Start()
scanner := bufio.NewScanner(stdout)
for scanner.Scan() {
    fmt.Println(scanner.Text())
}
cmd.Wait()

// Environment variables
cmd = exec.Command("sh", "-c", "echo $MY_VAR")
cmd.Env = append(os.Environ(), "MY_VAR=Hello from Go!")
out, _ = cmd.Output()
// Output: "Hello from Go!\n"

// Working directory
cmd = exec.Command("pwd")
cmd.Dir = "/tmp"
out, _ = cmd.Output()
// Output: "/tmp\n"
```

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
