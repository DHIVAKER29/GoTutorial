# 42 - Logging

> Structured logging and observability in Go applications.

---

## 📌 What You'll Learn

- Standard library log package
- Structured logging (slog - Go 1.21+)
- Log levels and handlers
- Production logging patterns

---

## 📝 Standard Log Package

### Basic Logging

```go
package main

import "log"

func main() {
    log.Println("This is a log message")
    log.Printf("User %s logged in", "alice")
}
// Output:
// 2026/02/10 12:34:56 This is a log message
// 2026/02/10 12:34:56 User alice logged in
```

### Custom Logger

```go
package main

import (
    "log"
    "os"
)

func main() {
    logger := log.New(os.Stdout, "[APP] ", log.Ldate|log.Ltime|log.Lshortfile)
    logger.Println("Custom logger message")
}
// Output:
// [APP] 2026/02/10 12:34:56 main.go:8 Custom logger message
```

### Log Flags Reference

| Flag | Description |
|------|-------------|
| `log.Ldate` | 2009/01/23 |
| `log.Ltime` | 01:23:23 |
| `log.Lmicroseconds` | 01:23:23.123123 |
| `log.Llongfile` | /full/path/file.go:23 |
| `log.Lshortfile` | file.go:23 |
| `log.LUTC` | Use UTC |
| `log.LstdFlags` | Ldate \| Ltime |

### Log to File

```go
package main

import (
    "log"
    "os"
)

func main() {
    file, _ := os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    defer file.Close()
    fileLogger := log.New(file, "", log.LstdFlags)
    fileLogger.Println("Logged to file")
}
// Output: (writes to app.log)
```

---

## 🏭 Structured Logging (slog - Go 1.21+)

### Basic slog Usage

```go
package main

import "log/slog"

func main() {
    slog.Info("Application started")
    slog.Warn("Low disk space", "available", "500MB")
    slog.Error("Connection failed", "host", "db.example.com", "error", "timeout")
}
// Output:
// time=2026-02-10T12:34:56.789Z level=INFO msg="Application started"
// time=2026-02-10T12:34:56.789Z level=WARN msg="Low disk space" available=500MB
// time=2026-02-10T12:34:56.789Z level=ERROR msg="Connection failed" host=db.example.com error=timeout
```

### JSON Handler (Production)

```go
package main

import (
    "log/slog"
    "os"
)

func main() {
    jsonHandler := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{Level: slog.LevelDebug})
    logger := slog.New(jsonHandler)

    logger.Info("User action",
        "user_id", 12345,
        "action", "login",
        "ip", "192.168.1.1",
    )
}
// Output:
// {"time":"2026-02-10T12:34:56.789Z","level":"INFO","msg":"User action","user_id":12345,"action":"login","ip":"192.168.1.1"}
```

### Log Levels

| Level | Value |
|-------|-------|
| `slog.LevelDebug` | -4 |
| `slog.LevelInfo` | 0 |
| `slog.LevelWarn` | 4 |
| `slog.LevelError` | 8 |

### Logger with Default Attributes

```go
package main

import (
    "log/slog"
    "os"
)

func main() {
    jsonHandler := slog.NewJSONHandler(os.Stdout, nil)
    logger := slog.New(jsonHandler)
    userLogger := logger.With("service", "user-api", "version", "1.0.0")
    userLogger.Info("User created", "user_id", 123)
}
// Output:
// {"time":"...","level":"INFO","msg":"User created","service":"user-api","version":"1.0.0","user_id":123}
```

### Group Attributes

```go
package main

import (
    "log/slog"
    "os"
)

func main() {
    logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
    logger.Info("Server config",
        slog.Group("server", "host", "localhost", "port", 8080),
        slog.Group("database", "driver", "postgres", "pool_size", 10),
    )
}
// Output:
// {"time":"...","level":"INFO","msg":"Server config","server":{"host":"localhost","port":8080},"database":{"driver":"postgres","pool_size":10}}
```

---

## 🔧 Custom Log Handler

```go
package main

import (
    "log/slog"
    "os"
)

func main() {
    base := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
        ReplaceAttr: func(groups []string, a slog.Attr) slog.Attr {
            if a.Key == slog.TimeKey {
                return slog.String("ts", a.Value.Time().Format("2006-01-02T15:04:05"))
            }
            if a.Key == slog.LevelKey {
                return slog.String("severity", a.Value.String())
            }
            return a
        },
    })
    logger := slog.New(base)
    slog.SetDefault(logger)
    slog.Info("Custom formatted log", "user", "alice", "action", "login")
}
// Output:
// {"ts":"2026-02-10T12:34:56","severity":"INFO","msg":"Custom formatted log","user":"alice","action":"login"}
```

---

## 🏭 Production Pattern

### Complete Example

```go
package main

import (
    "context"
    "log/slog"
    "os"
)

type Logger struct {
    *slog.Logger
}

func NewLogger(env string) *Logger {
    opts := &slog.HandlerOptions{AddSource: true}
    var handler slog.Handler
    if env == "production" {
        opts.Level = slog.LevelInfo
        handler = slog.NewJSONHandler(os.Stdout, opts)
    } else {
        opts.Level = slog.LevelDebug
        handler = slog.NewTextHandler(os.Stdout, opts)
    }
    return &Logger{slog.New(handler)}
}

func (l *Logger) WithRequest(requestID, userID string) *Logger {
    return &Logger{l.With("request_id", requestID, "user_id", userID)}
}

func main() {
    env := os.Getenv("ENV")
    if env == "" {
        env = "development"
    }
    logger := NewLogger(env)
    slog.SetDefault(logger.Logger)

    ctx := context.Background()
    reqLogger := logger.WithRequest("req-123", "user-456")
    reqLogger.InfoContext(ctx, "Processing request", "method", "POST", "path", "/api/orders")
    reqLogger.ErrorContext(ctx, "Failed to process", "error", "validation failed")
}
```

**Output:**
```
time=2026-02-10T12:34:56.789Z level=INFO source=main.go:35 msg="Processing request" request_id=req-123 user_id=user-456 method=POST path=/api/orders
time=2026-02-10T12:34:56.789Z level=ERROR source=main.go:36 msg="Failed to process" request_id=req-123 user_id=user-456 error="validation failed"
```

---

## 🎯 Key Takeaways

1. **log** package for simple logging
2. **slog** (Go 1.21+) for structured logging
3. **JSON handler** for production
4. **Text handler** for development
5. **With()** to add default attributes
6. **Context** for request-scoped logging

---

## ➡️ Next Steps

**Next Topic:** [43 - Reflection](./43-reflection.md)
