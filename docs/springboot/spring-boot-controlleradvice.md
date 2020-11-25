---
title: Spring Boot 使用 @ControllerAdvice
date: 2019-11-22 11:12:36
categories: Spring Boot
tags: [Spring Boot]
toc: true
---
学习在 Spring Boot 如何使用 `@ControllerAdvice` 注解。它其实是 Spring MVC 提供的功能，是一个增强的 Controller ，主要可以实现三个方面的功能：**全局异常处理、全局数据绑定、全局数据预处理**。
<!-- more -->

## 1 全局异常处理

```java
@ControllerAdvice
public class MyControllerAdvice {
    @ExceptionHandler(ArrayIndexOutOfBoundsException.class)
    public void globalException(ArrayIndexOutOfBoundsException e, HttpServletResponse resp) throws IOException {
        resp.setContentType("text/html;charset=utf-8");
        PrintWriter out = resp.getWriter();
        out.write("出错了：globalException");
        out.flush();
        out.close();
    }

    /*@ExceptionHandler(ArrayIndexOutOfBoundsException.class)
    public ModelAndView globalException2(ArrayIndexOutOfBoundsException e) throws IOException {
        ModelAndView mv = new ModelAndView("myerror");
        mv.addObject("error", "出错了：globalException2");
        return mv;
    }*/
}
```

1. 在类上添加 `@ControllerAdvice` 注解。
2. 在方法上添加 `@ExceptionHandler` 注解，用来指明异常的处理类型，比如这里指定为 ArrayIndexOutOfBoundsException ，则空指针异常就不会进到这个方法中来。
3. 在该类中，可以定义多个方法，不同的方法处理不同的异常，例如专门处理空指针的方法、专门处理数组越界的方法，或者也可以定义一个通用的方法处理所有的异常。

新建测试接口：

```java
@Controller
public class TestController {
    @ResponseBody
    @GetMapping("/globalException")
    public String globalException(Model model){
        String[] arr = {"a","b"};
        System.out.println(arr[2]);
        return "success";
    }
}
```

启动项目，访问 http://120.0.0.1:8080/globalException 来验证。

## 2 全局数据绑定

全局数据绑定功能可以用来做一些初始化的数据操作，我们可以将一些公共的数据定义在添加了 @ControllerAdvice 注解的类中，这样，在每一个 Controller 的接口中，就都能够访问导致这些数据。

```java
@ControllerAdvice
public class MyControllerAdvice {
    @ModelAttribute(value = "dataKey")
    public Map<String,Object> globalData() {
        Map<String, Object> map = new HashMap<>();
        map.put("name", "cxy35");
        map.put("address", "https://cxy35.com");
        return map;
    }
}
```

1. 在类上添加 `@ControllerAdvice` 注解。
2. 在方法上添加 `@ModelAttribute` 注解，标记该方法的返回数据是一个全局数据，默认情况下，这个全局数据的 key 就是返回的变量名，value 就是方法返回值，当然可以通过 @ModelAttribute 注解的 name 属性去重新指定 key。

新建测试接口：

```java
@Controller
public class TestController {
    @ResponseBody
    @GetMapping("/globalData")
    public String globalData(Model model) {
        String globalData = "";
        Map<String, Object> map = model.asMap();
        Set<String> keySet = map.keySet();
        for (String key : keySet) {
            globalData += ("【"+key + ":" + map.get(key)+"】");
        }
        return globalData;
    }
}
```

启动项目，访问 http://120.0.0.1:8080/globalData 来验证。

## 3 全局数据预处理

新建 2 个实体类， Book 和 Author ：

```java
public class Book {
    private String name;
    private Double price;

    // getter/setter
}
```

```java
public class Author {
    private String name;
    private Integer age;

    // getter/setter
}
```

新建测试接口：

```java
@Controller
public class TestController {
    @PostMapping("/initPreParam")
    public void initPreParam(Book book, Author author) {
        System.out.println(book);
        System.out.println(author);
    }
}
```

因为 2 个实体类都有一个 name 属性，从前端传递时，无法区分。可以通过 @ControllerAdvice 的全局数据预处理来解决这个问题。

```java
@ControllerAdvice
public class MyControllerAdvice {
    @InitBinder("a")
    public void initPreParamA(WebDataBinder binder) {
        binder.setFieldDefaultPrefix("a.");
    }

    @InitBinder("b")
    public void initPreParamB(WebDataBinder binder) {
        binder.setFieldDefaultPrefix("b.");
    }
}
```

1. 在类上添加 `@ControllerAdvice` 注解。
2. 在方法上添加 `@InitBinder` 注解，比如 @InitBinder("b") 注解表示该方法用来处理和 Book 相关的参数，统一给参数添加一个 b 前缀，即请求参数要有 b 前缀。

修改测试接口：

```java
@Controller
public class TestController {
    @PostMapping("/initPreParam")
    public void initPreParam(@ModelAttribute("b") Book book, @ModelAttribute("a") Author author) {
        System.out.println(book);
        System.out.println(author);
    }
}
```

启动项目，通过 Postman 的 POST 请求 http://120.0.0.1:8080/initPreParam?b.name=三国演义&b.price=88.8&a.name=罗贯中&a.age=99 来验证。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web/spring-boot-controlleradvice](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web/spring-boot-controlleradvice)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)