---
title: Git 安装
date: 2020-02-05 16:13:41
categories: Git
tags: [Git, 安装]
toc: true
thumbnail: /images/git/git-thumbnail.jpg
---
手把手带你安装 Git 。
<!-- more -->

## Windows

windows 安装 Git 整体上来说有两种解决方案：

1. 安装 [Cygwin](http://cygwin.com/) 用来模拟 Linux 运行环境，但是 Cygwin 配置非常麻烦，容易出错，所以一般不推荐这种方式。
2. 安装独立的 Git，也就是 [msysGit](https://git-for-windows.github.io/)，下载 `Git-2.23.0-64-bit.exe` ，安装成功后，在你的开始菜单中找到 Git Bash，或者鼠标右键 Git Bash Here 可打开命令行。

```bash
$ git --version
git version 2.23.0.windows.1
```

## Ubuntu

```bash
sudo apt-get install git
```

---

- [Git 教程合集](https://mp.weixin.qq.com/s/S_wAUhlN1hqTjl4CwFS19Q)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)