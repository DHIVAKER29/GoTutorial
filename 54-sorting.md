# 54 - sort Package

> Sorting slices and custom collections in Go.

---

## 📌 What You'll Learn

- Sorting basic types
- Sorting custom types
- The sort.Interface
- Searching sorted slices

---

## 📝 Sorting Basics

```go
// sort_basics.go
package main

import (
    "fmt"
    "sort"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           sort Package Basics                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Sort ints
    fmt.Println("\n📊 Sort Ints:")
    ints := []int{5, 2, 8, 1, 9}
    fmt.Printf("   Before: %v\n", ints)
    sort.Ints(ints)
    fmt.Printf("   After:  %v\n", ints)
    
    // Sort strings
    fmt.Println("\n📊 Sort Strings:")
    strs := []string{"banana", "apple", "cherry"}
    fmt.Printf("   Before: %v\n", strs)
    sort.Strings(strs)
    fmt.Printf("   After:  %v\n", strs)
    
    // Sort float64s
    fmt.Println("\n📊 Sort Float64s:")
    floats := []float64{3.14, 1.41, 2.71}
    sort.Float64s(floats)
    fmt.Printf("   Sorted: %v\n", floats)
    
    // Check if sorted
    fmt.Println("\n📊 Check if Sorted:")
    fmt.Printf("   IntsAreSorted: %t\n", sort.IntsAreSorted(ints))
    
    // Reverse sort
    fmt.Println("\n📊 Reverse Sort:")
    nums := []int{1, 2, 3, 4, 5}
    sort.Sort(sort.Reverse(sort.IntSlice(nums)))
    fmt.Printf("   Reversed: %v\n", nums)
}
```

---

## 🔧 sort.Slice (Simple Custom Sorting)

```go
// sort_slice.go
package main

import (
    "fmt"
    "sort"
)

type Person struct {
    Name string
    Age  int
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           sort.Slice (Custom Sorting)                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    people := []Person{
        {"Alice", 30},
        {"Bob", 25},
        {"Charlie", 35},
        {"Diana", 25},
    }
    
    // Sort by age
    fmt.Println("\n📊 Sort by Age:")
    sort.Slice(people, func(i, j int) bool {
        return people[i].Age < people[j].Age
    })
    for _, p := range people {
        fmt.Printf("   %s: %d\n", p.Name, p.Age)
    }
    
    // Sort by name
    fmt.Println("\n📊 Sort by Name:")
    sort.Slice(people, func(i, j int) bool {
        return people[i].Name < people[j].Name
    })
    for _, p := range people {
        fmt.Printf("   %s: %d\n", p.Name, p.Age)
    }
    
    // Sort by age, then name (stable sort)
    fmt.Println("\n📊 Sort by Age, then Name (Stable):")
    sort.SliceStable(people, func(i, j int) bool {
        if people[i].Age != people[j].Age {
            return people[i].Age < people[j].Age
        }
        return people[i].Name < people[j].Name
    })
    for _, p := range people {
        fmt.Printf("   %s: %d\n", p.Name, p.Age)
    }
}
```

---

## 🎯 sort.Interface (Full Control)

```go
// sort_interface.go
package main

import (
    "fmt"
    "sort"
)

// The sort.Interface
// type Interface interface {
//     Len() int
//     Less(i, j int) bool
//     Swap(i, j int)
// }

type Person struct {
    Name string
    Age  int
}

// ByAge implements sort.Interface for []Person
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }

// ByName implements sort.Interface for []Person
type ByName []Person

func (a ByName) Len() int           { return len(a) }
func (a ByName) Less(i, j int) bool { return a[i].Name < a[j].Name }
func (a ByName) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           sort.Interface                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    people := []Person{
        {"Alice", 30},
        {"Bob", 25},
        {"Charlie", 35},
    }
    
    fmt.Println("\n📊 Sort using sort.Interface:")
    
    // Sort by age
    sort.Sort(ByAge(people))
    fmt.Println("   By Age:")
    for _, p := range people {
        fmt.Printf("      %s: %d\n", p.Name, p.Age)
    }
    
    // Sort by name
    sort.Sort(ByName(people))
    fmt.Println("   By Name:")
    for _, p := range people {
        fmt.Printf("      %s: %d\n", p.Name, p.Age)
    }
    
    // Reverse sort
    sort.Sort(sort.Reverse(ByAge(people)))
    fmt.Println("   By Age (Descending):")
    for _, p := range people {
        fmt.Printf("      %s: %d\n", p.Name, p.Age)
    }
}
```

---

## 🔍 Searching

```go
// sort_search.go
package main

import (
    "fmt"
    "sort"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Searching Sorted Slices                         ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Must be sorted first!
    ints := []int{1, 3, 5, 7, 9, 11}
    strs := []string{"apple", "banana", "cherry", "date"}
    
    // Binary search for ints
    fmt.Println("\n📊 sort.SearchInts:")
    idx := sort.SearchInts(ints, 5)
    fmt.Printf("   Index of 5: %d\n", idx)
    
    idx = sort.SearchInts(ints, 6)
    fmt.Printf("   Index for 6 (not found, insertion point): %d\n", idx)
    
    // Binary search for strings
    fmt.Println("\n📊 sort.SearchStrings:")
    idx = sort.SearchStrings(strs, "banana")
    fmt.Printf("   Index of 'banana': %d\n", idx)
    
    // Generic search
    fmt.Println("\n📊 sort.Search (generic):")
    target := 7
    idx = sort.Search(len(ints), func(i int) bool {
        return ints[i] >= target
    })
    if idx < len(ints) && ints[idx] == target {
        fmt.Printf("   Found %d at index %d\n", target, idx)
    } else {
        fmt.Printf("   %d not found\n", target)
    }
}
```

---

## 🎯 Key Takeaways

1. **sort.Ints/Strings/Float64s** for basic types
2. **sort.Slice** for quick custom sorting
3. **sort.Interface** for reusable sort types
4. **sort.Reverse** for descending order
5. **sort.SliceStable** preserves equal element order
6. **sort.Search** for binary search

---

## ➡️ Next Steps

**Next Topic:** [55 - Templates](./55-templates.md)

