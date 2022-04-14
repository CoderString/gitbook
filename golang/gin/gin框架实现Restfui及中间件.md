### gin框架实现Restfui Api及中间件

> 下面的案例结合gin实现一个单体项目的Restfui风格的API架构设计，及编写基于gin框架的中间件。

#### 项目的架构

```
Demo/
├── api
├── cache
├── conf
├── middleware
├── model
├── pkg
│  ├── e
│  ├── logging
│  ├── util
├── routes
├── serializer
└── service
```

- api : 用于定义接口函数
- cache : 放置redis缓存
- conf : 用于存储配置文件
- middleware : 应用中间件
- model : 应用数据库模型
- pkg/e : 封装错误码
- pkg/logging : 日志打印
- pkg/util : 工具函数
- routes : 路由逻辑处理
- serializer : 将数据序列化为 json 的函数
- service : 接口函数的实现

#### api

```go
func UserRegister(c *gin.Context) {
	if err := c.ShouldBind(&userRegisterService); err == nil {
		res := userRegisterService.Register()
		c.JSON(200, res)
	} else {
		c.JSON(400, ErrorResponse(err))
		util.Logger().Info(err)
	}
}

func UserLogin(c *gin.Context) {
	var userLoginService service.UserService
	if err := c.ShouldBind(&userLoginService); err == nil {
		res := userLoginService.Login()
		c.JSON(200, res)
	} else {
		c.JSON(400, ErrorResponse(err))
		util.Logger().Info(err)
	}
}

```

#### service

```go
type UserService struct {
	UserName  string `form:"user_name" json:"user_name" binding:"required,min=3,max=15" example:"FanOne"`
	Password  string `form:"password" json:"password" binding:"required,min=5,max=16" example:"FanOne666"`
}

func (service *UserService) Register() *serializer.Response {
	code := e.SUCCESS
	var user model.User
	var count int
	model.DB.Model(&model.User{}).Where("user_name=?",service.UserName).First(&user).Count(&count)
	//表单验证
	if count == 1 {
		code = e.ErrorExistUser
		return &serializer.Response{
			Status: code,
			Msg:    e.GetMsg(code),
		}
	}
	user.UserName=service.UserName
	//加密密码
	if err := user.SetPassword(service.Password); err != nil {
		logging.Info(err)
		code = e.ErrorFailEncryption
		return &serializer.Response{
			Status: code,
			Msg:    e.GetMsg(code),
		}
	}
	//创建用户
	if err := model.DB.Create(&user).Error; err != nil {
		logging.Info(err)
		code = e.ErrorDatabase
		return &serializer.Response{
			Status: code,
			Msg:    e.GetMsg(code),
		}
	}
	return &serializer.Response{
		Status: code,
		Msg:    e.GetMsg(code),
	}
}

//Login 用户登陆函数
func (service *UserService)Login() serializer.Response {
	var user model.User
	code := e.SUCCESS
	if err := model.DB.Where("user_name=?", service.UserName).First(&user).Error; err != nil {
		//如果查询不到，返回相应的错误
		if gorm.IsRecordNotFoundError(err) {
			logging.Info(err)
			code = e.ErrorNotExistUser
			return serializer.Response{
				Status: code,
				Msg:    e.GetMsg(code),
			}
		}
		logging.Info(err)
		code = e.ErrorDatabase
		return serializer.Response{
			Status: code,
			Msg:    e.GetMsg(code),
		}
	}
	if user.CheckPassword(service.Password) == false {
		code = e.ErrorNotCompare
		return serializer.Response{
			Status: code,
			Msg:    e.GetMsg(code),
		}
	}
	token, err := util.GenerateToken(user.ID,service.UserName, 0)
	if err != nil {
		logging.Info(err)
		code = e.ErrorAuthToken
		return serializer.Response{
			Status: code,
			Msg:    e.GetMsg(code),
		}
	}
	return serializer.Response{
		Status: code,
		Data:   serializer.TokenData{User: serializer.BuildUser(user), Token: token},
		Msg:    e.GetMsg(code),
	}
}
```

