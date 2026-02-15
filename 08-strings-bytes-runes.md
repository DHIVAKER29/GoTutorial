# 08 - Strings, Bytes & Runes

> Understanding how Go handles text, Unicode, and character encoding in depth.

---

## 📌 What You'll Learn

- How computers store text (from basics)
- ASCII, Unicode, UTF-8, and UTF-16 explained
- What `byte` and `rune` really are
- Why `len(string)` might surprise you
- How to properly iterate over strings
- Sample programs with real examples

---

## 🤔 The Problem: How Do Computers Store Text?

### Computers Only Understand Numbers

Computers work with binary (0s and 1s). To store text like "Hello" or "नमस्ते", we assign a **number** to each character. For example: H → 72 → 01001000. We need a standard mapping — that's what character encodings provide.

---

## 📜 The History: ASCII → Unicode → UTF-8

### ASCII (1963)

- 7 bits = 128 characters (0–127)
- Covers English: digits, letters, symbols
- **Limitation:** No Hindi, Chinese, Arabic, emoji

### Unicode (1991)

- One number (code point) for every character worldwide
- Written as U+XXXX (hex): e.g., 'A' = U+0041, '中' = U+4E2D, '😀' = U+1F600
- Unicode defines the mapping; encoding defines how to store it in bytes

### UTF-8 (1992)

