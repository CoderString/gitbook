### go的grpc

> rpc(Remote Procedure Call)是一种计算机通信协议。通过该协议可以允许运行在一台计算机的程序调用另一个地址空间的子程序，在这个过程中程序调用者无须关心具体的实现细节，就像平时调用本地服务一样。RPC是一种基于服务端和客户端的模式；而grpc是一种现代化开源的高性能RPC框架，能够运行在任何的黄精中，最初由Google 开发，使用HTTP/2作为传输协议。

#### grpc的优点

> 使用grpc，允许我们在一个.proto文件中定义服务并使用任何它支持的语言去实现客户端和服务端，除此之外grpc帮助我们解决了不同语言及环境之间通信的复杂性；使用protocol buffers还能够获取其他相关的一些关键的信息(高效的序列号、简单的IDL以及比较容易的接口更新)，归纳总结而言,grpc能够让我们更加容易的编写跨语言的分布式代码。

#### grpc的环境准备

> 在使用rrpc之前我们需要完成必要的环境配置：包括grpc的依赖安装、Protocol Buffers v3 协议编译器安装、protoc的go插件。

##### grpc的安装

```go
go get -u google.golang.org/grpc
```

##### Protocol Buffers v3的安装

> Protocol Buffers v3是用于生成grpc服务代码的协议编码器，需要根据自己电脑的操作系统提前进行安装

```
https://github.com/protocolbuffers/protobuf/releases
```

安装步骤：
    1、解压下载好的文件
    2、把protoc二进制文件的路径加到环境变量中

##### protoc插件protoc-gen-go安装

```
go get -u github.com/golang/protobuf/protoc-gen-go
```

#### grpc案例展示

> 通常而言，如果想要实现一个grpc服务，我们需要完成下面的三个步骤：<br/>
>   1、编写proto代码<br/>
>   2、编写服务端代码<br/>
>   3、编写客户端代码<br/>

#####  proto代码编写及*.pb.go文件生成

>   proto文件的编写上需要注意的要点是需要显示指定xxx.pb.go文件的存放的路径，这个如果不写的话会报错导致文件无法生成<br/>
>   另一个需要注意的点是生成文件的时候直接切换到proto文件所在的目录中，运行： protoc --go_out=plugins=grpc:. hello.proto<br/>
>   命令的参数的含义：第一个参数.表示生成xxx.pg.go的文件的存放路径；hello.proto表示proto源文件的存放的位置。

```go
syntax = "proto3"; // 版本声明，使用Protocol Buffers v3版本

option go_package = "./"; //设置xxx.pb.go文件的存放路径

package pb; // 包名


// 定义一个打招呼服务
service Greeter {
    // SayHello 方法
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// 包含人名的一个请求消息
message HelloRequest {
    string name = 1;
}

// 包含问候语的响应消息
message HelloReply {
    string message = 1;
}
```

##### 服务端代码编写

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
	pb "hello/proto/hello"
	"net"
)

type server struct{}

func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	return &pb.HelloReply{Message: "Hello " + in.Name}, nil
}

func main() {
	lis, err := net.Listen("tcp", ":8972")
	if err != nil {
		fmt.Printf("failed to listen: %v", err)
		return
	}
	s := grpc.NewServer()
	pb.RegisterGreeterServer(s, &server{}) // 在gRPC服务端注册服务

	reflection.Register(s) //在给定的gRPC服务器上注册服务器反射服务
	// Serve方法在lis上接受传入连接，为每个连接创建一个ServerTransport和server的goroutine。
	// 该goroutine读取gRPC请求，然后调用已注册的处理程序来响应它们。
	err = s.Serve(lis)
	if err != nil {
		fmt.Printf("failed to serve: %v", err)
		return
	}
}
```


##### 客户端代码编写

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	pb "hello/proto/hello"
)

func main() {
	// 连接服务器
	conn, err := grpc.Dial(":8972", grpc.WithInsecure())
	if err != nil {
		fmt.Printf("faild to connect: %v", err)
	}
	defer conn.Close()

	c := pb.NewGreeterClient(conn)
	// 调用服务端的SayHello
	r, err := c.SayHello(context.Background(), &pb.HelloRequest{Name: "q1mi"})
	if err != nil {
		fmt.Printf("could not greet: %v", err)
	}
	fmt.Printf("Greeting: %s !\n", r.Message)
}
```

#### 参考资料

1、https://blog.csdn.net/fujian9544/article/details/116809779

2、https://www.liwenzhou.com/posts/Go/gRPC/