#### serializer

```go
type User struct {
	ID 			uint   `json:"id" form:"id" example:"1"`  	// 用户ID
	UserName 	string `json:"user_name" form:"user_name" example:"FanOne"` // 用户名
	Status 		string `json:"status" form:"status"`  		// 用户状态
	CreateAt 	int64  `json:"create_at" form:"create_at"`  // 创建
}


//BuildUser 序列化用户
func BuildUser(user model.User) User {
	return User{
		ID:       user.ID,
		UserName: user.UserName,
		CreateAt: user.CreatedAt.Unix(),
	}
}
```

```go
type ResponseUser struct {
	Status int    `json:"status" example:"200"`
	Data   User   `json:"data"`
	Msg    string `json:"msg" example:"ok"`
	Error  string `json:"error" example:""`
}

type ResponseTask struct {
	Status int    `json:"status"`
	Data   Task   `json:"data"`
	Msg    string `json:"msg"`
	Error  string `json:"error"`
}

```

#### router

```go
//路由配置
func NewRouter() *gin.Engine {
	r := gin.Default() //生成了一个WSGI应用程序实例
	store := cookie.NewStore([]byte("something-very-secret"))
	r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler)) // 开启swag
	r.Use(sessions.Sessions("mysession", store))
	r.Use(middleware.NewLogger(),middleware.Cors(),gin.Recovery())
	v1 := r.Group("api/v1")
	{
		// 读取机型配置的json信息
		v1.GET("dp/info", api.Info)

		// 用户操作
		v1.POST("user/register", api.UserRegister)
		v1.POST("user/login", api.UserLogin)
		authed := v1.Group("/") //需要登陆保护
		authed.Use(middleware.JWT())
		{
			//任务操作
			authed.GET("tasks", api.ListTasks)
			authed.POST("task", api.CreateTask)
			authed.GET("task/:id", api.ShowTask)
			authed.DELETE("task/:id", api.DeleteTask)
			authed.PUT("task/:id", api.UpdateTask)
			authed.POST("search", api.SearchTasks)
		}
	}
	return r
}
```

#### middleware

- cors.go

```go
package middleware

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
	"strings"
)


// 跨域
func Cors() gin.HandlerFunc {
	return func(c *gin.Context) {
		method := c.Request.Method      					//请求方法
		origin := c.Request.Header.Get("Origin")        //请求头部
		var headerKeys []string                             // 声明请求头keys
		for k := range c.Request.Header {
			headerKeys = append(headerKeys, k)
		}
		headerStr := strings.Join(headerKeys, ", ")
		if headerStr != "" {
			headerStr = fmt.Sprintf("access-control-allow-origin, access-control-allow-headers, %s", headerStr)
		} else {
			headerStr = "access-control-allow-origin, access-control-allow-headers"
		}
		if origin != "" {
			c.Writer.Header().Set("Access-Control-Allow-Origin", "*")
			c.Header("Access-Control-Allow-Origin", "*")        // 这是允许访问所有域
			c.Header("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE,UPDATE")      //服务器支持的所有跨域请求的方法,为了避免浏览次请求的多次'预检'请求
			//  header的类型
			c.Header("Access-Control-Allow-Headers", "Authorization, Content-Length, X-CSRF-Token, Token,session,X_Requested_With,Accept, Origin, Host, Connection, Accept-Encoding, Accept-Language,DNT, X-CustomHeader, Keep-Alive, User-Agent, X-Requested-With, If-Modified-Since, Cache-Control, Content-Type, Pragma")
			// 允许跨域设置                                                                                                      可以返回其他子段
			c.Header("Access-Control-Expose-Headers", "Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers,Cache-Control,Content-Language,Content-Type,Expires,Last-Modified,Pragma,FooBar")      // 跨域关键设置 让浏览器可以解析
			c.Header("Access-Control-Max-Age", "172800")        // 缓存请求信息 单位为秒
			c.Header("Access-Control-Allow-Credentials", "false")       //  跨域请求是否需要带cookie信息 默认设置为true
			c.Set("content-type", "application/json")       // 设置返回格式是json
		}
		//放行所有OPTIONS方法
		if method == "OPTIONS" {
			c.JSON(http.StatusOK, "Options Request!")
		}
		// 处理请求
		c.Next()        //  处理请求
	}
}
```

