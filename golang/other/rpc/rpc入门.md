### go的rpc入门

> Go 标准包中已经提供了对 RPC 的支持，而且支持三个级别的 RPC：TCP、HTTP、JSONRPC。但 Go 的 RPC 包是独一无二的 RPC，它和传统的 RPC 系统不同，它只支持 Go 开发的服务器与客户端之间的交互，因为在内部，它们采用了 Gob 来编码。<br/>
> 使用golang中的net/rpc, 需要将被注册为rpc服务的对象满足下面的规则：

- 方法只能有两个可序列化的参数
- 方法中第二个参数是指针类型
- 方法返回一个 error 类型
- 方法是公开的方法

> rpc的一般格式

T、T1和T2类型必须能被encoding/gob包编解码。

```
func (t *T) MethodName(argType T1, replyType *T2) error
```



```
type HelloService struct {}

func (p *HelloService) Hello(request string, reply *string) error {
    *reply = "hello:" + request
    return nil
}
```

#### rpc简单实现

- server端代码

```go
package main

import (
	"log"
	"net"
	"net/rpc"
)

type HelloService struct {

}

func (p *HelloService) Hello(request string, reply *string) error {
	*reply = "hello" + request
	return nil
}

func main() {
	// 注册
	rpc.RegisterName("HelloService", new(HelloService))

	//持续监听端口
	listen, err := net.Listen("tcp", ":1234")
	if err != nil {
		log.Fatal("ListenTcp error:", err)
	}

	//监听数据
	conn, err := listen.Accept()
	if err != nil {
		log.Fatalln("Accept error:", err)
	}

	//建立rpc链接
	rpc.ServeConn(conn)
}
```

- client端代码

```go
package main

import (
	"fmt"
	"log"
	"net/rpc"
)

func main() {

	//客户端通过tcp连接到指定端口，直到连接成功，程序放行
	client, err := rpc.Dial("tcp", "localhost:1234")
	if err != nil {
		log.Fatal("dialing:", err)
	}

	var reply string
	err = client.Call("HelloService.Hello", "World", &reply)
	if err != nil {
		log.Fatalln(err)
	}

	fmt.Println(reply)
}
```

#### 更安全的rpc框架

> 在简单实现的基础上，我们可以针对客户端和服务端指定一些接口的规范，让调用的过程更加的规范.

接口规范

```go
// 服务端接口规范
const HelloServiceName = "path/to/pkg.HelloService"

type HelloServiceInterface interface {
	Hello(request string, reply *string) error
}

func RegisterHelloService(srv HelloServiceInterface) error {
	return rpc.RegisterName(HelloServiceName, srv)
}


type HelloService struct {

}


//客户端接口规范
type HelloServiceClient struct {
	*rpc.Client
}

var _ HelloServiceInterface = (*HelloServiceClient)(nil)

func DialHelloService(network, address string) (*HelloServiceClient, error) {
	c, err := rpc.Dial(network, address)
	if err != nil {
		return nil, err
	}
	return &HelloServiceClient{Client: c}, nil
}

func (p *HelloServiceClient) Hello(request string, reply *string) error {
	return p.Client.Call(HelloServiceName+".Hello", request, reply)
}
```

服务端代码

```go
func (p *HelloService) Hello(request string, reply *string) error {
	*reply = "hello:" + request
	return nil
}

func main() {
	RegisterHelloService(new(HelloService))
	listener, err := net.Listen("tcp", ":1234")
	if err != nil {
		log.Fatal("listenTCP error:", err)
	}

	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Fatalln("Accept error:", err)
		}
		go rpc.ServeConn(conn)
	}
}
```

客户端

```go
func main() {
	client, err := DialHelloService("tcp", "localhost:1234")
	if err != nil {
		log.Fatal("dialing:", err)
	}

	var reply string
	err = client.Hello("hello", &reply)
	if err != nil {
		log.Fatal(err)
	}
}

```

#### 跨语言的rpc

> 标准库的RPC默认采用Go语言特有的gob编码，因此从其它语言调用Go语言实现的RPC服务将比较困难。在互联网的微服务时代，每个RPC以及服务的使用者都可能采用不同的编程语言，因此跨语言是互联网时代RPC的一个首要条件。得益于RPC的框架设计，Go语言的RPC其实也是很容易实现跨语言支持的。

> Go语言的RPC框架有两个比较有特色的设计：一个是RPC数据打包时可以通过插件实现自定义的编码和解码；另一个是RPC建立在抽象的io.ReadWriteCloser接口之上的，我们可以将RPC架设在不同的通讯协议之上。这里我们将尝试通过官方自带的net/rpc/jsonrpc扩展实现一个跨语言的RPC。

服务端

```go
func main() {
    rpc.RegisterName("HelloService", new(HelloService))

    listener, err := net.Listen("tcp", ":1234")
    if err != nil {
        log.Fatal("ListenTCP error:", err)
    }

    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Fatal("Accept error:", err)
        }

        go rpc.ServeCodec(jsonrpc.NewServerCodec(conn))
    }
}
```

客户端

```go
func main() {
    conn, err := net.Dial("tcp", "localhost:1234")
    if err != nil {
        log.Fatal("net.Dial:", err)
    }

    client := rpc.NewClientWithCodec(jsonrpc.NewClientCodec(conn))

    var reply string
    err = client.Call("HelloService.Hello", "hello", &reply)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(reply)
}
```


#### 参考资料

1、https://chai2010.cn/advanced-go-programming-book/ch4-rpc/ch4-01-rpc-intro.html