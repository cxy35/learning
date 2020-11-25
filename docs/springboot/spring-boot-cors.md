---
title: Spring Boot 通过 CORS 解决跨域问题
date: 2019-11-25 19:08:33
categories: Spring Boot
tags: [Spring Boot, 跨域, CORS]
toc: true
---
学习在 Spring Boot 中通过 CORS 解决跨域问题。
<!-- more -->

## 1 介绍

先来了解下同源策略，它是由 Netscape 提出的一个著名的安全策略，是浏览器最核心，也最基本的安全功能，现在所有支持 JavaScript 的浏览器都会使用这个策略，**同源是指协议、域名以及端口要相同**。传统的跨域解决方案是 JSONP ， JSONP 虽然能解决跨域但是有一个很大的局限性，那就是只支持 GET 请求，不支持其他类型的请求。而 CORS （ Cross-origin resource sharing 跨域源资源共享）是一个 W3C 标准，它是一份浏览器技术的规范，提供了 Web 服务从不同网域传来沙盒脚本的方法，以避开浏览器的同源策略。

## 2 实战

新建 Spring Boot 工程 spring-boot-corsprovider ，用来提供服务，默认 8080 端口。新增测试类 HelloController ，如下：

```java
@RestController
public class HelloController {
    @GetMapping("/get")
    public String get() {
        return "get";
    }

    @PutMapping("/put")
    public String put() {
        return "put";
    }
}
```

新建 Spring Boot 工程 spring-boot-corsconsumer ，用来消费服务，配置 8081 端口。在 resources/static 下新建测试页面 index.html ，如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="jquery3.3.1.js"></script>
</head>
<body>
<div id="app"></div>
<input type="button" value="GET" onclick="getData()">
<input type="button" value="PUT" onclick="putData()">
<script>
    function getData() {
        $.get('http://127.0.0.1:8080/get', function (msg) {
            $("#app").html(msg);
        });
    }

    function putData() {
        $.ajax({
            type: 'put',
            url: 'http://127.0.0.1:8080/put',
            success: function (msg) {
                $("#app").html(msg);
            }
        })
    }
</script>
</body>
</html>
```

启动项目，访问 http://120.0.0.1:8081/index.html ，点击按钮后观察浏览器控制台，报跨域错误，如下：

```
Access to XMLHttpRequest at 'http://127.0.0.1:8080/get' from origin 'http://127.0.0.1:8081' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

接着修改 HelloController ，在类或方法上增加 CORS 的配置，如下：

```java
@RestController
// @CrossOrigin(origins = "http://127.0.0.1:8081")
public class HelloController {
    @GetMapping("/get")
    @CrossOrigin(origins = "http://127.0.0.1:8081")
    public String get() {
        return "get";
    }

    @PutMapping("/put")
    @CrossOrigin(origins = "http://127.0.0.1:8081")
    public String put() {
        return "put";
    }
}
```

重启 spring-boot-corsprovider 再次测试，可以正常获取到数据。观察对应请求，发现 Response Headers 中多了 `Access-Control-Allow-Origin: http://127.0.0.1:8081` 表示服务端愿意接收来自 http://127.0.0.1:8081 的请求，这样浏览器就不会再去限制这个请求了。

上述CORS 的配置实在类或方法上，此外在 Spring Boot 中也支持全局配置，增加 `MyWebMvcConfigurer` 配置类，重写 addCorsMappings 方法，如下：

```java
@Configuration
public class MyWebMvcConfigurer implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://127.0.0.1:8081")
                .allowedHeaders("*")
                .allowedMethods("*")
                .maxAge(30 * 1000);
    }
}
```

## 3 风险

跨域问题虽然解决了，但带来了潜在的威胁，如：CSRF（Cross-site request forgery）跨站请求伪造。跨站请求伪造也被称为 one-click attack 或者 session riding ，通常缩写为 CSRF 或者 XSRF ，是一种挟制用户在当前已登录的 Web 应用程序上执行非本意操作的攻击方法，如：

> 假如一家银行用以运行转账操作的URL地址如下：`http://icbc.com/aa?bb=cc` ，那么，一个恶意攻击者可以在另一个网站上放置如下代码：`<img src="http://icbc.com/aa?bb=cc">`，如果用户访问了恶意站点，而她之前刚访问过银行不久，登录信息尚未过期，那么她就会遭受损失。

基于此，浏览器在实际操作中，会对请求进行分类，分为简单请求，预先请求，带凭证的请求等，预先请求会首先发送一个 options 探测请求，和浏览器进行协商是否接受请求。默认情况下跨域请求是不需要凭证的，但是服务端可以配置要求客户端提供凭证，这样就可以有效避免 csrf 攻击。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web/spring-boot-cors](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web/spring-boot-cors)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)