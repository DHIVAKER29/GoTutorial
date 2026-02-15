# 35 - HTTP Clients and Servers

> Building web servers and making HTTP requests in Go.

---

## 📌 What You'll Learn

- Creating HTTP servers (the standard library way)
- HTTP handlers and routing
- Making HTTP requests (clients)
- Middleware patterns
- Context and timeouts
- Production best practices

---

## 🌐 HTTP Server Basics

**Architecture:**
- `http.ListenAndServe(":8080", handler)` — binds to port and routes requests to handler
- Handler processes each request

**Handler interface:**
```go
type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}
```

**Or use HandlerFunc:**
```go
func myHandler(w http.ResponseWriter, r *http.Request) { ... }
```

---

## 📝 Creating an HTTP Server

**Method 1: DefaultServeMux**

```go
http.HandleFunc("/", homeHandler)
http.HandleFunc("/hello", helloHandler)
http.ListenAndServe(":8080", nil)
// Output: Server starts on :8080
```

**Method 2: Custom ServeMux (recommended)**

```go
mux := http.NewServeMux()
mux.HandleFunc("/api/health", healthHandler)
mux.HandleFunc("/api/time", timeHandler)
http.ListenAndServe(":8080", mux)
```

**Method 3: Custom Server (full control)**

```go
server := &http.Server{
    Addr:         ":8080",
    Handler:      mux,
    ReadTimeout:  10 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  120 * time.Second,
}
server.ListenAndServe()
```

**Simple handler examples:**

```go
func helloHandler(w http.ResponseWriter, r *http.Request) {
    name := r.URL.Query().Get("name")
    if name == "" { name = "World" }
    fmt.Fprintf(w, "Hello, %s!", name)
}

func jsonHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{"message": "Hello, JSON!"})
}
```

### Complete Example: HTTP Server

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "time"
)

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Welcome!")
    })
    mux.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
        name := r.URL.Query().Get("name")
        if name == "" { name = "World" }
        fmt.Fprintf(w, "Hello, %s!", name)
    })
    mux.HandleFunc("/json", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(map[string]interface{}{
            "message": "Hello, JSON!",
            "time":    time.Now().Format(time.RFC3339),
        })
    })

    server := &http.Server{
        Addr:         ":8080",
        Handler:      mux,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
    }
    fmt.Println("Server on :8080 — GET /, /hello?name=Alice, /json")
    server.ListenAndServe()
}
```

**Output:**
```
Server on :8080 — GET /, /hello?name=Alice, /json
(Server starts listening and blocks)
```

---

## 🔀 Handling Different HTTP Methods

```go
switch r.Method {
case http.MethodGet:
    // List resources
    json.NewEncoder(w).Encode(users)
case http.MethodPost:
    body, _ := io.ReadAll(r.Body)
    var user User
    json.Unmarshal(body, &user)
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(user)
default:
    http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
}
```

**Path parameters:**

```go
var id int
fmt.Sscanf(r.URL.Path, "/users/%d", &id)
```

---

## 🔗 Middleware Pattern

```go
type Middleware func(http.Handler) http.Handler

func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)
        log.Printf("%s %s %v", r.Method, r.URL.Path, time.Since(start))
    })
}

func Chain(handler http.Handler, middlewares ...Middleware) http.Handler {
    for i := len(middlewares) - 1; i >= 0; i-- {
        handler = middlewares[i](handler)
    }
    return handler
}
```

**Usage:**
```go
handler := Chain(helloHandler, RecoveryMiddleware, LoggingMiddleware)
http.Handle("/", handler)
```

---

## 📤 HTTP Client

**Simple GET:**
```go
resp, err := http.Get("https://httpbin.org/get")
defer resp.Body.Close()
fmt.Printf("Status: %s\n", resp.Status)
// Output: Status: 200 OK
```

**Custom client with timeout:**
```go
client := &http.Client{Timeout: 10 * time.Second}
resp, err := client.Get("https://httpbin.org/get")
```

**POST with JSON:**
```go
data := map[string]string{"name": "Alice"}
jsonData, _ := json.Marshal(data)
resp, err := http.Post("https://httpbin.org/post", "application/json", bytes.NewBuffer(jsonData))
```

**Request with context (cancellation):**
```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
req, _ := http.NewRequestWithContext(ctx, "GET", "https://httpbin.org/delay/2", nil)
resp, err := client.Do(req)
```

**Custom headers:**
```go
req, _ := http.NewRequest("GET", "https://httpbin.org/headers", nil)
req.Header.Set("Authorization", "Bearer token123")
req.Header.Set("Accept", "application/json")
resp, err := client.Do(req)
```

### Complete Example: HTTP Client

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "time"
)

func main() {
    resp, err := http.Get("https://httpbin.org/get")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    defer resp.Body.Close()
    fmt.Printf("GET Status: %s\n", resp.Status)

    data := map[string]string{"name": "Alice"}
    jsonData, _ := json.Marshal(data)
    resp2, _ := http.Post("https://httpbin.org/post", "application/json", bytes.NewBuffer(jsonData))
    defer resp2.Body.Close()
    body, _ := io.ReadAll(resp2.Body)
    fmt.Printf("POST Status: %s, Body length: %d bytes\n", resp2.Status, len(body))
}
```

**Output:**
```
GET Status: 200 OK
POST Status: 200 OK, Body length: [varies] bytes
```

---

## 🏭 Production Server

**Graceful shutdown:**
```go
go func() {
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
    <-sigChan

    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    server.Shutdown(ctx)
}()

server.ListenAndServe()
```

**Server with timeouts:**
```go
server := &http.Server{
    Addr:         ":8080",
    Handler:      mux,
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
    IdleTimeout:  120 * time.Second,
}
```

---

## 🎯 Key Takeaways

1. **http.HandleFunc** for simple handlers
2. **http.ServeMux** for routing
3. **http.Server** for production (with timeouts!)
4. **Middleware** wraps handlers for cross-cutting concerns
5. **http.Client** needs timeouts in production
6. **Context** for request cancellation
7. **Graceful shutdown** for production servers
8. **Don't use default client** — always set timeouts

---

## ➡️ Next Steps

**Next Topic:** [36 - Testing](./36-testing.md)
