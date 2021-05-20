工欲善其事，必先利其器。本文收集 IntelliJ IDEA 常用插件，持续更新中......
<!-- more -->

## Alibaba Java Coding Guidelines

> 阿里巴巴《Java 开发手册》配套插件，可以实时检测代码中不符合手册规约的地方，助你码出高效，码出质量。

使用：

- 当我们违反手册规约时，该插件会自动检测并进行提示。
- 同时提供了一键检测所有代码规约情况和切换语言的功能。
- 如果你想修改某条规约的检测规则的话，可以通过设置的 `Editor -> Inspections` 进行修改。

## Codota AI Autocomplete for Java and JavaScript

> 查询代码使用示例。

使用：

- 在对应的代码上右键选择 `Get relevant examples` 或 `Ctrl + Shift + O` 快捷键，会展示很多对应代码的用法。
- 例子：当我们用 `stream().filter()` 对 `List` 操作，可是对 `filter()` 用法不熟，按常理我们会百度一下，而用 `Codota` 会提示很多 `filter()` 用法，节省不少查阅资料的时间。

## Easy Code

> EasyCode 用于**代码自动生成**，支持模板自定义、导入、导出，方便团队之间共享。

介绍：

- 基于 IntelliJ IDEA 开发的代码生成插件，支持自定义任意模板（Java，html，js，xml）。
- 只要是与数据库相关的代码都可以通过自定义模板来生成。支持数据库类型与 java 类型映射关系配置。
- 支持同时生成生成多张表的代码。每张表有独立的配置信息。完全的个性化定义，规则由你设置。

具体使用见：[IntelliJ IDEA 插件 EasyCode（代码自动生成）](https://mp.weixin.qq.com/s/aIUH2i0vnmsDUBQGzCwkjw)

## Lombok

> Lombok 为 Java 项目提供了非常有趣的附加功能，使用它的注解可以有效的地解决那些繁琐又重复的代码，如: Setter、Getter、toString、equals、hashCode 以及非空判断等。

使用：

比如给一个类添加 `@Getter` 和 `@Setter` 注解， Lombok 就会为我们自动生成所有属性的 Getter 和 Setter 方法。

## Free MyBatis plugin

> MyBatis 扩展插件，可以在 Mapper 接口的方法和 xml 实现之间自由跳转，也可以用来一键生成某些 xml 实现。

介绍：

- 生成 mapper xml 文件。
- 快速从代码跳转到 mapper 及从 mapper 返回代码。
- mybatis 自动补全及语法错误提示。
- 集成 mybatis generator gui 界面。

使用：

- 通过 `Alt+Enter` 生成新方法的 xml 实现。
- 通过 Mapper 接口中方法左侧的箭头直接跳转到对应的 xml 实现中去。
- 通过 xml 中 Statement 左侧的箭头直接跳转到对应的 Mapper 接口方法中去。

## MyBatis Log Plugin

> 可以把 Mybatis 输出的 SQL 日志还原成完整的 SQL 语句，无需手动转换。

使用：

- 打开这款插件的窗口，当控制台输出 Mybatis 的 SQL 日志时，该插件会自动帮我们转换成对应的 SQL 语句。
- 有的时候我们需要转换的日志并不在自己的控制台上，这时可以使用插件的 `SQL Text` 功能：直接复制我们需要转换的日志，然后点击 `Restore Sql` 按钮即可。

## RestfulToolkit

> 一套 Restful 服务开发辅助工具集，提供了项目中的接口概览信息，可以根据 URL 跳转到对应的接口方法中去，内置了 HTTP 请求工具，对请求方法做了一些增强功能，总之功能很强大！

介绍：

- 根据 URL 直接跳转到对应的方法定义 ( Ctrl \ or Ctrl Alt N );
- 提供了一个 Services tree 的显示窗口;
- 一个简单的 http 请求工具;
- 在请求方法上添加了有用功能: 复制生成 URL；复制方法参数...
- 其他功能: java 类上添加 Convert to JSON 功能，格式化 json 数据 ( Windows: Ctrl + Enter; Mac: Command + Enter )。

使用：

- 右上角的 RestServices 按钮可以显示项目中接口的概览信息。
- 可以通过搜索按钮根据 URL 搜索对应接口。
- 可以通过底部的 HTTP 请求工具来发起接口测试请求。
- 通过在接口方法上右键可以生成查询参数、请求参数、请求 URL 。
- 通过在实体类上右键可以直接生成实体类对应的 JSON 。

## GsonFormat

> 可以把 JSON 格式的字符串转化为实体类。

使用：

- 先创建一个实体类，然后在类名上右键选择 `Generate -> GsonFormat` 或 `Atl + S` 快捷键。
- 再输入我们需要转换的 JSON 字符串，选择性的更改属性名称和类型，点击确定后直接生成实体类。

## GenerateAllSetter

> 实际的开发中，可能会经常为某个对象中多个属性进行 `set` 赋值，尽管可以用 `BeanUtil.copyProperties()` 方式批量赋值，但这种方式有一些弊端，存在属性值覆盖的问题，所以不少场景还是需要手动 `set`。如果一个对象属性太多 `set` 起来也很痛苦，`GenerateAllSetter` 可以一键将对象属性都 `set` 出来。

- 快捷键：`Alt + Enter`

## Convert YAML and Properties File

> `YAML` 和 `Properties` 类型的配置文件之间互相转换。

## Maven Helper

> 解决 Maven 依赖冲突，可以快速查找项目中的依赖冲突，并予以解决。

使用：

- 可以通过 pom.xml 文件底部的依赖分析标签页 `Dependency Analyzer` 查看当前项目中的所有依赖。
- 通过冲突按钮我们可以筛选出所有冲突的依赖，选中有冲突的依赖，点击 `Exclude` 按钮可以直接排除该依赖。
- 同时 pom.xml 中也会对该依赖添加 `<exclusion>` 标签。

## Translation

> 翻译插件，支持 Google、有道、百度翻译，看源码时看注释很有帮助。

使用：

- 选中需要翻译的内容，右键翻译。
- 翻译整个文档。
- 右上角的翻译按钮翻译指定内容。

## Grep Console

> 分析控制台日志，对不同级别的日志进行不同颜色的高亮显示，还可以用来按关键字搜索日志内容。

使用：

- 打印日志时不同日志级别的日志会以不同颜色来显示。
- 可以通过 Tools 打开该插件的配置菜单，然后通过配置菜单修改配色方案。
- 可以通过在控制台右键并使用 Grep 按钮来调出日志分析的窗口，然后直接通过关键字来搜索即可。

## Statistic

> 代码统计工具，可以用来统计当前项目中代码的行数和大小。

使用：

- 可以通过顶部菜单中的 `View -> Tool Windows -> Statistic` 按钮开启该功能。
- 此时就可以看到我们项目代码的统计情况了。

## Vue.js

> Vue.js 支持插件，可以根据模板创建 .vue 文件，也可以对 Vue 相关代码进行智能提示。

使用：

- 启用该插件后，可以根据模板新建 .vue 文件。
- 当我们在标签中写入以 `v-` 开头的代码时，会提示 `Vue` 中的相关指令。

## element

> Element-UI 支持插件，可以对 Element-UI 中的标签进行智能提示。

使用：

- 当我们写入以 `el-` 开头的标签时，会提示 `Element-UI` 相关组件。


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)