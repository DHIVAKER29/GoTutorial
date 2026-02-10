# 12 - Switch Statements

> Go's powerful switch statement with unique features.

---

## 📌 What You'll Learn

- Basic switch syntax
- Switch without expression (cleaner if-else)
- Type switch for interfaces
- Fallthrough (opt-in, not default!)
- Sample programs for each pattern

---

## 🔀 Switch Basics

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GO's SWITCH IS DIFFERENT!                                      │
│                                                                 │
│  1. NO automatic fallthrough (unlike C/Java)                    │
│  2. Cases can have multiple values                              │
│  3. Cases can be expressions (not just constants)               │
│  4. Switch without expression = cleaner if-else                 │
│  5. Type switch for interface values                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Sample Program: Switch Basics

```go
// switch_basics.go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              SWITCH STATEMENTS                            ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Basic switch
    fmt.Println("\n📊 Basic Switch:")
    day := time.Now().Weekday()
    switch day {
    case time.Saturday, time.Sunday:  // Multiple values!
        fmt.Println("   It's the weekend! 🎉")
    case time.Monday:
        fmt.Println("   Monday blues... 😴")
    case time.Friday:
        fmt.Println("   TGIF! 🎊")
    default:
        fmt.Printf("   Regular day: %s\n", day)
    }
    
    // Switch with expression
    fmt.Println("\n📊 Switch with Expression:")
    score := 85
    switch {
    case score >= 90:
        fmt.Println("   Grade: A")
    case score >= 80:
        fmt.Println("   Grade: B")
    case score >= 70:
        fmt.Println("   Grade: C")
    case score >= 60:
        fmt.Println("   Grade: D")
    default:
        fmt.Println("   Grade: F")
    }
    
    // Switch with initialization
    fmt.Println("\n📊 Switch with Init Statement:")
    switch hour := time.Now().Hour(); {
    case hour < 12:
        fmt.Printf("   Good morning! (hour: %d)\n", hour)
    case hour < 17:
        fmt.Printf("   Good afternoon! (hour: %d)\n", hour)
    default:
        fmt.Printf("   Good evening! (hour: %d)\n", hour)
    }
    
    // No break needed!
    fmt.Println("\n📊 No Break Needed (Go's Gift!):")
    num := 2
    switch num {
    case 1:
        fmt.Println("   One")
        // No break! Automatically stops here
    case 2:
        fmt.Println("   Two")
        // No break! Automatically stops here
    case 3:
        fmt.Println("   Three")
    }
    
    // Multiple values per case
    fmt.Println("\n📊 Multiple Values per Case:")
    char := 'a'
    switch char {
    case 'a', 'e', 'i', 'o', 'u':
        fmt.Printf("   '%c' is a vowel\n", char)
    default:
        fmt.Printf("   '%c' is a consonant\n", char)
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              SWITCH STATEMENTS                            ║
╚══════════════════════════════════════════════════════════╝

📊 Basic Switch:
   Regular day: Wednesday

📊 Switch with Expression:
   Grade: B

📊 Switch with Init Statement:
   Good afternoon! (hour: 14)

📊 No Break Needed (Go's Gift!):
   Two

📊 Multiple Values per Case:
   'a' is a vowel
```

---

## 🔽 Fallthrough

```go
// fallthrough_demo.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              FALLTHROUGH                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 Fallthrough (Opt-in):")
    num := 1
    switch num {
    case 1:
        fmt.Println("   One")
        fallthrough  // Continue to next case
    case 2:
        fmt.Println("   Two (via fallthrough)")
        fallthrough
    case 3:
        fmt.Println("   Three (via fallthrough)")
        // No fallthrough here, stops
    case 4:
        fmt.Println("   Four (not reached)")
    }
    
    fmt.Println("\n⚠️ Key Difference from C/Java:")
    fmt.Println("   C/Java: fallthrough is DEFAULT (need break)")
    fmt.Println("   Go: NO fallthrough is DEFAULT (need fallthrough)")
    
    // Practical use: cumulative conditions
    fmt.Println("\n💡 Practical: Permission Levels")
    level := 2  // Admin level
    fmt.Printf("   Level %d permissions:\n", level)
    switch level {
    case 3:
        fmt.Println("   - Super Admin access")
        fallthrough
    case 2:
        fmt.Println("   - Admin access")
        fallthrough
    case 1:
        fmt.Println("   - User access")
        fallthrough
    case 0:
        fmt.Println("   - Guest access")
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              FALLTHROUGH                                  ║
╚══════════════════════════════════════════════════════════╝

📊 Fallthrough (Opt-in):
   One
   Two (via fallthrough)
   Three (via fallthrough)

⚠️ Key Difference from C/Java:
   C/Java: fallthrough is DEFAULT (need break)
   Go: NO fallthrough is DEFAULT (need fallthrough)

💡 Practical: Permission Levels
   Level 2 permissions:
   - Admin access
   - User access
   - Guest access
```

---

## 🔍 Type Switch

