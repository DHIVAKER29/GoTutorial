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

### Method 1: os.ReadFile (entire file)

```go
// Simple, for small files
content, err := os.ReadFile("sample.txt")
if err != nil {
    fmt.Printf("Error: %v\n", err)
    return
}
fmt.Printf("Content:\n%s", content)
// Output: Content:
// Line 1
// Line 2
// Line 3
```

### Method 2: os.Open + Read

```go
file, err := os.Open("sample.txt")
if err != nil {
    fmt.Printf("Error: %v\n", err)
    return
}
defer file.Close()

buf := make([]byte, 1024)
n, err := file.Read(buf)
if err != nil && err != io.EOF {
    fmt.Printf("Error: %v\n", err)
    return
}
fmt.Printf("Read %d bytes\n", n)
```

### Method 3: bufio.Scanner (line by line)

```go
// Best for large files
file, _ := os.Open("sample.txt")
defer file.Close()

scanner := bufio.NewScanner(file)
lineNum := 1
for scanner.Scan() {
    fmt.Printf("Line %d: %s\n", lineNum, scanner.Text())
    lineNum++
}
if err := scanner.Err(); err != nil {
    fmt.Printf("Error: %v\n", err)
}
```

### Method 4: bufio.Reader

```go
// More control over reading
reader := bufio.NewReader(file)
for {
    line, err := reader.ReadString('\n')
    if err == io.EOF {
        break
    }
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        break
    }
    fmt.Printf("Read: %q\n", line)
}
```

### Complete Example

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
    // Create sample file first
    os.WriteFile("sample.txt", []byte("Line 1\nLine 2\nLine 3\n"), 0644)
    defer os.Remove("sample.txt")

    // Method 1: Read entire file
    fmt.Println("Method 1: os.ReadFile (entire file):")
    content, err := os.ReadFile("sample.txt")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    fmt.Printf("Content:\n%s", content)

    // Method 2: Open and read manually
    fmt.Println("\nMethod 2: os.Open + Read:")
    file, err := os.Open("sample.txt")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    defer file.Close()
    buf := make([]byte, 1024)
    n, _ := file.Read(buf)
    fmt.Printf("Read %d bytes\n", n)

    // Method 3: bufio.Scanner (line by line)
    fmt.Println("\nMethod 3: bufio.Scanner (line by line):")
    file2, _ := os.Open("sample.txt")
    defer file2.Close()
    scanner := bufio.NewScanner(file2)
    for i := 1; scanner.Scan(); i++ {
        fmt.Printf("Line %d: %s\n", i, scanner.Text())
    }
}
```

**Output:**
```
Method 1: os.ReadFile (entire file):
Content:
Line 1
Line 2
Line 3

Method 2: os.Open + Read:
Read 18 bytes

Method 3: bufio.Scanner (line by line):
Line 1: Line 1
Line 2: Line 2
Line 3: Line 3
```

---

## ✍️ Writing Files

### Method 1: os.WriteFile (entire file)

```go
content := []byte("Hello, World!\nThis is line 2.\n")
err := os.WriteFile("output1.txt", content, 0644)
if err != nil {
    fmt.Printf("Error: %v\n", err)
    return
}
fmt.Println("Wrote output1.txt")
```

### Method 2: os.Create + Write

```go
file, err := os.Create("output2.txt")
if err != nil {
    fmt.Printf("Error: %v\n", err)
    return
}
defer file.Close()

