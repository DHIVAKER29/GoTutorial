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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO HTTP SERVER ARCHITECTURE                                    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  http.ListenAndServe(":8080", handler)                  │    │
│  │         │                        │                      │    │
│  │         ▼                        ▼                      │    │
│  │  ┌───────────┐            ┌─────────────┐               │    │
│  │  │   :8080   │            │   Handler   │               │    │
│  │  │  (port)   │            │ (processes  │               │    │
│  │  │           │            │  requests)  │               │    │
│  │  └───────────┘            └─────────────┘               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  HANDLER INTERFACE:                                             │
│  type Handler interface {                                       │
│      ServeHTTP(w ResponseWriter, r *Request)                    │
│  }                                                              │
│                                                                 │
│  OR use HandlerFunc:                                            │
│  func myHandler(w http.ResponseWriter, r *http.Request) {...}   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Creating an HTTP Server

```go
// server_basics.go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           HTTP SERVER BASICS                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Method 1: DefaultServeMux
    http.HandleFunc("/", homeHandler)
    http.HandleFunc("/hello", helloHandler)
    http.HandleFunc("/json", jsonHandler)
    http.HandleFunc("/user/", userHandler)  // Matches /user/*
    
    // Method 2: Custom ServeMux (recommended)
    mux := http.NewServeMux()
    mux.HandleFunc("/api/health", healthHandler)
    mux.HandleFunc("/api/time", timeHandler)
    
    // Method 3: Custom Server (full control)
    server := &http.Server{
        Addr:         ":8080",
        Handler:      mux,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  120 * time.Second,
    }
    
    fmt.Println("\n📊 Starting server on :8080")
    fmt.Println("   Endpoints:")
    fmt.Println("   • GET /")
    fmt.Println("   • GET /hello?name=Alice")
    fmt.Println("   • GET /json")
    fmt.Println("   • GET /api/health")
    fmt.Println("   • GET /api/time")
    
    log.Fatal(server.ListenAndServe())
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    // Check exact path (ServeMux is prefix-based for /)
    if r.URL.Path != "/" {
        http.NotFound(w, r)
        return
    }
    fmt.Fprintf(w, "Welcome to the homepage!")
}

func helloHandler(w http.ResponseWriter, r *http.Request) {
    name := r.URL.Query().Get("name")
    if name == "" {
        name = "World"
    }
    fmt.Fprintf(w, "Hello, %s!", name)
}

func jsonHandler(w http.ResponseWriter, r *http.Request) {
    data := map[string]interface{}{
        "message": "Hello, JSON!",
        "time":    time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(data)
}

func userHandler(w http.ResponseWriter, r *http.Request) {
    // Extract user ID from path
    userID := r.URL.Path[len("/user/"):]
    fmt.Fprintf(w, "User ID: %s", userID)
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{"status": "ok"})
}

func timeHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{
        "time": time.Now().Format(time.RFC3339),
    })
}
```

---

## 🔀 Handling Different HTTP Methods

```go
// http_methods.go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "net/http"
)

type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
}

var users = map[int]User{
    1: {ID: 1, Name: "Alice"},
    2: {ID: 2, Name: "Bob"},
}

func main() {
    http.HandleFunc("/users", usersHandler)
    http.HandleFunc("/users/", singleUserHandler)
    
    fmt.Println("Server running on :8080")
    http.ListenAndServe(":8080", nil)
}

func usersHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    
    switch r.Method {
    case http.MethodGet:
        // GET /users - List all users
        userList := make([]User, 0, len(users))
        for _, u := range users {
            userList = append(userList, u)
        }
        json.NewEncoder(w).Encode(userList)
        
    case http.MethodPost:
        // POST /users - Create user
        body, _ := io.ReadAll(r.Body)
        var user User
        if err := json.Unmarshal(body, &user); err != nil {
            http.Error(w, err.Error(), http.StatusBadRequest)
            return
        }
        user.ID = len(users) + 1
        users[user.ID] = user
        
        w.WriteHeader(http.StatusCreated)
        json.NewEncoder(w).Encode(user)
        
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

func singleUserHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    
    // Parse user ID
    var id int
    _, err := fmt.Sscanf(r.URL.Path, "/users/%d", &id)
    if err != nil {
        http.Error(w, "Invalid user ID", http.StatusBadRequest)
        return
    }
    
    switch r.Method {
    case http.MethodGet:
        // GET /users/{id}
        user, ok := users[id]
        if !ok {
            http.Error(w, "User not found", http.StatusNotFound)
            return
        }
        json.NewEncoder(w).Encode(user)
        
    case http.MethodPut:
        // PUT /users/{id}
        body, _ := io.ReadAll(r.Body)
        var user User
        json.Unmarshal(body, &user)
        user.ID = id
        users[id] = user
        json.NewEncoder(w).Encode(user)
        
    case http.MethodDelete:
        // DELETE /users/{id}
        delete(users, id)
        w.WriteHeader(http.StatusNoContent)
        
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}
```

---

## 🔗 Middleware Pattern

