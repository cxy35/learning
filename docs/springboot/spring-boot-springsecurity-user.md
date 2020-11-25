---
title: Spring Boot 整合 Spring Security（配置用户/角色-基于内存）
date: 2020-01-09 15:54:45
categories: Spring Boot
tags: [Spring Boot, Spring Security]
toc: true
---
Spring Boot 整合 Spring Security 之后，默认用户名为 user ，密码在项目启动时打印在控制台。这个随机生成的密码，每次项目启动时都会变，不是很方便。我们可以自己配置 Spring Security 的用户和角色，有三种方式可以实现：

- 通过 `application.properties` 配置文件配置**在内存中**。
- 通过 Java 代码配置**在内存中**。
- 配置**在数据库中**，然后通过 Java 代码从数据库中加载。

本文学习前面两种**在内存中**的配置方式，**在数据库中**的配置方式可以参考 [Spring Boot 整合 Spring Security（配置用户/角色-基于数据库）]() 。
<!-- more -->

## 1 创建工程

创建 Spring Boot 项目 `spring-boot-springsecurity-user` ，添加 `Web/Spring Security` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-1d23871e606c43a843a073470a40bc2081d.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
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
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 2 配置用户和角色

1. 通过 `application.properties` 配置文件。

```properties
# 方法1：通过配置文件配置用户/角色
# spring.security.user.name=admin
# spring.security.user.password=123456
# spring.security.user.roles=admin
```

2. 通过 Java 代码。

新增 `SecurityConfig` 配置类，如下：

```java
// 方法2：通过 SecurityConfig 配置用户和角色
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        // return NoOpPasswordEncoder.getInstance();// 密码不加密
        return new BCryptPasswordEncoder();// 密码加密
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 在内存中配置2个用户
        /*auth.inMemoryAuthentication()
                .withUser("admin").password("123456").roles("admin")
                .and()
                .withUser("user").password("123456").roles("user");// 密码不加密*/

        auth.inMemoryAuthentication()
                .withUser("admin").password("$2a$10$fB2UU8iJmXsjpdk6T6hGMup8uNcJnOGwo2.QGR.e3qjIsdPYaS4LO").roles("admin")
                .and()
                .withUser("user").password("$2a$10$3TQ2HO/Xz1bVHw5nlfYTBON2TDJsQ0FMDwAS81uh7D.i9ax5DR46q").roles("user");// 密码加密
    }
}
```

配置类说明：

- 在 configure 方法中配置了两个用户，密码是加密之后的字符串（明文为 123456）。
- 从 Spring 5 开始，强制要求密码要加密。如果坚持不想加密，可以使用一个过期的 `PasswordEncoder` 的实例 `NoOpPasswordEncoder` ，但不建议。
- Spring Security 中提供了 `BCryptPasswordEncoder` 密码编码工具，可以非常方便的实现密码的加密加盐，相同明文加密出来的结果总是不同，这样就不需要用户去额外保存 `盐` 的字段了，这一点比 Shiro 要方便很多。

```java
@SpringBootTest
class SpringBootSpringsecurityUserApplicationTests {
    @Test
    void contextLoads() {
        // 同样的明文加密后不重复
        for (int i = 0; i < 10; i++) {
            BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
            System.out.println(encoder.encode("123456"));
        }
    }
}
```

## 3 测试

新增测试类 `HelloController` ，如下：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

项目启动之后，浏览器访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) ，跳转到登录页面，用上述配置的用户名和密码就能登录了。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-springsecurity-user](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-springsecurity-user)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)