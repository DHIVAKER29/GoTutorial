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

**Purpose:** Limit concurrency, control resource usage (DB connections, API rate limits), backpressure handling.

```go
func worker(id int, jobs <-chan Job, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()
    for job := range jobs {
        time.Sleep(100 * time.Millisecond)
        results <- Result{JobID: job.ID, Output: fmt.Sprintf("Worker %d: %s", id, job.Data)}
    }
}

func main() {
    jobs := make(chan Job, 10)
    results := make(chan Result, 10)
    var wg sync.WaitGroup

    for w := 1; w <= 3; w++ {
        wg.Add(1)
        go worker(w, jobs, results, &wg)
    }

    for j := 1; j <= 10; j++ {
        jobs <- Job{ID: j, Data: fmt.Sprintf("Job-%d", j)}
    }
    close(jobs)

    go func() {
        wg.Wait()
        close(results)
    }()

    for r := range results {
        fmt.Println(r.Output)
    }
}
// Output: Worker 1/2/3 processed jobs (order varies)
```

---

## 📤📥 Fan-Out / Fan-In Pattern

**Fan-out:** One source → Multiple processors  
**Fan-in:** Multiple sources → One destination

```go
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
            out <- n * n
        }
        close(out)
    }()
    return out
}

func fanIn(channels ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan int) {
            defer wg.Done()
            for n := range c {
                out <- n
            }
        }(ch)
    }
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}

// Usage: nums := source(1,2,3,4,5,6,7,8)
// c1, c2, c3 := square(nums), square(nums), square(nums)
// merged := fanIn(c1, c2, c3)
// Output: 1, 4, 9, 16, 25, 36, 49, 64 (order varies)
```

---

## 🔗 Pipeline Pattern

**Each stage:** Receives from upstream → Performs operation → Sends to downstream

```go
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

// Pipeline: generate → filterEven → double
nums := generate(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
evens := filterEven(nums)
doubled := double(evens)
// Output: 4, 8, 12, 16, 20
```

---

## 🚦 Semaphore Pattern

**Limit concurrent operations** using a buffered channel.

```go
type Semaphore chan struct{}

func NewSemaphore(max int) Semaphore {
    return make(Semaphore, max)
}

func (s Semaphore) Acquire() { s <- struct{}{} }
func (s Semaphore) Release() { <-s }

sem := NewSemaphore(3)
var wg sync.WaitGroup
for i := 1; i <= 10; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        sem.Acquire()
        defer sem.Release()
        // Do work (max 3 concurrent)
        time.Sleep(200 * time.Millisecond)
    }(i)
}
wg.Wait()
// Output: Max 3 tasks run at a time
```

---

## ⚠️ errgroup Pattern

**Like WaitGroup but:** Returns first error, cancels others on error, uses context.

```go
g, ctx := errgroup.WithContext(context.Background())

g.Go(func() error {
    time.Sleep(100 * time.Millisecond)
    return nil
})
g.Go(func() error {
    time.Sleep(50 * time.Millisecond)
    return errors.New("task 2 failed")
})
g.Go(func() error {
    select {
    case <-ctx.Done():
        return ctx.Err()
    case <-time.After(200 * time.Millisecond):
        return nil
    }
})

if err := g.Wait(); err != nil {
    fmt.Println("Error:", err)
}
// Output: Task 2 fails, Task 3 is cancelled
```

---

## 🎯 Pattern Selection Guide

| Pattern | Use When |
|---------|----------|
| **Worker Pool** | Fixed workers, rate limiting, DB connection pool |
| **Fan-Out/Fan-In** | Parallelize independent ops, aggregate from multiple sources |
| **Pipeline** | Multi-stage processing, data transformation chains |
| **Semaphore** | Simple concurrency limiting, resource access control |
| **errgroup** | Multiple tasks, need error handling, cancel on first error |

---

## ➡️ Next Steps

**Next Topic:** [58 - Cryptography Basics](./58-crypto.md)
