### gorm的基本用法

gorm是一个适用于go语言数据库交互的orm框架，使用gorm可以在很大程度上简化同数据库的交互工作。


#### 主要的特性

- 全功能 ORM
- 关联 (Has One，Has Many，Belongs To，Many To Many，多态，单表继承)
- Create，Save，Update，Delete，Find 中钩子方法
- 支持 Preload、Joins 的预加载
- 事务，嵌套事务，Save Point，Rollback To Saved Point
- Context、预编译模式、DryRun 模式
- 批量插入，FindInBatches，Find/Create with Map，使用 SQL 表达式、Context Valuer 进行 CRUD
- SQL 构建器，Upsert，数据库锁，Optimizer/Index/Comment Hint，命名参数，子查
- 复合主键，索引，约束
- Auto Migration
- 自定义 Logger
- 灵活的可扩展插件 API：Database Resolver（多数据库，读写分离）、Prometheus…


#### 安装

> go get -u gorm.io/gorm <br/>
> go get -u gorm.io/driver/mysql


```go
package main

import (
  "gorm.io/gorm"
  "gorm.io/driver/mysql"
)

type Product struct {
  gorm.Model
  Code  string
  Price uint
}
ORM 定义一个 gorm.Model 结构体，其包括
func main() {
  dsn := "user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
  if err != nil {
    panic("failed to connect database")
  }

  db.AutoMigrate(&Product{})

  db.Create(&Product{Code: "D42", Price: 100})

  var product Product
  db.First(&product, 1) 
  db.First(&product, "code = ?", "D42")

  db.Model(&product).Update("Price", 200)
  db.Model(&product).Updates(map[string]interface{}{"Price": 200, "Code": "F42"})
  db.Delete(&product, 1)
}
```

#### 模型定义

模型是标准的struct，由go的基本数据类型，实现了 Scanner 和 Valuer 接口的自定义类型及其指针或别名组成。

例如：

```go
type User struct {
  ID           uint
  Name         string
  Email        *string
  Age          uint8
  Birthday     *time.Time
  MemberNumber sql.NullString
  ActivatedAt  sql.NullTime
  CreatedAt    time.Time
  UpdatedAt    time.Time
}
```

gorm.Model 等价于：

```go
type Model struct {
  ID        uint           `gorm:"primaryKey"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt gorm.DeletedAt `gorm:"index"`
}
```

模型定义时的字段权限控制

```go
type User struct {
  Name string `gorm:"<-:create"` // allow read and create
  Name string `gorm:"<-:update"` // allow read and update
  Name string `gorm:"<-"`        // allow read and write (create and update)
  Name string `gorm:"<-:false"`  // allow read, disable write permission
  Name string `gorm:"->"`        // readonly (disable write permission unless it configured)
  Name string `gorm:"->;<-:create"` // allow read and create
  Name string `gorm:"->:false;<-:create"` // createonly (disabled read from db)
  Name string `gorm:"-"`            // ignore this field when write and read with struct
  Name string `gorm:"-:all"`        // ignore this field when write, read and migrate with struct
  Name string `gorm:"-:migration"`  // ignore this field when migrate with struct
}
```

创建/更新时间追踪(纳秒、毫秒、秒、Time)

```go
type User struct {
  CreatedAt time.Time // 在创建时，如果该字段值为零值，则使用当前时间填充
  UpdatedAt int       // 在创建时该字段值为零值或者在更新时，使用当前时间戳秒数填充
  Updated   int64 `gorm:"autoUpdateTime:nano"` // 使用时间戳填纳秒数充更新时间
  Updated   int64 `gorm:"autoUpdateTime:milli"` // 使用时间戳毫秒数填充更新时间
  Created   int64 `gorm:"autoCreateTime"`      // 使用时间戳秒数填充创建时间
}
```

结构体字段嵌入

```go
type Author struct {
    Name  string
    Email string
}