- Variable-length encoding (created by Go's creators!)
- ASCII: 1 byte; European: 2 bytes; Asian: 3 bytes; Emoji: 4 bytes
- Backward compatible with ASCII, compact for English, no null bytes in the middle

### UTF-8 vs UTF-16

| Encoding | "Hello中😀" | Used By |
|----------|-------------|---------|
| UTF-8 | 12 bytes | Go, Rust, Python 3, Web, Linux, macOS |
| UTF-16 | 16 bytes | Java, JavaScript (internal), Windows |

---

## 📦 byte vs rune in Go

### Definition

- **`byte`** = `uint8` — one byte of data (may be part of a character)
- **`rune`** = `int32` — one complete Unicode code point (character)

**Example:** The character "中" is 1 rune but 3 bytes in UTF-8 (E4 B8 AD).

### Snippets

```go
ascii := "Hello"
fmt.Println(len(ascii)) // Output: 5 (bytes)
```

```go
import "unicode/utf8"
hindi := "नमस्ते"
fmt.Println(len(hindi), utf8.RuneCountInString(hindi)) // Output: 18 6
```

```go
emoji := "👋"
fmt.Println(len(emoji), utf8.RuneCountInString(emoji)) // Output: 4 1
```

```go
chinese := "中"
for i, b := range []byte(chinese) {
    fmt.Printf("Byte %d: 0x%02X\n", i, b)
}
// Output: Byte 0: 0xE4, Byte 1: 0xB8, Byte 2: 0xAD
```

```go
for i, r := range "Hello中" {
    fmt.Printf("%d: '%c' U+%04X\n", i, r, r)
}
// Output: 0: 'H' U+0048, 1: 'e' U+0065, ... 5: '中' U+4E2D
```

### Complete Example: byte vs rune

```go
package main

import (
    "fmt"
    "unicode/utf8"
)

func main() {
    ascii := "Hello"
    hindi := "नमस्ते"
    fmt.Printf("Hello: %d bytes, %d runes\n", len(ascii), utf8.RuneCountInString(ascii))
    fmt.Printf("नमस्ते: %d bytes, %d runes\n", len(hindi), utf8.RuneCountInString(hindi))

    for i, r := range "Go语" {
        fmt.Printf("Index %d: '%c' (U+%04X)\n", i, r, r)
    }
}
```

**Output:**
```
Hello: 5 bytes, 5 runes
नमस्ते: 18 bytes, 6 runes
Index 0: 'G' (U+0047)
Index 1: 'o' (U+006F)
Index 2: '语' (U+8BED)
```

---

## 🔄 String Iteration: Two Ways

### Method 1: By Index (iterates BYTES — wrong for Unicode!)

```go
str := "Go语"
for i := 0; i < len(str); i++ {
    fmt.Printf("str[%d] = %d\n", i, str[i])
}
// Returns individual bytes, not characters
```

### Method 2: Range (iterates RUNES — correct!)

```go
for i, r := range "Go语" {
    fmt.Printf("Index %d: '%c'\n", i, r)
}
// Output: Index 0: 'G', Index 1: 'o', Index 2: '语'
```

### Method 3: Convert to []rune for random access

```go
runes := []rune("Hello世界")
fmt.Println(runes[5])     // 19990 (code point for '世')
fmt.Printf("%c\n", runes[5]) // Output: 世
```

### Complete Example: String Iteration

```go
package main

import "fmt"

func main() {
    str := "Go语言"
    runes := []rune(str)

    fmt.Println("By range (runes):")
    for i, r := range str {
        fmt.Printf("  %d: '%c'\n", i, r)
    }

    fmt.Printf("\n3rd character: %c\n", runes[2])

    reversed := reverseString("Hello世界")
    fmt.Printf("Reversed: %s\n", reversed)
}

func reverseString(s string) string {
    runes := []rune(s)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}
```

**Output:**
```
By range (runes):
  0: 'G'
  1: 'o'
  2: '语'
  5: '言'

3rd character: 语
Reversed: 界世olleH
```

---

## 🔄 Conversions: string ↔ []byte ↔ []rune

### Snippets

```go
s := "Hello"
bytes := []byte(s)
fmt.Println(bytes) // Output: [72 101 108 108 111]
```

```go
back := string(bytes)
fmt.Println(back) // Output: Hello
```

```go
runes := []rune("Hello世界")
fmt.Println(len(runes)) // Output: 7
```

```go
singleRune := '世'
fmt.Println(string(singleRune)) // Output: 世
```

```go
mutable := []byte("Hello")
mutable[0] = 'h'
fmt.Println(string(mutable)) // Output: hello
```

### Complete Example: Conversions

```go
package main

import "fmt"

func main() {
    s := "Hello"
    bytes := []byte(s)
    fmt.Printf("[]byte: %v\n", bytes)

    runes := []rune("Hello世界")
    fmt.Printf("[]rune length: %d\n", len(runes))
    for i, r := range runes {
        fmt.Printf("  [%d] '%c'\n", i, r)
    }

    mutable := []byte("Hello")
    mutable[0] = 'h'
    fmt.Println(string(mutable))
}
```

**Output:**
```
[]byte: [72 101 108 108 111]
[]rune length: 7
  [0] 'H'
  [1] 'e'
  [2] 'l'
  [3] 'l'
  [4] 'o'
  [5] '世'
  [6] '界'
hello
```

---

## 💡 When to Use What

| Use | Type | When |
|-----|------|------|
| **string** | Display, JSON, map keys, function params | Text that won't be modified |
| **[]byte** | Files, network I/O, binary data | Reading/writing, performance-critical building |
| **[]rune** | Character count, random access, reversing | Unicode-aware text processing |

---

## 🏭 Production Patterns

```go
func TruncateDisplayName(name string, maxRunes int) string {
    runes := []rune(name)
    if len(runes) <= maxRunes {
        return name
    }
    return string(runes[:maxRunes-1]) + "…"
}

func IsWithinCharLimit(s string, limit int) bool {
    return utf8.RuneCountInString(s) <= limit
}
```

### Complete Example: Production Patterns

```go
package main

import (
    "fmt"
    "unicode/utf8"
)

func TruncateDisplayName(name string, maxRunes int) string {
    runes := []rune(name)
    if len(runes) <= maxRunes {
        return name
    }
    return string(runes[:maxRunes-1]) + "…"
}

func main() {
    names := []string{"John Doe", "राहुल कुमार", "你好世界"}
    for _, name := range names {
        fmt.Println(TruncateDisplayName(name, 8))
    }

    fmt.Println(utf8.RuneCountInString("Hello") <= 10)
    fmt.Println(utf8.RuneCountInString("Hello World") <= 10)
}
```

**Output:**
```
John Do…
राहुल कु…
你好世…
true
false
```

---

## 🆚 Java Comparison

| Aspect | Java | Go |
|--------|------|-----|
| String encoding | UTF-16 internally | UTF-8 internally |
| Character type | `char` (16-bit) | `rune` (32-bit, full Unicode) |
| Length | `str.length()` (code units) | `utf8.RuneCountInString(s)` (runes) |
| Get character | `str.charAt(i)` (may break emoji) | `[]rune(s)[i]` (complete character) |
| To bytes | `str.getBytes("UTF-8")` | `[]byte(s)` (already UTF-8) |
| Emoji | `"😀".length() = 2` (surrogate pair) | `[]rune("😀")` = 1 rune |

---

## 🎯 Key Takeaways

1. **`len(string)`** returns BYTES, not characters!
2. **`byte`** = `uint8` = one byte of data
3. **`rune`** = `int32` = one Unicode code point
4. **Use `range`** to iterate over runes (characters)
5. **Use `[]rune(s)`** for random access to characters
6. **UTF-8** is Go's native encoding (created by Go's authors!)
7. **`utf8.RuneCountInString()`** for character count

---

## ➡️ Next Steps

You now understand Go's text handling. Let's explore type conversions.

**Next Topic:** [09 - Type Conversions](./09-type-conversions.md)
