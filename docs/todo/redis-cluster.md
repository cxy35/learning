cluster的出现是为了解决单机Redis容量有限的问题，将Redis的数据根据一定的规则分配到多台机器。对cluster的一些理解：
•	cluster可以说是sentinel和主从模式的结合体，通过cluster可以实现主从和master重选功能，所以如果配置两个副本三个分片的话，就需要六个Redis实例。
•	因为Redis的数据是根据一定规则分配到cluster的不同机器的，当数据量过大时，可以新增机器进行扩容
　　这种模式适合数据量巨大的缓存要求，当数据量不是很大使用sentinel即可。



---

- [Redis 教程合集](https://mp.weixin.qq.com/s/iivXrj1cfTiPy89ueE_53Q)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)