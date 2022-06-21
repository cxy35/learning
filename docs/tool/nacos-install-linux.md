手把手带你在 Linux 上安装 `Nacos` 。
<!-- more -->

## 1 准备工作

- 访问 [https://github.com/alibaba/nacos/releases](https://github.com/alibaba/nacos/releases) 下载对应**稳定版**的安装包，如：`nacos-server-2.1.0.tar.gz`。
- 上传到服务器的 `/usr/local/mydata/temp` 目录下。如果没有，则手动新建。

## 2 安装

### 2.1 解压

```bash
# 其中 nacos-server-2.1.0.tar.gz 换成实际的名称
cd /usr/local/mydata/temp
tar -xvzf nacos-server-2.1.0.tar.gz -C /usr/local/mydata/soft
```

### 2.2 启动

```bash
# 切换到 Nacos 安装目录
cd /usr/local/mydata/soft/nacos
# standalone 代表单机模式运行，非集群模式
sh /usr/local/mydata/soft/nacos/bin/startup.sh -m standalone
# ps -ef|grep nacos
```

### 2.4 验证

Nacos 启动成功后，访问 [http://127.0.0.1:8848/nacos](http://127.0.0.1:8848/nacos) 就能看到后台页面。如果有登录页面，登录的默认用户名/密码都是 `nacos` 。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)