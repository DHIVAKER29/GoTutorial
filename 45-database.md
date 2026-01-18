# 45 - Database Access

> Working with SQL databases in Go using database/sql.

---

## 📌 What You'll Learn

- database/sql package basics
- Connection pooling
- CRUD operations
- Prepared statements
- Transactions
- Best practices

---

## 🔌 Database Connection

```go
// db_connection.go
package main

import (
    "database/sql"
    "fmt"
    "log"
    "time"
    
    _ "github.com/go-sql-driver/mysql"  // MySQL driver
    // _ "github.com/lib/pq"            // PostgreSQL driver
    // _ "github.com/mattn/go-sqlite3"  // SQLite driver
)

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           DATABASE CONNECTION                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Connection string (DSN)
    dsn := "user:password@tcp(localhost:3306)/dbname?parseTime=true"
    
    // Open connection pool (doesn't actually connect yet!)
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        log.Fatalf("Error opening database: %v", err)
    }
    defer db.Close()
    
    // Actually test connection
    if err := db.Ping(); err != nil {
        log.Fatalf("Error connecting: %v", err)
    }
    fmt.Println("   Connected to database!")
    
    // Configure connection pool
    db.SetMaxOpenConns(25)                  // Max open connections
    db.SetMaxIdleConns(5)                   // Max idle connections
    db.SetConnMaxLifetime(5 * time.Minute)  // Max connection lifetime
    db.SetConnMaxIdleTime(1 * time.Minute)  // Max idle time
    
    // Get pool stats
    stats := db.Stats()
    fmt.Printf("   Open connections: %d\n", stats.OpenConnections)
}
```

---

## 📝 CRUD Operations

```go
// db_crud.go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "time"
)

type User struct {
    ID        int64
    Name      string
    Email     string
    CreatedAt time.Time
}

func main() {
    // Assume db is connected...
    var db *sql.DB
    ctx := context.Background()
    
    // CREATE
    fmt.Println("📊 INSERT:")
    result, err := db.ExecContext(ctx,
        "INSERT INTO users (name, email) VALUES (?, ?)",
        "Alice", "alice@example.com",
    )
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    }
    
    lastID, _ := result.LastInsertId()
    rowsAffected, _ := result.RowsAffected()
    fmt.Printf("   Inserted ID: %d, Rows: %d\n", lastID, rowsAffected)
    
    // READ (single row)
    fmt.Println("\n📊 SELECT (single):")
    var user User
    err = db.QueryRowContext(ctx,
        "SELECT id, name, email, created_at FROM users WHERE id = ?",
        1,
    ).Scan(&user.ID, &user.Name, &user.Email, &user.CreatedAt)
    
    if err == sql.ErrNoRows {
        fmt.Println("   No user found")
    } else if err != nil {
        fmt.Printf("   Error: %v\n", err)
    } else {
        fmt.Printf("   User: %+v\n", user)
    }
    
    // READ (multiple rows)
    fmt.Println("\n📊 SELECT (multiple):")
    rows, err := db.QueryContext(ctx,
        "SELECT id, name, email, created_at FROM users LIMIT 10",
    )
    if err != nil {
        fmt.Printf("   Error: %v\n", err)
    }
    defer rows.Close()  // IMPORTANT: Always close!
    
    var users []User
    for rows.Next() {
        var u User
        if err := rows.Scan(&u.ID, &u.Name, &u.Email, &u.CreatedAt); err != nil {
            fmt.Printf("   Scan error: %v\n", err)
            continue
        }
        users = append(users, u)
    }
    
    // Check for iteration errors
    if err := rows.Err(); err != nil {
        fmt.Printf("   Iteration error: %v\n", err)
    }
    
    fmt.Printf("   Found %d users\n", len(users))
    
    // UPDATE
    fmt.Println("\n📊 UPDATE:")
    result, err = db.ExecContext(ctx,
        "UPDATE users SET name = ? WHERE id = ?",
        "Alice Smith", 1,
    )
    
    // DELETE
    fmt.Println("\n📊 DELETE:")
    result, err = db.ExecContext(ctx,
        "DELETE FROM users WHERE id = ?",
        1,
    )
}
```

---

## 🔒 Prepared Statements

