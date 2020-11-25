---
title: Spring Boot 整合 Thymeleaf
date: 2019-11-13 10:01:40
categories: Spring Boot
tags: [Spring Boot, Thymeleaf]
toc: true
---
学习 Spring Boot 整合页面模板 Thymeleaf 。
<!-- more -->

## 1 Thymeleaf 简介

Thymeleaf 是新一代 Java 模板引擎，它类似于 Velocity 、 FreeMarker 等传统 Java 模板引擎，但是与传统 Java 模板引擎不同的是，Thymeleaf 支持 HTML 原型。

它既可以让前端工程师在浏览器中直接打开查看样式，也可以让后端工程师结合真实数据查看显示效果，同时，Spring Boot 提供了 Thymeleaf 自动化配置解决方案，因此在 SpringBoot 中使用 Thymeleaf 非常方便。

事实上， Thymeleaf 除了展示基本的 HTML ，进行页面渲染之外，也可以作为一个 HTML 片段进行渲染，例如我们在做邮件发送时，可以使用 Thymeleaf 作为邮件发送模板。

另外，由于 Thymeleaf 模板后缀为 .html，可以直接被浏览器打开，因此，预览时非常方便。

## 2 整合 Thymeleaf

创建 Spring Boot 项目 spring-boot-thymeleaf ，增加 Web 和 Thymeleaf 依赖。

![](https://oscimg.oschina.net/oscnet/up-d1270f3ecc45f3a31946fa6f6773f24af6a.png)

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
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

Thymeleaf 提供了一整套的自动化配置方案，对应的源码如下：

- `org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration`

```java
@Configuration
@EnableConfigurationProperties(ThymeleafProperties.class)
@ConditionalOnClass({ TemplateMode.class, SpringTemplateEngine.class })
@AutoConfigureAfter({ WebMvcAutoConfiguration.class, WebFluxAutoConfiguration.class })
public class ThymeleafAutoConfiguration {
}
```

1. `@Configuration` 注解表示这是一个配置类。
2. `@EnableConfigurationProperties` 注解表示开启 ConfigurationProperties ，即使得 ThymeleafProperties 类上配置的 @ConfigurationProperties 生效。
3. `@ConditionalOnClass` 表示当项目 classpath 下存在 TemplateMode 和 SpringTemplateEngine 时，当前的自动化配置类才会生效。只要项目中引入了 Thymeleaf 相关的依赖，这个配置就会生效。
4. `@AutoConfigureAfter` 表示当前自动化配置在 WebMvcAutoConfiguration 和 WebFluxAutoConfiguration 之后完成。

- `org.springframework.boot.autoconfigure.thymeleaf.ThymeleafProperties`

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {
        private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;
        public static final String DEFAULT_PREFIX = "classpath:/templates/";
        public static final String DEFAULT_SUFFIX = ".html";
        private boolean checkTemplate = true;
        private boolean checkTemplateLocation = true;
        private String prefix = DEFAULT_PREFIX;
        private String suffix = DEFAULT_SUFFIX;
        private String mode = "HTML";
        private Charset encoding = DEFAULT_ENCODING;
        private boolean cache = true;
        
        // ...
}
```

1. 通过 `@ConfigurationProperties` 注解，将 `application.properties` 中前缀为 `spring.thymeleaf` 的配置和这个类中的属性绑定。
2. 前三个 static 变量定义了默认的编码格式、视图解析器的前缀、后缀等。
3. 从前三行配置中，可以看出来，Thymeleaf 模板的默认位置在 `classpath:/templates/` 目录下，默认的后缀是 html 。
4. 这些配置，如果开发者不自己提供，则使用默认的，如果自己提供，则在 application.properties 中以 spring.thymeleaf 开头进行相关的配置。

在 src/main/java 下相应的包中新建 Book 类，如下：

```java
public class Book {
    private Integer id;
    private String name;
    private String author;
    private Double price;

    // getter/setter
}
```

在 src/main/java 下相应的包中新建 BookController 类，如下：

```java
@Controller
public class BookController {
    @GetMapping("/book")
    public String book(Model model) {
        List<Book> bookList = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Book book = new Book();
            book.setId(i);
            book.setName("三国演义:" + i);
            book.setAuthor("罗贯中:" + i);
            book.setPrice(30.0);
            bookList.add(book);
        }
        model.addAttribute("books", bookList);
        model.addAttribute("username","张三");

        // 返回视图，默认为 src/main/resources/templates/book.html
        return "book";
    }
}
```

在 src/main/resources/templates 下新建 book.html ，如下：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<table border="1">
    <tr>
        <td>图书编号</td>
        <td>图书名称</td>
        <td>图书作者</td>
        <td>图书价格</td>
    </tr>
    <tr th:each="book :${books}">
        <td th:text="${book.id}"></td>
        <td th:text="${book.name}"></td>
        <td th:text="${book.author}"></td>
        <td th:text="${book.price}"></td>
    </tr>
</table>

<script th:inline="javascript">
    var username = [[${username}]];
    console.log("Thymeleaf 支持在 js 中直接获取 Model 中的变量。username = " + username);
</script>
</body>
</html>
```

1. 通过 `<html lang="en" xmlns:th="http://www.thymeleaf.org">` 引入 thymeleaf 名称空间。
2. 通过 `th:each` 指令遍历集合，通过 `th:text` 指令展示数据。
3. 通过 `[[${username}]]` 在 js 中直接获取 Model 中的变量。
4. Thymeleaf 的其他用法可以参考官方文档：[https://www.thymeleaf.org](https://www.thymeleaf.org) 。

启动项目，访问 http://120.0.0.1:8080/book 来验证。

## 3 手动渲染 Thymeleaf

另外我们可以使用 TemplateEngine 实例手动渲染 Thymeleaf 模板，一般在发邮件时用到，可查看文章 [Spring Boot 整合邮件发送](TODO) 。

## 4 配置说明

在 `application.properties` 中配置，以 `spring.thymeleaf` 开头：

```properties
# 这里可以覆盖 thymeleaf 的默认配置
# 模板文件位置，默认为 classpath:/templates/
# spring.thymeleaf.prefix=classpath:/templates/thymeleaf/
# 是否开启缓存
# spring.thymeleaf.cache=false
# ......
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web//spring-boot-thymeleaf](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web//spring-boot-thymeleaf)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)