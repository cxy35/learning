---
title: Apache 实现 Tomcat 集群 - mod_jk（老版本）
date: 2018-05-05 11:51:35
categories: Apache
tags: [Apache, Tomcat, 集群, mod_jk]
toc: true
---
使用 Apache 实现 Tomcat 集群，采用 mod_jk 组件（老版本）。
<!-- more -->

## 1 准备工作

- Apache 版本：httpd-2.4.23-x64
- 下载地址：[http://httpd.apache.org/download.cgi](http://httpd.apache.org/download.cgi)

---

- Tomcat 版本：apache-tomcat-8.0.53-windows-x64
- 下载地址：[http://tomcat.apache.org/download-70.cgi](http://tomcat.apache.org/download-70.cgi)

- tomcat1（http-8080，ajp-9009，webapp/test）
- tomcat2（http-8081，ajp-9010，webapp/test）
- tomcat3（http-8082，ajp-9011，webapp/test）

安装并分别启动成功。

## 2 配置 Apache

### 2.1 下载 mod_jk.so

下载 mod_jk.so，放到 Apache 根目录下的 modules 文件夹下。

下载地址：[http://tomcat.apache.org/download-connectors.cgi](http://tomcat.apache.org/download-connectors.cgi)

### 2.2 配置 httpd.conf

编辑 Apache根目录下的 conf/httpd.conf 文件，增加配置如下：

```bash
##### modify #####
#Include conf/mod_jk.conf

# 加载 mod_jk.so 模块
LoadModule jk_module modules/mod_jk.so

# 加载 mod_jk.so 配置文件
JkWorkersFile conf/workers.properties

# 指定那些请求交给 tomcat 处理，"controller" 为在 workers.propertise 里指定的负载分配控制器，这里表示所有请求都是 Tomcat 处理
JkMount /* controller
```

### 2.3 配置 workers.properties

在 Apache 根目录下的 conf 下新建 workers.properties 文件，配置如下：

```bash
# server
# ===== 配置服务器集群列表 =====
worker.list = controller,tomcat1,tomcat2

# tomcat1
# ===== 此名字对应 tomcat 中 server.xml <Engine ...... jvmRoute="tomcat1"> =====
worker.tomcat1.port=9009	# tomcat1 中 AJP 协议端口
worker.tomcat1.host=localhost	# tomcat 的主机地址，如不为本机，请填写 ip 地址
worker.tomcat1.type=ajp13	# 指定 tomcat 与 apache 的通讯协议为 AJP
worker.tomcat1.lbfactor=1	# 指定负载平衡因数，只有启用了负载平衡才有用。值越高，分得的请求越多

# tomcat2
# ===== 此名字对应 tomcat 中 server.xml <Engine ...... jvmRoute="tomcat2"> =====
worker.tomcat2.port=9010	# tomcat1 中 AJP 协议端口
worker.tomcat2.host=localhost	# tomcat 的主机地址，如不为本机，请填写 ip 地址
worker.tomcat2.type=ajp13	# 指定 tomcat 与 apache 的通讯协议为 AJP
worker.tomcat2.lbfactor=1	# 指定负载均衡因数，只有启用了负载均衡才有用。值越高，分得的请求越多

# ===== controller，负载均衡控制器 =====
worker.controller.type=lb
# 指定分担请求的 tomcat
worker.controller.balance_workers=tomcat1,tomcat2
# 会话是否有粘性，false 表示无粘性，同一个会话的请求会到不同的 tomcat 中处理
worker.controller.sticky_session=false
# 当 sticky_session 设为 true 时，该参数才有意义。当一个节点蹦了，如果为 true ，那么服务器返回 500 错误给客户端，如果为 false ，则转发给其他的 tomcat ，但是会丢失会话信息
# worker.controller.sticky_session_force=true
```

## 3 配置 Tomcat

编辑 Tomcat 根目录下的 conf/server.xml 文件，修改配置：

```xml
<!-- 修改 AJP 协议端口，与上面的 worker.properties 中 port 对应 -->
<Connector port="9009" protocol="AJP/1.3" redirectPort="9443"/>

<!-- 添加 jvmRoute ，与上面的 worker.properties 中对应，集群时候用 -->
<Engine defaultHost="localhost" name="Catalina" jvmRoute="tomcat1">
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
```

## 4 测试

在 Tomcat 根目录下的 webapps 下新建 test 目录，再在里面新建 test.jsp 文件：

```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<%@page import="java.util.*"%>

<html>
<head>
    <title>Tomcat集群测试</title>
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
</head>
<body>
<%  
out.println(new Date() + "<br/>");
out.println("[tomcat1:8080]" + "<br/>");
String sessionid = request.getSession().getId();
out.println("sessionid: " + sessionid + "<br/>");
System.out.println("sessionid: " + sessionid);
out.println("=============================================<br/>");

Enumeration enu = request.getHeaderNames();  
while(enu.hasMoreElements()){  
	String key = (String)enu.nextElement();  
	out.println(key + ": " + request.getHeader(key) + "<br/>");  
}
out.println("=============================================<br/>");

out.println("request.remote: "+request.getRemoteAddr()+":"+request.getRemotePort() + "<br/>");
out.println("request.local: "+request.getLocalAddr()+":"+request.getLocalPort() + "<br/>");
out.println("=============================================<br/>");
%>
</body>
</html>
```

再在里面新建 WEB-INF/web.xml 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_5.xsd" version="2.5">
	<display-name>test</display-name>
	<distributable />
</web-app>
```

通过 [http://127.0.0.1/test/test.jsp](http://localhost/test/test.jsp) 访问，查看页面与控制台。

## 5 其他说明

### 5.1 mod_jk 与 mod_proxy

- **Apache2.2 之前，主要用 mod_jk 组件，也比较稳定。**
- **Apache2.2 之后，可用 Apache 自带的 mod_proxy 组件。**

### 5.2 Apache 日志

Apache 根目录下的 logs 文件夹下：

- access.log：用于记录 Apache 接收到请求以及响应状态的日志。
- error.log：用于记录 Apache 的运行错误。
- httpd.pid：用于记录进程 pid 。
- mod_jk.log：用于记录请求转发给 Tomcat 的日志。

### 5.3 Tomcat 日志

Tomcat 根目录下的 logs 文件夹下：

- localhost_access_log.日期.txt：用于记录 Tomcat 接收到的请求以及响应的状态等，作用与 Apache 的 access.log 类似。
- catalina.日期.log：用于记录 Tomcat 启动时候控制台的一些信息以及服务端错误信息。
- localhost.日期.log：用于记录站点访问信息， Tomcat 下内部代码丢出的日志。

### 5.4 其他

Apache 根目录下的 htdocs 文件夹的作用与 Tomcat 的 webapps 类似。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)