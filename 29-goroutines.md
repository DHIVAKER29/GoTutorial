# 29 - Goroutines & Concurrency Fundamentals

> Understanding concurrency, parallelism, the Go scheduler, and goroutines in depth.

---

## 📌 What You'll Learn

- Concurrency vs Parallelism (the critical difference!)
- Processes, Threads, and Goroutines
- How the Go Runtime Scheduler works (M:N scheduling)
- GOMAXPROCS and controlling parallelism
- Creating and managing goroutines
- Common gotchas and best practices
- Race conditions introduction

---

## 🧠 Concurrency vs Parallelism

### The Most Misunderstood Concept

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CONCURRENCY ≠ PARALLELISM                                      │
│                                                                 │
│  CONCURRENCY = "Dealing with many things at once"               │
│  PARALLELISM = "Doing many things at once"                      │
│                                                                 │
│  ────────────────────────────────────────────────────────────   │
│                                                                 │
│  REAL WORLD ANALOGY: A Coffee Shop                              │
│                                                                 │
│  CONCURRENCY (1 barista, multiple orders):                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  1 Barista handles multiple orders by SWITCHING:        │   │
│  │                                                         │   │
│  │  • Take order A                                         │   │
│  │  • Start brewing A, while waiting...                    │   │
│  │  • Take order B                                         │   │
│  │  • A is done, serve A                                   │   │
│  │  • Start brewing B, while waiting...                    │   │
│  │  • Take order C                                         │   │
│  │  • B is done, serve B                                   │   │
│  │  ... and so on                                          │   │
│  │                                                         │   │
│  │  ONE person, managing MULTIPLE tasks                    │   │
│  │  Tasks INTERLEAVE but don't run SIMULTANEOUSLY          │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  PARALLELISM (3 baristas, multiple orders):                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  3 Baristas each handle different orders SIMULTANEOUSLY:│   │
│  │                                                         │   │
│  │  Barista 1: Making order A ─────────────────            │   │
│  │  Barista 2: Making order B ─────────────────            │   │
│  │  Barista 3: Making order C ─────────────────            │   │
│  │                                                         │   │
│  │  THREE people, doing work AT THE SAME TIME              │   │
│  │  Tasks run SIMULTANEOUSLY on multiple CPUs              │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  KEY INSIGHT:                                                   │
│  • CONCURRENCY is about STRUCTURE (code organization)           │
│  • PARALLELISM is about EXECUTION (hardware)                    │
│  • You can have concurrency WITHOUT parallelism (1 CPU)         │
│  • Go gives you concurrency; runtime decides parallelism        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### In Programming Terms

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  SINGLE CORE (Concurrency only):                                │
│                                                                 │
│  Time ──────────────────────────────────────────────►           │
│  CPU:  [Task A][Task B][Task A][Task C][Task B][Task A]         │
│        ↑                                                        │
│        Tasks take turns (time-slicing)                          │
│        Only ONE task runs at any moment                         │
│                                                                 │
│  ────────────────────────────────────────────────────────────   │
│                                                                 │
│  MULTI CORE (Parallelism possible):                             │
│                                                                 │
│  Time ──────────────────────────────────────────────►           │
│  CPU1: [Task A][Task A][Task A][Task A]                         │
│  CPU2: [Task B][Task B][Task B][Task B]                         │
│  CPU3: [Task C][Task C][Task C][Task C]                         │
│        ↑                                                        │
│        Multiple tasks run SIMULTANEOUSLY                        │
│        True parallelism on multiple cores                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Processes vs Threads vs Goroutines

