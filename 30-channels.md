# 30 - Channels

> Go's way of communication between goroutines - "Don't communicate by sharing memory; share memory by communicating."

---

## 📌 What You'll Learn

- What channels are and why they exist
- Creating and using channels
- Buffered vs unbuffered channels
- Channel operations: send, receive, close
- Channel patterns and idioms

---

## 🤔 What is a Channel?

### Definition

> A **channel** is a typed conduit through which you can send and receive values between goroutines, providing synchronization and communication.

### Real-World Analogy

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CHANNEL = PIPE / CONVEYOR BELT                                 │
│                                                                 │
│  Goroutine A              Channel               Goroutine B     │
│  (Producer)                                     (Consumer)      │
│                                                                 │
│  ┌─────────┐       ┌─────────────────┐        ┌─────────┐      │
│  │ Creates │  ───► │  ══════════════ │  ───► │ Receives│      │
│  │  data   │  send │  ══════════════ │  recv │  data   │      │
│  └─────────┘       └─────────────────┘        └─────────┘      │
│                                                                 │
│  Properties:                                                    │
│  • Type-safe: chan int only carries ints                        │
│  • Blocking: send waits for receiver (unbuffered)               │
│  • Thread-safe: no locks needed!                                │
│  • Synchronization built-in                                     │
│                                                                 │
│  UNBUFFERED (default): Synchronous handoff                      │
│  ┌─────┐     ┌─────┐                                           │
│  │ --> │ --> │ --> │  Sender blocks until receiver ready        │
│  └─────┘     └─────┘                                           │
│                                                                 │
│  BUFFERED (capacity N): Can hold N items                        │
│  ┌─────┬─────┬─────┐                                           │
│  │  1  │  2  │  3  │  Sender blocks only when buffer full      │
│  └─────┴─────┴─────┘                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Channel Basics

```go
// channel_basics.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CHANNEL BASICS                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Creating channels
    fmt.Println("\n📊 Creating Channels:")
    ch := make(chan int)          // Unbuffered channel
    bufferedCh := make(chan string, 3)  // Buffered with capacity 3
    
    fmt.Printf("   ch: %T\n", ch)
    fmt.Printf("   bufferedCh: %T, cap=%d\n", bufferedCh, cap(bufferedCh))
    
    // Basic send and receive
    fmt.Println("\n📊 Basic Send and Receive:")
    go func() {
        ch <- 42  // Send
        fmt.Println("   goroutine: sent 42")
    }()
    
    value := <-ch  // Receive (blocks until data available)
    fmt.Printf("   main: received %d\n", value)
    
    // Multiple values
    fmt.Println("\n📊 Multiple Values:")
    go func() {
        for i := 1; i <= 3; i++ {
            ch <- i * 10
            time.Sleep(50 * time.Millisecond)
        }
        close(ch)  // Signal no more values
    }()
    
    for val := range ch {  // Receive until closed
        fmt.Printf("   Received: %d\n", val)
    }
    
    // Buffered channel
    fmt.Println("\n📊 Buffered Channel (no blocking until full):")
    bufferedCh <- "first"
    bufferedCh <- "second"
    bufferedCh <- "third"
    // bufferedCh <- "fourth"  // Would block! Buffer full
    
    fmt.Printf("   Received: %s\n", <-bufferedCh)
    fmt.Printf("   Received: %s\n", <-bufferedCh)
    fmt.Printf("   Received: %s\n", <-bufferedCh)
}
```

---

## 📤📥 Send and Receive Operations

```go
// channel_operations.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CHANNEL OPERATIONS                              ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    ch := make(chan int, 2)
    
    // Send operation: ch <- value
    fmt.Println("\n📊 Send Operation (ch <- value):")
    ch <- 10
    ch <- 20
    fmt.Println("   Sent 10 and 20")
    
    // Receive operation: value := <-ch
    fmt.Println("\n📊 Receive Operation (value := <-ch):")
    v1 := <-ch
    v2 := <-ch
    fmt.Printf("   Received: %d, %d\n", v1, v2)
    
    // Receive with ok (check if channel closed)
    fmt.Println("\n📊 Receive with OK (check if closed):")
    ch2 := make(chan int)
    go func() {
        ch2 <- 42
        close(ch2)
    }()
    
    time.Sleep(10 * time.Millisecond)
    
    if val, ok := <-ch2; ok {
        fmt.Printf("   Received: %d (channel open)\n", val)
    }
    
    if val, ok := <-ch2; !ok {
        fmt.Printf("   Received: %d (channel CLOSED)\n", val)
    }
    
    // Range over channel
    fmt.Println("\n📊 Range Over Channel:")
    ch3 := make(chan int)
    go func() {
        for i := 1; i <= 5; i++ {
            ch3 <- i
        }
        close(ch3)  // MUST close for range to terminate!
    }()
    
    fmt.Print("   Values: ")
    for v := range ch3 {
        fmt.Printf("%d ", v)
    }
    fmt.Println("(channel closed, range exited)")
    
    // Channel direction in function signatures
    fmt.Println("\n📊 Channel Direction (restrict access):")
    fmt.Println("   chan int     - bidirectional")
    fmt.Println("   chan<- int   - send only")
    fmt.Println("   <-chan int   - receive only")
}
```

