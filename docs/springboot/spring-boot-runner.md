---
title: Spring Boot 配置系统启动任务
date: 2019-11-27 19:11:18
categories: Spring Boot
tags: [Spring Boot]
toc: true
---
学习如何在 Spring Boot 中配置系统启动任务。
<!-- more -->

先来回顾下在普通的 web 项目中如何在项目启动的时做一些初始化操作，一般会自己定义一个 Listener 实现 ServletContextListener 接口，这样就能监听到项目的启动和销毁，并做相应的数据初始化和销毁操作，如下：

```java
public class MyServletContextListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        // 在这里做数据初始化操作
    }
    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        // 在这里做数据备份操作
    }
}
```

在 Spring Boot 中，对系统启动任务有 2 种解决方案，分别是 CommandLineRunner 和 ApplicationRunner 。

## 1 CommandLineRunner

新建 MyCommandLineRunner1 和 MyCommandLineRunner2 配置类，分别如下：

```java
@Component
// 数字越小，优先级越大，默认情况下，优先级的值为 Integer.MAX_VALUE，表示优先级最低
@Order(99)
public class MyCommandLineRunner1 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("MyCommandLineRunner1>>>"+Arrays.toString(args));
    }
}
```

```java
@Component
// 数字越小，优先级越大，默认情况下，优先级的值为 Integer.MAX_VALUE，表示优先级最低
@Order(100)
public class MyCommandLineRunner2 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("MyCommandLineRunner2>>>"+Arrays.toString(args));
    }
}
```

此时启动项目， run 方法就会被自动执行。参数传递有 2 种方式，无需指定 key ，直接写 value ：

1. 通过 IDEA 配置参数，如下：

![](https://oscimg.oschina.net/oscnet/up-a87f262969cdd81b0fa942559dd670aad08.png)

2. 项目打包后通过命令行启动时传入参数，如下：`java -jar spring-boot-commandlinerunner-0.0.1-SNAPSHOT.jar 三国演义 西游记`

## 2 ApplicationRunner

ApplicationRunner 与 CommandLineRunner 类似，区别是它支持更多形式的参数，如 key/value 。

新建 MyApplicationRunner1 和 MyApplicationRunner2 配置类，分别如下：

```java
@Component
// 数字越小，优先级越大，默认情况下，优先级的值为 Integer.MAX_VALUE，表示优先级最低
@Order(99)
public class MyApplicationRunner1 implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        // 获取命令行中的所有参数
        String[] sourceArgs = args.getSourceArgs();
        System.out.println("sourceArgs:" + Arrays.toString(sourceArgs));

        // 获取命令行中的无 key 参数（和 CommandLineRunner 一样）
        List<String> nonOptionArgs = args.getNonOptionArgs();
        System.out.println("nonOptionArgs:" + nonOptionArgs);

        // 获取所有 key/value 形式的参数的 key
        Set<String> optionNames = args.getOptionNames();
        System.out.println("optionNames/optionValues>>>:");
        for (String optionName : optionNames) {
            // 根据 key 获取 key/value 形式的参数的 value
            System.out.println(optionName + ":" + args.getOptionValues(optionName));
        }

        System.out.println(">>>>>>>>>>>>>>> MyApplicationRunner1结束 >>>>>>>>>>>>>>>>");
    }
}
```

```java
@Component
// 数字越小，优先级越大，默认情况下，优先级的值为 Integer.MAX_VALUE，表示优先级最低
@Order(100)
public class MyApplicationRunner2 implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        // 获取命令行中的所有参数
        String[] sourceArgs = args.getSourceArgs();
        System.out.println("sourceArgs:" + Arrays.toString(sourceArgs));

        // 获取命令行中的无key参数（和 CommandLineRunne r一样）
        List<String> nonOptionArgs = args.getNonOptionArgs();
        System.out.println("nonOptionArgs:" + nonOptionArgs);

        // 获取所有 key/value 形式的参数的 key
        Set<String> optionNames = args.getOptionNames();
        System.out.println("optionNames/optionValues>>>:");
        for (String optionName : optionNames) {
            // 根据 key 获取 key/value 形式的参数的 value
            System.out.println(optionName + ":" + args.getOptionValues(optionName));
        }

        System.out.println(">>>>>>>>>>>>>>> MyApplicationRunner2结束 >>>>>>>>>>>>>>>>");
    }
}
```

ApplicationArguments 类型的参数说明：

1. args.getSourceArgs()：用来获取命令行中的所有参数。
2. args.getNonOptionArgs()：用来获取命令行中的无 key 参数（和 CommandLineRunner 一样）。
3. args.getOptionNames()：用来获取所有 key/value 形式的参数的 key 。
4. args.getOptionValues(key))：根据 key 获取 key/value 形式的参数的 value 。

此时启动项目， run 方法就会被自动执行。参数传递有 2 种方式：

1. 通过 IDEA 配置参数，如下：

![](https://oscimg.oschina.net/oscnet/up-a9e9d61cbb4cc4535258efa5357c8cd5d56.png)

2. 项目打包后通过命令行启动时传入参数，如下：`java -jar spring-boot-applicationrunner-0.0.1-SNAPSHOT.jar 三国演义 西游记 --age=99`

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web/spring-boot-runner](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-web/spring-boot-runner)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)