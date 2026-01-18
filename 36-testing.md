# 36 - Testing in Go

> Writing tests, benchmarks, and examples - Go's built-in testing framework.

---

## 📌 What You'll Learn

- Writing unit tests
- Table-driven tests
- Test coverage
- Benchmarks
- Example tests (documentation)
- Subtests and test helpers
- Mocking and test doubles
- Integration testing patterns

---

## 🧪 Testing Basics

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO TESTING CONVENTIONS                                         │
│                                                                 │
│  FILE NAMING:                                                   │
│  • Source: math.go                                              │
│  • Test:   math_test.go (must end with _test.go)                │
│                                                                 │
│  FUNCTION NAMING:                                               │
│  • Tests:      func TestXxx(t *testing.T)                       │
│  • Benchmarks: func BenchmarkXxx(b *testing.B)                  │
│  • Examples:   func ExampleXxx()                                │
│                                                                 │
│  RUNNING TESTS:                                                 │
│  • go test              - Run all tests in current package      │
│  • go test ./...        - Run all tests recursively             │
│  • go test -v           - Verbose output                        │
│  • go test -run TestAdd - Run specific test                     │
│  • go test -cover       - Show coverage                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Writing Your First Test

### Source File (math.go)

```go
// math.go
package math

func Add(a, b int) int {
    return a + b
}

func Subtract(a, b int) int {
    return a - b
}

func Multiply(a, b int) int {
    return a * b
}

func Divide(a, b int) (int, error) {
    if b == 0 {
        return 0, ErrDivisionByZero
    }
    return a / b, nil
}

var ErrDivisionByZero = fmt.Errorf("division by zero")
```

### Test File (math_test.go)

```go
// math_test.go
package math

import (
    "errors"
    "testing"
)

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    expected := 5
    
    if result != expected {
        t.Errorf("Add(2, 3) = %d; want %d", result, expected)
    }
}

func TestSubtract(t *testing.T) {
    result := Subtract(5, 3)
    expected := 2
    
    if result != expected {
        t.Errorf("Subtract(5, 3) = %d; want %d", result, expected)
    }
}

func TestDivide(t *testing.T) {
    // Test successful division
    result, err := Divide(10, 2)
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if result != 5 {
        t.Errorf("Divide(10, 2) = %d; want 5", result)
    }
    
    // Test division by zero
    _, err = Divide(10, 0)
    if err == nil {
        t.Error("expected error for division by zero")
    }
    if !errors.Is(err, ErrDivisionByZero) {
        t.Errorf("expected ErrDivisionByZero, got %v", err)
    }
}
```

---

## 📊 Table-Driven Tests (Recommended!)

```go
// table_test.go
package math

import "testing"

func TestAddTableDriven(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -2, -3, -5},
        {"mixed numbers", -2, 3, 1},
        {"zero", 0, 0, 0},
        {"large numbers", 1000000, 2000000, 3000000},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d",
                    tt.a, tt.b, result, tt.expected)
            }
        })
    }
}

func TestDivideTableDriven(t *testing.T) {
    tests := []struct {
        name      string
        a, b      int
        want      int
        wantErr   bool
    }{
        {"10/2", 10, 2, 5, false},
        {"9/3", 9, 3, 3, false},
        {"5/2", 5, 2, 2, false},  // Integer division
        {"divide by zero", 10, 0, 0, true},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Divide(tt.a, tt.b)
            
            if (err != nil) != tt.wantErr {
                t.Errorf("Divide() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            
            if got != tt.want {
                t.Errorf("Divide() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

---

## 🎯 Testing Methods

```go
// testing_methods.go
package main

import (
    "fmt"
    "testing"
)

// t.Error vs t.Fatal vs t.Skip

func TestErrorVsFatal(t *testing.T) {
    // t.Error - log error and CONTINUE
    if 1+1 != 2 {
        t.Error("math is broken")
    }
    fmt.Println("This line runs after t.Error")
    
    // t.Fatal - log error and STOP immediately
    if 2+2 != 4 {
        t.Fatal("math is really broken")
    }
    fmt.Println("This line won't run if Fatal triggered")
}

func TestSkip(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping long test in short mode")
    }
    // Long running test...
}

// Common assertions pattern
func assertEqual(t *testing.T, got, want interface{}) {
    t.Helper()  // Marks this as helper (better error line numbers)
    if got != want {
        t.Errorf("got %v, want %v", got, want)
    }
}

