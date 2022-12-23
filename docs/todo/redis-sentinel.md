sentinel的中文含义是哨兵、守卫。也就是说既然主从模式中，当master节点挂了以后，slave节点不能主动选举一个master节点出来，那么我就安排一个或多个sentinel来做这件事，当sentinel发现master节点挂了以后，sentinel就会从slave中重新选举一个master。

　　对sentinel模式的理解：
•	sentinel模式是建立在主从模式的基础上，如果只有一个Redis节点，sentinel就没有任何意义
•	当master节点挂了以后，sentinel会在slave中选择一个做为master，并修改它们的配置文件，其他slave的配置文件也会被修改，比如slaveof属性会指向新的master
•	当master节点重新启动后，它将不再是master而是做为slave接收新的master节点的同步数据
•	sentinel因为也是一个进程有挂掉的可能，所以sentinel也会启动多个形成一个sentinel集群
•	当主从模式配置密码时，sentinel也会同步将配置信息修改到配置文件中，不许要担心。
•	一个sentinel或sentinel集群可以管理多个主从Redis。
•	sentinel最好不要和Redis部署在同一台机器，不然Redis的服务器挂了以后，sentinel也挂了
•	sentinel监控的Redis集群都会定义一个master名字，这个名字代表Redis集群的master Redis。

 　　当使用sentinel模式的时候，客户端就不要直接连接Redis，而是连接sentinel的ip和port，由sentinel来提供具体的可提供服务的Redis实现，这样当master节点挂掉以后，sentinel就会感知并将新的master节点提供给使用者。
　　sentinel模式基本可以满足一般生产的需求，具备高可用性。但是当数据量过大到一台服务器存放不下的情况时，主从模式或sentinel模式就不能满足需求了，这个时候需要对存储的数据进行分片，将数据存储到多个Redis实例中，就是下面要讲的。



---

- [Redis 教程合集](https://mp.weixin.qq.com/s/iivXrj1cfTiPy89ueE_53Q)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)