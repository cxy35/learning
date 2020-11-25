---
title: Spring Boot 整合 JSON（Jackson / Gson / FastJson）
date: 2019-11-18 14:56:51
categories: Spring Boot
tags: [Spring Boot, JSON]
toc: true
---
学习 Spring Boot 整合 JSON（Jackson / Gson / FastJson） 。
<!-- more -->

## 1 Spring MVC 整合 JSON

先来回顾下在 Spring MVC 中如何整合 JSON 。 Spring MVC 可以接收 JSON 参数，也可以返回 JSON 参数，这一切依赖于 HttpMessageConverter 。它可以将一个 JSON 字符串转为对象，也可以将一个对象转为 JSON 字符串，实际上它的底层还是依赖于具体的 JSON 库。因此所有的 JSON 库要在 Spring MVC 中自动返回或者接收 JSON ，都必须提供和自己相关的 HttpMessageConverter 。

Spring MVC 中，默认提供了 Jackson 和 Gson 的 HttpMessageConverter ，分别是： `MappingJackson2HttpMessageConverter` 和 `GsonHttpMessageConverter` 。正因为如此，我们在 Spring MVC 中，如果要使用 JSON ，对于 Jackson 和 Gson 我们只需要添加依赖，加完依赖就可以直接使用了。具体的配置是在 AllEncompassingFormHttpMessageConverter 类中完成的。

但是如果我们使用 FastJson ，默认情况下，Spring MVC 并没有提供 FastJson 的 HttpMessageConverter ，因此需要我们自己提供，如果是在 XML 配置中， FastJson 除了添加依赖，还要显式配置 HttpMessageConverter ，如下：

```xml
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

## 2 Spring Boot 整合 JSON

Spring Boot 中关于 JSON 的默认解析方案是 Jackson ，会自动导入相关依赖。如果想要使用 Gson 或 FastJson ，则需要我们手动添加相关依赖，并排除掉默认的 JSON 依赖。

### 2.1 Jackson

创建 Spring Boot 项目 spring-boot-jackson ，增加 Web 依赖。

```xml
<dependencies>
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

在 src/main/java 下相应的包中新建 User 类，如下：

```java
public class User {
    private Integer id;
    private String username;
    private String address;
    private Date birthday;

    // getter/setter
}
```

在 src/main/java 下相应的包中新建 UserController 类，如下：

```java
@RestController
public class UserController {
    @GetMapping("/user")
    public List<User> getAllUser() {
        List<User> users = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            User user = new User();
            user.setUsername("cxy35 >> " + i);
            user.setAddress("https://cxy35.com >> " + i);
            user.setId(i);
            user.setBirthday(new Date());
            users.add(user);
        }
        return users;
    }
}
```

启动项目，访问 http://120.0.0.1:8080/user 来验证，此时就可以正常返回 json 了，但发现 birthday 的日期格式展示不是很友好，可以通过覆盖 Jackson 的默认 Bean ，并增加日期格式化相关代码来解决（这里只增加处理日期格式化的功能，如果有其他需求，增加相应的代码即可），有 2 种方法覆盖。

在 src/main/java 下相应的包中新建 WebMvcConfig 配置类，如下：

```java
@Configuration
public class WebMvcConfig {

    // 实现日期格式化方法1：
    // 覆盖 JacksonHttpMessageConvertersConfiguration 中默认的 MappingJackson2HttpMessageConverter
    /*@Bean
    MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter() {
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        ObjectMapper om = new ObjectMapper();
        om.setDateFormat(new SimpleDateFormat("yyyy/MM/dd"));
        converter.setObjectMapper(om);
        return converter;
    }*/

    // 实现日期格式化方法2（更小的粒度）：
    // 覆盖 JacksonHttpMessageConvertersConfiguration 中默认的 ObjectMapper（由 JacksonAutoConfiguration 中注入）
    @Bean
    ObjectMapper objectMapper() {
        ObjectMapper om = new ObjectMapper();
        om.setDateFormat(new SimpleDateFormat("yyyy-MM-dd"));
        return om;
    }
}
```

另外还有 1 种方式可以实现日期格式化，通过在对应 pojo 的属性上加注解，如下：

```java
public class User {
    private Integer id;
    private String username;
    private String address;
    @JsonFormat(pattern = "yyyy-MM-dd")
    private Date birthday;

    // getter/setter
}
```

启动项目，访问 http://120.0.0.1:8080/user 来验证。

