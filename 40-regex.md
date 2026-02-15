# 40 - Regular Expressions

> Pattern matching and text manipulation with regex in Go.

---

## 📌 What You'll Learn

- Compiling and using regex patterns
- Matching, finding, and replacing
- Capture groups
- Common patterns

---

## 📝 Regex Basics

**Compile pattern:**
```go
re, err := regexp.Compile(`\d+`)
// MustCompile panics on error — use for known patterns
reEmail := regexp.MustCompile(`[\w.]+@[\w.]+\.\w+`)
```

**MatchString (returns bool):**
```go
text := "Order #12345 confirmed"
re.MatchString(text)  // true
// Output: true
```

**FindString (first match):**
```go
match := re.FindString(text)
fmt.Printf("%q\n", match)
// Output: "12345"
```

**FindAllString (all matches):**
```go
text2 := "Items: 10, 20, 30"
matches := re.FindAllString(text2, -1)  // -1 = all
fmt.Printf("%v\n", matches)
// Output: [10 20 30]
```

**FindStringIndex (position):**
```go
loc := re.FindStringIndex(text)
fmt.Printf("Found at %d-%d\n", loc[0], loc[1])
// Output: Found at 7-12
```

### Complete Example: Regex Basics

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    re := regexp.MustCompile(`\d+`)
    text := "Order #12345 confirmed"
    fmt.Printf("Has digits: %t\n", re.MatchString(text))
    fmt.Printf("First match: %q\n", re.FindString(text))

    text2 := "Items: 10, 20, 30"
    matches := re.FindAllString(text2, -1)
    fmt.Printf("All matches: %v\n", matches)
}
```

**Output:**
```
Has digits: true
First match: "12345"
All matches: [10 20 30]
```

---

## 🔄 Replace and Split

**ReplaceAllString:**
```go
re := regexp.MustCompile(`\d+`)
text := "Order 123 and Order 456"
replaced := re.ReplaceAllString(text, "XXX")
fmt.Println(replaced)
// Output: Order XXX and Order XXX
```

**ReplaceAllStringFunc:**
```go
doubled := re.ReplaceAllStringFunc(text, func(s string) string {
    var n int
    fmt.Sscanf(s, "%d", &n)
    return fmt.Sprintf("%d", n*2)
})
// Output: Order 246 and Order 912
```

**Replace with capture groups:**
```go
reDate := regexp.MustCompile(`(\d{4})-(\d{2})-(\d{2})`)
date := "Date: 2024-12-25"
newFormat := reDate.ReplaceAllString(date, "$2/$3/$1")  // MM/DD/YYYY
// Output: Date: 12/25/2024
```

**Split:**
```go
reSep := regexp.MustCompile(`[,;]\s*`)
parts := reSep.Split("apple, banana; cherry", -1)
// Output: [apple banana cherry]
```

### Complete Example: Replace and Split

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    re := regexp.MustCompile(`\d+`)
    text := "Order 123 and Order 456"
    fmt.Printf("Replace: %s\n", re.ReplaceAllString(text, "XXX"))

    reDate := regexp.MustCompile(`(\d{4})-(\d{2})-(\d{2})`)
    date := "Date: 2024-12-25"
    fmt.Printf("Reformat: %s\n", reDate.ReplaceAllString(date, "$2/$3/$1"))

    reSep := regexp.MustCompile(`[,;]\s*`)
    parts := reSep.Split("apple, banana; cherry", -1)
    fmt.Printf("Split: %v\n", parts)
}
```

**Output:**
```
Replace: Order XXX and Order XXX
Reformat: Date: 12/25/2024
Split: [apple banana cherry]
```

---

## 📦 Capture Groups

**Named groups:**
```go
re := regexp.MustCompile(`(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})`)
match := re.FindStringSubmatch("Date: 2024-12-25")
// match[0]=full, match[1]=year, match[2]=month, match[3]=day
```

**All matches with groups:**
```go
reLog := regexp.MustCompile(`\[(\w+)\] (.+)`)
allMatches := reLog.FindAllStringSubmatch(logs, -1)
for _, m := range allMatches {
    level, message := m[1], m[2]
}
```

### Complete Example: Capture Groups

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    re := regexp.MustCompile(`(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})`)
    text := "Date: 2024-12-25"
    match := re.FindStringSubmatch(text)
    if match != nil {
        fmt.Printf("Full: %s, year=%s, month=%s, day=%s\n",
            match[0], match[1], match[2], match[3])
    }

    reLog := regexp.MustCompile(`\[(\w+)\] (.+)`)
    logs := "[INFO] Server started\n[ERROR] Connection failed"
    for _, m := range reLog.FindAllStringSubmatch(logs, -1) {
        fmt.Printf("Level: %-5s Message: %s\n", m[1], m[2])
    }
}
```

**Output:**
```
Full: 2024-12-25, year=2024, month=12, day=25
Level: INFO  Message: Server started
Level: ERROR Message: Connection failed
```

---

## 🎯 Key Takeaways

1. **regexp.MustCompile** for known patterns
2. **MatchString** returns bool
3. **FindString/FindAllString** for matches
4. **ReplaceAllString** for replacements
5. **`(?P<name>...)`** for named groups
6. **Go uses RE2** (no backtracking, linear time)

---

## ➡️ Next Steps

**Next Topic:** [41 - Command Line Tools](./41-cli.md)
