---
title: Linux 常用命令 - 文件传输
date: 2019-01-13 16:17:03
categories: Linux
tags: [Linux, 命令]
toc: true
---
通过本文学习 Linux 常用命令-文件传输：scp rz sz 等。
<!-- more -->

## ssh 命令行传输：scp

```bash
# 文件上传，默认 22 端口
scp /usr/local/test.txt root@192.168.1.53:/usr/local

# 文件上传，指定 8122 端口
# test.txt 会被上传到 local 目录下( /usr/local/test.txt )
scp -P 8122 /usr/local/test.txt root@192.168.1.53:/usr/local

# 目录上传
# test 目录会被上传到 local 目录下( /usr/local/test )
scp -r /usr/local/test root@192.168.1.53:/usr/local

# 文件下载
scp root@192.168.1.53:/usr/local/test.txt /usr/local

# 目录下载
scp -r root@192.168.1.53:/usr/local/test /usr/local
```

## xshell 结合 lrzsz 工具传输：rz sz

```bash
# 检查是否安装过工具 lrzsz
rpm -qa |grep lrzsz

# 安装工具 lrzsz
yum -y install lrzsz

# 文件上传，会打开本地选择文件对话框
rz

# 文件下载，会弹出选择本地保存文件对话框
sz

# xshell 中直接拖拽文件到窗口中也可以上传。 Alt+P 打开属性框，打开[文件传输]可以调整传输的一些属性
```


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)