```go
// db_prepared.go
package main

import (
    "context"
    "database/sql"
    "fmt"
)

func main() {
    var db *sql.DB
    ctx := context.Background()
    
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           PREPARED STATEMENTS                             ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    // Prepare statement (reusable, prevents SQL injection)
    stmt, err := db.PrepareContext(ctx,
        "INSERT INTO users (name, email) VALUES (?, ?)",
    )
    if err != nil {
        fmt.Printf("   Prepare error: %v\n", err)
        return
    }
    defer stmt.Close()
    
    // Execute multiple times efficiently
    users := []struct{ name, email string }{
        {"Alice", "alice@example.com"},
        {"Bob", "bob@example.com"},
        {"Charlie", "charlie@example.com"},
    }
    
    for _, u := range users {
        _, err := stmt.ExecContext(ctx, u.name, u.email)
        if err != nil {
            fmt.Printf("   Insert error: %v\n", err)
            continue
        }
        fmt.Printf("   Inserted: %s\n", u.name)
    }
    
    // SQL Injection Prevention
    fmt.Println("\n📊 SQL Injection Prevention:")
    fmt.Println("   ❌ NEVER: fmt.Sprintf(\"SELECT * WHERE name='%s'\", userInput)")
    fmt.Println("   ✅ ALWAYS: db.Query(\"SELECT * WHERE name=?\", userInput)")
}
```

---

## 💰 Transactions

```go
// db_transactions.go
package main

import (
    "context"
    "database/sql"
    "fmt"
)

func transferMoney(ctx context.Context, db *sql.DB, fromID, toID int64, amount float64) error {
    // Begin transaction
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return fmt.Errorf("begin tx: %w", err)
    }
    
    // Defer rollback (no-op if committed)
    defer tx.Rollback()
    
    // Debit from account
    _, err = tx.ExecContext(ctx,
        "UPDATE accounts SET balance = balance - ? WHERE id = ?",
        amount, fromID,
    )
    if err != nil {
        return fmt.Errorf("debit: %w", err)
    }
    
    // Credit to account
    _, err = tx.ExecContext(ctx,
        "UPDATE accounts SET balance = balance + ? WHERE id = ?",
        amount, toID,
    )
    if err != nil {
        return fmt.Errorf("credit: %w", err)
    }
    
    // Commit transaction
    if err := tx.Commit(); err != nil {
        return fmt.Errorf("commit: %w", err)
    }
    
    return nil
}

func main() {
    fmt.Println("╔══════════════════════════════════════════════════════════╗")
    fmt.Println("║           TRANSACTIONS                                    ║")
    fmt.Println("╚══════════════════════════════════════════════════════════╝")
    
    fmt.Println("\n📊 Transaction Pattern:")
    fmt.Println("   1. tx, err := db.BeginTx(ctx, nil)")
    fmt.Println("   2. defer tx.Rollback()  // Safety net")
    fmt.Println("   3. Execute queries with tx")
    fmt.Println("   4. tx.Commit() on success")
}
```

---

## 🏭 Production Pattern

```go
// db_production.go
package main

import (
    "context"
    "database/sql"
    "fmt"
)

// Repository pattern
type UserRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) *UserRepository {
    return &UserRepository{db: db}
}

func (r *UserRepository) FindByID(ctx context.Context, id int64) (*User, error) {
    query := `SELECT id, name, email, created_at FROM users WHERE id = ?`
    
    var user User
    err := r.db.QueryRowContext(ctx, query, id).
        Scan(&user.ID, &user.Name, &user.Email, &user.CreatedAt)
    
    if err == sql.ErrNoRows {
        return nil, nil  // Not found
    }
    if err != nil {
        return nil, fmt.Errorf("query user: %w", err)
    }
    
    return &user, nil
}

func (r *UserRepository) Create(ctx context.Context, user *User) error {
    query := `INSERT INTO users (name, email) VALUES (?, ?)`
    
    result, err := r.db.ExecContext(ctx, query, user.Name, user.Email)
    if err != nil {
        return fmt.Errorf("insert user: %w", err)
    }
    
    id, err := result.LastInsertId()
    if err != nil {
        return fmt.Errorf("get last id: %w", err)
    }
    
    user.ID = id
    return nil
}

// Type alias for clarity
type User struct {
    ID        int64
    Name      string
    Email     string
    CreatedAt time.Time
}

import "time"
```

---

## 🎯 Key Takeaways

1. **sql.Open** doesn't connect - use **Ping** to verify
2. **Always close** rows and statements
3. **Use placeholders** (?, $1) - never string concat
4. **Prepared statements** for repeated queries
5. **Transactions** for atomic operations
6. **Repository pattern** for clean architecture

---

## ➡️ Next Steps

**Next Topic:** [46 - gRPC](./46-grpc.md)

