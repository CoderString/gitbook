### go整合Redis

redis作为缓存的中间件，已经成为了事实上NOSQL应用的标准，鉴于次有必要针对go语言的Redis实现go-redis展开讲解。


#### 安装

> go get -u github.com/go-redis/redis

#### 常见的全局命令

```
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"time"
)
// 定义一组常量
const (
	REDIS_IP   = "127.0.0.1"
	REDIS_PORT = "6379"
	REDIS_PWD  = ""
	REDIS_DB   = 0
)
// 定义一个redis.client类型的变量
var client *redis.Client
// 初始化函数
func init() {
	// 实例化一个redis客户端
	client = redis.NewClient(&redis.Options{
        Addr:     REDIS_IP + ":" + REDIS_PORT, // ip:port
		Password: REDIS_PWD,				   // redis连接密码
		DB:       REDIS_DB,					   // 选择的redis库
		PoolSize: 20,						   // 设置连接数,默认是10个连接
	})
}
func main() {
	// redis 全局命令
	// 获取redis所有的键,返回包含所有键的slice
	keys := client.Keys("*").Val()
	fmt.Println(keys)
	// 获取redis中的有多少个键,返回整数
	size := client.DbSize().Val()
	fmt.Println(size)
	// 判断一个键是否存在,有一个存在返回整数1,有两个存在返回整数2...
	exist := client.Exists("age","name").Val()
	fmt.Println(exist)
	// 删除键,删除成功返回删除的数,删除失败返回0
	del := client.Del("unknownKey").Val()
	fmt.Println(del)
	// 查看键的有效时间
	ttl := client.TTL("age").Val()
	fmt.Println(ttl)
	// 给键设置有效时间,设置成功返回true,失败返回false
	expire := client.Expire("age",time.Second*86400).Val()
	fmt.Println(expire)
	// 查看键的类型(string,hash,list,set,zset...)
	Rtype := client.Type("store:finish:bill:list").Val()
	fmt.Println(Rtype)
	// 给键重命令,成功返回true,失败false
	Rname := client.RenameNX("age","newAge").Val()
	fmt.Println(Rname)
	// 从redis中随机返回一个键
	key := client.RandomKey().Val()
	fmt.Println(key)

}
```

#### string类型操作

```
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"time"
)

// 定义一组常量
const (
	REDIS_IP   = "127.0.0.1"
	REDIS_PORT = "6379"
	REDIS_PWD  = ""
	REDIS_DB   = 0
)

// 定义一个redis.client类型的变量
var client *redis.Client

// 初始化函数
func init() {
	// 实例化一个redis客户端
	client = redis.NewClient(&redis.Options{
		Addr:     REDIS_IP + ":" + REDIS_PORT, // ip_port
		Password: REDIS_PWD,                   // redis连接密码
		DB:       REDIS_DB,                    // 选择的redis库
		PoolSize: 20,                          // 设置连接数,默认是10个连接
	})
}
func main() {
	defer client.Close()
	defer func() {
		if err := recover(); err != nil {
			fmt.Println(err)
		}
	}()
	// 设置一组键值对,并社会有效期
	set1 := client.Set("age", 10, time.Hour*24).Val()
	fmt.Println(set1) // OK
	//设置一组键值对,设置的键不存在的时候才能设置成功
	set2 := client.SetNX("age", "20", time.Hour*12).Val()
	fmt.Println(set2) // false
	//设置一组键值对,设置的键必须存在的时候才能设置成功
	set3 := client.SetXX("age", "30", time.Second*86400).Val()
	fmt.Println(set3) // true
	// 批量设置
	set4 := client.MSet("age1", "40", "age2", "50").Val()
	fmt.Println(set4) // OK
	// 获取一个键的值
	get1 := client.Get("age2").Val()
	fmt.Println(get1) // 50
	// 批量获取,获取成功返回slice类型的结果数据
	get2 := client.MGet("age", "age1", "age2").Val()
	fmt.Println(get2) // [30 40 50]
	// 对指定的键进行自增操作
	incr1 := client.Incr("age").Val()
	fmt.Println(incr1) // 31
	// 对指定键进行自减操作
	decr1 := client.Decr("age1").Val()
	fmt.Println(decr1) //39
	// 自增指定的值
	incr2 := client.IncrBy("age", 10).Val()
	fmt.Println(incr2) // 41
	// 自减指定的值
	decr2 := client.DecrBy("age1", 10).Val()
	fmt.Println(decr2) // 29
	// 在key后面追加指定的值,返回字符串的长度
	append1 := client.Append("age2", "abcd").Val()
	fmt.Println(append1) // 6
	// 获取一个键的值得长度
	strlen1 := client.StrLen("age2").Val()
	fmt.Println(strlen1) //6
	// 设置一个键的值,并返回原有的值
	getset1 := client.GetSet("age2", "hello golang").Val()
	fmt.Println(getset1) // 50abcd
	// 设置键的值,在指定的位置
	_ = client.SetRange("age2", 0, "H")
	fmt.Println(client.Get("age2").Val()) // Hello golang
	// 截取一个键的值的部分,返回截取的部分
	newStr := client.GetRange("age2", 6, 11).Val()
	fmt.Println(newStr) //golang
}
```

