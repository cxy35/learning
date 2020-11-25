---
title: Spring Boot 配置定时任务（@Scheduled / Quartz）
date: 2019-12-20 09:53:43
categories: Spring Boot
tags: [Spring Boot]
toc: true
---
学习在 Spring Boot 中如何配置定时任务。一般有两种方案，一种是使用 Spring 自带的定时任务处理器 `@Scheduled` 注解来实现（业务比较简单时），另一种是使用第三方框架 `Quartz` 来实现。
<!-- more -->

## 1 @Scheduled

创建 Spring Boot 项目 `spring-boot-scheduled` ，添加 `Web` 依赖，最终的依赖如下：

```xml
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

首先在项目启动类上增加 `@EnableScheduling` 注解，开启定时任务，如下：

```java
@SpringBootApplication
@EnableScheduling // 开启定时任务
public class SpringBootScheduledApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootScheduledApplication.class, args);
    }

}
```

---

最后新建 `MyScheduled` 类，配置定时任务，配置说明见代码注释，如下：

```java
@Component
public class MyScheduled {
    // @Scheduled 注解表示开启一个定时任务
    // fixedRate 表示任务执行之间的时间间隔（单位是毫秒），具体是指两次任务的开始时间间隔，即第二次任务开始时，第一次任务可能还没结束。
    @Scheduled(fixedRate = 2000)
    public void fixedRate() {
        System.out.println("fixedRate >>>" + new Date());
    }

    // fixedDelay 表示任务执行之间的时间间隔（单位是毫秒），具体是指本次任务结束到下次任务开始之间的时间间隔。
    @Scheduled(fixedDelay = 2000)
    public void fixedDelay() {
        System.out.println("fixedDelay >>>" + new Date());
    }

    // initialDelay 表示首次任务启动的延迟时间（单位是毫秒）。
    @Scheduled(initialDelay = 2000, fixedDelay = 2000)
    public void initialDelay() {
        System.out.println("initialDelay >>>" + new Date());
    }

    // cron 表达式，每隔 5 秒触发一次
    @Scheduled(cron = "0/5 * * * * ?")
    public void cron() {
        System.out.println("cron >>>" + new Date());
    }
}
```

启动项目，观察控制台来验证结果。

其中 cron 表达式的格式为： `[秒] [分] [小时] [日] [月] [周] [年]` ，具体取值如下：

|序号|说明|是否必填|允许填写的值|允许的通配符|
|:-|:-|:-|:-|:-|
|1|秒|是|0-59|- * /|
|2|分|是|0-59|- * /|
|3|时|是|0-23|- * /|
|4|日|是|1-31|- * ? / L W|
|5|月|是|1-12 or JAN-DEC|- * /|
|6|周|是|1-7 or SUN-SAT|- * ? / L #|
|7|年|否|1970-2099|- * /|

**注意：日期和周（星期）可能会起冲突，因此在配置时这两个得有一个是 `?` 。**

通配符的含义如下：

- `?` 表示不指定值，即不关心某个字段的取值时使用，如配置日期和周（星期）时这两个得有一个是 `?` 。
- `*` 表示所有值，如配置秒为 `*` 时表示每一秒都会触发。
- `,` 表示分开多个值，如配置周为 `MON,WED,FRI` 时表示周一三五都会触发。
- `-` 表示区间，如配置小时为 `8-10` 时表示 8,9,10 点都会触发。
- `/` 表示递增触发，如配置分为 `5/15` 时表示从5分开始，每增15分触发(5,20,35,50)。
- `#` 表示序号（每月的第几个周几），如配置周为 `6#3` 时表示在每月的第3个周6（非常适合母亲节和父亲节）。
- `周` 字段的配置，不区分大小写，即 MON 与 mon 相同。
- `L` 表示最后的意思，如配置日期时表示当月的最后一天（如果是二月还会自动判断是否是润年）, 配置周时表示星期六（相当于 "7" 或 "SAT" ，注意周日算是第一天）。如果在 "L" 前加上数字，则表示该数据的最后一个，如配置周为 `6L` 时表示本月最后一个星期五。
- `W` 表示离指定日期的最近工作日（周一至周五），如配置日期为 `15W` 时表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发，如果15号正好在工作日(周一至周五)，则就在该天触发。如果指定格式为 `1W` ，则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，"W"前只能设置具体的数字，不允许区间 "-" )。
- `L` 和 `W` 可以一组合使用。如果在日字段上设置 "LW" ，则表示在本月的最后一个工作日触发(一般指发工资 )。

## 2 Quartz

