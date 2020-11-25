---
title: Spring Boot 构建 Rest 服务（Jpa）
date: 2019-12-24 15:11:47
categories: Spring Boot
tags: [Spring Boot, Rest, Jpa]
toc: true
---
学习在 Spring Boot 中结合 Jpa 构建 Rest 服务，只需要几行代码就能快速实现一个 RESTful 风格的增删改查接口。
<!-- more -->

## 1 概述

在当前移动互联网大环境下，前后端分离开发越来越普及，一般是一套后台对应多个前端项目，此时 RESTful 就有了用武之地。 Spring Boot 中相关的注解主要有（其实在Spring MVC 中也能使用）：

- `@RestController`
- `@GetMapping`
- `@PutMapping`
- `@PostMapping`
- `@DeleteMapping`
- `@ResponseBody`

## 2 创建工程并配置

创建 Spring Boot 项目 `spring-boot-jparest` ，添加 `Web/JPA/MySQL/Rest Repositories` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-b46904c2ffa6874713414e4656417feca50.png)

之后手动在 pom 文件中添加 Druid 数据库连接池依赖（Spring Boot 版本），最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
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

接着在 `application.properties` 配置文件中添加数据库相关信息的配置和 Jpa 的基本配置，如下：

```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.username=root
spring.datasource.password=000000
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/cxy35?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true

# JPA 配置
# 数据库为 MySQL
spring.jpa.database=mysql
# 在控制台打印 SQL
spring.jpa.show-sql=true
# 数据库平台
spring.jpa.database-platform=mysql
# 每次启动项目时，数据库初始化策略
spring.jpa.hibernate.ddl-auto=update
# 指定默认的存储引擎为 InnoDB ，否则默认情况下，自动创建表的时候会使用 MyISAM 作为表的存储引擎
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
```

## 3 使用

首先新建 `Book` 实体类，如下：

```java
@Entity(name = "t_book")
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    // @Column(name = "c_name")
    private String name;
    private String author;

    // getter/setter
}
```

接着新建 `BookDao` 接口，如下：

```java
public interface BookDao extends JpaRepository<Book, Integer> {
    // 默认接口
    // 新增：POST http://127.0.0.1:8080/books
    // 根据 id 删除：DELETE http://127.0.0.1:8080/books/{id}
    // 根据 id 编辑：PUT http://127.0.0.1:8080/books/{id}
    // 查询分页列表：GET http://127.0.0.1:8080/books
    // 根据 id 查询详情：GET http://127.0.0.1:8080/books/{id}
    // 查询自定义接口列表：GET http://127.0.0.1:8080/books/search
}
```

