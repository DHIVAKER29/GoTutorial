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

```go
// regex_basics.go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           REGULAR EXPRESSIONS                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Compile pattern
    fmt.Println("\n📊 Compile Pattern:")
    pattern := `\d+`  // One or more digits
    re, err := regexp.Compile(pattern)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
        return
    }
    fmt.Printf("   Compiled: %s\n", pattern)
    
    // MustCompile (panics on error - use for known patterns)
    reEmail := regexp.MustCompile(`[\w.]+@[\w.]+\.\w+`)
    fmt.Printf("   Email pattern compiled\n")
    
    // Match (returns bool)
    fmt.Println("\n📊 MatchString (bool):")
    text := "Order #12345 confirmed"
    fmt.Printf("   Text: %q\n", text)
    fmt.Printf("   Has digits: %t\n", re.MatchString(text))
    
    // Find (returns first match)
    fmt.Println("\n📊 FindString (first match):")
    match := re.FindString(text)
    fmt.Printf("   First digits: %q\n", match)
    
    // FindAll (returns all matches)
    fmt.Println("\n📊 FindAllString (all matches):")
    text2 := "Items: 10, 20, 30"
    matches := re.FindAllString(text2, -1)  // -1 = all
    fmt.Printf("   All digits: %v\n", matches)
    
    // FindStringIndex (position)
    fmt.Println("\n📊 FindStringIndex (position):")
    loc := re.FindStringIndex(text)
    fmt.Printf("   Found at index %d-%d\n", loc[0], loc[1])
    
    // Email matching
    fmt.Println("\n📊 Email Matching:")
    emails := []string{
        "user@example.com",
        "invalid-email",
        "test.user@company.org",
    }
    for _, email := range emails {
        valid := reEmail.MatchString(email)
        fmt.Printf("   %s: %t\n", email, valid)
    }
}
```

---

## 🔄 Replace and Split

```go
// regex_replace.go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           REPLACE & SPLIT                                 ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Replace
    fmt.Println("\n📊 ReplaceAllString:")
    re := regexp.MustCompile(`\d+`)
    text := "Order 123 and Order 456"
    replaced := re.ReplaceAllString(text, "XXX")
    fmt.Printf("   Before: %s\n", text)
    fmt.Printf("   After:  %s\n", replaced)
    
    // Replace with function
    fmt.Println("\n📊 ReplaceAllStringFunc:")
    doubled := re.ReplaceAllStringFunc(text, func(s string) string {
        var n int
        fmt.Sscanf(s, "%d", &n)
        return fmt.Sprintf("%d", n*2)
    })
    fmt.Printf("   Doubled: %s\n", doubled)
    
    // Replace with capture groups
    fmt.Println("\n📊 Replace with Groups:")
    reDate := regexp.MustCompile(`(\d{4})-(\d{2})-(\d{2})`)
    date := "Date: 2024-12-25"
    newFormat := reDate.ReplaceAllString(date, "$2/$3/$1")  // MM/DD/YYYY
    fmt.Printf("   Before: %s\n", date)
    fmt.Printf("   After:  %s\n", newFormat)
    
    // Split
    fmt.Println("\n📊 Split:")
    reSep := regexp.MustCompile(`[,;]\s*`)
    items := "apple, banana; cherry, date"
    parts := reSep.Split(items, -1)
    fmt.Printf("   Split: %v\n", parts)
}
```

---

## 📦 Capture Groups

```go
// regex_groups.go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CAPTURE GROUPS                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Named groups
    fmt.Println("\n📊 Named Capture Groups:")
    re := regexp.MustCompile(`(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})`)
    text := "Date: 2024-12-25"
    
    match := re.FindStringSubmatch(text)
    if match != nil {
        fmt.Printf("   Full match: %s\n", match[0])
        for i, name := range re.SubexpNames() {
            if i > 0 && name != "" {
                fmt.Printf("   %s: %s\n", name, match[i])
            }
        }
    }
    
    // Multiple matches with groups
    fmt.Println("\n📊 All Matches with Groups:")
    reLog := regexp.MustCompile(`\[(\w+)\] (.+)`)
    logs := `[INFO] Server started
[ERROR] Connection failed
[WARN] Slow response`
    
    allMatches := reLog.FindAllStringSubmatch(logs, -1)
    for _, m := range allMatches {
        fmt.Printf("   Level: %-5s  Message: %s\n", m[1], m[2])
    }
}
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

