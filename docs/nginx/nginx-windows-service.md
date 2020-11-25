---
title: Nginx 添加到 Windows 系统服务
date: 2020-03-02 18:17:54
categories: Nginx
tags: [Nginx]
toc: true
---
借助第三方工具 WinSW 将 Nginx 添加到 Windows 系统服务，实现自启动。
<!-- more -->

## 1 准备工作

WinSW 是一个可执行的二进制文件，可用于将自定义进程包装和管理为 Windows 服务。下载安装包后，可以重命名为任何名称，如： myService.exe 。

下载地址：[https://github.com/kohsuke/winsw/releases](https://github.com/kohsuke/winsw/releases)，下载 .NET 对应的 exe 文件，如： `WinSW.NET4.exe` 。

## 2 配置

1. 将 WinSW.NET4.exe 拷贝到 nginx 安装目录下，如：`D:\nginx-1.16.0` ，并重命名为 `nginx-service.exe` 。
2. 在 nginx 安装目录下新建 `nginx-service.xml` 文件，如下：

```xml
<service>
  <!-- id 服务唯一标识 -->
  <id>nginx</id>
  <!-- name 在windowServer中显示的名字 -->
  <name>Nginx Service</name>
  <!-- description 描述 -->
  <description>High Performance Nginx Service</description>
  <!-- logpath winsw的日志输出地址 -->
  <logpath>D:\nginx-1.16.0\logs-service</logpath>
  <!-- log 日志信息的配置 -->
  <log mode="roll-by-size">
    <sizeThreshold>10240</sizeThreshold>
    <keepFiles>8</keepFiles>
  </log>
  <!-- executable windows服务启动时要执行的命令 -->
  <executable>D:\nginx-1.16.0\nginx.exe</executable>
  <!-- startarguments 启动时要带的参数 -->
  <startarguments>-p D:\nginx-1.16.0</startarguments>
  <!-- stopexecutable windows服务停止时要执行的命令 -->
  <stopexecutable>D:\nginx-1.16.0\nginx.exe</stopexecutable>
  <!-- stoparguments 停止时要带的参数 -->
  <stoparguments>-p D:\nginx-1.16.0 -s stop</stoparguments>
</service>
```

3. 在 nginx 安装目录下以管理员身份打开命令行，并执行：

```bash
# 安装服务
nginx-service install
# 卸载服务
# nginx-service uninstall
```

之后在 Windows 系统服务中就能看到 `Nginx Service` 这个服务了，如下：

![](https://oscimg.oschina.net/oscnet/up-b5a377097316c54f54f5a459d0031f0fa22.png)

---

- [Nginx 教程合集](https://mp.weixin.qq.com/s/TdLki2vnjW4hKUz_BgzEHg)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)