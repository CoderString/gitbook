## viper解析各种格式配置文件

> viper是一个第三方的配置文件解析库，可以帮助我们快速的解析各种格式的配置文件，下面讲简单的说明下viper如何解析各种配置文件。

```go
go get github.com/spf13/viper
```

### viper解析yaml

#### yaml解析数据

```go
app:
  name: viper解析
  version: v1.0
mysql:
  host: 127.0.0.1
  port: 3306
  user: root
  password: 123456
```

#### 读取程序

```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

type APP struct {
	Name string `yaml:"name"`
	Version string `yaml:"version""`
}

type MySQL struct {
	Host string `yaml:"host"`
	Port string `yaml:"port"`
	User string `yaml:"password"`
}

type YamlConfig struct {
	App APP `yaml:"app"`
	MySQL MySQL `yaml:"mysql"`
}

var config *viper.Viper
var yamlConfig YamlConfig

func Init() {
	config := viper.New()
	config.SetConfigFile("vipers/yaml/test.yaml")
	if err := config.ReadInConfig(); err != nil {
		if _, ok := err.(viper.ConfigFileNotFoundError); ok {
			fmt.Printf("配置文件未找到%v\n", err)
			return
		}
		fmt.Printf("配置文件解析错误%v\n", err)
		return
	}
	config.Unmarshal(&yamlConfig)
}

func main() {

	Init()
	fmt.Println(yamlConfig.App)
	fmt.Println(yamlConfig.App.Name)

	fmt.Println(yamlConfig.MySQL)
	fmt.Println(yamlConfig.MySQL.User)
}
```

#### 监控并重新读取配置文件

> 监控导入的包是：github.com/fsnotify/fsnotify

```
viper.WatchConfig()
viper.OnConfigChange(func(e fsnotify.Event) {
  // 配置文件发生变更之后会调用的回调函数
	fmt.Println("Config file changed:", e.Name)
})
```


### viper解析json文件

> 方式一

```go
func init() {
    path, _ := os.Getwd()
    cfg := viper.New()
    cfg.AddConfigPath(path + "/config")
    cfg.SetConfigName("cfg")
    cfg.SetConfigType("json")
    if err := cfg.ReadInConfig(); err != nil { 
        panic(err)
    }
    if err := cfg.Unmarshal(&GlobalConfig); err != nil { 
        panic("读取配置文件出错")
    }
    viper.WatchConfig()
    fmt.Println(GlobalConfig)
}
```

> 方式二

```go
func main() {
	v := viper.New()
	v.SetConfigType("json") 
	var jsonExample = []byte(`
		{
		  "port": 10666,
		  "mysql": {
			"url": "(127.0.0.1:3306)/biezhi",
			"username": "root",
			"password": "123456"
		  },
		  "redis": ["127.0.0.1:6377", "127.0.0.1:6378", "127.0.0.1:6379"],
		  "smtp": {
			"enable": true,
			"addr": "mail_addr",
			"username": "mail_user",
			"password": "mail_password",
			"to": ["xxx@gmail.com", "xxx@163.com"]
		  }
		}
		`)
	//创建io.Reader
	v.ReadConfig(bytes.NewBuffer(jsonExample))

	fmt.Println("获取配置文件的port", v.GetInt("port"))
	fmt.Println("获取配置文件的mysql.url", v.GetString(`mysql.url`))
	fmt.Println("获取配置文件的mysql.username", v.GetString(`mysql.username`))
	fmt.Println("获取配置文件的mysql.password", v.GetString(`mysql.password`))
	fmt.Println("获取配置文件的redis", v.GetStringSlice("redis"))
	fmt.Println("获取配置文件的smtp", v.GetStringMap("smtp"))
}
```

### viper解析Env中的变量

> 针对Env中的变量其实是可以通过go原生的类库来进行读取的：getenv := os.Getenv("JAVA_HOME")


```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

func main() {
	// 绑定全部环境变量。
	viper.AutomaticEnv()
	fmt.Println("HOME: ", viper.Get("HOME"))
	if env := viper.Get("JAVA_HOME"); env == nil {
		println("error!")
	} else {
		fmt.Printf("%#v\n", env)
	}
}
```

### viper保存运行时的数据

```go
package main

import (
	"github.com/spf13/viper"
	"log"
)

func main() {
	viper.SetConfigName("config")
	viper.SetConfigType("toml")
	viper.AddConfigPath(".")

	viper.Set("app_name", "awesome web")
	viper.Set("log_level", "DEBUG")
	viper.Set("mysql.ip", "127.0.0.1")
	viper.Set("mysql.port", 3306)
	viper.Set("mysql.user", "root")
	viper.Set("mysql.password", "123456")
	viper.Set("mysql.database", "awesome")

	viper.Set("redis.ip", "127.0.0.1")
	viper.Set("redis.port", 6381)

	err := viper.SafeWriteConfig()
	if err != nil {
		log.Fatal("write config failed: ", err)
	}
}
```

### 参考资料

1、https://blog.51cto.com/u_15301988/3127353