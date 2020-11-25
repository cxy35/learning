---
title: Nginx 安装 - Linux
date: 2020-03-05 10:29:48
categories: Nginx
tags: [Nginx, 安装]
toc: true
---
手把手带你在 Linux 上安装 Nginx-1.16.1 。
<!-- more -->

## 1 准备工作

### 1.1 下载安装包

- Nginx 下载地址：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)
- 下载当前稳定版本 Stable version ，如：`nginx-1.16.1.tar.gz`

### 1.2 安装编译工具及库文件

```bash
# 根据系统实际情况选择安装
# gcc 环境
# pcre 库：一个 Perl 库，包括 perl 兼容的正则表达式库。 Nginx 的 http 模块使用 pcre 来解析正则表达式
# zlib 库：提供了很多种压缩和解压缩的方式。 Nginx 使用 zlib 对 http 包的内容进行 gzip
# openssl 库：一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。 Nginx 不仅支持 http 协议，还支持 https （即在 ssl 协议上传输 http ）

# 下面是全部安装
yum -y install gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel make libtool
```

## 2 安装 Nginx

- 上传 `nginx-1.16.1.tar.gz` 或通过 `wget http://nginx.org/download/nginx-1.16.1.tar.gz` 在线下载。
- 解压到 `/usr/local` 目录下。

```bash
tar -xzvf nginx-1.16.1.tar.gz -C /usr/local
```

- 编译安装

```bash
# 进入 nginx-1.16.1 目录
cd /usr/local/nginx-1.16.1

# 执行 cofigure 命令创建一个 makeFile 文件，如果没有，编译的时候会报错
# \ 表示命令还没有输入完，换行的意思
# --prefix=/usr/local/nginx 指定 Nginx 安装目录
./configure \
--prefix=/usr/local/nginx \
--error-log-path=/usr/local/nginx/logs/error.log \
--http-log-path=/usr/local/nginx/logs/access.log \
--pid-path=/usr/local/nginx/logs/nginx.pid \
--lock-path=/usr/local/nginx/logs/nginx.lock \
--http-client-body-temp-path=/usr/local/nginx/temp/client-body \
--http-proxy-temp-path=/usr/local/nginx/temp/proxy \
--http-fastcgi-temp-path=/usr/local/nginx/temp/fastcgi \
--http-uwsgi-temp-path=/usr/local/nginx/temp/uwsgi \
--http-scgi-temp-path=/usr/local/nginx/temp/scgi \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-file-aio \
--with-http_realip_module

# 增加第三方模块（需要提前准备模块源码，供编译时用）
# \
# --add-module=/usr/local/nginx-upstream-fair-master \
# --add-module=/usr/local/nginx_upstream_check_module-master

# 创建上述相关目录
mkdir -p /usr/local/nginx/{logs,temp}

# 编译安装
make && make install

# 执行完成之后，在 Nginx 安装目录下( /usr/local/nginx )多个几个目录：conf/html/sbin

# 切换到 Nginx 安装目录
cd /usr/local/nginx

# 查看 Nginx 版本
/usr/local/nginx/sbin/nginx -v
# nginx version: nginx/1.16.1

# 查看 Nginx 安装时的配置参数
/usr/local/nginx/sbin/nginx -V
# configure arguments: --prefix=/usr/local/nginx --error-log-path=/usr/local/nginx/logs/error.log --http-log-path=/usr/local/nginx/logs/access.log --pid-path=/usr/local/nginx/logs/nginx.pid --lock-path=/usr/local/nginx/logs/nginx.lock --http-client-body-temp-path=/usr/local/nginx/temp/client-body --http-proxy-temp-path=/usr/local/nginx/temp/proxy --http-fastcgi-temp-path=/usr/local/nginx/temp/fastcgi --http-uwsgi-temp-path=/usr/local/nginx/temp/uwsgi --http-scgi-temp-path=/usr/local/nginx/temp/scgi --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-file-aio --with-http_realip_module

# 启动
/usr/local/nginx/sbin/nginx

# 检查是否启动
ps -ef|grep nginx
# root     26063     1  0 14:14 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
# nobody   26064 26063  0 14:14 ?        00:00:00 nginx: worker process      
# root     26066 22818  0 14:15 pts/1    00:00:00 grep nginx

# 关闭
/usr/local/nginx/sbin/nginx -s stop
# /usr/local/nginx/sbin/nginx -s quit

# 重启
/usr/local/nginx/sbin/nginx -s reopen

# 重新载入配置文件 nginx.conf ，有失效的风险
/usr/local/nginx/sbin/nginx -s reload

# 检查配置文件 nginx.conf 的正确性
/usr/local/nginx/sbin/nginx -t
```

启动后，访问 http://127.0.0.1 验证，效果如下：

![](https://oscimg.oschina.net/oscnet/up-2817c8f57921388ed8b69729f77eff74a1f.png)


## 3 配置 Nginx

[Nginx 参数配置优化](https://mp.weixin.qq.com/s/wS-ly5O_xSJbVzJ24_yAKQ)

```bash
vi /usr/local/nginx/conf/nginx.conf

# ......
```

---

- [Nginx 教程合集](https://mp.weixin.qq.com/s/TdLki2vnjW4hKUz_BgzEHg)（微信左下方**阅读全文**可直达）。
- [Nginx 参数配置优化](https://mp.weixin.qq.com/s/wS-ly5O_xSJbVzJ24_yAKQ)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)