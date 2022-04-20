### go的pprof运用和讲解

> go中使用pprof包能做代码的性能监控，主要监控的点有下面的几个方面：cpu profile、mem profile、block profile，分析的过程中使用到的包有runtime/pprof和net/http/pprof， 后者在前者的基础上做了些简单的包装，使其在http的端口上进行了暴露。两者的区别：

- runtime/pprof：调用了就开始采样，数据会不断的被写入到文件中

- net/http/pprof：只有在调用某个API的时候，才会发起采样，并且sleep一段时间，时间到了之后就会关闭采样，因此日志文件是按照需求来写入的。


### 使用引入


> WEB相关新能分析

如果你的go程序是用http包启动的web服务器，你想查看自己的web服务器的状态。这个时候就可以选择net/http/pprof。你只需要引入包_"net/http/pprof"，然后就可以在浏览器中使用http://localhost:port/debug/pprof/直接看到当前web服务的状态，包括CPU占用情况和内存使用情况等。具体使用情况你可以看godoc的说明。

> 常驻进程

如果你的go程序不是web服务器，而是一个服务进程，那么你也可以选择使用net/http/pprof包，同样引入包_ "net/http/pprof"，然后在开启另外一个goroutine来开启端口监听。


> 相关pprof参数说明


- allocs :过去所有内存分配的抽样
- block :导致在同步基元上阻塞的堆栈跟踪
- cmdline :当前程序的命令行调用
- goroutine :goroutine堆栈跟踪
- heap :活动对象的内存分配的采样
- mutex :争用互斥锁的持有者的堆栈跟踪
- profile :CPU摘要信息。您可以在seconds GET参数中指定持续时间。获得概要文件之后，使用go工具pprof命令来调查该概要文件。
- threadcreate : 操作系统线程堆栈跟踪
- trace : 当前程序的执行轨迹。您可以在seconds GET参数中指定持续时间。获得跟踪文件后，使用go工具跟踪命令来调查跟踪。


无论是常驻线程还是WEB服务器场景，pprof都会注册下面的几个路由：

```go
func init() {
	http.HandleFunc("/debug/pprof/", Index)
	http.HandleFunc("/debug/pprof/cmdline", Cmdline)
	http.HandleFunc("/debug/pprof/profile", Profile)
	http.HandleFunc("/debug/pprof/symbol", Symbol)
	http.HandleFunc("/debug/pprof/trace", Trace)
}
```

### 实战案例

```go
import "net/http"
import _ "net/http/pprof"
func main() {
    http.ListenAndServe("0.0.0.0:8080", nil)
}
```

> 运行程序之后，访问http://127.0.0.1:8080/debug/pprof/， 就可以看到相关的性能分析情况。

> 火焰图生成，运行下面的命令稍等一会后，浏览器上会出现火焰图UI相关的界面。

```
go tool pprof -http=:1234 http://localhost:8080/debug/pprof/profile
```

### Gin框架使用pprof

> gin框架如果想要使用pprof，需要首先安装pprof的适配的包.

```go
 go get github.com/gin-contrib/pprof
```

> 使用案例

```go
package main

import (
    "github.com/gin-contrib/pprof"
    "github.com/gin-gonic/gin"
)

func main() {
  router := gin.Default()
  pprof.Register(router)
  router.Run(":8080")
}
```

> 浏览器访问

http://127.0.0.1:8080/debug/pprof/

> 生成火焰图

go tool pprof -http=:1234 http://localhost:8080/debug/pprof/goroutine

### 参考资料

1、https://blog.csdn.net/raoxiaoya/article/details/118493465

2、https://studygolang.com/articles/30771