### The Evolution

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  PROCESS (Heaviest)                                             │
│  ─────────────────                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Own memory space (isolated)                            │   │
│  │  Own file descriptors, security context                 │   │
│  │  Created via fork() - expensive!                        │   │
│  │  Communication via IPC (pipes, sockets)                 │   │
│  │  Crash doesn't affect other processes                   │   │
│  │                                                         │   │
│  │  Memory: ~10MB+ overhead                                │   │
│  │  Creation: Milliseconds                                 │   │
│  │  Context switch: ~1000+ microseconds                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  THREAD (Medium)                                                │
│  ──────────────                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Share process memory (same address space)              │   │
│  │  Own stack (~1-8MB fixed on Linux)                      │   │
│  │  Managed by OS kernel                                   │   │
│  │  Communication via shared memory                        │   │
│  │  Crash can affect entire process                        │   │
│  │                                                         │   │
│  │  Memory: ~1-8MB stack                                   │   │
│  │  Creation: ~100 microseconds                            │   │
│  │  Context switch: ~1-10 microseconds                     │   │
│  │  Max: ~10,000 threads per process                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  GOROUTINE (Lightest)                                           │
│  ────────────────────                                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Share process memory                                   │   │
│  │  Own stack (~2KB initial, grows dynamically!)           │   │
│  │  Managed by Go runtime (NOT OS)                         │   │
│  │  Communication via channels (or shared memory)          │   │
│  │  Multiplexed onto OS threads (M:N scheduling)           │   │
│  │                                                         │   │
│  │  Memory: ~2KB initial (grows to MB if needed)           │   │
│  │  Creation: ~0.3 microseconds                            │   │
│  │  Context switch: ~0.2 microseconds                      │   │
│  │  Max: MILLIONS of goroutines!                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  COMPARISON TABLE:                                              │
│  ┌────────────────┬───────────┬───────────┬───────────────┐    │
│  │                │ Process   │ Thread    │ Goroutine     │    │
│  ├────────────────┼───────────┼───────────┼───────────────┤    │
│  │ Stack size     │ N/A       │ 1-8 MB    │ 2 KB (grows)  │    │
│  │ Creation time  │ ~100ms    │ ~100µs    │ ~0.3µs        │    │
│  │ Context switch │ ~1000µs   │ ~1-10µs   │ ~0.2µs        │    │
│  │ Max practical  │ ~100s     │ ~10,000   │ ~1,000,000+   │    │
│  │ Managed by     │ OS        │ OS        │ Go Runtime    │    │
│  │ Scheduling     │ Preemptive│ Preemptive│ Cooperative*  │    │
│  └────────────────┴───────────┴───────────┴───────────────┘    │
│                                                                 │
│  * Go 1.14+ added preemptive scheduling for goroutines          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚙️ Go Runtime Scheduler (GMP Model)

### Understanding M:N Scheduling

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO SCHEDULER: GMP MODEL                                        │
│                                                                 │
│  G = Goroutine (the work to be done)                            │
│  M = Machine (OS thread that executes)                          │
│  P = Processor (scheduling context, holds runqueue)             │
│                                                                 │
│  ┌────────────────────────────────────────────────────────────┐│
│  │                                                            ││
│  │     ┌───┐ ┌───┐ ┌───┐ ┌───┐     Global Run Queue          ││
│  │     │ G │ │ G │ │ G │ │ G │ ◄── (waiting goroutines)      ││
│  │     └───┘ └───┘ └───┘ └───┘                                ││
│  │                                                            ││
│  │  ┌──────────────────────────────────────────────────────┐  ││
│  │  │  P0 (Processor 0)              P1 (Processor 1)      │  ││
│  │  │  ┌─────────────────┐           ┌─────────────────┐   │  ││
│  │  │  │ Local Run Queue │           │ Local Run Queue │   │  ││
│  │  │  │ ┌───┬───┬───┐   │           │ ┌───┬───┬───┐   │   │  ││
│  │  │  │ │ G │ G │ G │   │           │ │ G │ G │ G │   │   │  ││
│  │  │  │ └───┴───┴───┘   │           │ └───┴───┴───┘   │   │  ││
│  │  │  └────────┬────────┘           └────────┬────────┘   │  ││
│  │  │           │                             │             │  ││
│  │  │           ▼                             ▼             │  ││
│  │  │      ┌─────────┐                   ┌─────────┐        │  ││
│  │  │      │ M (OS   │                   │ M (OS   │        │  ││
│  │  │      │ Thread) │                   │ Thread) │        │  ││
│  │  │      └────┬────┘                   └────┬────┘        │  ││
│  │  └───────────┼─────────────────────────────┼─────────────┘  ││
│  │              │                             │                ││
│  │              ▼                             ▼                ││
│  │         ┌─────────┐                   ┌─────────┐           ││
│  │         │  CPU 0  │                   │  CPU 1  │           ││
│  │         └─────────┘                   └─────────┘           ││
│  │                                                            ││
│  └────────────────────────────────────────────────────────────┘│
│                                                                 │
│  HOW IT WORKS:                                                  │
│  1. Goroutines (G) are created and added to queues              │
│  2. Processors (P) pick up Gs from their local queue            │
│  3. Machines (M) execute the G that P gives them                │
│  4. Number of Ps = GOMAXPROCS (default = CPU cores)             │
│  5. Ms can be more than Ps (some may be blocked on syscalls)    │
│                                                                 │
│  WORK STEALING:                                                 │
│  • If P's local queue is empty, steal from other Ps             │
│  • If all local queues empty, check global queue                │
│  • Ensures load balancing across all Ps                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 GOMAXPROCS

