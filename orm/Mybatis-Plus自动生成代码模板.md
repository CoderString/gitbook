### Mybatis-Plus自动生成代码模板

项目大开发过程中如果为了尽可能的提高开发的效率，有的时候我们会借助各种各样的开发工具，针对Mybatis-Plus， 我们一般会采用自动生成我们需要的entity、dao、service、controller代码，那么到底应该如何生成呢？下面将展开讲解。

### 依赖引入

```
   <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-generator</artifactId>
        <version>3.3.2</version>
    </dependency>

    <!-- velocity 模板引擎 -->
    <dependency>
        <groupId>org.apache.velocity</groupId>
        <artifactId>velocity-engine-core</artifactId>
        <version>2.3</version>
    </dependency>

    <!-- Mybatis plus逆向模板引擎依赖 -->
    <dependency>
        <groupId>org.freemarker</groupId>
        <artifactId>freemarker</artifactId>
        <version>2.3.30</version>
    </dependency>
```

### 自动生成代码

```
public class Generator {

    public static void main(String[] args) {

        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();
        // set freemarker 模板引擎
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        // gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("CoderString");
        gc.setOpen(false);
        gc.setBaseResultMap(true);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/rbac?serverTimezone=UTC&useUnicode=true&useSSL=false&characterEncoding=utf8");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("root");
        mpg.setDataSource(dsc);

        /**
         * 包配置
         * 简单来说 就是写绝对路径
         */
        PackageConfig pc = new PackageConfig();
        //pc.setModuleName("rbac-base");
        pc.setParent("com.coderstring.rbac");
        //指定生成文件导入的包
        pc.setEntity("base.entity");
        pc.setService("service");
        pc.setServiceImpl("service.impl");
        pc.setController("web.controller");
        pc.setMapper("dao");
        pc.setXml(null);
        //指定生成文件的绝对路径
        Map<String, String> pathInfo = new HashMap<>();
        String parentPath = "/src/main/java/com/coderstring/rbac/";
        String conStr = "/rbac-";

        String entityPath = projectPath.concat(conStr).concat("base").concat(parentPath).concat("base/entity");
        String mapper_path = projectPath.concat(conStr).concat("dao").concat(parentPath).concat("dao");
        String xml_path = projectPath.concat(conStr).concat("dao").concat("/src/main/resources/dao");
        String service_path = projectPath.concat(conStr).concat("service").concat(parentPath).concat("service");
        String service_impl_path = projectPath.concat(conStr).concat("service").concat(parentPath).concat("service/impl");
        String controller_path = projectPath.concat(conStr).concat("web").concat(parentPath).concat("web/controller");

        pathInfo.put("entity_path", entityPath);
        pathInfo.put("mapper_path", mapper_path);
        pathInfo.put("xml_path", xml_path);
        pathInfo.put("service_path", service_path);
        pathInfo.put("service_impl_path", service_impl_path);
        pathInfo.put("controller_path", controller_path);
        pc.setPathInfo(pathInfo);
        mpg.setPackageInfo(pc);

        // 自定义配置
        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };
        mpg.setCfg(cfg);
        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        strategy.setControllerMappingHyphenStyle(true);//驼峰转
        strategy.setInclude("sys_role_menu");//表名
        strategy.setControllerMappingHyphenStyle(true);
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
}

```

### 参考资料

1、https://juejin.cn/post/6844903921098424328