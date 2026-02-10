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

```go
// time_basics.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           TIME BASICS                                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Current time
    fmt.Println("\n📊 Current Time:")
    now := time.Now()
    fmt.Printf("   Now: %v\n", now)
    fmt.Printf("   UTC: %v\n", now.UTC())
    
    // Time components
    fmt.Println("\n📊 Time Components:")
    fmt.Printf("   Year:   %d\n", now.Year())
    fmt.Printf("   Month:  %s (%d)\n", now.Month(), now.Month())
    fmt.Printf("   Day:    %d\n", now.Day())
    fmt.Printf("   Hour:   %d\n", now.Hour())
    fmt.Printf("   Minute: %d\n", now.Minute())
    fmt.Printf("   Second: %d\n", now.Second())
    fmt.Printf("   Weekday: %s\n", now.Weekday())
    
    // Create specific time
    fmt.Println("\n📊 Create Specific Time:")
    birthday := time.Date(1990, time.January, 15, 12, 0, 0, 0, time.UTC)
    fmt.Printf("   Birthday: %v\n", birthday)
    
    // Unix timestamp
    fmt.Println("\n📊 Unix Timestamp:")
    fmt.Printf("   Unix seconds: %d\n", now.Unix())
    fmt.Printf("   Unix milliseconds: %d\n", now.UnixMilli())
    fmt.Printf("   Unix nanoseconds: %d\n", now.UnixNano())
    
    // From Unix timestamp
    fromUnix := time.Unix(1609459200, 0)  // 2021-01-01 00:00:00 UTC
    fmt.Printf("   From Unix: %v\n", fromUnix)
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           TIME BASICS                                     ║
╚══════════════════════════════════════════════════════════╝

📊 Current Time:
   Now: 2026-02-10 [current time with nanoseconds]
   UTC: 2026-02-10 [UTC time]

📊 Time Components:
   Year:   2026
   Month:  February (2)
   Day:    10
   Hour:   [current hour]
   Minute: [current minute]
   Second: [current second]
   Weekday: [current weekday]

📊 Create Specific Time:
   Birthday: 1990-01-15 12:00:00 +0000 UTC

📊 Unix Timestamp:
   Unix seconds: [current Unix timestamp]
   Unix milliseconds: [current Unix milliseconds]
   Unix nanoseconds: [current Unix nanoseconds]
   From Unix: 2021-01-01 00:00:00 +0000 UTC
```

*Note: Current time values will vary based on when the program runs.*

---

## 📅 Formatting and Parsing

```go
// time_format.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           TIME FORMATTING & PARSING                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    now := time.Now()
    
    // Go's reference time: Mon Jan 2 15:04:05 MST 2006
    // (1 2 3 4 5 6 7 - easy to remember!)
    fmt.Println("\n📊 Go's Magic Reference Time:")
    fmt.Println("   Mon Jan 2 15:04:05 MST 2006")
    fmt.Println("   (Month=1, Day=2, Hour=3PM=15, Min=4, Sec=5, Year=6)")
    
    // Formatting
    fmt.Println("\n📊 Formatting (Time → String):")
    fmt.Printf("   RFC3339:    %s\n", now.Format(time.RFC3339))
    fmt.Printf("   Kitchen:    %s\n", now.Format(time.Kitchen))
    fmt.Printf("   Date:       %s\n", now.Format("2006-01-02"))
    fmt.Printf("   DateTime:   %s\n", now.Format("2006-01-02 15:04:05"))
    fmt.Printf("   US Format:  %s\n", now.Format("01/02/2006"))
    fmt.Printf("   Full:       %s\n", now.Format("Monday, January 2, 2006"))
    fmt.Printf("   12-hour:    %s\n", now.Format("3:04 PM"))
    
    // Parsing
    fmt.Println("\n📊 Parsing (String → Time):")
    
    dateStr := "2024-12-25"
    parsed, err := time.Parse("2006-01-02", dateStr)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   Parsed %q: %v\n", dateStr, parsed)
    }
    
    // Parse with timezone
    datetimeStr := "2024-12-25 14:30:00"
    loc, _ := time.LoadLocation("America/New_York")
    parsedLocal, _ := time.ParseInLocation("2006-01-02 15:04:05", datetimeStr, loc)
    fmt.Printf("   With timezone: %v\n", parsedLocal)
    
    // Common formats
    fmt.Println("\n📊 Predefined Formats:")
    fmt.Println("   time.RFC3339     = 2006-01-02T15:04:05Z07:00")
    fmt.Println("   time.RFC1123     = Mon, 02 Jan 2006 15:04:05 MST")
    fmt.Println("   time.Kitchen     = 3:04PM")
    fmt.Println("   time.UnixDate    = Mon Jan _2 15:04:05 MST 2006")
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           TIME FORMATTING & PARSING                       ║
╚══════════════════════════════════════════════════════════╝

📊 Go's Magic Reference Time:
   Mon Jan 2 15:04:05 MST 2006
   (Month=1, Day=2, Hour=3PM=15, Min=4, Sec=5, Year=6)

📊 Formatting (Time → String):
   RFC3339:    2026-02-10T[time]Z
   Kitchen:    [time in 12-hour format]
   Date:       2026-02-10
   DateTime:   2026-02-10 [time]
   US Format:  02/10/2026
   Full:       [day], [month] [day], [year]
   12-hour:    [time in 12-hour format]

📊 Parsing (String → Time):
   Parsed "2024-12-25": 2024-12-25 00:00:00 +0000 UTC
   With timezone: 2024-12-25 14:30:00 -0500 EST

📊 Predefined Formats:
   time.RFC3339     = 2006-01-02T15:04:05Z07:00
   time.RFC1123     = Mon, 02 Jan 2006 15:04:05 MST
   time.Kitchen     = 3:04PM
   time.UnixDate    = Mon Jan _2 15:04:05 MST 2006
```

