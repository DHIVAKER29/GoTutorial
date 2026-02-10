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

```go
// log_basic.go
package main

import (
    "log"
    "os"
)

func main() {
    // Default logger
    log.Println("This is a log message")
    log.Printf("User %s logged in", "alice")
    
    // Custom logger
    logger := log.New(os.Stdout, "[APP] ", log.Ldate|log.Ltime|log.Lshortfile)
    logger.Println("Custom logger message")
    
    // Log flags
    /*
    log.Ldate         // 2009/01/23
    log.Ltime         // 01:23:23
    log.Lmicroseconds // 01:23:23.123123
    log.Llongfile     // /full/path/file.go:23
    log.Lshortfile    // file.go:23
    log.LUTC          // use UTC
    log.Lmsgprefix    // prefix before message
    log.LstdFlags     // Ldate | Ltime
    */
    
    // Log to file
    file, _ := os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    defer file.Close()
    fileLogger := log.New(file, "", log.LstdFlags)
    fileLogger.Println("Logged to file")
    
    // Fatal (logs and calls os.Exit(1))
    // log.Fatal("Fatal error!")
    
    // Panic (logs and panics)
    // log.Panic("Panic!")
}
```

**Output:**
```
2026/02/10 12:34:56 This is a log message
2026/02/10 12:34:56 User alice logged in
[APP] 2026/02/10 12:34:56 log_basic.go:34 Custom logger message
2026/02/10 12:34:56 Logged to file
```

---

## 🏭 Structured Logging (slog - Go 1.21+)

```go
// slog_structured.go
package main

import (
    "context"
    "log/slog"
    "os"
)

func main() {
    // Default text handler
    slog.Info("Application started")
    slog.Warn("Low disk space", "available", "500MB")
    slog.Error("Connection failed", "host", "db.example.com", "error", "timeout")
    
    // JSON handler (production)
    jsonHandler := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
        Level: slog.LevelDebug,
    })
    jsonLogger := slog.New(jsonHandler)
    
    jsonLogger.Info("User action",
        "user_id", 12345,
        "action", "login",
        "ip", "192.168.1.1",
    )
    
    // Log levels
    /*
    slog.LevelDebug = -4
    slog.LevelInfo  = 0
    slog.LevelWarn  = 4
    slog.LevelError = 8
    */
    
    // With context
    ctx := context.Background()
    jsonLogger.InfoContext(ctx, "Request handled",
        "method", "GET",
        "path", "/api/users",
        "duration_ms", 45,
    )
    
    // Logger with default attributes
    userLogger := jsonLogger.With("service", "user-api", "version", "1.0.0")
    userLogger.Info("User created", "user_id", 123)
    
    // Group attributes
    jsonLogger.Info("Server config",
        slog.Group("server",
            "host", "localhost",
            "port", 8080,
        ),
        slog.Group("database",
            "driver", "postgres",
            "pool_size", 10,
        ),
    )
}

/*
JSON Output:
{"time":"2024-01-15T10:30:00Z","level":"INFO","msg":"User action","user_id":12345,"action":"login","ip":"192.168.1.1"}
*/
```

**Output:**
```
time=2026-02-10T12:34:56.789Z level=INFO msg="Application started"
time=2026-02-10T12:34:56.789Z level=WARN msg="Low disk space" available=500MB
time=2026-02-10T12:34:56.789Z level=ERROR msg="Connection failed" host=db.example.com error=timeout
{"time":"2026-02-10T12:34:56.789Z","level":"INFO","msg":"User action","user_id":12345,"action":"login","ip":"192.168.1.1"}
{"time":"2026-02-10T12:34:56.789Z","level":"INFO","msg":"Request handled","method":"GET","path":"/api/users","duration_ms":45}
{"time":"2026-02-10T12:34:56.789Z","level":"INFO","msg":"User created","service":"user-api","version":"1.0.0","user_id":123}
{"time":"2026-02-10T12:34:56.789Z","level":"INFO","msg":"Server config","server":{"host":"localhost","port":8080},"database":{"driver":"postgres","pool_size":10}}
```

---

## 🔧 Custom Log Handler

```go
// slog_custom.go
package main

import (
    "context"
    "log/slog"
    "os"
    "time"
)

// Custom handler that adds timestamp format
type CustomHandler struct {
    slog.Handler
}

func (h *CustomHandler) Handle(ctx context.Context, r slog.Record) error {
    r.AddAttrs(slog.String("timestamp", time.Now().Format(time.RFC3339)))
    return h.Handler.Handle(ctx, r)
}

func main() {
    // Create base handler
    base := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
        Level: slog.LevelInfo,
        ReplaceAttr: func(groups []string, a slog.Attr) slog.Attr {
            // Customize time format
            if a.Key == slog.TimeKey {
                return slog.String("ts", a.Value.Time().Format("2006-01-02T15:04:05"))
            }
            // Rename level
            if a.Key == slog.LevelKey {
                return slog.String("severity", a.Value.String())
            }
            return a
        },
    })
    
    logger := slog.New(base)
    slog.SetDefault(logger)
    
    slog.Info("Custom formatted log",
        "user", "alice",
        "action", "login",
    )
}
```

**Output:**
```
{"ts":"2026-02-10T12:34:56","severity":"INFO","msg":"Custom formatted log","user":"alice","action":"login"}
```

---

## 🏭 Production Pattern

```go
// logging_production.go
package main

import (
    "context"
    "log/slog"
    "os"
)

// Logger wrapper for your app
type Logger struct {
    *slog.Logger
}

func NewLogger(env string) *Logger {
    var handler slog.Handler
    
    opts := &slog.HandlerOptions{
        AddSource: true,  // Add file:line
    }
    
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
    return &Logger{l.With(
        "request_id", requestID,
        "user_id", userID,
    )}
}

func main() {
    env := os.Getenv("ENV")
    if env == "" {
        env = "development"
    }
    
    logger := NewLogger(env)
    slog.SetDefault(logger.Logger)
    
    // Use throughout app
    ctx := context.Background()
    reqLogger := logger.WithRequest("req-123", "user-456")
    
    reqLogger.InfoContext(ctx, "Processing request",
        "method", "POST",
        "path", "/api/orders",
    )
    
    reqLogger.ErrorContext(ctx, "Failed to process",
        "error", "validation failed",
    )
}
```

**Output:**
```
time=2026-02-10T12:34:56.789Z level=INFO source=logging_production.go:242 msg="Processing request" request_id=req-123 user_id=user-456 method=POST path=/api/orders
time=2026-02-10T12:34:56.789Z level=ERROR source=logging_production.go:247 msg="Failed to process" request_id=req-123 user_id=user-456 error="validation failed"
```

(Note: Timestamps and source file paths will vary based on actual execution time and file location)

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

