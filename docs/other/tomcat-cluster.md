---
title: Tomcat 集群
date: 2018-05-02 11:44:00
categories: Tomcat
tags: [Tomcat, 集群]
toc: true
---
多种方式实现 Tomcat 集群。
<!-- more -->

## 1 概述

### 1.1 集群能带来什么

- 提高服务的性能，例如计算处理能力、并发能力等，以及实现服务的高可用性。
- 提供项目架构的横向扩展能力，增加集群中的机器就能提高集群的性能。
- 提升对静态文件的处理性能。
- 利用 Web 服务器来做负载均衡以及容错。
- 无缝的升级应用程序。

### 1.2 集群实现方式

- Tomcat 集群的实现方式有多种，最简单的就是通过 Nginx 负载进行请求转发来实现。
- session 共享问题。

### 1.3 集群解决方案

- 采用 Nginx 中的 ip hash policy 来保持某个ip始终连接在某一个机器上
    - 优点：可以不改变现有的技术架构，直接实现横向扩展，省事。但是缺陷也很明显，在实际的生产环境中，极少使用这种方式
    - 缺点：1.单止服务器请求（负载）不均衡，这是完全依赖 ip hash 的结果。2.客户机 ip 动态变化频繁的情况下，无法进行服务，因为可能每次的 ip hash 都不一样，就无法始终保持只连接在同一台机器上。
- 采用 redis 或 memchche 等 nosql 数据库，实现一个缓存 session 的服务器，当请求过来的时候，所有的 Tomcat Server 都统一往这个服务器里读取 session 信息。这是企业中比较常用的一种解决方案，所以大致的 Tomcat 集群的架构图如下：

![](https://oscimg.oschina.net/oscnet/66e6b896677b6165798c7844a654c38a434.jpg)

## 2 Apache

### 2.1 mod_jk 组件（老版本）

具体配置见：[Apache 实现 Tomcat 集群 - mod_jk（老版本）](https://mp.weixin.qq.com/s/TGgO7suN_F3I_PqoVAN9ww)

### 2.2 mod_proxy 组件（新版本）

具体配置见：[Apache 实现 Tomcat 集群 - mod_proxy（新版本）](https://mp.weixin.qq.com/s/wDrhTHxf0mW72DgJFkSftA)

## 3 Nginx（推荐）

具体配置见：[Nginx 实现 Tomcat 集群（推荐）](https://mp.weixin.qq.com/s/mq54xYiVd76EJFruHGjtbQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)