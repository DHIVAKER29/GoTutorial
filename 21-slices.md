# 21 - Slices

> Dynamic, flexible views into arrays - the most commonly used collection in Go.

---

## 📌 What You'll Learn

- What slices are and how they differ from arrays
- Slice internals: length, capacity, and underlying array
- Creating, modifying, and growing slices
- The `append` function and how it works
- Slice expressions and sub-slices
- Common pitfalls (shared backing arrays!)
- Sample programs for every concept

---

## 🤔 What is a Slice?

### Definition

> A **slice** is a dynamically-sized, flexible view into an underlying array. It provides a more powerful and convenient interface than arrays.

### Real-World Analogy

A slice is like a window into a bookshelf (the underlying array). The slice doesn't own the data—it's a view. Changes through the slice affect the original array. You can "slide" the window to see different portions (slice expressions).

---

## 📊 Slice Internals: The Slice Header

A slice variable holds a **slice header** with three components:

- **Pointer**: Points to the first element in the underlying array
- **Length (len)**: Number of elements in the view
- **Capacity (cap)**: Number of elements from start to end of underlying array

Example: For `s := arr[1:4]` where `arr` has 6 elements, `len(s)=3`, `cap(s)=5` (room to grow within the array before reallocation).

---

## 📝 Creating Slices

```go
s1 := []int{10, 20, 30, 40, 50}
fmt.Println(s1, len(s1), cap(s1))  // Output: [10 20 30 40 50] 5 5
```

```go
s2 := make([]int, 5)
fmt.Println(s2)  // Output: [0 0 0 0 0]
```

```go
s3 := make([]int, 3, 10)
fmt.Println(s3, len(s3), cap(s3))  // Output: [0 0 0] 3 10
```

```go
arr := [6]int{10, 20, 30, 40, 50, 60}
s4 := arr[1:4]
fmt.Println(s4, len(s4), cap(s4))  // Output: [20 30 40] 3 5
```

```go
var s5 []int  // nil slice
fmt.Println(s5 == nil, len(s5))  // Output: true 0
```

```go
s6 := []int{}  // empty slice (not nil)
fmt.Println(s6 == nil)  // Output: false
```

### Complete Example: Creating Slices

```go
package main

import "fmt"

func main() {
    s1 := []int{10, 20, 30, 40, 50}
    s2 := make([]int, 3, 10)
    arr := [6]int{10, 20, 30, 40, 50, 60}
    s3 := arr[1:4]

    fmt.Println(s1, len(s1), cap(s1))  // [10 20 30 40 50] 5 5
    fmt.Println(s2, len(s2), cap(s2))  // [0 0 0] 3 10
    fmt.Println(s3, len(s3), cap(s3))  // [20 30 40] 3 5
}
```

**Output:**
```
[10 20 30 40 50] 5 5
[0 0 0] 3 10
[20 30 40] 3 5
```

---

## 🔄 Slice Expressions

```go
s := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
fmt.Println(s[2:5])   // Output: [2 3 4]
fmt.Println(s[:5])    // Output: [0 1 2 3 4]
fmt.Println(s[5:])    // Output: [5 6 7 8 9]
fmt.Println(s[:])     // Output: [0 1 2 3 4 5 6 7 8 9]
```

```go
s2 := s[2:5:7]  // Full slice: limits capacity
fmt.Println(s2, len(s2), cap(s2))  // Output: [2 3 4] 3 5
```

**Capacity rule**: For `s[low:high]`, `cap = len(s) - low`.

---

## ➕ The append Function

```go
s := []int{1, 2, 3}
s = append(s, 4)  // MUST reassign!
fmt.Println(s)    // Output: [1 2 3 4]
```

```go
s = append(s, 5, 6, 7)
more := []int{8, 9, 10}
s = append(s, more...)  // Spread operator
fmt.Println(s)  // Output: [1 2 3 4 5 6 7 8 9 10]
```

```go
var nilSlice []int
nilSlice = append(nilSlice, 1, 2, 3)
fmt.Println(nilSlice)  // Output: [1 2 3] (append works on nil!)
```

**Important**: Always reassign: `s = append(s, x)`. The result of `append` may be a new slice if capacity was exceeded.

### Complete Example: append