#### hash类型操作

```
package main

import (
	"fmt"
	"github.com/go-redis/redis"
)

// 定义一组常量
const (
	REDIS_IP   = "127.0.0.1"
	REDIS_PORT = "6379"
	REDIS_PWD  = ""
	REDIS_DB   = 0
)

// 定义一个redis.client类型的变量
var client *redis.Client

// 初始化函数
func init() {
	// 实例化一个redis客户端
	client = redis.NewClient(&redis.Options{
		Addr:     REDIS_IP + ":" + REDIS_PORT, // ip_port
		Password: REDIS_PWD,                   // redis连接密码
		DB:       REDIS_DB,                    // 选择的redis库
		PoolSize: 20,                          // 设置连接数,默认是10个连接
	})
}
func main() {
	defer client.Close()
	defer func() {
		if err := recover(); err != nil {
			fmt.Println(err)
		}
	}()
	key := "account"
	field1 := "name"
	fields := map[string]interface{}{
		"addr":"beijing",
		"age":99,
		"skills":"golang",
		"demo1":"aaa",
		"demo2":"bbb",
	}
	//hash 设置一个键的field
	_ = client.HSet(key,field1,"zhangsan")
	// hash 批量设置 ,第二个参数是map类型
	status := client.HMSet(key,fields).Val()
	fmt.Println(status) //ok
	// hash 删除键的field,返回删除field的个数
	 _ = client.HDel(key,"demo2").Val()
	 //hash 获取field的值
	name := client.HGet(key,"name").Val()
	fmt.Println(name) //zhangsan
	//hash 获取多个field值,返回slice
	values := client.HMGet(key,"name","age").Val()
	fmt.Println(values)//[zhangsan 99]
	//hash 获取所有的值 返回map
	valueAll := client.HGetAll(key).Val()
	fmt.Println(valueAll) //map[addr:beijing age:99 demo1:aaa name:zhangsan skills:golang]
	// hash 获取所有field 返回slice
	fs := client.HKeys(key).Val()
	fmt.Println(fs) //[name addr age skills demo1]
	// hash 获取所有filed的值 返回slice
	vs := client.HVals(key).Val()
	fmt.Println(vs) //[zhangsan beijing 99 golang aaa]
	// 判断一个filed是否存在 返回bool
	e := client.HExists(key,"skills").Val()
	fmt.Println(e) //true
	// hash field自增
	n := client.HIncrBy(key,"age",1).Val()
	fmt.Println(n) //100
}
```

#### List类型操作

