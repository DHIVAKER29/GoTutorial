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

**JSON workflow:**

| Direction | Function | Description |
|-----------|----------|-------------|
| Go → JSON | `json.Marshal()` | Converts Go struct or map to JSON `[]byte` |
| JSON → Go | `json.Unmarshal()` | Converts JSON `[]byte` to Go struct or map |

**Key points:**
- Only **exported fields** (uppercase) are marshaled
- Struct tags control JSON field names
- `json.Marshal` returns `[]byte`, not `string`

---

## 📝 Basic Marshaling (Go → JSON)

**Struct to JSON:**

```go
type Person struct {
    Name  string `json:"name"`
    Age   int    `json:"age"`
}
person := Person{Name: "Alice", Age: 30}
jsonBytes, _ := json.Marshal(person)
fmt.Printf("%s\n", jsonBytes)
// Output: {"name":"Alice","age":30}
```

**Pretty print with MarshalIndent:**

```go
prettyJSON, _ := json.MarshalIndent(person, "", "  ")
fmt.Printf("%s\n", prettyJSON)
// Output:
// {
//   "name": "Alice",
//   "age": 30
// }
```

**Map to JSON:**

```go
data := map[string]interface{}{"name": "Bob", "age": 25, "active": true}
mapJSON, _ := json.Marshal(data)
fmt.Printf("%s\n", mapJSON)
// Output: {"active":true,"age":25,"name":"Bob"}
```

**Slice to JSON:**

```go
people := []Person{{Name: "Alice", Age: 30}, {Name: "Bob", Age: 25}}
sliceJSON, _ := json.Marshal(people)
fmt.Printf("%s\n", sliceJSON)
// Output: [{"name":"Alice","age":30},{"name":"Bob","age":25}]
```

### Complete Example: Basic Marshaling

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Person struct {
    Name  string `json:"name"`
    Age   int    `json:"age"`
    Email string `json:"email"`
}

func main() {
    person := Person{Name: "Alice", Age: 30, Email: "alice@example.com"}
    jsonBytes, _ := json.Marshal(person)
    fmt.Printf("JSON: %s\n", jsonBytes)

    prettyJSON, _ := json.MarshalIndent(person, "", "  ")
    fmt.Printf("Pretty:\n%s\n", prettyJSON)
}
```

**Output:**
```
JSON: {"name":"Alice","age":30,"email":"alice@example.com"}
Pretty:
{
  "name": "Alice",
  "age": 30,
  "email": "alice@example.com"
}
```

---

## 📥 Basic Unmarshaling (JSON → Go)

**JSON to struct:**

```go
type User struct {
    ID    int      `json:"id"`
    Name  string   `json:"name"`
    Roles []string `json:"roles"`
}
jsonStr := `{"id": 1, "name": "Alice", "roles": ["admin", "user"]}`
var user User
json.Unmarshal([]byte(jsonStr), &user)
fmt.Printf("User: %+v\n", user)
// Output: User: {ID:1 Name:Alice Roles:[admin user]}
```

**JSON to map (dynamic):**

```go
var data map[string]interface{}
json.Unmarshal([]byte(jsonStr), &data)
fmt.Printf("Name: %v\n", data["name"])
// Output: Name: Alice
```

**Extra fields are ignored; missing fields get zero values.**

### Complete Example: Basic Unmarshaling

```go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    ID     int      `json:"id"`
    Name   string   `json:"name"`
    Email  string   `json:"email"`
    Active bool     `json:"active"`
    Roles  []string `json:"roles"`
}

func main() {
    jsonStr := `{"id": 1, "name": "Alice", "email": "alice@example.com", "active": true, "roles": ["admin", "user"]}`
    var user User
    json.Unmarshal([]byte(jsonStr), &user)
    fmt.Printf("User: %+v\n", user)
    fmt.Printf("Roles: %v\n", user.Roles)
}
```

**Output:**
```
User: {ID:1 Name:Alice Email:alice@example.com Active:true Roles:[admin user]}
Roles: [admin user]
```

---

## 🏷️ Struct Tags Deep Dive

| Tag | Purpose |
|-----|---------|
| `json:"field_name"` | Custom JSON field name |
| `json:"field_name,omitempty"` | Omit if zero value |
| `json:"-"` | Always omit from JSON |
| `json:",omitempty"` | Keep name, omit if zero |
| `json:"id,string"` | Encode number as string (for JS big int) |

**Omitempty example:**

```go
type Example struct {
    Name string `json:"name"`
    Age  int    `json:"user_age"`
    Score int   `json:"score,omitempty"`  // Omitted when 0
    Email string `json:",omitempty"`       // Omitted when ""
}
withZeros := Example{Name: "Bob", Age: 0, Score: 0}
jsonBytes, _ := json.Marshal(withZeros)
fmt.Printf("%s\n", jsonBytes)
// Output: {"name":"Bob","user_age":0}
```

**String encoding for large numbers:**

```go
type BigID struct {
    ID int64 `json:"id,string"`
}
b := BigID{ID: 9007199254740993}
jsonBytes, _ := json.Marshal(b)
fmt.Printf("%s\n", jsonBytes)
// Output: {"id":"9007199254740993"}
```

---

## 🔄 Custom JSON Encoding

Implement `json.Marshaler` and `json.Unmarshaler` for custom types:

```go
type Status int
const (StatusPending Status = iota; StatusActive; StatusCompleted)

