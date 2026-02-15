# 39 - Time and Date

> Working with time, durations, and timezones in Go.

---

## 📌 What You'll Learn

- Creating and formatting time
- Parsing time strings
- Duration and time arithmetic
- Timers and tickers
- Timezones

---

## ⏰ Time Basics

**Current time:**
```go
now := time.Now()
fmt.Println(now)
fmt.Println(now.UTC())
// Output: 2026-02-15 [current time]
```

**Time components:**
```go
now.Year()
now.Month()
now.Day()
now.Hour()
now.Weekday()
```

**Create specific time:**
```go
birthday := time.Date(1990, time.January, 15, 12, 0, 0, 0, time.UTC)
```

**Unix timestamp:**
```go
now.Unix()       // Seconds
now.UnixMilli()  // Milliseconds
time.Unix(1609459200, 0)  // From Unix to time.Time
```

---

## 📅 Formatting and Parsing

**Go's reference time:** `Mon Jan 2 15:04:05 MST 2006`  
(Month=1, Day=2, Hour=15, Min=4, Sec=5, Year=6)

**Formatting (Time → String):**
```go
now.Format(time.RFC3339)     // 2006-01-02T15:04:05Z07:00
now.Format("2006-01-02")     // Date only
now.Format("2006-01-02 15:04:05")
now.Format(time.Kitchen)     // 3:04PM
```

**Parsing (String → Time):**
```go
parsed, err := time.Parse("2006-01-02", "2024-12-25")
```

**Parse with timezone:**
```go
loc, _ := time.LoadLocation("America/New_York")
parsed, _ := time.ParseInLocation("2006-01-02 15:04:05", "2024-12-25 14:30:00", loc)
```

### Complete Example: Time Formatting

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Printf("RFC3339: %s\n", now.Format(time.RFC3339))
    fmt.Printf("Date:    %s\n", now.Format("2006-01-02"))
    fmt.Printf("Kitchen: %s\n", now.Format(time.Kitchen))

    parsed, _ := time.Parse("2006-01-02", "2024-12-25")
    fmt.Printf("Parsed:  %v\n", parsed)
}
```

**Output:**
```
RFC3339: 2026-02-15T[time]Z
Date:    2026-02-15
Kitchen: [12-hour time]
Parsed:  2024-12-25 00:00:00 +0000 UTC
```

---

## ⏱️ Duration and Arithmetic

**Duration constants:**
```go
time.Nanosecond
time.Microsecond
time.Millisecond
time.Second
time.Minute
time.Hour
```

**Creating durations:**
```go
d1 := 5 * time.Second
d2 := 2*time.Hour + 30*time.Minute
d3, _ := time.ParseDuration("1h30m45s")
```

**Time arithmetic:**
```go
tomorrow := now.Add(24 * time.Hour)
nextWeek := now.AddDate(0, 0, 7)   // years, months, days
nextMonth := now.AddDate(0, 1, 0)
```

**Time difference:**
```go
diff := now.Sub(past)
diff.Hours()
diff.Hours() / 24  // Days
```

**Comparison:**
```go
t1.Before(t2)
t1.After(t2)
t1.Equal(t2)
```

### Complete Example: Duration and Arithmetic

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    tomorrow := now.Add(24 * time.Hour)
    fmt.Printf("Tomorrow: %s\n", tomorrow.Format("2006-01-02"))

    past := time.Date(2020, 1, 1, 0, 0, 0, 0, time.UTC)
    diff := now.Sub(past)
    fmt.Printf("Since 2020-01-01: %.0f days\n", diff.Hours()/24)

    t1 := time.Now()
    t2 := t1.Add(1 * time.Hour)
    fmt.Printf("t1.Before(t2): %t\n", t1.Before(t2))
}
```

**Output:**
```
Tomorrow: [tomorrow's date]
Since 2020-01-01: [days] days
t1.Before(t2): true
```

---

## ⏲️ Timers and Tickers

**Sleep:**
```go
time.Sleep(100 * time.Millisecond)
```

**Timer (one-shot):**
```go
timer := time.NewTimer(200 * time.Millisecond)
<-timer.C  // Block until fired
```

**AfterFunc (callback):**
```go
time.AfterFunc(100*time.Millisecond, func() {
    fmt.Println("Callback executed!")
})
```

**Ticker (repeating):**
```go
ticker := time.NewTicker(50 * time.Millisecond)
for t := range ticker.C {
    fmt.Println("Tick at", t)
    ticker.Stop()
    break
}
```

**time.After (convenience):**
```go
select {
case <-time.After(100 * time.Millisecond):
    fmt.Println("Fired!")
}
```

### Complete Example: Timers and Tickers

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("Sleeping 50ms...")
    time.Sleep(50 * time.Millisecond)
    fmt.Println("Awake!")

    timer := time.NewTimer(50 * time.Millisecond)
    <-timer.C
    fmt.Println("Timer fired!")

    ticker := time.NewTicker(30 * time.Millisecond)
    count := 0
    for range ticker.C {
        count++
        fmt.Printf("Tick %d\n", count)
        if count >= 2 {
            ticker.Stop()
            break
        }
    }
}
```

**Output:**
```
Sleeping 50ms...
Awake!
Timer fired!
Tick 1
Tick 2
```

---

## 🎯 Key Takeaways

1. **Reference time**: `Mon Jan 2 15:04:05 MST 2006`
2. **time.Now()** for current time
3. **time.Parse()** / **time.Format()** for conversion
4. **Duration** = time.Second, time.Hour, etc.
5. **Add/Sub** for time arithmetic
6. **Timer** = one-shot, **Ticker** = repeating

---

## ➡️ Next Steps

**Next Topic:** [40 - Regular Expressions](./40-regex.md)