type Blog struct {
  ID      int
  Author  Author `gorm:"embedded"`
  Upvotes int32
}
// 等效于
type Blog struct {
  ID    int64
  Name  string
  Email string
  Upvotes  int32
}
```

字段属性控制

|  标签名   | 说明  |
|  ----  | ----  |
| column  | 指定 db 列名 |
| type  | 列数据类型，推荐使用兼容性好的通用类型，例如：所有数据库都支持 bool、int、uint、float、string、time、bytes 并且可以和其他标签一起使用，例如：not null、size, autoIncrement… 像 varbinary(8) 这样指定数据库数据类型也是支持的。在使用指定数据库数据类型时，它需要是完整的数据库数据类型，如：MEDIUMINT UNSIGNED not NULL AUTO_INCREMENT |
| size  | 指定列大小，例如：size:256 |
| primaryKey  | 指定列为主键
 |
| unique  | 指定列为唯一 |
| default  | 指定列的默认值 |
| precision  | 指定列的精度 |
| scale  | 指定列大小
 |
| not null  | 指定列为 NOT NULL |
| autoIncrement  | 指定列为自动增长 |
| autoIncrementIncrement  | 自动步长，控制连续记录之间的间隔 |
| embedded  | 嵌套字段 |
| embeddedPrefix  | 嵌入字段的列名前缀 |
| autoCreateTime  | 创建时追踪当前时间，对于 int 字段，它会追踪秒级时间戳，您可以使用 nano/milli 来追踪纳秒、毫秒时间戳，例如：autoCreateTime:nano |
| autoUpdateTime  | 创建/更新时追踪当前时间，对于 int 字段，它会追踪秒级时间戳，您可以使用 nano/milli 来追踪纳秒、毫秒时间戳，例如：autoUpdateTime:milli |
| index  | 根据参数创建索引，多个字段使用相同的名称则创建复合索引，查看 索引 获取详情 |
| uniqueIndex  | 与 index 相同，但创建的是唯一索引 |
| <-  | 设置字段写入的权限， <-:create 只创建、<-:update 只更新、<-:false 无写入权限、<- 创建和更新权限 |
| ->  | 设置字段读的权限，->:false 无读权限 |
| -  | ignore this field, - no read/write permission, -:migration no migrate permission, -:all no read/write/migrate permission |
| comment  | 迁移时为字段添加注释 |


#### 连接到数据库

MySQL

```
import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {
  // 参考 https://github.com/go-sql-driver/mysql#dsn-data-source-name 获取详情
  dsn := "user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
}
```

高级配置

```
db, err := gorm.Open(mysql.New(mysql.Config{
  DSN: "gorm:gorm@tcp(127.0.0.1:3306)/gorm?charset=utf8&parseTime=True&loc=Local", // DSN data source name
  DefaultStringSize: 256, // string 类型字段的默认长度
  DisableDatetimePrecision: true, // 禁用 datetime 精度，MySQL 5.6 之前的数据库不支持
  DontSupportRenameIndex: true, // 重命名索引时采用删除并新建的方式，MySQL 5.7 之前的数据库和 MariaDB 不支持重命名索引
  DontSupportRenameColumn: true, // 用 `change` 重命名列，MySQL 8 之前的数据库和 MariaDB 不支持重命名列
  SkipInitializeWithVersion: false, // 根据当前 MySQL 版本自动配置
}), &gorm.Config{})
```

连接池

```
sqlDB, err := db.DB()

// SetMaxIdleConns 设置空闲连接池中连接的最大数量
sqlDB.SetMaxIdleConns(10)

// SetMaxOpenConns 设置打开数据库连接的最大数量。
sqlDB.SetMaxOpenConns(100)

// SetConnMaxLifetime 设置了连接可复用的最大时间。
sqlDB.SetConnMaxLifetime(time.Hour)
```

#### crud实现SQL

使用gorm框架的表达方式来实现我们的sql语句是非常重要的，下面将针对具体的实现方式展开总结

##### 创建

###### 创建记录

```
user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}
result := db.Create(&user) // 通过数据的指针来创建

user.ID             // 返回插入数据的主键
result.Error        // 返回 error
result.RowsAffected // 返回插入记录的条数
```

###### 用指定的字段创建记录

```
db.Select("Name", "Age", "CreatedAt").Create(&user)
// INSERT INTO `users` (`name`,`age`,`created_at`) VALUES ("jinzhu", 18, "2020-07-04 11:05:21.775")
```

###### 忽略指定的字段插入剩余字段

```
db.Omit("Name", "Age", "CreatedAt").Create(&user)
// INSERT INTO `users` (`birthday`,`updated_at`) VALUES ("2020-01-01 00:00:00.000", "2020-07-04 11:05:21.775")
```

###### 批量插入

```
var users = []User{{Name: "jinzhu1"}, {Name: "jinzhu2"}, {Name: "jinzhu3"}}
db.Create(&users)

for _, user := range users {
  user.ID // 1,2,3
}

```


###### 批量插入并控制批次数量

```
var users = []User{{name: "jinzhu_1"}, ...., {Name: "jinzhu_10000"}}

// 数量为 100
db.CreateInBatches(users, 100)
```

###### 使用Map结构插入数据

GORM 支持根据 map[string]interface{} 和 []map[string]interface{}{} 创建记录。

```
db.Model(&User{}).Create(map[string]interface{}{
  "Name": "jinzhu", "Age": 18,
})

// batch insert from `[]map[string]interface{}{}`
db.Model(&User{}).Create([]map[string]interface{}{
  {"Name": "jinzhu_1", "Age": 18},
  {"Name": "jinzhu_2", "Age": 20},
})
```

> 根据 map 创建记录时，association 不会被调用，且主键也不会自动填充

###### 关联创建

创建关联数据时，如果关联值是非零值，这些关联会被 upsert，且它们的 Hook 方法也会被调用

```
type CreditCard struct {
  gorm.Model
  Number   string
  UserID   uint
}

type User struct {
  gorm.Model
  Name       string
  CreditCard CreditCard
}

