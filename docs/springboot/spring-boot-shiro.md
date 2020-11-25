---
title: Spring Boot 整合 Shiro
date: 2020-01-05 09:39:59
categories: Spring Boot
tags: [Spring Boot, Shiro]
toc: true
---
学习在 Spring Boot 中用两种方式整合 `Shiro` 。虽然在 Spring Boot 中的安全管理框架主流是使用 `Spring Security` ，但使用 Shiro 技术上也是可行的。
<!-- more -->

## 1 概述

Spring Security 和 Shiro 的比较如下：

- Spring Security 是一个重量级的安全管理框架； Shiro 则是一个轻量级的安全管理框架。
- Spring Security 概念复杂，配置繁琐； Shiro 概念简单、配置简单。
- Spring Security 功能强大； Shiro 功能简单，但在一般的 SSM/SSH 项目中也够用了。
- Spring Security 一般与 `Spring Boot/Spring Cloud` 项目组合使用； Shiro 一般与 `SSM/SSH` 项目结合使用。

虽然在 Spring Boot 项目中一般使用 Spring Security ，但也可以使用 Shiro ，有两种方式整合：

1. **原生整合**：将 SSM/SSH 项目中整合 Shiro 的配置用 Java 重写一遍。
2. **Shiro Starter 整合**：使用 Shiro 官方提供的 Starter 来配置。

## 2 原生整合

### 2.1 创建工程

创建 Spring Boot 项目 `spring-boot-shirojava` ，添加 `Web` 依赖。

创建成功之后手动在 pom 文件中添加 `Shiro` 相关的依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-web</artifactId>
        <version>1.4.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.4.0</version>
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

### 2.2 自定义核心组件 Realm

新增 `MyRealm` ，如下：

```java
public class MyRealm extends AuthorizingRealm {
    // 授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        return null;
    }

    // 认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String username = (String) token.getPrincipal();
        if ("cxy35".equals(username)) {
            return new SimpleAuthenticationInfo(username, "123456", getName());
        }
        return null;
    }
}
```

这里只在 Realm 中实现简单的认证操作（用户名和密码为 cxy35/123456 就能登录成功），不做授权，授权的具体写法和 SSM 中的 Shiro 一样，这里不再赘述。

### 2.3 配置 Shiro

新建 `ShiroConfig` 配置类，如下：

```java
@Configuration
public class ShiroConfig {
    @Bean
    MyRealm myRealm() {
        return new MyRealm();
    }

    @Bean
    SecurityManager securityManager() {
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        manager.setRealm(myRealm());
        return manager;
    }

    @Bean
    ShiroFilterFactoryBean shiroFilterFactoryBean() {
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        bean.setSecurityManager(securityManager());
        bean.setLoginUrl("/login");// 指定登录页面
        bean.setSuccessUrl("/index");// 指定登录成功的跳转页面
        bean.setUnauthorizedUrl("/unauthorized");// 指定访问未获授权的页面时，默认的跳转路径
        // 配置路径拦截规则（注意顺序）
        Map<String, String> map = new LinkedHashMap<>();
        map.put("/doLogin", "anon");
        map.put("/**", "authc");
        bean.setFilterChainDefinitionMap(map);
        return bean;
    }
}
```

配置类说明：

- 配置一个 `Realm` 实例。
- 配置一个 `SecurityManager` ，指定 Realm 。
- 配置一个 `ShiroFilterFactoryBean` ，指定 SecurityManager 、登录页面、路径拦截规则等。

## 3 Shiro Starter 整合

### 3.1 创建工程

创建 Spring Boot 项目 `spring-boot-shirostarter` ，添加 `Web` 依赖。

创建成功之后手动在 pom 文件中添加 `shiro-spring-boot-web-starter` 依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring-boot-web-starter</artifactId>
        <version>1.4.0</version>
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

### 3.2 自定义核心组件 Realm

新增 `MyRealm` ，如下：

```java
public class MyRealm extends AuthorizingRealm {
    // 授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        return null;
    }

    // 认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String username = (String) token.getPrincipal();
        if ("cxy35".equals(username)) {
            return new SimpleAuthenticationInfo(username, "123456", getName());
        }
        return null;
    }
}
```

这里只在 Realm 中实现简单的认证操作（用户名和密码为 cxy35/123456 就能登录成功），不做授权，授权的具体写法和 SSM 中的 Shiro 一样，这里不再赘述。

### 3.3 配置 Shiro

在 `application.properties` 配置文件中配置 Shiro 的基本信息，如下：

```properties
# 是否允许将 sessionId 放到 cookie 中
shiro.sessionManager.sessionIdCookieEnabled=true
# 是否允许将 sessionId 放到 Url 地址拦中
shiro.sessionManager.sessionIdUrlRewritingEnabled=true
# 开启 shiro
shiro.enabled=true
# 开启 shiro web
shiro.web.enabled=true
# 指定登录页面
shiro.loginUrl=/login
# 指定登录成功的跳转页面
shiro.successUrl=/index
# 指定访问未获授权的页面时，默认的跳转路径
shiro.unauthorizedUrl=/unauthorized
```

---

新建 `ShiroConfig` 配置类，如下：

```java
@Configuration
public class ShiroConfig {
    /*@Bean
    Realm realm() {
        TextConfigurationRealm realm = new TextConfigurationRealm();
        realm.setUserDefinitions("cxy35=123456,user \n admin=123456,admin");
        realm.setRoleDefinitions("user=read \n admin=read,write");
        return realm;
    }*/

    @Bean
    MyRealm myRealm() {
        return new MyRealm();
    }

    @Bean
    DefaultWebSecurityManager securityManager() {
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        manager.setRealm(myRealm());
        return manager;
    }

    @Bean
    ShiroFilterChainDefinition shiroFilterChainDefinition() {
        // 配置路径拦截规则
        DefaultShiroFilterChainDefinition definition = new DefaultShiroFilterChainDefinition();
        definition.addPathDefinition("/doLogin", "anon");
        definition.addPathDefinition("/**", "authc");
        return definition;
    }
}
```

配置类说明：

- 配置一个 `Realm` 实例。
- 配置一个 `SecurityManager` ，指定 Realm 。
- 配置一个 `ShiroFilterChainDefinition` ，指定路径拦截规则等。

## 4 测试

新增 `HelloController` 测试类，如下：

```java
@RestController
public class HelloController {
    @GetMapping("/login")
    public String login() {
        return "please login";
    }

    @PostMapping("/doLogin")
    public void doLogin(String username, String password) {
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(new UsernamePasswordToken(username, password));
            System.out.println("登录成功!");
        } catch (AuthenticationException e) {
            e.printStackTrace();
            System.out.println("登录失败!" + e.getMessage());
        }
    }

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

启动项目，用 HTTP 请求工具来测试（如 `Postman` ）。

首先访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) ，因为未登录过，所以会跳到 `http://127.0.0.1:8080/login` 接口要求登录，如下：

![](https://oscimg.oschina.net/oscnet/up-007b3b71d486381778bbc22f4777fb811e7.png)

接着调用 `http://127.0.0.1:8080/doLogin` 接口完成登录，如下：

![](https://oscimg.oschina.net/oscnet/up-b5857a9e5fc15717e11d99df6479d2c87c1.png)

最后再次访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) 就可以正常访问了，如下：

![](https://oscimg.oschina.net/oscnet/up-7c253c19b5a76a1d33cc044715c71252b64.png)

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码（原生）：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-shirojava](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-shirojava)
- 本文示例代码（Shiro Starter）：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-shirostarter](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-security/spring-boot-shirostarter)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)