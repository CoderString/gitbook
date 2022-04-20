### Map结构

> map 是在 Go 中将值（value）与键（key）关联的内置类型。通过相应的键可以获取到值。

#### Map的创建

> make(map[type of key]type of value) 是创建 map 的语法。map 的零值是 nil。如果你想添加元素到 nil map 中，会触发运行时 panic。因此 map 必须使用 make 函数初始化。

```go
personSalary := make(map[string]int)
```

#### Map添加元素

```go
package main

import (
    "fmt"
)

func main() {
    personSalary := make(map[string]int)
    personSalary["steve"] = 12000
    personSalary["jamie"] = 15000
    personSalary["mike"] = 9000
    fmt.Println("personSalary map contents:", personSalary)
}
```

#### 获取Map中的元素

```go
package main

import (  
    "fmt"
)

func main() {
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    employee := "jamie"
    fmt.Println("Salary of", employee, "is", personSalary[employee])
    fmt.Println("Salary of joe is", personSalary["joe"])
}
```

> 判断map中是否包含key

```go
value, ok := map[key]
```

#### 遍历Map

```go
package main

import (
    "fmt"
)

func main() {
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("All items of a map")
    for key, value := range personSalary {
        fmt.Printf("personSalary[%s] = %d\n", key, value)
    }
}
```

#### 删除Map中的元素

> 删除 map 中 key 的语法是 delete(map, key)。这个函数没有返回值。

```go
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("map before deletion", personSalary)
    delete(personSalary, "steve")
    fmt.Println("map after deletion", personSalary)
}
```

#### 获取Map长度

```go
package main

import (
    "fmt"
)

func main() {
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("length is", len(personSalary))
}
```

#### Map是引用类型

> 和 slices 类似，map 也是引用类型。当 map 被赋值为一个新变量的时候，它们指向同一个内部数据结构。因此，改变其中一个变量，就会影响到另一变量。

```go
package main

import (
    "fmt"
)

func main() {
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("Original person salary", personSalary)
    newPersonSalary := personSalary
    newPersonSalary["mike"] = 18000
    fmt.Println("Person salary changed", personSalary)
}
```

#### Map相等性

> map 之间不能使用 == 操作符判断，== 只能用来检查 map 是否为 nil，判断两个 map 是否相等的方法是遍历比较两个 map 中的每个元素

```go
for k, v := range m1 {
    if v != m2[k] {
        return false
    }
}
```



#### 借助Map实现List & Set

> 实现List的思路先借助sort包对存放key的切片进行排序，然后按照顺序遍历就可以了。


> 实现Set

```go
func main() {
	// 切片
	sli := []int{1, 2, 3, 4, 5}
	// 一个 map
	set := make(map[int]bool)
	// 切片赋值给 map 的 key
	for _, v := range sli {
		set[v] = true
	}
	
	// 判定某个一个值在 「set」 中是否存在
	if set[3] {
		fmt.Println(set[3])
	}
	
	// set 的输出，即 map key 的输出
	for k, _ := range set {
		fmt.Println(k)
	}
}
```