# 33 - Context Package

> Managing deadlines, cancellations, and request-scoped values across API boundaries.

---

## 📌 What You'll Learn

- What context is and why it matters
- Creating contexts (Background, TODO, WithCancel, WithTimeout, WithDeadline)
- Passing context through function calls
- Using context values
- Production patterns

---

## 🤔 What is Context?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CONTEXT = REQUEST SCOPE CARRIER                                │
│                                                                 │
│  HTTP Request ──► Handler ──► Service ──► Database              │
│       │              │            │            │                │
│       └──────────────┴────────────┴────────────┘                │
│                      Context flows through                      │
│                                                                 │
│  Context carries:                                               │
│  • Cancellation signal ("stop working, client disconnected")    │
│  • Deadline/timeout ("must finish by X")                        │
│  • Request-scoped values (user ID, trace ID)                    │
│                                                                 │
│  WITHOUT CONTEXT:                                               │
│  • How do you tell a goroutine to stop?                         │
│  • How do you timeout a slow database call?                     │
│  • How do you pass request metadata?                            │
│                                                                 │
│  WITH CONTEXT:                                                  │
│  • Parent cancels → all children automatically cancelled        │
│  • Timeout expires → all operations stop                        │
│  • Request ID flows through all layers                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Creating Contexts

```go
// context_creation.go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CREATING CONTEXTS                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Background context (root, never cancelled)
    fmt.Println("\n📊 context.Background():")
    ctx := context.Background()
    fmt.Printf("   Root context: %v\n", ctx)
    
    // TODO context (placeholder during development)
    fmt.Println("\n📊 context.TODO():")
    todo := context.TODO()
    fmt.Printf("   TODO context: %v (use when unsure)\n", todo)
    
    // WithCancel - manual cancellation
    fmt.Println("\n📊 context.WithCancel():")
    ctx, cancel := context.WithCancel(context.Background())
    fmt.Println("   Created cancellable context")
    cancel()  // Cancel it
    fmt.Printf("   After cancel: err = %v\n", ctx.Err())
    
    // WithTimeout - auto-cancel after duration
    fmt.Println("\n📊 context.WithTimeout():")
    ctx, cancel = context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel()  // Always defer cancel!
    
    select {
    case <-time.After(200 * time.Millisecond):
        fmt.Println("   Work completed")
    case <-ctx.Done():
        fmt.Printf("   Timeout: %v\n", ctx.Err())
    }
    
    // WithDeadline - cancel at specific time
    fmt.Println("\n📊 context.WithDeadline():")
    deadline := time.Now().Add(50 * time.Millisecond)
    ctx, cancel = context.WithDeadline(context.Background(), deadline)
    defer cancel()
    
    <-ctx.Done()
    fmt.Printf("   Deadline reached: %v\n", ctx.Err())
    
    // WithValue - store request-scoped values
    fmt.Println("\n📊 context.WithValue():")
    type key string
    ctx = context.WithValue(context.Background(), key("userID"), "user-123")
    
    if id, ok := ctx.Value(key("userID")).(string); ok {
        fmt.Printf("   UserID from context: %s\n", id)
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           CREATING CONTEXTS                               ║
╚══════════════════════════════════════════════════════════╝

📊 context.Background():
   Root context: context.Background

📊 context.TODO():
   TODO context: context.TODO (use when unsure)

📊 context.WithCancel():
   Created cancellable context
   After cancel: err = context canceled

📊 context.WithTimeout():
   Timeout: context deadline exceeded

📊 context.WithDeadline():
   Deadline reached: context deadline exceeded

📊 context.WithValue():
   UserID from context: user-123
```

---

## 🔄 Context Propagation

```go
// context_propagation.go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CONTEXT PROPAGATION                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Create context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 200*time.Millisecond)
    defer cancel()
    
    // Pass through function chain
    fmt.Println("\n📊 Context Through Function Chain:")
    err := handler(ctx)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    }
}

func handler(ctx context.Context) error {
    fmt.Println("   handler: starting")
    return service(ctx)
}

func service(ctx context.Context) error {
    fmt.Println("   service: calling database")
    return database(ctx)
}

func database(ctx context.Context) error {
    // Simulate slow query
    select {
    case <-time.After(300 * time.Millisecond):
        fmt.Println("   database: query completed")
        return nil
    case <-ctx.Done():
        fmt.Println("   database: query cancelled!")
        return ctx.Err()
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           CONTEXT PROPAGATION                             ║
╚══════════════════════════════════════════════════════════╝

📊 Context Through Function Chain:
   handler: starting
   service: calling database
   database: query cancelled!
   Error: context deadline exceeded
```

