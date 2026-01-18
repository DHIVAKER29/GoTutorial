# 38 - File I/O

> Reading, writing, and manipulating files in Go.

---

## 📌 What You'll Learn

- Reading files (whole file, line by line, buffered)
- Writing files (create, append, overwrite)
- Working with directories
- File paths and os package
- Temporary files

---

## 📖 Reading Files

```go
// reading_files.go
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           READING FILES                                   ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Create sample file first
    os.WriteFile("sample.txt", []byte("Line 1\nLine 2\nLine 3\n"), 0644)
    defer os.Remove("sample.txt")
    
    // Method 1: Read entire file (simple, small files)
    fmt.Println("\n📊 Method 1: os.ReadFile (entire file):")
    content, err := os.ReadFile("sample.txt")
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    fmt.Printf("   Content:\n%s", content)
    
    // Method 2: Open and read manually
    fmt.Println("\n📊 Method 2: os.Open + Read:")
    file, err := os.Open("sample.txt")
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    defer file.Close()
    
    buf := make([]byte, 1024)
    n, err := file.Read(buf)
    if err != nil && err != io.EOF {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    fmt.Printf("   Read %d bytes\n", n)
    
    // Method 3: Read line by line (large files)
    fmt.Println("\n📊 Method 3: bufio.Scanner (line by line):")
    file2, _ := os.Open("sample.txt")
    defer file2.Close()
    
    scanner := bufio.NewScanner(file2)
    lineNum := 1
    for scanner.Scan() {
        fmt.Printf("   Line %d: %s\n", lineNum, scanner.Text())
        lineNum++
    }
    
    if err := scanner.Err(); err != nil {
        fmt.Printf("   Error: %v\n", err)
    }
    
    // Method 4: Buffered reader (more control)
    fmt.Println("\n📊 Method 4: bufio.Reader:")
    file3, _ := os.Open("sample.txt")
    defer file3.Close()
    
    reader := bufio.NewReader(file3)
    for {
        line, err := reader.ReadString('\n')
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Printf("   Error: %v\n", err)
            break
        }
        fmt.Printf("   Read: %q\n", line)
    }
}
```

---

## ✍️ Writing Files

```go
// writing_files.go
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           WRITING FILES                                   ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Method 1: Write entire file (simple)
    fmt.Println("\n📊 Method 1: os.WriteFile (entire file):")
    content := []byte("Hello, World!\nThis is line 2.\n")
    err := os.WriteFile("output1.txt", content, 0644)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    fmt.Println("   Wrote output1.txt")
    defer os.Remove("output1.txt")
    
    // Method 2: Create and write
    fmt.Println("\n📊 Method 2: os.Create + Write:")
    file, err := os.Create("output2.txt")
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    defer file.Close()
    defer os.Remove("output2.txt")
    
    file.WriteString("Line 1\n")
    file.Write([]byte("Line 2\n"))
    fmt.Fprintf(file, "Line %d\n", 3)  // Formatted write
    fmt.Println("   Wrote output2.txt")
    
    // Method 3: Append to file
    fmt.Println("\n📊 Method 3: Append to File:")
    appendFile, err := os.OpenFile("output2.txt",
        os.O_APPEND|os.O_WRONLY, 0644)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    defer appendFile.Close()
    
    appendFile.WriteString("Appended line\n")
    fmt.Println("   Appended to output2.txt")
    
    // Method 4: Buffered writer (efficient for many writes)
    fmt.Println("\n📊 Method 4: Buffered Writer:")
    file3, _ := os.Create("output3.txt")
    defer file3.Close()
    defer os.Remove("output3.txt")
    
    writer := bufio.NewWriter(file3)
    for i := 1; i <= 5; i++ {
        fmt.Fprintf(writer, "Buffered line %d\n", i)
    }
    writer.Flush()  // Don't forget to flush!
    fmt.Println("   Wrote output3.txt with buffer")
    
    // File permissions
    fmt.Println("\n📊 File Permissions (Unix):")
    fmt.Println("   0644 = rw-r--r-- (owner: rw, group: r, other: r)")
    fmt.Println("   0755 = rwxr-xr-x (owner: rwx, group: rx, other: rx)")
    fmt.Println("   0600 = rw------- (owner only)")
}
```

