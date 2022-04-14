### go原生实现GET、POST、DELETE、PUT请求

> go原生实现GET、POST、DELETE、PUT相关的访问不借助第三方包，总结如下：

#### GET

##### 直接访问

> 不适合需要header携带参数的方式

```go
import (
	"net/http"
)

response, err := http.Get("https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1")
```

##### 自定义参数访问

> header中携带参数的方式

```go
import (
	"net/http"
)

url := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"
req, _ := http.NewRequest("GET", url, nil)
req.Header.Add("Authorization", "xxxx")
response, err := http.DefaultClient.Do(req)
```

#### POST

##### kv形式传参

> 方式一
```go
import (
	"net/http"
    "strings"
)

url := "https://blog.csdn.net/zyndev"
payload := strings.NewReader("a=111")
response, err := http.Post(url, "application/x-www-form-urlencoded", payload)
```

> 方式二

```go

import (
	"net/http"
	"net/url"
)

targetUrl := "https://blog.csdn.net/zyndev"
payload := url.Values{"key":{"value"}, "id": {"123"}}
response, err := http.PostForm(targetUrl, payload)
```

##### json形式传参

```go
targetUrl := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"
payload := strings.NewReader("{\"name\":\"张瑀楠\"}")
req, _ := http.NewRequest("POST", targetUrl, payload)
req.Header.Add("Content-Type", "application/json")
response, err := http.DefaultClient.Do(req)
```

#### PUT

```go
targetUrl := "https://b959e645-00ae-4bc3-8a55-7224d08b1d91.mock.pstmn.io/user/1"
payload := strings.NewReader("{\"name\":\"张瑀楠\"}")
req, _ := http.NewRequest("PUT", targetUrl, payload)
req.Header.Add("Content-Type", "application/json")
response, err := http.DefaultClient.Do(req)
```

#### DELETE

```go
targetUrl := "https://ddbc5ffb-c596-4f78-a99d-a6ea93bdc14f.mock.pstmn.io/user/1"
req, _ := http.NewRequest("DELETE", targetUrl, nil)
req.Header.Add("Authorization", "xxxx")
response, err := http.DefaultClient.Do(req)
```

#### response处理

```go

... // 一堆请求方法构建方式
response, err := http.DefaultClient.Do(req)

if err != nil {
	// 错误逻辑处理
}

// 这步是必要的，防止以后的内存泄漏，切记
defer response.Body.Close() 

fmt.Println(response.StatusCode)  	// 获取状态码 
fmt.Println(response.Status)		// 获取状态码对应的文案
fmt.Println(response.Header)		// 获取响应头
body, _ := ioutil.ReadAll(response.Body) // 读取响应 body, 返回为 []byte
fmt.Println(string(body))			// 转成字符串看一下结果
```


#### 参考资料

1、https://blog.csdn.net/zyndev/article/details/113745637