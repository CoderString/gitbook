### gin系列教程

gin框架是一个使用go语言编写的web框架，使用上相对于其他的编程语言的框架而言比较小巧灵活，只要安装了gin的依赖库，就可以非常简单的开启web的服务，本篇文章主要从基本的使用开始讲解，后续可能会适当的分析下gin的实现源码。


#### hello world 

在按照好go的开发环境的基础上，在安装gin的安装包。

- gin包安装

```
go get -u github.com/gin-gonic/gin
```

- hello wold 

```
package main

import "github.com/gin-gonic/gin"

func main() {

	r := gin.Default()
	r.GET("ping", func(c *gin.Context){
		c.JSON(200, gin.H{
			"message" : "pong",
		})
	})
	r.Run()
}
```

---

- 路由参数


实现方式一

```
func main() {
    router := gin.Default()

    // 此 handler 将匹配 /user/john 但不会匹配 /user/ 或者 /user
    router.GET("/user/:name", func(c *gin.Context) {
        name := c.Param("name")
        c.String(http.StatusOK, "Hello %s", name)
    })

    // 此 handler 将匹配 /user/john/ 和 /user/john/send
    // 如果没有其他路由匹配 /user/john，它将重定向到 /user/john/
    router.GET("/user/:name/*action", func(c *gin.Context) {
        name := c.Param("name")
        action := c.Param("action")
        message := name + " is " + action
        c.String(http.StatusOK, message)
    })

    router.Run(":8080")
}
```

实现方式二

```
package main

import "github.com/gin-gonic/gin"

type Person struct {
    ID string `uri:"id" binding:"required,uuid"`
    Name string `uri:"name" binding:"required"`
}

func main() {
    route := gin.Default()
    route.GET("/:name/:id", func(c *gin.Context) {
        var person Person
        if err := c.ShouldBindUri(&person); err != nil {
            c.JSON(400, gin.H{"msg": err})
            return
        }
        c.JSON(200, gin.H{"name": person.Name, "uuid": person.ID})
    })
    route.Run(":8088")
}
```


- 绑定查询字符创或者表单数据


实现方式一

```
package main

import (
    "log"
    "time"

    "github.com/gin-gonic/gin"
)

type Person struct {
    Name     string    `form:"name"`
    Address  string    `form:"address"`
    Birthday time.Time `form:"birthday" time_format:"2006-01-02" time_utc:"1"`
}

func main() {
    route := gin.Default()
    route.GET("/testing", startPage)
    route.Run(":8085")
}

func startPage(c *gin.Context) {
    var person Person
    // 如果是 `GET` 请求，只使用 `Form` 绑定引擎（`query`）。
    // 如果是 `POST` 请求，首先检查 `content-type` 是否为 `JSON` 或 `XML`，然后再使用 `Form`（`form-data`）。
    // 查看更多：https://github.com/gin-gonic/gin/blob/master/binding/binding.go#L48
    if c.ShouldBind(&person) == nil {
        log.Println(person.Name)
        log.Println(person.Address)
        log.Println(person.Birthday)
    }

    c.String(200, "Success")
}

curl -X GET "localhost:8085/testing?name=appleboy&address=xyz&birthday=1992-03-15"
```

实现方式二

```
func main() {
    router := gin.Default()

    // 使用现有的基础请求对象解析查询字符串参数。
    // 示例 URL： /welcome?firstname=Jane&lastname=Doe
    router.GET("/welcome", func(c *gin.Context) {
        firstname := c.DefaultQuery("firstname", "Guest")
        lastname := c.Query("lastname") // c.Request.URL.Query().Get("lastname") 的一种快捷方式

        c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
    })
    router.Run(":8080")
}
```

- POST请求方式的URL参数

```
func main() {
    router := gin.Default()

    router.POST("/post", func(c *gin.Context) {

        id := c.Query("id")
        page := c.DefaultQuery("page", "0")
        name := c.PostForm("name")
        message := c.PostForm("message")

        fmt.Printf("id: %s; page: %s; name: %s; message: %s", id, page, name, message)
    })
    router.Run(":8080")
}
```

