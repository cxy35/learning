---
title: 如何在人力资源管理系统中提高 RabbitMQ 消息发送的可靠性
date: 2020-05-05 18:50:18
categories: HR
tags: [HR, RabbitMQ]
toc: true
---
学习如何在人力资源管理系统中提高 RabbitMQ 消息发送的可靠性，避免因为网络抖动等原因导致消息发送失败。
<!-- more -->

## 1 概述

我们在**人力资源管理系统**中引入了消息中间件 `RabbitMQ` ，并结合 RabbitMQ 搭建了独立的邮件服务器 `hr-mail` 。当在系统中录入一个员工后，系统会自动向消息中间件 RabbitMQ 发送一条消息，这条消息包含了新入职员工的基本信息。然后邮件服务器从 RabbitMQ 上消费消息，根据收到的消息，自动的发送一封入职欢迎邮件。

RabbitMQ 虽然用着方便，有很多优势，但是也带来了很多问题，例如网络抖动怎么办？如何确保消息的可靠性？在理想的环境下这些问题都不存在，但是在复杂的生产环境中，什么都是有可能的。所以，我们需要通过技术手段去处理这些问题。比如增加**消息发送确认机制**和**消息发送失败自动重试机制**，可以有效的提高消息发送的可靠性。

![](https://oscimg.oschina.net/oscnet/up-e6638b74275db538c4caf1b5875278f5e88.png)

## 2 准备工作

新增**邮件发送日志**模块，用于记录消息发送相关信息，如消息投递状态、重试时间、重试次数等。

```java
public class MailSendLog {
    private Integer id;
    private String msgId;
    private Integer empId;
    private Integer status; // 0投递中 1投递成功 2投递失败
    private String routeKey;
    private String exchange;
    private Integer count; // 重试次数
    private Date tryTime; // 第一次重试时间
    private Date createTime;
    private Date updateTime;

    // getter/setter
}
```

```java
public interface MailSendLogMapper {
    Integer updateMailSendLogStatus(@Param("msgId") String msgId, @Param("status") Integer status);

    Integer insert(MailSendLog mailSendLog);

    List<MailSendLog> getMailSendLogsByStatus();

    Integer updateCount(@Param("msgId") String msgId, @Param("date") Date date);
}
```

```xml
<mapper namespace="com.zhengjian.hr.mapper.MailSendLogMapper">
    <insert id="insert" parameterType="com.zhengjian.hr.model.MailSendLog">
        insert into t_mail_send_log (msgId,empId,routeKey,exchange,tryTime,createTime,updateTime) values (#{msgId},#{empId},#{routeKey},#{exchange},#{tryTime},#{createTime},#{updateTime});
    </insert>

    <update id="updateMailSendLogStatus">
        update t_mail_send_log set status = #{status} where msgId=#{msgId};
    </update>

    <select id="getMailSendLogsByStatus" resultType="com.zhengjian.hr.model.MailSendLog">
        select * from t_mail_send_log where status=0 and tryTime &lt; sysdate()
    </select>

    <update id="updateCount">
        update t_mail_send_log set count=count+1,updateTime=#{date} where msgId=#{msgId};
    </update>
</mapper>
```

```java
@Service
public class MailSendLogService {
    @Autowired
    MailSendLogMapper mailSendLogMapper;

    public Integer insert(MailSendLog mailSendLog) {
        return mailSendLogMapper.insert(mailSendLog);
    }

    public Integer updateMailSendLogStatus(String msgId, Integer status) {
        return mailSendLogMapper.updateMailSendLogStatus(msgId, status);
    }

    public List<MailSendLog> getMailSendLogsByStatus() {
        return mailSendLogMapper.getMailSendLogsByStatus();
    }

    public Integer updateCount(String msgId, Date date) {
        return mailSendLogMapper.updateCount(msgId, date);
    }
}
```

```java
public class MailConstants {
    public static final Integer STATUS_DELIVERING = 0; // 消息投递中
    public static final Integer STATUS_SUCCESS = 1; // 消息投递成功
    public static final Integer STATUS_FAILURE = 2; // 消息投递失败
    public static final Integer MAX_TRY_COUNT = 3; // 最大重试次数
    public static final Integer MSG_TIMEOUT = 1; // 消息超时时间（分钟），超过这个时间才会开始重试
    public static final String QUEUE_NAME = "hr.mail.employee.welcome.queue";
    public static final String EXCHANGE_NAME = "hr.mail.employee.welcome.exchange";
    public static final String ROUTING_KEY_NAME = "hr.mail.employee.welcome.routingKey";
}
```

## 3 消息发送确认

开启消息发送失败回调，路由失败回调。

新增 `RabbitMQConfig` 配置类，自定义 `RabbitTemplate` ，增加了消息发送回调，提高消息发送的可靠性。

```java
@Configuration
public class RabbitMQConfig {
    public final static Logger logger = LoggerFactory.getLogger(RabbitMQConfig.class);
    @Autowired
    CachingConnectionFactory cachingConnectionFactory;
    @Autowired
    MailSendLogService mailSendLogService;

    @Bean
    RabbitTemplate rabbitTemplate() {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(cachingConnectionFactory);
        // 消息发送到交换机的回调
        rabbitTemplate.setConfirmCallback((data, ack, cause) -> {
            String msgId = data.getId();
            if (ack) {
                // 修改数据库中的记录，消息投递成功
                mailSendLogService.updateMailSendLogStatus(msgId, MailConstants.STATUS_SUCCESS);
                logger.info("消息发送成功：" + msgId);
            } else {
                logger.info("消息发送失败：" + msgId);
            }
        });
        // 消息从交换机发送到队列的回调
        rabbitTemplate.setReturnCallback((msg, repCode, repText, exchange, routingkey) -> {
            logger.info("消息发送失败");
        });
        return rabbitTemplate;
    }

    @Bean
    Queue mailQueue() {
        return new Queue(MailConstants.QUEUE_NAME, true);
    }

    @Bean
    DirectExchange mailExchange() {
        return new DirectExchange(MailConstants.EXCHANGE_NAME, true, false);
    }

    @Bean
    Binding mailBinding() {
        return BindingBuilder.bind(mailQueue()).to(mailExchange()).with(MailConstants.ROUTING_KEY_NAME);
    }

}
```

注意，需要在配置文件中开启 RabbitMQ 的相关回调，如下：

```properties
## 开启 confirm 回调
spring.rabbitmq.publisher-confirms=true
## 开启 return 回调
spring.rabbitmq.publisher-returns=true
```

---

修改 `EmployeeService` 中新增员工的方法，保存邮件发送日志，并向 RabbitMQ 投递消息，如下：

```java
public int add(Employee employee) {
    handleContractTerm(employee);
    int r = employeeMapper.insertSelective(employee);
    if (r == 1) {
        // 获取关联信息
        Employee employeeWithAll = employeeMapper.selectWithAllByPrimaryKey(employee.getId());

        // jmsMessagingTemplate.convertAndSend(MailConstants.QUEUE_NAME, employeeWithAll);

        // 生成消息的唯一id
        String msgId = UUID.randomUUID().toString();
        MailSendLog mailSendLog = new MailSendLog();
        mailSendLog.setMsgId(msgId);
        Date date = new Date();
        mailSendLog.setCreateTime(date);
        mailSendLog.setUpdateTime(date);
        mailSendLog.setExchange(MailConstants.EXCHANGE_NAME);
        mailSendLog.setRouteKey(MailConstants.ROUTING_KEY_NAME);
        mailSendLog.setEmpId(employeeWithAll.getId());
        mailSendLog.setTryTime(new Date(System.currentTimeMillis() + 1000 * 60 * MailConstants.MSG_TIMEOUT));
        mailSendLogService.insert(mailSendLog);
        rabbitTemplate.convertAndSend(MailConstants.EXCHANGE_NAME, MailConstants.ROUTING_KEY_NAME, employeeWithAll, new CorrelationData(msgId));
    }
    return r;
}
```

## 4 消息发送失败自动重试

开启定时任务巡查，发现有发送失败的消息自动重新投递。

新增**消息发送定时任务**，针对发送失败的消息会自动重试一定的次数，提高消息发送的可靠性。

```java
@Component
public class MailSendTask {
    @Autowired
    MailSendLogService mailSendLogService;
    @Autowired
    RabbitTemplate rabbitTemplate;
    @Autowired
    EmployeeMapper employeeMapper;

    @Scheduled(cron = "0/10 * * * * ?")
    public void mailResendTask() {
        List<MailSendLog> logs = mailSendLogService.getMailSendLogsByStatus();
        if (logs == null || logs.size() == 0) {
            return;
        }
        logs.forEach(mailSendLog -> {
            if (mailSendLog.getCount() >= MailConstants.MAX_TRY_COUNT) {
                // 设置该条消息发送失败
                mailSendLogService.updateMailSendLogStatus(mailSendLog.getMsgId(), MailConstants.STATUS_FAILURE);
            } else {
                mailSendLogService.updateCount(mailSendLog.getMsgId(), new Date());
                Employee emp = employeeMapper.selectWithAllByPrimaryKey(mailSendLog.getEmpId());
                rabbitTemplate.convertAndSend(MailConstants.EXCHANGE_NAME, MailConstants.ROUTING_KEY_NAME, emp, new CorrelationData(mailSendLog.getMsgId()));
            }
        });
    }
}
```

注意，需要在启动类上增加 `@EnableScheduling` 注解，开启定时任务，如下：

```java
@SpringBootApplication
@MapperScan(basePackages = "com.zhengjian.hr.mapper")
@EnableCaching // 开启缓存
@EnableScheduling // 开启定时任务
public class HrApplication {

    public static void main(String[] args) {
        SpringApplication.run(HrApplication.class, args);
    }

}
```

---

- [Spring Boot 实战项目（人力资源管理系统）教程合集](https://mp.weixin.qq.com/s/2m9if4Skd2LR6vNezccwqw)（微信左下方**阅读全文**可直达）。
- Spring Boot 实战项目（人力资源管理系统）源码：[https://github.com/cxy35/hr](https://github.com/cxy35/hr)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)