func TestWithHelper(t *testing.T) {
    result := 2 + 2
    assertEqual(t, result, 4)
}
```

---

## 📈 Benchmarks

```go
// benchmark_test.go
package math

import "testing"

func BenchmarkAdd(b *testing.B) {
    // b.N is determined by the testing framework
    for i := 0; i < b.N; i++ {
        Add(100, 200)
    }
}

func BenchmarkAddParallel(b *testing.B) {
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            Add(100, 200)
        }
    })
}

// Benchmark with setup
func BenchmarkDivide(b *testing.B) {
    // Setup (not counted)
    a, divisor := 1000000, 7
    
    b.ResetTimer()  // Reset timer after setup
    
    for i := 0; i < b.N; i++ {
        Divide(a, divisor)
    }
}

// Benchmark with sub-benchmarks
func BenchmarkOperations(b *testing.B) {
    b.Run("Add", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            Add(1, 2)
        }
    })
    
    b.Run("Subtract", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            Subtract(5, 3)
        }
    })
    
    b.Run("Multiply", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            Multiply(4, 5)
        }
    })
}

/*
Run benchmarks:
  go test -bench=.
  go test -bench=BenchmarkAdd
  go test -bench=. -benchmem    (show memory allocations)
  go test -bench=. -count=5    (run 5 times)
  
Output:
  BenchmarkAdd-8    1000000000    0.25 ns/op    0 B/op    0 allocs/op
                    │             │              │         │
                    iterations    time/op        memory    allocations
*/
```

---

## 📚 Example Tests (Documentation)

```go
// example_test.go
package math

import "fmt"

func ExampleAdd() {
    result := Add(2, 3)
    fmt.Println(result)
    // Output: 5
}

func ExampleAdd_negative() {
    result := Add(-2, -3)
    fmt.Println(result)
    // Output: -5
}

func ExampleDivide() {
    result, err := Divide(10, 2)
    if err != nil {
        fmt.Println("error:", err)
        return
    }
    fmt.Println(result)
    // Output: 5
}

/*
Examples serve two purposes:
1. Documentation (appear in godoc)
2. Tests (Output comment is verified)

Naming:
  ExampleFunction()              - Example for Function
  ExampleFunction_case()         - Specific case
  ExampleType_Method()           - Method on Type
*/
```

---

## 🔧 Test Helpers and Setup

```go
// setup_test.go
package mypackage

import (
    "os"
    "testing"
)

// TestMain for package-level setup/teardown
func TestMain(m *testing.M) {
    // Setup before all tests
    setup()
    
    // Run tests
    code := m.Run()
    
    // Teardown after all tests
    teardown()
    
    os.Exit(code)
}

func setup() {
    // Create test database, files, etc.
}

func teardown() {
    // Clean up
}

// Per-test setup with t.Cleanup
func TestWithCleanup(t *testing.T) {
    // Setup
    file, err := os.CreateTemp("", "test")
    if err != nil {
        t.Fatal(err)
    }
    
    // Register cleanup (runs after test, even if test fails)
    t.Cleanup(func() {
        os.Remove(file.Name())
    })
    
    // Test using file...
}

// Helper function
func setupTestDB(t *testing.T) *DB {
    t.Helper()  // Mark as helper for better error reporting
    
    db, err := NewDB(":memory:")
    if err != nil {
        t.Fatalf("failed to create test DB: %v", err)
    }
    
    t.Cleanup(func() {
        db.Close()
    })
    
    return db
}
```

---

## 🎭 Mocking and Test Doubles

```go
// mocking_test.go
package service

import (
    "errors"
    "testing"
)

// Interface to mock
type UserRepository interface {
    GetByID(id int) (*User, error)
    Save(user *User) error
}

type User struct {
    ID   int
    Name string
}

// Service we're testing
type UserService struct {
    repo UserRepository
}

func (s *UserService) GetUser(id int) (*User, error) {
    return s.repo.GetByID(id)
}

// Mock implementation
type MockUserRepo struct {
    users   map[int]*User
    saveErr error
}

func (m *MockUserRepo) GetByID(id int) (*User, error) {
    user, ok := m.users[id]
    if !ok {
        return nil, errors.New("user not found")
    }
    return user, nil
}

func (m *MockUserRepo) Save(user *User) error {
    if m.saveErr != nil {
        return m.saveErr
    }
    m.users[user.ID] = user
    return nil
}