OK ，大功告成，一个 RESTful 风格的增删改查应用构建好了。惊讶，这么简单！关于 `Jpa` 的使用，可以参考 [Spring Boot 整合 Jpa](https://mp.weixin.qq.com/s/AWQPu02VY9BD9u_PLKrL7Q) ，这里不再赘述。

## 4 默认接口

启动项目，用 HTTP 请求工具来测试（如 `Postman` ）。此时我们的服务具备了一些默认的接口，如下：

1. 新增： POST `http://127.0.0.1:8080/books`

请求数据通过 JSON 的形式传递，新增成功之后，默认会返回新增成功的数据。

![](https://oscimg.oschina.net/oscnet/up-99efd760d9080dd8006bdc37ce20bbee389.png)

![](https://oscimg.oschina.net/oscnet/up-7c82be68291fa6e2920dd175ee36a41f9ab.png)

---

2. 根据 id 删除： DELETE `http://127.0.0.1:8080/books/{id}`

删除成功后，是没有返回值的。

![](https://oscimg.oschina.net/oscnet/up-5c25458ff943ba617c9adebf677b1758f75.png)

---

3. 根据 id 编辑： PUT `http://127.0.0.1:8080/books/{id}`

编辑的参数也是通过 JSON 的形式传递，编辑成功之后，默认会返回编辑成功的数据。

![](https://oscimg.oschina.net/oscnet/up-29bd3637e7bbb5fbf034e74fa85b62df868.png)

![](https://oscimg.oschina.net/oscnet/up-276f073aad572a9fd62f42e03386dad3773.png)

---

4. 查询分页列表： GET `http://127.0.0.1:8080/books`

默认请求路径是类名首字母小写，并且再加一个 s 后缀。没有传参数，表示查询第 1 页，每页 20 条数据。

![](https://oscimg.oschina.net/oscnet/up-f7f341c473074f34e506f22be7b5d1bb5b4.png)

查询结果中，除了该有的数据之外，也包含了分页数据：

![](https://oscimg.oschina.net/oscnet/up-463bf6a92c3051cb6a53469ced951e928f7.png)

分页数据说明：

- `size` 表示每页查询记录数。
- `totalElements` 表示总记录数。
- `totalPages` 表示总页数。
- `number` 表示当前页数，从 0 开始计。

如果要分页或者排序查询，可以使用 _links 中的链接。`http://127.0.0.1:8080/books?page=1&size=3&sort=id,desc` 。

![](https://oscimg.oschina.net/oscnet/up-b79d0aa0f087a8517e06f6315bf97308177.png)

---

5. 根据 id 查询详情： GET `http://127.0.0.1:8080/books/{id}`

![](https://oscimg.oschina.net/oscnet/up-9b71fe8587a1b7882ef1d0310384ef1a8bd.png)

6. 查询自定义接口列表： GET `http://127.0.0.1:8080/books/search`

## 5 自定义接口

一般自定义查询接口比较多，在 `BookDao` 中增加对应的接口即可（关于 `Jpa` 的使用，可以参考 [Spring Boot 整合 Jpa](https://mp.weixin.qq.com/s/AWQPu02VY9BD9u_PLKrL7Q) ），如下：

```java
public interface BookDao extends JpaRepository<Book, Integer> {
    // 自定义查询：GET http://127.0.0.1:8080/books/search/findBookByNameContaining?name=国
    // @RestResource(path = "byName", rel = "findByName")
    List<Book> findBookByNameContaining(@Param("name") String name);
}
```

重启项目，通过 GET `http://127.0.0.1:8080/books/search` 查看和 book 相关的自定义接口。

![](https://oscimg.oschina.net/oscnet/up-1dfe9ae088916b53985ce5491a8e447d1b3.png)

调用自定义接口： GET `http://127.0.0.1:8080/books/search/findBookByNameContaining?name=国`

![](https://oscimg.oschina.net/oscnet/up-be6582461970d69019ea5d813fb18e2fae5.png)

---

`@RestResource` 注解说明：

- `rel` 表示接口查询中，这个方法的 key 。
- `path` 表示请求路径。
- `exported` 表示是否暴露接口，默认为 true ，表示暴露接口，即方法可以在前端调用，如果仅仅只是想定义一个方法，不需要在前端调用这个方法，可以设置 exported 属性为 false 。

![](https://oscimg.oschina.net/oscnet/up-f51d3bbf961b068220aecac5f7fc9312b7c.png)

如果不想暴露官方定义好的方法，例如根据 id 删除数据，只需要在自定义接口中重写该方法，然后在该方法上加 `@RestResource `注解并且配置相关属性 `exported = false` 即可，如：

```java
public interface BookDao extends JpaRepository<Book, Integer> {
    // 避免暴露官方定义好的方法
    // @Override
    // @RestResource(exported = false)
    // void deleteById(Long id);
}
```

## 6 更多配置

- 请求路径和生成的 JSON 字符串中的相关名称配置。

```java
// @RepositoryRestResource(path = "bs", collectionResourceRel = "bs", itemResourceRel = "b")
public interface BookDao extends JpaRepository<Book, Integer> {
}
```

![](https://oscimg.oschina.net/oscnet/up-ffa2922a30a7a89a2c06b5dcdd4cc0745a4.png)

- Rest 基本参数配置

有两种方案：

1. 在 `application.properties` 配置文件中增加相关配置（推荐），如下：

```properties
# Rest 配置方式1（推荐）：优先级低于自定义配置类
# 给所有的接口添加统一的前缀，默认无
# spring.data.rest.base-path=/api
# 配置排序参数的 key ，默认 sort
# spring.data.rest.sort-param-name=sort
# 配置分页查询时页码的 key，默认 page
# spring.data.rest.page-param-name=page
# 配置分页查询时每页查询页数的 key，默认 size
# spring.data.rest.limit-param-name=size
# 配置每页最大查询记录数，默认 20
spring.data.rest.max-page-size=2
# 分页查询时默认的页码，默认 0
# spring.data.rest.default-page-size=0
# 更新成功时是否返回更新记录，默认 true
# spring.data.rest.return-body-on-update=true
# 添加成功时是否返回添加记录，默认 true
# spring.data.rest.return-body-on-create=true
```

2. 新增自定义配置类 `RestConfig` ，如下：

```java
// Rest 配置方式2：自定义配置类，优先级高于配置文件
@Configuration
public class RestConfig implements RepositoryRestConfigurer {
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config) {
        // config.setBasePath("/api").setDefaultPageSize(2);
    }
}
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-rest/spring-boot-jparest](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-rest/spring-boot-jparest)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)