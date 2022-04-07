### flag包的用法

go的flag包的基本使用方法是在启动项目的时候从终端传入一些初始化的变量的值，简单的使用方式如下：


```
package main

import (
	"flag"
	"fmt"
)

var(
	v1 string
	v2 string
	v3 string
)

func init() {
	flag.StringVar(&v1, "v1", "", "v1通过flag方式注入的值")
	flag.StringVar(&v2, "v2", "", "v2通过flag方式注入的值")
	flag.StringVar(&v3, "v3", "", "v3通过flag方式注入的值")
	flag.Parse()
	fmt.Println(v1)
	fmt.Println(v2)
	fmt.Println(v3)
}

func main() {
}
```

终端运行的时候输入：go run Test.go -v1 r1 -v2 r2 -v3 r3
输出：r1、r2、r3



### 常见的应用场景

常常在启动一些应用的时候，进行一些变量的初始化的赋值。