```
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"strconv"
)

// 定义一组常量
const (
	REDIS_IP   = "127.0.0.1"
	REDIS_PORT = "6379"
	REDIS_PWD  = ""
	REDIS_DB   = 0
)

// 定义一个redis.client类型的变量
var client *redis.Client

// 初始化函数
func init() {
	// 实例化一个redis客户端
	client = redis.NewClient(&redis.Options{
		Addr:     REDIS_IP + ":" + REDIS_PORT, // ip_port
		Password: REDIS_PWD,                   // redis连接密码
		DB:       REDIS_DB,                    // 选择的redis库
		PoolSize: 20,                          // 设置连接数,默认是10个连接
	})
}
func main() {
	defer client.Close()
	defer func() {
		if err := recover(); err != nil {
			fmt.Println(err)
		}
	}()
	key := "demo"
	client.Del(key)
	for i := 0; i < 5; i++ {
		client.LPush(key, "e-"+strconv.Itoa(i))
	}
	// 获取list 长度
	length := client.LLen(key).Val()
	fmt.Println(length) //5
	// 获取指定下标元素
	value1 := client.LIndex(key, 0).Val()
	fmt.Println(value1) //e-4
	// 获取所有元素
	vs := client.LRange(key, 0, -1).Val()
	fmt.Println(vs) //[e-4 e-3 e-2 e-1 e-0]
	// 修改指定下标的元素值
	status := client.LSet(key, 0, "golang").Val()
	fmt.Println(status) //ok
	// 从右边插入元素
	client.RPush(key, "e-right")
	// 从左边插入元素
	client.LPush(key, "e-left")
	// 从list最右边弹出元素
	v1 := client.RPop(key).Val()
	fmt.Println(v1) // e-right
	// 从list最左边弹出元素
	v2 := client.LPop(key).Val()
	fmt.Println(v2) // e-left
	// 删除指定元素
	n := client.LRem(key, 0, "e-3").Val()
	fmt.Println(n) //1
	status1 := client.LTrim(key, 0, 1).Val()
	fmt.Println(status1) //ok
}
```

#### Set类型操作

```
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"strconv"
)

// 定义一组常量
const (
	REDIS_IP   = "127.0.0.1"
	REDIS_PORT = "6379"
	REDIS_PWD  = ""
	REDIS_DB   = 0
)

// 定义一个redis.client类型的变量
var client *redis.Client

// 初始化函数
func init() {
	// 实例化一个redis客户端
	client = redis.NewClient(&redis.Options{
		Addr:     REDIS_IP + ":" + REDIS_PORT, // ip_port
		Password: REDIS_PWD,                   // redis连接密码
		DB:       REDIS_DB,                    // 选择的redis库
		PoolSize: 20,                          // 设置连接数,默认是10个连接
	})
}
func main() {
	defer client.Close()
	defer func() {
		if err := recover(); err != nil {
			fmt.Println(err)
		}
	}()
	key := "sdemo"
	key1 := "sdemo1"
	client.Del(key)
	client.Del(key1)
	for i := 0; i < 6; i++ {
		// set 类型添加元素
		client.SAdd(key, "ele-"+strconv.Itoa(i))
	}
	for i := 3; i < 9; i++ {
		// set 类型添加元素
		client.SAdd(key1, "ele-"+strconv.Itoa(i))
	}
	// 计算key中的元素个数
	n1 := client.SCard(key).Val()
	fmt.Println(n1) //5
	// 判断元素是否在集合中
	e1 := client.SIsMember(key, "ele-0").Val()
	fmt.Println(e1) // true
	// 随机从集合中返回一个元素
	v1 := client.SRandMember(key).Val()
	fmt.Println(v1)
	// 随机返回指定个数的元素,返回包含元素的slice
	v2 := client.SRandMemberN(key, 3).Val()
	fmt.Println(v2)
	// 获取集合中的所有元素,无序的slice
	v3 := client.SMembers(key).Val()
	fmt.Println(v3) //[ele-1 ele-0 ele-3 ele-2 ele-4]
	// 从集合中随机弹出一个元素
	v4 := client.SPop(key).Val()
	fmt.Println(v4)
	// 从集合中删除元素
	n2 := client.SRem(key, "ele-5").Val()
	fmt.Println(n2) //1
	// 求多个集合的交集
	s1 := client.SInter(key, key1).Val()
	fmt.Println(s1) // [ele-3 ele-4]
	//求多个集合的并集
	s2 := client.SUnion(key, key1).Val()
	fmt.Println(s2) // [ele-3 ele-5 ele-1 ele-2 ele-4 ele-8 ele-6 ele-0 ele-7]
	// 求多个集合的差集
	s3 := client.SDiff(key, key1).Val()
	fmt.Println(s3) // [ele-0 ele-1]
	// 将多个交集结果存为一个新的集合
	s4 := client.SInterStore("sdemo2", key, key1).Val()
	fmt.Println(s4)                              //2
	fmt.Println(client.SMembers("sdemo2").Val()) // [ele-4 ele-3]
	// 将多个交集的并集结果存为新的集合
	s5 := client.SUnionStore("sdemo3", key, key1).Val()
	fmt.Println(s5)                              //8
	fmt.Println(client.SMembers("sdemo3").Val()) // [ele-3 ele-5 ele-1 ele-4 ele-7 ele-8 ele-6 ele-0]
	// 将多个差集的结果存为新的集合
	s6 := client.SDiffStore("sdemo4", key, key1).Val()
	fmt.Println(s6)                              //2
	fmt.Println(client.SMembers("sdemo4").Val()) // [ele-0 ele-1]
}
```

