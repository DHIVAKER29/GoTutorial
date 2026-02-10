# 34 - JSON Encoding and Decoding

> Working with JSON data in Go - marshaling, unmarshaling, and best practices.

---

## 📌 What You'll Learn

- JSON encoding (marshaling) and decoding (unmarshaling)
- Struct tags for JSON field mapping
- Handling optional fields, omitempty, and custom types
- Working with dynamic/unknown JSON
- Streaming JSON for large data
- Common patterns and gotchas

---

## 🤔 JSON in Go

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JSON WORKFLOW IN GO                                            │
│                                                                 │
│  MARSHALING (Go → JSON):                                        │
│  ┌─────────────┐    json.Marshal()    ┌─────────────────┐      │
│  │ Go Struct   │ ─────────────────► │ JSON []byte     │      │
│  │ or map      │                      │ string          │      │
│  └─────────────┘                      └─────────────────┘      │
│                                                                 │
│  UNMARSHALING (JSON → Go):                                      │
│  ┌─────────────────┐  json.Unmarshal()  ┌─────────────┐        │
│  │ JSON []byte     │ ─────────────────► │ Go Struct   │        │
│  │ string          │                     │ or map      │        │
│  └─────────────────┘                     └─────────────┘        │
│                                                                 │
│  KEY POINTS:                                                    │
│  • Only EXPORTED fields (Uppercase) are marshaled               │
│  • Struct tags control JSON field names                         │
│  • json.Marshal returns []byte, not string                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Basic Marshaling (Go → JSON)

```go
// marshal_basics.go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

type Person struct {
    Name    string    `json:"name"`
    Age     int       `json:"age"`
    Email   string    `json:"email"`
    Created time.Time `json:"created_at"`
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           JSON MARSHALING (Go → JSON)                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic struct to JSON
    fmt.Println("\n📊 Basic Marshaling:")
    person := Person{
        Name:    "Alice",
        Age:     30,
        Email:   "alice@example.com",
        Created: time.Now(),
    }
    
    jsonBytes, err := json.Marshal(person)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    fmt.Printf("   JSON: %s\n", jsonBytes)
    
    // Pretty print with MarshalIndent
    fmt.Println("\n📊 Pretty Print (MarshalIndent):")
    prettyJSON, _ := json.MarshalIndent(person, "", "  ")
    fmt.Printf("%s\n", prettyJSON)
    
    // Map to JSON
    fmt.Println("\n📊 Map to JSON:")
    data := map[string]interface{}{
        "name":   "Bob",
        "age":    25,
        "active": true,
        "scores": []int{85, 90, 95},
    }
    mapJSON, _ := json.Marshal(data)
    fmt.Printf("   %s\n", mapJSON)
    
    // Slice to JSON
    fmt.Println("\n📊 Slice to JSON:")
    people := []Person{
        {Name: "Alice", Age: 30},
        {Name: "Bob", Age: 25},
    }
    sliceJSON, _ := json.Marshal(people)
    fmt.Printf("   %s\n", sliceJSON)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           JSON MARSHALING (Go → JSON)                     ║
╚══════════════════════════════════════════════════════════╝

📊 Basic Marshaling:
   JSON: {"name":"Alice","age":30,"email":"alice@example.com","created_at":"2024-12-10T10:30:45.123456789Z"}

📊 Pretty Print (MarshalIndent):
{
  "name": "Alice",
  "age": 30,
  "email": "alice@example.com",
  "created_at": "2024-12-10T10:30:45.123456789Z"
}

📊 Map to JSON:
   {"active":true,"age":25,"name":"Bob","scores":[85,90,95]}

📊 Slice to JSON:
   [{"name":"Alice","age":30,"email":"","created_at":"0001-01-01T00:00:00Z"},{"name":"Bob","age":25,"email":"","created_at":"0001-01-01T00:00:00Z"}]
```

---

## 📥 Basic Unmarshaling (JSON → Go)