```go
// middleware.go
package main

import (
    "fmt"
    "log"
    "net/http"
    "time"
)

// Middleware type
type Middleware func(http.Handler) http.Handler

// Logging middleware
func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // Call next handler
        next.ServeHTTP(w, r)
        
        // Log after request completes
        log.Printf("%s %s %v", r.Method, r.URL.Path, time.Since(start))
    })
}

// Auth middleware
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        
        if token != "Bearer valid-token" {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }
        
        // Token valid, proceed
        next.ServeHTTP(w, r)
    })
}

// Recovery middleware (catch panics)
func RecoveryMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if err := recover(); err != nil {
                log.Printf("Panic recovered: %v", err)
                http.Error(w, "Internal Server Error", http.StatusInternalServerError)
            }
        }()
        
        next.ServeHTTP(w, r)
    })
}

// Chain middlewares
func Chain(handler http.Handler, middlewares ...Middleware) http.Handler {
    for i := len(middlewares) - 1; i >= 0; i-- {
        handler = middlewares[i](handler)
    }
    return handler
}

func main() {
    // Create handler
    hello := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, World!")
    })
    
    // Apply middlewares
    handler := Chain(hello,
        RecoveryMiddleware,
        LoggingMiddleware,
        // AuthMiddleware,  // Uncomment to require auth
    )
    
    http.Handle("/", handler)
    
    fmt.Println("Server with middleware on :8080")
    http.ListenAndServe(":8080", nil)
}
```

---

## 📤 HTTP Client

```go
// http_client.go
package main

import (
    "bytes"
    "context"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           HTTP CLIENT                                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Method 1: Simple GET
    fmt.Println("\n📊 Simple GET (http.Get):")
    resp, err := http.Get("https://httpbin.org/get")
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        defer resp.Body.Close()
        fmt.Printf("   Status: %s\n", resp.Status)
    }
    
    // Method 2: Custom client with timeout
    fmt.Println("\n📊 Custom Client with Timeout:")
    client := &http.Client{
        Timeout: 10 * time.Second,
    }
    
    resp, err = client.Get("https://httpbin.org/get")
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        defer resp.Body.Close()
        body, _ := io.ReadAll(resp.Body)
        fmt.Printf("   Response length: %d bytes\n", len(body))
    }
    
    // Method 3: POST with JSON body
    fmt.Println("\n📊 POST with JSON:")
    postJSON()
    
    // Method 4: Request with context
    fmt.Println("\n📊 Request with Context (cancellation):")
    requestWithContext()
    
    // Method 5: Custom request
    fmt.Println("\n📊 Custom Request (headers, etc.):")
    customRequest()
}

func postJSON() {
    data := map[string]string{"name": "Alice", "email": "alice@example.com"}
    jsonData, _ := json.Marshal(data)
    
    resp, err := http.Post(
        "https://httpbin.org/post",
        "application/json",
        bytes.NewBuffer(jsonData),
    )
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    defer resp.Body.Close()
    
    fmt.Printf("   Status: %s\n", resp.Status)
}

func requestWithContext() {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    req, _ := http.NewRequestWithContext(ctx, "GET", "https://httpbin.org/delay/2", nil)
    
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    defer resp.Body.Close()
    
    fmt.Printf("   Status: %s\n", resp.Status)
}

func customRequest() {
    req, _ := http.NewRequest("GET", "https://httpbin.org/headers", nil)
    
    // Add headers
    req.Header.Set("Authorization", "Bearer token123")
    req.Header.Set("X-Custom-Header", "custom-value")
    req.Header.Set("Accept", "application/json")
    
    client := &http.Client{Timeout: 10 * time.Second}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    defer resp.Body.Close()
    
    fmt.Printf("   Status: %s\n", resp.Status)
}
```

---

## 🏭 Production Server

```go
// production_server.go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION HTTP SERVER                         ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Create router
    mux := http.NewServeMux()
    mux.HandleFunc("/health", healthCheck)
    mux.HandleFunc("/api/", apiHandler)
    
    // Create server with timeouts
    server := &http.Server{
        Addr:         ":8080",
        Handler:      mux,
        ReadTimeout:  5 * time.Second,   // Max time to read request
        WriteTimeout: 10 * time.Second,  // Max time to write response
        IdleTimeout:  120 * time.Second, // Max time for keep-alive
    }
    
    // Graceful shutdown
    go func() {
        sigChan := make(chan os.Signal, 1)
        signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
        <-sigChan
        
        log.Println("Shutting down gracefully...")
        
        ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
        defer cancel()
        
        if err := server.Shutdown(ctx); err != nil {
            log.Fatalf("Shutdown error: %v", err)
        }
    }()
    
    log.Printf("Starting server on %s", server.Addr)
    if err := server.ListenAndServe(); err != http.ErrServerClosed {
        log.Fatalf("Server error: %v", err)
    }
    
    log.Println("Server stopped")
}

func healthCheck(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{
        "status": "healthy",
        "time":   time.Now().Format(time.RFC3339),
    })
}

func apiHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{
        "message": "API endpoint",
        "path":    r.URL.Path,
    })
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
8. **Don't use default client** - always set timeouts

---

## ➡️ Next Steps

**Next Topic:** [36 - Testing](./36-testing.md)

