---
title: Git 配置 SSH keys
date: 2020-02-10 16:57:21
categories: Git
tags: [Git, SSH]
toc: true
# thumbnail: /images/git/git-ssh-keys-thumbnail.jpg
---
GitHub 支持 HTTPS 和 SSH 两种协议。使用 HTTPS 协议时，每次提交都要求输入用户名和密码，显得有点麻烦。本文介绍如何通过配置 SSH keys 实现愉快的提交。
<!-- more -->

配置 SSH keys 的原理很简单，采用非对称加密方式生成公钥和私钥，公钥告诉 GitHub ，私钥留在自己电脑上(私钥不可泄露)，当我们向 GitHub 上提交数据时，GitHub 会用我们留给它的公钥加密一段消息返回给我们的电脑，如果我们能够用私钥解密成功，说明是合法的用户，这样就避免我们输入用户名密码了。大致的原理就是这样，现在很多免登录的系统都采用了这种方式，比如 Hadoop 免登录配置也是这样。

## 1 查看本地是否已有 SSH keys

查看当前用户目录下是否有 `.ssh` 文件夹，如果有就跳过第 2 和 3 步。

```bash
$ ls -la ~/.ssh
total 32
drwxr-xr-x 1 Administrator 197121    0 八月   27 15:29 ./
drwxr-xr-x 1 Administrator 197121    0 二月   10 16:44 ../
-rw-r--r-- 1 Administrator 197121 3381 八月   27 15:21 id_rsa
-rw-r--r-- 1 Administrator 197121  742 八月   27 15:21 id_rsa.pub
-rw-r--r-- 1 Administrator 197121 1593 九月    5 17:05 known_hosts
```

## 2 生成 SSH 指纹

```bash
ssh-keygen -t rsa -b 4096 -C "你的邮箱地址"
```

## 3 添加 SSH 到 ssh-agent 中

```bash
eval "$(ssh-agent -s)"
```

执行完上述语句之后，我们当前用户目录下已经有了一个名为 `.ssh` 的隐藏文件夹了，打开这个目录，会发现有一个名为 `id_rsa.pub` 的文件，这就是我们一会要使用的公钥文件。

## 4 将公钥告诉 GitHub
登录 GitHub ，点击右上角的向下的箭头，选择 Settings ，在新打开的页面中左边侧栏选择 SSH and GPG keys ，再右边选择 New SSH key，输入 Title 和 Key 。

Title 的值建议能标识出哪台设备，如你的电脑型号、操作系统名称等信息。Key 的值为上述 `id_rsa.pub` 文件中的内容。

---

- [Git 教程合集](https://mp.weixin.qq.com/s/S_wAUhlN1hqTjl4CwFS19Q)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)