```go
// unmarshal_basics.go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    ID       int      `json:"id"`
    Name     string   `json:"name"`
    Email    string   `json:"email"`
    Active   bool     `json:"active"`
    Roles    []string `json:"roles"`
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           JSON UNMARSHALING (JSON → Go)                   ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // JSON string to struct
    fmt.Println("\n📊 JSON to Struct:")
    jsonStr := `{
        "id": 1,
        "name": "Alice",
        "email": "alice@example.com",
        "active": true,
        "roles": ["admin", "user"]
    }`
    
    var user User
    err := json.Unmarshal([]byte(jsonStr), &user)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    fmt.Printf("   User: %+v\n", user)
    fmt.Printf("   Name: %s\n", user.Name)
    fmt.Printf("   Roles: %v\n", user.Roles)
    
    // JSON to map (dynamic)
    fmt.Println("\n📊 JSON to Map (dynamic):")
    var data map[string]interface{}
    json.Unmarshal([]byte(jsonStr), &data)
    fmt.Printf("   Map: %v\n", data)
    fmt.Printf("   Name: %v\n", data["name"])
    
    // JSON array to slice
    fmt.Println("\n📊 JSON Array to Slice:")
    jsonArray := `[
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"}
    ]`
    var users []User
    json.Unmarshal([]byte(jsonArray), &users)
    for _, u := range users {
        fmt.Printf("   User: %s\n", u.Name)
    }
    
    // Extra fields are ignored
    fmt.Println("\n📊 Extra JSON Fields (ignored):")
    jsonExtra := `{"id": 1, "name": "Alice", "unknown_field": "ignored"}`
    var u User
    json.Unmarshal([]byte(jsonExtra), &u)
    fmt.Printf("   User: %+v (unknown_field ignored)\n", u)
    
    // Missing fields get zero values
    fmt.Println("\n📊 Missing JSON Fields (zero values):")
    jsonPartial := `{"id": 1, "name": "Alice"}`
    var u2 User
    json.Unmarshal([]byte(jsonPartial), &u2)
    fmt.Printf("   User: %+v\n", u2)
    fmt.Printf("   Active: %t (zero value)\n", u2.Active)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           JSON UNMARSHALING (JSON → Go)                   ║
╚══════════════════════════════════════════════════════════╝

📊 JSON to Struct:
   User: {ID:1 Name:Alice Email:alice@example.com Active:true Roles:[admin user]}
   Name: Alice
   Roles: [admin user]

📊 JSON to Map (dynamic):
   Map: map[active:true email:alice@example.com id:1 name:Alice roles:[admin user]]
   Name: Alice

📊 JSON Array to Slice:
   User: Alice
   User: Bob

📊 Extra JSON Fields (ignored):
   User: {ID:1 Name:Alice Email: Active:false Roles:[]} (unknown_field ignored)

📊 Missing JSON Fields (zero values):
   User: {ID:1 Name:Alice Email: Active:false Roles:[]}
   Active: false (zero value)
```

---

## 🏷️ Struct Tags Deep Dive

```go
// struct_tags.go
package main

import (
    "encoding/json"
    "fmt"
)

type Example struct {
    // Basic: field name in JSON
    Name string `json:"name"`
    
    // Different JSON name
    Age int `json:"user_age"`
    
    // Omit if zero value
    Score int `json:"score,omitempty"`
    
    // Always omit from JSON
    Password string `json:"-"`
    
    // Keep field name but allow omitempty
    Email string `json:",omitempty"`
    
    // String encoding for numbers
    ID int64 `json:"id,string"`
    
    // Unexported = never in JSON
    internal string
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           STRUCT TAGS DEEP DIVE                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Tag syntax
    fmt.Println("\n📊 Tag Syntax:")
    fmt.Println("   `json:\"field_name\"`           - Custom JSON name")
    fmt.Println("   `json:\"field_name,omitempty\"` - Omit if zero")
    fmt.Println("   `json:\"-\"`                    - Always omit")
    fmt.Println("   `json:\",omitempty\"`           - Keep name, omit if zero")
    fmt.Println("   `json:\"id,string\"`            - Encode number as string")
    
    // Demonstration
    fmt.Println("\n📊 Omitempty Behavior:")
    
    withValues := Example{
        Name:     "Alice",
        Age:      30,
        Score:    95,
        Password: "secret",
        Email:    "alice@example.com",
        ID:       12345,
    }
    
    withZeros := Example{
        Name:     "Bob",
        Age:      0,
        Score:    0,      // Will be omitted!
        Password: "pwd",  // Will be omitted (always)!
        Email:    "",     // Will be omitted!
        ID:       0,
    }
    
    json1, _ := json.MarshalIndent(withValues, "   ", "  ")
    fmt.Println("   With values:")
    fmt.Printf("%s\n", json1)
    
    json2, _ := json.MarshalIndent(withZeros, "   ", "  ")
    fmt.Println("\n   With zeros (omitempty in action):")
    fmt.Printf("%s\n", json2)
    
    // String encoding for numbers
    fmt.Println("\n📊 String Encoding (JavaScript big int issue):")
    fmt.Println("   Problem: JavaScript can't handle int64 precisely")
    fmt.Println("   Solution: `json:\"id,string\"` encodes as \"12345\"")
    
    type BigID struct {
        ID int64 `json:"id,string"`
    }
    b := BigID{ID: 9007199254740993}
    jsonB, _ := json.Marshal(b)
    fmt.Printf("   Result: %s\n", jsonB)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           STRUCT TAGS DEEP DIVE                           ║
