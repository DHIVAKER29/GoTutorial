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

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  SELECT = "WHICHEVER CHANNEL IS READY FIRST"                    │
│                                                                 │
│          ┌─── Channel A ───┐                                    │
│          │     (ready!)    │ ←─┐                                │
│          └─────────────────┘   │                                │
│                                │                                │
│  select ─┼─── Channel B ───┼───┼──► Process whichever           │
│          │    (waiting)    │   │     is ready first!            │
│          └─────────────────┘   │                                │
│                                │                                │
│          ┌─── Channel C ───┐   │                                │
│          │    (waiting)    │ ──┘                                │
│          └─────────────────┘                                    │
│                                                                 │
│  Like a waiter taking the first order that's ready              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Basic Select

```go
// select_basics.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           SELECT BASICS                                   ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    // Goroutines sending to channels
    go func() {
        time.Sleep(100 * time.Millisecond)
        ch1 <- "from channel 1"
    }()
    
    go func() {
        time.Sleep(50 * time.Millisecond)
        ch2 <- "from channel 2"
    }()
    
    // Select waits on multiple channels
    fmt.Println("\n📊 Basic Select (first one wins):")
    select {
    case msg1 := <-ch1:
        fmt.Printf("   Received: %s\n", msg1)
    case msg2 := <-ch2:
        fmt.Printf("   Received: %s\n", msg2)
    }
    
    // Multiple receives with select in loop
    fmt.Println("\n📊 Select in Loop (receive both):")
    ch1 = make(chan string)
    ch2 = make(chan string)
    
    go func() {
        time.Sleep(100 * time.Millisecond)
        ch1 <- "message 1"
    }()
    
    go func() {
        time.Sleep(50 * time.Millisecond)
        ch2 <- "message 2"
    }()
    
    for i := 0; i < 2; i++ {
        select {
        case msg := <-ch1:
            fmt.Printf("   ch1: %s\n", msg)
        case msg := <-ch2:
            fmt.Printf("   ch2: %s\n", msg)
        }
    }
}
```

**Output:**
```
(goroutine output order may vary)
╔══════════════════════════════════════════════════════════╗
║           SELECT BASICS                                   ║
╚══════════════════════════════════════════════════════════╝

📊 Basic Select (first one wins):
   Received: from channel 2

📊 Select in Loop (receive both):
   ch2: message 2
   ch1: message 1
```

---

## ⏱️ Timeouts with Select

```go
// select_timeout.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           TIMEOUTS WITH SELECT                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Timeout pattern
    fmt.Println("\n📊 Timeout Pattern:")
    ch := make(chan string)
    
    go func() {
        time.Sleep(200 * time.Millisecond)
        ch <- "slow response"
    }()
    
    select {
    case msg := <-ch:
        fmt.Printf("   Received: %s\n", msg)
    case <-time.After(100 * time.Millisecond):
        fmt.Println("   Timeout! Took too long")
    }
    
    // Successful before timeout
    fmt.Println("\n📊 Response Before Timeout:")
    ch = make(chan string)
    
    go func() {
        time.Sleep(50 * time.Millisecond)
        ch <- "fast response"
    }()
    
    select {
    case msg := <-ch:
        fmt.Printf("   Received: %s\n", msg)
    case <-time.After(100 * time.Millisecond):
        fmt.Println("   Timeout!")
    }
    
    // Production timeout with error
    fmt.Println("\n📊 Production: Fetch with Timeout")
    result, err := fetchWithTimeout(50 * time.Millisecond)
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   Result: %s\n", result)
    }
}

func fetchWithTimeout(timeout time.Duration) (string, error) {
    ch := make(chan string)
    
    go func() {
        // Simulate slow operation
        time.Sleep(100 * time.Millisecond)
        ch <- "data from server"
    }()
    
    select {
    case result := <-ch:
        return result, nil
    case <-time.After(timeout):
        return "", fmt.Errorf("operation timed out after %v", timeout)
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           TIMEOUTS WITH SELECT                            ║
╚══════════════════════════════════════════════════════════╝

📊 Timeout Pattern:
   Timeout! Took too long

📊 Response Before Timeout:
   Received: fast response

📊 Production: Fetch with Timeout
   Error: operation timed out after 50ms
```

---

## 🚫 Non-Blocking with Default

