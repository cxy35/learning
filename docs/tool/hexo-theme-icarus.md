---
title: Hexo 主题配置 - Icarus
date: 2020-01-20 16:46:04
categories: Hexo
tags: [Hexo, Icarus]
toc: true
# thumbnail: /images/hexo/icarus-thumbnail.png
---
hexo-theme-icarus 主题配置大全，图文并茂，持续更新中。
<!-- more -->

官方效果图：![](https://oscimg.oschina.net/oscnet/up-13d2d39a8b06e29b618c516c45b1c76bd8c.png)

我的站点：[https://cxy35.com](https://cxy35.com)

## 1 安装 Icarus 主题

参考 [hexo-theme-icarus 主题官网](https://blog.zhangruipeng.me/hexo-theme-icarus/)

### 1.1 下载 Icarus 主题

建议你使用 克隆最新版本 的方式，之后的更新可以通过 git pull 来快速更新， 而不用再次下载压缩包替换。

```bash
cd blog
git clone https://github.com/ppoffice/hexo-theme-icarus.git themes/icarus
```

### 1.2 启用 Icarus 主题

与所有 Hexo 主题启用的模式一样。 当 克隆/下载 完成后，打开 **`站点配置文件`**， 找到 `theme` 字段，并将其值更改为 `icarus`。

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: icarus
```

到此，Icarus 主题安装完成。下一步我们将验证主题是否正确启用。在切换主题之后、验证之前， 我们最好使用 `hexo clean` 来清除 Hexo 的缓存。

## 2 配置 Icarus 主题

### 2.1 设置站点信息

编辑 **`站点配置文件`**"，具体配置参考 [Hexo 搭建个人博客网站](https://mp.weixin.qq.com/s/2Spv9KHO6mLHui0CFBmCnQ) 。

### 2.2 设置页面文章的篇数

编辑 **`站点配置文件`**，具体配置参考 [Hexo 搭建个人博客网站](https://mp.weixin.qq.com/s/2Spv9KHO6mLHui0CFBmCnQ) 。

### 2.3 主题配置文件

编辑 **`主题配置文件`**，具体配置参考 `themes/icarus/_config.yml`。

```yaml
# ......
```

### 2.4 网站访问量与访客量统计

1. 编辑 **`主题配置文件`**，修改配置如下：

```yaml
# BuSuanZi site/page view counter
# https://busuanzi.ibruce.info
# 网站访问量与访客量统计
busuanzi: true
```

2. 编辑 `/themes/icarus/layout/common/footer.ejs` 文件，修改如下：

```html
# 注释掉原来的
<!-- <% if (busuanzi) { %>
<br>
<span id="busuanzi_container_site_uv">
<%- _p('plugin.visitor', '<span id="busuanzi_value_site_uv">0</span>') %>
</span>
<% } %> -->

# 新增新的，换了显示的位置
<div style="text-align: center;">
    <br>
    <p class="is-size-7">
        <% if (busuanzi) { %>
        <span id="busuanzi_container_site_uv">
            <i class="fa fa-user"></i> 本站访客数 <span id="busuanzi_value_site_uv"></span> 人次
        </span>
        <span id="busuanzi_container_site_pv">
            |  <i class="fa fa-eye"></i> 本站总访问量 <span id="busuanzi_value_site_pv"></span> 次
        </span>
        <% } %>
    </p>
</div>
```

### 2.5 网站运行时间统计

编辑 `/themes/icarus/layout/common/footer.ejs` 文件，修改如下：

```html
<span id="timeDate">载入天数...</span><span id="times">载入时分秒...</span>
<script>
    var now = new Date(); 
    function createtime() { 
        var grt= new Date("1/2/2020 08:00:00");// 此处修改你的建站时间或者网站上线时间 
        now.setTime(now.getTime()+250); 
        days = (now - grt ) / 1000 / 60 / 60 / 24; dnum = Math.floor(days); 
        hours = (now - grt ) / 1000 / 60 / 60 - (24 * dnum); hnum = Math.floor(hours); 
        if(String(hnum).length ==1 ){hnum = "0" + hnum;} minutes = (now - grt ) / 1000 /60 - (24 * 60 * dnum) - (60 * hnum); 
        mnum = Math.floor(minutes); if(String(mnum).length ==1 ){mnum = "0" + mnum;} 
        seconds = (now - grt ) / 1000 - (24 * 60 * 60 * dnum) - (60 * 60 * hnum) - (60 * mnum); 
        snum = Math.round(seconds); if(String(snum).length ==1 ){snum = "0" + snum;} 
        document.getElementById("timeDate").innerHTML = "本站已安全运行 "+dnum+" 天 "; 
        document.getElementById("times").innerHTML = hnum + " 小时 " + mnum + " 分 " + snum + " 秒"; 
    } 
    setInterval("createtime()",250);
</script>
```

### 2.6 侧边栏社交链接

编辑 **`主题配置文件`**，具体配置参考文件中 `widgets - profile - social_links`。

### 2.7 文章首页/详情页显示文章缩略图

编辑 **`主题配置文件`**，具体配置参考文件中 `article - thumbnail`。

需要对应的文章顶部 Front-matter 中需要增加参数 `thumbnail: /gallery/thumbnails/desert.jpg`。

```yaml
---
title: Getting Started with Icarus
thumbnail: /gallery/thumbnails/desert.jpg
---
Post content...
```

### 2.8 文章详情页时在侧边栏显示目录

编辑 **`主题配置文件`**，具体配置参考文件中 `widgets - profile - toc`。

文章详情页时在侧边栏显示，需要对应的文章顶部 Front-matter 中需要增加参数 `toc: true`。

```yaml
---
title: Table of Contents Example
toc: true
---
Post content...
```

### 2.9 文章详情页和 page 页布局修改

由于 Icarus 主题的文章详情页和 page 页默认与主页布局一致，皆为三栏布局。但是三栏布局限制了文章内容的展示，因此试图将其改为两栏布局。

1. 编辑 `/themes/icarus/layout/layout.ejs` 文件，修改如下：

```html
# 新增 col() 函数
<% function col() {
    if(!is_post() && !is_page()) {
        return main_column_class();
    } else {
        return 'is-6-tablet is-6-desktop is-9-widescreen';
    } 
} %>

# 修改 section ，将 main_column_class() 改为 col()
<section class="section">
    <div class="container">
        <div class="columns">
            <!-- 将 main_column_class() 改为 col() -->
            <div class="column <%= col() %> has-order-2 column-main"><%- body %></div>
            <%- partial('common/widget', { position: 'left' }) %>
            <%- partial('common/widget', { position: 'right' }) %>
        </div>
    </div>
</section>
```

通过上面的修改，发现文章详情页的文章栏确实放大了，但是右侧的部件栏被挤出了屏幕外一部分，极不美观。

2. 编辑 `/themes/icarus/layout/common/widget.ejs` 文件，修改如下：

```html
<!-- 原始代码，处理非 post 页面和非 page 页面文章详情页 -->
<% if (get_widgets(position).length && !is_post() && !is_page()) { %> <!-- 修改，增加  && !is_post() && !is_page() 判断 -->
<% function side_column_class() {
    switch (column_count()) {
        case 2:
            return 'is-4-tablet is-4-desktop is-4-widescreen';
        case 3:
            return 'is-4-tablet is-4-desktop is-3-widescreen';
    }
    return '';
} %>
<% function visibility_class() {
    if (column_count() === 3 && position === 'right') {
        return 'is-hidden-touch is-hidden-desktop-only';
    }
    return '';
} %>
<% function order_class() {
    return position === 'left' ? 'has-order-1' : 'has-order-3';
} %>
<% function sticky_class(position) {
    return get_config('sidebar.' + position + '.sticky', false) ? 'is-sticky' : '';
} %>
<div class="column <%= side_column_class() %> <%= visibility_class() %> <%= order_class() %> column-<%= position %> <%= sticky_class(position) %>">
    <% get_widgets(position).forEach(widget => {%>
        <%- _partial('widget/' + widget.type, { widget }) %>
    <% }) %>
    <% if (position === 'left') { %>
        <div class="column-right-shadow is-hidden-widescreen <%= sticky_class('right') %>">
        <% get_widgets('right').forEach(widget => {%>
            <%- _partial('widget/' + widget.type, { widget }) %>
        <% }) %>
        </div>
    <% } %>
</div>
<% } %>

<!-- 粘贴的部分，处理 post 页面和 page 页面 -->
<% if (position === 'left' && (is_post() || is_page())) { %> <!-- 修改，可选保留的栏，这里是左栏 -->
<% function side_column_class() {
    switch (column_count()) {
        case 2:
            return 'is-4-tablet is-4-desktop is-4-widescreen';
        case 3:
            return 'is-4-tablet is-4-desktop is-3-widescreen';
    }
    return '';
} %>
<% function visibility_class() {
    if (column_count() === 3 && position === 'right') {
        return 'is-hidden-touch is-hidden-desktop-only';
    }
    return '';
} %>
<% function order_class() {
    return position === 'left' ? 'has-order-1' : 'has-order-3';
} %>
<% function sticky_class(position) {
    return get_config('sidebar.' + position + '.sticky', false) ? 'is-sticky' : '';
} %>
<div class="column <%= side_column_class() %> <%= visibility_class() %> <%= order_class() %> column-<%= position %> <%= sticky_class(position) %>">
    <% get_widgets(position).forEach(widget => {%>
        <%- _partial('widget/' + widget.type, { widget }) %>
    <% }) %>
    <% if (position === 'left') { %>
        <div class="column-right-shadow is-hidden-widescreen <%= sticky_class('right') %>">
        <% get_widgets('right').forEach(widget => {%>
            <%- _partial('widget/' + widget.type, { widget }) %>
        <% }) %>
        </div>
    <% } %>
</div>
<% } %>
```

### 2.10 文章详情页分享功能

编辑 **`主题配置文件`**，具体配置参考文件中 `share - type - ...`。

[官方分享插件汇总](https://blog.zhangruipeng.me/hexo-theme-icarus/categories/Plugins/Share/)

### 2.11 文章详情页评论功能

编辑 **`主题配置文件`**，具体配置参考文件中 `comment - type - ...`。

[官方评论插件汇总](https://blog.zhangruipeng.me/hexo-theme-icarus/categories/Plugins/Comment/)

#### Gitment

**注意：开启文章的 Gitment 评论需要在对应的文章详情页登录后点击 `Initialize Comments` 按钮。当然也可用脚本自动触发。**

- 问题1：文章详情页登录时报错，跳到：

```bash
https://yoursite.com/?error=redirect_uri_mismatch&error_description=The+redirect_uri+MUST+match+the+registered+callback+URL+for+this+application.&error_uri=https%3A%2F%2Fdeveloper.github.com%2Fapps%2Fmanaging-oauth-apps%2Ftroubleshooting-authorization-request-errors%2F%23redirect-uri-mismatch
```
 
原因1：**请求地址 url 中的文件名有空格（标题没影响），可以用 - 隔开。**  
原因2：申请或配置 GitHub - OAuth Apps 时 **Authorization callback URL** 的值填写错误，比如 http 和 https 等。

- 问题2：文章详情页登录授权之后回调提示 `[object ProgressEvent]`

编辑 `/themes/icarus/layout/comment/gitment.ejs` 文件，修改如下：

```html
# 搜索
<link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
# 修改成：
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/theme-next/theme-next-gitment@1/default.css"/>
<script src="https://cdn.jsdelivr.net/gh/theme-next/theme-next-gitment@1/gitment.browser.js"></script>
# 或
<link rel="stylesheet" href="https://wzxjayce.github.io/gitment.css">
<script src="https://wzxjayce.github.io/gitment.js"></script>
# 或（汉化）
<link rel="stylesheet" href="https://billts.site/extra_css/gitment.css">
<script src="https://billts.site/js/gitment.js"></script>
```

### 2.12 文章详情页结尾处增加固定的内容

编辑 `/themes/icarus/layout/common/articles.ejs` 文件，修改如下：

```html
# 在标签 tag 的表单之后增加自己的表单
<% if (!index && post.layout === 'post') { %>
<div class="level is-size-6" style="margin-bottom:0px">
    <p>扫码关注微信公众号 <b>程序员35</b> ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。</p>
</div>
<div class="level is-size-7">
    <img src="/images/config/cxy35.jpg">
</div>
<% } %>
```

### 2.13 TODO - 添加雪花飘落效果

### 2.14 TODO - 鼠标点击特效

### 2.15 TODO - 看板娘插件

---

- [Hexo 教程合集](https://mp.weixin.qq.com/s/UIlKCAMJsV1B0tfT0Bvklg)（微信左下方**阅读全文**可直达）。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)