```go
// type_switch.go
package main

import "fmt"

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║              TYPE SWITCH                                  ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 Type Switch Examples:")
    
    describe(42)
    describe("hello")
    describe(3.14)
    describe(true)
    describe([]int{1, 2, 3})
    describe(nil)
}

func describe(value interface{}) {
    switch v := value.(type) {
    case nil:
        fmt.Println("   nil value")
    case int:
        fmt.Printf("   Integer: %d (doubled: %d)\n", v, v*2)
    case string:
        fmt.Printf("   String: %q (length: %d)\n", v, len(v))
    case float64:
        fmt.Printf("   Float: %f\n", v)
    case bool:
        fmt.Printf("   Boolean: %t\n", v)
    case []int:
        fmt.Printf("   Int slice: %v (length: %d)\n", v, len(v))
    default:
        fmt.Printf("   Unknown type: %T = %v\n", v, v)
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║              TYPE SWITCH                                  ║
╚══════════════════════════════════════════════════════════╝

📊 Type Switch Examples:
   Integer: 42 (doubled: 84)
   String: "hello" (length: 5)
   Float: 3.140000
   Boolean: true
   Int slice: [1 2 3] (length: 3)
   nil value
```

---

## 🏭 Production Patterns

```go
// switch_production.go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PRODUCTION SWITCH PATTERNS                      ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // HTTP status handling
    fmt.Println("\n📊 HTTP Status Handling:")
    handleStatus(200)
    handleStatus(404)
    handleStatus(500)
    
    // State machine
    fmt.Println("\n📊 State Machine:")
    processOrder("pending")
    processOrder("paid")
    processOrder("shipped")
    
    // Expression switch (cleaner than if-else chain)
    fmt.Println("\n📊 Expression Switch (replaces if-else chain):")
    checkAge(5)
    checkAge(15)
    checkAge(25)
    checkAge(70)
}

func handleStatus(code int) {
    switch code {
    case http.StatusOK:
        fmt.Printf("   %d: Success ✅\n", code)
    case http.StatusNotFound:
        fmt.Printf("   %d: Not Found 🔍\n", code)
    case http.StatusInternalServerError:
        fmt.Printf("   %d: Server Error 🔥\n", code)
    default:
        fmt.Printf("   %d: Other status\n", code)
    }
}

func processOrder(status string) {
    switch status {
    case "pending":
        fmt.Println("   Order pending → waiting for payment")
    case "paid":
        fmt.Println("   Order paid → processing shipment")
    case "shipped":
        fmt.Println("   Order shipped → tracking available")
    case "delivered":
        fmt.Println("   Order delivered → complete")
    default:
        fmt.Printf("   Unknown status: %s\n", status)
    }
}

func checkAge(age int) {
    switch {  // No expression = true
    case age < 0:
        fmt.Printf("   Age %d: Invalid\n", age)
    case age < 13:
        fmt.Printf("   Age %d: Child\n", age)
    case age < 20:
        fmt.Printf("   Age %d: Teenager\n", age)
    case age < 65:
        fmt.Printf("   Age %d: Adult\n", age)
    default:
        fmt.Printf("   Age %d: Senior\n", age)
    }
}
```

**Output:**
```
╔══════════════════════════════════════════════════════════╗
║           PRODUCTION SWITCH PATTERNS                      ║
╚══════════════════════════════════════════════════════════╝

📊 HTTP Status Handling:
   200: Success ✅
   404: Not Found 🔍
   500: Server Error 🔥

📊 State Machine:
   Order pending → waiting for payment
   Order paid → processing shipment
   Order shipped → tracking available

📊 Expression Switch (replaces if-else chain):
   Age 5: Child
   Age 15: Teenager
   Age 25: Adult
   Age 70: Senior
```

---

## 🆚 Java Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  JAVA                              GO                           │
│  ────                              ──                           │
│                                                                 │
│  switch (day) {                    switch day {                 │
│      case 1:                       case 1:                      │
│          doSomething();                doSomething()            │
│          break;  // Required!          // No break needed!      │
│      case 2:                       case 2:                      │
│          doOther();                    doOther()                │
│          break;                                                 │
│      default:                      default:                     │
│          doDefault();                  doDefault()              │
│  }                                 }                            │
│                                                                 │
│  // Multiple values               // Multiple values           │
│  case 1: case 2: case 3:          case 1, 2, 3:                │
│      doSomething();                   doSomething()            │
│      break;                                                     │
│                                                                 │
│  // No expression switch          // Expression switch         │
│  // Must use if-else              switch {                     │
│                                   case x > 0:                   │
│                                       positive()               │
│                                   case x < 0:                   │
│                                       negative()               │
│                                   }                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Takeaways

1. **No automatic fallthrough** - Go is safer by default
2. **No break needed** - each case stops automatically
3. **Multiple values** per case: `case 1, 2, 3:`
4. **Expression switch** (`switch { case x > 0: }`) replaces if-else chains
5. **Type switch** for interface values: `switch v := x.(type)`
6. **Init statement** supported: `switch x := getValue(); x {`
7. **fallthrough** is explicit and opt-in

---

## ➡️ Next Steps

**Next Topic:** [13 - For Loops](./13-for-loops.md)

