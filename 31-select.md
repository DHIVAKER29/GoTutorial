# 31 - Select

> Multiplexing channel operations - waiting on multiple channels simultaneously.

---

## 📌 What You'll Learn

- What select does and why it's powerful
- Select with multiple channels
- Non-blocking operations with default
- Timeouts with select
- Common patterns

---

## 🤔 What is Select?

### Definition

> **Select** lets a goroutine wait on multiple channel operations, proceeding with whichever is ready first.

- Waits on multiple channels simultaneously
- Proceeds with the first case that becomes ready
- Like a waiter taking the first order that's ready

---

## 📝 Basic Select

### First One Wins

```go
ch1 := make(chan string)
ch2 := make(chan string)
go func() { time.Sleep(100 * time.Millisecond); ch1 <- "from 1" }()
go func() { time.Sleep(50 * time.Millisecond); ch2 <- "from 2" }()
select {
case msg := <-ch1:
    fmt.Println(msg)
case msg := <-ch2:
    fmt.Println(msg)
}
// Output: from 2 (arrives first)
```

### Select in Loop

```go
ch1, ch2 := make(chan string), make(chan string)
go func() { ch1 <- "msg1" }()
go func() { ch2 <- "msg2" }()
for i := 0; i < 2; i++ {
    select {
    case m := <-ch1:
        fmt.Println("ch1:", m)
    case m := <-ch2:
        fmt.Println("ch2:", m)
    }
}
// Output: receives both (order may vary)
```

### Complete Example: Basic Select

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(100 * time.Millisecond)
        ch1 <- "from channel 1"
    }()
    go func() {
        time.Sleep(50 * time.Millisecond)
        ch2 <- "from channel 2"
    }()

    select {
    case msg := <-ch1:
        fmt.Println("Received:", msg)
    case msg := <-ch2:
        fmt.Println("Received:", msg)
    }
}
```

**Output:**
```
Received: from channel 2
```

---

## ⏱️ Timeouts with Select

### Timeout Pattern

```go
ch := make(chan string)
go func() {
    time.Sleep(200 * time.Millisecond)
    ch <- "slow response"
}()
select {
case msg := <-ch:
    fmt.Println(msg)
case <-time.After(100 * time.Millisecond):
    fmt.Println("Timeout!")
}
// Output: Timeout!
```

### Fetch with Timeout

```go
func fetchWithTimeout(timeout time.Duration) (string, error) {
    ch := make(chan string)
    go func() {
        time.Sleep(100 * time.Millisecond)
        ch <- "data"
    }()
    select {
    case result := <-ch:
        return result, nil
    case <-time.After(timeout):
        return "", fmt.Errorf("timed out after %v", timeout)
    }
}
```

### Complete Example: Timeouts

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan string)
    go func() {
        time.Sleep(200 * time.Millisecond)
        ch <- "slow response"
    }()
    select {
    case msg := <-ch:
        fmt.Println("Received:", msg)
    case <-time.After(100 * time.Millisecond):
        fmt.Println("Timeout! Took too long")
    }

    ch = make(chan string)
    go func() {
        time.Sleep(50 * time.Millisecond)
        ch <- "fast response"
    }()
    select {
    case msg := <-ch:
        fmt.Println("Received:", msg)
    case <-time.After(100 * time.Millisecond):
        fmt.Println("Timeout!")
    }
}
```

**Output:**
```
Timeout! Took too long
Received: fast response
```

---

## 🚫 Non-Blocking with Default

### Non-Blocking Receive

```go
ch := make(chan int)
select {
case val := <-ch:
    fmt.Println("Received:", val)
default:
    fmt.Println("No value ready")
}
// Output: No value ready
```

### Non-Blocking Send

```go
ch := make(chan int, 1)
ch <- 1
select {
case ch <- 2:
    fmt.Println("Sent")
default:
    fmt.Println("Channel full")
}
// Output: Channel full
```

### Complete Example: Non-Blocking Select

```go
package main

import "fmt"

func main() {
    ch := make(chan int)
    select {
    case val := <-ch:
        fmt.Println("Received:", val)
    default:
        fmt.Println("No value ready (would have blocked)")
    }

    ch = make(chan int, 1)
    ch <- 1
    select {
    case ch <- 2:
        fmt.Println("Sent successfully")
    default:
        fmt.Println("Channel full, send would block")
    }
}
```

**Output:**
```
No value ready (would have blocked)
Channel full, send would block
```

---

## 🏭 Production Patterns

### Graceful Shutdown with Done Channel

```go
done := make(chan struct{})
go worker(done)
time.Sleep(150 * time.Millisecond)
close(done)
```

### Ticker with Shutdown

```go
done := make(chan struct{})
ticker := time.NewTicker(40 * time.Millisecond)
go func() {
    time.Sleep(150 * time.Millisecond)
    close(done)
}()
for {
    select {
    case <-done:
        ticker.Stop()
        return
    case t := <-ticker.C:
        fmt.Println("Tick:", t)
    }
}
```

### Complete Example: Graceful Shutdown

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    done := make(chan struct{})
    go worker(done)

    time.Sleep(150 * time.Millisecond)
    close(done)
    time.Sleep(50 * time.Millisecond)
}

func worker(done <-chan struct{}) {
    ticker := time.NewTicker(50 * time.Millisecond)
    defer ticker.Stop()

    for {
        select {
        case <-done:
            fmt.Println("worker: shutting down gracefully")
            return
        case t := <-ticker.C:
            fmt.Println("worker: tick at", t.Format("15:04:05.000"))
        }
    }
}
```

**Output:**
```
worker: tick at ...
worker: tick at ...
worker: shutting down gracefully
```

---

## 🎯 Key Takeaways

1. **select** waits on multiple channels
2. **First ready wins** - random if multiple ready
3. **default** makes select non-blocking
4. **time.After** for timeouts
5. **Ticker** for periodic operations
6. **Context** for cancellation (production)
7. **select {}** blocks forever (keep main alive)

---

## ➡️ Next Steps

**Next Topic:** [32 - Sync Package](./32-sync-package.md)
