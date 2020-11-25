---
title: Spring Boot 配置 properties
date: 2019-11-03 15:17:15
categories: Spring Boot
tags: [Spring Boot, properties]
toc: true
---
学习 Spring Boot 项目中的配置文件（ properties 格式），如： application.properties 。
<!-- more -->

## 1 文件位置

Spring Boot 项目中的配置文件 `application.properties` 最常见的位置在 `src/main/resources` 目录下，其实共有 4 个默认位置能放，如下（优先级: 1 > 2 > 3 > 4 ）：

1. 项目根目录下的 config 目录下。
2. 项目的根目录下。
3. classpath 下的 config 目录下。
4. classpath 目录下。

Spring Boot 启动时，默认会从这 4 个位置按顺序去查找相关属性并加载，重复的属性以优先级高的为准。

但并非绝对的，我们也可以自定义位置（如：`src/main/resources/cxy35/application.properties` ），并在项目启动时通过 **`spring.config.location`** 属性来手动的指定配置文件的位置，指定方式如下：

1. IntelliJ IDEA 中。

![](https://oscimg.oschina.net/oscnet/up-6a944dabac87e2b73cbb36225d871b587a9.png)

2. 命令行中。

```bash
java -jar spring-boot-properties-0.0.1-SNAPSHOT.jar --spring.config.location=classpath:/cxy35/
```

**注意**：通过 **`spring.config.location`** 属性指定时，表示自己重新定义配置文件的位置，项目启动时就按照定义的位置去查找配置文件，这种定义方式会覆盖掉默认的 4 个位置。另外可以通过 **`spring.config.additional-location`** 属性来指定，表示在默认的 4 个位置的基础上，再添加几个位置，新添加的位置的优先级大于原本的位置。

## 2 文件名

Spring Boot 项目中的配置文件默认文件名是 `application.properties` ，与文件位置类似，也可以自定义，比如叫 `app.properties` ，并在项目启动时通过 **`spring.config.name`** 属性来手动的指定配置文件的文件名，如：`java -jar spring-boot-properties-0.0.1-SNAPSHOT.jar --spring.config.name=app` 。

当然，配置文件的位置和文件名可以同时自定义。

## 3 普通的属性注入

首先在 `application.properties` 配置文件中定义属性：

```properties
book.id=99
book.name=三国演义
book.author=罗贯中
```

再定义一个 Book 类，并通过 `@Value` 注解将这些属性注入到 Book 对象中（**注意： Book 对象必须要交给 Spring 容器去管理**）：

```java
@Component
public class Book {
    @Value("${book.id}")
    private Long id;
    @Value("${book.name}")
    private String name;
    @Value("${book.author}")
    private String author;

    // getter/setter
}
```

因为 `application.properties` 配置文件会被自动加载，所以上述属性可以注入成功，可通过在 controller 或者单元测试中注入 Book 对象来测试。

但 `application.properties` 我们一般用来放系统相关的配置，可以自定义 properties 文件来存在自定义配置，如新建 `src/main/resources/book.properties` ，内容如下：

```properties
book.id=99
book.name=三国演义
book.author=罗贯中
```

此时 `book.properties` 文件并不会被自动加载，需要在 Book 类中通过 `@PropertySource` 来引入：

```java
@Component
@PropertySource("classpath:book.properties")
public class Book {
    @Value("${book.id}")
    private Long id;
    @Value("${book.name}")
    private String name;
    @Value("${book.author}")
    private String author;

    // getter/setter
}
```

这样 `book.properties` 文件中的属性就可以注入成功了。

上述方式在 Spring 中也可以使用，和 Spring Boot 没有关系。

## 4 类型安全的属性注入（推荐）

当配置的属性非常多的时候，上述方式工作量大且容易出错，所以就不合适了。在 Spring Boot 中引入了类型安全的属性注入，通过 `@ConfigurationProperties` 注解来实现，如下：

```java
@Component
@PropertySource("classpath:book.properties")
@ConfigurationProperties(prefix = "book")
public class Book {
    private Long id;
    private String name;
    private String author;

    // getter/setter
}
```

## 5 properties 与 yaml 配置的区别

1. properties 配置无序，yaml 配置有序。在有些配置中顺序是非常有用的，例如 Spring Cloud Zuul 的配置。
2. yaml 配置目前不支持 `@PropertySource` 注解。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-properties](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-properties)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)