```go
// gomaxprocs.go
package main

import (
    "fmt"
    "runtime"
    "sync"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           GOMAXPROCS - CONTROLLING PARALLELISM            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Check current values
    fmt.Println("\n📊 System Information:")
    fmt.Printf("   NumCPU (logical cores): %d\n", runtime.NumCPU())
    fmt.Printf("   GOMAXPROCS (default): %d\n", runtime.GOMAXPROCS(0))
    fmt.Printf("   NumGoroutine (current): %d\n", runtime.NumGoroutine())
    
    // GOMAXPROCS explanation
    fmt.Println("\n📊 GOMAXPROCS Explained:")
    fmt.Println("   • Controls max number of OS threads running Go code simultaneously")
    fmt.Println("   • Default = number of CPU cores (since Go 1.5)")
    fmt.Println("   • GOMAXPROCS(1) = no parallelism (concurrency only)")
    fmt.Println("   • GOMAXPROCS(4) = up to 4 goroutines run in parallel")
    
    // Demonstrate difference
    fmt.Println("\n📊 Experiment: GOMAXPROCS Effect")
    
    // With GOMAXPROCS = 1 (no parallelism)
    fmt.Println("\n   GOMAXPROCS = 1 (sequential on single thread):")
    runtime.GOMAXPROCS(1)
    runWorkers()
    
    // With GOMAXPROCS = NumCPU (max parallelism)
    fmt.Printf("\n   GOMAXPROCS = %d (parallel on multiple cores):\n", runtime.NumCPU())
    runtime.GOMAXPROCS(runtime.NumCPU())
    runWorkers()
    
    // When to change GOMAXPROCS
    fmt.Println("\n💡 When to Change GOMAXPROCS:")
    fmt.Println("   • Usually DON'T - default is optimal")
    fmt.Println("   • Container limits: may need to match cgroup CPU limit")
    fmt.Println("   • CPU-bound work: might limit to leave room for GC")
    fmt.Println("   • Debugging: GOMAXPROCS=1 to simplify race condition debugging")
}

func runWorkers() {
    start := time.Now()
    var wg sync.WaitGroup
    
    for i := 0; i < 4; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            // Simulate CPU work
            sum := 0
            for j := 0; j < 10000000; j++ {
                sum += j
            }
            _ = sum
        }(i)
    }
    
    wg.Wait()
    fmt.Printf("   Completed in: %v\n", time.Since(start))
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           GOMAXPROCS - CONTROLLING PARALLELISM            ║
╚══════════════════════════════════════════════════════════╝

📊 System Information:
   NumCPU (logical cores): 8
   GOMAXPROCS (default): 8
   NumGoroutine (current): 1

📊 GOMAXPROCS Explained:
   • Controls max number of OS threads running Go code simultaneously
   • Default = number of CPU cores (since Go 1.5)
   • GOMAXPROCS(1) = no parallelism (concurrency only)
   • GOMAXPROCS(4) = up to 4 goroutines run in parallel

📊 Experiment: GOMAXPROCS Effect

   GOMAXPROCS = 1 (sequential on single thread):
   Completed in: 45.234ms

   GOMAXPROCS = 8 (parallel on multiple cores):
   Completed in: 12.456ms

💡 When to Change GOMAXPROCS:
   • Usually DON'T - default is optimal
   • Container limits: may need to match cgroup CPU limit
   • CPU-bound work: might limit to leave room for GC
   • Debugging: GOMAXPROCS=1 to simplify race condition debugging
```