创建 Spring Boot 项目 `spring-boot-quartz` ，添加 `Web/Quartz` 依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-b143c1dc036bb61d27995ec53b16d113711.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-quartz</artifactId>
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
</dependencies>
```

首先在项目启动类上增加 `@EnableScheduling` 注解，开启定时任务，如下：

```java
@SpringBootApplication
@EnableScheduling // 开启定时任务
public class SpringBootQuartzApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootQuartzApplication.class, args);
    }

}
```

---

Quartz 主要有两个概念，一个是 `JobDetail` （要做的事情），另一个是 `Trigger` （触发器，即什么时候做），要定义 JobDetail ，需要先定义 `Job` ，Job 的定义有两种方式：

1. 新建 `MyJob1` ，如下：

```java
// Job 定义方式1：直接定义一个 Bean 并注册到 Spring 容器中（无法传参）。
@Component
public class MyJob1 {
    public void sayHello() {
        System.out.println("MyJob1>>>"+new Date());
    }
}
```

2. 新建 `MyJob2` ，如下：

```java
// Job 定义方式2：继承 QuartzJobBean 并实现默认的方法（支持传参）
public class MyJob2 extends QuartzJobBean {
    private String name;

    public void setName(String name) {
        this.name = name;
    }

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println("MyJob2 >>> " + name + ":" + new Date());
    }
}
```

---

Job 定义完成之后，就可以开始配置 `JobDetail` 和 `Trigger` 了，新建 `QuartzConfig` 配置类，如下：

```java
/**
 * 在 Quartz 配置类中，主要配置两个东西：1.JobDetail（要做的事情） 2.Trigger（什么时候做）
 * <p>
 * JobDetail 有两种不同的配置方式：
 * 1. MethodInvokingJobDetailFactoryBean
 * 2. JobDetailFactoryBean
 */
@Configuration
public class QuartzConfig {
    // JobDetail1
    // JobDetail 配置方式1：这里使用 MyJob1 测试，指定 bean 和方法，无法传参
    @Bean
    MethodInvokingJobDetailFactoryBean methodInvokingJobDetailFactoryBean() {
        MethodInvokingJobDetailFactoryBean bean = new MethodInvokingJobDetailFactoryBean();
        bean.setTargetBeanName("myJob1");
        bean.setTargetMethod("sayHello");
        return bean;
    }

    // JobDetail2
    // JobDetail 配置方式2：这里使用 MyJob2 测试，支持传参
    @Bean
    JobDetailFactoryBean jobDetailFactoryBean() {
        JobDetailFactoryBean bean = new JobDetailFactoryBean();
        JobDataMap map = new JobDataMap();
        map.put("name", "cxy35");
        bean.setJobDataMap(map);
        bean.setJobClass(MyJob2.class);
        return bean;
    }

    // Trigger1：这里使用 JobDetail1 测试
    @Bean
    SimpleTriggerFactoryBean simpleTriggerFactoryBean() {
        SimpleTriggerFactoryBean bean = new SimpleTriggerFactoryBean();
        bean.setStartTime(new Date());
        bean.setRepeatCount(5);
        bean.setJobDetail(methodInvokingJobDetailFactoryBean().getObject());
        bean.setRepeatInterval(3000);
        return bean;
    }

    // Trigger2：这里使用 JobDetail2 测试
    @Bean
    CronTriggerFactoryBean cronTriggerFactoryBean() {
        CronTriggerFactoryBean bean = new CronTriggerFactoryBean();
        bean.setCronExpression("0/10 * * * * ?");
        bean.setJobDetail(jobDetailFactoryBean().getObject());
        return bean;
    }

    // 注册 Trigger1 和 Trigger2
    @Bean
    SchedulerFactoryBean schedulerFactoryBean() {
        SchedulerFactoryBean bean = new SchedulerFactoryBean();
        bean.setTriggers(simpleTriggerFactoryBean().getObject(), cronTriggerFactoryBean().getObject());
        return bean;
    }
}
```

启动项目，观察控制台来验证结果。

配置类说明：

- JobDetail 的配置有两种方式：MethodInvokingJobDetailFactoryBean 和 JobDetailFactoryBean 。
- 使用 MethodInvokingJobDetailFactoryBean 可以配置目标 Bean 的名字和目标方法的名字，这种方式不支持传参。
- 使用 JobDetailFactoryBean 可以配置 JobDetail ，任务类继承自 QuartzJobBean ，这种方式支持传参，将参数封装在 JobDataMap 中进行传递。
- Trigger 是指触发器，Quartz 中定义了多个触发器，这里向大家展示其中两种的用法，SimpleTrigger 和 CronTrigger 。
- SimpleTrigger 有点类似于前面说的 @Scheduled 的基本用法。
- CronTrigger 则有点类似于 @Scheduled 中 cron 表达式的用法。

![](https://oscimg.oschina.net/oscnet/up-9d9cb6af7fe484351040973a883d238f2fc.png)

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码（@Scheduled）：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-task/spring-boot-scheduled](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-task/spring-boot-scheduled)
- 本文示例代码（Quartz）：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-task/spring-boot-quartz](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-task/spring-boot-quartz)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)