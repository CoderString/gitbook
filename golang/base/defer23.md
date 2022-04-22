### defer使用

> defer 语句的用途是：含有 defer 语句的函数，会在该函数将要返回之前，调用另一个函数。

```go
package main

import (  
    "fmt"
)

func finished() {  
    fmt.Println("Finished finding largest")
}

func largest(nums []int) {  
    defer finished()
    fmt.Println("Started finding largest")
    max := nums[0]
    for _, v := range nums {
        if v > max {
            max = v
        }
    }
    fmt.Println("Largest number in", nums, "is", max)
}

func main() {  
    nums := []int{78, 109, 2, 563, 300}
    largest(nums)
}
```

> 上面的程序很简单，就是找出一个给定切片的最大值。largest 函数接收一个 int 类型的切片作为参数，然后打印出该切片中的最大值。largest 函数的第一行的语句为 defer finished()。这表示在 finished() 函数将要返回之前，会调用 finished() 函数。运行该程序，你会看到有如下输出：

> Started finding largest<br/>
> Largest number in [78 109 2 563 300] is 563  <br/>
> Finished finding largest<br/>


#### 延迟方法

> defer 不仅限于函数的调用，调用方法也是合法的。

```go
package main

import (  
    "fmt"
)


type person struct {  
    firstName string
    lastName string
}

func (p person) fullName() {  
    fmt.Printf("%s %s",p.firstName,p.lastName)
}

func main() {  
    p := person {
        firstName: "John",
        lastName: "Smith",
    }
    defer p.fullName()
    fmt.Printf("Welcome ")  
}
```

#### 实参取值

```go
package main

import (  
    "fmt"
)

func printA(a int) {  
    fmt.Println("value of a in deferred function", a)
}
func main() {  
    a := 5
    defer printA(a)
    a = 10
    fmt.Println("value of a before deferred function call", a)

}
```

> 输出：<br/>
> value of a before deferred function call 10  <br/>
> value of a in deferred function 5<br/>

#### defer栈

> 当一个函数内多次调用 defer 时，Go 会把 defer 调用放入到一个栈中，随后按照后进先出（Last In First Out, LIFO）的顺序执行。


```go
package main

import (  
    "fmt"
)

func main() {  
    name := "Naveen"
    fmt.Printf("Orignal String: %s\n", string(name))
    fmt.Printf("Reversed String: ")
    for _, v := range []rune(name) {
        defer fmt.Printf("%c", v)
    }
}
```