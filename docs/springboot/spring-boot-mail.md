---
title: Spring Boot 整合邮件发送
date: 2020-01-31 11:08:00
categories: Spring Boot
tags: [Spring Boot]
toc: true
---
学习在 Spring Boot 中发送邮件，使用对应的自动化配置类，实现非常方便。
<!-- more -->

## 1 邮件概述

常用的邮件协议有 `SMTP、POP3、IMAP` 。现在假设从 aaa@qq.com 发送邮件到 111@163.com ，邮件投递过程如下：

1. aaa@qq.com 先将邮件投递到腾讯的邮件服务器。
2. 腾讯的邮件服务器将我们的邮件投递到网易的邮件服务器。
3. 111@163.com 登录网易的邮件服务器查看邮件。

`SMTP` 协议全称为 Simple Mail Transfer Protocol，译作简单邮件传输协议，它定义了邮件客户端软件与 SMTP 服务器之间，以及 SMTP 服务器与 SMTP 服务器之间的通信规则。SMTP 是一个基于 TCP/IP 的应用层协议，江湖地位有点类似于 HTTP ， SMTP 服务器默认监听的端口号为 25 。 aaa@qq.com 用户先将邮件投递到腾讯的 SMTP 服务器这个过程就使用了 SMTP 协议，然后腾讯的 SMTP 服务器将邮件投递到网易的 SMTP 服务器这个过程也依然使用了 SMTP 协议，SMTP 服务器就是用来收邮件。

`POP3` 协议全称为 Post Office Protocol ，译作邮局协议，它定义了邮件客户端与 POP3 服务器之间的通信规则，那么该协议在什么场景下会用到呢？当邮件到达网易的 SMTP 服务器之后， 111@163.com 用户需要登录服务器查看邮件，这个时候就该协议就用上了。邮件服务商都会为每一个用户提供专门的邮件存储空间，SMTP 服务器收到邮件之后，就将邮件保存到相应用户的邮件存储空间中，如果用户要读取邮件，就需要通过邮件服务商的 POP3 邮件服务器来完成。

`IMAP` 协议是对 POP3 协议的扩展，功能更强，作用类似，这里不再赘述。

## 2 准备工作

目前国内大部分的邮件服务商都不允许直接使用用户名和密码的方式在代码中发送邮件，都是要先申请**授权码**，这里以 QQ 邮箱为例，演示授权码的申请流程。

首先登录 QQ 邮箱网页版，点击左上方的设置按钮：