---

## 📝 Creating Goroutines - Complete Guide

```go
// goroutines_complete.go
package main

import (
    "fmt"
    "runtime"
    "sync"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           GOROUTINES - COMPLETE GUIDE                     ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Method 1: Named function
    fmt.Println("\n📊 Method 1: Named Function")
    var wg sync.WaitGroup
    wg.Add(1)
    go namedFunction(&wg)
    wg.Wait()
    
    // Method 2: Anonymous function
    fmt.Println("\n📊 Method 2: Anonymous Function")
    wg.Add(1)
    go func() {
        defer wg.Done()
        fmt.Println("   Hello from anonymous function!")
    }()
    wg.Wait()
    
    // Method 3: With parameters (IMPORTANT!)
    fmt.Println("\n📊 Method 3: With Parameters (Pass by Value)")
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go func(n int) {  // n is a COPY of i
            defer wg.Done()
            fmt.Printf("   Goroutine received: %d\n", n)
        }(i)  // Pass i here!
    }
    wg.Wait()
    
    // Method 4: Method as goroutine
    fmt.Println("\n📊 Method 4: Method as Goroutine")
    worker := &Worker{ID: 1}
    wg.Add(1)
    go worker.DoWork(&wg)
    wg.Wait()
    
    // Check goroutine count
    fmt.Println("\n📊 Goroutine Statistics:")
    fmt.Printf("   Active goroutines: %d\n", runtime.NumGoroutine())
    
    // Spawn many goroutines
    fmt.Println("\n📊 Spawning 10,000 Goroutines:")
    for i := 0; i < 10000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            time.Sleep(10 * time.Millisecond)
        }()
    }
    fmt.Printf("   Peak goroutines: %d\n", runtime.NumGoroutine())
    wg.Wait()
    fmt.Printf("   After completion: %d\n", runtime.NumGoroutine())
}
```

**Output:**
```
(goroutine output order may vary)
╔══════════════════════════════════════════════════════════╗
║           GOROUTINES - COMPLETE GUIDE                     ║
╚══════════════════════════════════════════════════════════╝

📊 Method 1: Named Function
   Hello from named function!

📊 Method 2: Anonymous Function
   Hello from anonymous function!

📊 Method 3: With Parameters (Pass by Value)
   Goroutine received: 1
   Goroutine received: 2
   Goroutine received: 3

📊 Method 4: Method as Goroutine
   Worker 1 doing work!

📊 Goroutine Statistics:
   Active goroutines: 1

📊 Spawning 10,000 Goroutines:
   Peak goroutines: 10001
   After completion: 1
```

func namedFunction(wg *sync.WaitGroup) {
    defer wg.Done()
    fmt.Println("   Hello from named function!")
}

type Worker struct {
    ID int
}

