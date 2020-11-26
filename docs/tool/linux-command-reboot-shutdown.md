---
title: Linux 常用命令 - 重启、关机
date: 2019-01-12 16:16:45
categories: Linux
tags: [Linux, 命令]
toc: true
---
通过本文学习 Linux 常用命令-重启、关机：reboot shutdown 等。
<!-- more -->

## 重启：

- reboot --> 注：立刻重启
- shutdown -r now --> 注：立刻重启（ root 用户使用）
- shutdown -r 10 --> 注： 10 分钟后重启（ root 用户使用）
- shutdown -r 20:35 --> 注： 20:35 时重启（ root 用户使用）

注：如果是通过 shutdown 命令设置重启的话，可以用 shutdown -c 命令取消重启。

## 关机：

- halt --> 注：立刻关机
- shutdown -h now --> 注：立刻关机（ root 用户使用）
- shutdown -h 10 --> 注： 10 分钟后关机
- poweroff --> 注：立刻关机

注：如果是通过 shutdown 命令设置关机的话，可以用 shutdown -c 命令取消关机。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)