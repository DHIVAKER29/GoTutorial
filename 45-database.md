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
    "log"
    "time"

    _ "github.com/go-sql-driver/mysql"
)

func main() {
    dsn := "user:password@tcp(localhost:3306)/dbname?parseTime=true"
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        log.Fatalf("Error opening database: %v", err)
    }
    defer db.Close()

    if err := db.Ping(); err != nil {
        log.Fatalf("Error connecting: %v", err)
    }

    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(5)
    db.SetConnMaxLifetime(5 * time.Minute)
    db.SetConnMaxIdleTime(1 * time.Minute)

    stats := db.Stats()
    // stats.OpenConnections shows pool status
}

// Output: Connected (stats vary by connection state)
```

---

## 📝 CRUD Operations

```go
// INSERT
result, err := db.ExecContext(ctx,
    "INSERT INTO users (name, email) VALUES (?, ?)",
    "Alice", "alice@example.com",
)
lastID, _ := result.LastInsertId()
rowsAffected, _ := result.RowsAffected()
// Output: lastID=1, rowsAffected=1
```

```go
// SELECT single row
var user User
err := db.QueryRowContext(ctx,
    "SELECT id, name, email, created_at FROM users WHERE id = ?",
    1,
).Scan(&user.ID, &user.Name, &user.Email, &user.CreatedAt)
if err == sql.ErrNoRows {
    // No user found
}
```

```go
// SELECT multiple rows
rows, err := db.QueryContext(ctx, "SELECT id, name FROM users LIMIT 10")
defer rows.Close()
for rows.Next() {
    var u User
    rows.Scan(&u.ID, &u.Name)
    users = append(users, u)
}
if err := rows.Err(); err != nil {
    // Iteration error
}
```

```go
// UPDATE and DELETE
db.ExecContext(ctx, "UPDATE users SET name = ? WHERE id = ?", "Alice Smith", 1)
db.ExecContext(ctx, "DELETE FROM users WHERE id = ?", 1)
```

---

## 🔒 Prepared Statements

```go
stmt, err := db.PrepareContext(ctx, "INSERT INTO users (name, email) VALUES (?, ?)")
defer stmt.Close()

for _, u := range []struct{ name, email string }{
    {"Alice", "alice@example.com"},
    {"Bob", "bob@example.com"},
} {
    stmt.ExecContext(ctx, u.name, u.email)
}

// SQL Injection Prevention:
// ❌ NEVER: fmt.Sprintf("SELECT * WHERE name='%s'", userInput)
// ✅ ALWAYS: db.Query("SELECT * WHERE name=?", userInput)
```

---

## 💰 Transactions

```go
func transferMoney(ctx context.Context, db *sql.DB, fromID, toID int64, amount float64) error {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer tx.Rollback()

    _, err = tx.ExecContext(ctx, "UPDATE accounts SET balance = balance - ? WHERE id = ?", amount, fromID)
    if err != nil {
        return err
    }
    _, err = tx.ExecContext(ctx, "UPDATE accounts SET balance = balance + ? WHERE id = ?", amount, toID)
    if err != nil {
        return err
    }
    return tx.Commit()
}

// Transaction pattern:
// 1. tx, err := db.BeginTx(ctx, nil)
// 2. defer tx.Rollback()  // Safety net
// 3. Execute queries with tx
// 4. tx.Commit() on success
```

---

## 🏭 Production Pattern

```go
type UserRepository struct {
    db *sql.DB
}

func (r *UserRepository) FindByID(ctx context.Context, id int64) (*User, error) {
    var user User
    err := r.db.QueryRowContext(ctx, "SELECT id, name, email FROM users WHERE id = ?", id).
        Scan(&user.ID, &user.Name, &user.Email)
    if err == sql.ErrNoRows {
        return nil, nil
    }
    return &user, err
}

func (r *UserRepository) Create(ctx context.Context, user *User) error {
    result, err := r.db.ExecContext(ctx, "INSERT INTO users (name, email) VALUES (?, ?)", user.Name, user.Email)
    if err != nil {
        return err
    }
    user.ID, _ = result.LastInsertId()
    return nil
}
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
