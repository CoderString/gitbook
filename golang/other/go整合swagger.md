### golang 整合swagger

#### 集成方式

```go
import (
	swaggerFiles "github.com/swaggo/files"
	ginSwagger "github.com/swaggo/gin-swagger"
	_ "to-do-list/docs" // 这里需要引入本地已生成文档
)

func NewRouter() *gin.Engine {
	r := gin.Default()
	r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler)) // 开启swag
}
```

#### 实现方式

1、go get -u github.com/swaggo/swag/cmd/swag 下载安装swag

2、gin.Engine对象增加swagger映射r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))

3、在方法接口上添加相应的注释

```go
// @Tags USER
// @Summary 用户登录
// @Produce json
// @Accept json
// @Param     data    body     service.UserService    true      "user_name, password"
// @Success 200 {object} serializer.ResponseUser "{"success":true,"data":{},"msg":"登陆成功"}"
// @Failure 500 {object} serializer.ResponseUser "{"status":500,"data":{},"Msg":{},"Error":"error"}"
// @Router /user/login [post]
func UserLogin(c *gin.Context) {
}
```

4、在main方法上进行总的配置

```go
// @title ToDoList API
// @version 0.0.1
// @description This is a sample Server pets
// @name FanOne
// @BasePath /api/v1
// http://localhost:3000/swagger/index.html
func main() {
}
```

5、在与入口函数main.go 同级路径下执行 swag init 命令，会在同级生成一个docs 目录，会根据你的注释生成swagger启动需要读取的json文档

6、一定要注意要将docs 目录增加到路径读取的go文件中_ "to-do-list/docs" 

7、启动项目后访问 http://localhost:8080/swagger/index.html


#### 注解含义

1、Tags 分组名

2、@Title
这个 API 所表达的含义，是一个文本，空格之后的内容全部解析为 title

3、@Description
这个 API 详细的描述，是一个文本，空格之后的内容全部解析为 Description

4、@Param
参数，表示需要传递到服务器端的参数，有五列参数，使用空格或者 tab 分割，五个分别表示的含义如下：
    1.参数名
    2.参数类型，可以有的值是 formData、query、path、body、header，formData 表示是 post 请求的数据，query 表示带在 url 之后的参数，path 表示请求路径上得参数，例如上面例子里面的 key，body 表示是一个 raw 数据请求，header 表示带在 header 信息中得参数。
    3.参数类型
    4.是否必须
    5.注释

5、@Success
成功返回给客户端的信息，三个参数，第一个是 status code。第二个参数是返回的类型，必须使用 {} 包含，第三个是返回的对象或者字符串信息，如果是 {object} 类型，那么 bee 工具在生成 docs 的时候会扫描对应的对象，这里填写的是想对你项目的目录名和对象，例如 models.ZDTProduct.ProductList 就表示 /models/ZDTProduct 目录下的 ProductList 对象。
三个参数必须通过空格分隔

6、Failure
失败返回的信息，包含两个参数，使用空格分隔，第一个表示 status code，第二个表示错误信息

7、@router
路由信息，包含两个参数，使用空格分隔，第一个是请求的路由地址，支持正则和自定义路由，和之前的路由规则一样，第二个参数是支持的请求方法,放在 [] 之中，如果有多个方法，那么使用 , 分隔。


#### 参考文献

1、https://blog.csdn.net/weixin_41774732/article/details/112571953