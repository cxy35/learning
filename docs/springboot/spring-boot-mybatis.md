---
title: Spring Boot 整合 MyBatis
date: 2019-12-07 09:24:59
categories: Spring Boot
tags: [Spring Boot, MyBatis]
toc: true
---
学习在 Spring Boot 中使用 MyBatis 来操作数据库。与 JdbcTemplate 相比，MyBatis 比较灵活，功能也很强大。在 Spring Boot 使用 MyBatis ，和 SSM 中相比简单的不得了。
<!-- more -->

## 1 创建工程并配置

创建 Spring Boot 项目 `spring-boot-mybatis` ，添加 `Web/MyBatis/MySQL` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-56cb7455ee36a82792cf477716a8916895e.png)

之后手动在 pom 文件中添加 Druid 数据库连接池依赖（Spring Boot 版本），最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.1</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
        <version>5.1.27</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

接着在 `application.properties` 配置文件中添加数据库相关信息的配置，如下：

```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.username=root
spring.datasource.password=000000
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/cxy35?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true
```

## 2 使用

新建 User 实体类，如下：

```java
public class User {
    private Integer id;
    private String username;
    private String address;

    // getter/setter
}
```

---

有两种配置方式，分别如下：

1. SQL 写在 XML 文件中

新建 `UserMapper.java` ，定义相关接口，如下：

```java
// 方式1：在 XML 中写 SQL >> UserMapper.xml
// @Mapper // 可在启动类中全局配置
public interface UserMapper {
    Integer addUser(User user);

    Integer deleteUserById(Integer id);

    Integer updateUserById(User user);

    List<User> getAllUser();
}
```

**关于 `UserMapper.java` 文件的扫描配置说明**：

- 在每个 XxxMapper.java 文件上都增加注解 `@Mapper` ，比较繁琐。
- 在 Spring Boot 启动类上增加注解 `@MapperScan(basePackages = "com.cxy35.sample.springboot.mybatis.mapper")` 指定 XxxMapper.java 文件所在的目录，完成全局配置。（推荐）

新建 `UserMapper.xml` ，定义相关接口的 SQL , 如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.cxy35.sample.springboot.mybatis.mapper.UserMapper">
    <insert id="addUser" parameterType="com.cxy35.sample.springboot.mybatis.pojo.User">
        insert into t_user (username,address) values (#{username},#{address});
    </insert>

    <delete id="deleteUserById">
        delete from t_user where id=#{id}
    </delete>

    <update id="updateUserById" parameterType="com.cxy35.sample.springboot.mybatis.pojo.User">
        update t_user set username=#{username},address=#{address} where id=#{id}
    </update>

    <select id="getAllUser" resultType="com.cxy35.sample.springboot.mybatis.pojo.User">
        select * from t_user;
    </select>
</mapper>
```

**关于 `UserMapper.xml` 文件的位置说明**：

- 放在 UserMapper.java 所在的包下面，会被自动扫描到。**但在项目打包时会被忽略掉，因此需要在 pom.xml 中配置 Maven 构建时的资源路径**。（推荐）

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
    </resources>
</build>
```

- 放在 `src/main/resources/mapper` 目录下，不能自动被扫描到，不用配置 Maven ，但需要在 application.properties 配置 mapper 路径。

```properties
# 默认与 XxxMapper.java 在同一个目录下（推荐），这里自定义目录
mybatis.mapper-locations=classpath:/mapper/*.xml
```

---

2. SQL 写在 JAVA 文件中

新建 `UserMapper2.java` ，定义相关接口和 SQL ，如下：

```java
// 方式2：通过全注解的方式来写 SQL >> UserMapper2.java，不写 XML 文件
// @Mapper // 可在启动类中全局配置
public interface UserMapper2 {

    @Insert({"insert into t_user(username,address) values(#{username},#{address})"})
    @SelectKey(statement = "select last_insert_id()", keyProperty = "id", before = false, resultType = Integer.class)
    Integer addUser(User user);

    @Delete("delete from t_user where id=#{id}")
    Integer deleteUserById(Integer id);

    @Update("update t_user set username=#{username},address=#{address} where id=#{id}")
    Integer updateUserById(User user);

    @Select("select * from t_user")
    List<User> getAllUsers();

    @Results({
            @Result(property = "id", column = "id"),
            @Result(property = "username", column = "u"),
            @Result(property = "address", column = "a")
    })
    @Select("select username as u,address as a,id as id from t_user where id=#{id}")
    User getUserById(Long id);

    @Select("select * from t_user where username like concat('%',#{name},'%')")
    List<User> getUsersByName(String name);
}
```

---

最后在测试类中注入 userMapper 或 userMapper2 完成测试，如下：

```java
@SpringBootTest
class SpringBootMybatisApplicationTests {

    @Autowired
    UserMapper userMapper;

    @Test
    public void addUser() {
        User user = new User();
        user.setUsername("zhangsan");
        user.setAddress("杭州");
        userMapper.addUser(user);
    }

    @Test
    public void deleteUserById() {
        userMapper.deleteUserById(1);
    }

    @Test
    public void updateUserById() {
        User user = new User();
        user.setId(1);
        user.setUsername("zhangsan2");
        user.setAddress("上海");
        userMapper.updateUserById(user);
    }

    @Test
    public void getAllUsers() {
        List<User> allUsers = userMapper.getAllUser();
        System.out.println(allUsers);
    }

}
```

## 3 源码解读

Mybatis 对应的自动化配置类是 `org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration` ，部分源码如下：

```java
@Configuration
@ConditionalOnClass({SqlSessionFactory.class, SqlSessionFactoryBean.class})
@ConditionalOnSingleCandidate(DataSource.class)
@EnableConfigurationProperties({MybatisProperties.class})
@AutoConfigureAfter({DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class})
public class MybatisAutoConfiguration implements InitializingBean {
    private static final Logger logger = LoggerFactory.getLogger(MybatisAutoConfiguration.class);
    private final MybatisProperties properties;
    private final Interceptor[] interceptors;
    private final TypeHandler[] typeHandlers;
    private final LanguageDriver[] languageDrivers;
    private final ResourceLoader resourceLoader;
    private final DatabaseIdProvider databaseIdProvider;
    private final List<ConfigurationCustomizer> configurationCustomizers;

    @Bean
    @ConditionalOnMissingBean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        
        // ......

        return factory.getObject();
    }

    @Bean
    @ConditionalOnMissingBean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        ExecutorType executorType = this.properties.getExecutorType();
        return executorType != null ? new SqlSessionTemplate(sqlSessionFactory, executorType) : new SqlSessionTemplate(sqlSessionFactory);
    }

    @Configuration
    @Import({MybatisAutoConfiguration.AutoConfiguredMapperScannerRegistrar.class})
    @ConditionalOnMissingBean({MapperFactoryBean.class, MapperScannerConfigurer.class})
    public static class MapperScannerRegistrarNotFoundConfiguration implements InitializingBean {
        public MapperScannerRegistrarNotFoundConfiguration() {
        }

        public void afterPropertiesSet() {
            MybatisAutoConfiguration.logger.debug("Not found configuration for registering mapper bean using @MapperScan, MapperFactoryBean and MapperScannerConfigurer.");
        }
    }

    // ......
}
```

在 SSM 中使用 Mybatis 一般需要自己提供 SqlSessionFactoryBean 和 MapperScannerConfigurer 两个 Bean ，从上述源码中可以看出 Spring Boot 提供了默认的，实现了开发者零配置使用。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-mybatis](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-mybatis)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)