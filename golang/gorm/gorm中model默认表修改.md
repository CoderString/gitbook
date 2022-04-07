### gorm表名称修改

#### 复数表名

GORM 使用结构体名的 蛇形命名 作为表名。对于结构体 User，根据约定，其表名为 users.

如果想要修改默认的表的名称，可以在数据库对应的结构体中实现TableName方法

```
// TableName 会将 User 的表名重写为 `profiles`
func (User) TableName() string {
  return "profiles"
}
```

#### 动态表名

注意： TableName 不支持动态变化，它会被缓存下来以便后续使用。想要使用动态表名，你可以使用 Scopes，例如：

```
func UserTable(user User) func (tx *gorm.DB) *gorm.DB {
  return func (tx *gorm.DB) *gorm.DB {
    if user.Admin {
      return tx.Table("admin_users")
    }

    return tx.Table("users")
  }
}

db.Scopes(UserTable(user)).Create(&user)
```

#### 参考资料

1、https://gorm.io/zh_CN/docs/conventions.html