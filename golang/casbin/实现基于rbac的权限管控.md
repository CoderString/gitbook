### Casbin 实现基于rbac的权限管控

> 之前的文章已经简单的介绍了关于casbin的基本使用案例，但是从实际的项目的角度而言，这些是远远不能实现系统的权限分级的，因此接下来我们会针对如何更好的使用casbin来实现基于rbac的权限管控展开讲解。

#### 什么是Casbin

> Casbin 是一个强大的、高效的开源访问控制框架，其权限管理机制支持多种访问控制模型。

**Casbin可以:**

- 支持自定义请求的格式，默认的请求格式为{subject, object, action}。
- 具有访问控制模型model和策略policy两个核心概念。
- 支持RBAC中的多层角色继承，不止主体可以有角色，资源也可以具有角色。
- 支持内置的超级用户 例如：root 或 administrator。超级用户- 可以执行任何操作而无需显式的权限声明。
- 支持多种内置的操作符，如 keyMatch，方便对路径式的资源进行管理，如 /foo/bar 可以映射到 /foo*

**Casbin不能:**

- 身份认证 authentication（即验证用户的用户名和密码），Casbin 只负责访问控制。应该有其他专门的组件负责身份认证，然后由 Casbin 进行访问控制，二者是相互配合的关系。
- 管理用户列表或角色列表。 Casbin 认为由项目自身来管理用户、角色列表更为合适， 用户通常有他们的密码，但是 Casbin 的设计思想并不是把它作为一个存储密码的容器。 而是存储RBAC方案中用户和角色之间的映射关系。


#### Casbin配置文件

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

**域租户的角色定义**

> 在Casbin中的RBAC角色可以是全局或是基于特定于域的。 特定域的角色意味着当用户处于不同的域/租户群体时，用户所表现的角色也不尽相同。 这对于像云服务这样的大型系统非常有用，因为用户通常分属于不同的租户群体。

支持域/租户的RBAC

```
[request_definition]
r = sub, dom, obj, act

[policy_definition]
p = sub, dom, obj, act

[role_definition]
g = _, _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub, r.dom) && r.dom == p.dom && r.obj == p.obj && r.act == p.act
```

#### 初始化Casbin

```go
package casbin

import (
	"errors"
	"github.com/casbin/casbin/v2"
	"github.com/casbin/casbin/v2/model"
	"github.com/casbin/gorm-adapter/v3"
	"go.uber.org/zap"
	"yt.yin/core/global"
)

func InitCasbin(casbinConfPath ...string) (err error) {
	// Gorm适配器
	adapter, err := gormadapter.NewAdapterByDB(global.DB)
	if err != nil {
		global.LOG.Error("创建Casbin Gorm适配器错误：", zap.Any("err", err))
		return  errors.New("Casbin Gorm适配器错误：" + err.Error())
	}
	global.LOG.Info("创建Casbin Gorm适配器成功")
	if len(casbinConfPath) > 0 {
		// 通过ORM新建一个执行者
		global.Enforcer, err = casbin.NewEnforcer(casbinConfPath[0], adapter)
		if err != nil {
			global.LOG.Error("新建Casbin执行者异常：", zap.Any("err", err))
			return  errors.New("新建Casbin执行者异常：" + err.Error())
		}
	} else {
		m, _ := model.NewModelFromString(`
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
			`)
		global.Enforcer, err = casbin.NewEnforcer(m, adapter)
		if err != nil {
			global.LOG.Error("新建Casbin执行者异常：", zap.Any("err", err))
			return  errors.New("新建Casbin执行者异常：" + err.Error())
		}
	}

	// 导入访问策略
	err = global.Enforcer.LoadPolicy()
	if err != nil {
		global.LOG.Error("导入访问策略异常：", zap.Any("err", err))
		return  errors.New("导入访问策略异常：" + err.Error())
	}
	return  nil
}
```

#### rbac相关API