func (w *Worker) DoWork(wg *sync.WaitGroup) {
    defer wg.Done()
    fmt.Printf("   Worker %d doing work!\n", w.ID)
}
```

---

## ⚠️ Goroutine Gotchas (Detailed)

```go
// goroutine_gotchas.go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           GOROUTINE GOTCHAS - DETAILED                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // GOTCHA 1: Loop variable capture
    fmt.Println("\n❌ GOTCHA 1: Loop Variable Capture")
    fmt.Println("   The Classic Bug:")
    
    var wg sync.WaitGroup
    values := []int{1, 2, 3, 4, 5}
    
    fmt.Println("\n   BAD (all print same value):")
    for _, v := range values {
        wg.Add(1)
        go func() {
            defer wg.Done()
            fmt.Printf("   value: %d\n", v)  // v is SHARED!
        }()
    }
    wg.Wait()
    
    fmt.Println("\n   GOOD (pass as parameter):")
    for _, v := range values {
        wg.Add(1)
        go func(val int) {  // val is a COPY
            defer wg.Done()
            fmt.Printf("   value: %d\n", val)
        }(v)  // Pass v here
    }
    wg.Wait()
    
    fmt.Println("\n   ALSO GOOD (shadow variable - Go 1.22+ default):")
    for _, v := range values {
        v := v  // Shadow - creates new variable each iteration
        wg.Add(1)
        go func() {
            defer wg.Done()
            fmt.Printf("   value: %d\n", v)
        }()
    }
    wg.Wait()
    
    // GOTCHA 2: Main exits, goroutines die
    fmt.Println("\n❌ GOTCHA 2: Main Exits = Goroutines Die")
    fmt.Println("   func main() {")
    fmt.Println("       go doWork()  // May NEVER run!")
    fmt.Println("   }  // main exits immediately")
    fmt.Println("")
    fmt.Println("   FIX: Use WaitGroup, channels, or select{}")
    
    // GOTCHA 3: Goroutine leak
    fmt.Println("\n❌ GOTCHA 3: Goroutine Leak")
    fmt.Println("   ch := make(chan int)")
    fmt.Println("   go func() {")
    fmt.Println("       val := <-ch  // Blocks forever if nothing sent!")
    fmt.Println("   }()")
    fmt.Println("   // Forgot to send or close ch - goroutine leaks!")
    fmt.Println("")
    fmt.Println("   FIX: Always ensure goroutines can exit")
    fmt.Println("        Use context for cancellation")
    
    // GOTCHA 4: Data race
    fmt.Println("\n❌ GOTCHA 4: Data Race")
    fmt.Println("   counter := 0")
    fmt.Println("   for i := 0; i < 1000; i++ {")
    fmt.Println("       go func() { counter++ }()  // RACE!")
    fmt.Println("   }")
    fmt.Println("")
    fmt.Println("   FIX: Use sync.Mutex, sync/atomic, or channels")
    fmt.Println("   DETECT: Run with 'go run -race program.go'")
    
    // GOTCHA 5: Panic in goroutine crashes everything
    fmt.Println("\n❌ GOTCHA 5: Panic in Goroutine")
    fmt.Println("   go func() {")
    fmt.Println("       panic(\"oops\")  // Crashes ENTIRE program!")
    fmt.Println("   }()")
    fmt.Println("")
    fmt.Println("   FIX: Use recover in deferred function:")
    fmt.Println("   go func() {")
    fmt.Println("       defer func() {")
    fmt.Println("           if r := recover(); r != nil {")
    fmt.Println("               log.Printf(\"Recovered: %v\", r)")
    fmt.Println("           }")
    fmt.Println("       }()")
    fmt.Println("       // risky code here")
    fmt.Println("   }()")
}
```

**Output:**
```
(goroutine output order may vary)
╔══════════════════════════════════════════════════════════╗
║           GOROUTINE GOTCHAS - DETAILED                    ║
╚══════════════════════════════════════════════════════════╝

❌ GOTCHA 1: Loop Variable Capture
   The Classic Bug:

   BAD (all print same value):
   value: 5
   value: 5
   value: 5
   value: 5
   value: 5

   GOOD (pass as parameter):
   value: 1
   value: 2
   value: 3
   value: 4
   value: 5

   ALSO GOOD (shadow variable - Go 1.22+ default):
   value: 1
   value: 2
   value: 3
   value: 4
   value: 5

❌ GOTCHA 2: Main Exits = Goroutines Die
   func main() {
       go doWork()  // May NEVER run!
   }  // main exits immediately

   FIX: Use WaitGroup, channels, or select{}