---

## 📁 Working with Directories

```go
// directories.go
package main

import (
    "fmt"
    "os"
    "path/filepath"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           WORKING WITH DIRECTORIES                        ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Create directory
    fmt.Println("\n📊 Create Directory:")
    err := os.Mkdir("testdir", 0755)
    if err != nil && !os.IsExist(err) {
        fmt.Printf("   Error: %v\n", err)
    }
    defer os.RemoveAll("testdir")
    fmt.Println("   Created testdir/")
    
    // Create nested directories
    err = os.MkdirAll("testdir/sub1/sub2", 0755)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    }
    fmt.Println("   Created testdir/sub1/sub2/")
    
    // List directory contents
    fmt.Println("\n📊 List Directory:")
    entries, _ := os.ReadDir(".")
    for _, entry := range entries[:5] {  // First 5
        info, _ := entry.Info()
        fmt.Printf("   %s\t%d bytes\n", entry.Name(), info.Size())
    }
    
    // Walk directory tree
    fmt.Println("\n📊 Walk Directory Tree:")
    filepath.Walk("testdir", func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        if info.IsDir() {
            fmt.Printf("   [DIR]  %s\n", path)
        } else {
            fmt.Printf("   [FILE] %s\n", path)
        }
        return nil
    })
    
    // Current directory
    fmt.Println("\n📊 Current Directory:")
    cwd, _ := os.Getwd()
    fmt.Printf("   %s\n", cwd)
    
    // Home directory
    home, _ := os.UserHomeDir()
    fmt.Printf("   Home: %s\n", home)
    
    // Temp directory
    fmt.Printf("   Temp: %s\n", os.TempDir())
}
```

---

## 🛣️ File Paths

```go
// file_paths.go
package main

import (
    "fmt"
    "path/filepath"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           FILE PATHS (filepath package)                   ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    path := "/home/user/documents/file.txt"
    
    fmt.Println("\n📊 Path Components:")
    fmt.Printf("   Full path: %s\n", path)
    fmt.Printf("   Dir:       %s\n", filepath.Dir(path))
    fmt.Printf("   Base:      %s\n", filepath.Base(path))
    fmt.Printf("   Ext:       %s\n", filepath.Ext(path))
    
    // Join paths (cross-platform!)
    fmt.Println("\n📊 Join Paths:")
    joined := filepath.Join("home", "user", "file.txt")
    fmt.Printf("   filepath.Join: %s\n", joined)
    
    // Clean path
    fmt.Println("\n📊 Clean Path:")
    messy := "/home/user/../user/./documents//file.txt"
    fmt.Printf("   Before: %s\n", messy)
    fmt.Printf("   After:  %s\n", filepath.Clean(messy))
    
    // Absolute path
    fmt.Println("\n📊 Absolute Path:")
    abs, _ := filepath.Abs("relative/path.txt")
    fmt.Printf("   Absolute: %s\n", abs)
    
    // Pattern matching
    fmt.Println("\n📊 Glob Pattern Matching:")
    matches, _ := filepath.Glob("*.go")
    fmt.Printf("   *.go files: %v\n", matches[:min(3, len(matches))])
    
    // Split
    fmt.Println("\n📊 Split Path:")
    dir, file := filepath.Split(path)
    fmt.Printf("   Dir: %q, File: %q\n", dir, file)
}

func min(a, b int) int {
    if a < b { return a }
    return b
}
```

---

## 🎯 Key Takeaways

1. **os.ReadFile/WriteFile** for simple file operations
2. **bufio** for efficient line-by-line or buffered I/O
3. **Always close files** - use `defer file.Close()`
4. **filepath.Join** for cross-platform paths
5. **os.MkdirAll** creates nested directories
6. **File permissions** in octal (0644, 0755)

---

## ➡️ Next Steps

**Next Topic:** [39 - Time and Date](./39-time.md)

