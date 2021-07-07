手把手带你安装 Kibana。Kibana 是一个开源分析和可视化平台，旨在与 Elasticsearch 协同工作。 您使用 Kibana 搜索，查看和与存储在 Elasticsearch 索引中的数据进行交互。 您可以轻松执行高级数据分析，并在各种图表，表格和地图中可视化您的数据。
<!-- more -->

## 1 准备工作

> 建议 Kibana 的版本号与 Elasticsearch 保持一致。

从 [https://www.elastic.co/downloads/kibana](https://www.elastic.co/downloads/kibana) 中下载最新稳定版本的安装文件，如：`https://artifacts.elastic.co/downloads/kibana/kibana-7.13.1-linux-x86_64.tar.gz`，历史版本可以访问：[https://www.elastic.co/downloads/kibana](https://www.elastic.co/downloads/kibana)。如果是在 `Linux` 下安装，可以直接使用 `wget` 命令。

![](https://oscimg.oschina.net/oscnet/up-cad4c10b2645221aeac0fe9371445312eb7.png)

另外，如果想要更快速的下载（但版本不一定是最新的），可以访问：[https://elasticsearch.cn/download/](https://elasticsearch.cn/download/)，里面包含 `Elastic` 技术栈所需所有安装文件的下载。

## 2 安装

```bash
cd /usr/local/elastic
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.13.1-linux-x86_64.tar.gz
tar -xvzf kibana-7.13.1-linux-x86_64.tar.gz
cd kibana-7.13.1-linux-x86_64
```

---

文件目录布局：

|类型|描述|默认地址|设置|
|:-|:-|:-|:-|
|home|Kibana主目录或$KIBANA_HOME|通过解压缩归档创建的目录||
|bin|二进制脚本包括用于启动Kibana服务器的kibana和用于安装插件的kibana-plugin|$KIBANA_HOME\bin|| 
|config|配置文件包括kibana.yml|$KIBANA_HOME\config|| 
|data|Kibana及其插件写入磁盘的数据文件的位置|$KIBANA_HOME\data|| 
|optimize|透明的源代码。 某些管理操作（例如，插件安装）导致源代码在运行中被重新传输。|$KIBANA_HOME\optimize|| 
|plugins|插件文件位置。 每个插件都将包含在一个子目录中。|$KIBANA_HOME\plugins||

默认情况下，所有文件和目录都包含在 $KIBANA_HOME 中 - 解压缩归档时创建的目录。但是，建议更改配置和数据目录的默认位置，以便以后不删除重要数据。

## 3 配置

修改配置文件 `./config/kibana.yml`，常用配置如下：

```yml
#配置端口，默认5601
#http.port: 5601

#配置允许访问的IP地址，默认只允许本机访问
server.host: "0.0.0.0"

#配置 Elasticsearch 的地址
elasticsearch.hosts: ["http://localhost:9200"]

#配置elasticsearch认证信息（如果 Elasticsearch 有启用），用户名和密码在 Elasticsearch 启用安全认证时设置过
elasticsearch.username: "kibana_system"
elasticsearch.password: "123456"

#配置语言为中文
#i18n.locale: "zh-CN"
```

另外，也可以在启动时配置 Kibana，见下文。

## 4 启动

首先在 Linux 系统中新建 elastic 用户，用于启动 kibana，否则会报错：

```bash
groupadd elastic
useradd elastic -g elastic
#passwd es

cd /usr/local/elastic
chown -R elastic:elastic kibana-7.13.1-linux-x86_64

su elastic
```

```bash
cd /usr/local/elastic/kibana-7.13.1-linux-x86_64

#前台运行
# ./bin/kibana

#后台运行
nohup ./bin/kibana &
```

浏览器访问 [http://localhost:5601](http://localhost:5601) 验证是否启动成功。

---

默认情况下，`Kibana` 从 `$KIBANA_HOME/config/kibana.yml` 文件加载其配置。也可以在命令行中指定配置文件中指定的任何配置，它们将覆盖配置文件中的配置。
```bash
#启动时同时指定多个参数
# ./bin/kibana --elasticsearch.hosts="http://localhost:9200" --elasticsearch.username=kibana --elasticsearch.password=123456
```

我们可以通过如下的命令来查看可以配置的参数：

```bash
./bin/kibana -h
 
 # Usage: bin/kibana [command=serve] [options]
 # 
 # Kibana is an open source (Apache Licensed), browser based analytics and search dashboard for Elasticsearch.
 # 
 # Commands:
 #   serve  [options]  Run the kibana server
 #   help  <command>   Get the help for a specific command
 # 
 # "serve" Options:
 # 
 #   -e, --elasticsearch <uri1,uri2>  Elasticsearch instances
 #   -c, --config <path>              Path to the config file, use multiple --config args to include multiple config files (default: ["/Users/liuxg/elastic#/kibana-7.8.0-darwin-x86_64/config/kibana.yml"])
 #   -p, --port <port>                The port to bind to
 #   -q, --quiet                      Prevent all logging except errors
 #   -Q, --silent                     Prevent all logging
 #   --verbose                        Turns on verbose logging
 #   -H, --host <host>                The host to bind to
 #   -l, --log-file <path>            The file to log to
 #   --plugin-dir <path>              A path to scan for plugins, this can be specified multiple times to specify multiple directories (default: ["/Users/liuxg#/elastic/kibana-7.8.0-darwin-x86_64/plugins","/Users/liuxg/elastic/kibana-7.8.0-darwin-x86_64/src/legacy/core_plugins"])
 #   --plugin-path <path>             A path to a plugin which should be included by the server, this can be specified multiple times to specify multiple paths# (default: [])
 #   --plugins <path>                 an alias for --plugin-dir
 #   --optimize                       Run the legacy plugin optimizer and then stop the server
 #   -h, --help                       output usage information
```

## 5 关闭

```bash
ps -ef|grep kibana
kill -9 xxx
```

## 6 启用安全认证

启用 Elasticsearch 安全功能后，用户必须使用有效的用户 ID 和密码登录 Kibana，有三种方式配置。

- 如果您不介意在配置文件中显示密码，修改配置文件 `./config/kibana.yml`，增加如下配置：

```yml
#配置elasticsearch认证信息（如果 Elasticsearch 有启用），用户名和密码在 Elasticsearch 启用安全认证时设置过
elasticsearch.username: "kibana_system"
elasticsearch.password: "123456"
```

- 如果您不想将你的用户 ID 和密码放在 kibana.yml 文件中，请将它们存储在密钥库中。运行以下命令以创建 Kibana 密钥库并添加安全设置：

```bash
./bin/kibana-keystore create
./bin/kibana-keystore add elasticsearch.username
./bin/kibana-keystore add elasticsearch.password
```

出现提示时，请为这些设置值指定 kibana 内置用户及其密码。启动 Kibana 时会自动应用这些设置。

- 你也可以在启动 Kibana 带有参数，比如：

```bash
./bin/kibana --elasticsearch.hosts="http://localhost:9200" --elasticsearch.username=kibana --elasticsearch.password=123456
```

## 7 报错汇总

- 报错描述：`./bin/../node/bin/node: /lib64/libc.so.6: version GLIBC_2.17 not found (required by ./bin/../node/bin/node)`。
- 报错原因：程序运行时候，没有找到 GLIBC_2.17 这个版本库，可以用 `strings /lib64/libc.so.6 |grep GLIBC_` 查看目前支持的 glibc 版本。
- 报错解决：更新系统 glibc 库（如果缺少多个版本，安装最新版本即可）：

```bash
wget http://ftp.gnu.org/gnu/glibc/glibc-2.17.tar.gz 
tar -xvf  glibc-2.17.tar.gz 
mkdir glibc-2.17/build
cd glibc-2.17/build 
../configure  --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
make
make install
```

---

- 报错描述：`./bin/../node/bin/node: /usr/lib64/libstdc++.so.6: version 'GLIBCXX_3.4.18' not found (required by ./bin/../node/bin/node)`。
- 报错原因：程序运行时候，没有找到 GLIBCXX_3.4.18 这个版本库，可以用 `strings /usr/lib64/libstdc++.so.6 | grep GLIBCXX` 查看目前支持的 libstdc++ 版本。
- 报错解决：安装最新版本：

```bash
TODO
```

---

- [Kibana：如何在 Linux，MacOS 及 Windows 上安装 Elastic 栈中的 Kibana](https://elasticstack.blog.csdn.net/article/details/99433732)
- [Elasticsearch：设置 Elastic 账户安全](https://elasticstack.blog.csdn.net/article/details/100548174)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)