func (s Status) MarshalJSON() ([]byte, error) {
    var str string
    switch s {
    case StatusPending: str = "pending"
    case StatusActive: str = "active"
    case StatusCompleted: str = "completed"
    default: str = "unknown"
    }
    return json.Marshal(str)
}

func (s *Status) UnmarshalJSON(data []byte) error {
    var str string
    json.Unmarshal(data, &str)
    switch str {
    case "pending": *s = StatusPending
    case "active": *s = StatusActive
    case "completed": *s = StatusCompleted
    }
    return nil
}
```

### Complete Example: Custom Encoding

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

type Status int
const (StatusPending Status = iota; StatusActive; StatusCompleted)

func (s Status) MarshalJSON() ([]byte, error) {
    var str string
    switch s {
    case StatusPending: str = "pending"
    case StatusActive: str = "active"
    case StatusCompleted: str = "completed"
    default: str = "unknown"
    }
    return json.Marshal(str)
}

type Task struct {
    Name   string `json:"name"`
    Status Status `json:"status"`
}

func main() {
    task := Task{Name: "Complete project", Status: StatusActive}
    jsonBytes, _ := json.MarshalIndent(task, "", "  ")
    fmt.Printf("%s\n", jsonBytes)
}
```

**Output:**
```
{
  "name": "Complete project",
  "status": "active"
}
```

---

## 🌊 Streaming JSON (Large Data)

**Encoder (write to stream):**

```go
var buf bytes.Buffer
encoder := json.NewEncoder(&buf)
encoder.SetIndent("", "  ")
encoder.Encode(record)  // Writes directly, no intermediate []byte
```

**Decoder (read from stream):**

```go
decoder := json.NewDecoder(strings.NewReader(jsonStream))
for decoder.More() {
    var r Record
    decoder.Decode(&r)
    // Process r
}
```

**When to use streaming:**
- Large JSON files (won't fit in memory)
- HTTP request/response bodies
- Newline-delimited JSON (NDJSON)

| Method | Use case |
|--------|----------|
| `json.Marshal()` | Returns `[]byte`, allocates memory |
| `encoder.Encode()` | Writes to `io.Writer`, streams |

---

## 🔧 Working with Dynamic JSON

**Unknown structure — use `map[string]interface{}`:**

```go
var data map[string]interface{}
json.Unmarshal([]byte(jsonStr), &data)
name := data["name"].(string)
age := data["age"].(float64)  // Numbers are float64!
```

**json.RawMessage — delay parsing:**

```go
type Response struct {
    Type string          `json:"type"`
    Data json.RawMessage `json:"data"`
}
var resp Response
json.Unmarshal([]byte(jsonResp), &resp)
if resp.Type == "user" {
    var user User
    json.Unmarshal(resp.Data, &user)
}
```

**json.Number for precise numbers:**

```go
decoder := json.NewDecoder(strings.NewReader(jsonNum))
decoder.UseNumber()
var p struct { Amount json.Number `json:"amount"` }
decoder.Decode(&p)
intVal, _ := p.Amount.Int64()
```

---

## 🏭 Production Patterns

**API response wrapper:**

```go
type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   string      `json:"error,omitempty"`
}
```

**Pointer for optional vs required:**

```go
type CreateUser struct {
    Name  string  `json:"name"`   // Required
    Age   *int    `json:"age,omitempty"`  // Optional (nil = not provided)
}
```

**HTTP JSON handler pattern:**

```go
func handler(w http.ResponseWriter, r *http.Request) {
    var req RequestBody
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
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
