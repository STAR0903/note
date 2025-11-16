# 基本介绍

[gRPC-Gateway](https://github.com/grpc-ecosystem/grpc-gateway) 是一个 protoc 插件。它读取 gRPC 服务定义并生成一个反向代理服务器，该服务器将 RESTful JSON API 转换为 gRPC。此服务器根据 gRPC 定义中的自定义选项生成。

鉴于复杂的外部环境 gRPC 并不是万能的工具。在某些情况下，我们仍然希望提供传统的 HTTP/JSON API，来满足维护向后兼容性或者那些不支持 gRPC 的客户端。但是为我们的RPC服务再编写另一个服务只是为了对外提供一个 HTTP/JSON API，这是一项相当耗时和乏味的任务。

gRPC-Gateway 能帮助你同时提供 gRPC 和 RESTful 风格的 API。gRPC-Gateway 是 Google protocol buffers 编译器 protoc 的一个插件。它读取 Protobuf 服务定义并生成一个反向代理服务器，该服务器将 RESTful HTTP API 转换为 gRPC。该服务器是根据服务定义中的 google.api.http 注释生成的。

![1763211429126](image/gRPC-Gateway/1763211429126.png)

# 使用示例

首先按照我们之前的gRPC开发步骤完成前置工作。即一个简单的gRPC服务

### gRPC 服务

编写 `.proto`文件  --->  `protoc` 命令生成代码  --->  编写业务逻辑代码。

##### 编写proto代码

```protobuf
syntax = "proto3";
package pb;
option go_package = "add_server/pb";

message addRequest{
  double x = 1;
  double y = 2;
}

message addResponse{
  optional double result = 1;
}

service add{
  rpc Add(addRequest) returns (addResponse);
}
```

##### 生成代码

```bash
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative pb/add.proto
```

生成pb和gRPC相关代码后，在 `main`函数中注册RPC服务并启动gRPC Server。

```go
package main

import (
	"add_server/pb"
	"context"
	"google.golang.org/grpc"
	"google.golang.org/protobuf/proto"
	"log"
	"net"
)

type server struct {
	pb.UnimplementedAddServer
}

func (s server) Add(ctx context.Context, req *pb.AddRequest) (*pb.AddResponse, error) {
	x, y := req.GetX(), req.GetY()
	return &pb.AddResponse{Result: proto.Float64(x + y)}, nil
}
func main() {
	// 监听
	lis, err := net.Listen("tcp", ":8972")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	// 创建服务器
	s := grpc.NewServer()
	// 注册服务
	pb.RegisterAddServer(s, &server{})
	// 启动
	err = s.Serve(lis)
	if err != nil {
		log.Fatalf("failed to server: %v", err)
	}

}
```

此时的文件目录如下：

```bash
add_server
├── go.mod
├── go.sum
├── main.go
└── pb
    ├── add.pb.go
    ├── add.proto
    └── add_grpc.pb.go
```

至此一个简单的gRPC服务就写好了。

接下来我们将介绍如何快速的为gRPC服务生成HTTP API代码。

### gRPC-Gateway

##### 添加注释

现在我们已经有了一个可以运行的 Go gRPC 服务器，接下来需要添加 gRPC-Gateway 注释。这些注释定义了 gRPC 服务如何映射到 JSON 请求和响应。使用 protocol buffers时，每个 RPC 服务必须使用 `google.api.HTTP` 注释来定义 HTTP 方法和路径。

因此，我们需要将 `google/api/http.proto` 导入到 `proto` 文件中。我们还需要添加所需的 HTTP-> gRPC 映射。

修改的内容如下。

```protobuf
import "google/api/annotations.proto";
service add{
  rpc Add(addRequest) returns (addResponse){
    option (google.api.http) = {
      post: "/v1/add"
      body: "*"
    };
  }
}
```

##### 生成stubs

现在我们已经将 gRPC-Gateway 注释添加到 proto 文件中，接下来需要使用 gRPC-Gateway 生成器来生成存根。

###### 引入依赖包

在我们可以使用 `protoc` 生成存根之前，我们需要将一些依赖项复制到我们的 proto 文件结构中。下面两个方法都可以。

将 `googleapis` 的一个子集从[官方库](https://github.com/googleapis/googleapis)复制到您的本地原型文件结构中。拷贝后的目录应该是这样的(这里添加了一级add目录，需要更改相关路径内容):

```bash
greeter
├── go.mod
├── go.sum
├── main.go
└── pb
    ├── google
    │   └── api
    │       ├── annotations.proto
    │       └── http.proto
    └── add
        ├── add.pb.go
        ├── add.proto
        └── add.pb.go
```

也可以在  `.\protoc\include\google` 目录下载[官方库api文件](https://github.com/googleapis/googleapis/tree/master/google/api)。

###### 安装插件

需要安装 `protoc-gen-grpc-gateway`插件来生成对应的 grpc-gateway 代码。

```bash
go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@v2
```

###### 生成代码

现在我们需要将 gRPC-Gateway 生成器添加到 protoc 的调用命令中:

```bash
protoc -I=pb --go_out=pb --go_opt=paths=source_relative --go-grpc_out=pb --go-grpc_opt=paths=source_relative --grpc-gateway_out=pb --grpc-gateway_opt=paths=source_relative pb/add.proto
```

执行上述命令应该会生成一个 `*.gw.pb.go` 文件。

### HTTP代码

我们还需要在 `main.go` 文件中添加和启动 `gRPC-Gateway mux`。按如下代码所示修改我们的 `main`函数。

```go
package main

import (
	"add_server/pb"
	"context"
	"github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
	"google.golang.org/protobuf/proto"
	"log"
	"net"
	"net/http"
)

type server struct {
	pb.UnimplementedAddServer
}

func (s server) Add(ctx context.Context, req *pb.AddRequest) (*pb.AddResponse, error) {
	x, y := req.GetX(), req.GetY()
	return &pb.AddResponse{Result: proto.Float64(x + y)}, nil
}
func main() {
	// 监听
	lis, err := net.Listen("tcp", ":8972")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	// 创建服务器
	s := grpc.NewServer()
	// 注册服务
	pb.RegisterAddServer(s, &server{})
	// 启动
	log.Printf("Serving gRPC on 0.0.0.0:8972")
	go func() {
		log.Fatalf("failed to server: %v", s.Serve(lis))
	}()

	// 创建 gRPC 客户端连接
	// gRPC-Gateway 就是通过它来代理请求（将HTTP请求转为RPC请求）
	conn, err := grpc.NewClient(
		"0.0.0.0:8972",
		grpc.WithTransportCredentials(insecure.NewCredentials()),
	)
	if err != nil {
		log.Fatalf("failed to dail server: %v", err)
	}

	gwmux := runtime.NewServeMux()
	// 注册Add
	err = pb.RegisterAddHandler(context.Background(), gwmux, conn)
	if err != nil {
		log.Fatalf("Failed to register gateway: %v", err)
	}

	gwServer := &http.Server{
		Addr:    ":8080",
		Handler: gwmux,
	}
	// 8080端口提供gRPC-Gateway服务
	log.Println("Serving gRPC-Gateway on http://0.0.0.0:8080")
	log.Fatalln(gwServer.ListenAndServe())
}

```

**注意**

1. 导入的"github.com/grpc-ecosystem/grpc-gateway/v2/runtime"是v2版本。
2. 需要使用单独的goroutine启动gRPC服务。

### 测试

首先启动服务。

```bash
go run main.go
```

然后我们使用 cURL 发送 HTTP 请求:

```bash
curl -X POST -k http://localhost:8080/v1/add -d "{\"x\":22.2,\"y\":44.4}"
```

得到响应结果。

```bash
{"result":66.6}
```

### 同一个端口提供HTTP API和gRPC API

上面的程序在 `8972`端口提供了gRPC API，在 `8080`端口提供了HTTP API。但是在有些场景下我们可能希望由同一个端口同时提供gRPC API和HTTP API两种服务，由请求方来决定具体使用哪个协议。

下面的代码将同时在本机的 `8972`端口对外提供gRPC API和HTTP API服务。

因为我们的示例中没有启用 TLS加密通信，所以这里使用 `h2c`包实现对HTTP/2的支持。h2c 协议是 HTTP/2的非 TLS 版本。

```
func ListeningOnSamePorts() {
	// 监听
	lis, err := net.Listen("tcp", ":8972")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	// 创建gRPC服务器
	s := grpc.NewServer()
	// 注册服务
	pb.RegisterAddServer(s, &server{})
	reflection.Register(s)

	gwmux := runtime.NewServeMux()
	// 注册Add
	err = pb.RegisterAddHandlerServer(context.Background(), gwmux, &server{})
	if err != nil {
		log.Fatalf("Failed to register gateway: %v", err)
	}

	mux := http.NewServeMux()
	mux.Handle("/", gwmux)

	// 8080端口提供gRPC-Gateway服务
	log.Println("Serving gRPC-Gateway on http://127.0.0.1:8080")
	log.Fatalln(http.Serve(lis, grpcHandlerFunc(s, mux)))
}

// grpcHandlerFunc 将gRPC请求和HTTP请求分别调用不同的handler处理
func grpcHandlerFunc(grpcServer *grpc.Server, otherHandler http.Handler) http.Handler {
	return h2c.NewHandler(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		if r.ProtoMajor == 2 && strings.Contains(r.Header.Get("Content-Type"), "application/grpc") {
			grpcServer.ServeHTTP(w, r)
		} else {
			otherHandler.ServeHTTP(w, r)
		}
	}), &http2.Server{})
}
```

或者使用连接多路复用的库 `cmux`的无 TLS加密通信用法。

```go
func ListeningOnSamePorts() {
	// 1.监听
	lis, err := net.Listen("tcp", "0.0.0.0:8972")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	defer lis.Close()

	// 2.创建 cmux 多路复用器
	mux := cmux.New(lis)

	// 3. 定义匹配规则
	// 匹配 gRPC 流量 (HTTP/2 + 特定的 Content-Type)
	grpcL := mux.MatchWithWriters(
		cmux.HTTP2MatchHeaderFieldSendSettings("content-type", "application/grpc"),
	)
	// 匹配 HTTP/1.1 流量
	httpL := mux.Match(cmux.HTTP1Fast())

	// 4.创建gRPC服务器
	grpcServer := grpc.NewServer()
	// 注册服务
	pb.RegisterAddServer(grpcServer, &server{})
	reflection.Register(grpcServer)

	// 5. 创建 HTTP 服务器
	gwmux := runtime.NewServeMux()
	// 注册Add
	err = pb.RegisterAddHandlerServer(context.Background(), gwmux, &server{})
	if err != nil {
		log.Fatalf("Failed to register gateway: %v", err)
	}

	httpMux := http.NewServeMux()
	httpMux.Handle("/", gwmux)
	httpServer := &http.Server{Handler: httpMux}

	// 6. 启动服务
	go grpcServer.Serve(grpcL)
	go httpServer.Serve(httpL)

	// 7. 开始多路复用
	log.Printf("Serving on http://127.0.0.1:8972")
	if err := mux.Serve(); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}

}
```

将上述代码编译后运行，在 `8091`端口启动。 测试gRPC API：

```bash
grpcurl -d "{\"x\":22.2,\"y\":44.4}" -plaintext 127.0.0.1:8972 pb.add.Add
```

得到响应结果。

```bash
{
  "result": 66.6
}
```

测试HTTP API：

```bash
curl -X POST -k http://localhost:8972/v1/add -d "{\"x\":22.2,\"y\":44.4}"
```

得到响应结果。

```bash
{"result":66.6}
```