#### Zset类型操作

```
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"math/rand"
	"strconv"
)

// 定义一组常量
const (
	REDIS_IP   = "127.0.0.1"
	REDIS_PORT = "6379"
	REDIS_PWD  = ""
	REDIS_DB   = 0
)

// 定义一个redis.client类型的变量
var client *redis.Client

// 初始化函数
func init() {
	// 实例化一个redis客户端
	client = redis.NewClient(&redis.Options{
		Addr:     REDIS_IP + ":" + REDIS_PORT, // ip_port
		Password: REDIS_PWD,                   // redis连接密码
		DB:       REDIS_DB,                    // 选择的redis库
		PoolSize: 20,                          // 设置连接数,默认是10个连接
	})
}
func randString(n int) string {
	s := "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
	b := make([]byte, n)
	for i := range b {
		b[i] = s[rand.Intn(len(s))]
	}
	return string(b)
}
func main() {
	defer client.Close()
	defer func() {
		if err := recover(); err != nil {
			fmt.Println(err)
		}
	}()
	key := "zdemo"
	key1 := "zdemo1"
	client.Del(key)
	client.Del(key1)
	for i := 0; i < 10; i++ {
		score := float64(rand.Intn(100) - 50)
		member := "golang-" + strconv.Itoa(i)
		data := &redis.Z{
			score,
			member,
		}
		// 向有序集合中添加成员
		client.ZAdd(key, data).Val()
	}
	for i := 0; i < 5; i++ {
		score := float64(rand.Intn(100) - 50)
		member := "golang-" + strconv.Itoa(i)
		data := &redis.Z{
			score,
			member,
		}
		// 向有序集合中添加成员
		client.ZAdd(key1, data).Val()
	}
	// 计算成员个数
	n1 := client.ZCard(key).Val()
	fmt.Println(n1) //10
	// 获取成员分数
	s1 := client.ZScore(key, "golang-6").Val()
	fmt.Println(s1) // -25
	// 修改成员分数
	v1 := client.ZIncrBy(key, 60.00, "golang-6").Val()
	fmt.Println(v1)
	// 从低到高返回排名
	s2 := client.ZRank(key, "golang-6").Val()
	fmt.Println(s2) //8
	// 从高到低返回排名
	s3 := client.ZRevRank(key, "golang-6").Val()
	fmt.Println(s3) //1
	// 获取指定范围的成员排名,从低到高排名
	s4 := client.ZRange(key, 0, n1-5).Val()
	fmt.Println(s4) //[golang-9 golang-5 golang-7 golang-2 golang-8 golang-3]
	// 获取指定范围的成员排名,从高到低排名
	s5 := client.ZRevRange(key, 0, n1-5).Val()
	fmt.Println(s5) //[golang-1 golang-6 golang-4 golang-0 golang-3 golang-8]
	// 删除成员
	v2 := client.ZRem(key, "golang-6").Val()
	fmt.Println(v2) //1
	// 计算两个有序集合的交集
	key2 := "zdemo2"
	kslice := []string{key, key1}
	wslice := []float64{1.00, 1.00}
	z := &redis.ZStore{
		kslice,
		wslice,
		"SUM",
	}
	r1 := client.ZInterStore(key2, z).Val()
	fmt.Println(r1) //5
	// 计算两个有序集合的并集
	key3 := "zdemo3"
	r2 := client.ZUnionStore(key3,z).Val()
	fmt.Println(r2) // 9
}
```

