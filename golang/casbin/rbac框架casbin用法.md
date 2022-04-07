### go整合rbac框架casbin

casbin是一个强大的rbac框架，主要进行权限管控，并支持多种访问控制模型

#### 基本用法

- 安装

```
go get github.com/casbin/casbin/v2
```

- 创建模型文件model.conf

```
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
```

- 创建策略文件policy.csv

p代表了具体的策略信息，比如alice访问/users的GET访问权限 g代表角色的策略信息，比如alice属于admin组，具有访问/users下的GET和POST权限

```
p, alice, /users, GET
p, bob, /users, POST
p, admin, /users, GET
p, admin, /users, POST
p, admin, /users, DELETE
g, alice, admin
```


- 入门案例

```
package main

import (
	"github.com/casbin/casbin/v2"
	"github.com/gin-gonic/gin"
	"github.com/sirupsen/logrus"
	"net/http"
)

type PermissionObj struct {
	User string `json:"user" form:"user"`
	Sub  string `json:"sub" form:"sub"`
}

var (
	e   *casbin.Enforcer
	err error
)

func init() {
	e, err = casbin.NewEnforcer("model.conf", "policy.csv")
	if err != nil {
		logrus.Fatal("load file failed, %v", err.Error())
	}
}

func main() {

	r := gin.Default()

	r.GET("/users", func(ctx *gin.Context) {
		sub := ctx.Query("username")
		obj := ctx.Request.URL.Path
		act := ctx.Request.Method
		checkPermission(ctx, sub, obj, act)
	})

	r.POST("/users", func(ctx *gin.Context) {
		sub := ctx.Query("username")
		obj := ctx.Request.URL.Path
		act := ctx.Request.Method
		checkPermission(ctx, sub, obj, act)
	})


	r.DELETE("/users", func(ctx *gin.Context) {
		// 先验证用户权限
		var p PermissionObj
		ctx.ShouldBind(&p)
		obj := ctx.Request.URL.Path
		act := ctx.Request.Method
		logrus.Infof("sub = %s obj = %s act = %s", p.User, obj, act)
		ok, err := e.Enforce(p.User, obj, act)
		if err != nil {
			ctx.String(http.StatusInternalServerError, "内部服务器错误")
			return
		}
		if !ok {
			ctx.String(http.StatusOK, "您没有权限")
			return
		}
		logrus.Println("权限验证通过")
		// 删除指定用户
		ok, err = e.DeleteUser(p.Sub)
		if err != nil {
			logrus.Info(err.Error())
			ctx.String(http.StatusInternalServerError, "内部服务器错误")
			return
		}
		if !ok {
			ctx.String(http.StatusOK, "删除用户失败")
			return
		}
		ctx.String(http.StatusOK, "删除用户成功")
		e.SavePolicy()
	})

	r.Run(":6000")
}

func checkPermission(ctx *gin.Context, sub, obj, act string) {
	logrus.Infof("sub = %s obj = %s act = %s", sub, obj, act)
	ok, err := e.Enforce(sub, obj, act)
	if err != nil {
		logrus.Print("enforce failed %s", err.Error())
		ctx.String(http.StatusInternalServerError, "内部服务器错误")
		return
	}
	if !ok {
		logrus.Println("权限验证不通过")
		ctx.String(http.StatusOK, "权限验证不通过")
		return
	}
	logrus.Println("权限验证通过")
	ctx.String(http.StatusOK, "权限验证通过")
}
```

#### 接口访问测试

- 验证访问权限

```
curl --location --request POST 'localhost:6000/users?username=bob'
```


- 验证删除权限

```
curl --location --request DELETE 'localhost:6000/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "user" : "admin",
    "sub" : "bob"
}'
```


#### casbin模型生成规则

1、在线规则生成：https://casbin.org/editor/

2、使用参考文档：https://casbin.org/docs/en/get-started

3、中文参考文档：https://casbin.org/docs/zh-CN/get-started