- jwt.go


```go
package middleware

import (
	"github.com/gin-gonic/gin"
	"time"
	"to-do-list/pkg/e"
	"to-do-list/pkg/util"
)

//JWT token验证中间件
func JWT() gin.HandlerFunc {
	return func(c *gin.Context) {
		var code int
		var data interface{}
		code = 200
		token := c.GetHeader("Authorization")
		if token == "" {
			code = 404
		} else {
			claims, err := util.ParseToken(token)
			if err != nil {
				code = e.ErrorAuthCheckTokenFail
			} else if time.Now().Unix() > claims.ExpiresAt {
				code = e.ErrorAuthCheckTokenTimeout
			}
		}
		if code != e.SUCCESS {
			c.JSON(400, gin.H{
				"status": code,
				"msg":    e.GetMsg(code),
				"data":   data,
			})
			c.Abort()
			return
		}
		c.Next()
	}
}
```

- logger.go

```go
package middleware

import (
	"github.com/gin-gonic/gin"
	"time"
	"to-do-list/pkg/util"
)

func NewLogger() gin.HandlerFunc {
	logger := util.Logger()
	return func(c *gin.Context) {
		// 开始时间
		startTime := time.Now()               // 处理请求
		c.Next()                              // 结束时间
		endTime := time.Now()                 // 执行时间
		latencyTime := endTime.Sub(startTime) // 请求方式
		reqMethod := c.Request.Method         // 请求路由
		reqUri := c.Request.RequestURI        // 状态码
		statusCode := c.Writer.Status()       // 请求IP
		clientIP := c.ClientIP()              //日志格式
		logger.Infof("| %3d | %13v | %15s | %s | %s |",
			statusCode,
			latencyTime,
			clientIP,
			reqMethod,
			reqUri,
		)
	}
}
```

#### 响应错误枚举类

- code.go
```go
package e

const (
	SUCCESS       = 200
	ERROR         = 500
	InvalidParams = 400

	//成员错误
	ErrorExistUser      = 10002
	ErrorNotExistUser   = 10003
	ErrorFailEncryption = 10006
	ErrorNotCompare     = 10007

	ErrorAuthCheckTokenFail        = 30001 //token 错误
	ErrorAuthCheckTokenTimeout     = 30002 //token 过期
	ErrorAuthToken                 = 30003
	ErrorAuth                      = 30004
	ErrorDatabase                  = 40001
)
```

- msg.go

```go
package e

var MsgFlags = map[int]string{
	SUCCESS			: "ok",
	ERROR			: "fail",
	InvalidParams   : "请求参数错误",

	ErrorAuthCheckTokenFail:        "Token鉴权失败",
	ErrorAuthCheckTokenTimeout:     "Token已超时",
	ErrorAuthToken:                 "Token生成失败",
	ErrorAuth:                      "Token错误",
	ErrorNotCompare:                "不匹配",
	ErrorDatabase:                  "数据库操作出错,请重试",

}

// GetMsg 获取状态码对应信息
func GetMsg(code int) string {
	msg, ok := MsgFlags[code]
	if ok {
		return msg
	}
	return MsgFlags[ERROR]
}
```

#### 主启动类

- main.go
```go
func main() { 
	conf.Init()
	r := routes.NewRouter()
	_ = r.Run(conf.HttpPort)
}
```

#### 参考资料

1、https://github.com/CocaineCong/TodoList