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

## 🧪 Testing Conventions

| Convention | Rule |
|------------|------|
| **File naming** | `math.go` → `math_test.go` |
| **Test functions** | `func TestXxx(t *testing.T)` |
| **Benchmarks** | `func BenchmarkXxx(b *testing.B)` |
| **Examples** | `func ExampleXxx()` |

```bash
go test
go test ./...
go test -v
go test -run TestAdd
go test -cover
```

---

## 📝 Writing Your First Test

```go
// math.go
func Add(a, b int) int { return a + b }
func Divide(a, b int) (int, error) {
    if b == 0 {
        return 0, ErrDivisionByZero
    }
    return a / b, nil
}
var ErrDivisionByZero = errors.New("division by zero")
```

```go
// math_test.go
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Add(2, 3) = %d; want 5", result)
    }
}

func TestDivide(t *testing.T) {
    result, err := Divide(10, 2)
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if result != 5 {
        t.Errorf("Divide(10, 2) = %d; want 5", result)
    }

    _, err = Divide(10, 0)
    if err == nil {
        t.Error("expected error for division by zero")
    }
}
```

---

## 📊 Table-Driven Tests

```go
func TestAddTableDriven(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 2, 3, 5},
        {"negative", -2, -3, -5},
        {"zero", 0, 0, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, result, tt.expected)
            }
        })
    }
}
```

---

## 🎯 Testing Methods

```go
// t.Error - log and continue
// t.Fatal - log and stop
// t.Skip - skip test
// t.Helper() - mark as helper (better error lines)

func assertEqual(t *testing.T, got, want interface{}) {
    t.Helper()
    if got != want {
        t.Errorf("got %v, want %v", got, want)
    }
}
```

---

## 📈 Benchmarks

```go
func BenchmarkAdd(b *testing.B) {
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
```

```bash
go test -bench=.
go test -bench=BenchmarkAdd -benchmem
```

---

## 📚 Example Tests

```go
func ExampleAdd() {
    result := Add(2, 3)
    fmt.Println(result)
    // Output: 5
}
// Examples serve as documentation AND tests
```

---

## 🔧 Test Helpers and Setup

```go
func TestMain(m *testing.M) {
    setup()
    code := m.Run()
    teardown()
    os.Exit(code)
}

func TestWithCleanup(t *testing.T) {
    file, _ := os.CreateTemp("", "test")
    t.Cleanup(func() { os.Remove(file.Name()) })
    // Test using file...
}
```

---

## 🎭 Mocking

```go
type UserRepository interface {
    GetByID(id int) (*User, error)
}

type MockUserRepo struct {
    users map[int]*User
}

func (m *MockUserRepo) GetByID(id int) (*User, error) {
    return m.users[id], nil
}

func TestUserService(t *testing.T) {
    mockRepo := &MockUserRepo{users: map[int]*User{1: {Name: "Alice"}}}
    service := &UserService{repo: mockRepo}
    user, _ := service.GetUser(1)
    // Assert user.Name == "Alice"
}
```

---

## 📊 Test Coverage

```bash
go test -cover
go test -coverprofile=coverage.out
go tool cover -func=coverage.out
go tool cover -html=coverage.out
```

---

## 📋 Testing Commands Quick Reference

| Command | Purpose |
|---------|---------|
| go test | Run tests |
| go test -v | Verbose |
| go test -run TestName | Specific test |
| go test -short | Skip long tests |
| go test -race | Race detector |
| go test -cover | Coverage |
| go test -bench=. | Benchmarks |
| go test -timeout=30s | Timeout |

---

## 🎯 Key Takeaways

1. **Test files** end with `_test.go`
2. **Table-driven tests** are the Go way
3. **t.Helper()** for better error locations
4. **t.Cleanup()** for automatic teardown
5. **Benchmarks** use `b.N` loop
6. **Examples** serve as documentation AND tests
7. **httptest** for testing HTTP handlers
8. **Use interfaces** for easy mocking
9. **go test -race** in CI

---

## ➡️ Next Steps

**Review:** [README](./README.md) for the full table of contents
