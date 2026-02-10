# 13 - For Loops

> Go has only ONE loop construct: `for`. But it can do everything!

---

## 📌 What You'll Learn

- Why Go has only `for` (no while, do-while)
- All four forms of the for loop
- Range loops for collections
- Loop control: break, continue
- Common patterns and best practices
- Sample programs for each pattern

---

## 🤔 Why Only `for`?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  OTHER LANGUAGES:                                               │
│  ─────────────────                                              │
│  for (int i = 0; i < 10; i++) { }   // C-style for              │
│  while (condition) { }               // While loop              │
│  do { } while (condition);           // Do-while loop           │
│  for (item : collection) { }         // For-each                │
│  foreach (item in collection) { }    // C# foreach              │
│                                                                 │
│  GO'S PHILOSOPHY:                                               │
│  ────────────────                                               │
│  "One construct that does it all"                               │
│                                                                 │
│  for i := 0; i < 10; i++ { }   // Classic for                   │
│  for condition { }              // While-style                  │
│  for { }                        // Infinite loop                │
│  for i, v := range collection { }  // Range loop                │
│                                                                 │
│  Same keyword, same syntax, less to remember!                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 The Four Forms of `for`

### Form 1: Classic For Loop

```go
for init; condition; post {
    // code
}
```

### Form 2: While-Style Loop

```go
for condition {
    // code
}
```

### Form 3: Infinite Loop

```go
for {
    // code (use break to exit)
}
```

### Form 4: Range Loop

```go
for index, value := range collection {
    // code
}
```

---

## 📝 Sample Program: All For Loop Forms

```go
// for_loops.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              FOR LOOPS IN GO                              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Form 1: Classic for loop
    fmt.Println("\n📊 Form 1: Classic For Loop")
    fmt.Print("   ")
    for i := 0; i < 5; i++ {
        fmt.Printf("%d ", i)
    }
    fmt.Println()
    
    // Descending
    fmt.Print("   Descending: ")
    for i := 5; i > 0; i-- {
        fmt.Printf("%d ", i)
    }
    fmt.Println()
    
    // Step by 2
    fmt.Print("   Step by 2: ")
    for i := 0; i <= 10; i += 2 {
        fmt.Printf("%d ", i)
    }
    fmt.Println()
    
    // Form 2: While-style loop
    fmt.Println("\n📊 Form 2: While-Style Loop")
    count := 0
    fmt.Print("   ")
    for count < 5 {
        fmt.Printf("%d ", count)
        count++
    }
    fmt.Println()
    
    // Form 3: Infinite loop (with break)
    fmt.Println("\n📊 Form 3: Infinite Loop (with break)")
    sum := 0
    fmt.Print("   Sum until > 10: ")
    for {
        sum++
        fmt.Printf("%d ", sum)
        if sum > 10 {
            break
        }
    }
    fmt.Println()
    
    // Form 4: Range loop over slice
    fmt.Println("\n📊 Form 4: Range Loop - Slice")
    fruits := []string{"apple", "banana", "cherry"}
    for index, fruit := range fruits {
        fmt.Printf("   [%d] %s\n", index, fruit)
    }
    
    // Range over string (iterates runes!)
    fmt.Println("\n📊 Range Loop - String (runes)")
    greeting := "Hello世界"
    for index, char := range greeting {
        fmt.Printf("   [%d] %c (U+%04X)\n", index, char, char)
    }
    
    // Range over map
    fmt.Println("\n📊 Range Loop - Map")
    ages := map[string]int{
        "Alice": 25,
        "Bob":   30,
        "Carol": 28,
    }
    for name, age := range ages {
        fmt.Printf("   %s is %d years old\n", name, age)
    }
    
    // Range with blank identifier
    fmt.Println("\n📊 Ignoring Index or Value")
    numbers := []int{10, 20, 30, 40, 50}
    
    // Ignore index
    fmt.Print("   Values only: ")
    for _, num := range numbers {
        fmt.Printf("%d ", num)
    }
    fmt.Println()
    
    // Ignore value (just need index)
    fmt.Print("   Indices only: ")
    for i := range numbers {
        fmt.Printf("%d ", i)
    }
    fmt.Println()
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              FOR LOOPS IN GO                              ║
╚══════════════════════════════════════════════════════════╝

📊 Form 1: Classic For Loop
   0 1 2 3 4 
   Descending: 5 4 3 2 1 
   Step by 2: 0 2 4 6 8 10 

📊 Form 2: While-Style Loop
   0 1 2 3 4 

📊 Form 3: Infinite Loop (with break)
   Sum until > 10: 1 2 3 4 5 6 7 8 9 10 11 

📊 Form 4: Range Loop - Slice
   [0] apple
   [1] banana
   [2] cherry

📊 Range Loop - String (runes)
   [0] H (U+0048)
   [1] e (U+0065)
   [2] l (U+006C)
   [3] l (U+006C)
   [4] o (U+006F)
   [5] 世 (U+4E16)
   [8] 界 (U+754C)

📊 Range Loop - Map
   Alice is 25 years old
   Bob is 30 years old
   Carol is 28 years old

📊 Ignoring Index or Value
   Values only: 10 20 30 40 50 
   Indices only: 0 1 2 3 4 
```