db.Create(&User{
  Name: "jinzhu",
  CreditCard: CreditCard{Number: "411111111111"}
})
// INSERT INTO `users` ...
// INSERT INTO `credit_cards` ...
```

您也可以通过 Select、 Omit 跳过关联保存，例如

```
db.Omit("CreditCard").Create(&user)

// 跳过所有关联
db.Omit(clause.Associations).Create(&user)
```

###### 默认值

注意 对于声明了默认值的字段，像 0、''、false 等零值是不会保存到数据库。您需要使用指针类型或 Scanner/Valuer 来避免这个问题，例如：

```
type User struct {
  gorm.Model
  Name string
  Age  *int           `gorm:"default:18"`
  Active sql.NullBool `gorm:"default:true"`
}
```

###### 创建钩子(类似Java的aop)

GORM 允许用户定义的钩子有 BeforeSave, BeforeCreate, AfterSave, AfterCreate 创建记录时将调用这些钩子方法。

```
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
  u.UUID = uuid.New()

    if u.Role == "admin" {
        return errors.New("invalid role")
    }
    return
}
```

如果您想跳过 钩子 方法，您可以使用 SkipHooks 会话模式。

```
DB.Session(&gorm.Session{SkipHooks: true}).Create(&user)
DB.Session(&gorm.Session{SkipHooks: true}).Create(&users)
DB.Session(&gorm.Session{SkipHooks: true}).CreateInBatches(users, 100)
```

##### 更新

###### 保存所有字段

Save 会保存所有的字段，即使字段是零值

```
db.First(&user)

user.Name = "jinzhu 2"
user.Age = 100
db.Save(&user)
// UPDATE users SET name='jinzhu 2', age=100, birthday='2016-01-01', updated_at = '2013-11-17 21:34:10' WHERE id=111;
```


###### 更新单列

当使用 Update 更新单个列时，你需要指定条件，否则会返回 ErrMissingWhereClause 错误。

```
// 条件更新
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;

// User 的 ID 是 `111`
db.Model(&user).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// 根据条件和 model 的值进行更新
db.Model(&user).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;
```

###### 更新多列

Updates 方法支持 struct 和 map[string]interface{} 参数。当使用 struct 更新时，默认情况下，GORM 只会更新非零值的字段。

```
// 根据 `struct` 更新属性，只会更新非零值的字段
db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// 根据 `map` 更新属性
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
```

> 注意 当通过 struct 更新时，GORM 只会更新非零字段。 如果您想确保指定字段被更新，你应该使用 Select 更新选定字段，或使用 map 来完成更新操作

###### 更新指定列

如果您想要在更新时选定、忽略某些字段，您可以使用 Select、Omit。

```
// 使用 Map 进行 Select
// User's ID is `111`:
db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET name='hello' WHERE id=111;

db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;

// 使用 Struct 进行 Select（会 select 零值的字段）
db.Model(&user).Select("Name", "Age").Updates(User{Name: "new_name", Age: 0})
// UPDATE users SET name='new_name', age=0 WHERE id=111;

// Select 所有字段（查询包括零值字段的所有字段）
db.Model(&user).Select("*").Update(User{Name: "jinzhu", Role: "admin", Age: 0})

// Select 除 Role 外的所有字段（包括零值字段的所有字段）
db.Model(&user).Select("*").Omit("Role").Update(User{Name: "jinzhu", Role: "admin", Age: 0})
```

###### 批量更新

如果您尚未通过 Model 指定记录的主键，则 GORM 会执行批量更新

```
// 根据 struct 更新
db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})
// UPDATE users SET name='hello', age=18 WHERE role = 'admin';

// 根据 map 更新
db.Table("users").Where("id IN ?", []int{10, 11}).Updates(map[string]interface{}{"name": "hello", "age": 18})
// UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);
```

###### 阻止全局更新

如果在没有任何条件的情况下执行批量更新，默认情况下，GORM 不会执行该操作，并返回 ErrMissingWhereClause 错误，对此，你必须加一些条件，或者使用原生 SQL，或者启用 AllowGlobalUpdate 模式。

```
db.Model(&User{}).Update("name", "jinzhu").Error // gorm.ErrMissingWhereClause

db.Model(&User{}).Where("1 = 1").Update("name", "jinzhu")
// UPDATE users SET `name` = "jinzhu" WHERE 1=1

db.Exec("UPDATE users SET name = ?", "jinzhu")
// UPDATE users SET name = "jinzhu"

db.Session(&gorm.Session{AllowGlobalUpdate: true}).Model(&User{}).Update("name", "jinzhu")
// UPDATE users SET `name` = "jinzhu"
```

###### 更新的记录数

获取受更新影响的行数

```
// 通过 `RowsAffected` 得到更新的记录数
result := db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})
// UPDATE users SET name='hello', age=18 WHERE role = 'admin';

