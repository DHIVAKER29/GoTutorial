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

```go
// best_practices.go
package main

/*
GO PROVERBS (https://go-proverbs.github.io/)

1. Don't communicate by sharing memory, share memory by communicating.
2. Concurrency is not parallelism.
3. Channels orchestrate; mutexes serialize.
4. The bigger the interface, the weaker the abstraction.
5. Make the zero value useful.
6. interface{} says nothing.
7. Gofmt's style is no one's favorite, yet gofmt is everyone's favorite.
8. A little copying is better than a little dependency.
9. Syscall must always be guarded with build tags.
10. Cgo must always be guarded with build tags.
11. Cgo is not Go.
12. With the unsafe package there are no guarantees.
13. Clear is better than clever.
14. Reflection is never clear.
15. Errors are values.
16. Don't just check errors, handle them gracefully.
17. Design the architecture, name the components, document the details.
18. Documentation is for users.
*/
```

---

## 📁 Project Structure

```
myproject/
├── cmd/                    # Main applications
│   ├── api/
│   │   └── main.go        # API server entry point
│   └── worker/
│       └── main.go        # Worker entry point
│
├── internal/              # Private application code
│   ├── config/           # Configuration
│   ├── handler/          # HTTP/gRPC handlers
│   ├── service/          # Business logic
│   ├── repository/       # Database access
│   └── model/            # Domain models
│
├── pkg/                   # Public library code
│   └── utils/            # Shared utilities
│
├── api/                   # API definitions
│   └── proto/            # Protocol buffer files
│
├── scripts/              # Build/deployment scripts
├── configs/              # Configuration files
├── docs/                 # Documentation
├── test/                 # Additional test data
│
├── go.mod
├── go.sum
├── Makefile
├── Dockerfile
└── README.md
```

---

## ✅ Coding Conventions

```go
// NAMING CONVENTIONS

// Package names: lowercase, short, no underscores
package userservice  // ❌
package user         // ✅

// Variables: camelCase
var userName string      // ✅
var user_name string     // ❌

// Exported = Capitalized
type User struct{}       // Exported (public)
type user struct{}       // Unexported (private)

// Interfaces: -er suffix for single method
type Reader interface{ Read(p []byte) (n int, err error) }
type Stringer interface{ String() string }

// Getters: no "Get" prefix
func (u *User) Name() string { return u.name }  // ✅
func (u *User) GetName() string {}              // ❌

// Acronyms: all caps or all lower
var userID int    // ✅
var userId int    // ❌
var httpClient    // ✅
var HTTPClient    // ✅
var HttpClient    // ❌
```

---

## ⚠️ Error Handling

```go
// ERROR HANDLING BEST PRACTICES

// 1. Always check errors
result, err := doSomething()
if err != nil {
    return fmt.Errorf("doSomething failed: %w", err)  // Wrap with context
}

// 2. Return errors, don't panic
func process() error {  // ✅
    return errors.New("failed")
}
func process() {        // ❌
    panic("failed")
}

// 3. Handle errors at the right level
// Low level: return error
// High level: log and take action

// 4. Use sentinel errors for expected cases
var ErrNotFound = errors.New("not found")

// 5. Use error wrapping
if err != nil {
    return fmt.Errorf("failed to save user %d: %w", userID, err)
}

// 6. Check wrapped errors
if errors.Is(err, ErrNotFound) {
    // Handle not found
}

// 7. Custom error types for extra info
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
// CONCURRENCY BEST PRACTICES

// 1. Use channels for communication
results := make(chan Result)
go func() {
    results <- doWork()
}()

// 2. Use context for cancellation
func worker(ctx context.Context) {
    select {
    case <-ctx.Done():
        return
    case data := <-dataChan:
        process(data)
    }
}

// 3. Use sync.WaitGroup for goroutine coordination
var wg sync.WaitGroup
for i := 0; i < 10; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        doWork()
    }()
}
wg.Wait()

// 4. Always handle goroutine lifetime
// Don't start goroutines without knowing how they'll stop

// 5. Use errgroup for error propagation
g, ctx := errgroup.WithContext(context.Background())
g.Go(func() error { return task1(ctx) })
g.Go(func() error { return task2(ctx) })
if err := g.Wait(); err != nil {
    // Handle first error
}

// 6. Use -race flag in tests
// go test -race ./...
```

---

## 🏭 Production Checklist

```
PRE-PRODUCTION CHECKLIST:

□ Error Handling
  □ All errors are handled or propagated
  □ Errors include context (wrapped)
  □ No panics in business logic
  □ Graceful degradation

□ Logging & Observability
  □ Structured logging (JSON in prod)
  □ Request IDs for tracing
  □ Metrics exposed (Prometheus)
  □ Health check endpoint

□ Configuration
  □ Environment-based config
  □ No secrets in code
  □ Sensible defaults
  □ Validation on startup

□ HTTP/gRPC
  □ Timeouts configured
  □ Graceful shutdown
  □ Rate limiting
  □ Input validation

□ Database
  □ Connection pooling configured
  □ Prepared statements (SQL injection safe)
  □ Transactions where needed
  □ Context with timeout for queries

□ Concurrency
  □ Race detector passes (go test -race)
  □ Context propagation
  □ No goroutine leaks
  □ Deadlock prevention

□ Testing
  □ Unit tests (80%+ coverage)
  □ Integration tests
  □ Benchmarks for hot paths
  □ Load testing

□ Build & Deploy
  □ Multi-stage Dockerfile
  □ Static binary (CGO_ENABLED=0)
  □ Version info embedded
  □ CI/CD pipeline
```

---

## 📝 Code Review Checklist

```
REVIEW CHECKLIST:

✓ Code
  - Is the logic correct?
  - Is it readable and maintainable?
  - Are there any edge cases missed?
  - Is error handling complete?

✓ Performance
  - Any N+1 queries?
  - Unnecessary allocations?
  - Appropriate use of caching?

✓ Security
  - Input validation?
  - SQL injection safe?
  - Secrets exposure?
  - Access control?

✓ Testing
  - Tests cover new functionality?
  - Edge cases tested?
  - Tests are maintainable?

✓ Documentation
  - Exported functions documented?
  - Complex logic explained?
  - README updated if needed?
```

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

**What's Next?**
- Build real projects
- Read open source Go code
- Contribute to Go projects
- Keep learning and practicing

**Resources:**
- [Effective Go](https://golang.org/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Go Proverbs](https://go-proverbs.github.io/)
- [Awesome Go](https://awesome-go.com/)

Happy coding! 🚀

