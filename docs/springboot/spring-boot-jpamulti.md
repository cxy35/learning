---
title: Spring Boot 整合 Jpa 多数据源
date: 2019-12-11 10:47:24
categories: Spring Boot
tags: [Spring Boot, Jpa, 多数据源]
toc: true
---
学习在 Spring Boot 中使用 Jpa 多数据源来操作不同的数据库。
<!-- more -->

## 1 创建工程并配置

创建 Spring Boot 项目 `spring-boot-jpamulti` ，添加 `Web/JDBC/MySQL` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-d56d0986b2b4f777fece8ba276bc5013d93.png)

之后手动在 pom 文件中添加 Druid 数据库连接池依赖（Spring Boot 版本），最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
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

# JPA配置
# 数据库为 MySQL
spring.jpa.properties.database=mysql
# 在控制台打印 SQL
spring.jpa.properties.show-sql=true
# 数据库平台
spring.jpa.properties.database-platform=mysql
# 每次启动项目时，数据库初始化策略
spring.jpa.properties.hibernate.ddl-auto=update
# 指定默认的存储引擎为 InnoDB ，否则默认情况下，自动创建表的时候会使用 MyISAM 作为表的存储引擎
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
```

1. 数据源配置

因为配置数据库的 key 变化了，导致上述配置无法被 Spring Boot 自动加载，需要我们自己去加载。增加 `DataSourceConfig` 数据源配置类，使用 Spring Boot 提供的类型安全的属性注入方式来加载上述配置，并创建对应的两个数据源 `DataSource` 实例，如下：

```java
@Configuration
public class DataSourceConfig {
    @Bean
    @Primary
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

> 注意这里多了一个 `@Primary` 注解，表示当某一个类存在多个实例时，优先使用哪个实例，不配置的话在项目启动时会出错。

---

2. Jpa 配置

接下来是 Jpa 的配置，新增 `JpaConfig1` 和 `JpaConfig2` 两个配置类，用上述两个数据源分别创建对应的 `LocalContainerEntityManagerFactoryBean` 和 `PlatformTransactionManager` 实例（注意 Bean 的名称要不一样），分别如下：

```java
@Configuration
@EnableJpaRepositories(basePackages = "com.cxy35.sample.springboot.jpamulti.dao1", entityManagerFactoryRef = "localContainerEntityManagerFactoryBean1", transactionManagerRef = "platformTransactionManager1")
public class JpaConfig1 {
    // 此时 Spring 容器中有两个 DataSource 类型的 Bean ，所以这里需要按名称 byName 查找
    @Autowired
    @Qualifier("dsOne")
    DataSource dsOne;

    @Autowired
    JpaProperties jpaProperties;

    @Bean
    @Primary
    LocalContainerEntityManagerFactoryBean localContainerEntityManagerFactoryBean1(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(dsOne)
                .properties(jpaProperties.getProperties())
                .persistenceUnit("pu1")
                .packages("com.cxy35.sample.springboot.jpamulti.pojo")
                .build();
    }

    @Bean
    PlatformTransactionManager platformTransactionManager1(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(localContainerEntityManagerFactoryBean1(builder).getObject());
    }
}
```

```java
@Configuration
@EnableJpaRepositories(basePackages = "com.cxy35.sample.springboot.jpamulti.dao2", entityManagerFactoryRef = "localContainerEntityManagerFactoryBean2", transactionManagerRef = "platformTransactionManager2")
public class JpaConfig2 {
    // 此时 Spring 容器中有两个 DataSource 类型的 Bean ，所以这里需要按名称 byName 查找
    @Autowired
    @Qualifier("dsTwo")
    DataSource dsTwo;

    @Autowired
    JpaProperties jpaProperties;

    @Bean
    LocalContainerEntityManagerFactoryBean localContainerEntityManagerFactoryBean2(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(dsTwo)
                .properties(jpaProperties.getProperties())
                .persistenceUnit("pu2")
                .packages("com.cxy35.sample.springboot.jpamulti.pojo")
                .build();
    }

    @Bean
    PlatformTransactionManager platformTransactionManager2(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(localContainerEntityManagerFactoryBean2(builder).getObject());
    }
}
```

**关于 Jpa 配置类的说明**：

- 配置 dao 的位置：通过 `basePackages` 分别配置了扫描 `dao1` 和 `dao2` 路径，之后在这两个路径下放 `XxxDao.java` ，所有操作会自动对应着不同的数据源。
- 通过 `entityManagerFactoryRef` 和 `transactionManagerRef` 分别指定不同 Bean 的引用名字。
- 配置实体类的位置：在提供 `LocalContainerEntityManagerFactoryBean` 的时候，需要指定 `packages` ，即这个数据源对应的实体类所在的位置。
- 注意 persistenceUnit 的名字要不同。
- 注意实体类可以共用。

## 2 使用

配置完成之后，在对应的位置分别提供实体类和 dao 即可。关于 Jpa 的使用，可以参考 [Spring Boot 整合 Jpa](https://mp.weixin.qq.com/s/AWQPu02VY9BD9u_PLKrL7Q) ，这里不再赘述。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-jpamulti](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-dao/spring-boot-jpamulti)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)