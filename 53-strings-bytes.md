# 53 - strings and bytes Packages

> Essential string and byte slice manipulation utilities.

---

## 📌 What You'll Learn

- strings package functions
- bytes package functions
- strings.Builder for efficient concatenation
- Common string operations

---

## 📜 strings Package

```go
s := "Hello, World!"

// Searching
strings.Contains(s, "World")     // true
strings.HasPrefix(s, "Hello")    // true
strings.Index(s, "o")            // 4
strings.Count(s, "l")            // 3

// Case
strings.ToUpper(s)               // "HELLO, WORLD!"
strings.ToLower(s)               // "hello, world!"

// Trimming
strings.TrimSpace("  Hello  ")   // "Hello"
strings.Trim("!!!hello!!!", "!") // "hello"

// Splitting/Joining
strings.Split("a,b,c", ",")      // []string{"a","b","c"}
strings.Join([]string{"a","b"}, "-")  // "a-b"

// Replace
strings.ReplaceAll("oink oink", "oink", "moo")  // "moo moo"
strings.Repeat("ab", 3)          // "ababab"
```

---

## 🔧 strings.Builder (Efficient Concatenation)

```go
// Problem: s += "x" creates new string each time (O(n²))
// Solution: strings.Builder
var builder strings.Builder
builder.WriteString("Hello")
builder.WriteString(", ")
builder.WriteString("World")
builder.WriteByte('!')
result := builder.String()
// Output: "Hello, World!"

// Pre-allocate for known size
var b strings.Builder
b.Grow(100)
for i := 0; i < 10; i++ {
    fmt.Fprintf(&b, "item%d ", i)
}
// Output: ~1000x faster than += for many concatenations
```

---

## 📦 bytes Package

```go
b := []byte("Hello, World!")

// Same functions as strings, for []byte
bytes.Contains(b, []byte("World"))  // true
bytes.ToUpper(b)                    // []byte
bytes.Split(b, []byte(", "))        // [][]byte

// bytes.Buffer (io.Reader AND io.Writer)
var buf bytes.Buffer
buf.WriteString("Hello ")
buf.Write([]byte("World"))
buf.WriteByte('!')
// Output: buf.String()="Hello World!"

// Reading from buffer
data := make([]byte, 5)
n, _ := buf.Read(data)
// Output: n=5, data="Hello", buf.String()=" World!"

// Comparison
bytes.Equal([]byte("abc"), []byte("abc"))  // true
bytes.Compare([]byte("abc"), []byte("abd")) // -1
```

---

## 🔄 strings.Reader

```go
r := strings.NewReader("Hello, World!")
r.Size()   // 13
r.Len()    // 13 (remaining)

buf := make([]byte, 5)
n, _ := r.Read(buf)
// Output: n=5, buf="Hello", r.Len()=8

r.Seek(0, io.SeekStart)
all, _ := io.ReadAll(r)
// Output: "Hello, World!"
```

---

## 🎯 Quick Reference

| Category | Functions |
|----------|-----------|
| **Searching** | Contains, ContainsAny, HasPrefix, HasSuffix, Index, LastIndex, Count |
| **Transforming** | ToUpper, ToLower, TrimSpace, Trim, Replace, ReplaceAll |
| **Splitting/Joining** | Split, SplitN, Fields, Join |
| **Builders/Buffers** | strings.Builder, bytes.Buffer, strings.Reader |

**Tip:** bytes package mirrors strings for `[]byte`.

---

## ➡️ Next Steps

**Next Topic:** [54 - sort Package](./54-sorting.md)
