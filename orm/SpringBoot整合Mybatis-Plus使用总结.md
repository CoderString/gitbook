### Spring Boot 整合Mybatis-Plus

鉴于之前的项目中单表操作采用xml的交互方式过于繁琐，因此后续的项目dao交互层全部采用Mybatis Plus，整体上而言，相对于单表操作确实节省了很多时间，与其他的orm框架类比，Mybatis Plus可以直接生成dao、service、controller层的代码，根据实际的需要我们可能只需要dao数据库交互层的，当然也可以使用自动生成的其他模块的代码。下面将针对Mybatis Plus展开讲解。

---

### MyBatis-Plus 是什么

MyBatis-Plus是一个Mybatis的增强工具，在MyBatis的基础上只做增强，不做改变，为简化开发，提高效率而诞生的，官网的网址是(https://baomidou.com/)

---


### 特性

- 无侵入：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑

- 损耗小：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作

- 强大的 CRUD 操作：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求

- 支持 Lambda 形式调用：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错

- 支持主键自动生成：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题

- 支持 ActiveRecord 模式：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作

- 支持自定义全局通用操作：支持全局通用方法注入（ Write once, use anywhere ）

- 内置代码生成器：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用

- 内置分页插件：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询

- 分页插件支持多种数据库：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库

- 内置性能分析插件：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询

- 内置全局拦截插件：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

---

### 整合

- 引入关键依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.1</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.10</version>
</dependency>

```

- 代码配置

项目的主启动类添加@MapperScan("com.coderstring.rbac.dao")  ， 这里扫描的是对应的mapper接口的存放的包路径

数据库交互配置如下

```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/rbac?useUnicode=true&characterEncoding=utf-8&serverTimeZone=GMT
    username: root
    password: xxx

mybatis-plus:
  mapper-locations: classpath*:dao/*Mapper.xml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

- mapper接口配置

mapper接口类直接继承BaseMapper<T>，就可以拥有基类的操作数据库的方法，具体实现方式例如下面的案例

```
public interface SysUserMapper extends BaseMapper<SysUser> {

}
```

- 实体类配置

```
@TableName("sys_user")
@Getter
@Setter
@ToString
@Accessors(chain = true)
public class SysUser implements Serializable{

    private static final long serialVersionUID = 1L;

    /**
     * 主键
     */
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    /**
     * 用户名
     */
    @TableField("user_name")
    private String username;

    /**
     * 密码
     */
    @TableField("password")
    private String password;

    /**
     * 账号添加操作人
     */
    @TableField("operator")
    private Long operator;

    /**
     * 创建时间
     */
    @TableField(value = "created_at", fill = FieldFill.INSERT)
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone="GMT+8")
    private LocalDateTime createdAt;

    /**
     * 修改时间
     */
    @TableField(value = "updated_at", fill = FieldFill.INSERT_UPDATE)
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone="GMT+8")
    private LocalDateTime updatedAt;

    @TableField(value = "is_deleted")
    private Boolean isDeleted;
}
```

### 基础crud API汇总

- 查询

根据id查询
```
 UserBean userBean = simpleMapper.selectById(1);
```

根据id批量查询
```
List<Integer> ids = Arrays.asList(1, 2, 3);
List<UserBean> userBeanList = simpleMapper.selectBatchIds(ids);
```

根据构建的Wrapper条件查询
```
QueryWrapper<UserBean> wrapper = new QueryWrapper<>();
wrapper.le("user_id", 1);
UserBean userBean = simpleMapper.selectOne(wrapper);
```

获取数据的总数(也可以根据Wrapper过滤)
```
QueryWrapper<UserBean> wrapper = new QueryWrapper<>();
wrapper.gt("age", 28);
int count = simpleMapper.selectCount(wrapper);
```

map作为参数查询
```
Map<String,Object> map = new HashMap<>();
map.put("user_id", userId);
if(StringUtils.isNotEmpty(name)) {
    map.put("name", name);
}
if(null != age) {
    map.put("age", age);
}

List<UserBean> userBeanList = simpleMapper.selectByMap(map);
```

动态查询
```
QueryWrapper<UserBean> wrapper = new QueryWrapper<>();
wrapper.eq("user_id", userId);

// 第一个参数为是否执行条件，为true则执行该条件
// 下面实现根据 name 和 age 动态查询
wrapper.eq(StringUtils.isNotEmpty(name), "name", name);
wrapper.eq(null != age, "age", age);

List<UserBean> userBeanList = simpleMapper.selectList(wrapper);
```

分页查询
```
// 根据版本的不同配置的方式有所区别
@Configuration
public class MybatisPlusConfig {
 
    /**
     * 分页插件。如果你不配置，分页插件将不生效
     */
    @Bean
    public MybatisPlusInterceptor paginationInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 指定数据库方言为 MYSQL
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}


QueryWrapper<UserBean> wrapper = new QueryWrapper<>();
wrapper.isNotNull("user_id");

// 创建分页对象（1表示第一页；4表示每页大小为4）
Page<UserBean> page = new Page<>(1, 4);
Page<UserBean> result = simpleMapper.selectPage(page, wrapper);
System.out.println("page == result: " + (page == result));
System.out.println("size: " + result.getSize());
System.out.println("total: " + result.getTotal());
for(UserBean userBean : result.getRecords()) {
    System.out.println(userBean);
}
```

- 添加

```
SysUser userBean = new SysUser();
userBean.setUserId(999);
userBean.setName("insert test");
userBean.setAge(30);
userBean.setSex("男");
userBean.setFace("Hello World".getBytes());
userBean.setBorthday(new Date());

int result = simpleMapper.insert(userBean);
```

- 更新


根据id更新数据
```
SysUser userBean = new SysUser();
userBean.setUserId(999);
userBean.setName("insert test update");
userBean.setAge(40);
userBean.setSex("女");
userBean.setFace("welcome".getBytes());
userBean.setBorthday(new Date());

int result = simpleMapper.updateById(userBean);
```

根据条件批量更新数据
```
UpdateWrapper<SysUser> wrapper = new UpdateWrapper<>();
wrapper.ge("age", 30); // 大于等于 30

SysUser userBean = new SysUser();
userBean.setAge(80);
int result = simpleMapper.update(userBean, wrapper);
```

- 删除

根据id进行删除
```
int result = sysUserMapper.deleteById(999);
```

根据id批量删除
```
List<Integer> ids = Arrays.asList(2, 4, 6, 7);
int result = sysUserMapper.deleteBatchIds(ids);
```

根据Wrapper条件删除
```
QueryWrapper<SysUser> wrapper = new QueryWrapper<>();
wrapper.le("age", 30); // 小于等于30
int result = sysUserMapper.delete(wrapper);
```

根据Map条件删除
```
Map<String,Object> map = new HashMap<>();
map.put("sex", "女");
int result = sysUserMapper.deleteByMap(map);
```

### 参考资料

1、https://www.hxstrive.com/subject/mybatis_plus.htm?id=269