请求数据

```
POST /post?id=1234&page=1 HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=manu&message=this_is_great
```

- Multipart/Urlencoded 表单

```
func main() {
    router := gin.Default()

    router.POST("/form_post", func(c *gin.Context) {
        message := c.PostForm("message")
        nick := c.DefaultPostForm("nick", "anonymous")

        c.JSON(200, gin.H{
            "status":  "posted",
            "message": message,
            "nick":    nick,
        })
    })
    router.Run(":8080")
}
```

- Multipart/Urlencoded 绑定

```
package main

import (
    "github.com/gin-gonic/gin"
)

type LoginForm struct {
    User     string `form:"user" binding:"required"`
    Password string `form:"password" binding:"required"`
}

func main() {
    router := gin.Default()
    router.POST("/login", func(c *gin.Context) {
        // 你可以使用显式绑定声明绑定 multipart form：
        // c.ShouldBindWith(&form, binding.Form)
        // 或者简单地使用 ShouldBind 方法自动绑定：
        var form LoginForm
        // 在这种情况下，将自动选择合适的绑定
        if c.ShouldBind(&form) == nil {
            if form.User == "user" && form.Password == "password" {
                c.JSON(200, gin.H{"status": "you are logged in"})
            } else {
                c.JSON(401, gin.H{"status": "unauthorized"})
            }
        } else {
                c.JSON(401, gin.H{"status": "missing params"})
        }
    })
    router.Run(":8080")
}

curl -v --form user=user --form password=password http://localhost:8080/login
```

- 绑定表单参数到结构体

```
package main

import (
    "log"
    "time"

    "github.com/gin-gonic/gin"
)

type Person struct {
    Name     string    `form:"name"`
    Address  string    `form:"address"`
    Birthday time.Time `form:"birthday" time_format:"2006-01-02" time_utc:"1"`
}

func main() {
    route := gin.Default()
    route.GET("/testing", startPage)
    route.Run(":8085")
}

func startPage(c *gin.Context) {
    var person Person
    // 如果是 `GET` 请求，只使用 `Form` 绑定引擎（`query`）。
    // 如果是 `POST` 请求，首先检查 `content-type` 是否为 `JSON` 或 `XML`，然后再使用 `Form`（`form-data`）。
    // 查看更多：https://github.com/gin-gonic/gin/blob/master/binding/binding.go#L48
    if c.ShouldBind(&person) == nil {
        log.Println(person.Name)
        log.Println(person.Address)
        log.Println(person.Birthday)
    }

    c.String(200, "Success")
}

curl -X GET "localhost:8085/testing?name=appleboy&address=xyz&birthday=1992-03-15"
```

- 将Request Body中的数据绑定到不同的结构体中

```
func SomeHandler(c *gin.Context) {
  objA := formA{}
  objB := formB{}
  // 读取 c.Request.Body 并将结果存入上下文。
  if errA := c.ShouldBindBodyWith(&objA, binding.JSON); errA == nil {
    c.String(http.StatusOK, `the body should be formA`)
  // 这时, 复用存储在上下文中的 body。
  } else if errB := c.ShouldBindBodyWith(&objB, binding.JSON); errB == nil {
    c.String(http.StatusOK, `the body should be formB JSON`)
  // 可以接受其他格式
  } else if errB2 := c.ShouldBindBodyWith(&objB, binding.XML); errB2 == nil {
    c.String(http.StatusOK, `the body should be formB XML`)
  } else {
    ...
  }
}
```

- 输出不同格式进行渲染