result.RowsAffected // 更新的记录数
result.Error        // 更新的错误
```

###### 子查询更新

使用子查询更新表

```
db.Model(&user).Update("company_name", db.Model(&Company{}).Select("name").Where("companies.id = users.company_id"))
// UPDATE "users" SET "company_name" = (SELECT name FROM companies WHERE companies.id = users.company_id);

db.Table("users as u").Where("name = ?", "jinzhu").Update("company_name", db.Table("companies as c").Select("name").Where("c.id = u.company_id"))

db.Table("users as u").Where("name = ?", "jinzhu").Updates(map[string]interface{}{}{"company_name": db.Table("companies as c").Select("name").Where("c.id = u.company_id")})
```

###### 使用SQL表达式更新

GORM 允许使用 SQL 表达式更新列，例如：

```
// product 的 ID 是 `3`
db.Model(&product).Update("price", gorm.Expr("price * ? + ?", 2, 100))
// UPDATE "products" SET "price" = price * 2 + 100, "updated_at" = '2013-11-17 21:34:10' WHERE "id" = 3;

db.Model(&product).Updates(map[string]interface{}{"price": gorm.Expr("price * ? + ?", 2, 100)})
// UPDATE "products" SET "price" = price * 2 + 100, "updated_at" = '2013-11-17 21:34:10' WHERE "id" = 3;

db.Model(&product).UpdateColumn("quantity", gorm.Expr("quantity - ?", 1))
// UPDATE "products" SET "quantity" = quantity - 1 WHERE "id" = 3;

db.Model(&product).Where("quantity > 1").UpdateColumn("quantity", gorm.Expr("quantity - ?", 1))
// UPDATE "products" SET "quantity" = quantity - 1 WHERE "id" = 3 AND quantity > 1;
```

###### 不使用Hook和时间跟踪

```
// 更新单个列
db.Model(&user).UpdateColumn("name", "hello")
// UPDATE users SET name='hello' WHERE id = 111;

// 更新多个列
db.Model(&user).UpdateColumns(User{Name: "hello", Age: 18})
// UPDATE users SET name='hello', age=18 WHERE id = 111;

// 更新选中的列
db.Model(&user).Select("name", "age").UpdateColumns(User{Name: "hello", Age: 0})
// UPDATE users SET name='hello', age=0 WHERE id = 111;
```

###### 返回被修改行的数据

```
// 返回所有列
var users []User
DB.Model(&users).Clauses(clause.Returning{}).Where("role = ?", "admin").Update("salary", gorm.Expr("salary * ?", 2))
// UPDATE `users` SET `salary`=salary * 2,`updated_at`="2021-10-28 17:37:23.19" WHERE role = "admin" RETURNING *
// users => []User{{ID: 1, Name: "jinzhu", Role: "admin", Salary: 100}, {ID: 2, Name: "jinzhu.2", Role: "admin", Salary: 1000}}

// 返回指定的列
DB.Model(&users).Clauses(clause.Returning{Columns: []clause.Column{{Name: "name"}, {Name: "salary"}}}).Where("role = ?", "admin").Update("salary", gorm.Expr("salary * ?", 2))
// UPDATE `users` SET `salary`=salary * 2,`updated_at`="2021-10-28 17:37:23.19" WHERE role = "admin" RETURNING `name`, `salary`
// users => []User{{ID: 0, Name: "jinzhu", Role: "", Salary: 100}, {ID: 0, Name: "jinzhu.2", Role: "", Salary: 1000}}
```

###### 更新Hook

对于更新操作，GORM 支持 BeforeSave、BeforeUpdate、AfterSave、AfterUpdate 钩子，这些方法将在更新记录时被调用。

```
func (u *User) BeforeUpdate(tx *gorm.DB) (err error) {
    if u.Role == "admin" {
        return errors.New("admin user not allowed to update")
    }
    return
}
```

##### 删除

删除一条记录时，删除对象需要指定主键，否则会触发 批量 Delete

###### 根据主键删除

GORM 允许通过主键(可以是复合主键)和内联条件来删除对象，它可以使用数字

```
db.Delete(&User{}, 10)
// DELETE FROM users WHERE id = 10;

db.Delete(&User{}, "10")
// DELETE FROM users WHERE id = 10;

db.Delete(&users, []int{1,2,3})
// DELETE FROM users WHERE id IN (1,2,3);
```

###### 批量删除

如果指定的值不包括主属性，那么 GORM 会执行批量删除，它将删除所有匹配的记录

```
db.Where("email LIKE ?", "%jinzhu%").Delete(&Email{})
// DELETE from emails where email LIKE "%jinzhu%";

db.Delete(&Email{}, "email LIKE ?", "%jinzhu%")
// DELETE from emails where email LIKE "%jinzhu%";
```

###### 阻止全局删除

如果在没有任何条件的情况下执行批量删除，GORM 不会执行该操作，并返回 ErrMissingWhereClause 错误。对此，你必须加一些条件，或者使用原生 SQL，或者启用 AllowGlobalUpdate 模式

```
db.Delete(&User{}).Error // gorm.ErrMissingWhereClause

