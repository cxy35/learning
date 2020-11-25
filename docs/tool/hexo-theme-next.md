---
title: Hexo 主题配置 - NexT
date: 2019-09-20 11:55:49
categories: Hexo
tags: [Hexo, NexT]
toc: true
# thumbnail: /images/hexo/next-thumbnail.jpg
---
hexo-theme-next 主题配置大全，图文并茂，持续更新中。
<!-- more -->

## 1 安装 NexT 主题

参考 [hexo-theme-next 主题官网](https://theme-next.iissnan.com/)

### 1.1 下载 NexT 主题

建议你使用 克隆最新版本 的方式，之后的更新可以通过 git pull 来快速更新， 而不用再次下载压缩包替换。

```bash
cd blog
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

### 1.2 启用 NexT 主题

与所有 Hexo 主题启用的模式一样。 当 克隆/下载 完成后，打开 **`站点配置文件`**， 找到 `theme` 字段，并将其值更改为 `next`。

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

到此，NexT 主题安装完成。下一步我们将验证主题是否正确启用。在切换主题之后、验证之前， 我们最好使用 `hexo clean` 来清除 Hexo 的缓存。

## 2 配置 NexT 主题

### 设置站点信息

效果图：

![](https://oscimg.oschina.net/oscnet/4bfd9e9aebe88decd8ad2967d77408ef94f.jpg)

编辑 **`站点配置文件`**，具体配置参考 [Hexo 搭建个人博客网站](https://mp.weixin.qq.com/s/2Spv9KHO6mLHui0CFBmCnQ) 。

### 设置页面文章的篇数

编辑 **`站点配置文件`**，具体配置参考 [Hexo 搭建个人博客网站](https://mp.weixin.qq.com/s/2Spv9KHO6mLHui0CFBmCnQ) 。

### 主题配置文件

编辑 **`主题配置文件`**，具体配置参考 `themes/next/_config.yml`。

```yaml
# ......
```

### 选择 Scheme

Scheme 是 NexT 提供的一种特性，借助于 Scheme，NexT 为你提供多种不同的外观。同时，几乎所有的配置都可以 在 Scheme 之间共用。目前 NexT 支持三种 Scheme，他们是：

- Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
- Mist - Muse 的紧凑版本，整洁有序的单栏外观
- Pisces - 双栏 Scheme，小家碧玉似的清新
- Gemini

Scheme 的切换通过更改 **`主题配置文件`**，搜索 scheme 关键字，将你需用启用的 scheme 前面注释 # 去除即可。

```yaml
# Schemes
#scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```

### 设置菜单

效果图：

![](https://oscimg.oschina.net/oscnet/d46eb3d1749027f835600d106e5fef72055.jpg)

菜单配置包括三个部分，第一是菜单项（名称和链接），第二是菜单项的显示文本，第三是菜单项对应的图标。 NexT 使用的是 [Font Awesome](http://fontawesome.io/) 提供的图标， Font Awesome 提供了 600+ 的图标，可以满足绝大的多数的场景，同时无须担心在 Retina 屏幕下 图标模糊的问题。

编辑 **`主题配置文件`**，修改以下内容：

1. 设定菜单内容，对应的字段是 `menu`。 菜单内容的设置格式是：`item name: link`。其中 `item name` 是一个名称，这个名称并不直接显示在页面上，它将用于匹配图标以及翻译。

```yaml
# When running the site in a subdirectory (e.g. domain.tld/blog), remove the leading slash from link value (/archives -> archives).
# Usage: `Key: /link/ || icon`
# Key is the name of menu item. If translate for this menu will find in languages - this translate will be loaded; if not - Key name will be used. Key is case-senstive.
# Value before `||` delimeter is the target link.
# Value after `||` delimeter is the name of FontAwesome icon. If icon (with or without delimeter) is not specified, question icon will be loaded.
menu:
  home: / || home
  咖啡豆: /cherry/ || coffee
  咖啡粉: /powder/ || coffee
  categories: /categories/ || th
  archives: /archives/ || archive
  tags: /tags/ || tags
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
  about: /about/ || user
```

> 若你的站点运行在子目录中，请将链接前缀的 / 去掉。

NexT 默认的菜单项有（标注 ! 的项表示需要手动创建这个页面）：

|键值|设定值|显示文本（简体中文）|
|:-|:-|:-|
|home|home: /|主页|
|archives|archives: /archives|归档页|
|categories|categories: /categories|分类页 !|
|tags|tags: /tags|标签页 !|
|about|about: /about|关于页面 !|
|commonweal|commonweal: /404.html|公益 404 !|

2. 设置菜单项的显示文本。在第一步中设置的菜单的名称并不直接用于界面上的展示。Hexo 在生成的时候将使用 这个名称查找对应的语言翻译，并提取显示文本。这些翻译文本放置在 NexT 主题目录下的 `languages/{language}.yml` （`{language}` 为你所使用的语言）。

以简体中文为例，若你需要添加一个菜单项，比如 `something`。那么就需要修改简体中文对应的翻译文件 `languages/zh-Hans.yml`，在 `menu` 字段下添加一项：

```yaml
menu:
  home: 首页
  archives: 归档
  categories: 分类
  tags: 标签
  about: 关于
  search: 搜索
  commonweal: 公益404
  something: 有料
```

3. 设定菜单项的图标。enable 可用于控制是否显示图标，你可以设置成 false 来去掉图标。

```yaml
# Enable/Disable menu icons.
menu_icons:
  enable: true
```

> 在菜单图标开启的情况下，如果菜单项与菜单未匹配（没有设置或者无效的 Font Awesome 图标名字） 的情况下，NexT 将会使用 ? 作为图标。

> 请注意键值（如 home）的大小写要严格匹配。

### 设置侧栏

效果图：

![](https://oscimg.oschina.net/oscnet/106c0f4a46ce519c6592d3967e6234b7011.jpg)

默认情况下，侧栏仅在文章页面（拥有目录列表）时才显示，并放置于右侧位置。 可以通过修改 **`主题配置文件`** 中的 `sidebar` 字段来控制侧栏的行为。侧栏的设置包括两个部分，其一是侧栏的位置， 其二是侧栏显示的时机。

1. 设置侧栏的位置，修改 `sidebar.position` 的值，支持的选项有：

- left - 靠左放置
- right - 靠右放置

```yaml
sidebar:
  position: left
```

> 目前仅 Pisces Scheme 支持 `position` 配置。影响版本`5.0.0`及更低版本。

2. 设置侧栏显示的时机，修改 `sidebar.display` 的值，支持的选项有：

- `post` - 默认行为，在文章页面（拥有目录列表）时显示
- `always` - 在所有页面中都显示
- `hide` - 在所有页面中都隐藏（可以手动展开）
- `remove` - 完全移除

```yaml
sidebar:
  display: post
```

> 已知侧栏在 `use motion: false` 的情况下不会展示。 影响版本`5.0.0`及更低版本。

### 设置头像

效果图：

![](https://oscimg.oschina.net/oscnet/cb096a300be4d86677d89a16e7c8bb2161d.jpg)

编辑 **`主题配置文件`**， 修改字段 `avatar`， 值设置成头像的链接地址。其中，头像的链接地址可以是：

- 完整的互联网 URI，如：http://example.com/avatar.png
- 站点内的地址，主题或站点的source目录下。

```yaml
# Sidebar Avatar
# in theme directory(source/images): /images/avatar.gif
# in site  directory(source/uploads): /uploads/avatar.gif
avatar: /images/custom/avatar.jpg
```

### 设置 RSS

效果图：

![](https://oscimg.oschina.net/oscnet/961f256f4a8ed62d66aaea6271c7cceda19.jpg)

安装RSS插件: `npm install --save hexo-generator-feed`

编辑 **`站点配置文件`**，新增如下配置：

```yaml
####################### add #######################
# rss
plugins: hexo-generate-feed
```

编辑 **`主题配置文件`**，配置如下：

```yaml
# Set rss to false to disable feed link.
# Leave rss as empty to use site's feed link.
# Set rss to specific value if you have burned your feed already.
rss: /atom.xml
```

### 侧边栏社交链接

效果图：

![](https://oscimg.oschina.net/oscnet/9e7ff77ba7160075e66846cd7f204e92df7.jpg)

侧栏社交链接的修改包含两个部分，第一是链接，第二是链接图标。 两者配置均在 **`主题配置文件`** 中。

1. 链接放置在 social 字段下，一行一个链接。其键值格式是 显示文本: 链接地址。

```yaml
# Social Links.
# Usage: `Key: permalink || icon`
# Key is the link label showing to end users.
# Value before `||` delimeter is the target permalink.
# Value after `||` delimeter is the name of FontAwesome icon. If icon (with or without delimeter) is not specified, globe icon will be loaded.
social:
  微信: https://wx.qq.com/ || weixin
  微博: https://weibo.com/ || weibo
  GitHub: https://github.com/ || github
  #E-Mail: mailto:yourname@gmail.com || envelope
  #Google: https://plus.google.com/yourname || google
  Twitter: https://twitter.com/ || twitter
  #FB Page: https://www.facebook.com/yourname || facebook
  #VK Group: https://vk.com/yourname || vk
  #StackOverflow: https://stackoverflow.com/yourname || stack-overflow
  #YouTube: https://youtube.com/yourname || youtube
  #Instagram: https://instagram.com/yourname || instagram
  #Skype: skype:yourname?call|chat || skype
```

2. 设定链接的图标，对应的字段是 social_icons。其键值格式是 匹配键: Font Awesome 图标名称， 匹配键 与上一步所配置的链接的 显示文本 相同（大小写严格匹配），图标名称 是 Font Awesome 图标的名字（不必带 fa- 前缀）。 enable 选项用于控制是否显示图标，你可以设置成 false 来去掉图标。

```yaml
social_icons:
  enable: true
  icons_only: false
  transition: false
```

### 开启打赏功能

效果图：

![](https://oscimg.oschina.net/oscnet/9ba911cfb032efaee101a586b6c191a5a01.jpg)

越来越多的平台（微信公众平台，新浪微博，简书，百度打赏等）支持打赏功能，付费阅读时代越来越近，特此增加了打赏功能，支持微信打赏和支付宝打赏。 只需要 **`主题配置文件`** 中填入 微信 和 支付宝 收款二维码图片地址 即可开启该功能。

```yaml
# Reward
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
wechatpay: /images/custom/wechatpay.jpg
alipay: /images/custom/alipay.jpg
#bitcoin: /images/bitcoin.png
```

### 设置友情链接

效果图：

![](https://oscimg.oschina.net/oscnet/75148b4f2101cc9a61885739710d6dc4821.jpg)

编辑 **`主题配置文件`** ，配置如下：

```yaml
links_icon: link
links_title: 友情链接
#links_layout: block
links_layout: inline
links:
  Hexo: https://hexo.io/zh-cn/
  NexT: https://theme-next.iissnan.com/
  Hexo Admin: https://github.com/jaredly/hexo-admin
```

### 腾讯公益404页面

腾讯公益404页面，寻找丢失儿童，让大家一起关注此项公益事业！效果如下 http://www.ixirong.com/404.html

使用方法，新建 404.html 页面，放到主题的 `source` 目录下，内容如下：

```html
<!DOCTYPE HTML>
<html>
<head>
  <meta http-equiv="content-type" content="text/html;charset=utf-8;"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="robots" content="all" />
  <meta name="robots" content="index,follow"/>
  <link rel="stylesheet" type="text/css" href="https://qzone.qq.com/gy/404/style/404style.css">
</head>
<body>
  <script type="text/plain" src="http://www.qq.com/404/search_children.js"
          charset="utf-8" homePageUrl="/"
          homePageName="回到我的主页">
  </script>
  <script src="https://qzone.qq.com/gy/404/data.js" charset="utf-8"></script>
  <script src="https://qzone.qq.com/gy/404/page.js" charset="utf-8"></script>
</body>
</html>
```

### 站点建立时间

这个时间将在站点的底部显示，例如 `© 2015 - 2019`。 编辑 **`主题配置文件`**，配置如下：

```yaml
footer:
  # Specify the date when the site was setup.
  # If not defined, current year will be used.
  #since: 2015
```

### 订阅微信公众号

效果图：

![](https://oscimg.oschina.net/oscnet/be3e31376be237503ca4765d687357f48f6.jpg)

> 注意： 此特性在版本 5.0.1 中引入，要使用此功能请确保所使用的 NexT 版本在此之后

在每篇文章的末尾显示微信公众号二维码，扫一扫，轻松订阅博客。

在微信公众号平台下载您的二维码，并将它存放于主题 source/images/custom/ 目录下。

然后编辑 **`主题配置文件`**，配置如下：

```yaml
# Wechat Subscriber
wechat_subscriber:
  enabled: true
  qcode: /images/custom/wechat-qcode.jpg
  description: 欢迎您扫一扫上面的微信公众号，订阅我的博客！
```

### 设置动画效果

NexT 默认开启动画效果，效果使用 JavaScript 编写，因此需要等待 JavaScript 脚本完全加载完毕后才会显示内容。 如果您比较在乎速度，可以将设置此字段的值为 `false` 来关闭动画。

编辑 **`主题配置文件`**， 搜索 `use_motion`，根据您的需求设置值为 `true` 或者 `false` 即可：

```yaml
# Use velocity to animate everything.
motion:
  enable: true
  async: true
  transition:
    # Transition variants:
    # fadeIn | fadeOut | flipXIn | flipXOut | flipYIn | flipYOut | flipBounceXIn | flipBounceXOut | flipBounceYIn | flipBounceYOut
    # swoopIn | swoopOut | whirlIn | whirlOut | shrinkIn | shrinkOut | expandIn | expandOut
    # bounceIn | bounceOut | bounceUpIn | bounceUpOut | bounceDownIn | bounceDownOut | bounceLeftIn | bounceLeftOut | bounceRightIn | bounceRightOut
    # slideUpIn | slideUpOut | slideDownIn | slideDownOut | slideLeftIn | slideLeftOut | slideRightIn | slideRightOut
    # slideUpBigIn | slideUpBigOut | slideDownBigIn | slideDownBigOut | slideLeftBigIn | slideLeftBigOut | slideRightBigIn | slideRightBigOut
    # perspectiveUpIn | perspectiveUpOut | perspectiveDownIn | perspectiveDownOut | perspectiveLeftIn | perspectiveLeftOut | perspectiveRightIn | perspectiveRightOut
    post_block: fadeIn
    post_header: slideDownIn
    post_body: slideDownIn
    coll_header: slideLeftIn
    # Only for Pisces | Gemini.
    sidebar: slideUpIn
```

### 设置背景动画

> 注意： three_waves 在版本 5.1.1 中引入。只能同时开启一种背景动画效果。

编辑 **`主题配置文件`**，配置如下：

```yaml
# Canvas-nest
canvas_nest: false

# three_waves
three_waves: false

# canvas_lines
canvas_lines: false

# canvas_sphere
canvas_sphere: false

# Only fit scheme Pisces
# Canvas-ribbon
# size: The width of the ribbon.
# alpha: The transparency of the ribbon.
# zIndex: The display level of the ribbon.
canvas_ribbon:
  enable: false
  size: 300
  alpha: 0.6
  zIndex: -1
```

### 设置阅读全文

效果图：

![](https://oscimg.oschina.net/oscnet/494c4b6fcead87cd20a41c9c360107e1347.jpg)

在首页显示一篇文章的部分内容，并提供一个链接跳转到全文页面是一个常见的需求。 NexT 提供三种方式来控制文章在首页的显示方式。 也就是说，在首页显示文章的摘录并显示 **`阅读全文`** 按钮，可以通过以下方法：

- 在文章中使用 `<!-- more -->` 手动进行截断，Hexo 提供的方式 **推荐**。
- 在文章的 front-matter 中添加 description，并提供文章摘录
- 自动形成摘要，在 **`主题配置文件`** 中添加：

  ```yaml
  # Automatically Excerpt. Not recommend.
  # Please use <!-- more --> in the post to control excerpt accurately.
  auto_excerpt:
    enable: true
    length: 150
  ```

> 建议使用 `<!-- more -->`（即第一种方式），除了可以精确控制需要显示的摘录内容以外， 这种方式也可以让 Hexo 中的插件更好的识别。

### 设置字数统计/阅读时长

效果图：

![](https://oscimg.oschina.net/oscnet/be257fd5b1266d3b47edce9514f1720becf.jpg)

安装字数统计插件: `npm i --save hexo-wordcount`

编辑 **`主题配置文件`**，配置如下：

```yaml
# Post wordcount display settings
# Dependencies: https://github.com/willin/hexo-wordcount
post_wordcount:
  item_text: true
  wordcount: true
  min2read: true
  totalcount: false
  separated_meta: true
```

### 设置顶部加载条

效果图：

![](https://oscimg.oschina.net/oscnet/fa4ba2d19c59dacde9899355f317ffb2473.jpg)

编辑 **`主题配置文件`**，配置如下：

```yaml
# Progress bar in the top during page loading.
pace: true
# Themes list:
#pace-theme-big-counter
#pace-theme-bounce
#pace-theme-barber-shop
#pace-theme-center-atom
#pace-theme-center-circle
#pace-theme-center-radar
#pace-theme-center-simple
#pace-theme-corner-indicator
#pace-theme-fill-left
#pace-theme-flash
#pace-theme-loading-bar
#pace-theme-mac-osx
#pace-theme-minimal
# For example
# pace_theme: pace-theme-center-simple
pace_theme: pace-theme-minimal
```

### 设置底部内容

效果图：

![](https://oscimg.oschina.net/oscnet/577ce264b463d99890b30a631331953dfbb.jpg)

编辑 **`主题配置文件`**，配置如下：

```yaml
footer:
  # Specify the date when the site was setup.
  # If not defined, current year will be used.
  #since: 2015

  # Icon between year and copyright info.
  icon: user

  # If not defined, will be used `author` from Hexo main config.
  copyright:
  # -------------------------------------------------------------
  # Hexo link (Powered by Hexo).
  powered: false

  theme:
    # Theme & scheme info link (Theme - NexT.scheme).
    enable: false
    # Version info of NexT after scheme info (vX.X.X).
    version: false
  # -------------------------------------------------------------
  # Any custom text can be defined here.
  #custom_text: Hosted by <a target="_blank" href="https://pages.github.com">GitHub Pages</a>
```

### TODO-评论系统

### 数据统计与分析

#### 不蒜子统计

效果图：

![](https://oscimg.oschina.net/oscnet/4ff285e0f5ef27dd9d8d4e74a56fbfe6dd0.jpg)

![](https://oscimg.oschina.net/oscnet/41b0654e8e169b24873dd32b8b341008c78.jpg)

> 注意： 此特性在版本 5.0.1 中引入，要使用此功能请确保所使用的 NexT 版本在此之后

编辑 **`主题配置文件`**，配置如下：

```yaml
# Show PV/UV of the website/page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi/
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: <i class="fa fa-user"></i> 本站访客数
  site_uv_footer: 人次
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"></i> 本站总访问量
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: true
  page_pv_header: <i class="fa fa-file-o"></i> 本文总阅读量
  page_pv_footer: 次
```

### 内容分享服务

#### 百度分享

效果图：

![](https://oscimg.oschina.net/oscnet/4b1305580179ce94bb566a41f740235d8e3.jpg)

编辑 **`主题配置文件`**，配置如下：

```yaml
# Baidu Share
# Available value:
#    button | slide
# Warning: Baidu Share does not support https.
baidushare:
  type: slide
```

> type（=slide测试成功，=button测试未成功）

### 搜索服务

#### Local Search

效果图：

![](https://oscimg.oschina.net/oscnet/d39064267e9dbc87fcb1b166fa0d5f13422.jpg)

安装Local Search搜索插件: `npm install --save hexo-generator-searchdb`

编辑 **`站点配置文件`**，新增如下配置：

```yaml
####################### add #######################
# hexo-generator-searchdb
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

编辑 **`主题配置文件`**，配置如下：

```yaml
# Local search
# Dependencies: https://github.com/flashlab/hexo-generator-search
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1
```

### 增加最新文章模块

效果图：

![](https://oscimg.oschina.net/oscnet/d516773f83386138b2c96de3c78e72510fe.jpg)

编辑主题目录下的 `layout/_custom/sidebar.swig` 文件，新增如下内容：

```html
{# recent_posts #}
{% if theme.recent_posts %}
  <div class="links-of-blogroll motion-element {{ "links-of-blogroll-" + theme.recent_posts_layout | default('block') }}">
    <div class="links-of-blogroll-title">
      <i class="fa  fa-fw fa-{{ theme.recent_posts_icon | default('globe') | lower }}"></i>
      {{ theme.recent_posts_title }}
    </div>
    <ul class="links-of-blogroll-list">
      {% set posts = site.posts.sort('-date') %}
      {% for post in posts.slice('0', '5') %}
	<li class="links-of-blogroll-item">
	  <a href="{{ url_for(post.path) }}" title="{{ post.title }}" target="_blank">{{ post.title }}</a>
	</li>
      {% endfor %}
    </ul>
  </div>
{% endif %}
```

编辑 **`主题配置文件`**，新增如下配置：

```yaml
####################### add #######################
recent_posts: true
recent_posts_icon: history
recent_posts_title: 最新文章
recent_posts_layout: block
#recent_posts_layout: inline
```

### 增加博客后台管理功能

Hexo Admin 插件 GitHub 首页：[https://github.com/jaredly/hexo-admin](https://github.com/jaredly/hexo-admin)

演示 Demo：[https://jaredforsyth.com/hexo-admin/](https://jaredforsyth.com/hexo-admin/)

1. 安装 Hexo Admin 插件

```bash
npm install --save hexo-admin
```

2. 配置用户名密码

启动服务 `hexo server -d` 之后，访问 [http://127.0.0.1:4000/admin](http://127.0.0.1:4000/admin) ，按下列步骤完成配置：

![](https://oscimg.oschina.net/oscnet/df7058cd0303d70a2579a6f31b90f312654.jpg)

---

![](https://oscimg.oschina.net/oscnet/4a5e6e7abe2603c733e5e35ab170896e6b0.jpg)

> **注意：我们只能在本地进行后台管理，无法在自己的域名下进行后台管理。**

3. 编辑 **`站点配置文件`**，新增如下配置（从2中拷贝）：

```yaml
####################### add #######################
# hexo-admin authentification
admin:
  username: admin
  password_hash: $2a$10$d/U.IVXaAL86EWnsBQU2GuGgfbETQ6TK1UUgLsL3U5h4883tg3YyK
  secret: 1234567890abc
```

4. 验证

```bash
hexo clean
hexo generate
hexo server -d
```

访问 [http://127.0.0.1:4000/admin](http://127.0.0.1:4000/admin) 登录。

![](https://oscimg.oschina.net/oscnet/5445c69f50fbd0f17e8808e8f4a7a036c3a.jpg)

---

![](https://oscimg.oschina.net/oscnet/a71adf4fdab802378e4657bdb8f6baf0284.jpg)

---

- [Hexo 教程合集](https://mp.weixin.qq.com/s/UIlKCAMJsV1B0tfT0Bvklg)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)