file.WriteString("Line 1\n")
file.Write([]byte("Line 2\n"))
fmt.Fprintf(file, "Line %d\n", 3)  // Formatted write
fmt.Println("Wrote output2.txt")
```

### Method 3: Append to File

```go
appendFile, err := os.OpenFile("output2.txt", os.O_APPEND|os.O_WRONLY, 0644)
if err != nil {
    fmt.Printf("Error: %v\n", err)
    return
}
defer appendFile.Close()
appendFile.WriteString("Appended line\n")
```

### Method 4: Buffered Writer

```go
file, _ := os.Create("output3.txt")
defer file.Close()
writer := bufio.NewWriter(file)
for i := 1; i <= 5; i++ {
    fmt.Fprintf(writer, "Buffered line %d\n", i)
}
writer.Flush()  // Don't forget to flush!
```

### File Permissions (Unix)

| Octal | Meaning |
|-------|---------|
| 0644 | rw-r--r-- (owner: rw, group: r, other: r) |
| 0755 | rwxr-xr-x (owner: rwx, group: rx, other: rx) |
| 0600 | rw------- (owner only) |

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
    // Create directory
    fmt.Println("Create Directory:")
    err := os.Mkdir("testdir", 0755)
    if err != nil && !os.IsExist(err) {
        fmt.Printf("Error: %v\n", err)
    }
    defer os.RemoveAll("testdir")
    fmt.Println("Created testdir/")

    // Create nested directories
    err = os.MkdirAll("testdir/sub1/sub2", 0755)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
    fmt.Println("Created testdir/sub1/sub2/")

    // List directory contents
    fmt.Println("\nList Directory:")
    entries, _ := os.ReadDir(".")
    for _, entry := range entries[:5] {
        info, _ := entry.Info()
        fmt.Printf("%s\t%d bytes\n", entry.Name(), info.Size())
    }

    // Walk directory tree
    fmt.Println("\nWalk Directory Tree:")
    filepath.Walk("testdir", func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        if info.IsDir() {
            fmt.Printf("[DIR]  %s\n", path)
        } else {
            fmt.Printf("[FILE] %s\n", path)
        }
        return nil
    })

    // Current, home, temp directories
    fmt.Println("\nPaths:")
    cwd, _ := os.Getwd()
    home, _ := os.UserHomeDir()
    fmt.Printf("Current: %s\n", cwd)
    fmt.Printf("Home: %s\n", home)
    fmt.Printf("Temp: %s\n", os.TempDir())
}
```

**Output:**
```
Create Directory:
Created testdir/

Create Directory:
Created testdir/sub1/sub2/

List Directory:
[Shows first 5 directory entries with their sizes]

Walk Directory Tree:
[DIR]  testdir
[DIR]  testdir/sub1
[DIR]  testdir/sub2

Paths:
Current: [Shows current working directory path]
Home: [Shows home directory path]
Temp: [Shows temp directory path]
```

*Note: Directory listings and paths will vary based on your system.*

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
    path := "/home/user/documents/file.txt"

    fmt.Println("Path Components:")
    fmt.Printf("Full path: %s\n", path)
    fmt.Printf("Dir:       %s\n", filepath.Dir(path))
    fmt.Printf("Base:      %s\n", filepath.Base(path))
    fmt.Printf("Ext:       %s\n", filepath.Ext(path))

    // Join paths (cross-platform!)
    fmt.Println("\nJoin Paths:")
    joined := filepath.Join("home", "user", "file.txt")
    fmt.Printf("filepath.Join: %s\n", joined)

    // Clean path
    fmt.Println("\nClean Path:")
    messy := "/home/user/../user/./documents//file.txt"
    fmt.Printf("Before: %s\n", messy)
    fmt.Printf("After:  %s\n", filepath.Clean(messy))

    // Absolute path
    fmt.Println("\nAbsolute Path:")
    abs, _ := filepath.Abs("relative/path.txt")
    fmt.Printf("Absolute: %s\n", abs)

    // Pattern matching
    fmt.Println("\nGlob Pattern Matching:")
    matches, _ := filepath.Glob("*.go")
    fmt.Printf("*.go files: %v\n", matches[:min(3, len(matches))])

    // Split
    fmt.Println("\nSplit Path:")
    dir, file := filepath.Split(path)
    fmt.Printf("Dir: %q, File: %q\n", dir, file)
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

**Output:**
```
Path Components:
Full path: /home/user/documents/file.txt
Dir:       /home/user/documents
Base:      file.txt
Ext:       .txt

Join Paths:
filepath.Join: home/user/file.txt

Clean Path:
Before: /home/user/../user/./documents//file.txt
After:  /home/user/documents/file.txt

Absolute Path:
Absolute: [Shows absolute path based on current directory]

Glob Pattern Matching:
*.go files: [Shows first 3 .go files in current directory]

Split Path:
Dir: "/home/user/documents/", File: "file.txt"
```

*Note: Absolute paths and glob matches will vary based on your system.*

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
