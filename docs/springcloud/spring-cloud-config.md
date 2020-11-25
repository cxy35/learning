---
title: Spring Cloud Config 分布式配置中心
date: 2020-05-11 18:54:54
categories: Spring Cloud
tags: [Spring Cloud, Config]
toc: true
---
学习在 Spring Cloud 中使用 Config 实现分布式配置中心，包括基本使用、配置文件位置、配置文件加解密、Config Server 安全管理、结合注册中心、配置文件动态刷新、请求失败重试等功能。
<!-- more -->

## 1 概述

常见的分布式配置中心的解决方案有：`Spring Cloud Conﬁg、QConf（360）、diamond（淘宝）、disconf（百度）、Apache Commons Conﬁguration、owner、cfg4j` 等。

`Spring Cloud Conﬁg` 是一个分布式系统配置管理的解决方案，它包含了 `Server` 和 `Client` 。配置文件放在 Server 端，通过接口的形式提供给 Client ，主要功能如下：

- 集中管理各个环境、各个微服务的配置文件。
- 提供服务端和客户端支持。
- 配置文件修改后，可以快速生效。
- 配置文件通过 Git/SVN 进行管理，天然支持版本回退功能。
- 支持高并发查询、也支持多种开发语言。

## 2 基本使用

### 2.1 准备工作

首先，本地准备好开发/测试/生产环境对应的配置文件 `config-client-dev.properties/config-client-test.properties/config-client-prod.properties` ，内容f分别如下：

```properties
# dev
cxy35=dev-123456

# test
cxy35=test-123456

# prod
cxy35=prod-123456
```

