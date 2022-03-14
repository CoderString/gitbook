### 自定义实现一个starter

---

> 一般针对一些常用的功能，如果我们不想通过复制拷贝文件的方式来达到功能共用的目的，这个时候我们其实可以参考springboot项目创建一个属于我们自己的定制starter，当其他的项目需要这部分的功能的时候，我们就可以通过starter的方式引入。那么，starter究竟应该如何定制呢?如果有兴趣的话可以接着往下看。


#### 自定义starter实践

定义属于自己的starer其实也很简单，主要的步骤分为下面的几个步骤：引入创建starer所需的依赖、编写自定义的properties属性文件、创建自动配置类bean构造类、创建spring.factories文件引入自动创建bean的配置类。


- maven依赖引入

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

- 属性配置类配置

```
@ConfigurationProperties(prefix = "coderstring.person")
public class PersonProperties implements Serializable {


    /**
     * 属性配置的用户名
     */
    private String name;

    /**
     * 属性配置的年龄
     */
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

- 业务逻辑处理类

```
public class PersonService {

    private PersonProperties properties;

    public PersonService() {}

    public PersonService(PersonProperties properties) {
        this.properties = properties;
    }

    public void sayHello() {
        String message = String.format("大家好，我叫：%s，今年　%s岁", properties.getName(), properties.getAge());
        System.out.println(message);
    }
}

```

- 自动注入bean装配类

```
@Configuration
@EnableConfigurationProperties(PersonProperties.class)
@ConditionalOnClass(PersonService.class)
@ConditionalOnProperty(prefix = "coderstring.person", value = "enabled", matchIfMissing = true)
public class PersonServiceAutoConfiguration {

    @Resource
    private PersonProperties properties;


    @Bean
    @ConditionalOnMissingBean(PersonService.class)
    public PersonService personService() {
        return new PersonService(properties);
    }
}
```

- 配置自定义属性装配类(resources/META-INF/spring.factories)

org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.coderstring.starter.config.PersonServiceAutoConfiguration

#### 使用

直接引入项目的依赖就可以了，类似于spring-boot-starter-web的引入方式

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>xxx-spring-boot-starter</artifactId>
    <version>yyy</version>
</dependency>
```