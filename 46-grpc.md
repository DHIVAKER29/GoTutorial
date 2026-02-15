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

| Aspect | REST | gRPC |
|--------|------|------|
| Format | JSON/HTTP (text-based) | Protobuf/HTTP2 (binary) |
| Speed | Slower | ~10x faster |
| Typing | Runtime | Compile-time |
| Streaming | Limited | Bi-directional |
| Code gen | Manual | Automatic |

**gRPC advantages:** Strongly typed, bi-directional streaming, language agnostic, code generation.

**Use gRPC for:** Microservices communication, high-performance APIs, real-time streaming.

---

## 📝 Protocol Buffer Definition

```protobuf
// user.proto
syntax = "proto3";
package user;
option go_package = "myapp/pb/user";

message User {
    int64 id = 1;
    string name = 2;
    string email = 3;
    repeated string roles = 4;
}

message GetUserRequest { int64 id = 1; }
message GetUserResponse { User user = 1; }
message CreateUserRequest { string name = 1; string email = 2; }

service UserService {
    rpc GetUser(GetUserRequest) returns (GetUserResponse);
    rpc CreateUser(CreateUserRequest) returns (User);
    rpc ListUsers(ListUsersRequest) returns (stream User);
}
```

---

## 🛠️ Generate Go Code

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
protoc --go_out=. --go-grpc_out=. user.proto
# Generates: user.pb.go, user_grpc.pb.go
```

---

## 🖥️ Server Implementation

```go
type userServer struct {
    pb.UnimplementedUserServiceServer
    users map[int64]*pb.User
}

func (s *userServer) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    user, exists := s.users[req.Id]
    if !exists {
        return nil, fmt.Errorf("user not found: %d", req.Id)
    }
    return &pb.GetUserResponse{User: user}, nil
}

func main() {
    lis, _ := net.Listen("tcp", ":50051")
    grpcServer := grpc.NewServer()
    pb.RegisterUserServiceServer(grpcServer, NewUserServer())
    grpcServer.Serve(lis)
}
// Output: Server listening on :50051
```

---

## 📱 Client Implementation

```go
conn, _ := grpc.Dial("localhost:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
defer conn.Close()

client := pb.NewUserServiceClient(conn)
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()

user, _ := client.CreateUser(ctx, &pb.CreateUserRequest{Name: "Alice", Email: "alice@example.com"})
resp, _ := client.GetUser(ctx, &pb.GetUserRequest{Id: user.Id})

// Server streaming
stream, _ := client.ListUsers(ctx, &pb.ListUsersRequest{})
for {
    u, err := stream.Recv()
    if err == io.EOF {
        break
    }
    fmt.Printf("Streamed: %+v\n", u)
}
// Output: Created user, got user, streamed users
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
