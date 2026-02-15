# 44 - Embedding Files (go:embed)

> Embedding static files directly into your Go binary.

---

## 📌 What You'll Learn

- What go:embed is and why it's useful
- Embedding single files, multiple files, directories
- Working with embed.FS
- Use cases and patterns

---

## 🤔 Why Embed Files?

**Before go:embed (Go 1.16):** Binary + config/static folders = two things to deploy.

**After go:embed:** Single binary containing code, config, HTML, and static assets.

**Benefits:**
- Single file deployment
- No missing file issues
- Files compressed in binary
- Perfect for: templates, static web assets, config

---

## 📝 Embedding Basics

### Single File as String

```go
package main

import (
    _ "embed"
    "fmt"
)

//go:embed config.txt
var configString string

func main() {
    fmt.Println(configString)
}
// Output:
// host=localhost
// port=8080
```

*Create `config.txt` in same directory with content: `host=localhost` and `port=8080`*

### Single File as Bytes

```go
package main

import (
    _ "embed"
    "fmt"
)

//go:embed config.txt
var configBytes []byte

func main() {
    fmt.Printf("Length: %d bytes\n", len(configBytes))
}
// Output:
// Length: 22 bytes
```

---

## 📁 Embedding Directories

### List and Read from embed.FS

```go
package main

import (
    "embed"
    "fmt"
    "io/fs"
)

//go:embed static/*
var staticFiles embed.FS

func main() {
    fs.WalkDir(staticFiles, ".", func(path string, d fs.DirEntry, err error) error {
        if err != nil {
            return err
        }
        if !d.IsDir() {
            info, _ := d.Info()
            fmt.Printf("%s (%d bytes)\n", path, info.Size())
        }
        return nil
    })

    content, _ := staticFiles.ReadFile("static/index.html")
    fmt.Printf("index.html length: %d\n", len(content))
}
// Output:
// static/index.html (256 bytes)
// static/style.css (1024 bytes)
// index.html length: 256
```

*Directory structure: `static/index.html`, `static/style.css`*

### Multiple Patterns

```go
package main

import (
    "embed"
)

//go:embed templates/*.html
//go:embed templates/*.tmpl
var templates embed.FS
```

---

## 🌐 Web Server with Embedded Files

### Complete Example

```go
package main

import (
    "embed"
    "io/fs"
    "net/http"
)

//go:embed static
var staticFiles embed.FS

func main() {
    staticFS, _ := fs.Sub(staticFiles, "static")
    http.Handle("/", http.FileServer(http.FS(staticFS)))
    http.ListenAndServe(":8080", nil)
}
```

**Output:** Server runs on http://localhost:8080 and serves embedded files.

*With `static/index.html`, `static/css/style.css`, `static/js/app.js`:*
- `http://localhost:8080/` → index.html
- `http://localhost:8080/css/style.css`
- `http://localhost:8080/js/app.js`

---

## 🎯 Key Takeaways

1. **//go:embed** directive embeds files at compile time
2. **string** or **[]byte** for single files
3. **embed.FS** for multiple files/directories
4. **fs.Sub()** to strip directory prefix
5. Works with **http.FileServer** for web serving

---

## ➡️ Next Steps

**Next Topic:** [45 - Database Access](./45-database.md)
