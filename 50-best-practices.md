# 50 - Go Best Practices

> Writing idiomatic, maintainable, and production-ready Go code.

---

## 📌 What You'll Learn

- Coding conventions
- Project structure
- Error handling patterns
- Concurrency best practices
- Production readiness checklist

---

## 📜 Code Style

**Go Proverbs:** Don't communicate by sharing memory; share memory by communicating. Concurrency is not parallelism. Channels orchestrate; mutexes serialize. The bigger the interface, the weaker the abstraction. Make the zero value useful. Errors are values. Clear is better than clever.

---

## 📁 Project Structure

| Directory | Purpose |
|-----------|---------|
| `cmd/` | Main applications (api/main.go, worker/main.go) |
| `internal/` | Private app code (config, handler, service, repository, model) |
| `pkg/` | Public library code |
| `api/` | API definitions (proto files) |
| `scripts/` | Build/deployment scripts |
| `configs/` | Configuration files |
| `docs/` | Documentation |

---

## ✅ Coding Conventions

| Convention | Good | Bad |
|------------|------|-----|
| Package names | `user` | `userService` |
| Variables | `userName` | `user_name` |
| Exported | `User` (capitalized) | `user` |
| Getters | `Name()` | `GetName()` |
| Acronyms | `userID`, `httpClient` | `userId`, `HttpClient` |

---

## ⚠️ Error Handling

```go
// Always check and wrap errors
result, err := doSomething()
if err != nil {
    return fmt.Errorf("doSomething failed: %w", err)
}

// Sentinel errors
var ErrNotFound = errors.New("not found")
if errors.Is(err, ErrNotFound) { ... }

// Custom error types
type ValidationError struct {
    Field   string
    Message string
}
func (e *ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}
```

---

## 🔄 Concurrency

```go
// Channels for communication
results := make(chan Result)
go func() { results <- doWork() }()

// Context for cancellation
func worker(ctx context.Context) {
    select {
    case <-ctx.Done():
        return
    case data := <-dataChan:
        process(data)
    }
}

// sync.WaitGroup for coordination
var wg sync.WaitGroup
for i := 0; i < 10; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        doWork()
    }()
}
wg.Wait()
```

---

## 🏭 Production Checklist

| Area | Checklist |
|------|-----------|
| **Error Handling** | All errors handled, no panics in business logic |
| **Logging** | Structured logging, request IDs, metrics |
| **Configuration** | Env-based, no secrets in code |
| **HTTP/gRPC** | Timeouts, graceful shutdown, rate limiting |
| **Database** | Connection pooling, prepared statements, transactions |
| **Concurrency** | Race detector passes, context propagation |
| **Testing** | Unit tests, integration tests, benchmarks |
| **Build** | Multi-stage Dockerfile, static binary, CI/CD |

---

## 📝 Code Review Checklist

- Logic correct? Readable? Edge cases?
- Error handling complete?
- N+1 queries? Unnecessary allocations?
- Input validation? SQL injection safe?
- Tests cover new functionality?
- Exported functions documented?

---

## 🎯 Key Takeaways

1. **gofmt** all code - no style debates
2. **Handle all errors** - don't ignore them
3. **Use context** for cancellation
4. **Prefer composition** over inheritance
5. **Make zero values useful**
6. **Small interfaces** are better
7. **Document exported symbols**
8. **Test with -race flag**
9. **Log structured data**
10. **Graceful shutdown** always

---

## 🎉 Congratulations!

You've completed the comprehensive Go tutorial!

**Resources:** [Effective Go](https://golang.org/doc/effective_go), [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments), [Go Proverbs](https://go-proverbs.github.io/), [Awesome Go](https://awesome-go.com/)
