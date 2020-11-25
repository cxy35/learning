---
title: Nginx 常见问题
date: 2020-03-02 14:02:59
categories: Nginx
tags: [Nginx, 问题]
toc: true
---
本文记录 Nginx 使用过程中出现的问题汇总，持续更新中。
<!-- more -->

## 1 maximum number of descriptors supported by select() is 1024

原因：

使用 `nginx-1.16.0.zip` 类似这种官方版本的 Nginx ，在 Windows 下因为文件访问句柄数被限制为 1024 了（参数 worker_connections 配置无效），当访问量大时就会无法响应。

解决：

使用专门的 Windows 版本的 Nginx ，已修改了文件句柄数据的限制。

- 官网：[http://nginx-win.ecsds.eu/](http://nginx-win.ecsds.eu/)
- 下载地址：[http://nginx-win.ecsds.eu/download/](http://nginx-win.ecsds.eu/download/)

下载后 `nginx 1.17.8.1 Unicorn.zip` 里面有个简要的更新信息和安装指南 Readme nginx-win version.txt 。

复制 conf 文件夹中的 nginx-win.conf 并重命名为 nginx.conf ，然后在此文件中做配置。用 nginx.exe 启动如果无效，改成用 nginx_basic.exe 启动。

**如果条件允许的话建议使用 linux 版本。**

## 2 集群中的某个节点正在关闭中或启动中，请求还会被转发过去吗？

不会，正在执行的请求会立刻被转发到其他节点上。

## 3 集群中的某个节点被关闭后，请求还会被转发过去吗？

不会。但有些时候请求要等 1 分钟才能收到其他节点的返回结果，偶尔出现，为什么？

## 4 集群中的某个节点关闭再启动，请求还会被转发过去吗？

居然不会，要重启 Nginx 才会生效，为什么？

---

- [Nginx 教程合集](https://mp.weixin.qq.com/s/TdLki2vnjW4hKUz_BgzEHg)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)