db.Where("1 = 1").Delete(&User{})
// DELETE FROM `users` WHERE 1=1

db.Exec("DELETE FROM users")
// DELETE FROM users

db.Session(&gorm.Session{AllowGlobalUpdate: true}).Delete(&User{})
// DELETE FROM users
```
###### 软删除

如果您的模型包含了一个 gorm.deletedat 字段（gorm.Model 已经包含了该字段)，它将自动获得软删除的能力！拥有软删除能力的模型调用 Delete 时，记录不会被数据库。但 GORM 会将 DeletedAt 置为当前时间， 并且你不能再通过普通的查询方法找到该记录。

```
// user 的 ID 是 `111`
db.Delete(&user)
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;

// 批量删除
db.Where("age = ?", 20).Delete(&User{})
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE age = 20;

// 在查询时会忽略被软删除的记录
db.Where("age = 20").Find(&user)
// SELECT * FROM users WHERE age = 20 AND deleted_at IS NULL;
```

> 如果您不想引入 gorm.Model，您也可以这样启用软删除特性：

```
type User struct {
  ID      int
  Deleted gorm.DeletedAt
  Name    string
}
```

###### 永久删除

您也可以使用 Unscoped 永久删除匹配的记录

```
db.Unscoped().Delete(&order)
// DELETE FROM orders WHERE id=10;
```

###### 查询软删除数据

您可以使用 Unscoped 找到被软删除的记录

```
db.Unscoped().Where("age = 20").Find(&users)
// SELECT * FROM users WHERE age = 20;
```

###### 返回删除行数据

返回被删除的数据，仅适用于支持 Returning 的数据库，例如：

```
// 返回所有列
var users []User
DB.Clauses(clause.Returning{}).Where("role = ?", "admin").Delete(&users)
// DELETE FROM `users` WHERE role = "admin" RETURNING *
// users => []User{{ID: 1, Name: "jinzhu", Role: "admin", Salary: 100}, {ID: 2, Name: "jinzhu.2", Role: "admin", Salary: 1000}}

// 返回指定的列
DB.Clauses(clause.Returning{Columns: []clause.Column{{Name: "name"}, {Name: "salary"}}}).Where("role = ?", "admin").Delete(&users)
// DELETE FROM `users` WHERE role = "admin" RETURNING `name`, `salary`
// users => []User{{ID: 0, Name: "jinzhu", Role: "", Salary: 100}, {ID: 0, Name: "jinzhu.2", Role: "", Salary: 1000}}
```

###### 删除Hook

对于删除操作，GORM 支持 BeforeDelete、AfterDelete Hook，在删除记录时会调用这些方法


```
func (u *User) BeforeDelete(tx *gorm.DB) (err error) {
    if u.Role == "admin" {
        return errors.New("admin user not allowed to delete")
    }
    return
}
```

##### 查询

###### 查询单个对象

GORM 提供了 First、Take、Last 方法，以便从数据库中检索单个对象。当查询数据库时它添加了 LIMIT 1 条件，且没有找到记录时，它会返回 ErrRecordNotFound 错误。

```
// 获取第一条记录（主键升序）
db.First(&user)
// SELECT * FROM users ORDER BY id LIMIT 1;

// 获取一条记录，没有指定排序字段
db.Take(&user)
// SELECT * FROM users LIMIT 1;

// 获取最后一条记录（主键降序）
db.Last(&user)
// SELECT * FROM users ORDER BY id DESC LIMIT 1;

result := db.First(&user)
result.RowsAffected // 返回找到的记录数
result.Error        // returns error or nil

// 检查 ErrRecordNotFound 错误
errors.Is(result.Error, gorm.ErrRecordNotFound)
```

First 和 Last 会根据主键排序，分别查询第一条和最后一条记录。 只有在目标 struct 是指针或者通过 db.Model() 指定 model 时，该方法才有效。 此外，如果相关 model 没有定义主键，那么将按 model 的第一个字段进行排序。 例如:

```
var user User
var users []User  

// 有效，因为目标 struct 是指针
db.First(&user)
// SELECT * FROM `users` ORDER BY `users`.`id` LIMIT 1

// 有效，因为通过 `db.Model()` 指定了 model
result := map[string]interface{}{}
db.Model(&User{}).First(&result)
// SELECT * FROM `users` ORDER BY `users`.`id` LIMIT 1

// 无效
result := map[string]interface{}{}
db.Table("users").First(&result)

// 配合 Take 有效
result := map[string]interface{}{}
db.Table("users").Take(&result)

// 未指定主键，会根据第一个字段排序(即：`Code`)
type Language struct {
  Code string
  Name string
}
db.First(&Language{})
// SELECT * FROM `languages` ORDER BY `languages`.`code` LIMIT 1
```

###### 主键查询

如果主键是数字类型，您可以使用 内联条件 来检索对象。 传入字符串参数时，需要特别注意 SQL 注入问题，查看 安全 获取详情。

```
db.First(&user, 10)
// SELECT * FROM users WHERE id = 10;

