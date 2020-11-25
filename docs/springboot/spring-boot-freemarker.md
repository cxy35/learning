---
title: Spring Boot 整合 Freemarker
date: 2019-11-15 10:02:03
categories: Spring Boot
tags: [Spring Boot, Freemarker]
toc: true
---
学习 Spring Boot 整合页面模板 Freemarker 。
<!-- more -->

## 1 Freemarker 简介

Freemarker 是一个相当老牌的开源的免费的模版引擎。通过 Freemarker 模版，我们可以将数据渲染成 HTML 网页、电子邮件、配置文件以及源代码等。Freemarker 不是面向最终用户的，而是一个 Java 类库，我们可以将之作为一个普通的组件嵌入到我们的产品中。来看一张来自 Freemarker 官网的图片：

![](https://oscimg.oschina.net/oscnet/up-650ad98c8be47b2ae93f67ec26fbcf30627.png)

可以看到，Freemarker 可以将模版和数据渲染成 HTML 。

Freemarker 模版后缀为 .ftl(FreeMarker Template Language)。FTL 是一种简单的、专用的语言，它不是像 Java 那样成熟的编程语言。在模板中，你可以专注于如何展现数据，而在模板之外可以专注于要展示什么数据。

## 2 整合 Freemarker

创建 Spring Boot 项目 spring-boot-freemarker ，增加 Web 和 Freemarker 依赖。

![](https://oscimg.oschina.net/oscnet/up-42dfdd244beecc890959659c8c27421b8f0.png)

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
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

Freemarker 提供了一整套的自动化配置方案，对应的源码如下：

- `org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration`

```java
@Configuration
@ConditionalOnClass({ freemarker.template.Configuration.class, FreeMarkerConfigurationFactory.class })
@EnableConfigurationProperties(FreeMarkerProperties.class)
@Import({ FreeMarkerServletWebConfiguration.class, FreeMarkerReactiveWebConfiguration.class,
                FreeMarkerNonWebConfiguration.class })
public class FreeMarkerAutoConfiguration {
}
```

1. `@Configuration` 注解表示这是一个配置类。
2. `@EnableConfigurationProperties` 注解表示开启 ConfigurationProperties ，即使得 FreemarkerProperties 类上配置的 @ConfigurationProperties 生效。
3. `@ConditionalOnClass` 表示当项目 classpath 下存在 freemarker.template.Configuration 和 FreeMarkerConfigurationFactory 时，当前的自动化配置类才会生效。只要项目中引入了 Freemarker 相关的依赖，这个配置就会生效。
4. `@Import` 导入了 FreeMarkerServletWebConfiguration 配置。

- `org.springframework.boot.autoconfigure.freemarker.FreeMarkerServletWebConfiguration`

```java
@Configuration
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass({ Servlet.class, FreeMarkerConfigurer.class })
@AutoConfigureAfter(WebMvcAutoConfiguration.class)
class FreeMarkerServletWebConfiguration extends AbstractFreeMarkerConfiguration {
        protected FreeMarkerServletWebConfiguration(FreeMarkerProperties properties) {
                super(properties);
        }
        @Bean
        @ConditionalOnMissingBean(FreeMarkerConfig.class)
        public FreeMarkerConfigurer freeMarkerConfigurer() {
                FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
                applyProperties(configurer);
                return configurer;
        }
        @Bean
        @ConditionalOnMissingBean(name = "freeMarkerViewResolver")
        @ConditionalOnProperty(name = "spring.freemarker.enabled", matchIfMissing = true)
        public FreeMarkerViewResolver freeMarkerViewResolver() {
                FreeMarkerViewResolver resolver = new FreeMarkerViewResolver();
                getProperties().applyToMvcViewResolver(resolver);
                return resolver;
        }
}
```

1. `@ConditionalOnWebApplication` 表示当前配置在 web 环境下才会生效。
2. `@ConditionalOnClass` 表示当前配置在存在 Servlet 和 FreeMarkerConfigurer 时才会生效。
3. `@AutoConfigureAfter` 表示当前自动化配置在 WebMvcAutoConfiguration 之后完成。
4. 在构造方法中，注入了 FreeMarkerProperties。
5. 在代码中提供了两个 bean： FreeMarkerConfigurer 和 FreeMarkerViewResolver。
6. FreeMarkerConfigurer 是 Freemarker 的一些基本配置，例如 templateLoaderPath、defaultEncoding 等。
7. FreeMarkerViewResolver 是视图解析器的基本配置，包含了 viewClass、suffix、allowRequestOverride、allowSessionOverride 等属性。

- `org.springframework.boot.autoconfigure.freemarker.FreeMarkerProperties`

```java
@ConfigurationProperties(prefix = "spring.freemarker")
public class FreeMarkerProperties extends AbstractTemplateViewResolverProperties {
        public static final String DEFAULT_TEMPLATE_LOADER_PATH = "classpath:/templates/";
        public static final String DEFAULT_PREFIX = "";
        public static final String DEFAULT_SUFFIX = ".ftl";
        /**
         * Well-known FreeMarker keys which are passed to FreeMarker's Configuration.
         */
        private Map<String, String> settings = new HashMap<>();
}
```

1. 通过 `@ConfigurationProperties` 注解，将 `application.properties` 中前缀为 `spring.freemarker` 的配置和这个类中的属性绑定。
2. 前三个 static 变量定义了默认的模板位置、视图解析器的前缀、后缀等。
3. 从前三行配置中，可以看出来，Freemarker 模板的默认位置在 `classpath:/templates/` 目录下，默认的后缀是 ftl 。
4. 这些配置，如果开发者不自己提供，则使用默认的，如果自己提供，则在 application.properties 中以 spring.freemarker 开头进行相关的配置。

在 src/main/java 下相应的包中新建 User 类，如下：

```java
public class User {
    private Long id;
    private String username;
    private String address;
    private Integer gender;

    // getter/setter
}
```

在 src/main/java 下相应的包中新建 UserController 类，如下：

```java
@Controller
public class UserController {
    @GetMapping("/user")
    public String user(Model model) {
        List<User> users = new ArrayList<>();
        Random random = new Random();
        for (int i = 0; i < 10; i++) {
            User user = new User();
            user.setId((long) i);
            user.setUsername("cxy35 >>> " + i);
            user.setAddress("https://cxy35.com >>> " + i);
            // 0 表示 男 1 表示 女 其他数字表示未知
            user.setGender(random.nextInt(3));
            users.add(user);
        }
        model.addAttribute("users", users);
        model.addAttribute("hello", "<h1>hello</h1>");
        model.addAttribute("world", "<h1>world</h1>");

        // 返回视图，默认为 src/main/resources/templates/user.ftl
        return "user";
    }
}
```

在 src/main/resources/templates 下新建 user.ftl ，如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<table border="1">
    <tr>
        <td>编号</td>
        <td>用户名</td>
        <td>用户地址</td>
    </tr>
    <#list users as u>
        <tr>
            <td>${u.id}</td>
            <td>${u.username}</td>
            <td>${u.address}</td>
        </tr>
    </#list>
</table>
</body>
</html>
```

启动项目，访问 http://120.0.0.1:8080/user 来验证。

## 3 配置说明

在 `application.properties` 中配置，以 `spring.freemarker` 开头：

```properties
# 这里可以覆盖 freemarker 的默认配置
# 模板文件位置，默认为 classpath:/templates/
# spring.freemarker.template-loader-path=classpath:/templates/freemarker/
# 是否开启缓存
# spring.freemarker.cache=false
# 模板文件后缀
# spring.freemarker.suffix=.ftl
# 模板文件编码
# spring.freemarker.charset=UTF-8
# 是否检查模板位置
# spring.freemarker.check-template-location=true
# Content-Type 的值
# spring.freemarker.content-type=text/html
# HttpServletRequest 的属性是否可以覆盖 controller 中 model 的同名项
# spring.freemarker.allow-request-override=false
# HttpSession的 属性是否可以覆盖 controller 中 model 的同名项
# spring.freemarker.allow-session-override=false
# 是否将 HttpServletRequest 中的属性添加到 Model 中
# spring.freemarker.expose-request-attributes=false
# 是否将 HttpSession中的 属性添加到 Model 中
# spring.freemarker.expose-session-attributes=false
# ......
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web//spring-boot-freemarker](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web//spring-boot-freemarker)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)