╚══════════════════════════════════════════════════════════╝

📊 Tag Syntax:
   `json:"field_name"`           - Custom JSON name
   `json:"field_name,omitempty"` - Omit if zero
   `json:"-"`                    - Always omit
   `json:",omitempty"`           - Keep name, omit if zero
   `json:"id,string"`            - Encode number as string

📊 Omitempty Behavior:
   With values:
   {
     "name": "Alice",
     "user_age": 30,
     "score": 95,
     "id": "12345"
   }

   With zeros (omitempty in action):
   {
     "name": "Bob",
     "user_age": 0
   }

📊 String Encoding (JavaScript big int issue):
   Problem: JavaScript can't handle int64 precisely
   Solution: `json:"id,string"` encodes as "12345"
   Result: {"id":"9007199254740993"}
```

---

## 🔄 Custom JSON Encoding

```go
// custom_encoding.go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

// Custom type with MarshalJSON
type Status int

const (
    StatusPending Status = iota
    StatusActive
    StatusCompleted
)

func (s Status) MarshalJSON() ([]byte, error) {
    var str string
    switch s {
    case StatusPending:
        str = "pending"
    case StatusActive:
        str = "active"
    case StatusCompleted:
        str = "completed"
    default:
        str = "unknown"
    }
    return json.Marshal(str)
}

func (s *Status) UnmarshalJSON(data []byte) error {
    var str string
    if err := json.Unmarshal(data, &str); err != nil {
        return err
    }
    switch str {
    case "pending":
        *s = StatusPending
    case "active":
        *s = StatusActive
    case "completed":
        *s = StatusCompleted
    }
    return nil
}

// Custom time format
type CustomTime struct {
    time.Time
}

const customFormat = "2006-01-02"

func (ct CustomTime) MarshalJSON() ([]byte, error) {
    return json.Marshal(ct.Time.Format(customFormat))
}

func (ct *CustomTime) UnmarshalJSON(data []byte) error {
    var s string
    if err := json.Unmarshal(data, &s); err != nil {
        return err
    }
    t, err := time.Parse(customFormat, s)
    if err != nil {
        return err
    }
    ct.Time = t
    return nil
}

type Task struct {
    Name    string     `json:"name"`
    Status  Status     `json:"status"`
    DueDate CustomTime `json:"due_date"`
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CUSTOM JSON ENCODING                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Custom marshaling
    fmt.Println("\n📊 Custom Marshaling:")
    task := Task{
        Name:    "Complete project",
        Status:  StatusActive,
        DueDate: CustomTime{time.Now()},
    }
    
    jsonData, _ := json.MarshalIndent(task, "   ", "  ")
    fmt.Printf("%s\n", jsonData)
    
    // Custom unmarshaling
    fmt.Println("\n📊 Custom Unmarshaling:")
    jsonStr := `{"name":"Review code","status":"pending","due_date":"2024-12-31"}`
    
    var task2 Task
    json.Unmarshal([]byte(jsonStr), &task2)
    fmt.Printf("   Task: %+v\n", task2)
    fmt.Printf("   Status int value: %d\n", task2.Status)
    
    // Implementing interfaces
    fmt.Println("\n📊 Interfaces to Implement:")
    fmt.Println("   json.Marshaler:")
    fmt.Println("     MarshalJSON() ([]byte, error)")
    fmt.Println("")
    fmt.Println("   json.Unmarshaler:")
    fmt.Println("     UnmarshalJSON([]byte) error")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           CUSTOM JSON ENCODING                            ║
╚══════════════════════════════════════════════════════════╝

📊 Custom Marshaling:
   {
     "name": "Complete project",
     "status": "active",
     "due_date": "2024-12-10"
   }

📊 Custom Unmarshaling:
   Task: {Name:Review code Status:0 DueDate:{Time:2024-12-31 00:00:00 +0000 UTC}}
   Status int value: 0

📊 Interfaces to Implement:
   json.Marshaler:
     MarshalJSON() ([]byte, error)

   json.Unmarshaler:
     UnmarshalJSON([]byte) error
```

---

## 🌊 Streaming JSON (Large Data)

```go
// streaming_json.go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "strings"
)

