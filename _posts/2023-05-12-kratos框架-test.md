---
title: Kratos框架
description: Kratos框架教程
categories:
- go
tags:
- Kratos
---

<br>


[>>参考 《go微服务框架Kratos 》连载](https://segmentfault.com/a/1190000043365097?utm_source=sf-backlinks)



### 安装, 搭建过程

`Kratos` 是一套轻量级 Go 微服务框架，包含大量微服务相关框架及工具，下面介绍一下Kratos的安装启动过程。


#### 安装 `go` mac 环境



方法一：pkg包安装

    该安装包会将Go发行版安装到 /usr/local/go 中

方法二：homebrew安装

    brew install go
    brew list go可以查看go的安装目录
    
    查看是否安装成功
    go --version
    查看环境信息
    go env
    开启go mod
    go env -w GO111MODULE=on

配置环境变量

    GOROOT路径是go的安装路径，一般是/usr/local/go或者 /usr/local/Cellar/go/1.18/libexec
    GOPATH是Go项目的工作目录，可以自定义，这里指定为$HOME/go
    GOPROXY：go get 下载依赖时使用的代理地址列表，使用逗号 (,) 或竖杠 (|) 分隔。

    将$GOPATH/bin加入 $PATH 变量，这样在终端的任何路径都能使用go包的bin目录下面的工具
        
        export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
        source ~/.zshrc　　# 如果是zsh


#### 安装protoc、protoc-gen-go

    gRPC 是一个高性能、开源、通用的RPC框架，由Google推出，基于HTTP2协议标准设计开发，默认采用Protocol Buffers数据序列化协议，支持多种开发语言。
    
    Protocol Buffers（Protobuf）是一种与语言、平台无关，可扩展的序列化结构化数据的数据描述语言，我们常常称其为 IDL，常用于通信协议，数据存储等等，相较于 JSON、XML，它更小、更快。
    
    在 gRPC 开发中，我们常常需要与 Protobuf 进行打交道，而在编写了.proto 文件后，我们会需要到一个编译器，那就是 protoc，protoc 是 Protobuf 的编译器，是用 C++ 所编写的，其主要功能是用于编译.proto 文件。

    针对不同的语言，还需要不同的运行时的 protoc 插件，那么对应 Go 语言就是 protoc-gen-go 插件

安装

    brew install grpc
    brew install protobuf
    brew install protoc-gen-go
    brew install protoc-gen-go-grpc

验证

    protoc --version
    protoc-gen-go --version
    protoc-gen-go-grpc --version

#### 安装`kratos`脚手架工具

`kratos` 是与 `Kratos 框架`配套的脚手架工具，kratos 能够
    
    - 通过模板快速创建项目
    - 快速创建与生成 protoc 文件
    - 使用开发过程中常用的命令

    go install github.com/go-kratos/kratos/cmd/kratos/v2@latest

    该命令会将编译的中间文件放在 GOPATH 的 pkg 目录下，以及将编译的可执行文件放在 GOPATH 的 bin 目录下。



### `kratos` 框架使用



#### 创建项目



    kratos new helloworld

    # 国内环境拉项目模板会失败 换个源
    kratos new helloworld -r https://gitee.com/go-kratos/kratos-layout.git


查看目录结构
    
    ➜ cd helloworld
    ➜ tree


``` 
├── Dockerfile
├── LICENSE
├── Makefile
├── README.md
├── api
│   └── helloworld
│       └── v1
│           ├── error_reason.pb.go
│           ├── error_reason.proto
│           ├── greeter.pb.go
│           ├── greeter.proto
│           ├── greeter_grpc.pb.go
│           └── greeter_http.pb.go
├── cmd
│   └── helloworld
│       ├── main.go
│       ├── wire.go
│       └── wire_gen.go
├── configs
│   └── config.yaml
├── go.mod
├── go.sum
├── internal
│   ├── biz
│   │   ├── README.md
│   │   ├── biz.go
│   │   └── greeter.go
│   ├── conf
│   │   ├── conf.pb.go
│   │   └── conf.proto
│   ├── data
│   │   ├── README.md
│   │   ├── data.go
│   │   └── greeter.go
│   ├── server
│   │   ├── grpc.go
│   │   ├── http.go
│   │   └── server.go
│   └── service
│       ├── README.md
│       ├── greeter.go
│       └── service.go
├── openapi.yaml
└── third_party
    ├── README.md
    ├── errors
    │   └── errors.proto
    ├── google
    │   ├── api
    │   │   ├── annotations.proto
    │   │   ├── client.proto
    │   │   ├── field_behavior.proto
    │   │   ├── http.proto
    │   │   └── httpbody.proto
    │   └── protobuf
    │       ├── any.proto
    │       ├── api.proto
    │       ├── compiler
    │       │   └── plugin.proto
    │       ├── descriptor.proto
    │       ├── duration.proto
    │       ├── empty.proto
    │       ├── field_mask.proto
    │       ├── source_context.proto
    │       ├── struct.proto
    │       ├── timestamp.proto
    │       ├── type.proto
    │       └── wrappers.proto
    ├── openapi
    │   └── v3
    │       ├── annotations.proto
    │       └── openapi.proto
    └── validate
        ├── README.md
        └── validate.proto

```


#### 目录结构介绍


`Makefile` 

    项目的编译、运行、测试、构建等命令都在 Makefile 中定义，通过 make 命令来执行。 


> make init


      init:
      go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
      go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
      go install github.com/go-kratos/kratos/cmd/kratos/v2@latest
      go install github.com/go-kratos/kratos/cmd/protoc-gen-go-http/v2@latest
      go install github.com/google/gnostic/cmd/protoc-gen-openapi@latest
      go install github.com/google/wire/cmd/wire@latest

会初始化安装框架的依赖

如果之后自己还有一些工具之类的需要安装，可以放到这里



> make config 


    config:
    protoc --proto_path=./internal \
    --proto_path=./third_party \
    --go_out=paths=source_relative:./internal \
    $(INTERNAL_PROTO_FILES)

会生成 `internal/conf/conf.pb.go` 文件，提供对 `internal/conf/conf.proto`文件的调用方法。

`internal/conf/conf.proto` 文件对应的配置文件为 `configs/config.yaml`文件


> make api 


    api:
    protoc --proto_path=./api \
    --proto_path=./third_party \
    --go_out=paths=source_relative:./api \
    --go-http_out=paths=source_relative:./api \
    --go-grpc_out=paths=source_relative:./api \
    --openapi_out=fq_schema_naming=true,default_response=false:. \
    $(API_PROTO_FILES)

主要用来把 api目录里的定义的proto文件生成对应的go文件。

比如示例的`api/helloworld/v1/greeter.proto`文件生成 `greeter.pb.go`文件。



> make generate

生成`wire`依赖注入文件


    generate:
    go mod tidy
    go get github.com/google/wire/cmd/wire@latest
    go generate ./...

主要用来把 `helloword `这个项目里的 `cmd/helloword/wire.go`文件生成
`cmd/helloword/wire_gen.go`依赖注入文件。


> make build
  
  
    build:
    mkdir -p bin/ && go build -ldflags "-X main.Version=$(VERSION)" -o ./bin/ ./...



若遇到： `missing go.sum entry` 异常， 则执行 `go mod tidy`



生成可执行编译文件, 我们执行一下，会在 `bin`目录下生成一个 `helloworld可执行文件`


    执行 

    ./bin/helloworld    -conf ./configs/config.yaml


成功启动:

``` 
2023/06/06 17:29:57 maxprocs: Leaving GOMAXPROCS=8: CPU quota undefined
DEBUG msg=config loaded: config.yaml format: yaml
INFO ts=2023-06-06T17:29:57+08:00 caller=http/server.go:302 service.id=bogon service.name= service.version= trace.id= span.id= msg=[HTTP] server listening on: [::]:8000
INFO ts=2023-06-06T17:29:57+08:00 caller=grpc/server.go:205 service.id=bogon service.name= service.version= trace.id= span.id= msg=[gRPC] server listening on: [::]:9000
```


#### 项目运行

启动：

``` 
go run ./cmd/helloworld -conf configs/config.yaml

或者

./bin/helloworld  -conf ./configs/config.yaml

或者

kratos run 
```


请求示例：
  

    http://localhost:8000/helloworld/1


返回结果：
    
    {
       "message": "Hello 1"
    }


#### 项目调整 （增加接口）


> `GET`

1. 修改 `greeter.proto`文件

加一个接口SayHi如下：
``` 
// Sends a hi
rpc SayHi (HelloRequest) returns (HelloReply) {
option (google.api.http) = {
get: "/hi/{name}"
};
}
```

2.  执行`make api`，生产api


3. 实现 api接口 `SayHi()`

``` 
// SayHi implements helloworld.GreeterServer.
func (s *GreeterService) SayHi(ctx context.Context, in *v1.HelloRequest) (*v1.HelloReply, error) {
    g, err := s.uc.CreateGreeter(ctx, &biz.Greeter{Hello: in.Name})
    if err != nil {
        return nil, err
    }
    return &v1.HelloReply{Message: "hi " + g.Hello}, nil
}
```

4. 重新运行， 访问 `http://localhost:8000/hi/1`


    go run ./cmd/helloworld  -conf  configs/config.yaml


    http://localhost:8000/hi/1 


    返回结果：
    
        {
           "message": "hi 1"
        }



> `POST`


1. 修改 `greeter.proto`文件

加一个接口SayPost如下：
``` 
  rpc SayPost (HelloRequest) returns (HelloReply) {
    option (google.api.http) = {
      post: "/sayPost"
      body: "*"
    };
  };
```

2.  执行`make api`，生产api


3. 实现 api接口 `SayPost()`

``` 
func (s *GreeterService) SayPost(ctx context.Context, in *v1.HelloRequest) (*v1.HelloReply, error) {
	g, err := s.uc.CreateGreeter(ctx, &biz.Greeter{Hello: in.Name})
	if err != nil {
		return nil, err
	}
	return &v1.HelloReply{Message: "Post " + g.Hello}, nil
}
```

4. 重新运行， 访问 `http://localhost:8000/sayPost`


    go run ./cmd/helloworld  -conf  configs/config.yaml


    http://localhost:8000/sayPost  -d {"name": "jimmy"}


    返回结果：
    
        {
          "message": "Post jimmy"
      }





#### 新建项目接口


> 生成一个`user.proto`


    kratos proto add api/helloworld/v1/user.proto

``` 
syntax = "proto3";

package api.helloworld.v1;

option go_package = "helloworld/api/helloworld/v1;v1";
option java_multiple_files = true;
option java_package = "api.helloworld.v1";

service User {
	rpc CreateUser (CreateUserRequest) returns (CreateUserReply);
	rpc UpdateUser (UpdateUserRequest) returns (UpdateUserReply);
	rpc DeleteUser (DeleteUserRequest) returns (DeleteUserReply);
	rpc GetUser (GetUserRequest) returns (GetUserReply);
	rpc ListUser (ListUserRequest) returns (ListUserReply);
}

message CreateUserRequest {}
message CreateUserReply {}

message UpdateUserRequest {}
message UpdateUserReply {}

message DeleteUserRequest {}
message DeleteUserReply {}

message GetUserRequest {}
message GetUserReply {}

message ListUserRequest {}
message ListUserReply {}
```

    生成了user的增删改查，接口
    
    CreateUser() 表示增
    DeleteUser() 表示删
    UpdateUser() 表示修改
    GetUser() 查询一条
    ListUser() 查询多条



> 修改`user.proto`文件，定义restful路由

先引入proto的http的类库

    import "google/api/annotations.proto";



修改 `CreateUser()`的http api 路由

我们把源码的

    rpc CreateUser (CreateUserRequest) returns (CreateUserReply);
修改如下：
    
    rpc CreateUser (CreateUserRequest) returns (CreateUserReply){
    option (google.api.http) = {
    post: "/user",
    body: "*",
    };
    };


修改 `DeleteUser()`的http api 路由
我们把源码的
    
    rpc DeleteUser (DeleteUserRequest) returns (DeleteUserReply);
修改如下：

    rpc DeleteUser (DeleteUserRequest) returns (DeleteUserReply){
    option (google.api.http) = {
    delete: "/user/{id}",
    };
    };

因为这里接受了 id 参数，所以我们需要定义下`DeleteUserRequest`

把源码的
    
    message DeleteUserRequest {}
修改如下：
    
    message DeleteUserRequest {
    int64 id = 1;
    }


...


其他的接口，我们也可以按照这个思路去修改


> 执行`make api`，生成对应的api接口


    在 api/helloworld/v1 目录下，生成了好几个文件如下：
    
    user.pb.go
    user_grpc.pb.go
    user_http.pb.


> 生成service，并注入到 kratos 中的http或者grpc服务中

使用官方自带的 proto server 工具来生成service

    kratos proto server api/helloworld/v1/user.proto -t internal/service


生成了 `user.go`文件，内容如下：

```
package service

import (
	"context"

	pb "helloworld/api/helloworld/v1"
)

type UserService struct {
	pb.UnimplementedUserServer
}

func NewUserService() *UserService {
	return &UserService{}
}

func (s *UserService) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.CreateUserReply, error) {
	return &pb.CreateUserReply{}, nil
}
func (s *UserService) UpdateUser(ctx context.Context, req *pb.UpdateUserRequest) (*pb.UpdateUserReply, error) {
	return &pb.UpdateUserReply{}, nil
}
func (s *UserService) DeleteUser(ctx context.Context, req *pb.DeleteUserRequest) (*pb.DeleteUserReply, error) {
	return &pb.DeleteUserReply{}, nil
}
func (s *UserService) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserReply, error) {
	return &pb.GetUserReply{Id: req.Id, Name: "Name"}, nil
}
func (s *UserService) ListUser(ctx context.Context, req *pb.ListUserRequest) (*pb.ListUserReply, error) {
	return &pb.ListUserReply{}, nil
}
```

工具帮我们生成了对应的方法，我们只要实现就可以了。


> 依赖注入UserService


修改 `internal/service/service.go`文件，注入UserService

把原来的：

    var ProviderSet = wire.NewSet(NewGreeterService)
修改成

    var ProviderSet = wire.NewSet(NewGreeterService,NewUserService)


> 修改 `internal/server/http.go`文件，注入UserService

把UserService注入到 http服务中 `internal/server/http.go`

在`NewHTTPServer()`函数中，增加参数`userService *service.UserService`

以及函数内实现 `v1.RegisterUserHTTPServer(srv, userService)`


> `make generate`



> 实现 UserService 接口中的方法



    func (s *UserService) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserReply, error) {
    return &pb.GetUserReply{}, nil
    }

