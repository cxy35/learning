---
title: Spring Boot 整合 DevTools（实现热部署）
date: 2020-04-06 10:35:06
categories: Spring Boot
tags: [Spring Boot, DevTools, LiveReload]
toc: true
---
Spring Boot 整合 DevTools ，实现类文件和静态资源文件的热部署，只需要添加 `spring-boot-devtools` 依赖就可以轻松实现。
<!-- more -->

## 1 自动编译配置

在 Eclipse 中文件修改后，保存就会自动编译，但在 IDEA 中没有显示的文件保存操作，因此默认情况下文件修改后不会自动编译，需要手动编译（快捷键： `Ctrl + F9` ），从而触发项目自动重启。当然我们可以通过配置来实现 IDEA 中文件修改后的自动编译（可能会比较耗电脑资源），如下：

开启自动编译：

![](https://oscimg.oschina.net/oscnet/up-fa66732cd6d18916fad6521db1c60c5cba5.png)

Registry 配置（快捷键： `Ctrl + Shift + Alt + /` ）：

![](https://oscimg.oschina.net/oscnet/up-24f26358269d4db1fae4c778113ad6baf89.png)

![](https://oscimg.oschina.net/oscnet/up-380f2b5b05d445d825a7455f180f4802c91.png)

注意：在 Maven 项目的子 Module 内自动编译和自动重启貌似无效？

## 2 创建项目

创建 Spring Boot 项目 `spring-boot-devtools` ，添加 `Web/DevTools` 依赖，最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
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

## 3 类文件热部署

DevTools 针对类文件的热部署，主要由两个不同的 classloader 实现： `base classloader` 和 `restart classloader` 。其中 base classloader 用来加载那些不会变化的类，例如各种第三方依赖。而 restart classloader 则用来加载那些会发生变化的类，例如你自己写的代码。 Spring Boot 中热部署的原理就是当类文件发生变化时， base classloader 不变，而 restart classloader 则会被废弃，被另一个新的 restart classloader 代替。在整个过程中，因为只重新加载了变化的类，所以启动速度要比整个项目启动要快。

新建 `HelloController` 测试类，如下：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String  hello() {
        return "Hello DevTools";
    }
}
```

项目启动之后，访问 [http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello) ，接着修改内容，稍等一会观察控制台会发现类文件自动编译，且项目自动重启了，刷新页面后发现修改的内容也生效了。如果没有自动重启，可以尝试按一下 `Ctrl + S` 或切换一下视图离开 IDEA ，应该就会自动重启了。

## 4 静态资源文件热部署

### 4.1 配置项目自动重启（技术上可行）

在使用 DevTools 时，**默认情况下当静态资源文件发生变化时，并不会触发项目自动重启**，当然我们可以通过配置来实现（实际上没必要），有两种方式，如下：

```properties
# 配置 static 目录下的文件变化后（前提要手动/自动编译），触发项目自动重启
# 方式1
spring.devtools.restart.exclude=classpath:/static/**
# 方式2
# spring.devtools.restart.additional-paths=src/main/resources/static
```

在 `src/main/resources/static` 目录下新增 `index.html` ，如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 style="color: #ff1a0e;">hello devtools!</h1>
</body>
</html>
```

项目启动之后，访问 [http://127.0.0.1:8080/index.html](http://127.0.0.1:8080/index.html) ，接着修改 index.html 的内容，稍等一会观察控制台会发现项目自动重启了，刷新页面后发现修改的内容也生效了。如果没有自动重启，可以尝试按一下 `Ctrl + S` 或切换一下视图离开 IDEA ，应该就会自动重启了。

---

> **虽然技术上我们可以通过配置解决这一问题，但是没有必要**。因为静态资源文件发生变化后不需要编译，按理说保存后刷新下就可以访问到了。那么如何才能实现静态资源文件发生变化时，不编译就能自动刷新呢？ `LiveReload` 可以帮助我们实现这一功能。

### 4.2 LiveReload 实现页面自动刷新（推荐）

DevTools 中默认嵌入了 LiveReload 服务器， **LiveReload 可以在静态资源文件发生变化时自动触发浏览器页面刷新**， LiveReload 支持 Chrome、Firefox 以及 Safari 。以 Chrome 为例，在 Chrome 应用商店搜索 LiveReload 并添加到 Chrome 中（也可以通过离线安装），添加成功后，在 Chrome 右上角有一个 LiveReload 图标。

首先，注释掉上面关于静态资源文件触发项目自动重启的配置，如下：

```properties
# 配置 static 目录下的文件变化后（前提要手动/自动编译），触发项目自动重启
# 方式1
# spring.devtools.restart.exclude=classpath:/static/**
# 方式2
# spring.devtools.restart.additional-paths=src/main/resources/static
```

项目启动之后，访问 [http://127.0.0.1:8080/index.html](http://127.0.0.1:8080/index.html) ，并在当前页面开启 LiveReload 。**注意： LiveReload 是和浏览器的选项卡绑定在一起的，在哪个选项卡中开启了 LiveReload，就在哪个选项卡中访问页面，这样才有效果**。

接着修改下 index.html 的内容，回到浏览器，不用做任何操作，就会**发现浏览器自动刷新了，页面已经更新了，整个过程中项目并没有重启**。

---

LiveReload 默认是开启的，我们可以通过配置禁用 LiveReload ，如下：

```properties
# 禁用 LiveReload
spring.devtools.livereload.enabled=false
```

## 5 其他配置

### 5.1 手动控制自动重启功能

默认情况下， DevTools 中的自动重启功能是由它自己控制的，每次文件变化都会触发。**实际上没必要这么频繁，一般我们是在完成一个功能后才需要重启**。下面通过配置来实现手动控制自动重启，有两种方式，如下：

- 方式1：项目配置文件

```properties
# 手动控制自动重启功能：项目文件变化，且 .trigger-file 文件变化，则项目自动重启
spring.devtools.restart.trigger-file=.trigger-file
```

- 方式2：全局配置

在当前用户目录下新建 `.spring-boot-devtools.properties` 文件，如下：

```properties
spring.devtools.restart.trigger-file=.trigger-file
```

---

最后在 `src/main/resources` 目录下新建 `.trigger-file` 文件，当每次完成一个功能后，手动修改下这个文件（内容随意），项目才会自动重启。

### 5.2 关闭自动重启功能

引入了 spring-boot-devtools 依赖后，项目的自动重启功能默认是启用的，我们可以通过配置来关闭该功能，有两种方式，如下：

- 方式1：配置文件

```properties
# 方式1：关闭自动重启功能
spring.devtools.restart.enabled=false
```

- 方式2：启动类

```java
@SpringBootApplication
public class SpringBootDevtoolsApplication {

    public static void main(String[] args) {
        // 方式2：关闭自动重启功能
        System.setProperty("spring.devtools.restart.enabled", "false");
        SpringApplication.run(SpringBootDevtoolsApplication.class, args);
    }
}
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-devtools](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-devtools)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)