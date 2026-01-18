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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  THE FUNDAMENTAL PROBLEM                                        │
│                                                                 │
│  Computers only understand binary (0s and 1s)                   │
│  But humans want to store text like "Hello" or "नमस्ते"          │
│                                                                 │
│  SOLUTION: Assign a NUMBER to each CHARACTER                    │
│                                                                 │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐                 │
│  │    H    │  →   │   72    │  →   │01001000 │                 │
│  │(letter) │      │(number) │      │ (binary)│                 │
│  └─────────┘      └─────────┘      └─────────┘                 │
│                                                                 │
│  But... which number for which character?                       │
│  We need a STANDARD mapping!                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📜 The History: ASCII → Unicode → UTF-8

### Era 1: ASCII (1963)

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ASCII (American Standard Code for Information Interchange)     │
│                                                                 │
│  Created in 1963 for English only                               │
│  Uses 7 bits = 128 possible characters (0-127)                  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  0-31    : Control characters (newline, tab, etc.)       │  │
│  │  32-47   : Symbols (space, !, ", #, $, etc.)             │  │
│  │  48-57   : Digits (0-9)                                  │  │
│  │  65-90   : Uppercase letters (A-Z)                       │  │
│  │  97-122  : Lowercase letters (a-z)                       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Examples:                                                      │
│  'A' = 65    'a' = 97    '0' = 48    ' ' = 32                   │
│                                                                 │
│  PROBLEM: Only 128 characters!                                  │
│  ❌ No Hindi (नमस्ते)                                            │
│  ❌ No Chinese (你好)                                            │
│  ❌ No Arabic (مرحبا)                                            │
│  ❌ No emoji (😀)                                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Era 2: Unicode (1991)

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  UNICODE: One Standard to Rule Them All                         │
│                                                                 │
│  Goal: Assign a unique number to EVERY character                │
│        in EVERY language in the world!                          │
│                                                                 │
│  Each character gets a "CODE POINT" (a unique number)           │
│  Written as: U+XXXX (hexadecimal)                               │
│                                                                 │
│  Examples:                                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  'A'   = U+0041 (65 in decimal)                        │    │
│  │  'a'   = U+0061 (97 in decimal)                        │    │
│  │  '中'  = U+4E2D (20013 in decimal)                      │    │
│  │  'न'   = U+0928 (2344 in decimal)                      │    │
│  │  '😀'  = U+1F600 (128512 in decimal)                    │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Unicode currently has 149,186 characters!                      │
│  (and growing with each version)                                │
│                                                                 │
│  BUT WAIT: Unicode is just a LIST of code points.               │
│  How do we actually STORE them in memory?                       │
│  That's where ENCODING comes in...                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Era 3: UTF-8 (1992)

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  UTF-8: The Smart Encoding                                      │
│                                                                 │
│  Created by Ken Thompson (Go creator!) and Rob Pike (Go!)       │
│                                                                 │
│  KEY IDEA: Variable-length encoding                             │
│  - ASCII characters: 1 byte (backwards compatible!)             │
│  - Most European chars: 2 bytes                                 │
│  - Asian characters: 3 bytes                                    │
│  - Emoji and rare chars: 4 bytes                                │
│                                                                 │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  Character    Code Point    UTF-8 Bytes                │    │
│  │  ─────────────────────────────────────────────────────  │    │
│  │  'A'          U+0041        41              (1 byte)   │    │
│  │  'é'          U+00E9        C3 A9           (2 bytes)  │    │
│  │  '中'         U+4E2D        E4 B8 AD        (3 bytes)  │    │
│  │  '😀'         U+1F600       F0 9F 98 80     (4 bytes)  │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
│  WHY UTF-8 IS BRILLIANT:                                        │
│  ✅ ASCII text is still valid UTF-8 (no conversion needed)      │
│  ✅ English text is compact (1 byte per char)                   │
│  ✅ No null bytes in the middle (C string compatible)           │
│  ✅ Can detect start of any character                           │
│  ✅ Self-synchronizing (can find boundaries)                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### UTF-8 vs UTF-16

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  UTF-8 vs UTF-16 COMPARISON                                     │
│                                                                 │
│  String: "Hello中😀"                                            │
│                                                                 │
│  UTF-8 Encoding:                                                │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  H    e    l    l    o    中         😀                  │    │
│  │  48   65   6C   6C   6F   E4B8AD     F09F9880            │    │
│  │  1    1    1    1    1    3          4         bytes     │    │
│  │  Total: 12 bytes                                        │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
│  UTF-16 Encoding:                                               │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  H      e      l      l      o      中      😀          │    │
│  │  0048   0065   006C   006C   006F   4E2D   D83DDE00     │    │
│  │  2      2      2      2      2      2      4     bytes  │    │
│  │  Total: 16 bytes                                        │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
│  WHO USES WHAT:                                                 │
│  UTF-8:  Go, Rust, Python 3, Web, Linux, macOS, JSON, HTML      │
│  UTF-16: Java, JavaScript (internal), Windows (internal)        │
│                                                                 │
│  Go chose UTF-8 because its creators INVENTED UTF-8!            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📦 byte vs rune in Go

### Definition

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  byte AND rune ARE TYPE ALIASES                                 │
│                                                                 │
│  type byte = uint8   // 8 bits, 0-255                           │
│  type rune = int32   // 32 bits, any Unicode code point         │
│                                                                 │
│  byte  →  One byte of data (could be part of a character)       │
│  rune  →  One complete Unicode character                        │
│                                                                 │
│  Think of it like:                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  WORD: "中"                                              │   │
│  │                                                         │   │
│  │  As RUNE (what you see):                                │   │
│  │  ┌───────────────────────────┐                          │   │
│  │  │           中              │  = 1 rune                │   │
│  │  └───────────────────────────┘                          │   │
│  │                                                         │   │
│  │  As BYTES (how it's stored in UTF-8):                   │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐                    │   │
│  │  │   E4    │ │   B8    │ │   AD    │  = 3 bytes         │   │
│  │  └─────────┘ └─────────┘ └─────────┘                    │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Program: byte vs rune

```go
// bytes_runes.go
package main

import (
    "fmt"
    "unicode/utf8"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              BYTES vs RUNES IN GO                         ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // ASCII string
    ascii := "Hello"
    fmt.Println("\n📊 ASCII String: \"Hello\"")
    fmt.Printf("   len() = %d bytes\n", len(ascii))
    fmt.Printf("   Rune count = %d characters\n", utf8.RuneCountInString(ascii))
    fmt.Println("   (Same! Each ASCII char is 1 byte)")
    
    // Hindi string
    hindi := "नमस्ते"
    fmt.Println("\n📊 Hindi String: \"नमस्ते\"")
    fmt.Printf("   len() = %d bytes\n", len(hindi))
    fmt.Printf("   Rune count = %d characters\n", utf8.RuneCountInString(hindi))
    fmt.Println("   (Different! Hindi chars use multiple bytes)")
    
    // Emoji string
    emoji := "👋🌍"
    fmt.Println("\n📊 Emoji String: \"👋🌍\"")
    fmt.Printf("   len() = %d bytes\n", len(emoji))
    fmt.Printf("   Rune count = %d characters\n", utf8.RuneCountInString(emoji))
    fmt.Println("   (Emoji use 4 bytes each!)")
    
    // Mixed string
    mixed := "Hello中😀"
    fmt.Println("\n📊 Mixed String: \"Hello中😀\"")
    fmt.Printf("   len() = %d bytes\n", len(mixed))
    fmt.Printf("   Rune count = %d characters\n", utf8.RuneCountInString(mixed))
    
    // Show bytes vs runes
    fmt.Println("\n📊 Byte-by-byte view of \"中\":")
    chinese := "中"
    for i, b := range []byte(chinese) {
        fmt.Printf("   Byte %d: %d (0x%02X)\n", i, b, b)
    }
    
    fmt.Println("\n📊 Rune view of \"中\":")
    for i, r := range chinese {
        fmt.Printf("   Index %d: '%c' (U+%04X, decimal %d)\n", i, r, r, r)
    }
    
    // Accessing by index
    fmt.Println("\n⚠️ Danger: Indexing returns BYTES, not RUNES!")
    str := "Hello中"
    fmt.Printf("   str[0] = %d ('%c') - OK for ASCII\n", str[0], str[0])
    fmt.Printf("   str[5] = %d (0x%02X) - This is NOT '中'!\n", str[5], str[5])
    fmt.Printf("   '中' starts at index 5 but spans bytes 5,6,7\n")
    
    // Safe iteration
    fmt.Println("\n✅ Safe: Use range to iterate runes:")
    fmt.Printf("   String: %q\n", mixed)
    for i, r := range mixed {
        fmt.Printf("   Index %d: '%c' (U+%04X)\n", i, r, r)
    }
}
```

---

## 🔄 String Iteration: Two Ways

### Sample Program: Iterating Strings

```go
// string_iteration.go
package main

import (
    "fmt"
    "unicode/utf8"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           STRING ITERATION IN GO                          ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    str := "Go语言😀"
    
    fmt.Printf("\n📝 String: %q\n", str)
    fmt.Printf("   Bytes: %d, Runes: %d\n", len(str), utf8.RuneCountInString(str))
    
    // Method 1: Iterate by BYTES (usually wrong for Unicode!)
    fmt.Println("\n❌ Method 1: By Index (iterates BYTES)")
    fmt.Println("   This breaks for non-ASCII!")
    for i := 0; i < len(str); i++ {
        fmt.Printf("   str[%d] = %d (0x%02X)\n", i, str[i], str[i])
    }
    
    // Method 2: Range over string (iterates RUNES - correct!)
    fmt.Println("\n✅ Method 2: Range (iterates RUNES)")
    fmt.Println("   This handles Unicode correctly!")
    for i, r := range str {
        fmt.Printf("   Index %2d: '%c' (U+%04X, %d bytes)\n", 
            i, r, r, utf8.RuneLen(r))
    }
    
    // Converting to rune slice for random access
    fmt.Println("\n✅ Method 3: Convert to []rune for random access")
    runes := []rune(str)
    fmt.Printf("   []rune length: %d\n", len(runes))
    for i, r := range runes {
        fmt.Printf("   runes[%d] = '%c'\n", i, r)
    }
    
    // Getting specific character
    fmt.Println("\n💡 Getting the 3rd character:")
    fmt.Printf("   str[2] = %d - WRONG! Gets 3rd byte\n", str[2])
    fmt.Printf("   []rune(str)[2] = '%c' - CORRECT!\n", runes[2])
    
    // Practical example: Reversing a string
    fmt.Println("\n💡 Practical: Reversing a Unicode string")
    original := "Hello世界"
    reversed := reverseString(original)
    fmt.Printf("   Original: %q\n", original)
    fmt.Printf("   Reversed: %q\n", reversed)
}

func reverseString(s string) string {
    runes := []rune(s)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}
```

---

## 🔄 Conversions: string ↔ []byte ↔ []rune

### Sample Program: Type Conversions

```go
// string_conversions.go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           STRING CONVERSIONS IN GO                        ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    original := "Hello世界"
    
    fmt.Printf("\n📝 Original string: %q\n", original)
    
    // String to []byte
    fmt.Println("\n📊 String → []byte (UTF-8 bytes)")
    bytes := []byte(original)
    fmt.Printf("   %v\n", bytes)
    fmt.Printf("   Length: %d bytes\n", len(bytes))
    
    // []byte back to String
    fmt.Println("\n📊 []byte → String")
    backToString := string(bytes)
    fmt.Printf("   %q\n", backToString)
    
    // String to []rune (Unicode code points)
    fmt.Println("\n📊 String → []rune (Unicode code points)")
    runes := []rune(original)
    fmt.Printf("   %v\n", runes)
    fmt.Printf("   Length: %d runes\n", len(runes))
    
    // Show each rune
    fmt.Println("   Each rune:")
    for i, r := range runes {
        fmt.Printf("     [%d] '%c' = %d (U+%04X)\n", i, r, r, r)
    }
    
    // []rune back to String
    fmt.Println("\n📊 []rune → String")
    backFromRunes := string(runes)
    fmt.Printf("   %q\n", backFromRunes)
    
    // Single rune to string
    fmt.Println("\n📊 Single rune → String")
    singleRune := '世'  // This is a rune literal
    singleString := string(singleRune)
    fmt.Printf("   rune '%c' → string %q\n", singleRune, singleString)
    
    // Integer to string (code point)
    fmt.Println("\n📊 Integer → String (treats as code point)")
    codePoint := 19990  // Unicode for '世'
    fromInt := string(rune(codePoint))
    fmt.Printf("   int %d → string %q\n", codePoint, fromInt)
    
    // Modifying string via []byte
    fmt.Println("\n📊 Modifying String (via []byte)")
    mutable := []byte("Hello")
    fmt.Printf("   Original: %q\n", string(mutable))
    mutable[0] = 'h'
    fmt.Printf("   Modified: %q\n", string(mutable))
    
    // Practical: Reading file content
    fmt.Println("\n💡 Practical Use Cases:")
    fmt.Println("   []byte: Reading files, network data, binary data")
    fmt.Println("   []rune: Character-level processing, text manipulation")
    fmt.Println("   string: General text, display, JSON keys")
}
```

---

## 💡 When to Use What

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  WHEN TO USE WHAT                                               │
│                                                                 │
│  USE string WHEN:                                               │
│  • Displaying text to users                                     │
│  • JSON keys/values                                             │
│  • Map keys                                                     │
│  • Function parameters (most cases)                             │
│  • Text that won't be modified                                  │
│                                                                 │
│  USE []byte WHEN:                                               │
│  • Reading/writing files                                        │
│  • Network I/O                                                  │
│  • Binary data                                                  │
│  • Performance-critical string building                         │
│  • Modifying text character by character (ASCII only)           │
│                                                                 │
│  USE []rune WHEN:                                               │
│  • Counting characters (not bytes)                              │
│  • Random access to characters                                  │
│  • Reversing strings                                            │
│  • Character-level text processing                              │
│  • Any operation needing Unicode awareness                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🏭 Real Production Example

```go
// production_example.go
package main

import (
    "fmt"
    "unicode/utf8"
)

// Production pattern: Safely truncate display name
func TruncateDisplayName(name string, maxRunes int) string {
    runes := []rune(name)
    if len(runes) <= maxRunes {
        return name
    }
    return string(runes[:maxRunes-1]) + "…"
}

// Production pattern: Validate UTF-8 input
func ValidateUTF8Input(input string) error {
    if !utf8.ValidString(input) {
        return fmt.Errorf("invalid UTF-8 encoding")
    }
    return nil
}

// Production pattern: Count visible characters (for limits)
func IsWithinCharLimit(s string, limit int) bool {
    return utf8.RuneCountInString(s) <= limit
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION PATTERNS                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Truncation
    fmt.Println("\n📊 Display Name Truncation:")
    names := []string{
        "John Doe",
        "राहुल कुमार",
        "王小明很长的名字",
        "😀😀😀😀😀😀😀😀",
    }
    
    for _, name := range names {
        truncated := TruncateDisplayName(name, 8)
        fmt.Printf("   %-20s → %s\n", name, truncated)
    }
    
    // Character limit validation
    fmt.Println("\n📊 Character Limit (10 chars max):")
    inputs := []string{
        "Hello",
        "HelloWorld",
        "Hello World",
        "你好世界",
    }
    
    for _, input := range inputs {
        within := IsWithinCharLimit(input, 10)
        runeCount := utf8.RuneCountInString(input)
        byteLen := len(input)
        status := "✅"
        if !within {
            status = "❌"
        }
        fmt.Printf("   %s %q (runes: %d, bytes: %d)\n", 
            status, input, runeCount, byteLen)
    }
}
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  String (UTF-16 internally)        string (UTF-8 internally)    │
│  char (16-bit, UTF-16)             rune (32-bit, full Unicode)  │
│  byte (8-bit)                      byte (8-bit)                 │
│                                                                 │
│  str.length()                      utf8.RuneCountInString(s)    │
│  (returns code units)              (returns runes)              │
│                                                                 │
│  str.charAt(i)                     []rune(s)[i]                 │
│  (may return half of emoji!)       (always complete character)  │
│                                                                 │
│  str.getBytes("UTF-8")             []byte(s)                    │
│  (explicit encoding)               (already UTF-8)              │
│                                                                 │
│  Java's char problem:                                           │
│  "😀".length() = 2 (surrogate pair!)                            │
│  "😀".charAt(0) = high surrogate (broken!)                      │
│                                                                 │
│  Go's solution:                                                 │
│  []rune("😀") = [128512] (one complete rune)                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

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