❌ GOTCHA 3: Goroutine Leak
   ch := make(chan int)
   go func() {
       val := <-ch  // Blocks forever if nothing sent!
   }()
   // Forgot to send or close ch - goroutine leaks!

   FIX: Always ensure goroutines can exit
        Use context for cancellation

❌ GOTCHA 4: Data Race
   counter := 0
   for i := 0; i < 1000; i++ {
       go func() { counter++ }()  // RACE!
   }

   FIX: Use sync.Mutex, sync/atomic, or channels
   DETECT: Run with 'go run -race program.go'

❌ GOTCHA 5: Panic in Goroutine
   go func() {
       panic("oops")  // Crashes ENTIRE program!
   }()

   FIX: Use recover in deferred function:
   go func() {
       defer func() {
           if r := recover(); r != nil {
               log.Printf("Recovered: %v", r)
           }
       }()
       // risky code here
   }()
```

---

## 🔄 sync.WaitGroup In-Depth

```go
// waitgroup_indepth.go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           WAITGROUP - IN DEPTH                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    var wg sync.WaitGroup
    
    // Basic usage
    fmt.Println("\n📊 Basic WaitGroup Pattern:")
    for i := 1; i <= 3; i++ {
        wg.Add(1)  // BEFORE launching goroutine!
        go func(id int) {
            defer wg.Done()  // ALWAYS use defer
            fmt.Printf("   Worker %d starting\n", id)
            time.Sleep(time.Duration(id*50) * time.Millisecond)
            fmt.Printf("   Worker %d done\n", id)
        }(i)
    }
    wg.Wait()  // Block until counter = 0
    fmt.Println("   All workers completed!")
    
    // Common mistake: Add inside goroutine
    fmt.Println("\n⚠️ Common Mistake:")
    fmt.Println("   // WRONG!")
    fmt.Println("   go func() {")
    fmt.Println("       wg.Add(1)  // Race condition!")
    fmt.Println("       defer wg.Done()")
    fmt.Println("   }()")
    fmt.Println("")
    fmt.Println("   // RIGHT!")
    fmt.Println("   wg.Add(1)  // Add BEFORE go")
    fmt.Println("   go func() {")
    fmt.Println("       defer wg.Done()")
    fmt.Println("   }()")
    
    // Add with count
    fmt.Println("\n📊 Add with Count:")
    numWorkers := 5
    wg.Add(numWorkers)  // Add all at once
    for i := 0; i < numWorkers; i++ {
        go func(id int) {
            defer wg.Done()
            fmt.Printf("   Worker %d\n", id)
        }(i)
    }
    wg.Wait()
    
    // WaitGroup is not reusable while waiting
    fmt.Println("\n📊 WaitGroup Lifecycle:")
    fmt.Println("   • Can reuse after Wait() returns")
    fmt.Println("   • Cannot Add() while Wait() is blocking")
    fmt.Println("   • counter must reach 0 for Wait() to return")
}
```

**Output:**
```
(goroutine output order may vary)
╔══════════════════════════════════════════════════════════╗
║           WAITGROUP - IN DEPTH                            ║
╚══════════════════════════════════════════════════════════╝

📊 Basic WaitGroup Pattern:
   Worker 1 starting
   Worker 2 starting
   Worker 3 starting
   Worker 1 done
   Worker 2 done
   Worker 3 done
   All workers completed!

⚠️ Common Mistake:
   // WRONG!
   go func() {
       wg.Add(1)  // Race condition!
       defer wg.Done()
   }()

   // RIGHT!
   wg.Add(1)  // Add BEFORE go
   go func() {
       defer wg.Done()
   }()

📊 Add with Count:
   Worker 0
   Worker 1
   Worker 2
   Worker 3
   Worker 4

📊 WaitGroup Lifecycle:
   • Can reuse after Wait() returns
   • Cannot Add() while Wait() is blocking
   • counter must reach 0 for Wait() to return
```

---

## 🏎️ Race Condition Detection

```go
// race_detection.go
package main