```go
// select_default.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           NON-BLOCKING WITH DEFAULT                       ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Non-blocking receive
    fmt.Println("\n📊 Non-Blocking Receive:")
    ch := make(chan int)
    
    select {
    case val := <-ch:
        fmt.Printf("   Received: %d\n", val)
    default:
        fmt.Println("   No value ready (would have blocked)")
    }
    
    // Non-blocking send
    fmt.Println("\n📊 Non-Blocking Send:")
    ch = make(chan int, 1)
    ch <- 1  // Fill the buffer
    
    select {
    case ch <- 2:
        fmt.Println("   Sent successfully")
    default:
        fmt.Println("   Channel full, send would block")
    }
    
    // Polling pattern
    fmt.Println("\n📊 Polling Pattern (try until success):")
    ch = make(chan int, 1)
    
    // Try to receive 3 times
    for i := 0; i < 3; i++ {
        select {
        case val := <-ch:
            fmt.Printf("   Received: %d\n", val)
        default:
            fmt.Printf("   Attempt %d: no value yet\n", i+1)
            // In a real scenario, you might do other work here
            ch <- i + 10  // Simulate producer
        }
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           NON-BLOCKING WITH DEFAULT                       ║
╚══════════════════════════════════════════════════════════╝

📊 Non-Blocking Receive:
   No value ready (would have blocked)

📊 Non-Blocking Send:
   Channel full, send would block

📊 Polling Pattern (try until success):
   Attempt 1: no value yet
   Received: 10
   Attempt 3: no value yet
```

---

## 🏭 Production Patterns

```go
// select_production.go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION SELECT PATTERNS                      ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: Graceful shutdown with done channel
    fmt.Println("\n📊 Pattern 1: Graceful Shutdown")
    done := make(chan struct{})
    go worker(done)
    
    time.Sleep(150 * time.Millisecond)
    close(done)  // Signal shutdown
    time.Sleep(50 * time.Millisecond)
    
    // Pattern 2: Context cancellation
    fmt.Println("\n📊 Pattern 2: Context Cancellation")
    ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel()
    
    go workerWithContext(ctx)
    <-ctx.Done()
    time.Sleep(50 * time.Millisecond)
    
    // Pattern 3: Ticker with shutdown
    fmt.Println("\n📊 Pattern 3: Ticker with Shutdown")
    tickerDemo()
    
    // Pattern 4: Multiple event sources
    fmt.Println("\n📊 Pattern 4: Multiple Event Sources")
    multipleEvents()
}

func worker(done <-chan struct{}) {
    ticker := time.NewTicker(50 * time.Millisecond)
    defer ticker.Stop()
    
    for {
        select {
        case <-done:
            fmt.Println("   worker: shutting down gracefully")
            return
        case t := <-ticker.C:
            fmt.Printf("   worker: tick at %v\n", t.Format("15:04:05.000"))
        }
    }
}

func workerWithContext(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("   worker: stopped (%v)\n", ctx.Err())
            return
        default:
            fmt.Println("   worker: working...")
            time.Sleep(30 * time.Millisecond)
        }
    }
}

func tickerDemo() {
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
            fmt.Println("   ticker: stopped")
            return
        case t := <-ticker.C:
            fmt.Printf("   tick: %v\n", t.Format("15:04:05.000"))
        }
    }
}

func multipleEvents() {
    userInput := make(chan string)
    networkData := make(chan string)
    done := make(chan struct{})
    
    // Simulate events
    go func() {
        time.Sleep(50 * time.Millisecond)
        userInput <- "button clicked"
    }()
    go func() {
        time.Sleep(80 * time.Millisecond)
        networkData <- "data received"
    }()
    go func() {
        time.Sleep(120 * time.Millisecond)
        close(done)
    }()
    
    for {
        select {
        case input := <-userInput:
            fmt.Printf("   User event: %s\n", input)
        case data := <-networkData:
            fmt.Printf("   Network event: %s\n", data)
        case <-done:
            fmt.Println("   Event loop ended")
            return
        }
    }
}
```

**Output:**
```
(goroutine output order may vary)
╔══════════════════════════════════════════════════════════╗
║           PRODUCTION SELECT PATTERNS                      ║
╚══════════════════════════════════════════════════════════╝

📊 Pattern 1: Graceful Shutdown
   worker: tick at 15:04:05.123
   worker: tick at 15:04:05.173
   worker: tick at 15:04:05.223
   worker: shutting down gracefully

📊 Pattern 2: Context Cancellation
   worker: working...
   worker: working...
   worker: working...
   worker: stopped (context deadline exceeded)

📊 Pattern 3: Ticker with Shutdown
   tick: 15:04:05.123
   tick: 15:04:05.163
   tick: 15:04:05.203
   ticker: stopped

📊 Pattern 4: Multiple Event Sources
   User event: button clicked
   Network event: data received
   Event loop ended
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