type Record struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           STREAMING JSON                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Encoder - write to stream
    fmt.Println("\n📊 json.Encoder (write to stream):")
    var buf bytes.Buffer
    encoder := json.NewEncoder(&buf)
    encoder.SetIndent("", "  ")
    
    records := []Record{
        {ID: 1, Name: "Alice"},
        {ID: 2, Name: "Bob"},
        {ID: 3, Name: "Charlie"},
    }
    
    for _, r := range records {
        encoder.Encode(r)  // Writes directly, no intermediate []byte
    }
    fmt.Printf("%s\n", buf.String())
    
    // Decoder - read from stream
    fmt.Println("📊 json.Decoder (read from stream):")
    jsonStream := `
        {"id": 1, "name": "Alice"}
        {"id": 2, "name": "Bob"}
        {"id": 3, "name": "Charlie"}
    `
    
    decoder := json.NewDecoder(strings.NewReader(jsonStream))
    
    for decoder.More() {
        var r Record
        if err := decoder.Decode(&r); err != nil {
            break
        }
        fmt.Printf("   Decoded: %+v\n", r)
    }
    
    // When to use streaming
    fmt.Println("\n💡 When to Use Streaming:")
    fmt.Println("   • Large JSON files (won't fit in memory)")
    fmt.Println("   • HTTP request/response bodies")
    fmt.Println("   • Newline-delimited JSON (NDJSON)")
    fmt.Println("   • Processing JSON as it arrives")
    
    // Encoder.Encode vs json.Marshal
    fmt.Println("\n📊 Encoder vs Marshal:")
    fmt.Println("   json.Marshal()    → returns []byte, allocates memory")
    fmt.Println("   encoder.Encode()  → writes to io.Writer, streams")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           STREAMING JSON                                  ║
╚══════════════════════════════════════════════════════════╝

📊 json.Encoder (write to stream):
{
  "id": 1,
  "name": "Alice"
}
{
  "id": 2,
  "name": "Bob"
}
{
  "id": 3,
  "name": "Charlie"
}
📊 json.Decoder (read from stream):
   Decoded: {ID:1 Name:Alice}
   Decoded: {ID:2 Name:Bob}
   Decoded: {ID:3 Name:Charlie}

💡 When to Use Streaming:
   • Large JSON files (won't fit in memory)
   • HTTP request/response bodies
   • Newline-delimited JSON (NDJSON)
   • Processing JSON as it arrives

📊 Encoder vs Marshal:
   json.Marshal()    → returns []byte, allocates memory
   encoder.Encode()  → writes to io.Writer, streams
```

---

## 🔧 Working with Dynamic JSON

```go
// dynamic_json.go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           DYNAMIC JSON                                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Unknown structure - use map[string]interface{}
    fmt.Println("\n📊 Unknown Structure (map[string]interface{}):")
    jsonStr := `{
        "name": "Alice",
        "age": 30,
        "address": {
            "city": "NYC",
            "zip": "10001"
        },
        "tags": ["admin", "user"]
    }`
    
    var data map[string]interface{}
    json.Unmarshal([]byte(jsonStr), &data)
    
    // Accessing values (type assertions needed!)
    name := data["name"].(string)
    age := data["age"].(float64)  // Numbers are float64!
    address := data["address"].(map[string]interface{})
    city := address["city"].(string)
    
    fmt.Printf("   Name: %s\n", name)
    fmt.Printf("   Age: %.0f\n", age)
    fmt.Printf("   City: %s\n", city)
    
    // json.RawMessage - delay parsing
    fmt.Println("\n📊 json.RawMessage (delay parsing):")
    type Response struct {
        Type string          `json:"type"`
        Data json.RawMessage `json:"data"`  // Parse later!
    }
    
    jsonResp := `{"type": "user", "data": {"id": 1, "name": "Bob"}}`
    var resp Response
    json.Unmarshal([]byte(jsonResp), &resp)
    
    fmt.Printf("   Type: %s\n", resp.Type)
    fmt.Printf("   Raw data: %s\n", resp.Data)
    
    // Now parse based on type
    if resp.Type == "user" {
        var user struct {
            ID   int    `json:"id"`
            Name string `json:"name"`
        }
        json.Unmarshal(resp.Data, &user)
        fmt.Printf("   Parsed user: %+v\n", user)
    }
    
    // json.Number for precise numbers
    fmt.Println("\n📊 json.Number (precise numbers):")
    type Precise struct {
        Amount json.Number `json:"amount"`
    }
    
    jsonNum := `{"amount": "9007199254740993"}`
    decoder := json.NewDecoder(strings.NewReader(jsonNum))
    decoder.UseNumber()  // Enable json.Number
    
    var p Precise
    decoder.Decode(&p)
    fmt.Printf("   Amount as string: %s\n", p.Amount)
    
    intVal, _ := p.Amount.Int64()
    fmt.Printf("   Amount as int64: %d\n", intVal)
}