---

## 🏭 Production Patterns

```go
// context_production.go
package main

import (
    "context"
    "fmt"
    "time"
)

// Key type for context values (avoids collisions)
type contextKey string

const (
    requestIDKey contextKey = "requestID"
    userIDKey    contextKey = "userID"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION CONTEXT PATTERNS                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: HTTP handler context
    fmt.Println("\n📊 Pattern 1: Request Context")
    ctx := context.Background()
    ctx = context.WithValue(ctx, requestIDKey, "req-abc-123")
    ctx = context.WithValue(ctx, userIDKey, "user-456")
    
    handleRequest(ctx)
    
    // Pattern 2: Timeout for external calls
    fmt.Println("\n📊 Pattern 2: External Call Timeout")
    ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel()
    
    result, err := callExternalAPI(ctx)
    if err != nil {
        fmt.Printf("   API call failed: %v\n", err)
    } else {
        fmt.Printf("   API result: %s\n", result)
    }
    
    // Pattern 3: Graceful shutdown
    fmt.Println("\n📊 Pattern 3: Graceful Shutdown")
    ctx, cancel = context.WithCancel(context.Background())
    
    go worker(ctx, "background-worker")
    
    time.Sleep(100 * time.Millisecond)
    cancel()  // Signal shutdown
    time.Sleep(50 * time.Millisecond)
}

func handleRequest(ctx context.Context) {
    reqID := ctx.Value(requestIDKey).(string)
    userID := ctx.Value(userIDKey).(string)
    
    fmt.Printf("   Processing request=%s for user=%s\n", reqID, userID)
    
    // Pass context to downstream calls
    processOrder(ctx)
}

func processOrder(ctx context.Context) {
    reqID := ctx.Value(requestIDKey).(string)
    fmt.Printf("   Order processing (request=%s)\n", reqID)
}

func callExternalAPI(ctx context.Context) (string, error) {
    // Simulate slow external call
    select {
    case <-time.After(200 * time.Millisecond):
        return "success", nil
    case <-ctx.Done():
        return "", fmt.Errorf("api timeout: %w", ctx.Err())
    }
}

func worker(ctx context.Context, name string) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("   %s: shutting down\n", name)
            return
        default:
            fmt.Printf("   %s: working...\n", name)
            time.Sleep(30 * time.Millisecond)
        }
    }
}
```

---

## 📋 Context Rules

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CONTEXT BEST PRACTICES                                         │
│                                                                 │
│  1. Context should be the FIRST parameter:                      │
│     func DoSomething(ctx context.Context, arg string)           │
│                                                                 │
│  2. Never store context in a struct:                            │
│     ❌ type Server struct { ctx context.Context }               │
│     ✅ Pass through function parameters                         │
│                                                                 │
│  3. Always call cancel() when done:                             │
│     ctx, cancel := context.WithTimeout(...)                     │
│     defer cancel()  // Always!                                  │
│                                                                 │
│  4. Don't pass nil context:                                     │
│     ❌ doWork(nil, data)                                        │
│     ✅ doWork(context.Background(), data)                       │
│                                                                 │
│  5. Use custom types for context keys:                          │
│     type myKey string                                           │
│     ctx = context.WithValue(ctx, myKey("id"), "123")            │
│                                                                 │
│  6. Context values are for request-scoped data:                 │
│     ✅ Request ID, User ID, Trace ID                            │
│     ❌ Database connections, Config, Optional parameters        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Context** carries cancellation, deadlines, and values
2. **First parameter** in functions that need it
3. **Always defer cancel()** to release resources
4. **WithTimeout/WithDeadline** for time limits
5. **WithValue** for request-scoped data only
6. **Never store in structs**, pass explicitly
7. **Check ctx.Done()** in long operations

---

## ➡️ Next Steps

**Next Topic:** [34 - JSON](./34-json.md)

