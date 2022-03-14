### Spring Boot集成通用Mybatis 通用mapper教程

Mybatis虽然好用，但是随着开发的推进，本身的弊端也在不断的显现，吐槽最多的点无非就是即使是单表操作非常简单的功能，也需要通过xml的方式来完成，为了摆脱这种困境，我们通过引入通用mapper的方式来避免xml的书写方式，提高开发的效率。

### maven依赖

```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 数据库依赖 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.4.1</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <!-- 通用mapper依赖-->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper</artifactId>
            <version>4.1.5</version>
        </dependency>

        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>2.1.5</version>
        </dependency>

        <!-- 通用mapper逆向依赖 -->
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.6</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>

    </dependencies>
```


### yml配置

```
server:
  port: 8070
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/rbac?useUnicode=true&characterEncoding=utf-8&serverTimeZone=GMT
    username: root
    password: root

pagehelper:
  helperDialect: mysql
  reasonable: true
  support-methods-arguments: true
  params: count:countSql

mybatis:
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml
  type-aliases-package: com.coderstring.starter.model


logging:
  level:
    com.coderstring.starter.dao: debug
```

### mybatis-config.xml配置

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <setting name="logImpl" value="SLF4J"/>
    </settings>

    <typeAliases>
        <package name="com.coderstring.starter.model"/>
    </typeAliases>

    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor"/>
    </plugins>
</configuration>
```

### 启动类配置

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tk.mybatis.spring.annotation.MapperScan;

@SpringBootApplication
@MapperScan(basePackages = "com.coderstring.starter.dao")
public class StarterApplication {

    public static void main(String[] args) {
        SpringApplication.run(StarterApplication.class, args);
    }
}
```

### dao配置

```
public interface SysUserMapper extends Mapper<SysUser> {

}
```

### 基本API用法

#### select

- 根据实体中的属性值进行查询

```
List<T> select(T var1);
```

- 根据主键字段进行查询，参数必须包含完整的主键属性

```
T selectByPrimaryKey(Object var1);
```

- 查询全部的结果

```
List<T> selectAll();
```

- 根据实体类中的属性进行查询，只能返回一个属性值，有多个结果会抛出异常

```
T selectOne(T var1);
```

- 根据实体中的属性查询总数

```
int selectCount(T var1);
```

#### insert

- 保存一个实体，null的属性也会保存，不会使用数据库默认值

```
int insert(T var1);
```

- 保存一个实体，null的属性值不会保存，会使用数据库的默认值

```
int insertSelective(T var1);
```

#### update

- 根据主键更新全部实体字段，null值也会被更新

```
int updateByPrimaryKey(T var1);
```

- 根据主键更新属性不为null的值

```
int updateByPrimaryKeySelective(T var1);
```


#### delete

- 根据实体属性作为条件进行删除

```
int delete(T var1);
```
- 根据主键字段进行删除，方法参数必须包含完整的主键属性

```
int deleteByPrimaryKey(Object var1);
```

### Example的用法


Example example = new Example(Goods.class);
Example.Criteria criteria = example.createCriteria();

- 条件设置(字段名为实体类的属性名，非数据库字段名)



|  序号   | 方法  | 作用  |
|  ----  | ----  | ----  |
| 1  | example.orderBy(字段名).asc(); |添加升序排列条件 |
| 2  | example.orderBy(字段名).desc(); |添加降序排序条件 |
| 3  | example. setDistinct(false); |去除重复,boolean型,true为选择不重复的记录 |
| 4  | criteria. andIsNull(字段名); |添加字段xx为null的条件 |
| 5  | criteria. andIsNotNull(字段名); |添加字段xx不为null的条件 |
| 6  | criteria.andEqualTo(字段名,value); |添加xx字段等于value条件 |
| 7  | criteria.andNotEqualTo(字段名,value); |添加xx字段不等于value条件 |
| 8 | criteria.andGreaterThan(字段名,value); |添加xx字段大于value条件 |
| 9  | criteria.andGreaterThanOrEqualTo(字段名,value);	 |添加xx字段大于等于vaue条件 |
| 10  | 	criteria.andLessThan(字段名,value); |添加xx字段小于value条件 |
| 11  | criteria.andLessThanOrEqualTo(字段名,value); |添加xx字段小于等于value条件 |
| 12  | criteria.andIn(字段名,list); |添加xx字段值在List条件 |
| 13  | criteria. andNotIn(字段名,list); |添加xx字段值不在List条件 |
| 14  | criteria.andLike(字段名,“%”+value+”%”); |添加xx字段值为vaue的模糊查询条件 |
| 15  | criteria.andNotLike(字段名,“%”+value+”%”); |添加xx字段值不为 value的模糊查询条件 |
| 16  | criteria.andBetween(字段名,value1,value2); |添加xx字段值在value1和value2之间条件 |
| 17  | criteria.andNotBetween(字段名,value1,value2); |添加xx字段值不在value1和value2之间条件 |