![](https://oscimg.oschina.net/oscnet/up-14e112064b775102b5d70bb0d92c1b1e2ff.png)

然后点击账户选项卡：

![](https://oscimg.oschina.net/oscnet/up-cf836163e60c882b3364075d7747699221b.png)

在账户选项卡中找到开启 POP3/SMTP 选项，如下：

![](https://oscimg.oschina.net/oscnet/up-cc14fdf2b0eaf478c5d10ef3cf0de940a18.png)

点击开启，开启相关功能，开启过程需要手机号码验证，开启成功之后，即可获取一个授权码，保存好，下面会用到。

## 3 实战

### 3.1 创建工程并配置

创建 Spring Boot 项目 `spring-boot-mail` ，添加 `Web/Mail` 依赖，另外后面会演示使用 `Thymeleaf` 和 `Freemarker` 模板发送邮件，所以额外增加了这两个依赖，如下：

![](https://oscimg.oschina.net/oscnet/up-c11c4e3fa0715eead43ec113ebaa2b46063.png)

最终的依赖如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
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

接着在 `application.properties` 配置文件中添加邮件相关信息的配置，如下：

```properties
# 邮件服务配置
# SMTP 服务器地址
spring.mail.host=smtp.qq.com
# SMTP 服务器的端口
spring.mail.port=587
# 邮箱用户名
spring.mail.username=799737179@qq.com
# 邮箱开通 POP3/SMTP 服务时所给的授权码，换成自己申请的
spring.mail.password=voeswedefedvcahabc
# 默认的邮件编码
spring.mail.default-encoding=UTF-8
# SSL 加密工厂
spring.mail.properties.mail.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory
# 开启 DEBUG 模式，邮件发送过程的日志会在控制台打印出来，方便排查错误
spring.mail.properties.mail.debug=true
```

### 3.2 测试

下面的测试代码都写在 `SpringBootMailApplicationTests` 测试类中，先注入相关 Bean ：

```java
@Autowired
JavaMailSender javaMailSender;
@Autowired
MailProperties mailProperties;
```

1. 发送简单邮件

简单邮件就是指邮件内容是一个普通的文本文档。

```java
@Test
public void sendSimpleMail() {
    SimpleMailMessage msg = new SimpleMailMessage();
    msg.setSubject("这是测试邮件主题"); // 邮件主题
    msg.setFrom(mailProperties.getUsername()); // 邮件发送者
    msg.setTo("799737179@qq.com"); // 邮件接收者，可以有多个
    msg.setCc("799737179@qq.com"); // 邮件抄送人，可以有多个
    msg.setBcc("799737179@qq.com"); // 隐秘抄送人，可以有多个
    msg.setSentDate(new Date()); // 邮件发送日期
    msg.setText("这是测试邮件内容"); // 邮件正文
    javaMailSender.send(msg);
}
```

最终效果如下：

![](https://oscimg.oschina.net/oscnet/up-4df28296e8eee85b1012165c8d94d77c2b0.png)

---

2. 发送带附件的邮件

邮件的附件可以是图片，也可以是普通文件，都是支持的。

```java
@Test
public void sendAttachFileMail() throws MessagingException {
    MimeMessage msg = javaMailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(msg, true);
    helper.setSubject("这是测试邮件主题(带附件)");
    helper.setFrom(mailProperties.getUsername());
    helper.setTo("799737179@qq.com");
    // helper.setCc("799737179@qq.com");
    // helper.setBcc("799737179@qq.com");
    helper.setSentDate(new Date());
    helper.setText("这是测试邮件内容(带附件)");
    helper.addAttachment("1.png", new File("D:\\1.png"));
    javaMailSender.send(msg);
}
```

最终效果如下：

![](https://oscimg.oschina.net/oscnet/up-2ad853363a45e8a99d592319a506fa30b6e.png)

---

3. 发送带图片资源的邮件

图片资源和附件有什么区别呢？图片资源是放在邮件正文中的，即一打开邮件，就能看到图片。但是一般来说，不建议使用这种方式，一些公司会对邮件内容的大小有限制（因为这种方式是将图片一起发送的）。

```java
@Test
public void sendImgResMail() throws MessagingException {
    MimeMessage msg = javaMailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(msg, true);
    helper.setSubject("这是测试邮件主题(带图片)");
    helper.setFrom(mailProperties.getUsername());
    helper.setTo("799737179@qq.com");
    // helper.setCc("799737179@qq.com");
    // helper.setBcc("799737179@qq.com");
    helper.setSentDate(new Date());
    helper.setText("这是测试邮件内容(带图片)，这是第一张图片：<img src='cid:p01'/>，这是第二张图片：<img src='cid:p02'/>", true);
    helper.addInline("p01", new FileSystemResource(new File("D:\\1.png")));
    helper.addInline("p02", new FileSystemResource(new File("D:\\2.png")));
    javaMailSender.send(msg);
}
```

最终效果如下：

![](https://oscimg.oschina.net/oscnet/up-74cad4072398623acae2850b756da9eb758.png)

---

4. 使用 `Freemarker` 作邮件模板

先在 `resources/templates` 目录下创建 `mail.ftl` 作为邮件发送模板，内容如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <div>
        hello ${username}，欢迎加入 XXX 大家庭！
    </div>
    您的入职信息如下：
    <table border="1">
        <tr>
            <td>职位</td>
            <td>${position}</td>
        </tr>
        <tr>
            <td>职称</td>
            <td>${joblevel}</td>
        </tr>
        <tr>
            <td>薪水</td>
            <td>${salary}</td>
        </tr>
        <tr>
            <td>部门</td>
            <td>${dep}</td>
        </tr>
    </table>
    <div style="color: #ff1a0e;font-size: 20px">希望在未来的日子里，携手共进！</div>
</body>
</html>
```

接下来，将邮件模板渲染成 HTML ，然后发送即可。

```java
@Test
public void sendFreemarkerMail() throws MessagingException, IOException, TemplateException {
    MimeMessage msg = javaMailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(msg, true);
    helper.setSubject("这是测试邮件主题(Freemarker 模板)");
    helper.setFrom(mailProperties.getUsername());
    helper.setTo("799737179@qq.com");
    // helper.setCc("799737179@qq.com");
    // helper.setBcc("799737179@qq.com");
    helper.setSentDate(new Date());

    // 构建 Freemarker 的基本配置
    Configuration configuration = new Configuration(Configuration.VERSION_2_3_28);
    // 配置模板位置
    configuration.setClassLoaderForTemplateLoading(this.getClass().getClassLoader(), "templates");
    // 加载模板
    Template template = configuration.getTemplate("mail.ftl");
    Map<String, Object> map = new HashMap<>();
    map.put("username", "zhangsan");
    map.put("position", "Java工程师");
    map.put("dep", "产品研发部");
    map.put("salary", 999999);
    map.put("joblevel", "高级工程师");
    StringWriter out = new StringWriter();
    // 模板渲染，渲染的结果将被保存到 out 中 ，将 out 中的 html 字符串发送即可
    template.process(map, out);
    helper.setText(out.toString(), true);
    javaMailSender.send(msg);
}
```

需要注意的是，虽然引入了 Freemarker 的自动化配置，但是我们在这里是直接 `new Configuration` 来重新配置 Freemarker 的，所以 Freemarker 默认的配置这里不生效，因此要手动配置模板位置为 `templates` 。

最终效果如下：

![](https://oscimg.oschina.net/oscnet/up-b4b5c64a51e04dbec118d745b2521f3a7f6.png)

---

5. **使用 `Thymeleaf` 作邮件模板（推荐在 Spring Boot 中使用 Thymeleaf 来构建邮件模板）**

推荐在 Spring Boot 中使用 Thymeleaf 来构建邮件模板。因为 Thymeleaf 的自动化配置提供了一个 `TemplateEngine` ，通过 TemplateEngine 可以方便的将 Thymeleaf 模板渲染为 HTML ，同时，Thymeleaf 的自动化配置在这里是继续有效的。

先在 `resources/templates` 目录下创建 `mail.html` 作为邮件发送模板，内容如下：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <div>
        hello <span th:text="${username}"></span>，欢迎加入 XXX 大家庭！
    </div>
    您的入职信息如下：
    <table border="1">
        <tr>
            <td>职位</td>
            <td th:text="${position}"></td>
        </tr>
        <tr>
            <td>职称</td>
            <td th:text="${joblevel}"></td>
        </tr>
        <tr>
            <td>薪水</td>
            <td th:text="${salary}"></td>
        </tr>
        <tr>
            <td>部门</td>
            <td th:text="${dep}"></td>
        </tr>
    </table>
    <div style="color: #ff1a0e;font-size: 20px">希望在未来的日子里，携手共进！</div>
</body>
</html>
```

---

```java
@Autowired
TemplateEngine templateEngine;

@Test
public void sendThymeleafMail() throws MessagingException {
    MimeMessage msg = javaMailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(msg, true);
    try {
        helper.setSubject("这是测试邮件主题(Thymeleaf 模板)");
        helper.setFrom(mailProperties.getUsername());
        helper.setTo("799737179@qq.com");
        // helper.setCc("799737179@qq.com");
        // helper.setBcc("799737179@qq.com");
        helper.setSentDate(new Date());

        Context context = new Context();
        context.setVariable("username", "zhangsan");
        context.setVariable("position", "Java工程师");
        context.setVariable("dep", "产品研发部");
        context.setVariable("salary", 999999);
        context.setVariable("joblevel", "高级工程师");
        String process = templateEngine.process("mail.html", context);
        helper.setText(process, true);
        javaMailSender.send(msg);
    } catch (MessagingException e) {
        e.printStackTrace();
    }
}
```

最终效果如下：

![](https://oscimg.oschina.net/oscnet/up-7e51e311da22c61b7762cb675862df64f52.png)

## 4 源码解读

邮件发送对应的自动化配置类是 `org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration` ，源码如下：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({MimeMessage.class, MimeType.class, MailSender.class})
@ConditionalOnMissingBean({MailSender.class})
@Conditional({MailSenderAutoConfiguration.MailSenderCondition.class})
@EnableConfigurationProperties({MailProperties.class})
@Import({MailSenderJndiConfiguration.class, MailSenderPropertiesConfiguration.class})
public class MailSenderAutoConfiguration {
    public MailSenderAutoConfiguration() {
    }

    static class MailSenderCondition extends AnyNestedCondition {
        MailSenderCondition() {
            super(ConfigurationPhase.PARSE_CONFIGURATION);
        }

        @ConditionalOnProperty(
            prefix = "spring.mail",
            name = {"jndi-name"}
        )
        static class JndiNameProperty {
            JndiNameProperty() {
            }
        }

        @ConditionalOnProperty(
            prefix = "spring.mail",
            name = {"host"}
        )
        static class HostProperty {
            HostProperty() {
            }
        }
    }
}
```

上述代码中导入了另外一个配置 `MailSenderPropertiesConfiguration` 类，提供了邮件发送相关的工具类，源码如下：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnProperty(
    prefix = "spring.mail",
    name = {"host"}
)
class MailSenderPropertiesConfiguration {
    MailSenderPropertiesConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean({JavaMailSender.class})
    JavaMailSenderImpl mailSender(MailProperties properties) {
        JavaMailSenderImpl sender = new JavaMailSenderImpl();
        this.applyProperties(properties, sender);
        return sender;
    }

    private void applyProperties(MailProperties properties, JavaMailSenderImpl sender) {
        sender.setHost(properties.getHost());
        if (properties.getPort() != null) {
            sender.setPort(properties.getPort());
        }

        sender.setUsername(properties.getUsername());
        sender.setPassword(properties.getPassword());
        sender.setProtocol(properties.getProtocol());
        if (properties.getDefaultEncoding() != null) {
            sender.setDefaultEncoding(properties.getDefaultEncoding().name());
        }

        if (!properties.getProperties().isEmpty()) {
            sender.setJavaMailProperties(this.asProperties(properties.getProperties()));
        }

    }

    private Properties asProperties(Map<String, String> source) {
        Properties properties = new Properties();
        properties.putAll(source);
        return properties;
    }
}
```

其中 `JavaMailSenderImpl` 是 `JavaMailSender` 的一个实现，我们将使用 JavaMailSenderImpl 来完成邮件的发送工作。

---

- [Spring Boot 教程合集](https://mp.weixin.qq.com/s/9vOiAxHFnfJnRwSlTfAHwg)（微信左下方**阅读全文**可直达）。
- Spring Boot 教程合集示例代码：[https://github.com/cxy35/spring-boot-samples](https://github.com/cxy35/spring-boot-samples)
- 本文示例代码：[https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-mail](https://github.com/cxy35/spring-boot-samples/tree/master/spring-boot-mail)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)