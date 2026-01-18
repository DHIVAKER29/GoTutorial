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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  BEFORE go:embed (Go 1.16):                                     │
│  ┌─────────────┐    ┌─────────────┐                            │
│  │   Binary    │ +  │  config/    │  = Two things to deploy!   │
│  │   myapp     │    │  static/    │                            │
│  └─────────────┘    └─────────────┘                            │
│                                                                 │
│  AFTER go:embed:                                                │
│  ┌─────────────────────────────────────┐                       │
│  │           Single Binary             │                       │
│  │  ┌───────┐  ┌────────┐  ┌───────┐  │                       │
│  │  │ Code  │  │ Config │  │ HTML  │  │  Everything in one!   │
│  │  └───────┘  └────────┘  └───────┘  │                       │
│  └─────────────────────────────────────┘                       │
│                                                                 │
│  BENEFITS:                                                      │
│  • Single file deployment                                       │
│  • No missing file issues                                       │
│  • Files compressed in binary                                   │
│  • Perfect for: templates, static web, config                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Embedding Basics

```go
// embed_basics.go
package main

import (
    _ "embed"
    "fmt"
)

// Embed single file as string
//go:embed config.txt
var configString string

// Embed single file as bytes
//go:embed config.txt
var configBytes []byte

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           EMBEDDING SINGLE FILES                          ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 As String:")
    fmt.Printf("   %s\n", configString)
    
    fmt.Println("\n📊 As Bytes:")
    fmt.Printf("   Length: %d bytes\n", len(configBytes))
}

/*
Create config.txt in same directory:
    host=localhost
    port=8080

Then run:
    go run embed_basics.go
*/
```

---

## 📁 Embedding Directories

```go
// embed_directory.go
package main

import (
    "embed"
    "fmt"
    "io/fs"
)

// Embed entire directory
//go:embed static/*
var staticFiles embed.FS

// Embed multiple patterns
//go:embed templates/*.html
//go:embed templates/*.tmpl
var templates embed.FS

// Embed with subdirectories
//go:embed assets
var assets embed.FS

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           EMBEDDING DIRECTORIES                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // List embedded files
    fmt.Println("\n📊 List Embedded Files:")
    fs.WalkDir(staticFiles, ".", func(path string, d fs.DirEntry, err error) error {
        if err != nil {
            return err
        }
        if !d.IsDir() {
            info, _ := d.Info()
            fmt.Printf("   %s (%d bytes)\n", path, info.Size())
        }
        return nil
    })
    
    // Read file from embedded FS
    fmt.Println("\n📊 Read Embedded File:")
    content, err := staticFiles.ReadFile("static/index.html")
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   Content length: %d\n", len(content))
    }
}

/*
Directory structure:
    static/
        index.html
        style.css
    templates/
        home.html
        about.html
*/
```

---

## 🌐 Web Server with Embedded Files

```go
// embed_webserver.go
package main

import (
    "embed"
    "fmt"
    "io/fs"
    "net/http"
)

//go:embed static
var staticFiles embed.FS

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           WEB SERVER WITH EMBEDDED FILES                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Create sub-filesystem to strip "static" prefix
    staticFS, _ := fs.Sub(staticFiles, "static")
    
    // Serve embedded files
    http.Handle("/", http.FileServer(http.FS(staticFS)))
    
    fmt.Println("\nServer running on http://localhost:8080")
    http.ListenAndServe(":8080", nil)
}

/*
With this structure:
    static/
        index.html
        css/style.css
        js/app.js

URLs will be:
    http://localhost:8080/           → static/index.html
    http://localhost:8080/css/style.css
    http://localhost:8080/js/app.js
*/
```

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