*Note: Time formatting output will vary based on when the program runs.*

---

## ⏱️ Duration and Arithmetic

```go
// time_duration.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           DURATION & ARITHMETIC                           ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Duration constants
    fmt.Println("\n📊 Duration Constants:")
    fmt.Printf("   Nanosecond:  %v\n", time.Nanosecond)
    fmt.Printf("   Microsecond: %v\n", time.Microsecond)
    fmt.Printf("   Millisecond: %v\n", time.Millisecond)
    fmt.Printf("   Second:      %v\n", time.Second)
    fmt.Printf("   Minute:      %v\n", time.Minute)
    fmt.Printf("   Hour:        %v\n", time.Hour)
    
    // Creating durations
    fmt.Println("\n📊 Creating Durations:")
    d1 := 5 * time.Second
    d2 := 2*time.Hour + 30*time.Minute
    d3, _ := time.ParseDuration("1h30m45s")
    
    fmt.Printf("   5 seconds: %v\n", d1)
    fmt.Printf("   2h30m: %v\n", d2)
    fmt.Printf("   Parsed: %v\n", d3)
    
    // Time arithmetic
    fmt.Println("\n📊 Time Arithmetic:")
    now := time.Now()
    
    // Add
    tomorrow := now.Add(24 * time.Hour)
    fmt.Printf("   Tomorrow: %v\n", tomorrow.Format("2006-01-02"))
    
    nextWeek := now.AddDate(0, 0, 7)  // years, months, days
    fmt.Printf("   Next week: %v\n", nextWeek.Format("2006-01-02"))
    
    nextMonth := now.AddDate(0, 1, 0)
    fmt.Printf("   Next month: %v\n", nextMonth.Format("2006-01-02"))
    
    // Subtract (difference)
    fmt.Println("\n📊 Time Difference:")
    past := time.Date(2020, 1, 1, 0, 0, 0, 0, time.UTC)
    diff := now.Sub(past)
    fmt.Printf("   Since 2020-01-01: %v\n", diff)
    fmt.Printf("   In hours: %.0f\n", diff.Hours())
    fmt.Printf("   In days: %.0f\n", diff.Hours()/24)
    
    // Comparison
    fmt.Println("\n📊 Time Comparison:")
    t1 := time.Now()
    t2 := t1.Add(1 * time.Hour)
    
    fmt.Printf("   t1.Before(t2): %t\n", t1.Before(t2))
    fmt.Printf("   t1.After(t2):  %t\n", t1.After(t2))
    fmt.Printf("   t1.Equal(t1):  %t\n", t1.Equal(t1))
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           DURATION & ARITHMETIC                           ║
╚══════════════════════════════════════════════════════════╝

📊 Duration Constants:
   Nanosecond:  1ns
   Microsecond: 1µs
   Millisecond: 1ms
   Second:      1s
   Minute:      1m0s
   Hour:        1h0m0s

📊 Creating Durations:
   5 seconds: 5s
   2h30m: 2h30m0s
   Parsed: 1h30m45s

📊 Time Arithmetic:
   Tomorrow: [tomorrow's date]
   Next week: [date 7 days from now]
   Next month: [date 1 month from now]

📊 Time Difference:
   Since 2020-01-01: [duration since 2020-01-01]
   In hours: [number of hours]
   In days: [number of days]

📊 Time Comparison:
   t1.Before(t2): true
   t1.After(t2):  false
   t1.Equal(t1):  true
```

*Note: Time arithmetic and difference values will vary based on when the program runs.*

---

## ⏲️ Timers and Tickers

```go
// timers_tickers.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           TIMERS & TICKERS                                ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Sleep
    fmt.Println("\n📊 Sleep:")
    fmt.Println("   Sleeping for 100ms...")
    time.Sleep(100 * time.Millisecond)
    fmt.Println("   Awake!")
    
    // Timer (one-shot)
    fmt.Println("\n📊 Timer (one-shot):")
    timer := time.NewTimer(200 * time.Millisecond)
    fmt.Println("   Timer started, waiting...")
    <-timer.C  // Block until timer fires
    fmt.Println("   Timer fired!")
    
    // AfterFunc (callback)
    fmt.Println("\n📊 AfterFunc (callback):")
    done := make(chan bool)
    time.AfterFunc(100*time.Millisecond, func() {
        fmt.Println("   Callback executed!")
        done <- true
    })
    <-done
    
    // Ticker (repeating)
    fmt.Println("\n📊 Ticker (repeating):")
    ticker := time.NewTicker(50 * time.Millisecond)
    count := 0
    
    for t := range ticker.C {
        count++
        fmt.Printf("   Tick %d at %v\n", count, t.Format("15:04:05.000"))
        if count >= 3 {
            ticker.Stop()
            break
        }
    }
    
    // time.After (convenience)
    fmt.Println("\n📊 time.After (convenience):")
    select {
    case <-time.After(100 * time.Millisecond):
        fmt.Println("   time.After fired!")
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           TIMERS & TICKERS                                ║
╚══════════════════════════════════════════════════════════╝

📊 Sleep:
   Sleeping for 100ms...
   Awake!

📊 Timer (one-shot):
   Timer started, waiting...
   Timer fired!

📊 AfterFunc (callback):
   Callback executed!

📊 Ticker (repeating):
   Tick 1 at [time]
   Tick 2 at [time]
   Tick 3 at [time]

📊 time.After (convenience):
   time.After fired!
```

*Note: Timestamps in ticker output will show actual times when ticks occur.*

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

