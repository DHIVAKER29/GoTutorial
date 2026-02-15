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

- **Channel = pipe / conveyor belt** between producer and consumer goroutines
- **Properties:**
  - Type-safe: `chan int` only carries ints
  - Blocking: send waits for receiver (unbuffered)
  - Thread-safe: no locks needed
  - Synchronization built-in
- **Unbuffered (default):** Synchronous handoff — sender blocks until receiver ready
- **Buffered (capacity N):** Can hold N items — sender blocks only when buffer full

---

## 📝 Channel Basics

### Creating Channels

```go
ch := make(chan int)           // Unbuffered
bufferedCh := make(chan string, 3)  // Buffered, capacity 3
// Output: ch is chan int, bufferedCh has cap=3
```

### Basic Send and Receive

```go
ch := make(chan int)
go func() { ch <- 42 }()
value := <-ch  // Blocks until data available
// Output: value is 42
```

### Range Over Channel

```go
ch := make(chan int)
go func() {
    for i := 1; i <= 3; i++ {
        ch <- i * 10
    }
    close(ch)
}()
for val := range ch {
    fmt.Println(val)
}
// Output: 10, 20, 30
```

### Buffered Channel

```go
ch := make(chan string, 3)
ch <- "first"
ch <- "second"
ch <- "third"
// No blocking until buffer full
fmt.Println(<-ch, <-ch, <-ch)
// Output: first second third
```

### Complete Example: Channel Basics

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int)
    bufferedCh := make(chan string, 3)

    go func() {
        ch <- 42
    }()
    fmt.Println("Received:", <-ch)

    go func() {
        for i := 1; i <= 3; i++ {
            ch <- i * 10
            time.Sleep(50 * time.Millisecond)
        }
        close(ch)
    }()
    for val := range ch {
        fmt.Println("Received:", val)
    }

    bufferedCh <- "a"
    bufferedCh <- "b"
    bufferedCh <- "c"
    fmt.Println(<-bufferedCh, <-bufferedCh, <-bufferedCh)
}
```

**Output:**
```
Received: 42
Received: 10
Received: 20
Received: 30
a b c
```

---

## 📤📥 Send and Receive Operations

### Receive with OK (Check if Closed)

```go
ch := make(chan int)
go func() {
    ch <- 42
    close(ch)
}()
val, ok := <-ch
// ok is true when channel open, false when closed
// Output: val=42, ok=true
```

### Channel Direction in Signatures

| Syntax | Meaning |
|--------|---------|
| `chan int` | Bidirectional |
| `chan<- int` | Send only |
| `<-chan int` | Receive only |

### Complete Example: Channel Operations

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int, 2)
    ch <- 10
    ch <- 20
    fmt.Println(<-ch, <-ch)

    ch2 := make(chan int)
    go func() {
        ch2 <- 42
        close(ch2)
    }()
    time.Sleep(10 * time.Millisecond)
    if val, ok := <-ch2; ok {
        fmt.Println("Received:", val)
    }
    if _, ok := <-ch2; !ok {
        fmt.Println("Channel closed")
    }
}
```

**Output:**
```
10 20
Received: 42
Channel closed
```

---

## 🔒 Closing Channels

### Reading from Closed Channel

```go
ch := make(chan int)
go func() {
    ch <- 1
    ch <- 2
    close(ch)
}()
for val, ok := <-ch; ok; val, ok = <-ch {
    fmt.Println(val)
}
// Output: 1, 2, then loop exits when ok=false
```

### Channel Closing Rules

| Rule | Description |
|------|-------------|
| Only sender closes | Receiver should not close |
| Send to closed | **PANIC** |
| Receive from closed | Zero value, `ok=false` |
| Close already closed | **PANIC** |

### Complete Example: Closing Channels

```go
package main

import "fmt"

func main() {
    ch := make(chan int)
    go func() {
        for i := 1; i <= 3; i++ {
            ch <- i
        }
        close(ch)
    }()
    for {
        val, ok := <-ch
        if !ok {
            fmt.Println("Channel closed!")
            break
        }
        fmt.Println("Received:", val)
    }
}
```

**Output:**
```
Received: 1
Received: 2
Received: 3
Channel closed!
```

---

## 🔀 Channel Patterns

### Done Channel (Signal Completion)

```go
done := make(chan struct{})
go func() {
    // ... work ...
    close(done)
}()
<-done  // Wait for signal
```

### Producer-Consumer

```go
jobs := make(chan int, 5)
results := make(chan int, 5)
go func() {
    for i := 1; i <= 5; i++ {
        jobs <- i
    }
    close(jobs)
}()
go func() {
    for job := range jobs {
        results <- job * 2
    }
    close(results)
}()
for r := range results {
    fmt.Println(r)
}
// Output: 2, 4, 6, 8, 10
```

### Complete Example: Pipeline Pattern

```go
package main

import "fmt"

func main() {
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
    for n := range sq(gen(1, 2, 3, 4, 5)) {
        fmt.Println("Squared:", n)
    }
}
```

**Output:**
```
Squared: 1
Squared: 4
Squared: 9
Squared: 16
Squared: 25
```

---

## 🆚 Unbuffered vs Buffered

| Aspect | Unbuffered `make(chan T)` | Buffered `make(chan T, N)` |
|--------|---------------------------|----------------------------|
| Behavior | Synchronous handoff | Asynchronous until full |
| Sender blocks | Until receiver ready | Only when buffer full |
| Guarantee | Message received when send completes | Message may be in buffer |
| Use when | Need synchronization | Decoupling, rate limiting |
| Default | Prefer for simplicity | Use for performance |

**Choosing:** Default to unbuffered. Use buffered for known workloads. Buffer size should have meaning.

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
