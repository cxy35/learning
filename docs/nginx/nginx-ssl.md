---
title: Nginx 配置 SSL 支持 HTTPS（自签证书）
date: 2020-05-22 11:08:53
categories: Nginx
tags: [Nginx, SSL, HTTPS]
toc: true
---
Nginx 配置 SSL ，使其支持 HTTPS（自签证书）。
<!-- more -->

## 1 检查 Nginx 是否支持 SSL

```bash
/usr/local/nginx/sbin/nginx  -V
```

查看是否包含 `--with-http_ssl_module` 模块，如果没有，则需要在编译时指定或增加该模块。

1. 未安装过 Nginx

具体安装步骤参考： [Nginx 安装 - Linux](https://mp.weixin.qq.com/s/UypOmZsfZmiAz3_FTk3z7Q)

```bash
# 只需要在 ./configure 时指定 ssl 模块
--with-http_ssl_module
```

2. 已安装过 Nginx

如果已经安装过 Nginx ，又不想重新安装，则可以单独添加 ssl 模块。

```bash
# 关闭 Nginx
/usr/local/nginx/sbin/nginx -s stop

# 查看 Nginx 安装时的配置参数，复制备用
/usr/local/nginx/sbin/nginx -V
# configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_gzip_static_module ...

# 进入 nginx-1.16.1 目录
cd /usr/local/nginx-1.16.1

# 重新执行 cofigure 命令，增加 ssl 模块的配置
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

# 编译（不安装）
make

# 备份原来的 nginx 命令
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx-bak

# 替换原来的 nginx 命令
cp /usr/local/nginx-1.16.1/objs/nginx /usr/local/nginx/sbin/nginx
```

## 2 生成证书

```
# 创建存放证书的目录
mkdir /usr/local/ssl

cd /usr/local/ssl

# 创建服务器私钥，命令会让你输入一个口令。
openssl genrsa -des3 -out server.key 1024
# 再生成一个不带密码的（非必须）
# openssl rsa -in server.key -out server-nopassword.key

# 创建签名请求的证书（CSR）
openssl req -new -key server.key -out server.csr

# 标记证书使用上述私钥和 CSR
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
# 再生成一个不带密码的（非必须）
# openssl x509 -req -days 365 -in server.csr -signkey server-nopassword.key -out server-nopassword.crt
```

## 3 配置 Nginx

修改配置文件，开启 ssl ，并指定标记的证书和私钥。

```bash
server {
    listen       80;
    server_name  localhost;
    
    listen 443 ssl;
    ssl_certificate     /usr/local/ssl/server-nopassword.crt;
    ssl_certificate_key /usr/local/ssl/server-nopassword.key;
    #rewrite ^(.*)$ https://$host$1 permanent;

    #...
}
```

重启 Nginx ，此时 http 和 https 都支持。

---

- [Nginx 教程合集](https://mp.weixin.qq.com/s/TdLki2vnjW4hKUz_BgzEHg)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)