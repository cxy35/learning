---
title: Spring Boot 整合 JdbcTemplate 多数据源
date: 2019-12-05 14:08:57
categories: Spring Boot
tags: [Spring Boot, JdbcTemplate, 多数据源]
toc: true
---
学习在 Spring Boot 中使用 JdbcTemplate 多数据源来操作不同的数据库。JdbcTemplate 是 Spring 自带的，虽然功能没有 MyBatis 强大，但使用简单。
<!-- more -->

## 1 创建工程并配置

创建 Spring Boot 项目 `spring-boot-jdbctemplatemulti` ，添加 `Web/JDBC/MySQL` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-fdcae0dc81e9b6c2b895d96a6ccd5477fc8.png)

之后手动在 pom 文件中添加 Druid 数据库连接池依赖（Spring Boot 版本），最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
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

接着在 `application.properties` 配置文件中添加数据库相关信息的配置，有两组配置，用 one 和 two 区分，如下：

```properties
spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.one.username=root
spring.datasource.one.password=000000
spring.datasource.one.url=jdbc:mysql://127.0.0.1:3306/cxy35?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true

spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.two.username=root
spring.datasource.two.password=000000
spring.datasource.two.url=jdbc:mysql://127.0.0.1:3306/cxy35_2?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true
```

1. 数据源配置

因为配置数据库的 key 变化了，导致上述配置无法被 Spring Boot 自动加载，需要我们自己去加载。增加 `DataSourceConfig` 数据源配置类，使用 Spring Boot 提供的类型安全的属性注入方式来加载上述配置，并创建对应的两个数据源 `DataSource` 实例，如下：

```java
@Configuration
public class DataSourceConfig {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.one")
    DataSource dsOne() {
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.two")
    DataSource dsTwo() {
        return DruidDataSourceBuilder.create().build();
    }
}
```

---

2. JdbcTemplate 配置

接下来是 JdbcTemplate 的配置，新增 `JdbcTemplateConfig` 配置类，用上述两个数据源分别创建两个对应 `JdbcTemplate` 实例，如下：

```java
@Configuration
public class JdbcTemplateConfig {
    @Bean
    // 此时 Spring 容器中有两个 DataSource 类型的 Bean ，所以这里需要按名称 byName 查找
    JdbcTemplate jdbcTemplateOne(@Qualifier("dsOne") DataSource dsOne) {
        return new JdbcTemplate(dsOne);
    }

    @Bean
    // 此时 Spring 容器中有两个 DataSource 类型的 Bean ，所以这里需要按名称 byName 查找
    JdbcTemplate jdbcTemplateTwo(@Qualifier("dsTwo") DataSource dsTwo) {
        return new JdbcTemplate(dsTwo);
    }
}
```

## 2 使用

配置完成之后，在对应的位置分别提供实体类和 service 即可。关于 JdbcTemplate 的使用，可以参考 [Spring Boot 整合 JdbcTemplate](https://mp.weixin.qq.com/s/ExH3Z_9bAzBKWLaOB1IIpw) ，这里不再赘述。但同样需要注意 Spring 容器中的 JdbcTemplate 也有两个，因此不能通过 byType 的方式注入，可通过 `@Resource` 或 `@Autowired/@Qualifier` 来解决，如下：

```java
@Service
public class UserService1 {
    // 因为同类型的有多个，所以需要指定名称
    // 注入方法1
    @Autowired
    @Qualifier("jdbcTemplateOne")
    JdbcTemplate jdbcTemplateOne;

    // 注入方法2
    // @Resource(name = "jdbcTemplateOne")
    // JdbcTemplate jdbcTemplateOne;
}
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-jdbctemplatemulti](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-jdbctemplatemulti)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)