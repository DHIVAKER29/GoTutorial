# 57 - Concurrency Patterns

> Common concurrency patterns used in production Go applications.

---

## 📌 What You'll Learn

- Worker Pool pattern
- Fan-out / Fan-in pattern
- Pipeline pattern
- Semaphore pattern
- errgroup for error handling

---

## 👷 Worker Pool Pattern

```go
// worker_pool.go
package main

import (
    "fmt"
    "sync"
    "time"
)

/*
WORKER POOL PATTERN

┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌───────┐                                                      │
│  │ Job 1 │──┐                                                   │
│  ├───────┤  │    ┌──────────┐   ┌──────────┐                   │
│  │ Job 2 │──┼──► │ Worker 1 │──►│          │                   │
│  ├───────┤  │    ├──────────┤   │ Results  │                   │
│  │ Job 3 │──┼──► │ Worker 2 │──►│ Channel  │                   │
│  ├───────┤  │    ├──────────┤   │          │                   │
│  │ Job 4 │──┼──► │ Worker 3 │──►│          │                   │
│  └───────┘  │    └──────────┘   └──────────┘                   │
│  Jobs Chan  │    (Fixed # of workers)                          │
│                                                                 │
│  WHY?                                                           │
│  • Limit concurrency (don't spawn 10000 goroutines)             │
│  • Control resource usage (DB connections, API rate limits)     │
│  • Backpressure handling                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
*/

type Job struct {
    ID   int
    Data string
}

type Result struct {
    JobID  int
    Output string
}

func worker(id int, jobs <-chan Job, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()
    for job := range jobs {
        // Simulate work
        time.Sleep(100 * time.Millisecond)
        results <- Result{
            JobID:  job.ID,
            Output: fmt.Sprintf("Worker %d processed: %s", id, job.Data),
        }
    }
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Worker Pool Pattern                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    const numJobs = 10
    const numWorkers = 3
    
    jobs := make(chan Job, numJobs)
    results := make(chan Result, numJobs)
    
    // Start workers
    var wg sync.WaitGroup
    for w := 1; w <= numWorkers; w++ {
        wg.Add(1)
        go worker(w, jobs, results, &wg)
    }
    
    // Send jobs
    for j := 1; j <= numJobs; j++ {
        jobs <- Job{ID: j, Data: fmt.Sprintf("Job-%d", j)}
    }
    close(jobs)
    
    // Wait for workers and close results
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // Collect results
    fmt.Println("\n📊 Results:")
    for result := range results {
        fmt.Printf("   %s\n", result.Output)
    }
}
```

---

## 📤📥 Fan-Out / Fan-In Pattern

```go
// fan_out_fan_in.go
package main

import (
    "fmt"
    "sync"
    "time"
)

/*
FAN-OUT: One source → Multiple processors
FAN-IN:  Multiple sources → One destination

┌────────┐                                    ┌────────┐
│        │──► Worker 1 ──┐                    │        │
│ Source │──► Worker 2 ──┼──► Merged Output ─►│ Sink   │
│        │──► Worker 3 ──┘                    │        │
└────────┘                                    └────────┘
   Fan-Out                 Fan-In
*/

func source(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            time.Sleep(50 * time.Millisecond)  // Simulate work
            out <- n * n
        }
        close(out)
    }()
    return out
}

func fanIn(channels ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    
    // Start goroutine for each input channel
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan int) {
            defer wg.Done()
            for n := range c {
                out <- n
            }
        }(ch)
    }
    
    // Close output when all inputs are done
    go func() {
        wg.Wait()
        close(out)
    }()
    
    return out
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Fan-Out / Fan-In Pattern                        ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Source
    nums := source(1, 2, 3, 4, 5, 6, 7, 8)
    
    // Fan-out: distribute to multiple workers
    c1 := square(nums)
    c2 := square(nums)
    c3 := square(nums)
    
    // Fan-in: merge results
    merged := fanIn(c1, c2, c3)
    
    // Consume results
    fmt.Println("\n📊 Squared Results:")
    for n := range merged {
        fmt.Printf("   %d\n", n)
    }
}
```

---

## 🔗 Pipeline Pattern

