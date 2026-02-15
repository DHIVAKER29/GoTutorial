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

- **Context = request scope carrier** — flows through Handler → Service → Database
- **Context carries:**
  - Cancellation signal ("stop working, client disconnected")
  - Deadline/timeout ("must finish by X")
  - Request-scoped values (user ID, trace ID)
- **Without context:** How to stop goroutines? Timeout slow calls? Pass request metadata?
- **With context:** Parent cancels → all children cancelled. Timeout expires → all operations stop.

---

## 📝 Creating Contexts

### Background and TODO

```go
ctx := context.Background()  // Root, never cancelled
todo := context.TODO()      // Placeholder during development
// Output: (no output)
```

### WithCancel

```go
ctx, cancel := context.WithCancel(context.Background())
cancel()
fmt.Println(ctx.Err())  // context canceled
```

### WithTimeout

```go
ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
defer cancel()
<-ctx.Done()
fmt.Println(ctx.Err())  // context deadline exceeded
```

### WithValue

```go
type key string
ctx := context.WithValue(context.Background(), key("userID"), "user-123")
id := ctx.Value(key("userID")).(string)
// Output: id is "user-123"
```

### Complete Example: Creating Contexts

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    ctx := context.Background()
    fmt.Println("Root:", ctx)

    ctx, cancel := context.WithCancel(context.Background())
    cancel()
    fmt.Println("After cancel:", ctx.Err())

    ctx, cancel = context.WithTimeout(context.Background(), 50*time.Millisecond)
    defer cancel()
    <-ctx.Done()
    fmt.Println("Timeout:", ctx.Err())

    type key string
    ctx = context.WithValue(context.Background(), key("userID"), "user-123")
    if id, ok := ctx.Value(key("userID")).(string); ok {
        fmt.Println("UserID:", id)
    }
}
```

**Output:**
```
Root: context.Background
After cancel: context canceled
Timeout: context deadline exceeded
UserID: user-123
```

---

## 🔄 Context Propagation

### Passing Through Function Chain

```go
func handler(ctx context.Context) error {
    return service(ctx)
}
func service(ctx context.Context) error {
    return database(ctx)
}
func database(ctx context.Context) error {
    select {
    case <-time.After(300 * time.Millisecond):
        return nil
    case <-ctx.Done():
        return ctx.Err()
    }
}
```

### Complete Example: Context Propagation

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 200*time.Millisecond)
    defer cancel()

    err := handler(ctx)
    if err != nil {
        fmt.Println("Error:", err)
    }
}

func handler(ctx context.Context) error {
    return service(ctx)
}

func service(ctx context.Context) error {
    return database(ctx)
}

func database(ctx context.Context) error {
    select {
    case <-time.After(300 * time.Millisecond):
        fmt.Println("Query completed")
        return nil
    case <-ctx.Done():
        fmt.Println("Query cancelled")
        return ctx.Err()
    }
}
```

**Output:**
```
Query cancelled
Error: context deadline exceeded
```

---

## 🏭 Production Patterns

### Request Context with Values

```go
type contextKey string
ctx := context.WithValue(ctx, contextKey("requestID"), "req-123")
ctx = context.WithValue(ctx, contextKey("userID"), "user-456")
```

### External Call with Timeout

```go
ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
defer cancel()
result, err := callExternalAPI(ctx)
```

### Graceful Shutdown

```go
ctx, cancel := context.WithCancel(context.Background())
go worker(ctx, "worker-1")
// ... later ...
cancel()
```

### Complete Example: Production Patterns

```go
package main

import (
    "context"
    "fmt"
    "time"
)

type contextKey string

func main() {
    ctx := context.Background()
    ctx = context.WithValue(ctx, contextKey("requestID"), "req-abc")
    ctx = context.WithValue(ctx, contextKey("userID"), "user-456")
    handleRequest(ctx)

    ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel()
    result, err := callExternalAPI(ctx)
    if err != nil {
        fmt.Println("API error:", err)
    } else {
        fmt.Println("API result:", result)
    }

    ctx, cancel = context.WithCancel(context.Background())
    go worker(ctx)
    time.Sleep(100 * time.Millisecond)
    cancel()
    time.Sleep(50 * time.Millisecond)
}

func handleRequest(ctx context.Context) {
    reqID := ctx.Value(contextKey("requestID")).(string)
    userID := ctx.Value(contextKey("userID")).(string)
    fmt.Printf("Processing request=%s for user=%s\n", reqID, userID)
}

func callExternalAPI(ctx context.Context) (string, error) {
    select {
    case <-time.After(200 * time.Millisecond):
        return "success", nil
    case <-ctx.Done():
        return "", fmt.Errorf("timeout: %w", ctx.Err())
    }
}

func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("worker: shutting down")
            return
        default:
            fmt.Println("worker: working...")
            time.Sleep(30 * time.Millisecond)
        }
    }
}
```

**Output:**
```
Processing request=req-abc for user=user-456
API error: timeout: context deadline exceeded
worker: working...
worker: working...
worker: shutting down
```

---

## 📋 Context Best Practices

| Rule | Description |
|------|-------------|
| First parameter | `func DoSomething(ctx context.Context, arg string)` |
| Never store in struct | Pass through function parameters |
| Always call cancel | `defer cancel()` when using WithCancel/WithTimeout |
| Don't pass nil | Use `context.Background()` instead |
| Custom key types | `type myKey string` to avoid collisions |
| Request-scoped only | Request ID, User ID, Trace ID — not config or connections |

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
