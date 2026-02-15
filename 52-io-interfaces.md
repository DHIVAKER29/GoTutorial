# 52 - io.Reader and io.Writer Interfaces

> The fundamental interfaces that power all I/O in Go.

---

## 📌 What You'll Learn

- io.Reader and io.Writer interfaces
- Why they're so powerful
- Common implementations
- Composing readers and writers
- The Stringer interface

---

## 🤔 The Power of Simple Interfaces

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

**Everything is a stream of bytes:** Files, network, HTTP body, buffers, compression, encryption, strings.

**Write code once, works with any source/destination.**

---

## 📝 Basic Usage

```go
// io.Reader examples
strReader := strings.NewReader("Hello from string!")
bufReader := bytes.NewBufferString("Hello from buffer!")

buf := make([]byte, 100)
n, _ := strReader.Read(buf)
// Output: n=19, buf[:n]="Hello from string!"

// io.Writer examples
var buffer bytes.Buffer
buffer.Write([]byte("Hello "))
buffer.WriteString("World!")
// Output: buffer.String()="Hello World!"
```

---

## 🔄 io.Copy - The Universal Copier

```go
src := strings.NewReader("Hello, io.Copy!\n")
io.Copy(os.Stdout, src)
// Output: Hello, io.Copy!

src = strings.NewReader("Data to copy")
var dst bytes.Buffer
n, _ := io.Copy(&dst, src)
// Output: n=12, dst.String()="Data to copy"

// File copy pattern:
// io.Copy(dstFile, srcFile)

// io.CopyN with limit
io.CopyN(&limited, src, 5)
// Output: "Only " (first 5 bytes)
```

---

## 🧩 Composing Readers and Writers

```go
// Writer chain: data → gzip → base64 → stdout
b64Writer := base64.NewEncoder(base64.StdEncoding, os.Stdout)
gzWriter := gzip.NewWriter(b64Writer)
gzWriter.Write([]byte("Hello, compressed!"))
gzWriter.Close()
b64Writer.Close()

// io.MultiReader (concatenate)
multi := io.MultiReader(r1, r2, r3)
io.Copy(os.Stdout, multi)
// Output: "First Second Third"

// io.TeeReader (read + copy to second destination)
tee := io.TeeReader(src, &buf)
io.Copy(os.Stdout, tee)
// Output: Data written to both stdout and buf
```

---

## 📜 The Stringer Interface

```go
type User struct {
    ID    int
    Name  string
    Email string
}

func (u User) String() string {
    return fmt.Sprintf("User{%d: %s <%s>}", u.ID, u.Name, u.Email)
}

user := User{ID: 1, Name: "Alice", Email: "alice@example.com"}
fmt.Printf("%v\n", user)
// Output: User{1: Alice <alice@example.com>}
```

```go
// Enum with String()
func (s Status) String() string {
    switch s {
    case StatusPending: return "PENDING"
    case StatusActive:  return "ACTIVE"
    default: return "UNKNOWN"
    }
}
// Output: StatusActive.String() = "ACTIVE"
```

---

## 🎯 Key Takeaways

1. **io.Reader/Writer** = Go's most important interfaces
2. **One method each** - incredibly simple yet powerful
3. **Everything is bytes** - files, network, buffers
4. **io.Copy** moves data between any reader/writer
5. **Composable** - chain readers/writers together
6. **Stringer** for custom string representation
7. **Write to interface** for maximum flexibility

---

## ➡️ Next Steps

**Next Topic:** [53 - strings and bytes Packages](./53-strings-bytes.md)
