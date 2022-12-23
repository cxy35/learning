手把手带你在 Linux 上安装 `RabbitMQ` 。
<!-- more -->

## 1 准备工作

### 1.1 依赖软件和版本说明

- `RabbitMQ` 需要 `Erlang` 和 `Socat` 环境，`RabbitMQ` 版本和 `Erlang` 版本兼容性关系可参考 [https://www.rabbitmq.com/which-erlang.html](https://www.rabbitmq.com/which-erlang.html) 。
- **特别注意操作系统版本**，可以执行 `cat /etc/centos-release` 查看，`CentOS 6.x` 对应 `el6` 的安装包版本，`CentOS 7.x` 对应 `el7` 的安装包版本，`CentOS 8.x` 对应 `el8` 的安装包版本。

### 1.2 下载安装包

- 访问 [https://github.com/rabbitmq/erlang-rpm/releases](https://github.com/rabbitmq/erlang-rpm/releases) 下载 `Erlang` 对应**稳定版**的安装包，如：`erlang-23.3.4.11-1.el7.x86_64.rpm`。
- 访问 [http://www.rpmfind.net/linux/rpm2html/search.php?query=socat(x86-64)](http://www.rpmfind.net/linux/rpm2html/search.php?query=socat(x86-64)) 下载 `Socat` 对应**稳定版**的安装包，如：`socat-1.7.4.3-2.3.x86_64.rpm`。
- 访问 [https://github.com/rabbitmq/rabbitmq-server/releases](https://github.com/rabbitmq/rabbitmq-server/releases) 下载 `RabbitMQ` 对应**稳定版**的安装包，如：`rabbitmq-server-3.9.16-1.el7.noarch.rpm`。
- 上传到服务器的 `/usr/local/mydata/temp` 目录下。如果没有，则手动新建。

### 1.3 安装相关依赖环境

```bash
yum install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz
```

## 2 安装

### 2.1 安装 Erlang

```bash
# 安装，其中 erlang-23.3.4.11-1.el7.x86_64.rpm 换成实际的名称
cd /usr/local/mydata/temp
rpm -ivh erlang-23.3.4.11-1.el7.x86_64.rpm

# 验证
erl -v
# Erlang/OTP 23 [erts-11.2.2.10] [source] [64-bit] [smp:4:4] [ds:4:4:10] [async-threads:1] [hipe]
# 
# Eshell V11.2.2.10  (abort with ^G)

rpm -qa|grep -i erlang
# erlang-23.3.4.11-1.el7.x86_64
```

### 2.2 安装 Socat

```bash
# 安装，其中 socat-1.7.3.2-2.el7.x86_64.rpm 换成实际的名称
cd /usr/local/mydata/temp
rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm

# 验证
rpm -qa|grep -i socat
# socat-1.7.3.2-2.el7.x86_64
```

> 如果报 `错误：依赖检测失败` 安装对应的依赖之后，重新安装即可。

### 2.3 安装 RabbitMQ

```bash
# 安装，其中 rabbitmq-server-3.9.16-1.el7.noarch.rpm 换成实际的名称
cd /usr/local/mydata/temp
rpm -ivh rabbitmq-server-3.9.16-1.el7.noarch.rpm

# 验证
rpm -qa|grep -i rabbitmq
# rabbitmq-server-3.9.16-1.el7.noarch
```

## 3 配置

### 3.1 开启管理界面

执行：`rabbitmq-plugins enable rabbitmq_management` 开启管理界面。

### 3.2 用户相关操作

`RabbitMQ` 默认的用户 `guest/guest` 只能在本机访问，所以需要新增一个远程登录的用户。

执行：`systemctl start rabbitmq-server` 启动服务。

```bash
# 新增用户
rabbitmqctl add_user admin 123456

# 设置用户分配操作权限
rabbitmqctl set_user_tags admin administrator
# administrator：可以登录控制台、查看所有信息、可以对 rabbitmq 进行管理
# monitoring：监控者 登录控制台，查看所有信息
# policymaker：策略制定者 登录控制台，指定策略
# managment：普通管理员 登录控制台

# 为用户添加资源权限
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"

# ---------------

# 修改用户密码
rabbitmqctl change_password admin 654321
# 查询用户列表
rabbitmqctl list_users
# 删除用户
rabbitmqctl delete_user admin
```

## 4 启动停止重启

```bash
# 启动服务
systemctl start rabbitmq-server
# 停止服务
systemctl stop rabbitmq-server
# 重启服务
systemctl restart rabbitmq-server
# 查看服务状态
systemctl status rabbitmq-server
# 开机自启动
systemctl enable rabbitmq-server
```

启动成功后，访问 [http://127.0.0.1:15672/](http://127.0.0.1:15672/)，可以看到登录页面，如果无法访问，排查下防火墙端口是否开放。

## 5 卸载

```bash
# 停止服务
systemctl stop rabbitmq-server
# 查看安装的服务
rpm -qa|grep -i rabbitmq
# rabbitmq-server-3.9.16-1.el7.noarch
# 卸载，名称以实际为准
rpm -e rabbitmq-server-3.9.16-1.el7.noarch --nodeps

# 查看安装的服务
rpm -qa|grep -i erlang
# erlang-23.3.4.11-1.el7.x86_64
# 卸载，名称以实际为准
rpm -e erlang-23.3.4.11-1.el7.x86_64 --nodeps


# 查看安装的服务
rpm -qa|grep -i socat
# socat-1.7.3.2-2.el7.x86_64
# 卸载，名称以实际为准
rpm -e socat-1.7.3.2-2.el7.x86_64 --nodeps

# 查询并删除相关文件，名称和目录以实际为准
find / -name rabbitmq
find / -name erlang
find / -name socat
rm -rf /run/rabbitmq /etc/rabbitmq /var/lib/rabbitmq /var/log/rabbitmq /usr/lib/rabbitmq /usr/lib64/erlang /usr/share/java/erlang /usr/bin/socat
```


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.top](https://cxy35.top)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)