#### Select

- 根据Example条件进行查询

```
List<T> selectByExample(Object example);
```

- 根据Example条件进行总数查询

```
int selectCountByExample(Object example);
```

- 查询指定字段方法

```
Example example = new Example(User.class);
     example.selectProperties("headImg","username","introduction")
       .and()
      .andEqualTo("userId",userId);
User user = userMapper.selectOneByExample(example);
```

#### Update

- 根据Example条件更新实体record包含的全部属性，null值会被更新

```
int updateByExample(@Param("record") T record, @Param("example") Object example);
```

- 根据Example条件更新实体record包含的不是null的属性值

```
int updateByExampleSelective(@Param("record") T record, @Param("example") Object example);
```

#### Delete

- 根据Example条件删除数据

```
int deleteByExample(Object example);
```


### WeekendSqls 用法

相比于Example，优势是不用输入属性名，可以避免数据库有变动或者输入错误出错的概率。

- 案例一

```

// 获取WeekendSqls对象
WeekendSqls<MybatisDemo> sqls = WeekendSqls.<MybatisDemo>custom();

// 进行动态sql拼接
sqls = sqls.andEqualTo(MybatisDemo::getCount,0).andLike(MybatisDemo::getName,"%d%");

// 获取结果
List<MybatisDemo> demos = mybatisDemoMapper.selectByExample(Example.builder(MybatisDemo.class).where(sqls).orderByDesc("count","name").build());

```

- 案例二

```
WeekendSqls<ArchiveDO> sqls = WeekendSqls.<ArchiveDO>custom();
if (StringUtils.isNotBlank(so.getImei())) {
    sqls.andEqualTo(ArchiveDO::getImei, so.getImei());
}
if (so.getIsDeleted() != null) {
    sqls.andEqualTo(ArchiveDO::getIsDeleted, so.getIsDeleted());
}
if (StringUtils.isNotBlank(so.getCreatePerson())) {
    sqls.andEqualTo(ArchiveDO::getCreatePerson, so.getCreatePerson());
}
if (StringUtils.isNotBlank(so.getNote())) {
    sqls.andLike(ArchiveDO::getNote, "%" + so.getNote() + "%");
}
Example example = Example.builder(ArchiveDO.class).where(sqls).orderByDesc("id").build();
int total = archiveMapper.selectCountByExample(example);
```



### 自动生成model、dao、mapper代码

- 生成程序

```
import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.internal.DefaultShellCallback;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

public class Generator {
    public void generator() throws Exception {
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        
        File configFile = new File("./src/main/resources/mybatis/generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }

    public static void main(String[] args) {
        try {
            Generator generatorSqlmap = new Generator();
            generatorSqlmap.generator();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```


- 生成配置文件

```
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<!--suppress MybatisGenerateCustomPluginInspection -->
<generatorConfiguration>
    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <property name="javaFileEncoding" value="UTF-8"/>
<!--        <property name="useMapperCommentGenerator" value="false"/>-->

        <!-- 通用Mapper插件，生成带注解的实体类 -->
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
            <property name="caseSensitive" value="true"/>
            <property name="forceAnnotation" value="true"/>
            <property name="beginningDelimiter" value="`"/>
            <property name="endingDelimiter" value="`"/>
            <property name="lombok" value="Getter,Setter,ToString,Accessors"/>
        </plugin>

        <!-- 通用代码生成器插件生成dao -->
        <plugin type="tk.mybatis.mapper.generator.TemplateFilePlugin">
<!--            <property name="baseMapper" value="com.xiaomi.dlc.dal.common.MobileSecurityBaseMapper"/>-->
            <property name="targetProject" value="src/main/java"/>
<!--            <property name="targetPackage" value="com.coderstring.starter.dao"/>-->
            <property name="templatePath" value="generator/mapper.ftl"/>
            <property name="mapperSuffix" value="Mapper"/>
            <property name="fileName" value="${tableClass.shortClassName}${mapperSuffix}.java"/>
        </plugin>


        <!-- 数据库连接 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/rbac?characterEncoding=UTF-8&amp;useSSL=false&amp;zeroDateTimeBehavior=convertToNull"
                        userId="root"
                        password="root">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!-- 数据库表对应的model层 -->
        <javaModelGenerator targetPackage="com.coderstring.starter.model"
                            targetProject="./src/main/java"/>

        <sqlMapGenerator targetPackage="mapper"
        targetProject="./src/main/resources/mybatis"/>

        <javaClientGenerator targetPackage="com.coderstring.starter.dao"
                             targetProject="./src/main/java" type="XMLMAPPER"/>

        <table tableName="sys_user" domainObjectName="SysUser">
            <generatedKey column="id" sqlStatement="JDBC"/>
        </table>
    </context>
</generatorConfiguration>
```