---
title: Spring Boot 项目创建
date: 2019-11-01 09:16:23
categories: Spring Boot
tags: [Spring Boot]
toc: true
---
3 种方式实现 Spring Boot 项目创建。
<!-- more -->

## 1 官方网站在线创建

官方网站：[https://start.spring.io/](https://start.spring.io/)，如下：

![](https://oscimg.oschina.net/oscnet/up-6f370b0a9e14e2112d0bdb1af9cf87ed782.png)

配置说明如下：

- Project：项目构建工具，这里选择 Maven ，Gradle 一般在 Android 中使用较多。
- Language：开发语言，这里选择 Java 。
- Spring Boot：版本，一般用最新稳定版。
- Project Metadata：Maven 工程相关信息，如项目坐标、项目描述等。Packing 表示项目打包方式，这里选择 jar 包，因为 Spring Boot 内嵌了 Servlet 容器，打成 jar 包即可直接运行，当然也可根据实际情况选择 war 包。Java 表示选择构建的 JDK 版本。
- Dependencies：选择所需要的依赖，这里先加入 web 依赖，可通过关键字搜索。

最后点击下面的按钮生成并导出后，用 IntelliJ IDEA 或者 Eclipse 打开继续开发。

## 2 IDE 创建

这里演示使用 IntelliJ IDEA 作为 IDE 来创建。

File -> New Project -> Spring Initializr 。

可以看到这里实际上也是用了官方网站的地址来创建，只是 IntelliJ IDEA 把里面的东西集成进来了。

![](https://oscimg.oschina.net/oscnet/up-545be15a0deb538585002702c2ef6195e8b.png)

填写 Maven 工程相关信息：

![](https://oscimg.oschina.net/oscnet/up-545be15a0deb538585002702c2ef6195e8b.png)

选择依赖：

![](https://oscimg.oschina.net/oscnet/up-1ecbd1f416c18b57a5a0561044700baa7d7.png)

填写项目相关信息：

![](https://oscimg.oschina.net/oscnet/up-fbe4918c67e4d4e10c28a61613857ef80e5.png)

## 3 Maven 创建

File -> New Project -> Maven 。

选择项目骨架（我这里不选择，有兴趣的可以试下里面的 Spring Boot 相关项目骨架）：

![](https://oscimg.oschina.net/oscnet/up-295b7dcaf127120d53ac40a77fa5d90fa92.png)

填写 Maven 工程相关信息：

![](https://oscimg.oschina.net/oscnet/up-48a4e2e0780feb7a78f4361a076768214d7.png)

填写项目相关信息：

![](https://oscimg.oschina.net/oscnet/up-fbe4918c67e4d4e10c28a61613857ef80e5.png)

创建完成后，打开 pom.xml 添加依赖：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.4.RELEASE</version>
</parent>

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

接着在 `com.cxy35.sample.springboot.helloworld` 包中创建启动类 `SpringBootHelloworldApplication.java` ：

```java
@SpringBootApplication
public class SpringBootHelloworldApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootHelloworldApplication.class, args);
    }

}
```

## 4 启动项目测试

在 `com.cxy35.sample.springboot.helloworld.controller` 包中创建测试类 `HelloController.java` ：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello spring boot!";
    }
}
```

运行 `SpringBootHelloworldApplication.java` 中的 main 方法启动项目，浏览器访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) 测试。

最终的项目结构如下：

![](https://oscimg.oschina.net/oscnet/up-cbb2c4870227aa2d08aa1b2b97e273e8cc7.png)

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-helloworld](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-helloworld)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)