```
func main() {
    r := gin.Default()

    // gin.H 是 map[string]interface{} 的一种快捷方式
    r.GET("/someJSON", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
    })

    r.GET("/moreJSON", func(c *gin.Context) {
        // 你也可以使用一个结构体
        var msg struct {
            Name    string `json:"user"`
            Message string
            Number  int
        }
        msg.Name = "Lena"
        msg.Message = "hey"
        msg.Number = 123
        // 注意 msg.Name 在 JSON 中变成了 "user"
        // 将输出：{"user": "Lena", "Message": "hey", "Number": 123}
        c.JSON(http.StatusOK, msg)
    })

    r.GET("/someXML", func(c *gin.Context) {
        c.XML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
    })

    r.GET("/someYAML", func(c *gin.Context) {
        c.YAML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
    })

    r.GET("/someProtoBuf", func(c *gin.Context) {
        reps := []int64{int64(1), int64(2)}
        label := "test"
        // protobuf 的具体定义写在 testdata/protoexample 文件中。
        data := &protoexample.Test{
            Label: &label,
            Reps:  reps,
        }
        // 请注意，数据在响应中变为二进制数据
        // 将输出被 protoexample.Test protobuf 序列化了的数据
        c.ProtoBuf(http.StatusOK, data)
    })

    // 监听并在 0.0.0.0:8080 上启动服务
    r.Run(":8080")
}
```

- HTML各种参数的绑定

```
type myForm struct {
    Colors []string `form:"colors[]"`
}

func formHandler(c *gin.Context) {
    var fakeForm myForm
    c.ShouldBind(&fakeForm)
    c.JSON(200, gin.H{"color": fakeForm.Colors})
}


<form action="/" method="POST">
    <p>Check some colors</p>
    <label for="red">Red</label>
    <input type="checkbox" name="colors[]" value="red" id="red">
    <label for="green">Green</label>
    <input type="checkbox" name="colors[]" value="green" id="green">
    <label for="blue">Blue</label>
    <input type="checkbox" name="colors[]" value="blue" id="blue">
    <input type="submit">
</form>


结果：{"color":["red","green","blue"]}

```

- 设置和获取cookie

```
import (
    "fmt"

    "github.com/gin-gonic/gin"
)

func main() {

    router := gin.Default()
    router.GET("/cookie", func(c *gin.Context) {
    cookie, err := c.Cookie("gin_cookie")
        if err != nil {
            cookie = "NotSet"
            c.SetCookie("gin_cookie", "test", 3600, "/", "localhost", false, true)
        }
        fmt.Printf("Cookie value: %s \n", cookie)
    })
    router.Run()
}
```

- 单个文件或多个文件的上传

单个文件

```
func main() {
    router := gin.Default()
    // 为 multipart forms 设置较低的内存限制 (默认是 32 MiB)
    // router.MaxMultipartMemory = 8 << 20  // 8 MiB
    router.POST("/upload", func(c *gin.Context) {
        // 单文件
        file, _ := c.FormFile("file")
        log.Println(file.Filename)

        // 上传文件至指定目录
        // c.SaveUploadedFile(file, dst)

        c.String(http.StatusOK, fmt.Sprintf("'%s' uploaded!", file.Filename))
    })
    router.Run(":8080")
}


curl -X POST http://localhost:8080/upload \
  -F "file=@/Users/appleboy/test.zip" \
  -H "Content-Type: multipart/form-data"

```

多个文件
```
func main() {
    router := gin.Default()
    // 为 multipart forms 设置较低的内存限制 (默认是 32 MiB)
    // router.MaxMultipartMemory = 8 << 20  // 8 MiB
    router.POST("/upload", func(c *gin.Context) {
        // Multipart form
        form, _ := c.MultipartForm()
        files := form.File["upload[]"]

        for _, file := range files {
            log.Println(file.Filename)

            // 上传文件至指定目录
            // c.SaveUploadedFile(file, dst)
        }
        c.String(http.StatusOK, fmt.Sprintf("%d files uploaded!", len(files)))
    })
    router.Run(":8080")
}


curl -X POST http://localhost:8080/upload \
  -F "upload[]=@/Users/appleboy/test1.zip" \
  -F "upload[]=@/Users/appleboy/test2.zip" \
  -H "Content-Type: multipart/form-data"
```

- Log控制是否高亮显示

