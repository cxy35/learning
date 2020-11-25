---
title: Hexo 搭建个人博客网站
date: 2019-09-19 15:36:20
categories: Hexo
tags: [Hexo, 博客]
toc: true
thumbnail: /images/hexo/hexo-thumbnail.jpg
---
你的门面你做主，手把手带你使用 Hexo 搭建个人博客网站。
<!-- more -->

## 1 什么是 Hexo？

[Hexo](https://hexo.io/zh-cn/) 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。详见 [文档](https://hexo.io/zh-cn/docs/)、[主题](https://hexo.io/themes/)。

## 2 安装

参考 [官方文档](https://hexo.io/zh-cn/docs/)

### 2.1 安装前提

安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：

- [Node.js](http://nodejs.org/) (Node.js 版本需不低于 8.6，建议使用 Node.js 10.0 及以上版本)
- [Git](http://git-scm.com/)

### 2.2 安装 Git

- Windows：下载并安装 [git](https://git-scm.com/download/win)，如 Git-2.23.0-64-bit.exe

	> 从上面的链接下载git for windows最好挂上一个代理，否则下载速度十分缓慢。也可以参考这个页面，收录了存储于百度云的下载地址。

- Linux (Ubuntu, Debian)：`sudo apt-get install git-core`
- Linux (Fedora, Red Hat, CentOS)：`sudo yum install git-core`

### 2.3 安装 Node.js

- Windows：对于windows用户来说，建议使用 [安装程序](http://nodejs.org/) 进行安装，安装时，请勾选**Add to PATH**选项。

	> 另外，您也可以使用**Git Bash**，这是git for windows自带的一组程序，提供了Linux风格的shell，在该环境下，您可以直接用下面的命令来安装Node.js。打开它的方法很简单，在任意位置单击右键，选择“Git Bash Here”即可。由于Hexo的很多操作都涉及到命令行，您可以考虑始终使用**Git Bash**来进行操作。
	
- Linux：安装 Node.js 的最佳方式是使用 [nvm](https://github.com/nvm-sh/nvm)。nvm 的开发者提供了一个自动安装 nvm 的简单脚本：
	- curl：`$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | sh`
	- wget：`$ wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | sh`
	- 安装完成后，重启终端并执行下列命令即可安装 Node.js：`$ nvm install stable`

### 2.4 安装 Hexo

所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。

```bash
$ npm install -g hexo-cli
```

## 3 建站

参考 [官方文档](https://hexo.io/zh-cn/docs/setup)

### 3.1 建站

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

```bash
$ hexo init blog
$ cd blog
$ npm install
```

新建完成后，指定文件夹的目录如下：

```bash
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

- _config.yml：网站的 [配置](https://hexo.io/zh-cn/docs/configuration) 信息，如网站的 title，描述，关键字、图标等。也称做 **`站点配置文件`** ，注意区分 **`主题配置文件`**。
- package.json：应用程序的信息。EJS, Stylus 和 Markdown renderer 已默认安装，您可以自由移除。
- scaffolds：[模版](https://hexo.io/zh-cn/docs/writing) 文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。Hexo的模板是指在新建的文章文件中默认填充的内容。例如，如果您修改scaffold/post.md中的Front-matter内容，那么每次新建一篇文章时都会包含这个修改。
- source：资源文件夹是存放用户资源的地方。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。
- themes：[主题](https://hexo.io/zh-cn/docs/themes) 文件夹。Hexo 会根据主题来生成静态页面。

配置完成后，定位到 blog 目录，执行 `hexo s` 就可以在本地启动项目了，启动成功后，浏览器中输入 [http://127.0.0.1:4000](http://127.0.0.1:4000) 就可以看到网站了。

### 3.2 [命令](https://hexo.io/zh-cn/docs/commands)

|命令|简写|中文含义|
|:-|:-:|:-|
|hexo server|hexo s|本地启动|
|hexo generate|hexo g|生成静态文件|
|hexo deploy|hexo d|部署网站|
|hexo clean||清除缓存和已经生成的静态文件|

### 3.3 [配置](https://hexo.io/zh-cn/docs/configuration)

在 Hexo 中有两份主要的配置文件，其名称都是 `_config.yml`。 其中，一份位于站点根目录下，主要包含 Hexo 本身的配置；另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。

为了描述方便，在以下说明中，将前者称为 **`站点配置文件`**， 后者称为 **`主题配置文件`**。

Hexo 默认使用的主题是 landscape，对应 ./themes 目录下的 landscape 文件夹。可切换成其他 [主题](https://hexo.io/themes/)，选好之后，先将对应的主题 clone 到或下载好拷贝到 ./themes 目录下，再启用即可。建议使用 clone ，使用 clone ，假如有一天这个主题更新了，只需要 pull 一下就可以获取到最新样式了。

#### [网站](https://hexo.io/zh-cn/docs/configuration#%E7%BD%91%E7%AB%99)

编辑 **`站点配置文件`**， 设置站点标题 `title` 、子标题 `subtitle` 、描述 `description` 、关键字 `keywords` 、作者 `author` 、语言 `language` 等。

```yaml
# Site
title: 微博客
subtitle: 微博客官方网站上线了
description: 微博客官方网站上线了，微信/微博同步上线，欢迎订阅
keywords: 微博客,博客
author: 微博客
language: zh-Hans
timezone:
```

language 支持的语言：zh-Hans / zh-CN / zh-hk / zh-tw / en / ja / ......，取决于你的主题目录下的 languages 中要有 zh-Hans.yml / zh-CN.yml / ...... 。

#### [文章](https://hexo.io/zh-cn/docs/configuration#%E6%96%87%E7%AB%A0)

```bash
# 安装插件
npm install --save hexo-filter-auto-spacing
```

编辑 **`站点配置文件`**，增加这项配置：`auto_spacing: true`，默认为 false。

```yaml
# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
auto_spacing: true
titlecase: false # Transform title into titlecase
```

#### [分页](https://hexo.io/zh-cn/docs/configuration#%E5%88%86%E9%A1%B5)

在 Hexo 里可以为首页和归档页面设置不同的文章篇数，但可能需要安装 Hexo 插件。详细步骤如下。

1. 使用 npm install --save 命令来安装需要的 Hexo 插件。

```bash
npm install --save hexo-generator-index
npm install --save hexo-generator-archive
npm install --save hexo-generator-tag
```

2. 等待扩展全部安装完成后，在 **`站点配置文件`** 中，设定如下选项（per_page 即文章的数量）：

```yaml
# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date

archive_generator:
  per_page: 10
  yearly: true
  monthly: true

tag_generator:
  per_page: 10
```

#### Hexo 主题配置之 Next

具体配置见：Hexo 主题配置 - NexT（微信左下方**阅读全文**可直达 **Hexo 教程合集**）。

#### Hexo 主题配置之 Icarus

具体配置见：Hexo 主题配置 - Icarus（微信左下方**阅读全文**可直达 **Hexo 教程合集**）。

### 3.4 [写作](https://hexo.io/zh-cn/docs/writing)

**【重要】注意：请求地址 url 中的文件名不能有空格（标题没影响），可以用 - 隔开，否则会出问题，比如使用评论插件 `Gitment` 时会导致登陆时跳到首页。**

---

> [增加博客后台管理功能](https://mp.weixin.qq.com/s/vrDCnSHL7YBmN9PBxqVyjQ) 之后也可在后台写作，[http://127.0.0.1:4000/admin](http://127.0.0.1:4000/admin)

你可以执行下列命令来创建一篇新文章或者新的页面。

```bash
$ hexo new [layout] <title>
```

您可以在命令中指定文章的布局（layout），默认为 **`post`**，可以通过修改 `_config.yml` 中的 `default_layout` 参数来指定默认布局。

- 新建页面：`hexo new page about`
- 新建文章：`hexo new 文章标题` = `hexo new post 文章标题`

#### 布局（Layout）

Hexo 有三种默认布局：`post`、`page` 和 `draft`。在创建者三种不同类型的文件时，它们将会被保存到不同的路径；而您自定义的其他布局和 `post` 相同，都将储存到 `source/_posts` 文件夹。

|布局|路径|
|:-|:-|
|post|source/_posts|
|page|source|
|draft|source/_drafts|

> 如果你不想你的文章被处理，你可以将 Front-Matter 中的`layout`: 设为 `false` 。

#### 文件名称

Hexo 默认以标题做为文件名称，但您可编辑 `new_post_name` 参数来改变默认的文件名称，举例来说，设为 :`year-:month-:day-:title.md` 可让您更方便的通过日期来管理文章。

|变量|描述|
|:-|:-|
|:title|标题（小写，空格将会被替换为短杠）|
|:year|建立的年份，比如， 2015|
|:month|建立的月份（有前导零），比如， 04|
|:i_month|建立的月份（无前导零），比如， 4|
|:day|建立的日期（有前导零），比如， 07|
|:i_day|建立的日期（无前导零），比如， 7|

#### 草稿

刚刚提到了 Hexo 的一种特殊布局：`draft`，这种布局在建立时会被保存到 `source/_drafts` 文件夹，您可通过 `publish` 命令将草稿移动到 `source/_posts` 文件夹，该命令的使用方式与 `new` 十分类似，您也可在命令中指定 `layout` 来指定布局。

```bash
$ hexo publish [layout] <title>
```

草稿默认不会显示在页面中，您可在执行时加上 `--draft` 参数，或是把 `render_drafts` 参数设为 `true` 来预览草稿。

#### 模版（Scaffold）

在新建文章时，Hexo 会根据 `scaffolds` 文件夹内相对应的文件来建立文件，例如：

```bash
$ hexo new photo "My Gallery"
```

在执行这行指令时，Hexo 会尝试在 `scaffolds` 文件夹中寻找 `photo.md`，并根据其内容建立文章，以下是您可以在模版中使用的变量：

|变量|描述|
|:-|:-|
|layout|布局|
|title|标题|
|date|文件建立日期|

## 4 绑定到 GitHub

大家可能已经迫不及待想要把博客上传到 GitHub 了，绑定到 Github 步骤也很简单，首先以 自己的GitHub ID.github.io 为名创建一个 public 仓库，例如我的 ID 为 abc，创建的仓库为 abc.github.io。

可能需要安装git开发者插件：

```bash
npm install --save hexo-deployer-git
```

创建成功之后，编辑 **`站点配置文件`**，配置 GitHub 地址，如下：

```yaml
deploy:
  type: git
  repo: git@github.com:abc/abc.github.io.git
  branch: master
```

这里根据自己的地址来配置即可，配置完成后，执行如下命令：

```bash
hexo g
hexo d
```

执行完成后，就可以将数据上传到 GitHub 了（当然这里需要大家提前配置一下 GitHub 的公钥，具体可以参考 [Git 配置 SSH keys](https://mp.weixin.qq.com/s/bmGs3e3tI-_jYiEz9uQA9g)）。

上传成功后，访问 https://cxy35.github.io 就可以看到自己的个人站点了。

如果你对 GitHub 提供的域名不满意，也可以自己申请一个域名，分分钟就配置好了。

## 5 绑定到域名

域名申请成功之后，接下来的配置，也分为两部分。

### 5.1 GitHub 配置

首先在博客所在目录下的 source 目录中，创建一个 CNAME 文件，文件内容就是你的域名，如下：www.cxy35.com。

然后执行 `hexo d` 命令将这个文件上传到 GitHub就可以了。

### 5.2 域名解析配置

添加两条 A 记录，指向 GitHub 的 IP 地址，再添加一条 CNAME ，指向你的 GitHub 域名就可以了。

![](https://oscimg.oschina.net/oscnet/up-73c650e05f1b6119e5bd7ba5b7022cfc739.png)

配置成功后，访问 https://www.cxy35.com 就可以看到自己的个人站点了。

---

- [Hexo 教程合集](https://mp.weixin.qq.com/s/UIlKCAMJsV1B0tfT0Bvklg)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)