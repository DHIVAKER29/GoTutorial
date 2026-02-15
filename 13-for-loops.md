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

Go uses one keyword for all looping. Other languages have `for`, `while`, `do-while`, `foreach`; Go uses `for` for all of these.

| Other Languages | Go Equivalent |
|-----------------|---------------|
| `for (i=0; i<10; i++)` | `for i := 0; i < 10; i++` |
| `while (condition)` | `for condition` |
| `do { } while` | `for { }` with `break` |
| `for (item : collection)` | `for _, item := range collection` |

---

## 📝 The Four Forms of `for`

### Form 1: Classic For Loop

```go
for i := 0; i < 5; i++ {
    fmt.Print(i, " ")
}
// Output: 0 1 2 3 4
```

### Form 2: While-Style Loop

```go
count := 0
for count < 5 {
    fmt.Print(count, " ")
    count++
}
// Output: 0 1 2 3 4
```

### Form 3: Infinite Loop

```go
sum := 0
for {
    sum++
    if sum > 3 {
        break
    }
    fmt.Print(sum, " ")
}
// Output: 1 2 3
```

### Form 4: Range Loop

```go
fruits := []string{"apple", "banana", "cherry"}
for i, fruit := range fruits {
    fmt.Printf("[%d] %s\n", i, fruit)
}
// Output: [0] apple
//         [1] banana
//         [2] cherry
```

Range over map (order is random):

```go
ages := map[string]int{"Alice": 25, "Bob": 30}
for name, age := range ages {
    fmt.Printf("%s is %d\n", name, age)
}
```

Use `_` to ignore index or value:

```go
for _, num := range []int{10, 20, 30} {
    fmt.Print(num, " ")
}
// Output: 10 20 30

for i := range []int{10, 20, 30} {
    fmt.Print(i, " ")
}
// Output: 0 1 2
```

**Note:** Range over strings iterates runes, not bytes.

### Complete Example: For Loop Forms

```go
package main

import "fmt"

func main() {
    for i := 0; i < 3; i++ {
        fmt.Print(i, " ")
    }
    fmt.Println()

    fruits := []string{"apple", "banana"}
    for i, f := range fruits {
        fmt.Printf("%d:%s ", i, f)
    }
}
```

**Output:**
```
0 1 2
0:apple 1:banana
```

---

## 🔄 Break and Continue

`break` exits the loop; `continue` skips to the next iteration.

```go
for i := 1; i <= 5; i++ {
    if i%2 == 0 {
        continue
    }
    fmt.Print(i, " ")
}
// Output: 1 3 5
```

```go
for i := 1; i <= 10; i++ {
    if i == 4 {
        break
    }
    fmt.Print(i, " ")
}
// Output: 1 2 3
```

**Labels** let you break or continue an outer loop:

```go
outer:
    for i := 1; i <= 2; i++ {
        for j := 1; j <= 2; j++ {
            if i == 2 && j == 2 {
                break outer
            }
            fmt.Printf("i=%d j=%d ", i, j)
        }
    }
// Output: i=1 j=1 i=1 j=2 i=2 j=1
```

### Complete Example: Break and Continue

```go
package main

import "fmt"

func main() {
    for i := 1; i <= 5; i++ {
        if i%2 == 0 {
            continue
        }
        fmt.Print(i, " ")
    }
    fmt.Println()
    for i := 1; i <= 5; i++ {
        if i == 3 {
            break
        }
        fmt.Print(i, " ")
    }
}
```

**Output:**
```
1 3 5
1 2
```

---

## 🏭 Production Patterns

**Retry with backoff:**

```go
for attempt := 1; attempt <= 3; attempt++ {
    if doWork() {
        break
    }
    time.Sleep(time.Duration(attempt) * 100 * time.Millisecond)
}
```

**Batch processing:**

```go
items := []string{"a", "b", "c", "d", "e"}
batchSize := 2
for i := 0; i < len(items); i += batchSize {
    end := i + batchSize
    if end > len(items) {
        end = len(items)
    }
    batch := items[i:end]
    fmt.Println(batch)
}
// Output: [a b]
//         [c d]
//         [e]
```

**Finding element:**

```go
nums := []int{10, 25, 30, 45}
for i, n := range nums {
    if n > 30 {
        fmt.Println(n, "at", i)
        break
    }
}
// Output: 45 at 3
```

### Complete Example: Production Pattern

```go
package main

import "fmt"

func main() {
    items := []string{"a", "b", "c", "d"}
    for i := 0; i < len(items); i += 2 {
        end := i + 2
        if end > len(items) {
            end = len(items)
        }
        fmt.Println(items[i:end])
    }
}
```

**Output:**
```
[a b]
[c d]
```

---

## ⚠️ Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Modifying slice while iterating | Unpredictable behavior | Iterate backwards or build new slice |
| Capturing loop variable in goroutine | All goroutines see last value | Pass as parameter: `go func(v int) { ... }(v)` |
| Infinite loop without exit | Runs forever | Always have `break` or `return` |

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `for (int i = 0; i < 10; i++)` | `for i := 0; i < 10; i++` |
| `while (condition)` | `for condition` |
| `do { } while (condition)` | No equivalent; use `for { }` + `break` |
| `for (String s : list)` | `for _, s := range slice` |
| `outer: for (...) { break outer; }` | `outer: for ... { break outer }` |

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
