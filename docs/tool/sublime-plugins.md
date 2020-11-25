---
title: Sublime Text 常用插件
date: 2020-11-02 14:15:36
categories: IDE
tags: [IDE, Sublime, 插件]
toc: true
---
工欲善其事，必先利其器。本文收集 Sublime Text 常用插件，持续更新中......
<!-- more -->

`Package Control` 为插件管理包，通过快捷键 `Ctrl + Shift + P` 打开，之后可以很方便的浏览、安装和卸载 Sublime Text 中的插件。

- 插件安装：输入 `install` 后选择 `Package Control: Install Package`
- 插件列表：输入 `list` 后选择 `Package Control: List Packages`
- 插件删除：输入 `remove` 后选择 `Package Control: Remove Packages`

## ConvertToUTF8

- 功能说明：ConvertToUTF8 能将除 UTF8 编码之外的其他编码文件在 Sublime Text 中转换成 UTF8 编码，在打开文件的时候一开始会显示乱码，然后一刹那就自动显示出正常的字体，当然，在保存文件之后原文件的编码格式不会改变。

## BracketHighlighter

- 功能说明：高亮显示匹配的括号、引号和标签。
- 插件地址：[https://github.com/facelessuser/BracketHighlighter/tree/BH2ST3](https://github.com/facelessuser/BracketHighlighter/tree/BH2ST3)

## Emmet

- 功能说明：Emmet 的前身是大名鼎鼎的 `Zen codin`。前端开发必备，HTML、CSS 代码快速编写神器。
- 使用方法：默认快捷键 `Tab`
- 插件地址：[https://github.com/sergeche/emmet-sublime](https://github.com/sergeche/emmet-sublime)
- 辅助工具：`PyV8` 下载地址：[https://github.com/emmetio/pyv8-binaries](https://github.com/emmetio/pyv8-binaries)
- 注意：Emmet 插件需要 PyV8 插件的支持，所以在安装 Emmet 时，会自动安装 PyV8 插件，如果安装后 Emmet 不能正常使用，很有可能是因为 PyV8 没有安装完全，Sublime Text 2 和 3 容易出现这个问题。你可以删除它，然后手动下载，采用方法二安装 PyV8 插件。
- 使用方法示例：书写代码 `ul#nav>li.item$*8>a{Item $}`，然后把光标定在这行代码的最后面，按 Tab 键，就会自动生成。更多更详细的使用方法，请查阅 Emmet 官网：[http://docs.emmet.io/](http://docs.emmet.io/)

## JsFormat

- 功能说明：JavaScript 代码格式化。
- 使用方法：在打开的 JavaScript 文件里点右键，选择 `JsFormat`。
- 插件地址：[https://github.com/jdc0589/jsformat](https://github.com/jdc0589/jsformat)

## Compact Expand CSS Command

- 功能说明：使 CSS 属性展开及收缩，格式化 CSS 代码。
- 使用方法：按 `Ctrl+Alt+[` 收缩 CSS 代码为一行显示，按` Ctrl+Alt+]` 展开 CSS 代码为多行显示。
- 插件地址：[https://gist.github.com/vitaLee/2863474](https://gist.github.com/vitaLee/2863474) 或者：[https://github.com/TooBug/CompactExpandCss](https://github.com/TooBug/CompactExpandCss)

## Color Highlighter

- 功能说明：显示所选颜色值的颜色，并集成了 `ColorPicker`
- 使用方法：在 16 进制的颜色值上点右键，选择 `Choose color` ，会弹出颜色拾色器，在需要的色块上单击。
- 插件地址：[https://github.com/Monnoroch/ColorHighlighter](https://github.com/Monnoroch/ColorHighlighter)

## SublimeTmpl

- 功能说明：快速生成文件模板。
- 使用方法：SublimeTmp l默认的快捷键如下，如果快捷键设置冲突可能无效。
    - `Ctrl+Alt+H`：新建 `html` 文件
    - `Ctrl+Alt+J`：新建 `javascript` 文件
    - `Ctrl+Alt+C`：新建 `css` 文件
    - `Ctrl+Alt+P`：新建 `php` 文件
    - `Ctrl+Alt+R`：新建 `ruby` 文件
    - `Ctrl+Alt+Shift+P`：新建 `python` 文件
- 插件地址：[https://github.com/kairyou/SublimeTmpl](https://github.com/kairyou/SublimeTmpl)
- 新增语言：你还可以增加模板文件夹中没有的文件模板，并做相应的设置来使用这一功能。具体可以参考它的中文文档：[http://www.fantxi.com/blog/archives/sublime-template-engine-sublimetmpl/](http://www.fantxi.com/blog/archives/sublime-template-engine-sublimetmpl/)

## Alignment

- 功能说明：使代码格式的自动对齐。
- 使用方法：快捷键 `Ctrl+Alt+A`，可能与 QQ 截图冲突，二者中的一个要重置快捷键。
- 插件地址：[https://github.com/kevinsperrine/sublime_alignment])(https://github.com/kevinsperrine/sublime_alignment)

## AutoFileName

- 功能说明：自动补全文件（目录）名。
- 使用方法：安装好后就可以来测试如何使用 AutoFileName，先以 `<link href="">` 档案来示范，当输入 `href=""` 的同时，Sublime Text 就会将现在编辑档案的路径为中心，判断该路径内的所有档案。其他的也类似，如：`<img src="">` 等。
- 插件地址：[https://github.com/BoundInCode/AutoFileName](https://github.com/BoundInCode/AutoFileName)

## DocBlockr

- 功能说明：快速生成 `JavaScript (including ES6), PHP, ActionScript, Haxe, CoffeeScript, TypeScript, Java, Groovy, Objective C, C, C++ and Rust` 语言函数注释。
- 使用方法：在函数上面输入 `/**` ,然后按 `Tab` 就会自动生成注释。
- 插件地址：[https://github.com/spadgos/sublime-jsdocs](https://github.com/spadgos/sublime-jsdocs)

## SublimeCodeIntel

- 功能说明：智能提示。
- 插件地址：[https://github.com/SublimeCodeIntel/SublimeCodeIntel](https://github.com/SublimeCodeIntel/SublimeCodeIntel)

## HTML-CSS-JS Prettify

- 功能说明：`HTML、CSS、JS` 格式化。
- 插件地址：[https://github.com/victorporof/Sublime-HTMLPrettify](https://github.com/victorporof/Sublime-HTMLPrettify)
- 安装方法：安裝这个套件前必须先安裝 `node.js`，指定 `node.exe` 的执行档所在位置。进而安装 `HTML-CSS-JS Prettify`。
- 使用方法一：`View -> Show console` 或者使用快捷键（Ctrl + `），在命令列的地方輸入：`view.run_command("htmlprettify")`，然后按下 `Enter`。
- 使用方法二：默认快捷键：`Ctrl+Shift+H`。你也可以自行设置快捷键，如：设置成 `Ctrl+Shfit+O`，选择菜单 `Preferences---> Key Bindings – User` 里新增：
    ```
    { 
        "keys": ["ctrl+shift+o"], 
        "command": "htmlprettify" 
    }
    ```

## SideBarEnhancements

功能说明：侧栏菜单扩充功能。
插件地址：[https://github.com/titoBouzout/SideBarEnhancements/tree/st3](https://github.com/titoBouzout/SideBarEnhancements/tree/st3)

## View In Browser

- 功能说明：Sublime Text 保存后网页自动同步更新。
- 插件地址：[https://github.com/adampresley/sublime-view-in-browser](https://github.com/adampresley/sublime-view-in-browser)
- 使用方法：在打开的文档任一处点右键，选择 `View In Browser`，就会用默认的浏览器自动打开该文件。同时 Chrome 浏览器也要安装 `LiveReload` 的扩展插件。

## LiveReload

- 功能说明：调试网页实时自动更新。
- 使用说明：快捷键 `Ctr+Alt+V`
- 插件地址：[https://github.com/dz0ny/LiveReload-sublimetext2](https://github.com/dz0ny/LiveReload-sublimetext2)

## TortoiseSVN

- 功能说明：版本控制工具。
- 插件地址：[https://github.com/dexbol/sublime-TortoiseSVN](https://github.com/dexbol/sublime-TortoiseSVN)
- 其他说明：win 下需要安装有 TortoiseSVN 客户端支持

## Theme-Soda

- 功能说明：最受欢迎的 Sublime Text 主题之一。
- 使用说明：安装完成后，点菜单 `Preferences -> Settings - User`，根据需要的主题效果，添加对应的配置。
- 插件地址：[https://github.com/buymeasoda/soda-theme](https://github.com/buymeasoda/soda-theme)

## Theme-Flatland

- 功能说明：最受欢迎的 Sublime Text 主题之一。
- 插件地址：[https://github.com/thinkpixellab/flatland](https://github.com/thinkpixellab/flatland)

## Theme-Nexus

- 功能说明：最受欢迎的 Sublime Text 主题之一。
- 插件地址：[https://github.com/EleazarCrusader/nexus-theme](https://github.com/EleazarCrusader/nexus-theme)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)