// Tests using mock
func TestUserService_GetUser(t *testing.T) {
    // Setup mock
    mockRepo := &MockUserRepo{
        users: map[int]*User{
            1: {ID: 1, Name: "Alice"},
        },
    }
    
    service := &UserService{repo: mockRepo}
    
    // Test found
    user, err := service.GetUser(1)
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if user.Name != "Alice" {
        t.Errorf("expected Alice, got %s", user.Name)
    }
    
    // Test not found
    _, err = service.GetUser(999)
    if err == nil {
        t.Error("expected error for non-existent user")
    }
}
```

---

## 📊 Test Coverage

```bash
# Run tests with coverage
go test -cover

# Generate coverage profile
go test -coverprofile=coverage.out

# View coverage in terminal
go tool cover -func=coverage.out

# View coverage in browser (HTML)
go test -coverprofile=coverage.out
go tool cover -html=coverage.out

# Coverage for specific package
go test -cover ./...

# Set minimum coverage (in CI)
go test -cover ./... | grep -E "coverage: [0-9]+\.[0-9]+% of statements"
```

---

## 🏭 Production Testing Patterns

```go
// production_test.go
package main

import (
    "net/http"
    "net/http/httptest"
    "testing"
)

// Testing HTTP handlers
func TestHealthEndpoint(t *testing.T) {
    handler := http.HandlerFunc(healthHandler)
    
    // Create test request
    req := httptest.NewRequest("GET", "/health", nil)
    
    // Create response recorder
    rr := httptest.NewRecorder()
    
    // Call handler
    handler.ServeHTTP(rr, req)
    
    // Check status code
    if rr.Code != http.StatusOK {
        t.Errorf("expected status 200, got %d", rr.Code)
    }
    
    // Check body
    expected := `{"status":"ok"}`
    if rr.Body.String() != expected {
        t.Errorf("expected %s, got %s", expected, rr.Body.String())
    }
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.Write([]byte(`{"status":"ok"}`))
}

// Integration test with test server
func TestAPIIntegration(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test in short mode")
    }
    
    // Create test server
    server := httptest.NewServer(http.HandlerFunc(healthHandler))
    defer server.Close()
    
    // Make real HTTP request
    resp, err := http.Get(server.URL + "/health")
    if err != nil {
        t.Fatalf("request failed: %v", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        t.Errorf("expected 200, got %d", resp.StatusCode)
    }
}
```

---

## 📋 Testing Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  TESTING COMMANDS                                               │
│                                                                 │
│  go test                    Run tests                           │
│  go test -v                 Verbose                             │
│  go test -run TestName      Run specific test                   │
│  go test -run TestAdd/pos   Run subtest                         │
│  go test -short             Skip long tests                     │
│  go test -race              Detect race conditions              │
│  go test -cover             Show coverage                       │
│  go test -bench=.           Run benchmarks                      │
│  go test -benchmem          Show memory in benchmarks           │
│  go test -count=5           Run N times                         │
│  go test -timeout=30s       Set timeout                         │
│  go test -parallel=4        Run N tests in parallel             │
│                                                                 │
│  TESTING.T METHODS                                              │
│                                                                 │
│  t.Error(args...)           Log error, continue                 │
│  t.Errorf(format, args...)  Formatted error, continue           │
│  t.Fatal(args...)           Log error, stop test                │
│  t.Fatalf(format, args...)  Formatted error, stop test          │
│  t.Skip(args...)            Skip test                           │
│  t.Helper()                 Mark as helper function             │
│  t.Parallel()               Run test in parallel                │
│  t.Run(name, func)          Run subtest                         │
│  t.Cleanup(func)            Register cleanup function           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Test files** end with `_test.go`
2. **Test functions** start with `Test` and take `*testing.T`
3. **Table-driven tests** are the Go way
4. **t.Helper()** for better error locations
5. **t.Cleanup()** for automatic teardown
6. **Benchmarks** use `*testing.B` and `b.N` loop
7. **Examples** serve as documentation AND tests
8. **httptest** for testing HTTP handlers
9. **Use interfaces** for easy mocking
10. **go test -race** in CI to catch race conditions

---

## ➡️ Next Steps

Congratulations! You've completed the Go tutorial! 🎉

**Review:** [README](./README.md) for the full table of contents

**Practice:** Apply these concepts to real projects

**Next Level:**
- Build a REST API
- Create a CLI tool
- Contribute to open source Go projects

