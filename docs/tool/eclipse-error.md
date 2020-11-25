---
title: Eclipse 问题汇总
date: 2018-04-06 09:44:15
categories: IDE
tags: [IDE, Eclipse, 问题]
toc: true
---
本文记录 Eclipse 使用过程中的相关问题汇总，持续更新中。
<!-- more -->

## 1 打包时没有生成目录信息，导致 Spring 扫描时找不到 jar 包中的 Bean

原因：

打包错误，没有选中这个选项：Add directory entries 。

![](https://static.oschina.net/uploads/space/2017/1124/115344_nFcp_593078.png)

区别如下：

```bash
# 选中
cn/
cn/com/
cn/com/log/
cn/com/log/dao/
cn/com/log/dao/ReadLogInfoDao.class
cn/com/log/dao/LogInfoDao.class

# 不选中（错误）
cn/com/log/dao/ReadLogInfoDao.class
cn/com/log/dao/LogInfoDao.class
```

解决：打包时需要选中上述选项。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)