db.First(&user, "10")
// SELECT * FROM users WHERE id = 10;

db.Find(&users, []int{1,2,3})
// SELECT * FROM users WHERE id IN (1,2,3);
```

如果主键是字符串（例如像 uuid），查询将被写成这样：

```
db.First(&user, "id = ?", "1b74413f-f3b8-409f-ac47-e8c062e3472a")
// SELECT * FROM users WHERE id = "1b74413f-f3b8-409f-ac47-e8c062e3472a";
```

###### 查询全部

```
// 获取全部记录
result := db.Find(&users)
// SELECT * FROM users;

result.RowsAffected // 返回找到的记录数，相当于 `len(users)`
result.Error        // returns error
```

###### String条件查询

```
// 获取第一条匹配的记录
db.Where("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE name = 'jinzhu' ORDER BY id LIMIT 1;

// 获取全部匹配的记录
db.Where("name <> ?", "jinzhu").Find(&users)
// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN ?", []string{"jinzhu", "jinzhu 2"}).Find(&users)
// SELECT * FROM users WHERE name IN ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
// SELECT * FROM users WHERE name LIKE '%jin%';

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;

// Time
db.Where("updated_at > ?", lastWeek).Find(&users)
// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';

// BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';
```


###### Not条件查询

构建 NOT 条件，用法与 Where 类似

```
db.Not("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE NOT name = "jinzhu" ORDER BY id LIMIT 1;

// Not In
db.Not(map[string]interface{}{"name": []string{"jinzhu", "jinzhu 2"}}).Find(&users)
// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Struct
db.Not(User{Name: "jinzhu", Age: 18}).First(&user)
// SELECT * FROM users WHERE name <> "jinzhu" AND age <> 18 ORDER BY id LIMIT 1;

// 不在主键切片中的记录
db.Not([]int64{1,2,3}).First(&user)
// SELECT * FROM users WHERE id NOT IN (1,2,3) ORDER BY id LIMIT 1;
```

###### Or条件查询

```
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';

// Struct
db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2", Age: 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);

// Map
db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2", "age": 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);
```

###### Order条件查询

指定从数据库检索记录时的排序方式

```
db.Order("age desc, name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

// 多个 order
db.Order("age desc").Order("name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

db.Clauses(clause.OrderBy{
  Expression: clause.Expr{SQL: "FIELD(id,?)", Vars: []interface{}{[]int{1, 2, 3}}, WithoutParentheses: true},
}).Find(&User{})
// SELECT * FROM users ORDER BY FIELD(id,1,2,3)
```

###### Limit & Offset查询

Limit 指定获取记录的最大数量 Offset 指定在开始返回记录之前要跳过的记录数量。

```
db.Limit(3).Find(&users)
// SELECT * FROM users LIMIT 3;

// 通过 -1 消除 Limit 条件
db.Limit(10).Find(&users1).Limit(-1).Find(&users2)
// SELECT * FROM users LIMIT 10; (users1)
// SELECT * FROM users; (users2)

db.Offset(3).Find(&users)
// SELECT * FROM users OFFSET 3;

db.Limit(10).Offset(5).Find(&users)
// SELECT * FROM users OFFSET 5 LIMIT 10;

// 通过 -1 消除 Offset 条件
db.Offset(10).Find(&users1).Offset(-1).Find(&users2)
// SELECT * FROM users OFFSET 10; (users1)
// SELECT * FROM users; (users2)
```

###### Group By & Having 条件查询

```
type result struct {
  Date  time.Time
  Total int
}

db.Model(&User{}).Select("name, sum(age) as total").Where("name LIKE ?", "group%").Group("name").First(&result)
// SELECT name, sum(age) as total FROM `users` WHERE name LIKE "group%" GROUP BY `name` LIMIT 1


db.Model(&User{}).Select("name, sum(age) as total").Group("name").Having("name = ?", "group").Find(&result)
// SELECT name, sum(age) as total FROM `users` GROUP BY `name` HAVING name = "group"

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Rows()
defer rows.Close()
for rows.Next() {
  ...
}

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Rows()
defer rows.Close()
for rows.Next() {
  ...
}

type Result struct {
  Date  time.Time
  Total int64
}
db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Scan(&results)
```

###### Distinct条件查询

从模型中选择不相同的值

```
db.Distinct("name", "age").Order("name, age desc").Find(&results)
```

###### Joins条件

指定 Joins 条件

```
type result struct {
  Name  string
  Email string
}

db.Model(&User{}).Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&result{})
// SELECT users.name, emails.email FROM `users` left join emails on emails.user_id = users.id

rows, err := db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Rows()
for rows.Next() {
  ...
}

db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&results)

// 带参数的多表连接
db.Joins("JOIN emails ON emails.user_id = users.id AND emails.email = ?", "jinzhu@example.org").Joins("JOIN credit_cards ON credit_cards.user_id = users.id").Where("credit_cards.number = ?", "411111111111").Find(&user)
```


###### Joins预加载

您可以使用 Joins 实现单条 SQL 预加载关联记录，例如：

```
db.Joins("Company").Find(&users)
// SELECT `users`.`id`,`users`.`name`,`users`.`age`,`Company`.`id` AS `Company__id`,`Company`.`name` AS `Company__name` FROM `users` LEFT
```

带条件的 Join

```
db.Joins("Company", DB.Where(&Company{Alive: true})).Find(&users)
// SELECT `users`.`id`,`users`.`name`,`users`.`age`,`Company`.`id` AS `Company__id`,`Company`.`name` AS `Company__name` FROM `users` LEFT JOIN `companies` AS `Company` ON `users`.`company_id` = `Company`.`id` AND `Company`.`alive` = true;
```


###### Struct & Map条件查询

```
// Struct
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 ORDER BY id LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// 主键切片条件
db.Where([]int64{20, 21, 22}).Find(&users)
// SELECT * FROM users WHERE id IN (20, 21, 22);
```

> 注意 当使用结构作为条件查询时，GORM 只会查询非零值字段。这意味着如果您的字段值为 0、''、false 或其他 零值，该字段不会被用于构建查询条件，例如：

```
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu";
```

如果想要包含零值查询条件，你可以使用 map，其会包含所有 key-value 的查询条件，例如

```
db.Where(map[string]interface{}{"Name": "jinzhu", "Age": 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;
```

###### 指定结构体查询字段

当使用 struct 进行查询时，你可以通过向 Where() 传入 struct 来指定查询条件的字段、值、表名，例如：

```
db.Where(&User{Name: "jinzhu"}, "name", "Age").Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;

db.Where(&User{Name: "jinzhu"}, "Age").Find(&users)
// SELECT * FROM users WHERE age = 0;
```

###### 内联条件

查询条件也可以被内联到 First 和 Find 之类的方法中，其用法类似于 Where。

```
// 根据主键获取记录，如果是非整型主键
db.First(&user, "id = ?", "string_primary_key")
// SELECT * FROM users WHERE id = 'string_primary_key';

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
// SELECT * FROM users WHERE name = "jinzhu";

db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
// SELECT * FROM users WHERE age = 20;

// Map
db.Find(&users, map[string]interface{}{"age": 20})
// SELECT * FROM users WHERE age = 20;
```

###### 指定指定查询

Select 允许您指定从数据库中检索哪些字段， 默认情况下，GORM 会检索所有字段

```
db.Select("name", "age").Find(&users)
// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
// SELECT COALESCE(age,'42') FROM users;
```

###### Scan接口


```
type Result struct {
  Name string
  Age  int
}

var result Result
db.Table("users").Select("name", "age").Where("name = ?", "Antonio").Scan(&result)

// Raw SQL
db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)
```


##### 高级查询

###### 智能选择字段

GORM 允许通过 Select 方法选择特定的字段，如果您在应用程序中经常使用此功能，你也可以定义一个较小的结构体，以实现调用 API 时自动选择特定的字段，例如：

```
type User struct {
  ID     uint
  Name   string
  Age    int
  Gender string
  // 假设后面还有几百个字段...
}

type APIUser struct {
  ID   uint
  Name string
}

// 查询时会自动选择 `id`, `name` 字段
db.Model(&User{}).Limit(10).Find(&APIUser{})
// SELECT `id`, `name` FROM `users` LIMIT 10
```

###### 子查询

子查询可以嵌套在查询中，GORM 允许在使用 *gorm.DB 对象作为参数时生成子查询。

```
db.Where("amount > (?)", db.Table("orders").Select("AVG(amount)")).Find(&orders)
// SELECT * FROM "orders" WHERE amount > (SELECT AVG(amount) FROM "orders");

subQuery := db.Select("AVG(age)").Where("name LIKE ?", "name%").Table("users")
db.Select("AVG(age) as avgage").Group("name").Having("AVG(age) > (?)", subQuery).Find(&results)
// SELECT AVG(age) as avgage FROM `users` GROUP BY `name` HAVING AVG(age) > (SELECT AVG(age) FROM `users` WHERE name LIKE "name%")
```

###### From子查询

GORM 允许您在 Table 方法中通过 FROM 子句使用子查询，例如：

```
db.Table("(?) as u", db.Model(&User{}).Select("name", "age")).Where("age = ?", 18}).Find(&User{})
// SELECT * FROM (SELECT `name`,`age` FROM `users`) as u WHERE `age` = 18

subQuery1 := db.Model(&User{}).Select("name")
subQuery2 := db.Model(&Pet{}).Select("name")
db.Table("(?) as u, (?) as p", subQuery1, subQuery2).Find(&User{})
// SELECT * FROM (SELECT `name` FROM `users`) as u, (SELECT `name` FROM `pets`) as p
```

###### Group条件

使用 Group 条件可以更轻松的编写复杂 SQL

```
db.Where(
    db.Where("pizza = ?", "pepperoni").Where(db.Where("size = ?", "small").Or("size = ?", "medium")),
).Or(
    db.Where("pizza = ?", "hawaiian").Where("size = ?", "xlarge"),
).Find(&Pizza{}).Statement

// SELECT * FROM `pizzas` WHERE (pizza = "pepperoni" AND (size = "small" OR size = "medium")) OR (pizza = "hawaiian" AND size = "xlarge")
```

###### 带多个条件的In

```
db.Where("(name, age, role) IN ?", [][]interface{}{{"jinzhu", 18, "admin"}, {"jinzhu2", 19, "user"}}).Find(&users)
// SELECT * FROM users WHERE (name, age, role) IN (("jinzhu", 18, "admin"), ("jinzhu 2", 19, "user"));
```

###### Find至Map

GORM 允许扫描结果至 map[string]interface{} 或 []map[string]interface{}，此时别忘了指定 Model 或 Table，例如：

```
result := map[string]interface{}{}
db.Model(&User{}).First(&result, "id = ?", 1)

var results []map[string]interface{}
db.Table("users").Find(&results)
```

###### FirstOrInit

获取第一条匹配的记录，或者根据给定的条件初始化一个实例（仅支持 sturct 和 map 条件）

```
// 未找到 user，则根据给定的条件初始化一条记录
db.FirstOrInit(&user, User{Name: "non_existing"})
// user -> User{Name: "non_existing"}

// 找到了 `name` = `jinzhu` 的 user
db.Where(User{Name: "jinzhu"}).FirstOrInit(&user)
// user -> User{ID: 111, Name: "Jinzhu", Age: 18}

// 找到了 `name` = `jinzhu` 的 user
db.FirstOrInit(&user, map[string]interface{}{"name": "jinzhu"})
// user -> User{ID: 111, Name: "Jinzhu", Age: 18}
```

###### FirstOrCreate

获取第一条匹配的记录，或者根据给定的条件创建一条新纪录（仅支持 sturct 和 map 条件）

```
// 未找到 user，则根据给定条件创建一条新纪录
db.FirstOrCreate(&user, User{Name: "non_existing"})
// INSERT INTO "users" (name) VALUES ("non_existing");
// user -> User{ID: 112, Name: "non_existing"}

// 找到了 `name` = `jinzhu` 的 user
db.Where(User{Name: "jinzhu"}).FirstOrCreate(&user)
// user -> User{ID: 111, Name: "jinzhu", "Age": 18}
```

###### Pluck

Pluck 用于从数据库查询单个列，并将结果扫描到切片。如果您想要查询多列，您应该使用 Select 和 Scan。

```
var ages []int64
db.Model(&users).Pluck("age", &ages)

var names []string
db.Model(&User{}).Pluck("name", &names)

db.Table("deleted_users").Pluck("name", &names)

// Distinct Pluck
db.Model(&User{}).Distinct().Pluck("Name", &names)
// SELECT DISTINCT `name` FROM `users`

// 超过一列的查询，应该使用 `Scan` 或者 `Find`，例如：
db.Select("name", "age").Scan(&users)
db.Select("name", "age").Find(&users)
```

###### Count

Count 用于获取匹配的记录数

```
var count int64
db.Model(&User{}).Where("name = ?", "jinzhu").Or("name = ?", "jinzhu 2").Count(&count)
// SELECT count(1) FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2'

db.Model(&User{}).Where("name = ?", "jinzhu").Count(&count)
// SELECT count(1) FROM users WHERE name = 'jinzhu'; (count)

db.Table("deleted_users").Count(&count)
// SELECT count(1) FROM deleted_users;

// Count with Distinct
db.Model(&User{}).Distinct("name").Count(&count)
// SELECT COUNT(DISTINCT(`name`)) FROM `users`

db.Table("deleted_users").Select("count(distinct(name))").Count(&count)
// SELECT count(distinct(name)) FROM deleted_users

// Count with Group
users := []User{
  {Name: "name1"},
  {Name: "name2"},
  {Name: "name3"},
  {Name: "name3"},
}

db.Model(&User{}).Group("name").Count(&count)
count // => 3
```


###### 命名参数

GORM 支持 sql.NamedArg 和 map[string]interface{}{} 形式的命名参数，例如：

```
db.Where("name1 = @name OR name2 = @name", sql.Named("name", "jinzhu")).Find(&user)
// SELECT * FROM `users` WHERE name1 = "jinzhu" OR name2 = "jinzhu"

db.Where("name1 = @name OR name2 = @name", map[string]interface{}{"name": "jinzhu"}).First(&user)
// SELECT * FROM `users` WHERE name1 = "jinzhu" OR name2 = "jinzhu" ORDER BY `users`.`id` LIMIT 1
```

#### 参考资料

1、https://gorm.io/zh_CN/docs/index.html