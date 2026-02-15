# 66 - Go Commands Cheatsheet

> Complete reference for all Go CLI commands with explanations.

---

## 📋 Quick Reference

```bash
go run main.go
go build
go test ./...
go mod tidy
go fmt ./...
```

---

## 🏗️ Build & Run

| Command | Purpose |
|---------|---------|
| `go run main.go` | Compile and execute |
| `go build` | Create binary |
| `go build -o myapp` | Specify output name |
| `go build -race` | Include race detector |
| `go build -ldflags="-s -w"` | Strip debug info |
| `go install` | Build and install to GOBIN |
| `go clean -modcache` | Clear module cache |

**Cross-compile:** `GOOS=linux GOARCH=amd64 go build`

---

## 📦 Module Commands

```bash
go mod init github.com/user/project
go mod tidy
go mod download
go mod verify
go mod graph
go mod why github.com/pkg/errors
go mod edit -require pkg@v1.0.0
go mod vendor
```

```bash
go get pkg@latest
go get pkg@v1.9.1
go get -u pkg
go get pkg@none  # Remove
```

---

## 🧪 Testing

```bash
go test
go test -v
go test -run TestName
go test -race
go test -cover
go test -coverprofile=cover.out
go test -bench=.
go test -benchmem
go test -cpuprofile=cpu.out -bench=.
```

---

## 🔍 Analysis

```bash
go vet ./...
go fmt ./...
gofmt -w .
go doc fmt
go doc fmt.Println
```

---

## 🛠️ Tool Commands

```bash
go tool compile -S main.go
go tool compile -m main.go
go tool pprof cpu.out
go tool cover -html=cover.out
go generate ./...
```

---

## 🔧 Environment

```bash
go env
go env GOPATH
go env -w GOPRIVATE=github.com/myorg/*
go version
```

---

## 📊 Profiling Flags

```bash
go build -gcflags="-m"
go build -gcflags="-S"
go build -gcflags="-N -l"
GOSSAFUNC=main go build
```

---

## 🚀 Common Workflows

**New project:** `go mod init` → write code → `go mod tidy` → `go build`

**Add dep:** `go get pkg@latest` → `go mod tidy`

**Production build:** `CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w -X main.Version=1.0.0"`

**Code quality:** `go fmt ./...` → `go vet ./...` → `go test -race ./...`

---

## 📝 Quick Reference Table

| Category | Commands |
|----------|----------|
| **Build & Run** | go run, go build, go install |
| **Modules** | go mod init, go mod tidy, go get |
| **Testing** | go test, go test -race, go test -cover, go test -bench=. |
| **Quality** | go fmt, go vet, go doc |
| **Environment** | go env, go version, go list -m all |

---

## 🎉 You now have a complete Go command reference!

Print this cheatsheet or keep it handy while developing! 🚀
