---
title: Nginx 负载均衡 - fair
date: 2020-03-05 17:10:14
categories: Nginx
tags: [Nginx, fair]
toc: true
---
学习在 Nginx 中使用 fair 模块（第三方）来实现负载均衡，fair 采用的不是内建负载均衡使用的轮换的均衡算法，而是**可以根据页面大小、响应时间智能的进行负载均衡**。
<!-- more -->

## 1 准备工作

- nginx-upstream-fair 官方下载地址：[https://github.com/gnosek/nginx-upstream-fair](https://github.com/gnosek/nginx-upstream-fair)
- 版本问题：如果使用的 Nginx 版本 >= 1.14.0 时，使用上述模块源码编译时会报错误，需要对源码做一些修改，参考：[https://github.com/gnosek/nginx-upstream-fair/pull/27/commits/ff979a48a0ccb9217437021b5eb9378448c2bd9e](https://github.com/gnosek/nginx-upstream-fair/pull/27/commits/ff979a48a0ccb9217437021b5eb9378448c2bd9e) 。也可以直接下载已经修改好的源码包：[https://files.cnblogs.com/files/ztlsir/nginx-upstream-fair-master.zip](https://files.cnblogs.com/files/ztlsir/nginx-upstream-fair-master.zip)

## 2 配置

- 上传 `nginx-upstream-fair-master.zip` 。
- 解压到 `/usr/local` 目录下。

```bash
unzip nginx-upstream-fair-master.zip
```

1. 未安装过 Nginx

具体安装步骤参考： [Nginx 安装 - Linux](https://mp.weixin.qq.com/s/UypOmZsfZmiAz3_FTk3z7Q)

```bash
# 只需要在 ./configure 时额外增加 fair 模块
--add-module=/usr/local/nginx-upstream-fair-master
```

2. 已安装过 Nginx

如果已经安装过 Nginx ，又不想重新安装，则可以单独添加 fair 模块。

```bash
# 关闭 Nginx
/usr/local/nginx/sbin/nginx -s stop

# 查看 Nginx 安装时的配置参数，复制备用
/usr/local/nginx/sbin/nginx -V
# configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module ...

# 进入 nginx-1.16.1 目录
cd /usr/local/nginx-1.16.1

# 重新执行 cofigure 命令，增加 fair 模块的配置
./configure --prefix=/usr/local/nginx --error-log-path=/usr/local/nginx/logs/error.log --http-log-path=/usr/local/nginx/logs/access.log --pid-path=/usr/local/nginx/logs/nginx.pid --lock-path=/usr/local/nginx/logs/nginx.lock --http-client-body-temp-path=/usr/local/nginx/temp/client-body --http-proxy-temp-path=/usr/local/nginx/temp/proxy --http-fastcgi-temp-path=/usr/local/nginx/temp/fastcgi --http-uwsgi-temp-path=/usr/local/nginx/temp/uwsgi --http-scgi-temp-path=/usr/local/nginx/temp/scgi --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-file-aio --with-http_realip_module --add-module=/usr/local/nginx-upstream-fair-master

# 编译（不安装）
make

# 备份原来的 nginx 命令
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx-bak

# 替换原来的 nginx 命令
cp /usr/local/nginx-1.16.1/objs/nginx /usr/local/nginx/sbin/nginx
```

修改配置文件 `nginx.conf` ，如下：

```bash
upstream tomcat_test {
	fair;
	server 192.168.71.57:8080;
    server 192.168.71.57:8081;
}
```

最后启动 Nginx 服务，验证。**本地采用 tomcat 里面 sleep 的方式测试，结果不对，奇怪！？**。

---

- [Nginx 教程合集](https://mp.weixin.qq.com/s/TdLki2vnjW4hKUz_BgzEHg)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)