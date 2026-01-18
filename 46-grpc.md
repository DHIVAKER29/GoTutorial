# 46 - gRPC

> Building high-performance RPC services with Protocol Buffers.

---

## 📌 What You'll Learn

- What gRPC is and when to use it
- Protocol Buffers (protobuf)
- Defining services
- Client and server implementation
- Streaming

---

## 🤔 What Is gRPC?

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  REST vs gRPC                                                   │
│                                                                 │
│  REST:                                                          │
│  ┌──────────┐    JSON/HTTP    ┌──────────┐                     │
│  │  Client  │ ───────────────► │  Server  │                     │
│  └──────────┘   Text-based    └──────────┘                     │
│                 Slower                                          │
│                                                                 │
│  gRPC:                                                          │
│  ┌──────────┐   Protobuf/HTTP2  ┌──────────┐                   │
│  │  Client  │ ─────────────────► │  Server  │                   │
│  └──────────┘   Binary, Fast    └──────────┘                   │
│                 Streaming                                       │
│                                                                 │
│  gRPC ADVANTAGES:                                               │
│  • 10x faster than REST/JSON                                    │
│  • Strongly typed (compile-time checks)                         │
│  • Bi-directional streaming                                     │
│  • Code generation                                              │
│  • Language agnostic                                            │
│                                                                 │
│  USE gRPC FOR:                                                  │
│  • Microservices communication                                  │
│  • High-performance APIs                                        │
│  • Real-time streaming                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Protocol Buffer Definition

```protobuf
// user.proto
syntax = "proto3";

package user;

option go_package = "myapp/pb/user";

// Message definitions
message User {
    int64 id = 1;
    string name = 2;
    string email = 3;
    repeated string roles = 4;
}

message GetUserRequest {
    int64 id = 1;
}

message GetUserResponse {
    User user = 1;
}

message CreateUserRequest {
    string name = 1;
    string email = 2;
}

message ListUsersRequest {
    int32 page_size = 1;
    string page_token = 2;
}

message ListUsersResponse {
    repeated User users = 1;
    string next_page_token = 2;
}

// Service definition
service UserService {
    // Unary RPC
    rpc GetUser(GetUserRequest) returns (GetUserResponse);
    rpc CreateUser(CreateUserRequest) returns (User);
    
    // Server streaming
    rpc ListUsers(ListUsersRequest) returns (stream User);
    
    // Client streaming
    rpc BatchCreateUsers(stream CreateUserRequest) returns (ListUsersResponse);
    
    // Bi-directional streaming
    rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}

message ChatMessage {
    string text = 1;
}
```

---

## 🛠️ Generate Go Code

```bash
# Install tools
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# Generate code
protoc --go_out=. --go-grpc_out=. user.proto

# This generates:
#   user.pb.go       - Message types
#   user_grpc.pb.go  - Client and server interfaces
```

---

## 🖥️ Server Implementation

```go
// server.go
package main

import (
    "context"
    "fmt"
    "log"
    "net"
    
    "google.golang.org/grpc"
    pb "myapp/pb/user"
)

// Implement the UserServiceServer interface
type userServer struct {
    pb.UnimplementedUserServiceServer  // Required for forward compatibility
    users map[int64]*pb.User
}

func NewUserServer() *userServer {
    return &userServer{
        users: make(map[int64]*pb.User),
    }
}

// Unary RPC implementation
func (s *userServer) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    user, exists := s.users[req.Id]
    if !exists {
        return nil, fmt.Errorf("user not found: %d", req.Id)
    }
    return &pb.GetUserResponse{User: user}, nil
}

func (s *userServer) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.User, error) {
    user := &pb.User{
        Id:    int64(len(s.users) + 1),
        Name:  req.Name,
        Email: req.Email,
    }
    s.users[user.Id] = user
    return user, nil
}

// Server streaming implementation
func (s *userServer) ListUsers(req *pb.ListUsersRequest, stream pb.UserService_ListUsersServer) error {
    for _, user := range s.users {
        if err := stream.Send(user); err != nil {
            return err
        }
    }
    return nil
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }
    
    grpcServer := grpc.NewServer()
    pb.RegisterUserServiceServer(grpcServer, NewUserServer())
    
    log.Println("gRPC server running on :50051")
    if err := grpcServer.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

---

## 📱 Client Implementation

```go
// client.go
package main

import (
    "context"
    "fmt"
    "io"
    "log"
    "time"
    
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    pb "myapp/pb/user"
)

func main() {
    // Connect to server
    conn, err := grpc.Dial("localhost:50051",
        grpc.WithTransportCredentials(insecure.NewCredentials()),
    )
    if err != nil {
        log.Fatalf("failed to connect: %v", err)
    }
    defer conn.Close()
    
    client := pb.NewUserServiceClient(conn)
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    // Create user
    user, err := client.CreateUser(ctx, &pb.CreateUserRequest{
        Name:  "Alice",
        Email: "alice@example.com",
    })
    if err != nil {
        log.Fatalf("CreateUser failed: %v", err)
    }
    fmt.Printf("Created: %+v\n", user)
    
    // Get user
    resp, err := client.GetUser(ctx, &pb.GetUserRequest{Id: user.Id})
    if err != nil {
        log.Fatalf("GetUser failed: %v", err)
    }
    fmt.Printf("Got: %+v\n", resp.User)
    
    // Server streaming
    stream, err := client.ListUsers(ctx, &pb.ListUsersRequest{})
    if err != nil {
        log.Fatalf("ListUsers failed: %v", err)
    }
    
    for {
        user, err := stream.Recv()
        if err == io.EOF {
            break
        }
        if err != nil {
            log.Fatalf("stream error: %v", err)
        }
        fmt.Printf("Streamed: %+v\n", user)
    }
}
```

---

## 🎯 Key Takeaways

1. **Protocol Buffers** define messages and services
2. **protoc** generates Go code
3. **Unary RPC** = single request/response
4. **Streaming** = server, client, or bi-directional
5. **HTTP/2** enables multiplexing
6. **10x faster** than REST/JSON

---

## ➡️ Next Steps

**Next Topic:** [47 - Profiling and Performance](./47-profiling.md)

