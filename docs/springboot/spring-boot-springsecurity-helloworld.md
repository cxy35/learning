---
title: Spring Boot 整合 Spring Security（初次体验）
date: 2020-01-07 15:17:46
categories: Spring Boot
tags: [Spring Boot, Spring Security]
toc: true
---
通过本文体验 Spring Boot 整合 `Spring Security` 。 Spring Security 是 Spring 家族中的一个安全管理框架，但在 Spring Boot 出现之前，使用的没有 Shiro 多，因为在 SSM/SSH 项目中整合 Spring Security 比较麻烦，直到 Spring Boot 的出现。目前关于安全管理框架的整合模式一般有两种，一种是 `SSM/SSH + Shiro` ，另一种是 `Spring Boot/Spring Cloud + Spring Security` ，但并非绝对。
<!-- more -->

## 1 创建工程

创建 Spring Boot 项目 `spring-boot-springsecurity-helloworld` ，添加 `Web/Spring Security` 依赖，如下：

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

## 2 测试

新增测试类 `HelloController` ，如下：

```java
@RestController
public class HelloController {
    // 默认用户名为 user ，密码在项目启动时打印在控制台
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

启动项目之后，浏览器访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) ，会跳转到登录页面（**默认用户名为 user ，密码在项目启动时打印在控制台**），这是因为加入 `Spring Security` 依赖之后，接口就被自动保护起来了。

控制台信息：`Using generated security password: 3629761f-b103-4392-8318-c7a8d43ed80d`

![](https://oscimg.oschina.net/oscnet/up-bd58156ab4d447fc54ef6f5c96f577a438e.png)

当用户从浏览器发送请求访问 `/hello` 接口时，服务端会返回 `302` 响应码，让客户端重定向到 `/login` 页面，用户在 `/login` 页面登录，登陆成功之后，就会自动跳转到 `/hello` 接口。

当然也可以用 HTTP 请求工具来测试（如 `Postman` ），将用户信息放在请求头中，这样可以避免重定向到登录页面。

![](https://oscimg.oschina.net/oscnet/up-8b7a034114dcb6ddb3863e545dfde6bf891.png)

通过以上两种不同的登录方式可以看出 Spring Security 支持两种不同的认证方式：

- 通过 `form` 表单来认证。
- 通过 `HttpBasic` 来认证。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-springsecurity-helloworld](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-springsecurity-helloworld)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)