import (
    "fmt"
    "sync"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           RACE CONDITION DETECTION                        ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 What is a Data Race?")
    fmt.Println("   When two goroutines access the same memory")
    fmt.Println("   AND at least one is a write")
    fmt.Println("   AND they're not synchronized")
    
    fmt.Println("\n📊 Race Detector:")
    fmt.Println("   go run -race program.go")
    fmt.Println("   go test -race ./...")
    fmt.Println("   go build -race program.go")
    
    fmt.Println("\n❌ Example Race Condition:")
    racyCounter()
    
    fmt.Println("\n✅ Fixed with Mutex:")
    safeCounter()
    
    fmt.Println("\n✅ Fixed with Atomic:")
    atomicCounter()
    
    fmt.Println("\n✅ Fixed with Channel:")
    channelCounter()
}

func racyCounter() {
    counter := 0
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            counter++  // DATA RACE!
        }()
    }
    wg.Wait()
    fmt.Printf("   Racy result: %d (should be 1000)\n", counter)
}

func safeCounter() {
    counter := 0
    var mu sync.Mutex
    var wg sync.WaitGroup
    
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            mu.Lock()
            counter++
            mu.Unlock()
        }()
    }
    wg.Wait()
    fmt.Printf("   Mutex result: %d\n", counter)
}

func atomicCounter() {
    // Using sync/atomic
    fmt.Println("   (See sync/atomic examples in sync package file)")
}

func channelCounter() {
    counter := 0
    done := make(chan bool)
    increment := make(chan bool)
    
    // Counter goroutine (single owner)
    go func() {
        for range increment {
            counter++
        }
        done <- true
    }()
    
    // Send 1000 increments
    var wg sync.WaitGroup
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            increment <- true
        }()
    }
    wg.Wait()
    close(increment)
    <-done
    
    fmt.Printf("   Channel result: %d\n", counter)
}
```

**Output:**
```
(goroutine output order may vary)
╔══════════════════════════════════════════════════════════╗
║           RACE CONDITION DETECTION                        ║
╚══════════════════════════════════════════════════════════╝

📊 What is a Data Race?
   When two goroutines access the same memory
   AND at least one is a write
   AND they're not synchronized

📊 Race Detector:
   go run -race program.go
   go test -race ./...
   go build -race program.go

❌ Example Race Condition:
   Racy result: 987 (should be 1000)

✅ Fixed with Mutex:
   Mutex result: 1000

✅ Fixed with Atomic:
   (See sync/atomic examples in sync package file)

✅ Fixed with Channel:
   Channel result: 1000
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│                                                                 │
│  new Thread(() -> {                go func() {                  │
│      doWork();                         doWork()                 │
│  }).start();                       }()                          │
│                                                                 │
│  ExecutorService pool =            // Just use goroutines!      │
│    Executors.newFixedThreadPool(n) // Go runtime handles it     │
│                                                                 │
│  synchronized (lock) {             mu.Lock()                    │
│      // critical section           // critical section          │
│  }                                 mu.Unlock()                  │
│                                                                 │
│  thread.join();                    wg.Wait()                    │
│                                                                 │
│  // 10,000 threads = ~80GB RAM     // 1,000,000 goroutines      │
│  // and very slow                  // = ~2GB RAM and fast       │
│                                                                 │
│  Thread.currentThread()            runtime.NumGoroutine()       │
│                                                                 │
│  Runtime.getRuntime()              runtime.GOMAXPROCS()         │
│    .availableProcessors()          runtime.NumCPU()             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **Concurrency ≠ Parallelism** - structure vs execution
2. **Goroutines are cheap** - 2KB stack, create millions
3. **GMP Model** - Go schedules G on M via P
4. **GOMAXPROCS** - controls max parallel execution (default = CPU cores)
5. **Always use WaitGroup** or channels to synchronize
6. **Pass loop variables** as parameters to avoid capture bugs
7. **Use `go run -race`** to detect data races
8. **Main exits = goroutines die** - ensure proper synchronization

---

## ➡️ Next Steps

**Next Topic:** [30 - Channels](./30-channels.md)
