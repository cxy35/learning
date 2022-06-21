手把手带你在 Linux 上安装 `jdk` 。
<!-- more -->

## 1 准备工作

- 访问 [https://www.oracle.com/java/technologies/downloads/](https://www.oracle.com/java/technologies/downloads/) 下载对应**稳定版**的安装包，如：`jdk-8u321-linux-x64.tar.gz`。
- 上传到服务器的 `/usr/local/mydata/temp` 目录下。如果没有，则手动新建。

## 2 安装

### 2.1 解压

```bash
# 其中 jdk-8u321-linux-x64.tar.gz 换成实际的名称
cd /usr/local/mydata/temp
tar -xvzf jdk-8u321-linux-x64.tar.gz -C /usr/local/mydata/soft
```

### 2.2 重命名

```bash
# 其中 jdk1.8.0_321 换成实际的名称
cd /usr/local/mydata/soft
mv jdk1.8.0_321 jdk
```

### 2.3 配置环境变量

执行：`vi /etc/profile` 修改配置。

```bash
# 在文件末尾添加下面内容并保存成功
JAVA_HOME=/usr/local/mydata/soft/jdk
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=$CLASSPATH:$JAVA_HOME/lib
export JAVA_HOME PATH CLASSPATH
```

执行：`source /etc/profile` 使配置生效。

### 2.4 验证

执行：`java -version`

```bash
# 显示如下信息表示验证通过
java version "1.8.0_321"
Java(TM) SE Runtime Environment (build 1.8.0_321-b07)
Java HotSpot(TM) 64-Bit Server VM (build 25.321-b07, mixed mode)
```


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)