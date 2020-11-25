---
title: 如何在人力资源管理系统中提高 RabbitMQ 消息消费的可靠性
date: 2020-05-06 18:40:44
categories: HR
tags: [HR, RabbitMQ]
toc: true
---
学习如何在人力资源管理系统中提高 RabbitMQ 消息消费的可靠性，避免消息被重复消费。
<!-- more -->

## 1 概述

在 [如何在人力资源管理系统中提高 RabbitMQ 消息发送的可靠性](https://mp.weixin.qq.com/s/w3GiA7QhGLI6T-yeSHwfLQ) 一文中，我们确保了消息发送的可靠性。但是，在这样的机制下，又带来了新的问题，就是消息可能会重复投递，进而导致消息重复消费。例如一个员工入职了，结果收到了两封入职欢迎邮件。我们需要通过技术手段去处理这个问题。比如增加**消息消费确认机制**，可以有效的提高消息消费的可靠性。

说到这个话题，我们就不得不先来说说消息幂等性。

## 2 幂等性

幂等性本身是数学上的概念，即使公式：f(x)=f(f(x)) 能够成立的数学性质。在开发领域，则表示对于同一个系统，使用相同的条件，一次请求和多次请求对系统资源的影响是一致的。

在分布式系统中幂等性尤为重要，因为分布式系统中，我们经常会用到接口调用失败进而进行重试这个功能，这样就带来了对一个接口可能会使用相同的条件进行重复调用，在这样的条件下，保证接口的幂等性就尤为重要了。常见的解决方案有：

- MVCC 机制： MVCC 多版本并发控制，这种方式就是在数据更新的时候需要去比较所持有的数据版本号，版本号不一致的话，操作会失败，这样每个 version 就只有一次执行成功的机会，一旦失败了必须重新获取。
- **Token 机制**： Token 则是目前使用比较广的一种方式，核心思想就是每个操作都有一个唯一凭证 token ，一旦执行成功，对于重复的请求，总是返回同一个结果。
- 设计去重表
- ...

人力资源管理系统中的 RabbitMQ 消费端实际上就是采用了 Token 这种方式。

## 3 Token 机制

大致的思路是这样，首先将 RabbitMQ 的消息自动确认机制改为手动确认，然后每当有一条消息消费成功了，就把该消息的唯一 ID 记录在 Redis 上，然后每次收到消息时，都先去 Redis 上查看是否有该消息的 ID，如果有，表示该消息已经消费过了，不再处理，否则再去处理。

首先，修改 `hr-mail` 中的 `pom` 文件，增加 Redis 依赖，如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

接着，修改 `application.properties` 文件，增加 Redis 配置，并修改 RabbitMQ 配置，开启消息消费手动确认，如下：

```properties
# 配置 RabbitMQ
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
## 开启消息消费手动确认，默认自动
spring.rabbitmq.listener.simple.acknowledge-mode=manual
spring.rabbitmq.listener.simple.prefetch=100

# 配置 Redis ，实现 RabbitMQ 消息消费的幂等性，避免消息重复消费，见 RabbitMQReceiver
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.database=0
spring.redis.password=
```

---

![](https://oscimg.oschina.net/oscnet/up-e6638b74275db538c4caf1b5875278f5e88.png)

最后，修改 `RabbitMQReceiver` 中的消息消费方法，增加**消息消费确认机制**，如下：

```java
// 监听队列
@RabbitListener(queues = MailConstants.QUEUE_NAME)
public void employeeWelcome(Message message, Channel channel) throws IOException {
    Employee employee = (Employee) message.getPayload();
    MessageHeaders headers = message.getHeaders();
    Long tag = (Long) headers.get(AmqpHeaders.DELIVERY_TAG);
    String msgId = (String) headers.get("spring_returned_message_correlation");
    if (stringRedisTemplate.opsForHash().entries("mail_log").containsKey(msgId)) {
        // redis 中包含该 key，说明该消息已经被消费过
        logger.info("消息已经被消费：" + msgId);
        channel.basicAck(tag, false); // 手动确认消息已消费
        return;
    }
    logger.info(employee.toString());

    // 发送邮件
    MimeMessage msg = javaMailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(msg);
    try {
        helper.setSubject("入职通知");
        helper.setFrom(mailProperties.getUsername());
        helper.setTo(employee.getEmail());
        helper.setSentDate(new Date());

        Context context = new Context();
        context.setVariable("name", employee.getName());
        context.setVariable("positionName", employee.getPosition().getName());
        context.setVariable("jobTitlelName", employee.getJobTitle().getName());
        context.setVariable("departmentName", employee.getDepartment().getName());
        String process = templateEngine.process("employee/welcome", context);
        helper.setText(process, true);
        javaMailSender.send(msg);

        stringRedisTemplate.opsForHash().put("mail_log", msgId, "cxy35");
        channel.basicAck(tag, false); // 手动确认消息已消费
        logger.info("邮件发送成功：" + msgId);
    } catch (MessagingException e) {
        channel.basicNack(tag, false, true); // 消息消费失败，重回队列
        e.printStackTrace();
        logger.error("邮件发送失败：" + e.getMessage());
    }
}
```

---

- [Spring Boot 实战项目（人力资源管理系统）教程合集](https://mp.weixin.qq.com/s/2m9if4Skd2LR6vNezccwqw)（微信左下方**阅读全文**可直达）。
- Spring Boot 实战项目（人力资源管理系统）源码：[https://github.com/cxy35/hr](https://github.com/cxy35/hr)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)