```
func main() {
    // 关闭高亮
    gin.DisableConsoleColor()

    // 强制 log 高亮
    //gin.ForceConsoleColor()


    // Creates a gin router with default middleware:
    // logger and recovery (crash-free) middleware
    router := gin.Default()

    router.GET("/ping", func(c *gin.Context) {
        c.String(200, "pong")
    })

    router.Run(":8080")
}
```

- 定义Log的输出格式

```
func main() {
    router := gin.New()
    // LoggerWithFormatter middleware will write the logs to gin.DefaultWriter
    // By default gin.DefaultWriter = os.Stdout
    router.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {
        // your custom format
        return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
                param.ClientIP,
                param.TimeStamp.Format(time.RFC1123),
                param.Method,
                param.Path,
                param.Request.Proto,
                param.StatusCode,
                param.Latency,
                param.Request.UserAgent(),
                param.ErrorMessage,
        )
    }))
    router.Use(gin.Recovery())
    router.GET("/ping", func(c *gin.Context) {
        c.String(200, "pong")
    })
    router.Run(":8080")
}


output：::1 - [Fri, 07 Dec 2018 17:04:38 JST] "GET /ping HTTP/1.1 200 122.767µs "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36" "
```

- 定义路由日志的格式

```
import (
    "log"
    "net/http"

    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()
    gin.DebugPrintRouteFunc = func(httpMethod, absolutePath, handlerName string, nuHandlers int) {
        log.Printf("endpoint %v %v %v %v\n", httpMethod, absolutePath, handlerName, nuHandlers)
    }

    r.POST("/foo", func(c *gin.Context) {
        c.JSON(http.StatusOK, "foo")
    })

    r.GET("/bar", func(c *gin.Context) {
        c.JSON(http.StatusOK, "bar")
    })

    r.GET("/status", func(c *gin.Context) {
        c.JSON(http.StatusOK, "ok")
    })

    // 监听并在 0.0.0.0:8080 上启动服务
    r.Run()
}
```



- 记录日志

```
func main() {
    // 禁用控制台颜色，将日志写入文件时不需要控制台颜色。
    gin.DisableConsoleColor()

    // 记录到文件。
    f, _ := os.Create("gin.log")
    gin.DefaultWriter = io.MultiWriter(f)

    // 如果需要同时将日志写入文件和控制台，请使用以下代码。
    // gin.DefaultWriter = io.MultiWriter(f, os.Stdout)

    router := gin.Default()
    router.GET("/ping", func(c *gin.Context) {
        c.String(200, "pong")
    })

    router.Run(":8080")
}
```

- 运行多个服务

```
package main

import (
    "log"
    "net/http"
    "time"

    "github.com/gin-gonic/gin"
    "golang.org/x/sync/errgroup"
)

var (
    g errgroup.Group
)

func router01() http.Handler {
    e := gin.New()
    e.Use(gin.Recovery())
    e.GET("/", func(c *gin.Context) {
        c.JSON(
            http.StatusOK,
            gin.H{
                "code":  http.StatusOK,
                "error": "Welcome server 01",
            },
        )
    })

    return e
}

func router02() http.Handler {
    e := gin.New()
    e.Use(gin.Recovery())
    e.GET("/", func(c *gin.Context) {
        c.JSON(
            http.StatusOK,
            gin.H{
                "code":  http.StatusOK,
                "error": "Welcome server 02",
            },
        )
    })

    return e
}

func main() {
    server01 := &http.Server{
        Addr:         ":8080",
        Handler:      router01(),
        ReadTimeout:  5 * time.Second,
        WriteTimeout: 10 * time.Second,
    }

    server02 := &http.Server{
        Addr:         ":8081",
        Handler:      router02(),
        ReadTimeout:  5 * time.Second,
        WriteTimeout: 10 * time.Second,
    }

    g.Go(func() error {
        return server01.ListenAndServe()
    })

    g.Go(func() error {
        return server02.ListenAndServe()
    })

    if err := g.Wait(); err != nil {
        log.Fatal(err)
    }
}
```