##### 用户角色

- 为用户添加角色

```go
e.AddRoleForUser("user", "role")
```

- 为用户添加多个角色

```go
roles := []string{
    "role1",
    "role2",
    "role3",
    "role4",
    "role5",
	}
_, err = Enforcer.AddRolesForUser("user1", roles)
if err != nil {
    log.Fatalln("为用户批量添加角色失败")
}
```

- 删除用户指定角色

```go
e.DeleteRoleForUser("user", "role")
```

- 删除用户的所有角色

```go
e.DeleteRolesForUser("user")
```

- 获取用户的所有角色

```go
res, err := Enforcer.GetRolesForUser("user1")
if err != nil {
	log.Fatalln("获取用户角色失败")
} else {
	log.Println("用户角色：", res)
}
```

- 获取有角色的用户

```go
res := e.GetUsersForRole("role")
```

- 确定用户是否具有角色

```go
res := e.HasRoleForUser("user", "role")
```

##### 角色权限

- 添加API权限

```go
_, err = Enforcer.AddPolicy("role1", "/api/5", "POST")
if err != nil {
	log.Fatalln("添加API权限（策略）失败")
} else {
	log.Println("添加API权限（策略）成功")
}
```

- 批量添加API权限

```go
rules := [][]string{
		{"role1", "/api/1", "POST"},
		{"role2", "/api/1", "POST"},
		{"role3", "/api/1", "POST"},
		{"role1", "/api/2", "POST"},
		{"role1", "/api/3", "POST"},
		{"role1", "/api/4", "POST"},
		{"role1", "/api/5", "POST"},
	}

_, err = Enforcer.AddPolicies(rules)
if err != nil {
	log.Fatalln("批量添加API权限（策略）失败")
} else {
	log.Println("批量添加API权限（策略）成功")
}
```

- 批量删除API权限

```go
_, err = Enforcer.DeletePermissionsForUser("role1")
if err != nil {
	log.Fatalln("批量删除API权限（策略）失败")
} else {
	log.Println("批量删除API权限（策略）成功")
}
```

- API权限查询

```go
answer := Enforcer.GetFilteredPolicy(0, "", "/api/4", "POST")
log.Println("answer:", answer)
```

- 查询角色的API权限列表

```go
result := Enforcer.GetPermissionsForUser("role")
log.Println("策略:", result)
```

- 查询用户的API权限列表

```go
result ,err= Enforcer.GetImplicitPermissionsForUser("user")
log.Println("策略:", result)
```

- 权限校验

```go
sub := "user1"
obj := "/api/2"
act := "POST"
ok, err := Enforcer.Enforce(sub, obj, act)
if ok {
	fmt.Println("通过权限")
} else || err!=nil {
	fmt.Println("权限没有通过")
}
```


##### 域内基于角色的访问控制 API

- 为域内的用户添加角色

> 如果用户已经拥有该角色，则返回false

```go
ok, err := e.AddRoleForUserInDomain("用户", "角色", "域")
```

- 在域内删除用户的角色

> 如果用户没有任何角色，则返回false

```go
ok, err := e.DeleteRoleForUserInDomain("用户", "角色", "域")
```

- 在域内为角色批量添加API权限

```go
rules := [][]string{
    { "role1", "domain1", "www.baidu.com", "GET"},
    { "role2", "domain1", "www.baidu.com", "GET"},
    { "role3", "domain1", "www.baidu.com", "GET"},
    { "role4", "domain1", "www.baidu.com", "GET"},
    { "role5", "domain1", "www.baidu.com", "GET"},
    { "role6", "domain1", "www.baidu.com", "GET"},
    { "role7", "domain1", "www.baidu.com", "GET"},
}
global.Enforcer.AddPolicies(rules)
```

#### 参考资料

1、https://casbin.org/docs/zh-CN/rbac-with-domains

2、https://blog.csdn.net/qq_29537269/article/details/121951328