---

## 🔒 Closing Channels

```go
// channel_close.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CLOSING CHANNELS                                ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    ch := make(chan int)
    
    go func() {
        for i := 1; i <= 3; i++ {
            ch <- i
        }
        close(ch)  // Signal: no more values
    }()
    
    // Reading from closed channel
    fmt.Println("\n📊 Reading from Closed Channel:")
    for {
        val, ok := <-ch
        if !ok {
            fmt.Println("   Channel closed!")
            break
        }
        fmt.Printf("   Received: %d\n", val)
    }
    
    // Important rules
    fmt.Println("\n⚠️ Channel Closing Rules:")
    fmt.Println("   • Only SENDER should close")
    fmt.Println("   • Sending to closed channel = PANIC")
    fmt.Println("   • Receiving from closed = zero value, ok=false")
    fmt.Println("   • Closing already closed = PANIC")
    fmt.Println("   • Don't need to close if garbage collected")
    
    // Receiving from closed channel returns zero value
    fmt.Println("\n📊 Closed Channel Returns Zero Values:")
    closedCh := make(chan int)
    close(closedCh)
    
    fmt.Printf("   <-closedCh = %d (zero value)\n", <-closedCh)
    fmt.Printf("   <-closedCh = %d (still works, zero value)\n", <-closedCh)
}
```

---

## 🔀 Channel Patterns

```go
// channel_patterns.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           CHANNEL PATTERNS                                ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Pattern 1: Done channel (signal completion)
    fmt.Println("\n📊 Pattern 1: Done Channel")
    done := make(chan struct{})
    go func() {
        fmt.Println("   worker: working...")
        time.Sleep(100 * time.Millisecond)
        fmt.Println("   worker: done!")
        close(done)  // Signal completion
    }()
    <-done  // Wait for signal
    fmt.Println("   main: worker finished")
    
    // Pattern 2: Producer-Consumer
    fmt.Println("\n📊 Pattern 2: Producer-Consumer")
    jobs := make(chan int, 5)
    results := make(chan int, 5)
    
    // Producer
    go func() {
        for i := 1; i <= 5; i++ {
            jobs <- i
        }
        close(jobs)
    }()
    
    // Consumer
    go func() {
        for job := range jobs {
            results <- job * 2  // Process
        }
        close(results)
    }()
    
    for result := range results {
        fmt.Printf("   Result: %d\n", result)
    }
    
    // Pattern 3: Fan-out, Fan-in
    fmt.Println("\n📊 Pattern 3: Fan-out, Fan-in")
    fanOutIn()
    
    // Pattern 4: Pipeline
    fmt.Println("\n📊 Pattern 4: Pipeline")
    pipeline()
}

func fanOutIn() {
    input := make(chan int)
    output := make(chan int)
    
    // Fan-out: 3 workers
    for w := 0; w < 3; w++ {
        go func(id int) {
            for n := range input {
                output <- n * 2
            }
        }(w)
    }
    
    // Send work
    go func() {
        for i := 1; i <= 6; i++ {
            input <- i
        }
        close(input)
    }()
    
    // Fan-in: collect results
    for i := 0; i < 6; i++ {
        fmt.Printf("   Result: %d\n", <-output)
    }
}

func pipeline() {
    // Stage 1: Generate numbers
    gen := func(nums ...int) <-chan int {
        out := make(chan int)
        go func() {
            for _, n := range nums {
                out <- n
            }
            close(out)
        }()
        return out
    }
    
    // Stage 2: Square numbers
    sq := func(in <-chan int) <-chan int {
        out := make(chan int)
        go func() {
            for n := range in {
                out <- n * n
            }
            close(out)
        }()
        return out
    }
    
    // Pipeline: gen -> sq -> print
    for n := range sq(gen(1, 2, 3, 4, 5)) {
        fmt.Printf("   Squared: %d\n", n)
    }
}
```

---

## 🆚 Unbuffered vs Buffered

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  UNBUFFERED CHANNEL (make(chan T))                              │
│                                                                 │
│  • Synchronous: sender blocks until receiver ready              │
│  • Guarantees: message received when send completes             │
│  • Use when: you need synchronization                           │
│                                                                 │
│  BUFFERED CHANNEL (make(chan T, N))                             │
│                                                                 │
│  • Asynchronous: sender blocks only when buffer full            │
│  • No guarantee: message might be in buffer, not received       │
│  • Use when: you need decoupling, rate limiting                 │
│                                                                 │
│  CHOOSING:                                                      │
│  • Default to unbuffered (simpler, safer)                       │
│  • Use buffered for performance or known workloads              │
│  • Buffer size should have meaning (not arbitrary!)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Channels** are typed pipes for goroutine communication
2. **make(chan T)** creates unbuffered (synchronous)
3. **make(chan T, N)** creates buffered (async until full)
4. **<-ch** receives, **ch <- v** sends
5. **close(ch)** signals no more values
6. **Only sender closes** - receiver should not close
7. **range ch** receives until closed
8. **val, ok := <-ch** checks if closed

---

## ➡️ Next Steps

**Next Topic:** [31 - Select](./31-select.md)