- 自定义启动中间件

```
func main() {
    // 新建一个没有任何默认中间件的路由
    r := gin.New()

    // 全局中间件
    // Logger 中间件将日志写入 gin.DefaultWriter，即使你将 GIN_MODE 设置为 release。
    // By default gin.DefaultWriter = os.Stdout
    r.Use(gin.Logger())

    // Recovery 中间件会 recover 任何 panic。如果有 panic 的话，会写入 500。
    r.Use(gin.Recovery())

    // 你可以为每个路由添加任意数量的中间件。
    r.GET("/benchmark", MyBenchLogger(), benchEndpoint)

    // 认证路由组
    // authorized := r.Group("/", AuthRequired())
    // 和使用以下两行代码的效果完全一样:
    authorized := r.Group("/")
    // 路由组中间件! 在此例中，我们在 "authorized" 路由组中使用自定义创建的 
    // AuthRequired() 中间件
    authorized.Use(AuthRequired())
    {
        authorized.POST("/login", loginEndpoint)
        authorized.POST("/submit", submitEndpoint)
        authorized.POST("/read", readEndpoint)

        // 嵌套路由组
        testing := authorized.Group("testing")
        testing.GET("/analytics", analyticsEndpoint)
    }

    // 监听并在 0.0.0.0:8080 上启动服务
    r.Run(":8080")
}
```

- 重定向


HTTP重定向

```
r.GET("/test", func(c *gin.Context) {
    c.Redirect(http.StatusMovedPermanently, "http://www.google.com/")
})
```

路由重定向

```
r.GET("/test", func(c *gin.Context) {
    c.Request.URL.Path = "/test2"
    r.HandleContext(c)
})
r.GET("/test2", func(c *gin.Context) {
    c.JSON(200, gin.H{"hello": "world"})
})
```

- 路由组

```
func main() {
    router := gin.Default()

    // 简单的路由组: v1
    v1 := router.Group("/v1")
    {
        v1.POST("/login", loginEndpoint)
        v1.POST("/submit", submitEndpoint)
        v1.POST("/read", readEndpoint)
    }

    // 简单的路由组: v2
    v2 := router.Group("/v2")
    {
        v2.POST("/login", loginEndpoint)
        v2.POST("/submit", submitEndpoint)
        v2.POST("/read", readEndpoint)
    }

    router.Run(":8080")
}
```

- HTTP RESTful API

```
func main() {
    // 禁用控制台颜色
    // gin.DisableConsoleColor()

    // 使用默认中间件（logger 和 recovery 中间件）创建 gin 路由
    router := gin.Default()

    router.GET("/someGet", getting)
    router.POST("/somePost", posting)
    router.PUT("/somePut", putting)
    router.DELETE("/someDelete", deleting)
    router.PATCH("/somePatch", patching)
    router.HEAD("/someHead", head)
    router.OPTIONS("/someOptions", options)

    // 默认在 8080 端口启动服务，除非定义了一个 PORT 的环境变量。
    router.Run()
    // router.Run(":3000") hardcode 端口号
}
```

- 中间件中使用Goroutine

```
func main() {
    r := gin.Default()

    r.GET("/long_async", func(c *gin.Context) {
        // 创建在 goroutine 中使用的副本
        cCp := c.Copy()
        go func() {
            // 用 time.Sleep() 模拟一个长任务。
            time.Sleep(5 * time.Second)

            // 请注意您使用的是复制的上下文 "cCp"，这一点很重要
            log.Println("Done! in path " + cCp.Request.URL.Path)
        }()
    })

    r.GET("/long_sync", func(c *gin.Context) {
        // 用 time.Sleep() 模拟一个长任务。
        time.Sleep(5 * time.Second)

        // 因为没有使用 goroutine，不需要拷贝上下文
        log.Println("Done! in path " + c.Request.URL.Path)
    })

    // 监听并在 0.0.0.0:8080 上启动服务
    r.Run(":8080")
}
```