---
title: Spring Boot 配置静态资源
date: 2019-11-20 10:51:04
categories: Spring Boot
tags: [Spring Boot, 静态资源]
toc: true
---
学习 Spring Boot 配置静态资源。
<!-- more -->

## 1 Spring MVC 配置静态资源

先来回顾下在 Spring MVC 中如何配置静态资源。使用 Spring MVC 时，静态资源会被拦截，需要添加额外的配置，一般在 `spring-mvc.xml` 中配置，如下：

```xml
<mvc:resources mapping="/favicon.ico" location="favicon.ico" />
<mvc:resources mapping="/static/**" location="/static/" />
```

## 2 Spring Boot 配置静态资源

### 2.1 默认位置

Spring Boot 项目中的静态资源最常见的位置在 `src/main/resources/static` 目录下，其实共有 5 个默认位置能放，重复的资源以优先级高的为准。如下（优先级: 1 > 2 > 3 > 4 > 5 ）：

1. classpath:/META-INF/resources/
2. classpath:/resources/
3. classpath:/static/
4. classpath:/public/
5. /

其中，/ 表示**类似** webapp 目录，即 webapp 中的静态文件也可以直接访问。

如果在 `src/main/resources/static` 目录下有一个 1.png 的文件，那么访问路径是 `http://127.0.0.1:8080/1.png` ，不需要加 static 。类似 Spring MVC 中的配置 `<mvc:resources mapping="/**" location="/static/"/>` ，实际上系统会去 /static/1.png 目录下查找相关的文件。

### 2.2 源码解读

打开 `org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration` ，找到了静态资源拦截的配置，如下：

```java
String staticPathPattern = this.mvcProperties.getStaticPathPattern();
if (!registry.hasMappingForPattern(staticPathPattern)) {
    this.customizeResourceHandlerRegistration(
        registry.addResourceHandler(new String[]{staticPathPattern})
            .addResourceLocations(
                WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())
            )
            .setCachePeriod(this.getSeconds(cachePeriod))
            .setCacheControl(cacheControl));
}
```

1. `this.mvcProperties.getStaticPathPattern()` 返回 "/**" 。
2. `this.resourceProperties.getStaticLocations()` 返回 4 个位置 "classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/" ，然后 `WebMvcAutoConfiguration.getResourceLocations` 又添加了 "/" ，这样总共就是上述 5 个位置。

### 2.3 自定义位置

上述 5 个是系统默认的位置，有 2 种办法可以实现自定义位置。

1. 通过 `application.properties` 配置文件。

```xml
# 自定义配置静态资源的匹配规则和路径
# 定义请求 URL 规则
# spring.mvc.static-path-pattern=/**
# 定义资源位置
# spring.resources.static-locations=classpath:/cxy35/
```

2. 通过 Java 代码。

新增配置类 `WebMvcConfig`， 如下：

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    // 自定义配置静态资源的匹配规则和路径
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("classpath:/cxy35/");
    }
}
```

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web/spring-boot-staticresources](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web/spring-boot-staticresources)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)