最后提交到 GitHub 上 spring-cloud-config 仓库 [https://github.com/cxy35/spring-cloud-config](https://github.com/cxy35/spring-cloud-config) 下的 `config-client` 目录下。

### 2.2 Conﬁg Server

创建 Spring Boot 项目 `config-server` ，添加 `Web/Config Server` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-42ae801ad1ed08d4cf27e4d8e4c9733eb25.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
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
 
项目创建成功后，项目启动类上添加 `@EnableConfigServer` 注解，开启 Conﬁg Server 功能：

```java
@SpringBootApplication
@EnableConfigServer // 开启 Conﬁg Server 功能
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }

}
```

然后在 `application.properties` 配置文件中配置 Git 仓库的基本信息：

```properties
spring.application.name=config-server
server.port=8080

# 配置文件仓库地址
spring.cloud.config.server.git.uri=https://github.com/cxy35/spring-cloud-config
# 仓库中配置文件的目录
spring.cloud.config.server.git.search-paths=config-client
# 仓库的用户名密码
spring.cloud.config.server.git.username=799737179@qq.com
spring.cloud.config.server.git.password=
```

---

启动项目，通过 [http://127.0.0.1:8080/config-client/dev/master](http://127.0.0.1:8080/config-client/dev/master) 就可以访问到对应的配置文件了。访问地址有如下规则：

```bash
/{application}/{proﬁle}/[{label}]
/{application}-{proﬁle}.properties
/{label}/{application}-{proﬁle}.properties
```
 
- `applicaiton`：表示配置文件名，如：config-client
- `proﬁle`：表示配置文件 proﬁle ，如：dev/dev/prod
- `label`：表示 git 分支，此参数可选，默认为 master

接下来，可以修改配置文件，并且重新提交到 GitHub ，重新访问就可以看到最新的配置内容。

### 2.3 Conﬁg Client

创建 Spring Boot 项目 `config-client` ，添加 `Web/Config Client` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-80f79ac9ccbeb0bb85efdb7980135dbeb6b.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
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

项目创建成功后，在 resources 目录下，添加 `bootstrap.properties` 配置文件（比 application.properties 先加载），内容如下：

```properties
# 对应 config-server 中的 {application} 占位符
spring.application.name=config-client
server.port=8082

# 对应 config-server 中的 {profile} 占位符
spring.cloud.config.profile=dev
# 对应 config-server 中的 {label} 占位符
spring.cloud.config.label=master
# config-server 的地址
spring.cloud.config.uri=http://127.0.0.1:8080
```

---

接下来创建一个 HelloController 进行测试：

```java
@RestController
public class HelloController {
    @Value("${cxy35}")
    String cxy35;

    @GetMapping
    public String hello(){
        return cxy35;
    }
}
```

最后访问 [http://127.0.0.1:8082/hello](http://127.0.0.1:8082/hello) 可以访问到对应的配置文件。

## 3 配置文件位置

- `{application}` 占位符动态控制

在 conﬁg-server 中修改配置文件，使用 `{application}` 占位符动态控制配置文件的目录：

```properties
# 仓库中配置文件的目录，动态
# spring.cloud.config.server.git.search-paths={application}
```

这里的 `{application}` 占位符，表示连接上来的 config-client 中 `spring.application.name` 属性的值。同理， `{proﬁle}` 和 `{label}` 分别对应 config-client 中 `spring.cloud.conﬁg.proﬁle` 和 `spring.cloud.conﬁg.label` 属性的值。

- `classpath` 或本地磁盘查找（不常用）

在 conﬁg-server 中修改配置文件，添加如下配置：

```properties
# classpath 下查找，而不是去 Git 仓库中查找
# spring.profiles.active=native
# 本地磁盘下查找
# spring.cloud.config.server.native.search-locations=file:/E:/properties/
```

## 4 配置文件加解密

上述 Git 仓库中配置文件的内容都是明文，不安全，需要加密处理。

### 4.1 介绍

常见的加密方案有：**不可逆加密**和**可逆加密**。不可逆加密是指理论上无法根据加密后的密文推算出明文的一种加密方式，一般用在密码加密上，常见的算法有：`MD5 消息摘要算法`、`SHA 安全散列算法`。可逆加密是指可以根据加密后的密文推算出明文的一种加密方式，可逆加密一般又分为两种：

- 对称加密：加密的密钥和解密的密钥是一样的，常见算法有：`des`、`3des`、`aes` 。
- 非对称加密：加密的密钥和解密的密钥不一样，加密的叫做公钥，可以告诉任何人，解密的叫做私钥，只有自己知道，常见算法有： `RSA` 。

首先，下载不限长度的 JCE ：[https://www.oracle.com/java/technologies/javase-jce8-downloads.html](https://www.oracle.com/java/technologies/javase-jce8-downloads.html) 。将下载的文件解压，解压出来的 jar 拷贝到 Java 安装目录中，如：`D:\Java\jdk1.8.0_181\lib\security` 。

### 4.2 对称加密

在 conﬁg-server 中新增 `bootstrap.properties` 配置文件，添加如下内容配置密钥：

```properties
# 配置对称加密的密钥
encrypt.key=cxy35
```

---

重新启动 config-server ，访问 [http://127.0.0.1:8080/encrypt/status](http://127.0.0.1:8080/encrypt/status) ，查看密钥配置是否成功。

然后，访问 [http://127.0.0.1:8080/encrypt](http://127.0.0.1:8080/encrypt)（POST 请求），可以对一段明文进行加密，把加密后的密文保存到 Git 仓库下的对应的配置文件中，**注意要加一个 `{cipher}` 前缀**。

![](https://oscimg.oschina.net/oscnet/up-12ff92fc1cb4e25967f57a31767b7ee5ec7.png)

```properties
# cxy35=dev-123456
cxy35={cipher}fd4c57fa240a3ca16a43cd60c1365645c9ebdb60e765a20719bc2c282fa7fc5a
```

再次访问 [http://127.0.0.1:8080/config-client/dev/master](http://127.0.0.1:8080/config-client/dev/master)（config-server） 或 [http://127.0.0.1:8082/hello](http://127.0.0.1:8082/hello)（config-client）可以看到返回的内容是**明文**。

### 4.3 非对称加密

使用非对称加密，我们需要先生成一个密钥对，在命令行使用`密钥和证书管理工具命令 keytool` 生成：

```bash
keytool -genkeypair -alias config-server -keyalg RSA -keypass 111111 -keystore D:\config-server.jks -storepass 111111
```

![](https://oscimg.oschina.net/oscnet/up-402855a6eeb56805e3cbaac782bb37e9029.png)

命令执行完成后，拷贝生成的 config-server.jks 文件到 conﬁg-server 的 resources 目录下。然后在 conﬁg-server 的 `bootstrap.properties` 配置文件中，添加如下配置：

```properties
# 配置非对称加密
encrypt.key-store.location=classpath:config-server.jks
encrypt.key-store.alias=config-server
encrypt.key-store.password=111111
encrypt.key-store.secret=111111
```

注意，需要在 `pom.xml` 的 build 节点中，添加如下配置，防止 jks 文件被过滤掉。

```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <includes>
            <include>**/*.properties</include>
            <include>**/*.jks</include>
        </includes>
    </resource>
</resources>
```

---

重新启动 config-server ，访问 [http://127.0.0.1:8080/encrypt/status](http://127.0.0.1:8080/encrypt/status) ，查看密钥配置是否成功。

然后，访问 [http://127.0.0.1:8080/encrypt](http://127.0.0.1:8080/encrypt)（POST 请求），可以对一段明文进行加密，把加密后的密文保存到 Git 仓库下的对应的配置文件中，**注意要加一个 `{cipher}` 前缀**。

![](https://oscimg.oschina.net/oscnet/up-d1848d21b539329a89616f8395c60a8549c.png)

```properties
# cxy35=dev-123456
# cxy35={cipher}fd4c57fa240a3ca16a43cd60c1365645c9ebdb60e765a20719bc2c282fa7fc5a
cxy35={cipher}AQBPxgHh1+yoTT1q/xcd5S77xJVZduSnYSzb0keKbkckHjFIxYmVcbPpdfVyKRlB7JSp23xATAWNYsXS7wywfYIG2OeO/HIufqyM+t0pm/tqycn8/jjPMu447rvFVMLcF6TUgr2Xqo5Is1C73u/rGYyO3OuLXhq9J3Rp4DOn6CJnZdMNKjGl0fvLDFoujRk/avfag0aR4SKFPrTUabJIFVTKyMyLSS42N1X8QOlm4YXbIGQ9oCIJYpBTf/0R+HZ44lFnj7fMs5LpKBqqZvhd6mx+auXU2AzX6jQCkHmet+D74UHQG2jdH2UqC9MnH6O5sn6phwSApQElbTG+CY61wFPZDO7YFenXtWNeGWv7F3ZozIgPiM8oDiAaR9Ob+Tad6kc=
```

再次访问 [http://127.0.0.1:8080/config-client/dev/master](http://127.0.0.1:8080/config-client/dev/master)（config-server） 或 [http://127.0.0.1:8082/hello](http://127.0.0.1:8082/hello)（config-client）可以看到返回的内容是**明文**。

## 5 Config Server 安全管理

为了防止用户直接通过访问 conﬁg-server 看到配置文件内容，我们可以用 Spring Security 来保护 conﬁg-server 接口。

首先在 conﬁg-server 中添加 Spring Security 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

在 conﬁg-server 的 `bootstrap.properties` 配置文件中，添加如下配置，配置用户名密码：

```properties
# 配置 Spring Security 的用户和密码
spring.security.user.name=cxy35
spring.security.user.password=123456
```

这样 conﬁg-server 中的接口就被保护起来了，需要登录才能访问了。但需要让 conﬁg-client 知道，否则 conﬁg-client 无法获取配置文件。

在 conﬁg-client 的 `bootstrap.properties` 配置文件中，添加如下配置：

```properties
# 配置 conﬁg-server 中 Spring Security 的用户和密码
spring.cloud.config.username=cxy35
spring.cloud.config.password=123456
```

## 6 结合注册中心

前面的配置都是直接在 conﬁg-client 中写死 conﬁg-server 的地址，下面开始结合注册中心 Eureka。

首先，在 conﬁg-server 和 conﬁg-client 中添加 Eureka Client 依赖，让它们都能注册到 Eureka Server 上，如下：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

然后，修改 conﬁg-server 中的 application.properties 配置文件，配置注册信息：

```properties
# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

然后，修改 conﬁg-client 中的 bootstrap.properties 配置文件，配置注册信息，并配置 config-server 的服务名称，不再写死 config-server 的地址：

```properties
# 对应 config-server 中的 {application} 占位符
spring.application.name=config-client
server.port=8082

# 对应 config-server 中的 {profile} 占位符
spring.cloud.config.profile=dev
# 对应 config-server 中的 {label} 占位符
spring.cloud.config.label=master
# config-server 的地址
# spring.cloud.config.uri=http://127.0.0.1:8080
# 开启通过 eureka 获取 config-server 的功能
spring.cloud.config.discovery.enabled=true
# 配置 config-server 服务名称
spring.cloud.config.discovery.service-id=config-server

# 配置 conﬁg-server 中 Spring Security 的用户和密码
spring.cloud.config.username=cxy35
spring.cloud.config.password=123456

# 服务注册中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:1111/eureka
```

最后，启动 Eureka Server/config-server/config-client，再次访问 [http://127.0.0.1:8080/config-client/dev/master](http://127.0.0.1:8080/config-client/dev/master)（config-server） 或 [http://127.0.0.1:8082/hello](http://127.0.0.1:8082/hello)（config-client）也可以正常返回。

## 7 配置文件动态刷新

当配置文件发生变化之后，conﬁg-server 可以及时感知到变化，但是 conﬁg-client 不会及时感知到变化，默认情况下， conﬁg-client 只有重启才能加载到最新的配置文件。这里我们可以结合 actuator 中的 refresh 来实现，更好的方案是结合消息总线来实现。

首先，给 conﬁg-client 添加如下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

然后，修改 conﬁg-client 中的 bootstrap.properties 配置文件，使 refresh 端点暴露出来：

```properties
# 暴露 refresh 端点
management.endpoints.web.exposure.include=refresh
```

最后，再给 conﬁg-client 使用了配置文件的地方加上 `@RefreshScope` 注解，这样，当配置改变后，只需要调用 refresh 端点， conﬁg-client 中的配置就可以自动刷新。

```java
@RestController
@RefreshScope
public class HelloController {
    @Value("${cxy35}")
    String cxy35;

    @GetMapping("/hello")
    public String hello(){
        return cxy35;
    }
}
```
 
重启 config-client , 以后，如果配置文件发生变化，只要调用 [http://127.0.0.1:8082/actuator/refresh](http://127.0.0.1:8082/actuator/refresh) [POST]，配置文件就会自动刷新了。

## 8 请求失败重试

conﬁg-client 在调用 conﬁg-server 时，一样也可能发生请求失败的问题，这个时候，我们可以配置一个请求重试的功能。

首先，给 conﬁg-client 添加如下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

然后，修改 conﬁg-client 中的 bootstrap.properties 配置文件，开启失败快速响应（启动就报错，无需等使用时才报错）：

```properties
# 开启失败快速响应
spring.cloud.config.fail-fast=true
```

测试时可以注释掉配置文件的用户名和密码，重启 conﬁg-client ，观察控制台，此时加载配置文件失败，就会自动重试。

![](https://oscimg.oschina.net/oscnet/up-9434cfc2af86b9692a4af23bb6cf009e06b.png)

另外，也可以增加如下配置保证服务的可用性：

```properties
# 开启失败快速响应
spring.cloud.config.fail-fast=true
# 请求重试的初始间隔时间
spring.cloud.config.retry.initial-interval=1000
# 最大重试次数
spring.cloud.config.retry.max-attempts=6
# 重试时间间隔乘数
spring.cloud.config.retry.multiplier=1.1
# 最大间隔时间
spring.cloud.config.retry.max-interval=2000
```

---

- [Spring Cloud 教程合集](https://mp.weixin.qq.com/s/SBmcs2bxumhNz4kky1pl-A)（微信左下方**阅读全文**可直达）。
- Spring Cloud 教程合集示例代码：[https://github.com/cxy35/spring-cloud-samples](https://github.com/cxy35/spring-cloud-samples)
- 本文示例代码：[https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-config](https://github.com/cxy35/spring-cloud-samples/tree/master/spring-cloud-config)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)