```go
package main

import "fmt"

func main() {
    s := []int{1, 2, 3}
    s = append(s, 4)
    s = append(s, 5, 6, 7)
    fmt.Println(s)    // [1 2 3 4 5 6 7]
    fmt.Println(len(s), cap(s))  // 7 8 (capacity doubles when needed)
}
```

**Output:**
```
[1 2 3 4 5 6 7]
7 8
```

---

## ⚠️ Critical: Shared Backing Array!

Slices created from the same array share the underlying data. Modifying one affects the other.

```go
original := []int{1, 2, 3, 4, 5}
sub := original[1:4]
sub[0] = 999
fmt.Println(original)  // Output: [1 999 3 4 5] (changed!)
fmt.Println(sub)       // Output: [999 3 4]
```

**Solution**: Use `copy()` for independent copies.

```go
original := []int{1, 2, 3, 4, 5}
sub := make([]int, 3)
copy(sub, original[1:4])
sub[0] = 999
fmt.Println(original)  // Output: [1 2 3 4 5] (unchanged!)
fmt.Println(sub)       // Output: [999 3 4]
```

**Prevention**: Full slice expression `s[low:high:max]` limits capacity, forcing new allocation on append when needed.

---

## 📋 The copy Function

```go
src := []int{1, 2, 3, 4, 5}
dst := make([]int, len(src))
n := copy(dst, src)
fmt.Println(n, dst)  // Output: 5 [1 2 3 4 5]
```

```go
small := make([]int, 3)
n = copy(small, src)
fmt.Println(n, small)  // Output: 3 [1 2 3]
```

```go
s := []int{0, 1, 2, 3, 4, 5}
copy(s[0:], s[2:])  // Shift left
fmt.Println(s)  // Output: [2 3 4 5 4 5]
```

---

## 🏭 Production Patterns

```go
// Pre-allocate when size known
result := make([]int, 0, 1000)
for i := 0; i < 1000; i++ {
    result = append(result, i)
}
```

```go
// Stack (LIFO)
stack := []int{}
stack = append(stack, 1, 2, 3)
top := stack[len(stack)-1]
stack = stack[:len(stack)-1]
fmt.Println(top, stack)  // Output: 3 [1 2]
```

```go
// Queue (FIFO)
queue := []int{}
queue = append(queue, 1, 2, 3)
front := queue[0]
queue = queue[1:]
fmt.Println(front, queue)  // Output: 1 [2 3]
```

```go
// Remove at index
func removeAt(s []string, i int) []string {
    return append(s[:i], s[i+1:]...)
}
```

### Complete Example: Production Patterns

```go
package main

import "fmt"

func main() {
    // Pre-allocate
    result := make([]int, 0, 100)
    for i := 0; i < 100; i++ {
        result = append(result, i)
    }
    fmt.Println(len(result), cap(result))  // 100 100

    // Stack
    stack := []int{}
    stack = append(stack, 1, 2, 3)
    top := stack[len(stack)-1]
    stack = stack[:len(stack)-1]
    fmt.Println("Popped:", top, "Stack:", stack)
}
```

**Output:**
```
100 100
Popped: 3 Stack: [1 2]
```

---

## 🆚 Java Comparison

| Java | Go |
|------|-----|
| `ArrayList<Integer> list = new ArrayList<>();` | `slice := []int{}` |
| `list.add(x);` | `slice = append(slice, x)` |
| `list.get(i);` | `slice[i]` |
| `list.size();` | `len(slice)` |
| `list.subList(1, 4);` | `slice[1:4]` |
| Sublist is a VIEW (changes affect original) | Slice is a VIEW (changes affect original) |
| `new ArrayList<>(list)` for copy | `copy(dst, src)` |
| `int[] arr = new int[5];` | `arr := make([]int, 5)` |

---

## 🎯 Key Takeaways

1. **Slice = pointer + length + capacity** (not the data itself!)
2. **Dynamic size** - can grow with `append`
3. **Views share data** - modifications affect original!
4. **Use `copy()`** for independent copies
5. **`append` MUST be reassigned** - `s = append(s, x)`
6. **nil slice is valid** - len=0, cap=0, works with append
7. **Pre-allocate** when size is known for performance
8. **Full slice expression** `s[low:high:max]` limits capacity

---

## ➡️ Next Steps

**Next Topic:** [22 - Maps](./22-maps.md)