修改成如下，增加了一个返回的id，把请求的id返回出来
    
    func (s *UserService) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserReply, error) {
    return &pb.GetUserReply{Id: req.Id}, nil
    }
    





#### `kratos` 生成的`_http.pb.go`文件解读


`user_http.pb.go` 文件，主要有两大部分组成。

服务端接口`UserHTTPServer`，以及路由注册函数`RegisterUserHTTPServer()`


客户端接口`UserHTTPClient`，以及初始化客户端命令`NewUserHTTPClient()`供调用方使用。




- 1 创建服务器
  

自动生成的`user_http.pb.go`，帮助我们创建服务端，主要内容有2步。
  

`UserHTTPServer` 接口

    kratos自带的service工具，生成了user.go文件来作为 user_http.pb.go文件中UserHTTPServer接口的具体实现


    kratos proto server api/helloworld/v1/user.proto -t internal/service

      生成的UserService结构体，内嵌了user_grpc.pb.go里面的UnimplementedUserServer方法，作为一个标识，代表是使用protobuf生成的接口的具体实现


`RegisterUserHTTPServer` 注册路由


在server的地方调用RegisterUserHTTPServer，就可以直接把实现`user_http.pb.go`中UserHTTPServer接口的 service注册到http服务器中

``` 
// NewHTTPServer new a HTTP server.
func NewHTTPServer(c *conf.Server, greeter *service.GreeterService, userService *service.UserService, logger log.Logger) *http.Server {
    var opts = []http.ServerOption{
        http.Middleware(
            recovery.Recovery(),
        ),
    }
    if c.Http.Network != "" {
        opts = append(opts, http.Network(c.Http.Network))
    }
    if c.Http.Addr != "" {
        opts = append(opts, http.Address(c.Http.Addr))
    }
    if c.Http.Timeout != nil {
        opts = append(opts, http.Timeout(c.Http.Timeout.AsDuration()))
    }
    srv := http.NewServer(opts...)
    v1.RegisterGreeterHTTPServer(srv, greeter)
    v1.RegisterUserHTTPServer(srv, userService) //在这里注册我们的service到http服务器中。
    return srv
}
```



- 2 创建客户端

自动生成的`user_http.pb.go`，帮助我们创建client，主要内容有2步。



UserHTTPClient接口，提供对外的可以接入的方法


NewUserHTTPClient，创建客户端。


``` 
func TestUserService_GetUser(t *testing.T) {
    client, err := http.NewClient(
        context.Background(),
        http.WithEndpoint("http://localhost:8000"),
    )
    if err != nil {
        t.Error(err)
    }
    userClient := v1.NewUserHTTPClient(client)
    resp, err := userClient.GetUser(context.Background(), &v1.GetUserRequest{Id: 1})
    t.Log(resp, err)
}
```

这样就提供了一个restful客户端接入方法, 填入一个 http://localhost:8000这个地址为http服务器地址。
和grpc的调用很类似，都是连接到服务器，然后调用方法