import "strings"
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           DYNAMIC JSON                                    ║
╚══════════════════════════════════════════════════════════╝

📊 Unknown Structure (map[string]interface{}):
   Name: Alice
   Age: 30
   City: NYC

📊 json.RawMessage (delay parsing):
   Type: user
   Raw data: {"id": 1, "name": "Bob"}
   Parsed user: {ID:1 Name:Bob}

📊 json.Number (precise numbers):
   Amount as string: 9007199254740993
   Amount as int64: 9007199254740993
```

---

## 🏭 Production Patterns

```go
// json_production.go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
)

// API Response wrapper
type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   string      `json:"error,omitempty"`
}

// User model
type User struct {
    ID       int    `json:"id"`
    Name     string `json:"name"`
    Email    string `json:"email"`
    Password string `json:"-"`  // Never in JSON!
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION JSON PATTERNS                        ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: API response wrapper
    fmt.Println("\n📊 Pattern 1: API Response Wrapper")
    
    successResp := APIResponse{
        Success: true,
        Data:    User{ID: 1, Name: "Alice", Email: "alice@example.com"},
    }
    json1, _ := json.MarshalIndent(successResp, "   ", "  ")
    fmt.Printf("%s\n", json1)
    
    errorResp := APIResponse{
        Success: false,
        Error:   "User not found",
    }
    json2, _ := json.MarshalIndent(errorResp, "   ", "  ")
    fmt.Printf("%s\n", json2)
    
    // Pattern 2: Validate required fields
    fmt.Println("\n📊 Pattern 2: Pointer for Optional vs Required")
    type CreateUser struct {
        Name  string  `json:"name"`            // Required (zero = invalid)
        Email string  `json:"email"`           // Required
        Age   *int    `json:"age,omitempty"`   // Optional (nil = not provided)
        Bio   *string `json:"bio,omitempty"`   // Optional
    }
    
    jsonOptional := `{"name": "Bob", "email": "bob@example.com"}`
    var cu CreateUser
    json.Unmarshal([]byte(jsonOptional), &cu)
    
    if cu.Age == nil {
        fmt.Println("   Age was not provided (optional)")
    }
    
    // Pattern 3: HTTP handler
    fmt.Println("\n📊 Pattern 3: HTTP JSON Handler (conceptual)")
    fmt.Println("   func handler(w http.ResponseWriter, r *http.Request) {")
    fmt.Println("       // Decode request")
    fmt.Println("       var req RequestBody")
    fmt.Println("       if err := json.NewDecoder(r.Body).Decode(&req); err != nil {")
    fmt.Println("           http.Error(w, err.Error(), http.StatusBadRequest)")
    fmt.Println("           return")
    fmt.Println("       }")
    fmt.Println("")
    fmt.Println("       // Process...")
    fmt.Println("")
    fmt.Println("       // Encode response")
    fmt.Println("       w.Header().Set(\"Content-Type\", \"application/json\")")
    fmt.Println("       json.NewEncoder(w).Encode(response)")
    fmt.Println("   }")
    
    _ = http.NewRequest  // Suppress unused import
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           PRODUCTION JSON PATTERNS                        ║
╚══════════════════════════════════════════════════════════╝

📊 Pattern 1: API Response Wrapper
   {
     "success": true,
     "data": {
       "id": 1,
       "name": "Alice",
       "email": "alice@example.com"
     }
   }
   {
     "success": false,
     "error": "User not found"
   }

📊 Pattern 2: Pointer for Optional vs Required
   Age was not provided (optional)

📊 Pattern 3: HTTP JSON Handler (conceptual)
   func handler(w http.ResponseWriter, r *http.Request) {
       // Decode request
       var req RequestBody
       if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
           http.Error(w, err.Error(), http.StatusBadRequest)
           return
       }

       // Process...

       // Encode response
       w.Header().Set("Content-Type", "application/json")
       json.NewEncoder(w).Encode(response)
   }
```

---

## 🎯 Key Takeaways

1. **Marshal** (Go → JSON), **Unmarshal** (JSON → Go)
2. **Only exported fields** (Uppercase) are encoded
3. **Struct tags** control JSON field names: `` `json:"name"` ``
4. **omitempty** skips zero values
5. **`json:"-"`** always excludes field
6. **Numbers in interface{}** are `float64`
7. **Use pointers** to distinguish missing vs zero
8. **json.Decoder/Encoder** for streaming
9. **json.RawMessage** for delayed parsing

---

## ➡️ Next Steps

**Next Topic:** [35 - HTTP Clients and Servers](./35-http.md)