#### 发布和订阅


> 订阅

```
package main

import (
	"fmt"
	"github.com/go-redis/redis"
)

// 定义一组常量
const (
	REDIS_IP   = "127.0.0.1"
	REDIS_PORT = "6379"
	REDIS_PWD  = ""
	REDIS_DB   = 0
)

// 定义一个redis.client类型的变量
var client *redis.Client

// 初始化函数
func init() {
	// 实例化一个redis客户端
	client = redis.NewClient(&redis.Options{
		Addr:     REDIS_IP + ":" + REDIS_PORT, // ip_port
		Password: REDIS_PWD,                   // redis连接密码
		DB:       REDIS_DB,                    // 选择的redis库
		PoolSize: 20,                          // 设置连接数,默认是10个连接
	})
}
func main() {
	defer client.Close()
	defer func() {
		if err := recover(); err != nil {
			fmt.Println(err)
		}
	}()
	// 订阅多个频道
	sub := client.PSubscribe("news", "it", "sports", "shopping")
	_, err := sub.Receive()
	if err != nil {
		fmt.Println(err)
	}
	// 消息通道
	ch := sub.Channel()
	//  从通道中读取消息
	for message := range ch {
		fmt.Println(message.Channel, message.Payload)
	}
}
```

> 发布

```
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"math/rand"
)

// 定义一组常量
const (
	REDIS_IP   = "127.0.0.1"
	REDIS_PORT = "6379"
	REDIS_PWD  = ""
	REDIS_DB   = 0
)

// 定义一个redis.client类型的变量
var client *redis.Client

// 初始化函数
func init() {
	// 实例化一个redis客户端
	client = redis.NewClient(&redis.Options{
		Addr:     REDIS_IP + ":" + REDIS_PORT, // ip_port
		Password: REDIS_PWD,                   // redis连接密码
		DB:       REDIS_DB,                    // 选择的redis库
		PoolSize: 20,                          // 设置连接数,默认是10个连接
	})
}

// 生成指定长度的随机字符串
func randString(n int) string {
	s := "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890"
	b := make([]byte, n)
	for i := range b {
		b[i] = s[rand.Intn(len(s))]
	}
	return string(b)
}
func main() {
	defer client.Close()
	defer func() {
		if err := recover(); err != nil {
			fmt.Println(err)
		}
	}()
	var data string
	var channels []string = []string{"news", "it", "sports", "shopping"}
	for {
		fmt.Printf("please input some data:")
		fmt.Scanln(&data)
		// 退出
		if data == "quit" {
			break
		}
		channel := channels[rand.Intn(4)]
		// 向一个频道发布消息
		result := client.Publish(channel, data+randString(10)).Val()
		if result == 1 {
			fmt.Println("send info to", channel,"success")
		}
	}
}
```

#### 参考资料

1、https://redis.uptrace.dev/guide/server.html#connecting-to-redis-server
2、https://blog.csdn.net/weixin_37717557/article/details/102874062