```go
// pipeline.go
package main

import "fmt"

/*
PIPELINE PATTERN

Each stage:
1. Receives from upstream via inbound channel
2. Performs some operation
3. Sends to downstream via outbound channel

┌───────┐   ┌──────────┐   ┌──────────┐   ┌───────┐
│ Source│──►│ Stage 1  │──►│ Stage 2  │──►│ Sink  │
│       │   │ (filter) │   │(transform)│  │       │
└───────┘   └──────────┘   └──────────┘   └───────┘
*/

// Generate numbers
func generate(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}

// Filter even numbers
func filterEven(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            if n%2 == 0 {
                out <- n
            }
        }
        close(out)
    }()
    return out
}

// Double values
func double(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * 2
        }
        close(out)
    }()
    return out
}

// Add prefix
func format(in <-chan int) <-chan string {
    out := make(chan string)
    go func() {
        for n := range in {
            out <- fmt.Sprintf("Value: %d", n)
        }
        close(out)
    }()
    return out
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Pipeline Pattern                                ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Build pipeline: generate → filter even → double → format
    nums := generate(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    evens := filterEven(nums)
    doubled := double(evens)
    formatted := format(doubled)
    
    // Consume
    fmt.Println("\n📊 Pipeline Output:")
    for s := range formatted {
        fmt.Printf("   %s\n", s)
    }
}
```

---

## 🚦 Semaphore Pattern

```go
// semaphore.go
package main

import (
    "fmt"
    "sync"
    "time"
)

/*
SEMAPHORE PATTERN

Limit concurrent operations using a buffered channel.

Buffered channel of size N = max N concurrent operations
*/

type Semaphore chan struct{}

func NewSemaphore(max int) Semaphore {
    return make(Semaphore, max)
}

func (s Semaphore) Acquire() {
    s <- struct{}{}
}

func (s Semaphore) Release() {
    <-s
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           Semaphore Pattern                               ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Limit to 3 concurrent operations
    sem := NewSemaphore(3)
    var wg sync.WaitGroup
    
    fmt.Println("\n📊 Processing (max 3 concurrent):")
    
    for i := 1; i <= 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            sem.Acquire()
            defer sem.Release()
            
            fmt.Printf("   [%s] Task %d started\n", 
                time.Now().Format("15:04:05.000"), id)
            time.Sleep(200 * time.Millisecond)
            fmt.Printf("   [%s] Task %d done\n", 
                time.Now().Format("15:04:05.000"), id)
        }(i)
    }
    
    wg.Wait()
    fmt.Println("   All tasks completed!")
}
```

---

## ⚠️ errgroup Pattern

```go
// errgroup_pattern.go
package main

import (
    "context"
    "errors"
    "fmt"
    "time"
    
    "golang.org/x/sync/errgroup"
)

/*
ERRGROUP PATTERN

Like WaitGroup but:
- Returns first error
- Cancels other goroutines on error
- Uses context for cancellation
*/

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           errgroup Pattern                                ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    ctx := context.Background()
    g, ctx := errgroup.WithContext(ctx)
    
    // Task 1: Success
    g.Go(func() error {
        fmt.Println("   Task 1: Starting...")
        time.Sleep(100 * time.Millisecond)
        fmt.Println("   Task 1: Done")
        return nil
    })
    
    // Task 2: Fails
    g.Go(func() error {
        fmt.Println("   Task 2: Starting...")
        time.Sleep(50 * time.Millisecond)
        fmt.Println("   Task 2: Error!")
        return errors.New("task 2 failed")
    })
    
    // Task 3: Checks context
    g.Go(func() error {
        fmt.Println("   Task 3: Starting...")
        select {
        case <-ctx.Done():
            fmt.Println("   Task 3: Cancelled!")
            return ctx.Err()
        case <-time.After(200 * time.Millisecond):
            fmt.Println("   Task 3: Done")
            return nil
        }
    })
    
    // Wait for all and get first error
    if err := g.Wait(); err != nil {
        fmt.Printf("\n   Error: %v\n", err)
    }
}

/*
Install: go get golang.org/x/sync/errgroup
*/
```

---

## 🎯 Pattern Selection Guide

```
┌─────────────────────────────────────────────────────────────────┐
│  WHEN TO USE WHICH PATTERN                                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  WORKER POOL                                                    │
│    • Fixed number of concurrent workers                         │
│    • Rate limiting / throttling                                 │
│    • Database connection pool                                   │
│    • API rate limits                                            │
│                                                                 │
│  FAN-OUT / FAN-IN                                               │
│    • Parallelize independent operations                         │
│    • Aggregate results from multiple sources                    │
│    • Map-reduce style processing                                │
│                                                                 │
│  PIPELINE                                                       │
│    • Multi-stage processing                                     │
│    • Data transformation chains                                 │
│    • ETL operations                                             │
│                                                                 │
│  SEMAPHORE                                                      │
│    • Simple concurrency limiting                                │
│    • Resource access control                                    │
│    • When you don't need full worker pool                       │
│                                                                 │
│  ERRGROUP                                                       │
│    • Multiple concurrent tasks                                  │
│    • Need error handling                                        │
│    • Want cancellation on first error                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ➡️ Next Steps

**Next Topic:** [58 - Cryptography Basics](./58-crypto.md)

