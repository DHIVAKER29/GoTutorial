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
ints := []int{5, 2, 8, 1, 9}
sort.Ints(ints)
// Output: [1 2 5 8 9]

strs := []string{"banana", "apple", "cherry"}
sort.Strings(strs)
// Output: [apple banana cherry]

floats := []float64{3.14, 1.41, 2.71}
sort.Float64s(floats)
// Output: [1.41 2.71 3.14]

sort.IntsAreSorted(ints)  // true

// Reverse sort
sort.Sort(sort.Reverse(sort.IntSlice([]int{1, 2, 3, 4, 5})))
// Output: [5 4 3 2 1]
```

---

## 🔧 sort.Slice (Simple Custom Sorting)

```go
type Person struct {
    Name string
    Age  int
}

people := []Person{
    {"Alice", 30}, {"Bob", 25}, {"Charlie", 35}, {"Diana", 25},
}

// Sort by age
sort.Slice(people, func(i, j int) bool {
    return people[i].Age < people[j].Age
})
// Output: Bob, Diana, Alice, Charlie

// Sort by name
sort.Slice(people, func(i, j int) bool {
    return people[i].Name < people[j].Name
})

// Stable sort (preserves equal element order)
sort.SliceStable(people, func(i, j int) bool {
    if people[i].Age != people[j].Age {
        return people[i].Age < people[j].Age
    }
    return people[i].Name < people[j].Name
})
```

---

## 🎯 sort.Interface (Full Control)

```go
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }

people := []Person{{"Alice", 30}, {"Bob", 25}, {"Charlie", 35}}

sort.Sort(ByAge(people))
// Output: Bob, Alice, Charlie

sort.Sort(ByName(people))
sort.Sort(sort.Reverse(ByAge(people)))
```

---

## 🔍 Searching

```go
ints := []int{1, 3, 5, 7, 9, 11}
strs := []string{"apple", "banana", "cherry", "date"}

// Must be sorted first!
idx := sort.SearchInts(ints, 5)
// Output: 2

idx = sort.SearchInts(ints, 6)
// Output: 3 (insertion point, 6 not found)

idx = sort.SearchStrings(strs, "banana")
// Output: 1

// Generic search
target := 7
idx = sort.Search(len(ints), func(i int) bool {
    return ints[i] >= target
})
if idx < len(ints) && ints[idx] == target {
    // Found
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