---

## 🔄 Break and Continue

### Sample Program: Loop Control

```go
// break_continue.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           BREAK AND CONTINUE                              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // break - exit the loop
    fmt.Println("\n📊 break - Exit Loop Early")
    fmt.Print("   Finding first even: ")
    for i := 1; i <= 10; i++ {
        if i%2 == 0 {
            fmt.Printf("Found %d!\n", i)
            break  // Exit the loop
        }
        fmt.Printf("%d ", i)
    }
    
    // continue - skip to next iteration
    fmt.Println("\n📊 continue - Skip Iteration")
    fmt.Print("   Odd numbers only: ")
    for i := 1; i <= 10; i++ {
        if i%2 == 0 {
            continue  // Skip even numbers
        }
        fmt.Printf("%d ", i)
    }
    fmt.Println()
    
    // break in nested loops (only exits inner loop!)
    fmt.Println("\n📊 break in Nested Loops")
    fmt.Println("   break only exits INNER loop:")
    for i := 1; i <= 3; i++ {
        for j := 1; j <= 3; j++ {
            if j == 2 {
                break  // Only exits inner loop!
            }
            fmt.Printf("   i=%d, j=%d\n", i, j)
        }
    }
    
    // Using labels to break outer loop
    fmt.Println("\n📊 Labels - Break Outer Loop")
outer:
    for i := 1; i <= 3; i++ {
        for j := 1; j <= 3; j++ {
            if i == 2 && j == 2 {
                fmt.Println("   Breaking outer loop!")
                break outer  // Breaks outer loop!
            }
            fmt.Printf("   i=%d, j=%d\n", i, j)
        }
    }
    
    // Using labels with continue
    fmt.Println("\n📊 Labels - Continue Outer Loop")
outerLoop:
    for i := 1; i <= 3; i++ {
        for j := 1; j <= 3; j++ {
            if j == 2 {
                fmt.Printf("   Skipping rest of inner loop at i=%d\n", i)
                continue outerLoop  // Continues outer loop!
            }
            fmt.Printf("   i=%d, j=%d\n", i, j)
        }
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           BREAK AND CONTINUE                              ║
╚══════════════════════════════════════════════════════════╝

📊 break - Exit Loop Early
   Finding first even: 1 Found 2!

📊 continue - Skip Iteration
   Odd numbers only: 1 3 5 7 9 

📊 break in Nested Loops
   break only exits INNER loop:
   i=1, j=1
   i=2, j=1
   i=3, j=1

📊 Labels - Break Outer Loop
   i=1, j=1
   i=1, j=2
   i=1, j=3
   i=2, j=1
   Breaking outer loop!

📊 Labels - Continue Outer Loop
   i=1, j=1
   Skipping rest of inner loop at i=1
   i=2, j=1
   Skipping rest of inner loop at i=2
   i=3, j=1
   Skipping rest of inner loop at i=3
```

---

## 🏭 Production Patterns

### Sample Program: Real-World Patterns