### 2.2 Gson

创建 Spring Boot 项目 spring-boot-gson ，增加 Web 和 Gson 依赖，并排除掉默认的 JSON 依赖。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-json</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
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

在 src/main/java 下相应的包中新建 User 类，如下：

```java
public class User {
    private Integer id;
    private String username;
    private String address;
    private Date birthday;

    // getter/setter
}
```

在 src/main/java 下相应的包中新建 UserController 类，如下：

```java
@RestController
public class UserController {
    @GetMapping("/user")
    public List<User> getAllUser() {
        List<User> users = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            User user = new User();
            user.setUsername("cxy35 >> " + i);
            user.setAddress("https://cxy35.com >> " + i);
            user.setId(i);
            user.setBirthday(new Date());
            users.add(user);
        }
        return users;
    }
}
```

启动项目，访问 http://120.0.0.1:8080/user 来验证，此时就可以正常返回 json 了，但发现 birthday 的日期格式展示不是很友好，可以通过覆盖 Gson 的默认 Bean ，并增加日期格式化相关代码来解决（这里只增加处理日期格式化的功能，如果有其他需求，增加相应的代码即可），有 2 种方法覆盖。

在 src/main/java 下相应的包中新建 WebMvcConfig 配置类，如下：

```java
@Configuration
public class WebMvcConfig {

    // 实现日期格式化方法1：
    // 覆盖 GsonHttpMessageConvertersConfiguration 中默认的 GsonHttpMessageConverter
    /*@Bean
    GsonHttpMessageConverter gsonHttpMessageConverter() {
        GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
        converter.setGson(new GsonBuilder().setDateFormat("yyyy/MM/dd").create());
        return converter;
    }*/

    // 实现日期格式化方法2（更小的粒度）：
    // 覆盖 GsonHttpMessageConvertersConfiguration 中默认的 Gson（由 GsonAutoConfiguration 中注入）
    @Bean
    Gson gson() {
        return new GsonBuilder().setDateFormat("yyyy-MM-dd").create();
    }
}
```

另外还有 1 种方式可以实现日期格式化，通过在对应 pojo 的属性上加注解，如下：

```java
public class User {
    private Integer id;
    private String username;
    private String address;
    @JsonFormat(pattern = "yyyy-MM-dd")
    private Date birthday;

    // getter/setter
}
```

启动项目，访问 http://120.0.0.1:8080/user 来验证。

### 2.3 FastJson

创建 Spring Boot 项目 spring-boot-fastjson ，增加 Web 和 FastJson 依赖，并排除掉默认的 JSON 依赖。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-json</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.60</version>
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

在 src/main/java 下相应的包中新建 User 类，如下：

```java
public class User {
    private Integer id;
    private String username;
    private String address;
    private Date birthday;

    // getter/setter
}
```

在 src/main/java 下相应的包中新建 UserController 类，如下：

```java
@RestController
public class UserController {
    @GetMapping("/user")
    public List<User> getAllUser() {
        List<User> users = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            User user = new User();
            user.setUsername("cxy35 >> " + i);
            user.setAddress("https://cxy35.com >> " + i);
            user.setId(i);
            user.setBirthday(new Date());
            users.add(user);
        }
        return users;
    }
}
```

启动项目，访问 http://120.0.0.1:8080/user 来验证，此时就可以正常返回 json 了，但发现 birthday 的日期格式展示不是很友好，可以通过覆盖 FastJson 的默认 Bean ，并增加日期格式化相关代码来解决（这里只增加处理日期格式化的功能，如果有其他需求，增加相应的代码即可）。

在 src/main/java 下相应的包中新建 WebMvcConfig 配置类，如下：

```java
@Configuration
public class WebMvcConfig {

    // 实现日期格式化：
    // 提供自定义的 FastJsonHttpMessageConverter
    @Bean
    FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        FastJsonConfig config = new FastJsonConfig();
        config.setDateFormat("yyyy-MM-dd");
        converter.setFastJsonConfig(config);
        return converter;
    }
}
```

另外还有 1 种方式可以实现日期格式化，通过在对应 pojo 的属性上加注解，如下：

```java
public class User {
    private Integer id;
    private String username;
    private String address;
    @JsonFormat(pattern = "yyyy-MM-dd")
    private Date birthday;

    // getter/setter
}
```

启动项目，访问 http://120.0.0.1:8080/user 来验证。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web/spring-boot-json](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web/spring-boot-json)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)