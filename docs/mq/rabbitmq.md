- RabbitMQ
  - 缺点：
    - 稳定性问题（消息高可靠传递-不丢失、消息幂等性传递-不重复、消息积压问题）。
    - 分布式一致性问题。
  - 数据丢失问题及解决方案：
    - 生产者投递消息到MQ的过程中丢失，比如：网络故障等。解决方案1：采用事务消息机制，但性能差。解决方案2：采用轻量级的 confirm 机制，类似消费者的手动 ack 消息机制。
    - 消息在MQ中丢失，比如：1.MQ宕机时，消息还未投递到消费者，此时如果没有配置持久化，MQ重启后数据就丢了。解决方案：创建 queue 时和投递 message 时都设置持久化（durable 机制），持久化到磁盘。2.MQ宕机时，消息还没来得及持久化到磁盘上，同时也还没来得及投递到消费者，这样内存中的数据就丢了。解决方案：采用轻量级的 confirm 机制，类似消费者的手动 ack 消息机制。
    - 消费者从MQ消费消息的过程中丢失，比如：1.消费者服务宕机。解决方案：消费时开启手动 ack （注意参数值），消费完之后手动 ack 消息，告知 MQ 自己处理成功了，之后 MQ 会删除这条消息。2.消费消息时发生异常，未成功。解决方案：消费异常时 nack 消息（注意参数值），告知 MQ 自己未处理成功，之后 MQ 会将这条消息重新投递到其他消费者。3.unack 消息积压，导致内存溢出。解决方案：可以通过 `channel.basicQos(10)` 这个方法来设置当前 channel 的 prefetch count，这样的话，就意味着 RabbitMQ 正在投递到 channel 过程中的 unack message，以及消费者服务在处理中的 unack message，以及异步 ack 之后还没完成 ack 的 unack message，所有这些 message 加起来，一个 channel 也不能超过 10 个。RabbitMQ 官方给出的建议是 prefetch count 一般设置在 100-300 之间。



![ack-消费未确认会重新投递](https://oscimg.oschina.net/oscnet/up-39c2850f8d2db939760640b2234626180a8.png)
![队列和消息持久化](https://oscimg.oschina.net/oscnet/up-8ec4fee2aef6df28cf6d16da43b81f00bf2.png)

![ack原理](https://oscimg.oschina.net/oscnet/up-c90139f5fbe1a218f862b4ff1707b91a407.png)
![nack-消费失败会重新投递](https://oscimg.oschina.net/oscnet/up-faeb045b12e6cf0ddcdd1ee10aeb073f640.png)

![prefetch-解决unack消息积压](https://oscimg.oschina.net/oscnet/up-1d52102b1acbd24b227edbac69adc670690.png)



---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)