```go
// for_production.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION FOR LOOP PATTERNS                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: Retry with backoff
    fmt.Println("\n📊 Pattern 1: Retry with Backoff")
    success := retryWithBackoff(3)
    fmt.Printf("   Final result: %v\n", success)
    
    // Pattern 2: Pagination
    fmt.Println("\n📊 Pattern 2: Pagination")
    processAllPages()
    
    // Pattern 3: Batch processing
    fmt.Println("\n📊 Pattern 3: Batch Processing")
    items := []string{"a", "b", "c", "d", "e", "f", "g", "h"}
    batchSize := 3
    processBatches(items, batchSize)
    
    // Pattern 4: Finding element
    fmt.Println("\n📊 Pattern 4: Finding Element")
    numbers := []int{10, 25, 30, 45, 50}
    found, index := findFirst(numbers, func(n int) bool {
        return n > 30
    })
    fmt.Printf("   First number > 30: %d at index %d\n", found, index)
    
    // Pattern 5: Filtering
    fmt.Println("\n📊 Pattern 5: Filtering")
    allUsers := []string{"alice", "admin", "bob", "charlie"}
    activeUsers := filter(allUsers, func(s string) bool {
        return s != "admin"
    })
    fmt.Printf("   Active users: %v\n", activeUsers)
    
    // Pattern 6: Transforming (map)
    fmt.Println("\n📊 Pattern 6: Transforming")
    nums := []int{1, 2, 3, 4, 5}
    doubled := transform(nums, func(n int) int {
        return n * 2
    })
    fmt.Printf("   Doubled: %v\n", doubled)
    
    // Pattern 7: Two-pointer technique
    fmt.Println("\n📊 Pattern 7: Two-Pointer (Palindrome Check)")
    words := []string{"radar", "hello", "level", "world"}
    for _, word := range words {
        result := "❌"
        if isPalindrome(word) {
            result = "✅"
        }
        fmt.Printf("   %s: %s\n", word, result)
    }
}

// Retry with exponential backoff
func retryWithBackoff(maxRetries int) bool {
    for attempt := 1; attempt <= maxRetries; attempt++ {
        fmt.Printf("   Attempt %d...\n", attempt)
        
        // Simulate operation (fails first 2 times)
        if attempt >= 2 {
            fmt.Println("   Success!")
            return true
        }
        
        // Wait before retry
        backoff := time.Duration(attempt) * 100 * time.Millisecond
        fmt.Printf("   Failed, waiting %v\n", backoff)
        time.Sleep(backoff)
    }
    return false
}

// Pagination pattern
func processAllPages() {
    page := 1
    for {
        items := fetchPage(page)
        if len(items) == 0 {
            break  // No more pages
        }
        fmt.Printf("   Page %d: %v\n", page, items)
        page++
    }
}

func fetchPage(page int) []string {
    // Simulate paginated data
    data := map[int][]string{
        1: {"item1", "item2"},
        2: {"item3", "item4"},
        3: {},  // Empty = last page
    }
    return data[page]
}

// Batch processing
func processBatches(items []string, batchSize int) {
    for i := 0; i < len(items); i += batchSize {
        end := i + batchSize
        if end > len(items) {
            end = len(items)
        }
        batch := items[i:end]
        fmt.Printf("   Batch: %v\n", batch)
    }
}

// Find first matching element
func findFirst(nums []int, predicate func(int) bool) (int, int) {
    for i, n := range nums {
        if predicate(n) {
            return n, i
        }
    }
    return 0, -1
}

// Filter elements
func filter(items []string, predicate func(string) bool) []string {
    result := make([]string, 0)
    for _, item := range items {
        if predicate(item) {
            result = append(result, item)
        }
    }
    return result
}

// Transform elements
func transform(nums []int, fn func(int) int) []int {
    result := make([]int, len(nums))
    for i, n := range nums {
        result[i] = fn(n)
    }
    return result
}

// Palindrome check with two pointers
func isPalindrome(s string) bool {
    runes := []rune(s)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        if runes[i] != runes[j] {
            return false
        }
    }
    return true
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           PRODUCTION FOR LOOP PATTERNS                    ║
╚══════════════════════════════════════════════════════════╝

📊 Pattern 1: Retry with Backoff
   Attempt 1...
   Failed, waiting 100ms
   Attempt 2...
   Success!
   Final result: true

📊 Pattern 2: Pagination
   Page 1: [item1 item2]
   Page 2: [item3 item4]

📊 Pattern 3: Batch Processing
   Batch: [a b c]
   Batch: [d e f]
   Batch: [g h]

📊 Pattern 4: Finding Element
   First number > 30: 45 at index 3

📊 Pattern 5: Filtering
   Active users: [alice bob charlie]

📊 Pattern 6: Transforming
   Doubled: [2 4 6 8 10]

📊 Pattern 7: Two-Pointer (Palindrome Check)
   radar: ✅
   hello: ❌
   level: ✅
   world: ❌
```

---

## ⚠️ Common Mistakes

```go
// mistakes.go

// MISTAKE 1: Modifying slice while iterating
// ❌ BAD
for i, v := range slice {
    if v == target {
        slice = append(slice[:i], slice[i+1:]...)  // Modifies slice!
    }
}

// ✅ GOOD: Iterate backwards or build new slice
for i := len(slice) - 1; i >= 0; i-- {
    if slice[i] == target {
        slice = append(slice[:i], slice[i+1:]...)
    }
}

// MISTAKE 2: Capturing loop variable in goroutine
// ❌ BAD
for _, v := range values {
    go func() {
        fmt.Println(v)  // v is shared, will print last value!
    }()
}

// ✅ GOOD: Pass as parameter
for _, v := range values {
    go func(val int) {
        fmt.Println(val)  // Each goroutine has its own copy
    }(v)
}

// MISTAKE 3: Infinite loop without exit
// ❌ BAD
for {
    // No break or return - runs forever!
}

// ✅ GOOD: Always have an exit condition
for {
    if shouldStop {
        break
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
│  // Traditional for               // Same                       │
│  for (int i = 0; i < 10; i++)     for i := 0; i < 10; i++       │
│                                                                 │
│  // While loop                    // While-style for            │
│  while (condition) { }            for condition { }             │
│                                                                 │
│  // Do-while                      // No equivalent              │
│  do { } while (condition);        // Use infinite + break       │
│                                                                 │
│  // For-each                      // Range                      │
│  for (String s : list)            for _, s := range slice       │
│                                                                 │
│  // Labeled break                 // Labeled break              │
│  outer: for (...) {               outer: for ... {              │
│      break outer;                     break outer               │
│  }                                }                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Go has only `for`** - but it does everything!
2. **Classic**: `for i := 0; i < n; i++ {}`
3. **While-style**: `for condition {}`
4. **Infinite**: `for {}`
5. **Range**: `for i, v := range collection {}`
6. **Use `_`** to ignore index or value in range
7. **Use labels** to break/continue outer loops
8. **Range over strings** iterates runes, not bytes!

---

## ➡️ Next Steps

You now understand for loops. Let's explore break, continue, and labels in more detail.

**Next Topic:** [14 - Break, Continue & Labels](./14-break-continue-labels.md)

